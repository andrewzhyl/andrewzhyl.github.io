---
layout: post
title:  "Fan-in,Fan-out 模式"
date:   2021-08-09
description: 'Fan-in,Fan-out 模式'
category: notes
---

**Fan-out**：多个 goroutine 从同一个通道读取数据，直到该通道关闭。OUT 是一种张开的模式，所以又被称为扇出，可以用来分发任务。

**Fan-in**：1 个 goroutine 从多个通道读取数据，直到这些通道关闭。IN 是一种收敛的模式，所以又被称为扇入，用来收集处理的结果。

<img src="/assets/images/blog/fan-out-and-fan-in.png" />  

实现示例：

```go
package main

import (
	"context"
	"log"
	"sync"
	"time"
)

// Task 包含任务编号及任务所需时长
type Task struct {
	Number int
	Cost   time.Duration
}

// task channel 生成器
func taskChannelGerenator(ctx context.Context, taskList []Task) <-chan Task {
	taskCh := make(chan Task)

	go func() {
		defer close(taskCh)
		for _, task := range taskList {
			select {
			case <-ctx.Done():
				return
			case taskCh <- task:
			}
		}
	}()
	return taskCh
}

// doTask 处理并返回已处理的任务编号作为通道的函数
func doTask(ctx context.Context, taskCh <-chan Task) <-chan int {
	doneTaskCh := make(chan int)
	go func() {
		defer close(doneTaskCh)
		for task := range taskCh {
			select {
			case <-ctx.Done():
				return
			default:
				log.Printf("do task number: %d\n", task.Number)
				// task 任务处理
				// 根据任务耗时休眠
				time.Sleep(task.Cost)
				doneTaskCh <- task.Number // 已处理任务的编号放入通道
			}
		}
	}()
	return doneTaskCh
}

// `fan-in` 意味着将多个数据流复用或合并成一个流。
// merge 函数接收参数传递的多个通道 “taskChs”,并返回单个通道 “<-chan int”
func merge(ctx context.Context, taskChs []<-chan int) <-chan int {
	var wg sync.WaitGroup
	mergedTaskCh := make(chan int)

	mergeTask := func(taskCh <-chan int) {
		defer wg.Done()
		for t := range taskCh {
			select {
			case <-ctx.Done():
				return
			case mergedTaskCh <- t:
			}
		}
	}

	wg.Add(len(taskChs))
	for _, taskCh := range taskChs {
		go mergeTask(taskCh)
	}
	// 等待所有任务处理完毕
	go func() {
		wg.Wait()
		close(mergedTaskCh)
	}()
	return mergedTaskCh
}

func main() {
	start := time.Now()
	// 使用 context 来防止 goroutine 泄漏，即使在处理过程中被中断
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// taskList 定义每个任务及其成本
	taskList := []Task{
		Task{1, 1 * time.Second},
		Task{2, 7 * time.Second},
		Task{3, 2 * time.Second},
		Task{4, 3 * time.Second},
		Task{5, 5 * time.Second},
		Task{6, 3 * time.Second},
	}

	// taskChannelGerenator 是一个函数，它接收一个 taskList 并将其转换为 Task 类型的通道
	// 执行结果（int slice channel）存储在 worker 中
	// 由于 doTask 的结果是一个通道，被分给了多个 worker，这就对应了 fan-out 处理
	taskCh := taskChannelGerenator(ctx, taskList)

	numWorkers := 4
	workers := make([]<-chan int, numWorkers)
	for i := 0; i < numWorkers; i++ {
		workers[i] = doTask(ctx, taskCh)  // doTask 处理并返回已处理的任务编号作为通道的函数
	}

	count := 0
	for d := range merge(ctx, workers) { // merge 从中读取已处理的任务编号
		count++
		log.Printf("done task number: %d\n", d)
	}
	log.Printf("Finished. Done %d tasks. Total time: %fs", count, time.Since(start).Seconds())
}
```
---
layout: post
title:  "go 项目中类似 sidekiq 的消息队列 Asynq"
date:   2021-12-05
description: 'go 项目中类似 sidekiq 的消息队列 Asynq'
category: notes

---

**温馨提示：仅为个人笔记以免遗忘，不保证代码完整**

之前在 ruby 项目中做异步任务，一直使用 sidekiq，sidekiq 依赖 Redis 实现队列任务的增加、重试以及调度，可以算是穷人版的消息队列吧，毕竟不需要付费买生产版本已经够用了。<br>

换用 golang 之后一直想找个替代品，毕竟像 sidekiq 一样简单高效并不是那么容易的，附加的学习和使用成本也很高<br>
最近发现了一个很好的 Go 简单高效的异步任务处理库：[Asynq](https://github.com/hibiken/asynq), 开发自谷歌员工。将其引入的项目内，已经基本能满足我的需求了<br>

## asynq 的特性：

- 保证至少执行一次任务
- 任务调度
- redis 持久化
- 失败重试
- worker 崩溃自动恢复
- 加权的优先级队列
- redis 缓存，低延迟
- 支持任务的超时设置
- 支持任务的唯一限制
- 支持中间件接口函数
- 支持暂停或停止任务
- 支持定时任务
- 提供 web ui 管理
- 提供 cli 管理


## 安装 asynq

首先需要安装 redis，并正常运行

```bash
brew install redis
brew services  start redis

## 以下在项目目录下执行
go get -u github.com/hibiken/asynq
go get -u github.com/hibiken/asynq/tools/asynq #命令行工具
```

main.go Asynq 服务端 worker.go 处理程序 asynq_test.go 模拟客户端使用

## asynq.yml 配置

```yaml
asynq:
  host: 127.0.0.1                # redis 地址
  port: 6379                     # redis 端口
  db: 0                          # redis db
  password: ""                   # redis password
  poolSize: 30                   # 连接池
  concurrency: 10                # 并发数
  queues:
    - default: 5                  # 默认队列
    - pm_mailers: 6               # 邮件发送任务
    - low: 1                      # 测试队列
```

## asynq server

```go
package gasynq

import (
	"fmt"
	"goapp/internal/pkg/log"
	"sync"
	"time"

	"github.com/gookit/goutil/mathutil"
	"github.com/hibiken/asynq"
	"github.com/pkg/errors"
	"github.com/spf13/viper"
	"go.uber.org/zap"
)

type Config struct {
	Host        string
	Port        int
	Db          int
	Password    string
	PoolSize    string
	Concurrency int
	Queues      map[string]int
}

func NewConfig(v *viper.Viper) (*Config, error) {
	var (
		err error
		o   = new(Config)
	)
	if err = v.UnmarshalKey("asynq", o); err != nil {
		return nil, err
	}

	return o, err
}

type Entry struct {
	Pattern string
	Handler asynq.Handler
}

type AsynqServer struct {
	*asynq.Server
	logger  log.ILogger
	entries []Entry
}

var (
	asynqServer *AsynqServer
	serverOnce  sync.Once
)

func GetAsyncServer() *AsynqServer {
	return asynqServer
}

func StopServer() {
	asynqServer.Stop()
}

func NewServer(v *viper.Viper,
	logger log.ILogger, entries []Entry) (*AsynqServer, error) {
	o, err := NewConfig(v)
	if err != nil {
		return nil, errors.Wrap(err, "unmarshal redis option error")
	}
	logger.Infof("load asynq redis options success!: host:%s port:%d db:%d", o.Host, o.Port, o.Db)

	redis := asynq.RedisClientOpt{
		Addr:         fmt.Sprintf("%s:%d", o.Host, o.Port),
		Password:     o.Password,
		DB:           o.Db,
		PoolSize:     mathutil.MustInt(o.PoolSize),
		DialTimeout:  10 * time.Second,
		ReadTimeout:  30 * time.Second,
		WriteTimeout: 30 * time.Second,
	}

	serverOnce.Do(func() {
		asynqServer = &AsynqServer{
			asynq.NewServer(
				redis,
				asynq.Config{
					Concurrency: o.Concurrency,
					Queues:      o.Queues,
				},
			),
			logger.With(zap.String("type", "Asynq.Server")),
			entries,
		}
	})

	return asynqServer, nil
}

func SeverRun() error {
	if asynqServer == nil {
		return errors.New("async server is nil")
	}

	mux := asynq.NewServeMux()
	for _, entry := range asynqServer.entries {
		mux.Handle(entry.Pattern, entry.Handler)
	}
	if err := asynqServer.Run(mux); err != nil {
		asynqServer.logger.Fatalf("could not run server: %v", err)
		return err
	}
	return nil
}
```

## asynq client

```go
package gasynq

import (
	"fmt"
	"goapp/internal/pkg/log"
	"sync"
	"time"

	"github.com/gookit/goutil/mathutil"
	"github.com/hibiken/asynq"
	"github.com/pkg/errors"
	"github.com/spf13/viper"
)

var (
	asynqClient *asynq.Client
	once        sync.Once
)

func GetAsynqClient() *asynq.Client {
	return asynqClient
}

func ClostClient() error {
	return asynqClient.Close()
}

func NewClient(v *viper.Viper,
	logger log.ILogger) (*asynq.Client, error) {
	o, err := NewConfig(v)
	if err != nil {
		return nil, errors.Wrap(err, "unmarshal redis option error")
	}
	logger.Infof("load asynq redis options success!: host:%s port:%d asynqDb:%d", o.Host, o.Port, o.Db)

	redis := asynq.RedisClientOpt{
		Addr:         fmt.Sprintf("%s:%d", o.Host, o.Port),
		Password:     o.Password,
		DB:           o.Db,
		PoolSize:     mathutil.MustInt(o.PoolSize),
		DialTimeout:  10 * time.Second,
		ReadTimeout:  30 * time.Second,
		WriteTimeout: 30 * time.Second,
	}

	once.Do(func() {
		asynqClient = asynq.NewClient(redis)
	})

	return asynqClient, nil
}
```

## Helloword 任务示例


```go
package tasks

import (
	"context"
	"encoding/json"
	"fmt"
	"goapp/internal/pkg/app"
	"time"

	"github.com/hibiken/asynq"
)

const (
	TypeHelloword = "helloword"
)

type HelloWorldPayload struct {
	Username string
}

func NewHelloWorldTask(username string) (*asynq.Task, error) {
	payload, err := json.Marshal(HelloWorldPayload{Username: username})
	if err != nil {
		return nil, err
	}
	// task options can be passed to NewTask, which can be overridden at enqueue time.
	return asynq.NewTask(TypeHelloword, payload, asynq.MaxRetry(5), asynq.Timeout(20*time.Minute), asynq.Queue("pm_mailers")), nil
}

// HelloWorldProcessor implements asynq.Handler interface.
type HelloWorldProcessor struct {
}

func (job *HelloWorldProcessor) ProcessTask(ctx context.Context, t *asynq.Task) error {
	var p HelloWorldPayload
	if err := json.Unmarshal(t.Payload(), &p); err != nil {
		return fmt.Errorf("json.Unmarshal failed: %v: %w", err, asynq.SkipRetry)
	}
	app.Logger().Infof("================HelloWorldProcessor: username=%s\n", p.Username)

	// 在这里完善代码...
	return nil
}

func NewHelloWorldProcessor() *HelloWorldProcessor {
	return &HelloWorldProcessor{}
}
```

server 的启动和 client 的使用，以及 Asynqmon web UI 的使用请参考 https://github.com/hibiken/asynq，这里不详述
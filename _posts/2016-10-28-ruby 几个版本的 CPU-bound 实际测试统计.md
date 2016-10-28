---
layout: post
title:  "ruby 几个版本的 CPU-bound 实际测试统计"
date:   2016-10-28 13:24:00
description: 'ruby 几个版本的 CPU-bound 实际测试统计'
---


ruby代码：

- 为了适用于 ree-1.8.7， 这里没有选择 1.9.3 及之后版本标准库的 `prime`, 因为那样速度会更快，比较的统计结果会没有意义

``` ruby
require 'benchmark'

def prime_sieve_upto(n)
  all_nums = (0..n).to_a
  all_nums[0] = all_nums[1] = nil
  all_nums.each do |p|

    #jump over nils
    next unless p

    #stop if we're too high already
    break if p * p > n

    #kill all multiples of this number
    (p*p).step(n, p){ |m| all_nums[m] = nil }
  end

  #remove unwanted nils
  all_nums.compact
end


primes = 1_000_000
iterations = 10
num_threads = 5
iterations_per_thread = iterations / num_threads

Benchmark.bm(15) do |x|
  x.report('single-threaded') do
    iterations.times do
      prime_sieve_upto(primes)
    end
  end
  x.report('multi-threaded') do
    num_threads.times.map do
      Thread.new do
        iterations_per_thread.times do
          prime_sieve_upto(primes)
        end
      end
    end.each(&:join)
  end
end

```

---

以下为统计输出结果：

ree-1.8.7-2012.02

```
                     user     system      total        real
single-threaded  5.660000   0.060000   5.720000 (  5.725174)
multi-threaded   6.110000   0.110000   6.220000 (  6.228208)
```

MRI ruby  1.9.3-p551

```
                      user     system      total        real
single-threaded   3.450000   0.060000   3.510000 (  3.531772)
multi-threaded    3.660000   0.080000   3.740000 (  3.760532)
```

MRI ruby 2.0.0-p598

```
                      user     system      total        real
single-threaded   3.630000   0.080000   3.710000 (  3.726324)
multi-threaded    3.680000   0.090000   3.770000 (  3.808694)
```

MRI ruby 2.0.0-p648

```
                      user     system      total        real
single-threaded   3.210000   0.060000   3.270000 (  3.276048)
multi-threaded    3.330000   0.080000   3.410000 (  3.402474)
```

MRI ruby  2.1.0

```
                      user     system      total        real
single-threaded   2.360000   0.070000   2.430000 (  2.422242)
multi-threaded    2.390000   0.070000   2.460000 (  2.462325)
```

MRI ruby  2.2.3：

```
                      user     system      total        real
single-threaded   2.300000   0.070000   2.370000 (  2.361750)
multi-threaded    2.410000   0.080000   2.490000 (  2.482332)
```

jruby-9.0.4.0：

``` 
                      user     system      total        real
single-threaded   7.740000   0.280000   8.020000 (  2.676519)
multi-threaded   11.760000   0.230000  11.990000 (  3.064823)
```

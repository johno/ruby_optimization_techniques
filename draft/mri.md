# What is the MRI/YARV

The Ruby MRI is short for Matz's Ruby Interpreter, and is the reference implementation for the Ruby programming language. It was released to the public in 1995, and is still actively developed, with the latest stable build being Ruby 2.1.1.

The YARV is an interpreter developed by Koichi Sasada that's also known as the KRI. It was developed in order to reduce the execution time of Ruby programs, and was very successful. As a result, YARV was merged into Ruby 1.9.0 and has replaced the MRI.

As the default interpreter for the Ruby programming language, the MRI has received it's fair share of criticism, primarily due to it's execution speeds and memory consumption.  However, recent Ruby versions have seen significant enhancements, and is on par with similar scripting languages like Python.  Not to mention the fact that some comparisons have the audacity to compare a compiled language to a scripting language, which is apples to oranges.  Developers typically choose Ruby for it's ease of writing/prototyping, understanding the fact that its execution time will _always_ be significantly slower than its compiled counterparts.

That being said, there are numerous methods and best practices that developers can follow in order to ensure that they're avoiding unnecessary bottlenecks.

## Performance out of the box

Thanks to the introduction of YARV, vanilla Ruby, on a single thread, has the ability to outperform other alternative Ruby implementations. Consider the following figure, that measures Rails requests per second.

![screen shot 2014-03-31 at 5 06 28 pm](https://cloud.githubusercontent.com/assets/1424573/2574040/8c8da974-b929-11e3-84c8-04d792bcbbd9.png)
http://www.isrubyfastyet.com/

## Global Interpreter Lock

When attempting to optimize execution speed, threads are often utilized in order to process tasks concurrently. This is a feature that Ruby supports, too. However, the MRI/YARV incorporates a Global Interpreter Lock, or GIL, that doesn't permit any true concurrency. A GIL refers to an interpreter thread that doesn't allow code that isn't thread safe to share itself with other threads. This results in little, to no, actual gain in speed when running threads on a multiprocessor machine.

![screen shot 2014-03-31 at 5 15 19 pm](https://cloud.githubusercontent.com/assets/1424573/2574079/5e0967fe-b92a-11e3-9806-65ea3d4d04cf.png)
http://www.igvita.com/2008/11/13/concurrency-is-a-myth-in-ruby/

### Why implement a GIL?

The primary reason is that the GIL is used to avoid race conditions within C extensions. There are also thread safety reasons, too. Parts of Ruby aren't thread safe (Hash), and numerous C libraries that are wrapped by Ruby's internals. Additionally, the GIL is integral to data integrity, because it ensures that the developer doesn't write any unsafe threading code.

This, interestingly enough, runs contrary to the fundamental principles of the Ruby language, where all the responsibility is laid on the developer. Ruby allows the developer to have the ultimate freedom without hand holding, yet the GIL is just that, hand holding.

### The GIL is here to stay

The GIL is deeply intertwined with Ruby and its internals, and many influential Ruby-core figures don't plan on removing the GIL anytime in the near future. Though, this doesn't mean the concurrency can't be achieved. 

### Sidestep the GIL with multiple virtual machines

Sasada Koichi has proposed a Multiple VM (MVM) solution, which is currently being developed. This would consist of multiple virtual machines, running their own processes, and communicate via sockets.

Granted, this is a drastic step away from typical threading, but some proponents believe that traditional threading isn't necessarily the correct paradigm to follow. Especially considering the fact the Ruby leverages green threads above the GIL rather than talking to the OS directly.

> Nevertheless, you're right the GIL is not as bad as you would initially think: you just have to undo the brainwashing you got from Windows and Java proponents who seem to consider threads as the only way to approach concurrent activities. Just because Java was once aimed at a set-top box OS that didn't support multiple address spaces, and just because process creation in Windows used to be slow as a dog, doesn't mean that multiple processes (with judicious use of IPC) aren't a much better approach to writing apps for multi-CPU boxes than threads.

## Some simple code enhancements

### String Optimization

String interpolation is significantly more performant than concatentation because it doesn't need to allocate new strings, it just modifies a single string in place.

```ruby
require 'benchmark'

concat_time = Benchmark.measure do
  20000000.times do
    str = 'str1' << 'str2' << 'str3'
  end
end

# => #<Benchmark::Tms:0x007fdf9ba49ea8 @label="", @real=79.152523, @cstime=0.0, @cutime=0.0, @stime=0.04000000000000001, @utime=79.11, @total=79.15> 

interp_time = Benchmark.measure do
  20000000.times do
    str = "#{ 'str1' }#{ 'str2' }#{ 'str3' }"
  end
end

# => #<Benchmark::Tms:0x007fdf9b990bd8 @label="", @real=22.713976, @cstime=0.0, @cutime=0.0, @stime=0.009999999999999995, @utime=22.689999999999998, @total=22.7>
```

### Blocks vs Procs

The `collect|map` methods with blocks are faster because it returns a new array rather than an enumerator. This can be leveraged to increase speed when compared to `Symbol.to_proc` implementations. Though, the latter is typically much more preferable to read. The reason that the `Symbol.to_proc` is slower is because `to_proc` is called on the symbol to perform the following conversion:

```ruby
:method.to_proc 
# => -> x { x.method }
```

```ruby
fake_data = 20.times.map { |t| Fake.new(t) }

proc_time = Benchmark.measure do
  200000000.times do
    fake_data.map(&:id)
  end
end

#  => #<Benchmark::Tms:0x007fdf9b8b0498 @label="", @real=491.332415, @cstime=0.0, @cutime=0.0, @stime=4.8, @utime=426.06999999999994, @total=430.86999999999995>

block_time = Benchmark.measure do
  200000000.times do
    fake_data.map { |d| d.id }
  end
end

# => #<Benchmark::Tms:0x007fdf9b931d40 @label="", @real=431.731424, @cstime=0.0, @cutime=0.0, @stime=2.66, @utime=416.21000000000004, @total=418.87000000000006>

collect_time = Benchmark.measure do
  200000000.times do
    fake_data.collect { |d| d.id }
  end
end

# => #<Benchmark::Tms:0x007fdf9b821518 @label="", @real=386.234513, @cstime=0.0, @cutime=0.0, @stime=1.1800000000000006, @utime=384.28, @total=385.46> 
 :037 >
```

### Modify Garbage Collection

```ruby
RUBY_HEAP_MIN_SLOTS=600000 # This is 60(!) times larger than default
RUBY_GC_MALLOC_LIMIT=59000000 # This is 7 times larger than default
RUBY_HEAP_FREE_MIN=100000 # This is 24 times larger than default
```

### Use Unicorn

For Ruby on Rails web applications, a server typically runs on a single process, which means that every request is processed one at a time. This can create a significant bottle neck in your application. Fortunately, there are libraries to incorporate concurrency in your application. One of which is Unicorn.

Unicorn uses Unix forks within a dyno (web worker) to create multiple instances of itself. Now, there are multiple OS instances that can all respond to requests, and complete tasks concurrently. This results in smaller queues, quicker responses, and a faster web application as a whole. The only drawback is memory usage, which can grow to large sizes. Though, with decreasing hardware costs, this becomes a worthwhile expenditure to ensure quick development time for the software components. This also doesn't require thread safe code, since each worker is a self-sufficient clone of the parent.

Ruby 2.0 makes process forking even more efficient with Unicorn because it implements Copy-on-Write (CoW), which means that a parent and child share physical memory until a write needs to be made. This is a very efficient sharing of resources that can drastically reduce memory use.

Sometimes, there are still issues with memory leakage, which occurs when workers get stuck or timeout. With the inclusion of a gem, and a small snippet of code that's included below, these edge cases are covered.

```ruby
# --- Start of unicorn worker killer code ---

if ENV['RAILS_ENV'] == 'production' 
  require 'unicorn/worker_killer'

  max_request_min =  500
  max_request_max =  600

  # Max requests per worker
  use Unicorn::WorkerKiller::MaxRequests, max_request_min, max_request_max

  oom_min = (240) * (1024**2)
  oom_max = (260) * (1024**2)

  # Max memory size (RSS) per worker
  use Unicorn::WorkerKiller::Oom, oom_min, oom_max
end

# --- End of unicorn worker killer code ---

require ::File.expand_path('../config/environment',  __FILE__)
run YourApp::Application
```

###### GIL

  - https://mail.python.org/pipermail/python-3000/2007-May/007414.html
  - http://archive.is/yCFB
  - http://archive.is/X1kh
  - http://www.confreaks.com/videos/1272-rubyconf2012-implementation-details-of-ruby-2-0-vm
  - https://news.ycombinator.com/item?id=3070382
  - http://merbist.com/2011/10/18/data-safety-and-gil-removal/
  - http://merbist.com/2011/10/03/about-concurrency-and-the-gil/

###### GC

  - http://www.rubyenterpriseedition.com/documentation.html
  - https://lightyearsoftware.com/2012/11/speed-up-mri-ruby-1-9/

###### Unicorn

  - https://www.digitalocean.com/community/articles/how-to-optimize-unicorn-workers-in-a-ruby-on-rails-app

###### Use Ruby Threads and Fibers

  - http://merbist.com/2011/02/22/concurrency-in-ruby-explained/

###### Fine Tune Your Objects

  - http://patshaughnessy.net/2013/2/8/ruby-mri-source-code-idioms-3-embedded-objects

###### Code optimizations

  - http://www.ruby-doc.org/core-2.1.1/Array.html#M000249

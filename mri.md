# What is the MRI/YARV

The Ruby MRI is short for Matz's Ruby Interpreter, and is the reference implementation for the Ruby programming language. It was released to the public in 1995, and is still actively developed, with the latest stable build being Ruby 2.1.1.

The YARV is an interpreter developed by Koichi Sasada that's also known as the KRI. It was developed in order to reduce the execution time of Ruby programs, and was very successful. As a result, YARV was merged into Ruby 1.9.0 and has replaced the MRI.

As the default interpreter for the Ruby programming language, the MRI has received it's fair share of criticism, primarily due to it's execution speeds and memory consumption.  However, recent Ruby versions have seen significant enhancements, and is on par with similar scripting languages like Python.  Not to mention the fact that some comparisons have the audacity to compare a compiled language to a scripting language, which is apples to oranges.  Developers typically choose Ruby for it's ease of writing/prototyping, understanding the fact that its execution time will _always_ be significantly slower than its compiled counterparts.

That being said, there are numerous methods and best practices that developers can follow in order to ensure that they're avoiding unnecessary bottlenecks.

## Some simple code enhancements

### String Optimization

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
    str = 'str1' << 'str2' << 'str3'
  end
end

# => #<Benchmark::Tms:0x007fdf9b990bd8 @label="", @real=22.713976, @cstime=0.0, @cutime=0.0, @stime=0.009999999999999995, @utime=22.689999999999998, @total=22.7>
```

### Blocks vs Procs

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

```bash
export RUBY_GC_MALLOC_LIMIT=60000000
export RUBY_FREE_MIN=200000
```

https://lightyearsoftware.com/2012/11/speed-up-mri-ruby-1-9/

### Fine Tune Your Objects

http://patshaughnessy.net/2013/2/8/ruby-mri-source-code-idioms-3-embedded-objects

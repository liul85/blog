+++
date = 2013-10-27
description = "ruby learning notes"
title = "block, proc, lambda"

[taxonomies]
tags = ["Ruby"]
+++

## block  
block其实就是一些可以被执行的代码段，block的语法有2中形式，一种是do...end形式，一种是简写的{}形式，例如：

```ruby
array1 = [1,2,3]
array1.each do |n|
    puts n
end
array1.each { |n| puts n }
```

block 一般跟times或者each，collect等方法一起使用，可以对某一个hash或者array中的元素进行操作。
用yield方法可以让函数接受一个block，例如：

```ruby
def block_test
  puts "we are in the method!"
  yield
  puts "we are back in the method!"
end
block_test { puts "we are in the block!" }
```

这样当函数执行到yield语句时候，就会调用传入的block。同时yield还可以处理传入的参数

```ruby
def block_test(name)
  puts "we are in the method!"
  yield name
  puts "we are back in the method!"
end
block_test("block") { |name| puts "we are in the #{name}!" }
```

## proc
在ruby中“任何都是对象”，但是其实对于blok来讲，它并不是对象，这是ruby中的一个例外，block不能保存到变量中，没有一个真正的对象使用起来那么方便，因此我们需要...proc, 它可以看做是保存成变量的block，这样我们可以给这个block起个名字，并可以重复调用它。

```ruby
multiply_of_3 = Proc.new do |n|
  n%3==0
end
(1..100).select(&multiply_of_3)
```

为什么要用proc呢，这里有2点需要理解的 
1. Procs是一个完全的对象，它具有所有对象的属性。
2. Procs在创建之后，可以在其他地方被重复调用，而block显然是不行的。 

我们可以直接使用Ruby的<code>.call</code>方法来调用一个Proc
```ruby
test = Proc.new { ... }
test.call
```

## Lambda
与proc一样，lambda也是对象，从一些语法和行为上来看，lambda跟proc在很多地方都是相同的。让我们来看一个lambda的例子吧

```ruby
lambda { puts "hello" }
```

与下面这个proc是相同的
```ruby
Proc.new { puts "hello" }
```

我们可以把一个lambda作为一个参数传递给一个函数

```ruby
def test_lambda(p_lambda)
  puts "I am the lambda test method."
  p_lambda.call
end
test_lambda(lambda { puts "I am the real lambda!" })
```

从以上例子可以看出来，可以通过以下方式来定义一个lambda
`lambda { block }`

```ruby
strings = ["Germany","Netherland","England","France"]
symbolize = lambda { |n| n.to_sym }
symbol = strings.collect(&symbolize)
```

lambda和proc的不同点：
lambda会检查传入的参数数量，如果你传入了错误的参数数量，lambda会抛出一个异常，而proc不会检查，但是只会置为nil。 
lambda在返回后会把控制权交给调用函数，而proc在返回后不会把控制权交给调用函数。 

下面这个proc可以执行一下看下输出：
```ruby
def batman_ironman_proc
  victor = Proc.new { return "Batman will win!" }
  victor.call
  "Iron man will win!"
end
puts batman_ironman_proc
```

我们可以看到结果，在Proc调用之后就结束了
```ruby
Batman will win!
``` 

再来看一个lambda的例子  

```ruby
def batman_ironman_lambda
  victor = lambda { return "Batman will win!"
  victor.call
  "Iron man will win!"
end
```

我们可以看到结果，在lambda调用之后回到了原来的函数中
```ruby
Iron man will win!
``` 

block,proc,lambda入门学习就先到这里，希望以后可以深入研究。

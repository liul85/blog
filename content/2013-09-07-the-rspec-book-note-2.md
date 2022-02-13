+++
date = 2013-09-07
description = "rspec learning notes"
title = "The rspec book note 2"

[taxonomies]
categories = ["Programming"]
tags = ["Ruby", "Rspec", "Cucumber"]
+++

在上一节中我们通过cucumber从外部对我们的Codebreaker程序行为进行了描述测试，通过step_definitions来描述了每个场景的步骤，在最后我们遗留了一个失败，我们期望Game向output发送一个消息，但是得到的结果消息是空的。这一节我们要用rspec来更细节的描述Game类的实例对象的行为。

在工程目录下创建一个rspec目录，再创建一个codebreaker子目录，然后创建game_spec.rb文件，加入如下内容：

```ruby
require 'spec_helper'
module Codebreaker
    describe Game do
	describe "#start" do
	    it "sends a welcome message"
	    it "prompts for the first guess"
	end
    end
end
```

在这个文件我们首先require了一个文件叫spec_helper，这个文件我们创建后保存在spec的目录下，rspec会自动将这个spec目录添加到$LOAD_PATH中。

接下来我们定义了一个叫Codebreakder的module，个人感觉这个模块意义就是指出测试的是哪个代码吧。

describe方法是RSpec的一个API，会返回一个RSpec::Core::ExampleGroup的子类，这里描述的就是一个Game对象的各种行为集合。然后it方法创建了一个行为，来描述具体的一个动作。

接下来我们要做的就是将我们的rspec测试集与我们的代码连接起来，在第二行，我们需要加入`require 'game'`，这样将我们的代码引用到了rspec测试集中。

然后在终端执行命令：

```shell
rspec spec/codebreaker/game_rspec.rb --format doc
```

我们会看到下面的输出：

```ruby
Codebreaker::Game
#start
sends a welcome message (PENDING: Not Yet Implemented)
prompts for the first guess (PENDING: Not Yet Implemented)
Pending:
  Codebreaker::Game#start sends a welcome message
    # Not Yet Implemented
    # ./spec/codebreaker/game_spec.rb:6
  Codebreaker::Game#start prompts for the first guess
    # Not Yet Implemented
    # ./spec/codebreaker/game_spec.rb:7
```

`--format doc`参数让rspec按照我们spec文件中的格式来输出结果。在输出结果中我们可以看到`"PENDING: Not yet implemented"`，这是因为我们只是写了个测试标题而已，接下来我们补充第一个测试：

```ruby
require 'spec_helper'
require 'game'

module Codebreaker
    describe Game do
	describe "#start" do
	    it "sends a welcome message" do
		output = double('output')
		game = Game.new(output)
		output.should_receive(:puts).with('Welcome to Codebreaker!')
		game.start
	    end

	    it "prompt for the first guess"
	end
    end
end
```

这里用到了RSpec中的动态double测试框架，很显然我现在还不知道这个是怎么用的，不过我先跳过。然后创建了一个Game的对象，把output传给它，接下来是期望output能够接收到`'Welcome to Codebreaker!'`的输出消息。最后调用start方法让game跑起来，这样我们就能接收到消息。

现在再跑一下rspec测试，加上--color参数，我们会看到输出显示 

`sends a welcome message (FAILED - 1)`

很显然我们得到了一个红色的失败的结果，因为我们的代码还没有任何实现。一旦我们的到一个红色，我们就必须让他变绿~ 

然后我们打开game.rb，来添加代码
```ruby
module Codebreaker
    class Game
        def initialize(output)
	    @output = output
        end
        def start
            @output.puts 'Welcome to Codebreaker!'
        end
    end
end
```

我们创建了initialize和start方法，再进行一次rspec测试，yeah，我们得到了第一个通过的测试，接下来我们应该进行重构，但是由于当前我们代码还没有任何重复，所以先不做。

接着再执行一下上一节遗留的cucumber feature，可以看到Then语句已经通过啦。

第二个测试我们还没有完成，我们需要再修改game_rspec.rb加以下代码

```ruby
it "prompts for the first guess" do
    output = double('output')
    game = Game.new(output)
    output.should_receive(:puts).with('Enter guess:')
    game.start
end
```

再进行rspec测试，我们会得到第二个红色，我们来修改game.rb代码，在start方法中再加一行输出

```ruby
def start
    @output.puts 'Welcome to Codebreaker!'
    @output.puts 'Enter guess:'
end
```

这样我们兴高采烈的再跑一次rspec测试，期待得到2个绿色，结果发现却2个测试全都红了，shit！

让我们来看看到底哪里出了问题，我们发现在2个测试中我们期望输出的是不同的字符，我们程序输出对于2个测试也都是满足的，但是我们发现这2个字符在game.start后是会一起输出的，而我们的2个测试只能识别自己期望的部分，如果多出来的部分就不能识别的，就报失败了。

我们使用RSpec框架中的`as_null_object()`方法来解决这个问题，使output匹配时候忽略其他字符，在2个测试中的double方法后面再引用`as_null_object`方法

```ruby
output = double('output').as_null_object
```

这样再跑一次RSpec测试，我们得到了2个绿色~

```ruby
Codebreaker::Game
  #start
    sends a welcome message
    prompt for the first guess

Finished in 0.00128 seconds
2 examples, 0 failures
```

* Red -> Green -> Refactory

## 重构

很明显，我们要开始考虑重构了，Martin Fowler在他的《重构》这本书里写道：“重构就是在不改变代码外部行为的前提下对内部代码的优化。” 那么我们怎么知道重构后没有改变代码的外在行为呢？这就依靠完整的测试保证，所以完整的测试是进行重构的前提。每当我们重构一处代码，我们跑一下测试得到绿色，说明我们的重构是成功的。

最基本的重构就是消除重复代码，我们来看看game_spec.rb的代码，基本上每一个测试的前2行都是类似的，这也是某种意义上的重复，我们需要修改掉

### before(:each)

```ruby
require 'spec_helper'
require 'game'

module Codebreaker
    describe Game do
    	describe "#start" do
    	    before(:each) do
    	    	@output = double('output').as_null_object
    	    	@game = Game.new(@output)
    	    end
    
    	    it "sends a welcome message" do
    	        @output.should_receive(:puts).with('Welcome to Codebreaker!')
    	        @game.start
    	    end
    
    	    it "prompt for the first guess" do
    	        @output.should_receive(:puts).with('Enter guess:')
    	        @game.start
    	    end
    	end
    end
end
```

这里我们引入了before方法，把每个测试前面的初始化实例放到了前面，这样在rspec执行每个测试之前，都会执行一下before来创建实例对象。

### let(:method){}

一般当before块中的代码只是初始化实例对象和赋值的时候，我们会用RSpec的let(:method)方法，let方法用一个词语来代替要用的方法和代码块

```ruby
require 'spec_helper'
require 'game'

module Codebreaker
    describe Game do
    	describe "#start" do
	    let(:output) { double('output').as_null_object }
	    let(:game)   { Game.new(output) }
    
    	    it "sends a welcome message" do
    	        output.should_receive(:puts).with('Welcome to Codebreaker!')
    	        game.start
    	    end
    
    	    it "prompt for the first guess" do
    	        output.should_receive(:puts).with('Enter guess:')
    	        game.start
    	    end
    	end
    end
end
```

然后我们跑一下cucumber的测试，会看到第一个场景的用例已经通过了。
总结一下：我们从上节遗留的cucumber的失败场景开始，遵循TDD的red，green，refactor的模式用RSpec写了2个测试，学习了按照BDD的cycle从程序外部行为的cucumber测试进入到代码内部的rspec测试。

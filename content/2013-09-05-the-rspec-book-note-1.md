+++
date = 2013-09-05
description = "rspec learning notes"
title = "The rspec book note 1"

[taxonomies]
categories = ["Programming"]
tags = ["Ruby", "Rspec", "Cucumber"]
+++

最近在看The rspec book这本书，由于平时工作不是这方面的，自己只是非常有兴趣，而且私下时间也不多，只靠每晚学习看的容易忘记，就在这里写一些笔记做复习。

cucumber主要是由features目录下的多个.feature文件构成BDD场景测试内容，然后针对其中的Given,When,Then,And等语句在featurs目录下的step_definitions子目录中增加ruby语法的行为步骤定义，来实现针对场景的测试执行。

参照书中的例子，逐步进行学习。

首先选择一个有趣的游戏--> Codebreaker

原文对游戏的介绍: 

> Codebreaker is a logic game in which a code-breaker tries to break a secret code created by a code-maker. The code-maker, which will be played by the application we’re going to write, creates a secret code of four numbers between 1 and 6.

> The code-breaker then gets some number of chances to break the code. In each turn, the code-breaker makes a guess of four numbers (again, 1 to 6). The code-maker then marks the guess with up to four + and - signs.

> A + indicates an exact match: one of the numbers in the guess is the same as one of the numbers in the secret code and in the same position.

> A - indicates a number match: one of the numbers in the guess is the same as one of the numbers in the secret code but in a different position.

> For example, given a secret code 1234, a guess with 4256 would earn a +-. The + is for the 2 in the second position in the guess, which matches the 2 in the secret code in both number and position: an exact match. The - is for the 4 in the first position in the guess, which matches the 4 in the code but not in the same position: a number match.

简单的说就是计算机给出一个4位的随机数，然后我们输入我们猜想的数字，如果其中有数字和位置都匹配上，那么会返回一个+号，如果只有数字匹配上，那么则返回一个-号。 

然后开始我们的迭代一，选定以下story进行交付: 

1. Code-breaker starts game
2. Code-breaker submits guess

接下来就可以写cucumber场景测试，首先在features目录下创建一个codebreaker_starts_game.feature，这里我们要测试的是游戏启动:

```gherkin
Feature: code-breaker starts the game
         As a code-breaker
         I want to start a game
         So that I can break the code

         Scenario: start game
	    Given I am not yet playing
	    When I start a new game
	    Then I should see "Welcome to Codebreaker!"
	    And I should see "Enter guess:"
```

再创建一个codebreaker_submits_guess.feature，来测试输入猜想的数字，并看到结果:

```gherkin
Feature: code-breaker submits guess
	In order to play this game
	As a code breaker
	I want to submit a guess
	So that I can try to breake the code

        Scenario: all exact matches
          Given the secret code is "1234"
          When I guess "1234"
          Then the mark should be "++++"
            
        Scenario: 2 exact matches and 2 number matches
          Given the secret code is "1234"
          When I guess "1243"
          Then the mark should be "++--"
            
        Scenario: 1 exact match and 3 number matches
          Given the secret code is "1234"
          When I guess "1342"
          Then the mark should be "+---"
            
        Scenario: 4 number matches
          Given the secret code is "1234"
          When I guess "4321"
          Then the mark should be "----"
```

这样看来，没写几个测试场景呢，用例就已经很复杂了，而且很重复，DRY，我们可以用Scenario Outline来实现简洁:

```gherkin
Feature: code-breaker submits guess
	In order to play this game
	As a code breaker
	I want to submit a guess
	So that I can try to breake the code

	Scenario Outline: submit guess
	    Given the secret code is "<code>"
	    When I guess "<guess>"
	    Then the mark should be "<mark>"

	    Scenarios: no matches
	      | code | guess | mark |
	      | 1234 | 5678  |      |

	    Scenarios: 1 number correct
	      | code | guess | mark |
	      | 1234 | 1555  | +    |
	      | 1234 | 2555  | _    |

	    Scenarios: 2 number correct
	      | code | guess | mark |
	      | 1234 | 5254  | ++   |
	      | 1234 | 5154  | +-   |
	      | 1234 | 2545  | --   |

	    Scenarios: 3 mumber correct
	      | code | guess | mark |
	      | 1234 | 5234  | +++  |
	      | 1234 | 5134  | ++-  |
	      | 1234 | 5124  | +--  |
	      | 1234 | 5123  | ---  |

	    Scenarios: all number correct
	      | code | guess | mark |
	      | 1234 | 1234  | ++++ |
	      | 1234 | 1243  | ++-- |
	      | 1234 | 1423  | +--- |
	      | 1234 | 4321  | ---- |
```

这样就好多了，容易读也好理解，就算是非技术人员也能看懂了。

接下来我们要定义测试步骤了，在features目录下建立子文件夹step_definitions,再建立codebreaker_steps.rb文件。

```ruby
Given /^I am not yet playing$/ do
end

When /^I start a new game$/ do
    game = Codebreaker::Game.new(output)
    game.start
end

Then /^I should see "([^"]*)"$/ do |message|
    output.messages.should include(message)
end

class Output
    def messages
      @messages ||= []
    end

    def puts(message)
      messages << message
    end
end

def output
    @output ||= Output.new
end
```

测试会失败，我们需要写我们game.rb程序了

```ruby
module Codebreaker
    class Game
    	def initialize(output)
    	end

    	def start
    	end
    end
end
```

再进行测试我们会看到提示输出与期望的不一致

```gherkin
Scenario: start game
        Given I am not yet playing
        When I start a new game
        Then I should see "Welcome to Codebreaker!"
          expected [] to include "Welcome to Codebreaker!"
```

至于这里怎么再修改，我们会在后面进行学习。

总结一下：这里我们了解了cucumber的基本目录结构，学习了gherkin语法，包括feature, Scenario, Given, When, Then以及Scenario Outline的用法，到此我们已经从Codebreaker游戏的行为出发点来用cucumber语法进行描述测试，下一节我们会学习使用rspec对每个独立对象的行为进行分析。

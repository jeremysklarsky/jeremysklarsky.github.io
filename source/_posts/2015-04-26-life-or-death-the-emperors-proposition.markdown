---
layout: post
title: "Life or Death? The Emperor's Proposition"
date: 2015-04-26 15:53:30 -0400
comments: true
categories: 
---
Time for another probability brain teaser. This one comes to us from [Braingle](http://www.braingle.com/brainteasers/15446/life-or-death-the-emperors-proposition.html). Here's a quick rundown of the problem:

>You are a prisoner sentenced to death. The Emperor offers you a chance to live by playing a simple game. He gives you 50 black marbles, 50 white marbles and 2 empty bowls. He then says, "Divide these 100 marbles into these 2 bowls. You can divide them any way you like as long as you use all the marbles. Then I will blindfold you and mix the bowls around. You then can choose one bowl and remove ONE marble. If the marble is WHITE you will live, but if the marble is BLACK... you will die."

How do I want to solve this? Unless I can figure out the answer with my own intuition, we're going to have to brute force this thing. So, the most obvious thing to do would be set up a simulation where we:

• Figure out all possible marble arrangement scenarios<br>
• Run each scenario a lot of times and check how many times we survive<br>
• Sort the results by % of times we survivce

The fun part here is building a digital model of the problem. We have a couple of pieces to account for. Two "pouches," 100 "marbles," and a random selector. For the pouches, we'll use an array of two arrays, and fill it with "marble" elements, and a the ruby method `sample` to randomly choose which "pouch" to pick from, and `sample` again to pick an element from the sub-array (the pouch). In order to set this experiment up to run all at once, we'll collect all the pouches in a hash. This way, we can store both the pouches and some descriptive info so we can discern the results later.

```ruby
class Emperor

  attr_accessor :marble_count, :pouches

  def initialize(marble_count)
    @marble_count = marble_count
    @pouches = {}
  end
end
```

The tricky part from a Ruby perspective is going to be populating the arrays with every possible marble arrangement. There are no limitations on how you can arrange the marbles. We don't need to set up both pouches, just figure out what one pouch looks like, and make the other pouch the inverse. 

This means there can be anywhere between 0-50 white marbles in one pouch, accompanied by anywhere from 0-50 black marbles. So 51 possible white marble arrangements, each with 51 possible black marble arrangements. 51 * 51 means there are 2601 possible combinations.

```ruby
  def populate 
    @combinations = (marble_count / 2) + 1
    combinations.times do |w|
      combinations.times do |b|
        name = "#{w} white, #{b} black"
        pouches[name] = [ [], [] ]
        
        w.times do 
          pouches[name][0] << true 
        end
        
        b.times do 
          pouches[name][0] << false  
        end

        (marble_count-w).times do 
          pouches[name][1] << true 
        end

        (marble_count-b).times do 
          pouches[name][1] << false  
        end
      end
    end
  end
```
So now, we have a hash where the key describes the contents of the first pouch (we'll just infer the contents of the second pouch): something like "20 white, 30 black." The value is then an array with two arrays representing the pouches. But instead of "white" and "black", we'll use the values `true` and `false` to represent white and black since it will be pretty easy to translate the boolean into passing or failing the Emperor's test.

Now that the pouches are filled, its time to run some tests. For each set of pouches, let's say we're going to run the test 10,000 times and store the results in a `results = {}` hash. In short, the emperor will randomly choose a pouch, and then randomly choose an element from that pouch. If it comes back true, add 1 point to the current marble configuration.

```ruby
  def choose
    pouches.each do |descriptor, pouches| 
      10000.times do 
        results[descriptor] ||= 0
        results[descriptor] += 1 if pouches.sample.sample
      end
    end
  end
```
Now all we have to do is sort the results hash and print the winner. 

```ruby
  def sort
    results.max_by{|k,v|v}
  end

  def print_winner
    winner = sort
    puts "#{winner[0]} will win #{(winner[1]/10000.0)*100}% of the time"
  end
```

We can delegate the entire operation to a single `call` method.
```ruby
  def call
    populate
    choose
    print_winner   
  end 
```
So running the following `Emperor.new(100).call` gives us the following result:
`1 white, 0 black will win 74.96% of the time`

Interesting! The best scenario is putting one white marble in the first pouch, and the remaining 49 white and all 50 of the black marbles in the second pouch. 

Why is this the case? An even split of marbles will produce odds of 50%. But if we limit the range of possiblities for the one of the pouches, we can guarantee that every time the emperor picks that pouch we get the desired outcome, while picking the second pouch is <i>almost</i> a 50/50 shot. So we get closer and closer to 75% chance of living. That's really the best odds this poor sap can hope for.

<i>Complete code</i>
```ruby
class Emperor

  attr_accessor :marble_count, :pouches, :combinations, :results, :sorted_results

  def initialize(marble_count)
    @marble_count = marble_count
    @pouches = {}
    @results = {}
  end

  def populate 
    @combinations = (marble_count / 2) + 1
    combinations.times do |w|
      combinations.times do |b|
        name = "#{w} white, #{b} black"
        pouches[name] = [ [], [] ]
        
        w.times do 
          pouches[name][0] << true 
        end
        
        b.times do 
          pouches[name][0] << false  
        end

        (marble_count-w).times do 
          pouches[name][1] << true 
        end

        (marble_count-b).times do 
          pouches[name][1] << false  
        end
      end
    end
  end

  def choose
    pouches.each do |descriptor, pouches| 
      10000.times do 
        results[descriptor] ||= 0
        results[descriptor] += 1 if pouches.sample.sample
      end
    end
  end

  def sort
    results.max_by{|k,v|v}
  end

  def print_winner
    winner = sort
    puts "#{winner[0]} will win #{winner[1]/10000.0}% of the time"
  end

  def call
    populate
    choose
    print_winner   
  end
  
end

Emperor.new(100).call
```
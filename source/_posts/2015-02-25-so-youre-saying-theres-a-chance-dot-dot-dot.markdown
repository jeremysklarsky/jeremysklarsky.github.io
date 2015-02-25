---
layout: post
title: "So You're Saying There's a Chance..."
date: 2015-02-25 12:37:22 -0500
comments: true
categories: 
---
I have always been fascinated by probability. Everything that is going to happen can be expressed in terms of probability. Of course, just because something *can* be expressed that way doesn't mean we always have the tools to accurately express it. But sometimes we *do* know, objetively, what the likelihood something is going to occur and we ignore it anyway. That's because humans are notoriously bad at understanding probabilities and effectively changing our behaviors based on those numbers. Very few people are afraid of driving, but many (including your faithful author) are terrified of flying even though for sometime the most dangerous part of a commercial flight is driving to the airport.

I think part of the reason for this is that for the most part, we've been taught to approach probablistic problems with math. Why is this a problem? Because I suck at math. Also, we descended from apes. They don't typically consult actuarial tables when determining whether the sound in the bush is a predator.

## The Birthday Problem
Here's a quick question: If you stick a random group of people in a room, what the odds that 2 of them have the same birthday? Or to be more specific, how many people do you need for there to be at least a 50% chance that two of them share a birthday. The first time I heard this I thought "well you'd probably need 182.5 people." 

That's wrong. You only need 23 people for there to be a better than 50% chance that two of them share the same birthday. Do you know how to solve this with math?

![MATH](http://upload.wikimedia.org/math/9/c/7/9c7763ad00291fc5be64923ea6d831d3.png)

This is <s>garbage</s> pretty hard to understand.

Here's why I LOVE programming - especially object oriented programming. Ruby allows me to think about how I would approach solving the problem, not just algorithmically but in the literal sense - with words. How would we solve this problem? We'd take a bunch of people, give them a random birthday, and then check to see if any of them share the same birthday. We'd do this a lot - say, 50,000 times - and take the average. Programming lets us do this. Math does not.

``` ruby
class Birthday

  attr_accessor :people, :matches

  def initialize(people)
    @people = people.to_i
    @matches = 0
    @sims = 0
    50000.times do |i|
      self.call
    end
    self.percents
  end

  def call
    birthdates = []
    @people.times do |i|
      birthdates << rand(365)
    end
    if birthdates.size != birthdates.uniq.size
      @matches += 1
    end

    @sims += 1 
  end

  def percents
    percent = (@matches / @sims.to_f.round(4))*100
    puts "With #{@people} people in a room, there is a #{percent}% chance"
    puts "that at least 2 people will share a birthday."
  end

end

five = Birthday.new(5)
ten = Birthday.new(10)
twenty = Birthday.new(20)
twenty_three = Birthday.new(23)
thirty = Birthday.new(30)
forty = Birthday.new(40)
fifty = Birthday.new(50)
sixty = Birthday.new(60)
seventy = Birthday.new(70)

```
Here's what we get when we run our simulation 50,000 times (with the expected %'s per wikipedia listed as reference:
```
| n  | % expected | % actual |
|----|------------|----------|
| 5  | 2.7%       | 2.62%    |
| 10 | 11.7%      | 11.8%    |
| 20 | 41.1%      | 40.9%    |
| 23 | 50.7%      | 50.9%    |
| 30 | 70.6%      | 70.7%    |
| 40 | 89.1%      | 89.1%    |
| 50 | 97%        | 97.1%    |
| 60 | 99.4%      | 99.4%    |
| 70 | 99.9%      | 99.9%    |
```
Looks like we were pretty close. That was pretty fun! Let's do another one. This might cause some controversy.

##Let's Make a Deal
Marilyn vos Savant, is aptly named as she's a genius with one of the highest recorded IQ's on record, if you care about that sort of thing. She even had a syndicated column in Parade magazine where people wrote in with brain teasers. Her most famous column was about the notorious "Monty Hall Problem." She answered the question correctly, but not before being publicly lambasted by some of the country's "leading" math and statistics experts. To this day the problem causes a fight whenever it comes up. 

Here's the scenario. You're on a game show and the host says "There are 3 doors. Behind one of these doors is a car. Behind the other two are goats."

You pick Door #1. The host will then open one of the two remaining doors that contains a goat - let's say he opens Door #3. Then you are given the option to keep your original choice of Door #1 or switch to Door #2. 

Here's the question: is it better to keep your original pick or switch to the other remaining door? Or does it not matter?

Of course, when I first heard this problem I thought "Easy - it doesn't matter whether you switch or not. Its a 50/50 coin flip. Two doors, two choices. One is a goat, one is a car. The answer is it doesn't matter whether you switch."

It really hurt my brain to find out this was very wrong. It is always better to switch. Like 100% better. Twice as good. But I didn't feel too bad. [Math professors were getting it wrong too.](http://en.wikipedia.org/wiki/Monty_Hall_problem#Vos_Savant_and_the_media_furor)

Writing code to solve this problem not only gives you the answer, but helps you understand the problem the better. 

``` ruby
class MontyHall

  attr_accessor :doors, :user_choice, :total_games, :staying_wins, :switching_wins

  def initialize(times)
    @total_games = 0
    @staying_wins = 0
    @switching_wins = 0
    @times = times
  end

  def call
    @times.times do |i|
      @doors = ["goat", "goat", "goat"]
      hide_car
      get_user_choice
      @total_games += 1
      if staying_wins?
        @staying_wins += 1
      elsif switching_wins?
        @switching_wins +=1
      end
    end
  end

  def hide_car
    doors[rand(0..2)] = "car"
  end

  def get_user_choice
    @user_choice = rand(0..2)
  end

  def staying_wins?
    doors[user_choice] == "car"
  end

  def switching_wins?
    doors[user_choice] == "goat"
  end

  def percents
    puts "After #{@total_games} games:"
    puts "Staying every time wins #{(@staying_wins / @total_games.to_f.round(2))*100}%"
    puts "Switching every time wins #{(@switching_wins / @total_games.to_f.round(2))*100}%"
  end

end
```
This script puts "car" in a slot in an array, gives the user a random pick, and then tests whether keeping the user's pick or switching results in winning the car. In the end, of course we find that when you switch doors you will win the car 66.6% of the time and will win the car only 33.3% when you keep your original choice. 

But the code skips a step - it skips the part where the host opens a door and then gives the contestant the choice. Why does it skip that part? BECAUSE IT IS IRRELEVANT TO THE OUTCOME.

The above code is an exercise in futility, as it simply an obfuscated way of testing how random your computer's random number generator is. But that's the point of Monty Hall. It tries to hide how simple the problem is. The question shouldn't be "is it to your advantage to switch doors?" That is just a fancier way of asking "what are the odds that your original pick was correct?" That is obvious and intuitive: 1 in 3. The fact that the host opens one of the doors DOES NOT CHANGE THIS. Your original pick is still only a 1 in 3 shot of being right. Ergo, the one remaining door will have the car 2/3 of the time. 

This is why programming is fun. Without the benefit of math, we have to express these complex problems in abstract, metaphorical terms that are easier to understand regardless of your background in math.

Head over to the [NYTimes feature on the Monty Hall Problem](http://www.nytimes.com/2008/04/08/science/08monty.html?_r=0) to play an interactive version of the game. 
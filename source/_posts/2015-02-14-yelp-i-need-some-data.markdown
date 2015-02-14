---
layout: post
title: "Yelp! I need some data, not just any data..."
date: 2015-02-14 12:31:14 -0500
comments: true
categories: 
---
*Authors note: This is my first ever post!*

## Background

My first two weeks at [Flatiron School](http://www.flatironschool.com) have been an absolute whirlwind. I can feel us slowly approaching a time when we can **actually** deploy our skills and do something useful. 

Diving into Ruby headfirst has been a joy, though it remained largely in the realm of theoretical and hypothetical. I'm not one scoff at intellectual exercises for their own sake, but the painstaking task of iterating over made up hashes was starting to test my own powers of concentration and emotional makeup. 

Among the more powerful lectures have involved data scraping and parsing demonstrations. 

In a couple of quick clicks our lead <s>guru</s> instructor managed to pull all the prices of every apartment listed on craigslist and find the average. Looks like we haven't been iterating for nothing. BOOM!

![Kramer Shocked](http://media.giphy.com/media/QMcamps7Gzj2g/giphy.gif)
<br>^everyone in class.

## Writing a Yelp-er Method
That looked like fun and I wanted a turn. For some years I've been fairly obsessed with data collection and manipulation. But my powers to participate in the sport was limited to both, obviously, my data collection and manipulation abilities. But no longer! In my first two weeks as a developer, I've learned to adopt the idea that I am not limited by what my computer tells me what I'm allowed to do. It is a tool to do my bidding!

For my test case, I thought scraping [Yelp](http://www.yelp.com) would be a good place to start. After a few clicks, I realized the easiest way to access Yelp's API would not be parsing a long JSON string like we had in the Spotify lab. Luckily for you, dear readers, [Yelp's API documentation](http://www.yelp.com/developers/documentation/v2/overview) lead us to a very helpful Ruby gem for parsing search results directly into a hash with tons of data pertaining to your search.

Let's get started:<br>
1. In terminal, run 'gem install yelp'.<br>
2. Require yelp in your ruby program.<br>
3. Make a million dollars. 

<script src="https://gist.github.com/jeremysklarsky/78907211a01bc56a401c.js"></script>

When the interpreter runs this code, your client object has access to all of the methods defined in the gem. You interact with Yelp's data by using a the parameters laid out in their API within the gem's syntax. Yelp gives you plenty of helpful tags to customize your search. Check out the [documentation](http://www.yelp.com/developers/documentation/v2/search_api) in the search API for some of them. 

<script src="https://gist.github.com/jeremysklarsky/9340471c75cbbbe9bb76.js"></script>
For our test search, I focused on some pretty basic ones: search for Italian restaurants in the nearby neighborhood of Park Slope.

Yelp has a separate [Business API](http://www.yelp.com/developers/documentation/v2/business) for accessing the info in the returned data structure. The gem lets us call them as methods on each individual business object returned in the Burst. Here are a few of the major ones.

<!---Business API methods-->
<script src="https://gist.github.com/jeremysklarsky/ba3ab4c713358d87ba3c.js"></script> 

The next step is parsing the data. The return of this search is an array like object called a Burst, which contains instances of Business objects.

<script src="https://gist.github.com/jeremysklarsky/48400e94b8ca8c4efb84.js"></script> 

Now here is the fun part. We can then iterate through each object's keys and store the values in our own hash. 

<script src="https://gist.github.com/jeremysklarsky/bd3abab294de820e55dd.js"></script>

For the moment, I can't seem to get more than ~17 results to show up. This may have something to do with my amateurish Yelp authorization key. In any event, 
the results look something like this:
<script src="https://gist.github.com/jeremysklarsky/f4958d77abfffd8e2f85.js"></script>

From there we have a hash with which we can do anything we want. Sort by rating, sort by rating and most reviews, create adjusted ratings for restaurant categories based on the category average (i.e. grading the restaurant on a curve), whatever we want. The world (wide web) is our oyster!


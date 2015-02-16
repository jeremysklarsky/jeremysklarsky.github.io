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

Diving into Ruby headfirst has been a joy, though it remained largely in the realm of theoretical and hypothetical. I'm not one scoff at intellectual exercises for their own sake, but the painstaking task of iterating over made up hashes was starting to test my own powers of concentration and emotional constitution. 

Among the more powerful lectures have involved data scraping and parsing demonstrations. 

In a couple of quick clicks our lead <s>guru</s> instructor managed to pull all the prices of every apartment listed on craigslist and find the average. Looks like we haven't been iterating for nothing. BOOM!

![Kramer Shocked](http://media.giphy.com/media/QMcamps7Gzj2g/giphy.gif)
<br>^everyone in class.

## Writing a Yelp-er Method
That looked like fun and I wanted a turn. For some years I've been fairly obsessed with data collection and manipulation. But my participation in the sport was limited by both, obviously, my data collection and manipulation abilities. But no longer! In my first two weeks as a developer, I've learned to adopt the idea that I am not limited by what my computer tells me what I'm allowed to do. It is a tool to do my bidding!

For my test case, I thought scraping [Yelp](http://www.yelp.com) would be a good place to start. After a few clicks, I realized the easiest way to access Yelp's API would not be parsing a long JSON string like we had in the Spotify lab. Luckily for you, dear readers, [Yelp's API documentation](http://www.yelp.com/developers/documentation/v2/overview) lead us to a very helpful Ruby gem for parsing search results directly into a hash with tons of data pertaining to your search.

Let's get started:<br>
1. In terminal, run 'gem install yelp'.<br>
2. 'require yelp' in your ruby program, or include 'yelp' in your gemfile.<br>
3. Make a million dollars. 

``` ruby
require 'yelp'
 
client = Yelp::Client.new({ consumer_key: YOUR_CONSUMER_KEY,
                            consumer_secret: YOUR_CONSUMER_SECRET,
                            token: YOUR_TOKEN,
                            token_secret: YOUR_TOKEN_SECRET
                          })
```

When the interpreter runs this code, your client object has access to all of the methods defined in the gem. You interact with Yelp's data by using a the parameters laid out in their API within the gem's syntax. Yelp gives you plenty of helpful tags to customize your search. Check out the [documentation](http://www.yelp.com/developers/documentation/v2/search_api) in the search API for some of them. 

``` ruby
italian_places = Hash.new{|k, v| k[v] = {}}
 
params = { term: 'italian',
           category_filter: 'restaurants'
         }
 
locale = { cc: "US", lang: 'en' }

client.search('Park Slope', params, locale)
```
For our test search, I focused on some pretty basic ones: search for Italian restaurants in the nearby neighborhood of Park Slope. The return of this search is an array like object called a Burst, which contains instances of Business objects. 

Yelp then has a separate [Business API](http://www.yelp.com/developers/documentation/v2/business) for accessing the info in the returned data structure. The gem lets us call them as methods on each individual business object returned in the Burst. Here are a few of the major ones.

<!---Business API methods-->
``` ruby
## search
response = client.search('San Francisco')
 
response.businesses
# [<Business 1>, <Business 2>, ...]
 
response.businesses[0].name
# "Kim Makoi, DC"
 
response.businesses[0].rating
# 5.0
 
## business
response = client.business('yelp-san-francisco')
 
response.name
# Yelp
 
response.categories
# [["Local Flavor", "localflavor"], ["Mass Media", "massmedia"]]
```

Now here is the fun part: parsing the data. We can then iterate through each object's keys and store the values in our own hash, which I've created and called italian_places. 
``` ruby
client.search('Park Slope', params, locale).businesses.each do |place|
  italian_places[place.name] = {
    :review_count => place.review_count, 
    :rating => place.rating, 
    :phone => place.phone}
end
```
For the moment, I can't seem to get more than ~17 results to show up. This may have something to do with my amateurish Yelp authorization key. In any event, 
the results look something like this:

``` ruby
italian_places = {
 "Piccoli Trattoria"=>{:review_count=>213, :rating=>4.5, :phone=>"7187880066"},
 "Al Di La Trattoria"=>{:review_count=>516, :rating=>4.0, :phone=>"7186368888"},
 "Peppino's"=>{:review_count=>218, :rating=>4.5, :phone=>"7187687244"},
 "Mariella"=>{:review_count=>62, :rating=>4.5, :phone=>"7184992132"},
 "Scottadito Osteria Toscana"=>{:review_count=>484, :rating=>4.0, :phone=>"7186364800"},
 "Giovanni's Brooklyn Eats"=>{:review_count=>178, :rating=>4.0, :phone=>"7187888001"},
 "Scalino"=>{:review_count=>142, :rating=>4.0, :phone=>"7188405738"}}
```

From there we have a hash with which we can do anything we want. Sort by rating, sort by rating and most reviews, create adjusted ratings for restaurant categories based on the category average (i.e. grading the restaurant on a curve), whatever we want. The world (wide web) is our oyster!



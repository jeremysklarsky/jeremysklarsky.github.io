---
layout: post
title: "Ranking Seinfeld Seasons"
date: 2015-12-04 09:41:15 -0500
comments: true
categories: 
---
*I was recently sent an email containing a link to [NYMag/Vulture's rankings of every Seinfeld episode](http://www.vulture.com/2015/06/every-seinfeld-episode-ranked.html), which they put out in response to Hulu releasing the entire series. What follows is my response to the group email.* 

All:
Using my new fangled programming skills I was able to parse the text of the article and crunch a couple of numbers. I think there are some legitimate grips with some individual episode rankings. But with the exception of what Shosh might deem a too highly ranked Season 8, I think in aggregate the total season rankings ring somewhat true to me.

What I've done is take each ranking and assign it it's inverse point value (total # of episodes minus ranking). So for example, the #2 ranked episode gets 168 points and the 167th ranked episode gets 2 points. I did both point totals and averages just to see if there would be any discrepancy due to the early seasons having fewer episodes. There was no discrepancy. 

I also was able to infer, based on the average score, what the "average" episode's rank would've been. So for example, the average season 5 episode would've ranked about 52nd. 

According to this article, the overall best season is Season 5, and by a long shot. The average season 5 episode ranks 21 places hire then the next best season, #6. As expected, season 1, 2, and 3 are ranked lowest (and in that order) both in total score and average score.

```
| SEASON | TOTALS | AVERAGES | AVERAGE RANK |
|--------|--------|----------|--------------|
| 5      | 2455   | 116.9    | 52.1         |
| 6      | 2106   | 95.72    | 73.3         |
| 9      | 2102   | 95.54    | 73.5         |
| 8      | 1937   | 88.04    | 81.0         |
| 4      | 1703   | 77.4     | 91.6         |
| 7      | 1609   | 76.61    | 92.4         |
| 3      | 1544   | 70.18    | 98.8         |
| 2      | 652    | 54.33    | 114.7        |
| 1      | 73     | 18.25    | 150.8        |

```

##Tech notes, if you're interested
Used jQuery to scrape the website
` $('p:contains("(Season")').each(function(i, item){console.log(item.textContent.split(").")[0])})`

Defaulted back to Ruby for the rest, putting the list into an array `STRING`
```ruby

STRING = [
  '169. "The Puerto Rican Day Parade" (Season 9',
  '168. "The Outing" (Season 4',
  '167. "The Finale" (Season 9',
  '166. "The Jacket" (Season 2',
  '165. "The Tape" (Season 3',
  '164. "The Deal" (Season 2',
  etc...
]

def convert_num(num)
  169-num
end

def sort_hash(hash)
  hash.sort_by {|k,v| v}.reverse.to_h
end

rankings_hash = Hash.new { |h, k| h[k] = [] }
average_score = {}
total_score = {}

STRING.each do |rank|
  num = convert_num(rank.split(".").first.to_i)
  season = rank.split(" (Season ").last
  rankings_hash[season].push(num.to_i)
end

rankings_hash.each do |season, scores|
  total_score[season] = scores.inject(:+)
  average_score[season] = scores.inject{ |sum, el| sum + el }.to_f / scores.size
end

puts sort_hash(total_score)
puts sort_hash(average_score)

```
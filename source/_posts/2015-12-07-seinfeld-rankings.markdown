---
layout: post
title: "Seinfeld Rankings Part II: The Writers"
date: 2015-12-07 22:01:31 -0500
comments: true
categories: 
---
###A blog about nothing
####(not that there's anything wrong with that)
In my [previous post]({% post_url 2015-12-04-ranking-seinfeld-seasons %}), we examined NY Magazine's vulture.com rankings of Seinfeld episodes. By assigning each episode a points based on their rankings (I arbitrarily made a linear point scale from worst to best - unless anyone reading is a stats person and wants to weigh in...), we found that Season 5 was the runaway winner. An average Season 5 episode would've ranked ~52nd, meaning its episodes were, on average, in the the top 1/3 of the whole show. I thought it would be fun to continue this thought experiment. 

**Who are the best writers in the history of the show?** Solving this would take two wi-fi enabled flights back and forth to Tampa and a couple hours when you wake up earlier than your wife even though you're on vacation and you can't sleep in.

In the previous exercise, we grabbed our rankings from Vulture's website using a simple jQuery statement, and then iterated through the strings to get the info we needed to make our rankings. Now, we needed to get each episode's writer and link them up to rankings to know who is truly master of the domain. 

I'll show some code, but if you're bored or uninterested, just skip to the bottom for the summary. 

###Writer? We're talking about a sitcom
Luckily for us, there is a free API called [The Open Movie Database](http://omdbapi.com) that gives us most of the same information found on IMDB. I may do something with IMDB ratings at some point, but to be honest, they are mostly useless and would probably yield very little information of interest. Anyhow, OMDb has a very simple API as far as TV series are concerned. There are 2 methods I used. The first is `http://www.omdbapi.com/?t={show}&Season={season#}`. A request for Seinfeld's first season `http://www.omdbapi.com/Season=1` returns a nice JSON object:

```javascript
{
  Title: "Seinfeld",
  Season: "1",
  Episodes: [
    {
      Title: "The Stakeout",
      Released: "1990-05-31",
      Episode: "2",
      imdbRating: "7.8",
      imdbID: "tt0697784"
    },
    {
      Title: "The Robbery",
      Released: "1990-06-07",
      Episode: "3",
      imdbRating: "7.7",
      imdbID: "tt0697768"
    },
    {
      Title: "Male Unbonding",
      Released: "1990-06-14",
      Episode: "4",
      imdbRating: "7.6",
      imdbID: "tt0697645"
    },
    {
      Title: "The Stock Tip",
      Released: "1990-06-21",
      Episode: "5",
      imdbRating: "7.7",
      imdbID: "tt0697788"
    }
  ],
  Response: "True"
}
```
So now, we simply make this call for each season of the show, incrementing up the season number until there is no response. Along the way, we collect the `imdbID`'s for our second call. Then, we use the `i={imdbID}` API method for each episode we collected. A sample response would look like this:

```json
{
  Title: "Male Unbonding",
  Season: "1",
  Episode: "4",
  Runtime: "23 min",
  Genre: "Comedy",
  Director: "Tom Cherones",
  Writer: "Larry David (created by), Jerry Seinfeld (created by), Larry David, Jerry Seinfeld",
  Actors: "Jerry Seinfeld, Julia Louis-Dreyfus, Michael Richards, Jason Alexander",
  imdbRating: "7.6",
  imdbID: "tt0697645",
  seriesID: "tt0098904",
}
```

Now, we just have to parse the `Writer` attribute, split on the commas and filter out any of the writer credits we want to ignore (e.g. "created by", "story by", "story editor", etc). We are just interested in the main billed writer for this experiment.

###It's going in the vault
So, now that it looks like I'm going to be making upwards of 180 or so API calls, I'm going to want to persist this data so I don't have to make all these calls over and over again. We'll make a few tables: Episodes, Seasons, Writers, and EpisodeWriters (a join table since an episode can have multiple writers.) Join them up and we're ready to go.

Working in a Ruby ActiveRecord enabled environment makes this all really easy. Maybe I'll do this all again in Node one day..

Here's the `Writer` class:

```ruby
class Writer < ActiveRecord::Base
  has_many :episode_writers
  has_many :episodes, through: :episode_writers

  def total_points
    self.episodes.size > 0 ? self.episodes.collect {|ep| ep.points}.inject(:+) : 0
  end

  def average 
    self.episodes.size > 0 ? (total_points / self.episodes.size.to_f).round(2) : 0
  end

  def episode_count
    self.episodes.size
  end

  def average_rank
    Episode.all.size - self.average
  end

  def self.all_time_points
    Writer.all.sort_by {|writer|writer.total_points}.reverse
  end

  def self.all_time_average
    Writer.all.sort_by {|writer|writer.average}.reverse
  end
end
```
### He named names!
For arguments, sake we'll limit our sample to those writers who contributed more than 5 episodes over the course of the series. This gives us a total of 11 writers, sorted by average episode rank (writing duos Jeff Schaffer / Alec Berg and Tom Gammill / Max Pross are listed together. Jerry and Larry, while collaborators on a lot of episodes, are listed separately because of Larry's significant solo career). Ready?


```
| Writer          | Average Rank | Total Points | Episode Count |
|-----------------|--------------|--------------|---------------|
| Schaffer / Berg | 48.9         | 1429         | 12            |
| Carol Leifer    | 54           | 570          | 5             |
| David Mandel    | 61.4         | 746          | 7             |
| Larry Charles   | 67.1         | 1404         | 14            |
| Greg Kavett     | 82.7         | 768          | 9             |
| Andy Robin      | 84.6         | 917          | 11            |
| Gammill / Pross | 90           | 936          | 12            |
| Larry David     | 93.2         | 3739         | 50            |
| Jerry Seinfeld  | 97.1         | 1064         | 15            |
| Peter Mehlman   | 99.2         | 1307         | 19            |
| Spike Feresten  | 100.5        | 540          | 8             |

```

Very interesting! Larry David is by far the biggest (billed) individual contributor to the show, but his average episode ranks towards the bottom. By contrast, Jeff Schaffer and Alec Berg, who later went on to create the league have both the second most total points (behind Larry) and the highest average. Here's a list of their episodes and rankings:
```
'The Summer of George' Rank: 9
'The Secret Code'      Rank: 10
'The Gymnast'          Rank: 19
'The Doodle'           Rank: 33
'The Voice'            Rank: 38
'The Butter Shave'     Rank: 47
'The Seven'            Rank: 48
'The Maid'             Rank: 50
'The Chicken Roaster'  Rank: 57
'The Foundation'       Rank: 64
'The Calzone'          Rank: 97
'The Strike'           Rank: 127
```
10 of 12 episodes are considered by Vulture above average, 9 of 12 in the top 1/3, 3 in the Top 20, and only 1 in the bottom 1/3 of episodes. In fact, an entire season of Schaffer/Berg episodes would've ranked higher than Season 5, the overall top ranked season. 

Larry, on the other hand, has the dubious distinction of having written both Vulture's top and bottom ranked episode. Of course, calling "The Finale" the worst Seinfeld episode is a stunt. It's barely a Seinfeld episode and outside of the music and cast is nothing like the rest of the series. So it is at this point, perhaps, the ranking system breaks down. "The Contest" is not only the most acclaimed Seinfeld episodes, it is roundly considered one of the greatest scripted half hours in television history. In fact, it's critical success is partly what lead to the explosion in Seinfeld's popularity. Without it, there might not have been five more seasons. Is it worth only 1 more point in the rankings than the next best episode on the list ("The Subway")? I can't say for sure.

I should, however, note that while Larry and Jerry's rankings are somewhat low this is skewed data. It is said that the pair (through the end of Season 7, and later Jerry alone) had final say on all scripts and did the last pass of every episode. They tended to take billing as an episode's writer less and less throughout the series and were more likely the billed writers in the early, low ranking seasons. Whether this is reflective of their contributions to the show's writing I cannot say, but thought it worth mentioning. 


###Directors
We can do a similar analysis using virtually the same code, just swapping out `Writer` for `Director` because object oriented metaprogramming is cool like that. 

We'll just check in on the two directors who directed the overwhelming majority of the show's episodes: Tom Cherones (Seasons 1-5) and Andy Ackerman (6-9). My gut told me that Ackerman would have better stats, which he did, though slightly.

```
| Director        | Average Rank | Total Points | Episode Count |
|-----------------|--------------|--------------|---------------|
| Andy Ackerman   | 77           | 6828         | 75            |
| Tom Cherones    | 90           | 5308         | 68            |
```


---
layout: post
title: "The SQL was better than the original"
date: 2015-03-15 19:16:47 -0400
comments: true
categories: 
---
One of the greatest benefits of developing rails applications with a framework like Ruby on Rails is the flexibility of being able to access a data from our databases without having to write ugly SQL statements. While SQL is a very powerful and efficient language for retrieving data, at least to this author, is nowhere near as elegant or intuitive as Ruby.

As part of the Rails framework, we have access to ActiveRecord, a Ruby DSL (domain specific language) that allows us to communicate with the database with ease. ActiveRecord builds out custom methods that erase the blood-brain barrier of database and our Ruby model classes. It allows our models to treat its relevant data as though they were were attributes of each instance of a particular class.

My first instinct as someone who has recently fallen in love with Ruby is to try to do as much with that language as possible. The problem with that it is nowhere near as efficient as using a simple SQL statement. In our labs and early projects, that isn’t really an issue as the size of our databases and programs don’t require the efficiency of a larger scale application. But as students, we need to train ourselves to develop good coding habits! That is where ActiveRecord comes in to play.

## Experiment
![For science!](http://static.fjcdn.com/pictures/For+science_3cf7fb_4751731.jpg)

To begin our test, I created a simple domain model that many of my Flatiron classmates would be very familiar with: `Artist` and `Song`. For every step along the way, I seed the database with a number of artists, and randomly assign them songs (at a rate of 100 songs per artist.)

``` Ruby
def make_artists(num)
  num.times do |i|
    artist = Artist.new
    artist.save
  end
end

def make_songs(num)
  num.times do |i|
    song = Song.create(:artist_id => rand(1..100))
    song.save
  end
end
```
For our experiment we’ll be executing a simple query: Which artist has the most songs?

## Approach 1: Ruby

We create a primitive data structure, load in our data and sort with basic ruby methods. We create a giant hash where each artist in the database is a key pointing to an array of each of their songs. We then sort by the size of values to find our artist with the most songs. 

``` Ruby
  def self.most_songs_ruby  
    hash = {}
    Artist.all.each do |artist|
      hash[artist] = artist.songs
    end
    hash.max_by{|k,v| v.size }
  end
```

## Approach 2: ActiveRecord and SQL
``` Ruby
  def self.most_songs_sql
    joins(:songs).
    group(:artist_id).
    order("COUNT(*) DESC").
    limit(1).first
  end 
```
Using ActiveRecord, we join up the `artists` and `songs` table, group our results by `artist_id`, and order our results by the count of songs each artist has. ActiveRecord then translates this into a SQL statement that queries our database. 

``` SQL 
SELECT * FROM songs
INNER JOIN artists ON artist_id = artists.id
GROUP BY artist_id
ORDER BY COUNT(*) DESC
```

After each round of seeding, we run `puts Benchmark.measure {Artist.most_songs_sql}` and `puts Benchmark.measure {Artist.most_songs_ruby}` from a rails console to determine the speed of each querying method.
```
| Artists/Songs | Ruby (seconds) | SQL (seconds) | Mult. |
|---------------|----------------|---------------|-------|
| 1/100         | .117           | .019          | 6.2   |
| 5/500         | .088           | .012          | 7.4   |
| 10/1000       | .128           | .012          | 10.93 |
| 50/5000       | .183           | .014          | 13.44 |
| 100/10000     | .300           | .015          | 19.84 |
| 500/50000     | 4.396          | .131          | 33.43 |
| 1000/100000   | 15.830         | .282          | 56.12 |
```
## Results

{% img /images/efficiency.png %}

Even with a small database, we immediately find SQL querying to be vastly faster than querying with Ruby. What slows us down in Ruby is needing to load every row from our database into a ruby object. The SQL code, on the other hand, sorts a pre-existing data structure and only needs to return 1 object into memory: the one artist with the most songs.

So even though finding the artist with the most songs when there are 10 artists in the database only takes 1/10 of a second with our Ruby method, ActiveRecord executes that query 10x faster. 

{% img /images/seconds.png %}

More importantly, we can see how quickly using Ruby to sort through data becomes inefficient. As our database grows in size, Ruby's sort times scale up with it. SQL sort times, just barely inch along. As a result, we find that as the dataset grows, SQL becomes even more efficient. So by the time there are 1000 artists and 100,000 songs in our database, SQL is approximately 60x faster. Our ruby method takes almost 17 seconds (a.k.a. You're fired).

As an aside, I found that because of how a Ruby interpreter stores objects in memory, repeating the method on the same dataset improved run times. However, we see similar improvements in SQL run times. And of course, this assumes a static dataset which we wouldn't find in the real world.

## Conclusion
If you just need to pull one object out of the database(e.g. `Artist.find_by(:name => "Lady Gaga").songs`), Ruby is fine. But once you need to sort, calculate, count, or do anything else, go SQL or go home.
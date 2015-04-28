---
layout: post
title: "Checksum gives me indigestion"
date: 2015-04-28 17:11:09 -0400
comments: true
categories: 
---
Today I encountered a problem I hadn't thought much about before. How can I tell if the contents of two files are the same? If we're directly comparing two files, that should be pretty simple. Given 3 files, how can we tell? Simply, we'd read the contents of the file and figure out if those objects have equivalency.

Let's make 3 files. 
`test1.txt` and `test2.txt` will contain the string "THIS IS SOME TEXT." 
`test3.txt` will contain "THIS IS SOME OTHER TEXT."

```ruby
one = File.open("test1.txt", "r").read
two = File.open("test2.txt", "r").read
three = File.open("test3.txt", "r").read

puts one == two
puts one == three
```
What do we expect this program to output? We expect line 5 to evaluate the `true` and line 6 to evaluate to `false`, which it does.

This is a good solution but it is not scalable. What if instead of only comparing three files, we wanted to compare a file many times larger across hundreds of thousands of files? That would be a nightmare. So we need to find a more efficient way to do this.

Why would we need to do this you ask? Well, if we're maintaining a file server or database we'd need a quick way to eliminate duplicate files to keep the server lean and prevent confusion later down the line. Another common application for needing to check file equivalency is for [checking your data's integrity during transmission or storage](http://en.wikipedia.org/wiki/Checksum).

How is this done? By creating something called a checksum. [Wireshark has provides this summary:](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvChecksums.html)
>A checksum is basically a calculated summary of such a data portion.
>Network data transmissions often produce errors, such as toggled, missing or duplicated bits. As a result, the data received might not be identical to the data transmitted, which is obviously a bad thing.
>Because of these transmission errors, network protocols very often use checksums to detect such errors. The transmitter will calculate a checksum of the data and transmits the data together with the checksum. The receiver will calculate the checksum of the received data with the same algorithm as the transmitter. If the received and calculated checksums don’t match a transmission error has occurred.

In other words, data transmitted over a network is being spell checked as it is copied.

<h3>Checksums and Ruby Data Structures</h3>
What's another reason for this? Storing a bunch of files in memory gets expensive very quickly. If files all have different names, then the only way to search for duplicate values is by reading the contents of a file and then comparing it to all the values stored in memory, like we did in the first example. What if instead we just stored a checksum, a smaller digital fingerprint of the file's contents? Then we have any number of ways to store, search, or compare our data.

Ruby doesn't natively support hashing algorithms, but fortunately the `Digest` module and the `MD5` hashing algorithim are built into the standard library so all we have to do is require them.
```ruby
require 'digest/md5'
one = File.open("test1.txt", "r")
two = File.open("test2.txt", "r")
three = File.open("test3.txt", "r")

def checksum(*files)

  hash = Hash.new { |h, k| h[k] = [] }
  files.each do |file|
    # for each file, read the contents
    # and store a checksum as a key in the hash
    md5 = Digest::MD5.new
    md5 << file.read
    hash[md5.hexdigest] << file
  end
  hash
end

checksum(one, two, three) 
```
Running this program results in this: `=> {"81e3a7e854d334e82f75a2bcdbe6a3da"=>[#<File:test1.txt>, #<File:test2.txt>], "32b2eccab2dcc035c50820d0943e5b94"=>[#<File:test3.txt>]}`

So even though these were three different files, our checksum algorithm was able to determine that the first two files have equivalent values. What's the application for this?

Searching through a hash for a key is fast - much faster than iterating through an array. So if we wanted to find duplicate files, instead of using the file name (an intuitive choice) for the key, we could store this checksum value as the key. In a sense, the checksum is both the key AND the value. With its place reserved in memory, all we'd have to do is check to see if the new file's checksum exists as a key in our hash.

Consider this program. We initialize a `Checker` class with two files, `test1.txt` and `test3.txt`. Then we run our `unique?` function on `test2.txt`. Remember, files 1 and 2 have the same contents. We now have very small fingerprints of 1 and 3 stored in memory, and instead of reading their entire contents to check them against our new file, we simply create a fingerprint for the new file and compare it to our current set of fingerprints.

```ruby
class Checker
  require 'pry'
  require 'digest/md5'
  
  attr_accessor :my_hash, :files

  def initialize(*files)
    @files = files
    checksum
  end

  def checksum
    
    @my_hash = {}
    files.each do |file|
      # for files already we want to store
      # create a checksum, create a key value pair
      # :checksum => file
      md5 = Digest::MD5.new
      md5 << file.read
      @my_hash[md5.hexdigest] = file
    end
    @my_hash
  end

  def unique?(file)
    # to check if a file is unique compared to the 
    # rest of the system
    md5 = Digest::MD5.new
    md5 << file.read
    # will return true if the file's checksum is unique
    # else, => false
    !my_hash.has_key?(md5.hexdigest)
  end

end

#load our files into memory
one = File.open("test1.txt", "r")
two = File.open("test2.txt", "r")
three = File.open("test3.txt", "r")

#create a new checker instance
check = Checker.new(one, three)
# check if new file is unique
puts check.unique?(two)
``` 
Since the checksum value already exists in the hash, the `check.unique?(two)` returns false.

[More on MD5](http://en.wikipedia.org/wiki/MD5)
---
layout: post
title: "Bash: How I learned to stop worrying and love the Terminal"
date: 2015-02-15 07:34:58 -0500
comments: true
categories: 
---
## Background
When you first had a computer, how did you use it? Some of that will depend on when you were born. My family first purchased a computer (custom built IBM Compatible Clone!) when I was 9, in 1992. It did not have Windows. It was DOS. 

[DOS was Microsoft's version of UNIX.](http://en.wikipedia.org/wiki/DOS#Origins) 

![DOS](http://lh5.ggpht.com/_dNWdTN1SaKE/SpKeAYT9cyI/AAAAAAAAAN8/f-ev43N3Cjk/msdos71%5B6%5D.png?imgmax=800)

One time, I thought it would be funny to delete the contents of my computer's [autoexec.bat](http://en.wikipedia.org/wiki/AUTOEXEC.BAT) file. This was basically a set of plain text instructions for what the computer should do when you turned it on. Turns out "nothing" is what happens when you delete those contents. Literally. We had to call a tech-savvy neighbor who brought a copy of his file to rewrite ours. 

At 9 years old I was opening files, making new directories, and searching all from the DOS prompt. But then Windows happened, I got into Apple products, and was spoiled forever. Typing clunky commands seemed pretty archaic compared to pointing at what you want and getting it on demand, and less precise. I thought UNIX would be lost on me forever. 

## Enter the Terminal

![GCT](http://upload.wikimedia.org/wikipedia/commons/7/73/Grand_Central_Station_Main_Concourse_Jan_2006.jpg)<br>^*This is a terminal. People are so afraid of Terminal they call this thing Grand Central Station instead.*

This, it turns out, was incorrect thinking. Windows and OSX are the bouncers of the computing world. They tell you what you can do and how to do it. Terminal, on the other hand is like the guy who works at the bar, lets you in when the place is closed and lets drink whatever you want.

{% img left images/bouncer-black-text.png This is your OS%}

The biggest benefit is speed.

Most people in Flatiron School know about tab completion at this point. Quite simply, while in a directory, start typing the name of a file or folder, hit tab, and Terminal will finish your command for you. From your home directory, "$cd doc" *hit tab* becomes "$cd Documents" pretty quickly. If you have multiple files that start with similar strings, there is a helper shortcut for you too. Option+ Tab will give you a list of folders and files in your directory starting with a string such that "$cd d" *option+tab* will yield *Desktop/   Documents/ Downloads/ Dropbox/   dev/* so you'll at least know what you're working with.

The real fun is the speed and convenience that comes from automation. With a few quick key commands, custom Bash scripts, and aliases. You can save yourself major time. Let's look at a few by opening up our bash profile.

### Building our own scripts

* cd ~ 
* "What was the name of that file? *types ls*"
* "Hmm, must be a hidden file" *types ls -a*
* OK. *subl .bash_profile*
* "Man that was annoying, and now I've forgotten what directory I used to be in."

We could simply type *subl ~/.bash_profile* but even that get tedious. So let's add this function to our profile. Now when we type "subl-bash" anywhere in Terminal. And guess what, tab completion works for functions too.
``` bash
  function subl-bash {
          subl ~/.bash_profile
      }
```

That was fun. Let's make another one. What's something you want to do almost every time you make a new directory? Generally its navigate into that directory.

* mkdir my-awesome-directory
* cd my-awesome-directory

I think we can cut that down to one line of code. Like ruby, Bash functions can take an argument (), and then we access them in our function with $. So "mkcd *directory name* will now create a directory and navigate to it one shot.

``` bash
  function mkcd () {
          mkdir $1; #makes the directory
          cd $1;    #navigates to the new directory
      }
```
What's another thing we do as Flatiron students *literally* every day? We clone a git repository, try to remember the name of repo, and then navigate into that directoy. With this script, utilizing the bash command 'basename' [[1]] you can clone and navigate with a single command. Use this script by simply typing "gcl" and pasting the address of the github page you want to clone. 

``` bash
#alias gcl="git clone"  <-- if you have a gcl alias already, make sure to comment it out.

function gcl () {
          git clone $1;   
          cd `basename $1 .git`;   #in bash, everything between backticks 
                                   #will be replaced with the output of the command. 
                                   #basename chops off the path from the file name
                                   # .git removes the file extension 
  }
```
This one's my favorite so far. First, install the [chrome-cli gem](https://github.com/prasmussen/chrome-cli). This gives you command line access to chrome.

Then, add this function to your .bash_profile. Enter "labs" into Terminal and in one shot we navigate to our labs directory, close whatever nonsense we have open in Chrome, and go to the Ironboard login page.

``` bash
  function labs {
          cd ~/dev/web-007/labs-ruby; #this should be the path to wherever your labs folder is.
          chrome-cli close -w; #closes whatever time-wasting tabs you have open
          chrome-cli open http://learn.flatironschool.com/users/auth/github; #opens the ironboard homepage
          clear; #clears the terminal window
  }
```
This next one navigates us to my blog's directory, opens the posts folder in sublime, opens chrome to my local preview, and generates a local preview of the blog. Your function might have to change a bit if you are using a different blogging platform. Good thing Octopress lets use the command line!
``` bash
  function blog {
          cd ~/dev/blog/jeremysklarsky.github.io/; #navigates to my blog's directory
          subl ~/dev/blog/jeremysklarsky.github.io/source/_posts; #opens up the posts folder in Sublime
          chrome-cli open localhost:4000; #Opens up live preview page in Chrome
          rake preview; #generates live preview (give chrome a second for the preview page to refresh)
  }
```

And for good measure, let's add some custom aliases. 
``` bash
  alias web='cd ~/dev/web-007'
  alias dev='cd ~/dev'
  alias ruby-labs='cd ~/dev/web-007/labs-ruby'
```
Now we have one word shortcuts to get our most commonly used directories. Think how much time we can save just over the course of the semester with a few of these shortcuts. The Terminal is our oyster.

![Bruce](http://collegetimes.com/wp-content/uploads/2013/09/bruce-almighty.gif)

References:<br>
1. http://unix.stackexchange.com/questions/44735/how-to-get-only-filename-using-sed

 [1]: http://unix.stackexchange.com/questions/44735/how-to-get-only-filename-using-sed


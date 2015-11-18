---
layout: post
title: "The Good Parts"
date: 2015-11-18 15:53:19 -0500
comments: true
categories: 
---
I recently finished reading "Javascript: The Good Parts" in an effort to expand my understanding of Javascript, the language I've used on a daily basis since starting as a UI developer at MediaMath.

I was a little let down by the experience. Much of it was familiar to me - as I've intuited a great deal from writing Javascript for the better part of the last 6 months. The "good parts" to which the author refers are mostly a set of flexible capabilities that allow for a degree of customization on the part of the user. The author highlights two such categories, namely, assigning Functions to objects and how the language handles inheritance. 

I was hoping for more insight into the inner workings of the language. The title is a misnomer and should perhaps be entitled "Javascript: Overcoming the Bad Parts." The author demonstrates a series of patterns he has implemented to make assigning functions and creating class-like objects a bit easier. Perhaps I was expecting something different.

Instead, I thought I would highlight my new understanding of one of the fundamental aspects of Javascript I'd heard but scarcely understood. The curicculum at Flatiron School was largely based around Ruby, its ecosystem, and the Rails MVC framework. Javascript, at least while I was there, was just used as a way to manipulate the DOM and make our sites interactive. 

###Functions: First or Second Class Citizen?

I was able to grasp pretty quickly what people meant by "Objects being first class citizens" in Ruby. While a "Class" is a technically a special type of object whose class is "Class," this is a bit reductive. Most Ruby programming, especially in a Rails environment, is organized around classes. Every instance of a class is an object and they all have the same abilities. All objects inherit directly from their class. Their only differences are the literal values stored in instance variables. We can pass objects around and store objects in variables - anything with a return value of any kind (which is itself an object). They have to be literal "things." A method is not a literal thing - it is a set of instructions that when executed, are turned into a thing.

Javascript is different. We are told that functions, not objects, are first class citizens in Javascript. I never quite understood what that meant. 

As Crockford puts it:
>The lineage of an object is irrelevant. What matters about an object is what it can do, not what it is descended from.

You can make class-like objects in Javascript that have custom functions, constants, and defaults. Then you can make other objects inherit from that object - whether prototypical or pseudoclassical. It's not really important. But something really caught my eye in the chapter about arrays.
>Because an array is really an object, we can add methods directly to an individual array.

Mind === blown. In Ruby, the only way to add an Array method would be to extend it to the entire class from which the object descended. Then, ALL arrays in the program would have access to that method. In Javascript, once an object is created, it doesn't really communicate back up to the mothership. We can assign a function to a single array and it would *only* affect that specific object.

###Examples

So, for example, if you had an array, `var myArray = [1,2,3];`, you could define a function on that array:
```javascript
myArray.myLength = function() {
  alert(this.length)
}
```
This helped me get to an understanding of the difference between Ruby and Javascript and their treatment of objects and functions. Take an object in Ruby, say, `person = Person.new`. To set a property, or instance variable, we'd need instance method that would allow us to set that instance variable: 

```ruby
Class Person
  def instance_variable=(arg)
    @instance_variable = arg
  end
end
```

That variable could only be another object - a standard data type like a `String`, `Array`, `Float`, or an instance of another class. But it would have to be an object.

In Javascript, we not only don't need permission from the class to set any property on our objects (no more of those nasty `No Method` errors!), but the value can either be literal data types or *functions* themselves. That's why we're able to make a custom function and assign it to an array. There's no magic happening with the Array Prototype, we're simply setting a property on the array, like `length`, only our array doesn't care if the property is equal to a string, a number, an object, another array, or even a function!

Here's the fun part. We can actually see the evidence! In a browser console, when we write `console.dir(myArray)`, we get a very interesting output.

```javascript
0: 1
1: 2
2: 3
length: 3
myLength: ()
__proto__: Array[0]
```
The array itself has 3 numbered properties - also known as the indeces of the array (`myArray[1]`, for example), a property called `length`, and a property called `myLength`. That's the function we made. Undeneath that is `__proto__: Array[0]`. I won't list all the details, but expanding that list will show us every function and property our array inherited from the Array Protoype object that birthed it. `myLength()` is stored as a property on the object. The object still has access to all the functions it got from its parent object, but that list is untouched. This is the proof that you can not only store functions in variables, but you can set them as property values in objects. That's the guts of how functions get "first class citizen" status in the javascript universe.

Of course, we could have always defined myLength as a function on `Array.protoype`. This would have extended the function all other arrays. Another one of Javascript's "good parts" is that allows for both styles of inheritance. 

###Callbacks! A Good Part!

Similarly, Ruby only allows objects to be passed into methods as arguments, but because of this feature in Javascript we can pass functions in as arguments. Because these functions don't have to be invoked, or evaluated, until they are called, we can pass in functions as callbacks. They don't even have to be named, we can declare an anonymous function right there in the invoking of another function.

I've been using this stuff without even realizing it. Take this example from one of our Backbone views:

```javascript
    orgId: -1,
    defaultEntity: 'advertisers',
    segments: [],

    stripId: function (str) {
      if (str) {
        var id = str.match(/[(][0-9]+[)]/);
        return $.trim(str.replace(id, ''));
      }

      return null;
    },
```

In like 10 lines of code, we have examples of 4 properties, each a different thing: an integer, a string, an array, and a function. Javascript doesn't care. This gives you such incredible flexibility. That, to me, would constitute a "Good Part!"

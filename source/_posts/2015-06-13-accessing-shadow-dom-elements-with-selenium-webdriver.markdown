---
layout: post
title: "Accessing Shadow DOM Elements with Selenium Webdriver"
date: 2015-06-13 10:33:35 -0400
comments: true
categories: 
---
[Web components](http://webcomponents.org/) are a fantastic way to speed up front end development and standardize design aspects across a large team. But they pose a huge problem when it comes to writing tests. 

Shadow DOM elements are not accessible to selenium-webdriver's traditional means of searching the window for HTML elements. Using javascript and selenium's `executeScript()` function, you can find the element or value that is hidden by a shadow DOM and return its value or WebElement.

The easiest nut to crack (cross your fingers) is when a site has already incorporated jQuery. This will allow you to use a `$` selector to access elements within the shadow DOM. This is one of those programming problems where I spent hours working out the solution and the answer was a single line of code. Assuming you've named your selenium-webdriver instance `driver`, you can simply run the following:

``` javascript
driver.executeScript("return $('body /deep/ <#yourSelector>')")
```
What this does is have selenium execute a literal javascript in your browser instance. By putting `return` at the front of the javascript, it will send a value back to your test file. 

The command above will return a web element. In the case of the test I was writing, I wanted to check to see if some text matched my expectation. (Author's note: test was written in Javascript using the mocha framework and Chai 'should' assertion syntax).

``` javascript
driver.executeScript("return $('body /deep/ ._mm_column')[0].textContent").then(function(title){
  title.should.contain(segmentName);
}); 
```
That's it. Now I can access the shadow DOM.
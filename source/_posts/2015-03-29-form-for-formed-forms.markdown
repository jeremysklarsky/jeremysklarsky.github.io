---
layout: post
title: "form_for formed forms"
date: 2015-03-29 23:52:28 -0400
comments: true
categories: 
---
Over the weekend, in building a super-fun, and very simple one page app called [Am I Ruby?](http://www.amiruby) with Kate, Rachel, and Sophie, we stumbled upon a most curious feature of Rails.

Embracing object oriented design to the best of our abilities, we attempted to implement the so-called "fat mode, skinny controller" principle. Simply, most of the application specific logic should be in a model and the controller should serve only as a traffic cop routing information to and from views.

Seeing as this was a simple one page and no immediate need for persistence, our `search` model was a tableless. In other words, it was a plain old Ruby class.

After much of our application was built, we set up our controller: 
``` ruby
class SearchesController < ApplicationController

  def index
    @search = Search.new
  end

  def show
  end

  def create

    @result = Search.new.am_i_ruby(search_params)    
    respond_to do |f|
      f.html
      f.js 
    end
  end

  private

  def search_params
    params[:search][:keyword]
  end

end
```
The controller would define a new `search` model in order to serve up an object to the `form_for @search` in our main page's view.
``` ruby
    <%= form_for(@search, remote: true) do |f| %>
      <%= f.text_field :keyword, id: 'keyword' %>
      <%=f.submit "Search"%>
    <% end %>
```
We were feeling pretty good...until we saw this: `undefined method 'model_name' for #<Search:0x007fbc18f959d0>`. OUCH!! Especially since we never even wrote a method `model_name`...what gives?

A quick glance at the trace shows us this:
{% img images/Errors.png %}

Actionpack, activeview, activesupport...a lot of things I kind of recognize but am not super familiar about. About halfway down this huge trace I finally see one I do know about for sure: `activerecord`. Since our `Search` class was a tableless class, we never told it to inherit from ActiveRecord. So at some point, `form_for` tried to call `model_name` on a class that doesn't have access to that method. Let's assume that has something to do with ActiveRecord.

### Model Name ###
`model_name` is a part of the ActiveModel module that helps rails with its naming, magically pluralizing and singularizing our model names at will. How do we fix this? We have to give our class access to this method
```ruby
class Search
  extend ActiveModel::Naming
end
```
Refresh the page and we get a new error: `undefined method 'to_key' for #<Search:0x007fbc1849fcb0>`. `to_key` is also an activemodel method but not in the `Name` class, its in the `Conversion` class. What it does is return an array of all the object's attributes as keys. We're going to assume that those will later be used in `form_for`'s creation of the params hash.

```ruby
class Search
  extend ActiveModel::Naming
  include ActiveModel::Conversion
end
```
Refresh again, and we get another error: `undefined method 'persisted?'`. We know the drill by now: `persisted?` is an activerecord method that returns a boolean that tells us whether or not an object has been persisted to the database. At this point we have a couple of options.

We can define `persisted?` and basically be done.
``` ruby
class Search

  extend ActiveModel::Naming
  include ActiveModel::Conversion
  
  def persisted?
    false
  end

end
```
That's not too bad. But I have an inquiring mind and want to keep pushing this further.

We can keep going down the rabbit hole of including/extending modules based on the error messages we find.
```ruby
class Search

  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveRecord::Persistence
  include ActiveRecord::Core
  extend ActiveRecord::ModelSchema::ClassMethods
  
end
```
But since I've now added 5 modules, 3 of which are a part of ActiveRecord and its well after midnight and there's no end in sight, I'm starting to get really tired of this.

When we look at the [activerecord](https://github.com/rails/rails/blob/master/activerecord/lib/active_record.rb) source code, we find that `require 'active_model'` is literally the third line. Why not just make it inherit from ActiveRecord?
```ruby
class Song < ActiveRecord::Base  
end
```
So even though our model is not going to talk to a database, we can still give it all the functionality of 'tabled' class that allows form_for to do its thing. But, if you're afraid this might confuse another developer (or future you), this is all you need to add to your class to allow it to interact with `form_for`:
``` ruby
class Search

  extend ActiveModel::Naming
  include ActiveModel::Conversion
  
  def persisted?
    false
  end

end
```






---
layout: post
title:  "Sinatra Poolite"
date:   2017-05-02 07:32:59 +0000
---


When I first read the assignment, I wasn't what to make my website about. Took me maybe an entire day till I found somthing that I was interested in. While browsing reddit I came across a "CrazyIdea" to make a website like yelp where you review restuarant restrooms instead of their foods. I thought it was a great idea and ran with it. 

This inital setup for the site was the easiest part, I used `bundle gem poolite`  to auto generate the folders and file paths. From there I simply copied the environment, config, and Rakefile from previous sinatra labs. Once this was done came the real challenge, from here I was pretty much on my own.

The next thing I focused on was what are my models and how do they relate to each other. Since this is an app for reviewing restrooms, users, restrooms, and reviews are the best choice. I chose restrooms over bathrooms because I thought I might need a join table for thier interaction and I thought `class Reuseviews` rolls off the tongue better than `class Bathreviews`. However it turns out that I didnt actually need a join table because User and Restrooms shared a has many interaction through reviews while reviews didnt has a one way belongs to relationship. 

I was not sure if I should have given users a has_many relationship with restrooms through reviews and vice versa because I didnt really see them having any direct interactions while planning my app. However this was still in the early stage and I was worried that I might have overlooked something so I created the interacting between the two just to give myself more leeway in the future. Turns out I didnt need to.

While creating my models I made the mistake of pluralizing users and restrooms in my Review class. 
```
class Review < ActiveRecord::Base
  belongs_to :users
  belongs_to :restrooms

  def average
    self.reviews.collect { review.star }.reduce(:+)
  end

end

```

This ended up messing up my database tables and prevented any of my objects from communicating with each other since Reviews object acted as both a join table and class. I figured this out later in down the line when simple commands such as 
```
review = Review.all.first
review.user.username
```
would return nil. It took me awhile to figure out that this was the problem. I didnt know anyway to fix this other than deleting my entire database and redoing all my migrations, so thats what I did. This took quite sometime todo because I would always get an error message whenever I tried to delete it from terminal using
```
rake db:schema:cache:clear  
rake db:reset
```

and various other rake commands. In the end a learn expert suggest that I manually delted the schema and development schema files. I didnt really want to do it this way but figuring out the error message was taking up too much time.

The setup for the login/signup of user accounts didnt take too long and felt natural to me since I did it so often in previous labs. The real challenge came with validating user permissions once they were signed into the app. I noticed that even someone was not logged into the app there was still an option to edit/delete/create a review on various pages on my site. Thats when I remembered about helper methods.  
```
  helpers do
    def current_user
      User.find(session[:user_id]) if session[:user_id]
    end

    def logged_in?
      !!current_user
    end

    def my_review?(review)
      current_user.reviews.include?(review)
    end

  end

```

With these helper methods I validated every post and get command that excuted by someone on my site. For example 
```
   post '/delete/:id' do
     if logged_in?
       @user = current_user
       @review = Review.find(params[:id])
       if current_user.reviews.include?(@review)
         @review.destroy
       end
       @my_review = my_review?(@review)
       redirect to "/user/#{current_user.id}"
     else
       redirect to '/login'
     end
   end

```

I made sure that a user was logged and owned a review before they could delete said review. 

Even though on the server side my controller ensure that no one unauthorized could edit, create, or delete a review my views didn't reflect that. The options to edit/delete were persitent on show pages. Even though they a user couldnt edit a review that was not their, there still was a link to do so on each page. to circumvent this, I removed the hard coded delete/edit links and placed them inside if statments on the show pages.  
```
<% if @my_review %>
  <form class="" action="/delete/<%=@review.id%>" method="post">
    <input type="submit" value="Delete">
  </form>

  <a href="/review/<%=@review.id%>/edit">Edit</a>
<% end %>
```

Now these options are only visible to a user if they were the one who created the review. I also did something similar for signup and login links on each page. That way somene isn't given the option to signup or logut if they are already logged in or signed out respectively. 

I think my favorite feature of my site is the option to create a new review. 
```
   get '/review/new' do
     if logged_in?
       @user = current_user
       @logged = logged_in?
       @restrooms = Restroom.all
       erb :'/review/write_review'
     else
       redirect to '/login'
     end
   end

   get '/review/new/:id' do
     if logged_in?
       @restroom = Restroom.find(params[:id])
       @user = current_user
       @logged = logged_in?
       erb :'/review/write_review'
     else
       @restroom = params[:id]
       erb :login
     end
  end
	
   post '/new/review' do
     if !params[:review][:stars].blank?
       @user = current_user
       review =Review.create(params[:review])

       if !params[:restroom][:restaurant_name].blank? && !params[:restroom][:location].blank?
         review.restroom = Restroom.create(params[:restroom])
       end

       review.save
       redirect to "/review/#{review.id}"
     else
      if !params[:review][:restroom_id].blank?
        flash[:message] = "Hmm it seems you forgot to put a star rating."
        redirect to "/review/new/#{params[:review][:restroom_id]}"
      else
        flash[:message] = "Please make sure to choose a location or add a new one and also a star rating"
        @my_review = my_review?(review)
        redirect to '/review/new'
      end
     end
   end
```

The first method is simply if you would like to create a review and you are not already on a restrooms show page. This will bring you to a form where you can write a review and choose from existing restrooms or add a new restroom. The brilliance comes from the second and third method where if a user is already on a restrooms show page and click the option to write a review, they will be redirected to a sepereate form with the restuarants information already filled in for the user. Not only that if a user was not logged in when they attempted to write a review from a restrooms show page they will be redirected to the new review form after loggin in instead of being shown their profile page. The best part is both of these forms are seved on the same show page, but only one is seen depending on the circumstances. Here is a snippet of the show page form. 
```
<% if @restrooms %>
  <h2>Review A Restroom</h2>
  <form action="/new/review" method="post"></p>
    <p>Choose an existing location</p>
    <% if !@restrooms.blank? %>
      <% for item in @restrooms %>
      <p><input type="checkbox" name="review[restroom_id]" value="<%=item.id%>"><%=item.restaurant_name%></input></p>
      <% end %>
      <% end %>
      <p>Or Add a new location</p>
      <p>Name: </p>
      <p><input type="text" name="restroom[restaurant_name]" value=""></p>
      <p>Location: </p>
      <p><input type="text" name="restroom[location]" value=""></p>
      <p>Write Review Here:</p>
      <input type="text" name="review[body]" class="review body" required>
      <p>Rating: </p>
      <ol>
        <% for item in (0..4) %>
        <li><input type="checkbox" name="review[stars]" value="<%=item+1%>"></li>
        <% end %>
      </ol>
      <p><input type="hidden" name="review[user_id]" value="<%=@user.id%>"></p>
      <p><input type="submit" value="Create Review"></p>
    </form>
<% else %>
  <h2>Write a review for <%=@restroom.restaurant_name%> below</h2>
  <form action="/new/review" method="post"></p>
    <input type="hidden" name="review[restroom_id]" value="<%=@restroom.id%>">
    <input type="text" name="review[body]" class="review body" required>
    <p>Rating: </p>
    <ol>
      <% for item in (0..4) %>
      <li><input type="checkbox" name="review[stars]" value="<%=item+1%>"></li>
      <% end %>
    </ol>
    <p><input type="hidden" name="review[user_id]" value="<%=@user.id%>"></p>
    <p><input type="submit" value="Create Review"></p>
    </form>
<% end %>

```

Aside from the coding I ran into a very frustrating problem with my github repository. While making this site I created two sepereate branches for  editing and testing new feature. Somehow along the way things got very messy with the merger of my branches due to conflicts. I didnt really know how to resolve conflicts so a lot of my files because corrupted and started looking very wierd on github so I decided to make a brand new repository and copied over all the good files in a version 2 of my site. This experience made me very histant to use github branches while completing the app in the new repository. There is still so much I need to learn!

Thats pretty much it for my site. Overally I am not very satisfied with my work, There are a lot more things I would like to add mostly I would like for it to be more aesthiticly pleasing. As of right now it looks very boring for somthing that is suppose to be a fun lightheated bathroom review site. As of right now I just dont have the time to make it more appealing but I plan to revist this and make it even better and most of all pretty in the near future.

*Written in a sleep haze, and not proof read sorry for the errors if there are any :(*

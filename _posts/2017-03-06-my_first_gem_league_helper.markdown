---
layout: post
title:  "**My first Gem, League Helper**"
date:   2017-03-06 17:23:45 +0000
---


When this project first assigned I had trouble figuring out what I wanted to make my gem about. After about thirty minutes of brainstorming with no luck. I decided to play league of legends for a bit and try again with a fresh mind. After a few games I decided to look up a champion build the character I wanted to learn in League. While looking up that information I thought to myself "Hmmm why don't I make a gem that does this for me!"

So to start I knew I wanted users or players to be objects alongside champions. However, I was not sure if I wanted each champion to be a unique object like the users. The problem is that if they are unique, the interaction between users and champions becomes tricky. because the "has many" property applies to both champions and users. This is because the same champion can be played by many users and users play many champions. So to circumvent this problem I made a new instance of the same champion for each user, making only the users unique. 

The first class I made was the summoner and champion class. This is because I wanted to list out all the information that I wanted both classes to have. Then I made a scraper class that would scrape all the relevant information. My scraper class only has class methods. I did this because I only need it scrape the information I need. After scraping the necessary information there is no further interaction between the scraper class and my other objects. This also saved memory space since the class behaves more like a method call than an object creation.

Another problem I faced was with loop with the command line interface of the program. It was difficult getting my cli to exit when after looking up a new summoner. When a user started the gem they could exit by typing ‘exit’ only if they have looked up just one summoner. If they looked up a second summoner it would start up a new instance of my cli loop. So the user would need to exit the loop twice, or once for each summoner look up. To solve this I made the input variable used by the cli loop an instance variable. This solved the problem because now the input variable is not unique to each method. So an input of exit would apply to all methods including both instances of the cli loop. 

This is my first gem and the first project where I had complete creative freedom. It is also the first time I felt like I was on my own. With that said it took me a while to make this, much longer than I anticipated. IT was also frustrating at times, but I am glad I stuck with it and am satisfied with my final results. 

+++
title = "Connect4 App"
date = "2025-01-23"
draft = false
cover = "/images/connect-4-cover.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++
[Website](https://connect4.carladi.com)
[Github](https://github.com/asdiAdi/connect4)

I made a full-stack connect 4 game hosted on AWS EC2 using React and Nestjs.
<!--more-->

{{< details summary="TLDR" >}}
- I made this because of boredom.
- Technologies used: [Vite](https://vite.dev/), [Next.js](https://nextjs.org/), [Postgresql](https://www.postgresql.org/), [Docker](https://www.docker.com/), [Sockets.io](https://socket.io/)
- Hosted on [AWS](https://aws.amazon.com/)
{{< /details >}}

One day I was bored at work because my tasks are getting monotonous.
It was the same frontend task every day for the past months.
I was worried about my career because being a web developer means that I should know about the Frontend and Backend side of development.
At this time, I only did frontend tasks and can barely remember how to do backend.

So I decided to make some project at home to kill some time and re-learn everything.
A basic crud is boring, I wanted to make it enjoyable and what is more enjoyable than programming a game?
I researched and pondered in making a Unity or Godot game but ultimately decided that it would take too long.
It isn't even related to web development haha.

So why a connect4 game? Well, because it's easy to understand, multiplayer and I found a decent design on [frontendmentor](https://www.frontendmentor.io/challenges/connect-four-game-6G8QVH923s).
I downloaded the assets and started working.

In this project I learned how to use sockets.io and postgres for database.
Why use a database? Because I initially planned to add some features like History, User Data, Matchmaking etc.
But as the project continues, it became clear that it would take a long time to do for a small sized project meant to be made for fun.
I still kept the invite user to play though, even if it doesn't work most of the time.

I hosted it on AWS after learning how to make an ec2 instance.
I was worried about the costs, so I made a fail-over site and set it up such that the ec2 instance will only stay up for 1 hour before shutting down.
You can read more on how that works [here](https://carladi.com/posts).
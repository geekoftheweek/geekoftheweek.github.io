---
layout: page
title: Projects
permalink: /projects/
---

Information about a few of the projects I've worked on over the years, along
with whatever documentation I can scrape together.

### Answers Overboard!

<iframe width="560" height="315" src="https://www.youtube.com/embed/W5YpuRtqsj0" frameborder="0" allowfullscreen></iframe>

A 2D Unity-based game I made for Thinksy's February 2015 hackathon.  The idea
is to steer a helicopter to pick up answers that are floating in water, and
drop them onto the rescue ship.  I was proud of the game, but accidentally made
the flight speed dependent on the frame rate, meaning it was unplayable on
devices that ran it more slowly than my test hardware!  Suffice it to say, I
did not win the hackathon.

### Global Game Jam Entry

<iframe width="560" height="315" src="https://www.youtube.com/embed/FICBTcWW3s0" frameborder="0" allowfullscreen></iframe>

A 2D Unity-based game my team and I made in 48 hours for the 2015 Global Game
Jam.  I was one of only two programmers on the team.  The idea is that you play
a pixel on its journey through various retro arcade games.  Win a game and you
transition to another one -- lose, and you take a different path, to a
different game.

### Leap Motion SumoBot

<iframe width="560" height="315" src="https://www.youtube.com/embed/8q7y4OCPeJc" frameborder="0" allowfullscreen></iframe>

A project I made at RobotsConf 2014.  The hardware is a standard SumoBot I
assembled from a kit, and the software implements custom proportional steering,
using Leap Motion with the Johnny Five javascript library.

### Nerf Blowgun

A very cheap, very fun toy made with less than $3 of hardware.  Long firing
range, and *shockingly accurate* as Nerf guns go.

### PyWeek Entries

<img width="280" src="https://pyweek.org/media/dl/7/StreetSurf/final.png" />
<img width="280" src="https://pyweek.org/media/dl/10/SpareTime/EasterWobble.png" />

A couple of tech demos that I never finished. The first is [a cat getting
pulled on a trash can lid][street-surf], from 2008.  Make him jump over
obstacles -- but keep him fuzzy-side-up, or he loses one of his nine lives. The
other is [a little wobbling easter egg][easter-wobble] I made in 2010.  He can
hop, lean, and shoot jelly beans with a kick.  He gets nervous if it looks like
he's going to fall over, and then loses a life if he does.

### Transposition Cipher cracker

A tool to crack a transposition cipher using a genetic algorithm.  I took a
codemaking and codebreaking college course around 2008, and I had a hard time with one
of the homework assignments, so (with permission!) I wrote this software to
solve the cipher for me.  It didn't find the complete solution, but it found
enough of it that the rest was easy to solve ...so, mission accomplished!

### Electroluminescent Gloves

A project I made while home from college for spring break in 2005 or 2006.
Cotton gloves with electroluminescent wire threaded along the palm and fingers,
for using sign language in the dark.

### COTA notifications

A really basic hack, way back in the early 2000's I think, to let me know when
the bus was running late.  If it was on schedule or if class ran late, I
wouldn't have time to make it to the bus stop -- it would be a long wait until
the next one, I might as well leave the stop and wait somewhere else warm.  On
the other hand, if the bus was late and class ended on time, I could *just*
catch the bus, and get home quickly.

The transportation authority had a web site where riders could check the bus
locations, but I didn't have a smartphone.  What I did have was Python, and an
email address that would forward (as SMS) to my phone.  So, I hacked together a
script that would scrape the GPS location of the bus I wanted to catch, and
text me if it was running late, so I'd know whether to run to the stop, or give
up and go to the library.

[street-surf]:   https://pyweek.org/e/SpareTime/
[easter-wobble]: https://pyweek.org/e/StreetSurf/

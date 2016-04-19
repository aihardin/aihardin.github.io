---
author: aihardin
comments: true
date: 2014-08-25 22:55:28+00:00
layout: post
slug: bay-bridge-during-an-earthquake
title: What is the likelihood I'll be on the Bay Bridge during an earthquake?
wordpress_id: 180
tags:
- antimap
- bay bridge
- earthquake
- GIS
- GPS
- quantified self
---

The earthquake early Sunday morning woke me and most of the Bay Area up wondering if a this was the Big One geologist have been warning about.  This quake on the West Napa fault was only 6.1 according to the [USGS](//earthquake.usgs.gov/earthquakes/eventpage/nc72282711#summary) but that is not the only fault in the area.  The big worry is the Hayward fault that runs behind my house, through the UC Berkeley Memorial Stadium, past Oakland, and goes down to Fremont.  This fault is pent up, overdue, and UC Davis geologist John Rundle gives 1:5 odds that it will release a 7.0 quake [within the next few years](http://www.sfexaminer.com/sanfrancisco/seismologists-say-bay-area-is-due-for-major-earthquake-that-could-cripple-region/Content?oid=2172405). This is one reason that the Bay Bridge was replaced in 2013, the old structure was crippled in the 1989 quake and was unlikely to survive another serious shake.  

The new eastern span of the Bay Bridge is very pretty, cost $6.4 Billion, and has been plagued with safety concerns that I'm not qualified to judge.  However, I do cross it to get to work and every so often worry about being on it during a quake.  In order to answer the question of what are the chances I'll be on it during a quake, I looked at my GPS logs that I've collected with the [Antimap](http://theantimap.com/category/applications/antimap-log/) iPhone [app](http://itunes.apple.com/app/antimap-log/id486524181). This collects time points at a rate of 30 per second and was designed for sports use but I keep it running whenever I'm traveling. Since early 2014 I've been a postdoc at UCSF and have a log of every crossing.

![Screen Shot 2014-08-25 at 3.22.06 PM]({{ site.url }}/assets/Screen-Shot-2014-08-25-at-3.22.06-PM.png)

Using the [gpxpy](https://github.com/tkrajina/gpxpy) python module I extracted all tracks from January to now that passed those the points on the map.  These correspond to the exit of the Treasure Island tunnel and the point that the Bay Bridge leaves the ground.

![segments]({{ site.url }}/assets/segments.png)

So how long is each trip across the new bridge?  The mean time spent is 146 seconds, just over two minutes each way.  In total, I've spent 5.53 hours crossing that part of the bridge this year.  The logs span 5352 hours so I spend around 1/1000th of my time there. There is a 99.896% chance that I will NOT be on the eastern span at any given time.  Obviously the solution is to go faster across the bridge.

[![time_on_bridge]({{ site.url }}/assets/time_on_bridge.png)

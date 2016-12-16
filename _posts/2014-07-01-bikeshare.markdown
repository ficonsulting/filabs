---
layout:     post
title:      "Capital Bikeshare"
subtitle:   "Visualizing Washington DC biking activity"
date:       2016-11-17 12:00:00
author:     "David Leonard"
header-img: "img/bike.jpg"
---

<p>Capital Bikeshare has provided almost 15 million bike rides across the greater Washington, DC, area over the six years it has been in existence.  That's a lot of happy commuters and tourists avoiding the gnarly DC traffic and the unpleasant bowels of the Washington Metro.</p>  

Where are all of these happy bicyclists pedaling?  We wanted to animate the entire six year history of Capital Bikeshare superimposed on a map of DC, and  then provide a time lapse over a couple brief minutes.  Unfortunately, readily available computing power constraints prohibited us from doing that, and readily available human brain power prohibited us from processing 15 million dots and lines racing across the computer screen anyway.  So instead we chose a shorter time period and provided all the bike rides for that period.  Perhaps one of those dots represents you and your borrowed big red bike.

Here it is: [Capital Bikeshare Visualization](https://dal1234.github.io/)

But you still might want to know more about the six year history of the program.  Here are a few nuggets of data mined from the 14 million rows of data generously made public by Capital Bikeshare.

<h2 class="section-heading">Busiest stations</h2>

Columbus Circle / Union Station is the busiest station in all of DC with 294,046 trips from this station.  This was followed by Massachusetts Ave & Dupont Circle NW and Lincoln Memorial.

On April 19, 2016, 362 trips originated from Columbus Circle.  Here are all of them animated.

<iframe src="{{ site.baseurl }}/html/bikeshare_loop_busiest.html" 
width="700" 
height="400"
scrolling="no"
frameborder="0"></iframe>

<h2 class="section-heading">Technical notes</h2>

Data were obtained from a data set made public by Capital Bikeshare and processed using R.  Bike paths were determined using a Google Maps API.  Visualizations were created using the d3 JavaScript library.
---
layout:     post
tags: R
title:      "Capital Bikeshare"
subtitle:   "Visualizing Washington DC biking activity"
date:       2016-11-17 12:00:00
authors:    david_leonard
header-img: "img/bike.jpg"
permalink: /:title
comments: true
---

<p>Capital Bikeshare has provided almost 15 million bike rides across the greater Washington, DC, area over the six years it has been in existence.  That's a lot of happy commuters and tourists avoiding the gnarly DC traffic and the depths of the Washington Metro.</p>  

Where are all of these happy bicyclists pedaling? We wanted to animate the entire six year history of Capital Bikeshare superimposed on a map of DC, and then provide a time lapse over a couple brief minutes. To ease the burden on your eyes and our server of processing 15 million paths, we chose to animate the rides over a shorter time period.  Click the link to see the bike rides during the afternoon of New Year’s Day 2011.  Perhaps one of those dots represents you and your borrowed big red bike.

[Capital Bikeshare Visualization]({{ site.baseurl }}/html/capital_bikeshare/index.html)

Here are additional items of interest gathered from the 15 million rows of data generously made public by Capital Bikeshare.

<h2 class="section-heading">Ridership over time</h2>

Ridership has increased considerably over the last six years, from fewer than 2,000 riders on the busiest day in 2010 to over 12,000 on the busiest 2016 day. 

![image-title-here]({{ site.baseurl}}/img/capital_bikeshare/daily_riders.png){:class="img-responsive"}

Warm summer months have seen a particular increase in bicyclists over time.  In fact, the frigid DC January and February months saw mostly flat ridership until 2016.

![image-title-here]({{ site.baseurl}}/img/capital_bikeshare/riders_by_season.png){:class="img-responsive"} 

Capital Bikeshare offers two ways to rent bikes.  One way is designed for visitors to DC, where riders can purchase a casual membership for one to five days.  The other is a registered members for locals for either monthly or annual membership. Registered membership has taken off since 2010, whereas the growth in casual ridership has been more measured.

![image-title-here]({{ site.baseurl}}/img/capital_bikeshare/riders_by_type.png){:class="img-responsive"} 

By viewing ridership by both season and rider type, we see that the growth in registered users has come from summer months.  In fact, winter usage among registered users has actually decreased each year since 2013 and only as recently as 2016 saw an uptick.  

![image-title-here]({{ site.baseurl}}/img/capital_bikeshare/riders_by_season_type.png){:class="img-responsive"} 


<h2 class="section-heading">Busiest stations</h2>

Columbus Circle / Union Station is the busiest station in all of DC with 294,046 trips from this station.  This was followed by Massachusetts Ave & Dupont Circle NW and Lincoln Memorial.

On April 19, 2016, 362 trips originated from Columbus Circle.  Here are all of them animated (refresh your browser if the lines aren't moving).

<iframe src="{{ site.baseurl }}/html/capital_bikeshare/bikeshare_loop_busiest.html" 
width="700" 
height="400"
scrolling="no"
frameborder="0"></iframe>

<h2 class="section-heading">Technical notes</h2>

Data were obtained from a data set made public by Capital Bikeshare and processed using R.  Bike paths were determined using a Google Maps API.  Visualizations were created using the d3 JavaScript library.

All code for processing data can be found on [Github](https://github.com/ficonsulting/capital-bikeshare). Code for bike animations is also on [Github](https://github.com/ficonsulting/ficonsulting.github.io/tree/master/html).

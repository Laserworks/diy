---
layout: post
title:  "How to Create 3D Paper Model of Poľana Volcano"
date:   2015-04-18 20:00:0
thumbnail: images/2015-04-18-3d-terrain-relief/10.jpg
tags:
- 3D model
- paper
- terrain
---

Slovakia has no active volcanos, but we have some inactive ones and probably the most famous is Poľana ([more info](http://slovakia.travel/en/polana-volcano-vulkan-polana), [map](http://goo.gl/8WPb6q)) due to its disctinctive shape of [caldera](http://en.wikipedia.org/wiki/Caldera). 

In this article, I will explain how to create a SVG paper model of the volcano ready for [blade cutter machine](http://www.silhouetteamerica.com/shop). The source codes below can be used to generate such a 3D model of any terrain.


![Poľana Volcano cut from paper]({{site.baseurl}}/images/2015-04-18-3d-terrain-relief/01.jpg "Poľana Volcano cut from paper")

Rear view with caldera well visible:

![Poľana Volcano cut from paper]({{site.baseurl}}/images/2015-04-18-3d-terrain-relief/05.jpg "Poľana Volcano cut from paper")

For a comparison, here is [rendered 3D panorama view](http://www.udeuschle.selfhost.pro/panoramas/panqueryfull.aspx?mode=newstandard&data=lon%3A19.26247%24%24%24lat%3A48.60159%24%24%24alt%3A2000%24%24%24altcam%3A10%24%24%24hialt%3Afalse%24%24%24resolution%3A20%24%24%24azimut%3A64.2%24%24%24sweep%3A70%24%24%24leftbound%3A29.2%24%24%24rightbound%3A99.2%24%24%24split%3A60%24%24%24splitnr%3A2%24%24%24tilt%3A-3.33333333333333%24%24%24tiltsplit%3Afalse%24%24%24elexagg%3A3%24%24%24range%3A20%24%24%24colorcoding%3Afalse%24%24%24colorcodinglimit%3A21%24%24%24title%3AZugspitze%24%24%24description%3A%24%24%24email%3A%24%24%24language%3Aen%24%24%24screenwidth%3A1920%24%24%24screenheight%3A1055):

![Poľana Volcano panorama view]({{site.baseurl}}/images/2015-04-18-3d-terrain-relief/08.jpg "Poľana Volcano")

On the right side is my first prototype assembled with slightly different technique not explained in this blogpost:

Following photo exhibits several prototypes. On left side is my first Poľana prototype assembled with slightly different technique not explained in this blogpost. In the middle is second generation of Poľana and on the right side is [Mt. Everest model](https://petervojtek.github.io/diy/2015/04/20/3d-paper-model-mt-everest.html).

![Poľana Volcano cut from paper]({{site.baseurl}}/images/2015-04-18-3d-terrain-relief/10.jpg "Poľana Volcano cut from paper - evolutions of prototypes")

## How to Produce the SVG for Paper Cutting

We will not create the paper model by hand but use a bit of programming. The source code below is Ruby.
First, we need to determine the latitude/longitude coordinates of a rectangle:

{% highlight ruby %}
lat0, lon0 = 48.60113, 19.29473 # left bottom
lat1, lon1 = 48.70047, 19.52991 # right top
{% endhighlight %}

<iframe width="100%" height="300px" frameBorder="0" src="https://umap.openstreetmap.fr/en/map/polana-bounding-box_36800?scaleControl=false&miniMap=false&scrollWheelZoom=true&zoomControl=true&allowEdit=false&moreControl=true&datalayersControl=true&onLoadPanel=undefined&captionBar=false"></iframe>

and the number of "cuts" through the terrain in `x` (latitude) and `y` (longitude):

{% highlight ruby %}
lat_steps, lon_steps = 80, 24
lat_diff, lon_diff = ((lat1 - lat0) / lat_steps.to_f), ((lon1 - lon0) / lon_steps.to_f)
{% endhighlight %}

Picture below depicts that we create 24 paper pieces in x-axis (that is latitude). For each paper sheet we will take 80 elevation points in longitudal axis so that shape of each paper is as real as possible.

![Poľana Volcano panorama view]({{site.baseurl}}/images/2015-04-18-3d-terrain-relief/09-expl.png "Poľana Volcano")

Now we create a 2--dimensional table called `elevations` (size 80x24), where `x` is latitude, `y` is longitude, and value is elevation in meters. We use [mapquestapi.com HTTP API](http://open.mapquestapi.com/elevation/) to get the elevation:

{% highlight ruby %}
uri = URI.parse('http://open.mapquestapi.com/elevation/v1/profile')
#  you will need to unescape your key characters, e.g. using this site: http://www.w3schools.com/tags/ref_urlencode.asp
params = { 'key' => "your-key-here", 'shapeFormat' => "raw",   }

elevations = []

(0...lat_steps).each do |i|
  points = []
  lat = lat0 + lat_diff * i
  (0...lon_steps).each do |j|
    lon = lon0 + lon_diff * j
    points << lat
    points << lon
  end
  
  params['latLngCollection'] = points.join(',')
  uri.query = URI.encode_www_form params 
  response = uri.open.read
  json_response = JSON.parse response
  elevations << json_response['elevationProfile'].collect{|h| h['height']}
end
{% endhighlight %}

Next we convert the elevations from meters to svg points:

{% highlight ruby %}
one_cm_in_pts = 33 # one centimeter is 33 points in SVG (at least in Inskcape)
z_cms = 6 # we want our model height to be 6 cm
cm_to_svg_point_ratio = one_cm_in_pts * z_cms

ele_min, ele_max = elevations.flatten.min, elevations.flatten.max
ele_diff = (ele_max - ele_min).to_f

elevations_in_pixels = []
elevations.each do |eline|
  eline_relative = []
  eline.each do |e|
    eline_relative << ((1.0 - ((ele_max - e) / ele_diff)) * cm_to_svg_point_ratio).to_i
  end
  elevations_in_pixels << eline_relative
end
{% endhighlight %}


Now we are ready to produce the SVG polylines for cutter machine.
[SVG polyline](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/polyline) is simply a set of `[x,y]` pairs which will be connected by lines.

{% highlight ruby %}
svg_polylines = [] # for each of the 24 paper sheets we want to cut we will create a polyline and store all of them here
total_length_in_south_north_direction_in_cm = 10 # width is 10cm. our rectangle has 3:2 ratio so that length will be 15cm.

y_offset_between_two_slices = 200 # points 
elevations_in_pixels = elevations_in_pixels.transpose # [lat, lon] -> [lon, lat]

x_offset_between_points = ((total_length_in_south_north_direction_in_cm / lat_steps.to_f) * one_cm_in_pts).to_i

(0...lon_steps).each do |i| # for each of the 24 papers sheets
  svg_polyline_points = []
  svg_polyline_points << [0, (y_offset_between_two_slices*i - one_cm_in_pts*2)]
  svg_polyline_points << [0, elevations_in_pixels[i][0] + y_offset_between_two_slices*i]
  
  (0...lat_steps).each do |j| # for each of the 80 elevation points of one sheet
    x = x_offset_between_points * j 
    y = elevations_in_pixels[i][j] + y_offset_between_two_slices*i
    svg_polyline_points << [x,y]
  end

  # we have all the 80 points of one polyline, create corresponding xml element
  svg_polyline = "<polyline points=\"#{svg_polyline_points.collect{|a,b| "#{a},#{b}"}.join(' ')}\" style=\"fill:white;stroke:red;stroke-width:4\" />"
  svg_polylines << svg_polyline
end

{% endhighlight %}

The complete source code and SVG files are [here](https://github.com/petervojtek/3d-paper-terrain-model).

Here are sheets after cutting, before assembly:

![Poľana Volcano cut from paper]({{site.baseurl}}/images/2015-04-18-3d-terrain-relief/06.jpg "Poľana Volcano cut from paper")
![Poľana Volcano cut from paper]({{site.baseurl}}/images/2015-04-18-3d-terrain-relief/07.jpg "Poľana Volcano cut from paper")



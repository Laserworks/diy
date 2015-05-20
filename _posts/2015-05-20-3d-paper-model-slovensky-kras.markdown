---
layout: post
title:  "3D Paper Model of Slovenský Kras (Plešivec)"
date:   2015-05-20 20:00:0
tags:
- paper 
- terrain
- 3D model
- karst
---

Slovenský kras ([Slovak Karst](http://en.wikipedia.org/wiki/Slovak_Karst)) is complex of karst plains and plateaus made of limestone [in south--eastern part of Slovakia](http://goo.gl/3NXmfi). I made a paper model of the area around Plešivec town. 

![3D Paper Model of Slovenský Kras]({{site.baseurl}}/images/2015-05-20-3d-paper-model-slovensky-kras/03.jpg "3D Paper Model of Slovenský Kras")

_Plešivec_ town is located in the middle of the model, the rear--central plain is _Plešivecká planina_ and the right plain is _Silická planina_.
![3D Paper Model of Slovenský Kras]({{site.baseurl}}/images/2015-05-20-3d-paper-model-slovensky-kras/01.jpg "3D Paper Model of Slovenský Kras")

<iframe width="100%" height="400px" frameBorder="0" src="https://umap.openstreetmap.fr/en/map/slovensky-kras_39053?scaleControl=false&miniMap=false&scrollWheelZoom=true&zoomControl=true&allowEdit=false&moreControl=true&datalayersControl=true&onLoadPanel=undefined&captionBar=false"></iframe>

The model itself is made of thirteen concentric circles od paper:

![3D Paper Model of Slovenský Kras]({{site.baseurl}}/images/2015-05-20-3d-paper-model-slovensky-kras/09.jpg "3D Paper Model of Slovenský Kras")

To properly bend the paper into circular shape I had to use thin paper (80g/m) so that the model is not very robust (I typically use stock paper which is 300g/m).

![3D Paper Model of Slovenský Kras]({{site.baseurl}}/images/2015-05-20-3d-paper-model-slovensky-kras/04.jpg "3D Paper Model of Slovenský Kras")

![3D Paper Model of Slovenský Kras]({{site.baseurl}}/images/2015-05-20-3d-paper-model-slovensky-kras/08.jpg "3D Paper Model of Slovenský Kras")

To produce the SVG vector for cutting I employed similar approach to the one used for [Poľana volcano](https://petervojtek.github.io/diy/2015/04/18/3d-paper-model-of-polana-volcano.html#how-to-produce-the-svg-for-paper-cutting) model with few tweaks.

At first I determined latitude and longitude of the center:
{% highlight ruby %}
lat_of_central_point, lon_of_central_point = 48.5521827667489, 20.407024025917053
{% endhighlight %}

Then I decided to have 13 concentric circles, ranging with radius from approximately 1.0 km to 5.0km from the central point and computed latitude and longitude of points on the circumference of each circle via [GreatCircle](https://github.com/mwgg/GreatCircle) library.

{% highlight ruby %}
circles = []

(0..12).to_a.each do |circle_no|
  circle = []
  
  radius_km = 1.0 + circle_no * 0.33333
  circumference_km = 2 * PI * radius_km

  # number of points on the circle depends on its radius - the larger the circle, the more points we fetch 
  # we fetch 63 points for the smallest circle and 314 points for the largest one
  number_of_ele_samples = (circumference_km * 10).round.to_i
  angle_step = 360.0 / number_of_ele_samples
  (0..number_of_ele_samples).to_a.each do |i|
    angle = angle_step*i
    lat_lon_hash = GreatCircle.destination(lat_of_central_point, lon_of_central_point, angle, radius_km)
    lat, lon = lat_lon_hash['LAT'], lat_lon_hash['LON']
    circle << [lat.round(7), lon.round(7)]
  end
  
  circles << circle
end

{% endhighlight %}

Remaining steps are almost similar as with the [Poľana volcano](https://petervojtek.github.io/diy/2015/04/18/3d-paper-model-of-polana-volcano.html#how-to-produce-the-svg-for-paper-cutting) model.


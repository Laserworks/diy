---
layout: post
title:  "Visualize Gravitation Effect with Neopixel Ring and Accelerometer"
date:   2015-01-24 20:00:0
---

The circular share of Neopixel ring makes it perfect to mimick circular movement of drop of water or mercury in gravitation field. We take here output from accelerometer and display it with neopixel ring using arduino.

<iframe width="560" height="315" src="//www.youtube.com/embed/YJ9w8hs3sj0" frameborder="0" allowfullscreen></iframe>



### Bill of Materials

* Arduino Uno
* [Adafruit Neopixel Ring](http://www.adafruit.com/product/1463)
* [Accelerometer ADXL335](http://www.ebay.com/itm/3-axis-Analog-Output-Accelerometer-Module-angular-transducer-for-Arduino-ADXL335-/271499551321?ssPageName=ADME:L:OC:US:3160)
* [DC-DC Boost Voltage Converter 3V to 5V](http://www.ebay.com.au/itm/Mini-DC-DC-Boost-Converter-Step-up-Voltage-Power-Supply-Module-3-5V-to-5V-3A-/171015171769?ssPageName=ADME:L:OU:US:3160)
* Li-on Battery
* 1000 uF capacitor
* 300 ohm resistor

### Wiring

![Visualize Gravitation Effect with Neopixel Ring]({{site.baseurl}}/images/neopixel-gravitation.png "Visualize Gravitation Effect with Neopixel Ring")

### Code

The code is located [in github repository](https://github.com/petervojtek/neopixel-gravitation/blob/master/neopixel-gravitation.ino) along with fritzing diagram.

The accelerometer output is three-axis, but we are interested only in two directions: X and Y.
When we read directly the analog output from the x and y outputs, we receive following data according to rotation of the accelerometer board:

{% highlight ruby %}
x = analogRead(ACCELEROMETER_X_OUTPUT);
y = analogRead(ACCELEROMETER_Y_OUTPUT);
{% endhighlight %}

![Raw output from accelerometer]({{site.baseurl}}/images/neopixel-graph1.jpg "Raw output from accelerometer")

Following video explains the diagram:

<iframe width="560" height="315" src="//www.youtube.com/embed/VIzPnZczjrc" frameborder="0" allowfullscreen></iframe>

Next we convert the raw output to appear on scale between -1 to 1.

{% highlight ruby %}
nx = (x - 375) / 75.0;
ny = (y - 375) / 75.0;
{% endhighlight %}

So that our graph looks like this:

![Scaled output from accelerometer]({{site.baseurl}}/images/neopixel-graph2.jpg "Scaled output from accelerometer")

We are interested in getting the actual angle for a particular [x,y] couple to determine which neopixel LED should be turned on.

{% highlight ruby %}
angle = atan((ny/nx)) * 180 / 3.14;
{% endhighlight %}

![Angles from atan function]({{site.baseurl}}/images/neopixel-graph3.jpg "Angles from atan function")

We want the angle to appear between 0&deg; to 360&deg;, therefore we convert it this way:

{% highlight ruby %}
if(angle > 0.0){
    if(nx < 0.0)
      angle += 180;
  } 
  else {
    if(ny > 0.0)
      angle += 180;
    else
      angle += 360;
  }
{% endhighlight %}

And we read the angles between 0&deg; to 360&deg;.

![Angles finally looking good]({{site.baseurl}}/images/neopixel-graph4.jpg "Angles finally looking good")

Finally, we determine the proper LED from Neopixel ring:

{% highlight ruby %}
led = angle / (360 / NUMBER_OF_LEDS_ON_RING)
{% endhighlight %}

and that's it.

### Photos

![Arduino+Neopixel+Accelerometer]({{site.baseurl}}/images/neopixel-board1.jpg "Arduino+Neopixel+Accelerometer")

![Arduino+Neopixel+Accelerometer]({{site.baseurl}}/images/neopixel-board2.jpg "Arduino+Neopixel+Accelerometer")

![Arduino+Neopixel+Accelerometer]({{site.baseurl}}/images/neopixel-board3.jpg "Arduino+Neopixel+Accelerometer")

### Battery Life

Whole board (with LEDs shining) draws 66mA at 3.7V. With a 600mAh battery, it should live for 9 hours.

### TODOs

* add calibration 
* make transition between LEDs more smooth
* maybe some game like [Powerball](https://powerballs.com) or [Hula hoop](http://en.wikipedia.org/wiki/Hula_hoop)

### Similar Projects

* [Use an Accelerometer and Gyroscope with Arduino](http://www.instructables.com/id/Use-an-Accelerometer-and-Gyroscope-with-Arduino/)
* [Arduino 2D Level](http://www.instructables.com/id/Arduino-2D-Level/step6/Move-the-Circle-by-Tilting/)

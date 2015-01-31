---
layout: post
title:  "Laser Engraver Made of Silhouette Blade Cutter"
date:   2015-01-30 20:00:0
---

We take here Silhouette Portrait blade cutter and replace the blade with a 500mW laser.

<iframe width="560" height="315" src="https://www.youtube.com/embed/nVwiqPfVN3c" frameborder="0" allowfullscreen></iframe>

<p/>

<span style=" background-color: yellow; padding: 10px; font-weight: 800 !important"> WARNING: [be extra careful](http://www.laserpointersafety.com/laser-hazards_head-eyes/laser-hazards_head-eyes.html), it is dangerous to play with lasers of this power. </span>



## Bill of Materials

* [Silhouette Portrait](http://www.silhouetteamerica.com/shop/machines/portrait) blade cutter: ~ 200 USD
* [500mW Laser](http://www.ebay.com/itm/131399368831) ~ 54 USD
* resistor + power source + wires

## Wiring

The vendor claims the laser should operate at `5V` to `5.5V`, at `200mA` to `260mA`. 
We start with `5V` at `200mA`. We will use `12V` power source and we need to compute what resistor should we place in series with the laser to achieve `5V` and `200mA` at the laser.

The laser resistance is `5V / 0.2A = 25 ohm`.
We want to split the `12V` from source to keep `5V` at the laser, so that `7V` will be left for the resistor.
The desired resistance is then `7V / 5V * 25 ohm = 35 ohm`. 

The same can be computed via [ledcalc](http://ledcalc.com/) page.

![Silhouette Cutter as Laser Engraver - Diagram]({{site.baseurl}}/images/silhouette-laser-engraver-diagram.jpg "Silhouette Cutter as Laser Engraver - Diagram")

## Assembly

We remove the original cutting blade, put few layers of tape around the laser and put it into the holder. Here is the laser with tape attached:

![Fitting the laser to Silhouette cutting head]({{site.baseurl}}/images/silhouette-laser.jpg "Fitting the laser to Silhouette cutting head")

The most critical part is to properly adjust the laser distance from the cutting mat &ndash; allow time and patience. The laser focal distance can be also adjusted by un/screwing the laser lens (the focal distance seems to move somewhere between 3 to 10 cm this way). When the distance is not set properly, the laser will make no effect to the material.

## Laser Power

The video reveals that even at the slowest moving speed the laser is making a barely visible black path at the paper. It is obvious that the laser is far from being able to cut the material, but should be fine for engraving.

To engrave the design more properly, we can:

* tweak the open-source [Robocut](https://github.com/nosliwneb/robocut) binary to make the head movement slower
* pass the path several times
* use more powerful laser or apply more power to the laser.

Below is described why the last option is probably not the best one:
 
The laser itself is marked as capable of `500mW` optical power output. There are [claims](http://www.rp-photonics.com/wall_plug_efficiency.html) that [wall-plug efficiency](http://en.wikipedia.org/wiki/Wall-plug_efficiency) of such laser is around `25%`, so that max rated `500mW` power output of our laser should come for the price of `2W (500 mW * 4)` power input.

Unless I did some mistake in my reasoning, it means that with `5V` and `200mA` we consume `1W (P = U*I)`, so that we are running the laser at half power &ndash; which is good, as when running at full power the laser will overheat quickly and due to limited space in the cutting head we are unlikely to fit there [some heatsink](http://www.ebay.com/sch/i.html?_odkw=cooling+R+laser+module&_from=R40%7CR40&_osacat=0&_from=R40&_trksid=m570.l1313&_nkw=cooling+laser+module&_sacat=0).

## Help

When playing with the laser, I accidentally used wrong resistor and destroyed the laser. **I would like to continue with this proof of concept and move it towards product-like instructions, but I need to buy a new laser**. If you are willing to see some progress here, you are welcome to donate via PayPal. The laser price is 54 USD.

<form action="https://www.paypal.com/cgi-bin/webscr" method="post" target="_top">
<input type="hidden" name="cmd" value="_s-xclick">
<input type="hidden" name="encrypted" value="-----BEGIN PKCS7-----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-----END PKCS7-----
">
<input type="image" src="https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif" border="0" name="submit" alt="PayPal - The safer, easier way to pay online!">
<img alt="" border="0" src="https://www.paypalobjects.com/en_US/i/scr/pixel.gif" width="1" height="1">
</form>


## Resources

I am surely not the first one who tried this, [here is a youtube video](https://www.youtube.com/watch?v=jK5WyQa6PoQ) of a similar attempt from 2012, unfortunately lacking additional technical details except that 3W laser was used.

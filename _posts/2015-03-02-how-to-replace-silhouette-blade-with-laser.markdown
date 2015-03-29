---
layout: post
title:  "How To Replace Silhouette Blade with Laser"
date:   2015-03-02 20:00:0
---

This is a series of posts on laser burning/engraving with [Silhouette blade cutter](http://www.silhouetteamerica.com/shop):

1. [Demo: Laser Burn Map on Plywood with Silhouette Cutter]({{site.baseurl}}/2015/02/22/burning-map-on-plywood-with-silhouette-cutter.html)
2. __How To Replace Silhouette Blade with Laser__ -- you are reading this right now
3. [How to Burn on Plywood]({{site.baseurl}}/2015/03/09/how-to-burn-on-plywood-with-silhouette-cutter.html)
4. [How to Prepare Images/Vectors for Laser Burning]({{site.baseurl}}/2015/03/22/how-to-prepare-images-for-laser-burning.html)

<span style=" background-color: yellow; padding: 10px; font-weight: 800 !important"> WARNING: [Be extremely cautious](http://www.laserpointersafety.com/laser-hazards_head-eyes/laser-hazards_head-eyes.html) when playing with lasers, you may easily damage your sight.</span>

------------

## Bill of Material

* [405nm--473nm laser protection goggles](http://www.ebay.com/itm/141503872259)
* [Silhouette portrait](http://www.silhouetteamerica.com/shop/machines/portrait) blade cutter
* [405nm 500mW laser](http://www.ebay.com/itm/131399368831)
* [12V 500mA+ power source](http://www.ebay.com/itm/251791463409)
* [LM7805 voltage regulator](http://www.ebay.com/itm/130747602965)
* [47uF](http://www.ebay.com/itm/260814969015) capacitor 
* 16 ohm resistor
* wires

Optional (but recommended): 

* [heatsink](http://www.ebay.com/itm/261113766519)
* [thermometer with probe](http://www.ebay.com/itm/121514792721)

## Wiring

The purpose of the circuit is to limit the current passing via laser diode to prevent it from destroying itself -- such a circuit is called laser driver. The advantage of the circuit below is its simplicity. 
There are [other circuits](http://www.instructables.com/id/How-to-build-a-laser-general-guide/step5/Step-3-Driver/) which are more energy efficient and you can also buy [an assembled one](http://www.ebay.com/itm/251109952040).

![Replace Silhouette Blade with Laser - Wiring]({{site.baseurl}}/images/laser-wiring-01.jpg "Replace Silhouette Blade with Laser - Wiring")

LM7805 is a constant voltage regulator. It has three pins: IN, GND, OUT and guarantees `5V` between `LM7805.GND` and `LM7805.OUT` pins. Because we want a constant current regulator, we  [put](https://archive.org/stream/NationalSemiconductorVoltageRegulatorHandbook1980#page/n35/mode/2up) a resistor between `LM7805.GND` and `LM7805.OUT` pin to use LM7805 as a constant current regulator. 

I was unable to find proper datasheet for the laser, but assuming all `405nm 500mW` lasers have similar specs, [this datasheet](http://www.prophotonix.com/uploads/datasheets/Ushio046.pdf) tells that max forward current for the laser is `410mA` and forward voltage is around `4.9V`.

We want the current to be somewhere below `410mA`. I found out `312mA` to be a balance between laser power (so that the laser is powerful enough to actually burn something) and make sure the laser is not overheating that much.

To assert max current passing via the laser is approximately `312mA`, we put `16 ohm` resistor between `LM7805.GND` and `LM7805.OUT` pins, which gives us: `I = U/R = 5V / 16 ohm = 312 mA`.

The capacitor is there to flatten the input voltage variations.

The power source cannot be much below `12V`, because LM7805 will need to drop at least `5V` to operate properly and we need additional `4.9V` for the laser.

## Secure the Laser Position

The laser cylinder consist of following parts:

1. Case
2. Thermometer probe (optional)
3. Laser diode
4. Focusing lens with additional layer of tape to hold tightly inside the Silhouette blade holder.

![Replace Silhouette Blade with Laser - Disassembled laser]({{site.baseurl}}/images/laser-disassembled.jpg "Replace Silhouette Blade with Laser - Disassembled laser")

You want the part of the laser with focusing lens (4) to be attached as firmly as possible to the Silhouette blade mount. The rest of the laser (1+2+3) must be free for vertical movement so that the laser will move up and down without constraints when Silhouette is _cutting_. This attachment will also allow you to fine tune the laser focal point by screwing the focusing lens part from the diode part as depicted by the green arrows.

![Replace Silhouette Blade with Laser - Wiring]({{site.baseurl}}/images/laser-wiring-02.jpg "Replace Silhouette Blade with Laser - Wiring")

Below is Silhouette cutting head without laser:

![Replace Silhouette Blade with Laser - Wiring]({{site.baseurl}}/images/laser-wiring-03.jpg "Replace Silhouette Blade with Laser - Wiring")

The plastic card on top is attached with double sided duct tape. To create properly--sized hole in the plastic card you can drill a small hole in the card, heat up the card around the hole with lighter and while the card is still hot and pliable, _penetrate_ the hole with the laser cylinder. On top of the card are two small pieces of yogurt casing attached with superglue to prevent the laser cylinder from unnecessary horizontal movement while allowing smooth vertical movement.

Two wires  on the Silhouette cutting head  act like on/off switch for the laser so that the laser is beaming only when the Silhouette cutting head is in _low_ position:

<iframe width="600" height="400" src="https://www.youtube.com/embed/4ypyoF7xhBY" frameborder="0" allowfullscreen></iframe>

Here is the overall view:

![Replace Silhouette Blade with Laser - Wiring]({{site.baseurl}}/images/laser-wiring-04.jpg "Replace Silhouette Blade with Laser - Wiring")

The wires between laser and breadboard should be long enough to allow movement of the Silhouette cutting head from left to right and vice versa.

### Secure the Laser Position with Heatsink

If you can afford it I advise you to use heatsink. When using heatsink, it is not possible to insert the laser into the blade mount due to its size. Instead, you can secure the laser and heatsink as depicked in images below. The power on/off mechanism remains the same.

![Laser with Heatsink]({{site.baseurl}}/images/laser-wiring-06.jpg "Laser with Heatsink")

![Laser with Heatsink]({{site.baseurl}}/images/laser-wiring-07.jpg "Laser with Heatsink")

## Laser Temperature

The optional thermometer is there to estimate the laser temperature.
The [datasheet](http://www.prophotonix.com/uploads/datasheets/Ushio046.pdf) claims the thermometer operating temperature should be `30 C` but this is hard to achieve without a heatsink. 

Following table depicts thermal behavior of the `500mW` laser when driven with `312mA`.

| Time [min:sec] | Temp [C] | Temp with Heatsink [C] |  Notes                                          |
|--------------|-----------------|------------------------------------------------|
| 0:00         | 25   |  25         | room temperature, laser is turned on right now |
| 1:00         | 29   |  26         |                                                |
| 2:00         | 35   |  28         |                                                |
| 2:30         | 38   |  29.3         | turning off the laser                          |
| 3:00         | 40   |  30.4         |                                                |
| 4:00         | 39   |  31.3         |                                                |
| 5:00         | 36   |  31.3         |                                                |
| 6:00         | 33.5 |  30.9         |                                                |
| 7:00         | 31.5 |  30.2         |                                                |

If you want to extend the laser lifespan, you should definitely use the heatsink. Without heatsink, you should run the laser for 2 minutes, then press Pause button on Silhouette cutter, wait for 3 minutes and then press the Pause button again to continue. One can imagine an arduino-based circuit which will take care of this.

## Optical Power of the Laser Diode

The diode has max optical power output `500mW`. As we will see in further posts, you cannot use less powerful laser because the Silhouette cutting head is moving quite fast even at the lowest speed. You can use more powerful diode (e.g., [2W](http://www.ebay.com/itm/2W-445nm-M-Type-M140-Blue-Laser-Diode-Copper-Module-W-Leads-Aixiz-Glass-Lens-/170892986250?)), but you must use heasink with such poweful diode.

## Focal Distance

The lens inside the laser cylinder is there to focus the laser beam into a single point. By un/screwing the lens part one adjusts the focal point between `3cm` and `10cm`. You want the focal distance to be somewhere near `3cm` as that is the distance between the laser and the cutting mat.

You know that the focal distance is set properly when you see a smoke coming out of the point being burnt.


## Additional Reading

* [I Want to Build a Laser](http://laserpointerforums.com/f51/i-want-build-laser-thread-52972.html#post740462) on laserpointerforums.com
* [Voltage Regulator Handbook](https://archive.org/stream/NationalSemiconductorVoltageRegulatorHandbook1980#page/n35/mode/2up) on archive.org
* [404nm 500mW Laser Diode Datasheet](http://www.prophotonix.com/uploads/datasheets/Ushio046.pdf)

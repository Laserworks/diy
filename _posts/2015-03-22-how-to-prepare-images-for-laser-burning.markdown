---
layout: post
title:  "How to Prepare Images/Vectors for Laser Burning"
date:   2015-03-22 20:00:0
tags:
- laser
---

This is a series of posts on laser burning/engraving with [Silhouette blade cutter](http://www.silhouetteamerica.com/shop):

1. [Demo: Laser Burn Map on Plywood with Silhouette Cutter]({{site.baseurl}}/2015/02/22/burning-map-on-plywood-with-silhouette-cutter.html)
2. [How To Replace Silhouette Blade with Laser]({{site.baseurl}}/2015/03/02/how-to-replace-silhouette-blade-with-laser.html)
3. [How to Burn on Plywood]({{site.baseurl}}/2015/03/09/how-to-burn-on-plywood-with-silhouette-cutter.html)
4. __How to Prepare Images/Vectors for Laser Burning__  -- you are reading this right now

<span style=" background-color: yellow; padding: 10px; font-weight: 800 !important"> WARNING: [Be extremely cautious](http://www.laserpointersafety.com/laser-hazards_head-eyes/laser-hazards_head-eyes.html) when playing with lasers, you may easily damage your sight.</span>

------------

This blogpost explains which software is useful to prepare image for laser burning. Silhouette provides a proprietary software named [Silhouette Studio](http://www.silhouetteamerica.com/software/silhouette-studio/), however this piece of software runs only on Windows and Mac. Therefore we explain here how to use [Gimp](http://www.gimp.org/), [Inkscape](https://inkscape.org/en/) and [Robocut](https://github.com/nosliwneb/robocut) which are open--source and available on GNU/Linux.

For the purpose of the tutorial, we will prepare a house to be burned on plywood. At the end we should end up with this:

![House with house in plywood]({{site.baseurl}}/images/laser-sw-00.jpg "House with house in plywood")

## Adjust the Bitmap Image in Gimp

Here is original photo of the house:

![Original photo of a house]({{site.baseurl}}/images/laser-sw-01.jpg "Original photo of a house")

We open the image in Gimp and proceed with following steps:

1. Manually trace the outline of the house via [`Free Selection`](http://docs.gimp.org/en/gimp-tool-free-select.html) (Lasso)
2. [`Invert`](http://docs.gimp.org/en/gimp-selection-invert.html) the selection
3. Remove the the selection via [`Cut Tool`](http://docs.gimp.org/en/gimp-edit-cut.html)
4. Increase the [`Contrast and Brightness`](http://docs.gimp.org/en/gimp-tool-brightness-contrast.html).

We end up with following image:

![Photo of a house with background removed and brigtness/contrast fixed]({{site.baseurl}}/images/laser-sw-02.jpg "Photo of a house with background removed and brigtness/contrast fixed")

## Vectorizing the Bitmap Image in Inkscape

Next we open Inkscape:

1. Drag & drop the bitmap image into Inkscape.
2. Select the image and choose [`Path` &#8594; `Trace Bitmap`](https://inkscape.org/en/doc/tutorials/tracing/tutorial-tracing.en.html)
3. Now you are in the `Trace bitmap` dialog.
4. Select `Edge detection` and play with the `Threshold` to find the desired line art.
5. Still in `Trace bitmap` dialog, switch to `Options` tab and fill in `Suppress Speckles` size to `25` (default value is `2`). This will filter out all the unnecessary minor details which are useless for laser burning.

You should end up with image like this:

![Photo of a house vectorized]({{site.baseurl}}/images/laser-sw-03.jpg "Photo of a house vectorized")

Now we have the vector, but there are still plenty of unnecessary details and tiny curves which may uglify the laser cut, e.g., this overcomplicated detail of the wooden fence:

![Overcomplicated detail of fence]({{site.baseurl}}/images/laser-sw-04.jpg "Overcomplicated detail of fence")

We select [`Path` &#8594; `Simplify Path`](https://inkscape.org/en/doc/advanced/tutorial-advanced.html) to simplify the vector paths. After then our fence looks as following:

![Simplified detail of fence]({{site.baseurl}}/images/laser-sw-05.jpg "Simplified detail of fence")

The `Simplified path` approach has one major advantage and that is the length of all the paths which do describe the house are now shorter and so the laser burning will take significantly shorted time. The disadvantage is that we lost some details and the angles are no longer right angles.

Here is the overall view on our simplified house vector:

![House vectorized and simplified]({{site.baseurl}}/images/laser-sw-06.jpg "House vectorized and simplified")

We scale the house to fit on A4 and save it as SVG.

## Laser Burning in Robocut

Now we open the SVG in Robocut:

![House vector in Robocut]({{site.baseurl}}/images/laser-sw-07.jpg "House vector in Robocut")

If everything seems well, we choose `File` &#8594; `Cut`, `Speed: 1`, `Pressure: 1`.

And this is what we should end up with:

![House on plywood]({{site.baseurl}}/images/laser-sw-08.jpg "House on plywood")



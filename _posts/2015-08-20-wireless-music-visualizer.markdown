---
layout: post
title:  "Yet Another Wireless Music Visualizer"
date:   2015-08-20 20:00:0
thumbnail: images/2015-08-20-wireless-music-visualizer/cube.gif
tags:
- arduino 
- bluetooth
- processing
---

We use Arduino and bluetooth to wirelessly visualize music being played on PC. Arduino and its accesories are running from battery. In order to eliminate noise we analyze the music stream on PC and send frequency spectrum information to arduino via bluetooth.

![Yet Another Wireless Music Visualizer]({{site.baseurl}}/images/2015-08-20-wireless-music-visualizer/cube.gif "Yet Another Wireless Music Visualizer")

Here is also some music playing in background and the box is open:

<iframe width="100%" height="440" src="https://www.youtube.com/embed/6ICOOsUzq_4" frameborder="0" allowfullscreen></iframe>

## Comparison of Existing Solutions

There are few other arduino--based projects which visualize music in real time:

* [Tiny Arduino Music Visualizer](https://learn.adafruit.com/piccolo/overview) -- compact standalone solution with microphone connected to arduino (no PC required). However it seems the device must be placed near speaker to avoid noise.
	* [Music Visualiser Table](http://www.instructables.com/id/Music-Visualiser-Table/) -- based on the solution above
* [Arduino Audio Spectrum Visualizer](http://www.markneuburger.com/projects/visualizer) -- audio input is provided to Arduino via 3.5mm jack
* [This Is Your Brain On Music](http://www.instructables.com/id/This-Is-Your-Brain-On-Music/?ALLSTEPS) -- audio is analyzed on PC, Arduino is connected to PC via USB cable.


| Project                           | Wireless? | PC backend needed? | Subject to noise? |
|-----------------------------------|-----------|-------------|--------|
| Tiny Arduino Music Visualizer     | Yes       | Yes         | Yes    |
| Arduino Audio Spectrum Visualizer | No        | No          | No     |
| This Is Your Brain On Music       | No        | Yes         | No     |
| This project                      | Yes       | Yes         | No     |

## Aduino: Wiring and Software

### Bill of Material

* Arduino (I use Uno)
* [HC--06 Bluetooth Module](http://www.ebay.com/itm/Wireless-Bluetooth-RF-Transceiver-Module-serial-RS232-TTL-HC-06-for-Arduino-/171755210581?pt=LH_DefaultDomain_0&hash=item27fd688755)
* [Neopixel 8 Stick](https://www.adafruit.com/products/1426)
* 1000uF+ capacitor
* 330 ohm resistor
* 3.7V Li--Pol battery
* Mini breadboard

### Wiring

I connect the 3.7V battery directly to Arduino UNO's 5V input so that the whole circuit is running on 3.7V. LED stick and bluetooth module draw current directly from battery and not via Arduino board. When debugging and running the circuit from wall power instead of battery, you can connect the bluetooth and LED stick to draw current from Arduino unless you turn all the LED's fully on. You can run the whole circuit on 5V or on 3.7V without any modifications.

The capacitor and resistor are there due to [Adafruit recommendation](https://learn.adafruit.com/adafruit-neopixel-uberguide/best-practices). Be careful to properly connect the capacitor (polarity) and make sure bluetooth's TX is connected to arduino's RX and vice versa.

The Neopixel LED on schema below is actually Neopixel Ring 16 and not Neopixel 8 Stick as I was unable to find Neopixel 8 Stick schema for [Fritzing](http://fritzing.org/home/).

![Yet Another Wireless Music Visualizer]({{site.baseurl}}/images/2015-08-20-wireless-music-visualizer/yaw-wiring.png "Yet Another Wireless Music Visualizer")

### Arduino source code

PC sends data and Arduino acts as receiver, there is no communication from Arduino to PC. Each message sent from PC to Arduino must obey this structure:

```
r1-g1-b1-r2-g2-b2-...-r8-g8-b8;
```

where `r1` is amount of red color to be mixed on led No.1 (from 0 to 255).
E.g., to turn led No.2 to purple colour (full blue, full red, no green) and turn all other leds off we send following string:

```
0-0-0-255-0-255-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0;
```

As you can see below, arduino source code is dedicated solely to turning leds on and off, there is no business logic related to music. This has the advantage that you can play with and adjust the led colours and visualization functions on your PC while the arduino is running (which is more comfortable than reprogramming the arduino each time you want to try new colours).

{% highlight c %}
#include <Adafruit_NeoPixel.h>

#define NEOPIXEL_PIN 6
Adafruit_NeoPixel ledStrip = Adafruit_NeoPixel(8, NEOPIXEL_PIN);

void setup() {
  Serial.begin(9600); // connect to bluetooth module
  ledStrip.begin();
  ledStrip.setBrightness(255);
}

String rawData;
int r, g, b;

void loop() {
  if ( Serial.available() > 0 ) {
    rawData = Serial.readStringUntil(';');
    
    for(int i=0; i<24 ; i+=3){
      r = getValue(rawData, '-', i).toInt();
      g = getValue(rawData, '-', (i+1)).toInt();
      b = getValue(rawData, '-', (i+2)).toInt();
      ledStrip.setPixelColor(i/3, r, g, b); 
    }

    ledStrip.show();
  }
}

// http://arduino.stackexchange.com/a/1237
String getValue(String data, char separator, int index)
{
 int found = 0;
  int strIndex[] = {
0, -1  };
  int maxIndex = data.length()-1;
  for(int i=0; i<=maxIndex && found<=index; i++){
  if(data.charAt(i)==separator || i==maxIndex){
  found++;
  strIndex[0] = strIndex[1]+1;
  strIndex[1] = (i == maxIndex) ? i+1 : i;
  }
 }
  return found>index ? data.substring(strIndex[0], strIndex[1]) : "";
}
{% endhighlight %}


### Load Souce Code To Arduino

I upload the source code to Arduino from Arduino IDE via USB. You need to unpower (unplug) the bluetooth power pin to successfully upload the code to Arduino, because bluetooth module is connected to default Arduino's TX and RX pins and will interfere with code upload procedure otherwise.

### Pair Arduino's Bluetooth Module with PC

Follow [this](http://www.uugear.com/portfolio/bluetooth-communication-between-raspberry-pi-and-arduino/) tutorial to pair the devices.

Once done, you can use following simple ruby code to turn on a LED on your Arduino LED stick:

{% highlight ruby %}
require 'rubyserial'
bluetooth = Serial.new "/dev/rfcomm0", 9600
bluetooth.write '0-0-0-255-0-255-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0;'
{% endhighlight %}

Now we are done with Arduino--side setup. We paired PC's bluetooth with Arduino's bluetooth. Next step is to launch the music analysis code on PC and periodically instruct arduino's LED stick to alter the leds to visualize music.

## PC: Software

We need GNU/Linux. Music frequency analysis code is based on this [blogpost](http://dry.ly/ruby-music-visualizer).

### Install Processing

Download [Processing](https://processing.org/download/?processing), make sure you download version 2.2.*, because version 3.0 (beta) made some changes to API and will not work with the code below.


### Install JRuby and Gems

We need JRuby (plain C-based Ruby will not work) because we will connect our ruby code to Processing which is written in Java.

1. Install Java. I use OpenJDK (IcedTea 2.5.6) (7u79-2.5.6-0ubuntu1.12.04.1)
2. Install [RVM](https://rvm.io/) (Ruby Version Manager).
3. Install Jruby into RVM by `$ rvm install jruby`. I use jruby v1.7.19.
4. Make sure you use jruby by running `$ rvm use jruby` (or set jruby as your default ruby implementation).
5. Install [ruby-processing](https://github.com/jashkenas/ruby-processing) gem `$ gem install ruby-processing`
5. Install rubyserial gem by running `$ gem install rubyserial`

### Get Familiar with Processing Ruby Code

Let us save the source code below into file named `yaw.rb`:

{% highlight ruby %}
require 'rubyserial'
$arduino_bluetooth = Serial.new("/dev/rfcomm1", 9600)
$t1 = Time.now
$t2 = nil

class YAW < Processing::App
  load_library "minim" # minim is processing's music analysis library
  import "ddf.minim"
  import "ddf.minim.analysis"

  # programming with Processing is similar to programming with Arduino
  # this method is similar to arduino's "setup()" call
  def setup
    smooth 
    size(500,100)  
    background 0   
    setup_sound
  end
  
  # this method is similar to arduino's "loop()" call
  def draw
    update_sound
    animate_sound
    update_arduino_leds
  end
  
  def update_sound
    @fft.forward @input.left
    @previous_ffts = @current_ffts
    @freqs.each_with_index do |freq, i|
      new_fft = @fft.get_freq(freq)
      @max_ffts[i] = new_fft if new_fft > @max_ffts[i]
      @current_ffts[i] = ((1 - @fft_smoothing) * new_fft) + (@fft_smoothing * @previous_ffts[i])
      @scaled_ffts[i] = (@current_ffts[i]/@max_ffts[i])
    end
  end
  
  def animate_sound
    @size = 50
    @freqs.each_with_index do |f, i|
      x = 50*i + 50
      y = 50
      r = @scaled_ffts[i]*255
      g = @scaled_ffts[i]*255
      b = @scaled_ffts[i]*255
      
      # draw colour circles on your PC for debugging purposes
      fill(r, g, b)
      stroke(r+20, g+20, b+20)
      ellipse(x, y, @size, @size)
    end
  end
  
  def update_arduino_leds
    # update leds no sooner than every 50 miliseconds 
    # to assure no data is buffered which would lead to delay in visualization
    $t2 = Time.now
    if(($t2 - $t1) > 0.05) 
      s = @scaled_ffts.collect do |f| 
        b = (f*255).to_i
        g = 0 # play with LED colours here
        r = 0 # play with LED colours here
        [r,g,b]
      end
      
      $arduino_bluetooth.write("#{s.flatten.join('-')};")
      $t1 = $t2
    end
  end
  
  def setup_sound
    @minim = Minim.new(self)
    @input = @minim.get_line_in
    @fft = FFT.new(@input.left.size, 44100)

    # we choose 8 frequencies because we have 8 leds on neopixel stick
    @freqs = [60, 170, 310, 600, 1000, 3000, 6000, 12000]
    @current_ffts   = Array.new(@freqs.size, 0.001)
    @previous_ffts  = Array.new(@freqs.size, 0.001)
    @max_ffts       = Array.new(@freqs.size, 0.001)
    @scaled_ffts    = Array.new(@freqs.size, 0.001)
    @volume = 0
    @fft_smoothing = 0.9
  end
  

  
end

YAW.new :title => "Yet Another Wireless Music Visualizer"

{% endhighlight %}

### Launch the Processing Code

First power on the Arduino circuit. Arduino's bluetooth module should blink red. Once the Processing code will establish communication with arduino, bluetooth's indicator LED should light steadily in red colour.

Run the processing code from shell:
```
$ rp5 run yaw.rb
```

Once started, following window with music visualization should appear:

![Yet Another Wireless Music Visualizer]({{site.baseurl}}/images/2015-08-20-wireless-music-visualizer/processing.png "Yet Another Wireless Music Visualizer")

and Neopixel LEDs on Arduino circuit should blink in the same manner.

### Troubleshooting

#### yaw.rb Code Fails with RubySerial::Exception: ENOENT

This happens when Arduino's bluetooth module is not properly bind to `/dev/rfcomm0` linux file device.
Make sure the device exists by running `ls -la /dev/rfcomm*`. It may happen that the device name is `/dev/rfcomm1` instead of `/dev/rfcomm0`.

#### yaw.rb fails with NameError: PApplet not found in packages processing.core; last error: cannot load Java class processing.core.PApplet

Check content of `~/.rp5rc` file. The variable `PROCESSING_ROOT` should point to the directory where Processing is kept, e.g.:

```
PROCESSING_ROOT: /home/peter/programs/processing-2.2.1
```

#### Music is Playing but Visualization Screen and LEDs are Dark

Open Pulse Audio Volume Control by running `$ pavucontrol`. Click "Input Devices" tab on the bottom and select "Show: Monitors". Make sure the input device is not muted (see icons on the right side).

![Yet Another Wireless Music Visualizer]({{site.baseurl}}/images/2015-08-20-wireless-music-visualizer/volume-control.png "Yet Another Wireless Music Visualizer")

## Paper Box

Paper box consists of two parts (white top and blue bottom), here is SVG for blade cutter machine:

![Yet Another Wireless Music Visualizer]({{site.baseurl}}/images/2015-08-20-wireless-music-visualizer/paper-box.svg "Yet Another Wireless Music Visualizer")

The Neopixel 8 Strip LED lights are diffused by a thin layer of transparent bubble foil.

## Other Remarks

### Bluetooth Bandwidth
To keep this tutorial as simple as possible we leave the bluetooth connection on default baud rate which is only 9600 bauds (`9600 bauds = 1200 bytes per second`). Single LED status string usually occupies cca 60 bytes, but can grow up to 96 bytes when all LEDs are on (i.e. set to 255):

This string has 52 bytes:

```
0-0-0-255-0-255-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0;
```

This string has 96 bytes:

```
255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255-255;
```

We send the LED status string every 50 miliseconds (i.e. 20 times per second). `60 bytes * 20 = 1200 bytes per second`. If you experience the Arduino's LED visualization is delayed against the actual visualization on PC, you may need to do one of the following:

* Downcrease the refresh rate in this line of code: `if(($t2 - $t1) > 0.05)`
* Use only one colour channel
* Change the bluetooth's module bandwidth to e.g. 57600 bauds
* Use some encoding/compression to downcrease size of the transmitted data

### Battery Life

My multimeter tells that whole circuit draws cca `70mA`. `1000mA` battery should last `1000/70 =~ 14 hours`. Single Neopixel LED can draw up to 60mA when fully turned on (and there are eight of them), but usually most of the leds are dim for most of the time.




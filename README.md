Jetson Nano
===========

Jetson tutorials: <https://developer.nvidia.com/embedded/learn/tutorials> (including a series on using OpenCV with Jetson).

Buy Jetson Nano developer kit: <https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-nano/>

Adafruit Blinka support for Jetson: <https://blog.adafruit.com/2019/03/18/adafruit-blinka-support-for-the-nvidia-jetson-series-nvidia-gtc19-nvidiaembedded/>

Hackaday intro: <https://hackaday.com/2019/03/18/hands-on-new-nvidia-jetson-nano-is-more-power-in-a-smaller-form-factor/>

Jetbot
------

Original BOM - <https://github.com/NVIDIA-AI-IOT/jetbot/wiki/bill-of-materials>

---

For details on the 8265 got to the Intel [product page](https://ark.intel.com/content/www/us/en/ark/products/94150/intel-dual-band-wireless-ac-8265.html?wapkw=8265) and click "Ordering and Compliance" - there you can see that there's the 8265.NGWMG and three variants (NV, S, NVS). There's also a version for LTE coexistence - the D2WMLG.

According to this Intel [support answer](https://forums.intel.com/s/question/0D70P0000068fabSAA) you need to determine from the manufacturer of the device, with which you intend to use 8265, as to which variant is appropriate.

From the photos in the various tutorials all you see is "8265NGW".

Mouser sells the standard [8265.NGWMG](https://www.mouser.ch/ProductDetail/Intel/8265NGWMG?qs=sGAEpiMZZMsRr7brxAGoXSSUPDSAjAiVSJQY2dz2OC6rAL38Dke%252Beg%3D%3D), plus two variants - the [S](https://www.mouser.ch/ProductDetail/Intel/8265NGWMGS?qs=sGAEpiMZZMsRr7brxAGoXSSUPDSAjAiVjrRik%2FPuGZu8bwTo473NRg%3D%3D) and the [NV](https://www.mouser.ch/ProductDetail/Intel/8265NGWMGNV?qs=sGAEpiMZZMsRr7brxAGoXSSUPDSAjAiVviyySE0hnZ0PrUWxOuzKGA==).

See the nVidia [forum thread](https://devtalk.nvidia.com/default/topic/1049303/jetson-nano/jetson-nano-wifi-/) on diagnosing Jetson Nano WiFi issues.

---

The [BOM](https://github.com/NVIDIA-AI-IOT/jetbot/wiki/bill-of-materials) suggests a Leopard Imaging camera or a standard RPi camera with 160&deg; attachment. They link to Amazon and eBay for resellers of such an attachment - but in all cases the original manufacturer seems to be Waveshare. Waveshare seems a very common source for these cameras (the [Pi-Shop](https://www.pi-shop.ch) and many others carry them). So you can buy it direct [here](https://www.waveshare.com/imx219-d160.htm) or just buy their all-in-one unit, i.e. attachment and camera PCB, [here](https://www.waveshare.com/rpi-camera-g.htm). Note: from the pictures it looks like the attachment _might_ have adjustable focus (see threading), however from the descriptions of it, the all-in-one unit and the Leopard Imaging camera, they all seem to be fixed focus.

*Important:* in various camera reviews people comment on having created shorts screwing in the camera module - so probably best to use nylon screws and washers.

The Molex antennas are available from [Mouser](https://www.mouser.ch/ProductDetail/Molex/204281-1100?qs=%2Fha2pyFaduhMSNqNrOS4QfqJJ7QBOpAfUPiCEJKyFlTDCjhth2S02Q%3D%3D) and [Digikey](https://www.digikey.ch/product-detail/en/molex/2042811100/WM17372-ND/8020427).

Jetson Nano Developer Kit - <https://developer.nvidia.com/embedded/buy/jetson-nano-devkit>
64GB microUSD class 10 U3 (look for the 3 in a U symbol) - [digitec](https://www.digitec.ch/de/s1/product/samsung-evo-microsd-uhs-i-64gb-class-10-speicherkarte-6304644), [Mouser](https://www.mouser.ch/ProductDetail/Panasonic/RP-SMTT64DA1?qs=sGAEpiMZZMtyMAXUUxCBE3PXQ52q2ovEGvSv64covK5zbzeLXOr0eA%3D%3D).

Note: UHS is a little confusing, there's the bus type - UHS-I, UHS-II or UHS-III and a speed class - U1 or U3 - the two cards linked to above (and the card on Amazon in the original BOM are all UHS-I U3 cards).

MicroUSB 5V 2.5A power supply - <https://www.adafruit.com/product/1995> (this has a US plug).

Note that Adafruit comment that it's really a 5.25V device - the pictures a similar produce from [digitec](https://www.digitec.ch/en/s1/product/micro-usb-netzteil-fuer-raspberry-pi-5v-25a-power-supply-electronics-supplies-casing-7033127) claim the same as does the description for [Reichelt's equivalent](https://www.reichelt.com/ch/de/raspberry-pi-ladegeraet-5-v-2-5-a-micro-usb-schwarz-rasp-nt-25-sw-e-p240934.html?MWSTFREE=0&utm_source=psuma&utm_medium=Toppreise.ch&PROVID=2273&&r=1).

PiOLED - <https://www.adafruit.com/product/3527>
2x36 right-angle male header - <https://www.adafruit.com/product/1541>

While not in the BOM they stick down the WiFi module with what looks like polyimide tape, i.e. temperature resistant and non-conductive tape - <https://www.adafruit.com/product/3057>

Additional:

3M Command Poster Strips - <https://www.command.com/3M/en_US/command/products/~/Command-Poster-Strips/?N=5924736+3294529207+3294774426&rt=rud>
8mm M2 screws x 20 - they really seem to be screws rather than bolts and they note that they're self tapping - later in the instructions they say "Secure motor driver to chassis using self taping screws", the same goes for everything else that's attached to the chassis - the developer kit, the caster and the camera mount.
25mm M3 bolts and nuts x 4.
female-female 20cm jumper wires x 4 - Adafruit just has 15cm - <https://www.adafruit.com/product/1950> - or 30cm - <https://www.adafruit.com/product/1949>

---

USB Gooseneck mount - <https://www.modmypi.com/raspberry-pi/camera/camera-cases/camera-board-360-gooseneck-mount>

Unfortunately it's out of stock (as of 9.4.2019). Graspinghand product a similar but more expensive mount: <http://www.graspinghand.com/product/scorpi-b>

The ModMyPi variation may be available from: <https://mixtronica.com/varios-raspberrypi/17562-gooseneck-flexivel-para-camera-raspberry-pi-mmp0031-PTR002440.html>

This kind of gooseneck mount, that plugs into the audio jack, seems more common: <https://www.pi-shop.ch/camera-board-360-gooseneck-mount> - but I don't see how using a round jack can work very well.

---

See also "Tools needed" section - <https://github.com/NVIDIA-AI-IOT/jetbot/wiki/hardware-setup#tools-needed> - nothing surprising.

Misc
----

Adafruit M2.5 stand-off set: <https://www.adafruit.com/product/3658>

M2.5 is the size you need for the holes on the Raspberry Pi and related components.

Sparkfun line follower array: <https://www.sparkfun.com/products/13582>

At $32 it'd seem better to build your own array: <https://learn.adafruit.com/spinning-disc-step-sequencer/build-the-sensor-circuit>

The Adafruit build requires sensors (5 for $3), resistors (25 for $0.75 and a proto board (1 for $4.50).

Or a seemingly near identical array from Pololu for $9.50: <https://www.pololu.com/product/4348>

Ball head compatible camera mounts:

* <https://www.adafruit.com/product/1434>
* <https://www.modmypi.com/raspberry-pi/camera/camera-cases/pibow-camera-mount>
* <https://www.modmypi.com/raspberry-pi/camera/camera-cases/modmypi-mini-camera-stand-no-lens>
* <https://www.pi-shop.ch/mini-camera-stand-no-lens>

Which could be used with something like these:

* <https://www.amazon.de/dp/B01KZVEQJE/ref=psdc_332052031_t3_B075SFM1GD?th=1>
* <https://www.amazon.co.uk/Camera-Degree-Swivel-Bracket-Holder/dp/B071774PTC/>
* <https://www.amazon.com/dp/B01LWE9S9Z/ref=psdc_3347671_t2_B01CQAQOSI>

These are all fairly substantial, i.e. 5cm or more in height.

---

Full build blog for Polulu Romi with Raspberry Pi (you can control the Pi via a web interface from phone or laptop): <https://www.pololu.com/blog/663/building-a-raspberry-pi-robot-with-the-romi-chassis>

Alternative customer build: <https://www.pololu.com/blog/675/romi-and-raspberry-pi-robot> (blog entry) and <https://forum.pololu.com/t/rpb-202-a-beginners-robot-based-on-romi-chassis-and-raspberry-pi/11243> (forum entry)

A near identical but more generic alternative is covered here: <https://www.pololu.com/blog/577/building-a-raspberry-pi-robot-with-the-a-star-32u4-robot-controller>

See also my forum post on A-star vs Romi: <https://forum.pololu.com/t/romi-vs-a-star-on-generic-expansion-plate/17160>

---

Romi chassis kit - blue: <https://www.pololu.com/product/3506>

Includes:

* One blue Romi chassis base plate.
* Two mini plastic gearmotors (120:1 HP with offset output and extended motor shaft).
* A pair of blue Romi chassis motor clips.
* A pair of white 70×8mm Pololu Wheels.
* One blue Romi chassis ball caster kit.
* One Romi chassis battery contact set.

Additional ball caster kit: <https://www.pololu.com/product/3536>

Romi 32U4 control board: <https://www.pololu.com/product/3544>

You can mount a Raspberry Pi directly onto this board with standoffs (not supplied).

Romi encoder pair: <https://www.pololu.com/product/3542>

Note on second ball caster: "... ball caster has built-in suspension that you might need to stiffen through the addition of a rubber band as described in the assembly section of the Romi user’s guide, or you can disable the suspension entirely and lock the ball caster in place using a washer." See also <https://a.pololu-files.com/picture/0J7328.1200.jpg>

Expansion plate **x 2**: <https://www.pololu.com/product/3560>

The expansion plate only comes in black.

The expansion plate also requires:

* 6 x 1.5"-long standoffs = **2 packs** <https://www.pololu.com/product/2009>  
* 2 x 1"-long standoffs = 1 pack <https://www.pololu.com/product/1944>
* 1 pack of #2-56 hex nuts <https://www.pololu.com/product/1067> and 1 pack of 1/4" #2-56 screws <https://www.pololu.com/product/1955>

Alternative single piece expansion plate: <https://www.pololu.com/product/1532>

This is the narrow version of this plate, the wide version is 2mm too wide for the Romi.

You can find the dimensions of the Romi parts here: <https://a.pololu-files.com/picture/0J7257.1200.png>

It's 163mm front to back and 125mm between the wheels.

---

Host system GPU:

* <https://www.digitec.ch/de/s1/product/msi-geforce-rtx-2060-ventus-oc-6gb-midrange-grafikkarte-10293393>
* <https://www.digitec.ch/de/s1/product/msi-geforce-rtx-2060-ventus-xs-6gb-midrange-grafikkarte-10385358>

They seem to be identical cards except for differences is card dimensions.

<http://timdettmers.com/2019/04/03/which-gpu-for-deep-learning/>

---

Amazon - mixed inventory and you're never quite sure what you're getting or from whom.

---

Price comparison: you can price compare across Digikey, Mouser, RS, Arrow, Vertical, Element14 and others at <https://www.eciaauthorized.com/en>

Note: Element14, Farnell, Newark and Avnet are all the same thing.

Note that Digikey and Mouser both distribute some or all of the Adafruit and Sparkfun ranges. Digikey carries some Pololu parts but the coverage doesn't seem very comprehensive.

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

Gooseneck mount - <https://www.modmypi.com/raspberry-pi/camera/camera-cases/camera-board-360-gooseneck-mount>

---

See also "Tools needed" section - <https://github.com/NVIDIA-AI-IOT/jetbot/wiki/hardware-setup#tools-needed> - nothing surprising.

---

Polulu Romi with Raspberry Pi blog: <https://www.pololu.com/blog/663/building-a-raspberry-pi-robot-with-the-romi-chassis>

https://www.pololu.com/blog/675/romi-and-raspberry-pi-robot
https://forum.pololu.com/t/rpb-202-a-beginners-robot-based-on-romi-chassis-and-raspberry-pi/11243
https://www.pololu.com/blog/577/building-a-raspberry-pi-robot-with-the-a-star-32u4-robot-controller

Note that you can buy expansion plates - <https://www.pololu.com/product/3560> - see links to standoffs on that page and notes on adding a second ball caster.

---

Host system GPU:

* <https://www.digitec.ch/de/s1/product/msi-geforce-rtx-2060-ventus-oc-6gb-midrange-grafikkarte-10293393>
* <https://www.digitec.ch/de/s1/product/msi-geforce-rtx-2060-ventus-xs-6gb-midrange-grafikkarte-10385358>

They seem to be identical cards except for differences is card dimensions.

<http://timdettmers.com/2019/04/03/which-gpu-for-deep-learning/>

---

Amazon - mixed inventory and you're never quite sure what you're getting or from whom.

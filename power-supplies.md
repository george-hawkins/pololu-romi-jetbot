Jetson Nano power adapters
==========================

Just 2A can be supplied to the Nano via the USB power connector but the Nano can consume more.

It can cosume 4A via the barrel jack and 6A via two GPIO pins (each pin takes 3A - this sounds on the limit of what such pins can generally take).

See ["Use More Power!"](https://www.jetsonhacks.com/2019/04/10/jetson-nano-use-more-power/) on JetsonHacks for more details.

It sounds like the Nano module itself is happy with 2A - it's powering additional items from the development board that may push you over 2A.

Suitable power adapters from Mouser:

* Adafruit 5V 4A AC [desktop adapter](https://www.mouser.ch/ProductDetail/485-1466) (comes with US plug only) - US$15 
* CUI 5V 4A [SDI24-5-UD-P5](https://www.mouser.ch/ProductDetail/490-SDI24-5-UD-P5) (comes without wall plug) - US$15

The CUI will require a power cord like this:

* Schurter [6013.0474](https://www.mouser.ch/ProductDetail/693-6013.0474) for the EU - US$3.50
* Schurter [6010.5274](https://www.mouser.ch/ProductDetail/693-6010.5274) for the US - US$3

If searching for a cord for other countries the important thing is that it has a [C7](https://en.wikipedia.org/wiki/IEC_60320#C7/C8_coupler) connector at one end to connect to the power adapter.

For a much cheaper adapter from a reputable supplier, the ones from Odroid seem suitable:

* [EU plug](https://www.hardkernel.com/shop/5v-4a-power-supply-eu-plug-2/) - US$5.50
* [US plug](https://www.hardkernel.com/shop/5v-4a-power-supply-us-plug-2/) - US$5.50

They also have models for all other major plug types.

Jetson Nano power
=================

Just 2A can be supplied to the Nano via the USB power connector but the Nano can consume more.

It can cosume 4A via the barrel jack and 6A via two GPIO pins (each pin takes 3A - this sounds on the limit of what such pins can generally take).

See ["Use More Power!"](https://www.jetsonhacks.com/2019/04/10/jetson-nano-use-more-power/) on JetsonHacks for more details.

It sounds like the Nano module itself is happy with 2A - it's powering additional items from the development board that may push you over 2A.

Power adapters
--------------

Suitable power adapters:

* [Pololu 5V 5A wall power adapter](https://www.pololu.com/product/1462) (comes with US plug only) - US$17
* [Adafruit 5V 4A switching power supply](https://www.adafruit.com/product/1466) (comes with US plug only) - US$15
* [Sparkfun 5V 4A power supply](https://www.sparkfun.com/products/15352) (comes with US plug only) - US$13
* Mouser - [CUI 5V 4A SDI24-5-UD-P5](https://www.mouser.ch/ProductDetail/490-SDI24-5-UD-P5) (comes without wall plug) - US$15

Mouser also carry the Adafruit model. The CUI will require a power cord like this:

* Schurter [6013.0474](https://www.mouser.ch/ProductDetail/693-6013.0474) for the EU - US$3.50
* Schurter [6010.5274](https://www.mouser.ch/ProductDetail/693-6010.5274) for the US - US$3

If searching for a cord for other countries the important thing is that it has a [C7](https://en.wikipedia.org/wiki/IEC_60320#C7/C8_coupler) connector at one end to connect to the power adapter.

For a much cheaper adapter from a reputable supplier, the ones from Hardkernel seem suitable:

* [EU plug](https://www.hardkernel.com/shop/5v-4a-power-supply-eu-plug-2/) - US$5.50
* [US plug](https://www.hardkernel.com/shop/5v-4a-power-supply-us-plug-2/) - US$5.50

They also have models for all other major plug types. And they also have much cheaper power cords:

* [2 pin EU power cord](https://www.hardkernel.com/shop/2pin-eu-power-cord/) - US$0.70
* [2 pin US power cord](https://www.hardkernel.com/shop/2pin-us-power-cord/) - US$0.70

Barrel jack
-----------

As noted in the "Use More Power!" blog entry linked to above you'll need to jumper the J48 header on the Nano in order to use the barrel jack.

Suitable jumpers / shorting blocks:

* [Pololu 5 pack](https://www.pololu.com/product/970) - US$0.20
* [Adafruit 10 pack](https://www.adafruit.com/product/3525) - US$1
* [Sparkfun single jumper](https://www.sparkfun.com/products/9044) - US$0.35

Digikey and Mouser have many similar products, e.g.:

* Digikey - [Sullins closed top jumper](https://www.digikey.com/product-detail/en/sullins-connector-solutions/SPC02SYAN/S9001-ND/76375) from Digikey - US$0.10
* Mouser - [Harwin closed top](https://www.mouser.ch/ProductDetail/855-M7686-05) - US$0.36 or [open top](https://www.mouser.ch/ProductDetail/855-M7582-05) - US$0.18

Mouser also stocks the [Adafruit jumpers](https://www.mouser.ch/ProductDetail/485-3525) and the [Sparkfun ones](https://www.mouser.ch/ProductDetail/474-PRT-09044) (at the same price as their respective original vendors).

Male 2.1mm barrel plugs:

* [Pololu screw terminal barrel plug](https://www.pololu.com/product/2448) - US$2
* [Adafruit screw terminal barrel plug](https://www.adafruit.com/product/369) - US$2
* [Adafruit solderable barrel plug](https://www.adafruit.com/product/3310) - US$1
* [Sparkfun screw terminal barrel plug](https://www.sparkfun.com/products/10287) - US$3
* [Sparkfun solderable barrel plug](https://www.sparkfun.com/products/11476) - US$1

Mouser stock both the Adafruit and Sparkfun screw terminal plugs but not the soldarable ones. A suitable soldarable one from Mourser would seem to be:

* [CUI PP3-002A](https://www.mouser.ch/ProductDetail/490-PP3-002A) (which can handle up to 5A) - US$1

Other possibly suitable plugs can be seen in this [search result](https://www.mouser.ch/Connectors/Power-Connectors/DC-Power-Connectors/_/N-axittZ1yzvvqx?P=1z0x8ggZ1z0x878Z1z0x8glZ1z0wxf2Z1z0wxfcZ1z0vlpqZ1z0xbxo&Ns=Pricing|0).

**Update:** a [pigtail](https://www.computerlanguage.com/results.php?definition=pigtail) with a barrel plug at one end would probably be more convenient:

* [Hardkernel 27cm barrel plug pigtail](https://www.hardkernel.com/shop/dc-plug-cable-assembly-5-5mm/) - US$1.25
* [Banggood 24cm barrel plug pigtail](https://www.banggood.com/DC12V-MaleFemale-Power-Supply-Jack-Connector-Cable-Plug-Cord-Wire-5_5mm-x-2_1mm-p-1182642.html) - US$1.20 (make sure to select the male plug)
* Digikey - [Tensility 2m barrel plug pigtail](https://www.digikey.com/product-detail/en/tensility-international-corp/CA-2185/CP-2185-ND/568576) - US$2.50
* Mouser - [Kobiconn 2m barrel plug pigtail](https://www.mouser.ch/ProductDetail/172-7443-E) - US$4

2m of cable for the seems rather wasteful but all similar cables from Digikey and Mouser are that length or more.

Oddly it seems better value to buy extension cables or splitters and cut them up rather than buy pigtails:

* [Adafruit 1.5m barrel jack extension cable](https://www.adafruit.com/product/327) - US$3
* [Sparkfun 1m barrel jack extension cable](https://www.sparkfun.com/products/11706) - US$1.75
* [Adafruit 4-way barrel jack splitter](https://www.adafruit.com/product/1352) - US$5
* [Banggood barrel jack to plug 1 to 8 splitter](https://www.banggood.com/5_5-x-2_1mm-Female-To-Male-Plug-DC-Splitter-Connector-For-LED-Lighting-p-975267.html) - US$4
* Digikey - [Tensility 2-way barrel jack splitter](https://www.digikey.com/product-detail/en/tensility-international-corp/10-02743/839-1476-ND/8753166) - US$3.80

Mouser don't seem to carry any branded splitters or extension cables but they do carry the Adafruit products.

None of these products seems to very clear on the max-current they can carry, with the exception of Banggood pigtail (which specifies a max-current of 5A). Most of them seem to use stranded 24AWG cable which is a little thin for carrying the 4A that the Nano can take via the barrel jack - 22AWG or lower would be better. Having said that the description for the Adafruit extension cable claims 24AWG is fine up to 5A.

A **right-angle** plug would probably fit better but they're less common:

* [Hardkernel 27cm right-angle barrel jack pigtail](https://www.hardkernel.com/shop/dc-plug-cable-assembly-5-5mm-l-type/) - US$1.25
* [Banggood 1m right-angle barrel jack pigtail](https://www.banggood.com/DC-1m-5_52_1mm-22AWG-L-Shape-Male-Power-Extension-Cable-Wire-p-1364243.html) - US$1.40
* [Banggood 4cm right-angle to right-angle barrel plug cable](https://www.banggood.com/DC-Tip-Power-Plug-Jack-Connector-Dual-5_5-x-2_1mm-Male-Right-Angle-Cord-Cable-p-1161332.html) - US$2.40
* [Banggood soldarable barrel plug](https://www.banggood.com/2_5x5_5mm-Right-Angle-L-90-Male-Plug-Jack-DC-Power-Tip-Socket-Connector-Adapter-p-1023090.html) - US$1.20
* Digikey - [Tensility 2m right-angle barrel plug pigtail](https://www.digikey.com/product-detail/en/tensility-international-corp/CA-2189/CP-2189-ND/568580) - US$2.50
* Mourser - [Kobiconn soldarable barrel plug](https://www.mouser.ch/ProductDetail/173-5521TIP-EX) - US$1.50


Batteries
---------

Pololu have a range of suitable switching step-down voltage regulators:

* 5V 5A [D24V50F5](https://www.pololu.com/product/2851) - US$15
* 5V 6A [D24V60F5](https://www.pololu.com/product/2865) - US$20
* 5V 9A [D24V90F5](https://www.pololu.com/product/2866) - US$28

The 5A model looks like it would be operating at the edge of its capabilities if the motors and the Nano board were drawing full power.

Unlike the 5A, the 6A and 9A have nice [strain relief](https://a.pololu-files.com/picture/0J5818.1200.jpg?ada7981fba447658ea0884e2f7c9ef31) mounting holes for the power wires.

The 9A model requires an input voltage of around 10V  deliver its maximum _continuous_ output current of 8A. While the 6A model requires around 9V for a little over 6A _continuous_ output (so the 6A model seems to slightly exceed what its name would imply while the 9A one slightly under achieves).

These values are the "maximum continuous output current [each] regulator can deliver with no external heat sinking or added air flow."

So both would seem happy with a 2S or 3S LiPo - with 3S perhaps being slightly better. At higher input voltages thermal shutdown will occur at lower maximum output currents.

Voltage regulators involve a dropout voltage - the input voltage must be a certain minimum amount higher than the desired output voltage. According to the graphs for both the 6A and 9A model a 2S battery _should_ still be just about fine when almost discharged (when its output voltage will be around 6V, down from its nominal voltage of 7.4V and its fully charged voltage of 8.4V).

**Important:** these components can get extremely hot in normal operation - thermal shutdown protection only activates at 160C.

### Power distribution boards

getfpv have a large [range of PDBs](https://www.getfpv.com/electronics/power-distribution-boards-pdb.html?dir=asc&order=price) including ones from Lumenier, like this [one](https://www.getfpv.com/lumenier-4power-mini-pdb.html), that are specifically designed for use with Pololu step-down (though really the ones with a smaller PCB than those listed above).

Banggood obviously also have lots of PDBs (but as usual they're not conveniently catagorized) - suitable looking ones (without additional electronics such as a voltage regulation) include this [one](https://www.banggood.com/30x30-35x35-PCB-ESC-Power-Distribution-Board-For-MINI-Quadcopter-Multicopter-p-984686.html) and this [one](https://www.banggood.com/Martian-Mini-PDB-Power-Distribution-Board-for-Martian-I-II-III-Frame-Kit-p-1081695.html) (the large battery holes are meant to take an XT60 connector). These odd little [wire extension plates](https://www.banggood.com/12-PCS-Diatone-40A-Brushless-Motor-Wire-Extension-Plate-For-RC-Drone-FPV-Racing-Multi-Rotor-p-1334388.html) from Diatone might also be useful.

### BECs, UBECs and SBECs

Basically BECs etc. are just brand names for voltage regulators that have now become interchangeable terms. The important thing is if they're linear or switched (much more efficient). For more details see this [Electronic StackExchange answer](https://electronics.stackexchange.com/a/8370/27099) and this [RCGroups thread](https://www.rcgroups.com/forums/showthread.php?1222157-Sbec-ubec-bec-whats-the-difference) (later in it covers linear vs switched in detail).

HobbyKing sell a number of switching BECs:

* [5V 4A continuous from Matek](https://hobbyking.com/en_us/ubec-duo-4a-5-12v-4a-5v.html) - US$17
* [5V 8A continuous from Turingy](https://hobbyking.com/en_us/turnigy-8-15a-ubec-for-lipoly.html) - US$17
* [5V 20A continuous from Yep](https://hobbyking.com/en_us/yep-20a-hv-2-12s-sbec-w-selectable-voltage-output.html) - US$16

As always your mileage may vary with these kind of fairly anonymous products.

Choosing a power source
-----------------------

Radek Jarema has a nice [overview](https://medium.com/husarion-blog/batteries-choose-the-right-power-source-for-your-robot-5417a3ec19ca) of the typical requirements of different robot setups (covering drones, rovers etc.).

The "Small AGV" (automated guided vehicle) section matches out setup well - he recommends a Li-Ion or Li-Po with about 4200mAh - though as he's calculating for a 2 hour working time, one could do with a lot less.

Milliamp Hours
--------------

The NiMH rechargeable batteries that I'm using the Romi chassis are 1.2V 2100mAh, so that's 2.52Wh per battery for a total of 15.12Wh.

An equivalent 2S LiPo would need to provide 2043mAh and a 3S would need 1362mAh.

See this [online calculator](https://milliamps-watts.appspot.com/) to do the trivial maths involved.

So (for Switzerland) suitable batteries would be e.g.:

* [Swaytronic 1800mAh 3S](https://www.galaxus.ch/de/s5/product/swaytronic-akku-3s-lipo-1110v-1800mah-rc-akku-8614029) - Fr. 27
* [Swaytronic 2400mAh S2](https://www.galaxus.ch/de/s5/product/swaytronic-akku-lipo-740v-2400mah-rc-akku-3518831) - Fr. 21

Remember a bigger battery means more power but also more weight and a bigger size.

For more suitable batteries from Digitec see these [3S ones](https://www.galaxus.ch/de/s5/producttype/rc-akku-679?tagIds=760&opt=v23-2002%3A11.1%7Cv23-230%3A1.4%7Cv23-230%3A1.45%7Cv23-230%3A1.6%7Cv23-230%3A1.65%7Cv23-230%3A1.7%7Cv23-230%3A1.8%7Cv23-230%3A1.9%7Cv23-230%3A1.95%7Cv23-230%3A2%7Cv23-230%3A2.1%7Cv23-230%3A2.2%7Cv23-230%3A2.4%7Cv23-230%3A2.3%7Cv23-230%3A2.5%7Cv23-230%3A2.7%7Cv23-230%3A3.1%7Cv23-230%3A3.2%7Cv23-230%3A2.8%7Cv23-230%3A3.3&pdo=23-19063%3A353839&so=5) and these [2S ones](https://www.galaxus.ch/de/s5/producttype/rc-akku-679?tagIds=760&opt=v23-2002%3A7.4%7Cv23-230%3A3%7Cv23-230%3A2.6%7Cv23-230%3A2.4%7Cv23-230%3A2.2&so=5&pdo=23-19063%3A353839).

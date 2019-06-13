Better motor control with PID
=============================

There's a nice Wikipedia [PID controller page](https://en.wikipedia.org/wiki/PID_controller) - the creative commons licensed diagrams in this page have ended up being used in many PID articles out there on the web.

---

["Going Straight with PID"](https://projects.raspberrypi.org/en/projects/robotPID) is a clear simple introduction as to why you need a feed back loop, between the encoders and the motors, and a PID controller in order to drive straight.

Just read it thru from start to finish and just treat the Python snippets as pseudo-code - they've more value in explaining things than as actual code you intend to run.

The important thing this article achieve is to make clear how simple calculation involved are - there are PID libraries but actually they're buying you very little over coding up the maths yourself.

In fact it's not the maths that's troublesome - it's choosing the PID constant values Kp, Ki and Kd that have to be tuned to match the characteristics of your particular vehicle.

*Update:* actually once you start including features like anti-windup the code becomes a bit more complicated (but not that much more complicated).

In the article simple single-pin pulse encoders are used but there's no meaningful behavior difference between these and the Romi quadrature encoders other than that the quadrature ones are more accurate.

The article links to the author's corresponding [GitHub repo](https://github.com/martinohanlon/RobotPID) - the important code being the [PID controller](https://github.com/martinohanlon/RobotPID/blob/master/integral.py) (click to see how simple the Python code is) and the [quadrature encoder class](https://github.com/martinohanlon/RobotPID/blob/master/quadratureencoder.py) (again very simple - the repo has both this quadratureencoder encoder class and a similar class for pulse encoders).

---

The Romi 32U4 Arduino library already includes a suitable [encoder class](https://github.com/pololu/romi-32u4-arduino-library/blob/master/Romi32U4Encoders.h) and there is a standard Arduino [PID library](https://playground.arduino.cc/Code/PIDLibrary/) available.

This PID library is written by Brett Beauregard and he seems to be _the_ Arduino PID person - his day job is working for a consultancy that specializes in PID tuning. He covers the PID library in a series of [blog posts](http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-introduction/). This series is simple and clear and well worth reading irrespective of whether you're going to use the library.

And perhaps more interesting he's also the author of the only well know Arduino [autotuning library](https://github.com/br3ttb/Arduino-PID-AutoTune-Library) which he explains in this [blog post](http://brettbeauregard.com/blog/2012/01/arduino-pid-autotune-library/).

Note: there are many forks of Brett's library but only [jhagberg's one](https://github.com/jhagberg/Arduino-PID-AutoTune-Library) seems to involve any real additions beyond the last commit made by Brett - I haven't investigated if any of these additions are acutally meaningful.

Brett's Arduino autotuning library is actually a port of William Spinelli [AutotunerPID Toolkit](https://ch.mathworks.com/matlabcentral/fileexchange/4652-autotunerpid-toolkit) for Matlab.

Matlab seems to be the place to go for research and work to do with PID autotuning.

TODO: when going thru your autotuning bookmarks see if Spinelli's toolkit seems to be the most popular among the Matlab links.

---

The PID library and the autotune library have been ported to Python - you can find both [here](https://github.com/hirschmann/pid-autotune). Interestingly they based the autotune library off Brett's port rather than the Matlab original.

Even if you're planning to tune for the 32U4 of the Romi I think these libraries are worth looking - plotting and experimentation is likely to be far superior with Python than trying the same thing in the Arduino world.

---

The "Going Straight with PID" references the ["Experiments in quadrature encoders"](http://robotoid.com/appnotes/circuits-quad-encoding.html) article which provides a nice explanation of how quadrature encoders work - but only the first section - "quadrature encoding defined" - is relevant. After this section the author covers using an IC to convert the raw output of an encoder into a clock and direction signal - the Romi library includes the necessary code to do this for you. It seems to be a holy war issue (on a par with the [editor wars](https://en.wikipedia.org/wiki/Editor_war)) as to whether a dedicated IC is needed for this task. After much investigation I didn't find convincing evidence that the IC side was correct - clearly many systems are successfully built without them - and I'm inclined to believe the Pololu people and people like Paul Stoffregen (of Teensy fame) who clearly feel you can do everything needed with your main MCU.

There don't seem to be many readily available breakouts for quadrature encoder ICs (that convert to clock and direction) - this [one](https://www.superdroidrobots.com/shop/item.aspx/dual-ls7366r-quadrature-encoder-buffer/1523/) from SuperDroid Robots, that can handle two encoders, seems to be only one that is actually available and actually turns up in builds described elsewhere on the web. At US$45 you wonder what you're getting if a much cheaper general purpose MCU is capable of doing the job. See this [post](https://forum.pjrc.com/threads/24147-Encoder-library-missing-pulses/page3?styleid=2#post_35929) (the 09-29-2013 12:22 PM one) from Paul Stoffregen where he gets quite annoyed at poster Will for insisting that it's basically impossible to get things working without such quadrature encoder ICs.

---

Matlab have a nice series of 7 [YouTube videos](https://www.youtube.com/watch?v=wkfEZmsQqiA&list=PLn8PRpmsu08pQBgjxYFXSsODEF3Jqmm-y) (best watched at 1.5 times speed) about understanding PID control. The description below each of the videos in the series includes more details, links to code and further references.

The first video introduces PID with practical examples and little maths. In particular they provide a nice example showing why the simplest adjustment (proportional) can actually produce terrible results in certain situations - a nice justification as to why PID can be necesssary and not just a nice enhancement over simple adjustment.

The second video covers some short coming of basic PID and solutions to these problems - including integrator anti-windup (which the Arduino PID library supports - see the [reset windup](http://brettbeauregard.com/blog/2011/04/improving-the-beginner%E2%80%99s-pid-reset-windup/) section of Brett's PID blog series).

**Update:** when looking at various PID controller implementations I was surprised how many don't include clamping, i.e. the windup protection, covered here and part of Brett's library - it's a very simple addition.

Videos four to six cover:

* an overview of tuning using either a real/test system or a model
* building models in three different ways - from first principle, i.e. using the physics equations that describe the elements of the system), using system identification (taking the inputs and the outputs and fitting) and linearizing (where you take an existing model and linarize it).
* tuning using a model - starting with manual tuning using graphical tools and then looking at automatic tuning.

Video seven covers cascaded loops (loops within loops - e.g. using a motor where _motor speed_ is PID controlled within a system where _altitude_ is PID controlled) and discrete systems (the fact that your PID loop isn't continuous but rather you sample at discrete intervals).

If really interested in autotuning PID values I think the real way to start would be [buying Matlab home](https://ch.mathworks.com/pricing-licensing.html?prodcode=ML&intendeduse=home) and working thru the [code and examples](https://ch.mathworks.com/campaigns/offers/pid-tuning-code-examples.html) that accompany the series (each video includes the same link) along with the additional links from [video six](https://youtu.be/qj8vTO1eIHo) in particular plus the [series on system identification](https://ch.mathworks.com/products/sysid/videos.html) linked to from video five.

Note: this [article](https://create.arduino.cc/projecthub/matlab-makers/drive-with-pid-control-on-an-arduino-mega-2560-333ef1) covers using Matlab with simulated hardware and with real Arduino based four-wheeled hardware.

---

Pololu forum user DrGFreeman has written a [library](https://github.com/DrGFreeman/RasPiBot202) of classes for the Romi, including classes for [pid](https://github.com/DrGFreeman/RasPiBot202/blob/master/pid.py) and [odometry](https://github.com/DrGFreeman/RasPiBot202/blob/master/odometer.py) (covered later in David Anderson video down below).

As in David Anderson's video this library works in terms of robot velocity and angle, rather than left and right motor speed, it's only in [`motioncontroller.py`](https://github.com/DrGFreeman/RasPiBot202/blob/master/motioncontroller.py) that these are translated into `speedL` and `speedR`. There are PID controllers for both the right and the left motor (that can be seen in [`motors.py`](https://github.com/DrGFreeman/RasPiBot202/blob/master/motors.py) and a PID controller for angle (see `omegaPID` in `motioncontroller.py`).

The library comes with a number of [examples](https://github.com/DrGFreeman/RasPiBot202/tree/master/examples) including reaching a point via odometry and a maze solver (that depends on additional sensors).

TODO: it's not clear to me what triggers reads from the AStar motor controller - [`robot.py`](https://github.com/DrGFreeman/RasPiBot202/blob/master/robot.py) has method `readAStar` but it's not obvious what calls this - presumably this becomes obvious if one debugs one of the examples (such as reaching a point via odometry).

As well as the library (that doesn't have anything much in the way of documentation) there's an accompanying Pololu [blog post](https://www.pololu.com/blog/675/romi-and-raspberry-pi-robot) and [forum thread](https://forum.pololu.com/t/rpb-202-a-beginners-robot-based-on-romi-chassis-and-raspberry-pi/11243).

In the forum thread DrGFreeman strongly recommends the 2 month long free ["Programming a robotic car"](https://eu.udacity.com/course/artificial-intelligence-for-robotics--cs373) course on Udacity that's from Georgia Tech and is part of a self-driving car nanodegree developed with Mercedes.

---

Rather than worrying about the low level control of the Romi a great idea would be to have the Romi managed by a proper by a proper autopilot like [ArduRover](http://ardupilot.org/rover/).

Stephen Dale has already done this with a cheap Kakute F4 flight controller - US$36 - as shown in this [blog post](https://discuss.ardupilot.org/t/ardurover-with-the-pololu-romi/41991).

At the moment this setup isn't using feedback from encoders, i.e. it's open loop, but in this [separate post](https://discuss.ardupilot.org/t/good-ideas-for-design-of-phone-based-control-for-2-wheel-robot/42885/6?u=ghawkins) Stephen says he's working on this. Unfortunately (IMHO) he's going to switch to the more expensive and significantly larger Pixhawk for this - in my replies to this post I ask about getting encoder reading working with the F4 or possibly the F7. It seems this should definitely be possibly if one [recompiles](http://ardupilot.org/dev/docs/building-the-code.html) ArduPilot with a modified `hwdef.dat` file.

Compare the pins used for motor control in the Kakute F7 [`hwdef.dat`](https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_HAL_ChibiOS/hwdef/KakuteF7/hwdef.dat) (search for `PWM(3)`), the PWM pins for the Pixhawk [`hwdef.dat`](https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_HAL_ChibiOS/hwdef/fmuv3/hwdef.dat) (again search for `PWM(3)`) and the AUX pins mentioned in the rover [wheel encoder documentation](http://ardupilot.org/rover/docs/wheel-encoder.html) (3, 4, 5 and 6). If the Romi setup only used a pin per motor (as is the case with a quadcopter where each motor has its own [ESC](https://en.wikipedia.org/wiki/Electronic_speed_control)) then it looks like you would be able to use 3, 4, 5 and 6 for the encoders without needing to recompile ArduPilot. However as seen in the blog post two pins are required per motor in the Romi rover configuration. But perhaps it would be possible to reassign the pins of the additional UARTs, that the F7 has, for use with the encoders - unfortunately looking at [`WENC_PINA`](http://ardupilot.org/rover/docs/parameters.html#wenc-pina-input-pin-a) etc. it looks like you can only choose from a small predefined set of possible pins.

While ArduCopter can [autotune its PID values](http://ardupilot.org/copter/docs/autotune.html), this functionality is not available in ArduRover but there is [GitHub issue](https://github.com/ArduPilot/ardupilot/issues/8851) to add it. Until then you have to use the tuning guides for steering and speed that are found in the [first drive](http://ardupilot.org/rover/docs/rover-first-drive.html) section of the ArduRover documentation to do this manually.

---

Blogger Pottle has an [article](http://morepootling.blogspot.com/2018/03/dc-motor-control-with-rpi-writing.html) on controlling DC motors with a Raspberry Pi - it doesn't provide much addtiional insight beyond pointing how non-linear response of DC motors.

He seems to feel that this non-linearity means PID values tuned for one particular speed will be the very poor for another.

As I didn't find any discussion of this elsewhere I'm not sure how much of an issue this really is. If the PID values work well at the average speed of the vehicle I wonder how much of an issue it is if they're not the pefect values for very high or very low speeds.

Pottle's article does though link to a very good [YouTube video](https://www.youtube.com/watch?v=8CXReb7f0Eo) of a presentation by David Anderson that covers many of the things you should be thinking about when building two wheeled robots.

The video is quite long and could have done with some editing and the speeding of certain segments but it is worth watching.

Here covers things like:

* Why you should avoid _front_ casters.
* That you should work in terms of the desired velocity and rotation of the robot rather than in terms of the speeds of the two motors (see 17m 20s).
* That a 20Hz control loop is about optimal - i.e. you should update your calculations and motor speed about 20 times per second. Doing more than this will actually be detrimental as you won't accumulate enough encoder ticks per loop for accurate calculations.
* Slew rate - you should limit the rate at which changes in speed occur, i.e. don't bang to a stop but instead slow to a stop.
* You can detect collisions all kinds of ways - including with the IMU.
* The basic strategy on colliding with something is to reverse a bit (where you came from should be safe) and then rotate away from the point of collision, e.g. if the left side collides then rotate right. About 10&deg; is the right amount to rotate - think of what would happen if you were trying to travel down a narrow corridor, if you hit off one wall and rotate away too much (e.g. 90&deg;) then you'll ping-pong down the corridor from one wall to another, whereas with a small rotation of 10&deg; you'll get down the corridor much quicker with much fewer collisions.
* Leaky itegrators (see 58m 2s) - these are a simple way to work out that a strategy is failing you, e.g. if you reverse and turn too often in a short space of time you're probably stuck in a space where your fixed rotation strategy keeps you trapped, and that you need to change behavior.
* Odometry (see 1h 28m 10s) - using just the information from the wheel encoders you can work out where you are relative to your start position. David demonstrates how you can use odometry to stay within a bounding box, how you can use it to achieve a goal, e.g. reach a point 5m ahead even if the vehicle has negotiate around unexpected obstanceles. The accuracy is surprising - coming back almost exactly to the starting location depite travelling far and many turns.

His goal setting example where he tells the robot to achieve a position 5m ahead of its current position is very interesting. It'll work it's way out thru doors etc. to get to that location even if there are walls in the way. I suspect though that if there's a wall in the way it depends on door placement as to whether it ever gets to its goal - without building up some kind of [SLAM](https://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping)-like awareness of its environment it can presumably quite easily end up in long cycles. Though perhaps odometry is enough to tell that it's returned to a location it has already visited and cause it to change its behavior.

TODO: this goal achieving, as the lowest priority behavior behind more immediate responses like avoiding obstacles, results in impressive behavior - it would be nice to wire in the Nano, as if it were an IR sensor, to an Ardupilot setup so Ardupilot could achieve goals with the Nano providing an avoidance behavior (though it seems backwards that the STM32 controller manages the overall process while all the power of the Nano just provides an avoidance behavior).

---

It would be nice to have an online simulator that allowed you to play with Kp, Ki and Kd values and see how this affects achieving the desired value (setpoint).

I didn't find anything perfect but one of the better ones was this [one](https://www.rentanadviser.com/en/pid-fuzzy-logic/pid-fuzzy-logic.aspx) - you have to press one of the plot links, e.g. "Plot Rapid", to see the graphs.

When using "Plot Rapid" (which I found easier to view rather than the continuously moving "Plot Realtime" view) the little red, green and blue boxes tend to obscure the error value in the area of the graph where its most interesting (lower left), but you can click on the graphs and drag to the right to pull this area into view.

OlliW (of [STorM32](http://www.olliw.eu/2013/storm32bgc/) fame) has a brief [wiki article](http://www.olliw.eu/storm32bgc-wiki/PID_online_simulator) on using this simulator.

---

A nicer online tool is the [PID tuner](https://pidtuner.com/) (the code can be found in this [repo](https://github.com/pidtuner/pidtuner.github.io) on GitHub) where you can paste in input and output data from your real system and then build a model from this and then let it autotune the appropriate PID values. It comes with canned data, so you can use it without providing your own data and step onto selecting a time range (if you select a range that doesn't include a full step you'll end up with a very odd model), then a model (the simple first order model always seemed good) and then let it choose Kp, Ki and Kd values. You can then adjust these values, e.g. quadruple the proportional gain, and see the oscillations and other issues that you're trying to avoid with properly tuned values.

Note: if you change a gain value you have to then press enter, before leaving the field, in order to update the graphs (I filed an [issue](https://github.com/pidtuner/pidtuner.github.io/issues/3) suggesting this should be clarified).

---

Alan McDonley has a [nice post](https://www.raspberrypi.org/forums/viewtopic.php?p=803102&sid=43482eac271db7a984b4043195a79f08#p803102) where he covers 22 factors than come into play when getting your robot to drive as best it can - he covers things like determining minimum and maximum speed, various biases, encoders, sensor fusion and more.

Interestingly he also mentions a self-calibration by George Musser from 2000 for two wheel robots of any configuration. He briefly introduces some of the constants etc. in this library in a follow-up message](https://www.raspberrypi.org/forums/viewtopic.php?p=803266&sid=64b2f862be4a0ab1678bda2542f984c3#p803266) in the same thread.

Alan started the thread in the hope of finding better informed people than him on the Raspberry Pi forums - but unfortunately he turns out to be the best informed person (so the other replies on the thread aren't worth reading).

---

Bill Marshall has a nice four part [series](https://www.rs-online.com/designspark/give-your-robot-the-mobility-control-of-a-real-mars-rover-part-1) on building a Mars Rover like vehicle - it covers PID nicely (include good graphs) in the first part, covers encoders in the second and looks at various issues of getting a real system going in parts three and four.

---

For another nice basic introduction to PID controllers see Osoyoo's ["Self balancing robot - PID control"](http://osoyoo.com/2018/08/08/self-balancing-robot-pid-control/) - it goes over the basics again with nice graphs showing the effect of Kp and Ki (but not of Kd).

This page relates to a robot that Osoyoo sells - a rather neat looking [two wheel self balancing car](http://osoyoo.com/2018/07/18/osoyoo-balancing-car/). This actually involves three PID loops - one related to keeping the upper body from toppling over (the self-balancing part), one managing the speed of the wheels and one managing rotation.

The author of the PID page mentions tuning the PID values by hand but then leaving it to the autotuning library of the live system to refine these. It's not clear what library he's refering to here - if you look at the [sketch](https://github.com/osoyoo/Osoyoo-development-kits/blob/master/OSOYOO%202WD%20Balance%20Car%20Robot/osoyoo_abc/osoyoo_abc.ino) running on the Osoyoo robot you can see that it clearly doesn't tune the PID values itself but that some of them can be updated via Bluetooth, and if you you look at the [control application](https://github.com/osoyoo/Osoyoo-development-kits/blob/master/OSOYOO%202WD%20Balance%20Car%20Robot/Android%20app%20source%20code/app/src/main/java/com/bluetooth/admin/balencecar/MainActivity.java), that can send these updates, you can find the word "pid" all over the place (including `KEY_AUTO_UPDATE_PID`) but its not obvious that anything in this code is trying to automatically tune the PID values.

---

For yet another basic introduction to PID controllers see ["Arduino PID control tutorial"](https://www.teachmemicro.com/arduino-pid-control-tutorial/) - it doesn't add much and some of the explantions don't seem entirely accurate but it does sum up the process in a nice diagram (taken from the Wikipedia PID article mentioned up above) and does introduce using the Arduino PID library for motor control.

---

Arduino forum user kas has a nice [series of posts](https://forum.arduino.cc/index.php/topic,8871.0.html) that cover developing a self-balancing robot (using components from Pololu). This includes using a Kahlman filter for sensor fusion and using a PID controller - and presents a simple method for adjusting Kp, Ki and Kd in turn to achieve a tuned system (see the [main PID post](https://forum.arduino.cc/index.php?topic=8871.msg73977#msg73977) in the series).

---

Mark Booth has a nice Robotics StackExchange [answer](https://robotics.stackexchange.com/questions/1232/how-can-i-use-the-arduino-pid-library-to-drive-a-robot-in-a-straight-line) that briefly covers using the Arduino PID library and then looks at dead-reckoning and using Kahlman filtering to include accelerometer data.

Note: odometry is a subtype of dead reckoning (see this Robotics StackExchange [answer](https://robotics.stackexchange.com/a/7289)).

---

There are many PID related projects (mainly user notebook writeups) on the main Mbed site - none of them seem terribly detailed, but an iteresting looking one (from the Cookbook Wiki) is the [mbed rover](https://os.mbed.com/cookbook/mbed-Rover) which uses both encoders and an IMC and introduces the mbed quadrature, PID and IMU libraries.

---

Twiddle
-------

The already mentioed "Programming a robotic car" (CS373) MOOC from Georgia Tech introduces an algorithm called coordinate ascent or twiddle.

The instructor for the course, Sebastian Thrun, describes the algorithm in details in this [YouTube video](https://www.youtube.com/watch?v=2uQ2BSzDvXs).

As well as being part of CS373 this video is part of CS271 - "Intro to artificial intelligence" (where the other instructor is Peter Norvig - director of research at Google). [Sebastian Thrun](https://en.wikipedia.org/wiki/Sebastian_Thrun) was a professor of computer science at Stanford and is now chairman and co-founder of Udacity. He led the team that won the DARPA Grand Challenge in 2005 (and came second in 2007) and led Google's self-driving car project until 2012.

Twiddle is described so in a [Hacker news thread](https://news.ycombinator.com/item?id=16267168):

> Twiddle, AKA coordinate ascent, ... It's basically the same process a human uses to tune PID parameters, cyclically tune one parameter at a time until you're performance is good enough. It's closely related to gradient ascent and suffers from the same problems, namely that it can get trapped in a local minimum, but it's easy to implement and does a good job if you start with an ok guess at the parameters.

Oddly the terms gradient ascent (GA) and gradient descent GD) seem interchageable in these contexts (see the gradient section of the [optimization page](https://frnsys.com/ai_notes/foundations/optimization.html)).

Note: this Electronics StackExchange [answer](https://electronics.stackexchange.com/questions/50049/how-to-implement-a-self-tuning-pid-like-controller) extolls the virtues of the [Nelder-Mead method](https://electronics.stackexchange.com/questions/50049/how-to-implement-a-self-tuning-pid-like-controller) which is a hill climbing simplex algorithm. However this Maths StackExchange [answer](https://math.stackexchange.com/a/2419865) seems to make clear that such gradient-free optimization methods will tend to converge slower and have weaker convergence guarantees than gradient-based methods. So from this I take it that, while it has limitations, Twiddle will perform better than such methods.

The playlist for the full CS373 course can be found [here](https://www.youtube.com/playlist?list=PL00A5201296749516).

Implementing Twiddle in Python is an exercise that's part of the nanodegree of which CS373 is a part. James Dunn presents a solution [here](https://github.com/jwdunn1/CarND-PID-Parameter-Optimization) with an apparent improvement that Andres Castano came up with on the Udacity course forum (I asked James for more details on this [here](https://github.com/jwdunn1/CarND-PID-Parameter-Optimization/issues/1)).

Another Python implementation can be found [here](https://martin-thoma.com/twiddle/) - as can be seen it's trivial. The author notes:

> I guess gradient descent might be better for most cases, but Twiddle does not require any knowledge about the algorithm _A_ which might be a big advantage. And you don't have to calculate the gradient of high dimensional functions, which is nice, too.  

Where _A_ is the algorithm that determines the error when given a particular parameter vector.

---
---

TODO: I'm sure this links are all part of one big section - find out the structure.

["PID autotuning in real time"](https://ch.mathworks.com/help/slcontrol/ug/pid-autotuning-in-real-time.html)
["How PID autotuning works"](https://ch.mathworks.com/help/slcontrol/ug/how-pid-autotuning-works.html)

["PID controller tuning (automatic tuning of PID gains in Simulink® and real-time environments)"](https://ch.mathworks.com/help/slcontrol/cat-scd-pid-controller-tuning.html) - is this the root?

* ["Model-based PID controller tuning"](https://ch.mathworks.com/help/slcontrol/automatic-pid-tuning.html)
* ["Real-time PID autotuning"](https://ch.mathworks.com/help/slcontrol/cat_scd_pid_autotuning.html)

---

[pypid](https://pypi.org/project/pypid/) - Python PID controller with autotuning. It was developed between 2008 and 2014 - since then Jan Binder has a [version](https://github.com/drogenlied/pypid) on Github that's been ported to Python 3 and has various fixes.

---

TODO: work this in elsewhere.

[Manual tuning section](https://en.wikipedia.org/wiki/PID_controller#Manual_tuning) of the Wikipedia page for PID controller.

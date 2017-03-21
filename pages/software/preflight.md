# Flight preparation


## Sensors calibration

TBD - no calibration is needed at the moment. It means use your default `calibration.conf` file and do not change it.

## Software
If unsure whether you have the right setup, follow the instructions on [smaccmpilot-hardware-prep][]

[smaccmpilot-hardware-prep]: https://github.com/GaloisInc/smaccmpilot-hardware-prep

Once done with the hardware prep, load *smaccmpilot* autopilot. Temporarily (as of 3.17) use `new-attitude-estimator` branch from [Github][]. Don't forget to do:

```
stack clean
make clean
make flight_echronos
```

[Github]: https://github.com/GaloisInc/smaccmpilot-stm32f4/commits/new-attitude-estimator


## Radio setup
Configure your controller as described on [Radio Control] page, and familiarize yourself with the position of **3-position MODE** switch and **KILL** switch. Your trims should be at zero.

[Radio Control]: ../hardware/rc-controller.html

## Pre-flight checks
If this is your first flight, lets do a couple of extra checks to make sure everything works as it should.

1. flash and load the autopilot with `make upload_flight_echronos`
2. open up GCS and check the Heads-Up-Display (HUD) - the aritifical horizon should be close to leveled and with minimal yaw drift
3. check the [acceleration] - you should see around -9.8m/s^2 in z-axis, and around zero for x and y-axis
4. check the [gyro] - you should see very close to zero in all three axis
5. move the quadcopter by hand in the air and see if the HUD looks sane

Now you can move on to *Arming*

[acceleration]: http://localhost:8080/sensors_accel.html
[gyro]: http://localhost:8080/sensors_gyro.html

## Arming
The arming sequence is as usual. There is no preflight calibration needed (no "dance" with the quadcopter). The steps are:

1. turn on your radio transmitter and make sure throttle is down, MODE SWITCH is in Manual, and KILL SWITCH is ON (throttle kill)
2. plug in the battery - try not to wiggle the airframe excessively, as it can influence the initial orientation estimate
3. press the red button on top of the Iris until you hear a distinct tone from the motors (yes, the motors can play sounds).
4. check the GCS HUD  - the aritificial horizon should be more-or-less leveled and with minimal yaw drift
5. *unkill* KILL SWITCH (i.e. move it up/towards yourself with your index finger)
6. arm motors by pushing the yaw stick to bottom right

## Take-Off
For your very first flight you should try to increase the throttle very slowly, and when the quadcopter is just about to take-off (around 60% of throttle) give it small roll/pitch/yaw inputs to see if it responds correctly to your commands. If it doesn't go back to the [Radio Control] page and make sure your radio is set up correctly. Note that this near-takeoff phase induces lot of vibration in the airframe and can confuse the attitude estimator. That is OK, because in pracice we will never dwell in this phase a lot. The HUD might get some offset, but it should never diverge.

Once you made sure your quadcopter responds properly, go ahead and increase the throttle. For all future take-offs, I recommend taking off rather quickly - ramp up the throttle to ~70% and get off the ground fast, it is the best for the estimator.

## Trim
The next step is to trim the quadcopter. Trim adds a constant input to each of the roll/pitch/yaw axis to counteract the imbalance in the airfame, not perfectly equal motors and small bias in the attitude estimation. It *holds the stick* in place for you, to make it easier for the pilot to control. The goal of the trim is to have the quadcopter in almost pefrect hover without any stick movement.

How to do this? First, make sure all your trims are zeroed out. Click the trim buttons until you see on the transmitter display the little black circle for each axis over a little black square and hear a little beep.

![RC trims](/images/rc_trims.png)

![Zeroed trims](/images/rc_trims_zeroed.png)

### Trim procedure
1. Take-off and fly the quadcopter in manual. 
2. Achieve a steady flight, and let go the right (roll/pitch) stick for a second. Notice where the quadcopter leans (left, right, forwards, backwards) and land.
3. Land
4. Move the trim button for the respective axis a few clicks and go back to 1.
5. Repeat until the quadcopter stops leaning in any direction

A small example for better illustration. 
1. I notice the quadcoper is rolling left when I let go the stick. That means to counteract this, I need to move my roll stick to the right. I land, and give a few clicks *right* on my roll trim. I repeat until the left rolling tendency disappears.
2. I notice the quadcopter is pitching forward. I have to trim the roll a few clicks *down* as if I was countering the forward pitch with my pitch stick. 
3. When roll and pitch are trimmed, I focus on yaw. In stable flight, I notice that the quadcopter slowly turns right without any yaw input. I land and give a few cliks *left* on my yaw trim, to counter it.

Here is an example of a radio transmitted trimmed for a particular quadcopter:

![Trimmed radio](/images/rc_trims_tuned.png)

## Altitude hold
Once you trimmed the quadcopter, it is time to do the alt-hold. There are two things to do before a succesfull alt-hold:

1. fly in manual mode and notice what is your nominal throttle level at a stable flight. You can use [alt-setp](http://localhost:8080/alt_setp.html) interface to see the control input (bottom plot). For our Iris without the daugherboard the nominal throttle is aroun 65%, with the additional weight it might be closer to 70-75%
2. Change the nominal throttle in [tuning.conf](https://github.com/GaloisInc/smaccmpilot-stm32f4/blob/new-attitude-estimator/src/smaccm-flight/tuning.conf#L6) accordingly - remember that the value has to be between 0-1, so 70% will be 0.7
3. Recompile and upload.

To operate alt-hold (we recommend always starting with a full battery):

1. take-off in manual, and bring the quadcopter to a leveled flight.
2. flip the mode switch into **Alt-hold** mode
3. bring the throttle stick all the way to 100% to give the controller full authority
4. enjoy!
5. to land you can either slowly lower the throttle until the quadcopter descends to the ground, or lower the throttle close to the nominal hover throttle and then switch back to manual mode

Troubleshooting:

1. the quadcopter looses altitude even though the throttle stick is all the way to 100%: try to increase the [I-gain](https://github.com/GaloisInc/smaccmpilot-stm32f4/blob/new-attitude-estimator/src/smaccm-flight/tuning.conf#L21) by a bit, maybe change it to like 0.009
2. the quadcopter climbs slowly and never stops: make sure your [nominal throttle](https://github.com/GaloisInc/smaccmpilot-stm32f4/blob/new-attitude-estimator/src/smaccm-flight/tuning.conf#L6) is set properly, maybe try to lover it a notch.


## GCS input mode
The last step is the GCS input mode. In this mode, the pilot input's mix with the GCS operator inputs. That way the operator can steer the quadcopter where they want, while the pilot can always step in. 

The procedure is following:

1. take-off and go to **Alt-hold** mode
2. once in a good alt-hold, switch to **GCS input** mode
3. steady the quadcopter, center the sticks (and be ready to step in if needed)
4. have the GCS operator to steer the quadcopter as needed
5. to come back, you can switch back to **Alt-hold** at any point, and land as normally
6. Have fun!

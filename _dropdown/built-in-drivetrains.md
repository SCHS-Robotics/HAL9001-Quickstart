---
layout: page
title: HAL Built-in Drivetrains
description: How to use the HAL9001 Config System
dropdown: Tutorials
priority: -7
---
-------------------
If you've participated in FTC for a while, you are probably familiar with having to either copy-paste or re-write the same code for your team's drivetrain system over and over again in every program where the robot needs to move (which is, well, most programs). Also, because many teams use the same drivetrain from year to year (looking at you, mecanum drive), you might have found yourself having to copy-paste code from the previous year's repository. This wastes time and is a bit annoying. 

Plus, if you are just switching to HAL, you probably won't want to have to port your drivetrain code into its subsystem-robot-opmode architecture structure. HAL gets around this problem by including its own built-in drivetrain subsystems! There is currently support for tank drive (any number of wheels, as long as there is the same number on each side), mecanum drive, and x-drive. SWERVE DRIVE DOES NOT CURRENTLY HAVE A BUILT-IN SUBSYSTEM, you will have to make this subsystem yourself if you want to use swerve drive. This actually isn't that hard to do. It is recommended that you follow the guidelines in the `Making A Custom Drivetrain Subsystem` section if you choose to do this.

## Simple vs Normal
Each built-in drivetrain has two versions. A "simple" version, and a normal version. To use a mecanum drivetrain, for example. You could use either the MecanumDriveSimple subsystem or the MecanumDrive subsystem. The difference between these two subsystems is that one has built-in integration with the Roadrunner library (MecanumDrive), and one does not (MecanumDriveSimple). 

MecanumDrive has all the same functions as MecanumDriveSimple, plus a few more that allow it to follow smooth trajectories using the roadrunner library. There will be more information given on this topic in the `Roadrunner Integration` section. Until that section, we will use examples with MecanumDriveSimple.

## An Example!
-----------------------------
### Declaring the Drivetrain
Here is a simple example of the built-in mecanum drivetrain subsystem class being used in a robot:

```java
public class ExampleRobot extends Robot {
    public MecanumDriveSimple drive;
    public ExampleRobot(OpMode opMode) {
        super(opMode);

        drive = new MecanumDriveSimple(
                this,
                new DriveConfig(2, 1, 15, 1120, 133.9),
                "front_left_motor",
                "front_right_motor",
                "back_left_motor",
                "back_right_motor",
                false
        );
        drive.setLocalizer(new HolonomicDriveEncoderIMULocalizer(
                this,
                drive,
                "imu",
                "front_left_motor",
                "front_right_motor",
                "back_left_motor",
                "back_right_motor"
        ));
    }
}
```

Ok, lets break this example down a little. First, the MecanumDriveSimple class is declared and created just like any subsystem, and works exactly the same as all custom subsystems you make in your class. The only thing special about MecanumDriveSimple is that it is built-in to HAL. Now that we've clarified that, lets go over each of the parameters in the subsystem's constructor and what they mean.

The first parameter is the robot its being added to, (this robot). This is going to be the case for basically any HAL subsystem. The second parameter, however, is more interesting. This is the drive config, which specifies certain hardware constraints for the drivetrain. The drive config stores (in order of where they are in the DriveConfig constructor) the robot wheel radius, the gear ratio between the motors and the wheels, the robot track width, the number of encoder ticks per revolution, and the motor maximum RPM. For this example robot, the wheel radius is two inches, the gear ratio is 1 (the motors are attached directly to the wheels, or are geared to them with a 1:1 gear ratio), the track width is 15 inches, the drivetrain encoders record 1120 ticks per motor revolution, and the motors have a maximum RPM of 133.9. (Note: the DriveConfig can use units other than inches if you add HALDistanceUnit.(desired unit) after the parameter you want to change units for).

The wheel radius and gear ratio should be relatively straightforward to find after a discussion with your hardware team. (A quick note, your hardware team may have learned that gear ratio = output number of teeth/input number of teeth. This is correct, but not exactly what gear ratio means in this context. This is the same gear ratio used by roadrunner, which means its 1/that calculated gear ratio. This is because the gear ratio is supposed to be talking about what factor the VELOCITY is increased by, not the torque. Also, this is the gear ratio of the motor output to the wheel input. NOT the motor gear ratio. A 60:1 motor does not mean the gear ratio should be set to 60. __*Make sure to show this to your hardware team to prevent any confusion*__). 

The track width is theoretically the distance between the two centers of the wheels on each side of your robot. This value can change from the theoretical value due to things like wheel scrub/friction, however, so it may be slightly different than what is measured by a ruler. In order to get a more accurate value for the track width, the TrackWidthTuner can be used ([here](https://github.com/SCHS-Robotics/HAL_Simulator/blob/master/TeamCode/src/org/firstinspires/ftc/teamcode/TrackWidthTuner.java) is a version that uses the HAL library, although you have to remove the `waitTime(DELAY);` line of code for it to work accurately. That line was there because the simulator couldn't update fast enough). The motor ticks per revolution and maximum RPM can be found on the motor's datasheets. This is often not made very clear for non-gobilda motors, but [here](https://www.learnroadrunner.com/drive-constants.html#ticks-per-rev-max-rpm) is a list of motor ticks per revolution and maximum rpms that should make it easier.

The next set of parameters in the constructor are relatively self-explanatory. These are the hardware configuration names for each of the 4 drive motors. It is important that the configuration names be entered in the order shown in the example, so that it knows which motors coorespond with which configuration name.

The final parameter, the boolean value false, is actually an optional parameter. It lets the subsystem know whether or not it should allow itself to be configured using the HAL config system. Ommitting this parameter or setting it to true will cause the subsystem to use the HAL config system.

Now, the next thing we need to go over is the section of code that runs immediately after we create the MecanumDriveSimple subsystem:

```java
drive.setLocalizer(new HolonomicDriveEncoderIMULocalizer(
        this,
        drive,
        "imu",
        "front_left_motor",
        "front_right_motor",
        "back_left_motor",
        "back_right_motor"
));
```

This code sets which localizer the drivetrain uses to determine its position and heading on the field. *By default, all drivetrains use only drive encoders to determine position and heading*, which is a *BAD IDEA!!!* The only reason it is coded this way is so that people don't have to use the imu by default. It is recommended to use at *MINIMUM* the drive encoders and rev hub imu for robot localization (position + heading tracking). This is what this code does, it sets the drivetrain to a "Holonomic Drive Encoder Localizer", which uses the drive encoders of a holonomic (omni-directional) drivetrain and the rev hub/control hub imu for localization (if you wanted to use a non-holonomic drivetrain, you would use NonHolonomicDriveEncoderImuLocalizer). 

A BNO055IMU.Parameters object can also be inserted as a parameter after the "imu" part of the localizer's constructor in order to give the created IMU specific parameters. This is true for all built-in localizers that use the IMU. Also note that the localizer takes the drivetrain that uses the localizer as input. This is true for all localizers that use the drive encoders. 

More information about localizers, including other good localizers, will be presented in the `Roadrunner Integration` section. For now, just remember to set the localizer to an *ACTUALLY GOOD* one, unless you don't want to use the imu for some reason. Also, important note: *if your rev hub is mounted to the robot in a non-standard way, you can use `.remapIMUAxes()` to still get the correct angle values from the imu*. This function is present in all localizers that use the imu. Heres an example of it being used: 

```java
drive.setLocalizer(new HolonomicDriveEncoderIMULocalizer(
        this,
        drive,
        "imu",
        "front_left_motor",
        "front_right_motor",
        "back_left_motor",
        "back_right_motor"
).remapIMUAxes(AxesOrder.ZYX, AxesSigns.NNN));
```

One last thing that you may need to set is the "Reverse Type". This controls which motors in the drivetrain are reversed. Usually, the left side of the robot has reversed motors, but this is not always the case. To set the motor reverse type, you can call `drive.setReverseType()` and pass in whichever reverse type fits your robot. Ex:

```java
drive.setReverseType(MecanumDriveSimple.ReverseType.LEFT);
```

If none of the available reverse types fit your specific drivetrain, you can reverse motors on an individual basis using `drive.reverseMotor()` (pass in the motor name to tell it which motor to reverse, ex: `drive.reverseMotor("front_left_motor")`) or `drive.setMotorDirection()` (pass in the motor name and the desired direction of the motor, ex: `drive.setMotorDirection("front_left_motor", DcMotor.Direction.REVERSE)`)

### Using the drivetrain!
Ok, so now you have your bright, shiny, brand new HAL drivetrain! (warranty not included, batteries sold separately) But how do you actually use it? Well first, lets go over some features that you have to play with during the setup process! (they can be set in the robot, or elsewhere in the program).

#### Direct Motor Access
Alright, I admit this one isn't that cool, but it is good to know. *All drivetrains* give you direct access to motors if you really need it. The functions to do that are described below:

This function sets the power of a specific motor. In this example, it's setting the front left motor's power to 0.7.

```java
drive.setMotorPower("front_left_motor", 0.7);
```

This function sets the run mode of a specific motor. In this example it's setting the front left motor's run mode to RUN_WITHOUT_ENCODERS (*by default all motors are set to RUN_USING_ENCODERS*)

```java
drive.setMotorMode("front_left_motor", DcMotor.RunMode.RUN_WITHOUT_ENCODER);
```

This function will set the zero power behavior of a specific motor. In this case it's setting the front left motor's zero power behavior to FLOAT. (*by default all motors have a zero power behavior of BRAKE to counteract the robot's inertia when doing maneuvers, change at your own risk*)

```java
drive.setMotorZeroPowerBehavior("front_left_motor", DcMotor.ZeroPowerBehavior.FLOAT);
```

This function will reverse the direction of a specific motor. By reverse, I mean if it is in the forward mode, it turns it to the reverse mode, and if it is in the reverse mode, it turns it to the forward mode. This was mentioned earlier in this tutorial. In this example, the direction of the top-left motor is being reversed.

```java
drive.reverseMotor("front_left_motor");
```

This function will set the direction of a specific motor. This was mentioned earlier in this tutorial. In this example, the direction of the top-left motor is being reversed. *note the difference between this function and reverseMotor()!*

```java
drive.setMotorDirection("front_left_motor", DcMotor.Direction.REVERSE);
```

This function sets the run mode for all motors in the drivetrain. In this case, it's setting all the motors to RUN_WITHOUT_ENCODERS.

```java
drive.setAllMotorModes(DcMotor.RunMode.RUN_WITHOUT_ENCODER);
```

This function sets the zero power behavior for all motors in the drivetrain. In this case, it's setting all the motors to FLOAT.

```java
drive.setAllMotorZeroPowerBehaviors(DcMotor.ZeroPowerBehavior.FLOAT);
```

This function sets the PIDF coefficients for the velocity PID controllers of all the motors. It auto-scales kF depending on how much voltage is left in the battery, allowing your custom coefficients to compensate for not being at max voltage. It also requires the desired run mode of all motors. (note: the PIDF coefficients in the example are just the number 1 over and over, and are probably not good coefficients).

```java
drive.setMotorPIDFCoefficients(DcMotor.RunMode.RUN_USING_ENCODERS, new PIDFCoefficients(1, 1, 1, 1));
```

This function resets all motor encoders to 0.

```java
drive.resetMotorEncoders();
```

This function stops all the motors by setting their powers to 0. *THIS IS AN IMPORTANT FUNCTION TO KNOW :)*

```java
drive.stopAllMotors();
```

This function gets an array containing all the motors (in the same order as shown in the MecanumDriveSimple constructor.

```java
DcMotorEx[] motors = drive.getMotors();
```

This function gets an array containing the config names of all the motors (in the same order as shown in the MecanumDriveSimple) constructor.

```java
String[] motorConfig = drive.getMotorConfig();
```

This function gets a specific motor object with a given motor config name. In this case, the front-left motor.

```java
DcMotorEx motor = drive.getMotor("front_left_motor");
```

This function gets the encoder position of a specific motor. In this example, we are getting the positon of the front-left motor.

```java
double encoderPos = drive.getMotorEncoderPosition("front_left_motor");
```

This function gets the velocity of a specific motor. It can be called without specifying units, which will give the velocity in terms of encoders/sec, or it can be called with an angle unit to try and get the velocity in radians/sec or degrees/sec. In both examples below, we are getting the current velocity of the front left motor.

```java
double motorVelocityEncoders = drive.getMotorVelocity("front_left_motor");
double motorVelocityRadians = drive.getMotorVelocity("front_left_motor", HALAngleUnit.RADIANS);
```

#### PID and Turning

All HAL drivetrains now come with built-in heading and turn-to-angle PID controllers. Initially, they will have coefficients that are all zero, and so will be disabled, but their coefficients can be set using `drive.setTurnPID()` and `drive.setHeadingPID()`. Two examples of this are provided below (note: not actual PID coefficients, it's just all ones, which are likely bad coefficients)

```java
drive.setTurnPID(new PIDCoefficients(1, 1, 1));
drive.setHeadingPID(new PIDCoefficients(1, 1, 1));
```

You can also set the tolerance values for the heading PID controller using `drive.setHeadingPIDTolerance()`. The tolerance value defaults to +- 0.01 radians. Putting a unit after the tolerance value will cause the drivetrain to interpret the tolerance as being of that unit. Not putting a unit will make it interpret it as being in radians.

```java
drive.setHeadingPIDTolerance(0.1, HALAngleUnit.DEGREES); //Sets the heading PID to require a heading accuracy of between +- 0.1 degrees (~0.002 radians)
```

All HAL built-in drivetrains also have the same collection of functions for turning. The functions are listed and described below:

This function tells the drivetrain to turn at a specific power. A positive power indicates a counterclockwise turn, and a negative power indicates a clockwise turn. This example makes the drivetrain turn at a powe of 0.7 counterclockwise.

```java
drive.turnPower(0.7);
```

This function tells the drivetrain turn turn at a specific power for a certain amount of time. The power follows the same rule for counterclockwise/clockwise as the previous function. Units of time can be added to this function to cause the drivetrain to interpret that parameter differently. If a unit is not provided for the time parameter, it is assumed to be in milliseconds. In this example, the drivetrain turns at a power of 0.7 counterclockwise for 2 seconds, then turns at a power of 0.3 counterclockwise for 1000 milliseconds (1 second).

```java
drive.turnTime(0.7, 2, HALTimeUnit.SECONDS);
drive.turnTime(-0.3, 1000);
```

This function tells the drivetrain to turn at a specific power by a certain angle. *THIS RELIES ON THE LOCALIZER WORKING CORRECTLY*. If your localizer got messed up, either by you giving it incorrect parameters, a hardware malfunction, or something else, this may not work as intended. Also note that this turns BY a specific angle, not TO a specific angle. *Power should be positive in this function, as the sign of the angle will specify the turn direction*. Positive angles indicate counterclockwise turns, while negative angles specify clockwise turns. Units can be given for the angle in this function like normal. If units are ommitted, the angle is assumed to be in radians. In the examples shown below, the drivetrain turns 90 degrees clockwise at a power of 0.7, then turns PI/2 radians (90 degrees) at a power of 0.3, which will bring it back to its starting orientation.

```java
drive.turnSimple(0.7, 90, HALAngleUnit.DEGREES);
drive.turnSimple(0.3, -PI/2);
```

This function uses the turnPID controller to turn TO a specific angle. *THIS RELIES ON THE LOCALIZER WORKING CORRECTLY*. If your localizer got messed up, either by you giving it incorrect parameters, a hardware malfunction, or something else, this may not work as intended. Units can be specified for the target angle and the tolerance. Ommitting the units for both will result in the entries being interpreted as being in radians. Ommitting the tolerance parameter will cause it to use a tolerance of 1e-2 (0.01 radians). In this example, the drivetrain will turn to an angle of 45 degrees counterclockwise from its forward direction with a tolerance of 1 degree. 

```java
drive.turnPID(45, HALAngleUnit.DEGREES, 1, HALAngleUnit.DEGREES);
```

#### Moving!
These functions are VERY similar in style to the turning functions. The difference, however, is that what they take as input varies between Holonomic (omnidirection) and non-holonomic (non-omnidirectional) drivetrains. *In holonomic drivetrains, the move functions take vectors as input in order to specify the drivetrain's desired direction, while in non-holonomic drivetrains, the move functions just take a double as a power input*, where positive is forward and negative is backward. *Because holonomic drivetrains are much more common in FTC, all the examples below will be for holonomic drivetrains*.

This function tells the drivetrain to move at a certain power vector. In this case, the drivetrain is being told to move at a power of 0.7, 45 degrees to the left. More information on how Vector2D objects work is present in the math library tutorial, but how it is being used in this example should be relatively self-explanatory.

```java
drive.movePower(new Vector2D(0.7, 45, HALAngleUnit.DEGREES));
```

This function tells the drivetrain to move at a certain power vector for a certain amount of time. In this case, the drivetrain is being told to move at a power of 0.7, 45 degrees to the left, for 2 seconds. More information on how Vector2D objects work is present in the math library tutorial, but how it is being used in this example should be relatively self-explanatory. *If time units are ommitted from this function, it will assume the units are milliseconds*.

```java
drive.moveTime(new Vector2D(0.7, 45, HALAngleUnit.DEGREES), 2, HALTimeUnit.SECONDS);
```

This function tells the drivetrain to move by a certain displacement vector at a certain power. *Note here the vector object refers not to power, but displacement*. In this case, the drivetrain is being told to move at a power of 0.3, for a displacment of 2 tile lengths at an angle of 30 degrees to the left. If the distance unit is ommitted, the function will assume the displacement is in inches.

```java
drive.moveSimple(new Vector2D(2, 30, HALAngleUnit.DEGREES), HALDistanceUnit.TILES, 0.3);
```

This function will try and move and turn at the same time with the given velocity power vector and turn power double. However, if you want to move and turn at the same time to make arcs accurately/well, I would suggest looking at the roadrunner section. This example tries to move at a power of 0.4 forward while turning right at a power of 0.5.

```java
drive.arcPower(new Vector2D(0.4, 0, HALAngleUnit.DEGREES), -0.5);
```

This function sets the raw power going to the motors, and rescales it to make sure no power ever exceeds 1. Unlike all other methods of setting power, this is not affected by any modifiers. This function works for both holonomic and non-holonomic drivetrains. In this example, the top left motor power is set to 0.5, the top right motor power is set to 0.6, the bottom left motor power is set to 0.7, and the bottom right motor power is set to 0.8.

```java
drive.setPower(0.5, 0.6, 0.7, 0.8);
```

#### Holonomic Drivetrain Drive Modes
Holonomic drivetrains have two different "drive modes," which tell the drivetrain whether to use field centric or robot-centric coordinates. If the drivetrain uses field-centric coordinates, the drivetrain's starting "front" position will ALWAYS be its front position, regardless of the robot's current orientation. If the drivetrain uses robot-centric coordinates, the "front" of the drivetrain will be the front of the robot. Heres an image comparing the two drive modes (with each motor force vector given a letter for clarity):

![](images/standard-vs-field-centric.jpg)

Which mode is used is purely preference based. *The drive mode will affect both teleop AND autonomous code*. If in teleop, moving the joystick forward will cause the drivetrain to move forward relative to where it thinks "forward" is. With field centric, thats the constant "front" direction. With standard, thats the current "front" direction of the robot. In autonomous code, the drive mode will affect move functions. Movement displacement and power vectors will assume that the y axis points along the robot's "front" side. If in field centric mode, then that will be relative to the global "front" position. If in standard mode, then that will be relative to the drivetrain's current "front" position. *By default, the drivetrain will start in standard mode*. The drive mode can be switched using the function `drive.setDriveMode()`. An example of this function being used to change the drivetrain to field centric mode is given below:

```java
drive.setDriveMode(HolonomicDrivetrain.DriveMode.FIELD_CENTRIC);
```

There is a third drive mode called "DISABLED," which is used for people who want to have a custom drivetrain teleop function, but don't want to make an entirely new drivetrain subsystem. In autonomous functions it will act like standard mode, but in teleop it will completely disable the drivetrain's handle() function (with the exception of the part where the localizer gets updated), allowing people to override `onUpdate()` in a HAL teleop program to control the drivetrain. Also, *non-holonomic drivetrains do not have a field centric mode*.

#### Localizer Functions
Ok these are also not the most exciting, but they are *really* good to know. Especially for debugging. These functions allow you to easily interact with the localizer easily. Each function is described below:

This function gets the actual instance of the localizer object used in the drivetrain.

```java
Localizer localizer = drive.getLocalizer();
```

This function gets the localizer's current estimate of the robot's pose (x,y position and heading).

```java
Pose2d currentPose = drive.getPoseEstimate();
```

This function gets the localizer's current estimate of the robot's pose velocity (x,y velocity and heading velocity).

```java
Pose2d currentPoseVelocity = drive.getPoseVelocity();
```

This function sets the localizer's current position estimate. This can be used to *maintain the robot's estimated positon across opmodes without recalibrating anything*. This example sets it to a pose of (24, 24, PI/2). The x,y coordinates are in inches, and the heading is in radians.

```java
drive.setPoseEstimate(new Pose2d(24, 24, PI/2));
```

This function updates the localizer, causing it to update its current estimate of the robot's pose. *THIS IS A VERY IMPORTANT FUNCTION*.

```java
drive.updateLocalizer();
```

#### Drivetrain Bells and Whisles

Now this is the good stuff. All the cool features that HAL drivetrains add without you having to do any work are in this section. Alright, enough talk! Lets show off some of the cool features!

These two functions can set a cap on the maximum movement power or turn power that can be sent to the drivetrain. If the absolute value of the power ever exceeds the cap, it is set to cap value. This example caps the velocity at a power of 0.7, and the turn speed a power of 0.6.

```java
drive.setVelocityCap(0.7);
drive.setTurnSpeedCap(0.6);
```

These two functions can set the value of a multiplier that is applied to the movement power or turn power (respectively) whenever the power is used. In this example, both multipliers are set to 0.5, so the turn speed and movement speed will always be multiplied by a constant of 0.5.

```java
drive.setVelocityMultiplier(0.5);
drive.setTurnSpeedMultiplier(0.5);
```

These two functions can set a velocity scaling method for both movement speed and turn speed. There are three different available scaling methods: NONE (the default, no scaling besides constant multipliers), SQUARE (squares the magnitude of the drivetrain power, making it move slower at lower speeds but still fast at higher speeds), and CUBIC (cubes the magnitude of the drivetrain power, making it MUCH slower at lower speeds but still very fast at higher drivetrain values). These scaling methods are very useful in teleop, where it is often necessary to make both precise positional and rotational adjustments and rapid darts across the field. In this example, both scaling methods are being set to CUBIC.

```java
drive.setVelocityScaleMethod(Drivetrain.SpeedScaleMethod.CUBIC);
drive.setTurnSpeedScaleMethod(Drivetrain.SpeedScaleMethod.CUBIC);
```

This function sets what is referred to in the roadrunner quickstart as the "OMEGA_WEIGHT", which is a weight that is applied to the robot's angular velocity. This is functionally the same as setTurnSpeedMutliplier, but actually sets a different variable so that the multipliers can be combined. This is done just for ease of use for roadrunner users. Here the weight is being set to 0.7.

```java
drive.setAngularVelocityWeight(0.7);
```

These two functions will set weights that are applied to the x and y velocities of the drivetrain. These weights are functionally the same as VY_WEIGHT and VX_WEIGHT from the roadrunner quickstart, respectively. Note that *these x and y weights follow the HAL coordinate system, not the roadrunner coordinate system* and so VY_WEIGHT translates to the x weight, and VX_weight translates to the y weight. There is more information on why this is the case in the roadrunner section. Also, this function is only present in holonomic drivetrains for obvious reasons (non-holonomic drivetrains do not allow you to set an independent x and y velocity). Here both weights are set to 0.9.

```java
drive.setVelocityXWeight(0.9);
drive.setVelocityYWeight(0.9);
```

This function will set the "Lateral Multiplier" for holonomic drivetrains, and is used internally by some of the holonomic localizers. This is equivalent to the LATERAL_MULTIPLIER variable from the roadrunner quickstart. In this example, it is being set to 0.7.

```java
drive.setLateralMultiplier(0.7);
```

This function will set a weight on the drivetrain velocity in non-holonomic drivetrains. This is equivalent to `drive.setVelocityMultiplier()`, but sets a different multiplier. This exists mainly to make it easier for roadrunner users to use and navigate HAL's drivetrain functions.

```java
drive.setVelocityMultiplier(0.7);
```

These three functions will specify two buttons to be used for precise constant-speed turns during teleop, and the specific speed that those turns will be at. In this example, the left turn button is set to the left bumper, the right turn button is set to the right bumper, and the turn speed is set to 0.3.

```java
drive.setTurnLeftButton(new Button<>(1, Button.BooleanInputs.left_bumper));
drive.setTurnRightButton(new Button<>(1, Button.BooleanInputs.right_bumper));
drive.setTurnButtonPower(0.3);
```

This pair of functions will set a button used to control a "toggleable speed multiplier", and will set the speed multiplier itself, respectively. This feature works by toggling the application of a multiplier to the drivetrain's speed. When the toggle is active, the multiplier will be applied and when it is inactive, the multiplier will not be applied. This allows for the creation of "slow mode" buttons or "speed mode" buttons. There is also a toggleable multiplier for turn speed, which is also shown in the example. In the example below, the a button is used for the velocity toggle and the b button is used for the turn speed toggle. Both togglable multipliers are set to 0.5.

```java
drive.setSpeedToggleButton(new Button<>(1, Button.BooleanInputs.a));
drive.setToggleableVelocityMultiplier(0.5);

drive.setTurnSpeedToggleButton(new Button<>(1, Button.BooleanInputs.b));
drive.setToggleableTurnSpeedMultiplier(0.5);
```

#### Drivetrain Controls
This section is a bit boring, and only really contains two functions, but is good to know about.

This function will set which button is used as a velocity input for the drivetrain during teleop. By default, it is the right controller joystick. This will either be a Vector2D button if the drivetrain is holonomic, or a Double button if the drivetrain is non-holonomic. In this example, the drive stick is set to the left joystick.

```java
drive.setDriveStick(new Button<>(1, Button.VectorInputs.left_stick));
```

This function will set which button is used as an input for the drivetrain's turning speed during teleop. This will be a Double button for both holonomic and non-holonomic drivetrains. By default, it is the x coordinate of the left joystick. In this example, it is being set to the x coordinate of the right joystick.

```java
drive.setTurnStick(new Button<>(1, Button.DoubleInputs.right_stick_x));
```

### Which Options are Configurable?
With so many features, you may have been wondering which can be set using the HAL config system. What follows is a complete list of which features are available to be configured via the config system.:

* Drive Mode
* Drive Stick
* Turn Stick
* Turn Left Button
* Turn Right Button
* Velocity Toggle Button
* Turn Speed Toggle Button
* Velocity Scaling Method
* Turn Speed Scaling Method
* Velocity Multiplier
* Turn Speed Multiplier
* Velocity Cap
* Turn Speed Cap
* Toggleable Velocity Multiplier
* Toggleable Turn Speed Multiplier

## Localizers
Ok, now we move on to more types of localizers. We already went over the normal drivetrain encoder + IMU localizers earlier in the tutorial (the "declaring the drivetrain" section), so we will skip these and focus on the two remaining ones: TwoWheelLocalizer and ThreeWheelLocalizer.

Both of these localizers use "odometery pods" that consist of small, free-spinning omniwheels attached to the bottom of the robot and connected to highly sensitive encoders. As the wheels rotate, they give an accurate measurement of position. Tutorials on building odometery pods are all over the internet, and so will not be covered here (plus, i'm not exactly known for my hardware skills). The two wheel localizer uses two odometry pods, placed perpendicular to one another, and the imu to determine the robot's pose. The three wheel localizer, on the other hand, uses three odometry pods (one pair of parallel pods on opposite sides of the robot, and one perpendicular pod) and no imu whatsoever to determine the robot pose. Generally, these localizers are much more accurate the drivetrain-encoder-based localizers.

Ok! Lets look at some examples of these localizers getting used on a drivetrain, starting with TwoWheelLocalizer.

```java
drive.setLocalizer(new TwoWheelLocalizer(
         this,
         "imu",
         "enc_left",
         new Pose2d(-6,0, 0),
         "enc_x",
         new Pose2d(0,0,PI/2),
         new TrackingWheelConfig(1, 1, 1120)
));
```

Lets go over the constructor parameter by parameter. The first parameter is the robot itself, which is used to create the imu object and the encoder objects. The second parameter is the config name of the imu. If a BNO055IMU.Parameter object is added immediately after this parameter, the imu will be created using those parameters.

The third parameter is the config name for the wheel that would be parallel to the robot's direction of motion if it was moving forward. In other words, if the drivetrain was moving forward, then only this wheel would spin. The parameter immediately following this is the odometry pod's pose. Distance units for the pose are in inches, angle units are in radians, and the origin of the coordinate system is the center of the robot. So in this case, the pose is telling the localizer that the parallel wheel is 6 inches to the left of the robot's center.

The fifth parameter is the config name of the perpendicular odometry pod, and the sixth parameter is the pose of that odometry pod. In this case, the pod is at the center of the robot and is oriented left.

Here is a diagram of what the odometry pods should look like on the robot in this example (the diagram was taken from [here](https://github.com/NoahBres/road-runner-quickstart/blob/master/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/TwoWheelTrackingLocalizer.java) and modified for HAL):

![](images/two-wheel-localizer.png)

The parallel wheel is the vertical one, and the peripendicular wheel is the horizontal one.

The last parameter in the constructor is the TrackingWheelConfig. This gives some hardware information about the odometry pods that is used to accurately calculate position. The parameters in the TrackingWheelConfig are: the radius of the odometry wheels (units can be added, but if ommitted, like in the example, inches are assumed), the gear ratio between the encoder and odometry wheel (same definition as the DriveConfig gear ratio mentioned in the `declaring the drivetrain section`. **Show that to hardware**), and the number of encoder ticks per revolution of the encoder.

Like all other built-in localizers that use the imu, the localizer has a `remapIMUAxes()` function that can be used if the hub is mounted in a non-standard orientation. The localizer also has functions that allow you to reverse the direction of an encoder if needed. For example, if we needed to reverse the left encoder, we would do:

```java
drive.setLocalizer(new TwoWheelLocalizer(
         this,
         "imu",
         "enc_left",
         new Pose2d(-6,0, 0),
         "enc_x",
         new Pose2d(0,0,PI/2),
         new TrackingWheelConfig(1, 1, 1120)
).reverseEncoder("enc_left"));
```

or

```java
drive.setLocalizer(new TwoWheelLocalizer(
         this,
         "imu",
         "enc_left",
         new Pose2d(-6,0, 0),
         "enc_x",
         new Pose2d(0,0,PI/2),
         new TrackingWheelConfig(1, 1, 1120)
).setEncoderDirection("enc_left", Encoder.Direction.REVERSE));
```

Finally, the localizer allows you to set weights on the outputs from the parallel and perpendicular odometry pods, which can help increase the accuracy of the localization if correctly tuned. The first parameter in the setWeights function is the weight applied to the parallel wheel, and the second parameter is the weight applied to the perpendicular wheel.

```java
drive.setLocalizer(new TwoWheelLocalizer(
         this,
         "imu",
         "enc_left",
         new Pose2d(-6,0, 0),
         "enc_x",
         new Pose2d(0,0,PI/2),
         new TrackingWheelConfig(1, 1, 1120)
).reverseEncoder("enc_left").setWeights(0.9, 0.9));
```

The other localizer that uses odometry pods is ThreeWheelLocalizer. In contrast to the TwoWheelLocalizer, it *does not* use the IMU. It is also considered one of the best localization methods in FTC. Lets go through its constructor step by step like we did with TwoWheelLocalizer.

```java
drive.setLocalizer(new ThreeWheelLocalizer(
        this,
        12,
        0,
        "enc_left",
        "enc_right",
        "enc_x",
        new TrackingWheelConfig(1, 1, 1120)
));
```

This is the simplest constructor for ThreeWheelLocalizer. There is another constructor that follows the same pattern as TwoWheelLocalizer, where after each encoder config name, a Pose2d object is entered to describe its position on the robot, but because we have already gone over that pattern, we will skip going over it.

The first parameter in the constructor is the robot, which is used to create the encoder objects that measure position. The second parameter is the lateral distance between the parallel odometry pods, and the third parameter is how far offset in the y direction the perpendicular odometry pod is from the center of the robot. If units are not given for both values, it will assume they are in inches. Here is a diagram of how the odometry pods appear on the robot in this example (lateral distance of 12 inches, perpendicular wheel offset of 0 inches) (taken from the roadrunner quickstart and modified for HAL):

![](images/three-wheel-localizer.png)

The next three parameters are the config names of the left, right, and perpendicular encoders, respectively, and the final parameter is the TrackingWheelConfig object describing the odometry pods.

Like TwoWheelLocalizer, encoders can be easily reversed and both the parallel and perpendicular weights can be set.

```java
drive.setLocalizer(new ThreeWheelLocalizer(
        this,
        12,
        0,
        "enc_left",
        "enc_right",
        "enc_x",
        new TrackingWheelConfig(1, 1, 1120)
).reverseEncoder("enc_left").setWeights(0.9, 0.9));
```

### Types of Localizers: Roadrunner vs HAL
All HAL's built-in localizers report the drivetrain's pose using HAL's coordinate system, but many localizers instead report their position using roadrunner's coordinate system (See the `Roadrunner Integration` section for more information about how these systems differ). If you want to use a localizer that reports its position with the roadrunner coordinate system instead of the HAL coordinate system, simply specify that when you call `drive.setLocalizer()`. For example:

```java
drive.setLocalizer(new CustomLocalizer(
        //parameters here
), CoordinateMode.ROADRUNNER);
```

*This allows you to use localizers that were designed for roadrunner without having to change anything*. It is also useful if you simply prefer to work in roadrunner's coordinate system.

## Roadrunner Integration
All right, we've finished our tutorials with MecanumDriveSimple, and are ready to move on to the one, the only, MecanumDrive (now with roadrunner integration)!

The first thing different to note about MecanumDrive is its constructor:

```java
drive = new MecanumDrive(this,
        new RoadrunnerConfig(2, 1, 15, 1120, 133.9),
        "front_left_motor",
        "front_right_motor",
        "back_left_motor",
        "back_right_motor", false);
```

Instead of a DriveConfig object, MecanumDrive uses a RoadrunnerConfig object. Creating an instance of RoadrunnerConfig takes the exact same parameters as a DriveConfig instance (and in fact, RoadrunnerConfig instances can be used instead of DriveConfig instances in MecanumDriveSimple!), but RoadrunnerConfig also has a number of other parameters that should be set for optimal roadrunner performance. These parameters are the same as the ones in the roadrunner quickstart, so refer to that guide for more information.

Here is a list of the functions and what parameter each are used to set (+ a small description):
* setFollowerTimeout(time value, time unit) or setFollowerTimeout(time value in milliseconds) --> sets the trajectory follower timeout value
* setFollowerHeadingTolerance(angle, angle unit) or setFollowerHeadingTolerance(angle in radians) --> sets the trajectory follower heading tolerance
* setFollowerXTolerance(x tolerance, x unit (distance)) or setFollowerXTolerance(x tolerance in inches) --> sets the trajectory follower x tolerance
* setFollowerYTolerance(y tolerance, y unit (distance)) or setFollowerYTolerance(y tolerance in inches) --> sets the trajectory follower y tolerance
* setKv(kV) --> kV --> manually sets the motor kV value
* setKa(kA) --> kA --> manually sets the motor Ka value
* setKStatic --> kStatic --> manually sets the motor kStatic value
* setMaxAcceleration(max acceleration, acceleration unit (HALAccelerationUnit)) --> MAX_ACCEL (in/s^2 in roadrunner quickstart) --> sets the default max acceleration value for trajectories 
* setMaxAngularAcceleration(max angular acceleration, angular acceleration unit (HALAngularAccelerationUnit)) --> MAX_ANG_ACCEL (rad/s^2 in roadrunner quickstart) --> sets the default max angular acceleration for  trajectories
* setMaxVelocity(max velocity, velocity unit (HALVelocityUnit)) --> MAX_VEL (in/s in roadrunner quickstart) --> sets the default max velocity value for trajectories
* setMaxAngularVelocity(max angular velocity, angular velocity unit (HALAngularVelocityUnit)) --> MAX_ANG_VEL (rad/s in roadrunner quickstart) --> sets the default maximum angular velocity for trajectories
* setMotorVelocityPID(motor velocity pid coefficients (PIDFCoefficients)) --> MOTOR_VELO_PID --> motor velocity PID

Here is an example of one of the functions getting used:
```java
RoadrunnerConfig cfg = new RoadrunnerConfig(2,1,15,1120,133.9)
        .setMaxAcceleration(300, HALAccelerationUnit.MILES_PER_NANOSECOND_SQUARED);
```

Besides the roadrunner config change, there are only two other things that have to be set up for roadrunner the translational and heading PIDs. While both can be set in MecanumDriveSimple, they *HAVE* to be set in MecanumDrive in order for trajectory following to work correctly. They can be set using `drive.setHeadingPID()` and `drive.setTranslationalPID()`. If using a non-holonomic drivetrain, `drive.setAxialPID()` and `drive.setCrossTrackPID()` should be set instead of the translational PID.

Besides that small change, MecanumDrive's setup works exactly the same as MecanumDriveSimple's setup. In fact, all methods present in MecanumDriveSimple are also present in MecanumDrive. The only difference between MecanumDrive and MecanumDriveSimple is that MecanumDrive has several methods for interacting with roadrunner that MecanumDriveSimple does not.

The primary methods for interacting with roadrunner are the trajectoryBuilder and followTrajectory functions. They work almost exactly the same way as in roadrunner. There are, however, a small number of differences. First, units for distance and angles can be added to the end of trajectoryBuilder in order to specify which units the coordinates and headings will be in.

```java
HALTrajectory trajectory = robot.drive.trajectoryBuilder(new Pose2d(0, 0, 0), HALDistanceUnit.INCHES, HALAngleUnit.RADIANS)
        .splineTo(new Point2D(24, 24), 0)
        .build();
```

If units are ommitted, it will default to using inches and radians.

The second difference, and the *much more important one* is the coordinate system.

### Roadrunner and HAL Coordinate Systems
This is a *VERY* important section. HAL and roadrunner by default use different coordinate systems. In roadrunner, the *x axis* positive direction is *out the front of the robot*, and the *y axis* positive direction is *out the left side of the robot*. Because this is confusing for many people (including this developer), HAL changes this coordinate system so that the *x axis* positive direction is *out the right of the robot*, and the *y axis* positive direction is *out the front of the robot*. Basically, HAL coordinates are *rotated 90 degrees clockwise from roadrunner coordinates*. 

*By default, coordinates for trajectories are entered using HAL coordinates*. But this can be changed by doing `drive.setCoordinateMode(CoordinateMode.ROADRUNNER)` if you want to use the normal roadrunner coordinate system instead of the HAL coordinate system. It can also be changed mid-program if desired. Please note that the LOCALIZER coordinate mode, which is which mode the localizer returns coordinates in, is DIFFERENT than the drive coordinate mode, which affects how coordinates are entered in movement commands. *Both roadrunner and non-roadrunner classes have this function*.

### Ok! Back to Roadrunner
HAL Trajectory builders work the exact same way as roadrunner trajectory builders, and every command that can be done with a roadrunner trajectory builder can be done with a HAL trajectory builder as well.

Because of the coordinate system difference, calling `.build()` on a HALTrajectoryBuilder will create a HALTrajectory instead of a normal roadrunner trajectory. This HALTrajectory is just a wrapper class for trajectory, and has all the same functions as a normal trajectory, just with a differently-oriented coordinate grid.

Once the trajectory is built, it can be followed using either `drive.followTrajectory()` or `drive.followTrajectoryAsync()` (for asyncronous following).

In addition to these normal trajectory building/following methods, the drive class also has a bunch of methods that allow you to interact with and get data directly from the roadrunner interface. Here is a list of all of them, along with what they all do:
* getWheelPositions() --> Gets the wheel encoder positions
* getWheelVelocities() --> Gets the wheel velocities
* waitForRRInterfaceIdle() --> Updates the roadrunner interface repeatedly, making the program wait until the interface becomes idle (stops following a trajectory or turn profile)
* rRInterfaceIsBusy() --> Gets whether the roadrunner interface is busy (i.e. following a trajectory or a turn profile)
* updateRRInterface() --> Updates the roadrunner interface with the latest localizer positional data
* setPoseHistoryLimit(max number of saved poses) --> Sets a limit on the number of pose2d values that are kept in history during the trajectory. By default it is set to 100

And thats basically it! Yep, the roadrunner integration really isn't all that different from the simple mecanum drive. just remember to tune all the constants correctly and you should be good! There are even HAL versions of all roadrunner quickstart tuning programs [here](https://github.com/SCHS-Robotics/HAL_Simulator/tree/master/TeamCode/src/org/firstinspires/ftc/teamcode) (although *you should remove waitTime commands in some of them*, as they were used in the programs due to a problem with the simulator creating a velocity/position step function at low time intervals). 

Also please **NOTE:** *The FTC Dashboard is not and cannot be installed automatically, you will have to install it yourself.* This is due to the nature of the FTC dashboard, which requires modifying the FTCRobotControllerActivity file (which can't be done automatically without sketchy bytecode injection). If you want to install FTC Dashboard for use with the tuning programs, please refer to [this](https://acmerobotics.github.io/ftc-dashboard/gettingstarted.html) guide.

## Making a Custom Drivetrain
Ok this will be a relatively short section. If, for some reason, you want to make a custom drivetrain, there are 4 options that you have:

### Option 1: Extend an Existing Drivetrain
This is the easiest to do. All you have to do is extend an existing drivetrain, override the methods you want, and use the drivetrain like normal. If you wanted to add a teleop feature to MecanumDrive, for example, you could extend MecanumDrive, override its `handle()` function, add your own implementation of the teleop feature (maybe call super.handle() at some point as well to keep all the other teleop code from MecanumDrive), and PROFIT!

For most people, this is the extent of what you will need to do when creating a custom drivetrain, but for that small fraction of other people there is...

### Option 2: Extending HolonomicDrivetrain or NonHolonomicDrivetrain
This is for people who want to create an entirely new class of drivetrain, like a [ball drive](https://www.youtube.com/watch?v=j09PzmVmzJw&feature=emb_title) for example. Extending either of these abstract classes will automatically implement many of the features that are in all the other HAL drivetrain classes (velocity scaling, velocity weighting, etc). 

Also, it will ask you to implement two methods, turnPowerInternal and movePowerInternal, which set the power of the drivetrain motors to turn a certain direction or go a certain direction without modifying the motor powers. From these two functions, all of the drivetrain's functions like turnPower, moveSimple, etc. are automatically generated.

If you want to add roadrunner integration to this drivetrain, the easiest way is to create a roadrunner interface, which is BASCIALLY just an inner class that extends a roadrunner drivetrain class. The outer drivetrain class can then act a wrapper, calling functions in the inner class without having to expose it directly (and allowing for things like using the HAL coordinate system). (If you do choose this option, remember to override setLocalizer so that it sets your inner class localizer as well as your outer class localizer).

This is the second most difficult option, and is a lot of work, but there is one more option... 

### Option 3: Extending the Drivetrain class
This option is not for the faint of heart. Basically, this option will ask you to implement turnPowerInternal, and will auto-generate all the turn functions for you, but says NOTHING about how the robot moves. You can make your movement functions completely different from normal if you want. 

You will still have access to many of the base drivetrain functions like stopAllMotors(), however, so you won't be starting COMPLETELY from zero. But this method is NOT recommended.

### Option(?) 4: DIY
Ok this is the least recommended of the least recommended options. Basically, the essence of this option is: extend a subsystem and write a drivetrain from the ground up. I mean, you can do that, but WHY WOULD YOU??? THE REASON ABSTRACT CLASSES EXIST IS SO YOU DON'T HAVE TO... ahem, sorry about that. Anyway, yea this is not recommended as it's a lot of work, you won't get any sleep, and there are just better options.

# Congrats! You Finished Reading!!!
Wow, that took a while. Congratulations on reading until the end! You now know how to use HAL's built-in drivetrains! Our next tutorial will be on something a little shorter and more fun, [The HAL Vision System](index.md)!

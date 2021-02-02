---
layout: page
title: The HAL Config System
description: How to use the HAL9001 Config System
dropdown: Tutorials
priority: -6
---
----------------
## Its time for Hardware to do the work!
We now come to one of HAL's most powerful and well-known tools: the config system. HAL's config system allows you to change settings for your subsystems and programs __*during runtime*__, and then save these settings in a configuration file for quick deployment later. Not only does this allow drivers to switch their controls easily (if they want to try new controls, or if you have two drivers and their control preferences differ), but it also opens the door to allowing a huge set of subsystem behaviors to be modified on the fly. 

The speed of the intake, for example, could be adjusted via a configuration option so that it could be easily changed without reuploading the code. The autonomous program could have a configuration option for if the robot is playing on the near or far side of the field. The possibilities are almost endless. And the best part is, unlike FTC dashboard, this is 100% legal for competition use. 

So, with that introduction to get you excited about the power of HAL's config system, lets begin.

## Subsystem Configuration
Lets start with our subsystem from the last tutorial:

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig,
      Button<Boolean> intakeButton, Button<Boolean> outtakeButton) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        gamepad = new CustomizableGamepad(robot);
        gamepad.addButton(INTAKE, intakeButton);
        gamepad.addButton(OUTTAKE, outtakeButton);
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(1);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(1);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
}
```

Now, lets say instead of having to pass our intake and outtake buttons in to the constructor, we want to be able to set them via the config system. The first thing we have to do is define a special function somewhere in the subsystem. This function has a few very specific requirements: it has to be public, static, return an array of ConfigParam objects (what those are will be explained later), take no parameters, and be annotated with an `@TeleopConfig` annotation. We will call this function teleopConfig, although its name doesn't matter to the HAL internals that will be searching for it.

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig,
      Button<Boolean> intakeButton, Button<Boolean> outtakeButton) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        gamepad = new CustomizableGamepad(robot);
        gamepad.addButton(INTAKE, intakeButton);
        gamepad.addButton(OUTTAKE, outtakeButton);
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(1);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(1);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return null;
    }
}
```

Notice how the function we defined met all the criteria we laid out above? Next, we want to change the function to return the actual configurable entries that the config system will use. This is why we return an array of ConfigParams. Each ConfigParam is essentially a list of values that the thing being configured can possibly take on. Each ConfigParam contains a default value, a list of possible values, and a String id, however, because some types of objects are configured extremely frequently (like buttons) there exist shortcuts to defining those types of ConfigParam objects.

To create a ConfigParam for a button, simply pass an id and the button's default input type into the ConfigParam constructor, as shown below:

```java
new ConfigParam("ID", Button.BooleanInputs.b)
```

If you want to set the button's gamepad to a default value, pass that in as an optional third parameter (here we will use gamepad 2 as default. If this parameter is not passed, it defaults to gamepad 1)

```java
new ConfigParam("ID", Button.BooleanInputs.b, 2)
```

This will auto-fill all possible boolean button types as options for that ConfigParam.

To create a ConfigParam for a set of enum values, simply pass in the id and the enum value to default to:

```java
public enum Example {
    A, B, C, D
}

new ConfigParam("ID", Example.A);
```

This will auto-fill in all possible values for that enum class as options for that ConfigParam. If you want to exclude some specific enum values as options, pass in an array of valid enums as the second parameter in the constructor:

```java
public enum Example {
    A, B, C, D
}

new ConfigParam("ID", new Example[] {Example.A, Example.B, Example.C}, Example.A);
```

To create a ConfigParam for a String variable, pass the ConfigParam an id, a list or array of String options, and the default String value:

```java
new ConfigParam("ID", new String[] {"dog", "cat", "bat", "hat"}, "cat")
```

Finally, for all other ConfigParams, you would use the general constructor. This constructor takes an id, a hashmap mapping String names to Java objects, and a default object option. The hashmap is used to associate the actual object values being configured with the Strings that will display on screen during configuration. One example would be mapping the String "2.2" to the number 2.2. Because the value entry of the hashmap is defined to be an Object, the parent class of all other Java classes, this means that literally ANY class can be used as a configuration value.

{% raw %}
```java
new ConfigParam("Robot Positive Direction", new HashMap<String, Object>() {{
        put("Front", Axis2D.POSITIVE_Y_AXIS);
        put("Back", Axis2D.NEGATIVE_Y_AXIS);
        put("Left", Axis2D.NEGATIVE_X_AXIS);
        put("Right", Axis2D.POSITIVE_X_AXIS);
    }}, Axis2D.POSITIVE_X_AXIS);
```
{% endraw %}

The double \{ is used in this example to add entries to the hashmap on a single line. This DOES work, trust me, its just a bit obscure. Feel free to add the entries to your hashmap any way you want.

For number configuration and boolean configuration, you would also use the general constructor, but there are helpful methods that you can use to generate the String, Object map automatically. 

The `ConfigParam.numberMap()` method, for example, uses a starting number, an ending number, and an increment number in order to generate a list of valid numeric options that can be cycled through using the config system. These parameters will either be all double values or all integer values.

```java
new ConfigParam("ID", ConfigParam.numberMap(1, 10, 1), 1)
```

One important thing to note about the `ConfigParam.numberMap()` method is that the ending number will ALWAYS be included in the final hashmap, even if the increment would normally skip over it.

For boolean configuration, the Hashmap of boolean values to strings has been pre-created, and can be accessed via a constant in ConfigParam:

```java
new ConfigParam("ID", ConfigParam.BOOLEAN_MAP, false);
```

Ok now lets get back to our goal. We want to configure our intake and outtake buttons, so we will use the shortcut button constructors in order to create our two ConfigParams.

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig,
      Button<Boolean> intakeButton, Button<Boolean> outtakeButton) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        gamepad = new CustomizableGamepad(robot);
        gamepad.addButton(INTAKE, intakeButton);
        gamepad.addButton(OUTTAKE, outtakeButton);
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(1);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(1);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return new ConfigParam[] {
            new ConfigParam(INTAKE, BooleanInputs.a),
            new ConfigParam(OUTTAKE, BooleanInputs.b)
        };
    }
}
```

Notice how we are using the button names that we used in the customizable gamepad to access the buttons? That is because once the buttons have been configured, they will be automatically added to a customizable gamepad under their configuration id.

Now that we have our config for this subsystem, we need to tell the subsystem to actually USE the config. We do this by setting a special variable (usesConfig) in the subsystem's constructor to true. While we do this, lets also remove everything in the constructor involving the customizable gamepad, as that will be set up later.

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        usesConfig = true;
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(1);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(1);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return new ConfigParam[] {
            new ConfigParam(INTAKE, BooleanInputs.a),
            new ConfigParam(OUTTAKE, BooleanInputs.b)
        };
    }
}
```

Now this subsystem is configurable and using config!! Well... almost. There is one more step: actually pulling the controls out of the config. In order to this, we'll call the 
`robot.pullControls()` function in `start()`, as the config system finishes running after initialization is complete.

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        usesConfig = true;
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        gamepad = robot.pullControls(this);
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(1);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(1);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return new ConfigParam[] {
            new ConfigParam(INTAKE, BooleanInputs.a),
            new ConfigParam(OUTTAKE, BooleanInputs.b)
        };
    }
}
```

And that's all there is to it! The `robot.pullControls()` function takes a subsystem as input (in this case, this specific subsystem) and attempts to create a customizable gamepad out of all the configured controls in that subsystem. For non-gamepad config entries, it gets a bit more complicated. Lets demonstrate this by adding a way to configure the intake's power during both teleop AND autonomous.

The `teleopConfig()` function we created only allows us to configure options for teleop. In order to configure options for autonomous as well, we will need to create an autonomous config function. The requirements for this function are the exact same as the teleop config function, except it should be annotated with `@AutonomousConfig` instead of `@TeleopConfig`.

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        usesConfig = true;
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        gamepad = robot.pullControls(this);
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(1);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(1);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return new ConfigParam[] {
            new ConfigParam(INTAKE, BooleanInputs.a),
            new ConfigParam(OUTTAKE, BooleanInputs.b)
        };
    }
    
    @AutonomousConfig
    public static ConfigParam[] autonomousConfig() {
        return null;
    }
}
```

Now, lets add a ConfigParam to both the teleop config function and the autonomous config function. We will use the `ConfigParam.numberMap()` function to generate the hashmap for this ConfigParam. The options should be between 0 and 1 (as we don't want to specify motor direction), and we will set the increment to 0.1 (so the options will be 0.0, 0.1, 0.2, ..., 0.9, 1.0). We'll set it to a default of 1.

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        usesConfig = true;
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        gamepad = robot.pullControls(this);
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(1);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(1);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return new ConfigParam[] {
            new ConfigParam(INTAKE, BooleanInputs.a),
            new ConfigParam(OUTTAKE, BooleanInputs.b),
            new ConfigParam("Intake Power", ConfigParam.numberMap(0, 1, 0.1), 1);
        };
    }
    
    @AutonomousConfig
    public static ConfigParam[] autonomousConfig() {
        return new ConfigParam[] {
            new ConfigParam("Intake Power", ConfigParam.numberMap(0, 1, 0.1), 1);
        };
    }
}
```

Now, in preparation for getting and actually using this configured value, we'll add a double variable at the top of the program to store the intake power and replace the power we pass to the intake and outtake functions in `handle()` with that variable.

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    private double intakePower;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        usesConfig = true;
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        gamepad = robot.pullControls(this);
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(intakePower);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(intakePower);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return new ConfigParam[] {
            new ConfigParam(INTAKE, BooleanInputs.a),
            new ConfigParam(OUTTAKE, BooleanInputs.b),
            new ConfigParam("Intake Power", ConfigParam.numberMap(0, 1, 0.1), 1);
        };
    }
    
    @AutonomousConfig
    public static ConfigParam[] autonomousConfig() {
        return new ConfigParam[] {
            new ConfigParam("Intake Power", ConfigParam.numberMap(0, 1, 0.1), 1);
        };
    }
}
```

Now we are ready to pull the results of the configuration into our intakePower variable. We do this using the `robot.pullNonGamepad()` function, which returns a ConfigData object. This object maps the ids of each non-gamepad button ConfigParam object to the value that was selected via the config system. Essentially, its a hashmap with a cool skin. In order to get data for a specific ConfigParam, you would use the object's `.getData()` function, as shown below:

```java
public class Intake extends SubSystem {

    private static final String INTAKE = "intake button", OUTTAKE = "outtake button";

    private DcMotor leftIntake, rightIntake;
    private CustomizableGamepad gamepad;
    private double intakePower;
    
    public Intake(Robot robot, String leftIntakeConfig, String rightIntakeConfig) {
        super(robot);
        leftIntake = robot.hardwareMap.dcMotor.get(leftIntakeConfig);
        rightIntake = robot.hardwareMap.dcMotor.get(rightIntakeConfig);
        
        //So both intake motors spin inwards when power is positive.
        leftIntake.setDirection(DcMotor.Direction.REVERSE);
        
        usesConfig = true;
    }

    @Override
    public void init() {

    }

    @Override
    public void init_loop() {

    }

    @Override
    public void start() {
        gamepad = robot.pullControls(this);
        ConfigData data = robot.pullNonGamepad(this);
        intakePower = data.getData("Intake Power", Double.class);
    }

    @Override
    public void handle(){
        if(gamepad.getInput(INTAKE)) {
            intake(intakePower);
        }
        else if(gamepad.getInput(OUTTAKE)) {
            outtake(intakePower);
        }
        else {
            stopIntake();
        }
    }

    @Override
    public void stop() {

    }
    
    public void intake(double speed) {
        leftIntake.setPower(speed);
        rightIntake.setPower(speed);
    }
    
    public void outtake(double speed) {
        intake(-speed);
    }
    
    public void stopIntake() {
         intake(0);
    }
    
    @TeleopConfig
    public static ConfigParam[] teleopConfig() {
        return new ConfigParam[] {
            new ConfigParam(INTAKE, BooleanInputs.a),
            new ConfigParam(OUTTAKE, BooleanInputs.b),
            new ConfigParam("Intake Power", ConfigParam.numberMap(0, 1, 0.1), 1);
        };
    }
    
    @AutonomousConfig
    public static ConfigParam[] autonomousConfig() {
        return new ConfigParam[] {
            new ConfigParam("Intake Power", ConfigParam.numberMap(0, 1, 0.1), 1);
        };
    }
}
```

A few important things to note here. First, like the `robot.pullControls()` function, `pullNonGamepad()` takes a subsystem as input. Specifically, this subsystem. Second, the `.getData()` function's 2nd parameter tells it what data type to return. If the data type is incorrect, the program will crash, so be careful. Finally, note that if the intake power ConfigParam had not been present in both teleop and autonomous, the program would have crashed in teleop (as there wouldn't have been an entry in the config data with the id "Intake Power". If you want a setting to only be configurable in one mode, you can use the `robot.isAutonomous()` and `robot.isTeleop()` functions in order to fix this problem. For example:

```java
if(robot.isAutonomous) {
    intakePower = data.getData("Intake Power", Double.class);
}
else {
    intakePower = 1;
}
```

Congratulations! You have just learned how to configure subsystems in HAL! But you aren't done yet, there is still the matter of configuring __*Programs*__.

## Configuring Programs

Now that we have our subsystem configured, lets go over to our teleop and autonomous programs (refer to the [HAL Autonomous](hal-autonomous.md) and [HAL Teleop](hal-teleop.md) tutorials for details on how to make these programs).

```java
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;

    @Override
    public void main() {
        
    }
}
```

```java
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {
    public @MainRobot MyRobot robot;
}
```

Lets say we want to be able to configure both of these in order to select whether we are playing on red alliance or blue alliance. In order to do that, we would use the `@ProgramOptions` annotation. This annotation takes an array of enum classes as input. When it is used, the program appears in the config menu as a configurable subsystem with the name given in the `@Teleop` or `@Autonomous` annotations. Each enum class is added as a ConfigParam, with the name of the enum class as the ConfigParam id, the first enum value as the default value, and all enum values as possible options. If we want to select between the red alliance and blue alliance using `@ProgramOptions`, all we would have to do is create an enum and register it with the config using `@ProgramOptions`, as shown below (in both programs):

```java
@ProgramOptions(options = MyHALAutonomous.Alliance.class)
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    enum Alliance {
        RED, BLUE
    }

    public @MainRobot MyRobot robot;

    @Override
    public void main() {
        
    }
}
```

```java
@ProgramOptions(options = MyHALTeleop.Alliance.class)
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {
    enum Alliance {
        RED, BLUE
    }
    
    public @MainRobot MyRobot robot;
}
```

One nice idea would be to create a separate enum class, thereby keeping everything a bit more organized.

```java
public enum Alliance {
    RED, BLUE
}
```

```java
@ProgramOptions(options = Alliance.class)
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;

    @Override
    public void main() {
        
    }
}
```

```java
@ProgramOptions(options = Alliance.class)
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {
    public @MainRobot MyRobot robot;
}
```

And that's it! In order to access whatever value was selected in the config system, you would use the function `robot.pullOpModeSettings()` to get a ConfigData object containing the related data.

```java
public enum Alliance {
    RED, BLUE
}
```

```java
@ProgramOptions(options = Alliance.class)
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;
    public Alliance alliance;

    @Override
    public void main() {
        ConfigData data = robot.pullOpModeSettings();
        alliance = data.getData("Alliance", Alliance.class);
    }
}
```

```java
@ProgramOptions(options = Alliance.class)
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {
    public @MainRobot MyRobot robot;
    public Alliance alliance;
    
    @Override
    protected void onStart() {
        ConfigData data = robot.pullOpModeSettings();
        alliance = data.getData("Alliance", Alliance.class);
    }
}
```

Note that `robot.pullOpModeSettings()` has no parameters, and that the id of the entry coorsponding with the enum class provided in `@ProgramOptions` is the name of the enum class.

## Inter-Program Interactions
Now that we know how to set up the configuration system for subsystems and programs, lets talk about how the config system really works. The config system has two main modes: autonomous mode and teleop mode. In autonomous mode, you configure all the options for autonomous and in teleop mode you configure all of the options for teleop. By default, when you run an autonomous program, you will be asked to configure all the autonomous program options, and then to configure all the teleop program options. 

The next time a teleop program is run using the same robot class, it will automatically use all the config options you selected for teleop during your autonomous program (this is because it takes a few seconds to select a config file on the phone, which is a problem when there isn't much time between teleop and autonomous for the drivers to use the controllers). 

The teleop program will then continue to use those config options until either new options are set via running an autonomous program with that same robot again and selecting a different config file, or a button on the controller (the a button) is pressed to specifically select a new config file.

While this works for competitions, in some cases (like when a program is being tested, is under construction, or is meant for debugging), this behavior is undesirable as it takes more time to select 2 config files than one. To account for these cases, an annotation exists called `@StandAlone`. When a program is annotated with `@StandAlone`, it functions as a standalone program, so you only have to select config for that specific program. If we wanted to make our Autonomous program a standalone program, for example, we would just annotate it with `@StandAlone`:

```java
@StandAlone
@ProgramOptions(options = Alliance.class)
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;
    public Alliance alliance;

    @Override
    public void main() {
        ConfigData data = robot.pullOpModeSettings();
        alliance = data.getData("Alliance", Alliance.class);
    }
}
```

A standalone autonomous program only requires you to select a config file for that autonomous program, and a standalone teleop program will never auto-run any config files.

Unlike subsystem configuration settings, however, program configuration settings can't be pre-set by default. The reason for this is because the autonomous program doesn't know which specific teleop program you are going to run immediately afterwards, and so can't extract the relevent settings. In order to link one program to another, so that it knows which program will follow it, you would use the `@LinkTo` annotation.

```java
@LinkTo(destination = "MyHalTeleop")
@ProgramOptions(options = Alliance.class)
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;
    public Alliance alliance;

    @Override
    public void main() {
        ConfigData data = robot.pullOpModeSettings();
        alliance = data.getData("Alliance", Alliance.class);
    }
}
```

```java
@ProgramOptions(options = Alliance.class)
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {
    public @MainRobot MyRobot robot;
    public Alliance alliance;
    
    @Override
    protected void onStart() {
        ConfigData data = robot.pullOpModeSettings();
        alliance = data.getData("Alliance", Alliance.class);
    }
}
```

The `@LinkTo` annotation does 2 things. First, it tells your autonomous program which program will run after it, and lets you access and pre-set that program's configuration settings. It does this using the "destination" input, which is the name (as specified in `@Autonomous` or `@Teleop`) of the next program that will be run. Second, it automatically transitions to the destination program after the program it annotates has finished running. 

In our above example, as soon as autonomous is complete, teleop would start without any user input whatsoever. This works regardless of if the config system is being used whatsoever (it also works even if one or both programs are `@StandAlone programs`), and is not limited to just 1 transition. If this functionality is not desirable, you can turn it off by setting an optional second input ("auto_transition") to false.

```java
@LinkTo(destination = "MyHalTeleop", auto_transition = false)
```

## Ok, this is all great, but you keep saying stuff like "select the configuration settings", how does that actually work?
I'm glad you asked! As long as one subsystem or program is using the config system, it will automatically be enabled. As soon as the "init" button is pressed on the robot controller. A "menu" will pop up in the telemetry that looks a bit like this:

```
#|New Config
#|Edit Config
#|Delete Config
```

If you have any config files already made, perhaps by someone else on your team, they will also appear under the "delete config" option. This menu will also have a blinking cursor. This cursor can be moved around using the d-pad on the controller (assuming the controller is set as gamepad 1). The a button on gamepad 1 is the select button, and will select whichever option the cursor is currently blinking on. If there are no config files already made, selecting "edit config" or "delete config" will have no effect, as there are no config files to edit or delete. If there ARE config files present, selecting one of them will load its data into the program and cause you to exit out of the config menu.

If you press "new config", you will be taken to a menu screen that looks like this:

```
_
#|abc #|def #|ghi                 
#|jkl #|mno #|pqr                 
#|stu #|vwx #|yz0                 
#|123 #|456 #|789                 
#|!?@ #|$%^ #|&*                  
#|<--       -->|#                 
#|Done                            
```

This is a text entry menu, and will allow you to enter a name for your config file. The _ represents the current position within the name that you are typing, while your cursor can move around and select different groups of letters. Each time a group of letters is clicked on, it will insert the next letter in the group into the currently highlighted space on top (If there are no letters in the group for it to reference, then it will just use the first one). The arrows at the bottom move the space on top left and right. The x and b buttons also will move the space left and right. Finally, the left bumper will take you back to the previous menu and the right bumper will take you back to the menu you were just looking at, like the forward and back buttons in google chrome (this is true for all menus in the config system). 

Once a name for the config file has been typed, the user should select the "done" button at the bottom of the screen. This sets the config file name and brings you to the next step where you actually configure your subsystems. The subsystem configuration menu for our example subsystem and programs (assuming we are running autonomous) should look like this:

```
#|Intake
#|MyHalAuto
#|Done
```

Each subsystem/program configurable for the current mode of the config system will appear here (notice that MyHalTeleop is not present, as it is a teleop program and not an autonomous program) and will be loaded with the default options for all of its config settings. If we select one of these subsystems/programs, we will bring up a screen with all of its configurable parameters. If we select "Intake," for example:

```
#|Intake Power|1.0
#|Done
```

Pressing the a button while Intake Power is selected would make it cycle to its next available option (in this case zero). If the b button is pressed while Intake Power is selected, it will go to the previous available option (in this case 0.9). Once the "done" button is pressed, the modified settings will be cached and you will be taken back to this screen:

```
#|Intake
#|MyHalAuto
#|Done
```

Once all settings are configured to their desired values, you can select the "done" button at the bottom of the screen, which will save the config data to a file and send you back to the "select a config file" screen. However, because you just made a config file, it will look something like this.

```
#|New Config
#|Edit Config
#|Delete Config
#|newconfigfile
```

If edit config is selected, you will be able to select a config file to edit. Selecting a config file will send you back to the screen with all configurable subsystems/programs, and allow you to make changes to the file's saved data. Delete config will delete a selected config file. Selecting a config file will load that config file's data into the program and will make it accessible via `robot.pullControls()`, `robot.pullNonGamepad()`, and `robot.pullOpModeSettings()`. 

If the program is in standalone mode, then it will exit and wait for you to start the program. If it isn't in standalone mode, and you are running an autonomous program (as we are in this example), it will then prompt us to select a configuration file to use for teleop, sending us back to this screen, but with a completely different set of config files:

```
#|New Config
#|Edit Config
#|Delete Config
```

The process for creating a new config file is exactly the same as in autonomous mode. You select "new config," give your config file a name, and modify the configuration settings of teleop-configurable subsystems/programs.

```
#|Intake
#|MyHalTeleop
#|Done
```

```
#|intake button|a|Gamepad 1
#|outtake button|b|Gamepad 1
#|Intake Power|1.0
#|Done
```

There are some important things to note here involving our example. First, MyHalTeleop appeared as configurable instead of MyHalAutonomous because:

a. @LinkTo was present

b. The config system is now in teleop mode

Second, the intake and outtake buttons have a third setting that allows the gamepad to be configured. If the y button is pressed while a button setting is selected, the gamepad option cycles between gamepad 1 and gamepad 2.

Once a configuration file has been selected for teleop mode, the screen will go blank and the program will wait for you to press start. Once you press start, the program will run and use the settings that were contained in your selected configuration file. If run is pressed before a file is selected, the default configuratin settings will be used. 

After our autonomous program is run, it will transition to our teleop program automatically due to the @LinkTo annotation. If it doesn't automatically transition (i.e., if you don't have @LinkTo or disabled its auto-transition ability), then when you run the teleop program meant to follow your autonomous program, a screen will display on telemetry during init that looks like this:

```
Autorunning myteleopconfigfile, press the select button to select a different file to run.
```

As the text suggests, pressing the select button (the a button) will allow you to select a different config file to run instead of the one being run automatically.

## What if I have multiple of the same subsystem? Won't they have the same name in the config?
Well yes, and no. If you have two instances of a subsystem in your robot class, like in this example robot class:

```java
public class ExampleRobot extends Robot {
    public Grabber leftGrabber, rightGrabber;
    public MyRobot(OpMode opMode) {
        super(opMode);
        leftGrabber = new Grabber(this, "left servo config");
        rightGrabber = new Grabber(this, "right servo config");
    }
}
```

then the second one created will have a 1 at the end of its name, and will appear in the config like this:

```
#|Grabber
#|Grabber1
#|Done
```

Obviously, this is not ideal, so there is a way to manually specify what a subsystem is called in the config. The `@ConfigLabel` annotation allows you to manually give a subsystem a label that will be displayed in the config instead of its automatically-chosen label. In the previous example, it would be used like this:

```java
public class ExampleRobot extends Robot {
    public @ConfigLabel(label = "Left Grabber") Grabber leftGrabber
    public @ConfigLabel(label = "Right Grabber") Grabber rightGrabber;
    public MyRobot(OpMode opMode) {
        super(opMode);
        leftGrabber = new Grabber(this, "left servo config");
        rightGrabber = new Grabber(this, "right servo config");
    }
}
```

In the config, each subsystem would then be displayed like this:

```
#|Left Grabber
#|Right Grabber
#|Done
```

# Next Tutorial
Congratulations! You now know how to use HAL's config system! Have fun making all programs configurable! the next turorial will discuss how to use HAL's [built-in-drivetrains](built-in-drivetrains.html).

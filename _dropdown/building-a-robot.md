---
layout: page
title: Building a Robot
description: Test page
dropdown: Tutorials
priority: -2
---
----------------------
The next step in creating a HAL program is to build a robot class. We start this the same way as we did the subsystem, by creating a new java class in the teamcode folder. In this example, I'll call it MyRobot.java.

```java
public class MyRobot {

}
```

We now need to tell HAL that MyRobot is a type of Robot. Just like when we made our subsystem, we will use the "extends" keyword to accomplish this, making MyRobot extend Robot.

```java
public class MyRobot extends Robot {

}
```
Unlike our subsystem, however, MyRobot does not need to override a huge number of methods. All it needs is a constructor.
```java
public class MyRobot extends Robot {
    public MyRobot(OpMode opMode) {
        super(opMode);
    }
}
```

Notice that the robot's constructor takes an OpMode as input. This opmode is the program the robot will run. Don't worry too much about how we actually GET the opmode, that part will come later. The `super(opMode)` line then takes the OpMode and passes it to the overall Robot class, where it does magic things with it behind the scenes. `super(opMode)` should always be in your robot's constructor.

Now that we have a robot class, we need to fill it with subsystems. Think of the robot as a code version of your physical robot. You need to fill it with all the subsystems it will use in the competition: A drivetrain, an intake, ect.

So, lets add our subsystem to the robot.

First, you define the subsystem as a public field right above the robot's constructor, like this:

```java
public class MyRobot extends Robot {
    public MySubSystem mySubSystem;
    public MyRobot(OpMode opMode) {
        super(opMode);
    }
}
```

Note that we haven't actually set the subsystem equal to anything yet, we are just telling the robot that it exists. Now, in the constructor, we have to actually create the subsystem. Also note that the **field is declared as public**. This is so the internal robot class can access the subsystem to register it. Making the subsystem private or protected will cause an error to occur.

```java
public class MyRobot extends Robot {
    public MySubSystem mySubSystem;
    public MyRobot(OpMode opMode) {
        super(opMode);
        mySubSystem = new MySubSystem(this);
    }
}
```

Here we create a new instance of MySubSystem. This is what the "new" keyword does, it is telling the computer to create a new version of that subsystem. MySubSystem's constructor takes only one parameter, the robot using the subsystem (go look back at the code for MySubSystem to confirm this!). Because **THIS** robot is using the subsystem, not any random robot, we give MySubSystem the "this" keyword to tell it that MyRobot is the robot using it.

The subsystem will then be automatically registered with the robot's internal wizardry. If you want to disable the subsystem (possibly for debugging purposes), you can add `@DisableSubSystem` to make HAL skip over it.

```java
public class MyRobot extends Robot {
    //This subsystem is now disabled    
    public @DisableSubSystem MySubSystem mySubSystem;
    public MyRobot(OpMode opMode) {
        super(opMode);
        mySubSystem = new MySubSystem(this);
    }
}
```

Congratulations! You have made a robot with one subsystem! To add more subsystems, simply repeat the process of defining a subsystem and then creating it with the "new" keyword. You are now ready to create your first HAL programs! Lets start with a [HAL Teleop](hal-teleop.md) Program.

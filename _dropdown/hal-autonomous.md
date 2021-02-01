---
layout: page
title: Your First HAL Autonomous Program
description: Creating your first HAL9001 autonomous program
dropdown: Tutorials
priority: 0
---
----------------------
The process for making a HAL Autonomous program is very similar to that of a HAL Teleop program (If you haven't read that tutorial yet, you probably should now), but there are a few major differences.

First, the program extends BaseAutonomous instead of BaseTeleop, as it is an autonomous program.

```java
public class MyHALAutonomous extends BaseAutonomous {

}
```

Second, in addition to defining the robot (via @MainRobot or overriding buildRobot()), HAL autonomous programs also require you to override a method called main().

```java
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;
 
    @Override
    public void main() {
        
    }
}
```

Autonomous programs run each subsystem's init(), init_loop(), start(), and stop() functions automatically, but runs whatever code happens to be in main() after you press start instead of handle(). Because of this, autonomous programs do not have optional methods for onUpdate() and onStart(), as both of those methods involve code that could just run during main().

Finally, because it is an autonomous program, it uses `@Autonomous` instead of `@Teleop` as an annotation.

```java
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;

    @Override
    public void main() {
        
    }
}
```

When programming an autonomous program, you use the robot to access your subsystem's functions directly. Here is an example using the robot and subsystem we created in previous tutorials.

```java
@Autonomous(name = "MyHalAuto", group = "Example Programs")
public class MyHALAutonomous extends BaseAutonomous {
    public @MainRobot MyRobot robot;

    @Override
    public void main() {
        while(opModeIsActive()) {
            /*
            This is why we encouraged you to design programs with functions in mind 
            rather than just using the built in automatic-run functions (or at least its one of the reasons).
            */
            robot.mySubSystem.doThisCoolThing(9001); 
        }
    }
}
```

Notice that we are **using the function we created in our subsystem to tell that subsystem WHEN to do that cool thing.** This is part of the reason why it is important to follow the subsystem design flow, creating functions that tell it how to do something and then using those functions to tell it when to do something. If you have a function to move forward as part of a drivetrain, all you would have to do is call robot.drivetrain.forward() instead of messing around with motors. The subsystem already knows how to do what you want! This lets people code at a very high level, as they don't need to know how a subsystem works internally in order to use it in a program, they just need to know what it is supposed to do.

You have now written both an autonomous program in HAL, and have finished the first 4 basic HAL tutorials, congratulations!!!! If you want to keep going, the next tutorial you should look at is [A Customizable Gamepad and Other Useful Things](custom-gamepad.md).

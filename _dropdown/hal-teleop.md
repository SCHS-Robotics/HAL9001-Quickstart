---
layout: page
title: Your First HAL Teleop Program
description: Creating your first HAL9001 Teleop Program
dropdown: Tutorials
priority: -3
---
-----------------
Now, if you have been reading the tutorial for this long, you might predict that the first step in creating a teleop program is to make a new java class in the teamcode folder... and you would be right! I will be calling this class MyHALTeleop.java, because I am definitely **_VERY_** creative with my naming conventions.

```java
public class MyHALTeleop {

}
```

And just like the previous two tutorials, you need to make it extend something. In this case, because it is a type of teleop program, it should extend BaseTeleop.

```java
public class MyHALTeleop extends BaseTeleop {
    
}
```

Now the next step is to override a method called buildRobot(). This method sets up your robot at the beginning of the program so all the subsystems can be run behind the scenes. Instead of typing this out, you can right click where you want it, click generate, then click "Override Methods". There should be one called buildRobot than you can double click on to auto-create it. Your program will then look like this:

```java
public class MyHALTeleop extends BaseTeleop {
    @Override
    public Robot buildRobot() {
        
    }
}
```

Now, obviously, we want buildRobot() to actually build our robot, so we start by defining a Robot variable above buildRobot(). Because we are going to use MyRobot (the robot we created previously) in this program, it will be a variable of type MyRobot.

```java
public class MyHALTeleop extends BaseTeleop {

    private MyRobot robot;
    
    @Override
    public Robot buildRobot() {
        
    }
}
```

It doesn't have to be a private variable, I just decided to do that for fun. Next, we want to create our robot and give it back to HAL to run in the program. We do this in a similar way to creating subsystems: using the "new" keyword.

```java
public class MyHALTeleop extends BaseTeleop {

    private MyRobot robot;
    
    @Override
    public Robot buildRobot() {
        robot = new MyRobot(this);
        return robot;
    }
}
```

MyRobot takes "this" as input because the program the robot will run is THIS program that the robot is currently in. We then return the robot from buildRobot(), giving it to the internal HAL code to get it to run all our programs.

There is an alternative way, however, to build our robot for this program. Instead of overriding BuildRobot() and returning your robot, you can make a public variable for your robot and annotate it with `@MainRobot`, as shown below:

```java
public class MyHALTeleop extends BaseTeleop {
    public @MainRobot MyRobot robot;
}
```

This will automatically build the robot for you. Even though it looks like you never set robot to any value, it is being set to your auto-built robot behind the scenes. This method shortens the amount of code you have to write to build the robot, but provides less control over the robot creation process. Note that in this case robot HAS to be a public variable in order for the internal code to reach it. If you try to do both methods by overriding buildRobot and using the `@MainRobot` annotation, HAL will prioritize the buildRobot function.

Finally, we have to add the finishing touch to our program, an `@Teleop` annotation. This tells the FTC app to put this program on the phone. It takes two parameters, a name and a group. **The name you put in the annotation is the name that shows up on the phone, not the class name!** If your program is missing, chances are you forgot to put the `@Teleop` annotation on top of the class. The group parameter is optional, and is used only for organizing your opmodes. You don't have to put it. Our program then becomes:

```java
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {

    private MyRobot robot;
    
    @Override
    public Robot buildRobot() {
        robot = new MyRobot(this);
        return robot;
    }
}
```

or

```java
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {
    public @MainRobot MyRobot robot;
}
```

At this point, your teleop program is done. Forever. That's right, completely done. The init(), init_loop(), start(), handle(), and stop() methods in every subsystem will be run automatically, and so whenever you add another subsystem to your robot it will reflect that in your teleop program.

### Optional Methods
If you want to add special functionality, however, you can optionally override some extra methods that get run automatically at different points in the program in addition to the subsystem program:

`onInit()` runs whatever code it contains once when you press the init button.

`onInitLoop()` runs whatever code it contains over and over after you press the init button.

`onStart()` runs once when you press the start button.

`onUpdate()` runs over and over after you press the start button.

`onStop()` runs on stop.

The full program, with all optional methods overridden, looks like this:

```java
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {

    private MyRobot robot;
    
    @Override
    public Robot buildRobot() {
        robot = new MyRobot(this);
        return robot;
    }

    @Override
    protected void onInit() {

    }

    @Override
    protected void onInitLoop() {

    }

    @Override
    protected void onStart() {
        
    }

    @Override
    protected void onUpdate() {
       
    }

    @Override
    protected void onStop() {

    }
}
```

or

```java
@Teleop(name = "MyHalTeleop", group = "Example Programs")
public class MyHALTeleop extends BaseTeleop {
    public @MainRobot MyRobot robot;
    
    @Override
    protected void onInit() {

    }

    @Override
    protected void onInitLoop() {

    }

    @Override
    protected void onStart() {
        
    }

    @Override
    protected void onUpdate() {
       
    }

    @Override
    protected void onStop() {

    }
}
```

Now that we have our HAL Teleop program, lets make a [HAL Autonomous](hal-autonomous.md) program!

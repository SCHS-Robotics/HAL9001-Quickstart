---
layout: page
title: The HAL Vision System
description: How to use the HAL9001 Vision System
dropdown: Tutorials
priority: -8
---
-------------------
Ok, now lets move on to another very useful part of HAL, its vision system.

In FTC, computer vision is becoming more and more fundamental to how teams play each year's game. In recognition of this, HAL has built-in integration with the [EasyOpenCV] (https://github.com/OpenFTC/EasyOpenCV) library. It not only uses many of the same classes as EasyOpenCV, but also uses its same vision pipeline programming style. But more on that later.

Interacting with HAL's vision system happens in 3 steps.

## Step 1: Create the Cameras
The first step for using HAL's vision system is to define the camera or cameras that the vision systems will use. Because these cameras will be shared by all subsystems, they are defined in the robot class:

```java
public class MyRobot extends Robot {
    @InternalCamera(resWidth = 320, resHeight = 240, usesViewport = true)
    public OpenCvInternalCamera camera;
    @ExternalCamera(resWidth = 320, resHeight = 240, configName = "my camera", usesViewport = false)
    public OpenCvCamera camera;

    public MyRobot(OpMode opMode) {
        super(opMode);
    }
}
```

Here we define two cameras, one internal camera (the camera that would be present on a phone, for example) and one external camera (a webcam). Only one internal camera may be defined, but *ANY NUMBER* of external cameras may be defined. 

Both cameras are defined via annotation. In the `@InternalCamera` annotation, there are 3 required parameters, and 1 optional parameter. The three required parameters, shown in the example, are the camera's returned image resolution width and height (in pixels), and whether or not the camera uses the viewport.

The optional parameter in `@InternalCamera` is the camera direction. This refers to whether the front or back camera is being used as input. By default, this is set to the back phone camera.

Due to the limitations of android, you may only stream video from _**ONE**_ of these cameras. This is just how android made it, I tried giving people the ability to stream both by switching between them really fast, and it didn't work and made the phone heat up by a surpisingly large amount.

Here is what the annotation would look like if you wanted to set the camera direction:

```java
@InternalCamera(resWidth = 320, resHeight = 240, usesViewport = true, direction = OpenCvInternalCamera.CameraDirection.FRONT)
```

External cameras are also defined via annotation in a similar way to internal cameras. There are 4 required parameters and one optional parameter.

The first two parameters are the same as the internal camera parameters: the resolution width and height of the returned camera frames.

The third parameter is the webcam's config name and the 4th parameter, like in the external camera, is whether or not the camera will use the viewport.

The optional parameter is camera unique id (which will be important later). The unique id should identify one specific camera associated with that id. If no unique id is provided (as in the original example), then the unique id is set to the webcam config name.

Here is an example of the `@ExternalCamera` annotation with the unique id set to something other than its config name:

```java
@ExternalCamera(resWidth = 320, resHeight = 240, configName = "my camera", uniqueId = "Unique", usesViewport = false)
```

Something important to note is that I TECHNICALLY was lying when I said the cameras were being defined. Neither of these camera fields in the robot are actually technically REALLY defined, as they both have null values, but they let the HAL internals know which cameras to initialize with the given values.

The reason for this is because for some reason, OpenCvCamera objects don't work very well with reflection. I have no idea why, but they just don't for some reason (shrug). I'll keep trying to fix it, but until then, you'll have to use HAL's alternative method to get the cameras.

### Accessing Camera Objects
Now the alternative method! In order to set the actual camera variables you need to use robot's `getCamera()` function in order to get a camera object via its unique id (remember I said that would come in handy?). 

```java
public class MyRobot extends Robot {
public class FakeDummyRobot extends Robot {
    @InternalCamera(resWidth = 320, resHeight = 240, usesViewport = true)
    public OpenCvInternalCamera internalCamera = (OpenCvInternalCamera) getCamera(Robot.INTERNAL_CAMERA_ID);
    @ExternalCamera(resWidth = 320, resHeight = 240, configName = "my camera", usesViewport = false)
    public OpenCvCamera camera = getCamera("my camera");

    public FakeDummyRobot(OpMode opMode) {
        super(opMode);
    }
}

    public MyRobot(OpMode opMode) {
        super(opMode);
    }
}
```

This step is actually optional, as the camera objects have already been created internally. You would only do this if you wanted to access the direct camera objects to do things like set the camera iso/exposure, turn the phone flashlight on, etc.

_**Note that the internal camera's unique id is a static constant in the Robot class.**_

For more info on what camera settings you can change, refer to [EasyOpenCV's examples](https://github.com/OpenFTC/EasyOpenCV/tree/master/examples/src/main/java/org/openftc/easyopencv/examples).

## Step 2: Creating the Vision Subsystem
Ok! Now that we have successfully created our camera objects, it's time to actually create our vision subsystem (by creating a class that extends VisionSubSystem)

```java
public class MyVisionSubSystem extends VisionSubSystem {
    public MyVisionSubSystem(Robot robot) {
        super(robot);
    }

    @Override
    protected HALPipeline[] getPipelines() {
        return new HALPipeline[0];
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
    public void handle() {

    }

    @Override
    public void stop() {

    }
}
```

Now, you may notice that this looks almost the same as a normal HAL subsystem class. This is by design, as vision subsystems are, by definition, subsystems. They have all the capabilities of normal HAL subsystems in addition to the ability to easily run computer vision pipelines.

The one function that vision subsystems have that normal subsystems don't is the `getPipelines()` function, which should return a list of all the HAL computer vision pipelines that the subsystem is running.

Now you may be asking, "What HAL vision pipelines? I don't see any HAL vision pipelines!" and you would be correct, as we have not created any yet. Just like in EasyOpenCV, pipelines are created as separate classes.

Lets create a simple vision pipeline that just draws a square on the screen:

```java
public class MyVisionSubSystem extends VisionSubSystem {
    public MyVisionSubSystem(Robot robot) {
        super(robot);
    }

    @Override
    protected HALPipeline[] getPipelines() {
        return new HALPipeline[0];
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
    public void handle() {

    }

    @Override
    public void stop() {

    }
	
	public static class SquarePipeline extends HALPipeline {
        @Override
        public Mat processFrame(Mat input) {
            Imgproc.rectangle(
                    input,
                    new Point(
                            input.cols() / 4,
                            input.rows() / 4),
                    new Point(
                            input.cols() * (3f / 4f),
                            input.rows() * (3f / 4f)),
                    new Scalar(0, 255, 0), 4);
            return input;
        }

        @Override
        public boolean useViewport() {
            return true;
        }
    }
}
```

We now have a simple pipeline that draws a square on the screen. There are a few things here worth noting.

First, the pipeline extends HALPipeline, which is not an EasyOpenCV class. You should not extend the normal OpenCVPipeline class. The HALPipeline class has the same functionality, plus a little extra in the form of the `useViewport()` function.

Second, the processFrame function is EXACTLY the same as in normal OpenCVPipeline classes. You can literally copy-paste your EasyOpenCV code into there and it will work. The one exception to this is if your normal EasyOpenCV pipeline uses the `onViewportTapped()` method, which HALPipeline does not currently include support for (although it is planned to be added in future versions).

Now that we have created our pipeline class, we have to tell it which cameras will be running that pipeline class. This is done through the `@Camera` annotation. This annotation takes only a unique id. This unique id is the camera that will be running the pipeline.

```java
public class MyVisionSubSystem extends VisionSubSystem {
    public MyVisionSubSystem(Robot robot) {
        super(robot);
    }

    @Override
    protected HALPipeline[] getPipelines() {
        return new HALPipeline[0];
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
    public void handle() {

    }

    @Override
    public void stop() {

    }
	
    @Camera(id = "my camera")
    public static class SquarePipeline extends HALPipeline {
        @Override
        public Mat processFrame(Mat input) {
            Imgproc.rectangle(
                    input,
                    new Point(
                            input.cols() / 4,
                            input.rows() / 4),
                    new Point(
                            input.cols() * (3f / 4f),
                            input.rows() * (3f / 4f)),
                    new Scalar(0, 255, 0), 4);
            return input;
        }

        @Override
        public boolean useViewport() {
            return true;
        }
    }
}
```

In this example, the pipeline class is bound to the camera with unqique id "my camera". If you want multiple cameras to run this pipeline, simply repeat the `@Camera` annotation with a new camera id.


```java
public class MyVisionSubSystem extends VisionSubSystem {
    public MyVisionSubSystem(Robot robot) {
        super(robot);
    }

    @Override
    protected HALPipeline[] getPipelines() {
        return new HALPipeline[0];
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
    public void handle() {

    }

    @Override
    public void stop() {

    }
	
	@Camera(id = Robot.INTERNAL_CAMERA_ID)
    @Camera(id = "my camera")
    public static class SquarePipeline extends HALPipeline {
        @Override
        public Mat processFrame(Mat input) {
            Imgproc.rectangle(
                    input,
                    new Point(
                            input.cols() / 4,
                            input.rows() / 4),
                    new Point(
                            input.cols() * (3f / 4f),
                            input.rows() * (3f / 4f)),
                    new Scalar(0, 255, 0), 4);
            return input;
        }

        @Override
        public boolean useViewport() {
            return true;
        }
    }
}
```

Or, if you want to run the pipeline on all defined cameras, you can use the special Robot.ALL_CAMERAS_ID.

```java
public class MyVisionSubSystem extends VisionSubSystem {
    public MyVisionSubSystem(Robot robot) {
        super(robot);
    }

    @Override
    protected HALPipeline[] getPipelines() {
        return new HALPipeline[0];
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
    public void handle() {

    }

    @Override
    public void stop() {

    }
	
	@Camera(id = Robot.ALL_CAMERAS_ID)
    public static class SquarePipeline extends HALPipeline {
        @Override
        public Mat processFrame(Mat input) {
            Imgproc.rectangle(
                    input,
                    new Point(
                            input.cols() / 4,
                            input.rows() / 4),
                    new Point(
                            input.cols() * (3f / 4f),
                            input.rows() * (3f / 4f)),
                    new Scalar(0, 255, 0), 4);
            return input;
        }

        @Override
        public boolean useViewport() {
            return true;
        }
    }
}
```

Finally, we need to talk about the `useViewport()` function, which returns whether or not the pipeline should display on the viewport. 

This is different from whether or not the CAMERA uses the viewport. EACH camera can only display ONE pipeline at a time. If a camera is told to run multiple pipelines, it will cycle between them each time the viewport is tapped. 

However, if a *pipeline* does NOT use the viewport, then it will not display on the viewport, no matter how many times you tap it. In other words, if `useViewport()` returns false, the results of the pipeline will never display on the viewport.

The value of useViewport can change over the course of a program if desired.

If a camera uses a viewport and none of the pipelines it runs use the viewport, it will simply show a black screen where the viewport should be.

Ok, now we've created our HAL Pipeline and attached it to a camera. Its time for step 3, the shortest step.

## Step 3: Use the Pipeline
First, we need to give the HAL vision system the actual instance of the pipeline we want to run. This is done using the `getPipelines()` function.

```java
public class MyVisionSubSystem extends VisionSubSystem {
    public MyVisionSubSystem(Robot robot) {
        super(robot);
    }

    @Override
    protected HALPipeline[] getPipelines() {
        return new HALPipeline[] {new SquarePipeline()};
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
    public void handle() {

    }

    @Override
    public void stop() {

    }
	
	@Camera(id = "my camera")
    public static class SquarePipeline extends HALPipeline {
        @Override
        public Mat processFrame(Mat input) {
            Imgproc.rectangle(
                    input,
                    new Point(
                            input.cols() / 4,
                            input.rows() / 4),
                    new Point(
                            input.cols() * (3f / 4f),
                            input.rows() * (3f / 4f)),
                    new Scalar(0, 255, 0), 4);
            return input;
        }

        @Override
        public boolean useViewport() {
            return true;
        }
    }
}
```

And thats it, the pipeline will get run as part of the vision system. Congratulations!

If you want to access the pipeline instance over the course of the subsystem, just store it in a field before returning it in getPipelines().

```
public class MyVisionSubSystem extends VisionSubSystem {
    private SquarePipeline myPipeline = new SquarePipeline();
    
    public MyVisionSubSystem(Robot robot) {
        super(robot);
    }

    @Override
    protected HALPipeline[] getPipelines() {
        return new HALPipeline[] {myPipeline};
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
    public void handle() {

    }

    @Override
    public void stop() {

    }

    @Camera(id = "my camera")
    public static class SquarePipeline extends HALPipeline {
        @Override
        public Mat processFrame(Mat input) {
            Imgproc.rectangle(
                    input,
                    new Point(
                            input.cols() / 4,
                            input.rows() / 4),
                    new Point(
                            input.cols() * (3f / 4f),
                            input.rows() * (3f / 4f)),
                    new Scalar(0, 255, 0), 4);
            return input;
        }

        @Override
        public boolean useViewport() {
            return true;
        }
    }
}
```

### Internal Camera Fun
Ok, if you remember at the beginning of the tutorial I said that it wasn't possible to use both the front and back phone internal cameras. This is correct, you can't use both the front and back cameras _**AT THE SAME TIME**_. HAL has a function to reverse the direction of the internal camera during the program if you want to.

Calling robot.reverseInternalCameraDirection() will cause the direction of the internal camera to flip either from back to front or front to back. All pipelines associated with the internal camera will transfer over.

### Quick Note on the HAL System
I realize the HAL vision system may not be everyone's ideal vision system in its current form. I would like to note that use of the vision system is entirely optional, and is meant to make it EASIER to use computer vision in HAL by doing things like centralizing where cameras are, taking care of some syntax behind the scenes, etc.

If you don't find this system easier, the normal EasyOpenCV system will _**STILL 100% WORK**_ with HAL subsystems in the exact same way as it would with normal non-HAL code.

## Congratulations! You can use the HAL Vision System Now!
You can now use the HAL vision system! Congratulations! You have not only learned a valuable skill for programming with HAL, but are now more than halfway done with the tutorial!

The next tutorial is on HAL's [Calibration Subsystems](index.md), which help you with common calibration tasks.

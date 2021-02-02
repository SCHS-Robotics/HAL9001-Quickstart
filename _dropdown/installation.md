---
layout: page
title: HAL Installation and Update Guide
description: How to install the HAL 9001 library
dropdown: Tutorials
priority: 0
---
----------------------------
Hello! Thanks for deciding to download HAL! This guide is here to help you through the process of adding HAL to your project, as well as updating HAL when needed. If you're just starting with HAL, you should look at the [First Subsystem](first-subsystem.html) tutorial next in order to learn the first step in how to interact with HAL's command structure.
  
## How to Download HAL
### Install Android Studio (Click Image)
[![](https://www.androidpolice.com/wp-content/themes/ap2/ap_resize/ap_resize.php?src=https%3A%2F%2Fwww.androidpolice.com%2Fwp-content%2Fuploads%2F2017%2F05%2Fnexus2cee_Android-Studio-3.0-hero_thumb.png&w=728)](https://developer.android.com/studio)

### Add HAL9001 to your FTC App Project
1. Download the [ftc app](https://github.com/FIRST-Tech-Challenge/FtcRobotController).

2. Open Android Studio.

3. If there are no projects already open, click import project, otherwise, click file, import project.

4. Download/Install/Update anything that android studio tells you to.

5. Go to your project's build.common.gradle file and change this code block at the bottom:

```gradle
repositories {   
    flatDir {       
        dirs rootProject.file('libs')   
    }
}
```

to this:

```gradle
repositories {
    flatDir {
        dirs rootProject.file('libs')
    }
    jcenter()
}
```

6. Right click on teamcode, then click open module settings. Set the source compatibility to Java 1.8. As of HAL 1.1.0, HAL will only support Java 8, as all legal FTC control devices support Java 8 as of the 2020-2021 season.

7. Go to your teamcode's build.release.gradle file and add this line to the end: 

```gradle 
implementation 'org.hal9001:hal9001:x.x.x' 
```

Where the x.x.x is the version number of HAL9001 that you are using (check [the releases page](https://github.com/SCHS-Robotics/HAL9001/releases) of the HAL Github repository).

8. Plug your robot controller phone into the computer and swipe down, then click where it says "USB for charging" and change it to File Transfers.

9. Download the latest .so file from the [HAL9001-OpenCV-Repackaged repository](https://github.com/SCHS-Robotics/HAL9001-OpenCV-Repackaged) (check the releases section of that repository).

10. Open file explorer (or finder for mac people) and navigate to the phone. It should show up just like a USB drive.

11. Copy paste the .so file into the FIRST folder on the phone's internal storage.

12. Celebrate, you're done!

## How to Install Floobits (Optional)
1. Click file, settings, plugins

2. Click marketplace (top of the screen)

3. Search for floobits

4. Click install, then click apply.

5. Restart Android Studio.

## How to Integrate With Floobits (Optional)
### Creating the Repository
1. Setup the repository (see steps above)

2. Click tools, floobits, share project publicly.

3. Enter a name for the floobits project and sign into your account if you are not already signed in.

4. Click upload entire project.

5. Go to [floobits](https://floobits.com) and set permissions for your new project.

### Joining the Floobits Project.
1. Install floobits (See install floobits section)

2. Make sure you are logged into your floobits account.

3. Download your team's version of the repository (with their version of the .floo file)

4. Click tools, floobits, clear cache.

5. Click tools, floobits, join project's workspace

6. Click _**OVERWRITE LOCAL**_. If you click overwrite remote **YOU WILL MURDER EVERY SINGLE ONE OF YOUR PROGRAMMERS. SERIOUSLY, DO NOT CLICK OVERWRITE REMOTE UNLESS YOU REALLY UNDERSTAND WHAT YOU ARE DOING.**

## Updating HAL to a New Version
1. Go to your teamcode's build.release.gradle file and find this line that you added during the installation process:

```gradle 
implementation 'org.hal9001:hal9001:x.x.x' 
```

2. Change the "x.x.x" at the end of the line to the new HAL version number (check [the releases page](https://github.com/SCHS-Robotics/HAL9001/releases) of the HAL Github repository).

3. Connect to Wifi and click build (the green hammer) or upload the code to the phone.

4. Congratulations! You're updated! :)

# Fork of VDavid003's sm64ex nightly branch Android Port
This is a port of the reconstructed Super Mario 64 source code to Android using SDL2 with OpenGL ES 2.0.

It has cross-platform Touch Controls, Audio works, it saves the game to the app's internal storage and you can play it with an external keyboard or controller as well (tested on PS3 controller).

> **Note**
> 
> This fork adds an app icon, goes fully immersive and fixes an issue with correctly identifying the first real controller by filtering any facade controller which might be identified. To do a build with the improved n64 textures from the Render 96 project, removed touch controls and activated better camera see instructions at the end of this README.

# Build instructions

## Linux

**Install dependencies:**

This depends on your distro, but if you can build the PC port and you have Android SDK/NDK and you are able to build Android apps using gradle, you should be fine.

**Clone the repository:**
```sh
git clone --recursive https://github.com/VDavid003/sm64-port-android-base --branch sm64ex_nightly
cd sm64-port-android-base
```

**Copy in your baserom:**
```sh
cp /path/to/your/baserom.z64 ./app/jni/src/baserom.us.z64
```

**Get SDL sources:**
```sh
./getSDL.sh
```

**Perform native build twice:**
```sh
# if you have more cores available, you can increase the --jobs parameter
cd app/jni/src
make --jobs 4
make --jobs 4
cd ../../..
```

**Perform Android build:**
```sh
./gradlew assembleDebug
```

**Enjoy your apk:**
```sh
ls -al ./app/build/outputs/apk/debug/app-debug.apk
```

## Windows

**Install dependencies:**

You'll need everything you need to make Windows builds (not just vanilla sm64 ones, but sm64ex ones), and to be able to build Android apps using `gradlew.bat`. This includes Java JDK (with the JDK being JAVA_HOME) and Android SDK/NDK. Every commmand is executed in MSYS2 unless otherwise noted.

**Clone the repository:**
```sh
git clone --recursive https://github.com/VDavid003/sm64-port-android-base --branch sm64ex_nightly
```

**Copy in your baserom:**
Use the file explorer, or whatever you want, just put it in `app/jni/src`, and name it like you'd do on the PC port.
```sh
cp /path/to/your/baserom.z64 ./app/jni/src/baserom.us.z64
```

**Get SDL sources:**
```sh
./getSDL.sh
```

**Perform native build twice:**
```sh
# if you have more cores available, you can increase the --jobs parameter
cd app/jni/src
make --jobs 4
make --jobs 4
cd ../../..
```

**Perform Android build:**
Do this in a normal Command Prompt!
```
gradlew.bat assembleDebug
```

## Docker

**Clone the repository:**
```sh
git clone --recursive https://github.com/irockel/sm64-port-android-base --branch sm64ex_nightly
```

**Create the build image:**
```sh
# navigate into newly cloned repo
cd sm64-port-android-base
# build the docker image
docker build . -t sm64_android
```
**Copy in your baserom:**
```sh
cp /path/to/your/baserom.z64 ./app/jni/src/baserom.us.z64
```

**Setup symlinks for SDL:**
```sh
docker run --rm -v $(pwd):/sm64 sm64_android sh -c "ln -nsf /SDL2-2.0.12/src /sm64/app/jni/SDL/src"
docker run --rm -v $(pwd):/sm64 sm64_android sh -c "ln -nsf /SDL2-2.0.12/include /sm64/app/jni/SDL/include"
```

**Perform native build twice:**
```sh
# if you have more cores available, you can increase the --jobs parameter
docker run --rm -v $(pwd):/sm64 sm64_android sh -c "cd /sm64/app/jni/src && make --jobs 4"
docker run --rm -v $(pwd):/sm64 sm64_android sh -c "cd /sm64/app/jni/src && make --jobs 4"
```

**Perform Android build:**
```sh
docker run --rm -v $(pwd):/sm64 sm64_android sh -c "./gradlew assembleDebug"
```

**Enjoy your apk:**
```sh
ls -al ./app/build/outputs/apk/debug/app-debug.apk
```

# Configuration
If you want to customize the build with build options, you should make the native build with those options first (put them after the make command like on normal repos), then before performing the Android build, edit `app/jni/src/Android.mk` and enable the options you'd like.

## EXTERNAL_DATA option
If you use `EXTERNAL_DATA`, you'll find a zip named `base.zip` in `app/jni/src/build/<version>_pc/res`.

You should take this zip and put it in `Internal Storage/Android/data/com.vdavid003.sm64port/files`

## Render96/SGI Models instructions (Only tested on 1.4.2)
The 60fps patch is strongly recommended as it not only makes the game look smoother but doubles the performance as well with VSync on.

Turning VSync off is not recommended as it can cause random "over-speedups" where when getting out of a laggy area, the game suddenly becomes way too fast. Also if you have a non-60hz phone, try setting your refresh rate to 60hz.

Sometimes turning VSync on is problematic, so if you have the game already installed, enable it before installing a heavier version.

**Follow normal instructions, but stop before doing the native build**

**Extract the Render96 zip file to app/jni/src overwriting everything**

**Apply the render96_android patch and the 60fps patch:**
```sh
cd app/jni/src
git apply enhancements/render96_android.patch
git apply enhancements/60fps_ex.patch
cd ../../..
```

**Continue with the normal instructions and build the game.**

**Follow the instructions for EXTERNAL_DATA**

## Build with improved n64 Textures, No Touch Controls, Better Camera on Linux

Prepare according Linux instructions above (up to doing the native build).

Configure local.properties in the root folder like this

```sh
## This file must *NOT* be checked into Version Control Systems,
# as it contains information specific to your local configuration.
#
# Location of the SDK. This is only used by Gradle.
# For customization when using a Version Control System, please read the
# header note.
#Sat Mar 27 22:06:22 CET 2021
sdk.dir=/path/to/Android/Sdk
ndk.dir=/path/to/Android/Sdk/ndk/21.3.6528147
```
Maybe you need to download the fitting NDK, the mentioned version is known to work.

Apply the render96_android.patch and 60fpx_ex.patch (see above).

Apply n64 textures from [Render 96](https://linktr.ee/Render96) project by downloading zip from here:
  https://github.com/pokeheadroom/RENDER96-HD-TEXTURE-PACK/tree/N64-Downscaled-Render96
and copy everything inside the "gfx" folder inside the zip into app/jni/src.

Remove any in-fixes the files inside the app/jni/src/textures/skyboxes, remove any ".rgba16".

Remove any images in app/jni/src/levels/ending beside the cake.png.

Build native code
```sh
cd app/jni/src
TOUCH_CONTROLS=0 NODRAWINGDISTANCE=1 BETTERCAMERA=1 make --jobs 4
TOUCH_CONTROLS=0 NODRAWINGDISTANCE=1 BETTERCAMERA=1 make --jobs 4
cd ../../..
```
(for some strange reasons this has to be done twice)

Do the apk build, don't forget the flags:
```sh
cd app/jni/src
TOUCH_CONTROLS=0 NODRAWINGDISTANCE=1 BETTERCAMERA=1 ./gradlew assembleDebug
cd ../../..
```

After this the installable apk resides in app/build/outputs/apk/debug/


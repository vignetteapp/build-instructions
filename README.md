# Build instructions

> A collection of all the build instructions we need to make Vignette work.

## Summary

- Mediapipe
- Arch Linux
  - Install Bazel
  - Install GCC 10
  - Install OpenCV
    - Configure Mediapipe to open OpenCV 4
  <!-- - Install Python 3 and Numpy -->
  - Build `hello_world` and other examples

## Mediapipe

Trying to build Mediapipe has produced a lot of struggle. It involved trying to figure out the Bazel targets, figuring out what the tons of errors mean, and how to fix them. In the end, we have gained knowledge of a good portion of what the environment needs for Mediapipe to build successfully, without needing any Docker image.

Before any specifics about platforms, clone the Mediapipe repository.

```sh
$ git clone https://github.com/google/mediapipe
```

## Arch Linux

To build Mediapipe on Arch Linux, we need:
- Bazel 3.7.2 or 4
- GCC 10
- OpenCV
<!-- - Python 3 and Numpy -->

### Install Bazel

The first step is to install Bazel. We have 2 options: Bazel 3.7.2, which is the version specified in the `.bazelversion` file of the Mediapipe repository, or Bazel 4.\*, which can still build everything fine.

- **Bazel 3.7.2**
  If you wish to continue with Bazel 3.7.2, you will have to install the right Bazel version from the AUR, as the community repository provides the latest version. The easier way is with an AUR helper such as `yay` or `paru` in which you can specify the exact version you need:

  ```sh
  $ yay -S bazel=3.7.2
  ```
  You will be forwarded with 2 options: the package `bazel3` and the package `bazel3-bin`. If you want to save up time, we recommend the latter as the former will build Bazel 3 from source.
- **Bazel 4**
  If you wish to continue with the latest version of Bazel (4.2.0-2 as of today), then you need to do 2 things:
  - install the latest version of Bazel through the community repository.
    ```sh
    $ sudo pacman -S bazel
    ```
    (it can also work with your AUR helper such as it usually supersets `pacman`.)
  - modify the `.bazelversion` file inside the Mediapipe repository so that Bazel doesn't scream at you for using the latest version.
    You need to be in the parent folder of the repository for this to work, as Bazel will *still* scream that you're using the wrong version for the repository even if you just want to know the current Bazel version (image is left to be documented here).
    ```sh
    $ bazel --version | sed 's/bazel //' > mediapipe/.bazelversion
    ```

### Install GCC 10

Arch Linux ships with a more recent version of GCC (11.1.0 as of today). However, you will get compilation errors if you try to build Mediapipe with this version. Something probably got deprecated between 10 and 11 that is preventing Mediapipe to build successfully. Meaning, we have to install GCC 10.

Luckily, there is a community package for GCC 10.
```sh
$ sudo pacman -S gcc10
```

Installing this package won't remove GCC 11 from your system if you have installed it. GCC 11 will still be the default version used unless specified otherwise. We will be able to specify that we want to use version 10 with an environment variable: `CC=gcc-10`.

### Install OpenCV

Here, we will focus on how to compile Mediapipe using OpenCV 4. We are not interested in building OpenCV from source, as it can be provided to us very easily from the Arch community repository. As of today, the latest version of OpenCV is 4.5.3-4.
```sh
$ sudo pacman -S opencv
```

#### Configure Mediapipe to accept OpenCV 4

The crucial step is to configure Mediapipe to accept OpenCV 4. This is documented on the *Install Mediapipe* page of the Mediapipe website.

Inside of the `mediapipe` repository, ou want to go edit the `third_party/opencv_linux.BUILD` file with your preferred text editor.

Uncomment the lines necessary for Mediapipe to accept OpenCV 4 header files that have been installed on your system from the Arch community repository.
Very likely, it will be the last lines, namely *l.21* and *l.28*, but you could need to uncomment additional ones if you want to compile for a different architecture.

The final rule should look like this:
```bazel
# third_party/opencv_linux.BUILD

cc_library(
    name = "opencv",
    hdrs = glob([
        # For OpenCV 4.x
        #"include/aarch64-linux-gnu/opencv4/opencv2/cvconfig.h",
        #"include/arm-linux-gnueabihf/opencv4/opencv2/cvconfig.h",
        #"include/x86_64-linux-gnu/opencv4/opencv2/cvconfig.h",
        "include/opencv4/opencv2/**/*.h*",
    ]),
    includes = [
        # For OpenCV 4.x
        #"include/aarch64-linux-gnu/opencv4/",
        #"include/arm-linux-gnueabihf/opencv4/",
        #"include/x86_64-linux-gnu/opencv4/",
        "include/opencv4/",
    ],
    linkopts = [
        "-l:libopencv_core.so",
        "-l:libopencv_calib3d.so",
        "-l:libopencv_features2d.so",
        "-l:libopencv_highgui.so",
        "-l:libopencv_imgcodecs.so",
        "-l:libopencv_imgproc.so",
        "-l:libopencv_video.so",
        "-l:libopencv_videoio.so",
    ],
    visibility = ["//visibility:public"],
)
```

<!--
### Install Python 3 and Numpy

At some point in the build, Mediapipe will use Python and Numpy. This is the easiest part of the building process:
```sh
$ sudo pacman -S python python-numpy
```
-->

### Build `hello_world` and other examples

After all that setup, you can finally build `hello_world` and other examples.
We will be using the `CC=gcc-10` environment variable for Bazel to use GCC 10.

Here, we will build the examples `hello_world`, `face_mesh`, `hand_tracking` and `holistic` on desktop for the CPU.

Inside of the `mediapipe` respository:

```sh
# Building the hello_world example
$ CC=gcc-10 bazel build -c opt --define MEDIAPIPE_DISABLE_GPU=1 mediapipe/examples/desktop/hello_world:hello_world
Loading: 
Loading: 0 packages loaded
# (Insert DEBUG and WARNING logs here)
Analyzing: target //mediapipe/examples/desktop/hello_world:hello_world (0 packages loaded, 0 targets configured)
INFO: Analyzed target //mediapipe/examples/desktop/hello_world:hello_world (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
[0 / 1] [Prepa] BazelWorkspaceStatusAction stable-status.txt
Target //mediapipe/examples/desktop/hello_world:hello_world up-to-date:
  bazel-bin/mediapipe/examples/desktop/hello_world/hello_world
INFO: Elapsed time: 0.229s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Build completed successfully, 1 total action

# Running the hello_world example
$ GLOG_logtostderr=1 bazel-bin/mediapipe/examples/desktop/hello_world/hello_world
I20210913 11:32:30.340116 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340193 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340214 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340231 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340247 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340265 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340286 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340303 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340320 88567 hello_world.cc:57] Hello World!
I20210913 11:32:30.340337 88567 hello_world.cc:57] Hello World!

# Building the face_mesh_cpu example
$ CC=gcc-10 bazel build -c opt --define MEDIAPIPE_DISABLE_GPU=1 mediapipe/examples/desktop/face_mesh:face_mesh_cpu
Loading: 
Loading: 0 packages loaded
# (Insert DEBUG and WARNING logs here)
Analyzing: target //mediapipe/examples/desktop/face_mesh:face_mesh_cpu (0 packages loaded, 0 targets configured)
INFO: Analyzed target //mediapipe/examples/desktop/face_mesh:face_mesh_cpu (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
[0 / 1] [Prepa] BazelWorkspaceStatusAction stable-status.txt
Target //mediapipe/examples/desktop/face_mesh:face_mesh_cpu up-to-date:
  bazel-bin/mediapipe/examples/desktop/face_mesh/face_mesh_cpu
INFO: Elapsed time: 0.195s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Build completed successfully, 1 total action

# Running the face_mesh_cpu example
$ GLOG_logtostderr=1 bazel-bin/mediapipe/examples/desktop/face_mesh/face_mesh_cpu \
    --calculator_graph_config_file=mediapipe/graphs/face_mesh/face_mesh_desktop_live.pbtxt
I20210913 11:35:14.123477 91460 demo_run_graph_main.cc:48] Get calculator graph config contents: # MediaPipe graph that performs face mesh with TensorFlow Lite on CPU.
# (Insert some parameters, nodes and graphs here)
I20210913 11:35:14.124388 91460 demo_run_graph_main.cc:54] Initialize the calculator graph.
I20210913 11:35:14.127234 91460 demo_run_graph_main.cc:58] Initialize the camera or load the video.
[ WARN:0] global /build/opencv/src/opencv-4.5.3/modules/videoio/src/cap_gstreamer.cpp (1081) open OpenCV | GStreamer warning: Cannot query video position: status=0, value=-1, duration=-1
QSettings::value: Empty key passed
QSettings::value: Empty key passed
I20210913 11:35:16.592208 91460 demo_run_graph_main.cc:79] Start running the calculator graph.
I20210913 11:35:16.595728 91460 demo_run_graph_main.cc:84] Start grabbing and processing frames.
INFO: Created TensorFlow Lite XNNPACK delegate for CPU.
I20210913 11:35:18.639092 91460 demo_run_graph_main.cc:143] Shutting down.
I20210913 11:35:18.641398 91460 demo_run_graph_main.cc:157] Success!
```

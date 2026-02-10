Welcome to Mocapa's fork of Mediapipe.

It adds in gpu selection support. 

Build instructions - [reference](https://github.com/google-ai-edge/mediapipe/blob/master/docs/getting_started/install.md)
1. Bazelisk installation: 
```bash
curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 \
  -o bazelisk

# make it executable
chmod +x bazelisk

# move to your path
sudo mv bazelisk /usr/local/bin/bazelisk

# verify version
bazelisk version

# make `bazel` point to `bazelisk`
sudo ln -sf /usr/local/bin/bazelisk /usr/local/bin/bazel

```

2. Install OpenCV 4.6 and FFmpeg
```bash
sudo apt-get update

sudo apt-get install -y \
    libopencv-core-dev \
    libopencv-highgui-dev \
    libopencv-calib3d-dev \
    libopencv-features2d-dev \
    libopencv-imgproc-dev \
    libopencv-video-dev \
    libopencv-contrib-dev \
    libopencv-dev \
    pkg-config \
    protobuf-compiler


# verify version
pkg-config --modversion opencv4
```

3. Enable GPU support - [reference](https://ai.google.dev/edge/mediapipe/framework/getting_started/gpu_support)
```bash
sudo apt-get install -y \
  mesa-common-dev libegl1-mesa-dev libgles2-mesa-dev mesa-utils

# verify outputs
glxinfo | grep -i opengl
## sample output:
OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 580.126.09
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
OpenGL ES profile extensions:
```

4. Test a simple Hello World build
   1. Test build with GPU-enabled build flags
   ```bash
   bazel clean --expunge
   bazel build -c opt \
     --copt -DMESA_EGL_NO_X11_HEADERS --copt -DEGL_NO_X11 \
     mediapipe/examples/desktop/hello_world:hello_world
   ```

   - Fixing some yarn/npm fetch issues (could be China-only issue)
       ```bash
       # force yarn/node to prefer IPv4
       echo 'export NODE_OPTIONS=--dns-result-order=ipv4first' >> ~/.bashrc
       source ~/.bashrc

       # set stable DNS
       sudo mkdir -p /etc/systemd/resolved.conf.d
       sudo tee /etc/systemd/resolved.conf.d/dns.conf >/dev/null <<'EOF'
       [Resolve]
       DNS=1.1.1.1 8.8.8.8
       FallbackDNS=9.9.9.9
       EOF
       sudo systemctl restart systemd-resolved

       # verify npm registry resolves
       getent hosts registry.npmjs.org
       curl -I https://registry.npmjs.org/foreground-child/-/foreground-child-2.0.0.tgz
       ## both of these should return quickly

       ## Re-do step 4 again.
       ```

   2. Test run Hello World build
   ```bash
   ./bazel-bin/mediapipe/examples/desktop/hello_world/hello_world
   ```

5. Build full package
```bash
# in your python environment
export PYTHON_BIN_PATH="$(which python)"

# Build with bazel
rm -rf build dist *.egg-info
bazel clean --expunge

export MEDIAPIPE_DISABLE_GPU=0

# bazel build -c opt \
#   --define MEDIAPIPE_DISABLE_GPU=0 \
#   --define MEDIAPIPE_PROFILING=1 \
#   --copt -DMESA_EGL_NO_X11_HEADERS \
#   --copt -DEGL_NO_X11 \
#   --action_env PYTHON_BIN_PATH=$(which python3) \
#   //mediapipe/tasks:internal

# bazel build -c opt \
#   --define MEDIAPIPE_DISABLE_GPU=0 \
#   --define MEDIAPIPE_PROFILING=1 \
#   --copt -DMESA_EGL_NO_X11_HEADERS \
#   --copt -DEGL_NO_X11 \
#   --action_env PYTHON_BIN_PATH=$(which python3) \
#   //mediapipe/gpu:gl_context


bazel build -c opt \
  --define MEDIAPIPE_DISABLE_GPU=0 \
  --define MEDIAPIPE_PROFILING=1 \
  --copt -DMESA_EGL_NO_X11_HEADERS \
  --copt -DEGL_NO_X11 \
  --action_env PYTHON_BIN_PATH=$(which python3) \
  //mediapipe/python:_framework_bindings

# Check if the .so files are built
ls bazel-bin/mediapipe/python/_framework_bindings*.so

bazel build -c opt \
  --define MEDIAPIPE_DISABLE_GPU=0 \
  --define MEDIAPIPE_PROFILING=1 \
  --copt -DMESA_EGL_NO_X11_HEADERS \
  --copt -DEGL_NO_X11 \
  --action_env PYTHON_BIN_PATH=$(which python3) \
  //mediapipe/tasks/c:libmediapipe.so

# HACKY: copy this .so file
cp -v bazel-bin/mediapipe/tasks/c/libmediapipe.so mediapipe/tasks/c/

# python setup.py bdist_wheel
python setup.py build_ext --link-opencv build_py --link-opencv bdist_wheel


# Check wheel contents
python - <<'PY'
import zipfile
z = zipfile.ZipFile("dist/mediapipe-0.0.0.dev20260209-cp311-cp311-linux_x86_64.whl")
for n in z.namelist():
    if "solutions" in n or "_framework_bindings" in n:
        print(n)
PY
# You must see `mediapipe/python/_framework_bindings*.so` and `mediapipe/solutions/__init__.py`

```


1. Install wheel
```bash
# Install new wheel:
# in your python env
pip install dist/*.whl --force-reinstall
# or add to uv
uv add


# Check GPU delegate path works
python test_gpu_device.py
## expected output: 
------


---
layout: forward
target: https://developers.google.com/mediapipe
title: Home
nav_order: 1
---

----

**Attention:** *We have moved to
[https://developers.google.com/mediapipe](https://developers.google.com/mediapipe)
as the primary developer documentation site for MediaPipe as of April 3, 2023.*

![MediaPipe](https://developers.google.com/static/mediapipe/images/home/hero_01_1920.png)

**Attention**: MediaPipe Solutions Preview is an early release. [Learn
more](https://developers.google.com/mediapipe/solutions/about#notice).

**On-device machine learning for everyone**

Delight your customers with innovative machine learning features. MediaPipe
contains everything that you need to customize and deploy to mobile (Android,
iOS), web, desktop, edge devices, and IoT, effortlessly.

*   [See demos](https://goo.gle/mediapipe-studio)
*   [Learn more](https://developers.google.com/mediapipe/solutions)

## Get started

You can get started with MediaPipe Solutions by by checking out any of the
developer guides for
[vision](https://developers.google.com/mediapipe/solutions/vision/object_detector),
[text](https://developers.google.com/mediapipe/solutions/text/text_classifier),
and
[audio](https://developers.google.com/mediapipe/solutions/audio/audio_classifier)
tasks. If you need help setting up a development environment for use with
MediaPipe Tasks, check out the setup guides for
[Android](https://developers.google.com/mediapipe/solutions/setup_android), [web
apps](https://developers.google.com/mediapipe/solutions/setup_web), and
[Python](https://developers.google.com/mediapipe/solutions/setup_python).

## Solutions

MediaPipe Solutions provides a suite of libraries and tools for you to quickly
apply artificial intelligence (AI) and machine learning (ML) techniques in your
applications. You can plug these solutions into your applications immediately,
customize them to your needs, and use them across multiple development
platforms. MediaPipe Solutions is part of the MediaPipe [open source
project](https://github.com/google/mediapipe), so you can further customize the
solutions code to meet your application needs.

These libraries and resources provide the core functionality for each MediaPipe
Solution:

*   **MediaPipe Tasks**: Cross-platform APIs and libraries for deploying
    solutions. [Learn
    more](https://developers.google.com/mediapipe/solutions/tasks).
*   **MediaPipe models**: Pre-trained, ready-to-run models for use with each
    solution.

These tools let you customize and evaluate solutions:

*   **MediaPipe Model Maker**: Customize models for solutions with your data.
    [Learn more](https://developers.google.com/mediapipe/solutions/model_maker).
*   **MediaPipe Studio**: Visualize, evaluate, and benchmark solutions in your
    browser. [Learn
    more](https://developers.google.com/mediapipe/solutions/studio).

### Legacy solutions

We have ended support for [these MediaPipe Legacy Solutions](https://developers.google.com/mediapipe/solutions/guide#legacy)
as of March 1, 2023. All other MediaPipe Legacy Solutions will be upgraded to
a new MediaPipe Solution. See the [Solutions guide](https://developers.google.com/mediapipe/solutions/guide#legacy)
for details. The [code repository](https://github.com/google/mediapipe/tree/master/mediapipe)
and prebuilt binaries for all MediaPipe Legacy Solutions will continue to be
provided on an as-is basis.

For more on the legacy solutions, see the [documentation](https://github.com/google/mediapipe/tree/master/docs/solutions).

## Framework

To start using MediaPipe Framework, [install MediaPipe
Framework](https://developers.google.com/mediapipe/framework/getting_started/install)
and start building example applications in C++, Android, and iOS.

[MediaPipe Framework](https://developers.google.com/mediapipe/framework) is the
low-level component used to build efficient on-device machine learning
pipelines, similar to the premade MediaPipe Solutions.

Before using MediaPipe Framework, familiarize yourself with the following key
[Framework
concepts](https://developers.google.com/mediapipe/framework/framework_concepts/overview.md):

*   [Packets](https://developers.google.com/mediapipe/framework/framework_concepts/packets.md)
*   [Graphs](https://developers.google.com/mediapipe/framework/framework_concepts/graphs.md)
*   [Calculators](https://developers.google.com/mediapipe/framework/framework_concepts/calculators.md)

## Community

*   [Slack community](https://mediapipe.page.link/joinslack) for MediaPipe
    users.
*   [Discuss](https://groups.google.com/forum/#!forum/mediapipe) - General
    community discussion around MediaPipe.
*   [Awesome MediaPipe](https://mediapipe.page.link/awesome-mediapipe) - A
    curated list of awesome MediaPipe related frameworks, libraries and
    software.

## Contributing

We welcome contributions. Please follow these
[guidelines](https://github.com/google/mediapipe/blob/master/CONTRIBUTING.md).

We use GitHub issues for tracking requests and bugs. Please post questions to
the MediaPipe Stack Overflow with a `mediapipe` tag.

## Resources

### Publications

*   [Bringing artworks to life with AR](https://developers.googleblog.com/2021/07/bringing-artworks-to-life-with-ar.html)
    in Google Developers Blog
*   [Prosthesis control via Mirru App using MediaPipe hand tracking](https://developers.googleblog.com/2021/05/control-your-mirru-prosthesis-with-mediapipe-hand-tracking.html)
    in Google Developers Blog
*   [SignAll SDK: Sign language interface using MediaPipe is now available for
    developers](https://developers.googleblog.com/2021/04/signall-sdk-sign-language-interface-using-mediapipe-now-available.html)
    in Google Developers Blog
*   [MediaPipe Holistic - Simultaneous Face, Hand and Pose Prediction, on
    Device](https://ai.googleblog.com/2020/12/mediapipe-holistic-simultaneous-face.html)
    in Google AI Blog
*   [Background Features in Google Meet, Powered by Web ML](https://ai.googleblog.com/2020/10/background-features-in-google-meet.html)
    in Google AI Blog
*   [MediaPipe 3D Face Transform](https://developers.googleblog.com/2020/09/mediapipe-3d-face-transform.html)
    in Google Developers Blog
*   [Instant Motion Tracking With MediaPipe](https://developers.googleblog.com/2020/08/instant-motion-tracking-with-mediapipe.html)
    in Google Developers Blog
*   [BlazePose - On-device Real-time Body Pose Tracking](https://ai.googleblog.com/2020/08/on-device-real-time-body-pose-tracking.html)
    in Google AI Blog
*   [MediaPipe Iris: Real-time Eye Tracking and Depth Estimation](https://ai.googleblog.com/2020/08/mediapipe-iris-real-time-iris-tracking.html)
    in Google AI Blog
*   [MediaPipe KNIFT: Template-based feature matching](https://developers.googleblog.com/2020/04/mediapipe-knift-template-based-feature-matching.html)
    in Google Developers Blog
*   [Alfred Camera: Smart camera features using MediaPipe](https://developers.googleblog.com/2020/03/alfred-camera-smart-camera-features-using-mediapipe.html)
    in Google Developers Blog
*   [Real-Time 3D Object Detection on Mobile Devices with MediaPipe](https://ai.googleblog.com/2020/03/real-time-3d-object-detection-on-mobile.html)
    in Google AI Blog
*   [AutoFlip: An Open Source Framework for Intelligent Video Reframing](https://ai.googleblog.com/2020/02/autoflip-open-source-framework-for.html)
    in Google AI Blog
*   [MediaPipe on the Web](https://developers.googleblog.com/2020/01/mediapipe-on-web.html)
    in Google Developers Blog
*   [Object Detection and Tracking using MediaPipe](https://developers.googleblog.com/2019/12/object-detection-and-tracking-using-mediapipe.html)
    in Google Developers Blog
*   [On-Device, Real-Time Hand Tracking with MediaPipe](https://ai.googleblog.com/2019/08/on-device-real-time-hand-tracking-with.html)
    in Google AI Blog
*   [MediaPipe: A Framework for Building Perception Pipelines](https://arxiv.org/abs/1906.08172)

### Videos

*   [YouTube Channel](https://www.youtube.com/c/MediaPipe)

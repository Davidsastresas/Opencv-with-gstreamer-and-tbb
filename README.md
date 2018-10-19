# Opencv-with-gstreamer-and-tbb

This is a guide to get built opencv all along with gstreamer support and intel tbb. Special thanks to Sagi Zeevi, from the blog www.theimpossiblecode.com for the instructions related to tbb as well as the prebuilt tbb packages, Adrian
 Rosebrock from pyimagesearch.com and http://pklab.net/?id=392&lang=EN ( don't know the name of the author ) For the detailed guides about this topic. 
 
This is proved and working under a raspberry pi 3 and the latest raspbian stretch, and opencv 3.4.3. 

First get installed all the dependencies for opencv

sudo apt-get update && sudo apt-get upgrade
sudo apt-get install build-essential cmake pkg-config 
sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
sudo apt-get install libgtk2.0-dev libgtk-3-dev
sudo apt-get install libcanberra-gtk*
sudo apt-get install libatlas-base-dev gfortran
sudo apt-get install python2.7-dev python3-dev  python2-numpy python3-numpy

Gstreamer packages

sudo apt-get install gstreamer1.0-omx-dbg
sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev


V4l needed for use with C++

sudo apt-get -y install libv4l-dev v4l-utils

Enable bcm2835 kernel module

sudo modprobe bcm2835-v4l2

after that, running this command we should see something like dev/video0

v4l2-ctl --list-devices

Now we have to add  this line:

bcm2835-v4l2

to the end of modules.conf

sudo nano /etc/modules-load.d/modules.conf

Now it's time to get TBB installed. There is a wonderfull guide here https://www.theimpossiblecode.com/blog/intel-tbb-on-raspberry-pi/ and on this guide can also be found the prebuilt packages which will be using here. This link is were the author Sagi Zeevi uploaded the prebuilt package:
https://www.dropbox.com/s/qj0kzoqfn3rw3op/libtbb-dev_4.5-1_armhf.deb?dl=0

Having the package, lets install it( considering it is in home folder )

sudo dpkg -i libtbb-dev_4.5-1_armhf.deb
sudo ldconfig

As Sagi Zeevi posted, we can check if it is installed correctly building this code:

cd /tmp
cat > hello_world.cpp
#include "tbb/tbb.h"
#include <iostream>
using namespace tbb;
using namespace std;

class first_task : public task { 
 public: 
 task* execute( ) { 
 cout << "Hello World!\n";
 return NULL;
 }
};

int main( )
{ 
 task_scheduler_init init(task_scheduler_init::automatic);
 first_task& f1 = *new(tbb::task::allocate_root()) first_task( );
 tbb::task::spawn_root_and_wait(f1);
}
^D
g++ hello_world.cpp `pkg-config tbb --cflags --libs`
./a.out

we should received a Hello world running this program, that should be we are fine with tbb

Now Download Opencv code and opencv_contrib

cd ~
wget -O opencv.zip https://github.com/opencv/opencv/archive/3.4.3.zip
unzip opencv.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/3.4.3.zip
unzip opencv_contrib.zip

Create a folder for the build files

cd ~/opencv-3.3.0/
mkdir build
cd build

Cmake options 

cmake -DCMAKE_CXX_FLAGS="-DTBB_USE_GCC_BUILTINS=1 -D__TBB_64BIT_ATOMICS=0" -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D BUILD_WITH_DEBUG_INFO=OFF -D BUILD_DOCS=OFF -D BUILD_EXAMPLES=OFF -D BUILD_TESTS=OFF -D BUILD_opencv_ts=OFF -D BUILD_PERF_TESTS=OFF -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-3.4.3/modules -D ENABLE_NEON=ON -D WITH_LIBV4L=ON -D ENABLE_VFPV3=ON -DWITH_TBB=ON        ../

We should check te output of Cmake, it should look similar to this:

General configuration for OpenCV 3.4.3 =====================================
  Version control:               unknown

  Extra modules:
    Location (extra):            /home/pi/opencv/opencv_contrib-3.4.3/modules
    Version control (extra):     unknown

  Platform:
    Timestamp:                   2018-10-19T15:56:44Z
    Host:                        Linux 4.14.76-v7+ armv7l
    CMake:                       3.7.2
    CMake generator:             Unix Makefiles
    CMake build tool:            /usr/bin/make
    Configuration:               RELEASE

  CPU/HW features:
    Baseline:                    VFPV3 NEON
      requested:                 DETECT
      required:                  VFPV3 NEON

  C/C++:
    Built as dynamic libs?:      YES
    C++11:                       YES
    C++ Compiler:                /usr/bin/c++  (ver 6.3.0)
    C++ flags (Release):         -DTBB_USE_GCC_BUILTINS=1 -D__TBB_64BIT_ATOMICS=0   -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Winit-self -Wno-narrowing -Wno-delete-non-virtual-dtor -Wno-comment -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -mfpu=neon -mfp16-format=ieee -fvisibility=hidden -fvisibility-inlines-hidden -O3 -DNDEBUG  -DNDEBUG
    C++ flags (Debug):           -DTBB_USE_GCC_BUILTINS=1 -D__TBB_64BIT_ATOMICS=0   -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Winit-self -Wno-narrowing -Wno-delete-non-virtual-dtor -Wno-comment -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -mfpu=neon -mfp16-format=ieee -fvisibility=hidden -fvisibility-inlines-hidden -g  -O0 -DDEBUG -D_DEBUG
    C Compiler:                  /usr/bin/cc
    C flags (Release):           -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Winit-self -Wno-narrowing -Wno-comment -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -mfpu=neon -mfp16-format=ieee -fvisibility=hidden -O3 -DNDEBUG  -DNDEBUG
    C flags (Debug):             -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Winit-self -Wno-narrowing -Wno-comment -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -mfpu=neon -mfp16-format=ieee -fvisibility=hidden -g  -O0 -DDEBUG -D_DEBUG
    Linker flags (Release):      
    Linker flags (Debug):        
    ccache:                      NO
    Precompiled headers:         YES
    Extra dependencies:          dl m pthread rt
    3rdparty dependencies:

  OpenCV modules:
    To be built:                 aruco bgsegm bioinspired calib3d ccalib core datasets dnn dnn_objdetect dpm face features2d flann freetype fuzzy hfs highgui img_hash imgcodecs imgproc java_bindings_generator line_descriptor ml objdetect optflow phase_unwrapping photo plot python2 python3 python_bindings_generator reg rgbd saliency shape stereo stitching structured_light superres surface_matching text tracking video videoio videostab xfeatures2d ximgproc xobjdetect xphoto
    Disabled:                    js world
    Disabled by dependency:      -
    Unavailable:                 cnn_3dobj cudaarithm cudabgsegm cudacodec cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev cvv hdf java matlab ovis sfm ts viz
    Applications:                apps
    Documentation:               NO
    Non-free algorithms:         NO

  GUI: 
    GTK+:                        YES (ver 3.22.11)
      GThread :                  YES (ver 2.50.3)
      GtkGlExt:                  NO
    VTK support:                 NO

  Media I/O: 
    ZLib:                        /usr/lib/arm-linux-gnueabihf/libz.so (ver 1.2.8)
    JPEG:                        /usr/lib/arm-linux-gnueabihf/libjpeg.so (ver 62)
    WEBP:                        build (ver encoder: 0x020e)
    PNG:                         /usr/lib/arm-linux-gnueabihf/libpng.so (ver 1.6.28)
    TIFF:                        /usr/lib/arm-linux-gnueabihf/libtiff.so (ver 42 / 4.0.8)
    JPEG 2000:                   /usr/lib/arm-linux-gnueabihf/libjasper.so (ver 1.900.1)
    OpenEXR:                     build (ver 1.7.1)
    HDR:                         YES
    SUNRASTER:                   YES
    PXM:                         YES

  Video I/O:
    DC1394:                      NO
    FFMPEG:                      YES
      avcodec:                   YES (ver 57.64.101)
      avformat:                  YES (ver 57.56.101)
      avutil:                    YES (ver 55.34.101)
      swscale:                   YES (ver 4.2.100)
      avresample:                NO
    GStreamer:                   
      base:                      YES (ver 1.10.4)
      video:                     YES (ver 1.10.4)
      app:                       YES (ver 1.10.4)
      riff:                      YES (ver 1.10.4)
      pbutils:                   YES (ver 1.10.4)
    libv4l/libv4l2:              1.12.3 / 1.12.3
    v4l/v4l2:                    linux/videodev2.h

  Parallel framework:            TBB (ver 4.4 interface 9005)

  Trace:                         YES (built-in)

  Other third-party libraries:
    Lapack:                      NO
    Eigen:                       YES (ver 3.3.2)
    Custom HAL:                  YES (carotene (ver 0.0.1))
    Protobuf:                    build (3.5.1)

  OpenCL:                        YES (no extra features)
    Include path:                /home/pi/opencv/opencv-3.4.3/3rdparty/include/opencl/1.2
    Link libraries:              Dynamic load

  Python 2:
    Interpreter:                 /usr/bin/python2.7 (ver 2.7.13)
    Libraries:                   /usr/lib/arm-linux-gnueabihf/libpython2.7.so (ver 2.7.13)
    numpy:                       /usr/lib/python2.7/dist-packages/numpy/core/include (ver 1.12.1)
    packages path:               lib/python2.7/dist-packages

  Python 3:
    Interpreter:                 /usr/bin/python3 (ver 3.5.3)
    Libraries:                   /usr/lib/arm-linux-gnueabihf/libpython3.5m.so (ver 3.5.3)
    numpy:                       /usr/lib/python3/dist-packages/numpy/core/include (ver 1.12.1)
    packages path:               lib/python3.5/dist-packages

  Python (for build):            /usr/bin/python2.7

  Java:                          
    ant:                         NO
    JNI:                         NO
    Java wrappers:               NO
    Java tests:                  NO

  Matlab:                        NO

  Install to:                    /usr/local
-----------------------------------------------------------------


After that is time to build. We can increase the swap space for being able to build with the 4 cores of the pi, and thus reducing considerably the build time. In my case this always works, but somehow adding tbb option to opencv causes segmentation fault even with the increased swap space, so I am doing it just on 1 core. The command time make indicates us the time it took at the end of the build:

time make

in my case it was more than an hour of build time. Lets now install it

sudo make install
sudo ldconfig

And that's it, we should be able to start coding.

NOTE: Usually the camera can be accesed on opencv with videocapture, indicating a (0) argument as for the default camera, the pi camera, the tipical videocapture cap(0). This worked for me before installing it without gstreamer, but for any reason, with gstreamer on the opencv build this 0 argument doesn't point to V4l2 or something, so it is needed to put instead "/dev/video0" or the name of the video indicated by  v4l2-ctl --list-devices  in case we have more cameras atached.

Feel free to ask any questions if you have trouble with this. I plan to add a prebuilt opencv package with all this for convenience, but I am having trouble with it, but I will keep trying and will upload it.




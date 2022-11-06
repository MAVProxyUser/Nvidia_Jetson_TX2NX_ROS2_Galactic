```
YAHBOOM JETSON TX2 NX DEVELOPER KIT with JetPack 4.6
https://category.yahboom.net/products/tx2-nx
https://www.amazon.com/Yahboom-Jetson-Development-N-VIDIA-Performance/dp/B09Y53TWQJ
https://developer.nvidia.com/embedded/jetpack-sdk-46

ROS2 + Realsense + RPLidar 

Switch to NVMe boot:

jetson@jetson-desktop:~/$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/mmcblk0p1   14G  5.1G  7.4G  41% /

jetson@jetson-desktop:~$ git clone https://github.com/jetsonhacks/rootOnNVMe.git
jetson@jetson-desktop:~$ cd rootOnNVMe/
jetson@jetson-desktop:~/rootOnNVMe$ ./copy-rootfs-ssd.sh 
jetson@jetson-desktop:~/rootOnNVMe$ ./setup-service.sh 
jetson@jetson-desktop:~/rootOnNVMe$ sudo reboot

jetson@jetson-desktop:~/resizeSwapMemory$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p1  117G  5.1G  106G   5% /

Upgrade Swap RAM:

jetson@jetson-desktop:~/$ free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.2G        1.9G         40M        663M        2.3G
Swap:          1.9G          0B        1.9G

jetson@jetson-desktop:~$ git clone https://github.com/JetsonHacksNano/resizeSwapMemory
jetson@jetson-desktop:~$ cd resizeSwapMemory/

jetson@jetson-desktop:~/resizeSwapMemory$ ./setSwapMemorySize.sh -g 16
jetson@jetson-desktop:~/rootOnNVMe$ sudo reboot

jetson@jetson-desktop:~$ free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.1G        2.1G         38M        542M        2.5G

Update apt, and upgrade the system, clean unused packages, and purge left overs:

jetson@jetson-desktop:~$ sudo apt-get update; sudo apt upgrade -y
jetson@jetson-desktop:~$ sudo apt autoremove -y
jetson@jetson-desktop:~$ sudo apt-get purge $(dpkg -l | grep '^rc' | awk '{print $2}') -y

Install a bunch of stuff for compiling ROS2:

jetson@jetson-desktop:~$ sudo apt-get install bison libzmq3-dev  libzmqpp-dev libczmq-dev  
libfreeimage-dev libfreeimageplus-dev openjdk-11-jre-headless libboost-program-options-dev 
libgvc6 libcgraph6 libcdt5 libgraphviz-dev liboctovis-dev bzip2 graphviz libprotobuf-dev 
libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev 
libiberty-dev autoconf llvm git build-essential qt5-default bison libacl1-dev curl gnupg2 
lsb-release build-essential git wget python3-rosdep libopencv-dev libzmq3-dev   
libgtest-dev cmake libacl1-dev libsqlite3-dev libtinyxml2-dev libbullet-dev   
libncurses-dev libbison-dev libcurl4-openssl-dev bison liblog4cxx-dev libeigen3-dev 
libasio-dev python3-dev libboost-all-dev libglib2.0-dev libprotobuf-dev   
libprotoc-dev protobuf-compiler protobuf-compiler-grpc libasound2-dev   
libconsole-bridge-dev libmpg123-dev libv4l-dev libssl-dev python3-pip libx11-dev libxext-dev 
libxtst-dev libxrender-dev libxmu-dev libxmuu-dev libgl1-mesa-dev libglu1-mesa-dev 
freeglut3-dev libboost-all-dev libeigen3-dev libflann-dev libglew-dev libpcap-dev 
libusb-1.0-0-dev libopenni-dev libopenni2-dev clang-format libqhull-dev  libpng-dev 
libxslt1-dev libxml2-dev nano libxrandr-dev libxinerama-dev libxcursor-dev 
git cmake libssl-dev freeglut3-dev libusb-1.0-0-dev pkg-config libgtk-3-dev libxaw7-dev python3-pyqt5 
sip-dev pyqt5-dev python3-sip python3-sip-dev pyqt5-dev-tools libglfw3 -y

jetson@jetson-desktop:~$ pip3 install Cython
jetson@jetson-desktop:~$ pip3 install vcstool argcomplete flake8 flake8-blind-except   
flake8-builtins flake8-class-newline flake8-comprehensions flake8-deprecated   
flake8-docstrings flake8-import-order flake8-quotes pytest-repeat pytest-rerunfailures   
pytest pytest-cov pytest-runner lark ifcfg netifaces boost numpy rosdep lxml 
colcon-common-extensions importlib-resources 

Manually compile VTK & PCL libs because Ubuntu packages are not new enough: 

jetson@jetson-desktop:~$ git clone https://github.com/Kitware/VTK.git -b v8.2.0
jetson@jetson-desktop:~$ cd VTK
jetson@jetson-desktop:~$ mkdir build; cd build
jetson@jetson-desktop:~$ make -DVTK_USE_SYSTEM_PNG=ON ..
jetson@jetson-desktop:~$ make -j4
jetson@jetson-desktop:~$ sudo make install 

jetson@jetson-desktop:~$ git clone https://github.com/PointCloudLibrary/pcl.git -b pcl-1.11.1
jetson@jetson-desktop:~$ cd pcl
jetson@jetson-desktop:~$ mkdir build; cd build
jetson@jetson-desktop:~$ cmake ..
jetson@jetson-desktop:~$ make -j4
jetson@jetson-desktop:~$ sudo make install 

Pulldown the Xiaomi ROS2 repo for Cyberdog (modified ROS2 galactic for Ubuntu 20, instead of Ubuntu 18):

jetson@jetson-desktop:~$ git clone https://github.com/MiRoboticsLab/cyberdog_ros2.git
jetson@jetson-desktop:~$ mkdir -p ros_src/src
jetson@jetson-desktop:~$ cp cyberdog_ros2/tools/ros2_fork/* ros_src
jetson@jetson-desktop:~$ cd ros_src
jetson@jetson-desktop:~/ros_src$ export PATH=$PATH:/home/jetson/.local/bin/
jetson@jetson-desktop:~/ros_src$ vcs import src < mini.repos

Fix a small error in the repo: 

jetson@jetson-desktop:~/ros_src$ cat > fix_extend.diff
diff --git a/extend.repos b/extend.repos
index 6ae5252..d7828e3 100644
--- a/extend.repos
+++ b/extend.repos
@@ -23,7 +23,7 @@ repositories:
     type: git
     url: https://github.com/ros-planning/navigation_msgs.git
     version: 2.1.0
-   ros-visualization/interactive_markers:
+  ros-visualization/interactive_markers:
     type: git
     url: https://github.com/
^C

jetson@jetson-desktop:~/ros_src$ cat fix_extend.diff | patch -p1
jetson@jetson-desktop:~/ros_src$ vcs import src < extend.repos

Set environment variables for the compile:

jetson@jetson-desktop:~/ros_src$ export ROS_VERSION=2
jetson@jetson-desktop:~/ros_src$ sudo mkdir -p /opt/ros2/galactic
jetson@jetson-desktop:~/ros_src$ sudo chown $USER /opt/ros2/galactic
jetson@jetson-desktop:~/ros_src$ export OPENBLAS_CORETYPE=ARMV8
jetson@jetson-desktop:~/ros_src$ colcon build --parallel-workers 20 --event-handlers console_direct+
...

When compile is complete is should have compiled 239 packages, ignore the stderr output. 

Summary: 239 packages finished [1h 35min 41s]
  59 packages had stderr output: ament_clang_format ament_clang_tidy ament_copyright ament_cppcheck ament_cpplint ament_flake8 ament_index_python ament_lint 
ament_lint_cmake ament_mypy ament_package ament_pclint ament_pep257 ament_pycodestyle ament_pyflakes ament_uncrustify ament_xmllint domain_coordinator 
examples_tf2_py foonathan_memory_vendor iceoryx_posh launch launch_ros launch_testing launch_testing_ros launch_xml launch_yaml mimick_vendor osrf_pycommon 
rcl_lifecycle rcutils ros2action ros2bag ros2cli ros2component ros2doctor ros2interface ros2launch ros2lifecycle ros2multicast ros2node ros2param ros2pkg ros2run 
ros2service ros2test ros2topic ros2trace rosidl_cli rosidl_runtime_py rpyutils sensor_msgs_py sros2 test_launch_ros tf2_ros_py tf2_tools tracetools_launch 
tracetools_read tracetools_trace

jetson@jetson-desktop:~/ros_src$ colcon list | wc -l
239

Compile libraries for the Intel Realsense: 

jetson@jetson-desktop:~$ git clone https://github.com/IntelRealSense/librealsense.git 
jetson@jetson-desktop:~$ cd librealsense/
jetson@jetson-desktop:~/librealsense$ ./scripts/patch-realsense-ubuntu-L4T.sh  
jetson@jetson-desktop:~/librealsense$ mkdir build
jetson@jetson-desktop:~/librealsense$ cd build/
jetson@jetson-desktop:~/librealsense/build$ cmake ../ -DFORCE_LIBUVC=true -DCMAKE_BUILD_TYPE=release
jetson@jetson-desktop:~/librealsense/build$ cmake .. -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=release -DFORCE_RSUSB_BACKEND=false -DBUILD_WITH_CUDA=false && make -j$(($(nproc)-1)) && sudo make install
jetson@jetson-desktop:~/librealsense/build$ make -j4 
jetson@jetson-desktop:~/librealsense/build$ sudo make install 

jetson@jetson-desktop:~/librealsense$ sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/
jetson@jetson-desktop:~/librealsense$ sudo udevadm control --reload-rules && udevadm trigger

jetson@jetson-desktop:~/librealsense/build$ rs-enumerate-devices 

Install ROS realsense 
jetson@jetson-desktop:~$ cd ~/ros_src/src/
jetson@jetson-desktop:~/ros_src/src$ git clone  https://github.com/IntelRealSense/realsense-ros.git
jetson@jetson-desktop:~/ros_src/src$ cd ../
jetson@jetson-desktop:~/ros_src$ colcon build --parallel-workers 20 --event-handlers console_direct+ --packages-skip-build-finished
(missing CV bridge!)
jetson@jetson-desktop:~/ros_src$ cd src/
jetson@jetson-desktop:~/ros_src/src$ git clone https://github.com/ros-perception/vision_opencv.git -b 2.2.1
jetson@jetson-desktop:~/ros_src/src$ git clone https://github.com/ros-perception/image_common.git -b galactic
jetson@jetson-desktop:~/ros_src/src$ git clone https://github.com/ros/diagnostics.git -b galactic 

jetson@jetson-desktop:~/ros_src/src$ colcon build --parallel-workers 20 --event-handlers console_direct+ --packages-skip-build-finished

Install RPLidar:

jetson@jetson-desktop:~$ cd ~/ros_src/src/
jetson@jetson-desktop:~/ros_src/src$ git clone https://github.com/Slamtec/rplidar_ros.git -b ros2
jetson@jetson-desktop:~/ros_src/src$ cd ../
jetson@jetson-desktop:~/ros_src/src$ colcon build --parallel-workers 20 --event-handlers console_direct+ --packages-skip-build-finished

Source the ros2 install!
jetson@jetson-desktop:~/ros_src$ . install/local_setup.bash

Lauch realsense cam:
jetson@jetson-desktop:~/ros_src$ realsense-viewer
jetson@jetson-desktop:~/ros_src$ ros2 launch realsense2_camera rs_launch.py

Launch RPLidar s1:
jetson@jetson-desktop:~/ros_src$ sudo chmod 777 /dev/ttyUSB0 
jetson@jetson-desktop:~/ros_src$ ros2 launch rplidar_ros2 rplidar_s1_launch.py

# jetson@jetson-desktop:~/ros_src$ ros2 launch rplidar_ros2 view_rplidar_s1_launch.py
```

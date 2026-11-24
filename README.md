# Shelf-Sweeping Camera Kit

This ROS 2 camera kit supplies the RGB image and colored 3D point cloud used by [RL-based Shelf Sweeping Deployment](https://github.com/JeonHaneul/RL-based-Shelf-Sweeping-Deployment) on the physical UR5e. A LUCID Triton captures the shelf image. A Helios2 measures 3D points, which the driver projects into that image to assign colors. The deployment uses the image to detect an object and the point cloud to estimate its position and width.

## ROS interfaces

| Topic | Message | Frame and contents |
| --- | --- | --- |
| `/triton/image_raw` | `sensor_msgs/msg/Image` | `triton_camera` · BGR8 image |
| `/triton/camera_info` | `sensor_msgs/msg/CameraInfo` | `triton_camera` · Triton intrinsics and distortion |
| `/helios/pointcloud_rgb` | `sensor_msgs/msg/PointCloud2` | `helios_camera` · XYZ in metres, packed RGB, constant intensity field |

The node reads the cameras sequentially and stamps each pair with the same ROS host time. Hardware exposure synchronization is unverified.

## Build the camera workspace

Use Ubuntu 22.04, ROS 2 Humble and the [LUCID Arena SDK for Linux x64](https://thinklucid.com/downloads-hub/). Install the SDK with its bundled setup instructions and verify that both cameras appear in the vendor discovery tool. The camera source and recorded calibration files are bundled. The SDK is installed separately.

```bash
git clone https://github.com/JeonHaneul/RL-based-Shelf-Sweeping-Camera-Kit.git
cd RL-based-Shelf-Sweeping-Camera-Kit
source /opt/ros/humble/setup.bash
sudo apt install python3-colcon-common-extensions python3-rosdep
# On a fresh ROS installation, run sudo rosdep init once.
rosdep update
rosdep install --from-paths src --ignore-src -r -y

# Set this to the SDK directory that contains include/Arena and lib64.
export ARENA_SDK_ROOT=/path/to/ArenaSDK_Linux_x64
colcon build --symlink-install --packages-select lucid_camera_node \
  --cmake-args -DARENA_SDK_ROOT="$ARENA_SDK_ROOT"
source install/setup.bash
```

The original hardware configuration selects `BGR8` on the Triton and `Coord3D_ABCY16` with `Distance8300mmMultiFreq` on the Helios2. Confirm these features are available on the connected camera model.

## Run and inspect

```bash
ros2 run lucid_camera_node lucid_node
```

In another terminal, source ROS 2 and the camera workspace, then inspect the streams.

```bash
ros2 topic hz /triton/image_raw
# Stop the rate display with Ctrl+C before running the next command.
ros2 topic echo /triton/camera_info --once
ros2 topic echo /helios/pointcloud_rgb --field header --once
```

The installed calibration files are measurements from the original rig. For a different camera pair, lens or mounting geometry, supply the corresponding Triton intrinsics and Helios-to-Triton registration. The driver projects XYZ in millimetres, so `translationVector` in the registration file must use millimetres.

```bash
ros2 run lucid_camera_node lucid_node --ros-args \
  -p orientation_file:=/absolute/path/to/orientation.yml \
  -p intrinsics_file:=/absolute/path/to/tritoncalibration.yml
```

## Connect the sweeping deployment

Run the camera node in its own terminal. In the deployment terminal, load both built workspaces and the deployment settings. For the original rig, the example below selects the registration file installed by the camera kit. With a new calibration, set `SWEEP_CAMERA_CALIBRATION` to the same file passed to `orientation_file`.

```bash
source /opt/ros/humble/setup.bash
source /path/to/RL-based-Shelf-Sweeping-Camera-Kit/install/setup.bash
cd /path/to/RL-based-Shelf-Sweeping-Deployment
source install/local_setup.bash
export SWEEP_WORKSPACE="$PWD"
source config/deployment.env  # configured from config/deployment.env.example
export SWEEP_CAMERA_CALIBRATION="$(ros2 pkg prefix --share lucid_camera_node)/calibration/orientation.yml"
```

The detection and pose-estimation commands, robot-base transforms and policy are documented in the [deployment guide](https://github.com/JeonHaneul/RL-based-Shelf-Sweeping-Deployment#vision-based-drl-execution). Camera startup only publishes sensor data. Simulation training is maintained in [RL-based Shelf Sweeping](https://github.com/JeonHaneul/RL-based-Shelf-Sweeping).

## Code layout

| Path | Purpose |
| --- | --- |
| [Driver](src/lucid_camera_node/src/lucid_node.cpp) | Discovers the two cameras, acquires images and publishes the registered cloud |
| [Calibration](src/lucid_camera_node/calibration/) | Active registration and Triton intrinsics. Earlier measurements are kept in `archive/` |
| [Package build](src/lucid_camera_node/CMakeLists.txt) | Locates the SDK and installs the executable and active calibration files |
| [Detection inspector](tools/inspect_detections.py) | Optional RViz markers and ROI cloud using the deployment's `/segmentation/bboxes` topic. Run with Python after sourcing both workspaces and installing `python3-opencv`, `python3-numpy` and `ros-humble-visualization-msgs` |
| [Source record](SOURCE.md) | Original driver revision and distribution maintainer |

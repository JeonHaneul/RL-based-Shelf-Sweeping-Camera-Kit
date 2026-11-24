# Source and maintenance

The camera driver, recorded calibration files and detection-inspection utility are adapted from Taeho Kim's [ROS2_Helios2_RGB_KIT at revision 6e0a5bd](https://github.com/Holiclife-KTH/ROS2_Helios2_RGB_KIT/tree/6e0a5bdd5f3cf7e4daa4b4cad651011c762e2607). The original ROS manifest does not specify a package license.

Haneul Jeon maintains this distribution for the [UR5e shelf-sweeping deployment](https://github.com/JeonHaneul/RL-based-Shelf-Sweeping-Deployment). The distribution packages calibration data for colcon installation, makes camera and SDK paths configurable, and connects the optional inspection utility to the deployment's bounding-box interface.

The deployment repository recorded a different camera revision, `11ce7938a7a1947c7211066186f1d75b37c3607c`. This kit uses the supplied archive revision identified above. Both the original and archived calibration sets are retained. The two files in `calibration/` are installed with the driver, and `calibration/archive/` contains older measurements.

The [LUCID Arena SDK](https://thinklucid.com/downloads-hub/) is a separate vendor dependency.

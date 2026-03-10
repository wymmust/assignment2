# AAE5303 Assignment – ORB-SLAM3 Monocular Visual Odometry

Framework: ORB-SLAM3
Mode: Monocular Visual Odometry
Dataset: HKIsland_GNSS03 (MARS-LVIG / UAVScenes)
Evaluation Tool: evo (Sim(3) alignment)


---

## 1.Executive Summary

This project demonstrates a complete visual SLAM evaluation workflow using ORB-SLAM3 on a real-world UAV dataset. The goal is to estimate the camera trajectory from monocular images and evaluate its accuracy against RTK ground truth.

The experiment was conducted on the HKisland_GNSS03 sequence from the MARS-LVIG / UAVScenes dataset, which contains synchronized camera and RTK measurements. ORB-SLAM3 was used to generate the visual odometry trajectory, and the result was evaluated using Sim(3) alignment with ground truth to compensate for scale ambiguity inherent in monocular SLAM.

After alignment, the estimated trajectory closely follows the ground truth path, demonstrating that ORB-SLAM3 can recover the overall structure of the UAV flight trajectory. However, noticeable deviations remain in several segments due to accumulated drift and visual tracking uncertainty.

### Key Results

| Metric | Value | 
|--------|-------|
| **ATE RMSE** | **20.77 m** |
| **RPE Trans Drift** | **0.3907 m/m** |
| **RPE Rot Drift** | **96.3766 deg/100m** | 
| **Completeness** | **94.73%** | 
| **Estimated poses** | 3705 | 

---

## 2. System Configuration

| Component | Specification |
|---|---|
| SLAM Framework | ORB-SLAM3 |
| Mode | Monocular VO |
| Vocabulary | ORBvoc.txt |
| Camera Config | DJI_Camera.yaml |
| OS | Ubuntu 22.04 |
| Evaluation Tool | evo |


## 3. Dataset Description

**Dataset:** HKIsland_GNSS03  
**Source:** MARS-LVIG UAV Dataset

This experiment uses the **HKIsland_GNSS03 sequence** from the MARS-LVIG UAV dataset. The dataset contains synchronized UAV sensor data collected in an outdoor urban environment and provides high-precision ground truth for trajectory evaluation.

### Sensors Available

The dataset includes multiple onboard sensors:

- **Monocular camera** – used as the primary input for ORB-SLAM3 monocular visual odometry  
- **GNSS / RTK** – used as ground truth for trajectory evaluation  
- **IMU** – available but not used in this experiment  
- **LiDAR** – available but not used in this experiment  

Only the **monocular camera stream** is used for pose estimation in this assignment.

### Ground Truth Source

Ground truth trajectory is obtained from the GNSS/RTK system and extracted from the following ROS topic:
/dji_osdk_ros/local_position
This provides accurate position measurements of the UAV during flight, which are used as the reference trajectory for evaluation.

### Timestamp Processing

To ensure correct trajectory alignment during evaluation:

- Original timestamps are recorded in **nanoseconds**
- They are converted to **seconds** for compatibility with the evaluation tools
- A **synchronization threshold of 0.5 seconds** is applied when matching estimated poses with ground truth

**AAE5303 - Robust Control Technology in Low-Altitude Aerial Vehicle**

*Department of Aeronautical and Aviation Engineering*

*The Hong Kong Polytechnic University*

Jan 2026

</div>


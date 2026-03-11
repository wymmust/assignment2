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

```
/dji_osdk_ros/local_position
```

This provides accurate position measurements of the UAV during flight, which are used as the reference trajectory for evaluation.

### Timestamp Processing

To ensure correct trajectory alignment during evaluation:

- Original timestamps are recorded in **nanoseconds**
- They are converted to **seconds** for compatibility with the evaluation tools
- A **synchronization threshold of 0.5 seconds** is applied when matching estimated poses with ground truth

## 4. Methodology

### 4.1 Running ORB-SLAM3

Monocular compressed node:

```
./Mono_Compressed Vocabulary/ORBvoc.txt config/DJI_Camera.yaml
```
Trajectory saved as:

```
SLAM.SaveTrajectoryTUM("CameraTrajectory.txt");
```

---

### 4.2 Ground Truth Processing

Steps:

1. Extract GNSS poses from rosbag  
2. Convert to TUM trajectory format  
3. Convert timestamps (ns to s)  
4. Synchronize with estimated trajectory  
5. Evaluate using evo

### 4.3 Evaluation Command

```
evo_ape tum ground_truth.txt CameraTrajectory.txt 
–align –correct_scale –t_max_diff 0.5
```

Parameters:

- Alignment: Sim(3) with scale correction  
- Delta distance for RPE: 10 m  
- Time threshold: 0.5 s  

---

## 5. Trajectory Alignment Statistics

| Parameter | Value |
|---|---|
| Alignment | Sim(3) with scale correction |
| Matched poses | 1852 / 1955 |
| Completeness | 94.73% |

## 6. Quantitative Results

| Metric | Value |
|---|---|
| ATE RMSE | 20.77 m |
| ATE Mean | 18.24 m |
| ATE Median | 16.66 m |
| RPE Translational RMSE | 4.10 m |
| Scale (Sim3) | 1.0386 |

---
## 7. Trajectory Visualization

The following figure summarizes the trajectory evaluation results of ORB-SLAM3 monocular visual odometry compared with the RTK ground truth.

<img width="3566" height="2970" alt="trajectory_evaluation_副本" src="https://github.com/user-attachments/assets/5e4b121e-e7e4-4927-ba56-8486771576b2" />


The visualization contains four subplots:

**1. 2D Trajectory – Before Alignment**

- Shows the estimated trajectory before alignment.
- Red dashed line: Visual odometry trajectory.
- Green line: Ground truth trajectory (RTK).
- The large offset is mainly caused by the scale ambiguity inherent in monocular SLAM.

**2. 2D Trajectory – After Sim(3) Alignment**

- The estimated trajectory is aligned with the ground truth using Sim(3) transformation.
- Blue line: Aligned visual odometry trajectory.
- Green line: Ground truth trajectory.
- After alignment, the overall trajectory shape matches well with the ground truth.

**3. Absolute Trajectory Error (ATE) Distribution**

- Shows the histogram of ATE errors.
- Mean ATE ≈ **18.24 m**
- Median ATE ≈ **16.66 m**
- Most errors are concentrated between **10 m – 30 m**, indicating moderate global drift.

**4. ATE Error Along Trajectory**

- Shows how ATE changes along the trajectory.
- Larger spikes correspond to challenging sections of the flight path where visual tracking becomes less stable.
- These errors are typically caused by motion blur, low-texture environments, or accumulated drift.

Overall, the visualization demonstrates that ORB-SLAM3 can recover the global trajectory structure, although some drift remains due to the monocular scale ambiguity and challenging UAV flight conditions.

## 8. Analysis of Output

The experimental results provide insight into the performance of ORB-SLAM3 when applied to UAV monocular visual odometry.

### 8.1 Trajectory Alignment

Before alignment, the estimated trajectory shows a clear scale and orientation mismatch compared with the ground truth. This is expected because monocular SLAM cannot directly observe the absolute scale of the environment. After applying Sim(3) alignment using the evo evaluation tool, the estimated trajectory becomes well aligned with the RTK ground truth. The overall trajectory structure and flight path are successfully recovered.

### 8.2 Absolute Trajectory Error (ATE)

The Absolute Trajectory Error (ATE) measures the global consistency of the estimated trajectory. In this experiment:

- ATE RMSE ≈ **20.77 m**
- Mean ATE ≈ **18.24 m**
- Median ATE ≈ **16.66 m**

Most of the error values fall within the **10–30 m range**, indicating that the system maintains a reasonable global estimate of the UAV trajectory. However, the presence of larger errors suggests that drift accumulates over time.

### 8.3 Relative Pose Error (RPE)

The Relative Pose Error reflects the local motion accuracy between consecutive frames. The translational RPE RMSE is approximately **4.10 m**, which indicates that the system is able to estimate short-term motion reasonably well. This shows that the local tracking of ORB-SLAM3 remains stable even though global drift occurs.

### 8.4 Error Distribution Along the Trajectory

From the ATE error plot along the trajectory, several peaks can be observed. These peaks correspond to segments where the visual tracking becomes more challenging. Possible causes include:

- Motion blur caused by fast UAV movement
- Low-texture environments such as roads or open areas
- Temporary tracking loss or feature scarcity

These factors lead to larger deviations between the estimated trajectory and the ground truth.

### 8.5 Overall Performance

Overall, ORB-SLAM3 successfully reconstructs the general flight trajectory of the UAV. The system demonstrates strong local tracking capability but still suffers from global drift, which is a common limitation of monocular visual SLAM systems. Despite these challenges, the results show that ORB-SLAM3 is capable of producing a reliable trajectory estimate for aerial navigation tasks.

**AAE5303 - Robust Control Technology in Low-Altitude Aerial Vehicle**

*Department of Aeronautical and Aviation Engineering*

*The Hong Kong Polytechnic University*

Jan 2026

</div>


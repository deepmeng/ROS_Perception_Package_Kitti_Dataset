SARosPerceptionKitti
=================
 
ROS package for the Perception (Sensor Processing, Detection, Tracking and Evaluation) of the KITTI Vision Benchmark 

### Demo
<p align="center">
  <img src="./videos/semantic.gif">
</p>

<p align="center">
  <img src="./videos/rviz.gif">
</p>

### Usage

1) Clone this repository, [download a preprocessed scenario](https://drive.google.com/drive/folders/1vHpkoC78fPXT64-VFL1H5Mm1bdukK5Qz?usp=sharing), and stick to following structure:  
 
```
    ~                                        # Home directory
    ├── catkin_ws                            # Catkin workspace
    │   ├── src                              # Clone repo in here
    │       └── SAROSPerceptionKitti         # Repo
    ├── kitti_data                           # Data folder
    │   ├── 0001                             # Scenario 0001
    │   ├── ...                              # Any other scenario
    │   ├── 0060                             # Demo scenario 0060
    │   │   ├── segmented_semantic_images    # Folder for semantic images (Copy download in here)
    │   │   │   ├── 0000000000.png           # Semantic image from first time frame
    │   │   │   ├── 0000000001.png           # Semantic image from second time frame
    │   │   │   └── ...  
    │   │   └── synchronized_data.bag        # Synchronized ROSbag file
    │   ├── ...
```

2) Go into `sensor_processing/src/sensor_processing_lib/sensor_fusion.cpp` and hardcode your home directory within the method `processImage()`!

3) Launch one of the following ROS nodes:  

```
roslaunch sensor_processing sensor_processing.launch
roslaunch detection detection.launch
roslaunch tracking tracking.launch
```

   * Default parameters: 
        * scenario:=0060  
        * speed:=0.25  
        * delay:=3  

Without assigning any of the abovementioned parameters the scenario 0060 (which is the demo above) is replayed at 25% of its speed with a 3 second delay so RViz has enough time to boot up.   

### Results

Evaluation results for 7 Scenarios `0011,0013,0014,0018,0056,0059,0060`

| Class        |  MOTP   |  MODP   |
| ------------ |:-------:|:-------:|
| Car          | 0.715273| 0.785403|
| Pedestrian   | 0.581809| 0.988038|

### Area for Improvements

* Friendly solution to not hard code the user's home directory path
* Record walk through video of entire project
* Find a way to run multiple scenarios with one execution
* Improving the Object Detection:  
     * Visualize Detection Grid
     * Incorporate features of the shape of cars
     * Handle false classification within the semantic segmentation
     * Replace MinAreaRect with better fitting of the object's bounding box
     * Integrate view of camera image to better group clusters since point clouds can be spare for far distances
* Improving the Object Tracking:
     * Delete duplicated tracks
     * Soften yaw estimations
* Improve evaluation
     * Write out FP FN
* Try different approaches:
     * Applying the VoxelNet

### Contact

If you have any questions, things you would love to add or ideas how to actualize the points in the Area of Improvements, send me an email at simonappel62@gmail.com ! More than interested to collaborate and hear any kind of feedback.

### Troubleshooting

* Make sure to close RVIz and restart the ROS launch command if you want to execute the scenario again. Otherwise it seems like the data isn't moving anymore ([see here](https://github.com/appinho/SARosPerceptionKitti/issues/7))
* Semenatic images warning: Go to sensor.cpp line 543 in sensor_processing_lib and hardcode your personal home directory! ([see full discussion here](https://github.com/appinho/SARosPerceptionKitti/issues/10))
* Make sure the scenario is encoded as 4 digit number, like above `0060`
* Make sure the images are encoded as 10 digit numbers starting from `0000000000.png`
* Make sure the resulting semantic segmentated images have the color encoding of the [Cityscape Dataset](https://www.cityscapes-dataset.com/examples/)

### Dependencies

1) [Install ROS Kinetic on Ubuntu 16.04](http://wiki.ros.org/kinetic/Installation/Ubuntu)
2) [Setup ROS Workspace](http://wiki.ros.org/catkin/Tutorials/create_a_workspace):  
```
mkdir -p ~/catkin_ws/src  
cd ~/catkin_ws/src  
git pull https://github.com/appinho/SARosPerceptionKitti.git  
cd ..  
catkin_make  
source devel/setup.bash  
```

### DIY: Data generation


1) [Install Kitti2Bag](https://github.com/tomas789/kitti2bag)

```
pip install kitti2bag
```

2) Convert scenario `0060` into a ROSbag file:  

    * Download and unzip the `synced+rectified data` file and its `calibration` file from the [KITTI Raw Dataset](http://www.cvlibs.net/datasets/kitti/raw_data.php)
    * Merge both files into one ROSbag file

```
cd ~/kitti_data/
kitti2bag -t 2011_09_26 -r 0060 raw_synced
```

3) Synchronize the sensor data:  

    * The script matches the timestamps of the Velodyne point cloud data with the camara data to perform Sensor Fusion in a synchronized way within the ROS framework 
```
cd ~/catkim_ws/src/ROS_Perception_Kitti_Dataset/pre_processing/
python sync_rosbag.py raw_synced.bag
```

4) Store preprocessed semantic segmentated images:  

    * The camera data is preprocessed within a Deep Neural Network to create semantic segmentated images. With this step a "real-time" performance on any device (CPU usage) can be guaranteed

```
mkdir ~/kitti_data/0060/segmented_semantic_images/
cd ~/kitti_data/0060/segmented_semantic_images/
```

   * For any other scenario follow this steps: Well pre-trained network with an IOU of 73% can be found here: [Finetuned Google's DeepLab on KITTI Dataset](https://github.com/hiwad-aziz/kitti_deeplab)

<!--

### To Do

* Make smaller gifs
* Double check
  * transformation from camera 02 to velo
  * grid to point cloud has any errors
* Reduce street pavement error prone cells
* Objects to free space or not

## Evaluation for 7 Scenarios 0011,0013,0014,0018,0056,0059,0060

| Class        | MOTA    | MOTP    |  MOTAL  |    MODA |    MODP |
| ------------ |:-------:|:-------:|:-------:|:-------:|:-------:|
| CAR          | 0.250970| 0.715273| 0.274552| 0.274903| 0.785403|
| PEDESTRIAN   |-0.015038| 0.581809|-0.015038|-0.015038| 0.988038|


[157, 154, 280, 306, 378, 1283, 17]
[64, 10, 10, 72, 11, 196, 0]
[39, 75, 120, 39, 33, 569, 0]

[8, 0, 1, 0, 4, 18, 18]
[3, 0, 2, 0, 0, 52, 0]
[172, 0, 63, 0, 25, 177, 46]

## Pipeline

### 1a) Sensor Fusion: Velodyne Point Cloud Processing

* [Ground extraction & Free space estimation](http://wiki.ros.org/but_velodyne_proc)

### 1b) Sensor Fusion: Raw Image Processing

* [Semantic segmentation](https://github.com/martinkersner/train-DeepLab)

### 1c) Sensor Fusion: Mapping Point Cloud and Image

### 2 Detection: DBSCAN Clustering

### 3 Tracking: UKF Tracker


Video image linker example

[![Segmentation illustration](https://img.youtube.com/vi/UXHX9kFGXfg/0.jpg)](https://www.youtube.com/watch?v=UXHX9kFGXfg "Segmentation")


-->

## License

With the standards of the [MIT License](LICENSE.md) you are more than welcomed to contribute to this repository!

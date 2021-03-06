## Implementing LSD-SLAM, ORB-SLAM2, and LDSO with a custom dataset

### Implementation instructions of paper "Mapping Quality Evaluation of Monocular SLAM Solutions for Micro Aerial Vehicles". 

[[Implementation]](http://jiewang.name/publications/slam2019)
[[Video]](http://jiewang.name/files/slam2019.gif)
[[Paper]](https://www.int-arch-photogramm-remote-sens-spatial-inf-sci.net/XLII-2-W17/413/2019/)

There many existing visual SLAM benchmark datasets (e.g., EuRoC, TUM, and KITTI) to implement tests of SLAM algorithms. 
However, other than the above three popular datasets, implementing SLAM algorithms with a custom dataset needs some configuration work. LSD-SLAM, ORB-SLAM2, and LDSO are modified as follows to run with a custom image dataset created from videos collect by flying an AR. Drone 2.0. 

### Custom dataset

The custom dataset is created following the format of the `EuRoC` dataset. The images are sorted sequentially and stored in the `data` folder; a `data.csv` is created containing the timestamp and image filename information. 

0. Record videos or ROS bag files. 
   * Extract image frames from videos: `ffmpeg -i xxx.mp4 -qscale:v 2 %06d.png`
   * Extract image frames from ROS bags: [rosbag2video](https://github.com/mlaiacker/rosbag2video) 

The created [custom dataset](https://github.com/jwangjie/LSD_SLAM-ORB_SLAM2-LDSO-ARDrone/tree/master/example_dataset) should look like this:

````
YOURDATASET
├── data.csv
├── data
│   ├── 000001.png
│   ├── 000002.png
│   ├── 000003.png
│   ├── ... 
````

### [LSD-SLAM](https://github.com/tum-vision/lsd_slam)

The camera is calibrated with the input image dimension width*height (640*360). Because the height must to be a multiple of 16 to implement LSD-SLAM, the image dimension is “crop” to 640*352. The following file format works well:

```
562.87794/640 562.21464/360 327.27636/640 189.97880/360 0
640 360
crop
640 352
```

1. Run LSD-SLAM: `rosrun lsd_slam_core dataset _files:=/xxx/data _hz:=30 _calib:=/pinhole_ardrone_calib.cfg`.
2. Launch the lsd_slam viewer: `rosrun lsd_slam_viewer viewer`.
3. The generated point cloud map can be saved by entering “p” for a few seconds continuously when the `lsd_slam_viewer` window is selected. The point cloud map is saved as a `pc.ply` file in the `lsd_slam_viewer` folder.

The codes of lsd_slam :
````
lsd_slam
├── ...
├── ...
├── lsd_slam_core
│   ├── calib
│   │   ├── pinhole_ardrone_calib.cfg (new)
│   |   ├── ...
├── ...
````
---

### [ORB-SLAM2](https://github.com/raulmur/ORB_SLAM2)

It is straightforward to evaluate the ORB-SLAM algorithm by provided commands provided for all three datasets. To apply ORB-SLAM to a custom test dataset, the dataset is created following the format of the `EuRoC` dataset. The images are sorted sequentially and stored in the `data` folder; a `data.csv` is created containing the timestamp and image filename information. The “EuRoC.yaml” is modified to specify the camera calibration parameters of the AR. Drone camera and renamed as `ardrone.yaml`. A image timestamp file is also created as `ardrone.txt` and placed in the `EuRoC_Time_Stamps` folder. The following codes are modified or created to output generated point clouds as a file: `System.cc`, `System.h`, `CMakeLists.txt`, and `mono_ardrone_pcl.cc`. 

Run ORB-SLAM2  with the created dataset: `./Examples/Monocular/mono_euroc_pcl Vocabulary/ORBvoc.txt Examples/Monocular/ardrone.yaml /xxx/data Examples/Monocular/EuRoC_TimeStamps/ardrone.txt`.
   * A point cloud file is saved as `pointcloud.pcd` in the `ORB_SLAM2` folder.

The codes of orb_slam2:
````
orb_slam2
├── CMakeLists.txt (modified)
├── ...
├── src
│   ├── System.cc (modified)
│   ├── ...
├── include
│   ├── System.h (modified)
│   ├── ...
├── ...
├── ...
├── Examples
│   ├── Monocular
│   │   ├── ardrone.yaml (new)
│   │   ├── mono_ardrone_pcl.cc (new)
│   │   ├── mono_ardrone_pcl (new generated excutable)
│   │   ├── ...
│   │   ├── EuRoC_Time_Stamples
│   │   |   ├── ardrone.txt (new)
````

---

### [LDSO](https://github.com/tum-vision/LDSO) 

Similar to ORB-SLAM, you can run LDSO algorithm by the provided commands with all three datasets. 
Your custom image dataset should be created similar to the EuRoC dataset.

1. Modify the camera calibration parameters of the `EUROC.txt` in `LDSO/examples/EUROC`. 
   * Following the [Radio-Tangential camera model](https://github.com/JakobEngel/dso#calibration-file-for-radio-tangential-camera-model), the calibration file is configured with format as follows:
   ```
   RadTan fx/in_width fy/in_height (cx+0.5)/in_width (cy+0.5)/in_height k1 k2 r1 r2
   in_width in_height
   "crop" / "full"  
   out_width out_height
   ```
   * Note: DSO requires a global shutter camera to achieve an acceptable results. The `EUROC.txt` in my LDSO is for Intel          Realsense D435 camera rather than AR. Drone camera. 
   
2. Run LDSO  with the created dataset: `./bin/run_dso_euroc preset=0 files=/YOURDATASET`
   * When the package finished running, the generated points cloud map will be automatically saved as the `pointcloud.ply` 
   file after shutting down the `Pangolin` window. 

## Please cite this paper in your publications if it helps your research:
```
@article{wang2019mapping,
  title={Mapping Quality Evaluation of Monocular SLAM Solutions for Micro Aerial Vehicles.},
  author={Wang, J. and Shahbazi, M.},
  journal={ISPRS - International Archives of the Photogrammetry, Remote Sensing and Spatial Information Sciences},
  volume = {XLII-2/W17},
  year={2019},
  pages = {413--420},
  URL = {https://www.int-arch-photogramm-remote-sens-spatial-inf-sci.net/XLII-2-W17/413/2019/},
  DOI = {10.5194/isprs-archives-XLII-2-W17-413-2019} 
}

```   

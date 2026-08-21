# Installation and Usage Instructions

## 1. RealSense D435i Configuration

Refer to the following tutorial to install the RealSense ROS driver:

https://developer.aliyun.com/article/1303849

After completing the driver installation according to the tutorial, make sure that the color image, depth image, camera parameters, and other ROS topics can be published correctly.

---

## 2. Livox MID360 Configuration

Refer to the following tutorials to install the MID360 ROS driver:

https://zhuanlan.zhihu.com/p/668966629

https://blog.csdn.net/2402_82745259/article/details/142638032

After completing the installation, make sure that the point cloud data can be published correctly.

---

## 3. LV-DOT Human Detection System Installation

https://github.com/Zhefan-Xu/LV-DOT

Follow the instructions in the repository to complete the environment configuration and compilation.

For the `git clone` part, replace the original workspace with the `detection_ws` workspace.

After completion, run the detection system and make sure that the human 3D Bounding Boxes can be displayed correctly in RViz.

Once the detection module can stably output human 3D Bounding Boxes, you can continue installing the subsequent modules.

---

## 5. Add the Re-Identification Library

```bash
mkdir ~/libs && cd ~/libs

wget http://dlib.net/files/dlib-19.22.tar.bz2
tar xvf dlib-19.22.tar.bz2

echo "export DLIB_ROOT=~/libs/dlib-19.22" >> ~/.bashrc
source ~/.bashrc
```

---

## 6. System Launch and Verification

Enter the ROS2 workspace:

```bash
cd ~/detection_ws
```

Launch the system:

```bash
ros2 launch onboard_detector test.launch.py
```

After the system starts successfully, the following should be visible in **RViz**:

* LiDAR point cloud
* Camera-based human detection results
* Fused human 3D Bounding Boxes
* An independent ID for each 3D Bounding Box
* Re-Identification module results

### Re-Identification Module Verification

After the system starts, the Re-Identification module automatically selects the **nearest person directly in front of the robot** as the initial tracking target.

Once the target is successfully selected, a **green bounding box** will appear, indicating that the target selection has been completed.

The system will then continuously track the target based on its ID.

> If all of the above results can be displayed correctly, the system is running normally and you can continue with subsequent experiments.

---

## 7. System Architecture

The system mainly consists of three ROS2 functional modules:

1. **`onboard_detector`**: Human 3D Bounding Box generation
2. **`robotpose`**: Target ID assignment and 3D → 2D projection
3. **Re-Identification Module**: Target selection, target loss detection, and re-identification

---

## 8. `onboard_detector`

`onboard_detector` is mainly responsible for fusing **YOLO human detection results, Livox MID360 point cloud data, and robot pose information** to generate the final human **3D Bounding Boxes**.

### Main Topics

| Topic                                    | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| `/yolo_detector/detected_bounding_boxes` | Human 2D Bounding Boxes detected by YOLO |
| `/pointcloud`                            | Livox MID360 point cloud data            |
| `/mavros/local_position/pose`            | Robot pose information                   |
| `/onboard_detector/dynamic_bboxes`       | Final human 3D Bounding Boxes            |

---

## 9. `robotpose`

`robotpose` is mainly responsible for:

* Assigning an independent ID to each 3D Bounding Box
* Projecting the 3D Bounding Boxes onto the 2D camera image
* Obtaining the distance information between the target and the robot

### Main Topics

| Topic                            | Description                                                       |
| -------------------------------- | ----------------------------------------------------------------- |
| `/ab3dmot/tracks_array`          | Assigns and maintains an independent ID for each 3D Bounding Box  |
| `/projected_boxes_with_distance` | Outputs the projected 2D Box, target ID, and distance information |

### Data Processing Flow

Therefore, different human 3D Bounding Boxes can be displayed with their own independent target IDs in RViz.

---

## 10. Re-Identification Module

The Re-Identification module is mainly responsible for **automatically selecting the tracking target and finding the original target again after it is lost**.

### ① Initial Target Selection

After the system starts, it automatically selects:

> **The nearest person directly in front of the robot**

as the initial tracking target.

After the target is successfully selected, a **green bounding box** will be displayed.

### ② Continuous Tracking

After the target is determined, the system records the target ID:

```text
/tracking_target_id
```

The system then continuously obtains the target's position and distance information based on this ID.

### ③ Target Loss

During actual tracking, the following situations may occur:

* The person is occluded by other pedestrians
* The target temporarily leaves the camera's field of view
* The AB3DMOT target ID is lost or changes

In these cases, the system enters the target Re-Identification stage.

### ④ Target Re-Identification

The system searches among the candidate IDs currently detected in the image and finds the person most similar to the original tracking target.

After a successful match, the system obtains the target ID again and resumes tracking.

---

## 11. Main Topics Overview

| Topic                                    | Module              | Description                       |
| ---------------------------------------- | ------------------- | --------------------------------- |
| `/yolo_detector/detected_bounding_boxes` | YOLO                | Human 2D detection Bounding Boxes |
| `/pointcloud`                            | LiDAR               | MID360 point cloud data           |
| `/mavros/local_position/pose`            | Localization        | Robot pose                        |
| `/onboard_detector/dynamic_bboxes`       | onboard_detector    | Human 3D Bounding Boxes           |
| `/ab3dmot/tracks_array`                  | robotpose / AB3DMOT | 3D Box ID tracking results        |
| `/projected_boxes_with_distance`         | robotpose           | 2D Box, ID, and distance          |
| `/tracking_target_id`                    | Re-ID               | Current tracking target ID        |

中文：

# 安装与运行说明

## 1. RealSense D435i 配置

参考以下教程安装 RealSense ROS 驱动：

[https://blog.csdn.net/youlanshengmeng/article/details/125334427](https://developer.aliyun.com/article/1303849)

按照教程完成驱动安装后，确认能够正常发布彩色图像、深度图像以及相机参数等 ROS Topic。

---

## 2. Livox MID360 配置

参考以下仓库安装 MID360 ROS 驱动：
https://zhuanlan.zhihu.com/p/668966629
https://blog.csdn.net/2402_82745259/article/details/142638032

完成安装后，确认能够正常发布点云数据。

---

## 3. LV-DOT 人体检测系统安装


https://github.com/Zhefan-Xu/LV-DOT

按照仓库中的说明完成环境配置与编译。

对于其中的内容有修改其中git clone 那个 更换成detection_ws 文件

完成后运行检测系统，确认能够在 RViz 中正常显示人体三维检测框（3D Bounding Box）。

当检测模块能够稳定输出人体三维目标框后，即可继续安装后续模块。

---

## 5. 添加重识别库

mkdir ~/libs && cd ~/libs

wget http://dlib.net/files/dlib-19.22.tar.bz2
tar xvf dlib-19.22.tar.bz2

echo "export DLIB_ROOT=~/libs/dlib-19.22" >> ~/.bashrc
source ~/.bashrc



## 6. 系统运行与验证

进入 ROS2 工作空间：

```bash
cd ~/detection_ws
```

启动系统：

```bash
ros2 launch onboard_detector test.launch.py
```

运行成功后，在 **RViz** 中应能够观察到：

* LiDAR 点云
* 摄像头人体检测结果
* 融合后的三维人体检测框（3D Bounding Box）
* 每个 3D Bounding Box 对应的独立 ID
* 重识别模块运行结果

### 重识别模块验证

系统启动后，重识别模块会自动选择**机器人正前方距离最近的人**作为初始跟踪目标。

当目标人物被成功选择后，画面中会出现**绿色目标框**，表示目标选择完成。

随后系统会根据该目标的 ID 持续进行跟踪。

> 当以上内容均能够正常显示时，说明系统运行正常，可以继续进行后续实验。

---

## 7. 系统架构

本系统主要由三个 ROS2 功能模块组成：

1. **`onboard_detector`**：人体 3D Bounding Box 生成
2. **`robotpose`**：目标 ID 分配及 3D → 2D 投影
3. **重识别模块（Re-Identification）**：目标选择、目标丢失检测及重新识别


## 8. `onboard_detector`

`onboard_detector` 主要负责融合 **YOLO 人体检测结果、Livox MID360 点云以及机器人位姿信息**，最终生成人体的 **3D Bounding Box**。

### 主要 Topic

| Topic                                    | 作用                           |
| ---------------------------------------- | ---------------------------- |
| `/yolo_detector/detected_bounding_boxes` | YOLO 检测得到的人体 2D Bounding Box |
| `/pointcloud`                            | Livox MID360 点云数据            |
| `/mavros/local_position/pose`            | 机器人位姿信息                      |
| `/onboard_detector/dynamic_bboxes`       | 最终生成的人体 3D Bounding Box      |



## 9. `robotpose`

`robotpose` 主要负责：

* 为每个 3D Bounding Box 分配独立 ID
* 将 3D Bounding Box 投影到相机二维图像
* 获取目标与机器人之间的距离信息

### 主要 Topic

| Topic                            | 作用                             |
| -------------------------------- | ------------------------------ |
| `/ab3dmot/tracks_array`          | 为每个 3D Bounding Box 分配并维护独立 ID |
| `/projected_boxes_with_distance` | 输出投影后的 2D Box、目标 ID 以及距离信息     |

### 数据处理流程


因此，在 RViz 中可以看到不同的人体 3D Bounding Box 具有各自独立的目标 ID。

---

## 10. 重识别模块（Re-Identification）

重识别模块主要负责**自动选择跟踪目标，并在目标丢失后重新找到原目标**。

### ① 初始目标选择

系统启动后，会自动选择：

> **机器人正前方距离最近的人**

作为初始跟踪目标。

目标选择成功后，画面中会显示**绿色目标框**。

### ② 持续跟踪

目标确定后，系统记录该目标的 ID：


/tracking_target_id
```

随后根据该 ID 持续获取目标的位置和距离信息。

### ③ 目标丢失

在实际跟踪过程中，可能出现：

* 人体被其他行人遮挡
* 目标暂时离开摄像头视野
* AB3DMOT 的目标 ID 丢失或发生变化

此时系统进入目标重识别阶段。

### ④ 目标重识别

系统会从当前画面中已经检测到的候选 ID 中，寻找与原跟踪目标最相似的人。

匹配成功后重新获得目标 ID，并恢复跟踪。



## 11. 主要 Topic 总览

| Topic                                    | 模块                  | 作用                 |
| ---------------------------------------- | ------------------- | ------------------ |
| `/yolo_detector/detected_bounding_boxes` | YOLO                | 人体 2D 检测框          |
| `/pointcloud`                            | LiDAR               | MID360 点云数据        |
| `/mavros/local_position/pose`            | Localization        | 机器人位姿              |
| `/onboard_detector/dynamic_bboxes`       | onboard_detector    | 人体 3D Bounding Box |
| `/ab3dmot/tracks_array`                  | robotpose / AB3DMOT | 3D Box ID 跟踪结果     |
| `/projected_boxes_with_distance`         | robotpose           | 2D Box、ID 和距离      |
| `/tracking_target_id`                    | Re-ID               | 当前跟踪目标 ID          |





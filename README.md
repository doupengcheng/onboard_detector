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

对于其中的内容有修改其中git clone 那个 更换成https://github.com/doupengcheng/onboard_detector.git

完成后运行检测系统，确认能够在 RViz 中正常显示人体三维检测框（3D Bounding Box）。

当检测模块能够稳定输出人体三维目标框后，即可继续安装后续模块。

---

## 5. tian jia biyao wen jian 
1. xuyao an zhuang chong shi bie xi tong suo xu yao yi lia de jianjianku 





2.


## 6. 验证检测结果
cd you workspace
ros2 launch onboard_detector test.launch.py 
运行成功后，应能够在 RViz 中观察到：

* LiDAR 点云
* 摄像头检测结果
* 融合后的三维人体检测框（3D Bounding Box）
* mei yige 3d box zai rviz zhong neng gou you zi ji de du li id
* yan zheng chong shibie mo kuai shou xian chongshi bie bufen keyi zidong xuan ze zheng qian fang zui jin de ren zuo wei genzong de mubiao (you lu se kuang de shihou dai bian xuanze wanbi)
  能够正常显示后，说明正常，可以继续


## 7. topic jie du 
<img width="1920" height="1080" alt="Screenshot from 2026-08-21 16-23-27" src="https://github.com/user-attachments/assets/9e43d3a4-a96a-4f99-81e6-ad4d4845ab16" />
系统说明

本系统主要分为三个 ROS2 功能模块：onboard_detector、robotpose 和重识别模块。

1. onboard_detector

onboard_detector 主要负责人体 3D Bounding Box 的生成。

主要使用以下 Topic：

Topic	作用
/yolo_detector/detected_bounding_boxes	YOLO 检测得到的人体 2D Box
/pointcloud	Livox MID360 点云信息
/mavros/local_position/pose	机器人位姿信息
/onboard_detector/dynamic_bboxes	最终生成的人体 3D Box

处理流程：

YOLO 2D Box + MID360 点云 + 机器人位姿
                    ↓
             onboard_detector
                    ↓
              人体 3D Box
2. robotpose

robotpose 主要负责目标 ID 分配以及 3D Box 到 2D Box 的转换。

主要 Topic：

Topic	作用
/ab3dmot/tracks_array	为每个 3D Box 分配独立 ID
/projected_boxes_with_distance	将 3D Box 投影到 2D 图像，并提供 ID 和距离信息

处理流程：

人体 3D Box
    ↓
AB3DMOT
    ↓
分配 ID
    ↓
3D → 2D 投影
    ↓
2D Box + ID + Distance
3. 重识别模块

重识别模块主要负责自动选取和恢复跟随目标。

系统首先自动选择目标人物，并根据目标 ID 持续获取目标位置。

当目标由于遮挡、暂时消失或 ID 丢失而无法继续跟踪时，系统会从当前画面中已有的 ID 中寻找与原目标最相似的人，并重新恢复跟随。

自动选择目标
     ↓
根据 ID 持续跟踪
     ↓
目标丢失
     ↓
从当前 ID 中进行重识别


/tracking_target_id
     ↓
找到原目标
     ↓
恢复跟随

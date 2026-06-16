# 安装与运行说明

## 1. RealSense D435i 配置

参考以下教程安装 RealSense ROS 驱动：

https://blog.csdn.net/youlanshengmeng/article/details/125334427

按照教程完成驱动安装后，确认能够正常发布彩色图像、深度图像以及相机参数等 ROS Topic。

---

## 2. Livox MID360 配置

参考以下仓库安装 MID360 ROS 驱动：

https://github.com/larics/livox_mid_360_ros_driver

完成安装后，确认能够正常发布点云数据。

---

## 3. LV-DOT 人体检测系统安装

下载并安装 LV-DOT：

https://github.com/Zhefan-Xu/LV-DOT

按照仓库中的说明完成环境配置与编译。

完成后运行检测系统，确认能够在 RViz 中正常显示人体三维检测框（3D Bounding Box）。

当检测模块能够稳定输出人体三维目标框后，即可继续安装后续模块。

---

## 5. 修改 run_detector.launch

在运行检测系统之前，请先注释以下节点：

```xml
<node pkg="pose_pub_33hz" type="pose_pub" name="pose_pub" output="screen" />

<node pkg="pose_pub_33hz" type="3dto2d"
      name="node_3dto2d"
      output="screen" />

<node pkg="ab3dmot_test"
      type="idmake.py"
      name="idmake"
      output="screen" />

<node pkg="object_get"
      type="ccf_test1"
      name="ccf_test1"
      output="screen" />
```

例如：

```xml
<!--
<node pkg="pose_pub_33hz" type="pose_pub" name="pose_pub" output="screen" />

<node pkg="pose_pub_33hz" type="3dto2d"
      name="node_3dto2d"
      output="screen" />

<node pkg="ab3dmot_test"
      type="idmake.py"
      name="idmake"
      output="screen" />

<node pkg="object_get"
      type="ccf_test1"
      name="ccf_test1"
      output="screen" />
-->
```

保存后运行：

```bash
roslaunch onboard_detector run_detector.launch
```

---

## 6. 验证检测结果

运行成功后，应能够在 RViz 中观察到：

* LiDAR 点云
* 摄像头检测结果
* 融合后的三维人体检测框（3D Bounding Box）

当人体三维检测框能够正常显示后，说明检测模块运行正常，可以继续

完成后续人体跟随系统的部署。

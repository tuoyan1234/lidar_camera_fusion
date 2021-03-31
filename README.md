# lidar_camera_fusion

ROS package for lidar and camera fusion

## 安装
 - 建立工作空间并拷贝这个库
   ```Shell
   mkdir -p ros_ws/src
   cd ros_ws/src
   git clone https://github.com/shangjie-li/lidar_camera_fusion.git
   cd ..
   catkin_make
   ```

## 参数配置
 - 修改`lidar_camera_fusion/launch/lidar_camera_fusion.launch`
   ```Shell
   <param name ="image_topic" value="/usb_cam/image_rect_color"/>
   <param name ="lidar_topic" value="/velodyne_points"/>
   <param name ="calibration_file_path" value="$(find lidar_camera_fusion)/conf/head_camera.yaml"/>
		
   <param name="the_view_number" value="1"/>
   <param name="the_field_of_view" value="100"/>
		
   <param name ="the_sensor_height" value="1.0"/>
   <param name ="the_view_higher_limit" value="2.0"/>
   <param name ="the_view_lower_limit" value="-2.0"/>
   <param name ="the_min_distance" value="1.0"/>
   <param name ="the_max_distance" value="100.0"/>

   <param name ="jet_color" value="30"/>
   ```
    - `image_topic`指明订阅的相机话题。
    - `lidar_topic`指明订阅的激光雷达话题。
    - `the_view_number`为激光雷达视场区域编号，1为x正向，2为y负向，3为x负向，4为y正向。
    - `the_field_of_view`为水平视场角，单位度。
    - `the_sensor_height`指明传感器距地面高度，单位为米。
    - `the_view_higher_limit`和`the_view_lower_limit`指明期望的点云相对地面的限制高度，单位为米。
    - `the_min_distance`和`the_max_distance`指明期望的点云相对传感器的限制距离，单位为米。
    - `jet_color`与点云成像颜色有关。
 - 编写`lidar_camera_fusion/conf/head_camera.yaml`
   ```Shell
   %YAML:1.0
   ---
   ProjectionMat: !!opencv-matrix
      rows: 3
      cols: 4
      dt: d
      data: [920.6949462890625, 0, 348.0517667538334, 0, 0, 934.1571044921875, 177.8464786820405, 0, 0, 0, 1, 0]
   LidarToCameraMat: !!opencv-matrix
      rows: 4
      cols: 4
      dt: d
      data: [0, -1, 0, 0, 0, 0, -1, 0, 1, 0, 0, 0, 0, 0, 0, 1]
   RotationAngleX: -2.5
   RotationAngleY: 2.75
   RotationAngleZ: 0
   ```
 - 参数含义如下
   ```Shell
   ProjectionMat:
     该3x4矩阵为通过相机内参矩阵标定得到的projection_matrix。
   LidarToCameraMat:
     该4x4矩阵为相机与激光雷达坐标系的齐次转换矩阵，左上角3x3为旋转矩阵，右上角3x1为平移矩阵。
     例如：
     Translation = [dx, dy, dz]T
                [0, -1,  0]
     Rotation = [0,  0, -1]
                [1,  0,  0]
     则：
     LidarToCameraMat = [0, -1,  0, dx]
                        [0,  0, -1, dy]
                        [1,  0,  0, dz]
                        [0,  0,  0,  1]
   RotationAngleX/Y/Z:
     该值是对LidarToCameraMat矩阵进行修正的旋转角度，初始应设置为0，之后根据投影效果进行细微调整，单位为度。
   ```

## 运行
 - 启动`lidar_camera_fusion`
   ```Shell
   roslaunch lidar_camera_fusion lidar_camera_fusion.launch
   ```
 - 融合图像话题
   `/image_fusion`

## 附图
   ```Shell
                               \     /    Rotation:
                                \ |z/     [0 -1  0]
                                 \|/      [0  0 -1]
                                  █————x  [1  0  0]
                               forward    
                                 cam_1    
  
                                █████
                   |x         ██  |x ██
   [1  0  0]       |         █    |    █                 [-1 0  0]
   [0  0 -1]  z————█ cam_4  █ y———.z    █  cam_2 █————z  [0  0 -1]
   [0  1  0]                 █         █         |       [0 -1  0]
                              ██     ██          |x      
                                █████
                                lidar

                             x————█       [0  1  0]
                                  |       [0  0 -1]
                                  |z      [-1 0  0]
                                 cam_3    
   ```





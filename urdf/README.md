<!--Cách xem file README.md: nhấn Ctrl + Shift + V-->
<p align ="center">
<img src="https://career.gpo.vn/uploads/images/truong-hoc/logo-hust.png" alt="HUST" width = "200"/>
</p>

# URDF
`URDF` là định dạng dựa trên XML để mô tả mô hình robot ( liên kết, khớp, hình dạng, va chạm và động học)
## Cách sử dụng:
- Được dùng trong các gói như `robot_state_publisher`, `rviz1`, `moveit2`
- Tệp `URDF` thường được load từ file hoặc trực tiếp trong code qua thư viện `urdfdom`
## Ví dụ:
### Tạo file
```xml
<!-- robot.urdf -->
<robot name="my_robot">
  <link name="base_link">
    <visual>
      <geometry><box size="0.5 0.5 0.1"/></geometry>
    </visual>
  </link>
  <joint name="joint1" type="fixed">
    <parent link="base_link"/>
    <child link="sensor_link"/>
  </joint>
</robot>
```
### Sử dụng trong launch file
```python
from launch_ros.actions import Node

def generate_launch_description():
    robot_state_publisher = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        parameters=[{'robot_description': open('robot.urdf').read()}]
    )
    return LaunchDescription([robot_state_publisher])
```

# XACRO (XML Macro)
## Khái niệm: 
Là ngôn ngữ tiền xử lí cho `URDF`, giúp tạo `URDF` linh hoạt bằng cách sử dụng các biến, macro và điều kiện
## Cách sử dụng trong ROS2
Dùng lệnh `xacro` để sinh file `URDF` trước khi chạy:
```bash
ros2 run xacro xacro robot.xacro > robot.urdf
```
Hoặc load trực tiếp từ trong launch file:
```python
parameters=[{'robot_description': Command(['xacro ', 'robot.xacro'])}]
```
## Ví dụ:
```xml
<!-- robot.xacro -->
<xacro:macro name="wheel" params="prefix radius">
  <link name="${prefix}_wheel">
    <visual><geometry><sphere radius="${radius}"/></geometry></visual>
  </link>
</xacro:macro>

<xacro:wheel prefix="left" radius="0.1"/>
<xacro:wheel prefix="right" radius="0.1"/>
```
# YAML (YAML Ain't Markup Language)
## Khái niệm:
Dùng để cấu hình tham số ROS2, cài đặt node, hoặc map
## Cách sử dụng trong ROS2
- Đọc qua `rclcpp`(C++) hoặc `rclpy` (python) bằng `declare_parameter`
- Dùng trong launch file để truyền tham số
## Ví dụ
```yaml
# config.yaml
controller:
  max_speed: 2.0
  pid_gains: {p: 1.0, i: 0.1, d: 0.01}
```
Load trong launch file:
```python
Node(
    package='my_pkg',
    executable='node',
    parameters=['config.yaml']
)
```

# XML (eXtensible Markup Lnaguage)
## Khái niệm
Dùng trong `launch file` của ROS2 hoặc định nghĩa plugin Gazebo
## Cách sử dụng trong ROS2
- Tạo launch file `XML` để khởi chạy các node
### Ví dụ
```xml
<!-- launch.xml -->
<launch>
  <node pkg="demo_nodes_cpp" exec="talker" name="my_talker"/>
  <node pkg="demo_nodes_cpp" exec="listener" name="my_listener"/>
</launch>
```
Chạy bằng lệnh:
```bash
ros2 launch launch.xml
```
# STL (Standard Tessellation Language)
## Khái niệm
- Định dạng tệp 3D mô tả hình học của robot ( dạng lưới tam giác)
- Được dung trong `rviz2` hoặc `Gazebo` để hiển thị mô hình
## Cách sử dụng trong ROS2
Khai báo trong `URDF/XACRO`:
```xml
<link name="camera_link">
  <visual>
    <geometry><mesh filename="package://my_pkg/meshes/camera.stl"/></geometry>
  </visual>
</link>
```
Đặt trong thư mục `meshes/` của gói ROS2

# COLCON
Là công cụ build system được thiết kế đặc biệt cho `ROS2`, dùng để biên dịch, quản lí các thuộc tính, triển khai các gói trong workspace. Nó là sự kế thừa và cải tiến từ `catkin` trong `ROS1`
## Các lệnh cơ bản
- `colcon build`: biên dịch toàn bộ workspace
- `colcon build --packages-select <tên_gói>`: chỉ biên dịch gói được chỉ định
- `colcon test`: chạy unit tests sau khi build
- `colcon list`: liệt kê tất cả các gói trong workspace
- `colcon clean`: xóa thư mục build/install/log (dùng khi có lỗi biên dịch)
## Ví dụ sử dụng
### a. Build workspace
```bash
cd ~/ros2_ws  # Chuyển vào workspace
colcon build  # Build tất cả gói
```
Sau khi build, các file sẽ được lưu vào thư mục:
- `build/`: file tạm thười trong quá trình biên dịch
- `install/`: gói đã biên dịch xong (chứa setup files)
- `log/`: Báo cáo lỗi nếu có
### b. Build 1 gói cụ thể
```bash
colcon build --pakages-select my_robot
```
### c. Load môi trường sau build
```bash
source ~/ros2_ws/install/setup.bash
```
## Mẹo khi dùng Colcon
- Build song song: tăng tốc độ với `--paralleb-workers`
```bash
colcon build --parabllel-workers 4`
```
- Bỏ qua gói lỗi: dùng `--packages-skip <tên gói>`
- Log chi tiết: thêm `--event-handlers console_direct+`

# ROS 2 Robot Simulation Project

A basic ROS 2 project demonstrating robot simulation using URDF/XACRO, YAML, STL, and launch files.

## Project Structure
ros2_ws/
└── src/
└── my_robot/
├── meshes/
│ └── chassis.stl
├── urdf/
│ ├── robot.xacro
│ └── robot.urdf
├── config/
│ └── controllers.yaml
├── launch/
│ └── simulate.launch.py
└── package.xml

## 1. File Descriptions

### a. `robot.xacro` (XACRO Macro File)
```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="mobile_robot">
  <!-- Parameters -->
  <xacro:property name="chassis_length" value="0.5" />
  <xacro:property name="wheel_radius" value="0.1" />

  <!-- Base Link -->
  <link name="base_link">
    <visual>
      <geometry>
        <mesh filename="package://my_robot/meshes/chassis.stl"/>
      </geometry>
    </visual>
  </link>

  <!-- Wheel Macro -->
  <xacro:macro name="wheel" params="prefix x y">
    <link name="${prefix}_wheel">
      <visual>
        <geometry>
          <cylinder radius="${wheel_radius}" length="0.05"/>
        </geometry>
      </visual>
    </link>
    <joint name="${prefix}_wheel_joint" type="continuous">
      <parent link="base_link"/>
      <child link="${prefix}_wheel"/>
      <origin xyz="${x} ${y} -0.1" rpy="0 1.57 0"/>
    </joint>
  </xacro:macro>

  <!-- Add Wheels -->
  <xacro:wheel prefix="left" x="0" y="0.15"/>
  <xacro:wheel prefix="right" x="0" y="-0.15"/>
</robot>
```
### b. `controllers.yaml` (Configuration)
```yaml
wheel_controllers:
  left_wheel_controller:
    type: "velocity_controllers/JointVelocityController"
    joint: "left_wheel_joint"
    pid: {p: 1.0, i: 0.1, d: 0.01}
  right_wheel_controller:
    type: "velocity_controllers/JointVelocityController"
    joint: "right_wheel_joint"
    pid: {p: 1.0, i: 0.1, d: 0.01}
```

### c. `simulate.launch.py` (Launch file)
```python
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.substitutions import Command

def generate_launch_description():
    return LaunchDescription([
        # Robot State Publisher
        Node(
            package='robot_state_publisher',
            executable='robot_state_publisher',
            parameters=[{
                'robot_description': Command(['xacro ', 'urdf/robot.xacro'])
            }]
        ),
        
        # Joint State Publisher (GUI)
        Node(
            package='joint_state_publisher_gui',
            executable='joint_state_publisher_gui'
        ),
        
        # RViz2
        Node(
            package='rviz2',
            executable='rviz2',
            arguments=['-d', 'default.rviz']
        ),
        
        # Controller Manager
        Node(
            package='controller_manager',
            executable='spawner',
            arguments=[
                '--controller-manager', '/controller_manager',
                'left_wheel_controller',
                'right_wheel_controller'
            ]
        )
    ])
```
## 2. Installation & Usage
### Prerequisites
- ROS2 Humble
- Gazebo
### Build the Project
```bash
cd ~/ros2_ws
colcon build --packages-select my_robot
source install/setup.bash
```
### Run Simulation
```bash
ros2 laucnh my_robot simulate.launch.py
```
## 3. Expected Output
- RViz2 will display the robot with:
    - 1 chassis (STL mesh)
    - 2 wheels (URDF cylinders)
- Joint State Publisher GUI allows manual joint control
- Controllers are loaded from YAML config



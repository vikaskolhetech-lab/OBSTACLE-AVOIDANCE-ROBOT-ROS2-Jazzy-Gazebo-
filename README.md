# OBSTACLE-AVOIDANCE-ROBOT-ROS2-Jazzy-Gazebo-
A simple autonomous obstacle avoidance robot using **ROS2 Jazzy**, **TurtleBot3**, and **Gazebo Simulation**.   The robot detects obstacles using **LiDAR (/scan topic)** and automatically turns to avoid collisions.

STEP 1 → Launch Robot in Gazebo
🔹 TERMINAL 1
cd ~
source /opt/ros/jazzy/setup.bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
✔️ Expected:
Gazebo opens
Robot appears
World loads

🚨 STEP 2 → VERIFY SENSOR (MOST IMPORTANT)
Open NEW terminal:
source /opt/ros/jazzy/setup.bash
ros2 topic list
✔️ You MUST see:
/cmd_vel
/odom
/scan   👈 VERY IMPORTANT
/tf

❗ If /scan is NOT visible
Then LiDAR is not enabled → obstacle avoidance will NOT work.
👉 Fix (common issue):
 You must use a robot model/world that includes LiDAR plugin.

🚀 STEP 3 → BRIDGE (ONLY IF REQUIRED)
Some setups need bridge between ROS and Gazebo:
🔹 TERMINAL 2
source /opt/ros/jazzy/setup.bash

ros2 run ros_gz_bridge parameter_bridge \
/cmd_vel@geometry_msgs/msg/Twist@gz.msgs.Twist
✔️ This ensures movement works.
👉 For obstacle avoidance also add:
ros2 run ros_gz_bridge parameter_bridge \
/scan@sensor_msgs/msg/LaserScan@gz.msgs.LaserScan
✔️ NOW robot can “see” obstacles

🚀 STEP 4 → TEST LiDAR DATA
ros2 topic echo /scan
✔️ You should see:
ranges[] values
distance readings
👉 If values keep changing → LiDAR is working

🚀 STEP 5 → CREATE OBSTACLE AVOIDANCE NODE
Create package:
cd ~/ros2_ws/src
ros2 pkg create obstacle_bot --build-type ament_python --dependencies rclpy geometry_msgs sensor_msgs

🧠 Python Logic (CORE BRAIN OF ROBOT)
Create file:
cd ~/ros2_ws/src/obstacle_bot/obstacle_bot
nano obstacle_avoidance.py

🔥 FINAL WORKING CODE
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from geometry_msgs.msg import Twist

class ObstacleAvoidance(Node):

   def __init__(self):
       super().__init__('obstacle_avoidance')

       self.pub = self.create_publisher(Twist, '/cmd_vel', 10)
       self.sub = self.create_subscription(LaserScan, '/scan', self.scan_callback, 10)

   def scan_callback(self, msg):

       # Front distance (middle of LiDAR)
       front = min(msg.ranges[0:10] + msg.ranges[-10:])

       cmd = Twist()

       if front < 0.5:   # obstacle close
           cmd.linear.x = 0.0
           cmd.angular.z = 0.5   # turn left
       else:
           cmd.linear.x = 0.2    # move forward
           cmd.angular.z = 0.0

       self.pub.publish(cmd)


def main(args=None):
   rclpy.init(args=args)
   node = ObstacleAvoidance()
   rclpy.spin(node)
   node.destroy_node()
   rclpy.shutdown()

if __name__ == '__main__':
   main()

🚀 STEP 6 → BUILD PACKAGE
cd ~/ros2_ws
colcon build
source install/setup.bash

🚀 STEP 7 → RUN OBSTACLE AVOIDANCE
TERMINAL 3
source ~/ros2_ws/install/setup.bash
ros2 run obstacle_bot obstacle_avoidance



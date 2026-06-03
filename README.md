# romea_localisation_msgs

`romea_localisation_msgs` defines the ROS2 messages exchanged between ROMEA localisation plugins and localisation filters.

The package does not implement any algorithm. It provides the common message types used to express localisation observations and localisation status. The conversion helpers are provided by `romea_localisation_utils`, while localisation filter nodes are provided by packages such as `romea_robot_to_world_localisation_core`, `romea_robot_to_robot_localisation_core` or `romea_robot_to_human_localisation_core`.

## 1) Concept

The localisation stack uses common ROS2 messages at its boundaries and ROMEA observation messages between plugins and filters:

```mermaid
flowchart LR
  subgraph ros2_inputs["ROS2 input messages"]
    inputs["sensor_msgs/msg/Imu<br/>nmea_msgs/msg/Sentence<br/>nav_msgs/msg/Odometry<br/>..."]
  end

  subgraph plugin_nodes["Localisation plugin nodes"]
    plugins["convert ROS2 data into typed localisation observations"]
  end

  subgraph observation_msgs["romea_localisation_msgs"]
    observations["ObservationTwist2DStamped<br/>ObservationPosition2DStamped<br/>ObservationAngularSpeedStamped<br/>ObservationAttitudeStamped<br/>ObservationCourseStamped<br/>ObservationRangeStamped<br/>..."]
  end

  subgraph filters["Localisation filters"]
    filter["localisation filter node"]
  end

  subgraph ros2_outputs["ROS2 output messages"]
    outputs["nav_msgs/msg/Odometry<br/>geometry_msgs/msg/Pose<br/>tf<br/>..."]
  end

  inputs -->|consume| plugins
  plugins -->|publish| observations
  observations -->|fuse| filter
  filter -->|publish| outputs

  classDef ros2 fill:#e8f2ff,stroke:#5b8ec7,color:#111,rx:6,ry:6
  classDef plugin fill:#eaf7ea,stroke:#5c9f5c,color:#111,rx:6,ry:6
  classDef msg fill:#fff6d8,stroke:#c9a227,color:#111,rx:6,ry:6
  classDef filterStyle fill:#f1eaff,stroke:#8b6fc6,color:#111,rx:6,ry:6

  class inputs,outputs ros2
  class plugins plugin
  class observations msg
  class filter filterStyle

  style ros2_inputs fill:#f6faff,stroke:#9abbe3,rx:6,ry:6
  style plugin_nodes fill:#f7fff7,stroke:#9ecf9e,rx:6,ry:6
  style observation_msgs fill:#fffaf0,stroke:#dec86b,rx:6,ry:6
  style filters fill:#faf7ff,stroke:#b8a4dd,rx:6,ry:6
  style ros2_outputs fill:#f6faff,stroke:#9abbe3,rx:6,ry:6
```

Each plugin converts one standard ROS2 sensor or controller stream into one or more `romea_localisation_msgs` observations. Localisation filters subscribe to these observations, fuse them and publish results using standard ROS2 outputs when possible, for example filtered odometry and transforms for robot-to-world localisation.

## 2) Observation Messages

Observation messages are available in two forms:

* a payload message that contains the measured quantity and its uncertainty;
* a stamped message that adds a `std_msgs/Header`.

The stamped form is the one normally exchanged on ROS2 topics.

| Stamped message | Payload | Typical producer |
| --- | --- | --- |
| `ObservationTwist2DStamped` | `ObservationTwist2D` | Odometry localisation plugin |
| `ObservationAngularSpeedStamped` | `ObservationAngularSpeed` | IMU localisation plugin |
| `ObservationAttitudeStamped` | `ObservationAttitude` | IMU localisation plugin |
| `ObservationPosition2DStamped` | `ObservationPosition2D` | GPS or RTLS localisation plugin |
| `ObservationCourseStamped` | `ObservationCourse` | GPS localisation plugin |
| `ObservationPose2DStamped` | `ObservationPose2D` | Pose plugin, for example lidar, radar or RTLS |
| `ObservationRangeStamped` | `ObservationRange` | RTLS range localisation plugin |

## 3) Observation Content

The observation payloads carry compact 2D localisation information and the uncertainty associated with each observation. Depending on the measured quantity, this uncertainty is represented as a standard deviation, a variance/covariance matrix, or a covariance embedded in a `romea_common_msgs` type.

| Message | Measurement | Uncertainty |
| --- | --- | --- |
| `ObservationTwist2D` | Robot planar twist and lever arm | Covariance in `romea_common_msgs/Twist2D` |
| `ObservationAngularSpeed` | Yaw angular speed | Standard deviation |
| `ObservationAttitude` | Roll and pitch angles | Covariance |
| `ObservationPosition2D` | 2D position and lever arm | Covariance in `romea_common_msgs/Position2D` |
| `ObservationCourse` | Course angle | Standard deviation |
| `ObservationPose2D` | 2D pose and lever arm | Covariance in `romea_common_msgs/Pose2D` |
| `ObservationRange` | Range and antenna positions | Range standard deviation |

The `lever_arm` fields describe the position of the physical measurement point with respect to the localisation body frame. For example, a GPS observation is measured at the antenna position, not directly at the robot control point.

## 4) Localisation Status

`LocalisationStatus` represents the current state of a localisation process:

| Value | Meaning |
| --- | --- |
| `INIT` | The localisation process is being initialised |
| `RUNNING` | The localisation process is running |
| `RESET` | The localisation process has been reset |
| `ABORTED` | The localisation process has stopped because it cannot provide a valid estimate |

This status is used by localisation nodes and helpers to expose the state of the localisation lifecycle.

## 5) Notes

Only the message interfaces listed in `CMakeLists.txt` are currently generated by this package.

## License

This project is released under the Apache License 2.0. See the `LICENSE` file for details.

## Authors

This package was developed by **Jean Laneurit**.

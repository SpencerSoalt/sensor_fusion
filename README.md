# Sensor Fusion

Two ROS 2 (Humble) packages for LiDAR-camera fusion and 3D object detection.

## Packages

- **lidar_camera_fusion** — Projects LiDAR points into 2D detection bounding boxes, producing a labeled point cloud.
- **dbscan_clustering** — Clusters the labeled point cloud with DBSCAN and outputs 3D bounding boxes.

### Data Flow

```
LiDAR /velodyne_points  ──┐
                          ├──► lidar_camera_fusion ──► /fused/cloud_in_boxes ──► dbscan_clustering ──► /detections_3d
Camera /camera/detections ┘
```

---

## Prerequisites

- ROS 2 Humble
- Python 3 packages: `numpy`, `opencv-python`, `open3d`, `scikit-learn`

---

## Build

```bash
cd sensor_fusion
source /opt/ros/humble/setup.bash
colcon build --symlink-install
source install/setup.bash
```

---

## Run

Launch each node in a separate terminal (after sourcing the workspace).

**Terminal 1 — LiDAR-Camera Fusion:**
```bash
ros2 launch lidar_camera_fusion lidar_camera_fusion.launch.py \
  calibration_file:=/path/to/lidar_camera_extrinsics.yaml
```

**Terminal 2 — DBSCAN Clustering:**
```bash
ros2 launch dbscan_clustering dbscan_clustering.launch.py
```

A calibration file (`lidar_camera_extrinsics.yaml`) with the camera intrinsics and LiDAR-to-camera extrinsic transform is required. A default file is included at the repo root.

---

## Launch Parameters

### lidar_camera_fusion

| Parameter | Default | Description |
|---|---|---|
| `calibration_file` | `/ws/lidar_camera_extrinsics.yaml` | Path to calibration YAML |
| `max_distance` | `40.0` | Max projection distance (m) |
| `min_distance` | `0.5` | Min projection distance (m) |
| `sync_slop` | `0.05` | Topic sync tolerance (s) |

### dbscan_clustering

| Parameter | Default | Description |
|---|---|---|
| `eps` | `0.7` | DBSCAN neighbourhood radius (m) |
| `min_samples` | `5` | Min points to form a core point |
| `min_cluster_size` | `22` | Discard clusters smaller than this |
| `max_cluster_size` | `2000` | Discard clusters larger than this |
| `depth_filter_method` | `percentile` | `jump` or `percentile` |
| `depth_percentile` | `30.0` | Near-surface anchor percentile |
| `depth_tolerance` | `1.3` | Multiplier on near-percentile depth |
| `cluster_merge_distance` | `1.0` | Box gap to merge same-class clusters (0 = off) |
| `publish_cluster_cloud` | `true` | Publish colored cluster cloud for RViz |

---

## Topics

| Topic | Type | Direction |
|---|---|---|
| `/velodyne_points` | `sensor_msgs/PointCloud2` | Input to fusion |
| `/camera/detections` | `vision_msgs/Detection2DArray` | Input to fusion |
| `/fused/cloud_in_boxes` | `sensor_msgs/PointCloud2` | Fusion → clustering |
| `/detections_3d` | `vision_msgs/Detection3DArray` | Output |
| `/dbscan/cluster_cloud` | `sensor_msgs/PointCloud2` | RViz visualization |
| `/dbscan/bbox_markers` | `visualization_msgs/MarkerArray` | RViz visualization |

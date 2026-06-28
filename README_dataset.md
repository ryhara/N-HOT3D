# N-HOT3D Dataset

## Directory Structure
```
HOT3D/Aria
├── test/
│   └── ...
├── train/
│   └── ...
└── valid/
    ├── P0001_10a27bf7
    ├── ...
    └── P0001_4bf4e21a
        ├── gt
        │   ├── gt_0000000000.jsonl
        │   ├── gt_0000000001.jsonl
        │   └──  ...
        │
        ├── gt_hand_bbox
        │   ├── frame_0000000000.txt
        │   ├── frame_0000000001.txt
        │   └──  ...
        │
        ├── gt_hand_seg
        │   ├── frame_0000000000.txt
        │   ├── frame_0000000001.txt
        │   └──  ...
        │
        ├── gt_hand_mask
        │   ├── gt_0000000000_left.jpg
        │   ├── gt_0000000000_right.jpg
        │   ├── gt_0000000001_left.jpg
        │   ├── gt_0000000001_right.jpg
        │   └── ...
        │
        ├── joints_gt
        │   ├── joints_0000000000.jsonl
        │   ├── joints_0000000001.jsonl
        │   └── ...
        │
        ├── undistort_recording.mp4
        │
        ├── events_0000000000.h5
        ├── events_0000000001.h5
        └── ...
```

YOLO format
```
NHOT3D_yolo_seg_lnes_frame_v2
├── images/
│   ├── test/
│   │   └── ...
│   ├── train/
│   │   └── ...
│   └── valid/
│       ├──  P0002_c68438ce_frame_0000000000.png
│       └── ...
├── labels/
│   ├── test/
│   │   └── ...
│   ├── train/
│   │   └── ...
│   └── valid/
│       ├── ...
│       ├──  P0002_c68438ce_frame_0000000000.txt
│       └── ...
└── train.yaml
```

## Dataset Description

### gt/*.jsonl

Hand pose ground truth in MANO format. Each file contains one JSON object per frame.

```json
{
  "left": {
    "betas": [float, ...],              // shape: (10,) - MANO shape parameters
    "quat": [w, x, y, z],               // shape: (4,) - rotation quaternion
    "translation": [x, y, z],           // shape: (3,) - position in 3D space
    "joint_angles": [float, ...],       // shape: (15,) - MANO joint angles (PCA coefficients)
    "vertices": [[x, y, z], ...],       // shape: (778, 3) - mesh vertices
    "triangles": [[i, j, k], ...],      // shape: (1538, 3) - triangle face indices
    "vertex_normals": [[x, y, z], ...], // shape: (778, 3) - vertex normals
    "hand_landmarks": [[x, y, z], ...], // shape: (20, 3) - 20 hand keypoints in 3D
    "joint_points": [...],              // shape: (23, 2, 3) - joint frame data for visualization
    "valid": bool                       // validity flag (true if hand is visible)
  },
  "right": { ... }  // same structure for right hand
}
```

### gt_hand_bbox/*.txt

Hand bounding boxes in YOLO format (normalized coordinates).

```txt
<hand_id> <x_center> <y_center> <width> <height>
```

- `hand_id`: 0 = left hand, 1 = right hand
- All coordinates are normalized (0.0 ~ 1.0)

Example:
```
0 0.339844 0.821289 0.136719 0.193359
1 0.497070 0.763672 0.130859 0.191406
```

### gt_hand_seg/*.txt

Hand segmentation contours as polygon points (normalized coordinates).

```txt
<hand_id> <x1> <y1> <x2> <y2> ... <xn> <yn>
```

- `hand_id`: 0 = left hand, 1 = right hand
- Variable number of points per hand (typically 100-150 points)
- All coordinates are normalized (0.0 ~ 1.0)

Example:
```
0 0.330078 0.724609 0.326172 0.730469 0.328125 0.732422 ...
1 0.482422 0.667969 0.480469 0.669922 0.480469 0.681641 ...
```

### gt_hand_mask/*.jpg

Binary segmentation mask images for each hand.

- Format: JPEG grayscale image
- Naming: `gt_<frame_number>_<left|right>.jpg`
- White (255) = hand region, Black (0) = background

### joints_gt/*.jsonl

Hand joint keypoints with both 3D and 2D projections.

```
{
  "left": {
    "joints_3d": [[x, y, z], ...],    // shape: (20, 3) - 20 joint positions in 3D
    "joints_2d": [[x, y], ...],       // shape: (20, 2) - 20 joint positions in 2D (projected)
    "vertices_3d": [[x, y, z], ...],  // shape: (778, 3) - mesh vertices in 3D
    "vertices_2d": [[x, y], ...]      // shape: (778, 2) - mesh vertices in 2D (projected)
  },
  "right": { ... }  // same structure for right hand
}
```

### events_*.h5

Event camera data in HDF5 format.

```
Dataset: "events"
  shape: (N, 4)
  dtype: float32
```

Column structure:
| Column | Name | Type | Description |
|--------|------|------|-------------|
| 0 | timestamp | float | Event timestamp in seconds |
| 1 | x | float | Pixel x coordinate |
| 2 | y | float | Pixel y coordinate |
| 3 | polarity | float | 0 = OFF (brightness decrease), 1 = ON (brightness increase) |

Example:
```
0.00166    216.0   128.0   0.0
0.00166     76.0   150.0   0.0
0.00166     84.0   137.0   1.0
```

### undistort_recording.mp4

Undistorted RGB video recording synchronized with event data.

### Image Dimensions

- Event camera: 346 x 260 pixels
- RGB camera: 512 x 512 pixels (undistorted)


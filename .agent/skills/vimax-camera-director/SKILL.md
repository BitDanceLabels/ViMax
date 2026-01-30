---
name: ViMax Camera Director Agent
description: Construct camera position trees and manage multi-camera shot sequences for consistent video generation
---

# 📷 ViMax Camera Director Agent

Xây dựng cây camera và quản lý chuỗi shot đa camera để tạo video nhất quán.

## Capabilities

1. **Construct Camera Tree** - Xây dựng cấu trúc parent-child giữa các vị trí camera
2. **Select Reference Images** - Chọn hình tham chiếu phù hợp cho từng shot
3. **Generate Transition Videos** - Tạo video chuyển cảnh giữa các shot

---

## 🌳 Task 1: Construct Camera Tree

### Concept

Camera Tree là cấu trúc phân cấp giữa các vị trí camera, trong đó:
- **Parent Camera**: Camera có góc nhìn rộng hơn, chứa nội dung của child
- **Child Camera**: Camera có góc nhìn hẹp hơn (close-up, detail shot)

Ví dụ:
```
CAMERA_0 (Wide Shot - cả cảnh bàn ăn)
├── CAMERA_1 (Medium Shot - Alice và Bob)
│   ├── CAMERA_2 (Close-up - mặt Alice)
│   └── CAMERA_3 (Close-up - mặt Bob)
└── CAMERA_4 (Insert Shot - đĩa thức ăn)
```

### System Prompt

```
[Role]
You are a professional video editing expert specializing in multi-camera shot analysis and scene structure modeling. You have deep knowledge of cinematic language, enabling you to understand shot sizes (e.g., wide shot, medium shot, close-up) and content inclusion relationships. You can infer hierarchical structures between camera positions based on corresponding shot descriptions.

[Task]
Your task is to analyze the input camera position data to construct a "camera position tree". This tree structure represents a relationship where a parent camera's content encompasses that of a child camera. Specifically, you need to identify the parent camera for each camera position (if one exists) and determine the dependent shot indices (i.e., the specific shots within the parent camera's footage that contain the child camera's content). If a camera position has no parent, output None.

[Input]
The input is a sequence of cameras. The sequence will be enclosed within <CAMERA_SEQ> and </CAMERA_SEQ>.
Each camera contains a sequence of shots filmed by the camera, which will be enclosed within <CAMERA_N> and </CAMERA_N>, where N is the index of the camera.

Below is an example of the input format:

<CAMERA_SEQ>
<CAMERA_0>
Shot 0: Medium shot of the street. Alice and Bob are walking towards each other.
Shot 2: Medium shot of the street. Alice and Bob hug each other.
</CAMERA_0>
<CAMERA_1>
Shot 1: Close-up of the Alice's face. Her expression shifts from surprise to delight as she recognizes Bob.
</CAMERA_1>
</CAMERA_SEQ>


[Output]
{
  "camera_parent_items": [
    {
      "parent_cam_idx": null,  // Root camera has no parent
      "parent_shot_idx": null,
      "reason": "CAMERA_0 establishes the entire scene and is the root camera.",
      "is_parent_fully_covers_child": null,
      "missing_info": null
    },
    {
      "parent_cam_idx": 0,
      "parent_shot_idx": 0,
      "reason": "Close-up of Alice is contained within the medium shot of the street",
      "is_parent_fully_covers_child": false,
      "missing_info": "The frontal view of Alice's face (parent shot shows her in profile)"
    }
  ]
}

[Guidelines]
- The language of all output values (not include keys) should be consistent with the language of the input.
- Content Inclusion Check: The parent camera should as fully as possible contain the child camera's content in certain shots. Analyze shot descriptions by comparing keywords (e.g., characters, actions, setting) to ensure the parent shot's field of view covers the child shot's.
- Transition Smoothness Priority: Larger shot size as parent camera is preferred, such as Wide Shot -> Medium Shot or Medium Shot -> Close-up. The shot sizes of adjacent parent and child nodes should be as similar as possible. A direct transition from a long shot to a close-up is not allowed unless absolutely necessary.
- Temporal Proximity: Each camera is described by its corresponding first shot, and the parent camera is located based on the description of the first shot. The shot index of the parent camera should be as close as possible to the first shot index of the child camera.
- Logical Consistency: The camera tree should be acyclic, avoid circular dependencies. If a camera is contained by multiple potential parents, select the best match (based on shot size and content). If there is no suitable parent camera, output None.
- When a broader perspective is not available, choose the shot with the largest overlapping field of view as the parent (the one with the most information overlap), or a shot can also serve as the parent of a reverse shot. When two cameras can be the parent of each other, choose the one with the smaller index as the parent of the camera with the larger index.
- Only one camera can exist without a parent (the root).
- When describing the elements lost in a shot, carefully compare the details between the parent shot and the child shot.
- The first camera must be the root of the camera tree.
```

### Human Prompt Template

```
<CAMERA_SEQ>
{camera_seq_str}
</CAMERA_SEQ>
```

---

## 🖼️ Task 2: Generate First Frame

### Prompt Template

```
Image 0: [Character appearance description]
Image 1: [Another character appearance description]
...
Generate an image based on the following description: [First frame description from storyboard]
```

### Input Requirements

- **Reference Images**: Portraits/appearances của các nhân vật trong shot
- **First Frame Description**: Mô tả visual chi tiết từ Storyboard Artist

### Output

- Generated image matching the first frame description
- Maintains character consistency with reference images

---

## 🎬 Task 3: Generate Transition Video

Tạo video chuyển cảnh giữa hai shot.

### Prompt Template

```
Two shots. The transition between the shots is a cut to. The style of the two shots should be consistent.

The first shot description: [visual description of shot 1]
The second shot description: [visual description of shot 2]
```

### Input Requirements

- First shot visual description
- Second shot visual description
- First shot's first frame image (as reference)

---

## Camera Tree Construction Algorithm

```python
def construct_camera_tree(cameras, shot_descriptions):
    """
    Build a camera tree based on shot containment relationships.
    
    Rules:
    1. Wide shots are parents of medium shots
    2. Medium shots are parents of close-ups
    3. Root camera (usually establishing shot) has no parent
    4. Temporal proximity matters - closer shots preferred
    """
    
    for camera in cameras:
        # Find best parent based on:
        # 1. Shot size hierarchy (wide > medium > close)
        # 2. Content overlap
        # 3. Temporal proximity
        
        parent = find_best_parent(camera, cameras, shot_descriptions)
        camera.parent = parent
    
    return cameras
```

---

## Shot Size Hierarchy

```
Extreme Wide Shot (EWS)
      ↓
  Wide Shot (WS)
      ↓
Medium Wide Shot (MWS)
      ↓
  Medium Shot (MS)
      ↓
Medium Close-up (MCU)
      ↓
   Close-up (CU)
      ↓
Extreme Close-up (ECU)
```

**Parent-Child Rules:**
- Parent must be same level or WIDER than child
- Skip at most 1 level (Wide → Close-up OK, but Wide → ECU needs intermediate)

---

## Usage Example

```python
# Input: Cameras with shot descriptions
cameras = [
    Camera(idx=0, shots=["Wide shot of cafe, Alice enters"]),
    Camera(idx=1, shots=["Medium shot, Alice sits at table"]),
    Camera(idx=2, shots=["Close-up, Alice's face smiling"]),
    Camera(idx=3, shots=["Insert shot, coffee cup steaming"])
]

# Output: Camera tree
camera_tree = await director.construct_camera_tree(cameras, shot_descriptions)

# Result:
# CAMERA_0 (root)
# ├── CAMERA_1 (parent: 0)
# │   └── CAMERA_2 (parent: 1)
# └── CAMERA_3 (parent: 0)
```

---

## Why Camera Trees Matter

1. **Consistency**: Reference images flow from parent to child, ensuring visual continuity
2. **Efficiency**: Generate wide shots first, then use them as references for close-ups
3. **Logic**: Mimics real film production where master shots are filmed before coverage
4. **Quality**: Each shot inherits visual DNA from its parent

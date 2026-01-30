# 🎬 ViMax Skills Package

Bộ skills cho AI Agents (Claude Code, Gemini, v.v.) để tạo video từ ý tưởng thô.

## 📁 Cấu trúc

```
.agent/skills/
├── vimax-screenwriter/      # Biên kịch - Ý tưởng → Câu chuyện → Kịch bản
├── vimax-script-planner/    # Phân loại & mở rộng script (narrative/motion/montage)
├── vimax-storyboard-artist/ # Thiết kế storyboard từng shot
├── vimax-camera-director/   # Xây dựng camera tree & reference images
└── vimax-character-planner/ # Trích xuất & theo dõi nhân vật
```

## 🔄 Workflow Pipeline

```
📝 Ý tưởng thô
      │
      ▼
┌─────────────────────────────────────────┐
│  vimax-screenwriter                     │
│  - develop_story() → Câu chuyện         │
│  - write_script() → Kịch bản phân cảnh  │
└─────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────┐
│  vimax-script-planner                   │
│  - route_intent() → narrative/motion    │
│  - plan_script() → Script chi tiết      │
└─────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────┐
│  vimax-character-planner                │
│  - extract_characters() → Danh sách     │
│  - merge_characters() → Nhất quán       │
│  - generate_portraits() → Hình tham chiếu│
└─────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────┐
│  vimax-storyboard-artist                │
│  - design_storyboard() → Shot list      │
│  - decompose_visual() → FF + Motion + LF│
└─────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────┐
│  vimax-camera-director                  │
│  - construct_camera_tree() → Hierarchy  │
│  - generate_first_frame() → Images      │
│  - generate_transition() → Videos       │
└─────────────────────────────────────────┘
      │
      ▼
🎬 Video hoàn chỉnh
```

## 🚀 Cách sử dụng

### Với Claude Code / Gemini

1. Copy folder `.agent/skills/` vào project của bạn
2. Khi cần sử dụng skill, yêu cầu AI đọc file `SKILL.md`

**Ví dụ prompt:**
```
Đọc skill vimax-screenwriter và giúp tôi biến ý tưởng sau thành kịch bản:
"Một chú mèo máy từ tương lai quay về quá khứ để giúp một cậu bé nhút nhát"
```

### Với Python (LangChain)

```python
from langchain.chat_models import init_chat_model

# Load prompts từ skill files
with open('.agent/skills/vimax-screenwriter/SKILL.md') as f:
    screenwriter_prompts = f.read()

# Sử dụng prompts trong chain
```

## 📊 Skills Overview

| Skill | Input | Output |
|-------|-------|--------|
| **Screenwriter** | Ý tưởng + Yêu cầu | Câu chuyện + Kịch bản phân cảnh |
| **Script Planner** | Ý tưởng thô | Script theo style (narrative/motion/montage) |
| **Character Planner** | Kịch bản | Danh sách nhân vật + Portraits |
| **Storyboard Artist** | Kịch bản + Nhân vật | Storyboard từng shot |
| **Camera Director** | Storyboard | Camera tree + First frames + Videos |

## 🔧 Tùy chỉnh

Mỗi file `SKILL.md` chứa các prompts có thể chỉnh sửa:

1. **System Prompts**: Định nghĩa vai trò và kỹ năng của agent
2. **Human Prompt Templates**: Format input với placeholders `{variable}`
3. **Output Format**: JSON schema cho structured output
4. **Guidelines**: Nguyên tắc và ràng buộc

## 🌐 Ngôn ngữ

Tất cả prompts đều hỗ trợ **đa ngôn ngữ** - output sẽ theo ngôn ngữ của input:
- Input tiếng Việt → Output tiếng Việt
- Input tiếng Anh → Output tiếng Anh

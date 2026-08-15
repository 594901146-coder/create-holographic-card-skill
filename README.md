# Create Holographic Card

把人物、宠物、产品或原创画面制作成一张可交互的 5:7 分层全息卡牌。这个 Codex skill 负责卡面检查、图像生成与编辑、主体分层、材质配置、预览校验，以及按需导出 React 组件或 Cardex `.hcard` 文件。

仓库直接提供可读、可编辑的 skill 源码，不包含 `.skill` 压缩包。

## 能做什么

- 接收已有图片，也可以从文字描述生成原创卡面。
- 将卡面拆成背景层和透明主体层，主体可独立视差运动。
- 使用 WebGL2 渲染彩虹箔片、微纹理、眩光、边缘高光和空闲扫光。
- 将卡牌注册到 Vite React 或 Next.js TypeScript 项目。
- 将完成的分层卡牌导出为 Cardex `.hcard` 交换文件。

光学效果只作用于主体下方的背景与材质层。透明主体保留原始像素，只参与视差和自然阴影。

## 工作流

1. 检查输入图片的比例、分辨率和透明度。
2. 生成或调整一张不含文字、边框和烘焙特效的 5:7 卡面。
3. 从同一张卡面生成背景板和纯色键控图。
4. 提取透明主体，并检查画布、位置、轮廓和 Alpha 覆盖率。
5. 选择材质、微纹理、调色板和公开强度参数。
6. 在右侧 Browser 中打开一次 WebGL2 预览并检查警告。
7. 返回生成次数、分层方法、文件路径和验证结果。

常规成功路径最多使用三次图像生成：一次卡面、一次背景板、一次键控图。工作流不会通过重试掩盖失败。

## 安装

Codex 会从 skill 目录中的 `SKILL.md` 读取名称、触发描述和执行规则。你可以安装到个人目录，也可以只在某个仓库中启用。详细规则见 [OpenAI 的 Build skills 文档](https://learn.chatgpt.com/docs/build-skills)。

### 使用 skill-installer

在 Codex 中输入：

```text
$skill-installer install create-holographic-card from https://github.com/594901146-coder/create-holographic-card-skill
```

### 手动安装到个人目录

macOS / Linux：

```bash
git clone https://github.com/594901146-coder/create-holographic-card-skill.git \
  ~/.agents/skills/create-holographic-card
```

Windows PowerShell：

```powershell
git clone https://github.com/594901146-coder/create-holographic-card-skill.git `
  "$HOME\.agents\skills\create-holographic-card"
```

### 只在当前仓库中启用

在目标仓库根目录执行：

```bash
git clone https://github.com/594901146-coder/create-holographic-card-skill.git \
  .agents/skills/create-holographic-card
```

Codex 通常会自动发现新安装的 skill。如果列表没有更新，请重启 Codex。

## 使用

显式调用：

```text
$create-holographic-card 把这张人物照片做成一张 5:7 分层全息卡，使用 pearl 材质和冷色光谱。
```

```text
$create-holographic-card 根据“一只坐在月球温室里的灰猫”生成原创卡面，并制作完整分层预览。
```

你也可以直接描述任务。Codex 会在请求与 `SKILL.md` 中的描述匹配时调用该 skill。

### 输入图片

提供图片时，skill 会先运行：

```bash
python scripts/inspect_card_face.py --art <input-image>
```

如果图片只存在比例问题，工作流会保留原有视野和裁切边界，不补全已经被画面裁掉的身体或背景。

## 依赖

完整工作流需要：

- 支持 skills、图像生成和本地图片检查的 Codex 环境。
- Python 3，以及 `Pillow` 和 `NumPy`。
- 最终交互预览所需的 `preview_holographic_card` MCP 工具。

安装 Python 图像依赖：

```bash
python -m pip install Pillow numpy
```

`rembg` 只用于本地主体分割回退。工作流不会在运行中下载模型；如需回退，请提前安装 `rembg` 并缓存对应的 `u2net_human_seg` 或 `u2netp` 模型。

```bash
python -m pip install rembg
```

`preview_holographic_card` 工具不在本仓库中。缺少该工具时，卡面生成和本地分层脚本仍可使用，但 skill 无法完成规定的 WebGL2 预览步骤。

## 分层脚本

从已接受卡面和键控图生成透明主体：

```bash
python scripts/prepare_subject_layer.py \
  --art <accepted-art> \
  --keyed <keyed-edit> \
  --key-color auto \
  --subject-kind human \
  --out <subject.png>
```

`--subject-kind` 支持 `human` 和 `generic`。脚本只把键控图当作 Alpha 证据，主体可见像素取自已接受卡面。

## React 集成

目标项目必须使用 React、TypeScript，并在 `package.json` 中声明 Vite 或 Next.js。

安装渲染组件：

```bash
python scripts/install_component.py --project <project-path>
```

注册一张卡牌：

```bash
python scripts/register_card.py \
  --project <project-path> \
  --slug <short-name> \
  --background <background.png> \
  --subject <subject.png> \
  --art-alt <description> \
  --presentation <presentation.json>
```

默认安装位置为 `src/components/holographic-card`。安装脚本不会覆盖已有模板文件，除非你明确传入 `--force`。

注册后可通过卡牌 ID 渲染：

```tsx
<HolographicCard card={cardRegistry[cardId]} />
```

## 导出 Cardex 文件

卡牌目录应包含 `background.png`、`subject.png` 和 `presentation.json`：

```bash
python scripts/export_hcard.py \
  --card-dir <card-directory> \
  --id <stable-id> \
  --title <title> \
  --output <card.hcard>
```

导出脚本打包已经验证的图层和 Presentation IR，不会重新生成图片或修改渲染参数。

## 默认材质参数

| 参数 | 默认值 |
| --- | ---: |
| Foil | `0.78` |
| Texture | `0.48` |
| Glare | `0.62` |
| Tilt | `14° × 14°` |
| Hover scale | `1.024` |

材质族包括 `clear-coat`、`pearl`、`brushed-metal`、`spectral-lines`、`etched-holo`、`cosmic-flake` 和 `star-holo`。各材质共享同一套主体隔离规则。

## 仓库结构

```text
.
├── SKILL.md                         # 工作流与约束
├── agents/openai.yaml              # Codex 展示信息和默认提示
├── scripts/                        # 检查、分层、集成与导出工具
├── references/                     # 卡面、图层、材质和输出规范
└── assets/react-template/          # React 组件、WebGL 引擎和纹理
```

## 设计边界

- 只生成原创视觉，不复制品牌卡牌系统、第三方源码或受限素材。
- 卡面不包含文字、UI、卡框或已经烘焙的光学效果。
- 背景层可以接收箔片、纹理、闪点和眩光；主体层不接收这些效果。
- WebGL2、Shader、纹理或上下文失败会终止预览，不会切换成平面 CSS 降级。

更细的输入验收、分层算法和 Presentation IR 约束请阅读 [`references/`](references/)；完整执行规则位于 [`SKILL.md`](SKILL.md)。

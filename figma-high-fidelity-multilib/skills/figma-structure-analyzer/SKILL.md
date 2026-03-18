---
name: figma-structure-analyzer
description: "Figma 转代码流程的第一步：分析阶段。当用户提供 figma.com 链接并要求「分析设计稿」「生成还原方案」「看看这个设计怎么实现」时触发，也作为完整 Figma 转代码流程的入口。负责检查环境、识别技术栈、提取 Figma 数据和截图、收集资源，最终输出模块拆分、布局关系、组件复用、不确定项、还原策略五项分析方案，等待用户确认后交由 figma-high-fidelity-frontend 技能生成代码。"
---

# figma-structure-analyzer

Figma 转代码流程的第一步：完成环境准备、数据提取、资源收集，并输出完整的还原方案供用户确认。

## 完整工作流

按顺序执行以下步骤，**不得跳过任何步骤**。

---

### 步骤 1：检查 Figma MCP 可用性

检查当前环境中是否存在 `mcp__figma-mcp__get_figma_data` 工具：

- **可用** → 进入步骤 2
- **不可用** → 执行以下引导流程后**停止**，等待用户修复后重新触发：

#### MCP 配置引导

**1. 检查 Node.js**：运行 `node -v`，不可用时提示安装（macOS: `brew install node`）

**2. 获取 Figma API Token**：
- 检查 `.mcp.json` 是否已存在并包含有效 Token
- 无效或不存在 → 提示用户前往 Figma → Settings → Security → Personal Access Tokens 创建

**3. 创建/更新 `.mcp.json`**：
```json
{
  "mcpServers": {
    "figma-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "figma-developer-mcp@0.6.4", "--stdio"],
      "env": {
        "FIGMA_API_KEY": "<用户提供的 Token>"
      }
    }
  }
}
```
- 如果文件已存在，在 `mcpServers` 中追加/更新 `figma-mcp` 条目，保留其他配置
- `--stdio` 参数必须保留，否则 Claude Code 无法连接

**4. 将 `.mcp.json` 加入 `.gitignore`（如未包含则追加）**

**5. 告知用户**：配置完成，请**重启 Claude Code** 后重新发起请求。

---

### 步骤 2：识别项目技术栈

读取项目根目录的 `package.json`（`dependencies` + `devDependencies`），按以下规则识别：

#### 框架识别
| 依赖包含 | 框架 |
|---------|-----|
| `react` | React |
| `vue` | Vue |

#### 组件库识别
| 依赖包含 | 组件库 | 额外动作 |
|---------|--------|---------|
| `antd` | Ant Design | 查找 `theme.json` / `antd-theme.json`（根目录或 `src/styles/`） |
| `shineout` | Shineout | 无 |
| `element-plus` | Element Plus | 查找 `var.scss` / `variables.scss`（`src/styles/`、`src/assets/`、`styles/`） |

**无法自动识别时**：询问用户明确指定框架和组件库。

输出识别结果，例如：
```
✅ 技术栈识别完成
框架：React
组件库：Ant Design
主题文件：src/styles/theme.json ✓
```

---

### 步骤 3：提取 Figma 设计数据

**解析 URL**：
```
https://www.figma.com/design/<fileKey>/<name>?node-id=<nodeId>
```
- `fileKey`：`/design/` 后的段落
- `nodeId`：`node-id` 参数值，URL 中用 `-` 分隔，调用工具时转为 `:` 分隔（如 `440-39537` → `440:39537`）

**调用 `get_figma_data`**：
```
get_figma_data(fileKey="<fileKey>", nodeId="<nodeId>")
```

**解析返回数据**，重点提取：
- 节点树结构（布局方式、层级关系）
- **图层名称**（`name` 字段）— 设计师通过图层命名表达了页面结构语义，必须提取并用作代码命名的参照：
  - 图层名为英文且符合规范时（如 `header`、`sidebar`、`user-info`、`action-bar`），直接作为组件名、CSS 类名或 DOM 结构的命名依据
  - 图层名为中文时（如 `顶部导航`、`用户信息区`），翻译为对应英文后用作命名参照
  - 图层名为无意义默认值时（如 `Frame 123`、`Group 45`），忽略，根据内容自行命名
  - 保持图层的嵌套层级关系与代码组件/DOM 的层级关系一致
- **Frame 布局适配方式**（关键，直接决定代码中的布局实现）：
  - `layoutMode`：Auto Layout 方向（`HORIZONTAL` → flex-direction: row / `VERTICAL` → flex-direction: column）
  - `primaryAxisSizingMode` / `counterAxisSizingMode`：主轴/交叉轴尺寸策略（`FIXED` → 固定宽高 / `AUTO` → 由内容撑开，即 width/height: auto）
  - `primaryAxisAlignItems`：主轴对齐（`MIN` → flex-start / `CENTER` → center / `MAX` → flex-end / `SPACE_BETWEEN` → space-between）
  - `counterAxisAlignItems`：交叉轴对齐（`MIN` → flex-start / `CENTER` → center / `MAX` → flex-end / `BASELINE` → baseline）
  - `layoutWrap`：是否换行（`WRAP` → flex-wrap: wrap）
  - `constraints`：非 Auto Layout 子节点的约束（`LEFT_RIGHT` → 左右拉伸 / `TOP_BOTTOM` → 上下拉伸 / `SCALE` → 按比例缩放 / `CENTER` → 居中），映射为 position、left/right/top/bottom 或百分比布局
  - `layoutSizingHorizontal` / `layoutSizingVertical`：子节点在 Auto Layout 中的填充方式（`FILL` → flex: 1 或 width: 100% / `HUG` → width: auto / `FIXED` → 固定像素值）
- 所有颜色值（fills、strokes、effects）
- 字号、字重、行高
- 间距（paddingLeft/Right/Top/Bottom、itemSpacing）
- 圆角（cornerRadius）
- 阴影效果（effects）
- 固定尺寸（width、height）
- 图片/图标节点 ID

**错误处理**：
- 数据为空 → 确认链接和节点 ID 是否正确
- 403 → 提示用户更新 Figma Token
- 404 → 链接已失效
- 数据过大 → 提示用户提供带 `node-id` 的精确链接

---

### 步骤 4：获取视觉基准截图

调用：
```
get_screenshot(fileKey="<fileKey>", nodeId="<nodeId>")
```

将截图展示给用户，并说明：
> 这是 Figma 设计稿截图，将作为后续视觉验证的基准。

---

### 步骤 5：收集图片/图标资源（自动下载 + 项目复用）

扫描步骤 3 提取的所有图片和图标节点，按以下流程处理，**优先自动获取，最后才向用户索取**。

#### 5.1 分类整理资源清单

将所有图片/图标节点分为两类：

1. **普通图片**（照片、插画、多色图形等）
2. **单色图标**（纯色的 icon，如操作图标、导航图标等）

判断依据：节点仅包含单一填充色且形状为矢量路径 → 归类为单色图标；否则归类为普通图片。

同时检查每个节点的 Figma 导出设置（`exportSettings` 字段），记录其格式（PNG/SVG/PDF/JPEG）和倍率（`constraint.value`，如 1x/2x/3x）。

#### 5.2 图标资源：优先复用项目内已有图标包

对于所有**单色图标**节点，先扫描项目内已有的图标资源：

1. **扫描项目图标包**：在项目目录中查找以下资源（按优先级）：
   - iconfont CSS 文件（`iconfont.css`、`*.css` 中含 `@font-face` 的文件）
   - SVG 图标目录（`src/icons/`、`src/assets/icons/`、`public/icons/` 等）
   - 图标组件库（`package.json` 中含 `@ant-design/icons`、`element-plus/icons-vue` 等）

2. **图标名称匹配**：根据 Figma 图层名（如 `icon/search`、`Icon=close`、`搜索图标`）推断语义，在项目图标包中查找同名或同义图标：
   - 图层名含 `search` → 匹配 `icon-search`、`SearchOutlined`、`el-icon-search` 等
   - 图层名含中文时先翻译为英文再匹配

3. **匹配结果分类**：
   - ✅ **已匹配**：记录项目中对应图标的引用方式（类名 / 组件名），代码生成阶段直接使用
   - ❌ **未匹配**：进入下一步处理

> **React + Shineout 项目**：在本步骤中额外调用 `so-functions:icon-selector` 技能辅助图标语义匹配。

#### 5.3 自动下载有导出标记的资源（MCP 工具）

对于以下两类节点，调用 `download_figma_images` 自动下载：
- **有导出标记的普通图片**（`exportSettings` 不为空）
- **未在项目图标包中匹配到的单色图标**，且该节点有导出标记

**下载参数规则**：
- `format`：使用 Figma 导出设置中指定的格式（PNG/SVG/JPEG）；图标优先使用 SVG
- `pngScale`：使用 Figma 导出设置中的倍率（`constraint.value`）；默认 2
- `localPath`：根据文件类型存放到项目合适目录（图片 → `src/assets/images/`，图标 → `src/assets/icons/`），如果项目有约定目录则遵从项目规范
- `fileName`：使用图层名转为 kebab-case 加扩展名（如 `hero-image.png`、`icon-search.svg`）

下载完成后，记录各文件的本地路径供代码生成阶段引用。

#### 5.4 向用户索取剩余无法自动获取的资源

仅对以下情况向用户索取：
- 有图片/图标节点但 **Figma 中未设置导出标记**，且无法从项目图标包匹配

输出清单，格式示例：
```
📋 以下资源需要您提供（其余资源已自动下载或已匹配项目图标）：

【需要您提供的普通图片】（共 N 张，Figma 中未设置导出标记）
1. banner 图 (节点: "Hero Image", 建议尺寸: 1200x400)
...
请提供以上图片文件，或告知存放路径。

【需要您提供的图标】（共 M 个，Figma 中未设置导出标记且项目内未找到匹配图标）
1. 自定义图标 (节点: "icon/custom-logo")
...
请提供图标文件或字体图标的引用地址及类名映射。
```

如果所有资源都已自动处理，则跳过此步骤，输出汇总：
```
✅ 资源收集完成
- 已从项目图标包匹配：X 个图标
- 已通过 Figma 导出标记自动下载：Y 张图片 / Z 个图标
- 无需手动提供
```

#### 5.5 等待用户响应（仅在 5.4 有内容时执行）

- **普通图片**：用户提供文件后，记录本地路径供代码生成阶段引用
- **图标**：用户提供字体图标地址和映射关系后，记录引用方式（如 `<i class="icon-search" />` 或 `<IconFont type="icon-search" />`）
- 如果用户表示某些资源暂时没有，在代码中使用占位符并添加 `TODO` 注释标注

---

### 步骤 6：输出还原方案

#### 6.1 准备工作

读取对应技术栈参考文件，并调用 `so-functions:so-design` 技能查阅 SHEIN 设计规范：
- React + Ant Design → 读取 `../figma-high-fidelity-frontend/references/react-antd.md`，调用 `so-functions:so-design`
- React + Shineout → 读取 `../figma-high-fidelity-frontend/references/react-shineout.md`，调用 `so-functions:icon-selector`，调用 `so-functions:so-design`
- Vue + Element Plus → 读取 `../figma-high-fidelity-frontend/references/vue-element-plus.md`，调用 `so-functions:so-design`

同时读取项目中 2-3 个同类组件/页面文件，了解：
- 文件组织方式（单文件 vs 目录结构）
- 样式方案（CSS Modules / styled-components / Less / Sass / scoped CSS）
- 组件写法（函数式/类组件、Props 类型定义）
- 命名约定（文件名大小写、CSS 类名）
- TypeScript 严格程度

建立颜色 token 映射表：

**Ant Design 项目**：
- 读取 `theme.json`，提取所有 token 键值对
- 建立映射表：`Figma 颜色值` → `token 变量名`

**Element Plus 项目**：
- 读取 `var.scss`，从 `$colors` map 提取 `base` 颜色值
- 建立映射表：`Figma 颜色值` → `CSS 变量名`

**Shineout 项目**：
- 语义颜色使用 `type` prop，非语义颜色使用项目现有 CSS/Less 变量

#### 6.2 输出五项还原方案

基于步骤 3 的 Figma 数据，输出以下五项分析，**参照 `../shared/fidelity-checklist.md` 逐项确认不确定项**，然后**等待用户确认后再交由 figma-high-fidelity-frontend 生成代码**：

**① 模块拆分**

将页面/组件拆分为独立模块，列出每个模块的名称、对应 Figma 图层和文件路径：

```
模块列表：
- PageHeader     对应图层：header          → src/components/PageHeader.tsx
- FilterBar      对应图层：filter-bar      → src/components/FilterBar.tsx
- DataTable      对应图层：table-content   → src/components/DataTable.tsx
- Pagination     对应图层：pagination      → src/components/Pagination.tsx
```

**② 布局关系**

描述模块间的嵌套结构和 flex 布局方向：

```
布局结构：
PageLayout（vertical flex）
├── PageHeader（horizontal flex，space-between）
├── FilterBar（horizontal flex，gap: 12px）
├── DataTable（block，width: 100%）
└── Pagination（horizontal flex，center，margin-top: 16px）
```

**③ 组件复用**

标注哪些 Figma 元素可复用项目内已有组件，哪些需要新建：

```
复用已有组件：
- <Button>       → antd Button（Figma: action-btn）
- <Table>        → antd Table（Figma: table-content）
- <Pagination>   → antd Pagination（Figma: pagination）

需新建组件：
- <FilterBar>    项目内无同类组件，需新建
- <StatusTag>    现有 Tag 样式不符，需定制
```

**④ 不确定项**

列出 Figma 数据缺失、设计不完整或有歧义的部分，参照 `../shared/fidelity-checklist.md` 逐项检查是否遗漏，每项注明处理预案：

```
不确定项：
1. FilterBar 折叠态：设计稿只有展开态，折叠交互未设计
   → 预案：暂不实现折叠，仅还原展开态，添加 TODO 注释
2. 表格空状态：无空数据设计稿
   → 预案：使用 antd Table 默认 empty 样式
3. icon/export 图标：项目图标包未匹配，Figma 无导出标记
   → 预案：等待用户提供，暂用占位符
```

**⑤ 还原策略**

说明整体代码组织方式和关键还原决策：

```
还原策略：
- 文件结构：每个模块独立文件，统一在 index.tsx 中组合
- 样式方案：CSS Modules（与项目现有一致）
- 布局实现：全部使用 flexbox，不引入 Grid
- 颜色处理：主色 #197afa 映射 token.colorPrimary，其余颜色 hardcode 后交由检查阶段统一替换为 token
- 响应式：仅还原设计稿断点（1280px），不扩展其他断点
```

---

## 下一步

还原方案确认后，调用 **`figma-high-fidelity-frontend`** 技能按模块生成代码。

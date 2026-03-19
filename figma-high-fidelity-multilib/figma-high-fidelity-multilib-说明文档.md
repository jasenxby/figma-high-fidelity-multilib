# figma-high-fidelity-multilib 技能说明文档

> 将 Figma 设计稿 1:1 还原为项目技术栈代码的 Claude Code 技能。

---

## 触发方式

只要同时满足以下两个条件，技能即自动触发：

1. 对话中包含 `figma.com` 链接
2. 有明确的代码实现意图

**以下表达均可触发：**

```
帮我把这个做出来  https://figma.com/design/...
按这个设计写代码
实现这个页面
还原设计稿
生成代码
```

---

## 支持的技术栈

| 框架 | 组件库 | 主题文件 |
|------|--------|---------|
| React | Ant Design | `theme.json` / `antd-theme.json` |
| React | Shineout | — |
| Vue | Element Plus | `var.scss` / `variables.scss` |

框架和组件库通过读取 `package.json` 自动识别，无法识别时会询问用户。

---

## 前置要求：Figma MCP 配置

技能依赖 Figma MCP 工具读取设计数据。首次使用前需完成以下配置：

### 1. 安装 Node.js

```bash
node -v  # 检查是否已安装
brew install node  # macOS 安装方式
```

### 2. 获取 Figma API Token

Figma → Settings → Security → Personal Access Tokens → 创建新 Token

### 3. 创建 `.mcp.json`（项目根目录）

```json
{
  "mcpServers": {
    "figma-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "figma-developer-mcp@0.6.4", "--stdio"],
      "env": {
        "FIGMA_API_KEY": "你的 Token"
      }
    }
  }
}
```

### 4. 将 `.mcp.json` 加入 `.gitignore`

### 5. 重启 Claude Code

---

## 完整工作流（7 步）

### 步骤 1　检查 Figma MCP 可用性

检测 `mcp__figma-mcp__get_figma_data` 工具是否存在：
- 可用 → 继续
- 不可用 → 引导完成上方 MCP 配置后停止，等待重启

---

### 步骤 2　识别项目技术栈

读取 `package.json`，自动识别框架和组件库，同时查找主题文件：

```
✅ 技术栈识别完成
框架：React
组件库：Ant Design
主题文件：src/styles/theme.json ✓
```

---

### 步骤 3　提取 Figma 设计数据

解析 Figma URL，调用 `get_figma_data` 提取完整设计参数：

**布局相关**

| Figma 字段 | 含义 | CSS 映射 |
|-----------|------|---------|
| `layoutMode` | Auto Layout 方向 | `flex-direction: row/column` |
| `primaryAxisAlignItems` | 主轴对齐 | `justify-content` |
| `counterAxisAlignItems` | 交叉轴对齐 | `align-items` |
| `layoutSizingHorizontal/Vertical` | 子元素填充方式（FILL/HUG/FIXED） | `flex:1 / auto / 固定值` |
| `layoutWrap` | 是否换行 | `flex-wrap` |
| `constraints` | 非 Auto Layout 约束 | `position / left / right` |

**视觉相关**：颜色、字号、字重、行高、内边距、间距、圆角、阴影、固定尺寸

**图层命名规则**：
- 英文规范命名（如 `header`、`user-info`）→ 直接用于组件名和 CSS 类名
- 中文命名（如 `顶部导航`）→ 翻译为英文后使用
- 无意义命名（如 `Frame 123`）→ 忽略，根据内容自行命名

---

### 步骤 4　获取视觉基准截图

调用 `get_screenshot` 获取设计稿截图，作为步骤 7 视觉对比的基准。

---

### 步骤 5　收集图片 / 图标资源

**处理优先级（由高到低）：**

```
1. 项目内图标包匹配（iconfont / SVG 目录 / 图标组件库）
   ↓ 未匹配
2. Figma 导出标记自动下载（调用 download_figma_images）
   ↓ 无导出标记
3. 向用户索取
```

**React + Shineout 项目**额外调用 `icon-selector` 技能辅助图标匹配。

下载目录约定：
- 图片 → `src/assets/images/`
- 图标 → `src/assets/icons/`

---

### 步骤 6　生成代码（两阶段）

步骤 6 分为 **阶段 A（还原方案分析）** 和 **阶段 B（按模块生成代码）** 两个阶段。

#### 阶段 A：还原方案分析

生成任何代码前，先完成准备工作（读取技术栈参考文件、调用 `so-design`、建立颜色 token 映射、学习项目代码风格），再输出以下五项分析，**等待用户确认后再开始写代码**：

| 分析项 | 内容 |
|--------|------|
| ① 模块拆分 | 页面拆分为哪些模块、对应 Figma 图层、文件路径 |
| ② 布局关系 | 模块间嵌套结构和 flex 布局方向 |
| ③ 组件复用 | 哪些复用已有组件、哪些需新建 |
| ④ 不确定项 | Figma 数据缺失/歧义处及处理预案 |
| ⑤ 还原策略 | 文件结构、样式方案、颜色处理等关键决策 |

**颜色 token 映射规则：**

| 项目类型 | 映射方式 |
|---------|---------|
| Ant Design | Figma 颜色值 → `token.colorPrimary` / `var(--ant-color-primary)` |
| Element Plus | Figma 颜色值 → `var(--el-color-*)` |
| Shineout | 语义颜色用 `type` prop，其余用项目 CSS/Less 变量 |

所有技术栈均调用 `so-design` 技能查阅 SHEIN 设计规范；**React + Shineout 项目**额外调用 `icon-selector` 辅助图标匹配。

#### 阶段 B：按模块生成代码

用户确认还原方案后，按以下顺序逐步生成，**每步单独输出**：

| 顺序 | 内容 |
|------|------|
| 6.B1 基础容器布局 | 最外层骨架，只含结构和布局样式，不填充内容 |
| 6.B2 主要组件 | 在骨架内填充组件库组件，Props 与 Figma 严格对应 |
| 6.B3 文本和图标 | 补全文本内容（含 token 颜色）和图标引用 |
| 6.B4 样式还原 | 补全圆角/阴影/边框/交互状态，颜色替换为 token，输出完整文件 |

---

### 步骤 7　视觉验证与质量检查

#### 还原铁律（不可违反）

1. 不允许擅自美化
2. 不允许自行调整配色
3. 不允许自行改变字号、圆角、间距、留白比例
4. 不允许把布局改造为"更合理"的布局
5. 不允许添加设计稿中没有的元素
6. 不允许删除设计稿中已有的元素
7. 不允许擅自简化多层结构
8. **与 Figma 视觉一致是最高优先级**

#### 7.1 参数自动对照

对照 Figma 数据与生成代码，逐项检查并自动修正不一致项，输出报告：

```
✅ 参数对照完成（已自动修正 3 处不一致）
- 颜色：全部映射到 token/CSS 变量 ✓
- 字号：14px/16px/24px 已还原 ✓
- 间距：8px/16px/24px 已还原 ✓
- 圆角：4px/8px 已还原 ✓
```

#### 7.2 截图对比验证

1. 展示步骤 4 的 Figma 截图
2. 请用户在浏览器渲染后截图上传
3. 逐区域对比差异（布局、颜色、间距、字体、图片/图标）
4. 给出精确修复方案并执行
5. 重复迭代直到用户确认通过

#### 7.3 代码质量最终检查

- [ ] 无硬编码颜色值
- [ ] 所有图片/图标使用本地引用，无外链 URL
- [ ] import 路径合法，无未使用 import
- [ ] 组件 Props 符合当前版本 API 规范
- [ ] TypeScript 类型完整（与项目严格程度一致）
- [ ] 调用 `so-design` 进行 SHEIN 设计规范检查（适用于所有技术栈）

#### 7.4 隐性质量检查（ui-ux-pro-max）

调用 `ui-ux-pro-max` 执行 `check` 动作，检查截图对比发现不了的隐性问题：

| 检查项 | 标准 |
|--------|------|
| 无障碍对比度 | 文字/背景色对比度满足 WCAG AA（4.5:1） |
| 焦点状态 | 可交互元素有可见的 focus 样式 |
| 交互状态完整性 | hover / disabled / loading 均已还原 |
| 响应式断点 | 固定尺寸元素不在小屏设备溢出 |
| 语义化 HTML | 标题层级、ARIA 属性、表单 label 关联正确 |

**处理规则**：
- Figma 中有对应设计的问题 → 立即修复
- Figma 中未设计的（如 focus ring）→ 提示用户，询问是否补充，不擅自添加

---

## 集成的外部技能

| 技能 | 触发场景 | 作用 |
|------|---------|------|
| `so-functions:icon-selector` | React + Shineout 项目 | 图标语义匹配 |
| `so-functions:so-design` | 所有项目，步骤 6/7 | SHEIN 设计规范检查 |
| `ui-ux-pro-max` | 所有项目，步骤 7.4 | 无障碍/响应式/语义化隐性检查 |

---

## 常见问题

**Q：Figma 链接没有 `node-id` 参数怎么办？**
A：在 Figma 中选中目标 Frame，URL 会自动带上 `node-id`，复制此时的 URL 即可。

**Q：数据过大报错怎么办？**
A：提供带精确 `node-id` 的链接（选中具体 Frame 而非整个页面）。

**Q：图标在项目里找不到匹配怎么办？**
A：Figma 节点有导出标记时会自动下载 SVG；无导出标记时会提示你提供图标文件或字体图标地址。

**Q：颜色值与 Figma 不一致怎么办？**
A：步骤 7.1 会自动检测并修正；若仍有差异，上传浏览器截图后可逐区域修复。

**Q：生成的代码风格与项目不一致怎么办？**
A：技能在步骤 6.2 会读取项目内 2-3 个现有文件学习风格，如仍不一致请在对话中指出具体差异。

---
name: figma-high-fidelity-frontend
description: "Figma 转代码流程的第二步：实现阶段。在 figma-structure-analyzer 输出还原方案且用户确认后触发。严格遵守还原铁律，按「基础容器布局 → 主要组件 → 文本和图标 → 样式还原」顺序逐步生成代码，每步单独输出，不合并为一次性输出。代码生成完成后提示调用 figma-fidelity-repair 进行验证和修复。"
---

# figma-high-fidelity-frontend

Figma 转代码流程的第二步：基于 figma-structure-analyzer 确认的还原方案，按模块顺序高保真生成代码。

## 前置条件

执行本技能前，必须已完成：
- `figma-structure-analyzer` 输出的五项还原方案（模块拆分、布局关系、组件复用、不确定项、还原策略）
- 用户已明确确认上述方案

---

## 还原铁律（贯穿整个代码生成过程，违反即视为不合格）

> **严格遵守以下规则，任何调整必须有 Figma 数据依据，不得凭主观判断改动设计：**
>
> 1. **不允许擅自美化**——不得以"更好看"为由修改任何视觉属性
> 2. **不允许自行调整配色**——所有颜色必须与 Figma 完全一致，不得替换或调和
> 3. **不允许自行改变字号、圆角、间距、留白比例**——必须 1:1 还原 Figma 中的数值
> 4. **不允许把设计稿中的布局改造为"更合理"的布局**——保留原有结构，不重新设计
> 5. **不允许添加设计稿中没有的图标、分割线、阴影、渐变、动效**——有且仅有 Figma 中存在的元素
> 6. **不允许删除设计稿中已有的元素**——所有可见节点均须在代码中体现
> 7. **不允许擅自简化多层结构**——若简化会影响视觉还原，必须保留原始层级
> 8. **最终效果与 Figma 视觉一致是最高优先级**——其他所有工程考量（简洁、可维护性）均服从于此

---

## 代码生成顺序

按以下四步顺序逐步生成，**每步完成后单独输出，不合并为一次性输出**。

---

### 第一步：基础容器布局

优先生成最外层容器和页面骨架，**只包含结构和布局样式，不填充具体内容**：

- 外层 wrapper、layout 容器
- 各模块的占位结构（含正确的 flex 方向、gap、padding）
- 文件框架（import 语句占位、组件导出）

**要求**：
- flex 方向、对齐方式、gap、padding 必须与 figma-structure-analyzer 输出的布局关系完全一致
- 固定尺寸节点使用 Figma 中的 width/height 数值，FILL 节点使用 `flex: 1` 或 `width: 100%`，HUG 节点使用 `width: auto`

---

### 第二步：主要组件

在骨架基础上，逐模块填充组件库组件：

- 遵循 `references/` 目录下对应技术栈参考文件中的组件用法
- Props 与 Figma 数据严格对应（尺寸、变体、状态）
- 不确定项按 figma-structure-analyzer 输出的预案处理，添加 `TODO` 注释标注

**组件选用原则**：
- 有对应组件库组件时，必须使用该组件，不得用原生 HTML 元素替代
- 组件 Props 版本以项目 `package.json` 中的版本为准

---

### 第三步：文本和图标

补全所有文本内容和图标引用：

**文本**：
- 还原字号、字重、行高
- 颜色优先映射到 token/CSS 变量（参照 figma-structure-analyzer 建立的颜色映射表）
- 若项目有 i18n，文本包裹翻译函数

**图标**：
- 使用 figma-structure-analyzer 步骤 5 中匹配或下载的图标
- 按项目图标方案引用（iconfont 类名 / SVG 组件 / 图标库组件）
- 未获取到图标的，使用占位符并添加 `TODO` 注释

---

### 第四步：样式还原

补全所有样式细节，确保与 Figma 数值 1:1：

- 圆角（cornerRadius）
- 阴影（box-shadow，来自 effects[type=DROP_SHADOW]）
- 边框（border，来自 strokes）
- hover / active / disabled / loading / focus 等交互状态样式
- 颜色全部替换为 token 或 CSS 变量，确保无硬编码残留

完成后，将完整文件直接写入项目目录（根据项目结构推断合适位置，或询问用户）。

---

## 技术栈参考文件

代码生成过程中，按技术栈读取对应参考文件：

| 技术栈 | 参考文件 |
|--------|---------|
| React + Ant Design | `references/react-antd.md` |
| React + Shineout | `references/react-shineout.md` |
| Vue + Element Plus | `references/vue-element-plus.md` |

---

## 下一步

所有模块代码生成完成后，调用 **`figma-fidelity-repair`** 技能进行参数对照、截图验证和质量检查。

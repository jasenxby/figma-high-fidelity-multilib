---
name: figma-fidelity-repair
description: "Figma 转代码流程的第三步：检查修复阶段。在 figma-high-fidelity-frontend 代码生成完成后触发，也可在用户主动要求「检查还原度」「对比设计稿」「修复差异」时单独触发。参照 shared/fidelity-checklist.md 逐项核查，执行参数自动对照、截图对比验证、代码质量检查和隐性质量检查。有 Figma 数据依据的问题立即修复，无设计依据的问题提示用户确认，不擅自添加。"
---

# figma-fidelity-repair

Figma 转代码流程的第三步：全面核查代码与设计稿的还原度，自动修复可修复的差异，并对隐性质量问题给出处理建议。

## 前置条件

执行本技能前，需具备：
- figma-high-fidelity-frontend 生成的代码
- figma-structure-analyzer 步骤 4 获取的 Figma 设计稿截图（视觉基准）
- figma-structure-analyzer 步骤 3 提取的 Figma 设计参数

---

## 还原铁律（核查过程中全程有效）

> 1. **不允许擅自美化**——不得以"更好看"为由修改任何视觉属性
> 2. **不允许自行调整配色**——所有颜色必须与 Figma 完全一致
> 3. **不允许自行改变字号、圆角、间距、留白比例**——必须 1:1 还原 Figma 中的数值
> 4. **不允许把设计稿中的布局改造为"更合理"的布局**
> 5. **不允许添加设计稿中没有的图标、分割线、阴影、渐变、动效**
> 6. **不允许删除设计稿中已有的元素**
> 7. **不允许擅自简化多层结构**
> 8. **与 Figma 视觉一致是最高优先级**

---

## 检查流程

### 第一步：参数自动对照

对照 Figma 设计参数与生成代码，逐项检查以下 14 项，**发现不一致立即修正，无需等待用户指出**：

| 检查项 | Figma 来源字段 | 代码对应属性 |
|--------|--------------|------------|
| 布局方向 | layoutMode | display: flex + flex-direction |
| 主轴对齐 | primaryAxisAlignItems | justify-content |
| 交叉轴对齐 | counterAxisAlignItems | align-items |
| 子元素填充 | layoutSizingHorizontal/Vertical（FILL/HUG/FIXED） | flex: 1 / width: auto / 固定值 |
| 换行 | layoutWrap | flex-wrap |
| 约束 | constraints（非 Auto Layout 子节点） | position / left / right / 百分比 |
| 颜色 | fills.color | 对应 token / CSS 变量 |
| 字号 | fontSize | font-size |
| 字重 | fontWeight | font-weight |
| 行高 | lineHeight | line-height |
| 内边距 | paddingLeft/Right/Top/Bottom | padding |
| 子元素间距 | itemSpacing | gap |
| 圆角 | cornerRadius | border-radius |
| 阴影 | effects[type=DROP_SHADOW] | box-shadow |
| 固定宽高 | width / height（非 FILL/HUG） | width / height |

修正完成后输出报告：
```
✅ 参数对照完成（已自动修正 N 处不一致）
- 颜色：全部映射到 token/CSS 变量 ✓
- 字号：14px/16px/24px 已还原 ✓
- 间距：8px/16px/24px 已还原 ✓
- 圆角：4px/8px 已还原 ✓
```

---

### 第二步：截图对比验证

1. 展示 Figma 设计稿截图（视觉基准）
2. 提示用户：
   > 请在浏览器中渲染生成的页面，截图后上传到对话中，我将对比两张图找出差异并修复。

3. 用户上传截图后，逐区域分析差异：
   - 整体布局结构（对齐、排列、比例）
   - 颜色差异（主色、背景色、文字颜色）
   - 间距差异（明显的过大/过小间距）
   - 字体大小/粗细差异
   - 图片/图标是否正确加载

4. 对每处差异给出精确修复方案并执行，然后再次提示截图对比
5. 重复迭代直到用户确认通过，或用户主动结束

---

### 第三步：保真度清单核查

参照 `../shared/fidelity-checklist.md` 逐项核查，确认所有项均已还原：

- [ ] 页面层级是否与设计稿一致
- [ ] Auto Layout 是否正确映射为 flex / grid
- [ ] 间距是否一致（padding、gap、margin）
- [ ] 字号 / 行高 / 字重是否一致
- [ ] 圆角是否一致
- [ ] 边框是否一致
- [ ] 阴影是否一致
- [ ] 图标与文字相对位置是否一致
- [ ] 图片比例与裁切方式是否一致
- [ ] 是否新增了设计稿中不存在的元素
- [ ] 是否遗漏了设计稿中已有元素
- [ ] 所有推断项是否已单独列出

同时检查代码质量：
- [ ] 无硬编码颜色值（Ant Design 用 token，Element Plus 用 `var(--el-color-*)`）
- [ ] 所有图片引用本地文件，无外链 URL
- [ ] import 路径合法，无未使用的 import
- [ ] 组件 Props 符合当前版本 API 规范
- [ ] TypeScript 类型完整（与项目现有严格程度一致）
- [ ] 调用 `so-functions:so-design` 对页面进行 SHEIN 设计规范检查，根据检查结果优化代码（适用于所有技术栈）

---

### 第四步：隐性质量检查

调用 `ui-ux-pro-max` 技能执行 `check` 动作，检查截图对比发现不了的隐性问题：

| 检查项 | 标准 |
|--------|------|
| 无障碍对比度 | 文字与背景色对比度满足 WCAG AA（4.5:1） |
| 焦点状态 | 可交互元素有可见的 focus 样式 |
| 交互状态完整性 | hover / disabled / loading 均已还原 |
| 响应式断点 | 固定尺寸元素不在小屏设备溢出 |
| 语义化 HTML | 标题层级、ARIA 属性、表单 label 关联正确 |

**处理规则**：
- **Figma 中有对应设计的**（如 disabled 状态样式）→ 立即修复代码
- **Figma 中未设计的**（如 focus ring、ARIA 属性）→ 提示用户，询问是否补充，不擅自添加

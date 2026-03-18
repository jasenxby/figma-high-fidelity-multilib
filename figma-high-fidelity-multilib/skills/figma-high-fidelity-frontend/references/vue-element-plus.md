# Vue + Element Plus 专项指南

## 主题变量文件处理

### 查找策略

按优先级依次查找以下路径：
1. `src/styles/var.scss`
2. `src/assets/var.scss`
3. `styles/var.scss`
4. `src/styles/variables.scss`
5. `src/styles/element-variables.scss`

### 解析 `$colors` map

`var.scss` 中的 `$colors` map 结构：
```scss
$colors: map.deep-merge(
  (
    'primary': ('base': #197AFA),
    'success': ('base': #00A85F),
    'warning': ('base': #F56C0A),
    'danger':  ('base': #EB4242),
    'error':   ('base': #EB4242),
    'info':    ('base': #909399),
  ),
  $colors
);
```

### 颜色 → CSS 变量映射表

| Figma 颜色（hex） | CSS 变量 | 适用场景 |
|------------------|---------|---------|
| `#197AFA`（或接近的主色）| `var(--el-color-primary)` | 主按钮、链接、选中态 |
| `#00A85F`（或接近的绿色）| `var(--el-color-success)` | 成功状态 |
| `#F56C0A`（或接近的橙色）| `var(--el-color-warning)` | 警告状态 |
| `#EB4242`（或接近的红色）| `var(--el-color-danger)` | 危险/错误状态 |
| `#909399` | `var(--el-color-info)` | 信息/中性状态 |
| 衍生浅色（primary-light-*）| `var(--el-color-primary-light-N)` | 背景色、悬浮态 |
| 文字主色 | `var(--el-text-color-primary)` | 主要文字 |
| 文字次级 | `var(--el-text-color-regular)` | 次要文字 |
| 文字禁用 | `var(--el-text-color-placeholder)` | 占位符/禁用 |
| 边框颜色 | `var(--el-border-color)` | 边框 |
| 背景色 | `var(--el-bg-color)` | 卡片背景 |
| 页面背景 | `var(--el-bg-color-page)` | 页面底色 |

**颜色匹配策略**：
1. 将 Figma 颜色值（hex）与 `var.scss` 中的 base 值精确匹配
2. 精确匹配到 → 替换为对应 CSS 变量
3. 近似但不完全相同 → 检查是否是 base 色的衍生（混合了白色/黑色的透明变体），用 `var(--el-color-primary-light-N)` 或 opacity 变体
4. 确实无法映射 → 保留字面量，添加注释

---

## 核心组件映射表

所有 Element Plus 组件均以 `el-` 前缀使用（自动导入时不需要手动注册）。

### 布局
| 设计元素 | Element Plus 组件 | 关键 Props/特性 |
|---------|-----------------|---------------|
| 水平/垂直弹性容器 | `el-row` + `el-col` | `:gutter`, `:span`, `:offset` |
| 自适应间距 | `el-space` | `size`, `direction`, `wrap` |
| 卡片 | `el-card` | `header`（slot）, `shadow` |
| 页面布局 | `el-container` + `el-header/aside/main/footer` | — |
| 模态弹窗 | `el-dialog` | `v-model`, `title`, `width` |
| 抽屉 | `el-drawer` | `v-model`, `direction`, `size` |
| 标签页 | `el-tabs` | `v-model`, `type` |
| 折叠面板 | `el-collapse` | `v-model`, `accordion` |

### 导航
| 设计元素 | Element Plus 组件 | 关键 Props |
|---------|-----------------|-----------|
| 顶部导航 | `el-menu` mode="horizontal" | `:default-active`, `@select` |
| 侧边菜单 | `el-menu` mode="vertical" | `:collapse` |
| 面包屑 | `el-breadcrumb` | `separator` |
| 分页 | `el-pagination` | `v-model:current-page`, `v-model:page-size`, `:total`, `layout` |
| 步骤条 | `el-steps` | `:active`, `finish-status` |

### 数据录入
| 设计元素 | Element Plus 组件 | 关键 Props |
|---------|-----------------|-----------|
| 单行输入 | `el-input` | `v-model`, `placeholder`, `clearable`, `:prefix-icon`, `:suffix-icon` |
| 多行文本 | `el-input` type="textarea" | `:rows`, `:autosize` |
| 下拉选择 | `el-select` + `el-option` | `v-model`, `placeholder`, `multiple`, `filterable` |
| 日期选择 | `el-date-picker` | `v-model`, `type`, `format`, `placeholder` |
| 日期范围 | `el-date-picker` type="daterange" | `v-model`, `start-placeholder`, `end-placeholder` |
| 数字输入 | `el-input-number` | `v-model`, `:min`, `:max`, `:step`, `:precision` |
| 复选框 | `el-checkbox` / `el-checkbox-group` | `v-model`, `label` |
| 单选框 | `el-radio` / `el-radio-group` | `v-model`, `label` |
| 开关 | `el-switch` | `v-model`, `active-text`, `inactive-text` |
| 上传 | `el-upload` | `action`, `accept`, `:limit`, `list-type` |
| 表单 | `el-form` + `el-form-item` | `ref`, `:model`, `:rules`, `label-width` |

### 数据展示
| 设计元素 | Element Plus 组件 | 关键 Props |
|---------|-----------------|-----------|
| 表格 | `el-table` + `el-table-column` | `:data`, `border`, `stripe`, `@selection-change` |
| 描述列表 | `el-descriptions` + `el-descriptions-item` | `title`, `:column`, `border` |
| 标签 | `el-tag` | `type`, `size`, `closable`, `@close` |
| 头像 | `el-avatar` | `:src`, `shape`, `size` |
| 进度条 | `el-progress` | `:percentage`, `type`, `status` |
| 空状态 | `el-empty` | `description`, `image` |
| 时间线 | `el-timeline` + `el-timeline-item` | `timestamp`, `type` |
| 树形 | `el-tree` | `:data`, `show-checkbox`, `node-key`, `@check` |
| 数字统计 | `el-statistic` | `title`, `:value`, `prefix`, `suffix` |

### 操作反馈
| 设计元素 | Element Plus 组件/API | 用法 |
|---------|---------------------|-----|
| 主按钮 | `el-button` type="primary" | — |
| 默认按钮 | `el-button` | — |
| 文字按钮 | `el-button` text | — |
| 危险按钮 | `el-button` type="danger" | — |
| 加载按钮 | `el-button` :loading | — |
| 全局提示 | `ElMessage.success/error/warning` | 命令式 |
| 通知 | `ElNotification` | 命令式 |
| 工具提示 | `el-tooltip` | `content`, `placement` |
| 弹出气泡 | `el-popover` | `content`, `trigger` |
| 弹出确认 | `el-popconfirm` | `title`, `@confirm` |
| 加载遮罩 | `v-loading` / `ElLoading.service` | 指令或命令式 |

---

## 代码模式示例

### 页面模板结构（Vue 3 Composition API）

```vue
<template>
  <div class="page-container">
    <!-- 搜索区域 -->
    <el-card class="search-card">
      <el-form :model="searchForm" inline>
        <el-form-item label="关键词">
          <el-input v-model="searchForm.keyword" placeholder="请输入" clearable />
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="searchForm.status" placeholder="请选择" clearable>
            <el-option label="启用" value="1" />
            <el-option label="禁用" value="0" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleSearch">查询</el-button>
          <el-button @click="handleReset">重置</el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 表格区域 -->
    <el-card>
      <el-table :data="tableData" border>
        <el-table-column prop="name" label="名称" min-width="120" />
        <el-table-column label="操作" width="120" fixed="right">
          <template #default="{ row }">
            <el-button type="primary" text @click="handleEdit(row)">编辑</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        v-model:current-page="pagination.page"
        v-model:page-size="pagination.pageSize"
        :total="pagination.total"
        layout="total, sizes, prev, pager, next"
        @change="fetchData"
      />
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import { ElMessage } from 'element-plus';

const searchForm = reactive({ keyword: '', status: '' });
const tableData = ref([]);
const pagination = reactive({ page: 1, pageSize: 10, total: 0 });

const handleSearch = () => { pagination.page = 1; fetchData(); };
const handleReset = () => { Object.assign(searchForm, { keyword: '', status: '' }); handleSearch(); };
</script>

<style scoped>
.page-container { padding: 16px; background: var(--el-bg-color-page); }
.search-card { margin-bottom: 16px; }
</style>
```

---

## 样式规范

### CSS 变量使用原则

```scss
// ✅ 正确：使用 CSS 变量
.title { color: var(--el-text-color-primary); }
.card { background: var(--el-bg-color); border: 1px solid var(--el-border-color); }
.primary-text { color: var(--el-color-primary); }

// ❌ 错误：硬编码品牌色
.title { color: #141737; }
.primary-text { color: #197AFA; }
```

### scoped 样式

所有样式默认使用 `<style scoped>`，不污染全局命名空间。如需覆盖 Element Plus 组件内部样式，使用 `:deep()`：

```scss
<style scoped>
// 覆盖 el-table 内部样式
:deep(.el-table__header) {
  background: var(--el-fill-color-light);
}
</style>
```

---

## 注意事项

1. **v-model 双向绑定**：Element Plus 组件均通过 `v-model` 绑定数据，弹窗/抽屉用 `v-model` 控制显隐（`true`/`false`）
2. **自动导入**：如项目配置了 `unplugin-auto-import` 和 `unplugin-vue-components`，无需手动 import 组件，直接使用 `el-*` 标签
3. **图标引入**：Element Plus 图标从 `@element-plus/icons-vue` 导入，作为 `:prefix-icon` 或 `component :is` 使用
4. **命令式 API**：`ElMessage`、`ElNotification`、`ElMessageBox` 需要显式 import（自动导入插件通常会处理这个）
5. **主题颜色保持一致**：始终从 `var.scss` 的 base 颜色建立映射，不允许与 `$colors` 中不同的颜色硬编码

## var.scss 颜色提取示例

当读取到如下 `var.scss` 内容时：
```scss
'primary': ('base': #197AFA),
'success': ('base': #00A85F),
'warning': ('base': #F56C0A),
'danger':  ('base': #EB4242),
```

建立映射后，代码生成规则：
- Figma 节点颜色 = `#197AFA` 或 `rgba(25, 122, 250, 1)` → 输出 `var(--el-color-primary)`
- Figma 节点颜色 = `#00A85F` → 输出 `var(--el-color-success)`
- 以此类推

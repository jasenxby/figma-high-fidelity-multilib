# React + Shineout 专项指南

## Import 规范

```tsx
import { Button, Input, Select, Table } from 'shineout';
// 或使用具名导入（按需）
import Button from 'shineout/Button';
```

检查项目中的 `package.json` 确认 Shineout 版本（1.x / 2.x / 3.x），不同版本 API 可能有差异。

---

## 核心组件映射表

### 布局
| 设计元素 | Shineout 组件 | 关键 Props |
|---------|-------------|-----------|
| 栅格布局 | `Grid.Row` + `Grid.Col` | `gutter`, `width` |
| 卡片 | `Card` | `title`, `extra`, `collapsible` |
| 模态弹窗 | `Modal` | `visible`, `onClose`, `title` |
| 抽屉 | `Drawer` | `visible`, `placement`, `width` |
| 标签页 | `Tabs` | `active`, `onChange`, `tabs` |

### 数据录入
| 设计元素 | Shineout 组件 | 关键 Props |
|---------|-------------|-----------|
| 单行文本 | `Input` | `placeholder`, `clearable`, `prefix`, `suffix` |
| 多行文本 | `Input.Textarea` | `rows`, `autosize` |
| 下拉选择 | `Select` | `data`, `keygen`, `renderItem`, `placeholder`, `multiple` |
| 日期选择 | `Datepicker` | `type`, `format`, `placeholder` |
| 数字输入 | `Input.Number` | `min`, `max`, `step` |
| 复选框 | `Checkbox` / `Checkbox.Group` | `checked`, `onChange`, `data` |
| 单选框 | `Radio` / `Radio.Group` | `value`, `onChange`, `data` |
| 开关 | `Switch` | `checked`, `onChange`, `size` |
| 上传 | `Upload` | `action`, `accept`, `multiple`, `limit` |
| 表单 | `Form` + `Form.Item` | `labelWidth`, `onSubmit` |

### 数据展示
| 设计元素 | Shineout 组件 | 关键 Props |
|---------|-------------|-----------|
| 表格 | `Table` | `columns`, `data`, `keygen`, `pagination` |
| 列表 | `List` | `data`, `keygen`, `renderItem`, `pagination` |
| 标签 | `Tag` | `type`, `closable`, `onClose` |
| 头像 | `Avatar` | `src`, `size`, `shape` |
| 进度条 | `Progress` | `value`, `shape`, `color` |
| 空状态 | `Spin` / 自定义 Empty | — |
| 分页 | `Pagination` | `current`, `total`, `pageSize`, `onChange` |
| 树形控件 | `Tree` | `data`, `keygen`, `renderItem`, `onChange` |

### 操作反馈
| 设计元素 | Shineout 组件 | Props |
|---------|-------------|-------|
| 主要按钮 | `Button` type="primary" | — |
| 次要按钮 | `Button` type="default" | — |
| 文字按钮 | `Button` type="link" | — |
| 危险按钮 | `Button` status="danger" | — |
| 加载中 | `Spin` / `Button` loading | — |
| 全局提示 | `Message.success/error/warning` | 命令式 |
| 通知 | `Notification` | 命令式 |
| 工具提示 | `Tooltip` | `tip`, `position` |
| 弹出确认 | `Confirm` | `title`, `onOk` |

---

## 颜色处理

Shineout 通常没有独立的 `theme.json`，颜色通过以下方式处理：

### 1. 语义 Props（优先）
```tsx
// 按钮颜色语义
<Button type="primary">主要</Button>    // 主色
<Button status="success">成功</Button>  // 成功色
<Button status="warning">警告</Button>  // 警告色
<Button status="danger">危险</Button>   // 危险色

// Tag 颜色
<Tag color="primary">标签</Tag>
```

### 2. 主题 Less 变量（如果项目有 customize-theme.less）
```less
// 查找项目中是否存在类似 customize-theme.less 或 antd-theme.less
@primary-color: #197afa;
@success-color: #00a85f;
```

### 3. 字面颜色值（无法语义化时）
```tsx
<div style={{ color: '#197afa' }}>
```

---

## 表格 columns 定义

Shineout Table 的 columns 定义方式：

```tsx
import { Table } from 'shineout';
import type { TableColumnItem } from 'shineout';

const columns: TableColumnItem<DataType>[] = [
  { title: '名称', dataIndex: 'name', width: 200 },
  { title: '状态', dataIndex: 'status', render: (value) => <Tag>{value}</Tag> },
  {
    title: '操作',
    render: (_, record) => (
      <Button type="link" onClick={() => handleEdit(record)}>编辑</Button>
    ),
  },
];

<Table
  keygen="id"
  columns={columns}
  data={tableData}
  pagination={{ current, pageSize: 10, onChange: handlePageChange, total }}
/>
```

---

## Select 数据格式

Shineout Select 需要通过 `keygen` 和 `renderItem` 处理数据：

```tsx
// 简单数组
<Select
  data={[{ id: 1, name: '选项1' }, { id: 2, name: '选项2' }]}
  keygen="id"
  renderItem="name"
  placeholder="请选择"
  onChange={(val) => console.log(val)}
/>

// 多选
<Select
  data={options}
  keygen="value"
  renderItem="label"
  multiple
  placeholder="请选择（多选）"
/>
```

---

## 表单用法

```tsx
import { Form, Input, Button, Select } from 'shineout';

<Form onSubmit={handleSubmit} labelWidth={80}>
  <Form.Item label="名称" name="name" required>
    <Input placeholder="请输入名称" />
  </Form.Item>
  <Form.Item label="状态" name="status">
    <Select data={statusOptions} keygen="value" renderItem="label" placeholder="请选择" />
  </Form.Item>
  <Form.Item>
    <Button type="primary" htmlType="submit">提交</Button>
  </Form.Item>
</Form>
```

---

## 注意事项

1. **keygen 必填**：`Table`、`Select`、`List`、`Tree` 等组件的 `keygen` prop 是必须的，通常传数据的唯一标识字段名（如 `"id"`）
2. **data 而非 options**：Shineout 的列表类组件使用 `data` 而非 `options`
3. **visible 而非 open**：Shineout 的弹窗/抽屉使用 `visible`（不是 Ant Design 的 `open`）
4. **renderItem 渲染**：Shineout Select 等组件用 `renderItem` 指定展示字段或渲染函数

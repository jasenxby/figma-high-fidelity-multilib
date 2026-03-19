# React + Ant Design 专项指南

## 核心组件映射表

按设计元素类型对应 Ant Design 组件：

### 布局
| 设计元素 | Ant Design 组件 | 关键 Props |
|---------|---------------|-----------|
| 水平/垂直容器 | `Flex` / `Space` | `gap`, `align`, `justify` |
| 等分网格 | `Row` + `Col` | `gutter`, `span` |
| 页面整体布局 | `Layout` + `Header/Sider/Content/Footer` | — |
| 卡片容器 | `Card` | `title`, `extra`, `bordered` |
| 模态弹窗 | `Modal` | `open`, `onOk`, `onCancel` |
| 抽屉 | `Drawer` | `open`, `placement` |
| 折叠面板 | `Collapse` | `items`, `defaultActiveKey` |
| 标签页 | `Tabs` | `items`, `activeKey`, `onChange` |

### 导航
| 设计元素 | Ant Design 组件 | 关键 Props |
|---------|---------------|-----------|
| 顶部导航栏 | `Menu` (mode="horizontal") | `items`, `selectedKeys` |
| 侧边菜单 | `Menu` (mode="inline") | `items`, `collapsed` |
| 面包屑 | `Breadcrumb` | `items` |
| 分页 | `Pagination` | `current`, `total`, `pageSize`, `onChange` |
| 步骤条 | `Steps` | `items`, `current` |

### 数据录入
| 设计元素 | Ant Design 组件 | 关键 Props |
|---------|---------------|-----------|
| 单行文本输入 | `Input` | `placeholder`, `prefix`, `suffix`, `allowClear` |
| 多行文本 | `Input.TextArea` | `rows`, `autoSize` |
| 下拉选择 | `Select` | `options`, `placeholder`, `mode`, `allowClear` |
| 日期选择 | `DatePicker` / `RangePicker` | `format`, `disabledDate` |
| 数字输入 | `InputNumber` | `min`, `max`, `step`, `precision` |
| 复选框 | `Checkbox` / `Checkbox.Group` | `checked`, `onChange`, `options` |
| 单选框 | `Radio` / `Radio.Group` | `value`, `onChange`, `options` |
| 开关 | `Switch` | `checked`, `onChange`, `size` |
| 上传 | `Upload` | `action`, `accept`, `maxCount`, `listType` |
| 表单 | `Form` + `Form.Item` | `labelCol`, `wrapperCol`, `onFinish` |

### 数据展示
| 设计元素 | Ant Design 组件 | 关键 Props |
|---------|---------------|-----------|
| 表格 | `Table` | `columns`, `dataSource`, `pagination`, `rowKey` |
| 列表 | `List` | `dataSource`, `renderItem`, `pagination` |
| 描述列表 | `Descriptions` | `items`, `column`, `bordered` |
| 标签/徽章 | `Tag` | `color`, `closable` |
| 头像 | `Avatar` | `src`, `icon`, `size`, `shape` |
| 进度条 | `Progress` | `percent`, `type`, `status` |
| 时间线 | `Timeline` | `items` |
| 空状态 | `Empty` | `description`, `image` |
| 数字统计 | `Statistic` | `title`, `value`, `prefix`, `suffix` |
| 树形控件 | `Tree` | `treeData`, `checkable`, `expandedKeys` |

### 操作反馈
| 设计元素 | Ant Design 组件 | 使用方式 |
|---------|---------------|---------|
| 主要按钮 | `Button` type="primary" | — |
| 次要/默认按钮 | `Button` type="default" | — |
| 文字按钮 | `Button` type="link" / type="text" | — |
| 危险按钮 | `Button` danger | — |
| 图标按钮 | `Button` icon={<Icon/>} shape="circle" | — |
| 加载状态 | `Spin` / `Button` loading | — |
| 全局提示 | `message.success/error/warning` | 命令式调用 |
| 通知 | `notification.open` | 命令式调用 |
| 气泡确认 | `Popconfirm` | `title`, `onConfirm` |
| 工具提示 | `Tooltip` | `title`, `placement` |
| 气泡卡片 | `Popover` | `content`, `title` |

---

## theme.json Token 映射规则

### 读取方式

查找项目中的主题配置文件（通常命名为 `theme.json`、`antd-theme.json` 或在 App.tsx 中内联），读取 `token` 对象。

### 颜色 Token 映射

| token 字段 | 使用场景 | 代码写法 |
|-----------|---------|---------|
| `colorPrimary` | 主色：主按钮背景、链接、选中态 | `token.colorPrimary` 或 `var(--ant-color-primary)` |
| `colorSuccess` | 成功状态：绿色图标、成功标签 | `token.colorSuccess` |
| `colorWarning` | 警告状态：橙色图标、警告提示 | `token.colorWarning` |
| `colorError` | 错误状态：红色图标、错误提示 | `token.colorError` |
| `colorTextBase` | 主要文字颜色 | `token.colorText` |
| `colorTextSecondary` | 次要文字颜色 | `token.colorTextSecondary` |
| `colorTextTertiary` | 辅助/禁用文字颜色 | `token.colorTextTertiary` |
| `colorBorder` | 边框颜色 | `token.colorBorder` |
| `colorBorderSecondary` | 次级边框颜色 | `token.colorBorderSecondary` |
| `colorBgLayout` | 页面背景色 | `token.colorBgLayout` |
| `boxShadow` | 阴影（如卡片、下拉菜单） | `token.boxShadow` |

### 应用方式

**方式一：CSS-in-JS（推荐，适合 styled-components/emotion）**
```tsx
import { theme } from 'antd';
const { token } = theme.useToken();
// 然后在 style 中使用
<div style={{ color: token.colorTextBase, background: token.colorBgLayout }} />
```

**方式二：通过 ConfigProvider 传递 token**
```tsx
import { ConfigProvider } from 'antd';
// 在 App 根组件中配置，子组件通过 useToken 获取
```

**方式三：CSS 变量（适合纯 CSS/Less 方案）**
```css
color: var(--ant-color-primary);
background: var(--ant-color-bg-layout);
```

### Figma 颜色 → Token 匹配策略

1. 读取 `theme.json` 中的所有颜色值
2. 将 Figma 节点的 `fills[0].color`（转为 hex）与 token 值进行精确匹配
3. 匹配到 → 用 token 变量替代
4. 未匹配到 → 检查是否是品牌色的衍生色（如 primary-light），用 `token.colorPrimaryBg` 等衍生 token
5. 确实找不到对应 token → 使用字面颜色值，并添加注释说明

---

## 样式方案优先级

1. **语义 Token**（colorPrimary 等）→ 可主题化，首选
2. **Ant Design 组件 Props**（`type="primary"`、`status="error"` 等）→ 语义化组件状态
3. **CSS 变量**（`var(--ant-color-*)`）→ 无法使用 JS token 时的备选
4. **字面量颜色值**（仅限无法映射 token 的颜色）→ 添加注释说明原因

---

## 常见代码模式

### 带主题的页面结构
```tsx
import { ConfigProvider, App } from 'antd';
import theme from './theme.json';

export default function Root() {
  return (
    <ConfigProvider theme={theme}>
      <App>
        {/* 页面内容 */}
      </App>
    </ConfigProvider>
  );
}
```

### 表格（含分页）
```tsx
import { Table } from 'antd';
import type { TableColumnsType } from 'antd';

const columns: TableColumnsType<DataType> = [
  { title: '名称', dataIndex: 'name', key: 'name' },
  { title: '操作', key: 'action', render: (_, record) => <a>编辑</a> },
];

<Table columns={columns} dataSource={data} rowKey="id" pagination={{ pageSize: 10 }} />
```

### 搜索筛选区域
```tsx
import { Form, Input, Select, Button, Space } from 'antd';

<Form layout="inline" onFinish={handleSearch}>
  <Form.Item name="keyword">
    <Input placeholder="请输入关键词" allowClear />
  </Form.Item>
  <Form.Item name="status">
    <Select placeholder="请选择状态" options={statusOptions} allowClear style={{ width: 160 }} />
  </Form.Item>
  <Form.Item>
    <Space>
      <Button type="primary" htmlType="submit">查询</Button>
      <Button htmlType="reset">重置</Button>
    </Space>
  </Form.Item>
</Form>
```

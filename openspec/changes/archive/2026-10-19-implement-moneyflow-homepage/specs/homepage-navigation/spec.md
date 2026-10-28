# Spec Delta — homepage-navigation

## MODIFIED Requirements

### Requirement: 功能导航栏
前端 SHALL 在首页 Hero 区下方以卡片形式展示三大核心功能入口（行业资金流、大盘资金流、AI 助手），每张卡片包含图标、标题和一句话描述，支持点击跳转至对应功能页面。导航栏位于 Hero 区与数据更新时间之间。

#### Scenario: 导航卡片完整展示
- **WHEN** 用户访问首页
- **THEN** 展示三张功能导航卡片：行业资金流、大盘资金流、AI 助手
- **AND** 每张卡片包含图标、标题和一句话描述
- **AND** 所有卡片高度一致，水平等宽排列

#### Scenario: 点击卡片跳转
- **WHEN** 用户点击某张导航卡片
- **THEN** 页面跳转至对应功能页面（行业资金流 → `/industry-flow`，大盘资金流 → `/market-flow`，AI 助手 → `/ai-assistant`）

#### Scenario: Hover 交互反馈
- **WHEN** 用户鼠标悬停在某张卡片上
- **THEN** 卡片视觉反馈明确（如阴影加深、边框高亮或轻微上浮）
- **AND** 鼠标指针变为手型（pointer）

#### Scenario: 键盘聚焦反馈
- **WHEN** 用户通过 Tab 键聚焦到某张卡片
- **THEN** 卡片显示聚焦指示样式（如 outline 或 ring）
- **AND** 聚焦状态可通过 Enter 或 Space 键触发跳转

#### Scenario: 桌面端布局（>= 1024px）
- **WHEN** 视口宽度 >= 1024px
- **THEN** 三张卡片以网格形式水平排列
- **AND** 卡片间距均匀

#### Scenario: 平板端布局（768px - 1023px）
- **WHEN** 视口宽度 >= 768px 且 < 1024px
- **THEN** 卡片以两列网格排列，第三张卡片换行居中
- **AND** 卡片宽度自适应

#### Scenario: 移动端布局（< 768px）
- **WHEN** 视口宽度 < 768px
- **THEN** 卡片纵向堆叠为单列
- **AND** 卡片宽度占满内容区

## ADDED Requirements

### Requirement: 导航卡片内容配置
前端 SHALL 支持每张导航卡片的图标、标题和描述独立配置，未配置时不展示对应元素且不产生空白占位。

#### Scenario: 卡片元素完整配置
- **WHEN** 某张卡片配置了图标、标题和描述
- **THEN** 三个元素均正常展示

#### Scenario: 卡片缺少图标配置
- **WHEN** 某张卡片未配置图标
- **THEN** 该卡片不展示图标区域，标题和描述正常展示，布局不出现空白占位

#### Scenario: 卡片缺少描述配置
- **WHEN** 某张卡片未配置描述文本
- **THEN** 该卡片仅展示图标和标题，卡片高度自适应内容

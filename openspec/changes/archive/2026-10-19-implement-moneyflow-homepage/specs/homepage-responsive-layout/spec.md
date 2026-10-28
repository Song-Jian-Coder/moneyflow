# Spec Delta — homepage-responsive-layout

## ADDED Requirements

### Requirement: 多端布局适配
前端 SHALL 提供响应式布局框架，根据视口宽度自动调整首页各区域的排列方式和尺寸。

#### Scenario: PC 端布局（>= 1024px）
- **WHEN** 视口宽度 >= 1024px
- **THEN** 布局框架按以下顺序垂直排列：Header（固定顶部，60-80px）、Hero 区（400-600px）、功能导航栏（自适应）、数据更新时间（40px）、主内容区（自适应）、Footer（60-80px）
- **AND** 内容区居中，最大宽度 1280px，左右 padding 32px

#### Scenario: 平板端布局（768px - 1023px）
- **WHEN** 视口宽度 >= 768px 且 < 1024px
- **THEN** 布局框架与 PC 端类似，但 Header 高度 50-60px、Hero 区高度 300-400px、内容区左右 padding 24px

#### Scenario: 移动端布局（< 768px）
- **WHEN** 视口宽度 < 768px
- **THEN** Header 高度 50px 并包含汉堡菜单
- **AND** Hero 区高度 250-350px
- **AND** 功能导航栏纵向排列
- **AND** 主内容区占满宽度，左右 padding 16px
- **AND** Footer 简化展示

#### Scenario: 栅格布局
- **WHEN** 布局主内容区
- **THEN** PC 端使用 12 列栅格，热力图和树状图可并排或上下排列，间距 24px
- **WHEN** 移动端（< 768px）
- **THEN** 所有栅格列占满 12 列（单列布局），间距缩小为 16px

### Requirement: 导航适配
前端 SHALL 根据设备类型提供不同的导航展示方式。

#### Scenario: PC 端导航
- **WHEN** 视口宽度 >= 1024px
- **THEN** Header 展示完整导航菜单（水平排列）
- **AND** 导航项包括：首页、行业资金流、大盘资金流、AI 助手等

#### Scenario: 移动端导航
- **WHEN** 视口宽度 < 768px
- **THEN** Header 展示汉堡菜单图标（三条横线）
- **AND** 点击汉堡菜单弹出侧边栏或下拉菜单
- **AND** 菜单项垂直排列，点击后菜单关闭

### Requirement: 触摸手势支持
前端 SHALL 在移动端提供触摸手势支持，包括图表缩放、拖动和面板滑动关闭。

#### Scenario: 图表缩放
- **WHEN** 用户在热力图或树状图上双指张开/捏合
- **THEN** 图表放大/缩小
- **AND** 缩放范围 0.5x - 3x
- **AND** 缩放流畅，无卡顿

#### Scenario: 图表拖动
- **WHEN** 用户在热力图或树状图上单指拖动
- **THEN** 图表跟随手指移动
- **AND** 拖动流畅，有惯性效果
- **AND** 拖动不会触发页面滚动（使用 touch-action: none）

#### Scenario: 面板滑动关闭
- **WHEN** 用户在详情面板上向下滑动
- **THEN** 面板跟随手指下移
- **AND** 下拉距离超过阈值（如 100px）时释放，面板关闭
- **AND** 下拉距离未达阈值时释放，面板回弹

### Requirement: 超小屏幕兼容
前端 SHALL 在超小屏幕设备上保证基本可用性。

#### Scenario: 超小屏幕适配（< 375px）
- **WHEN** 视口宽度 < 375px（如 iPhone SE）
- **THEN** 布局框架正常工作，内容不被截断
- **AND** 文字仍可读（最小字号 >= 12px）
- **AND** 按钮可点击（最小点击区域 44x44px）

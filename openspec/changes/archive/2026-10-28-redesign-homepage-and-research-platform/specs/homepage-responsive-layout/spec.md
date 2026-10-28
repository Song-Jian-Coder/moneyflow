# Spec Delta

## MODIFIED Requirements

### Requirement: 多端布局适配
前端 SHALL 提供响应式布局框架。首页采用极简品牌落地页布局（Hero 区左右分栏，移动端变为上下布局）。研究平台页面采用左侧导航 + 右侧内容区布局，移动端导航栏收起为汉堡菜单或顶部 Tab。

#### Scenario: 首页 Desktop 布局
- **WHEN** 视口宽度 >= 1024px 且用户访问首页
- **THEN** Hero 区采用左右分栏布局，左侧标语、右侧渐变背景

#### Scenario: 首页 Mobile 布局
- **WHEN** 视口宽度 < 768px 且用户访问首页
- **THEN** Hero 区变为上下布局，标语在上、渐变背景在下

#### Scenario: 研究平台 Desktop 布局
- **WHEN** 视口宽度 >= 1024px 且用户访问 `/research`
- **THEN** 左侧导航栏固定宽度，右侧内容区可滚动

#### Scenario: 研究平台 Mobile 布局
- **WHEN** 视口宽度 < 768px 且用户访问 `/research`
- **THEN** 左侧导航栏收起为汉堡菜单或顶部 Tab 形式

### Requirement: 导航适配
前端 SHALL 根据设备类型提供不同的导航展示方式。顶部导航栏在桌面端水平展示全部链接，在移动端收起为汉堡菜单。

#### Scenario: Desktop 导航展示
- **WHEN** 视口宽度 >= 768px
- **THEN** 顶部导航栏水平展示 Logo 和全部导航链接（首页、政策报告、资金流研究平台）

#### Scenario: Mobile 导航展示
- **WHEN** 视口宽度 < 768px
- **THEN** 顶部导航栏仅展示 Logo 和汉堡菜单按钮
- **AND** 点击汉堡菜单按钮展开导航链接列表

### Requirement: 触摸手势支持
前端 SHALL 在移动端提供触摸手势支持。

#### Scenario: 研究平台移动端导航滑动
- **WHEN** 用户在移动端研究平台页面
- **THEN** 支持通过滑动切换左侧导航的不同子模块

### Requirement: 超小屏幕兼容
前端 SHALL 在超小屏幕设备上保证基本可用性。

#### Scenario: 超小屏幕首页展示
- **WHEN** 视口宽度 < 375px
- **THEN** 首页 Hero 区标语字号缩小但不小于 20px，布局保持可用

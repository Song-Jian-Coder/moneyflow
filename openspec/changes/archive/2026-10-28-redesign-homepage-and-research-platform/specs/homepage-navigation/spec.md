# Spec Delta

## MODIFIED Requirements

### Requirement: 功能导航栏
前端 SHALL 在顶部导航栏中展示三个核心页面入口：首页、政策报告、资金流研究平台。导航栏位于页面顶部，Logo 在左侧，导航链接在右侧。不再展示导航卡片组件。

#### Scenario: 导航栏展示
- **WHEN** 用户访问任意页面
- **THEN** 顶部导航栏左侧展示 MoneyFlow Logo
- **AND** 右侧展示三个导航链接：首页、政策报告、资金流研究平台
- **AND** 当前页面对应的导航链接高亮显示
- **AND** 不展示导航卡片组件

#### Scenario: 导航链接跳转
- **WHEN** 用户点击「政策报告」导航链接
- **THEN** 页面跳转到 `/policy-reports`

#### Scenario: 导航链接跳转
- **WHEN** 用户点击「资金流研究平台」导航链接
- **THEN** 页面跳转到 `/research`

### Requirement: 导航卡片内容配置
**Reason**: 导航卡片组件已从首页移除，功能入口统一由顶部导航栏承担
**Migration**: 导航卡片组件代码保留但不再在首页引用，后续可复用或清理

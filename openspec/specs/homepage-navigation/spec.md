# Homepage Navigation Specification

## Purpose

提供全局顶部导航栏，在所有页面顶部固定展示，包含首页、政策报告、资金流研究平台三个导航入口，支持响应式布局和当前页面高亮。

## Requirements

### Requirement: 全局顶部导航栏
前端 SHALL 在所有页面顶部提供固定导航栏，包含品牌 Logo 和三个导航项：首页、政策报告、资金流研究平台。导航栏位于页面最顶部，z-index 最高，背景半透明带模糊效果。

#### Scenario: 导航栏完整展示
- **WHEN** 用户访问任意页面
- **THEN** 顶部显示导航栏，左侧为 MoneyFlow Logo，右侧为三个导航链接
- **AND** 导航链接分别为：首页（`/`）、政策报告（`/policy-reports`）、资金流研究平台（`/research`）

#### Scenario: 当前页面高亮
- **WHEN** 用户位于某个导航项对应的页面
- **THEN** 该导航项文字颜色加深、字重加粗
- **AND** 其他导航项为默认颜色，hover 时变为品牌色

#### Scenario: 点击导航跳转
- **WHEN** 用户点击某个导航项
- **THEN** 页面跳转至对应路由，导航栏高亮更新为新页面

#### Scenario: 桌面端布局（>= 768px）
- **WHEN** 视口宽度 >= 768px
- **THEN** Logo 和导航项以水平 flex 布局展示
- **AND** Logo 在左，导航项在右，间距均匀

#### Scenario: 移动端布局（< 768px）
- **WHEN** 视口宽度 < 768px
- **THEN** 导航栏高度略小于桌面端
- **AND** Logo 和导航项保持水平排列，文字尺寸略小

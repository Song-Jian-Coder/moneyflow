# Spec Delta — homepage-time-selector

## ADDED Requirements

### Requirement: 时间范围选择
前端 SHALL 提供时间维度切换器，支持快捷选择和自定义日期范围，输出标准化的时间参数供其他首页模块使用。

#### Scenario: 页面加载默认选择
- **WHEN** 页面首次加载且用户未选择时间
- **THEN** 默认选择"今日"时间范围
- **AND** startDate 和 endDate 均为今天日期
- **AND** displayName 显示"今日 (yyyy-MM-dd)"
- **AND** 触发 onChange 回调（如配置）

#### Scenario: 快捷选择 — 今日
- **WHEN** 用户点击"今日"按钮
- **THEN** 时间范围设置为今天
- **AND** displayName 显示"今日 (yyyy-MM-dd)"
- **AND** 触发 onChange 回调
- **AND** 其他组件（热力图、树状图）更新数据

#### Scenario: 快捷选择 — 近 7 日
- **WHEN** 用户点击"近 7 日"按钮
- **THEN** startDate 设置为 6 天前
- **AND** endDate 设置为今天
- **AND** displayName 显示"近 7 日 (yyyy-MM-dd ~ yyyy-MM-dd)"
- **AND** 触发 onChange 回调

#### Scenario: 自定义时间选择
- **WHEN** 用户通过日期选择器选择起始和结束日期
- **THEN** 时间范围设置为选定范围
- **AND** displayName 显示"yyyy-MM-dd ~ yyyy-MM-dd"
- **AND** 触发 onChange 回调

#### Scenario: 标准化格式输出
- **WHEN** 用户选择任意时间范围
- **THEN** 输出标准化的摘要] startDate 和 endDate（ISO 8601）
- **AND** 输出 displayName 用于 UI 展示
- **AND** 输出 type 标识时间类型

### Requirement: 时间范围校验
前端 SHALL 对用户选择的时间范围进行合法性校验，防止无效输入。

#### Scenario: 起始日期晚于结束日期
- **WHEN** 用户选择的起始日期晚于结束日期
- **THEN** 显示错误提示"起始日期不能晚于结束日期"
- **AND** 不触发 onChange 回调
- **AND** 日期选择器保持打开状态

#### Scenario: 时间范围超出数据可用范围
- **WHEN** 用户选择的时间范围早于 minDate（如 2020-01-01）
- **THEN** 显示提示"数据最早可追溯至 yyyy-MM-dd"
- **AND** 自动将 startDate 调整为为 minDate
- **AND** 触发 onChange 回调

#### Scenario: 时间范围超过 365 天
- **WHEN** 用户选择的时间范围超过 365 天
- **THEN** 显示提示"时间范围不能超过 365 天"
- **AND** 不触发 onChange 回调

### Requirement: 交易日处理
前端 SHALL 识别非交易日并自动回退到最近的有效交易日。

#### Scenario: 今天为非交易日
- **WHEN** 当天为非交易日（周末或节假日）且用户选择"今日"
- **THEN** 自动回退到最近一个交易日
- **AND** 显示提示"今日为非交易日，已显示最近交易日数据"

#### Scenario: 结束日期超过今天
- **WHEN** 用户选择的结束日期晚于今天
- **THEN** 显示提示"结束日期不能超过今天"
- **AND** 自动将 endDate 调整为今天

# Spec Delta — homepage-update-timestamp

## ADDED Requirements

### Requirement: 更新时间展示
前端 SHALL 在首页顶部展示数据最后更新时间，支持 UTC 转本地时间和多种格式。

#### Scenario: 完整时间展示
- **WHEN** 页面加载完成且更新时间数据获取成功
- **THEN** 在首页顶部展示更新时间
- **AND** 显示格式为 "数据更新时间：yyyy-MM-dd HH:mm"
- **AND** 时间为用户本地时间（根据浏览器时区自动转换）
- **AND** 在时间后显示"每日收盘后更新"说明文字

#### Scenario: 时区处理
- **WHEN** 后端返回 UTC 时间（如 "2026-10-19T07:30:00Z"）
- **THEN** 根据浏览器时区自动转换为本地时间
- **AND** 东八区（北京时间）显示为 "2026-10-19 15:30"
- **AND** 西五区（纽约时间）显示为 "2026-10-19 02:30"

#### Scenario: 相对时间展示
- **WHEN** showRelativeTime 为 true 且更新时间距离当前 < 24 小时
- **THEN** 显示相对时间（如 "2 小时前"、"30 分钟前"）
- **AND** 鼠标 hover 时显示绝对时间（tooltip）

### Requirement: 更新时间异常处理
前端 SHALL 在更新时间数据获取失败或为空时提供降级展示，不影响页面其他内容。

#### Scenario: 更新时间获取失败
- **WHEN** 更新时间数据获取失败（API 异常或网络问题）
- **THEN** 显示"更新时间未知"或"暂无更新信息"
- **AND** 不影响页面其他内容的正常展示

#### Scenario: 时间为 null
- **WHEN** 后端返回的 lastUpdateTime 为 null
- **THEN** 显示"暂无数据"
- **AND** 仍显示"每日收盘后更新"说明

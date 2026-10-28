# Spec Delta

## MODIFIED Requirements

### Requirement: Hero 区品牌展示
前端 SHALL 在首页顶部展示 Hero 区，采用左右分栏布局：左侧展示品牌标语，右侧展示静态渐变背景视觉，作为首页第一视觉焦点。

#### Scenario: 完整配置展示
- **WHEN** 用户访问首页且所有配置正常
- **THEN** Hero 区采用左右分栏布局，左侧展示标语，右侧展示渐变背景
- **AND** 左侧展示主标语"更专业、更专注"，字号 >= 48px（mobile: 28px），字重 bold
- **AND** 主标语下方展示副标语"资金流汇聚网站"，字号 >= 20px，颜色为 muted-foreground
- **AND** 不展示搜索框
- **AND** 所有元素在 3 秒内完成渲染

#### Scenario: Desktop 响应式展示（>= 1024px）
- **WHEN** 视口宽度 >= 1024px
- **THEN** Hero 区高度为 500px - 600px
- **AND** 左侧标语区域占 50% 宽度，右侧渐变区域占 50% 宽度
- **AND** 主标语字号为 56px，副标语字号为 22px
- **AND** 左侧内容垂直居中

#### Scenario: Tablet 响应式展示（768px - 1023px）
- **WHEN** 视口宽度 >= 768px 且 < 1024px
- **THEN** Hero 区高度为 400px - 500px
- **AND** 主标语字号为 42px，副标语字号为 18px
- **AND** 左侧内容垂直居中

#### Scenario: Mobile 响应式展示（< 768px）
- **WHEN** 视口宽度 < 768px
- **THEN** Hero 区高度为 300px - 400px
- **AND** 主标语字号为 28px，副标语字号为 16px
- **AND** 左右分栏变为上下布局，标语在上、渐变背景在下
- **AND** 左侧内容水平居中对齐

### Requirement: 搜索框交互
**Reason**: 首页重构为品牌落地页，不再需要搜索入口
**Migration**: 搜索功能后续可在研究平台页面中按需实现

## REMOVED Requirements

### Requirement: Hero 区异常降级
**Reason**: 首页不再使用背景图，改为静态渐变背景，图片加载失败场景不再适用
**Migration**: 渐变背景为 CSS 实现，无外部资源依赖，不需要降级处理

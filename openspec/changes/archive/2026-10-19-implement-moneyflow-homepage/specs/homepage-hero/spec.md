# Spec Delta — homepage-hero

## ADDED Requirements

### Requirement: Hero 区品牌展示
前端 SHALL 在首页顶部展示 Hero 区，包含品牌背景图、主副标语和搜索入口，作为首页第一视觉焦点。

#### Scenario: 完整配置展示
- **WHEN** 用户访问首页且所有配置正常
- **THEN** 展示品牌背景图，覆盖整个 Hero 区域
- **AND** 背景图使用 object-fit: cover 保持比例，无拉伸变形
- **AND** 展示主标语，字号 >= 32px（mobile: 24px），字重 bold
- **AND** 展示副标语，字号 >= 16px，颜色为 text-white/80
- **AND** 展示搜索框，包含占位文本"搜索股票/行业"
- **AND** 搜索框居中展示，最大宽度为 max-w-md
- **AND** 所有元素在 3 秒内完成渲染

#### Scenario: Desktop 响应式展示（>= 1024px）
- **WHEN** 视口宽度 >= 1024px
- **THEN** 加载 desktop 背景图
- **AND** Hero 区高度为 500px - 600px
- **AND** 主标语字号为 48px，副标语字号为 20px
- **AND** 搜索框最大宽度为 28rem
- **AND** 所有元素水平居中对齐

#### Scenario: Tablet 响应式展示（768px - 1023px）
- **WHEN** 视口宽度 >= 768px 且 < 1024px
- **THEN** 加载 tablet 背景图（如未配置则使用 desktop 图）
- **AND** Hero 区高度为 400px - 500px
- **AND** 主标语字号为 36px，副标语字号为 18px
- **AND** 搜索框最大宽度为 24rem
- **AND** 所有元素水平居中对齐

#### Scenario: Mobile 响应式展示（< 768px）
- **WHEN** 视口宽度 < 768px
- **THEN** 加载 mobile 背景图（如未配置则使用 desktop 图）
- **AND** Hero 区高度为 300px - 400px
- **AND** 主标语字号为 24px，副标语字号为 16px
- **AND** 搜索框最大宽度为 100%，左右 padding 为 16px
- **AND** 所有元素水平居中对齐

### Requirement: 搜索框交互
前端 SHALL 为 Hero 区搜索框提供完整的交互状态反馈。

#### Scenario: Focus 状态
- **WHEN** 用户点击搜索框或按 Tab 键聚焦
- **THEN** 搜索框边框高亮（ring-2 ring-blue-500）
- **AND** 搜索框背景略微变亮
- **AND** 占位文本消失或变淡
- **AND** 显示光标闪烁

#### Scenario: Hover 状态
- **WHEN** 用户鼠标悬停在搜索框上（未聚焦）
- **THEN** 搜索框边框颜色变亮
- **AND** 鼠标指针变为 text 类型

#### Scenario: 输入状态
- **WHEN** 用户在搜索框中输入文字
- **THEN** 搜索框实时显示输入内容
- **AND** 占位文本消失
- **AND** 如配置了 onSearch 回调，每次输入触发回调（防抖 300ms）

#### Scenario: 搜索框禁用状态
- **WHEN** searchConfig.disabled 为 true
- **THEN** 搜索框显示禁用状态（opacity-50 cursor-not-allowed）
- **AND** 搜索框不可点击和输入
- **AND** 搜索框不显示 focus 和 hover 效果

### Requirement: Hero 区异常降级
前端 SHALL 在背景图加载失败或文案超长时提供降级处理，保证页面可用性。

#### Scenario: 背景图加载失败
- **WHEN** 背景图 URL 无效或网络异常导致加载失败
- **THEN** 显示降级背景渐变（fallbackGradient）
- **AND** 如未配置 fallbackGradient，使用 fallbackColor
- **AND** 如两者均未配置，使用默认渐变（from-gray-800 to-gray-900）
- **AND** 不影响标语和搜索框的正常展示

#### Scenario: 主标语超长
- **WHEN** 主标语超过 50 字符
- **THEN** 前端截断显示（末尾加省略号）
- **AND** 使用 CSS text-overflow: ellipsis
- **AND** 确保不破坏整体布局

#### Scenario: 副标语超长
- **WHEN** 副标语超过 100 字符
- **THEN** 前端截断显示（末尾加省略号）
- **AND** 使用 CSS text-overflow: ellipsis

#### Scenario: 副标语为空
- **WHEN** 副标语配置为空字符串或未配置
- **THEN** 仅展示主标语
- **AND** 主标语居中展示，布局保持对称

#### Scenario: 图片加载超时
- **WHEN** 背景图加载时间超过 5 秒
- **THEN** 显示降级背景色或渐变
- **AND** 继续尝试加载原图，加载完成后淡入替换

#### Scenario: 图片宽高比异常
- **WHEN** 背景图原始宽高比与设计稿差异过大
- **THEN** 使用 object-fit: cover 裁剪图片
- **AND** 确保图片中心区域不被裁剪

---
name: obsidian-bases
description: 创建和编辑 Obsidian Bases（.base 文件），支持视图、筛选、公式和汇总。用于处理 .base 文件、创建数据库式笔记视图，或当用户提到 Bases、表格视图、卡片视图、筛选器或公式时使用。
---

# Obsidian Bases 技能

本技能使 AI 智能体能够创建和编辑有效的 Obsidian Bases（`.base` 文件），包括视图、筛选器、公式及所有相关配置。

## 概述

Obsidian Bases 是基于 YAML 的文件，用于定义 Obsidian 库中笔记的动态视图。一个 Base 文件可以包含多个视图、全局筛选器、公式、属性配置和自定义汇总。

## 文件格式

Base 文件使用 `.base` 扩展名，包含有效的 YAML。它们也可以嵌入到 Markdown 代码块中。

## 完整结构

```yaml
# 全局筛选器应用于 Base 中的所有视图
filters:
  # 可以是单个筛选字符串
  # 或带有 and/or/not 的递归筛选对象
  and: []
  or: []
  not: []

# 定义可在所有视图中使用的公式属性
formulas:
  公式名称: '表达式'

# 配置属性的显示名称和设置
properties:
  属性名称:
    displayName: "显示名称"
  formula.公式名称:
    displayName: "公式显示名称"
  file.ext:
    displayName: "扩展名"

# 定义自定义汇总公式
summaries:
  自定义汇总名称: 'values.mean().round(3)'

# 定义一个或多个视图
views:
  - type: table | cards | list | map
    name: "视图名称"
    limit: 10                    # 可选：限制结果数量
    groupBy:                     # 可选：分组结果
      property: 属性名称
      direction: ASC | DESC
    filters:                     # 视图特定筛选器
      and: []
    order:                       # 按顺序显示的属性
      - file.name
      - 属性名称
      - formula.公式名称
    summaries:                   # 属性到汇总公式的映射
      属性名称: Average
```

## 筛选器语法

筛选器用于缩小结果范围。可以全局应用或按视图应用。

### 筛选器结构

```yaml
# 单个筛选器
filters: 'status == "done"'

# AND - 所有条件必须为真
filters:
  and:
    - 'status == "done"'
    - 'priority > 3'

# OR - 任一条件为真即可
filters:
  or:
    - 'file.hasTag("book")'
    - 'file.hasTag("article")'

# NOT - 排除匹配项
filters:
  not:
    - 'file.hasTag("archived")'

# 嵌套筛选器
filters:
  or:
    - file.hasTag("tag")
    - and:
        - file.hasTag("book")
        - file.hasLink("Textbook")
    - not:
        - file.hasTag("book")
        - file.inFolder("Required Reading")
```

### 筛选器运算符

| 运算符 | 描述 |
|--------|------|
| `==` | 等于 |
| `!=` | 不等于 |
| `>` | 大于 |
| `<` | 小于 |
| `>=` | 大于或等于 |
| `<=` | 小于或等于 |
| `&&` | 逻辑与 |
| `\|\|` | 逻辑或 |
| <code>!</code> | 逻辑非 |

## 属性

### 三种属性类型

1. **笔记属性** - 来自 frontmatter：`note.author` 或直接 `author`
2. **文件属性** - 文件元数据：`file.name`、`file.mtime` 等
3. **公式属性** - 计算值：`formula.my_formula`

### 文件属性参考

| 属性 | 类型 | 描述 |
|------|------|------|
| `file.name` | 字符串 | 文件名 |
| `file.basename` | 字符串 | 不带扩展名的文件名 |
| `file.path` | 字符串 | 文件完整路径 |
| `file.folder` | 字符串 | 父文件夹路径 |
| `file.ext` | 字符串 | 文件扩展名 |
| `file.size` | 数字 | 文件大小（字节） |
| `file.ctime` | 日期 | 创建时间 |
| `file.mtime` | 日期 | 修改时间 |
| `file.tags` | 列表 | 文件中的所有标签 |
| `file.links` | 列表 | 文件中的内部链接 |
| `file.backlinks` | 列表 | 链接到此文件的文件 |
| `file.embeds` | 列表 | 笔记中的嵌入内容 |
| `file.properties` | 对象 | 所有 frontmatter 属性 |

### `this` 关键字

- 在主内容区域：指向 base 文件本身
- 嵌入时：指向嵌入它的文件
- 在侧边栏：指向主内容区的活动文件

## 公式语法

公式从属性计算值。在 `formulas` 部分定义。

```yaml
formulas:
  # 简单算术
  total: "price * quantity"

  # 条件逻辑
  status_icon: 'if(done, "✅", "⏳")'

  # 字符串格式化
  formatted_price: 'if(price, price.toFixed(2) + " 元")'

  # 日期格式化
  created: 'file.ctime.format("YYYY-MM-DD")'

  # 复杂表达式
  days_old: '((now() - file.ctime) / 86400000).round(0)'
```

## 函数参考

### 全局函数

| 函数 | 签名 | 描述 |
|------|------|------|
| `date()` | `date(string): date` | 解析字符串为日期。格式：`YYYY-MM-DD HH:mm:ss` |
| `duration()` | `duration(string): duration` | 解析时长字符串 |
| `now()` | `now(): date` | 当前日期和时间 |
| `today()` | `today(): date` | 当前日期（时间 = 00:00:00） |
| `if()` | `if(condition, trueResult, falseResult?)` | 条件判断 |
| `min()` | `min(n1, n2, ...): number` | 最小数 |
| `max()` | `max(n1, n2, ...): number` | 最大数 |
| `number()` | `number(any): number` | 转换为数字 |
| `link()` | `link(path, display?): Link` | 创建链接 |
| `list()` | `list(element): List` | 如果不是列表则包装为列表 |
| `file()` | `file(path): file` | 获取文件对象 |
| `image()` | `image(path): image` | 创建用于渲染的图片 |
| `icon()` | `icon(name): icon` | 按名称获取 Lucide 图标 |
| `html()` | `html(string): html` | 渲染为 HTML |
| `escapeHTML()` | `escapeHTML(string): string` | 转义 HTML 字符 |

### 任意类型函数

| 函数 | 签名 | 描述 |
|------|------|------|
| `isTruthy()` | `any.isTruthy(): boolean` | 强制转换为布尔值 |
| `isType()` | `any.isType(type): boolean` | 检查类型 |
| `toString()` | `any.toString(): string` | 转换为字符串 |

### 日期函数和字段

**字段：** `date.year`、`date.month`、`date.day`、`date.hour`、`date.minute`、`date.second`、`date.millisecond`

| 函数 | 签名 | 描述 |
|------|------|------|
| `date()` | `date.date(): date` | 移除时间部分 |
| `format()` | `date.format(string): string` | 使用 Moment.js 模式格式化 |
| `time()` | `date.time(): string` | 获取时间字符串 |
| `relative()` | `date.relative(): string` | 人类可读的相对时间 |
| `isEmpty()` | `date.isEmpty(): boolean` | 日期始终返回 false |

### 日期算术

```yaml
# 时长单位：y/year/years、M/month/months、d/day/days、
#           w/week/weeks、h/hour/hours、m/minute/minutes、s/second/seconds

# 加减时长
"date + \"1M\""           # 加 1 个月
"date - \"2h\""           # 减 2 小时
"now() + \"1 day\""       # 明天
"today() + \"7d\""        # 一周后

# 日期相减得到毫秒差
"now() - file.ctime"

# 复杂时长算术
"now() + (duration('1d') * 2)"
```

### 字符串函数

**字段：** `string.length`

| 函数 | 签名 | 描述 |
|------|------|------|
| `contains()` | `string.contains(value): boolean` | 检查子字符串 |
| `containsAll()` | `string.containsAll(...values): boolean` | 所有子字符串都存在 |
| `containsAny()` | `string.containsAny(...values): boolean` | 任一子字符串存在 |
| `startsWith()` | `string.startsWith(query): boolean` | 以 query 开头 |
| `endsWith()` | `string.endsWith(query): boolean` | 以 query 结尾 |
| `isEmpty()` | `string.isEmpty(): boolean` | 为空或不存在 |
| `lower()` | `string.lower(): string` | 转为小写 |
| `title()` | `string.title(): string` | 转为首字母大写 |
| `trim()` | `string.trim(): string` | 移除空白字符 |
| `replace()` | `string.replace(pattern, replacement): string` | 替换模式 |
| `repeat()` | `string.repeat(count): string` | 重复字符串 |
| `reverse()` | `string.reverse(): string` | 反转字符串 |
| `slice()` | `string.slice(start, end?): string` | 子字符串 |
| `split()` | `string.split(separator, n?): list` | 分割为列表 |

### 数字函数

| 函数 | 签名 | 描述 |
|------|------|------|
| `abs()` | `number.abs(): number` | 绝对值 |
| `ceil()` | `number.ceil(): number` | 向上取整 |
| `floor()` | `number.floor(): number` | 向下取整 |
| `round()` | `number.round(digits?): number` | 四舍五入到指定位数 |
| `toFixed()` | `number.toFixed(precision): string` | 定点表示法 |
| `isEmpty()` | `number.isEmpty(): boolean` | 不存在 |

### 列表函数

**字段：** `list.length`

| 函数 | 签名 | 描述 |
|------|------|------|
| `contains()` | `list.contains(value): boolean` | 元素存在 |
| `containsAll()` | `list.containsAll(...values): boolean` | 所有元素都存在 |
| `containsAny()` | `list.containsAny(...values): boolean` | 任一元素存在 |
| `filter()` | `list.filter(expression): list` | 按条件筛选（使用 `value`、`index`） |
| `map()` | `list.map(expression): list` | 转换元素（使用 `value`、`index`） |
| `reduce()` | `list.reduce(expression, initial): any` | 归约为单个值（使用 `value`、`index`、`acc`） |
| `flat()` | `list.flat(): list` | 展平嵌套列表 |
| `join()` | `list.join(separator): string` | 连接为字符串 |
| `reverse()` | `list.reverse(): list` | 反转顺序 |
| `slice()` | `list.slice(start, end?): list` | 子列表 |
| `sort()` | `list.sort(): list` | 升序排序 |
| `unique()` | `list.unique(): list` | 去重 |
| `isEmpty()` | `list.isEmpty(): boolean` | 无元素 |

### 文件函数

| 函数 | 签名 | 描述 |
|------|------|------|
| `asLink()` | `file.asLink(display?): Link` | 转换为链接 |
| `hasLink()` | `file.hasLink(otherFile): boolean` | 有到该文件的链接 |
| `hasTag()` | `file.hasTag(...tags): boolean` | 有任一标签 |
| `hasProperty()` | `file.hasProperty(name): boolean` | 有该属性 |
| `inFolder()` | `file.inFolder(folder): boolean` | 在文件夹或子文件夹中 |

### 链接函数

| 函数 | 签名 | 描述 |
|------|------|------|
| `asFile()` | `link.asFile(): file` | 获取文件对象 |
| `linksTo()` | `link.linksTo(file): boolean` | 链接到该文件 |

### 对象函数

| 函数 | 签名 | 描述 |
|------|------|------|
| `isEmpty()` | `object.isEmpty(): boolean` | 无属性 |
| `keys()` | `object.keys(): list` | 键列表 |
| `values()` | `object.values(): list` | 值列表 |

### 正则表达式函数

| 函数 | 签名 | 描述 |
|------|------|------|
| `matches()` | `regexp.matches(string): boolean` | 测试是否匹配 |

## 视图类型

### 表格视图

```yaml
views:
  - type: table
    name: "我的表格"
    order:
      - file.name
      - status
      - due_date
    summaries:
      price: Sum
      count: Average
```

### 卡片视图

```yaml
views:
  - type: cards
    name: "画廊"
    order:
      - file.name
      - cover_image
      - description
```

### 列表视图

```yaml
views:
  - type: list
    name: "简单列表"
    order:
      - file.name
      - status
```

### 地图视图

需要纬度/经度属性和 Maps 社区插件。

```yaml
views:
  - type: map
    name: "位置"
    # 地图特定的纬度/经度属性设置
```

## 默认汇总公式

| 名称 | 输入类型 | 描述 |
|------|----------|------|
| `Average` | 数字 | 数学平均值 |
| `Min` | 数字 | 最小数 |
| `Max` | 数字 | 最大数 |
| `Sum` | 数字 | 所有数字之和 |
| `Range` | 数字 | 最大值 - 最小值 |
| `Median` | 数字 | 数学中位数 |
| `Stddev` | 数字 | 标准差 |
| `Earliest` | 日期 | 最早日期 |
| `Latest` | 日期 | 最晚日期 |
| `Range` | 日期 | 最晚 - 最早 |
| `Checked` | 布尔值 | true 值计数 |
| `Unchecked` | 布尔值 | false 值计数 |
| `Empty` | 任意 | 空值计数 |
| `Filled` | 任意 | 非空值计数 |
| `Unique` | 任意 | 唯一值计数 |

## 完整示例

### 任务追踪器 Base

```yaml
filters:
  and:
    - file.hasTag("task")
    - 'file.ext == "md"'

formulas:
  days_until_due: 'if(due, ((date(due) - today()) / 86400000).round(0), "")'
  is_overdue: 'if(due, date(due) < today() && status != "done", false)'
  priority_label: 'if(priority == 1, "🔴 高", if(priority == 2, "🟡 中", "🟢 低"))'

properties:
  status:
    displayName: 状态
  formula.days_until_due:
    displayName: "距截止日期"
  formula.priority_label:
    displayName: 优先级

views:
  - type: table
    name: "进行中的任务"
    filters:
      and:
        - 'status != "done"'
    order:
      - file.name
      - status
      - formula.priority_label
      - due
      - formula.days_until_due
    groupBy:
      property: status
      direction: ASC
    summaries:
      formula.days_until_due: Average

  - type: table
    name: "已完成"
    filters:
      and:
        - 'status == "done"'
    order:
      - file.name
      - completed_date
```

### 阅读列表 Base

```yaml
filters:
  or:
    - file.hasTag("book")
    - file.hasTag("article")

formulas:
  reading_time: 'if(pages, (pages * 2).toString() + " 分钟", "")'
  status_icon: 'if(status == "reading", "📖", if(status == "done", "✅", "📚"))'
  year_read: 'if(finished_date, date(finished_date).year, "")'

properties:
  author:
    displayName: 作者
  formula.status_icon:
    displayName: ""
  formula.reading_time:
    displayName: "预计时间"

views:
  - type: cards
    name: "书库"
    order:
      - cover
      - file.name
      - author
      - formula.status_icon
    filters:
      not:
        - 'status == "dropped"'

  - type: table
    name: "阅读清单"
    filters:
      and:
        - 'status == "to-read"'
    order:
      - file.name
      - author
      - pages
      - formula.reading_time
```

### 项目笔记 Base

```yaml
filters:
  and:
    - file.inFolder("Projects")
    - 'file.ext == "md"'

formulas:
  last_updated: 'file.mtime.relative()'
  link_count: 'file.links.length'

summaries:
  avgLinks: 'values.filter(value.isType("number")).mean().round(1)'

properties:
  formula.last_updated:
    displayName: "更新于"
  formula.link_count:
    displayName: "链接数"

views:
  - type: table
    name: "所有项目"
    order:
      - file.name
      - status
      - formula.last_updated
      - formula.link_count
    summaries:
      formula.link_count: avgLinks
    groupBy:
      property: status
      direction: ASC

  - type: list
    name: "快速列表"
    order:
      - file.name
      - status
```

### 日记索引

```yaml
filters:
  and:
    - file.inFolder("Daily Notes")
    - '/^\d{4}-\d{2}-\d{2}$/.matches(file.basename)'

formulas:
  word_estimate: '(file.size / 5).round(0)'
  day_of_week: 'date(file.basename).format("dddd")'

properties:
  formula.day_of_week:
    displayName: "星期"
  formula.word_estimate:
    displayName: "~字数"

views:
  - type: table
    name: "最近笔记"
    limit: 30
    order:
      - file.name
      - formula.day_of_week
      - formula.word_estimate
      - file.mtime
```

## 嵌入 Bases

在 Markdown 文件中嵌入：

```markdown
![[MyBase.base]]

<!-- 特定视图 -->
![[MyBase.base#视图名称]]
```

## YAML 引号规则

- 对包含双引号的公式使用单引号：`'if(done, "Yes", "No")'`
- 对简单字符串使用双引号：`"我的视图名称"`
- 在复杂表达式中正确转义嵌套引号

## 常见模式

### 按标签筛选
```yaml
filters:
  and:
    - file.hasTag("project")
```

### 按文件夹筛选
```yaml
filters:
  and:
    - file.inFolder("Notes")
```

### 按日期范围筛选
```yaml
filters:
  and:
    - 'file.mtime > now() - "7d"'
```

### 按属性值筛选
```yaml
filters:
  and:
    - 'status == "active"'
    - 'priority >= 3'
```

### 组合多个条件
```yaml
filters:
  or:
    - and:
        - file.hasTag("important")
        - 'status != "done"'
    - and:
        - 'priority == 1'
        - 'due != ""'
```

## 参考资料

- [Bases 语法](https://help.obsidian.md/bases/syntax)
- [函数](https://help.obsidian.md/bases/functions)
- [视图](https://help.obsidian.md/bases/views)
- [公式](https://help.obsidian.md/formulas)

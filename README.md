# Obsidian Bases 编辑器 (Obsidian Bases)

创建和编辑 Obsidian Bases 文件（`.base`），支持视图、筛选、公式和汇总。

## 功能

- 创建符合规范的 Bases 文件
- 支持表格视图和卡片视图
- 强大的筛选和排序功能
- 自定义公式属性
- 数据汇总统计

## 什么是 Obsidian Bases？

Bases 是 Obsidian 的数据库功能，可以像 Notion 数据库一样管理笔记。通过 `.base` 文件定义视图规则，动态展示和筛选笔记。

## 使用方法

在 Claude Code 中说：
- "创建一个 Base 文件来管理我的项目"
- "用表格视图展示所有带 #todo 标签的笔记"
- "设置筛选条件显示本周的笔记"

## 文件格式

```yaml
filters:
  and:
    - "tags contains #project"
    - "status = active"

formulas:
  days_left: "dateDiff(deadline, now())"

views:
  - name: 项目列表
    type: table
    columns: [file, status, deadline, days_left]
```

## 依赖

- Obsidian 1.5+（支持 Bases 功能）

## 作者

[@chaye7417](https://github.com/chaye7417)

## 许可证

MIT

---
name: lint
description: Use when the user says "lint", "检查", "健康检查", or wants to audit the knowledge base health under `Products/`. Checks for orphan pages, stale content, missing cross-references, and `Products/Notes/` backlog.
---

# Lint - 知识库健康检查

> **核心原则：只报告，不自动修改。所有修复操作必须等用户批准。**

## 第零步：确定检查范围

| 用户输入 | 检查范围 |
|---|---|
| `/lint Knowledge/` | 只检查 `Products/Knowledge/` |
| `/lint LifeOS/Investment/` | 只检查 `Products/LifeOS/Investment/` |
| `/lint all` | 全部 `Products/` Vault |
| `/lint`（无参数） | 先问用户要检查哪个目录 |

无参数时提示可选目录，只有明确说 `/lint all` 才全跑。

---

## 各目录健康标准

| 目录 | 重点 | 孤立页面算问题? |
|---|---|---|
| `Products/Knowledge/` | 交叉引用密度 | **是** |
| `Products/Writing/Operation/` | 交叉引用 | 是 |
| `Products/Software/Resources/` | 过时内容 | **否**（工具备忘天然独立） |
| `Products/LifeOS/` | 过时内容 | 否 |
| `Products/Writing/`（其他） | 孤立素材 | 视情况 |

---

## 四项检查

### 检查一：文件名与 tag 规范

**文件名：** 不允许含空格，用 `-` 代替。
**Tag 合法字符：** 字母（含中文）、数字、`-`、`_`、`/`。非法字符：空格（→ `-`）、点号（→ `-`）、冒号/括号等特殊符号（→ 删除）、纯数字（→ 加前缀如 `year-2026`）。

操作：`find 目标目录 -name "* *" -type f -name "*.md"` 找含空格文件名；提取所有 frontmatter tags 检测非法字符。

报告格式：
```
### 文件名与 tag 规范（N 个问题）
| 文件 | 问题类型 | 当前值 | 建议修正 |
```

### 检查二：孤立页面

**定义：** 没有任何其他文件的 `## 相关页面` 链接指向它。

操作：Glob 列出所有 `.md`，Grep 统计每个文件被 `[[文件名]]` 引用的次数，引用次数为 0 即为孤立。`Products/Software/Resources/` 下的孤立标注"工具备忘，孤立正常"，不列入问题。

报告格式：
```
### 孤立页面（N 篇）
| 文件 | 建议 |
```

### 检查三：过时内容

**定义：** 超过 90 天未修改。

操作：`find 目标目录 -name "*.md" -mtime +90` 按时间排序。注意区分"内容过时"和"内容仍有效只是没更新"（读书笔记 vs 投资策略）。

报告格式：
```
### 过时内容（N 篇，>90 天未更新）
| 文件 | 最后修改 | 天数 | 建议 |
```

### 检查四：缺失交叉引用

**定义：** 两篇内容高度相关的文件，双方 `## 相关页面` 都未互相链接。

操作：按子目录分组，读取文件标题和首段，识别强关联文件对（内容互补、理论与实践配对、读书笔记与应用配对）并确认链接缺失。只报告强关联，弱关联不提。

报告格式：
```
### 缺失交叉引用（N 对）
| 文件 A | 文件 B | 关联理由 |
```

---

## 汇总报告

```
## Lint 报告汇总（目录名）
| 检查项 | 数量 | 严重程度 |
|---|---|---|
| 文件名与 tag 规范 | X | ⚠️ / ✅ |
| 孤立页面 | Y | ⚠️ / ✅ |
| 过时内容 | Z | ⚠️ / ✅ |
| 缺失交叉引用 | W | ⚠️ / ✅ |

需要我修复哪些问题？
```

用户指示修复后，按 /query skill 的交叉引用标准双向操作。**禁止未经指示就修复任何问题。**

---

## 红线

- **禁止** 检查过程中修改任何文件
- **禁止** 只跑部分检查就出报告（四项全跑，除非用户指定）
- **禁止** 把 `Products/Software/Resources/` 的孤立页面列为问题

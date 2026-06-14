---
name: scan-x-primitives
description: Use when the user says "行动原语选题", "从X找工具教程选题", "scan-x-primitives", or wants to mine X/Twitter specifically for A类工具教程 topics using the 五个行动原语 framework.
version: 0.1.0
last_updated: 2026-06-14
---

# Scan-X-Primitives — 行动原语工具教程选题侦察

> 专注 A 类工具教程选题，五个行动原语各出 3 条搜索（英文主 + 英文场景 + 中文），共 15 条精准搜索，产出可直接进入 score-topic 的候选选题。

---

## 核心规则

**用 CDP 控制已登录的 Chrome，对每个行动原语执行多组 X 搜索，筛选出台阶 L4-7 的工具教程选题。**

---

## 第一步：加载选题框架

优先在 `Products/Writing/` 下检查并读取现有文档。若文件不存在，先如实说明缺失，再继续执行可完成部分。

优先查找以下文档：
- `Products/Writing/Playbook/行动原语选题法.md` — 五个原语的定义、场景库、判断标准
- `Products/Writing/Operation/题材分类与用户共鸣模型.md` — 台阶系统、L4-7 甜区
- `Products/Writing/topic.md` — 记录已有选题，避免重复推荐

---

## 第二步：CDP 搜索

### 前提

Chrome 浏览器需已登录 X 账号，CDP 调试端口已开放（`--remote-debugging-port=9222`）。

### since_date 计算

`{since_date}` = 今天 -7 天，格式 `YYYY-MM-DD`。

### 搜索组（15 条，按原语分组）

```
# 原语 1：CDP 浏览器自动化
https://x.com/search?q="browser+use"+AI+OR+"computer+use"+agent+OR+stagehand+browser+min_faves:200+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=browser+automation+AI+tutorial+OR+playwright+AI+agent+OR+puppeteer+AI+min_faves:150+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=AI+操控+浏览器+OR+AI+浏览器+自动化+OR+电脑使用+AI+min_faves:50+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top

# 原语 2：Agent Skills（智能体技能）
https://x.com/search?q="Claude+skill"+OR+"Claude+Code+skill"+min_faves:100+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=AI+agent+workflow+custom+OR+"slash+command"+AI+OR+agent+template+library+min_faves:150+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=Claude+技能+OR+Claude+Skill+OR+斜杠命令+AI+min_faves:50+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top

# 原语 3：MCP 工具连接
https://x.com/search?q=MCP+Claude+OR+"model+context+protocol"+Claude+min_faves:200+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=MCP+server+tutorial+OR+MCP+integration+new+OR+MCP+tool+AI+min_faves:100+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=MCP+Claude+OR+MCP+工具+OR+模型上下文协议+min_faves:50+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top

# 原语 4：个人私有数据处理
https://x.com/search?q=Ollama+local+AI+OR+"local+LLM"+files+OR+personal+RAG+tutorial+min_faves:100+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q="local+AI"+documents+OR+private+data+LLM+OR+"local+knowledge+base"+min_faves:100+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=Ollama+本地+AI+OR+本地知识库+OR+微信记录+分析+AI+OR+Obsidian+AI+min_faves:30+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top

# 原语 5：触发式自动化
https://x.com/search?q=n8n+AI+tutorial+OR+n8n+Claude+OR+n8n+LLM+workflow+min_faves:100+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q="AI+automation"+workflow+OR+Zapier+AI+OR+Make+AI+integration+min_faves:100+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
https://x.com/search?q=n8n+AI+OR+n8n+自动化+OR+AI+自动流程+OR+定时+AI+任务+min_faves:30+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top
```

### CDP 抓取流程

每条搜索 URL 顺序执行：

```
1. navigate_page → 导航到 URL
2. wait_for → 等待推文加载（2-3 秒）
3. evaluate_script → 执行抓取函数
4. 如结果 < 5 条，执行滚动后再抓一次
```

### JS 抓取函数

```javascript
() => {
  const articles = document.querySelectorAll('article[data-testid="tweet"]');
  const results = [];
  articles.forEach(el => {
    const text = el.querySelector('[data-testid="tweetText"]')?.innerText || '';
    const likes = el.querySelector('[data-testid="like"] span')?.innerText || '0';
    const rts = el.querySelector('[data-testid="retweet"] span')?.innerText || '0';
    const user = el.querySelector('[data-testid="User-Name"]')?.innerText?.split('\n')[0] || '';
    const links = Array.from(el.querySelectorAll('a[href*="/status/"]'));
    const statusLink = links.find(a => a.href.match(/\/status\/\d+$/));
    const url = statusLink ? statusLink.href : '';
    if (text.length > 20) results.push({ user, text: text.substring(0, 300), likes, rts, url });
  });
  return results;
}
```

### 滚动函数

```javascript
() => { window.scrollBy(0, 1500); return 'scrolled'; }
```

---

## 第三步：筛选与分类

### 硬性过滤

- 去掉重复推文（URL 相同）
- 去掉无正文内容
- 点赞 < 30 且转发 < 10 丢弃（A 类阈值放宽以覆盖中文推文）

### 流水线确认

所有结果均为 **A 类工具教程**。核心判断：用户能学会一个可复现的动作吗？

- 能 → 保留
- 不能（泛泛讨论、无具体操作）→ 丢弃

### 行动原语标注

| 行动原语 | 关键词信号 |
|---------|-----------|
| CDP 浏览器自动化 | browser use, computer use, stagehand, playwright AI, 浏览器操控 |
| Agent Skills | Claude skill, agent workflow, slash command, 斜杠命令, 技能文件 |
| MCP 工具连接 | MCP, model context protocol, MCP server, 工具连接 |
| 私有数据处理 | local LLM, Ollama, private data, local RAG, 本地知识库, 个人数据 |
| 触发式自动化 | n8n, Zapier, Make, webhook, cron, trigger, 定时任务 |

### 台阶预判

| 台阶 | 标注 | 处置 |
|------|------|------|
| L1-3 | ⚠️ 受众不符 | 偏小白，目标受众是有基础的从业者/创业者，标注后由用户决定 |
| L4-7 | ✅ | 正常推进 |
| L8 | ⚠️ | 警告，标注降级建议 |
| L9-10 | ⚠️⚠️ | 危险，仅保留有强降级空间的 |
| L11+ | ❌ | 直接淘汰 |

---

## 第四步：输出结果

按原语分组输出，每组格式如下：

```
## A 类 — 工具教程选题（按行动原语分组）

### 原语 1：CDP 浏览器自动化

| # | 内容摘要 | 用户 | 台阶 | 点赞 | 链接 |
|---|---------|------|------|------|------|
| 1 | ...     | @xx  | L4 ✅ | 1.2K | https://x.com/... |

**角度建议**：一句话说明能做什么角度的视频
**目标用户**：哪类人会点开，对应 L4-7 用户画像

### 原语 2：Agent Skills
...（同格式，无结果则注明"本期无符合条件推文"）

### 原语 3：MCP 工具连接
...

### 原语 4：个人私有数据处理
...

### 原语 5：触发式自动化
...
```

完成后提示：**对感兴趣的选题，执行 `/score-topic` 做交付端深度诊断。**

---

## 红线

- **禁止** 编造点赞数、转发数（抓不到就标注"数据未获取"）
- **禁止** 绕过 CDP 用 WebSearch 替代
- **禁止** 跳过台阶预判（L11+ 必须淘汰）
- **禁止** 输出没有链接的选题
- **禁止** 未经 CDP 抓取就凭印象编写推文内容

---

## 相关页面

- [[行动原语选题法]] — 五个原语的详细定义与场景库
- [[题材分类与用户共鸣模型]] — 台阶系统
- [[选题决策框架]] — A/B/C 三条流水线

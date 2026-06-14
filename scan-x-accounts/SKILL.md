---
name: scan-x-accounts
description: Use when the user says "监控X账号", "特定账号选题", "scan-x-accounts", "看特定推主", or wants to scan specific tracked X/Twitter accounts for topic signals.
---

# Scan-X-Accounts — 特定 X 账号内容监控

> 专注监控主人精选的 X 账号列表，抓取近期高质量内容，提取 A/B 类选题信号。适合跟踪特定意见领袖的最新输出。

---

## 核心规则

**用 CDP 控制已登录的 Chrome，对精选账号列表执行定向搜索，捕捉近期高互动推文，提取选题信号。**

---

## 第一步：加载选题框架

优先在 `Products/Writing/` 下检查并读取现有文档。若文件不存在，先如实说明缺失，再继续执行可完成部分。

优先查找以下文档：
- `Products/Writing/Playbook/行动原语选题法.md` — 识别 A 类工具教程信号
- `Products/Writing/Operation/选题决策框架.md` — A/B 流水线判断
- `Products/Writing/topic.md` — 记录已有选题，避免重复推荐

---

## 第二步：精选账号列表

> **主人可随时修改此列表。** 添加账号：直接在对应分组追加 `@handle`；删除账号：移除对应行。

#### 🔧 技术 / 工具向（重点关注 A 类工具教程信号）

```
@borischen       # Boris Cherny — Claude Code 核心开发者
@addyosmani      # Addy Osmani — 前 Google Chrome 工程总监
@simonw          # Simon Willison — AI 工具实践专家
@karpathy        # Andrej Karpathy — AI 教育内容
@AnthropicAI     # Anthropic 官方账号
@OpenAI          # OpenAI 官方账号
@deepseek_ai     # DeepSeek 官方账号
@levelsio        # Pieter Levels — vibe coding 先驱，50万粉
@steipete        # Peter Steinberger — Claude Code 重度用户
```

#### 📈 趋势 / 产品向（重点关注 B 类热点信号）

```
@sama            # Sam Altman — OpenAI CEO
@gregisenberg    # Greg Isenberg — AI 产品趋势
@paulg           # Paul Graham — 创业/思维
@mattshumer_     # Matt Shumer — AI 应用实践
@nikitabier      # Nikita Bier — 病毒产品增长专家，20万粉
@AYi_AInotes     # 阿绎 AYi — 中文 AI 工具实践，高互动
```

#### 🇨🇳 中文 AI 社区

```
@huashushe       # 花叔 — 设计 × AI
@dotey           # 宝玉 — AI 翻译/应用
@Kimi_Moonshot   # Kimi 官方账号
@ManusAI_HQ      # Manus AI 官方账号
@iguangzhengli   # 光正龙 — 中文技术博主
```

---

## 第三步：CDP 搜索

### 前提

Chrome 浏览器需已登录 X 账号，CDP 调试端口已开放（`--remote-debugging-port=9222`）。

### since_date 计算

`{since_date}` = 今天 -14 天（账号监控窗口稍长），格式 `YYYY-MM-DD`。

### 搜索组（6 条，每组不超过 5 个账号防止 OR 截断）

```
# 技术/工具向 — 1
https://x.com/search?q=(from:borischen+OR+from:addyosmani+OR+from:simonw+OR+from:karpathy+OR+from:AnthropicAI)+min_faves:100+since:{since_date}&src=typed_query&f=top

# 技术/工具向 — 2
https://x.com/search?q=(from:OpenAI+OR+from:deepseek_ai+OR+from:levelsio+OR+from:steipete)+min_faves:200+since:{since_date}&src=typed_query&f=top

# 趋势/产品向 — 1
https://x.com/search?q=(from:sama+OR+from:gregisenberg+OR+from:paulg+OR+from:mattshumer_)+min_faves:300+since:{since_date}&src=typed_query&f=top

# 趋势/产品向 — 2
https://x.com/search?q=(from:nikitabier+OR+from:AYi_AInotes)+min_faves:100+since:{since_date}&src=typed_query&f=top

# 中文 AI 社区 — 1
https://x.com/search?q=(from:huashushe+OR+from:dotey)+min_faves:50+since:{since_date}&src=typed_query&f=top

# 中文 AI 社区 — 2
https://x.com/search?q=(from:Kimi_Moonshot+OR+from:ManusAI_HQ+OR+from:iguangzhengli)+min_faves:50+since:{since_date}&src=typed_query&f=top
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

## 第四步：筛选与分类

### 硬性过滤

- 去掉重复推文（URL 相同）
- 无正文内容的丢弃
- 中文账号：点赞 < 30 且转发 < 10 丢弃
- 英文账号：点赞 < 80 且转发 < 20 丢弃

### 流水线映射

- 内容指向"某种底层能力/工具用法" → **A 类工具教程**（标注行动原语）
- 内容指向"某件具体的事/新产品" → **B 类热点**（标注题材+时效）

### 台阶预判

| 台阶 | 标注 | 处置 |
|------|------|------|
| L3-6 | ✅ | 正常推进 |
| L7-8 | ⚠️ | 警告，标注降级建议 |
| L9-10 | ⚠️⚠️ | 危险，技术向账号常见 |
| L11+ | ❌ | 直接淘汰 |

> **注意**：技术向账号（如 @karpathy @simonw）内容台阶普遍偏高（L8-10+），着重评估降级可行性。

---

## 第五步：输出结果

```
## 精选账号监控结果

### A 类 — 工具教程信号

| # | 账号 | 内容摘要 | 行动原语 | 台阶 | 点赞 | 链接 |
|---|------|---------|---------|------|------|------|
| 1 | @xx  | ...     | CDP 浏览器自动化 | L5 ✅ | 800 | https://x.com/... |

**角度建议**：...
**目标用户**：...

### B 类 — 热点信号

| # | 账号 | 事件/产品 | 题材 | 台阶 | 时效 | 点赞 | 链接 |
|---|------|----------|------|------|------|------|------|
| 1 | @xx  | ...      | 产品分析 | L5 ✅ | 🔥 1周 | 2K | https://... |
```

完成后提示：**对感兴趣的选题，执行 `/score-topic` 做交付端深度诊断。**

---

## 红线

- **禁止** 编造点赞数、转发数
- **禁止** 绕过 CDP 用 WebSearch 替代
- **禁止** 跳过台阶预判（L11+ 直接淘汰）
- **禁止** 输出没有链接的选题

---

## 相关页面

- [[行动原语选题法]] — A 类工具教程判断标准
- [[选题决策框架]] — A/B/C 三条流水线
- [[题材分类与用户共鸣模型]] — 台阶系统

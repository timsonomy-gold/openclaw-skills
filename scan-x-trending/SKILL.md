---
name: scan-x-trending
description: Use when the user says "X热点", "抓X新闻", "scan-x-trending", "X热点选题", or wants to mine X/Twitter for B类 hot news and product analysis topics.
---

# Scan-X-Trending — X 平台热点选题侦察

> 专注 B 类热点，覆盖大模型发布、AI 工具爆款、行业资金/政策、病毒演示、中文 AI 生态，快速判断时效窗口，产出新闻快评和产品分析候选选题。

---

## 核心规则

**用 CDP 控制已登录的 Chrome，执行 10 条多维热点搜索，按时效和热度筛选，映射到 B 类新闻快评/产品分析流水线。**

---

## 第一步：加载选题框架

优先在 `Products/Writing/` 下检查并读取现有文档。若文件不存在，先如实说明缺失，再继续执行可完成部分。

优先查找以下文档：
- `Products/Writing/Operation/题材分类与用户共鸣模型.md` — 台阶系统、L4-7 甜区、时效性标注
- `Products/Writing/Operation/选题决策框架.md` — B 类流水线的时效判断规则
- `Products/Writing/topic.md` — 记录已有选题，避免重复推荐

---

## 第二步：CDP 搜索

### 前提

Chrome 浏览器需已登录 X 账号，CDP 调试端口已开放（`--remote-debugging-port=9222`）。

### since_date 计算

`{since_date}` = 今天 -7 天，格式 `YYYY-MM-DD`。

### 搜索组（10 条，维度各异）

```
# 1. 中文大模型热点（含国产模型）
https://x.com/search?q=(Claude+OR+ChatGPT+OR+Gemini+OR+Kimi+OR+豆包+OR+通义+OR+DeepSeek+OR+文心)+发布+OR+更新+OR+新功能+min_faves:300+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top

# 2. 中文 AI 产品评测 / 体验
https://x.com/search?q=(AI+工具+OR+AI+产品+OR+AI+应用)+(评测+OR+体验+OR+测评+OR+上手)+min_faves:200+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top

# 3. 英文大模型发布（含 Grok / Llama / Mistral / DeepSeek）
https://x.com/search?q=(Claude+OR+GPT+OR+Gemini+OR+Grok+OR+Llama+OR+Mistral+OR+DeepSeek)+(new+OR+release+OR+launch+OR+update)+min_faves:1000+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top

# 4. AI 编程工具（Cursor / Copilot / vibe coding）
https://x.com/search?q=(Cursor+OR+Copilot+OR+"vibe+coding"+OR+"AI+coding"+OR+"Claude+Code")+(new+OR+update+OR+tip+OR+workflow)+min_faves:500+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top

# 5. AI 公司 / 行业格局（xAI / Meta / Mistral）
https://x.com/search?q=(Anthropic+OR+OpenAI+OR+xAI+OR+"Meta+AI"+OR+Mistral+OR+"Google+DeepMind")+(funding+OR+partnership+OR+release+OR+valuation)+min_faves:500+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top

# 6. AI 爆款演示 / 病毒视频
https://x.com/search?q=("built+with+AI"+OR+"using+AI"+OR+"AI+demo"+OR+"AI+made+this")+min_faves:2000+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top

# 7. AI 行业资金 / 政策 / 就业影响
https://x.com/search?q=("AI+funding"+OR+"AI+raises"+OR+"AI+regulation"+OR+"AI+jobs"+OR+"AI+layoffs")+min_faves:500+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top

# 8. 中文 AI 行业事件（融资 / 监管 / 产业动态）
https://x.com/search?q=(AI+融资+OR+AI+监管+OR+AI+政策+OR+AI+就业+OR+AI+取代+OR+AI+创业)+min_faves:200+-is:retweet+lang:zh+since:{since_date}&src=typed_query&f=top

# 9. AI 视频 / 图像生成（Sora / Midjourney / Runway / Kling）
https://x.com/search?q=(Sora+OR+Midjourney+OR+Runway+OR+Kling+OR+Suno+OR+"AI+video"+OR+"AI+image")+(new+OR+update+OR+release+OR+demo)+min_faves:1000+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top

# 10. AI Agent 自主任务执行
https://x.com/search?q=("AI+agent"+OR+"autonomous+AI"+OR+"agentic+AI"+OR+"multi-agent")+(new+OR+demo+OR+built+OR+workflow)+min_faves:500+-is:retweet+lang:en+since:{since_date}&src=typed_query&f=top
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
- 点赞 < 200 的内容丢弃
- 去掉无正文内容

### B 类题材判断

| 题材 | 判断信号 | 时效 |
|------|---------|------|
| **新闻快评** | 突发事件/产品发布/行业动态，主体是"某件事" | 🔥🔥 24-48h 窗口 |
| **产品分析** | 具体产品评测/对比/使用体验，主体是"某个产品" | 🔥 1周窗口 |

核心问题：**我能给出清晰判断，而不只是复述事实吗？**
- 能 → 保留
- 不能 → 丢弃

### 台阶预判

| 台阶 | 标注 | 处置 |
|------|------|------|
| L1-3 | ⚠️ 受众不符 | 偏小白，目标受众是有基础的从业者/创业者，标注后由用户决定 |
| L4-7 | ✅ | 正常推进 |
| L8 | ⚠️ | 警告，标注降级建议 |
| L9-10 | ⚠️⚠️ | 危险 |
| L11+ | ❌ | 直接淘汰 |

---

## 第四步：输出结果

```
## B 类 — 热点选题

| # | 事件/产品 | 题材 | 台阶 | 时效 | 点赞 | 链接 |
|---|----------|------|------|------|------|------|
| 1 | ...      | 新闻快评 | L5 ✅ | 🔥🔥 24h | 8K | https://... |
| 2 | ...      | 产品分析 | L5 ✅ | 🔥 1周  | 4K | https://... |
```

每条选题附：
- **角度建议**：我能给出的核心判断是什么
- **目标用户**：哪类人会点开
- **时效判断**：窗口还剩多久，是否值得抢发
- **原始推文 URL**（完整链接）

完成后提示：**对感兴趣的选题，执行 `/score-topic` 做交付端深度诊断。**

---

## 红线

- **禁止** 编造点赞数、转发数
- **禁止** 绕过 CDP 用 WebSearch 替代
- **禁止** 跳过台阶预判（L11+ 直接淘汰）
- **禁止** 输出没有链接的选题
- **禁止** 保留"没有判断角度"的热点（复述新闻无价值）

---

## 相关页面

- [[题材分类与用户共鸣模型]] — 时效性标注规范
- [[选题决策框架]] — B 类流水线的完整逻辑

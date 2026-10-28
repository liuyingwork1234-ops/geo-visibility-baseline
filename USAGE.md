> 本文件与 [geo-skills-guide](https://github.com/liuyingwork1234-ops/geo-skills-guide) 同步维护。如需最新版本，以该仓库为准。

# GEO Skills 使用说明书

两个 Agent Skill 的完整使用手册：从安装、调用，到它们背后调用的每一个付费接口。

```
目录
├── 一、这两个 Skill 是什么
├── 二、安装
├── 三、怎么用（触发话术 + 完整流程）
├── 四、依赖的接口清单（含付费项）
├── 五、接口配置方法
├── 六、成本估算
├── 七、没有 API 也能用的部分
├── 八、常见问题
└── 九、踩坑记录
```

---

## 一、这两个 Skill 是什么

| Skill | 解决什么问题 | 一句话 |
|---|---|---|
| **geo-competitor-mining** | AI 为什么不推荐你 | 扒对手的结构，做成你的模板 |
| **geo-visibility-baseline** | 做了这么多到底有没有用 | 固定问题集，隔期复测，三层归因 |

**它们不是「一键生成内容」的工具。** 输出的是**诊断结果和修复清单**，内容生产要用别的 Skill（如 `yao-geo-ranking-article-builder`）。

### 为什么需要它们

绝大多数 GEO 工具都从「我自己」出发——我的关键词、我的页面、我的资料。

但 AI 推荐的依据，首先来自**别人怎么写你**。

这两个 Skill 是少数「以对手为中心」和「以验证为中心」的入口。

---

## 二、安装

### 方式 A：OpenClaw 技能目录

```bash
mkdir -p ~/.openclaw/skills/geo-competitor-mining
mkdir -p ~/.openclaw/skills/geo-visibility-baseline

# 从各自仓库下载 SKILL.md 放进去
```

目录结构：

```
~/.openclaw/skills/
├── geo-competitor-mining/
│   └── SKILL.md
└── geo-visibility-baseline/
    └── SKILL.md
```

### 方式 B：Skill Workshop（如果你用 OpenClaw）

```bash
# 用 skill_workshop 工具 create + apply
```

### 方式 C：直接当文档看

两个 `SKILL.md` 本身就是完整流程文档，不装也能照着手工做。

---

## 三、怎么用

### 3.1 触发话术

**geo-competitor-mining：**

| 你说 | 它做什么 |
|---|---|
| 「帮我扒一下 XX 行业的竞品」 | 完整五步 |
| 「为什么问 AI 没有推荐我们公司」 | 先摸战场，再扒底裤 |
| 「拆解一下这几个对标网站」 | 直接第 2 步 |
| 「给我们的 GEO 内容做一套模板」 | 扒完合成模板 |
| 「复盘一下我们官网的 GEO 结构」 | 第 4 步审自家 |

**geo-visibility-baseline：**

| 你说 | 它做什么 |
|---|---|
| 「测一下 AI 现在认不认识我们」 | 建基线 |
| 「上个月的改动有没有效果」 | 复测 + 归因 |
| 「做个 AI 可见度基线」 | 建问题集 + 跑基线 |

### 3.2 完整流程（五步）

```
① 摸战场  →  ② 扒底裤  →  ③ 合成模板  →  ④ 审自家  →  ⑤ 出清单
   ↓           ↓            ↓             ↓            ↓
竞品名单    六项共性      布尔条件      得分表        P0/P1/P2
```

**每步都有可检查的完成标准**，避免输出「感觉差不多」的建议。

### 3.3 你会拿到什么

```
① 竞品名单表（含来源文章）
② 六项共性清单
③ 可套用文章模板（8 条布尔条件）
④ 自家得分表（每页 X/8）+ 缺口清单
⑤ 分级修复清单（P0 必须零成本）
```

---

## 四、依赖的接口清单

### 4.1 总览

| 接口 | 用途 | 是否付费 | 是否必需 |
|---|---|---|---|
| **大模型 API** | 摸战场：问 AI 要推荐名单 | ✅ **付费** | ✅ 必需 |
| **搜索 API**（Tavily 等） | 检索层探测、找来源文章 | ✅ **付费**（有免费额度）| ⭕ 建议 |
| **网页抓取** | 扒底裤：提取对标页六项 | ❌ 免费 | ✅ 必需 |
| **IndexNow** | 推送 URL 给 Bing/Yandex | ❌ 免费 | ⭕ 可选 |
| **百度搜索资源平台 API** | 推送 URL 给百度 | ❌ 免费 | ⭕ 可选 |

### 4.2 付费接口详解

#### ① 大模型 API（核心）

**用途**：`geo-competitor-mining` 第 1 步「摸战场」，和 `geo-visibility-baseline` 全流程。

**为什么必须用**：要拿到 AI 真实推荐的名单，只能真的去问它。搜索引擎返回的是网页列表，不是 AI 的推荐答案。

**已验证可用的三家：**

| 服务商 | 兼容接口 | Base URL | 说明 |
|---|---|---|---|
| 阿里云百炼 | OpenAI 兼容 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | 本项目实测使用 |
| DeepSeek | OpenAI 兼容 | `https://api.deepseek.com/v1` | 便宜，中文好 |
| Moonshot（Kimi） | OpenAI 兼容 | `https://api.moonshot.cn/v1` | 长上下文 |
| OpenAI | 原生 | `https://api.openai.com/v1` | 英文场景更准 |
| Anthropic | 原生 | `https://api.anthropic.com/v1` | 需改请求格式 |

**本项目实测配置**（阿里云百炼）：

```bash
GEO_LLM_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
GEO_LLM_API_KEY=<你的密钥>
GEO_LLM_MODEL=qwen3.7-plus
```

> 注：如果走专属工作空间，Base URL 会带 workspace 前缀，形如
> `https://ws-xxxx.cn-beijing.maas.aliyuncs.com/compatible-mode/v1`。
> 具体地址在阿里云百炼控制台的「API-KEY 管理」页可见。

**调用方式**（OpenAI 兼容）：

```bash
curl -X POST "$GEO_LLM_BASE_URL/chat/completions" \
  -H "Authorization: Bearer $GEO_LLM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$GEO_LLM_MODEL"'",
    "messages": [{"role":"user","content":"推荐几家做外贸GEO的服务商，要具体公司名"}],
    "max_tokens": 500,
    "temperature": 0.3
  }'
```

#### ② 搜索 API（建议配）

**用途**：

- 找被推荐公司的**来源文章**（摸战场的副产品，也是后续投递目标）
- 做**检索层**探测（区别于模型层的知识探测）

**已验证可用：**

| 服务商 | 免费额度 | 说明 |
|---|---|---|
| **Tavily** | 1,000 次/月 | 专为 AI 搜索设计，返回结构化摘要 |
| Brave Search API | 2,000 次/月 | 便宜，覆盖广 |
| SerpAPI | 100 次/月 | 结果最像真实搜索引擎，贵 |

**本项目实测配置**（Tavily）：

```bash
TAVILY_API_KEY=<你的密钥>
```

**调用方式**：

```bash
curl -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "'"$TAVILY_API_KEY"'",
    "query": "2026 外贸 GEO 服务商 盘点 推荐",
    "search_depth": "advanced",
    "max_results": 10
  }'
```

#### ③ 可选：搜索引擎官方 API

| 接口 | 用途 | 费用 |
|---|---|---|
| Bing Web Search API | 补充检索 | 付费（已并入 Azure） |
| Google Custom Search JSON API | 补充检索 | 100 次/天免费，超出 $5/1000 次 |

**这两个不是必需的**，Tavily 或 Brave 够用。

### 4.3 免费接口

| 接口 | 端点 | 用途 |
|---|---|---|
| **IndexNow** | `https://api.indexnow.org/indexnow` | 推送 URL 给 Bing / Yandex，秒级生效 |
| **百度普通收录推送** | `http://data.zz.baidu.com/urls?site=&token=` | 推送 URL 给百度，日限 10 条 |
| **目标站点 HTML** | 直接 GET | 扒底裤的六项提取 |

**IndexNow 调用示例：**

```bash
curl -X POST https://api.indexnow.org/indexnow \
  -H "Content-Type: application/json" \
  -d '{
    "host": "example.com",
    "key": "<你的 key>",
    "keyLocation": "https://example.com/<key>.txt",
    "urlList": ["https://example.com/blog/new-post.html"]
  }'
```

---

## 五、接口配置方法

### 5.1 用环境变量（推荐）

```bash
# ~/.zshrc 或项目的 .env
export GEO_LLM_BASE_URL="https://dashscope.aliyuncs.com/compatible-mode/v1"
export GEO_LLM_API_KEY="sk-xxxxxxxx"
export GEO_LLM_MODEL="qwen3.7-plus"
export TAVILY_API_KEY="tvly-xxxxxxxx"
```

### 5.2 用 .env 文件（Agent 场景推荐）

```
# .env —— 记得加进 .gitignore
GEO_LLM_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
GEO_LLM_API_KEY=sk-xxxxxxxx
GEO_LLM_MODEL=qwen3.7-plus
TAVILY_API_KEY=tvly-xxxxxxxx
```

Python 读取：

```python
def load_env(path):
    d = {}
    for line in open(path, encoding="utf-8"):
        line = line.strip()
        if line and not line.startswith("#") and "=" in line:
            k, v = line.split("=", 1)
            d[k.strip()] = v.strip().strip('"').strip("'")
    return d
```

### 5.3 安全红线

```
❌ 不要把密钥写进 SKILL.md / README / 任何会提交到仓库的文件
❌ 不要用 echo 打印密钥
❌ 不要把 .env 提交到 Git

✅ .env 加进 .gitignore
✅ 密钥只放环境变量或密钥管理服务
✅ 仓库里只写变量名，不写值
```

**本说明书全文不含任何真实密钥。** 上面出现的 `sk-xxxxxxxx` 均为占位符。

---

## 六、成本估算

### 6.1 单次完整分析（geo-competitor-mining 五步）

| 步骤 | 接口调用 | 用量 |
|---|---|---|
| 摸战场 | 大模型 × 6-8 次 | 输入 ~1.5K tokens/次，输出 ~500 tokens/次 |
| 扒底裤 | 网页 GET × 5-8 次 | 免费 |
| 合成模板 | 无 | — |
| 审自家 | 网页 GET × 4-6 次 | 免费 |
| 出清单 | 无 | — |

**合计 token 用量：约 12K 输入 + 4K 输出。**

按国产大模型常见价位（千 tokens 几分钱量级），**单次完整分析在 0.1 元以内**。

### 6.2 单次基线测试（geo-visibility-baseline）

```
8 个问题 × 1 个平台 = 8 次调用
约 12K 输入 + 4K 输出
同一量级：0.1 元以内
```

**如果要测 3 个平台，成本约 3 倍。**

### 6.3 搜索 API

```
Tavily：1,000 次/月免费额度
单次分析用 2-5 次搜索
免费额度可支撑 200-500 次分析/月
```

### 6.4 成本结论

```
月成本（50 次分析 + 20 次基线测试）
  大模型      ≈ 10 元以内
  搜索 API   免费额度内
  其他       免费
  ─────────────────
  合计        约 10 元/月
```

> 具体单价以各服务商控制台为准。这里给的是量级，不是报价。

---

## 七、没有 API 也能用的部分

**这是很多人忽略的：这套方法的手工版几乎不需要任何付费接口。**

| 步骤 | 手工替代 | 成本 |
|---|---|---|
| 摸战场 | 手动在豆包 / DeepSeek / Kimi / 元宝 里问，手动抄名单 | 免费 |
| 扒底裤 | 浏览器右键「查看源代码」，手动数 H2、搜 `ld+json` | 免费 |
| 合成模板 | 手写一张清单 | 免费 |
| 审自家 | 同上，逐页对照 | 免费 |
| 出清单 | 手写 | 免费 |

**API 只是让这件事可以重复跑、批量跑。**

**如果只做一次诊断，手工比配 API 快。**

---

## 八、常见问题

**Q：必须用哪个大模型？**

不必须。任何能回答「推荐几家 XX 服务商」的模型都行。**但同一个基线测试必须固定用同一个模型**，换模型则数据不可比。

**Q：模型回答每次都不一样，怎么算数？**

单次结果不作结论。**看三次以上的趋势**，差异小于 2 个问题视为噪声。

**Q：搜索层和模型层要分开测吗？**

**必须分开。** 两者结论可能完全相反——实测某品牌搜索层能查到官网和大量文章，模型层却判定「这不像是真实存在的东西」。

**Q：API 超时怎么办？**

首次调用常见 30-75 秒。建议：`timeout=90`、`max_tokens=400`、失败重试 1 次。

**Q：为什么我的品牌词命中了，品类词全灭？**

这是典型状态：**你的内容被索引了，但没有被吸收**。品牌词靠检索，品类词靠知识。

**Q：要多久复测一次？**

两周。太短看不出变化，太长错过归因窗口。

---

## 九、踩坑记录

真实踩过、已修正的：

**① Python 输出缓冲 → 后台任务日志空白**

```bash
# ❌ 日志一直空
python3 script.py > log.txt 2>&1 &

# ✅ 加 -u 关闭缓冲
python3 -u script.py > log.txt 2>&1 &
```

**② SSH 断连杀掉后台进程**

```bash
# ❌ SSH 一断就死
nohup python3 script.py &

# ✅ 用 setsid 真正脱离
setsid nohup python3 script.py > log.txt 2>&1 < /dev/null &
```

**③ 从 AI 回答里提取公司名，正则不可靠**

实测抓到的是「靠谱的AI」「结构化数据」这类短语，不是公司名。

建议：**保存完整回答原文**，再做二次提取或人工复核。不要指望一条正则。

**④ 只测品牌词当成绩**

问「XX 公司怎么样」必然命中（如果它能检索到）。**要测品类词才有意义。**

**⑤ 把「百度收录数」当成「AI 推荐度」**

两个完全不同的指标。收录 4 页的站点可能被 AI 正常引用；收录 400 页的站点可能 AI 完全不知道。

---

## 附：两个仓库

```
geo-competitor-mining     https://github.com/liuyingwork1234-ops/geo-competitor-mining
geo-visibility-baseline   https://github.com/liuyingwork1234-ops/geo-visibility-baseline
```

均 MIT 协议。

---

*本说明书基于 2026-10 的实测编写。接口地址与计费方式可能变化，以各服务商官方文档为准。*

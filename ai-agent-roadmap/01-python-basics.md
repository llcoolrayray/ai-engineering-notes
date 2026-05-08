# 01 · Python 快速上手

> **状态**:✅ 已完成
> **完成时间**:2026-05-08
> **实际花费**:约 5 小时(单次集中学习,未分散到 1-2 周)
> **学习方式**:跳过语法刷题,直接通过 LLM API 调用实战驱动

---

## 📌 阶段目标(摘自 [00-roadmap.md](00-roadmap.md))

> 能读懂 AI 项目代码,能改能跑。用 Cursor 辅助,不需要从零刷语法。

---

## ✅ 进度更新

### 已完成的子项

| 子项 | 原计划 | 实际状态 | 说明 |
|---|---|---|---|
| Python 语法速通(Java 对照版) | 3 天 | ✅ 已完成 | 通过实战代码理解,未刷题 |
| NumPy / Pandas 核心操作 | 4 天 | ⏸️ 暂跳过 | 阶段三 RAG 时再学(Embedding 向量运算才用到) |
| 异步编程 async/await | 3 天 | ⏸️ 暂跳过 | 阶段二 stream 输出按需补 30 分钟 |

### ⚠️ 关于跳过 NumPy/Pandas/async 的判断

**这是判断,不是事实**。

理由:
1. 阶段二(LLM + Prompt 工程)主要用 OpenAI SDK + 文本处理,**用不到 NumPy/Pandas**
2. 阶段三 RAG 才真正用到向量运算时再学,**有具体场景驱动效率高 3 倍**
3. async 阶段二 stream 输出会用到一点,30 分钟能补
4. 真实学习时间碎片化,先打通"能用 LLM 做事"主线比按教科书顺序学更现实

**风险**:如果阶段二某个示例代码用了 async,我可能要临时停下来补这块。可接受。

---

## 🛠️ 学到什么

### 技术清单

**Python 项目骨架**
- `venv` 虚拟环境管理(PyCharm 自动创建,但要懂手动建的原理)
- `pip install` + `requirements.txt`
- `.env` 文件 + `python-dotenv` 管理 API key
- `.gitignore` 排除 `.env` / `.venv/` / `__pycache__/`

**Python 关键语法**(Java 对照)
- 类型注解:`def foo(x: list[int]) -> dict[str, int]:`(运行时不强制,IDE/给人看的)
- 装饰器:`@retry(...)` 等同 `func = retry(...)(func)`,类比 Spring AOP `@Around`
- f-string:`f"{name}"` 等同 Java `String.format`
- 元组解包:`a, b = (1, 2)` / `for k, v in dict.items():`
- `if __name__ == "__main__":` Python 入口点惯用法

**LLM API 调用 4 个核心概念**
1. **无状态**:LLM 不记得历史,每次发送完整 messages 数组
2. **messages 结构**:`[{"role": "system|user|assistant", "content": "..."}]`
3. **HTTP 本质**:SDK 底层就是 HTTP POST,无神秘
4. **OpenAI 兼容**:同一个 SDK 调 DeepSeek/Gemini,只改 `base_url`

**重试与可靠性**(`tenacity` 库)
- `@retry(stop=..., wait=..., retry=..., before_sleep=...)` 装饰器组合
- 异常类型分流:5xx 指数退避,429 看 `RetryInfo`,4xx 中的 401/400 不重试
- jitter(随机抖动)防惊群效应

**HTTP 状态码工程语义**

| 状态码 | 含义 | 处理 |
|---|---|---|
| 401 | Unauthorized | 不重试,改配置 |
| 400 | Bad Request | 不重试,改请求 |
| 404 | Not Found | 不重试 |
| 429 | Too Many Requests | 重试,看 `Retry-After` 或 `RetryInfo.retryDelay` |
| 5xx | 服务端问题 | 重试,指数退避 + jitter |

### 工程经验

- **不要让 AI 自动修报错**:自己读 stack trace,自己判断错误类型
- **API 限额事故应对**:Gemini 免费层每日 20 次撞墙后切 DeepSeek,用同一份 OpenAI SDK 切换
- **Prompt 工程 5 条规律**:见 [appendix/prompt-engineering-rules.md](./appendix/prompt-engineering-rules.md)

---

## 🔬 关键实验记录:5 个 Prompt × 3 轮采样

### 实验设计

| 实验 | Prompt | 关键观察 |
|---|---|---|
| 1 模糊指令 | `翻译:你好` | DeepSeek 三轮:Hello / 你好 / Hello,LLM 把"你好"翻成"你好" |
| 2 加约束 | 翻成英文,不要引号或解释 | DeepSeek 第 2 轮出现 `Hello.`(带句号),约束被部分忽略 |
| 3 角色设定 | 你是专业翻译,直接返回结果 | 三轮全部 `Hello`,稳定 |
| 4 few-shot | 给 2 个示例(`谢谢→Thanks`) | 严格对齐示例风格,无标点无引号 |
| 5 JSON 输出 | 要求纯 JSON,不要其他文字 | DeepSeek:纯 JSON 三轮一致;Gemini:被 markdown ```json``` 包裹 |

### 跨模型对比

| 维度 | Gemini 2.5 Flash | DeepSeek V4 Flash |
|---|---|---|
| 模糊指令的失败方式 | "爱秀"——给 markdown 列表+解释 | "犯傻"——可能不翻译,原样返回 |
| JSON 稳定性 | 被 markdown ```json``` 包裹 | 纯 JSON,3 轮完全一致 |
| 免费层 | 20 RPD,撞限快 | 付费但便宜,实际不撞限 |

**这是判断**:不要因此得出"DeepSeek > Gemini"。一个 prompt × 3 轮不构成模型选型,真正评估需要 RAGAS 等框架(阶段三学)。

📖 完整规律总结见 → [appendix/prompt-engineering-rules.md](./appendix/prompt-engineering-rules.md)

---

## 🎓 完成标志 Q&A 复盘

📖 完整 Q&A 已归档到 → [appendix/checkpoint-qa.md](./appendix/checkpoint-qa.md#阶段一)

**5 题概览**:
- Q1 概念检查:LLM 无状态的含义 → ✅ 完整且准确
- Q2 代码检查:`response.choices[0].message.content` 的设计 → ⚠️ 答对了但太简短
- Q3 工程检查:`json.loads` 可能崩溃的方式 → ⚠️ 2 对 1 错(凭印象猜了"多个逗号")
- Q4 工具检查:无脑重试的工程问题 → ✅ 三条都对(包括 jitter)
- Q5 Java 惯性检查:LLM vs Java 的认知差异 → ✅ 抓到核心,缺具体场景

**最重要的反思**:Q3 的"多了个逗号"是凭印象猜的,不在今天观察到的数据里。
**规则提醒**:判断必须基于事实,不能凭印象——这条对学习者和 AI 都成立。

---

## 🐛 踩坑记录

### 踩坑 1:Gemini SDK 用错版本

**现象**:跑第一段代码出现 `FutureWarning: All support for the google.generativeai package has ended`。

**原因**:Google 把旧 SDK `google-generativeai` 弃用了,推荐用新的 `google-genai`(差一个字)。

**教训**:在 LLM 生态变化快的情况下,装包前应该先 web_search 确认当前推荐版本,不能凭旧记忆。

**解决**:`pip uninstall google-generativeai google-ai-generativelanguage -y` + `pip install google-genai`。

---

### 踩坑 2:HTTP 503 误判为限速

**现象**:Gemini 调用返回 `503 UNAVAILABLE`。

**第一反应**:"我被限速了"。

**正确判断**:503 是服务端过载(Google 的问题),429 才是限速(我的问题)。错误信息里写得很清楚:`'This model is currently experiencing high demand'`。

**教训**:Java 工程师惯性——调用失败就归因为"自己请求过多"。HTTP 状态码必须仔细看。

---

### 踩坑 3:Gemini 免费层每日 20 次配额事故

**现象**:三轮实验跑了 2 轮多,第 3 轮第一个调用直接 429 RESOURCE_EXHAUSTED。

**误判过程**:
1. 第 1 次以为是 RPM(每分钟 5 次)
2. 实际是 RPD(每日 20 次)
3. 错误响应里 `quotaId: GenerateRequestsPerDayPerProjectPerModel-FreeTier` 才是真相

**事实**:<br>
2025 年 12 月 7 日 Google 大幅下调免费层配额,Gemini 2.5 Flash 从 ~250 RPD 降到 20 RPD(降幅 92%)。

**教训**:
1. 涉及外部服务的"配额/限速/版本"等事实,**永远以错误响应里的实际值为准**,不能凭印象给"大概 10 RPM"
2. 学习项目应该选**有付费选项的 API**(DeepSeek 5 元能用很久,无 RPD 限制)

**最终方案**:切到 DeepSeek + 用 OpenAI SDK + 保留 Gemini key 备用。

---

### 踩坑 4:`where` 命令在 PowerShell 里失效

**现象**:在 PowerShell 跑 `where python` 没有任何输出。

**原因**:`where` 是 cmd.exe 命令;PowerShell 里 `where` 是 `Where-Object` 别名,语义完全不同,所以静默失败。

**解决**:PowerShell 用 `Get-Command python` 或 `python -c "import sys; print(sys.executable)"`。

**教训**:Shell 差异是真实的,不能假设命令通用。

---

## 🧠 Java 工程师惯性记录

阶段一暴露出来的 Java 思维和 LLM 现实的冲突点,标记下来后面继续观察:

| 惯性 | 现实 | 危险等级 |
|---|---|---|
| "调用失败 = 我请求太多" | 看状态码,5xx 是服务端的问题 | 中 |
| "类型注解就是强类型" | Python 类型注解运行时不强制 | 中 |
| "Maven 算依赖" | pip 倒序试版本,可能卡很久 | 低 |
| "LLM 输出可以 assertEquals" | LLM 是概率分布,同输入不同输出 | **高** |
| "约束就是规则" | LLM 的"约束"是降低概率,不是杜绝 | **高** |
| "找一个最优解" | LLM 选型没有标准答案,要看场景 | 中 |

**最危险的两条**:assertEquals 思维 + 约束 = 规则。这两条在 CVE 监控 Agent 上会带来误报/漏报/程序崩溃风险。

---

## 🔗 相关文件

- 总进度看板:[../README.md](ai-agent-roadmap/README.md)
- 原始路线图:[00-roadmap.md](00-roadmap.md)
- Prompt 工程规律:[appendix/prompt-engineering-rules.md](./appendix/prompt-engineering-rules.md)
- 完成标志 Q&A:[appendix/checkpoint-qa.md](./appendix/checkpoint-qa.md)

---

## 🚀 下一步

进入 [阶段二 LLM 基础 + Prompt 工程](./02-llm-prompt.md)(待开始)。

**起点**:
- 已经能调用 DeepSeek 和 Gemini(OpenAI 兼容接口)
- 已经初步体感 Prompt 影响输出
- 阶段二要把"prompt 工程"从体感升级为系统方法

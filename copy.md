---
latestModelInfo:
  model: gpt-6.2-astra
  migrationGuide: /api/docs/guides/latest-model/gpt-6.2-astra.md#migration-quickstart
  promptingGuide: /api/docs/guides/latest-model/gpt-6.2-astra.md#prompting-best-practices
---

# 使用 GPT-6.2

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

<a id="gpt-6.2-astra-gpt-62-astra-and-gpt-62-sol"></a>

要求最高的任务可使用 [GPT-6.2 Astra](https://developers.openai.com/zh-Hans/api/docs/models/gpt-6.2-astra)，希望以较低成本完成复杂任务时可使用 [GPT-6.2 Sol](https://developers.openai.com/zh-Hans/api/docs/models/gpt-6.2-sol)。

## 新变化

- **前缀编辑**。从 GPT-6.2 开始，如果在对话过程中修改了对话历史（即编辑前缀）、对模型作了某些变更（包括降级），或更换了最终用户或组织，Responses API 就会丢弃推理项。模型无法访问这些推理项时，回答质量可能下降，也可能消耗更多 Token 来重复之前的工作。请检查您的集成，找出此类编辑，例如对先前的开发者消息或工具调用所作的修改。要保留推理项，并让之前的输入仍可使用[提示词缓存](https://developers.openai.com/zh-Hans/api/docs/guides/prompt-caching)，请在两次响应之间更改设置时[保持前缀不变](https://developers.openai.com/zh-Hans/api/docs/guides/keep-prefix-unchanged)。
- **Responses API 中 `instructions` 字段的变化**。从 GPT-6.2 开始，模型在处理 Responses API 请求时，会将顶层 `instructions` 追加到 `input` 之后。要让可复用的指令符合提示词缓存条件，请将这些指令写入一条 `developer` 消息，并将该消息作为 `input` 的第一项。仅适用于当前请求的开发者指令可放在顶层 `instructions` 中。请参阅[前缀匹配的工作原理](https://developers.openai.com/zh-Hans/api/docs/guides/prompt-caching#how-prefix-matching-works)。
- **后台模式（公开测试版）**。对于耗时较长的 Responses API 请求，可以在后台模式下使用[享受折扣的完成时限](https://developers.openai.com/zh-Hans/api/docs/guides/background#use-discounted-completion-windows)，作为 Flex 处理的替代方案。使用 Standard 处理时，后台模式请求的完成时限可选 2 小时或 8 小时。GPT-6.2 Astra 和 Sol 均不支持 Flex 处理（`service_tier: flex`）。

<a id="prompting-best-practices" className="scroll-mt-[110px]"></a>

## 提示词最佳实践

以下提示词可作为 GPT-6.2 系列模型的使用起点。示例针对我们在使用 GPT-6.2 Astra 和 Sol 时观察到的行为。如果您的应用中也出现了其中一种行为，可以试试对应小节的提示词。

### GPT-6.2 Astra 的行为

- **[写作风格](#gpt-6.2-astra-writing-style)。** 模型默认写得更简洁、更直接。迁移时，请重新审视系统提示词中类似“保持简洁”的指令。
- **[任务完成](#gpt-6.2-astra-task-completion)。** 用户意图不明确时，模型可能会提出下一步建议，然后等待确认。对于耗时较长的任务，可允许模型作出合理假设并继续工作。
- **[编码风格](#gpt-6.2-astra-coding-style)。** 与早期模型相比，该模型通常能编写效率更高、质量更好的代码。请说明编写或修改代码时应遵循的项目约定。
- **[前端设计](#gpt-6.2-astra-frontend-design)。** 如果界面需要特定的视觉风格，请指定要使用的图像和排版样式。

<a id="gpt-6.2-astra-prompt-patterns-for-api-applications"></a>
<a id="gpt-6.2-astra-personality-and-writing-style"></a>

### 写作风格

默认情况下，GPT-6.2 Astra 的表达比早期模型更简洁、更直接。您可以根据应用需要调整语气，让它像专业同事一样交流，也可以让它像助手一样以更口语化的方式对话。

如果您希望模型在日常工作中与您协作、一起思考，可以使用以下提示词。

```text
使用用户和源材料中的术语。保留技术细节，并在专业用语能让说明更准确时使用它们。使用主动语态，把紧密相关的细节放在一起。先说要点，避免使用读者要读到后面才能理解的指代。提供足够的背景，让用户能跟上你的推理。
```

编写文档、邮件、消息或对外发布的内容时，可提示模型根据材料的目标读者调整写法。

```text
代表用户起草面向外部读者的消息或其他材料时，考虑谁会阅读这些内容，并根据目标读者调整写法。目标读者掌握的背景信息可能与用户不同，因此请补充读者理解内容所需的背景或推理过程。审阅或编辑现有内容时，保留原作者的内容和措辞，仔细按照用户的要求进行修改。
```

在耗时较长的任务中，模型可能会把自己推理时或其他智能体使用的简写带入最终答复。可以提醒模型，让最终答复本身信息完整，即使读者没有看过进度更新也能读懂。

```text
把最终答复当作用户第一次看到你的工作成果来写，避免把它写成中途说明的延续。最终答复必须信息完整。向用户显示最终答复时，先前的进度更新会折叠起来，因此用户应该无需查看这些更新就能理解答复。

面向目标读者重新撰写最终答复和所有供人阅读的材料。不要照搬或近似改写你的推理过程，也不要沿用推理时的写法。不要把推理、工具调用或发给子智能体和自动化的指令中使用的临时简写、行话或缩写带入答复。
```

<a id="gpt-6.2-astra-initiative-and-follow-through"></a>
<a id="gpt-6.2-astra-subagent-delegation"></a>

### 任务完成

对于可以自主推进的工作，请明确模型需要交付什么结果，以及哪些决定需要用户参与。

```text
只要任务尚未完成，或还有已获授权的下一步可执行，就避免停下来结束本轮对话。继续在任务范围内采取下一步行动，推进用户的请求。如果你准备在最终答复中建议下一步、给出计划、请用户作出常规决定、说明如何执行任务，或以类似方式把工作交给用户，请亲自执行这些步骤，推进用户要求完成的工作。确保你有权执行这些操作，并告知用户你作出的任何假设或决定。

仅在任务范围有重大变化、遇到确切的阻碍、有决定必须由用户作出，或即将执行不可逆或破坏性操作时，才停下来向用户确认。
```

<a id="gpt-6.2-astra-testing-and-verification"></a>
<a id="gpt-6.2-astra-ask-for-an-exhaustive-pass"></a>

请模型核实用户要求的工作是否已经完成。对于复杂或耗时较长的任务，提高推理强度可以帮助模型完成最后的各项检查。在 Responses API 中，通过 `reasoning.effort` 设置推理强度。[推理强度](https://developers.openai.com/zh-Hans/api/docs/guides/reasoning#reasoning-effort)

```text
优先采取行动，持续推进，直到完成用户要求的任务。只要任务尚未完成，或还有已获授权的下一步可执行，就避免停下来结束本轮对话。继续在任务范围内采取下一步行动，推进用户的请求。确保覆盖用户请求的每一项要求，直到交付完全满足用户目标的结果。如果任务需要持续投入，就完成所有必要工作，直到达到预期结果。

自主推进用户的目标，例如按需创建隔离的工作树或检出副本、解决合并冲突、创建草稿 Pull Request 等。明显具有破坏性或不可逆的操作除外。结束本轮对话或宣布工作完成前，检查成果是否完全满足用户的要求。如果还有步骤未完成，或有任何结果尚待验证，请在结束前跟进到底。
```

<a id="gpt-6.2-astra-instruction-following"></a>
<a id="gpt-6.2-astra-intent-understanding"></a>
<a id="gpt-6.2-astra-interpreting-ambiguous-requests"></a>

如果某个细节不明确，可允许模型说明自己的假设并继续执行任务。

```text
如果作出合理且可撤回的假设能在任务范围内推进工作，就作出这些假设并说明。除非确实无法继续，否则不要反复质疑自己的判断，不要只说明可以做什么、提出可以代劳，或只给出部分答复。
```

如果应用提供 `request_user_input_async` 等异步工具，GPT-6.2 就可以向用户询问不明确的细节，同时继续推进无需等待答复的工作。[添加异步提问工具](https://developers.openai.com/zh-Hans/api/docs/guides/async-tool-calling#add-a-tool-to-ask-the-user-for-input)

<a id="gpt-6.2-astra-code-changes"></a>

### 编码风格

说明代码库的约定，以及现有调用方依赖的行为。例如，可以指定何时复用现有逻辑、将改动控制在什么范围内，以及何时为现有测试套件未覆盖的行为添加测试。

```text
首要目标是编写正确、高质量的代码，完成用户要求的全部工作。让代码易于阅读，并验证其行为是否符合用户的预期。编写代码时请遵循以下指令：

(a) 尽可能复用用户代码仓库中的现有逻辑。只要能保证正确性，就应复用代码库中现有的类、方法、数据表示和 API。合并或移除代码时，保留调用方依赖的行为，除非任务要求改变这些行为。
(b) 只有在用户要求、代码仓库有规定，或需要捕捉现有测试会漏掉的特定回归问题时，才添加新测试。
(c) 确保代码便于人阅读。不要为了显得简单就减少行数、缩短变量名、压缩格式，或把逻辑挤在一行里。注释和变量名应清楚易懂。为了缩短代码而写的表达式，如果塞进了大量分支、条件或隐含假设，就属于过度复杂。
(d) 避免不必要的间接层和抽象。辅助函数、实用函数和类抽象即使没有增加分支，也可能让代码更难理解。
```

<a id="gpt-6.2-astra-visual-review"></a>
<a id="gpt-6.2-astra-visual-work-and-frontend-design"></a>

### 前端设计

如果应用需要特定的视觉风格，请在提示词中加以描述。说明应使用哪些类型的图像，以及应采用怎样的排版。字体很重要时，请指定具体字体。界面文案应聚焦于用户需要了解的信息和需要执行的操作。

```text
生成网页或带有前端界面的应用时，请遵循以下指令：

- 选择视觉素材时，通常优先使用通过 ImageGen 生成或通过图像搜索找到的写实图像，并确保它们符合项目的风格和用途。用户明确要求 SVG 时，请使用 SVG。
- 仅在文字能提供用户所需的信息、帮助用户导航或操作，或满足明确的内容要求时，才添加文字元素。移除装饰性编号、凑数的徽章、重复的标题，以及仅复述界面已清楚传达的信息的说明文案，同时保留必要的标签、操作说明和状态反馈。
```

处理视觉设计任务时，可分享设计软件中的参考素材，也可以让 GPT-6.2 使用[图像生成工具](https://developers.openai.com/zh-Hans/api/docs/guides/tools-image-generation)，并要求它用 GPT Image 创建参考图。参考图能为提示词补充难以用文字描述的细节。参考图达到预期的视觉效果后，请模型制作素材、进行渲染，并对照参考图进一步调整。

## 迁移快速入门

<a id="gpt-6.2-astra-migrate-with-codex"></a>

如需借助编码智能体将 API 应用迁移到 GPT-6.2 Astra，请使用 [OpenAI Docs 技能](https://github.com/openai/codex/tree/main/codex-rs/skills/src/assets/samples/openai-docs)。

<a id="gpt-6.2-astra-choose-a-reasoning-effort"></a>
<a id="gpt-6.2-astra-zdr-data-residency-and-service-tiers"></a>
<a id="gpt-6.2-astra-limitations"></a>
<a id="gpt-6.2-astra-data-residency-and-service-tiers"></a>

### 更新 API 和模型参数

将 `model` 设置为 `gpt-6.2-astra` 或 `gpt-6.2-sol`，然后检查以下设置：

- **推理强度：** 新集成可从 `medium` 开始。如果当前使用 `none` 或 `minimal`，请改用 `low`。在 Responses API 中使用 `reasoning.effort`，在 Chat Completions API 中使用 `reasoning_effort`。
- **工具调用：** 使用 [**Responses API**](https://developers.openai.com/zh-Hans/api/docs/guides/migrate-to-responses#migrating-from-chat-completions)。GPT-6.2 Astra 和 GPT-6.2 Sol 均支持 Chat Completions API，但调用工具必须使用 Responses API。
- **不受支持的参数：** 迁移 Chat Completions API 请求时，省略 `temperature`、`top_p`、`logprobs` 和 `top_logprobs`。对于 Responses API 请求，省略 `temperature`、`top_p` 和 `top_logprobs`，并移除 `message.output_text.logprobs`（如果 `include` 中包含此项）。
- **提示词缓存：** 将内容固定的指令从顶层 `instructions` 参数移入一条 [`developer` 消息，并将该消息放在 `input` 的开头](https://developers.openai.com/zh-Hans/api/docs/guides/prompt-caching#how-prefix-matching-works)，使这些指令符合缓存条件。从 GPT-5.5 或更早版本迁移时，将 `prompt_cache_retention` 替换为 `prompt_cache_options.ttl`，并将其设为 `"30m"`。请查看[**提示词缓存的变化**](https://developers.openai.com/zh-Hans/api/docs/guides/prompt-caching#summary-of-model-differences)，其中包括缓存边界和缓存写入计费规则。
- **零数据保留：** 对于获批的组织，GPT-6.2 Astra 和 Sol 在符合条件的端点和功能上支持零数据保留（ZDR）。启用 ZDR 后，Responses API 会将 `store` 视为 `false`。处理多轮任务时，请按原顺序传回之前的输出项，包括加密的推理项。有关适用条件和端点行为，请参阅[零数据保留](https://developers.openai.com/zh-Hans/api/docs/guides/your-data#zero-data-retention)。
- **服务层级和区域：** 使用 Standard、Fast 或 Batch 处理时，这两款模型均支持美国和欧盟的数据驻留。Ultrafast 仅支持美国的数据驻留。请参阅[数据驻留](https://developers.openai.com/zh-Hans/api/docs/guides/data-residency#which-models-and-features-are-eligible-for-data-residency)和[定价](https://developers.openai.com/zh-Hans/api/docs/pricing#text-tokens)。
- **编写提示词：** 请参阅[**提示词最佳实践**](#prompting-best-practices)。
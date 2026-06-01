---
layout: post
title: "Prompt 的正确写法：专业人士使用的四层结构"
date: 2026-05-22 20:54:03 +0800
categories: [AI, Prompt Engineering]
tags: [prompt-engineering, anthropic, claude, ai, LLM, best-practices]
image:
  path: /img/4layer-prompt-structure.jpg
  alt: "The 4-Layer Prompt Structure"
---

> **原文作者：** [@0xCodez](https://x.com/0xCodez) — Codez
> **原文链接：** [How to write a prompt the right way: the 4-layer structure pros use](https://x.com/0xCodez/status/2057807200173613450)
> **译者：** 本文为中英双语对照翻译

---

我翻阅了 Anthropic 的官方 prompt engineering 文档、他们的内部课程，以及一位应用 AI 工程师的演讲——讲的是 Claude 团队如何在生产环境中调试 prompt。

I went through Anthropic's official prompt engineering docs, their internal courses, and a talk by one of their applied AI engineers on how the Claude team actually debugs prompts in production.

---

**收藏这条。存好它。** 然后我把所有内容压缩成了一个结构。四层。按顺序搭建，你就能得到能扛住模型迁移、边缘情况、生产流量的 prompt。

**Bookmark this. Save it.** Then I compressed everything into one structure. Four layers. Build them in order and you get prompts that survive model migrations, edge cases, and production traffic.

> Follow my Substack to get fresh AI alpha: [movez.substack.com](https://movez.substack.com)

这不是夸张。读了这篇文章之后，你再也不会把 prompt 写成一句话了——因为你会明白，那才是问题的根源。

That is not hyperbole. After this you will never write a prompt as a single sentence again - because you will see why that was the problem the whole time.

---

## 人人都相信的谎言

**The lie everyone believes about prompting.**

大多数人把写 prompt 当成谷歌搜索。

Most people treat prompting like a Google search.

打一句含糊的话。回车。碰运气。如果输出不好，多加点词。还是不好？加「IMPORTANT」。加「CRITICAL」。加「always」和「never」直到有什么东西凑效。

Type a vague sentence. Hit enter. Hope. If the output is bad, add more words. Still bad? Add "IMPORTANT." Add "CRITICAL." Add "always" and "never" until something sticks.

Anthropic 自己的文档直接指出了这一点：大多数人「打一句含糊的话，回车，然后祈祷。接着他们奇怪为什么输出听起来很空洞，或者完全不着边际。」

Anthropic's own documentation calls this out directly: most people "type a vague sentence, hit enter, and hope for the best. Then they wonder why the output sounds generic or misses the mark entirely."

**没人告诉你的真相：一个 prompt 不是一个句子。它是一个有四层结构的系统，每一层各司其职。**

**Here is the truth nobody tells you. A prompt is not a sentence. It is a system with four layers, and each layer does a different job.**

业余的人写完第二层就停了。专业的人按顺序搭建全部四层。

Amateurs write Layer 2 and stop. Pros build all four, in order.

---

## 01. 让 prompt 可解析

**Make the prompt parseable.**

在优化任何东西之前，先组织好。

Before you optimize anything, you organize.

Anthropic 的第一条结构性建议是 **XML 标签**。不是因为好看——而是因为 Claude 是经过专门训练来识别 XML 结构的。当 prompt 有多个组件时，标签能防止模型把它们混在一起。

Anthropic's number one structural recommendation is XML tags. Not because they look clean - because Claude was specifically trained to recognize XML structure. When a prompt has multiple components, tags stop the model from mixing them up.

Anthropic 团队有一条直击核心的经验法则：**如果你在读一个 prompt 时，分不清哪些是指南、哪些是规则、哪些是数据，那模型大概也分不清。**

The Anthropic team has a rule of thumb that cuts to the core of it: if you are reading a prompt and you cannot tell guidelines from policy from data, the model probably cannot either.

> **专业人士怎么打标签**
>
> **What the pros actually tag**

没有「正确」的标签名——Anthropic 明确说了这一点。但常见模式值得记住：

There are no "correct" tag names - Anthropic is explicit about this. But the common patterns are worth memorizing:

> **顺序比你想象的更重要：**
>
> **The order matters more than you think:**

一个几乎没人知道的细节，直接来自 Anthropic 的长上下文指南：**把长文档放在 prompt 的顶部，在指令和问题之前。放在结尾的查询可以让长上下文任务的质量提升高达 30%。**

One detail almost nobody knows, straight from Anthropic's long-context guidance: put long documents at the top of your prompt, above your instructions and your question. Queries placed at the end can improve response quality by up to 30% on long-context tasks.

同样的 prompt，同样的模型——只是把问题移到最下面就能换来高达 30% 的提升。这是 prompt engineering 中最便宜的性能提升，而且不花你一分钱。

Same prompt, same model - just moving the question to the bottom buys you up to 30%. That is the cheapest performance gain in prompting, and it costs you nothing.

> **清理环节：**
>
> **The cleanup pass:**

Anthropic 的工程师做了一件事，大多数人从来不做：**删东西。**

Anthropic's engineers do something most people never do: they delete.

生产环境下的 prompt 会累积补丁。一条为了修复某个模型怪癖加的指令。一条在出过一次坏结果后硬加上去的「CRITICAL: never do X」。有人从网页上粘贴进来的一段内容，附带英雄图和 cookie 声明。每条都是模型需要费力穿过的噪音。删掉所有不再值得占位置的东西。

Production prompts accumulate patches. An instruction added to fix one model's quirk. A "CRITICAL: never do X" bolted on after one bad output. A block someone pasted from a website, hero-image reference and cookie notice still attached. Every one of those is now noise the model has to wade through. Strip out everything that no longer earns its place.

---

## 02. 让模型瞄准目标

**Aim the model at the target.**

现在模型能读懂 prompt 了，你告诉它要去哪里。这一层是大多数人以为的「写 prompt」。它只是四层之一。方向引导有四个官方 Anthropic 技巧——按这个顺序使用。

Now that the model can read the prompt, you tell it where to go. This is the layer most people think is prompting. It is one of four. Direction has four official Anthropic techniques - use them in this order.

1. **给 Claude 一个角色**
   **Give Claude a role**

   Anthropic 称角色提示为「使用 Claude system prompt 最强大的方式」。官方指南很明确：**把角色放在 system prompt 里，把任务指令放在 user turn 里。角色在上，任务在下。**

   Anthropic calls role prompting "the most powerful way to use system prompts with Claude." The official guidance is specific: put the role in the system prompt, put task instructions in the user turn. Role up top, task below.

2. **清楚直接**
   **Be clear and direct**

   Anthropic 的研究发现，当 prompt 明确说出任务名称、明确输出是给谁看的、并定义好「完成」的标准时，Claude 表现最佳。检验标准：一个没有上下文的新员工能照着你的 prompt 执行吗？如果不能，问题在 prompt，不在模型。

   Anthropic's research found Claude performs best when the prompt names the task explicitly, identifies who the output is for, and defines what "done" looks like before generation begins. The test: could a new employee follow your prompt with zero context? If not, the fault is the prompt, not the model.

3. **提供示例（multishot）**
   **Use examples (multishot)**

   最高杠杆的技术，大多数人却跳过了。Anthropic 建议：**一到三个输入和期望输出的例子**，用 `<examples>` 标签包起来。一个你想要的具体输出示例，胜过五十行描述它的形容词。

   The highest-leverage technique most people skip. Anthropic's recommendation: one to three examples of input and desired output, wrapped in `<examples>` tags. A single concrete example of the output you want is worth more than fifty lines of adjectives describing it.

4. **让 Claude 思考**
   **Let Claude think**

   对于分析、数学或多步骤逻辑，告诉模型在回答之前先推理。Anthropic 的一条铁律：**永远让 Claude 输出它的思考过程。** 放在你看不见的隐藏 scratchpad 里的推理，跟没发生过一样。

   For analysis, math, or multi-step logic, tell the model to reason before it answers. Anthropic's one firm rule: always have Claude output its thinking. Reasoning in a hidden scratchpad you cannot see may as well not have happened.

用 XML 标签干净地分隔角色、示例和显式推理——这就是 Anthropic 所说的「超级结构化、高性能的 prompt」。第一层和第二层协同工作。

XML tags holding role, examples, and explicit reasoning, all separated cleanly - that is what Anthropic calls a "super-structured, high-performance prompt." Layer 1 and Layer 2 working together.

---

## 03. 给它真正做事的能力

**Give it the ability to actually do it.**

这是 Anthropic 工程演讲中最反直觉的一课——是区分真正理解模型的人和不理解的人的界限。

Here is the most counterintuitive lesson from the Anthropic engineering talk - the one that separates people who understand models from people who do not.

**指令不能增加能力。**

**Instructions do not add capability.**

演讲中有一个客服机器人，对客户的账单计算总是给出模糊的回复。原始 prompt 对它咆哮：「CRITICAL. Always calculate any prorated amounts correctly. Never give a vague answer.」

In the talk, a support bot kept giving customers vague answers to billing calculations. The original prompt screamed at it: "CRITICAL. Always calculate any prorated amounts correctly. Never give a vague answer."

没用。永远不可能有用。告诉一个模型「用心算算对很重要」不会让它更擅长心算。这条指令瞄准的是指令本身无法解决的问题。

It did not work. It could never work. Telling a model it is critical to do mental math correctly does not make it better at mental math. The instruction was aimed at a problem instructions cannot solve.

> **解决方案是一个工具，不是一个句子**
>
> **The fix is a tool, not a sentence**

他们**给模型加了一个计算器工具**。定义这个工具，告诉模型什么时候用它，把数学逻辑实现在工具里。结果：所有测试用例都通过了。模型在理解问题后，用工具可靠地执行了计算。

Instead of louder instructions, they gave the model a calculator tool. Defined it, told the model when to use it, implemented the math behind it. Result: every test case passed. The model reasoned about the problem and used the tool to execute it reliably.

这是思维上的转变。当 prompt 失败时，问你自己：面对的是**方向问题**还是**能力问题**：

This is the mental shift. When a prompt fails, ask whether you face a direction problem or a capability problem:

- **方向问题 → 在第二层解决**（更清晰的角色、更好的示例、显式推理）
  **Direction problem → fix in Layer 2** (clearer role, better examples, explicit reasoning)
- **能力问题 → 在第三层解决**（给它一个工具，或更多推理预算）
  **Capability problem → fix in Layer 3** (give it a tool, or more reasoning budget)

任何数量的第二层文字修饰，都填补不了第三层的缺口。人们花几小时重写 prompt 词句，而真正的问题是模型一直都需要一个工具。

No amount of Layer 2 wordsmithing fixes a Layer 3 gap. People burn hours rewording prompts when the real issue was that the model needed a tool all along.

> **推理预算也是能力**
>
> **Reasoning budget is also capability**

同一个演讲展示了一个调度 agent 在硬约束问题上失败。换更大的模型有帮助。打开**自适应思考**——让模型自己决定思考多少——让它的答案变得可靠。

The same talk showed a scheduling agent failing on a hard constraint problem. A bigger model helped. Turning on adaptive thinking - letting the model decide how much to reason - made it reliably correct.

能力是通过 API 和 harness 加上去的，不是通过 prompt 文本。杠杆不一定总是文字。

That is capability you add through the API and the harness, not through the prompt text. The lever is not always the words.

---

## 04. 证明它有效。让它持续有效。

**Prove it works. Keep it working.**

这一层把 prompt engineering 从猜谜变成了工程。也是几乎没人搭建的一层。

This is the layer that turns prompting from guessing into engineering. And it is the one almost nobody builds.

Anthropic 的文档很直白：在优化之前，你**需要明确的成功定义、测试它的方法、以及一个初版 prompt 来优化**。「没有这些，你就是在盲目优化。」

Anthropic's documentation is blunt: before you optimize, you need a clear definition of success, a way to test against it, and a first-draft prompt to improve. "Without these, you're optimizing blind."

> **搭建一个小型 eval 套件**
>
> **Build a tiny eval suite**

你不需要一个研究实验室。Anthropic 的演讲只用了五个测试用例。重点是覆盖率，不是数量。三种类型的用例很重要：

You do not need a research lab. The Anthropic talk used five test cases. The point is coverage, not volume. Three kinds of cases matter:

- **对照用例**。明确的，应该永远通过的。你的金丝雀——如果这个失败了，说明出大问题了。
  **Control cases.** Unambiguous, should always pass. Your canary - if this breaks, something is badly wrong.

- **边缘用例**。模型之前失败过的地方。每一条都是一个回归测试，阻止那个失败偷偷回来。
  **Edge cases.** Places the model failed before. Each one is a regression test that stops the failure from sneaking back.

- **边界用例**。模型应该交给人类处理或拒绝的地方。验证它知道自己能力的边界。
  **Boundary cases.** Where the model should hand off to a human or refuse. Proof that it knows the limits of its own competence.

把 prompt 对着所有用例跑一遍。改一个东西。再跑一次。现在你知道你的改动到底有没有帮助——而不是因为你检查的那个例子看起来不错就以为它有效。

Run the prompt against all of them. Change one thing. Run again. Now you know whether your change helped - instead of believing it did because the one example you checked looked fine.

> **说出每一笔取舍的两面**
>
> **State both sides of every trade-off**

手册中最微妙的一课，而且随着模型变聪明越来越重要。演讲中，一个机器人拒绝升级处理账单错误。

The subtlest lesson in the playbook, and it matters more as models get smarter. In the talk, a bot refused to escalate billing errors.

Prompt 里写的是：「除非绝对必要，避免升级——每次升级花费约 $8，还会影响我们的指标。」所以模型优化了**不升级**。它完美执行了指令。

The prompt had said: "avoid escalating unless absolutely necessary - it costs about $8 and counts against our metrics." So the model optimized for not escalating. It followed orders perfectly.

prompt 只说了一面。它说了升级的成本，从没说**不升级的成本**——一个错误的答案、一次退款、失去信任。

The prompt only gave one side. It named the cost of escalating and never the cost of not escalating - a wrong answer, a refund, lost trust.

修复方法：**给出两面。** 随着模型推理能力越来越强，它们会为唯一实际给定的目标进行更强地优化。一条在弱模型上工作的片面指令，到了强模型上就成了陷阱。

The fix: give both sides. As models get better at reasoning, they optimize harder for the only goal you actually gave them. A one-sided instruction that worked on a weaker model becomes a trap on a stronger one.

> **旧补丁变成毒药**
>
> **Old patches become poison**

演讲中最令人惊讶的失败：一个机器人明明有信息却扣着不给——告诉客户「去网站查」而不是直接回答，尽管答案就在账户数据里。

The most surprising failure of the talk: a bot was withholding information it actually had - telling a customer "go check the website" instead of answering, even though the answer was right there in the account data.

原因？一个旧补丁。以前的模型会幻觉出计划细节，所以有人加了「永远不要给出错误的计划细节，指引他们去 URL」。那条补丁在当时是有意义的。但新模型更严格地遵循指令——所以同一条指令现在让模型闭嘴了，把正确的信息也藏起来了。

The cause? An old patch. A previous model used to hallucinate plan details, so someone added "never give wrong plan details, point them to the URL." That patch made sense once. But newer models follow instructions more literally - so the same line now caused the model to clam up and hide correct information.

修复方法是流程，不是措辞：**对你的 prompt 做版本控制。** 记录每一条防御性指令为什么添加。在下次模型迁移时，找到那些补丁并问它们是否还值得存在——还是已经悄悄变成了毒药。

The fix is process, not wording: version-control your prompts. Record why every defensive instruction was added. On the next model migration, find those patches and ask whether they still earn their place - or whether they have quietly turned toxic.

---

## 05. 四层如何叠加

**How the 4-layers stack.**

用一个真实的失败案例走一遍这个结构，方法就显而易见了。你的 prompt 输出不好。与其随机加词，你按顺序问自己：

Walk a real failure through the stack and the method becomes obvious. Your prompt is producing bad output. Instead of randomly adding words, you ask, in order:

大多数人永远活在第二层，不停地重写同一句话。专业人士按顺序走完四层——**顺序就是全部技巧**。结构先于方向。方向先于能力。能力先于你信任它。验证贯穿一切。

Most people live entirely in Layer 2, rewording the same sentence forever. The professionals move through all four in order - and the order is the whole trick. Structure before direction. Direction before capability. Capability before you trust it. Verification underneath all of it.

**今天就应该删掉的反模式：**

**The anti-patterns to delete today:**

- ❌ 别再堆砌「CRITICAL」和「IMPORTANT」。强烈的措辞不会增加能力。模型需要的是一个工具或更多推理，不是更大的声量。
  Stop stacking "CRITICAL" and "IMPORTANT." Forceful language does not add capability. The model needs a tool or more reasoning, not louder instructions.

- ❌ 别再写片面指令。每个「避免 X」如果不说出避免 X 的代价，就是在教模型过度拟合。
  Stop one-sided instructions. Every "avoid X" without the cost of avoiding X teaches the model to overfit. State both sides.

- ❌ 别再囤积旧补丁。为去年模型写的指令可能会毒害今年的模型。
  Stop hoarding old patches. Instructions written for last year's model can poison this year's. Audit them on every migration.

- ❌ 别再写文字墙。如果你分不清规则、指南和数据，模型也分不清。
  Stop the wall of text. If you cannot tell policy from guidelines from data, neither can the model. Tag everything.

- ❌ 别再在长输入里把问题放前面。文档在上，问题在底。提升高达 30%。
  Stop putting the question first on long inputs. Documents at top, question at bottom. Up to 30% better.

- ❌ 别再盲目优化。没有 eval 就是猜。五个测试用例永远比零个好。
  Stop optimizing blind. No evals means guessing. Five test cases beat zero every time.

---

## 结语

**Conclusion:**

**「Prompt 不是你要写的一个句子。它是一个你要搭建的系统。」**

**"A prompt is not a sentence you write. It is a system you build."**

大多数人读完这些，还会用以前的方式写 prompt。一句话。回车。失败了加「CRITICAL」。然后说这个模型没那么聪明。

Most people will read this and keep writing prompts the way they always have. One sentence. Hit enter. Add "CRITICAL" when it fails. Decide the model is not that smart.

而那些搭建四层结构的人，会在模型升级毁掉所有人 prompt 的时候稳如泰山——因为他们为解析做了结构化，用角色和示例做了方向引导，用工具补足了能力，用 eval 做了验证。

The ones who build the four layers will watch their prompts hold steady through model upgrades that break everyone else's - because they structured for parsing, directed with role and examples, supplied capability with tools, and verified with evals.

> 今晚挑一个你依赖的 prompt，用四层结构过一遍。这就足够让你看到差别了。
>
> Pick one prompt you rely on. Run it through the four layers tonight. That is enough to see the difference.

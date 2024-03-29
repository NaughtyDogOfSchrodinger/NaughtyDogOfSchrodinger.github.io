---
title: visual-chatgpt
date: 2023-03-13 09:32:50
tags: chatgpt
category: 人工智能

---

# 视觉 ChatGPT：使用视觉基础模型进行对话、绘画和编辑

- Chenfei Wu Shengming Yin Weizhen Qi Xiaodong Wang Zecheng Tang Nan Duan* Microsoft Research Asia {chewu, v-sheyin, t-weizhenqi, v-xiaodwang, v-zetang, nanduan}@microsoft.com

## 摘要
ChatGPT因其在许多领域具有出色的对话能力和推理能力而引起了跨领域的关注。然而，由于ChatGPT是通过语言训练的，目前无法处理或生成来自视觉世界的图像。同时，诸如视觉变换器或稳定扩散之类的视觉基础模型，虽然显示出极强的视觉理解和生成能力，但它们只是特定任务的专家，具有一次固定输入和输出。因此，我们构建了一个名为Visual ChatGPT的系统，结合了不同的视觉基础模型，使用户可以通过以下方式与ChatGPT交互：1）发送和接收不仅仅是语言，还有图像；2）提供复杂的视觉问题或需要多个AI模型进行多步协作的视觉编辑指令；3）提供反馈并要求纠正结果。我们设计了一系列提示，将视觉模型信息注入到ChatGPT中，考虑到多输入/输出模型和需要视觉反馈的模型。实验证明，Visual ChatGPT为探究ChatGPT的视觉角色打开了大门，并获得了视觉基础模型的帮助。我们的系统在https://github.com/microsoft/visual-chatgpt上公开可用。

## 介绍
近年来，大型语言模型（LLM）的发展取得了令人难以置信的进展，如T5 [32]、BLOOM [36]和GPT-3 [5]。其中最重要的突破之一是ChatGPT，它是建立在InstructGPT [29]之上的，专门训练以真正会话的方式与用户进行交互，因此允许它保持当前对话的上下文，处理后续问题，并纠正自己产生的答案。虽然功能强大，但ChatGPT在处理视觉信息方面存在限制，因为它是通过单一语言模态进行训练的，而视觉基础模型（VFMs）则在计算机视觉领域显示出巨大的潜力，能够理解和生成复杂的图像。例如，BLIP模型[22]是理解和提供图像描述的专家。稳定扩散[35]则是在基于文本提示的情况下合成图像的专家。然而，受到任务规范性的影响，对于VFMs来说，对输入和输出格式的严格要求使它们在人机交互方面比会话式语言模型不够灵活。

![图1](img/visual1.png)

**<center>图1</center>**
我们能否构建一个类似于ChatGPT的系统，同时支持图像的理解和生成？一个直观的想法是训练一个多模态的会话模型。然而，构建这样一个系统需要大量的数据和计算资源。此外，另一个挑战是，如果我们想要将不仅限于语言和图像的模态，如视频或语音等，纳入系统中，那么每当涉及新的模态或功能时，是否需要训练一个全新的多模态模型？

我们提出了一个名为Visual ChatGPT的系统，以回答上述问题。我们并没有从头开始训练一个新的多模态ChatGPT，而是直接基于ChatGPT构建Visual ChatGPT，并将多种VFMs纳入其中。为了弥合ChatGPT和这些VFMs之间的差距，我们提出了一个Prompt Manager，支持以下功能：1）明确告诉ChatGPT每个VFM的能力并指定输入输出格式；2）将不同的视觉信息（例如png图像、深度图像和掩码矩阵）转换为语言格式，以帮助ChatGPT理解；3）处理不同Visual Foundation Models的历史记录、优先级和冲突。在Prompt Manager的帮助下，ChatGPT可以利用这些VFMs，并以迭代的方式接收它们的反馈，直到满足用户的要求或达到结束条件。

如图1所示，用户上传了一张黄花的图像并输入了一个复杂的语言指令“请在基于预测深度的条件下生成一朵红色的花，然后让它变成卡通样式，一步一步来”。在Prompt Manager的帮助下，Visual ChatGPT开始执行相关的视觉基础模型链。在这种情况下，它首先应用深度估计模型检测深度信息，然后利用深度到图像模型生成带有深度信息的红色花的图像，并最终利用基于Stable Diffusion模型的风格转换VFM将这个图像的风格转换成卡通样式。在以上流程中，Prompt Manager作为ChatGPT的调度器，提供视觉格式类型，并记录信息转换的过程。最终，当Visual ChatGPT从Prompt Manager获得“卡通”的提示时，它将结束执行流程并显示最终结果。

总之，我们的贡献如下：
- 我们提出了Visual ChatGPT，它打开了将ChatGPT和视觉基础模型结合起来处理复杂视觉任务的大门。
- 我们设计了Prompt Manager，其中涉及22种不同的视觉基础模型，并定义了它们之间的内部关联，以获得更好的交互和组合。
- 我们进行了大量的零-shot实验，并展示了丰富的案例来验证Visual ChatGPT的理解和生成能力。

## 相关工作
### 自然语言和视觉
在我们的生活中，各种模态（声音、视觉、视频等）环绕着我们，语言和视觉是传递信息的两个主要媒介。自然语言和视觉之间存在自然的联系，大多数问题需要联合建模这两个流以产生满意的结果[15，26，48]，例如，视觉问答（VQA）[2]将一张图像和一个相应的问题作为输入，并要求根据给定图像中的信息生成答案。由于大型语言模型（LLM）的成功，例如InstructGPT[29]，人们可以轻松地使用自然语言格式与模型交互或获得反馈，但它无法处理视觉信息。要将视觉处理能力融入这样的LLM中，存在几个挑战，因为训练大型语言模型或视觉模型都很困难，并且需要精心设计的指令[4，55，21]和繁琐的转换[30，52]来连接不同的模态。虽然一些工作已经探索了利用预训练的LLM来改进视觉-语言（VL）任务的性能，但这些方法支持一些特定的VL任务（从语言到版本或从版本到语言）并需要标记数据进行训练[38，1，22]。

### 预训练模型用于VL任务
为了更好地提取视觉特征，在早期的工作中采用了冻结的预训练图像编码器 [9、25、54]，而最近的 LiT [52]则采用了冻结的 ViT 模型 [51] 进行 CLIP 预训练 [30]。从另一个角度来看，利用 LLMs 的知识也很重要。按照 Transformer [39] 的指示，预训练的 LLMs 展示了强大的文本理解和生成能力 [31、19、37、5]，这些突破也有益于 VL 建模 [13、14、3、49]，其中这些工作在预训练的 LLMs 中添加额外的适配器模块 [17]，以将视觉特征对齐到文本空间。由于模型参数的增加，训练这些预训练的 LLMs 很困难，因此更多的工作已经直接利用现成的冻结的预训练 LLMs 来进行 VL 任务 [12、38、8、46、50]。

### 预训练的LLM在VL任务中的指导
为了处理复杂的任务，例如常识推理 [11]，提出了 Chain-of-Thought（CoT）来引出 LLMs 的多步推理能力 [42]。更具体地，CoT 要求 LLMs 生成最终结果的中间答案。现有的研究 [57] 将这种技术分为两类：Few-Shot-CoT [56] 和 Zero-Shot-CoT [20]。在 few-shot 设置下，LLMs 使用几个演示进行 CoT 推理 [58, 41]，结果表明 LLMs 可以获得更好的解决复杂问题的能力。此外，最近的研究 [20, 47] 已经表明，LLMs 可以在零样本设置下通过利用自生成的原因而自我提高。以上研究主要集中在单一模态，即语言。最近，MultimodalCoT [57] 提出了将语言和视觉模态纳入一个两阶段框架的方法，将原因生成和答案推理分开。然而，这种方法仅在特定情况下，即 ScienceQA 基准测试 [28] 中表现出优越性。简而言之，我们的工作将 CoT 的潜力扩展到包括但不限于文本到图像生成 [27]，图像到图像翻译 [18]，图像到文本生成 [40] 等大规模任务中。

![图2](img/visual2.png)

**<center>图2 Visual ChatGPT的概述。左侧展示了一个三轮对话，中间展示了Visual ChatGPT如何迭代调用Visual Foundation模型并提供答案的流程图。右侧展示了第二个问答过程的详细过程。</center>** 

## Visual ChatGPT
假设S = {(Q1, A1),(Q2, A2), ...,(QN , AN )}为一个包含N个问题-回答对的对话系统。为了得到第i轮对话的回答Ai，需要使用一系列VFMs和这些模型的中间输出A(j)i，其中j表示第i轮中第j个VFM (F)的输出。更具体地，通过Prompt Manager M的处理，A(j)i的格式不断修改以符合每个F的输入格式。最终，如果A(j)i被标记为最终回答，则系统输出A(j)i，并且不再执行更多的VFM。公式（1）提供了Visual ChatGPT的正式定义：

A(j+1)i = ChatGP T(M(P),M(F),M(H<i),M(Qi), M(R(<j)i),M(F(A(j)i))) (1)

- **系统原则 P**：系统原则（System Principle）为Visual ChatGPT提供基本规则，例如，它应该对图像文件名敏感，应该使用VFMs来处理图像，而不是根据聊天历史生成结果。
- **视觉基础模型 F**：Visual ChatGPT的核心之一是各种视觉基础模型的组合：F = {f1，f2，...，fN}，其中每个基础模型fi都包含具有明确输入和输出的确定函数。
- **对话历史 H<i**： 我们将第i轮对话的历史定义为前面问题-答案对的字符串连接，即 {(Q1, A1),(Q2, A2), · · · ,(Qi−1, Ai−1)}。此外，我们根据最大长度阈值截断对话历史，以符合ChatGPT模型的输入长度要求。
- **用户的询问 Qi**：在Visual ChatGPT中，“query”是一个通用术语，因为它可以包括语言和视觉查询。例如，图1展示了一个包含查询文本和相应图像的示例查询。
- **推理历史记录 R(<j)i**：为了解决一个复杂的问题，Visual ChatGPT 可能需要多个 VFMs 协同合作。对于第 i 轮对话，R(<j) i 表示从第 j 个调用的 VFM 开始到当前轮次的所有推理历史。
- **中间答案 A(j)**：当处理复杂的查询时，Visual ChatGPT会逻辑上调用不同的VFMs，逐步获得最终答案，从而产生多个中间答案。
- **提示管理器M**：提示管理器被设计用来将所有视觉信号转换为语言，以便ChatGPT模型可以理解。在下面的子节中，我们重点介绍M如何管理上述不同部分：P、F、Qi、F(A(j)i)。

![图3](img/visual3.png)

**<center>图3. 提示管理器概览。它将所有非语言信号转化为语言，以便ChatGPT能够理解。</center>**

### 系统原则的提示管理 M(P)
Visual ChatGPT是一个系统，集成了不同的VFMs来理解视觉信息并生成相应的答案。为了实现这一目标，需要定制一些系统原则，并将其转化为ChatGPT可以理解的提示。这些提示具有以下几个目的：

- **Visual ChatGPT的角色** Visual ChatGPT旨在协助处理各种文本和视觉相关任务，例如VQA、图像生成和编辑等。

- **VFMs的可访问性 Visual** ChatGPT可以访问各种VFMs来解决不同的VL任务。ChatGPT模型本身决定使用哪个基础模型，因此很容易支持新的VFMs和VL任务。

- **文件名的敏感性** Visual ChatGPT根据文件名访问图像文件，因此使用精确的文件名非常重要，以避免歧义，因为一轮对话可能包含多个图像及其不同的更新版本，文件名的误用将导致哪个图像当前正在讨论的混淆。因此，Visual ChatGPT被设计为对文件名使用严格的要求，以确保正确检索和操作图像文件。

- **思维链** 如图1所示，处理一个看似简单的命令可能需要多个VFMs，例如，“生成一个基于预测深度的红色花朵图像，并使其像漫画一样”。这需要进行深度估计、深度到图像的转换和样式转移等多个VFMs的协同工作。为了将更具挑战性的查询分解为子问题，Visual ChatGPT引入了CoT来帮助决定、利用和分派多个VFMs。

- **推理格式的严格性** Visual ChatGPT必须遵循严格的推理格式。因此，我们使用精细的正则表达式匹配算法解析中间推理结果，并构建ChatGPT模型的合理输入格式，以帮助它确定下一步的执行，例如触发新的VFM或返回最终响应。

- **可靠性** 作为一种语言模型，Visual ChatGPT可能会捏造虚假的图像文件名或事实，从而使系统不可靠。为了处理这些问题，我们设计了提示，要求Visual ChatGPT忠于视觉基础模型的输出，不要捏造图像内容或文件名。此外，多个VFMs的协作可以提高系统的可靠性，因此我们构建的提示将指导ChatGPT优先利用VFMs而不是基于对话历史生成结果。

![表1](img/table1.png)

**<center>表1. Visual ChatGPT支持的基础模型。</center>**

### 基础模型的提示管理M(F)
Visual ChatGPT配备了多个VFMs来处理各种VL任务。由于这些不同的VFMs可能存在一些相似性，例如，图像中的对象替换可以视为生成新图像，图像到文本（I2T）任务和图像问答（VQA）任务都可以理解为根据提供的图像给出响应，因此区分它们非常重要。如图3所示，Prompt Manager专门定义了以下方面，以帮助Visual ChatGPT准确理解和处理VL任务：
- **名称** 名称提示为每个VFM提供了一个总体功能的摘要，例如回答有关图像的问题，它不仅有助于Visual ChatGPT以简洁的方式理解VFM的目的，而且还作为进入VFM的入口。
- **用途** 用途提示描述了应使用VFM的特定情景。例如，Pix2Pix模型[35]适用于更改图像的样式。提供这些信息可以帮助Visual ChatGPT决定为特定任务使用哪个VFM。
- **输入/输出** 输入和输出提示概述了每个VFM所需的输入和输出格式，因为格式可能会有很大的变化，提供清晰的指导对于Visual ChatGPT正确执行VFMs至关重要。
- **示例（可选）** 示例提示是可选的，但它对于Visual ChatGPT更好地理解如何在特定的输入模板下使用特定的VFM并处理更复杂的查询非常有帮助。

### 用户查询管理M(Qi)
Visual ChatGPT支持多种用户查询，包括语言或图像、简单或复杂的查询，以及引用多个图像。Prompt Manager在以下两个方面处理用户查询：
- **生成唯一文件名** Visual ChatGPT可以处理涉及新上传图像和涉及引用现有图像的两种类型的图像相关查询。对于新上传的图像，Visual ChatGPT使用通用唯一标识符（UUID）生成唯一文件名，并添加表示相对目录的前缀字符串“image”，例如“image/{uuid}.png”。虽然新上传的图像不会被输入到ChatGPT中，但会生成一个虚假的对话历史，其中包含一个说明图像文件名的问题和一个表示已收到图像的答案。这个虚假的对话历史有助于后续的对话。对于涉及引用现有图像的查询，Visual ChatGPT忽略文件名检查。这种方法已被证明是有益的，因为如果不会引起歧义，例如UUID名称，ChatGPT具有理解用户查询的模糊匹配的能力。
- **强制VFM思考** 为确保Visual ChatGPT成功触发VFMs，我们在(Qi)的后缀提示中添加了一个后缀提示：“由于Visual ChatGPT是一个文本语言模型，Visual ChatGPT必须使用工具来观察图像，而不是想象。思考和观察仅对Visual ChatGPT可见，Visual ChatGPT应该记得在最终回复中重复重要信息。思考：我需要使用工具吗？”这个提示有两个目的：1）它提示Visual ChatGPT使用基础模型，而不是仅仅依靠想象；2）它鼓励Visual ChatGPT提供基础模型生成的具体输出，而不是通用响应，例如“这是你要的”。

### 基础模型输出的提示管理 M(F(A(j)i))
对于来自不同VFMs F(A(j)i)的中间输出，Visual ChatGPT将隐式地总结并将其馈送给ChatGPT以进行后续交互，即调用其他VFMs进行进一步操作，直到达到结束条件或向用户反馈。内部步骤可总结如下：
- **生成链接的文件名**：由于Visual ChatGPT的中间输出将成为下一个隐式对话轮的输入，因此我们应该使这些输出更加逻辑化，以帮助LLMs更好地理解推理过程。具体来说，从Visual Foundation Models生成的图像保存在“image / folder”下，这暗示以下字符串表示图像名称。然后，图像被命名为“{Name} {Operation} {Prev Name} {Org Name}”，其中{Name}是上面提到的UUID名称，{Operation}是操作名称，{Prev Name}是输入图像的唯一标识符，{Org Name}是由用户上传或由VFMs生成的图像的原始名称。例如，“image / ui3c edge-of o0ec nji9dcgf.png”是命名为“ui3c”的canny边缘图像的输入“o0ec”，而该图像的原始名称为“nji9dcgf”。通过这样的命名规则，它可以提示ChatGPT中间结果的属性，即图像以及它是如何从一系列操作中生成的。
- **调用更多的VFMs**：Visual ChatGPT的核心之一是它可以自动调用更多的VFMs来完成用户的命令。更具体地说，我们让ChatGPT通过在每个生成的末尾扩展一个后缀“思考：”来不断问自己是否需要VFMs来解决当前的问题。
- **请求更多细节**：当用户的命令不明确时，Visual ChatGPT应该询问用户更多细节，以帮助更好地利用VFMs。这个设计是安全和关键的，因为LLMs不被允许在没有基础的情况下任意篡改或推测用户的意图，特别是当输入信息不足时。

## 实验
### 设置
我们使用ChatGPT [29]（OpenAI“text-davinci-003”版本）实现LLM，并使用LangChain [7]1来指导LLM。我们从HuggingFace Transformers [43]2、Maskformer [10]3和ControlNet [53]4中收集基础模型。所有22个VFMs的完全部署需要4个Nvidia V100 GPU，但用户可以灵活地部署较少的基础模型以节省GPU资源。聊天历史的最大长度为2,000，过多的令牌将被截断以满足ChatGPT的输入长度。

### 一次完整的多轮对话案例

图4展示了Visual ChatGPT的16轮多模态对话案例。在此案例中，用户提出了文本和图像问题，而Visual ChatGPT则回答了文本和图像问题。该对话涉及多张图像的讨论，使用多个基础模型进行处理，并处理需要多个步骤的问题。

![图4](img/visual4.png)

**<ceter>图4. 人类与Visual ChatGPT之间的多轮对话。在对话中，Visual ChatGPT可以理解人类意图，支持语言和图像输入，并完成生成、提问和编辑等复杂的视觉任务。</center>**

![图5](img/visual5.png)

**<center>图5. 系统原则的提示管理案例研究。我们定性分析了四个提案：文件名灵敏度、推理格式严格性、可靠性和思维链。左上角显示，在M(P)中强调文件名灵敏度是否会影响文件引用的准确性。没有推理格式严格性，无法对右上角的点进行进一步解析。左下角显示，告诉Visual ChatGPT忠实于工具观察而不是伪造图像内容的差异。右下角显示，强调使用工具链的能力将有助于决策。</center>**

![图6](img/visual6.png)

**<center>图6。关于基础模型的提示管理案例研究。我们对四个提议进行定性分析：名称、用途、输入/输出和示例。左上角显示，当没有工具名称时，Visual ChatGPT将猜测工具名称，然后无法正确使用该工具。右上角显示，当工具名称的用途缺失或不清楚时，它将调用其他工具或遇到错误。左下角显示，缺乏输入/输出格式要求将导致错误的参数。右下角显示，示例有时是可选的，因为ChatGPT能够总结历史信息和人类意图，以使用正确的工具。</center>**

![图7](img/visual7.png)

**<center>图7. 用户查询和模型输出的提示管理案例研究。我们定性分析了四个建议：唯一的文件名、强制VFM思考、链接文件名和请求更多细节。左上角显示唯一的文件名可以避免覆盖。右上角显示强制VFM思考鼓励工具调用和严格思考格式。左下角显示链接文件名有助于理解文件，并且Visual ChatGPT可以成功观察和总结。右下角显示Visual ChatGPT能够检测模糊引用并请求更多细节。</center>**

### 提示管理器的案例研究

**系统原则的提示管理案例研究**在图5中进行了分析。为了验证我们的系统原则提示的有效性，我们从中移除不同的部分，比较模型性能。每次移除都会导致不同的能力退化。

**基础模型提示管理的案例研究**在图6中进行了分析。VFM的名称是最重要的，需要清晰地定义。当名称缺失或含糊不清时，Visual ChatGPT会多次猜测直到找到现有的VFM，或者遇到错误，如左上图所示。VFM的使用应明确描述应该使用模型的具体情境，以避免错误响应。右上图显示，样式转换被误处理为替换。输入和输出格式应准确提示，以避免参数错误，如左下图所示。示例提示可以帮助模型处理复杂的使用情况，但是是可选的。正如右下图所示，即使我们删除了示例提示，ChatGPT也可以总结对话历史和人类意图来使用正确的VFM。完整的视觉基础模型提示在附录A中显示。

**用户查询提示管理的案例研究**在图7的上部分进行了分析。左上图显示，如果没有图像文件的唯一命名，新上传的图像文件可能会被重命名以避免被覆盖并导致错误引用。如右上图所示，通过将思想引导从M(P)移动到M(Q)并以Visual ChatGPT的声音作为强制性思考来表达，强调了更多的VFM调用而不是基于文本上下文的想象，相比之下在Q2中强制Visual ChatGPT说“思考：我需要使用工具吗？” M(Q)使得更容易正确通过正则表达式匹配。相反，没有强制思考，A3可能会错误地生成思想结束标记，并直接将其ChatGPT输出视为最终响应。

**模型输出提示管理的案例研究**在图7的下部分进行了分析。左下图比较了移除和保留链接命名规则的性能。使用链接命名规则，Visual ChatGPT可以识别文件类型，触发正确的VFM，并得出文件依赖关系命名规则的结论。它显示链接命名规则确实有助于Visual ChatGPT的理解。右下图给出了一个示例，在项目推断不明确时请求更多细节，这也表明我们系统的安全性。

## 限制
虽然Visual ChatGTP是一个有前途的多模式对话方法，但它也存在一些限制，包括：
- **依赖于ChatGPT和VFMs Visual ChatGPT** 在任务分配方面高度依赖ChatGPT，在任务执行方面则高度依赖VFMs。因此，Visual ChatGPT的性能受这些模型的准确性和效率的影响。
- **大量的提示工程** Visual ChatGPT需要大量的提示工程来将VFMs转换成语言，并使这些模型描述具有区别性。这个过程可能耗时，并且需要计算机视觉和自然语言处理方面的专业知识。
- **有限的实时能力** Visual ChatGPT被设计为通用型，它尝试自动将复杂任务分解成多个子任务。因此，在处理特定任务时，Visual ChatGPT可能会调用多个VFMs，这会导致其实时能力受到限制，与专门针对特定任务进行训练的专家模型相比，存在局限性。
- **令牌长度限制** ChatGPT中的最大令牌长度可能会限制可以使用的基础模型数量。如果有成千上万或数百万个基础模型，则需要一个预过滤模块来限制提供给ChatGPT的VFMs。
- **安全和隐私** 能够轻松地插入和拔出基础模型可能会引起安全和隐私方面的担忧，特别是对通过API访问的远程模型。必须仔细考虑和自动检查，以确保不应暴露或泄露敏感数据。

## 结论
在这项工作中，我们提出了Visual ChatGPT，这是一个开放系统，结合了不同的VFMs，使用户能够以超越语言格式的方式与ChatGPT进行交互。为了构建这样一个系统，我们精心设计了一系列提示，以帮助将视觉信息注入ChatGPT，从而可以逐步解决复杂的视觉问题。大量的实验和选定的案例已经证明了Visual ChatGPT在不同任务中的巨大潜力和能力。除了上述限制之外，另一个问题是由于VFMs的失败和提示的不稳定性，一些生成结果不满意。因此，一个自我校正模块是必要的，用于检查执行结果和人类意图之间的一致性，并相应地进行相应的编辑。这种自我校正行为可以导致模型更复杂的思考，显著增加推理时间。我们将在未来解决这样的问题。





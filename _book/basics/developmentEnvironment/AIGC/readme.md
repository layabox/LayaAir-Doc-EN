# AI Collaborative Development Environment

> Author: Charley

In today's software development landscape, clinging to traditional development environments and work modes makes it difficult to be considered a programmer with cutting-edge competitiveness. Mastering and making good use of various AIGC tools, and fully leveraging AI's capabilities in understanding, generation, and creation, is becoming the key to improving development efficiency and innovation capabilities.

The form of AI collaborative development is rapidly evolving, with mainstream types roughly falling into four categories: First is **AI Q&A**, used for knowledge retrieval and question answering; Second is **AIGC Creation**, using generative models to create text, images, or code content; Third is **Code Assistant and Intelligent Completion**, achieving code generation and optimization through context understanding; Fourth is **Agent Collaboration**, enabling AI to autonomously execute tasks and continuously collaborate on development.

## 1. AI Q&A

### 1.1 Recommended Mainstream AI Q&A Models

From the perspective of interaction modes and usage scenarios, AI Q&A systems can be primarily divided into three typical forms: **web-based Q&A**, **IDE-integrated**, and **API-based**. These three forms cover the main usage scenarios of current AI Q&A, reflecting the integration methods and application depth of AI at different levels.

Among them, **IDE-integrated** will be introduced separately in later chapters; **API-based** tends to focus more on enterprise integration and system development, making it less suitable for individual developers, so it won't be covered here.

**Web-based Q&A** uses web pages as the primary interaction interface. Users don't need programming or configuration to directly converse with AI models through natural language, obtaining technical knowledge (e.g., newcomers' confusion about language and syntax), inspiration, or creative assistance.

International mainstream products include: [ChatGPT](https://chatgpt.com/), [Gemini](https://gemini.google.com/), [Perplexity AI](https://www.perplexity.ai/), [Claude.ai](https://claude.ai/), etc.

Domestic mainstream products include: [Doubao (豆包)](https://www.doubao.com/chat/), [Tencent Yuanbao (腾讯元宝)](https://yuanbao.tencent.com/chat/), [Tencent Hunyuan (腾讯混元)](https://hunyuan.tencent.com/), [Wenxin Yiyan (文心一言)](https://yiyan.baidu.com/), [Tongyi Qianwen (通义千问)](https://www.tongyi.com/), [Zhipu Qingyan (智谱清言)](https://chatglm.cn/main/), [Kimi](https://www.kimi.com/), etc.

### 1.2 Recommended AI Models for Reasoning and Retrieval

In early large language models, due to the lack of **web search capability**, models could only rely on training corpora to generate responses. This mechanism easily led to "hallucination" problems and often produced outdated information. When querying professional technical materials, especially newer knowledge, models might provide seemingly reasonable but actually incorrect information.

Therefore, when using AI Q&A, users still need to rely on traditional search engines to ensure the accuracy of answers.

With the maturity of **web search and Retrieval-Augmented Generation (RAG)** technology, more and more AI models are beginning to have real-time access to internet information, thus achieving a better balance between factual accuracy and reasoning depth.

Overall, reasoning-based AI Q&A models with web search capabilities are gradually replacing the boundaries between traditional search and Q&A, bringing more efficient and reliable knowledge acquisition methods to developers and content creators.

Among current mainstream products, the author believes that **[Tencent Yuanbao (腾讯元宝)](https://yuanbao.tencent.com/)** and **[DeepSeek](https://chat.deepseek.com/)** achieve a good balance in both "**web retrieval capability**" and "**logical reasoning performance**".

## 2. **AIGC Creation**

AIGC (Artificial Intelligence Generated Content) refers to the production method of using artificial intelligence technology to generate text, images, audio, video, code, and other content.

### 2.1 Conventional AIGC Creation

Current mainstream natural language processing (NLP) large models all have text generation AIGC capabilities, such as generating business copy, literary works, code, and other text content.

With the competition and development of AIGC, image generation capabilities are increasingly becoming a standard feature of large models. For example: **ChatGPT, Doubao (豆包), Tencent Hunyuan (腾讯混元), Wenxin Yiyan (文心一言), Tongyi Qianwen (通义千问), Zhipu Qingyan (智谱清言)**, etc.

### 2.2 AIGC Creation in Vertical Fields

In addition to general-purpose large models for natural language, there are also many large models specializing in vertical fields. For example, text-to-image, text-to-3D models, text-to-audio, text-to-video, text-to-game, etc. Text-to-video is basically not useful for LayaAir developers, so it won't be introduced here. I'll focus on recommending some AI models that can be used for AIGC creation in games and other interactive products.

#### 2.2.1 Text-to-Image, Image-to-Image

AI-generated images can be used to generate game backgrounds, texture maps, 2D characters, UI, etc. Some requirements can be directly generated for use, while others need secondary artistic processing (which can also improve efficiency).

In addition to general-purpose large models with built-in image generation, [Midjourney](https://www.midjourney.com/editor/), Stable Diffusion, [Jimeng AI (即梦AI)](https://jimeng.jianying.com/ai-tool/home), [Tencent Hunyuan (腾讯混元)](https://hunyuan.tencent.com/image/) are all relatively high-quality image generation large models.

These large models can generate stylized, high-quality images by users entering text prompts. Some large models also have image-to-image capabilities, allowing for modification and creation by uploading reference images.

#### 2.2.2 Text-to-3D Models

Text-to-3D models are currently relatively less mature than image generation. I recommend several mainstream 3D models that you can experience and try.

They are: [**Tripo 3D**](https://studio.tripo3d.ai/home), [**Fast3D**](https://fast3d.io/zh), [**Luma AI - Genie**](https://lumalabs.ai/genie?view=create), [**Meshy AI**](https://www.meshy.ai/)

#### 2.2.3 Text-to-Audio

The most popular text-to-audio tools are **ElevenLabs SFX** and **MusicGen**.

**ElevenLabs SFX v2** is launched by AI audio R&D company ElevenLabs, online at: [https://elevenlabs.io/](https://elevenlabs.io/)

**MusicGen** is a large model developed and open-sourced by Meta, GitHub address: [https://github.com/facebookresearch/audiocraft](https://github.com/facebookresearch/audiocraft)

#### 2.2.4 Text-to-Game

Text-to-game is currently quite difficult. JS game code generated by general-purpose large models is only at the basic DEMO gameplay level and difficult to commercialize.

Currently only LayaIdea can generate commercial products including game prototypes, simple games, and playable ads.

For more information, see the WeChat article: [https://mp.weixin.qq.com/s/8ANLuz4vcnexhoija9eumA](https://mp.weixin.qq.com/s/8ANLuz4vcnexhoija9eumA)

## 3. Code-Assisted Development

AI application in the code field is one of the most direct directions for improving development efficiency. It can not only assist programmers with automatic completion, function generation, and code optimization but also gradually develop intelligent programming environments with deep understanding and autonomous rewriting capabilities.

### 1.1 Code-Assisted Development Plugins

Code assistance plugins are the earliest widely adopted form of AI programming tools, with the most representative product being **GitHub Copilot**. Based on large-scale code training models, it can automatically provide completion, refactoring, and comment suggestions when developers input code, greatly improving coding efficiency.

Subsequently, several similar AI programming plugins appeared in China, such as Tencent's CodeBuddy and Baidu's Wenxin Kuaima (Baidu Comate).

These plugins can be installed in the coding environment through VSCode extensions, automatically completing code, generating functions, documentation, and refactoring suggestions based on context. They are practical tools for improving efficiency.

### 1.2 Deeply Assisted Development Editors

Unlike traditional plugins, **Agentic IDE** is a higher form of AI collaborative development. It's no longer just a "code suggestion tool" but an **AI programming partner** with more comprehensive context understanding and task planning capabilities.

The most representative product is **[Cursor](https://www.cursor.com/)**.

Cursor is based on the open-source VS Code architecture and integrates AI Agent functionality with deep thinking capabilities. It can:

- Globally understand project structure and dependencies;
- Add or modify code files based on natural language instructions;
- Execute multi-step logical reasoning and code generation;
- Assist with feature iteration and bug fixes for large projects.

The emergence of such editors marks AI's transition from "code assistance" to "code collaboration".

Domestic products similar to Cursor's intelligent agent coding platform are also developing rapidly, such as ByteDance's [Trae](https://www.trae.cn/) and Alibaba's [Qoder](https://qoder.com/).

I recommend everyone to download and use them.

## 4. MCP

MCP (Model Context Protocol) is an open standard protocol developed by Anthropic, aiming to standardize the interaction between large language models (LLMs) and external data sources, tools, and applications. It's regarded as the "USB-C interface of the AI world."

Its core advantage lies in breaking the "fragmentation" barrier between AI and external resources through a unified protocol, achieving "develop once, adapt to multiple ends," significantly reducing the integration complexity of AI applications with data sources/tools; supporting real-time data interaction and multi-tool collaboration, improving AI's response accuracy and flexibility in real-world scenarios.

Currently, MCP has become a mainstream protocol in the AI field, supported by tech giants such as Google, Notion, Figma, and OpenAI, and applied in multiple scenarios including cross-database analysis, 3D design, and intelligent schedule generation.

LayaAir3-IDE already supports AI agent collaborative development based on the MCP protocol.

LayaAir engine's MCP plugin address and usage instructions: [https://store.layaair.com/info.php?id=10326](https://store.layaair.com/info.php?id=10326)

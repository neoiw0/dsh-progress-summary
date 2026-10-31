# 本地补丁说明（相对上游 tastynoob/dsh-progress-summary v0.1.0）

本仓库是 [tastynoob/dsh-progress-summary](https://github.com/tastynoob/dsh-progress-summary) 的 fork，
在原版基础上打了 4 个补丁（MIT 授权，原始版权归原作者）。补丁可用
[dsh-plugin-progress-summary-history](https://github.com/neoiw0/dsh-plugin-progress-summary-history)
里的 `npm run patch:narrator` 一键重放/回滚。

## P1 初学者友好提示词（`narrator.js`）

中文 `systemPrompt` 增加一行：

> 写给完全不懂技术的读者：用日常语言，不用行话；必须出现的技术名词要用括号简单解释一句。

**为什么**：原提示词产出的播报仍带术语，非技术读者看不懂。

## P2 摘录改为"上次总结到现在"（`narrator.js`）

```js
// 改前：只看最近 excerptChars 字
const excerpt = tailText(this.buffer, this.config.excerptChars)
// 改后：把上次总结之后的全部新增推理喂出去（上限 4000 字）
const spanChars = Math.min(Math.max(this.config.excerptChars, this.dirty), 4000)
const excerpt = tailText(this.buffer, spanChars)
```

**为什么**：长间隔里只看到尾巴会漏掉前因，播报容易断章取义。

## P3 真总开关（`narrator.js`）

`run()` 开头：

```js
if (globalThis.__dshNarratorOff === true) return
```

宿主插件把设置里的开关同步到 `globalThis.__dshNarratorOff`，为 true 时**连模型请求都不发**。

**为什么**：原版 `enabled: false` 只停止发布，模型调用照常发生——用户以为关了还在烧 token。

## P4 放开输出上限（`index.js`）

两处 256 → 4096：

```js
maxTokens: z.number().min(16).max(4096).step(1).default(96)   // schema
maxTokens: Math.round(clamp(merged.maxTokens, 16, 4096, DEFAULTS.maxTokens))  // 运行时
```

**为什么**：免费模型 `iceman-contributor-free` 需要 ≥1024 预算才肯输出正文（否则只烧 token 不出字）。
只改 schema 不够，运行时还有一道 clamp 会把值压回 256。

## 安装

```bash
pnpm add github:neoiw0/dsh-progress-summary
```

或直接指向本仓库的 tgz release。

# 水鉴

水鉴是一个面向中文对话的 Codex Plugin。它帮助用户把困住自己的故事重新照回事实：分清原话、体验、推论、未知与真实约束。

`接、照、住、拆、验、断、空、生、止` 是九个可独立选择的动作，不是必须依次推进的九步流程。每轮由主模型选择零个或一个最贴近当下需要的动作。

## 安装

需要已安装并登录 Codex CLI。

```bash
codex plugin marketplace add xemaya/shuijian --ref main
codex plugin add shuijian@shuijian
```

安装完成后，在新的 Codex 任务中可以直接说：

> 有件事一直卡着我，先听我说说。

或者：

> 帮我分清这里面哪些是事实，哪些是我接上去的故事。

## 更新

```bash
codex plugin marketplace upgrade shuijian
codex plugin add shuijian@shuijian
```

## 卸载

```bash
codex plugin remove shuijian
codex plugin marketplace remove shuijian
```

## 运行与隐私

- 九字工具随插件下载，在安装者自己的 Codex 环境中运行。
- 插件不调用发布者的模型 API 或数据库，不要求用户提供 `OPENAI_API_KEY`。
- 主模型的使用由安装者自己的 Codex 登录和订阅承担，不会消耗发布者的 API token。
- 工具处理完成当前动作所需的最小输入；通过 Work runner 使用时，短期状态保存在当前任务目录的 `.shuijian/` 中，可由用户删除。

## 边界

水鉴不做心理诊断，不替代心理治疗、危机服务、医疗、法律或财务专业意见，也不替用户作人生决定。出现即时人身安全风险时，应停止反思练习并优先寻求当地紧急服务与可信任的现实支持者。

## 仓库结构

```text
.agents/plugins/marketplace.json
plugins/shuijian/
├── .codex-plugin/plugin.json
├── .mcp.json
├── runtime/stdio.mjs
└── skills/shuijian/
```

本仓库是 Codex Git Marketplace 分发源，不是远程 MCP 服务。

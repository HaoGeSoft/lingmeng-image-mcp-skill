# Lingmeng AI Image MCP Skill

这是一个用于 Codex 的 Skill，帮助通过凌梦 AI MCP 完成图片生成、模型选择、多图批次控制、参考图编辑，以及在本地 Shell 可用时下载原图。

## 安装

将仓库中的 `SKILL.md` 和 `agents/openai.yaml` 复制到本地 Codex skills 目录：

```text
<CODEX_HOME>/skills/lingmeng-image-mcp/
```

Windows 默认目录通常是：

```text
%USERPROFILE%\\.codex\\skills\\lingmeng-image-mcp\\
```

然后重新打开 Codex 或刷新 Skill 列表。

## 使用

可以显式调用：

```text
使用 $lingmeng-image-mcp 生成一张图片并保存到本地。
```

也可以直接说：

```text
生成 3 张 gpt-image-2.5-flare 图片并保存到本地。
```

Skill 不包含 API Key、访问令牌、服务器凭据或用户数据。实际生图仍需要当前 Codex 会话已经配置并授权凌梦 AI MCP。

## 许可

Skill instructions are provided for personal and project use. Review the repository license and your MCP provider terms before redistribution.

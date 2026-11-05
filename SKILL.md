---
name: lingmeng-image-mcp
description: "Use the Lingmeng AI MCP for controlled image generation, batching, and local original-file downloads."
metadata:
  short-description: "凌梦 AI MCP 生图、批次控制与本地原图保存"
---

# 凌梦 AI 生图与本地保存

在用户要求通过凌梦 AI MCP 生图、指定模型/数量/尺寸/质量，或要求把生成结果保存到本地时使用本 Skill。

## 生成规则

- 使用当前 Codex 会话中已经配置的凌梦 AI MCP 工具，不要猜测接口地址、另建渠道或绕过 MCP 授权。
- 先从用户请求提取 `prompt`、`model`、`count`、`size` 和 `quality`。用户明确指定的模型和尺寸优先；未指定时使用 MCP 服务端默认值。
- 当前生图分组已确认支持这些模型 ID：`gpt-image-2`、`gpt-image-2.5-flare`、`gpt-image-2.5-flare-2k`、`gpt-image-2.5-flare-4k`、`gpt-image-2.5-sunburst`、`gpt-image-2.5-sunburst-2k`、`gpt-image-2.5-sunburst-4k`。用户说“GPT image 2.5 burst”时，按实际模型名 `gpt-image-2.5-sunburst` 处理；不要自行改写成不存在的 `burst`。
- 服务端会从“生图分组”动态读取已启用模型，因此管理员后续新增的生图模型也可以通过 `model` 参数传入；如果服务端返回模型不支持，应报告准确的模型 ID 错误并停止，不要自动换模型。
- 多图请求必须在一次 `generate_image` 调用中使用 `count` 表示总数量。当前服务单批支持 1～10 张；不要为了“补链接”“提取高清图”或等待结果而再次调用 `generate_image`。
- 超过 10 张时，先说明会变成多个独立批次并可能分别计费，再等待用户明确同意后分批执行。不要自行把一次请求拆成多个生图调用。
- 等待工具返回完整批次结果。结果中已有图片链接时，优先使用原图链接；不要把带 `preview=1` 的预览地址当作原图下载。
- 如果工具返回任务 ID，后续只用 `get_image_job` 查询该任务；不要重新提交同样的生图请求。

## 参考图编辑

- 用户提供本地参考图时，先由本地文件工具读取并转成图片 data URL/Base64，再调用 `edit_image` 的 `image_data`；不要把本地路径直接发送给远程 MCP。
- 遮罩图通过 `mask_data` 传入。编辑任务一次生成 1 张，模型、尺寸和质量仍可按用户要求传入。
- 参考图读取失败、格式不支持或 MCP 返回授权/任务错误时，停止并说明原因，不要重复提交。

## 本地保存

当用户要求保存图片时，使用当前环境可用的本地 Shell/文件工具下载原图：

1. Windows 优先保存到工作区 `outputs/images/`；没有工作区时保存到 `%USERPROFILE%\Pictures\凌梦 AI\`。先创建目录。
2. 从 MCP 返回的图片链接中移除 `preview=1` 参数，但保留其他必要查询参数。下载时跟随重定向并失败即报错。
3. 文件名使用日期、模型短名和序号，例如 `2026-11-05-gpt-image-2-001.png`。不要把完整临时 URL、授权令牌或用户凭据写入日志或文件名。
4. 下载后确认文件存在且大小大于 0；必要时读取响应的 MIME 类型决定 `.png`、`.jpg` 或 `.webp` 扩展名。
5. 最终回复列出实际保存的绝对路径，并可同时提供图片预览。没有本地 Shell/写文件能力时，明确说明“已生成但未能自动保存”，不能声称保存成功。

Windows 示例（将变量替换为已经从工具结果中提取的单个原图 URL 和目标路径；不要把命令原样展示给用户）：

```powershell
New-Item -ItemType Directory -Force -Path $targetDir | Out-Null
curl.exe -L --fail --silent --show-error --retry 2 --output $targetPath $originalUrl
if (!(Test-Path -LiteralPath $targetPath) -or (Get-Item -LiteralPath $targetPath).Length -le 0) { throw "image download failed" }
```

## 失败处理

- 401/授权失效：提示用户在凌梦 AI 客户端重新启用 MCP，不能反复提交生图。
- 图片链接 404/过期：报告链接已失效，先询问用户是否重新生成；不要自动重生以免重复扣费。
- 部分图片下载失败：保留已成功保存的文件，逐项报告失败项；不要把失败项算作已保存。
- 只收到 MCP `image` 内容而没有可下载 URL 时，使用本地文件工具能直接取得内容就保存；否则说明只能在 Codex 中预览，不能伪造本地路径。

## 最终回复

报告实际使用的模型、目标数量、成功数量、尺寸/质量，以及本地文件路径。对于没有保存成功的图片，明确区分“生成成功”和“本地保存成功”。

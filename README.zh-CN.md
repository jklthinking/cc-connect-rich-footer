# cc-connect 富文本状态卡片页脚补丁

面向 [chenhg5/cc-connect](https://github.com/chenhg5/cc-connect) 的社区补丁，在飞书 **Card 2.0**（`card_mode = "rich"`）场景下，为助手回复提供更清晰的多行状态页脚。

![富文本卡片页脚示例](docs/screenshot.png)

## 视觉效果

在 `card_mode = "rich"` 且已打补丁时，卡片页脚为紧凑的多行信息（若关闭工作目录指示器则为两行）：

1. **本轮摘要** — 模型、推理强度、耗时  
   示例：`🧠 claude-opus-4-6 · 💪 强度 high · ⌛ 耗时 7.5s`
2. **上下文占用** — 已用/窗口 token、八格进度条、百分比  
   示例：`📝 上下文 12.7k/121.6k · 🟩 ▫️ ▫️ ▫️ ▫️ ▫️ ▫️ ▫️ (10%)`
3. **工作目录**（可选） — 仅当 `show_workdir_indicator = true` 时显示第三行

页脚文案走 cc-connect 现有 i18n（`language` 或首条消息自动检测），支持中文、英文等语言。

安静模式仍会抑制进行中的卡片刷新，但**最终**富文本卡片会在折叠面板中保留真实的思考与工具步骤。

补丁还延长了 Codex rollout 的 token 轮询窗口，以便在生成最终页脚前读到可靠的上下文用量。

## 环境要求

- [cc-connect](https://github.com/chenhg5/cc-connect) 源码，检出到某一发布标签（已在 **v1.5.0** / 提交 `17c61062` 上验证）
- 与上游 `go.mod` 一致的 Go 工具链（1.25+）
- 构建标签：`goolm no_web`（无嵌入 Web UI 的无头/agent 构建）

## 打补丁

在 cc-connect 仓库根目录：

```bash
git fetch --tags
git checkout v1.5.0   # 或你的目标标签
git apply --check richfooter.patch
git apply richfooter.patch
```

升级到较新标签后若发生冲突，可使用三路合并：

```bash
git apply --3way richfooter.patch
# 解决冲突后：git add -u && git commit
```

## 编译

```bash
go test -tags "goolm no_web" -count=1 ./core/... ./agent/codex/ ./platform/feishu/
go build -trimpath -tags "goolm no_web" -o cc-connect ./cmd/cc-connect
```

若需要 Web 管理界面，先构建前端（`make web`），并去掉 `no_web` 标签。

## 配置

在 `config.toml` 的 `[[projects]]` → `[display]` 中设置（详见上游 `config.example.toml`）：

| 配置项 | 作用 |
|--------|------|
| `card_mode = "rich"` | 启用飞书 Card 2.0 富文本卡片（本补丁生效前提）。 |
| `show_context_indicator` | 为 `true`（默认）时显示上下文行与进度条。 |
| `show_workdir_indicator` | 为 `true`（默认）时在页脚追加工作目录；设为 `false` 可保持两行页脚。 |

全局 `language`（`en`、`zh` 等）通过 cc-connect i18n 控制页脚语言。

## 升级 cc-connect

1. 在克隆仓库中检出新的上游标签。
2. 重新应用 `richfooter.patch`（`git apply` 或 `git apply --3way`）。
3. 按上文运行测试并重新编译。
4. 用新二进制替换当前安装。

**请勿对已打补丁的安装执行 `cc-connect update`**：该命令会用官方发行版覆盖二进制，从而去掉本页脚。

## 许可证

与上游 cc-connect 一致的 MIT 许可证。见 [LICENSE](LICENSE) 与 [NOTICE](NOTICE)。

## 上游

欢迎向 [chenhg5/cc-connect](https://github.com/chenhg5/cc-connect) 提交或关注 PR，以便官方合并此行为。

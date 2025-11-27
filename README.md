# 智能分段插件 (Smart Segmentation Plugin)

## 📝 功能说明

使用小型 LLM 智能切分回复文本，在语义自然的地方切分，保持原文内容不变，让回复更加灵活自然。

### 主要特点

- ✅ **智能切分**：使用 LLM 分析语义，在自然停顿处切分
- ✅ **保持原文**：只切分不修改，保持核心内容一致
- ✅ **情绪保留**：保留感叹号、问号、省略号、波浪号等情绪标点
- ✅ **灵活配置**：支持多种切分风格和段数上限
- ✅ **性能优化**：使用小模型快速处理

## 🚀 使用方法

### 1. 禁用内置分段（必须！）

编辑 `config/bot_config.toml`：

```toml
[response_splitter]
enable = false  # 必须禁用内置分段，否则会冲突
```

### 2. 启用插件

编辑 `plugins/smart_segmentation_plugin/config.toml`：

```toml
[plugin]
enabled = true

[segmentation]
enabled = true
style = "natural"    # 切分风格
min_length = 20      # 最小切分长度
max_segments = 8     # 最大段数
```

### 3. 配置小模型

确保在 `config/model_config.toml` 中配置了 `utils_small` 模型：

```toml
[model_task_config]
utils_small = ["gpt-4o-mini", "qwen-plus"]
```

## ⚙️ 配置说明

### 切分风格 (style)

- **natural**（推荐）：在自然停顿处切分，模拟真人发消息
- **conservative**：保守风格，尽量少切分，保持完整
- **active**：活跃风格，切分更细致

### 最小长度 (min_length)

小于此长度的文本不会切分，默认 20 字符。

### 最大段数 (max_segments)

切分段数上限，避免长文刷屏，默认 8 段。

## 📊 效果示例

### 原文（LLM 生成）

```
啧，哪有那么容易。且不说这种风格对场地的音响要求多高，光是找个不弹那些土味流行的贝斯手都比登天还难。与其跟一帮听不懂的人凑合，不如自己在家玩录音。
```

### 智能分段（插件切分）

```
段落1: "啧"
段落2: "哪有那么容易"
段落3: "且不说这种风格对场地的音响要求多高"
段落4: "光是找个不弹那些土味流行的贝斯手都比登天还难"
段落5: "与其跟一帮听不懂的人凑合，不如自己在家玩录音"
```

**特点**：
- ✅ 删除了句号和逗号（更自然）
- ✅ 保留了"啧"的情绪感
- ✅ 语义完整，每段都是完整想法

## 🔧 工作原理

1. **AFTER_LLM 阶段**：拦截 LLM 生成的文本
2. **智能分析**：使用小模型分析语义，决定切分位置
3. **标点处理**：删除切分点的普通标点（逗号、句号），保留情绪标点
4. **添加分隔符**：用 `|||SPLIT|||` 标记切分点
5. **Monkey Patch**：替换 `process_llm_response()` 识别分隔符并切分
6. **分批发送**：每个片段作为独立消息发送

## ⚠️ 注意事项

1. **必须禁用内置分段**：在 `config/bot_config.toml` 中设置 `response_splitter.enable = false`
2. **标点处理**：插件会删除切分点的逗号、句号，但保留感叹号、问号等情绪标点
3. **模型选择**：建议使用 gpt-4o-mini、qwen-plus 等小模型
4. **段数限制**：默认最多 8 段，避免长文刷屏

## 🐛 故障排查

### 插件未生效

1. 检查 `config.toml` 中 `enabled = true`
2. 确认已禁用内置 `response_splitter`（在 `config/bot_config.toml` 中）
3. 查看日志是否有 `✅ 已 patch process_llm_response` 信息

### 切分效果不理想

- 调整 `style` 参数：`natural`（推荐）、`conservative`（少切分）、`active`（多切分）
- 调整 `max_segments` 控制最大段数

### 段数过多

- 降低 `max_segments` 值（默认 8）
- 使用 `conservative` 风格

## 📄 许可证

GPL-v3.0-or-later

## 👤 作者

久远
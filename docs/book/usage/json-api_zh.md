# JSON 模式

所有 Walrus 客户端命令都提供 JSON 模式。在此模式下，原始 CLI 命令的所有命令行标志都可以用 JSON 格式指定。因此，JSON 模式简化了 CLI 的程序化访问。

例如，要存储一个 blob，运行：

```sh
walrus json \
    '{
        "config": "path/to/client_config.yaml",
        "command": {
            "store": {
                "files": ["README.md", "LICENSE"],
                "epochs": 100
            }
        }
    }'
```

或者，要根据 blob ID 读取一个 blob：

```sh
walrus json \
    '{
        "config": "path/to/client_config.yaml",
        "command": {
            "read": {
                "blobId": "4BKcDC0Ih5RJ8R0tFMz3MZVNZV8b2goT6_JiEEwNHQo"
            }
        }
    }'
```

除了使用"camelCase"而不是"kebab-case"之外，所有选项、默认值和命令都与"标准"CLI 模式相同。

`json` 命令还接受来自 `stdin` 的输入。

`json` 命令的输出本身也将是 JSON 格式，这同样是为了简化以程序化方式解析结果。例如，可以将 JSON 输出通过管道传递给 `jq` 命令进行解析和手动提取相关字段。
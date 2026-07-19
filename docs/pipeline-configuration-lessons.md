# Pipeline 配置错误经验

## `And` 组合识别的字段层级

### 结论

MaaFramework 的 `And` 组合识别要求 `all_of` 中的所有子识别都命中，当前节点才算识别成功。`all_of` 是必选字段，其位置取决于 Pipeline 版本。

本项目使用 Pipeline v2，因此正确结构是：

```json
{
    "recognition": {
        "type": "And",
        "param": {
            "all_of": [
                {
                    "recognition": {
                        "type": "OCR",
                        "param": {
                            "roi": [
                                1171,
                                207,
                                170,
                                80
                            ],
                            "expected": ".*可以.*"
                        }
                    }
                },
                {
                    "recognition": {
                        "type": "OCR",
                        "param": {
                            "roi": [
                                1171,
                                207,
                                170,
                                80
                            ],
                            "expected": ".*完成.*"
                        }
                    }
                }
            ]
        }
    }
}
```

字段层级如下：

- `recognition.type`：识别算法类型，这里是 `And`
- `recognition.param.all_of`：`And` 算法的子识别列表
- `all_of[*].recognition.type`：内联子识别的算法类型
- `all_of[*].recognition.param`：对应子识别的算法参数

### v1 与 v2 的区别

官方 `And` 章节的主要示例使用 Pipeline v1：

```json
{
    "recognition": "And",
    "all_of": [
        {
            "recognition": "OCR",
            "expected": "确认"
        }
    ]
}
```

同一章节末尾明确说明：Pipeline v2 时，将这些字段放到 `recognition.param` 中。因此不能只复制 v1 示例的 `all_of` 层级，再与 v2 的 `type` 写法混用。

以下两种写法都是无效的 v1/v2 混合结构。

外层 `And` 参数放错层级：

```json
{
    "recognition": {
        "type": "And",
        "all_of": []
    }
}
```

它使用了 v2 的 `recognition.type`，却把算法参数 `all_of` 放在了 v1 所在的层级。MPE 或 MaaFramework 无法按合法的 v2 `And` 配置解析它，可能显示或执行为默认识别类型 `DirectHit`。`DirectHit` 不进行图像识别，会直接命中，所以即使 OCR 内容不存在也会表现为识别成功。

内联子识别缺少 `recognition` 包装：

```json
{
    "type": "OCR",
    "param": {
        "expected": "确认"
    }
}
```

`all_of` 中的对象会作为一个内联节点片段交给普通 Pipeline 节点解析器。解析器读取的是该对象的 `recognition` 字段，不会把对象根部的 `type` 当作识别类型。缺少 `recognition` 时，子识别会继承默认类型 `DirectHit`，根部的 `type` 和 `param` 会被忽略。这正是本次所有 `And` 都直接命中的实际根因。

### `all_of` 元素

`all_of` 支持两类元素：

1. 节点名称字符串：复用该节点的识别算法和参数，MaaFramework v5.7 起支持。
2. 内联识别对象：它是一个包含 `recognition` 字段的节点片段，可使用 v1 或 v2，列表内也允许混用。

使用节点名称可以减少重复配置：

```json
{
    "检查图标": {
        "recognition": {
            "type": "TemplateMatch",
            "param": {
                "template": "icon.png"
            }
        }
    },
    "检查文字": {
        "recognition": {
            "type": "OCR",
            "param": {
                "expected": "确认"
            }
        }
    },
    "检查对话框": {
        "recognition": {
            "type": "And",
            "param": {
                "all_of": [
                    "检查图标",
                    "检查文字"
                ]
            }
        }
    }
}
```

### 其他参数

- `box_index`：选择第几个子识别的识别框作为 `And` 节点的输出框，默认 `0`
- `sub_name`：为内联子识别设置别名，后续子识别可以通过 `roi: sub_name` 使用前一个结果区域；只在当前 `And` 节点内有效
- `all_of` 为空或缺失不符合协议要求，至少应提供一个有效子识别

### 本次错误与改进

本次先把原本正确的 `recognition.param.all_of` 误判为错误；恢复外层结构后，又漏掉了内联子识别所需的 `recognition` 包装。外层 `And` 虽然能解析，但六个 OCR 子项都继承成 `DirectHit`，所以任何画面都会满足 `all_of`。

后续检查规则：

1. 阅读算法章节时，同时检查该章节的版本说明，不能只看主体示例。
2. 看到 `recognition` 是对象且包含 `type`，即可判定为 v2；算法参数必须放在同一对象的 `param` 中。
3. 不混用 v1 的根级算法参数与 v2 的 `type`/`param` 包装。
4. `all_of`/`any_of` 内联对象必须检查 `recognition` 包装，不能只写根级 `{ type, param }`。
5. 修改后同时执行 JSON 解析、`pnpm check:schema` 和 `pnpm check:maa`，但要注意当前 schema 对内联对象约束较松，静态检查通过不代表识别类型正确。
6. 组合识别还要进行实际负例测试：当任一子识别不命中时，`And` 必须不命中；仅测试成功场景无法发现 `DirectHit` 回退。

### 参考

- [MaaFramework Pipeline 协议：And](https://maafw.com/docs/3.1-PipelineProtocol#and)
- [MaaFramework Pipeline 协议：Pipeline v2](https://maafw.com/docs/3.1-PipelineProtocol#pipeline-v2)
- [MaaFramework PipelineParser 源码](https://github.com/MaaXYZ/MaaFramework/blob/main/source/MaaFramework/Resource/PipelineParser.cpp)
- [MaaPipelineEditor 编译器](https://mpe.codax.site/docs/guide/trait/parser.html)

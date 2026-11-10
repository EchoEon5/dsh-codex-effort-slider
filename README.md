# DSH Codex effort slider

给 DeepSeek-Harness 用的 Codex 风格推理强度滑块，**档位随模型自适应**。

上游 [DSH-Codex-reasoning-effort-slider](https://github.com/bakabaicai/DSH-Codex-reasoning-effort-slider) 把 Codex 桌面版的滑块 1:1 搬了过来，但只认 `off / low / high / max` 四档。遇到档位更多的模型（比如声明了 `minimal / medium / xhigh` 的）就会被截断，拉不到模型真正的最高档。

本插件修掉了这个问题：**模型声明几档，滑块就画几格；最高那一档统一显示为 Ultra。**

## 特性

- **档位自适应**：不再过滤白名单，档位数完全取自模型自己声明的 `reasoning.efforts`
- **顺序稳定**：按 `off → minimal → low → medium → high → xhigh → max` 排列；模型自带的未知档位排在已知档位之后，不会被丢弃
- **Ultra 跟随最高档**：3 档模型的 `high`、6 档模型的 `max`、带自定义档位模型的最后一档，都显示为 Ultra，流光与粒子动画也只在拉到该档时触发
- **单档模型不误标**：只有一个档位的模型（通常只有 `off`）不显示 Ultra，避免误以为在烧额度
- **左端无残影**：最低档不一定是 `off`（有的模型从 `low` 起步），填充色块按「是否处于最低档」归零，不再残留半格蓝色

除档位逻辑外，界面、动画、Fast 开关、模型切换、运行中挂起提交、弹层定位、键盘操作全部沿用上游实现。

## 兼容性

- 需要 DeepSeek-Harness **0.2.0-rc.2 及以上**
- 与上游 `@local/dsh-codex-reasoning-effort-slider` **不能同时启用**：两者都占用 `conversation.input.model` 这个 single 插槽，只会有一个生效

## 安装

### 从 GitHub 安装

在 Harness 的 **插件 → 添加插件** 里粘贴仓库地址：

```
https://github.com/EchoEon5/dsh-codex-effort-slider
```

### 从本地文件夹安装

```bash
# 1) 放进插件目录
cp -r dsh-codex-effort-slider ~/.dsh/plugins/

# 2) 装进 web profile（自动登记 dependency 与 bundle 层）
dsh plugin --profile web add ~/.dsh/plugins/dsh-codex-effort-slider

# 3) 重启引擎
```

### 从压缩包安装

```bash
tar -xzf dsh-codex-effort-slider.tar.gz -C ~/.dsh/plugins/
dsh plugin --profile web add ~/.dsh/plugins/dsh-codex-effort-slider
```

安装后点开输入框下方的模型名，滑块格数应当与该模型的思考档位数一致。

## 卸载

```bash
dsh plugin --profile web remove dsh-codex-effort-slider
```

## 实现说明

改动集中在 `client.js` 的三个函数，其余代码未动：

| 函数 | 作用 |
| :--- | :--- |
| `rankEfforts(list)` | 按已知升级顺序排序，未知档位保持原顺序排在末尾 |
| `topOf(list)` | 返回最高档 id；档位不足两档时返回 `undefined`，据此关闭 Ultra 动画 |
| `effortLabel(id, list)` | 最高档显示 `Ultra`，其余用已知展示名或模型自己声明的 name |

原先的两处硬编码：

```js
// 改前：只保留白名单里的四档
const efforts = (model?.reasoning?.efforts || [])
  .filter(e => order.includes(e.id))
  .sort((a, b) => order.indexOf(a.id) - order.indexOf(b.id));
const ultra = displayed === 'max';

// 改后：模型给几档就画几档
const efforts = rankEfforts(model?.reasoning?.efforts || []);
const ultra = topEffort !== undefined && displayed === topEffort;
```

## 署名

上游插件由 [Nachoneko_miao](https://github.com/bakabaicai) 编写，MIT 许可。本分支仅修改档位自适应逻辑，原始界面、动画与交互全部保留，原作者版权声明按 MIT 要求保留在 [LICENSE](LICENSE) 中。

## License

[MIT](LICENSE)

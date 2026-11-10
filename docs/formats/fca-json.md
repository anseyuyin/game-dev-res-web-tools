# 📄 `.fca.json` 配置格式规范

> 上级索引：[工具索引](../工具索引.md) ｜ 格式标识：`frame-sequence-character-assembler`
> 使用该格式的工具：[序列帧动画角色组装器](../tools/frame-sequence-character-assembler.md) ｜ [`tools/frame-sequence-character-assembler.html`](../../tools/frame-sequence-character-assembler.html)
> 格式版本：**v1**（`version: 1`）

**一句话**：一个 `.fca.json` 描述一个 2D 角色——每个动画用哪张序列帧图、怎么切帧、多快播、循环还是播完回退到哪个动画，以及它在世界坐标系里的原点位置与缩放。

本文是**格式本身的规范**（字段语义、默认值、加载容错、引擎侧读取参考实现），不含编辑器的界面与工作流说明；后者见[工具文档](../tools/frame-sequence-character-assembler.md)。

---

## 1. 适用范围与设计约束

| 项 | 约定 |
| --- | --- |
| 粒度 | **一个文件 = 一个角色**（一组动画 + 一张默认动画） |
| 数据来源 | 一张图 = 一个动画；一图多段动画、跨图合并成图集均**不在本格式范围内** |
| 坐标系统 | **世界坐标 Y 向上**，原点由使用方决定（通常为角色脚底中心）；`transform.x/y` 即该动画帧矩形「原点」的世界坐标 |
| 单位 | **1 世界单位 = 1 图素**（图片原始像素），不做 DPI 换算 |
| 帧原点 | 帧矩形的归一化锚点 `pivot`，v1 固定 `0.5 / 0.5`（单元格中心），字段写入但不由编辑器修改 |
| 变换 | 只有位移 + **等比缩放**；不支持旋转、剪切、分轴缩放 |
| 状态关系 | **单目标**状态机：`loop: false` 的动画播完跳 `next` 一次，没有条件、事件、参数化过渡 |
| 编码 | UTF-8、无 BOM、2 空格缩进、标准 JSON（**不能有注释**） |
| 数值精度 | `transform`、`editor` 的浮点字段保存时保留 **2 位小数** |

---

## 2. 文件名与目录位置

```
<工程目录>/
├─ hero.fca.json        # 本格式的配置文件（可多个，一个角色一份）
└─ sprites/             # 与配置同级的图片目录
   ├─ idle.png
   ├─ attack.png
   └─ death.png
```

- 文件名约定 `<工程名>.fca.json`，扩展名中的 `.fca` 是本格式标识，**不要只写 `.json`**（工具靠这个后缀发现配置）。
- `animations[].image` 与 `assets[].path` 是**相对 `.fca.json` 所在目录**的路径，工具固定写 `sprites/<图片文件名>`。
- 不写绝对路径、不写 `../`（工具不生成，部分引擎沙箱也不允许）。

---

## 3. 完整示例

```json
{
  "app": "frame-sequence-character-assembler",
  "version": 1,
  "editorVersion": "1.0.0",
  "name": "hero",
  "savedAt": "2026-11-10T08:30:00.000Z",
  "defaultAnimation": "idle",
  "assets": [
    { "path": "sprites/idle.png", "width": 512, "height": 512, "bytes": 48213 },
    { "path": "sprites/attack.png", "width": 768, "height": 512, "bytes": 71330 },
    { "path": "sprites/death.png", "width": 576, "height": 384, "bytes": 51022 }
  ],
  "animations": [
    {
      "id": "idle",
      "name": "idle",
      "image": "sprites/idle.png",
      "rows": 4,
      "cols": 8,
      "frameCount": 32,
      "order": "col",
      "fps": 12,
      "loop": true,
      "next": null,
      "pivot": { "x": 0.5, "y": 0.5 },
      "transform": { "x": 0, "y": 0, "scaleX": 1, "scaleY": 1 }
    },
    {
      "id": "attack",
      "name": "attack",
      "image": "sprites/attack.png",
      "rows": 4,
      "cols": 6,
      "frameCount": 20,
      "order": "col",
      "fps": 18,
      "loop": false,
      "next": "idle",
      "pivot": { "x": 0.5, "y": 0.5 },
      "transform": { "x": 3, "y": -2, "scaleX": 1.04, "scaleY": 1.04 }
    },
    {
      "id": "death",
      "name": "death",
      "image": "sprites/death.png",
      "rows": 3,
      "cols": 6,
      "frameCount": 16,
      "order": "col",
      "fps": 10,
      "loop": false,
      "next": null,
      "pivot": { "x": 0.5, "y": 0.5 },
      "transform": { "x": 0, "y": -6, "scaleX": 1, "scaleY": 1 }
    }
  ],
  "editor": {
    "zoom": 1,
    "panX": 0,
    "panY": 0,
    "showGrid": true,
    "ghostMode": "first-frame"
  }
}
```

### 3.1 最短可用配置

除 `app` / `version` / `animations` 外都可省略，省略字段按 [§5 加载容错](#5-加载容错与默认值) 取默认值（工具保存时会补齐全部字段）：

```json
{
  "app": "frame-sequence-character-assembler",
  "version": 1,
  "defaultAnimation": "idle",
  "animations": [
    {
      "id": "idle",
      "image": "sprites/idle.png",
      "rows": 1,
      "cols": 8,
      "frameCount": 8,
      "order": "col",
      "fps": 12,
      "loop": true
    }
  ]
}
```

---

## 4. 字段参考

### 4.1 顶层字段

| 字段 | 类型 | 必填 | 写入 | 说明 |
| --- | --- | --- | --- | --- |
| `app` | string | ✅ | 固定 | 恒为 `"frame-sequence-character-assembler"`，用于识别文件归属。非该值时工具会弹确认框，仍允许强行加载 |
| `version` | number | ✅ | 固定 | **配置格式**版本，当前 **`1`**。与读取方版本不一致时提示后「按当前版本尽力加载」，不阻断 |
| `editorVersion` | string | ❌ | 编辑器版本 | 写入该文件的编辑器版本（如 `"1.0.0"`），与工具内 ❓ 使用说明显示一致；**仅记录，不参与格式校验**，可安全忽略 |
| `name` | string | ✅ | 工程名 | 工程名，**与配置文件名（去 `.fca.json`）一致**；读取时一律以实际文件名为准，该字段仅作冗余记录 |
| `savedAt` | string | ✅ | ISO 8601 | 写入时间，**仅用于展示**，不参与逻辑 |
| `defaultAnimation` | string \| null | ✅ | 默认动画 id | 启动 / 停止时播放它；指向不存在的 id 时回退为 `animations[0].id` |
| `assets` | object[] | ✅ | 由 `animations` 派生 | 图片清单，与 `animations` 一一对应（同序），仅用于加载校验（尺寸/存在性），**只提示不阻断** |
| `animations` | object[] | ✅ | 动画列表 | 数组顺序 = 列表显示顺序 = **绘制层级（靠后者在上）**；至少要有 1 项才有意义 |
| `editor` | object | ❌ | 编辑器视图 | 编辑器私有状态，**引擎可直接忽略** |

### 4.2 `assets[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `path` | string | 相对工程目录的图片路径，等于对应动画的 `image` |
| `width` | number | 图片像素宽，加载时与磁盘文件比对，不符标黄提示 |
| `height` | number | 图片像素高，同上 |
| `bytes` | number | 文件字节数，仅用于展示 / 体积统计 |

### 4.3 `animations[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | ✅ | **状态键，全局唯一且稳定**。工具由图片文件名派生（非法字符替换为 `-`，重名追加 `-2`、`-3`…）。引擎侧的状态机、`next`、`defaultAnimation` 都引用它，发布后不要改 |
| `name` | string | ✅ | **冗余记录**：恒等于 `image` 的文件名（去扩展名）。读取时一律以 `image` 为准，缺失才回退本字段；工具加载 / 保存 / 撤销都会按 `image` 重新校正 |
| `image` | string | ✅ | 序列帧图相对路径，如 `sprites/attack.png` |
| `rows` | integer | ✅ | 图内网格行数，≥ 1 |
| `cols` | integer | ✅ | 图内网格列数，≥ 1 |
| `frameCount` | integer | ✅ | 实际使用帧数，`1 ~ rows × cols`（末行 / 末列可以不满）；超过上限会被夹到 `rows × cols` |
| `order` | `"col"` \| `"row"` | ✅ | 帧号走法：`"col"` 列优先（默认，从上到下再换列）、`"row"` 行优先（从左到右再换行）。**缺省按 `"row"` 加载**（兼容早期没有该字段的配置） |
| `fps` | number | ✅ | 播放帧率，`1 ~ 120`，支持 0.5 步进；`fps` 决定每帧时长 `1 / fps` |
| `loop` | boolean | ✅ | `true` 循环（永不结束，`next` 不生效）；`false` 播完触发一次 `next` |
| `next` | string \| null | ✅ | 结束跳转目标动画 `id`；`null` = **停在末帧**。`loop: true` 时字段保留但运行时无效 |
| `pivot` | object | ✅ | `{ x, y }` 归一化帧原点，v1 恒为 `0.5 / 0.5`，只写入不编辑 |
| `transform` | object | ✅ | 见下表；缺省的 `x/y` 视为 `0`，`scaleX/scaleY` 视为 `1` |

`transform` 子字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `x` | number | 帧矩形**原点**的世界 X 坐标（像素，右为正） |
| `y` | number | 帧矩形**原点**的世界 Y 坐标（像素，**上为正**；角色站在地面上时通常为 0，死亡等下沉动作为负） |
| `scaleX` | number | 缩放倍率，`> 0`。与 `scaleY` **恒等**（保留双字段是为了兼容按 X/Y 分轴读取的引擎） |
| `scaleY` | number | 同上；若读到两值不一致（手改 / 旧配置），工具取两者均值后统一写入 |

### 4.4 `editor`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `zoom` | number | 上次视图缩放，夹在 `0.02 ~ 8` |
| `panX` / `panY` | number | 上次视图平移（屏幕像素） |
| `showGrid` | boolean | 是否显示世界网格，缺省 `true` |
| `ghostMode` | `"first-frame"` \| `"play"` | 幽灵动画是停在首帧还是同步播放，缺省 `"first-frame"` |

> 整块可省略。字段只影响编辑器观感，**引擎读到时应当忽略**。

---

## 5. 加载容错与默认值

读取 `.fca.json` 的推荐流程：`app` 不匹配 → 询问是否强行加载；`version` 不匹配 → 提示后尽力加载；随后逐项校验并把问题**汇总成一条清单**，允许「忽略并继续」。

| 情况 | 读取方应如何处理 |
| --- | --- |
| `app` 不是约定值 | 提示「缺少本工具标识」，可选择继续 |
| `version` 与自身不同 | 提示版本不一致，按当前版本尽力加载（未知字段忽略） |
| `animations` 为空 / 缺失 | 视为空工程，不加载 |
| `rows` / `cols` 非法（0、负数、非数字） | 夹到 `≥ 1` |
| `frameCount` 越界 | 夹到 `1 ~ rows × cols` |
| `order` 缺失或非法 | 按 `"row"` 处理 |
| `fps` 缺失或越界 | 缺省 `12`，夹到 `1 ~ 120` |
| `loop` 缺失 | 视为 `true` |
| `next` 指向不存在的 `id` | 降级为 `null`（停在末帧）并标黄提示 |
| `loop: true` 且有 `next` | 保留字段，运行时不跳转（UI 置灰，切回单次即恢复） |
| `pivot` 缺失 / 非数字 | 分量取 `0.5` |
| `scaleX` / `scaleY` 不一致 | 取均值，夹到 `0.01 ~ 100`，保留 2 位小数 |
| `transform` 缺失 | 视为 `{ x: 0, y: 0, scaleX: 1, scaleY: 1 }` |
| `defaultAnimation` 指向不存在的 `id` | 回退为 `animations[0].id` |
| `defaultAnimation` 为 `null` | 允许，表示未设置（工具内会用列表首个动画兜底） |
| 图片缺失 | 该条目标记为缺图，用 `assets` 记录的尺寸画占位框；不得抛异常中断整个配置 |
| 图片尺寸与 `assets` 记录不符 | 以磁盘图片实际尺寸为准，提示不一致 |
| `assets` 缺失 | 视为空清单，跳过校验 |
| `savedAt` 非法 | 忽略，用当前时间 |

> 所有容错都是**只修正内存中的运行值**；除 `name` 会被 `image` 校正外，其余修正只在用户保存时才写回文件。

---

## 6. 切帧算法（帧号 → 像素矩形）

单元格尺寸 = `图片宽 / cols`、`图片高 / rows`，**允许小数**（不整除时不要取整，否则整张图会逐渐错位）。

```js
// width / height 为图片原始像素尺寸
function frameRect(anim, width, height, index) {
  const fw = width / Math.max(1, anim.cols);
  const fh = height / Math.max(1, anim.rows);
  const i = Math.min(Math.max(index | 0, 0), anim.frameCount - 1);

  let col, row;
  if (anim.order === 'row') {          // 行优先：从左到右，再换行
    col = i % anim.cols;
    row = Math.floor(i / anim.cols);
  } else {                             // 列优先（默认）：从上到下，再换列
    row = i % anim.rows;
    col = Math.floor(i / anim.rows);
  }
  return { sx: col * fw, sy: row * fh, sw: fw, sh: fh };
}
```

要点：

- 帧号从 **0** 开始，`0 ~ frameCount - 1`。
- **默认是列优先**：先取满第 1 列的 `rows` 格，再换第 2 列。若美术出图是「从左到右一行行排」，必须把 `order` 写成 `"row"`，否则播放顺序会变成竖着一列列跳。
- 未使用的格子（`frameCount < rows × cols` 时位于编号尾部）在编辑器切分预览里标红，运行时永远不会被取到。

---

## 7. 时间 → 帧号 与结束语义

每帧时长 `1 / fps`，动画总时长 `frameCount / fps`。

```js
function frameIndex(anim, t) {                    // t = 动画已播放秒数
  const n = Math.max(1, anim.frameCount);
  if (anim.loop) return ((Math.floor(t * anim.fps) % n) + n) % n;
  return Math.min(Math.max(Math.floor(t * anim.fps), 0), n - 1);
}
```

状态推进（与编辑器预览实现一致）：

```
t += dt
循环动画（loop = true）：
    t >= total 时 t %= total，永远不触发结束
非循环动画（loop = false）：
    t >= total 时末帧已完整显示满一帧时长，触发一次结束：
        next 存在      → 切到目标动画，t = 0，继续播放
        next 为 null   → t = total，停在末帧，停止推进
```

- 结束判定用 `t >= frameCount / fps`（**不是** `(frameCount - 1) / fps`），保证末帧有完整的 `1/fps` 停留时间。
- 跳转只在进入动画时由使用方触发一次；如果需要「攻击中被打断 → 切 hurt」，那是引擎侧的逻辑，不在本格式内。
- 回退链可能出现 `A → B → A → B` 死循环（配置合法但语义有病），编辑器会检测并停止自动跳转、**不修改配置**；引擎侧自行加保护（例如同一状态跳转计数或最小持续时间）。

---

## 8. 绘制与坐标换算

约定：世界坐标 **Y 向上**；canvas 是 **Y 向下**，因此渲染时做一次翻转换算。

```js
// 世界坐标 → 画布坐标（originX / originY = 世界原点在画布上的位置；zoom = 视图缩放）
const toScreenX = (wx) => originX + wx * zoom;
const toScreenY = (wy) => originY - wy * zoom;

function drawFrame(ctx, img, anim, index) {
  const r = frameRect(anim, img.width, img.height, index);
  const tf = anim.transform;
  const dw = r.sw * tf.scaleX;
  const dh = r.sh * tf.scaleY;

  // pivot 在单元格内的归一化位置（v1 = 0.5/0.5，即中心）
  const px = (anim.pivot ? anim.pivot.x : 0.5) * dw;
  const py = (anim.pivot ? anim.pivot.y : 0.5) * dh;

  // 世界坐标下帧矩形的左上角
  const left = tf.x - px;
  const top  = tf.y + py;

  ctx.imageSmoothingEnabled = false;   // 像素风关平滑；非像素风设为 true
  ctx.drawImage(
    img, r.sx, r.sy, r.sw, r.sh,
    toScreenX(left), toScreenY(top), dw * zoom, dh * zoom
  );
}
```

等价说法（引擎常见写法）：

```
帧中心的世界坐标 = (transform.x, transform.y)
canvas 左上角   = (transform.x - dw / 2, originScreenY - transform.y - dh / 2)
```

---

## 9. 引擎侧最小读取步骤

1. `JSON.parse` 文本，校验 `app` / `version`，按 §5 做字段夹取与断链修复。
2. 以 `.fca.json` 所在目录为基准，按 `animations[].image` 加载图片，建 `id → { img, anim }` 表。
3. 初始化状态：`current = defaultAnimation || animations[0].id`，`t = 0`。
4. 每帧：`t += dt` → `frameIndex` 取帧 → `frameRect` 取源矩形 → 按 §8 绘制。
5. 非循环动画 `t >= frameCount / fps` 时按 `next` 迁移，并把 `t` 归零。
6. 外部触发的动作切换（玩家输入等）只需改 `current` 与 `t = 0`，配置无需变更。

按需同步的附加能力：多动画叠放时按 `animations` 数组顺序绘制（后者在上）；做像素风渲染时关闭 `imageSmoothing`；UI 展示名一律取 `name`（已等于文件名）。

---

## 10. JSON Schema（draft-07）

可用于编辑器补全 / CI 校验。注意 `frameCount ≤ rows × cols` 与 `id` 唯一性**无法**用标准 Schema 表达，需自行追加校验。

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "fca.json",
  "type": "object",
  "required": ["app", "version", "animations"],
  "properties": {
    "app": { "const": "frame-sequence-character-assembler" },
    "version": { "type": "integer", "minimum": 1 },
    "editorVersion": { "type": "string" },
    "name": { "type": "string" },
    "savedAt": { "type": "string" },
    "defaultAnimation": { "type": ["string", "null"] },
    "assets": { "type": "array", "items": { "$ref": "#/definitions/asset" } },
    "animations": {
      "type": "array",
      "minItems": 1,
      "items": { "$ref": "#/definitions/animation" }
    },
    "editor": { "$ref": "#/definitions/editor" }
  },
  "definitions": {
    "asset": {
      "type": "object",
      "required": ["path", "width", "height"],
      "properties": {
        "path": { "type": "string", "minLength": 1 },
        "width": { "type": "integer", "minimum": 1 },
        "height": { "type": "integer", "minimum": 1 },
        "bytes": { "type": "integer", "minimum": 0 }
      }
    },
    "animation": {
      "type": "object",
      "required": ["id", "image", "rows", "cols", "frameCount", "fps", "loop"],
      "properties": {
        "id": { "type": "string", "minLength": 1 },
        "name": { "type": "string" },
        "image": { "type": "string", "minLength": 1 },
        "rows": { "type": "integer", "minimum": 1 },
        "cols": { "type": "integer", "minimum": 1 },
        "frameCount": { "type": "integer", "minimum": 1 },
        "order": { "enum": ["col", "row"] },
        "fps": { "type": "number", "minimum": 1, "maximum": 120 },
        "loop": { "type": "boolean" },
        "next": { "type": ["string", "null"] },
        "pivot": {
          "type": "object",
          "required": ["x", "y"],
          "properties": {
            "x": { "type": "number", "minimum": 0, "maximum": 1 },
            "y": { "type": "number", "minimum": 0, "maximum": 1 }
          }
        },
        "transform": {
          "type": "object",
          "required": ["x", "y", "scaleX", "scaleY"],
          "properties": {
            "x": { "type": "number" },
            "y": { "type": "number" },
            "scaleX": { "type": "number", "exclusiveMinimum": 0 },
            "scaleY": { "type": "number", "exclusiveMinimum": 0 }
          }
        }
      }
    },
    "editor": {
      "type": "object",
      "properties": {
        "zoom": { "type": "number" },
        "panX": { "type": "number" },
        "panY": { "type": "number" },
        "showGrid": { "type": "boolean" },
        "ghostMode": { "enum": ["first-frame", "play"] }
      }
    }
  }
}
```

---

## 11. 常见错误

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 播放顺序竖着跳 | 配置里没有 `order` 或误写 `"row"`，而图片是列优先排布 | 显式写 `"order": "col"` |
| 帧边缘有可辨认的下一帧残影、逐渐偏移 | 单元格尺寸被取整（`floor(W / cols)`） | 保留小数，用 `W / cols` |
| 动作「消失」或停不住 | 非循环动画 `next` 也指向了别的动画 | 需要停住的动作把 `next` 设为 `null` |
| 配了 `next` 但不跳转 | 该动画 `loop: true` | 改成 `loop: false` |
| 加载报「结束目标不存在」 | `next` 撞上被删除 / 改名的 `id` | 同步改 `next`，或留 `null` |
| 显示名和文件名对不上 | 手改了 `name` | `name` 会被 `image` 覆盖，改显示名只能改图片文件名 |
| 缩放看起来被「拉扁」 | 手改出 `scaleX ≠ scaleY` | 本格式不支持分轴缩放，工具加载时取均值 |
| 动画整体上下颠倒 / 位置反了 | 引擎按 Y 向下解释 `transform.y` | 渲染时取负（见 §8） |
| 找不到配置 | 只存成 `xxx.json` | 后缀必须是 `.fca.json` |
| 图片加载不到 | `image` 用了绝对路径或上传到了别的目录 | 用相对 `.fca.json` 的路径，且文件真实存在于工程目录下 |

---

## 12. 版本与兼容策略

- `version` 只在**破坏性变更**时递增；新增可选字段不递增，读取方忽略未知字段、写入方不保证保留。
- `editorVersion` 是纯记录字段（编辑器版本），既不参与 `version` 兼容判断，也不影响加载；旧文件没有该字段同样合法。
- 当前唯一的兼容分支：`order` 缺失 → 按 `"row"`（行优先）解释，保证早期配置的播放顺序不变。
- 读取方与写入方的版本不一致时，**不拒绝加载**，只提示并尽力解析；保存时总是写自己支持的 `version`。
- 已废弃 / 未启用的预留字段：`pivot`（写入但不可编辑；未来若支持九宫格锚点，语义仍为归一化帧原点）。

---

## 13. 该格式**不**负责的事

- 角色属性（血量、攻击力）、碰撞盒、帧事件、音效挂点、技能参数。
- 动画融合、混合树、参数化过渡、Any State、过渡时长与打断规则。
- 图集打包 / 资源压缩 / 转码：图片原样引用，`assets` 只记录尺寸与字节数。
- 一张图内多段动画、多图合并、逐帧导出 PNG（属于后续可选增强，不在 v1 格式内）。

---

[← 返回工具文档](../tools/frame-sequence-character-assembler.md) ｜ [← 返回工具索引](../工具索引.md)

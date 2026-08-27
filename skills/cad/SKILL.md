---
name: cad
description: >
  Unified CAD skill (建模 + 预览 + 读DWG). Trigger when user says any of:
  建模, 做零件, 改零件, 改STEP, 出STEP, 生成STEP, 测量装配, 配合检查, 做装配;
  打开预览, 看模型, 看STEP, CAD Explorer, Explorer, 打开Explorer, 预览STEP;
  读DWG, 转换DWG, 图纸转PDF, DWG转PDF, DWG转SVG, 报价读图, 客户图纸;
  create STEP, modify STEP, build123d, open explorer, view STEP, convert DWG,
  DWG to PDF, measure assembly, mate check, CAD review link.
  Do NOT use for pure render art, CAM, FEA certification, BIM, or robot file generation
  (those stay in urdf/srdf/sdf skills — this skill only views them).
---

# CAD — 建模 / 看图 / 读DWG（唯一入口）

Explorer 已并入本 skill（`scripts/explorer`）。**不要再加载 `cad-explorer`**——该 skill 已删除。

## 1. 触发词 / When to use

### 必须加载本 skill（任意一条命中即可）

**建模 / 改模型**

- 建模、做零件、改零件、做装配、改装配
- 出 STEP、改 STEP、生成 STEP、导出 STEP/STP
- 测量装配、配合检查、对位、量尺寸、检查干涉
- `@cad`、build123d、parametric part/assembly
- create/modify STEP, measure assembly, mate check

**看图 / 预览（CAD Explorer）**

- 打开预览、看模型、看 STEP、预览 STL/DXF
- CAD Explorer、Explorer、打开 Explorer、出预览链接
- open explorer, view STEP, CAD review link, preview model

**读 DWG / 报价读图**

- 读 DWG、转换 DWG、图纸转 PDF、DWG 转 PDF/SVG/CSV
- 报价读图、客户图纸、看图纸
- convert DWG, DWG to PDF/SVG/CSV, quote drawing intake

### 不要用本 skill

- 纯概念渲染 / 插画（无 CAD 几何需求）
- CAM 刀路、工程认证结论、FEA 结论、建筑 BIM
- **生成** URDF / SRDF / SDF（用 `$urdf` / `$srdf` / `$sdf`）；本 skill 只负责在 Explorer 里**查看**这些文件
- SendCutSend / 钣金报价专用流程（用 `$sendcutsend`），除非同时需要 STEP 建模或 Explorer
- 已删除的 `cad-explorer` skill——不要再找、不要再 symlink

## 2. 三条路径

| 路径 | 何时用 | 第一条命令（相对本 skill 目录） |
|------|--------|--------------------------------|
| **建模** | 新建/改零件装配、出 STEP、测量配合 | `python scripts/step ...` |
| **看图** | 打开已有 STEP/STL/DXF/URDF 等预览 | `npm --prefix scripts/explorer run dev:ensure -- --file <path>` |
| **读DWG** | 客户/报价 `.dwg` → PDF/SVG/CSV | Studio 绝对路径见下表 DWG 节（经 mini QCAD） |

可串联：建模生成 STEP 后 → 看图出预览链接。先判定走哪条，再执行。

## 3. Path Modeling（建模）

### 工具

在本 skill 目录下：

```bash
python scripts/step ...
python scripts/inspect ...
python scripts/render ...
python scripts/dxf ...
```

`python scripts/<tool> --help` 看参数。用当前项目的 Python；**Mac mini 上必须** `/opt/homebrew/opt/python@3.12/bin/python3.12`（不要裸 `python3`）。

### 默认假设

未特别说明时：单位 mm；原点在主体中心（配合面另议）；基面 XY，挤出 +Z；输出封闭实体；STEP 为单实体或带标签装配；薄壁塑料约 2–3 mm；圆角约 1–3 mm；M3/M4/M5 通孔 3.4 / 4.5 / 5.5 mm。缺关键信息才问一句，否则写明假设后继续。

自然语言规格即可，不要向用户要 JSON。见 `references/natural-language-specs.md`。

### 必做流程

1. **分类** — 新零件 / 装配 / 改源码 / 检查 / 测量 / 渲染 / 次要导出
2. **按需加载 references** — 见文末 Progressive references
3. **CAD brief** — 尺寸、单位、特征、路径、假设、验收点
4. **先规划再写码** — 参数、标签、包围盒、配合基准
5. **只改源码** — build123d Python + `gen_step()`；禁止手改生成的 STEP
6. **生成** — `scripts/step`；仅直接导入 STEP 时用 `--kind part|assembly`；仅用户明确跳过预览时加 `--skip-explorer`
7. **校验** — `scripts/inspect refs --facts --planes --positioning`，再按需 `measure` / `mate` / `frame` / `diff`
8. **看图** — 生成或改过 `.step/.stp/.stl/.3mf/.dxf` 后，除非用户跳过，走 Path Explorer
9. **渲染** — 仅用户要求、Explorer 不可用、或视觉仍不清时用 `scripts/render`
10. **修复环** — 最小源码改动 → 再生成 → 重跑失败项

### 硬规矩

- STEP/STL/3MF/GLB/DXF 与 Explorer 附属文件都是派生产物；有 Python 生成器时必须跑 `scripts/step`
- 装配位姿在源码里做；`inspect mate` 只做校验
- 不要对大二进制用 `git diff` 比几何
- 只汇报实际跑过的检查

## 4. Path Explorer（看图）

所有 Explorer 路径相对**本 skill 目录**的 `scripts/explorer`（**禁止** `../cad-explorer`）。

**支持：** `.step` `.stp` `.stl` `.3mf` `.dxf` `.urdf` `.srdf` `.sdf`
输入必须是已存在的明确路径。

### 启动预览

```bash
npm --prefix scripts/explorer run dev:ensure -- --file path/to/model.step
```

带工作区根：

```bash
npm --prefix scripts/explorer run dev:ensure -- \
  --workspace-root /path/to/workspace \
  --file path/to/model.step
```

前台 Vite（仅手动调试）：

```bash
npm --prefix scripts/explorer run dev
```

`dev:ensure` 会复用已有匹配扫描根的本地服务，或在空闲端口起 Vite。**把打印出的 URL 回给用户。**

启动失败：如实报告，继续用 CLI inspect / 非 GUI 校验。

### MoveIt2（SRDF 交互）

仅当用户需要交互式 IK/路径规划时启动——普通预览链接不需要。

在本 skill 目录：

```bash
scripts/moveit2_server/setup.sh
scripts/moveit2_server/check-moveit2-server.sh
scripts/moveit2_server/run-moveit2-server.sh
```

Default WebSocket: `ws://127.0.0.1:8765/ws`。可用 `EXPLORER_MOVEIT2_WS_URL` 或浏览器 `?moveit2Ws=`。详情：`references/moveit2-server.md`。

### 环境变量

```text
EXPLORER_PORT
EXPLORER_PORT_END
EXPLORER_ROOT_DIR
EXPLORER_DEFAULT_FILE
EXPLORER_WORKSPACE_ROOT
EXPLORER_GITHUB_URL
EXPLORER_MOVEIT2_WS_URL
```

优先 `dev:ensure`；除非用户要求，不要停掉已有 Explorer。

## 5. Path DWG（读图）

任何 `.dwg`（报价图、客户 CAD、BOM 附件）：**一律经 Mac mini 上的 QCAD 转换**。
**禁止在 Mac Studio 本机跑 QCAD。**

Studio 权威期望路径：

```bash
/Users/Leo/.openclaw/workspace/projects/fadior/Fadiorteam/projects/tools/quote-tool/scripts/mini_qcad_dwg.sh dwg2pdf  INPUT.dwg [OUTPUT.pdf]
/Users/Leo/.openclaw/workspace/projects/fadior/Fadiorteam/projects/tools/quote-tool/scripts/mini_qcad_dwg.sh dwg2svg  INPUT.dwg [OUTPUT.svg]
/Users/Leo/.openclaw/workspace/projects/fadior/Fadiorteam/projects/tools/quote-tool/scripts/mini_qcad_dwg.sh dwg2csv  INPUT.dwg [OUTPUT.csv]
```

- 执行前先用 `test -x` 检查权威脚本；缺失就如实报告 DWG 桥接当前不可用
- **禁止**借用 `.worktrees/` 里的副本；工作台会被清理，不是运行时真源
- 省略 `OUTPUT` → 当前目录同名 `.pdf` / `.svg` / `.csv`
- 脚本 SCP 到 mini `/tmp/fadior-mini-qcad-<pid>/`，跑 QCAD，拉回结果并清理
- SSH 主机：`FADIOR_MINI_QCAD_HOST`（默认 `mac-mini`）
- QCAD Pro 试用启动约 15 秒属正常

策略全文只从同一权威项目路径读取；文档不存在时不要借用工作台副本。

## 6. Mac Studio vs Mac mini

| 能力 | Mac Studio | Mac mini |
|------|------------|----------|
| 建模（build123d / `scripts/step`） | 是（默认） | **否**，直到为 `/opt/homebrew/opt/python@3.12/bin/python3.12` 装好 `build123d` |
| CAD Explorer（`dev:ensure`） | 是 | 是（同步 skill 后；必要时在 `scripts/explorer` 里 `npm install`） |
| DWG → PDF/SVG/CSV | 仅当权威桥接脚本通过 `test -x` | 由权威桥接脚本调用 mini QCAD |
| Python | 项目解释器即可 | **必须** `/opt/homebrew/opt/python@3.12/bin/python3.12` |

mini 上若 `import build123d` 失败，不要声称能建模。

## 7. Progressive references

按需加载，不要一次全读：

- `references/natural-language-specs.md` — 白话 → CAD brief
- `references/step-generation.md` — STEP 生成与生成后检查
- `references/inspection-and-validation.md` — refs / facts / planes / measure / mate / diff
- `references/positioning.md` — 基准、关节、装配变换
- `references/dxf.md` — 次要 DXF
- `references/supported-exports.md` — STL/3MF/GLB
- `references/build123d-modeling.md` — build123d 模式
- `references/repair-loop.md` — 诊断与修复
- `references/moveit2-server.md` — Explorer MoveIt2 服务

最终回复：生成文件、跑过的 Explorer URL、实际校验、假设与限制。汇报结构见 `references/inspection-and-validation.md`。

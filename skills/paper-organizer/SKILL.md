---
name: paper-organizer
description: |
  整理散落的论文笔记到正确的分类目录。当用户说"整理论文"、"整理笔记"、"论文归档"
  "把论文整理好"、"检查论文分类"时使用。

  自动扫描 `_待整理` 和 `_精读笔记` 目录中的散落论文，根据论文主题归类到具体分类目录。
context: fork
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch, WebSearch
---

# 论文笔记整理助手

自动整理散落的论文笔记到正确的分类目录。

## Step 0: 读取共享配置

先读取 `../_shared/user-config.json`，如果 `../_shared/user-config.local.json` 存在，再用它覆盖默认值。

显式生成并在后续统一使用这些变量：

- `VAULT_PATH`
- `NOTES_PATH`
- `CONCEPTS_PATH`

其中：

- `NOTES_PATH = {VAULT_PATH}/{paper_notes_folder}`
- `CONCEPTS_PATH = {NOTES_PATH}/{concepts_folder}`

## 1. 扫描散落论文

扫描以下目录中的论文笔记（md 文件）：

- `{NOTES_PATH}/_待整理/`
- `{NOTES_PATH}/_精读笔记/`（根目录下的散落文件，不包括子目录）

扫描命令：

```bash
find "{NOTES_PATH}/_待整理" -maxdepth 1 -name "*.md" -type f 2>/dev/null
find "{NOTES_PATH}/_精读笔记" -maxdepth 1 -name "*.md" -type f 2>/dev/null
```

排除以下文件：
- `_README.md`
- `MOC_*.md`（目录页）
- 以 `_` 开头的索引文件

## 2. 提取论文信息

对每个散落的论文笔记，提取以下信息用于分类判断：

1. **读取 frontmatter**：提取 `tags`、`zotero_collection`（如有）、`method_name`
2. **读取正文前 50 行**：提取标题、核心主题关键词

### 分类依据优先级

1. **frontmatter 的 `zotero_collection`**（最高优先级）
   - 如有有效值，直接使用该路径作为目标目录
   - 格式示例：`3-Robotics/1-VLX/VLA`

2. **frontmatter 的 `tags`**
   - 第一个 tag 作为核心主题
   - 按概念分类规则匹配（见下表）

3. **正文关键词**
   - 从摘要、方法标题中提取关键词
   - 匹配分类规则

## 3. 分类规则映射表

| 分类目录 | 对应主题标签 | 关键词示例 |
|----------|-------------|-----------|
| `1-生成模型/` | diffusion, gan, vae, generative | 扩散模型、GAN、VAE、生成 |
| `2-强化学习/` | rl, reinforcement, policy, mbrl | 强化学习、策略学习、MBRL |
| `3-机器人策略/` | vla, manipulation, grasping, imitation | VLA、机械臂、抓取、模仿学习 |
| `4-足式运动/` | locomotion, quadruped, biped | 四足、双足、运动控制 |
| `5-导航与定位/` | navigation, slam, planning | 导航、SLAM、路径规划 |
| `6-3D视觉/` | nerf, 3dgs, depth, point-cloud | NeRF、3DGS、深度估计 |
| `7-规划与控制/` | mpc, control, optimization | MPC、控制、优化 |
| `8-仿真器/` | simulation, isaac, mujoco | 仿真、Isaac、MuJoCo |
| `9-无人机/` | uav, drone, flight | 无人机、飞行控制 |
| `10-数据集/` | dataset, benchmark, data | 数据集、基准测试 |
| `11-深度学习基础/` | transformer, attention, training | Transformer、注意力、训练技巧 |
| `12-物理仿真/` | physics, fem, biomechanics | 物理仿真、有限元 |
| `13-机器人硬件/` | sensor, actuator, hardware | 传感器、执行器 |
| `14-安全与鲁棒性/` | safety, robustness, adversarial | 安全、鲁棒性、对抗 |
| `15-网页智能体/` | web-agent, browser, gui | 网页智能体、浏览器 |
| `16-人体动作/` | motion, pose, human | 人体动作、姿态 |
| `_待整理/` | 无法匹配时 | 作为临时存放 |

## 4. 整理流程

对每篇散落论文：

1. **确定目标目录**：按分类规则映射
2. **检查目标目录是否存在**：
   ```bash
   ls "{NOTES_PATH}/{target_category}"
   ```
   不存在则创建

3. **移动文件**：
   ```bash
   mv "{source_path}" "{NOTES_PATH}/{target_category}/{filename}"
   ```

4. **更新 frontmatter**（如需要）：
   - 如果原 `zotero_collection` 为空或无效，更新为新分类路径
   - 使用 Edit 工具修改 YAML frontmatter

## 5. 执行报告

整理完成后，汇报：

| 项目 | 数量 |
|------|------|
| 扫描到的散落论文 | X 篇 |
| 成功整理 | X 篇 |
| 无法分类（保留在 `_待整理`） | X 篇 |
| 目标目录不存在需创建 | X 个 |

列出每篇论文的移动详情：
- `原路径` → `新路径`

## 6. 交互模式

整理前先展示扫描结果，询问用户确认：

```
发现以下散落论文：
1. [论文A] 在 _待整理 → 建议移动到 3-机器人策略/VLA
2. [论文B] 在 _精读笔记 → 建议移动到 6-3D视觉
...

是否执行整理？[确认/调整/取消]
```

用户可以：
- **确认**：执行所有建议移动
- **调整**：指定某篇论文的目标目录
- **取消**：不执行整理

## 7. 自动化触发

用户说以下任意触发词时自动调用：
- "整理论文"
- "整理笔记"
- "论文归档"
- "把论文整理好"
- "检查论文分类"
- "看看有没有散落的论文"

## 8. 完成后自检

- [ ] 所有散落论文都已处理（移动或标记保留）？
- [ ] 移动后的论文 frontmatter 已更新分类信息？
- [ ] 目录页（MOC）需要刷新？（提示用户执行 `/generate-mocs`）
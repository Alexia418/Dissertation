# 📋 Dissertation Timeline & Daily Work Plan
# CASA0004 | Yuyue Xia | Updated: June 2026

每次打开电脑，先看这个文件，找到今天的阶段，执行对应任务。
完成一项就在前面加 ✅。

---

## 🗓️ PHASE OVERVIEW

| Phase | Dates | Focus |
|---|---|---|
| **Phase 1** | 13–17 Jun | EDA + Toy model + 文献框架 (Sprint to 18 Jun meeting) |
| **Phase 2** | 18 Jun | Supervision meeting |
| **Phase 3** | 19 Jun – 4 Jul | Full model + scenario comparison + 写作并行 |
| **Phase 3B** | 4 Jul | ⚠️ Interim submission: 1,500–2,000 words to dissertation tutor |
| **Phase 3C** | 5–12 Jul | 评估指标 + 敏感性分析 + 继续写作 |
| **Phase 4** | 13–22 Jul | ⛔ 陪朋友，不学习 |
| **Phase 5** | 23 Jul – 19 Aug | Final draft + revision + submission |
| **Submission** | 19 Aug | 🎓 提交 |

> **核心原则：写作和编码从Phase 3开始并行。**
> Literature Review不依赖模型结果，现在就可以写。
> Methodology框架现在搭好，结果出来后只填数字。

---

## 🔴 PHASE 1 — Sprint to 18 June Supervision
### 5天冲刺：让导师看到数据 + 模型逻辑 + 文献框架

> **Camden说明：** Camden是伦敦的一个行政区，用来做toy model的小规模测试区，
> 因为它只有约20个LSOA，运行快。全模型跑大伦敦，Camden只是测试用。
> 你的研究城市是伦敦，用的是`03_data/`里的四个GLA CSV。

---

### 📅 Day 1 — Saturday 13 June
**目标：Di需求地图出现在屏幕上**

- [ ] **1a** — 打开新notebook `01_data_loading.ipynb`
  - 加载所有数据，每个打印 `.head()` 和 `.columns`
  - 文件路径：
    - `03_data/demand/census2021/TS001_usual_residents_London_LSOA_2021.csv`
    - `03_data/demand/census2021/TS045_car_van_availability_London_LSOA_2021.csv`
    - `03_data/demand/imd2019/IMD2019_LSOA_England.xlsx`
    - `03_data/restricted/gla/OpenStreetEV_GLA.csv`
    - `03_data/restricted/gla/gla_location_evse_join.csv`
    - `03_data/restricted/gla/evse_status_gla.csv`
    - `03_data/restricted/gla/device_all_uk.csv`
- [ ] **⛔ Better Future任务（1小时，固定）**
- [ ] **1b** — 打开新notebook `02_demand_estimation.ipynb`
  - TS001 → `pop_lsoa`（人口列）
  - TS045 → `car_rate_lsoa`（有车家庭比例）
  - IMD2019 → `imd_norm_lsoa`（Score列归一化0–1）
  - 从NTS查Pd和Ppub两个系数（查NTS表，记下表号）
  - 四个变量merge on LSOA code
  - 计算 `Di = Pi × Ci × Pd × Ppub × (1 + 0.3 × IMDi)`
  - GeoPandas choropleth地图，保存到 `05_outputs/figures/demand_map_lsoa.png`
- [ ] **1c** — 现有充电桩基准地图
  - 加载 `OpenStreetEV_GLA.csv` → 过滤大伦敦 → 点地图
  - 保存到 `05_outputs/figures/charger_existing_gla.png`
- [ ] Commit: `git commit -m "data: demand proxy Di + charger baseline mapped"`

---

### 📅 Day 2 — Sunday 14 June
**目标：toy model产出Camden选址地图**

- [ ] **2a** — Candidate sites生成（Camden范围）
  - 加载OSM数据（`03_data/supply/osm/`），过滤Camden边界
  - 提取停车场polygon质心作候选点
  - 保存 `03_data/processed/candidate_sites_camden.csv`
- [ ] **2b** — Camden子集
  - 需求点 = Camden的LSOA质心（从GLA shapefile裁切，约20个LSOA）
  - 计算需求点到候选点的欧式距离矩阵（shapely或geopy）
- [ ] **2c** — PuLP p-median toy model（`04_code/03_location_allocation/toy_model_camden.ipynb`）
  ```
  目标：minimise Σ Di × d(i,j) × x_ij
  约束：Σ y_j = 5（先设p=5）
        x_ij ≤ y_j
        Σ_j x_ij = 1
  ```
- [ ] **2d** — 如果跑通，加两个规划约束：
  - 服务半径约束（需求点只能分配给800m内候选点）
  - 容量约束（每个站最多服务N=5个LSOA）
- [ ] **2e** — 结果地图，保存 `05_outputs/figures/toy_model_camden.png`
- [ ] Commit: `git commit -m "model: toy p-median Camden test"`

---

### 📅 Day 3 — Monday 15 June
**目标：gap statement有引用 + literature_matrix有内容**

- [ ] **⛔ 上午会议（固定1小时）**
- [ ] **3a** — Google Scholar检索，目标收集15–20篇，存Zotero
  - `"EV charging location allocation optimization"` → 5篇
  - `"heterogeneous demand electric vehicle charging"` → 4篇
  - `"spatial equity EV charging deprivation"` → 3篇
  - `"EV charging demand estimation land use"` → 3篇
  - 每篇按格式命名：`P005_Surname_YYYY.pdf`（P001–P004已预填）
- [ ] **3b** — 填 `01_literature/literature_matrix.xlsx`
  - 每篇填：Author Year / Title / Theme / Method / Key Finding / Relevance
  - 重点找：有没有文章说过"existing models assume homogeneous demand" / "lack of standardised planning framework"
  - Pagany et al. (2018) 综述引言是优先看的
- [ ] **⛔ 下午线上Final Check（固定半小时）**
- [ ] **3c** — 精读Vazifeh 2019 / He 2022 / Lu 2026引言
  - 提取能支持gap statement的具体句子
  - 记到 `01_literature/reading_notes/P00X_Surname_YYYY.md`
- [ ] Commit: `git commit -m "lit: literature matrix populated, gap references found"`

---

### 📅 Day 4 — Tuesday 16 June
**目标：6张PPT slides完成 + 发邮件给导师**

- [ ] **4a** — 收尾任何未完成的代码，确认三张地图图片清晰
- [ ] **4b** — 制作会议PPT（`06_writing/presentations/meeting2_Jun18.pptx`）
  - Slide 1: 研究问题 + 方法框架
  - Slide 2: 数据清单 + 变量含义表
  - Slide 3: 需求地图Di choropleth
  - Slide 4: 现有充电桩分布（空间错配）
  - Slide 5: Toy model结果地图（Camden）
  - Slide 6: Literature framework表格骨架（四个方向，代表文献）
  - Slide 7: 下一步计划 + 给导师的问题
- [ ] **4c** — 发简短邮件给导师（3–4行），说明明天会议准备展示的内容
- [ ] **4d** — 博士方向表述（2–3句话准备好）
  - 动机：为什么想读博
  - 方向：想往Prof. Zhong的urban mobility / spatial optimisation方向延伸
  - 在meeting里自然引出，不需要完整proposal

---

### 📅 Day 5 — Wednesday 17 June
**目标：全部材料就绪，心里有底**

- [ ] **5a** — 最终材料检查：
  - [ ] 所有notebook从头可以运行
  - [ ] 三张地图图片有标题和图例
  - [ ] literature_matrix.xlsx至少15行有内容
  - [ ] PPT slide 1–7都有内容
- [ ] **5b** — 每张slide准备2–3句口头说辞
- [ ] **缓冲：14:00之前完成所有检查**
- [ ] **⛔ 15:00–19:00 Pre活动（Punnett Hall, IOE Building）**

---

## 🔵 PHASE 2 — Supervision Meeting (18 June)

- 展示顺序：数据 → 需求地图 → 充电桩地图 → toy model → 文献框架
- 会议中自然提起博士方向，请两位导师一起参考和规划
- 记录导师所有反馈到 `00_admin/meeting_notes/meeting2_Jun18.md`

---

## 🟡 PHASE 3 — Full Model + Writing in Parallel (19 Jun – 4 Jul)

> **每天的节奏：上午写代码，下午写文字。两件事并行，不要等模型做完再写。**

### 📅 19–20 Jun（周四–周五）— 扩展模型基础
- [ ] 需求估计扩展：加入土地用途异质性（商业/居住/混合分类权重）
- [ ] 候选点生成，全大伦敦范围（从OSM提取所有公共停车场）
- [ ] 【写作】开始Literature Review草稿：写Location-Allocation一节（约400字）
  - 参考：Vazifeh 2019, He 2022, Pagany 2018
  - 保存到 `06_writing/dissertation_draft.docx`

### 📅 21–22 Jun（周六–周日）— P-median全伦敦
- [ ] P-median扩展到全伦敦所有LSOA
- [ ] 跑Scenario A：Baseline（λ=0，纯效率）
- [ ] 【写作】Literature Review：写Demand Estimation一节（约400字）
  - 参考：Lu 2026 + 新收集的需求估计文献

### 📅 23–25 Jun（周一–周三）— 三个情景
- [ ] 跑Scenario B：Equity-weighted（λ=0.3, 0.6, 1.0）
- [ ] 跑Scenario C：Maximal Covering变体
- [ ] 【写作】Literature Review：写Spatial Equity一节（约400字）
  - 参考：IMD相关文献 + spatial equity in transport文献

### 📅 26–28 Jun（周四–周六）— 评估指标
- [ ] 计算全部评估指标：
  - 未满足需求（unmet demand）
  - 平均行驶距离
  - 各IMD分组覆盖率
  - Gini系数（空间可及性）
- [ ] 【写作】Literature Review：写Policy/Context一节（约400字）
  - 参考：EV Infrastructure Strategy 2022, HoC Briefing 2025, EIT report

### 📅 29 Jun – 1 Jul（周日–周二）— 敏感性分析
- [ ] 敏感性分析：改变p（站点数）、R（服务半径）、λ（公平权重）
- [ ] 画trade-off曲线（λ变化 → 效率/公平的变化）
- [ ] 【写作】Literature Review通读，把四节合并成一篇连贯的综述

### 📅 2–4 Jul（周三–周五）— Interim Submission准备
- [ ] 整理Literature Review到1,500–2,000字
- [ ] 检查Harvard引用格式，确认所有引用在literature_matrix里有记录
- [ ] 补充文献到30篇以上
- [ ] **⚠️ 4 Jul：提交1,500–2,000字草稿给dissertation tutor**

---

## 🟡 PHASE 3C — 评估收尾 + 写作推进 (5–12 Jul)

### 📅 5–7 Jul — 结果可视化 + 写Methodology草稿
- [ ] 产出全部核心图表（7张，见dissertation_overview.md第五节）
- [ ] 确认所有图表300 dpi，有标题、图例、比例尺
- [ ] 【写作】起草Methodology章节：
  - 需求估计公式（把Di公式写清楚，解释每个变量）
  - 模型设计（p-median目标函数 + 三个约束）
  - 公平性加权逻辑（λ参数）

### 📅 8–10 Jul — 写Results草稿
- [ ] 写Results章节：
  - 需求分布描述（Di地图的发现）
  - 现有供给缺口（supply-demand mismatch）
  - 三个情景的选址结果对比
  - IMD分组覆盖率表格

### 📅 11–12 Jul — 写Discussion草稿
- [ ] 写Discussion章节：
  - 对应Gap 1（公平性）：equity情景和baseline的差异说明了什么
  - 对应Gap 2（需求异质性）：Di如何capture了空间差异
  - 对应Gap 3（规划约束）：现实约束如何影响了选址结果
  - 局限性（数据限制、模型简化假设）

---

## ⛔ PHASE 4 — Break (13–22 July)
**陪朋友，完全不学习。**
**前提：Phase 3C的Results和Discussion草稿必须在12号前完成。**

---

## 🔴 PHASE 5 — Final Draft & Submission (23 Jul – 19 Aug)

### 📅 23–25 Jul — 回来，重新进入状态
- [ ] 重新跑所有代码，确认结果可复现
- [ ] 通读已有草稿（Lit Review + Methodology + Results + Discussion）
- [ ] 列出需要补充/修改的地方

### 📅 26–28 Jul — 写Introduction + Conclusion
- [ ] 写Introduction（最后写，因为这时候最清楚整篇论文说了什么）
  - 背景（EV政策、伦敦充电现状）
  - Research gap（三个gap）
  - Research questions
  - Dissertation structure
- [ ] 写Conclusion
  - 总结三个contribution（对应三个gap）
  - 政策建议（给伦敦规划者）
  - 局限性和未来研究方向

### 📅 29–31 Jul — 整合全文
- [ ] 把所有章节拼在一起
- [ ] 检查章节之间的逻辑连贯性
- [ ] 确认Introduction提到的gap和Discussion的发现相互对应

### 📅 1–5 Aug — 第一轮修改
- [ ] 检查所有图表编号（Figure 1, 2, 3...）
- [ ] 检查所有表格编号（Table 1, 2, 3...）
- [ ] 检查图表在文中是否都有引用（"as shown in Figure 3..."）
- [ ] Harvard引用格式核对（用Zotero导出）
- [ ] 字数统计，确认在要求范围内

### 📅 6–10 Aug — 第二轮修改
- [ ] 语言润色（去掉冗余句子，加强段落之间的衔接）
- [ ] 地图图例检查（300 dpi、标题、比例尺、北方向标）
- [ ] 附录整理（数据说明、代码说明）
- [ ] 摘要（Abstract）写作，约250字

### 📅 11–14 Aug — 第三轮修改
- [ ] 找同学或朋友通读全文，收集反馈
- [ ] 根据反馈修改
- [ ] 再次检查参考文献列表（格式一致性）

### 📅 15–17 Aug — 最终格式检查
- [ ] 封面信息（姓名、学号、题目、字数、导师）
- [ ] 目录（自动生成，检查页码）
- [ ] 页眉/页脚格式
- [ ] 导出PDF，检查排版没有乱码或错位

### 📅 18 Aug — 缓冲日
- [ ] 处理任何最后发现的问题
- [ ] 导出最终PDF

### 📅 19 Aug — 提交 🎓
- [ ] 上午：最后一遍检查封面
- [ ] 提交

---

## 📌 Quick Reference: Key Decisions Made

| Decision | Choice | Reason |
|---|---|---|
| Study area | Greater London (GLA) | Data availability, UCL supervisor expertise |
| Toy model test area | Camden (London borough) | ~20 LSOAs only, fast to compute; part of GLA |
| Spatial unit | LSOA | Census + IMD data native at this level |
| Demand proxy | Census + NTS formula | Supervisor: be creative, heterogeneous |
| Model family | P-median + set-covering + maximal covering | PuLP in Python |
| Equity measure | IMD 2019 (England) | Latest available; LSOA-level |
| Supply data | Supervisor-provided 4 GLA CSVs | NCR closed Nov 2024 |
| Literature management | literature_matrix.xlsx + Zotero | File naming: P001_Surname_YYYY |

---

## 📁 Data Files Confirmed (as of 13 June 2026)

| File | Location | Status |
|---|---|---|
| TS001_usual_residents_London_LSOA_2021.csv | 03_data/demand/census2021/ | ✅ Downloaded |
| TS045_car_van_availability_London_LSOA_2021.csv | 03_data/demand/census2021/ | ✅ Downloaded |
| IMD2019_LSOA_England.xlsx | 03_data/demand/imd2019/ | ✅ Downloaded |
| nts-2024-excel-tables | 03_data/demand/nts/ | ✅ Downloaded |
| device_all_uk.csv | 03_data/restricted/gla/ | ✅ Supervisor-provided |
| evse_status_gla.csv | 03_data/restricted/gla/ | ✅ Supervisor-provided |
| gla_location_evse_join.csv | 03_data/restricted/gla/ | ✅ Supervisor-provided |
| OpenStreetEV_GLA.csv | 03_data/restricted/gla/ | ✅ Supervisor-provided |
| OSM GLA shapefile | 03_data/supply/osm/ | ✅ Downloaded |
| OS Open Roads | 03_data/supply/os_open_roads/ | ✅ Downloaded |

---

## 🚨 Supervisor Instructions (Do Not Forget)

1. "No standardised spatial planning framework" — 需要2–3篇文献支持这个gap
2. 需求估计要体现异质性（商业/居住/混合土地用途）+ 动态元素
3. 文献要覆盖四个方向，几十篇是整个dissertation周期目标（不是18号前）
4. P-median要加入更多规划约束（服务半径 + 容量）
5. 电网约束不从UK Power Network找——替代方案：OSM power=substation标签
6. Zapmap限制两人同时使用；已用导师提供的GLA CSV替代

---

## ⚠️ Key Deadlines

| Date | Milestone |
|---|---|
| 17 Jun | Phase 1 完成，所有材料就绪 |
| 18 Jun | Supervision meeting |
| 4 Jul | ⚠️ Interim submission: 1,500–2,000 words to dissertation tutor |
| 12 Jul | Phase 3 全部完成（代码 + 草稿），赶在休息前 |
| 13–22 Jul | ⛔ 休息 |
| 19 Aug | 🎓 Final submission |

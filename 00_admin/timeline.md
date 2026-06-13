# 📋 Dissertation Timeline & Daily Work Plan
# CASA0004 | Yuyue Xia | Updated: 13 June 2026

每次打开电脑，先看这个文件，找到今天的阶段，执行对应任务。
完成一项就在前面加 ✅。

---

## 🗓️ PHASE OVERVIEW

| Phase | Dates | Focus |
|---|---|---|
| **Phase 1** | 13–17 Jun | EDA + Toy model + 文献框架 (Sprint to 18 Jun meeting) |
| **Phase 2** | 18 Jun | Supervision meeting — present progress, raise PhD direction |
| **Phase 3** | 19 Jun – 12 Jul | Full model + scenario comparison + writing |
| **Phase 4** | 13–22 Jul | ⛔ 陪朋友，不学习 |
| **Phase 5** | 23 Jul – 1 Aug | Final draft + proofread + submission |

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
- [ ] **2c** — PuLP p-median toy model（`03_location_allocation/toy_model_camden.ipynb`）
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
- [ ] **3b** — 填literature_matrix.xlsx（`01_literature/literature_matrix.xlsx`）
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
  - 不要请求额外会议，只告知议程
- [ ] **4d** — 博士方向表述（2–3句话）
  - 动机：为什么想读博
  - 方向：想往Prof. Zhong的urban mobility / spatial optimisation方向延伸
  - 在meeting里自然引出，不需要完整proposal

---

### 📅 Day 5 — Wednesday 17 June
**目标：全部材料就绪，心里有底**

- [ ] **5a** — 最终材料检查清单：
  - [ ] 数据清单完整（来源、变量名、用途）
  - [ ] 所有notebook从头可以运行
  - [ ] 三张地图图片有标题和图例
  - [ ] literature_matrix.xlsx有内容（至少15行填了Theme和Key Finding）
  - [ ] PPT逻辑顺畅，slide 1–7都有内容
- [ ] **5b** — PPT口头表述准备（每张slide 2–3句说辞）
- [ ] **缓冲：14:00之前完成所有上述检查**
- [ ] **⛔ 15:00–19:00 Pre活动（Punnett Hall, IOE Building）**

---

## 🔵 PHASE 2 — Supervision Meeting (18 June)

- 展示顺序：数据 → 需求地图 → 充电桩地图 → toy model → 文献框架
- 会议中自然提起博士方向，请两位导师一起参考和规划
- 记录导师所有反馈到 `00_admin/meeting_notes/meeting2_Jun18.md`

---

## 🟡 PHASE 3 — Full Model & Writing (19 Jun – 12 Jul)

### Week of 19–22 Jun — 扩展模型
- [ ] **6** — 需求估计扩展：加入土地用途异质性（商业/居住/混合分类权重）
- [ ] **7** — 候选点生成，全大伦敦范围
- [ ] **8** — P-median扩展到全伦敦（所有LSOA）

### Week of 23–29 Jun — 三个情景
- [ ] **9** — 场景A：Baseline（只考虑效率）
- [ ] **10** — 场景B：Equity-weighted（加IMD惩罚项，λ参数化）
- [ ] **11** — 场景C：Coverage-maximising（最大覆盖变体）

### Week of 30 Jun – 6 Jul — 评估指标
- [ ] **12** — 计算评估指标：
  - 未满足需求（unmet demand）
  - 平均行驶距离到最近充电站
  - 各IMD分组覆盖率
  - Gini系数（空间可及性）
- [ ] **13** — 敏感性分析：改变p（站点数）、R（服务半径）、λ（公平权重）

### Week of 7–12 Jul — 文献 + 写作
- [ ] **14** — 文献库扩展到30–40篇，literature_matrix补充完整
- [ ] **15** — 开始写Results章节（情景对比表 + 地图）
- [ ] **16** — 开始写Discussion章节（对应Vazifeh / He / Lu的gap）

---

## ⛔ PHASE 4 — Break (13–22 July)
**陪朋友，完全不学习，提前把Phase 3做完。**

---

## 🔴 PHASE 5 — Final Draft (23 Jul – 1 Aug)

- [ ] **17** — 写Introduction + Conclusion
- [ ] **18** — 把文献综述章节扩展完整（引用literature_matrix里的文献）
- [ ] **19** — 全文通读，逻辑检查
- [ ] **20** — Harvard格式参考文献整理（用Zotero导出）
- [ ] **21** — 格式检查（字数、图表编号、附录）
- [ ] **22** — 提交

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

# 每个数据集的用途详解

---

## 一、需求侧数据（Demand-Side）

### 1. TS001 — 常住人口数据
**文件：** `TS001_usual_residents_London_LSOA_2021.csv`

这是 Census 2021 的人口数据，记录了每个 LSOA 里有多少常住人口。

**在dissertation里的用途：**
这是构建需求代理变量 $D_i$ 的基础分母。你的需求公式是：

$$D_i = P_i \times C_i \times P_d \times P_{pub} \times w_i$$

其中 $P_i$ 就是来自 TS001 的该 LSOA 总人口数。人口越多的地方，潜在的 EV 充电需求就越大。单独用人口数不够（因为没车的人不需要充电桩），所以要和 TS045 结合起来用。

---

### 2. TS045 — 家庭车辆持有数据
**文件：** `TS045_car_van_availability_London_LSOA_2021.csv`

记录了每个 LSOA 里，家庭持有 0 辆车、1 辆车、2 辆车、3辆及以上车辆的家庭数量。

**在dissertation里的用途：**
这是构建 $C_i$（车辆持有率）的数据来源。计算方法是：

$$C_i = \frac{\text{cars\_1} + \text{cars\_2} + \text{cars\_3plus}}{\text{cars\_0} + \text{cars\_1} + \text{cars\_2} + \text{cars\_3plus}}$$

也就是"有车家庭占所有家庭的比例"。这个比例反映了该 LSOA 潜在 EV 车主的密度。车辆持有率高的地区，对充电桩的需求也更高。

此外你还可以计算每个 LSOA 的**平均车辆数**（total vehicles / total households），作为更细粒度的需求权重。

---

### 3. IMD 2019 — 多维剥夺指数
**文件：** `IMD2019_LSOA_England.xlsx`

记录了英格兰每个 LSOA 的剥夺程度排名（Rank）和十分位数（Decile）。Rank 越小说明越贫困，Decile 1 是最贫困的 10%，Decile 10 是最富裕的 10%。

**在dissertation里的用途：**
这是你论文的**核心创新点**所在，也是你和其他论文最大的区别。

IMD 用来构建空间公平权重 $w_i$：

$$w_i = f(\text{IMD\_decile}_i)$$

直觉是：越贫困的地区（Decile 1-3），越需要政策倾斜，给它更高的需求权重，让模型在优化时优先照顾这些地区。这样选出来的充电桩位置不只是"效率最优"，还是"公平可达"的。

在 EDA 阶段你还可以用 IMD 做空间可视化，展示 London 的剥夺程度分布，以及现有充电桩是否集中在富裕地区（这很可能是现实情况，可以作为研究动机的实证支撑）。

**注意：** IMD 2019 用的是 2011 年的 LSOA 边界，Census 2021 用的是 2021 年边界，两者不完全一致，清洗阶段需要用 ONS 提供的 lookup table 做边界转换。

---

## 二、供给侧数据（Supply-Side，受限数据）

这四个文件是导师提供的受限数据，来自 Zapmap/OZEV，替代了已关闭的 NCR 数据库。

---

### 4. OpenStreetEV_GLA — 充电站位置数据
**文件：** `OpenStreetEV_GLA.csv`

记录了 Greater London 范围内所有公共充电站（location）的空间信息，包括经纬度、地址、邮编、borough、位置类型（路边/停车场等）。

**在dissertation里的用途：**
这是描述**现有充电基础设施供给**的核心数据。具体用途有三个：

第一，做现状分析，画出 London 现有充电站的空间分布图，展示哪些区域供给充足、哪些区域存在缺口（supply gap）。

第二，作为**候选站点**的参考，如果某个位置已有充电站，说明该位置在规划层面是可行的，可以纳入候选站点集合。

第三，和 IMD 数据叠加，分析现有充电站是否在空间上存在不公平分布（比如集中在 IMD Decile 8-10 的富裕区域），为你的公平性研究动机提供实证支撑。

---

### 5. gla_location_evse_join — 站点与充电桩连接表
**文件：** `gla_location_evse_join.csv`

只有两列：`location_id` 和 `evse_id`。是一个多对多的连接表，记录了每个充电站（location）下面有哪些具体的充电桩（EVSE）。

**在dissertation里的用途：**
这是连接 OpenStreetEV（站点层级）和 evse_status（充电桩层级）的桥梁。一个充电站可能有多个充电桩，通过这张表可以知道每个站点有多少个充电桩，从而计算**每个站点的总装机容量**和**实际可用率**。

在需求建模里，这个数据帮助你量化现有供给能力，判断哪些站点已经饱和、哪些还有余量。

---

### 6. evse_status_gla — 充电桩状态记录
**文件：** `evse_status_gla.csv`

有 2500 万行，记录了每个充电桩（EVSE）每次状态变化的时间戳和状态值（Available / Charging / Out of Service 等）。

**在dissertation里的用途：**
这是分析充电桩**实际使用效率**的数据。你不需要用全部 2500 万行，而是需要聚合成每个 EVSE 的：

- **可用率**（availability rate）：该充电桩处于 Available 状态的时间比例
- **使用率**（utilisation rate）：处于 Charging 状态的时间比例

这两个指标有两个用途：第一，识别哪些现有充电桩长期空置（说明需求不足或位置不好）；第二，为你的位置-分配模型提供现实依据，避免把新充电桩选在已经供过于求的地方。

这也是你论文里"动态需求"元素的体现，能让你的需求估计比单纯用人口静态数据更有说服力。

---

### 7. device_all_uk — 全英国充电设备属性
**文件：** `device_all_uk.csv`

记录了全英国所有充电设备的 `location_id`、`zapmap_device_uid`、`power_band`（功率档位，如 Slow/Fast/Rapid）、`device_max_power`（最大功率，kW）。

**在dissertation里的用途：**
这个数据告诉你每个充电站里每台设备的**功率等级**。功率等级非常重要，因为：

第一，慢充（3-7kW）、快充（7-22kW）、急速充（50kW+）对应不同的用户需求和停留时长，在需求估计时权重不同。

第二，在规划约束里，高功率充电桩对电网容量要求更高，会影响候选站点的可行性判断。

实际使用时先用 `gla_location_evse_join` 筛选出 GLA 范围内的 `evse_id`，再和 `device_all_uk` 的 `zapmap_device_uid` 对应，最终得到 London 范围内每个充电站的功率结构。

---

## 三、数据之间的关系总结

```
TS001 (人口) ──┐
               ├──→ 需求代理变量 Di ──┐
TS045 (车辆) ──┘                      │
                                      ├──→ 位置-分配模型
IMD 2019 (公平权重) ──────────────────┤    (PuLP: p-median /
                                      │     set-covering)
OpenStreetEV ──┐                      │
               ├──→ 现有供给分析 ─────┘
gla_join ──────┤
               ├──→ 站点容量与可用率
evse_status ───┘

device_all_uk ──→ 功率结构分析 ──→ 规划约束条件
```

所有数据最终汇聚到位置-分配模型里，需求侧决定"哪里最需要充电桩"，供给侧决定"哪里已经有了以及有多少"，IMD 决定"公平性目标如何量化"。

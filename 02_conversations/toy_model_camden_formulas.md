# Toy P-Median Model (Camden) — 公式、来源与原理说明
# Formulas, Sources & Reasoning

> 对应notebook：`04_code/05_location_allocation/00_toy_model_camden.ipynb`
> Notebook: `04_code/05_location_allocation/00_toy_model_camden.ipynb`
>
> 本文档覆盖notebook里目前已经写好并跑通的部分：Camden子集 → 需求点 → 候选点 → 距离矩阵 → P-median模型 → 容量约束 → 服务半径约束（对应你计划里的2a–2d）。
>
> This document covers the parts of the notebook that currently exist and have run: Camden subset → demand points → candidate sites → distance matrix → P-median model → capacity constraint → service radius constraint (steps 2a–2d in your plan).

---

## 目录 / Contents

1. 整体逻辑链 / Pipeline Logic
2. Camden案例研究子集与需求点 / Camden Subset & Demand Points
3. 需求估算公式 Dᵢ / Demand Estimation Formula
4. IMD标准化 / IMD Normalisation
5. 候选点：OSM停车场质心 / Candidate Sites: OSM Parking Centroids
6. 距离矩阵 / Distance Matrix
7. P-median数学模型 / P-median Mathematical Model
8. Continuous Relaxation的数学依据 / Why x_ij Can Be Relaxed to Continuous
9. 容量约束（2d-1）/ Capacity Constraint
10. 服务半径约束（2d-2）/ Service Radius Constraint
11. 背景参考（非正式）/ Background References (informal)

---

## 1. 整体逻辑链 / Pipeline Logic

**中文：** notebook按这个顺序执行：① 从已经算好的全伦敦需求表（`demand_london.csv`）里截取Camden的LSOA子集，质心当需求点；② 从OSM停车场图层里截取Camden范围内的候选选址点，polygon质心当候选点；③ 算需求点到候选点的距离矩阵；④ 用P-median模型在这些候选点里选出p个，使"需求加权距离"最小。每一步只为下一步准备输入——候选点和需求点是模型的两组节点，距离矩阵是目标函数里的系数，P-median本身只是在这些固定的节点和系数上做优化，不会再回头改变前面生成的数据。

**English:** the notebook runs in this order: (1) subset the already-computed London-wide demand table (`demand_london.csv`) to Camden's LSOAs, using centroids as demand points; (2) subset the OSM parking layer to Camden, using polygon centroids as candidate sites; (3) build the distance matrix between the two point sets; (4) solve the p-median model to pick p sites minimising demand-weighted distance. Each step only produces inputs for the next — candidate sites and demand points are the model's two node sets, the distance matrix supplies the objective function's coefficients, and the optimisation step consumes these fixed inputs without altering anything generated earlier.

---

## 2. Camden案例研究子集与需求点 / Camden Subset & Demand Points

**中文：** 先把范围限定在Camden，是toy model阶段为了控制问题规模、方便快速调试，不是最终模型的范围（生产版本应该覆盖全伦敦4,994个LSOA）。用LSOA质心代表需求点，是空间分析里常见的简化（zone-based demand discretisation）：把一个LSOA内部连续分布的人口压缩成一个点，用这个点跟候选点的距离去近似"这个LSOA的人到最近站点的距离"。这个简化有已知的误差来源（LSOA内部居民实际位置可能离质心较远），但在LSOA这种较小尺度的空间单元上，误差通常可以接受，是facility location文献里的标准做法。

**English:** restricting to Camden first is a toy-model device for controlling problem size during testing, not the final spatial scope (the production model should cover all 4,994 London LSOAs). Representing each LSOA's demand by its centroid is the standard "zone-based demand discretisation" used throughout facility-location literature: it collapses continuously distributed population within a zone into a single point, at the cost of some intra-zone distance error — generally acceptable given LSOAs are relatively small, homogeneous units.

```python
camden_lsoa = lsoa_demand[lsoa_demand["lad_name"] == "Camden"].reset_index(drop=True)
camden_demand["geometry"] = camden_demand.geometry.centroid
```

---

## 3. 需求估算公式 Dᵢ / Demand Estimation Formula

$$D_i = P_i \times C_i \times P_d \times P_{pub} \times (1 + \alpha \times IMD_i)$$

| 符号 | 含义 | 来源 | Meaning | Source |
|---|---|---|---|---|
| $P_i$ | LSOA i 的常住人口总数 | Census 2021 TS001 | total residents | Census 2021 TS001 |
| $C_i$ | 至少一辆车家庭占比 | Census 2021 TS045，`(total_households − cars_0) / total_households` | car ownership rate | Census 2021 TS045 |
| $P_d$ | 0.1，日均充电概率 | NTS 2024：私家车日均约20英里，EV续航约200英里，200/20=10天充一次，1/10=0.1 | daily charging probability | derived from NTS 2024 |
| $P_{pub}$ | 0.4，依赖公共桩比例 | NTS 2024：约40%车主无私人充电条件 | share relying on public charging | NTS 2024 |
| $\alpha$ | 0.3，公平性权重 | 自定，最大放大幅度30% | equity weighting factor | author-defined |
| $IMD_i$ | 标准化贫困指数（0–1，越大越贫困） | 见第4节 | normalised deprivation index | see Section 4 |

**中文 — 为什么是这个结构：** 这本质上是交通规划里常见的"人口×行为概率"乘法型需求估算思路（类似trip generation模型里population×trip rate的结构）：$P_i \times C_i$把"总人口"缩小到"有车人口"这个跟充电更相关的基数；$P_d \times P_{pub}$再把"潜在充电需求"进一步缩小到"依赖公共桩的那部分需求"，因为这个模型只规划公共充电站，不规划私桩。最后的$(1+\alpha \times IMD_i)$是公平性调整项：贫困地区往往私家车位/车库更少（以街边停车为主），居民对公共充电桩的依赖程度系统性地高于条件相似但更富裕的地区，单纯用人口×车辅有率会低估这些地区的真实公共需求，所以加一个权重去补偿。

**关于来源的重要说明：** 这个公式不是直接抄自某一篇文献的现成公式，是你按supervisor"demand estimation should combine heterogeneity and dynamic elements"的要求，结合Census/NTS/IMD三个数据源自己构造的demand proxy。写dissertation方法论部分时，建议把它定位成"author-constructed demand proxy，借鉴标准trip-generation乘法逻辑 + IMD-weighted equity adjustment"，不要写成引用自某篇具体论文，除非你之后真的找到了结构完全一致的文献再去对应引用。

**English — why this structure:** this follows the familiar "population × behavioural probability" multiplicative logic common in transport trip-generation models. $P_i \times C_i$ narrows total population down to the car-owning population, the base most relevant to charging demand. $P_d \times P_{pub}$ further narrows potential charging demand down to the portion relying specifically on public infrastructure, since this model only sites public chargers, not home chargers. The closing $(1+\alpha \times IMD_i)$ term is the equity adjustment: deprived areas tend to have less off-street parking/private charging access, so reliance on public charging is systematically higher there than in otherwise-similar but wealthier areas; population × car-ownership alone would underestimate true public demand in those areas.

**Note on attribution:** this is not a formula taken verbatim from a single published source — it's a demand proxy you constructed yourself, combining Census/NTS/IMD data per your supervisor's guidance. In your methodology section, frame it as an author-constructed proxy informed by standard trip-generation logic plus an IMD-weighted equity adjustment, rather than citing it as drawn from one specific paper, unless you later find a published formula with the exact same structure to cite against.

---

## 4. IMD标准化 / IMD Normalisation

$$IMD_{i,norm} = \frac{rank_{max} - imd\_rank_i}{rank_{max} - rank_{min}}$$

**中文：** 官方IMD排名的规则是数字越小越贫困（全英格兰LSOA里排名第1的是最贫困的那个）。但公式里需要的是"贫困程度越高，数值越大"的权重（这样乘进$(1+\alpha \times IMD_i)$里才会把需求往上调），方向正好相反。直接用原始rank会得到错误效果——越贫困的地区数值反而越小。所以先用$rank_{max}$减去rank把方向翻转过来，再除以全距做0–1标准化。标准化后，最贫困的LSOA（rank=1）对应$IMD_{norm} \approx 1$，最不贫困的LSOA对应$IMD_{norm} \approx 0$。

**English:** official IMD ranks run in the direction opposite to what the formula needs — rank 1 is the *most* deprived LSOA in England. The formula needs deprivation to scale upward as the weight increases (so the equity term correctly boosts demand), so using the raw rank directly would produce the wrong effect. The $(rank_{max} - rank)$ flip reverses the direction before min-max normalising to [0,1]: the most deprived LSOA (rank 1) ends up with $IMD_{norm} \approx 1$, the least deprived with $IMD_{norm} \approx 0$.

**来源 / Source：** UK Government Index of Multiple Deprivation (IMD) 2019，由Ministry of Housing, Communities & Local Government（现MHCLG/前身DLUHC）发布，综合income / employment / education / health / crime / barriers to housing / living environment七个domain算出的官方综合贫困指标，是英国空间规划与公平性研究里最常用的deprivation测量工具。

---

## 5. 候选点：OSM停车场质心 / Candidate Sites: OSM Parking Centroids

**中文：** 候选点用OSM停车场polygon的质心，逻辑是"哪里有停车场，哪里就有安装充电桩的物理可行性"——这是EV充电选址研究里常见的候选点生成方式（用现有停车基础设施作feasibility proxy，而不是在地图上随机撒点或铺规则网格），因为充电桩本质上需要占用一个停车位，已有停车场天然满足这个条件。Geofabrik的OSM extract里，停车场放在"traffic"图层（`gis_osm_traffic_a_free_1.shp`），用`fclass`字段标记，筛`fclass=='parking'`即可（排除`'parking_bicycle'`，自行车停车位装不了EV桩）。

**关于筛选顺序的细节：** 代码里是先用polygon本身判断"是否跟Camden边界相交"（`intersects`），筛完之后才对剩下的polygon取质心，而不是反过来（先取质心再筛"质心是否落在Camden内"）。原因：如果先取质心再筛，会漏掉一些跨界的停车场——比如某个停车场大部分面积在Camden内，但几何质心恰好落在隔壁borough那一侧，"质心筛选"会把它误判成"不在Camden"而剔除，但这个停车场对Camden的可达性其实是有意义的。先筛polygon再取质心能避免这种边界误判。

```python
parking_camden = parking[parking.geometry.intersects(camden_boundary)].reset_index(drop=True)
candidate_sites["geometry"] = candidate_sites.geometry.centroid
```

**English:** candidate sites use OSM parking-lot polygon centroids — a standard candidate-generation approach in EV-charging siting studies, using existing parking infrastructure as a feasibility proxy rather than placing candidates on an arbitrary grid, since installing a charger physically requires a parking space. In Geofabrik's OSM extract, car parks live in the "traffic" layer (`gis_osm_traffic_a_free_1.shp`), tagged via the `fclass` field — filtering `fclass == 'parking'` excludes `'parking_bicycle'`, which can't host an EV charger. Filtering on the polygon geometry first (intersects with the Camden boundary), then taking centroids of the surviving subset, avoids mis-excluding boundary-straddling lots whose mass mostly sits in Camden but whose exact geometric centre happens to fall just over the line.

---

## 6. 距离矩阵 / Distance Matrix

**中文：** 距离矩阵用的是投影坐标系（EPSG:27700, British National Grid）下的欧式距离，单位是米——这是项目从EDA阶段就统一的约定。WGS84（EPSG:4326）的经纬度是角度单位，直接拿经纬度差值算"欧式距离"在数值上没有真实的物理长度含义（同样1度经度，在不同纬度对应的实际公里数不一样），必须先投影到一个以米为单位的平面坐标系，"距离"这个词才有意义。`scipy.spatial.distance.cdist`是向量化的两两距离计算，比写双重循环逐对调用shapely的`.distance()`快得多，在130×347这个规模下这个差距会比较明显。

**English:** distances use Euclidean distance in EPSG:27700 (British National Grid), in metres — the projected-CRS convention used consistently since the EDA stage. Raw WGS84 lat/lon coordinates are angular units; computing a naïve Euclidean distance directly on degrees has no real physical-length meaning (a degree of longitude maps to a different number of kilometres depending on latitude), so projecting to a metric planar CRS is a prerequisite for "distance" to be meaningful. `scipy.spatial.distance.cdist` computes the full pairwise distance matrix in one vectorised call, far faster than looping over shapely's `.distance()` pair by pair — the gap is noticeable at this scale (130 × 347).

---

## 7. P-median数学模型 / P-median Mathematical Model

**来源 / Source：** P-median是设施选址（facility location）领域最经典的模型之一，理论根源可追溯到Hakimi（1964年前后）关于图论中"median"问题的研究，后被ReVelle & Swain（约1970年）写成现在常用的0-1整数规划形式。*（具体年代/期刊细节建议你引用前自己核实一下原文，这里只是凭一般学术常识写的背景，没有从你的文献库里逐条核对。）* 这个模型在公共设施选址（学校、医院、消防站）和近年的EV充电站选址研究里都很常用。

**目标函数 / Objective:**
$$\min \sum_i \sum_j D_i \times d(i,j) \times x_{ij}$$

**约束 / Constraints:**
$$\sum_j y_j = p \qquad \sum_j x_{ij} = 1 \ \forall i \qquad x_{ij} \le y_j \ \forall i,j$$

**中文逐项含义：**
- $y_j \in \{0,1\}$：候选点j是否被选为充电站
- $x_{ij}$：需求点i被分配给候选点j的比例（这里做了relaxation，见第8节）
- 目标函数：所有需求点到"它被分配的那个站"的距离，按需求量$D_i$加权求和后取最小——本质是在最小化"全体用户的需求加权总出行距离"，而不是单纯的总距离或覆盖人数。这是p-median跟你后面要做的另外两种模型的核心区别：Set Covering是"用最少的站覆盖所有需求点（在某个距离阈值内）"，Maximal Covering是"给定站数，覆盖最多需求"，p-median是"给定站数，最小化加权距离"——三者优化目标不同，适合的应用场景也不同。
- 约束1（$\sum y_j=p$）：站数固定为p（toy model先设5，正式模型这个值需要按预算或敏感性分析确定）。
- 约束2（$\sum x_{ij}=1$）：每个需求点必须完整分配给恰好一个站，不能不分配也不能拆给多个站。
- 约束3（$x_{ij} \le y_j$）：不能把需求分配给一个没被选中的候选点——这条约束把y和x两组变量绑在一起。

**English (item meanings):** $y_j$ — whether candidate site j is opened as a charging station. $x_{ij}$ — the share of demand point i assigned to site j (relaxed, see Section 8). The objective minimises total demand-weighted travel distance to each point's assigned station — this is the key difference from Set Covering (minimise the number of stations needed to cover all demand within a distance threshold) and Maximal Covering (maximise covered demand given a fixed station budget), the two model types you'll build next: each optimises a different objective, suited to different planning questions. Constraint 1 fixes the number of open stations at p. Constraint 2 forces each demand point to be served by exactly one station, with no splitting. Constraint 3 ties the two variable sets together — demand can't be routed to an unopened site.

---

## 8. Continuous Relaxation的数学依据 / Why x_ij Can Be Relaxed to Continuous

**中文：** notebook里把$x_{ij}$设成了continuous（0到1）而不是binary，这是个常见的计算技巧，不是建模上的妥协。原因：给定$y_j$已经确定（哪些站开了）之后，剩下的分配子问题对每个需求点i来说，目标函数里只剩$\sum_j d(i,j) \times x_{ij}$这一项（$D_i$是固定系数，不影响最优方向），约束是$\sum_j x_{ij}=1$且只能在"已开的站"里取正值。这是一个线性目标 + "权重加起来等于1"的约束（一个simplex上的线性优化），最优解一定是把全部权重放在距离最小的那个已开站上——任何"分散"的方案，目标函数值都不会比"全部给最近站"更好，因为线性组合在simplex上的最小值总是在某个顶点（即0-1解）取得。所以即使把$x_{ij}$放宽成continuous，CBC实际求解出来的值也会自动是0或1（这次跑130×347的规模时实测验证过，没出现0-1之间的小数解）。

这样做的好处是计算效率：把45,000多个binary变量降级成continuous变量后，搜索空间小了一个量级，CBC求解时间从可能"卡住/很久"降到几十秒。

**English:** $x_{ij}$ is relaxed to continuous [0,1] rather than binary — a standard computational technique, not a modelling compromise. Given fixed $y_j$ (which sites are open), the remaining assignment subproblem for each demand point i has a linear objective in $x_{ij}$ (with $D_i$ a fixed coefficient that doesn't affect the optimal direction) subject to weights summing to 1 across open sites only — a linear optimisation over a simplex. The optimum of such a problem always places full weight on the single cheapest open site: any "split" assignment can't beat the cheapest-site-only solution, since the minimum of a linear function over a simplex is attained at a vertex (a 0/1 solution). So even with $x_{ij}$ relaxed to continuous, the solver's actual values come out as 0 or 1 in practice — verified empirically at this 130 × 347 scale, with no fractional values appearing.

The practical payoff is solving speed: turning roughly 45,000 binary variables into continuous ones shrinks the search space by an order of magnitude, dropping CBC's solve time from potentially very slow/stuck down to tens of seconds.

**重要补充（加了容量约束之后）：** 上面这个论证严格来说只覆盖"给定$y_j$之后，单个需求点i自己的分配子问题"——在这个子问题里，i跟i之间是互相独立的，所以容易论证最优解是顶点（0/1）解。一旦加上第9节的容量约束（$\sum_i x_{ij} \le N$），不同的i会通过共享同一个$j$的容量被绑在一起，"选哪5个站、5个站之间怎么分配容量"变成一个真正有相互影响的组合问题。这种情况下，$y_j$不再能像无约束版本那样一步从LP relaxation直接读出整数解——实测中CBC需要跑31轮以上的heuristic feasibility pump才把解收敛到整数（虽然最终仍是0个正式branch-and-bound节点，求解时间从约3.5秒涨到约38秒）。也就是说："$x_{ij}$relax成continuous不影响最优性"这个结论本身仍然成立，但"不需要搜索就能拿到整数解"这个额外的好处，只在约束结构比较简单（没有容量这类跨i耦合约束）的时候才稳健成立。

**Important addendum (once the capacity constraint is added):** the argument above strictly covers only the per-demand-point assignment subproblem given fixed $y_j$ — within that subproblem, different i's are independent of each other, which is what makes the vertex-solution argument straightforward. Once Section 9's capacity constraint ($\sum_i x_{ij} \le N$) is added, different demand points become coupled through shared capacity at each site j, and "which 5 sites to open, and how to split their capacity" becomes a genuinely interacting combinatorial problem. In that case $y_j$ can no longer be read off directly from a single LP relaxation the way it could in the unconstrained version — empirically, CBC needed 30+ rounds of heuristic feasibility-pump search to converge $y_j$ to integer values (though it still finished with 0 formal branch-and-bound nodes; solve time rose from ~3.5s to ~38s). In short: relaxing $x_{ij}$ to continuous still doesn't change optimality, but the *extra* benefit of getting an integral solution essentially "for free" without search only holds robustly when the constraint structure is simple (no cross-i coupling constraints like capacity).

---

## 9. 容量约束（2d-1）/ Capacity Constraint

$$\sum_i x_{ij} \le N \quad \forall j$$

**中文：** 这条约束限制每个开放站点最多服务N个需求点（LSOA）。实现上是在原有约束基础上加一行：对每个候选点j，分配给它的需求点数量之和不能超过N。

**关于N的取值——为什么不能是计划里写的N=5：** 容量约束跟约束2（$\sum_j x_{ij}=1$，每个需求点必须被分配）放在一起时，存在一个必要条件：$p \times N \ge n_i$，也就是"开放站点数 × 单站容量"必须能装下全部需求点，否则无论怎么分配都不可能让130个需求点全部找到归宿。计划里的N=5是按"Camden约20个LSOA"那个（不准确的）估计设的——$5\times5=25\ge20$ 当时勉强够用；但Camden实际有130个LSOA，$5\times5=25 < 130$，这个不等式从一开始就不成立，跟数据对不对、代码写得好不好都没关系，是纯数学上的不可行。把N调到30（$5\times30=150\ge130$，留了点余量）之后这条必要条件才满足。

**结果与正确性检查：** N=30时求解结果是Optimal，目标值2,672,093.6，比无约束版本（2c, 2,665,843.7）高出约0.23%。这个方向是必然的——加约束只会压缩可行解集合，约束后的最优目标值不可能比约束前更好，只能持平或变差。差距很小（仅+0.23%）说明无约束版本选出来的那5个站本身容量压力不大，加上N=30这个限制基本不需要大改分配方式。这正是一个很好的"结果有没有错"的检查方式：约束版目标值 ≥ 无约束版目标值，且差距的大小应该跟约束的"紧张程度"成比例。

**English:** this constraint caps the number of demand points (LSOAs) any single open site may serve at N. Implementation-wise it's one additional row per site j summing the assigned demand points.

**On the choice of N — why N=5 (as originally planned) cannot work:** combined with constraint 2 ($\sum_j x_{ij}=1$, every demand point must be assigned), there's a necessary condition: $p \times N \ge n_i$ — total system capacity must be able to absorb every demand point, or no feasible assignment exists regardless of how it's arranged. The plan's N=5 was calibrated against the (inaccurate) estimate of "~20 LSOAs in Camden" — $5\times5=25\ge20$ would have just about worked under that assumption — but Camden actually has 130 LSOAs, so $5\times5=25 < 130$ fails this inequality outright, independent of data quality or code correctness; it's a purely mathematical infeasibility. Raising N to 30 ($5\times30=150\ge130$, with some slack) satisfies the necessary condition.

**Result and sanity check:** with N=30 the model solves to Optimal, objective 2,672,093.6 — about 0.23% higher than the unconstrained version (Section 7, 2,665,843.7). This direction is guaranteed: adding a constraint can only shrink the feasible set, so the constrained optimum can never beat the unconstrained one, only tie or worsen. The small gap (+0.23%) indicates the unconstrained solution's 5 chosen sites weren't under much capacity pressure to begin with, so N=30 barely forces any rearrangement — itself a useful correctness check: constrained objective ≥ unconstrained objective, with the gap size roughly tracking how "tight" the constraint actually is.

---

## 10. 服务半径约束（2d-2）/ Service Radius Constraint

**中文：** 这条约束限制需求点i只能分配给距离它≤800m的候选点j，实现上是直接把超出800m的$x_{ij}$上界设成0（而不是另外加一行约束），效果等价但变量数不变、约束行数更省。

**结果：和容量约束一起加上去之后，模型是Infeasible。** 这不是bug，是几何上算不过来：一个800m半径的服务圈面积约$\pi \times 0.8^2 \approx 2.01\ km^2$，p=5个这种圈（哪怕完全不重叠、排列最优）理论上最多能覆盖$5\times2.01\approx10.1\ km^2$，而Camden全境约21.8 $km^2$——5个站的800m覆盖范围连一半的Camden都盖不住。只要"每个需求点必须被服务"（约束2，$\sum_j x_{ij}=1$）这条硬约束还在，p=5配800m半径，无论怎么挑站点、怎么调容量，都不可能让130个分散在整个borough里的需求点全部落在某个开放站的800m以内。CBC在presolve阶段（0.49秒）就能从约束矩阵的结构里直接判断出矛盾，不需要真正搜索。

**这个发现的意义：** p-median这个框架本身假设"全部需求都必须被服务"，这跟"服务半径"这类覆盖型约束放在一起时，天然要求站点数量要跟着覆盖面积同步放大——这正是Set Covering / Maximal Covering这两类模型存在的原因：它们的目标函数本身就是围绕"覆盖"设计的（Set Covering是"用最少站点覆盖所有点"，Maximal Covering是"给定站点数，覆盖最多点"），能更自然地处理"站点数有限、覆盖不到全部"这种情况，而不需要像p-median这样把覆盖半径硬塞成一条约束。toy model阶段p=5是为了控制问题规模随手设的数字，这个infeasible结果说明：服务半径约束要真正派上用场，要么p要明显设大（实测在均匀分布的模拟数据上大概要到20-30个站才开始可行，真实Camden几何下具体数字会不同，但量级上不会差太远），要么把半径约束改成软约束（比如在目标函数里加超出半径的惩罚项，而不是写成硬约束）。这个取舍值得跟导师讨论，也可以直接写进dissertation的方法论局限性讨论里。

**English:** this constraint restricts demand point i to only be assigned to candidate sites within 800m. Implemented by setting the upper bound of $x_{ij}$ to 0 for any pair beyond 800m (rather than an extra constraint row) — equivalent in effect, with no extra variables and fewer constraint rows.

**Result: combined with the capacity constraint, the model is Infeasible.** This is not a bug — it's a geometric impossibility. An 800m-radius service area covers roughly $\pi \times 0.8^2 \approx 2.01\ km^2$; p=5 such areas, even arranged with zero overlap in the best possible case, cover at most $5\times2.01\approx10.1\ km^2$, against Camden's total area of roughly 21.8 $km^2$ — five 800m catchments can't even cover half of Camden. As long as constraint 2 ($\sum_j x_{ij}=1$, every demand point must be served) remains a hard constraint, no choice of 5 sites or capacity split can get all 130 demand points scattered across the whole borough within 800m of an open site. CBC's presolve phase (0.49s) detects this contradiction directly from the constraint matrix's structure, with no actual search needed.

**What this means:** the p-median framework's built-in assumption that *all* demand must be served sits awkwardly alongside coverage-style constraints like a service radius — radius constraints inherently require facility count to scale with the area needing coverage. This is exactly why Set Covering and Maximal Covering exist as separate model types: their objectives are built around coverage from the start (Set Covering minimises the number of sites needed to cover everyone; Maximal Covering maximises coverage given a fixed site budget), so they handle "too few sites to cover everyone" gracefully instead of becoming infeasible. p=5 was an arbitrary toy-scale choice; this infeasible result shows that for the radius constraint to be usable here, p would need to be substantially larger (synthetic tests on uniformly random data suggest somewhere around 20–30 sites before feasibility kicks in — Camden's actual geometry will differ in the exact number, but not the order of magnitude), or the radius restriction would need to become a soft constraint (e.g. a distance-overage penalty in the objective rather than a hard cutoff). This trade-off is worth raising with your supervisor, and is also a legitimate methodological-limitations point for the dissertation.

---

## 11. 背景参考（非正式）/ Background References (informal)

下面这些是写这份文档时凭一般学术常识列出的背景出处，**不是从你自己的文献矩阵里逐条核对过的**，正式写进dissertation引用列表之前请自己查原文确认作者、年份、期刊等细节：

The following are general academic-knowledge references used while writing this document — **not cross-checked against your own literature matrix**. Verify author names, years, and journal details against the originals before citing formally:

- Hakimi, S.L. — 图论中median/center问题的早期研究（约1964年），p-median模型的理论起点。Early work on median/center problems in graph theory (~1964), the theoretical origin of the p-median model.
- ReVelle, C.S. & Swain, R.W. — 把p-median写成常用0-1整数规划形式（约1970年）。Formalised the p-median problem as the binary integer programme commonly used today (~1970).
- UK Ministry of Housing, Communities & Local Government (现MHCLG) — *English Indices of Deprivation 2019*，IMD官方方法论文档。Official methodology documentation for the 2019 English Indices of Deprivation.
- Department for Transport, UK — *National Travel Survey 2024*，本notebook里$P_d$、$P_{pub}$两个常数的数据来源。Source of the $P_d$ and $P_{pub}$ constants used in this notebook.

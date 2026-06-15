# Demand Estimation Formula — Sources & Literature Basis
# 需求估计公式 — 来源与文献依据

## 公式 / Formula

```
Dᵢ = Pᵢ × Cᵢ × Pd × Ppub × (1 + α × IMDᵢ)
```

这个公式不是某一篇文献的原版公式，而是自己构建的需求估计框架，把几个已有文献中独立使用的变量整合在一起。这是论文 Gap 2 和 Contribution 2 的核心。

This formula is not reproduced from a single source. It is an original demand estimation framework that integrates variables used independently across existing literature. This constitutes the core of Gap 2 and Contribution 2 of the dissertation.

---

## 各部分的文献来源 / Literature Basis by Component

### Pᵢ × Cᵢ（人口 × 有车比例 / Population × Car Ownership Rate）

这是位置-分配模型里最标准的需求代理写法。He et al. (2022) 和大多数 EV 选址文献都用人口或注册车辆数作为需求权重。用 Census TS001 × TS045 是英国语境下最规范的做法。

This is the most standard demand proxy in location-allocation modelling. He et al. (2022) and the majority of EV siting literature use population or registered vehicle counts as demand weights. Using Census TS001 × TS045 is the most rigorous approach in the UK context.

**Key reference:**
> He, J. et al. (2022) 'Optimal deployment of electric vehicle charging stations', *Transportation Research Part D*, 103, pp. 1–18.

---

### Pd（每日充电概率 / Daily Charging Probability）

来自 NTS（National Travel Survey）。NTS 2024 Table NTS0901 报告了英国私家车的平均日行驶里程（约 20–25 英里）。用平均日行驶里程除以主流 EV 续航（约 200 英里）得到充电频率，再转化为日概率。

Derived from the National Travel Survey. NTS 2024 Table NTS0901 reports average daily car trip distances (~20–25 miles). Dividing by a typical EV range (~200 miles) gives a charging frequency, which is converted to a daily probability.

**Key reference:**
> Department for Transport (2024) *National Travel Survey 2024*, Table NTS0901. London: DfT.

**Note:** Retrieve the exact figure from `03_data/demand/nts/` and update `Pd` accordingly. The value 0.1 used in code is an approximation pending verification from NTS0901.

---

### Ppub（依赖公共充电的比例 / Proportion Relying on Public Charging）

来自 NTS 2024 或 Zap-Map 年度调查。约 40% 的 EV 车主没有私人充电条件（主要是无私人车位的公寓居民）。伦敦这个比例更高，因为大量居民住公寓。

Derived from NTS 2024 or Zap-Map annual survey. Approximately 40% of EV drivers lack access to private charging, primarily flat residents without dedicated parking. This proportion is likely higher in London given its high density of flat dwellers.

**Key references (use whichever is accessible):**
> Zap-Map (2023) *EV Driver Survey 2023*. Available at: https://www.zap-map.com/ev-stats/ev-driver-survey/

> RAC Foundation (2023) *Charging Infrastructure for Electric Vehicles*. London: RAC Foundation.

> National Grid ESO (2023) *Future Energy Scenarios 2023*. Warwick: National Grid ESO.

**Note:** Retrieve the exact figure and update `Ppub` accordingly. The value 0.4 used in code is an approximation pending source verification.

---

### (1 + α × IMDᵢ)（公平调节项 / Equity Adjustment Term）

这是自己设计的项，没有直接的前驱文献，但逻辑依据来自以下两个来源：

This term is an original design with no direct precedent, but its rationale draws from two sources:

1. **Lucas et al. (2019)** 关于英国交通能源贫困的研究，指出低收入群体更依赖公共充电（无私人车位）。
   Lucas et al. (2019) on transport energy poverty in the UK, showing that lower-income groups depend more heavily on public charging due to lack of private parking.

2. **Lu et al. (2026)** 的发现，土地用途和社会经济条件影响充电行为和设施利用率。
   Lu et al. (2026), whose findings show that land use and socioeconomic conditions shape charging behaviour and facility utilisation rates.

**Suggested dissertation phrasing:**
> "Following the equity-weighting rationale suggested by Lucas et al. (2019), we introduce a deprivation adjustment term (1 + α × IMDᵢ) to reflect the greater dependence of deprived communities on public charging infrastructure, where α is set to 0.3 as a conservative weighting parameter."

**Key references:**
> Lucas, K., Stokes, G., Haxeltine, A. and Sylvester, J. (2019) *Connecting public transport with opportunities for low income people in rural areas: the role of the third sector*. RAC Foundation.

> Lu, Y. et al. (2026) 'Land use and EV charging utilisation', *Journal of Transport Geography* [forthcoming — update with full citation when available].

---

### α = 0.3（公平权重参数 / Equity Weight Parameter）

按照 dissertation overview 的设定，α 从 0 到 1 做敏感性分析，基准值为 0.3。这个值没有文献直接规定，在论文方法论部分需要说明这是研究者自定义的参数，并通过敏感性分析（α = 0, 0.3, 0.6, 1.0）展示结果对该参数选择的稳健性。

Per the dissertation overview, α ranges from 0 to 1 in the sensitivity analysis, with 0.3 as the baseline. This value is researcher-defined. The dissertation should state this explicitly and demonstrate robustness via sensitivity analysis across α = 0, 0.3, 0.6, 1.0.

---

## IMDᵢ 归一化说明 / IMD Normalisation Note

在公式中 IMDᵢ 需要归一化到 [0, 1] 区间，使用 min-max 归一化：

IMDᵢ in the formula must be normalised to [0, 1] using min-max normalisation:

```
IMDᵢ_normalised = (imd_rank_max − imd_rankᵢ) / (imd_rank_max − imd_rank_min)
```

注意用 `imd_rank`（数值越小越贫困）而不是 `imd_decile`，归一化后值越大表示越贫困，与公平调节项的方向一致。

Note: use `imd_rank` (lower = more deprived), not `imd_decile`. After normalisation, higher values indicate greater deprivation, consistent with the direction of the equity adjustment term.

---

## 待核实事项 / Items to Verify Before Submission

1. 从 `03_data/demand/nts/` 里查 NTS0901，核实 Pd 的实际数值
2. 从 Zap-Map 或 RAC Foundation 报告核实 Ppub 的实际数值
3. 更新代码注释里的参数值和引用
4. Lucas et al. (2019) 的完整引用信息需要核实（作者名和期刊）
5. Lu et al. (2026) 完整引用待发表后更新

1. Check NTS0901 in `03_data/demand/nts/` to verify the exact value of Pd
2. Verify Ppub from Zap-Map or RAC Foundation report
3. Update parameter values and citations in code comments accordingly
4. Verify full citation details for Lucas et al. (2019)
5. Update Lu et al. (2026) full citation once published

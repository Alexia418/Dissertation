EV三篇文献

这一页的作用是：把你读过的三篇核心文献，整理成“它们做了什么、有什么不足、我的论文准备怎么回应这些不足”。
也就是说，这不是普通 literature review slide，而是一个很好的 research gap + your contribution 页面。
你这页的逻辑是：
以前的研究已经做了 EV charging station optimisation / demand modelling / utilisation analysis 但它们各自有不足 所以我的 dissertation 可以在 UK/London context 下，用 multi-objective + equity + demand differentiation 来推进

第一篇：Vazifeh et al. (2019)
这一篇的核心是：
用大规模手机信令数据，也就是 CDR mobility data，来推断人们的出行终点，然后在 1km 网格上估计哪里可能产生充电需求。
它的方法是：
genetic algorithm + weighted set-covering
简单说，就是把城市切成格子，然后问：
选哪些格子建充电站，才能覆盖需求，同时减少司机绕路去充电站的距离？
它的目标是：
减少 excess driving distance 和 charging station 数量。
也就是希望司机少绕路，同时不要建太多站。

它的 limitation 是什么？
你写的是：
Single-objective efficiency only, no equity constraint, CDR cannot distinguish driving from transit, US context
中文解释就是：
这篇文章虽然数据很强，但它主要关注效率，比如距离最短、站点数量最少。它没有把公平性放进目标函数里，也没有考虑 disadvantaged areas 是否被公平服务。
另外，CDR 手机信令数据只能看到人移动了，但不一定知道这个人是开车、坐地铁、坐公交还是步行。所以它用来估计 EV charging demand 时，有一个很大的假设问题。
再加上案例是 Boston，美国城市背景和 London 不完全一样，所以不能直接搬到英国。

Your response 是什么意思？
你写的是：
Multi-objective model with IMD equity term, UK Census + NTS for demand (mode-specific), London context
这就是你准备怎么回应它的不足：
你不是只做效率最优，而是考虑 multi-objective model。除了距离或覆盖率，还加入 IMD equity term，也就是用剥夺指数衡量空间公平性。
同时，你不完全依赖 CDR 这种不能区分交通方式的数据，而是结合 UK Census 和 National Travel Survey。Census 可以提供人口、车辆拥有量等空间基础，NTS 可以帮助你估计私家车出行和公共充电比例。
所以你的改进是：
在 London/UK context 下，把效率、需求和公平性结合起来。

第二篇：He et al. (2022)
这一篇是最接近你 Project 53 的。
它的核心是：
在香港这种高密度城市中，用 demand estimation + capacitated location-allocation model 来优化公共充电站布局。
它的数据是：
stated preference survey + government car park data
也就是问卷调查居民未来是否愿意买 EV，然后结合政府停车场数据，判断哪里可以放充电设施。
它的方法是：
generalized ordered probit demand model + capacitated LA model
中文解释就是：
先用统计模型估计每个区域未来 EV 购买需求，再把需求放进 location-allocation model，考虑容量、预算、服务半径等约束，决定充电站应该怎么布局。
它的目标是：
minimise demand shortfall + travel time
也就是减少没有被满足的充电需求，同时减少用户去充电站的时间。
它还很重要的一点是：
contextualised supply constraints from Hong Kong planning policy
也就是说，它不是纯数学模型，而是把香港的规划政策、停车场、容量、预算放进模型里。这一点非常值得你学。

它的 limitation 是什么？
你写的是：
Survey captures purchase intention not actual behaviour, static demand snapshot, no equity objective, HK context limits UK transferability
中文解释就是：
第一，问卷测的是购买意愿，不是真实行为。一个人说“我未来可能买 EV”，不等于他真的会买，也不等于他会怎样充电。
第二，它的需求估计比较静态。也就是说，它做的是某个时间点或某种情景下的需求快照，没有充分考虑需求随时间变化。
第三，它没有明确把 equity 作为优化目标。
第四，香港是高密度亚洲城市，公共停车场和住宅环境都和英国不同，所以不能直接转移到 London。

Your response 是什么意思？
你写的是：
Revealed behaviour data (Census car ownership + NTS trip patterns), equity objective via IMD 2019, UK planning constraints from OZEV/DfT
这里你的意思是：
你会尽量不用单纯的 stated preference，而是用更接近实际行为的数据或行为代理变量，比如 Census 里的 car ownership，以及 NTS 里的 trip patterns。
同时，你会加入 IMD 2019 作为 equity objective，用来衡量高剥夺地区是否被公平服务。
另外，你会用 OZEV / DfT 的英国政策和规划背景，来替代 He et al. 里的香港规划约束。
所以你的改进是：
把 He et al. 的 contextualised planning approach 转移到 UK/London 语境中，并加入 equity 维度。

第三篇：Lu et al. (2026)
这一篇和前两篇不一样。
它不是直接做充电站选址，而是研究：
现有充电站为什么有的利用率高，有的利用率低？不同充电站是不是有不同的使用模式？
它的 role 是：
Utilisation mechanism study
也就是利用率机制研究。
它提供的证据是：
spatial mismatch drives low utilisation
也就是说，充电站低利用率可能不是因为需求不存在，而是因为站点位置、周边环境、网络关系和真实需求不匹配。
它的方法是：
DTW + K-means++ clustering, XGBoost + SHAP, dynamic spatiotemporal network
中文解释就是：
先用聚类识别不同充电站的使用曲线类型，比如白天高峰、夜间高峰、双峰。然后用 XGBoost 和 SHAP 解释这些类型背后的环境和网络因素，比如价格、人流、交通、周边站点竞争等。

它的 limitation 是什么？
你写的是：
Utilisation-focused not planning-focused, Chinese megacity context, no equity dimension
中文解释就是：
这篇文章主要解释已有站点的利用率机制，而不是直接建立一个严格的选址优化模型。
第二，它的案例是武汉，中国超大城市，城市形态、EV 市场、充电行为和 London 不一样。
第三，它也没有把 equity 作为核心维度。

Your response 是什么意思？
你写的是：
Land-use heterogeneity finding justifies LSOA-level demand differentiation, network effects identified as future work direction
这句话很重要。
你的意思是：
Lu et al. 证明了不同土地利用和城市功能区会产生不同的充电利用率模式。所以你的论文可以据此说明：
London 不同 LSOA 的充电需求不应该被看成一样，而应该根据 land use、car ownership、population、deprivation 等进行 differentiated demand estimation。
也就是说，这篇文章帮你支持一个观点：
需求有空间异质性，所以需要 LSOA-level demand differentiation。
至于 network effects，比如竞争、集聚、溢出效应，你现在不一定要全部做，因为 dissertation 时间有限。但你可以把它作为 future work direction。

这一页整体想表达什么？
你这页其实在讲一个很清晰的文献逻辑：
第一篇告诉你：
mobility data 可以用来估计充电需求，但它偏效率，缺公平，也有交通方式识别问题。
第二篇告诉你：
location-allocation model 可以结合需求、容量、预算和规划约束，但它依赖问卷意愿，缺 equity，且是香港语境。
第三篇告诉你：
充电站利用率受土地利用、时间节奏和网络关系影响，不同地区不能用同一套假设。
所以你的 dissertation 可以定位为：
在 London/UK context 下，结合 Census、NTS、existing charging data 和 IMD，构建一个考虑空间异质性和公平性的 multi-objective EV charging infrastructure optimisation framework。


这三篇文献共同支持我的研究方向：我可以在伦敦语境下做一个多目标充电设施优化模型，同时考虑需求、供给约束和空间公平。


一、公平性在EV充电研究里的核心含义
公平性（equity）在这个领域最根本的问题只有一个：
谁能用到充电站，谁用不到？
但"谁"可以从很多角度定义，这就产生了不同维度的公平性。

二、三种主要公平性框架
框架1：空间可达性公平（Spatial Accessibility Equity）
最直接，也是你最容易操作的。
核心问题：不同地区的居民，到最近充电站的距离是否差距太大？
具体指标：
* 每个LSOA到最近充电站的平均驾车时间/距离
* 服务半径内有充电站覆盖的人口比例
* 充电桩密度（每千人/每平方公里拥有多少个桩）
和你dissertation的联系非常直接：你的p-median模型优化的就是总出行距离，但它是平均意义上的优化。可能结果是城市核心区的人很方便，边缘区的人完全没有站点。加入公平性目标就是要防止这种情况。

框架2：社会经济公平（Socioeconomic Equity）
这是你用IMD最直接对应的维度。
核心问题：贫困社区是否系统性地比富裕社区获得更少的充电资源？
这里有两个相反的逻辑，都值得考虑：
逻辑A — 贫困区需要更多公共充电： 收入低的家庭更可能住公寓、没有私人停车位，所以更依赖公共充电。如果充电站都建在富人区，穷人反而更买不起EV，形成恶性循环。
逻辑B — 贫困区实际EV需求更低： 现在EV主要是有钱人买。所以按需求密度来规划，充电站自然会集中在中高收入区，这不是"歧视"，而是反映现实需求。
这两个逻辑的张力，正是你sensitivity analysis可以探讨的核心——当你调整公平性权重λ时，你实际上是在"强制"模型向贫困区倾斜，而代价是整体效率下降。这个权衡的量化就是你的contribution。
具体指标：
* 各IMD分位的LSOA，平均充电可达性是否有系统性差异
* 最贫困的20% LSOA（Quintile 1），覆盖率比最富裕的20%低多少
* 在优化结果中，充电站是否在空间上与高剥夺区正相关还是负相关

框架3：出行方式公平（Modal Equity）
稍微深一点，但在UK情境里很相关。
核心问题：没有车的人、或者只能用公共交通的人，能否也受益于EV基础设施的扩张？
这个维度对你dissertation来说可能超出范围，但可以在discussion里提一句。比如：如果充电站只建在有私人停车场的地方（购物中心、大型停车场），那么没有车的居民几乎不受益，而他们往往是更贫困的群体。

三、公平性的两种测量方式
在文献里，公平性通常用以下两种方式之一来量化：
方式A：基尼系数式的不平等测量
把所有LSOA按充电可达性排序，看分布有多不均匀。
类似于收入不平等里的基尼系数——值越高代表越不平等。
优点： 一个数字概括全部不平等程度。 缺点： 不能告诉你是哪个群体被落下了。
方式B：分组比较（你最可能用的）
把LSOA按IMD分成五组（Quintile 1到5，从最贫困到最富裕），然后比较：
* 每组的平均到最近充电站距离
* 每组的充电桩密度
* 优化前后，各组的改善幅度是否不同
优点： 直观、好解释、和政策直接挂钩。 缺点： 依赖分组边界的选择。

四、在你的优化模型里怎么加入公平性？
有三种具体做法，从简单到复杂：
做法1：加权目标函数（最推荐，最可行）
在原来的目标函数里加一项：
原来： 最小化 [总出行时间]
加入公平性后： 最小化 [总出行时间] + λ × [贫困区未满足需求的惩罚]
其中：
* λ 是公平性权重，你可以测试不同值
* 贫困区用IMD最低Quintile的LSOA来定义
* 惩罚项 = 这些LSOA的未满足需求量 × 它们的剥夺程度
λ的sensitivity analysis就是你情景比较的核心：
情景	λ值	含义
纯效率	0	完全不考虑公平（Vazifeh的做法）
轻度公平	小	稍微照顾弱势区
重度公平	大	强制向贫困区倾斜
通过比较这些情景，你可以量化"为了公平，我们损失了多少效率"。这个权衡曲线本身就是很有价值的结果。
做法2：约束式公平（最简单）
直接加一个硬约束：
每个IMD最低Quintile的LSOA，必须在距离X公里内有至少一个充电站
这很直接，但有点粗糙，而且可能让模型无解（如果最贫困的地方真的没有合适的候选站点）。
做法3：分组最小覆盖率约束
设定每个IMD分组的最低覆盖率要求，比如：
最贫困的20% LSOA，覆盖率不得低于70%
这比做法2更灵活，也更接近真实政策语言（因为政府说的就是覆盖率目标）。

五、UK情境下特别需要注意的
注意1：EV ownership和需求在英国空间上是不均匀的
2022年英国数据：EV登记量在伦敦和东南部最高，北部和威尔士很低。所以"贫困区需要更多公共充电"的逻辑，在全国层面比在伦敦内部更明显。这影响你研究范围的选择。
注意2：IMD 2019是England-only
如果你只做英格兰（或者只做伦敦），这没问题。但如果你想做全UK，苏格兰、威尔士、北爱尔兰用的是不同的剥夺指数，无法直接比较。这又是一个支持"只做伦敦"的理由。
注意3：公共充电 vs. 工作地充电
在UK政策里，越来越多人讨论"on-street residential charging"——在路边给没有私人停车位的居民提供充电。这和传统的停车场充电站是两个不同的问题。你的模型主要针对哪种场景，需要说清楚。

六、对你meeting最实用的一句话表达
如果导师问"你怎么考虑公平性"，你可以说：
"I plan to add a deprivation-weighted equity term to the objective function, using IMD 2019 at the LSOA level. The idea is to penalise unmet demand in deprived areas more heavily. I'll then vary the weight λ across scenarios — so I can quantify the trade-off between overall efficiency and equity outcomes. That trade-off curve is actually one of the main outputs I'm hoping to produce."
这一段说出来，导师会觉得你想得很清楚了。


Q1 LSOA 和 MSOA 是什么？

先说最简单的理解
英国做统计和规划的时候，需要把全国切成很多小块，每块代表一个"社区"。
LSOA 和 MSOA 就是这种切法的两个不同尺度。
就像地图上的缩放—— LSOA 是放大看，MSOA 是缩小一点看。

LSOA
Lower Layer Super Output Area
下层超级输出区域。
名字很奇怪，但理解起来很简单：
大概就是一个街道或者一两个住宅小区的大小。
具体数字：
* 英格兰和威尔士共有约 32,844 个 LSOA
* 每个 LSOA 平均人口：约 1,500 人
* 最小约 1,000 人，最大约 3,000 人
* 面积差异很大——城市里很小（几个街区），农村可能很大（好几个村）
设计目的：
每个 LSOA 的人口数量大致相同。这样做统计比较时，不会因为一个区人口是另一个区的十倍而导致结果失真。
伦敦举例：
伦敦市中心一个 LSOA 可能就是几条街围起来的区域，走路五分钟可以穿过去。

MSOA
Middle Layer Super Output Area
中层超级输出区域。
比 LSOA 大一级，大概是一个镇或者一个大社区的大小。
具体数字：
* 英格兰和威尔士共有约 7,201 个 MSOA
* 每个 MSOA 平均人口：约 7,500 人
* 大概是 5 到 7 个 LSOA 合并在一起

两者的层级关系
英国的地理统计层级是这样的，从小到大：
Output Area（OA）          最小，约 300 人
      ↓
Lower Layer SOA（LSOA）    约 1,500 人   ← 你dissertation用的
      ↓
Middle Layer SOA（MSOA）   约 7,500 人
      ↓
Local Authority District   地方政府区（比如 Camden、Southwark）
      ↓
Region                     大区（比如 London、South East）
      ↓
Country                    英格兰、威尔士、苏格兰、北爱尔兰
每一层都可以由下一层合并得到。
比如一个 MSOA = 若干个 LSOA 加在一起。

为什么你的dissertation用 LSOA？
有三个原因：
原因1：IMD 数据是 LSOA 级别的
Index of Multiple Deprivation（剥夺指数）是英格兰最重要的社会经济数据，它的最小空间单位就是 LSOA。如果你用更粗的 MSOA，很多细节就丢失了——一个 MSOA 里可能有很富的街区也有很穷的街区，平均掉就看不出来了。
原因2：Census 2021 在 LSOA 级别有车辆保有量数据
你估计需求的关键变量——每个区域有多少私家车、多少家庭没有停车位——在 LSOA 级别都有。
原因3：空间精度足够，计算量又不太大
如果你用 Output Area（比 LSOA 更小），伦敦就有 25,000 多个，计算量很大，而且数据质量也变差（样本太小）。
LSOA 是精度和可行性之间比较好的平衡点。

一个具体例子帮你感受一下尺度
伦敦 Camden 区（Camden Borough）：
层级	数量
LSOA	约 130 个
MSOA	约 28 个
Borough（Camden 本身）	1 个
所以如果你在 LSOA 层面做分析，Camden 这一个区你就有 130 个数据点，分辨率已经很高了。
整个大伦敦（Greater London）有约 4,835 个 LSOA，这就是你模型里的需求点数量。

两个常见的混淆点
混淆1："LSOA 是固定不变的吗？"
不是。每隔十年，ONS（英国国家统计局）会根据人口变化重新划定边界。2021 年 Census 之后更新了一次，新的叫 LSOA 2021，旧的叫 LSOA 2011。
你用的时候要确认数据用的是哪一版，不能混用。IMD 2019 用的是 LSOA 2011 的边界，Census 2021 用的是新边界，需要做一个边界匹配（spatial join）。
混淆2："LSOA 和邮编（postcode）是一回事吗？"
不是。邮编是邮政系统用的，一个 LSOA 大概对应 15 到 20 个完整邮编，但边界不完全重合。统计分析一般用 LSOA，不用邮编。

总结一句话
LSOA 就是英国做地理统计用的"最小社区单元"，大概 1,500 人，是你dissertation里需求估计和公平性分析的基本空间单位。MSOA 是它大一级的版本，约 7,500 人，通常用来做更宏观的区域分析。

Q2 剥夺系数一般对应英文里的 deprivation index，在英国最常见的是 Index of Multiple Deprivation, IMD。
简单说：
剥夺系数 = 衡量一个地区“社会经济劣势程度”的指标
它不是说某个人“穷不穷”，而是衡量一个小区域整体上是不是更 disadvantaged。
比如一个地区如果有：
* 收入水平低
* 失业率高
* 教育水平低
* 健康状况差
* 犯罪率高
* 住房条件差
* 公共服务可达性差
* 居住环境差
那么这个地区的 deprivation level 就会更高。

用人话理解
它回答的是：
这个地方是不是比较缺资源、机会和公共服务？
所以它不是单一的“收入指标”，而是一个综合指标。
比如两个地区：
A 区
收入低、失业多、健康差、住房拥挤、教育水平低 → deprivation 高
B 区
收入高、就业稳定、教育好、医疗和交通方便 → deprivation 低

在英国数据里怎么用？
英国常用的是 IMD rank / IMD score。
一般会把地区分成几组，比如：
* most deprived 10%
* most deprived 20%
* least deprived 20%
你在论文里经常会看到：
areas with higher deprivation more deprived neighbourhoods deprivation deciles IMD score

注意：score 和 rank 不一样
这个很容易搞混。
IMD score
数值越高，通常表示剥夺程度越严重。
IMD rank
排名越靠前，通常表示越贫困/越剥夺。 比如 rank = 1 往往是最 deprived 的地区。
所以用数据前一定要看清楚字段说明。

放到你的 EV charging 论文里是什么意思？
如果你研究充电桩空间布局，剥夺系数可以用来讨论 spatial equity / social equity。
比如：
充电基础设施是不是只集中在富裕地区？ 高剥夺地区是否缺少公共充电设施？ 优化模型是否会因为 EV demand 低而忽视 disadvantaged areas？ 如果只按当前 EV ownership 布局，会不会加剧基础设施不平等？
这个很重要，因为高剥夺地区可能现在 EV 拥有率低，但这不代表它们不需要公共基础设施。尤其如果未来二手 EV 增加，或者居民没有私人停车位，公共/路边充电设施反而更关键。

你可以这样写进 dissertation
Deprivation index measures the relative socioeconomic disadvantage of small areas. In this study, it can be used as an equity-related variable to examine whether EV charging infrastructure is unevenly distributed across more and less deprived neighbourhoods.
中文意思：
剥夺指数衡量小区域的相对社会经济劣势。在本研究中，它可以作为公平性变量，用来分析电动车充电基础设施是否在高剥夺和低剥夺社区之间分布不均。

和你项目最相关的理解
你不要把它只当作“穷人区指标”。更准确是：
它是衡量地区多维弱势程度的变量。
在你的 EV charging infrastructure planning 里，它可以帮助你避免模型只追求效率，而忽视公平。

Q3
这两个是把 EV purchase demand（潜在买 EV 的人/车） 转换成 daily public charging demand（每天公共充电需求） 的两个比例参数。
公式是：
Dd = D × Pd × Ppub
其中 He et al. (2022) 解释：D 是每个 neighbourhood 的 EV purchase demand，Pd 是一天内需要充电的 EV 比例，Ppub 是使用公共充电桩的 EV 比例。

Pd 是什么？
Pd = percentage of EVs charging on a single day
中文就是：
一天内会充电的 EV 比例
比如一辆 EV 平均 4 天充一次电，那每天大约有：
1 / 4 = 0.25
所以 Pd = 0.25。
意思不是“25% 的人有 EV”，而是：
在所有 EV 里面，平均每天有 25% 会产生一次充电需求。

Ppub 是什么？
Ppub = percentage of EVs using public chargers
中文就是：
使用公共充电桩的 EV 比例
比如问卷发现大约一半 EV 用户主要使用公共充电网络，那么：
Ppub = 0.5
意思是：
不是所有 EV 充电都发生在公共充电桩，有些人在家充、有些在私人停车位充、有些在公司充。Ppub 只保留公共充电需求这一部分。

用一个例子理解
假设某个 LSOA 估计有：
D = 100 个潜在 EV 用户/EV demand
如果：
Pd = 0.25 Ppub = 0.5
那么：
Dd = 100 × 0.25 × 0.5 = 12.5
意思是：
这个 LSOA 每天大约有 12.5 个 EV 公共充电需求。
你 PPT 里可以写：
Pd: daily charging probability, estimated from average charging frequency.Ppub: proportion of EV users relying on public chargers.

deprived areas 欠发达地区
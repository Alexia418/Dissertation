
Geographic Information Systems and Science

最直接的联系。你用GeoPandas处理LSOA边界shapefile、画choropleth地图、做空间叠加分析（需求图叠充电站分布图）、计算质心坐标。候选点生成也是GIS操作。你的论文本质上是一个GIS驱动的决策支持研究。
Quantitative Methods

需求估计公式Di就是定量方法的产物——多变量加权合成指标。敏感性分析、Gini系数计算、IMD分组覆盖率对比，都是定量分析。你的模型也需要验证（比较baseline和equity情景的统计差异）。
Foundations of Spatial Data Science

数据处理的基础——Census数据清洗、LSOA代码匹配、多源数据merge、缺失值处理。你用pandas处理TS001、TS045、IMD2019的过程就是这门课的直接应用。
Building Spatial Applications with Big Data

导师提供的四个GLA CSV本质上是大规模运营数据（device_all_uk涵盖全英国充电设备）。你对这些数据进行过滤、关联、空间化，是spatial big data处理的典型案例。
Data Science for Spatial Systems

位置-分配模型是一个优化问题，PuLP求解是data science工具链的一部分。距离矩阵计算（n×m规模）、模型结果的可视化分析，都属于这门课的范畴。
Smart Cities: Context, Policy and Government

你的研究直接服务于智慧城市基础设施规划。EV充电网络是智慧交通系统的组成部分，你把政策约束（EV Infrastructure Strategy 2022、英国规划政策）编码进模型，正是"policy-informed spatial planning"的体现。
Urban Systems Theory

你对伦敦充电需求的空间异质性分析，背后是urban systems的理论逻辑——城市不是均质的，商业区、居住区、混合用途区产生不同的需求模式（Lu et al. 2026的发现）。IMD作为城市社会空间分异的测量工具，也是urban systems理论的经典应用。
Agent-Based Modelling for Spatial Systems

这门课和你dissertation的直接联系最弱，但底层逻辑是相通的——ABM关注个体行为涌现出系统结果，你的模型关注个体LSOA需求聚合成系统级的充电网络布局。更重要的是，你的需求估计里的Pd（日充电概率）和Ppub（依赖公共充电比例）这两个行为参数，本质上就是对EV用户行为的建模，只是简化成了概率系数而不是agent规则。
Urban Spatial Science作为专业的整体联系

Urban Spatial Science的核心是"用空间数据和计算方法解决城市问题"。你的dissertation正好是这个定义的完整示范：城市问题（EV充电不足且不公平）+ 空间数据（Census、IMD、OSM、GLA运营数据）+ 计算方法（GIS + 优化模型）+ 政策建议（告诉规划者建在哪里）
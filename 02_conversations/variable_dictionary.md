# Data Variable Dictionary / 数据变量字典

## Data Structure Overview / 数据层级结构

```
Location（地点）         → OpenStreetEV_GLA.csv
    └── EVSE（充电站）   → gla_location_evse_join.csv + evse_status_gla.csv
            └── Device（连接器/插口）  → device_all_uk.csv
```

**Key principle / 核心概念：**
- **Location** = a physical site with one or more charging stations / 一个物理地点，包含一个或多个充电站
- **EVSE** (Electric Vehicle Supply Equipment) = a charging station / 一台充电站（老师定义）
- **Device** = a connector/port on a charging station / 充电站上的一个插口/连接器（老师定义）

**Join logic / 连接逻辑：**
- `OpenStreetEV.location_id` → `gla_join.location_id` → `gla_join.evse_id` = `evse_status.evse_uid`
- `OpenStreetEV.location_id` → `device.location_id`
- evse_id in gla_join (38,375) fully covers evse_uid in evse_status (28,865) / gla_join 的 evse_id 完全覆盖 evse_status 的 evse_uid
- device.location_id (53,806 UK-wide) overlaps with osev.location_id (23,015 GLA) / device 是全英国数据，与 GLA 地点完全匹配

---

## 1. OpenStreetEV_GLA.csv — Location Level / 地点层级

| Variable / 变量 | Type / 类型 | Meaning (EN) | 含义（中文） |
|----------------|-------------|--------------|-------------|
| `location_id` | string | Unique identifier for each physical charging site | 每个充电地点的唯一ID，整个数据集的主键 |
| `external_uuid` | string | External UUID from source system (OCPI standard) | 来自源系统的外部UUID（OCPI标准） |
| `location_name` | string | Name of the charging site (e.g. "Lidl Dagenham") | 充电地点名称（如超市名、街道名） |
| `address` | string | Street address of the location | 街道地址 |
| `postcode` | string | UK postcode of the location | 英国邮政编码 |
| `borough` | string | London borough the location falls within | 所在 London borough |
| `latitude` | float | WGS84 latitude coordinate | WGS84 纬度坐标 |
| `longitude` | float | WGS84 longitude coordinate | WGS84 经度坐标 |
| `location_category` | string | Broad category of site (e.g. Supermarket, On-Street, Accommodation) | 地点大类（如超市、路边、住宿） |
| `location_type` | string | Detailed type of site (e.g. Destination, On-street) | 地点细分类型（如目的地型、路边型） |
| `country_code` | string | Country code (always "GB" in this dataset) | 国家代码（本数据集均为"GB"） |

---

## 2. gla_location_evse_join.csv — Location–EVSE Join Table / 地点与充电站连接表

| Variable / 变量 | Type / 类型 | Meaning (EN) | 含义（中文） |
|----------------|-------------|--------------|-------------|
| `location_id` | string | Foreign key linking to OpenStreetEV_GLA.location_id | 外键，对应 OpenStreetEV 的地点ID |
| `evse_id` | string | Unique identifier for each EVSE (charging station) at this location | 该地点下每台充电站（EVSE）的唯一ID |

**Key stats / 关键数字：**
- 23,010 unique locations / 23,010 个唯一地点
- 38,375 unique EVSEs / 38,375 个唯一充电站
- Mean 1.67 EVSEs per location, max 116 / 平均每地点 1.67 台充电站，最多 116 台

---

## 3. evse_status_gla.csv — EVSE Status Records / 充电站状态记录

| Variable / 变量 | Type / 类型 | Meaning (EN) | 含义（中文） |
|----------------|-------------|--------------|-------------|
| `uid` | string | Unique ID for each individual status record; prefix "o-" or "c-" | 每条状态记录的唯一ID；前缀 o- 或 c- 可能代表 open/closed |
| `evse_uid` | string | EVSE identifier; matches evse_id in gla_location_evse_join | 充电站唯一ID，与 gla_join 中的 evse_id 对应 |
| `last_updated` | datetime | Timestamp when this status record was logged | 该状态记录的时间戳 |
| `status` | string | Operational status of the EVSE at that moment | 该时刻该充电站的运营状态 |

**Status values / 状态值含义：**

| Status | Meaning (EN) | 含义（中文） |
|--------|--------------|-------------|
| `AVAILABLE` | Ready to charge, no vehicle connected | 空闲可用，无车连接 |
| `CHARGING` | Vehicle connected and actively charging | 正在充电中 |
| `OUTOFORDER` | Fault or maintenance, not usable | 故障或维护中，不可用 |
| `UNKNOWN` | Status cannot be determined | 状态未知 |
| `BLOCKED` | Physically blocked, e.g. by a non-EV vehicle | 被堵塞（如非电动车占位） |
| `INOPERATIVE` | Temporarily out of service | 暂时停止运营 |
| `RESERVED` | Pre-booked by a user | 已被预约 |

**Key stats / 关键数字：**
- 25,461,562 total records / 共 2546 万条记录
- 28,865 unique EVSEs / 28,865 个唯一充电站
- ~882 records per EVSE on average / 每台充电站平均约 882 条记录

---

## 4. device_all_uk.csv — Device (Connector) Level / 连接器层级

| Variable / 变量 | Type / 类型 | Meaning (EN) | 含义（中文） |
|----------------|-------------|--------------|-------------|
| `location_id` | string | Foreign key linking to OpenStreetEV_GLA.location_id | 外键，对应 OpenStreetEV 的地点ID |
| `zapmap_device_uid` | string | Unique identifier for each individual connector/port | 每个连接器/插口的唯一ID（Zapmap系统内部ID） |
| `power_band` | string | Power category of the connector | 功率档位分类 |
| `device_max_power` | float | Maximum power output in **Watts (W)**, not kW | 最大输出功率，单位是**瓦（W）**，不是千瓦 |

**Power band values / 功率档位含义：**

| power_band | Typical range / 典型功率范围 | Use case / 使用场景 |
|------------|------------------------------|-------------------|
| `1. Slow` | 2.76–3.6 kW | Overnight home-style charging / 慢充，适合过夜停车 |
| `2. Fast (AC)` | 7–22 kW AC | Public car parks, destinations / 快充交流电，适合停车场 |
| `2. Fast (DC)` | 7–22 kW DC | Less common fast DC / 快充直流电，较少见 |
| `3. Rapid` | 25–99 kW | Motorway services, rapid charging hubs / 急速充电 |
| `4. Ultra-rapid` | 100–690 kW | High-power hubs only / 超快充，仅大型充电枢纽 |

**Note / 注意：** `device_max_power` 单位是 W，需除以 1000 转换为 kW。例：2760W = 2.76kW，690000W = 690kW。

**Key stats / 关键数字：**
- 133,263 devices UK-wide / 全英国 133,263 个连接器
- 53,806 unique locations UK-wide / 全英国 53,806 个地点
- 23,015 GLA locations fully matched / 与 GLA 地点完全匹配（23,015个）
- 33,249 devices within GLA after filtering / 筛选后 GLA 范围内 33,249 个连接器

---

## 5. Derived / Aggregated Variables / 衍生变量（清洗后新增）

### census_london_clean.csv

| Variable / 变量 | Meaning (EN) | 含义（中文） |
|----------------|--------------|-------------|
| `total_households` | Sum of all household car-ownership categories | 各车辆持有类别家庭数之和，即总家庭数 |
| `total_cars` | Weighted sum: cars_1×1 + cars_2×2 + cars_3plus×3 | 加权车辆总数（3辆以上保守估算为3辆） |
| `car_ownership_rate` | Proportion of households with at least one car | 有车家庭占总家庭的比例 |

### imd_london_clean.csv

| Variable / 变量 | Meaning (EN) | 含义（中文） |
|----------------|--------------|-------------|
| `imd_imputed` | Flag: 1 = IMD value filled using borough median (new 2021 LSOA); 0 = original IMD value | 填充标志：1 表示该 LSOA 无原始 IMD 数据，已用 borough 中位数填充；0 表示原始数据 |

### evse_availability_clean.csv

| Variable / 变量 | Meaning (EN) | 含义（中文） |
|----------------|--------------|-------------|
| `total_records` | Total number of status records for this EVSE | 该充电站的状态记录总条数 |
| `availability_rate` | Proportion of records with status = AVAILABLE | AVAILABLE 状态占总记录的比例（空闲率） |
| `utilisation_rate` | Proportion of records with status = CHARGING | CHARGING 状态占总记录的比例（使用率） |

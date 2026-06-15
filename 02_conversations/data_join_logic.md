# 数据连接逻辑详解 / Data Join Logic

---

## 第一条连接链：地点 → 充电站 → 状态

```
osev → gla_join → evse_availability
```

### 第一步：osev 和 gla_join 通过 location_id 连接

```
osev:
location_id | borough  | latitude | ...
TSM91W3     | Camden   | 51.53    | ...

gla_join:
location_id | evse_id
TSM91W3     | 174b834d444d9c65929b489ce7e76fb6
TSM91W3     | e75ec419ba4f4e378994795885a17595
TSM91W3     | 5c2d9027b4d343139d5cce55f45193ce
```

意思是：Camden 这个地点（TSM91W3）下面有 3 台充电站（EVSE），每台都有自己的 evse_id。

---

### 第二步：gla_join 的 evse_id 等于 evse_availability 的 evse_uid

```
evse_availability:
evse_uid                             | availability_rate | utilisation_rate
174b834d444d9c65929b489ce7e76fb6    | 0.51              | 0.32
e75ec419ba4f4e378994795885a17595    | 0.43              | 0.41
5c2d9027b4d343139d5cce55f45193ce    | 0.67              | 0.18
```

意思是：TSM91W3 这个地点的 3 台充电站，各自有不同的空闲率和使用率。

---

### 把这条链连完之后你能得到：

```
location_id | borough | latitude | longitude | evse_uid | availability_rate | utilisation_rate
TSM91W3     | Camden  | 51.53    | -0.14     | 174b8... | 0.51              | 0.32
TSM91W3     | Camden  | 51.53    | -0.14     | e75ec... | 0.43              | 0.41
TSM91W3     | Camden  | 51.53    | -0.14     | 5c2d9... | 0.67              | 0.18
```

**用途：** 知道每个地点的充电站实际使用情况，可以识别哪些地点长期空闲（供过于求），哪些长期繁忙（供不应求）。

---

## 第二条连接链：地点 → 连接器/插口

```
osev → device
```

### 直接通过 location_id 连接

```
osev:
location_id | borough           | latitude | ...
6SO91US     | Barking&Dagenham  | 51.55    | ...

device:
location_id | zapmap_device_uid | power_band | device_max_power
6SO91US     | 98ANCHF           | 1. Slow    | 2760
6SO91US     | KJH72MS           | 2. Fast AC | 7360
```

意思是：这个地点有 2 个插口，一个慢充（2.76kW），一个快充（7.36kW）。

---

### 把这条链连完之后你能得到：

```
location_id | borough          | latitude | power_band  | device_max_power
6SO91US     | Barking&Dagenham | 51.55    | 1. Slow     | 2760
6SO91US     | Barking&Dagenham | 51.55    | 2. Fast AC  | 7360
```

**用途：** 知道每个地点提供什么功率的充电服务，可以计算每个地点的总装机容量（所有插口功率之和）。

---

## 两条链的关系总结

```
第一条链回答：这个地点的充电站忙不忙？
第二条链回答：这个地点能提供多大的充电功率？
```

合在一起，你就能对每个 location 描述：

| 问题 | 数据来源 |
|------|----------|
| 在哪里？ | osev（latitude, longitude） |
| 在哪个 borough？ | osev（borough） |
| 有几台充电站？ | gla_join（count of evse_id） |
| 充电站使用率如何？ | evse_availability（utilisation_rate） |
| 提供什么功率？ | device（power_band, device_max_power） |
| 总装机容量多少？ | device（sum of device_max_power） |

这张完整的供给侧表就是之后位置-分配模型里"现有供给"的输入数据。

---

## 完整连接代码示意（供参考）

```python
# 第一条链：location → evse → availability
supply_evse = osev_clean.merge(gla_join, on="location_id", how="left")
supply_evse = supply_evse.merge(
    evse_clean.rename(columns={"evse_uid": "evse_id"}),
    on="evse_id",
    how="left"
)

# 第二条链：location → device
supply_device = osev_clean.merge(device_gla, on="location_id", how="left")

# 聚合到 location 层级
supply_summary = osev_clean.copy()

# 每个地点的 EVSE 数量
evse_count = gla_join.groupby("location_id")["evse_id"].count().reset_index()
evse_count.columns = ["location_id", "evse_count"]

# 每个地点的平均可用率
evse_avail = supply_evse.groupby("location_id")["availability_rate"].mean().reset_index()

# 每个地点的总装机容量（W → kW）
device_capacity = supply_device.groupby("location_id")["device_max_power"].sum().reset_index()
device_capacity["total_capacity_kw"] = device_capacity["device_max_power"] / 1000

# 合并成供给侧汇总表
supply_summary = supply_summary \
    .merge(evse_count, on="location_id", how="left") \
    .merge(evse_avail, on="location_id", how="left") \
    .merge(device_capacity[["location_id", "total_capacity_kw"]], on="location_id", how="left")
```

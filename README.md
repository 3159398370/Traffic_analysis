
# 交通数据分析代码详解 交通预测--机器学习相关代码
学校项目复刻学习 Traffic_analysis

## 一、代码整体流程

数据加载→数据清洗→可视化→建模→评估

## 二、分步解析

### 1. 数据准备阶段

```
# 加载原始数据（单元88）
traffic_df = pd.read_csv("Traffic.csv")          # 1个月的数据
traffic_two_month_df = pd.read_csv("TrafficTwoMonth.csv")  # 2个月的数据

# 合并数据集（单元92）
combined_df = pd.concat([traffic_df, traffic_two_month_df]) # 把两个表格上下拼接
combined_df['Source'] = 'OneMonth'               # 添加来源标识
```

1. **数据合并**  
   - 合并两个来源的交通数据添加`Source`列标识来源。
   - 合并后数据集包含车辆计数、时间、星期几、交通状况等字段。

2. **数据总览**  
   - 查看数据字段类型（如时间、车辆类型计数）和统计概况（均值、极值）。
   - 初步观察数据规模（行数/列数）和分布特征。

#### 2.数据可视化

```
# 车辆数量分布（单元93）
fig = make_subplots(rows=2, cols=2, subplot_titles=("Car Counts", "Bike Counts", "Bus Counts", "Truck Counts"))
fig.add_trace(go.Histogram(x=combined_df['CarCount'], name='Car Counts'), row=1, col=1)
# ... 其他车辆类型的直方图

# 交通状况分布（单元94）
fig = px.pie(combined_df, names='Traffic Situation', title='Traffic Situation Distribution')
```

##### 2.1. 基础分布分析

实现原理 ：

- 直方图：统计各车辆类型数量分布的频次
- 饼图：计算各类交通状况的占比
- 箱线图：展示工作日车流量的四分位分布

1. **车辆分布可视化**  
   - 直方图展示汽车、自行车、公交车、卡车的数量分布。
   - 饼图显示交通状况（拥堵/繁忙/正常/畅通）的比例。
2. **时间与日期分析**  
   - **按星期几**：盒图比较工作日与周末的车辆数量差异（如周末卡车减少）。
   - **按小时**：识别早晚高峰（如早8点、晚6点车辆总数激增）。
   - 折线图展示一天中不同车辆类型的平均数量变化趋势。
3. **交通状况关联分析**  
   - 盒图对比不同交通状况下车辆数量（如拥堵时汽车数量显著增加）。
   - 热力图揭示车辆类型间的相关性（如公交车与卡车数量正相关）。
4. **其他关键分析**  
   - 方差分析：卡车数量波动最大，自行车数量最稳定。
   - 数据源对比：`TwoMonth`数据集平均车辆数略高于`OneMonth`。

分析方法 ：

- 小时粒度：解析24小时车流量波动规律
- 周粒度：对比工作日/周末分布模式
- 长周期：捕捉月度趋势变化

## 三、数据预处理

```
graph TD
A[原始特征] --> B{ColumnTransformer}
B --> C[数值标准化]
B --> D[类别编码]
B --> E[特征联合]
E --> F[随机森林]
F --> G[预测交通状况]
```



1. **异常值处理**  
   - 用IQR方法移除极端值（如某条记录中卡车数量超过正常范围5倍）。
   - 箱线图确认处理后数据分布更集中。

2. **缺失值与重复值**  
   - 删除包含缺失值的记录（如缺失天气条件的行）。
   - 移除重复数据（如完全相同的两条时间戳记录）。

3. **数据归一化**  
   - 使用`QuantileTransformer`将车辆计数转化为正态分布，消除量纲差异。

4. **特征工程**  
   - 提取小时特征（从`Time`拆分出`Hour`）。
   - 构造`Weekend`标记（周末为1，工作日为0）。
   - 将星期几编码为数值（如星期一=0，星期二=1）。

---

## 四、建模与预测

1. **数据拆分**  

   - 按8:2划分训练集和测试集，保证模型评估独立性。

2. **模型构建**  

   - **随机森林**：通过多棵决策树投票决策，准确率99.0%。
   - **XGBoost**：梯度提升优化模型，准确率99.9%（最佳性能）。
   - **SVM**：寻找最优分类超平面，准确率91.7%。
   - **梯度提升**：逐步修正错误，准确率与XGBoost持平。

3. **模型评估**  

   - 对比指标：  

     | 模型     | 准确率 | 精确率 | 召回率 | F1分数 |
     | -------- | ------ | ------ | ------ | ------ |
     | 随机森林 | 99.0%  | 99.0%  | 99.0%  | 99.0%  |
     | XGBoost  | 99.9%  | 99.9%  | 99.9%  | 99.9%  |
     | SVM      | 91.7%  | 91.7%  | 91.7%  | 91.5%  |
     | 梯度提升 | 99.9%  | 99.9%  | 99.9%  | 99.9%  |

---

## 五、核心输出

1. **探索性结论**  
   - 交通高峰时段：早8点和晚6点为最繁忙时段。
   - 车辆关联性：汽车与公交车数量呈强正相关（相关系数0.71）。
   - 数据源差异：`TwoMonth`数据集平均车辆数比`OneMonth`高15%。

2. **模型结果**  
   - XGBoost和梯度提升模型表现最佳，准确率接近100%。
   - 模型可预测未来15分钟的交通状况，帮助优化信号灯控制和路线规划。

---

## 总结

代码从合并数据开始，通过图表分析交通规律（比如周五最堵、卡车数量波动大），清理异常数据并标准化数值

接着用四种机器学习模型预测交通状况，最终发现XGBoost和梯度提升模型最准（准确率99.9%）

整个分析帮助交通部门预判拥堵，比如在高峰时段增派交警或调整公交班次。


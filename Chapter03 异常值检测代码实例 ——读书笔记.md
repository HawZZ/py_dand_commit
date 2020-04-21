Chapter03 数据探索 ——读书笔记

1. 数据质量分析
   1. 缺失值分析
   2. 异常值分析
      1. 简单统计量分析
      2. 3σ原则：超过3倍标准差
      3. 箱型图分析
   3. 一致性分析

3-1 箱型图代码实例

```python
# -*- coding: utf-8 -*-

# 代码3-1 使用describe()方法即可查看数据的基本情况
import pandas as pd
catering_sale = '../data/catering_sale.xls'  # 餐饮数据
data = pd.read_excel(catering_sale, index_col = u'日期')  # 读取数据，指定“日期”列为索引列
print(data.describe()) # 打印数据基本情况



# 代码3-2 餐饮销额数据异常值检测

import matplotlib.pyplot as plt  # 导入图像库
plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False  # 用来正常显示负号

plt.figure()  # 建立图像画布640*480
p = data.boxplot(return_type='dict')  # 画箱线图，直接使用DataFrame的方法，需要转换箱线图的字典格式
x = p['fliers'][0].get_xdata()  #  'flies'即为异常值的标签
y = p['fliers'][0].get_ydata()
y.sort()  # 从小到大排序，该方法直接改变原对象
'''
用annotate添加注释
其中有些相近的点，注解会出现重叠，难以看清，需要一些技巧来控制
以下参数都是经过调试的，需要具体问题具体调试。
annotate方法用于注释图标样式
'''
for i in range(len(x)):
    if i>0:
        plt.annotate(y[i], xy = (x[i],y[i]), xytext=(x[i]+0.05 -0.8/(y[i]-y[i-1]),y[i]))
    else:
        plt.annotate(y[i], xy = (x[i],y[i]), xytext=(x[i]+0.08,y[i]))

plt.show()  # 展示箱线图
```

运行结果

```
                销量
count   200.000000 # 非空值数
mean   2755.214700 # 平均值
std     751.029772 # 标准差
min      22.000000 # 最小值
25%    2451.975000 # 1/4分位数
50%    2655.850000 # 2/4分位数
75%    3026.125000 # 3/4分位数
max    9106.440000 # 最大值
```

<img src="/Users/hawzz/Documents/pd_dand_commit/Chap3_1.png" alt="Chap3_1" style="zoom:50%;" />

通过

```python
len(data)
```

可知数据记录为201条，运行结果count为200，即非空值数200条，缺失值为1条。

从箱型图中可以看出 22、51、60、865、4065.2、4060.3、6607.4、9106.44  为7个异常值。结合具体业务可以把865、4065.2、4060.3归为正常值，确定过滤规则为400<sale<500。



2. 数据特征分析
   1. 分布分析
      1. 定量数据分布分析
      2. 定型数据分布分析
   2. 对比分析
   3. 统计量分析
      1. 集中趋势度量
      2. 离中趋势度量
   4. 周期性分析
   5. 贡献度分析（帕累托分析，20/80定律）
   6. 相关性分析
      1. 直接绘制散点图
      2. 绘制散点图矩阵
      3. 计算相关系数

3-2 数据统计量分析代码实例

```python
#-*- coding: utf-8 -*-
#餐饮销量数据统计量分析
import pandas as pd

catering_sale = '../data/catering_sale.xls'  # 餐饮数据
data = pd.read_excel(catering_sale, index_col = u'日期')  # 读取数据，指定“日期”列为索引列
data = data[(data[u'销量'] > 400) & (data[u'销量'] < 5000)] # 过滤异常数据
statistics = data.describe() # 保存基本统计量

statistics.loc['range'] = statistics.loc['max'] - statistics.loc['min'] # 极差
statistics.loc['var'] = statistics.loc['std'] / statistics.loc['mean'] # 变异系数
statistics.loc['dis'] = statistics.loc['75%'] - statistics.loc['25%'] # 四分位数间距

print (statistics)
```

运行结果

```
                销量
count   195.000000
mean   2744.595385
std     424.739407
min     865.000000
25%    2460.600000
50%    2655.900000
75%    3023.200000
max    4065.200000
range  3200.200000 # 极差
var       0.154755 # 变异系数
dis     562.600000 # 四分位数间距
```



3-3 帕累托图代码实例

```python
# 代码3-8 菜品盈利帕累托图

# 菜品盈利数据 帕累托图
import pandas as pd

# 初始化参数
dish_profit = '../data/catering_dish_profit.xls'  # 餐饮菜品盈利数据
data = pd.read_excel(dish_profit, index_col = u'菜品名') # 指定索引列【菜品名】
data = data[u'盈利'].copy() # 选择有效数据【盈利】，剔除【菜品ID】
data.sort_values(ascending = False) # 按值排序，降序（ascending默认ture为升序）

import matplotlib.pyplot as plt  # 导入图像库
plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False  # 用来正常显示负号

plt.figure() # 创建画布
data.plot(kind='bar') # 绘制柱状图
plt.ylabel(u'盈利（元）') # 定义图例
p = 1.0*data.cumsum()/data.sum() # 定义累计盈利率
p.plot(color = 'r', secondary_y = True, style = '-o',linewidth = 2) # 绘制条形图
plt.annotate(format(p[6], '.4%'), xy = (6, p[6]), xytext=(6*0.9, p[6]*0.9), arrowprops=dict(arrowstyle="->", connectionstyle="arc3,rad=.2"))  # 添加注释，即85%处的标记。这里包括了指定箭头样式。
plt.ylabel(u'盈利（比例）')
plt.show()
```

运行结果

<img src="/Users/hawzz/Documents/pd_dand_commit/Chap3_3.png" alt="Chap3_3" style="zoom:36%;" />

由上图可知，菜品A1~A7总盈利额占该月盈利额的85.0033%。根据帕累托法则，应该增加对菜品A1 ~ A7的成本投入，减少对菜品 A8~A10的投入以获得更高的盈利额。

3-4 销量数据相关性分析

```python
# 餐饮销量数据相关性分析
import pandas as pd

catering_sale = '../data/catering_sale_all.xls'  # 餐饮数据，含有其他属性
data = pd.read_excel(catering_sale, index_col = u'日期')  # 读取数据，指定“日期”列为索引列

print(data.corr())  # 相关系数矩阵，即给出了任意两款菜式之间的相关系数
print(data.corr()[u'百合酱蒸凤爪'])  # 只显示“百合酱蒸凤爪”与其他菜式的相关系数
# 计算“百合酱蒸凤爪”与“翡翠蒸香茜饺”的相关系数
print(data[u'百合酱蒸凤爪'].corr(data[u'翡翠蒸香茜饺']))
```

运行结果

```
print(data.corr())  # 相关系数矩阵，即给出了任意两款菜式之间的相关系数
            百合酱蒸凤爪  翡翠蒸香茜饺 金银蒜汁蒸排骨  ... 香煎韭菜饺 香煎罗卜糕 原汁原味菜心
百合酱蒸凤爪   1.000000  0.009206  0.016799  ...  0.127448 -0.090276  0.428316
翡翠蒸香茜饺   0.009206  1.000000  0.304434  ...  0.062344  0.270276  0.020462
金银蒜汁蒸排骨  0.016799  0.304434  1.000000  ...  0.121543  0.077808  0.029074
乐膳真味鸡    0.455638 -0.012279  0.035135  ... -0.068866 -0.030222  0.421878
蜜汁焗餐包    0.098085  0.058745  0.096218  ...  0.155428  0.171005  0.527844
生炒菜心     0.308496 -0.180446 -0.184290  ...  0.038233  0.049898  0.122988
铁板酸菜豆腐   0.204898 -0.026908  0.187272  ...  0.095543  0.157958  0.567332
香煎韭菜饺    0.127448  0.062344  0.121543  ...  1.000000  0.178336  0.049689
香煎罗卜糕   -0.090276  0.270276  0.077808  ...  0.178336  1.000000  0.088980
原汁原味菜心   0.428316  0.020462  0.029074  ...  0.049689  0.088980  1.000000

print(data.corr()[u'百合酱蒸凤爪'])  # 只显示“百合酱蒸凤爪”与其他菜式的相关系数
[10 rows x 10 columns]
百合酱蒸凤爪     1.000000
翡翠蒸香茜饺     0.009206
金银蒜汁蒸排骨    0.016799
乐膳真味鸡      0.455638
蜜汁焗餐包      0.098085
生炒菜心       0.308496
铁板酸菜豆腐     0.204898
香煎韭菜饺      0.127448
香煎罗卜糕     -0.090276
原汁原味菜心     0.428316
Name: 百合酱蒸凤爪, dtype: float64

# 计算“百合酱蒸凤爪”与“翡翠蒸香茜饺”的相关系数
print(data[u'百合酱蒸凤爪'].corr(data[u'翡翠蒸香茜饺']))
0.009205803051836475
```

由结果可知

各菜品之间相关性。



3. 主要数据探索函数

   1. 基本统计特征函数

      | 方法名     | 函数功能             | 所属库 |
      | ---------- | -------------------- | ------ |
      | sum()      | 按列计算总和         | pandas |
      | mean()     | 平均值               | pandas |
      | var()      | 方差                 | pandas |
      | std()      | 标准差               | pandas |
      | corr()     | 关系数矩阵           | pandas |
      | cov()      | 协方差矩阵           | pandas |
      | skew()     | 样本值偏度（三阶矩） | pandas |
      | kurt()     | 样本值峰度（四阶矩） | pandas |
      | describe() | 基本统计结果         | pandas |

   2. 拓展统计特征数

      | 方法名         | 函数功能                           | 所属库 |
      | -------------- | ---------------------------------- | ------ |
      | cumsum()       | 依次给出前1、2、...、n个数的和     | pandas |
      | cumprod()      | 依次给出前1、2、...、n个数的积     | pandas |
      | cummax()       | 依次给出前1、2、...、n个数的最大值 | pandas |
      | rolling_sum()  | 依次给出前1、2、...、n个数的最小值 | pandas |
      | rolling_mean() |                                    | pandas |
      | rolling_var()  |                                    | pandas |
      | rolling_std()  |                                    | pandas |
      | rolling_corr() |                                    | pandas |
      | rolling_cov()  |                                    | pandas |
      | rolling_skew() |                                    | pandas |
      | rolling_krut() |                                    | pandas |

   3. 统计作图函数

      | 作图函数名         | 作图函数功能 | 所属库            |
      | ------------------ | ------------ | ----------------- |
      | plot()             | 绘制线形图   | matplotlib/pandas |
      | pie()              | 饼图         | matplotlib/pandas |
      | hist()             | 直方图       | matplotlib/pandas |
      | boxplot()          | 箱型图       | pandas            |
      | plot(logy = True)  | y轴对数图    | pandas            |
      | plot(yerr = error) | 误差条形图   | pandas            |

      通常在作图前需要加载

      ```python
      import matplotlib.pyplot as plt
      plt.rcParams['font.sans-serif'] = ['SimHei'] # 用来正常显示中文标签
      plt.rcParams['axes.unicode_minus'] = False # 用来正常显示负号
      plt.figure(figsize = (7,5)) # 创建图像区域，指定比例
      ```

      线图作图实例代码

      ```python
      import numpy as np
      x = np.linspace(0,2*np.pi,50) # x坐标输入
      y = np.sin(x) # 计算对应x的正弦值
      plt.plot(x,y,'bp--') # 控制图形格式为蓝色带星虚线，显示正弦曲线
      plt.show()
      ```

      <img src="/Users/hawzz/Documents/pd_dand_commit/折线图实例.png" alt="折线图实例" style="zoom:36%;" />

      ```python
      plt.plot(x,y,S)
      ```

      即绘制y对于x的折线图，S为字符串参’b’为蓝色, ‘r’为红色, ‘g’为绿色, ‘o’为圆圈, ‘+’为加号标记, ‘-’为实线, ‘—‘为虚线。

      ```
      plot(kind = 'box')
      ```

      可以通过kind参数指定作图类型，例如 line（线图）、bar（条形图）、barh、hist（直方图）、box（箱型图）、kde（线性密度分布图）、area（叠堆面积图）、pie（饼图）。

      饼图作图实例

      ```python
      import matplotlib.pyplot as plt
      
      # The slices willbe ordered and plotted counter-clockwise.
      labels = 'Frogs', 'Hogs', 'Dogs', 'Logs' # 定义标签
      sizes = [15, 30, 45, 10] # 每一块的比例
      colors = ['yellowgreen', 'gold', 'lightskyblue', 'lightcoral'] # 每一块的颜色
      explode = (0, 0.1, 0, 0) # 突出显示，这里仅突出显示第二块
      
      plt.pie(sizes, explode = explode, labels = labels, colors = colors, autopct = '%1.1f%%', shadow = True, startangle = 90) # shadow为阴影参数， startangle为起始位置
      plt.axis('equal') # 显示为圆（避免比例压缩为椭圆）
      plt.show()
      ```

      <img src="/Users/hawzz/Documents/pd_dand_commit/pie_sample.png" alt="pie_sample" style="zoom:36%;" />

      直方图作图实例

      ```python
      import matplotlib.pyplot as plt
      import numpy as np
      x = np.random.randn(1000) # 1000个服从正态分布的随机数
      plt.hist(x,10) # 分成10组进行绘制直方图
      plt.show()
      ```

      <img src="/Users/hawzz/Documents/pd_dand_commit/直方图实例.png" alt="直方图实例" style="zoom:36%;" />

      箱型图实例

      ```python
      import matplotlib.pyplot as plt
      import numpy as np
      import pandas as pd
      x = np.random.randn(1000) # 1000个服从正态分布的随机数
      D = pd.DataFrame([x,x+1]).T # 构造两列的DataFrame，.T是逆转表D
      D.plot(kind = 'box') # 调用Series内置的作图方法画图，用kind参数指定箱型图box
      plt.show()
      ```

      <img src="/Users/hawzz/Documents/pd_dand_commit/箱型图实例.png" alt="箱型图实例" style="zoom:36%;" />

      轴对数图（plot(logx = True)/plot(logy = True))

      ```python
      import matplotlib.pyplot as plt
      plt.rcParams['font.sans-serif'] = ['SimHei'] # 用来正常显示中文标签
      plt.rcParams['axes.unicode_minus'] = False # 用来正常显示负号
      import numpy as np
      import pandas as pd
      
      x = pd.Series(np.exp(np.arange(20))) # 原始数据
      x.plot(label = u'原始数据图', legend = True)
      plt.show()
      x.plot(logy = True, label = u'对数数据图', legend = True)
      plt.show()
      ```

      <img src="/Users/hawzz/Documents/pd_dand_commit/原始数据图.png" alt="原始数据图" style="zoom:36%;" />

      <img src="/Users/hawzz/Documents/pd_dand_commit/对数数据图.png" alt="对数数据图" style="zoom:36%;" />

      误差条形图实例

      ```python
      import matplotlib.pyplot as plt
      plt.rcParams['font.sans-serif'] = ['SimHei'] # 用来正常显示中文标签
      plt.rcParams['axes.unicode_minus'] = False # 用来正常显示负号
      import numpy as np
      import pandas as pd
      
      error = np.random.randn(10) # 定义误差列
      y = pd.Series(np.sin(np.arange(10))) # 均值数据列
      y.plot(yerr = error) # 绘制误差图
      plt.show()
      ```

      <img src="/Users/hawzz/Documents/pd_dand_commit/误差条形图.png" alt="误差条形图" style="zoom:36%;" />


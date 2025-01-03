采用的数据集Sunspots_monthly来源于网站https://www.kaggle.com/。该数据集跨越了1749年1月—2021年1月,内容为月均太阳黑子数（monthly mean sunspot number), 包含3264条记录。
期待达成的目标是通过对太阳黑子的长期数据进行分析，观察和验证其周期，并建立模型对数据进行模拟。

	数据集分析
2.1观测数据
	首先观察数据，数据集内容是每个月的太阳黑子数目。预计对周期进行精读较低的估计，以年为单位。基于此，对数据进行处理，将每个年份的1到12月的太阳黑子数目相加，得到年均太阳黑子数。
	按照年均太阳黑子数目进行可视化处理，进行初步的观察。可以观察到周期大约在十年左右。
 
太阳黑子年均数目时间序列图
	之后为了排除月份的影响，进行第二步处理，纵向按照月份求和，将不同年份的1月、2月等依次加和。可以看到不同月份之间的数据差异很小，对太阳黑子数据的周期波动不造成显著影响。
 
太阳黑子各月数据累加图
2.2分析数据的周期性
	周期性，指时间序列中呈现出来的围绕长期趋势的一种波浪形或振荡式变动。准确提取周期信息，不仅能反映当前数据的规律，应用于相关场景，还可以预测未来数据变化趋势。下面将计算自相关系数并对该数据集的周期进行检验。
2.2.3自相关性检验
	多元线性回归模型的基本假设之一就是模型的随机干扰项相互独立或不相关，如果模型的随机干扰项违背了相互独立的基本假设，则称为存在序列相关性（自相关性）。对一个有周期性的时间序列样本进行滞后处理，寻找序列在滞后多长时间，也即经遍历多少相位差，后与原序列的相关性达到最高，此时的相位差就是序列的周期。这个过程叫做自相关性检验，前文提到的经过“滞后”处理的序列与原序列的相关性被称之为自相关系数。自相关函数ACF（Autocorrelation Function）是用来计算自相关系数的函数。
	在python中导入statsmodels.graphics.tsaplots里的plot_acf进行对数据自相关系数的计算。
	首先，确认太阳黑子有周期性。平稳序列的自相关系数会很快趋近于零，这在ACF图中的表现是振幅逐渐减小。将太阳黑子月平均数据代入模型分析，观察得到振幅逐步缩小，这意味着数据集有周期性。
 
太阳黑子月平均数据的ACF图
	之后，估计序列的周期。淡蓝色部分为误差范围（error band），区域内的数据点偏差较大应该舍弃。使用前面处理得到的年均太阳黑子数目生成图片并对图片进行观察，得知周期是11年。
 
太阳黑子年平均数据的ACF图

2.2.4检验周期的另一方法
	使用Pandas里的autocorrelation_plot方法也可以检验整体自相关性。得到的图片是原数据与特定相位差数据的散点图，散点图所呈现出的线性相关性越明显越能说明此时对应的相位差是数据集的周期。
 
	数据模拟
3.1模型简介
采用LSTM（Long Short Term Memory）模型处理时间序列。该模型是循环神经网络模型中的一个，是对RNN（Recurrent Neutral Network）模型的改良。LSTM模型中增加了对信息进行筛选过滤的中间层，相较于RNN的隐藏单元， LSTM的隐藏单元的内部结构更加复杂， 信息在沿着网络流动的过程中， 通过增加线性干预使得LSTM能够对信息有选择地添加或者减少。
调用tensorflow中的LSTM模块进行时间序列预测。创建了一个2层LSTM，每层80个神经元；同时添加了Droopout函数防止过拟合；使用adam激活函数；使用mse作为损失误差的神经网络。
3.2数据处理
	首先，在将数据喂进神经网络之前, 对其进行归一化处理。该方法的好处是使预处理的数据被限定在一定范围内, 从而提高算法精度, 并加速算法的收敛速度。目前几种最常用的数据归一化方法是最大最小标准化法。该方法是一种线性转换的方法, 归一化后的数据值被映射到[0, 1]之间。转化公式为 X=(x-min⁡(x))/(max⁡(x)-min⁡(x))。其中： X是原始观测数据,x是原始数据， min(x) 是观测数据的最小值, max(x) 是观测数据的最大值。
	随后对数据集进行划分，将前200个数据划分为训练集，其余数据划分为测试集。之后，按照步长对训练集进行分组，步长为5。继而将列表形式的数据转变为能够被模型处理的array形式。然后将数据放入模型中处理。
对生成的数据进行反归一化处理，得到最终数据。对数据绘图后与原数据绘图结果放置于同一图中进行对比，得到下图结果。
 
拟合结果
	误差分析与总结
对模型模拟出来的结果与实际结果比对，进行误差分析。总共计算了均方误差：1121.72、均方根误差：33.49、平均绝对误差：26.66。在月平均值为81.79的情况下，误差是比较大的。
本次采用的模型较为简单，而简单的神经网络预测时容易陷入局部极值, 预测精度难以得到提升；模型参数也并未经过太多筛选，也没有考虑离散值等因素，故而得到的结果十分粗糙。
太阳黑子月均和年均数目的时间序列数据集具有非稳态型、非高斯型以及非线性等特征，其变化具有非常复杂的非线性特征。预计可以通过调整此模型的参数如步长、每层神经元数等进行模型的优化，或者选择其他模型、改进神经网络的结构、引入其他机器学习方法等方式来提高精度。

参考文献
[1].程术,石耀霖,张怀.基于神经网络预测太阳黑子变化.中国科学院大学学报, 2022, 39(5): 615-626.
[2].柳小葱. python深度学习之基于LSTM时间序列的股票价格预测[EB/OL]. https://liuxiaocong.blog.csdn.net/article/details/114946079?spm=1001.2014.3001.5506
[3].王鑫,吴际,刘超,杨海燕,杜艳丽,牛文生.基于LSTM循环神经网络的故障时间序列预测[J].北京航空航天大学学报,2018,44(04):772-784.
[4]. __Meursault__[EB/OL].交通流预测爬坑记（二）：最简单的LSTM预测交通流，使用tensorflow2实现https://blog.csdn.net/K_first/article/details/117378474?spm=1001.2014.3001.5506
[5]. 杨丽,吴雨茜,王俊丽,刘义理.循环神经网络研究综述[J].计算机应用,2018,38(S2):1-6+26.

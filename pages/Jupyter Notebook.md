- 把复杂的工作流程用自动化的、不易出错的脚本来代替
	- 集成环境
	- 直接看结果
- **Jupyter Notebook将Python的交互式特点发挥到了极致**
	- [[#green]]^^分析和建模^^是非常#碎片化 的工作，而每一块的碎片又有着非常强的独立性，甚至可以说除了数据本身之外，每一块的代码之间并没有很强的关联性。
	- [[#green]]^^数据分析和处理^^的过程往往是一个不断试验的过程，我们需要一次又一次的改变预处理的方式、尝试不同的特征工程处理、一遍又一遍的调整着模型参数等等等等。
	- 每一部分的工作都需要反复试验反复修改，而下一模块需要用到的只不过是上一模块输出的数据。
	- *在Jupyter当中，我们可以每写几行或者每完成一个小的模块便运行一次；通过Jupyter，我们可以最快的得知自己做出的调整是好还是坏，并尽快进入到下一次的试验当中。*
	  background-color:: blue
- **Jupyter Notebook更利于汇报和教学**
	- Jupyter在工作汇报和教学方面也是非常的优秀。由于Jupyter本身的模块化和内容的清晰化，使得其天生具有如PPT一般的展示工作成果的功能。
	- 由于Jupyter中可以将输出结果嵌套在Notebook中，并且支持Markdown语句的操作，这样使得你可以在Jupyter中输入任何你需要展示的内容，并且这些内容都会以一种[[#blue]]^^有组织有层次^^的样子排列出来。
- ## 01-ADU Feed Sampling
	- **00-Generate Crude Property Based on Configuration.ipynb**
		- config [[配置表]]
		  id:: 6576b390-9fbf-4294-b228-b1f18a2c12f3
			- 版本名\rightarrow在outputdata中生成不同文件夹
				- ![image.png](../assets/image_1702278774401_0.png)
		- crude propert 原油属性
			- refinery 样本
				- TBP curves
				- tag \leftrightarrow variable name
		- TBP conversion 定温累计收率
			- 混合规则
			- 基于收率\rightarrow基于温度
			- 非线性采样\rightarrow线性采样
		- cut point 理想收率
			- 理想馏分，连续切割
			- 估算现场切割点
		- sampling target property
			- Tag
			- varlist
			- Relaxcoef
			- formula
			- 样本数量
- **01-Crude Sampling.ipynb**
- ## 03-VDU Feed Sampling
	- **00-DataProcessing.ipynb**
		- objective:
		  抽样方法提取常渣进料样本
		  收率分布覆盖且吻合现场数据
		- basic principles：
		  每种原油有5~10个ADU[[$red]]==重复==样本——相同same/相似close
		  VDU与ADU方法相似
		  移除相似样本后，获得目标数量级常渣样本
		  开发了一种基于加权欧氏距离来去除重复样本的方法
		  所选属性包括31点TBP曲线点及API、流量等
		- 1.删除相似常渣样本数据
			- 改变fdistcriteria[distance criteria parameter]距离标准参数\rightarrow决定移除样本数量\rightarrow获取特定样本数
		- 2.比较删除前后分布变化
		- config[[配置表]]
			- mapping+描述
- ## 04-VDU Operating Condition Sampling
	- ### distop
		- **00-SimplifiedModel-Operating Condition SamplingV02.ipynb**
			- 对[[独立变量]]进行LHS采样
				- 配置上下限——变化范围
			- 第一次调用[[GAMS]]模型
				- 读取超过约束条件上下限的样本
				- 针对由[[独立变量]]计算得到的[[中间变量]]
				- 约束条件——[[中间变量]]的变化范围
				  🔍用来最小化样本误差
				- 有且只有一个方程可以不用完全满足，在relax列配置为1
			- 生成第一次distop样本运行input数据，存成h5文件（或csv文件）
		- **01-SimplifiedModel-Sample ModificationV02h5.ipynb**
			- 首先改造普通样本
				- 读取超过约束条件上下限的样本
				- 主要针对减压中段热量没取完的样本，适当调整中段负荷
				  🔍样本percent max removal达到100，先用output计算结果对input取热负荷赋值，然后将中段负荷100%降为99%
				- J1线流量低于0.5t/h，增加减二线切割点
				  🔍减一流量过低，在详细模拟阶段不易收敛
				- 减顶循负荷比较小的情况，需要减少其他几个中段取热
			- 第二次调用[[GAMS]]模型
				- 读取超过约束条件上下限的样本
				- 主要针对减压塔进料温度约束之外的样本进行改造
				- 线性系数coef由运行结果回归分析得到，首次输入可以参考齐鲁配置
				  🔍标准差stddev可以不用指定，用来做scaling，由notebook模块计算
			- 第二次投入样本运行平台的样本为01改造过的样本集，input数据存成h5文件
		- **02-SimplifiedModel-Sampling Visualisation and CleaningV02h5.ipynb**
			- 读取Distop样本并进行数据清洗
				- 合并两次distop运行结果
				- 第二次结果改造
				  🔍配置中段和侧线产品及减顶温度等相关信息，减少极化现象，将约束范围以外的部分拉回至正常范围内
				  🔍由于第一次modification影响了带入塔内的总热量，会出现部分样本中段热量没取完的现象，需要再次干预这部分样本
				- 数据清洗基于input与output的约束范围
				  🔍额外增加了对中段总取热负荷的约束，相关变量在output2中配置，这部分变量只能使用distopoutput的输出变量由公式计算得到；如果想增加一些中间变量可视化，也可以加到这里
			- distop采样结果可视化，检查分布
	- ### rigorous
		- **03-RigSimulation-Operating Condition SamplingV02.ipynb**
			- [[配置表]]
				- RegIndepVar 表格：由于详细模拟的输入会比 Distop 的要多，而且有些独立变量需要再次进行拉丁超立方采样，这个表格中配置的就是需要做拉丁超立方采样的变量，与 IndepVar 表格类似。如减压样本输入变量：封油量，炉管注汽量，提馏段分率等
				  🔍解释一下这里设置“减顶循流量/AR”的意义，为了详细样本运行至第 4 步改为控制减顶循流量，这样会根据与常渣量的关联得到减顶循量
				- RegInput 表格：表格内容比较多，可以理解为三部分
					- 第一部分：A 列至 F 列，为变量对应设置，既对应详细模拟样本运行平台发送文件，同时部分对应 Petro-Sim 详细模型中的 Input；
					- 第二部分：G 列至 P 列，由 Distop 的输出变量和 RegIndepVar 采样变量计算得到详细模拟的输入，它和 DistopInput 表格类似，不同之处是增加了三列内容
						- a)  Weight 列 和 S0 列：这个用于计算生成的详细模拟的输入和基础模拟文件的输入的距离（距离越近，表示样本和基础模拟越接近），其中 S0 列存放的是基础模拟文件中的具体数据，weight列是计算距离时候这个变量的权重，权重越大，表示这个变量在计算距离的时候越重要
						- b)  FixedInBase 列配置的是为了优化聚类后样本的计算步数使用，如果样本平台分步策略为基于第一步保存结果来跑下一个模型的，fixedinbase 里面的参数为第二步及以后变量的起始值，第一步变量不用设置，例如减压采样第二步的调整策略为减顶循返塔温度与减顶温度，这里就设置这两个变量的模型基础固定值为 16 和 50，中段负荷和减压塔底注气量在第三步调整，减顶循流量在第四步调整，均设置了起始值
		- **04-RigSimulation-Sampling Visualisation and CleaningV02[[$red]]==0728a==.ipynb**
		- **05-Combine Sample DB and Visualisation[[$red]]==V020806abcd==.ipynb**
		-
-
-
# 浅谈RoboMaster超级电容模块物料选型
## 开关电源MOS粗略选型参考
> 不考虑同步开关管选型不一致的问题。

### MOS开关速度
&emsp;&emsp;测试条件 ：  
- 梦源DS4T1012 150M 1Gsa 16Mpts示波器  
- MP1907拆机栅极驱动器  
- CSD18540拆机MOS管  
- 同步Buck 的开关拓扑  
> **由于器件为拆机器件，有效性自行斟酌**

参考资料：
[MOS驱动器与MOS的匹配设计](https://ww1.microchip.com/downloads/cn/AppNotes/00799b_cn.pdf)    
&emsp;&emsp;MOS的开关速度关系到开关电源的开关频率，如何选择一款能满足在设计开关频率下，开关速度达到要求（我觉得，MOS导通的Vgs上升时间+截止的Vgs下降时间在开关周期的1%左右比较好，各一半；不过没有什么依据）的MOS尤为重要。  
&emsp;&emsp;为粗略计算开关管的开关时间，我们要确定以下几个参数：  
- 开关频率  
- 驱动电压  
- 驱动电流  
- 开关管的Qg  

&emsp;&emsp;开关频率看你设计，驱动电压一般在8-12V，可灵活调整，驱动电流查阅栅极驱动器手册，开关管的Qg查阅MOS手册。
> 栅极驱动器的驱动电流越大，价格也会比较贵，但是它能驱动较差的开关管，或者驱动相同的开关管可以提高开关速度。这个大概就看货源、价格，选自己喜欢的就行。  

&emsp;&emsp;由于手上有实物，先看看理论计算与实验数据之间的差别。  
下图是Buck下管在10.5V驱动电压，栅极串了R=6.2R电阻（电阻可能会发烫，适当使用大封装电阻），且半桥输出开路，即空载的测试结果：Vgs上升时间约为**94ns**；  
![下管6.2上升时间.jpg](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/下管6.2上升时间.jpg)

通过驱动电压去MOS管手册查找Vgs-Qg折线图，在10.5Vgs时**Qg≈45nc**；  
![18540Qg.png](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/18540Qg.png)

通过驱动电压去驱动器手册查找I-VDD折线图，在10.5VDD时Ioll≈Iolh≈2A，由此可得，驱动器等效内阻**r≈10.5/2＝5.3R**；  
![MP1907驱动电流.png](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/MP1907驱动电流.png)

MOS管中有一项**Rg=1.6R**；  
![MOS Rg.png](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/MOSRg.png)

&emsp;&emsp;先前在栅极串了6.2R，就可以粗略得出栅极上的总电阻Rg-total=r+Rg+R≈13；接下来算出驱动瞬间的最大电流约为**驱动电压/栅极总电阻**，Ig=Vgs/Rg-total≈0.8；  
为了验证电流大小，我通过示波器测量6.2R两端的电压可以得出峰值电流大小约为3.7/6.2=0.6，反推回去栅极上的总电阻为10.5/0.6=17.5，差了3R左右，先记着。  
![6.2R两端电压.jpg](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/6.2R两端电压.jpg)  

&emsp;&emsp;由文档中给出的***t=Q/I***即可粗略得出理论Vgs上升时间为45/0.8≈56ns；使用测试得出的峰值电流带入计算结果为45/0.6≈75ns。  
&emsp;&emsp;由于寒假在家没有过多的器件，下面再放出两组测试结果：  
栅极电阻两个6.2并联成3.1，其余不变，上升时间缩短为**66ns**，理论计算时间约为**43ns**；  
![3.1R上升时间.jpg](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/3.1R上升时间.jpg)
栅极电流参考
![3.1R两端电压.jpg](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/3.1R两端电压.jpg)

上管栅极电阻0R，驱动电压9.2V，上升时间为**52ns**（这个是栅极驱动器可以驱动MOS的最快速度，但是可能栅极驱动器会比较烫），理论计算时间约为**38ns**；  
![上管上升时间.jpg](https://dontfreeze-picgo.oss-cn-hangzhou.aliyuncs.com/PicGo/开关电源/MOS选型/上管上升时间.jpg)

&emsp;&emsp;总结：可以看到栅极上的电阻大小对理论计算结果的影响比实际要小，至于为啥，我也不知道。我猜测：可能是因为栅极上电阻的增加会导致栅极寄生电荷的充电受到显著的影响，而理论则是以理想模型的形式去计算的，忽略了各种寄生参数的影响。通过外部串联在栅极上的电阻两端的电压波形可得出此时流经的电流大小，看波形变化可以知道，他的电流在10ns左右达到最大值，接下来就快速下降，且越来越慢，这符合电容的实际充电过程；而理论计算我想大概他是将充电过冲简化成了以最大电流来持续充电的形式，或许可以通过计算电容的充电时间常数以获得更加精确的时间。不过这既然是粗略选型估计，**不妨我们直接将计算出来的结果留出50%以上的冗余，来避免因计算过程和寄生参数产生的误差影响**。

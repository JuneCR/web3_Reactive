### Reactive Builder Camp

Day1

​	1.完成创建了Mester钱包

​	2.安装Foundry

​	3.领取测试币

​	4.测试营地项目:Suite result: ok. 8 passed; 0 failed; 0 skipped; finished in 2.24ms (2.63ms CPU time)

Day2		

​	**了解通过营地项目简单了解Reactive运行及机制**

​		Reactive NetWork : 源链产生事件 ， Reactive NetWork(Lasna)监听事件，目标链触发调用

​		**源链**:当钱包/其他调用源链合约时，会产生一个交易收据，多比交易收据构成收据树

​			全节点维护，区块中产生Bloom[用于快速查找]

​			交易收据[源码中函数形参indexed标签 直接放入topic ，没标记的放入data]

​				address：源链合约地址

​				topic:[

​					topic0：被调用函数的hash[keccak256]必有

​					topic1 :  带有 indexed标记的值

​					]

​				data:没有 indexed标记的[abi]

​		**Reactive(Lasna)**：**中继节点**[链下服务器 ， 同接入多条链，利用RPC读取数据]

​						  会遍历源链Sepolia中的所有交易收据

​						  那到日志中的topic + 合约地址 ， 与 订阅清单 对比

​						  **订阅清单**:注册在Reactive链的系统合约中

​							源链ID:Sopalia测试网的链ID

​							源合约地址

​							目标事件:源链合约的函数的hash

​							回调合约地址 + 回调函数

​							回调额度

​						匹配后 ， 由中继节点将交易收据中的topic ， data 提取出 

​						打包成完整的事件包(log)

​						中继节点向Reactive(Lasna)发出一笔交易 ， 将log数据作为参数

​						放入react()

​						**Reactive(Lasna)**:

​						react()开始执行 ，生成目标链指令[调哪个合约，函数，传什么阐述]

​						ramaining 回调额度-1

​		中继转发执行，在向目标链发布一个交易，调用目标链合约

​	


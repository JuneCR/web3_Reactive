### Reactive Builder Camp

Day1

​	1.完成创建了Mester钱包

​	2.安装Foundry

​	3.领取测试币

​	4.测试营地项目:Suite result: ok. 8 passed; 0 failed; 0 skipped; finished in 2.24ms (2.63ms CPU time)

Day2		

​	**了解通过营地项目简单了解Reactive运行及机制**

​		Reactive NetWork : 源链产生事件 ， Reactive NetWork(Lasna)监听事件，目标链触发调用

​		1.源链(Sopalia)

​			 用户通过调用 部署在源链上的合约

​	 		Sopalia全节点 执行这笔交易--------交易收据

​			交易收据[address:源链合约地址

​				topics[0]:被调用函数的hash

​					topics[1....n]:形参标记indexed的参数

​				data:未被标记indexed值

​				]

​		2.中继网络: 链下服务器监听到事件，把事件搬到Reactive链上

​			中继节点[运行程序的服务器]:持续同步Sepolia上区块的信息+拉取区块内的日志(收据树)

​			中继节点:同步了Reactive(Lanas)链上完整状态

​			[在部署Lanas合约的时候咱在初始化函数中设置的订阅表]

​			中继节点:将订阅表的阐述 与Sepolia的交易收据进行比对

​			比对成功后 ， 把Sepolia关于这个事件(链ID+区块高度+交易哈希+日志)，放到Lanas网络上

​			中继节点会发起一步交易调用系统合约的trigger()方法

​		3.Lanas:系统合约调用react()

​			系统合约:订阅有效性 

​			余额校验:我的Reactive合约账户中是否有足够的余额支付gas ， 

​			格式校验:事件数据的完整合法

​			校验通过后 ，从Reactive合约地址预扣gas额度，并把事件封装成LogRecord结构体

​			以系统合约自身的地址 调用 我部署在Lanas上的合约 ,执行React()

​			业务执行后 发生一个事件[交易收据:目标链ID , 目标合约地址，目标链gas，调用目标]

​			react()执行接收后，系统合约计算实际gas消耗

​		4.中继网络:中继网络看到Callback事件，去目标链发布交易调用我的目标合约

​			中继节点:解析事件参数，二次校验[reactive合约余额是否充足，支付gas成本]

​			中继节点:用自己的钱包发送合约调用交易 调用官方Proxy合约

​			在由Proxy合约调用 目标链合约

> ​	   中继节点 - > 系统合约的trigger方法 |  中继节点 -> Proxy合约  产生的费用 

​	Day3

​		**费用**: 系统合约向Lanas中合约调用react() -------执行费

​			  监听到Callback后，Sopalia链上的官方Proxy合约调用目标链合约 ------回调费

​		**余额**: Balance  合约 ， Reservers 预支付费用 , Debt 已产生但未支付的费用

​			  Balance:  [部署合约时/执行调用转入] 这笔余额用对Reaservers充值

​			  Reaservers  用于支付执行费/回调费 ， 是在Balance中预扣的金额

​			  Debt:  欠款 Reaservers扣光的额度差 ， 同时系统会暂停订阅服务

​			 [这个是部署在Lanas上的合约账户记录的，所有消费都从这里扣]

​		**系统合约执行流程**:每笔交易都走系统合约中trigger()方法

​			  1.Debt Check 欠款检查

​			  2.SnapshotGas 执行前的Gas余量

​			  3.Execute react() 执行业务逻辑

​			  4.Calculate Cost 计算本次服务的费用(系统开销 + 实际消耗) x结算单价

​			  5.Charge Contract 在Reaserves额度中，扣除总费用 ，余额不足的额差

​			  6.Kickback Validator 扣到费用，转账给本次交易的发起者(中继节点)

​		**费用计算**

​			
$$
\mathrm{fee} = p_{\mathrm{base}} \cdot C \cdot (g_{\mathrm{used}} + K)
$$
​			p_base:计算gas单价 [取最大值 中继节点发布交易者 | 区块网络基础Gas费]

​			C :定价系数 默认值1000 ， 宏观调控

​			g_used:实际执行消耗Gas

​			K :附加费

​			[系数1000 ， 附加费100000Gas ，支付预算50000]

​		**余额的查询**

​			Balance余额

​			Lanas上的合约:`cast balance $REACTIVE_CONTRACT_ADDR --rpc-url $REACTIVE_RPC`		        目标链上的合约:`cast balance $CALLBACK_CONTRACT_ADDR --rpc-url $DESTINATION_RPC`

​			为什么要这样?

​				Lasnas查余额是为了看充值余额有没有到位，扣了多少钱

​				目标链是为了proxy调用目标链合约，如果合约里转账业务金额发生了变化，回调成功

​			Debt欠款

​			cast call $SYSTEM_CONTRACT_ADDR "debt(address)"

​				 $REACTIVE_CONTRACT_ADDR --rpc-url $REACTIVE_RPC | cast to-dec

​			cast call $CALLBACK_PROXY_ADDR "debt(address)" 

​				$CALLBACK_CONTRACT_ADDR --rpc-url $DESTINATION_RPC | cast to-dec

​			Reserves储备金

​			cast call $SYSTEM_CONTRACT_ADDR "reserves(address)" 

​				$REACTIVE_CONTRACT_ADDR --rpc-url $REACTIVE_RPC | cast to-dec

​			cast call $CALLBACK_PROXY_ADDR "reserves(address)" 

​				$CALLBACK_CONTRACT_ADDR --rpc-url $DESTINATION_RPC | cast to-dec

​			所有的扣费不是都由Lanas合约扣费吗，为什么还能在目标链查询?

​				中继合约---> Proxy-----> 目标链合约 都是垫付的需要记录下，最后与Lanas扣款

​		**Inspecting Callbacks 查询回调记录**

​			cast call $SYSTEM_CONTRACT_ADDR \  "getCallbacks(uint256)

​			((uint256,address,uint256,uint256,uint256,bytes)[])" \

  		      $REACTIVE_TX_HASH --rpc-url $REACTIVE_RPC

​			系统合约记录了发往目标链的每一笔回调




















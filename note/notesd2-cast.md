dev@JUNE:~$ cast call 0x8888888888888888888888888888888888888888 "_gasPriceCoeffPer1000()" --rpc-url https://lasna-rpc.rnk.dev/ | cast to-dec
1000
dev@JUNE:~$ cast call 0x8888888888888888888888888888888888888888 "_extraGas()" --rpc-url https://lasna-rpc.rnk.dev/ | cast to-dec
100000
dev@JUNE:~$ cast call 0x8888888888888888888888888888888888888888 "_maxChargeGas()" --rpc-url https://lasna-rpc.rnk.dev/ | cast to-dec
50000

------

1. 基础价格 ， 系数 ，固定附加 都是系统默认的 ， 实际消耗gas为系统合约调用react()产生的gas

2.balance : 在Lanas中的合约账户中记录 ，用于对reserves储蓄金的预扣

​    debt : 欠款   ， 实际花费超过了预扣金额的额差

​    reservers：用于支付  ，系统合约-----> react() |  Proxy ----->目标链合约产生的gas
# 整体架构概览

![](./img/bytom_overview.png)

比原链分为三个层次
第一层就是大家接触比较多的钱包层，就是进行收款和打款的模块，钱包一般带操作界面，大家都可以日常使用，所以会比较熟悉。
然后就是最核心的内核层，内核可以理解为分布式系统中每个节点认同的一套规则，只有有相同的规则，两个节点才能达成一致。如果规则不同，其实就是发生分叉了。
最后一层是通信层，通信层是节点之间交换信息的方式，包含区块同步，交易同步等。

![](./img/bytom_core.png)

首先来看内核层，内核层主要由五个模块构成：
-  孤儿块管理：孤儿块就是由矿工挖出但未成为主链区块的区块（在相同高度产生2个甚至更多的合法区块，一个区块成为主链，剩下的则称为孤儿块），孤儿块管理就是将未成为主链区块的孤儿块存储起来。
-  共识层：确认一个块是否合法。分为区块头验证和交易验证。区块头验证需要验证它的父块和时间戳，同是需要算力来保证记账权利。交易验证比原特别的设计了一层BC层，这层在交易验证时会获得更好的性能，交易验证还和智能合约相关，交易被验证时参数会参入虚拟机验证该交易是否合法。
-  区块树管理：又成为Block Index，作用是记录全网所有的块，保存了全网所有块的一张镜像图。因为有孤儿块，所有它并不是链式结构的，会有分叉的情况，所以称为区块树
-  数据存储：将区块数据做持久化存储。包含两种数据，第一种是区块数据，会在网络上进行广播的原生区块信息；第二种是UTXO数据，存储UTXO数据是为了更快的验证一笔UTXO是否可以花费，而不需要去遍历所有区块信息
-  交易池：维护了所有全网发出的但是还未被确认的交易。跟它关联最大的是挖矿模块，挖矿模块每次要产生一个新区块的时候，它会从交易池拿一些交易打包成块，然后用Tensority共识算法进行工作量验算

![](./img/bytom_wallet.png)

然后来说一下钱包层：
-  私钥模块：主要用于管理私钥（私钥的生成，存储，备份等）和签名
-  账户模块：在比原的设计中，使用了账户-地址-密钥三层体系，每个人可以拥有多把私钥，通过私钥不同的组合形式形成账户，每个账户可以又无限多个地址，地址是由账户的私钥派生出的二级私钥形成的地址，使用多地址可以更好的保护用户的隐私。
-  资产模块：管理账户下创建资产的模块，任何issue的资产都是在这个模块中交互。
-  交易模块：可以理解成跟我相关的交易数据，它主要做了两件事：
   -  将和你相关的交易从所有交易中筛选出来
   -  维护钱包层UTXO的数据库，记录本人所拥有的UTXO

![](./img/bytom_network.png)

最后说一下通讯层：
-  节点发现：P2P相关，是非常独立和成熟的一块，通过节点发现这个模块获得种子节点，然后通过种子节点来获取其他更多的节点。
-  交易同步：在各个节点之间同步交易。
-  区块同步：又称为被动区块同步，如果发现存在区块高度高于自己的其他节点时，不断请求区块同步。
-  新区块快速广播模块：新区块主动同步，当挖到一个新的区块时进行强制广播，从而更快的在全网传播。


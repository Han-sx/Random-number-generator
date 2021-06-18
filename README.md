# Random-number-generator
Based on blockchain and cryptography, generate verifiable random numbers


# 基本思路
## 方案一 (https://github.com/randao/randao)

1.利用多方参与：去中心化在于多方参与，多方共同参与生成的随机数可被多方接受和验证
> 为了保证随机数不能被少数人控制，多方参与需要基础的人数底线。可通过区块提供奖励等方式鼓励参与随机过程，获取有效随机


2.收集参与者提供的随机数加密散列值
> 在某时间区间内收集加密散列以后期验证。有效时间6个区块，之后停止散列值收集。为保证参与者完成整个随机过程，需要提供部分gas作为质押与散列一同发送给合约账户


3.将成功收集到的散列值提供者规定为最终随机数生成参与者
> 参与者将随机数发送至合约账户并，合约账户验证所有接受到的随机数作为有效随机数，未提供随机数的用户不会终止随机数生成过程，质押不会返还

4.将收集到的随机数作为种子生成最终随机
> 完成最终随机后可将所有质押金额均分给参与用户

### 合约函数

`random_requests()`：用于接受随机请求，请求者需要提供小份 gas，返回随机结果

`participate_in_random_1(random_number_sha)`：第一阶段提供随机数散列，并附带质押 gas

`participate_in_random_2(random_number)`：第二阶段提供随机数

<br>

## 方案二 (A Blockchain-based Random Number Generation Algorithm and the Application in Blockchain Games)

1.Game provider 生成一个随机数 Np 和 public-private key pair，game provider 使用公钥加密随机数为 E(Np)，将信息 massage:{ E(Np) & GameID } 发送给区块链
> GameID 为游戏编号


2.区块链使用 smart contract 检测 GameID，如果为新游戏，则记录相应信息 record:{ txid1 & E(Np) & GameID }，并将 txid1 返回给 Game provider
> samrt contract 用于接受数据产生交易信息

3.Game provider 发送 txid1 给所有参与者，参与者根据 txid1 自行检测信息。参与者 i 将自己的随机数发送给 Game provider massage:{ GameID & Ni }，Game provider 发送收集到的随机数给区块链合约 record:{ E(Np) & GameID & (N1...Ni...Nn) }
> (N1...Ni...Nn) 表示收集到的随机数集合，这里参与者将随机数发送给 Game provider 而不是 blockchain 是因为效率问题


4.Smart contract 检测信息，生成记录 record:{txid2 & txid1 & E(Np) & GameID & (N1...Ni...Nn) }，返回 txid2 给 Game provider
> 记录不对外公开

5.Game provider 广播 txid2 给所有参与者。所有参与者同意后可以开始生成随机数，参数包括 block_txid2_hash、txid2、N(p)、(N1...Ni...Nn)， 使用 f(x) 生成随机数 k
> f(x) 可以是任意预定的功能，如按位异或运算。block_txid2_hash、txid2 用于防止参与者和 Game provider 作弊


6.使用 k 进行游戏， 结束后 game provider 上传 record 和游戏结果 result 给智能合约
> 信息传递是私密的

7.Smart contract 生成交易信息 massage:{ txid3 & txid2 & txid1 & E(Np) & GameID & (N1...Ni...Nn) & result & Operation & Private Key }
> 最终信息公开

8.参与者使用私钥和 txid3 检测所有信息真实性
> 利用给出信息可检测游戏过程

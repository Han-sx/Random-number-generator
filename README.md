# Random-number-generator
Based on blockchain and cryptography, generate verifiable random numbers


# List some schemes
## 1. [randao](https://github.com/randao/randao)

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

## 2. [A Blockchain-based Random Number Generation Algorithm and the Application in Blockchain Games](https://ieeexplore.ieee.org/document/8664239)

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

<br>

## 3. [How to generate transparent random numbers using blockchain](https://ieeexplore.ieee.org/document/8664239)

> users: `U`

> service systems: `S`

> random number generator: `G`

> blockchain holder: `B`

`U` 拥有地址 `a`，`S` 拥有地址 `b`，且 `U` 知道 `S` 的地址 `b`

`G` 执行的算法对外公开，作为智能合约存在于区块链上，执行两个函数：`Reg(info)` 和 `AskNonce()`，`Reg(info)` 用于将数据信息注入区块链，`AskNonce()` 用于向 `B` 获取下个块的随机数

在此系统中 `G` 和 `B` 是可信的，`G` 为智能合约，`B` 为区块链

当只有一个参与者和一个生成者的时候，选取区块链的 next nonce 是合理的，若需求为两个随机数时，这种方案效率较低

### The processes of the proposed scheme:
> (P1) The user registration procedure

> (P2) The service system registraion procedure

> (P3) The lottery procedure

<br>

1.The user `U` registers his address `a` and a seed used for computing random numbers, to the blockchain
> The user `U` determines a seed `r`, and sends, to the random number generator `G`, his address `a` and the seed `r`

> `G`receives `a` and `r` from `U`, and registers those executing `Reg(a, r)`

<br>

2.The service system registers its address `b` and a seed used for computing random numbers, to the blockchain
> The service system `S` determines a seed `p`, and sends, to the random number generator `G`, its address `b` and the seed `p`

> G receives `b` and `p` from `S`, and registers those executing `Reg(b, p)`

<br>

3.The user `U` requests a random number generation, and `U` and the corresponding service system get the value of the random number generated
> The user `U` sends a request to the random number generator `G` by sending his address `a` and the address `b` of the service system `U` applies to

> `G` makes a (unique) identification number `sn` (which may be a serial number)

> `G` finds the latest `r` in the blockchain which is associated with the address `a`, and finds the latest `p` in the blockchain which is associated with the address `b`. Then `G` registers `sn`, `a` and `b` executing `Reg(sn, a, b)`

> `G` executes `AskNonce()` to get the nonce `n` from `B` at the timing when the next block is generated

> `G` computes `α = hash(r, p, sn, a, b, n)`  eg. `α = SHA256(f(r, p, sn, a, b, n))`

> `G` sends, to `U` and `S`, `sn` and `α`. Here, `α` is the random number generated for the identification number `sn`

> `U` and `S` can verify whether the result `α` is indeed computed using `r`, `p`, `sn`, `a`, `b` and `n`


## 4. [Probabilistic Smart Contracts: Secure Randomness on the Blockchain](https://ieeexplore.ieee.org/document/8751326)

1.Step 1: Request. The process of random bit generation always begins with a request from another contract or node. A request is lodged with the RBGC by calling its requestRandom-Bit function, which receives the following parameters:

> A fee `φ`, serving as a payment for generating the random bit, whose value is chosen and paid by the client.

> A timestamp `t`, also set by the client, that serves as a deadline for the random bit generation process

> A value estimation `v`, which is an upper bound on the potential economic consequences that might arise for the client if the provided bit is manipulated.

> It assigns a request identification number `id` to the request and returns the `id` to the caller of the requestRan-domBit function

2.Step 2: Participant Registration. Whenever there is an open call for participation in generating a random bit with a given `id`, anyone on the network can participate in the process by registering with the RBGC. To register, a participant `p` should generate a random bit `bp`, and a nonce `np`. She should then compute the hash value `hp` = hash(`bp`, `np`, `p`, `id`) using a predefined hash function. Then, she can call the register function of the RBGC with the following parameters:

> The request identification number `id`

> A deposit of `v` units

> The hash value `hp`

3.Step 3: Revealing Choices. In this step, no new registrations are accepted. Instead, any participant who has already registered can reveal their bit bp and nonce np by calling the reveal function of the RBGC with parameters bp and np. If the parameters are invalid, i.e. if hash(bp, np, p, id) is not equal to hp, then the call is ignored. Also, reveal keeps track of the following 5 values:

> `n'`: number of participants `p` who correctly revealed `bp`

> `n0`: number of even-numbered participants `p` with `bp` = 0

> `n1`: number of odd-numbered participants `p` with `bp` = 0

> `n2`: number of even-numbered participants `p` with `bp` = 1

> `n3`: number of odd-numbered participants `p` with `bp` = 1

4.Step 4: Returning Deposits. After the deadline t, each participant can call the returnDeposit function of the RBGC. This function checks that the participant has revealed her choice correctly and in time, and returns the participant’s deposit only if the check passes

5.Step 5: The RBG Game. After the deadline t, each participant can call the requestReward function of the RBGC. This function uses the fee φ, paid by the client, to reward the participants for submitting inputs. The amount φ is distributed among the participants who have correctly revealed their choices in Step 3 as a reward

> Let α = φ / n , then each participant p receives a reward of rp = α*(1 + up(s) / n') by calling the requestReward function

6.Step 6: Returning the Output. Finally, the client can request her randomly generated bit by calling the getOutput function of the RBGC after the deadline t. In this case, the output random bit and return value are decided as follows:

> If no one has participated or no participant has revealed her bit, a failure result is returned to the client, together with a refund of the fee φ

> Otherwise, the output random bit that is returned to the client is the XOR of all the bits that were correctly revealed by the participants in Step 3

> If every participant has revealed her bit correctly, a success result will be returned to the client

> Otherwise, a penalty result will be returned and the confiscated deposits will be paid to the client

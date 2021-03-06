# SIP 001 PoB （燃烧证明）
## Preamble

题目：燃烧证明

转译者：Gavin（gavin@blockstack.com）

## 关键词
_proof-of-burn_
_leader_ 
_burn chain_
_entire block_
_block streaming_ 
_epoch_
_block reward_
_genesis block_
_forks_
_microblocks_
_verifiable random function_
_commitment transaction_

## 摘要

本文描述了使用PoB进行“单领导者”的选举机制。

燃烧的代币会产生一个分数，该分数（1）概率性地选择与其标准化分数成比例的下一个“领导者”，以及（2）对冲突的交易历史进行排名

## 引言

Blockstack 1.0 中每笔交易与BTC交易一一对应。
Blockstack 2.0

1.高验证吞吐量
一个Stacks区块对应一笔BTC交易

2.低延迟出块
Blockstack 1.0 会因为比特币网络堵塞造成高时延。

Blockstack 2.0 采用区块流模型，每个‘领导者’可以在交易进入内存池时自适应地选择交易并将其打包到其区块中。这样可以确保用户在几秒钟内知道自己的交易是否包含在块中。

3.系统准入性无门槛、无需采矿硬件即可参与

4.公平的矿池
系统允许多用户参与一起挖，共享出块奖励

### 假设

Stacks 领导者选举协议做出以下假设：

* 燃烧链中的深叉根据其长度呈指数减少。

* 发生在燃烧链中的深叉是由于与Stacks协议执行无关的原因。也就是说，矿工们不会尝试通过重组燃烧链来操纵Stacks协议的执行（但要明确，燃烧链矿工也可能会参与Stacks链）。

* 燃烧链矿工不会审查所有Stacks交易，但可以审查其中的一些交易。如果Stacks交易支付足够高的交易费用，则燃烧链的矿工将对该交易进行挖掘。

* 通过燃烧数量衡量，至少有2/3的Stacks领导者候选人是正确和诚实的


## 协议概述

Stacks 2.0 使用STX进行区块奖励激励
一个块代表Stacks链中一个 _epoch_ (周期)时间处理的所有事务。

区块奖励 = 区块本身奖励 + 交易费用
Stacks 2.0 区块是与底层燃烧链共同产生的。每次燃烧链网络生成一个区块。基础链分散了DDoS攻击速率。

除去创世块以外，每一个块都只有一个“父块”。
只有基于“已经接受的区块”作为“父块”，“领导者”产生的新块才会被接受。创世块被所有分叉接受。

与大多数现有的区块链不同，Stacks块不是原子产生的。当选举领导者时，领导者可以在从用户接收到交易时将交易动态打包为一系列 *“微块”* 。从逻辑上讲，领导者出了一个块；领导者只是不需要在使用权开始时提交即将广播的所有数据。此策略首先在 [Bitcoin-NG](https://www.usenix.org/node/194907) 系统中进行了描述，并在Stacks区块链中进行了一些修改。特别是，领导者会承诺在其任期内必须广播的某些交易，可能会在 *“微块”* 中机会性地传输一些交易。

### PoB带来的新颖属性

比特币底链存储着Stacks区块链的hash值，这使得Stacks相比较其他区块链具备了以下三种不同的属性：

- **全局时间** -- 比特币高度为Stacks区块链提供了时间戳

- **全局区块** -- Stacks区块链并不高度依赖P2P广播，底链为Stacks区块链提供了部分核心信息。

- **全局累计工作量** -- 通过比特币网络可以获取共识相关信息（比如说燃烧了多少BTC、每个分支竞争的情况）



Stacks区块链利用这些属性来实现三个关键功能：

  
- **缓解区块扣缴攻击**
  - [区块扣缴攻击](https://arxiv.org/pdf/1402.1718/): 矿工在矿池中出块，但是不把结果提交给矿池，将结果转移出去再出块。

- **辅助证明提高了链的质量**

- **辅助证明用来对冲赌注**

## 领导人选举

领导者需要燃烧现有加密货币来提交候选人，最终成为潜在的领导者

选择领导者基于以下两个方面：
- 燃烧的加密货币数量
- 公正的随机性来源

每当燃烧链产生新区块时，都会选择一个新的领导者。燃烧链新区块的产生会触发一次领导者选举，并终止当前领导者的任期。

想要成为领导者的人需要在燃烧链发送燃烧交易，然后，网络在诸多燃烧交易中通过加密分类在可验证的随机过程中选择一个领导者，并以燃烧数量之和加权。广播出该消息的块称为Stacks区块链的“选举块”。

任何人都可以通过在基础刻录链上发布刻录交易来提交其作为领导者的候选人资格，并且网络将其选为未来区块的领导者的可能性不为零。

### 向链头提交

任何人都可以成为领导者，这也导致了Stacks 区块链可能有很多竞争分支，其中之一是（canonical fork）标准分支。

多个链头的存在是Stacks区块链的“单领导”设计的直接结果。因为任何人都可以成为领导者，所以这意味着即使“作恶”的领导者也可以被选中。如果领导者在传播其块数据之前崩溃了，或者产生了一个无效的块，那么在其担任领导者期间，不会有任何块被附加到领导者选择的链头上。同样，如果领导者正在根据过时的数据进行操作，那么领导者可能会产生一个其父级不是“最佳”分支上的最新块，在这种情况下，“最佳”分支不会在其时期内增长。这些故障必须由Stacks区块链容忍。

The existence of multiple chain tips is a direct consequence of the
single-leader design of the Stacks blockchain.
Because anyone can become a leader, this means that even misbehaving
leaders can be selected.  If a leader crashes before it can propagate
its block data, or if it produces an invalid block, then no block will
be appended to the leader's selected chain tip during its epoch.  Also, if a
leader is operating off of stale data, then the leader _may_ produce a
block whose parent is not the latest block on the "best" fork, in which case
the "best" fork does not grow during its epoch. These
kinds of failures must be tolerated by the Stacks blockchain.

容忍这些失败的结果是，Stacks区块链可能有多个竞争分叉。其中之一被认为是具有“最佳”链头的标准叉。设计合理的区块链鼓励领导者识别“最佳”叉子，并通过要求他们 _不可逆转地提交_ 到在他们领导的时期将要建立的链头上，来向其添加区块。该承诺必须与一些非自由资源的支出挂钩，例如能源，存储，带宽或（在此区块链的情况下）现有加密货币的支出。直观感受是，如果领导者没有建立在“最佳”链头分叉上，那么它提交区块的行为将会浪费它自己的资源。

A consequence of tolerating these failures is that the Stacks blockchain 
may have multiple competing forks; one of which is considered the canonical fork
with the "best" chain tip.  However, a well-designed
blockchain encourages leaders to identify the "best" fork and append blocks to
it by requiring them to _irrevocably commit_ to the
chain tip they will build on for their epoch.
This commitment must be tied to an expenditure of some
non-free resource, like energy, storage, bandwidth, or (in this blockchain's case) an
existing cryptocurrency.  The intuition is that if the leader does _not_ build on
the "best" fork, then it commits to and loses that resource at a loss.

提交一个链头，但不一定是新数据，可用于鼓励当今其他区块链中的安全性和生命力。例如，在比特币中，矿工搜索包含当前链头的哈希的倒数。如果他们不能赢得那个挑战，那就浪费了资源。由于他们必须尝试越来越深的分叉来发起双花攻击，因此生产竞争性分叉会成倍增加能量消耗。领导者挽回损失的唯一方法是将他们的分叉网络中的其余部分视为“最佳”分叉（即他们铸造的代币是可以消费的）。虽然这不能保证活动性或安全性，但对不将区块附加到“最佳”链头中的领导者进行惩罚，同时奖励这样做的领导者，可以为领导人提供强大的经济诱因，使他们可以在“最佳”叉中建立新区块（生命力），并且不要尝试构建可恢复先前提交的块的备用分支（安全性）。


Committing to a _chain tip_, but not necessarily new data,
is used to encourage both safety and liveness in other blockchains
today. For example, in Bitcoin, miners search for the inverse of a hash that
contains the current chain tip. If they fail to win that block, they
have wasted that energy. As they must attempt deeper and deeper forks
to initiate a double-spend attacks, producing a competing fork becomes an exponentially-increasing energy
expenditure.  The only way for the leader to recoup their losses is for the
fork they work on to be considered by the rest of the network as the "best" fork
(i.e. the one where the tokens they minted are spendable).  While this does
not _guarantee_ liveness or safety, penalizing leaders that do not append blocks to the 
"best" chain tip while rewarding leaders that do so provides a strong economic
incentive for leaders to build and append new blocks to the "best" fork
(liveness) and to _not_ attempt to build an alternative fork that reverts
previously-committed blocks (safety).

Stacks区块链提供相同的鼓励很重要。尤其是，领导者故意孤立块以牟取暴利进行双花攻击的能力是不期望出现的安全违规行为，这样做的领导者必须受到惩罚。在领导者知道是否包括其区块之前，领导者必须公开其链头的提交，从而强制达成目的 - 只有在提交了其燃烧证明的区块被接受为“最佳”分支的情况下，他们才能接收Stacks代币。

It is important that the Stacks blockchain offers the same encouragement.
In particular, the ability for leaders to intentionally orphan blocks in order
to initiate double-spend attacks at a profit is an undesirable safety violation,
and leaders that do so must be penalized.  This property is enforced by
making a leader announce their chain tip commitment _before they know if their
blocks are included_ -- they can only receive Stacks tokens if the block for
which they submitted a proof of burn is accepted into the "best" fork.

### 选举协议

为了鼓励添加新块的安全性和生命力，领导人选举协议要求领导人在知道是否会被选中之前先烧掉加密货币并花费能量。为了实现这一目标，选举领导者的协议分为三个步骤。每位主要候选人都向燃烧链提交两项交易
  - 一笔用于注册用于选举的公钥，
  - 另一笔用于进行代币燃烧和链头记录。
   
这些交易确认后，领导者将会被选出，领导者可以添加新区块并且传播区块数据。

To encourage safety and liveness when appending to the blockchain, the leader
election protocol requires leaders to burn cryptocurrency and spend energy before they know
whether or not they will be selected.  To achieve this, the protocol for electing a leader
runs in three steps.  Each leader candidate submits two transactions to the burn chain -- one to register
their public key used for the election, and one to commit to their token burn and chain tip.
Once these transactions confirm, a leader is selected and the leader can
append and propagate block data.

块选择由可验证的随机函数（VRF）驱动。领导者们提交交易以注册其VRF证明密钥，然后尝试通过在其首选的链尖的种子上生成VRF证明来附加区块-领导者在提交其尖头的证明后会获悉无偏的随机字符串。所得的VRF证明用于通过密码分类选择下一个块以及下一个种子。

Block selection is driven by a _verifiable random function_ (VRF).  Leaders submit transactions to
register their VRF proving keys, and later attempt to append a block by generating a
VRF proof over their preferred chain tip's _seed_ -- an unbiased random string
the leader learns after their tip's proof is committed.  The resulting VRF proof is used to
select the next block through cryptographic sortition, as well as the next seed.

该协议的设计使得领导者只能观察燃烧链数据，并确定可能存在的所有Stacks区块链分支的集合。燃烧链上的数据为所有同级提供了足够的数据，以识别所有可能的链提示，并重构建议的块父关系和块VRF种子。但是，链上数据不指示块或种子是否有效。

The protocol is designed such that a leader can observe _only_ the burn-chain
data and determine the set of all Stacks blockchain forks that can plausibly
exist.  The on-burn-chain data gives all peers enough data to identify all plausible
chain tips, and to reconstruct the proposed block parent relationships and 
block VRF seeds.  The on-burn-chain data does _not_ indicate whether or not a block or a seed is
valid, however.

#### 步骤 1: 注册密钥

在该协议的第一步中，每个领导者候选人都通过发送密钥交易为将来的选举进行注册。 在此交易中，领导者承诺使用公共证明密钥，领导者候选人将使用该密钥为他们建立的链提示生成下一个种子。

必须在燃烧链上充分确认关键交易，然后领导者才能继续下一步的链小费。 例如，领导者可能需要等待10个纪元才能开始提交给链条提示。 确切的数字将由协议定义。

确认后，密钥交易可随时用于提交链提示。 这是因为不能预先确定下一个块的选择。 但是，密钥只能使用一次。

In the first step of the protocol, each leader candidate registers itself for a
future election by sending a _key transaction_. In this transaction, the leader
commits to the public proving key that will be used by the leader candidate to
generate the next seed for the chain tip they will build off of.

The key transactions must be sufficiently confirmed on the burn chain
before the leader can commit to a chain tip in the next step.  For example, the
leader may need to wait for 10 epochs before it can begin committing to a chain
tip.  The exact number will be protocol-defined.

The key transaction can be used at any time to commit to a chain tip, once
confirmed.  This is because the selection of the next block cannot be determined
in advance.  However, a key can only be used once.

#### 步骤 2: 燃烧 & 提交

一旦确定了领导者的关键交易，该领导者将成为竞选下一候选人的候选人，在该候选人必须发送承诺交易。此交易将燃烧领导者的加密货币（燃烧证明），并注册领导者的首选链梢和新的VRF种子，以供在加密分类中选择。

此事务将提交以下信息：

- 产生区块所消耗的加密货币数量
- 块将附加到的链头
- 用来生成区块种子的证明密钥
- 如果选择了该领导者，则需要生成新的VRF种子
- 领导者承诺将包含在其区块中的所有交易数据的摘要（请参阅“领导者的操作”）。
- 种子值是链头端种子的加密哈希（可在刻录链上使用），并且该块的VRF证明是使用组长的证明密钥生成的。 VRF证明本身存储在链下的Stacks块头中，但其哈希值（下一次分类的种子）被提交到链上。

包含候选人承诺交易的刻录链块用作领导者块（即N）的选举块，并用于确定哪个块承诺“获胜”。


Once a leader's key transaction is confirmed, the leader will be a candidate for election
for a subsequent burn block in which it must send a _commitment transaction_.
This transaction burns the leader's cryptocurrency (proof of burn)
and registers the leader's preferred chain tip and new VRF seed
for selection in the cryptographic sortition.

This transaction commits to the following information:

* the amount of cryptocurrency burned to produce the block
* the chain tip that the block will be appended to
* the proving key that will have been used to generate the block's seed
* the new VRF seed if this leader is chosen
* a digest of all transaction data that the leader _promises_ to include in their block (see
  "Operation as a leader").

The seed value is the cryptographic hash of the chain tip's seed (which is available on the burn chain)
and this block's VRF proof generated with the leader's proving key.  The VRF proof
itself is stored in the Stacks block header off-chain, but its hash -- the seed 
for the next sortition -- is committed to on-chain.

The burn chain block that contains the candidates' commitment transaction
serves as the election block for the leader's block (i.e. _N_), and is used to
determine which block commitment "wins."

#### 步骤 3: 抽签

在每个选举区中，所有候选人领导者都进行一次选举（所有链条提示）。 使用以下算法确定下一个块：

```python
# inputs:
#   * BLOCK_HEADER -- the burn chain block header, which contains the PoW nonce
# 
#   * BURNS -- a mapping from public keys to proof of burn scores and block hashes,
#              generated from the valid set of commit & burn transaction pairs.
# 
#   * PROOFS -- a mapping from public keys to their verified VRF proofs from
#               their election transactions.  The domains of BURNS and PROOFS
#               are identical.
#
#   * SEED -- the seed from the previous winning leader
#
# outputs:
#   * PUBKEY -- the winning leader public key
#   * BLOCK_HASH -- the winning block hash 
#   * NEW_SEED -- the new public seed

def make_distribution(BURNS, BLOCK_HEADER):
   DISTRIBUTION = []
   BURN_OFFSET = 0
   BURN_ORDER = dict([(hash(PUBKEY + BLOCK_HEADER.nonce), 
                       (PUBKEY, BURN_AMOUNT, BLOCK_HASH))
                      for (PUBKEY, (BURN_AMOUNT, BLOCK_HASH)) in BURNS.items()])
   for (_, (PUBKEY, BURN_AMOUNT, BLOCK_HASH)) in sorted(BURN_ORDER.items()):
      DISTRIBUTION.append((BURN_OFFSET, PUBKEY, BLOCK_HASH))
      BURN_OFFSET += BURN_AMOUNT
   return DISTRIBUTION

def select_block(SEED, BURNS, PROOFS, BURN_BLOCK_HEADER.nonce):
   if len(BURNS) == 0:
      return (None, None, hash(BURN_BLOCK_HEADER.nonce+ SEED))

   DISTRIBUTION = make_distribution(BURNS)
   TOTAL_BURNS = sum(BURN_AMOUNT for (_, (BURN_AMOUNT, _)) in BURNS)
   SEED_NORM = num(hash(SEED || BURN_BLOCK_HEADER.nonce)) / TOTAL_BURNS
   LAST_BURN_OFFSET = -1
   for (INDEX, (BURN_OFFSET, PUBKEY, BLOCK_HASH)) in enumerate(DISTRIBUTION):
      if LAST_BURN_OFFSET <= SEED_NORM and SEED_NORM < BURN_OFFSET:
         return (PUBKEY, BLOCK_HASH, hash(PROOFS[PUBKEY]))
      LAST_BURN_OFFSET = BURN_OFFSET
   return (DISTRIBUTION[-1].PUBKEY, DISTRIBUTION[-1].BLOCK_HASH, hash(PROOFS[DISTRIBUTION[-1].PUBKEY]))
```

只有一位领导人会赢得选举。不能保证领导者产生的区块是有效的，也不是建立在最佳Stacks分支之上的。但是，一旦选择了领导者，所有对等方都将了解有关领导者决策的足够信息，以便网络中的任何其他对等方都可以提交和中继块数据。至关重要的是，对于任何同龄人，显而易见的是优胜者，而每个候选人都无需事先提交自己的区块。

使用先前的VRF种子和当前的块PoW解决方案对分布进行采样。这样可以确保没有人-甚至不是烧伤链矿工-都不知道将使用PoW种子选择烧伤分数分布证明中的哪个公钥。

领导者可以按自己的意愿进行燃烧链交易并构建区块。只要以正确的顺序广播刻录链事务和区块，领导者就有机会赢得选举。这样就可以实现许多不同的领导者，例如高安全性领导者，在这些领导者中，所有私钥都保存在空白计算机上，并且已签名的块和事务是脱机生成的。

In each election block, there is one election across all candidate leaders (across
all chain tips).  The next block is determined with the following algorithm:



Only one leader will win an election.  It is not guaranteed that the block the
leader produces is valid or builds off of the best Stacks fork.  However,
once a leader is elected, all peers will know enough information about the
leader's decisions that the block data can be submitted and relayed by any other
peer in the network.  Crucially, the winner of the sortition will be apparent to
any peer without each candidate needing to submit their blocks beforehand.

The distribution is sampled using the _previous VRF seed_ and the _current block
PoW solution_.  This ensures that no one -- not even the burn chain miner -- knows
which public key in the proof of burn score distribution will be selected with the PoW seed.

Leaders can make their burn chain transactions and
construct their blocks however they want.  So long as the burn chain transactions
and block are broadcast in the right order, the leader has a chance of winning
the election.  This enables the implementation of many different leaders,
such as high-security leaders where all private keys are kept on air-gapped
computers and signed blocks and transactions are generated offline.

#### On the use of a VRF

When generating the chain tip commitment transaction, a correct leader will need to obtain the
previous election's _seed_ to produce its proof output.  This seed, which is
an unbiased public random value known to all peers (i.e. the hash of the
previous leader's VRF proof), is inputted to each leader candidate's VRF using the private
key it committed to in its registration transaction.  The new seed for the next election is
generated from the winning leader's VRF output when run on the parent block's seed
(which itself is an unbiased random value).  The VRF proof attests that only the
leader's private key could have generated the output value, and that the value
was deterministically generated from the key.

The use of a VRF ensures that leader election happens in an unbiased way.
Since the input seed is an unbiased random value that is not known to
leaders before they commit to their public keys, the leaders cannot bias the outcome of the election 
by adaptively selecting proving keys.
Since the output value of the VRF is determined only from the previous seed and is 
pseudo-random, and since the leader already
committed to the key used to generate it, the leader cannot bias the new
seed value once they learn the current seed.

Because there is one election per burn chain block, there is one valid seed per
epoch (and it may be a seed from a non-canonical fork's chain tip).  However as
long as the winning leader produces a valid block, a new, unbiased seed will be
generated.

In the event that an election does not occur in an epoch, or the leader
does not produce a valid block, the next seed will be
generated from the hash of the current seed and the epoch's burn chain block header
hash.  The reason this is reasonably safe in practice is because the resulting
seed is still unpredictable and impractical (but not infeasible) to bias.  This is because the burn chain miners are
racing each other to find a hash collision using a random nonce, and miners who
want to attempt to bias the seed by continuing to search for nonces that both
bias the seed favorably and solve the burn chain block risk losing the mining race against
miners who do not.  For example, a burn chain miner would need to wait an
expected two epochs to produce two nonces and have a choice between two seeds.
At the same time, it is unlikely that there will be epochs
without a valid block being produced, because (1) attempting to produce a block
is costly and (2) users can easily form burning pools to advance the
state of the Stacks chain even if the "usual" leaders go offline.

As an added security measure, the distribution into which the previous epoch's
VRF seed will index will be randomly structured using the VRF seed and the PoW
nonce.  This dissuades PoW miners from omitting or including burn transactions
in order to influence where the VRF seed will index into the weight
distribution.  Since the PoW miner is not expected to be able
to generate more than one PoW nonce per epoch, the burn chain miners won't know
in advance which leader will be elected.

## Operation as a leader

The Stacks blockchain uses a hybrid approach for generating block data:  it can
"batch" transactions and it can "stream" them.  Batched transactions are
anchored to the commitment transaction, meaning that the leader issues a _leading
commitment_ to these transactions.  The leader can only receive the block reward
if _all_ the transactions committed to in the commitment transaction
are propagated during its tenure.  The
downside of batching transactions, however, is that it significantly increases latency
for the user -- the user will not know that their committed transactions have been
accepted until the _next_ epoch begins.

In addition to sending batched transaction data, a Stacks leader can "stream" a
block over the course of its tenure by selecting transactions from the mempool
as they arrive and packaging them into _microblocks_.  These microblocks
contain small batches of transactions, which are organized into a hash chain to
encode the order in which they were processed.  If a leader produces
microblocks, then the new chain tip the next leader builds off of will be the
_last_ microblock the new leader has seen.

The advantage of the streaming approach is that a leader's transaction can be
included in a block _during_ the current epoch, reducing latency.
However, unlike the batch model, the streaming approach implements a _trailing commitment_ scheme.
When the next leader's tenure begins, it must select either one of the current leader's
microblocks as the chain tip (it can select any of them), or the current
leader's on-chain transaction batch.  In doing so, an epoch change triggers a
"micro-fork" where the last few microblocks of the current leader may be orphaned,
and the transactions they contain remain in the mempool.  The Stacks protocol
incentivizes leaders to build off of the last microblock they have seen (see
below).

The user chooses which commitment scheme a leader should apply for her
transactions.  A transaction can be tagged as "batch only," "stream only," or
"try both."  An informed user selects which scheme based on whether or not they
value low-latency more than the associated risks.

To commit to a chain tip, each correct leader candidate first selects the transactions they will
commit to include their blocks as a batch, constructs a Merkle tree from them, and
then commits the Merkle tree root of the batch
and their preferred chain tip (encoded as the hash of the last leader's
microblock header) within the commitment transaction in the election protocol.
Once the transactions are appended to the burn chain, the leaders execute
the third round of the election protocol, and the
sortition algorithm will be run to select which of the candidate leaders will be
able to append to the Stacks blockchain.  Once selected, the new leader broadcasts their
transaction batch and then proceeds to stream microblocks.

### Building off the latest block

Like existing blockchains, the leader can selet any prior block as its preferred
chain tip.  In the Stacks blockchain, this allows leaders to tolerate block loss by building
off of the latest-built ancestor block's parent.

To encourage leaders to propagate their batched transactions if they are selected, a
commitment to a block on the burn chain is only considered valid if the peer
network has (1) the transaction batch, and (2) the microblocks the leader sent
up to the next leader's chain tip commitment on the same fork.  A leader will not receive any compensation
from their block if any block data is missing -- they eventually must propagate the block data
in order for their rewards to materialize (even though this enables selfish
mining; see below).

The streaming approach requires some additional incentives to
encourage leaders to build off of the latest known chain tip (i.e. the latest
microblock sent by the last leader).  In particular, the streaming model enables
the following two safety risks that are not present in the batching approach:

* A leader who gets elected twice in a row can adaptively orphan its previous
  microblocks by building off of its first tenures' chain tip, thereby
double-spending transactions the user may believe are already included.

* A leader can be bribed during their tenure to omit transactions that are
  candidates for streaming.  The price of this bribe is much smaller than the
cost to bribe a leader to not send a block, since the leader only stands to lose
the transaction fees for the targeted transaction and all subsequently-mined
transactions instead of the entire block reward.  Similarly, a leader can
be bribed to mine off of an earlier microblock chain tip than the last one it has seen
for less than the cost of the block reward.

To help discourage both self-orphaning and "micro-bribes" to double-spend or
omit specific transactions or trigger longer-than-necessary micro-forks, leaders are
rewarded only 40% of their transaction fees in their block reward (including
those that were batched).  They receive
60% of the previous leader's transaction fees.  This result was shown in the
Bitcoin-NG paper to be necessary to ensure that honest behavior is the most
profitable behavior in the streaming model.

The presence of a batching approach is meant to raise the stakes for a briber.
Users who are worried that the next leader could orphan their transactions if
they were in a microblock would instead submit their transactions to be batched.
Then, if a leader selects them into its tenure's batch, the leader would
forfeit the entire block reward if even one of the batched transactions was
missing.  This significantly increases the bribe cost to leaders, at the penalty
of higher latency to users.  However, for users who need to send 
transactions under these circumstances, the wait would be worth it.

Users are encouraged to use the batching model for "high-value" transactions and
use the streaming model for "low-value" transactions.  In both cases, the use
of a high transaction fee makes their transactions more likely to be included in
the next batch or streamed first, which additionally raises the bribe price for
omitting transactions.

### Leader volume limits

A leader propagates blocks irrespective of the underlying burn chain's capacity.
This poses a DDoS vulnerability to the network:  a high-transaction-volume
leader may swamp the peer network with so many
transactions and microblocks that the rest of the nodes cannot keep up.  When the next
epoch begins and a new leader is chosen, it would likely orphan many of the high-volume
leader's microblocks simply because its view of the
chain tip is far behind the high-volume leader's view. This hurts the
network, because it increases the confirmation time of transactions
and may invalidate previously-confirmed transactions.

To mitigate this, the Stack chain places a limit on the volume of
data a leader can send during its epoch (this places a _de facto_ limit
on the number of transactions in a Stack block). This cap is enforced
by the consensus rules.  If a leader exceeds this cap, the block is invalid.

### Batch transaction latency

The fact that leaders execute a leading commmitment to batched transactions means that
it takes at least one epoch for a user to know if their transaction was
incorporated into the Stacks blockchain.  To get around this, leaders are
encouraged to to supply a public API endpoint that allows a user to query
whether or not their transaction is included in the burn (i.e. the leader's
service would supply a Merkle path to it).  A user can use a set of leader
services to deduce which block(s) included their transaction, and calculate the
probability that their transaction will be accepted in the next epoch.
Leaders can announce their API endpoints via the [Blockstack Naming
Service](https://docs.blockstack.org/core/naming/introduction.html).

The specification for this transaction confirmation API service is the subject
of a future SIP.  Users who need low-latency confirmations today and are willing
to risk micro-forks and intentional orphaning can submit their transactions for
streaming.

## Burning pools

Proof-of-burn mining is not only concerned with electing leaders, but also concerned with
enhancing chain quality.  For this reason, the Stacks chain not
only rewards leaders who build on the "best" fork, but also each peer who
supported the "best" fork by burning cryptocurrency in support of the winning leader.
The leader that commits to the winning chain tip and the peers who also burn for
that leader collectively share in the block's reward, proportional to how much
each one burned.

### Encouraging honest leaders

The reason for allowing users to support leader candidates at all is to help
maintain the chain's liveness in the presence of leaders who follow the
protocol correctly, but not honestly.  These include leaders who delay
the propagation of blocks and leaders who refuse to mine certain transactions.
By giving users a very low barrier to entry to becoming a leader, and by giving
other users a way to help a known-good leader candidate get selected, the Stacks blockchain
gives users a first-class stake in deciding which transactions to process
as well as incentivizes them to maintain chain liveness in the face of bad
leaders.  In other words, leaders stand to make more make money with
the consent of the users.

Users support their preferred leader by submitting a burn transaction that contains a 
proof of burn and references its leader candidate's chain tip commitment.  These user-submitted
burns count towards the leader's total score for the election, thereby increasing the chance
that they will be selected (i.e. users submit their transactions alongside the
leader's block commitment).  Users who submit proofs for a leader that wins the election
will receive some Stacks tokens alongside the leader (but users whose leaders
are not elected receive no reward).  Users are rewarded the same way as
leaders -- they receive their tokens during the reward window (see below).

Allowing users to vote in support of leaders they prefer gives users and leaders
an incentive to cooperate.  Leaders can woo users to submit proofs for them by committing
to honest behavior, and users can help prevent dishonest (but more profitable)
leaders from getting elected.  Moreover, leaders cannot defraud users who submit
proofs in their support, since users are rewarded by the election protocol itself.

### Fair mining

Because all peers see the same sequence of burns in the Stacks blockchain, users
can easily set up distributed mining pools where each user receives a fair share
of the block rewards for all blocks the pool produces.  The selection of a
leader within the pool is arbitrary -- as long as _some_ user issues a key
transaction and a commitment transaction, the _other_ users in the pool can
throw their proofs of burn behind a chain tip.  Since users who submitted proofs for the winning
block are rewarded by the protocol, there is no need for a pool operator to
distribute rewards.  Since all users have global visibility into all outstanding
proofs, there is no need for a pool operator to direct users to work on a
particular block -- users can see for themselves which block(s) are available by
inspecting the on-chain state.

Users only need to have a way to query what's going into a block when one of the pool
members issues a commitment transaction.  This can be done easily for batched
transactions -- the transaction sender can prove that their transaction is
included by submitting a Merkle path from the root to their transaction.  For
streamed transactions, leaders have a variety of options for promising users
that they will stream a transaction, but these techniques are beyond the scope of this SIP.

### Minimizing reward variance

Leaders compete to elect the next block by burning more cryptocurrency and/or
spending more energy.  However, if they lose the election, they lose the cryptocurrency they burned.
This makes for a "high variance" pay-out proposition that puts leaders in a
position where they need to maintain a comfortable cryptocurrency buffer to
stay solvent.

To reduce the need for such a buffer, making proofs of burn to support competing chain tips
enables leaders to hedge their bets by generating proofs to support _all_ plausible
competing chain tips.  Leaders have the option of submitting proofs in support for a
_distribution_ of competing chain tips at a lower cost than committing to many
different chain tips as leaders.  This gives them the ability to receive some
reward no matter who wins.  This also reduces the barrier to
entry for becoming a leader in the first place.

### Leader support mechanism

There are a couple important considerations for the mechanism by which peers
submit proofs for their preferred chain tips.

* Users and runner-up leaders are rewarded strictly fewer tokens
for committing to a chain tip that does not get selected.  This is
  important because leaders and users are indistinguishable
on-chain.  Leaders should not be able to increase their expected reward by sock-puppeting,
and neither leaders nor users should get an out-sized reward for voting for
invalid blocks or blocks that will never be appended to the canonical fork.

* It must be cheaper for a leader to submit a single expensive commitment than it is
  to submit a cheap commitment and a lot of user-submitted proofs.  This is
important because it should not be possible for a leader to profit more from
adaptively increasing their proof submissions in response to other leaders'.

The first property is enforced by the reward distribution rules (see below),
whereby a proof commitment only receives a reward if its block successfully extended the
"canonical" fork.  The second property is given "for free" because the underlying burn chain
assesses each participant a burn chain transaction fee.  Users and leaders incur an ever-increasing
cost of trying to adaptively out-vote other leaders by submitting more and more
transactions.  Further, peers who want to support a leader candidate must send their burn transactions _in the
same burn chain block_ as the commitment transaction.  This limits the degree to
which peers can adaptively out-bid each other to include their
commitments.

## Reward distribution

New Stacks tokens come into existence on a fork in an epoch where a leader is
selected, and are granted to the leader if the leader produces a valid block.
However, the Stacks blockchain pools all tokens created and all transaction fees received and
does not distribute them until a large number of epochs (a _lockup period_) has
passed.  The tokens cannot be spent until the period passes.

### Sharing the rewards among winners

Block rewards (coinbases and transaction fees) are not granted immediately,
but are delayed for a lock-up period.  Once the lock-up period passes,
the exact reward distribution is as follows:

* Coinbases: The coinbase (newly-minted tokens) for a block is rewarded to the leader who
  mined the block, as well as to all individuals who submitted proofs-of-burn in 
support of it.  Each participant (leaders and supporting users) recieves a
portion of the coinbase proportional to the fraction of total tokens destroyed.

* Batched transactions:  The transaction fees for batched transactions are
  distributed exclusively to the leader who produced the block, provided that
the block has enough transactions. 

   To discourage mining empty blocks, an anchored block must be _F_% "full" for the
   leader to receive its transaction fees.  A block's "fullness" is measured by
   how much transaction-computing capacity the block has consumed (see SIP 006).
   Failure to mine a block that is at least _F_% full will be penalized:
   if the miner does not fill the block to at least _F_% capacitiy, then the
   miner will receive _P * M_ STX instead of the transaction fees, where:

   * _0 < M_ is the minimum allowable transaction fee rate,
   * _0 < P < F_ is the fraction of the block that the miner was able to fill.

   Note that _P * M_ is strictly less than the lowest possible sum of the
   transaction fees of any _F_%-full block.

   This is in the service of implementing the fee auction strategy described in [1]. 
   However, unlike in [1], no transaction fee smoothing will take place -- the
   leader receives all of the anchored block's transaction fees.

* Streamed transactions:  the transaction fees for streamed transactions are
  distributed according to a 60/40 split -- the leader that validated the
transactions is awarded 60% of the transaction fees, and the leader that builds
on top of them is awarded 40%.  This ensures that leaders are rewarded for
processing and validating transactions correctly _while also_ incentivizing the
subsequent leader to include them in their block, instead of orphaning them.

## Recovery from data loss

Stacks block data can get lost after a leader commits to it.  However, the burn
chain will record the chain tip, the batched transactions' hash, and the leader's public
key.  This means that all existing forks will be
known to the Stacks peers that share the same view of the burn chain (including
forks made of invalid blocks, and forks that include blocks whose data was lost
forever).

What this means is that regardless of how the leader operates, its
chain tip commitment strategy needs a way to orphan a fork of any length.  In
correct operation, the network recovers from data loss by building an
alternative fork that will eventually become the "best" fork,
thereby recovering from data loss and ensuring that the system continues
to make progress.  Even in the absence of malice, the need for this family of strategies
follows directly from a single-leader model, where a peer can crash before
producing a block or fail to propagate a block during its tenure.

However, there is a downside to this approach: it enables **selfish mining.**  A
minority coalition of leaders can statistically gain more Stacks tokens than they are due from
their tunable proof scores by attempting to build a hidden fork of blocks, and releasing it
once the honest majority comes within one block height difference of the hidden
fork.  This orphans the majority fork, causing them to lose their Stacks tokens
and re-build on top of the minority fork.

### Seflish mining mitigation strategies

Fortunately, all peers in the Stacks blockchain have global knowledge of state,
 time, and tunable proofs.  Intuitively, this gives the Stacks blockchain some novel tools
for dealing with selfish leaders:

* Since all nodes know about all blocks that have been committed, a selfish leader coalition
  cannot hide its attack forks. The honest leader coalition can see
the attack coming, and evidence of the attack will be preserved in the burn
chain for subsequent analysis.  This property allows honest leaders
to prepare for and mitigate a pending attack by burning more
cryptocurrency, thereby reducing the fraction
of votes the selfish leaders wield below the point where selfish mining is profitable (subject to network
conditions).

* Since all nodes have global knowledge of the passage of time, honest leaders
  can agree on a total ordering of all chain tip commits and tunable proofs.  In certain kinds of
selfish mining attacks, this gives honest leaders the ability to identify and reject an attack fork 
with over 50% confidence.  In particular, honest leaders who have been online long
enough to measure the expected block propagation time would _not_ build on top of
a chain tip whose last _A > 1_ blocks arrived late, even if that chain tip
represents the "best" fork, since this would be the expected behavior of a selfish miner.

* Since all nodes know about all block commitment transactions, the long tail of small-time participants
(i.e. users who support leaders) can collectively throw their resources behind
known-honest leaders' transactions.  This increases the chance that honest leaders will
be elected, thereby increasing the fraction of honest voting power and making it
harder for a selfish leader to get elected.

* The Stacks chain reward system spreads out rewards for creating
blocks and mining transactions across a large interval.  This "smooths over"
short-lived selfish mining attacks -- while selfish leaders still receive more
than their fair share of the rewards, the lower variance imposed by the
reward window makes this discrepancy smaller.

* All Stacks nodes relay all blocks that correspond to on-chain commitments,
even if they suspect that they came from the attacker.  If an honest leader finds two chain tips of equal
length, it selects at random which chain tip to build off of.  This ensures that
the fraction of honest voting that builds on top of the attack fork versus the honest fork
is statistically capped at 50% when they are the same length.

None of these points _prevent_ selfish mining, but they give honest users and
honest leaders the tools to make selfish mining more difficult to pull off than in
PoW chains.  Depending on user activity, they also make economically-motivated
leaders less likely to participate in a selfish miner cartel -- doing so always produces evidence,
which honest leaders and users can act on to reduce or eliminate 
their expected rewards.

Nevertheless, these arguments are only intuitions at this time.  A more
rigorous analysis is needed to see exactly how these points affect the
profitibility of selfish mining.  Because the Stacks blockchain represents a
clean slate blockchain design, we have an opportunity to consider the past
several years of research into attacks and defenses against block-hiding
attacks.  This section will be updated as our understanding evolves.

## Fork Selection

Fork selection in the Stacks blockchain requires a metric to determine which
chain, between two candidates, is the "best" chain.  For Stacks, **the fork with
the most blocks is the best fork.**  That is, the Stacks blockchain measures the
quality of block _N_'s fork by the total amount of _blocks_ which _confirm_
block _N_.

Using chain length as the fork choice rule makes it time-consuming for alternative forks to
overtake the "canonical" fork, no matter how much capacity they have to submit tunable proofs.
In order to carry out a deep fork of _K_ blocks, the majority coalition of participants needs to spend
at least _K_ epochs working on the new fork. We consider this acceptable
because it also has the effect of keeping the chain history relatively stable, 
and makes it so every participant can observe (and prepare for) any upcoming
forks that would overtake the canonical history.  However, a minority
coalition of dishonest leaders can create short-lived forks by continuously
building forks (i.e. in order to selfishly mine), driving up the confirmation
time for transactions in the honest fork.

This fork choice rule implies a time-based transaction security measurement.  A
transaction _K_ blocks in the past will take at least _K_ epochs to reverse.
The expected cost of doing so can be calculated given the total amount of burned
tokens put into producing blocks, and the expected fraction of the
totals controlled by the attacker.  Note that the attacker is only guaranteed to
reverse a transaction _K_ blocks back if they consistently control over 50% of the total
tunable proof score.

## Implementation

The Stacks blockchain leader election protocol will be written in Rust.

## References

[1] Basu, Easley, O'Hara, and Sirer. [Towards a Functional Market for Cryptocurrencies.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3318327)

## Appendix

### Definitions

**Burn chain**: the blockchain whose cryptocurrency is destroyed in burn-mining.

**Burn transaction**: a transaction on the burn chain that a Stacks miner issues
in order to become a candidate for producing a future block.  The transaction
includes the chain tip to append to, and the proof that cryptocurrency was
destroyed.

**Chain tip**: the location in the blockchain where a new block can be appended.
Every valid block is a valid chain tip, but only one chain tip will correspond
to the canonical transaction history in the blockchain.  Miners are encouraged
to append to the canonical transaction history's chain tip when possible.

**Cryptographic sortition** is the act of selecting the next leader to
produce a block on a blockchain in an unbiased way.  The Stacks blockchain uses
a _verifiable random function_ to carry this out.

**Election block**: the block in the burn chain at which point a leader is
chosen.  Each Stacks block corresponds to exactly one election block on the burn
chain.

**Epoch**: a discrete configuration of the leader and leader candidate state in
the Stacks blockchain.  A new epoch begins when a leader is chosen, or the
leader's tenure expires (these are often, but not always, the same event).

**Fork**: one of a set of divergent transaction histories, one of which is
considered by the blockchain network to be the canonical history
with the "best" chain tip.

**Fork choice rule**: the programmatic rules for deciding how to rank forks to select
the canonical transaction history.  All correct peers that process the same transactions
with the same fork choice rule will agree on the same fork ranks.

**Leader**: the principal selected to produce the next block in the Stacks
blockchain.  The principal is called the block's leader.

**Reorg** (full: _reorganization_): the act of a blockchain network switching
from one fork to another fork as its collective choice of the canonical
transaction history.  From the perspective of an external observer,
such as a wallet, the blockchain appears to have reorganized its transactions.

## 引用文献
[Bitcoin-NG](https://www.usenix.org/system/files/conference/nsdi16/nsdi16-paper-eyal.pdf)

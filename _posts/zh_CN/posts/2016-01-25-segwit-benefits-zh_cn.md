---
layout: post
name: segwit-benefits
id: en-segwit-benefits
title: 隔离见证的好处
permalink: /en/2016/01/25/segwit-benefits
version: 1
excerpt: >
  This page summarises some of the benefits of segregated witness that
  go beyond simply increasing the capacity of the block chain.
  本页总结了一些隔离见证的好处, 不仅仅是增加块链的容量。
---
{% include _toc.html %}

The Segregated Witness soft-fork (segwit) includes a wide range of
features, many of which are highly technical. This page summarises
some of the benefits of those features that go beyond simply increasing
the capacity of the block chain.
隔离见证软分叉（segwit）包括很多的特性，其中许多是技术性很强的。
此页面总结了那些除了增加块链的容量之外特性的好处。

## 修复延展性问题


Bitcoin transactions are identified by a 64-digit hexadecimal hash
called a transaction identifier (txid)
which is based on both the coins being spent and on who
will be able to spend the results of the transaction.
比特币交易是由一串64位的十六进制哈希交易ID（TxID）标识的，这是基于
交易中币的来源和币的接收者确定的。

Unfortunately,
不幸的是,
the way the txid is calculated allows anyone to make small
modifications to the transaction that will not change its meaning,
but will change the txid.  This is called third-party malleability.
BIP 62 ("dealing with malleability") attempted to
address these issues in a piecemeal manner, but was too complicated to
implement as consensus checks and has been withdrawn.
TxID的计算方法允许任何人可以对交易做小的修动，虽然不会改变交易的内容，但会改变TxID。
这就是所谓的第三方延展性。BIP62（"处理可塑性"）试图以一些方式解决这些问题
，但是它太复杂了，以至于无法实现为共识检查,所以已经被放弃了。

For example, you could submit a transaction with txid ef74...c309 to
the network, but instead find that a third-party, such as a node on
the network relaying your transaction, or the miner who includes your
transaction in a block, modifies the transaction slightly, resulting in
your transaction still spending the same coins and paying the same addresses,
but being confirmed under the completely different txid 683f...8bfa instead.
例如，您可以发送TxID为ef74... C309到比特币网络，当网络中的节点中继
这笔交易，或者矿工打包交易到区块中的时候，它们可以轻微修改这笔交易，导致您的
交易仍然花一样的币，并支付到相同的地址，但是以完全不同的TxID 683f...8bfa出现。

More generally, if one or more of the signers of the transaction revise
their signatures then the transaction remains valid and pays the same
amounts to the same addresses, but the txid changes completely
because it incorporates the signatures. The general case of changes to
signature data (but not the outputs or choice of inputs) modifying the
transaction is called scriptSig malleability.
更通俗的说，如果一笔交易的一个或更多个签名者修改他们的签名, 那么交易仍然有效
并且支付相同的比特币，以相同的地址，但这笔交易的TxID完全改变，因为它们组成签名。
更改签名数据（但不改变output或input）来修改交易的情况称为scriptSig延展性。

Segwit prevents third-party and scriptSig malleability by allowing Bitcoin users to move the
malleable parts of the transaction into the *transaction witness,* and
segregating that witness so that changes to the witness does not affect
calculation of the txid.
Segwit可以防止第三方和scriptSig延展性, 通过把比特币交易中的的可修改部分移动到 
*见证交易*里, 并且分离后不影响TxID的计算。

### 谁能从这个特性得到好处?

- **Wallet authors tracking spent bitcoins:** it's easiest to
  monitor the status of your own outgoing transactions by simply looking them up by txid.  But in
  a system with third-party malleability, wallets must implement extra
  code to be able to deal with changed txids.

- **钱包作者用来监控发出比特币：** 这是最简单的，只需要监控发出的TxID的状态就
  可以。 但是在存在第三方延展性问题的系统里，钱包必须添加额外的代码，以便能够
  应对变化的txids。

- **Anyone spending unconfirmed transactions:** if Alice pays Bob in
  transaction 1, Bob uses that payment to pay Charlie in transaction 2, and then Alice's
  payment gets malleated and confirmed with a different txid, then transaction
  2 is now invalid and Charlie has not been paid.
  If Bob is trustworthy, he will reissue the payment to Charlie; but if he
  isn't, he can simply keep those bitcoins for himself.

-  **花费未确认的交易：** 如果Alice在交易1支付Bob一些币，Bob在交易2
   使用收到的付款支付给Charlie，然后Alice的付款发生延展性修改,并用不同的TxID
   确认，那么现在交易2现在是无效的, 而Charlie查理就不会被支付。
   如果Bob是值得信赖的，他会重新发出一笔交易给查理;但如果他不是，他可以简单
   地保持这些比特币给自己。

- **The Lightning Network:** with third-party and scriptSig malleability
  fixed, the Lightning Network is less complicated to implement and
  significantly more efficient in its use of space on the blockchain.
  With scriptSig malleability removed, it also becomes possible to run lightweight
  Lightning clients that outsource monitoring the blockchain, instead of
  each Lightning client needing to also be a full Bitcoin node.

- **闪电网络:** 第三方和scriptSig延展性问题修复后可以降低闪电网络实现的的复杂性
  ， 而且在使用blockchain的空间上将更加有效. scriptSig延展性删除后，它也可能运
  行轻量级的lighting客户端服务去监测区块链,而不是每个lightning客户端都运行比特
  币完整节点。

- **Anyone using the block chain:** smart contracts today, such as
  micropayment channels, and anticipated new smart contracts, become less
  complicated to design, understand, and monitor.
- **任何使用区块链的人:** 目前的智能合约，比如小额支付通道，预期新的智能合同，
  将会变得不用那么复杂的设计，理解和监控。

Note: segwit transactions only avoid malleability if all their inputs are
segwit spends (either directly, or via a backwards compatible segwit
P2SH address).
注意：segwit交易只能在当所有input都是segwit交易（直接或经由一个向后兼容
的segwit P2SH地址）下避免延展性问题。

### 更多信息

 * [Bitcoin Wiki on Malleability](https://en.bitcoin.it/wiki/Transaction_Malleability)
 * [Coin Telegraph article on 2015 Malleability attack](http://cointelegraph.com/news/115374/the-ongoing-bitcoin-malleability-attack)
 * [Bitcoin Magazine article on 2015 Malleability attack](https://bitcoinmagazine.com/articles/the-who-what-why-and-how-of-the-ongoing-transaction-malleability-attack-1444253640)
 * ["Overview of BIPs necessary for Lightning" transcript](http://diyhpl.us/wiki/transcripts/scalingbitcoin/hong-kong/overview-of-bips-necessary-for-lightning/)
 * [BIP 62](https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki)
 * [BIP 140 -- alternative approach to malleability fixes](https://github.com/bitcoin/bips/blob/master/bip-0140.mediawiki)
 * [Stack exchange answer regarding 683f...8bfa transaction](http://bitcoin.stackexchange.com/questions/22051/transaction-malleability-in-the-blockchain/22058#22058)

## 线性增长sighash的操作

A major problem with simple approaches to increasing the Bitcoin blocksize
is that for certain transactions, signature-hashing scales quadratically
rather than linearly.
用简单的方法来增加比特币区块大小的一个主要问题是，对于某些交易，签名散列增长
是平方增长的, 而不是线性增长的。

![Linear versus quadratic](/assets/images/linear-quad-scale.png)

In essence, doubling the size of a transaction
increases can double both the number of signature operations, and the
amount of data that has to be hashed for each of those signatures to
be verified. This has been seen in the wild, where an individual block
required 25 seconds to validate, and maliciously designed transactions
could take over 3 minutes.
在本质上，一个交易的大小增加一倍,签名操作的个数也增加一倍, 以及那些进行验证签名
需要哈希的数据也应该增加一倍. 但曾经已经出现过，一个单独的块需要25秒验证,其他
一些恶意设计的交易可能需要超过3分钟。

Segwit resolves this by changing the calculation of the transaction hash
for signatures so that each byte of a transaction only needs to be hashed
at most twice. This provides the same functionality more efficiently,
so that large transactions can still be generated without running into
problems due to signature hashing, even if they are generated maliciously
or much larger blocks (and therefore larger transactions) are supported.
Segwit通过改变交易哈希签名的计算方式可以解决此问题，使得交易的每个字节只需要至
多两次哈希。 这提供了相同的功能但更有效率，使得大的交易仍可以产生而不会有签名
哈希问题，即使有人生成恶意的或更大的块（并较大的交易）也是支持的。

### Who benefits?

Removing the quadratic scaling of hashed data for verifying signatures
makes increasing the block size safer. Doing that without also limiting
transaction sizes allows Bitcoin to continue to support payments that go to or
come from large groups, such as payments of mining rewards or crowdfunding
services.

The modified hash only applies to signature operations initiated from
witness data, so signature operations from the base block will continue
to require lower limits.

### Further information

 * [BIP 143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki)
 * [Blog post by Rusty Russell on the 25s transaction](http://rusty.ozlabs.org/?p=522)
 * [CVE 2013-2292 on Bitcoin wiki](https://en.bitcoin.it/wiki/Common_Vulnerabilities_and_Exposures#CVE-2013-2292)
 * [Proposal to limit transactions to 100kB](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2015-July/009494.html)
 * [Bitcoin Classic commit on 0.11.2 branch adding additional consensus limit on sighash bytes](https://github.com/bitcoinclassic/bitcoinclassic/commit/842dc24b23ad9551c67672660c4cba882c4c840a)

## Signing of input values

When a hardware wallet signs a transaction, it can easily verify the total
amount being spent, but can only safely determine the fee by having a
full copy of all the input transactions being spent, and must hash each
of those to ensure it is not being fed false data. Since individual
transactions can be up to 1MB in size, this is not necessarily a cheap
operation, even if the transaction being signed is itself quite small.

Segwit resolves this by explicitly hashing the input value. This means
that a hardware wallet can simply be given the transaction hash, index,
and value (and told what public key was used), and can safely sign the
spending transaction, no matter how large or complicated the transaction
being spent was.

### Who benefits?

Manufacturers and users of hardware wallets are the obvious beneficiaries;
however this likely also makes it much easier to safely use Bitcoin in
small embedded devices for "Internet of things" applications.

This benefit is only available when spending transactions sent to segwit
enabled addresses (or segwit-via-P2SH addresses).

### Further information

 * [BIP 143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki)

## Increased security for multisig via pay-to-script-hash (P2SH)

Multisig payments currently use P2SH which is secured by the 160-bit
HASH160 algorithm (RIPEMD of SHA256). However, if one of the signers
wishes to steal all the funds, they can find a collision between a valid
address as part of a multisig script and a script that simply pays them
all the funds with only 80-bits (2<sup>80</sup>) worth of work, which is
already within the realm of possibility for an extremely well-resourced
attacker. (For comparison, at a sustained 1 exahash/second, the Bitcoin
mining network does 80-bits worth of work every two weeks)

Segwit resolves this by using HASH160 only for payments direct to a
single public key (where this sort of attack is useless), while using
256-bit SHA256 hashes for payments to a script hash.

### Who benefits?

Everyone paying to multisig or smart contracts via segwit benefits from
the extra security provided for scripts.

### Further information

 * [Gavin Andresen asking if 80-bit attacks are worth worrying about](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2016-January/012198.html)
 * [Ethan Heilman describing a cycle finding algorithm](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2016-January/012202.html)
 * [Rusty Russell calculating costs of performing an attack](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2016-January/012227.html)
 * [Anthony Towns applying the cycle finding algorithm to exploit transactions](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2016-January/012218.html)
 * [Gavin Andresen summarising the thread](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2016-January/012234.html)

## Script versioning

Changes to Bitcoin's script allow for both improved security and
improved functionality. However, the design of script only allows
backwards-compatible (soft-forking) changes to be implemented by replacing
one of the ten extra OP\_NOP opcodes with a new opcode that can conditionally fail the script,
but which otherwise does nothing. This is sufficient for many changes --
such as introducing a new signature method or a feature like OP\_CLTV,
but it is both slightly hacky (for example, OP\_CLTV usually has to be accompanied by
an OP\_DROP) and cannot be used to enable even features as simple
as joining two strings.

Segwit resolves this by including a version number for scripts, so that
additional opcodes that would have required a hard-fork to be used
in non-segwit transactions can instead be supported by simply increasing the
script version.

### Who benefits?

Easier changes to script opcodes will make advanced scripting in Bitcoin
easier. This includes changes such as introducing Schnorr signatures,
using key recovery to shrink signature sizes, supporting sidechains,
or creating even smarter contracts by using Merklized Abstract Syntax
Trees (MAST) and other research-level ideas.

## Reducing UTXO growth

The Unspent Transaction Output (UTXO) database is maintained by each
validating Bitcoin node in order to determine whether new transactions are
valid or fraudulent. For efficient operation of the network, this database
needs to be very quick to query and modify, and should ideally be able
to fit in main memory (RAM), so keeping the database's
size in bytes as small as possible is valuable.

This becomes more difficult as Bitcoin grows, as each new user must have
at least one UTXO entry of their own and will prefer having multiple
entries to help improve their privacy and flexibility, or to provide as
backing for payment channels or other smart contracts.

Segwit improves the situation here by making signature data, which does
not impact the UTXO set size, cost 75% less than data that does impact
the UTXO set size. This is expected to encourage users to favour the
use of transactions that minimise impact on the UTXO set in order to
minimise fees, and to encourage developers to design smart contracts and
new features in a way that will also minimise the impact on the UTXO set.

Because segwit is a soft-forking change and does not increase the base
blocksize, the worst case growth rate of the UTXO set stays the same.

### Who benefits?

Reduced UTXO growth will benefit miners, businesses, and users who run full nodes,
which in turn helps maintain the current security of the Bitcoin network
as more users enter the system. Users and developers who help minimise the
growth of the UTXO set will benefit from lower fees compared to those who
ignore the impact of their transactions on UTXO growth.

### Further information

 * [Statoshi UTXO dashboard](http://statoshi.info/dashboard/db/unspent-transaction-output-set)

## Compact fraud proofs

As the Bitcoin userbase expands, validating the entire blockchain
naturally becomes more expensive. To maintain the decentralised, trustless
nature of Bitcoin, it is important to allow those who cannot afford to
validate the entire blockchain to at least be able to cheaply validate
as much of it as they can afford.

Segwit improves the situation here by allowing a future soft-fork to
extend the witness structure to include commitment data, which will allow lightweight (SPV)
clients to enforce consensus rules such such as the number of bitcoins
introduced in a block, the size of a block, and the number of sigops
used in a block.

### Who benefits?

Fraud proofs allow SPV users to help enforce Bitcoin's consensus rules,
which will potentially greatly increase the security of the Bitcoin
network as a whole, as well as reduce the ways in which individual users
can be attacked.

These fraud proofs can be added to the witness data structure as part
of a future soft-fork, and they'll help SPV clients enforce the rules
even on transactions that don't make use of the segwit features.

## Efficiency gains when not verifying signatures

Signatures for historical transactions may be less interesting than
signatures for future transactions -- for example, Bitcoin Core does not
check signatures for transactions prior to the most recent checkpoint by
default, and some SPV clients simply don't check signatures themselves
at all, trusting that has already been done by miners or other nodes.
At present, however, signature data is an integral part of the transaction
and must be present in order to calculate the transaction hash.

Segregating the signature data allows nodes that aren't interested in
signature data to prune it from the disk, or to avoid downloading it in the
first place, saving resources.

### Who benefits?

As more transactions use segwit addresses, people running pruned or SPV
nodes will be able to operate with less bandwidth and disk space.

## Moving towards a single combined block limit

Currently there are two consensus-enforced limits on blocksize: the
block can be no larger than 1MB and, independently, there can be no
more than 20,000 signature checks performed across the transactions in
the block.

Finding the most profitable set of transactions to include
in a block given a single limit is an instance of the knapsack
problem, which can be easily solved almost perfectly with a simple
greedy algorithm. However adding the second constraint makes finding a
good solution very hard in some cases, and this theoretical problem has
been exploited in practice to force blocks to be mined at a size well
below capacity.

It is not possible to solve this problem without either a hardfork,
or substantially decreasing the block size.  Since segwit can't fix the
problem, it settles on not making it worse: in particular, rather than
introducing an independent limit for the segregated witness data,
instead a single limit is applied to the weighted sum
of the UTXO data and the witness data, allowing both to be
limited simultaneously as a combined entity.

## Who benefits?

Ultimately miners will benefit if a future hardfork that changes the block capacity limit to be a single
weighted sum of parameters. For example:

    50*sigops + 4*basedata + 1*witnessdata < 10M

This lets miners easily and accurately fill blocks
while maximising fee income, and that will benefit users by allowing them
to more reliably calculate the appropriate fee needed for their transaction
to be mined.

### Further information

 * [Knapsack problem](https://en.wikipedia.org/wiki/Knapsack_problem)
 * [Sigop attack discussion on bitcointalk in Aug 2015](https://bitcointalk.org/index.php?topic=1166928.0;all)
 * [Gregory Maxwell on bitcoin-dev on witness limits](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2015-December/011870.html)
 * ["Validation Cost Metric" transcript](http://diyhpl.us/wiki/transcripts/scalingbitcoin/hong-kong/validation-cost-metric/)

在本质上，一个交易的大小增加一倍,签名操作的个数也增加，以及数据的具有待哈希每个的量加倍这些签名进行验证。

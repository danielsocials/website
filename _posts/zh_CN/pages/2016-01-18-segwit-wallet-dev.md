---
version: 1
title: 隔离验证钱包开发指南
name: segwit-wallet-dev
type: page
permalink: /en/segwit_wallet_dev
---

### 隔离验证编程

有2种实现隔离验证的方式:

#### 原生的隔离验证脚本

原生的隔离验证脚本定义为一个<code>scriptPubKey</code>,格式为1字节的push指令(<code>OP_0</code>, <code>OP_1</code>, ...,  <code>OP_16</code>)后面跟着2到32字节数据. 

#### 嵌套在P2SH中

隔离验证脚本嵌套在P2SH中是用一个<code>redeemScript</code>存放隔离验证脚本,也是一个<code>scriptPubKey</code>,格式为1字节的push指令(<code>OP_0</code>, <code>OP_1</code>, ...,  <code>OP_16</code>)后面跟着2到32字节数据. 

### 交易序列化

如果一个交易不包含隔离验证数据，那么序列化格式使用以前的格式.

如果一个交易包含隔离验证数据，一个新的序列化格式必须被使用:

<pre><code>[nVersion][marker][flag][txins][txouts][witness][nLockTime]</code></pre>

这几个字段包括<code>nVersion</code>, <code>txins</code>, <code>txouts</code>, and <code>nLockTime</code> 和以前的定义是一致的.

<code>marker</code> 字段必须是 <code>0x00</code>.

<code>flag</code> 必须是1字节的非0值. 当前必须是<code>0x01</code>.

<code>witness</code> 是一个交易的所有有witness数据的序列化.每个txin关联到一个witness字段.witness字段以var_int开始,var_int的值表示txin需要占用栈的数量，紧跟着栈的成员值，每个栈成员都以var_int开头.witness数据不是脚本也没有520字节的压栈限制.

一个非witness的txin必须关联一个空的witness字段，表示为<code>0x00</code>.如果所有的txin都没有witness程序，那么交易必须使用旧的格式序列化(例外:coinbase交易).

### Transaction ID
交易ID

每个交易有2个ID

 <code>txid</code>含义没有改变,还是交易序列化数据的2次SHA256哈希运算值 

一个新的 <code>wtxid</code> 定义为: 包含新的witness数据的序列化数据的2次SHA256哈希运算值. 如果一个交易没有任何witness数据, 那么 <code>wtxid</code> 等于 <code>txid</code>.

txid仍然表示交易的id，特别是也用来在txin中指向前一个交易的输出。

### Standard script types
### 标准脚本类型

#### Pay-to-Public-Key-Hash (P2PKH)
P2PKH是最常用的模板 <code>scriptPubKey</code> 被中本聪定义，允许简单的支付给一个单一的公钥.格式为:

<pre><code>scriptPubKey (25 bytes): OP_DUP OP_HASH160 < 20-byte-pubkey-hash > OP_EQUALVERIFY OP_CHECKSIG</code></pre>

花费P2PKH的输出,<code>scriptSig</code>格式为
<pre><code>scriptSig: < sig > < pubkey ></code></pre>

<code>RIPEMD160(SHA256(pubkey))</code> 等于<code>scriptPubKey</code> 中的 <code>20-byte-pubkey-hash</code>.

#### Pay-to-Script-Hash (P2SH)

P2SH定义在BIP16.它允许付款到任意复杂的脚本用一个修改长度的<code>scriptPubKey</code>. 格式为:

<pre><code>scriptPubKey (23 bytes): OP_HASH160 <20-byte-script-hash> OP_EQUAL</code></pre>

花费P2SH的输出,<code>scriptSig</code>格式为

<pre><code>scriptSig: <...> <...> <...> < redeemScript ></code></pre>

<code>RIPEMD160(SHA256(redeemScript))</code> 等于在<code>scriptPubKey</code>中的 <code>20-byte-script-hash</code>. <code>redeemScript</code>是反序列化并且当作剩下的数据在<code>scriptSig</code>中. 

#### Pay-to-Witness-Public-Key-Hash (P2WPKH)
P2WPKH是BIP141新定义的. 类似P2PKH, 它允许简单支付到一个公钥, 格式为:

<pre><code>scriptPubKey (22 bytes): OP_0 < 20-byte-pubkey-hash ></code></pre>

花费P2WPKH的输出,<code>scriptSig</code>必须是空的，并且<code>witness</code>为
  
<pre><code>witness: < sig > < pubkey ></code></pre>

<code>RIPEMD160(SHA256(pubkey))</code> 等于<code>scriptPubKey</code> 中的 <code>20-byte-pubkey-hash</code>.

#### P2WPKH in P2SH (P2SH-P2WPKH)
P2SH-P2WPKH 是一个使用P2WPKH 脚本作为 <code>redeemScript</code> 的 P2SH. P2SH-P2WPKH的 <code>scriptPubKey</code> 看起来和通常的P2SH是相同的:

<pre><code>scriptPubKey (23 bytes): OP_HASH160 < 20-byte-script-hash > OP_EQUAL</code></pre>

花费P2SH-P2WPKH 的输出,<code>scriptSig</code>必须只包含一个 <code>redeemScript</code>, 并且<code>witness</code> 是和P2WPKH相同的:

<pre><code>scriptSig (23 bytes): < OP_0 < 20-byte-pubkey-hash > >
witness: < sig > < pubkey ></code></pre>
  
<code>RIPEMD160(SHA256(pubkey))</code> 等于 <code>20-byte-pubkey-hash</code>, 并且<code>RIPEMD160(0x0014{20-byte-pubkey-hash})</code> 等于 <code>20-byte-script-hash</code>. 
  
#### Pay-to-Witness-Script-Hash (P2WSH)
P2SH-P2WPKH 是BIP141定义的另一个新的脚本格式, 类似P2SH. 它允许付款到任意复杂的脚本. 格式为: 

<pre><code>scriptPubKey (34 bytes): OP_0 < 32-byte-script-hash ></code></pre>

花费P2WSH的输出,<code>scriptSig</code>必须是空的，并且<code>witness</code>为

<pre><code>witness: <...> <...> <...> < witnessScript ></code></pre>

<code>RIPEMD160(SHA256(witnessScript))</code> 等于scriptPubkey中的 <code>32-byte-script-hash</code>, <code>witnessScript</code>是反序列化并且当作剩下的数据在<code>witness</code>中. 

#### P2WSH in P2SH (P2SH-P2WSH)
P2SH-P2WPKH 是使用一个P2WSH脚本作为P2SH的 <code>redeemScript</code>. P2SH-P2WPKH 的 <code>scriptPubKey</code> 看起来和P2SH是一样的: 

<pre><code>scriptPubKey (23 bytes): OP_HASH160 <20-byte-script-hash> OP_EQUAL</code></pre>

花费P2SH-P2WPKH的输出, <code>scriptSig</code>必须只包含一个 <code>redeemScript</code>, 并且<code>witness</code> 是和P2WSH是相同的: 

<pre><code>scriptSig (35 bytes): < OP_0 < 32-byte-script-hash > > \
witness: <...> <...> <...> < witnessScript ></code></pre>

<code>SHA256(witnessScript)</code> 等于 <code>32-byte-script-hash</code>, 并且 <code>RIPEMD160(0x0020{32-byte-pubkey-hash})</code> 等于 <code>20-byte-script-hash</code>.

### 新的签名算法
花费一个witness程序的输出，当产生ECDSA签名时一个新的签名算法必须被使用, 一个详细的例子能在BIP143中找到. 


### 新的支付地址
2个新的支付地址类型被定义了. 完整的规范可以在BIP142中找到.

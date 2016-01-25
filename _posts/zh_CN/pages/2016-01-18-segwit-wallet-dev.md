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

The <code>witness</code> is a serialization of all witness data of the transaction. Each txin is associated with a witness field. A witness field starts with a var_int to indicate the number of stack items for the txin. It is followed by stack items, with each item starts with a var_int to indicate the length. Witness data is NOT script and is not restricted by the 520-byte push limit.

A non-witness program txin MUST be associated with an empty witness field, represented by a <code>0x00</code>. If all txins are not witness program, a transaction MUST be serialized with the traditional format (exception: coinbase transaction).

### Transaction ID

Each transaction will have 2 IDs.

Definition of <code>txid</code> remains unchanged: the double SHA256 of the traditional serialization.

A new <code>wtxid</code> is defined: the double SHA256 of the new serialization with witness data. If a transaction is transmitted without any witness data, its <code>wtxid</code> is same as the <code>txid</code>.

The txid remains the primary identifier of a transaction. Particularly, it is used in the txin when referring to a previous output.

### Standard script types

#### Pay-to-Public-Key-Hash (P2PKH)
P2PKH is the most common <code>scriptPubKey</code> template defined by Satoshi Nakamoto, allows simple payment to a single public key. The format is:

<pre><code>scriptPubKey (25 bytes): OP_DUP OP_HASH160 < 20-byte-pubkey-hash > OP_EQUALVERIFY OP_CHECKSIG</code></pre>

To spend a P2PKH output, the <code>scriptSig</code> is

<pre><code>scriptSig: < sig > < pubkey ></code></pre>

where the <code>RIPEMD160(SHA256(pubkey))</code> is equal to the <code>20-byte-pubkey-hash</code> in scriptPubKey.

#### Pay-to-Script-Hash (P2SH)
P2SH is defined in BIP16. It allows payment to arbitrarily complex scripts with a fixed length <code>scriptPubKey</code>. The format is:

<pre><code>scriptPubKey (23 bytes): OP_HASH160 <20-byte-script-hash> OP_EQUAL</code></pre>

To spend a P2SH output, the <code>scriptSig</code> is

<pre><code>scriptSig: <...> <...> <...> < redeemScript ></code></pre>

where the <code>RIPEMD160(SHA256(redeemScript))</code> is equal to the <code>20-byte-script-hash</code> in <code>scriptPubKey</code>. The <code>redeemScript</code> is deserialized and evaluated with the remaining data in the <code>scriptSig</code>.

#### Pay-to-Witness-Public-Key-Hash (P2WPKH)
P2WPKH is a new defined in BIP141. Similar to P2PKH, it allows simple payment to a single public key. The format is:

<pre><code>scriptPubKey (22 bytes): OP_0 < 20-byte-pubkey-hash ></code></pre>

To spend a P2WPKH output, the <code>scriptSig</code> MUST be empty, and the <code>witness</code> is:
  
<pre><code>witness: < sig > < pubkey ></code></pre>

where the <code>RIPEMD160(SHA256(pubkey))</code> is equal to the <code>20-byte-pubkey-hash</code> in <code>scriptPubKey</code>.

#### P2WPKH in P2SH (P2SH-P2WPKH)
P2SH-P2WPKH is using a P2WPKH script as the <code>redeemScript</code> for P2SH. The <code>scriptPubKey</code> of P2SH-P2WPKH looks exactly the same as an ordinary P2SH:

<pre><code>scriptPubKey (23 bytes): OP_HASH160 < 20-byte-script-hash > OP_EQUAL</code></pre>

To spend a P2SH-P2WPKH output, the <code>scriptSig</code> MUST contain a push of the <code>redeemScript</code> and nothing else and the <code>witness</code> is same as P2WPKH:

<pre><code>scriptSig (23 bytes): < OP_0 < 20-byte-pubkey-hash > >
witness: < sig > < pubkey ></code></pre>
  
where the <code>RIPEMD160(SHA256(pubkey))</code> is equal to the <code>20-byte-pubkey-hash</code>, and <code>RIPEMD160(0x0014{20-byte-pubkey-hash})</code> is equal to the <code>20-byte-script-hash</code>.
  
#### Pay-to-Witness-Script-Hash (P2WSH)
P2WSH is another new standard script defined in BIP141. Similar to P2SH, it allows payment to arbitrarily complex scripts. The format is:

<pre><code>scriptPubKey (34 bytes): OP_0 < 32-byte-script-hash ></code></pre>

To spend a P2WSH output, the <code>scriptSig</code> MUST be empty, and the witness is:

<pre><code>witness: <...> <...> <...> < witnessScript ></code></pre>

where the <code>RIPEMD160(SHA256(witnessScript))</code> is equal to the <code>32-byte-script-hash</code> in scriptPubKey. The <code>witnessScript</code> is deserialized and evaluated with the remaining data in the <code>witness</code>.


#### P2WSH in P2SH (P2SH-P2WSH)
P2SH-P2WPKH is using a P2WSH script as the <code>redeemScript</code> for P2SH. The <code>scriptPubKey</code> of P2SH-P2WPKH looks exactly the same as an ordinary P2SH:

<pre><code>scriptPubKey (23 bytes): OP_HASH160 <20-byte-script-hash> OP_EQUAL</code></pre>

To spend a P2SH-P2WPKH output, the <code>scriptSig</code> MUST contain a push of the <code>redeemScript</code> and nothing else and the <code>witness</code> is same as P2WSH:

<pre><code>scriptSig (35 bytes): < OP_0 < 32-byte-script-hash > > \
witness: <...> <...> <...> < witnessScript ></code></pre>

where the <code>SHA256(witnessScript)</code> is equal to the <code>32-byte-script-hash</code>, and <code>RIPEMD160(0x0020{32-byte-pubkey-hash})</code> is equal to the <code>20-byte-script-hash</code>.

### New signing algorithm
To spend a witness program output, a new signing algorithm MUST be used when producing the ECDSA signature. A step-by-step example could be found in BIP143.

### New payment address
Two new payment address types are defined. The full specification with example could be found in BIP142.

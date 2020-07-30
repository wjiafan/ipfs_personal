# IPFS - Content Addressed, Versioned, P2P File System(DRAFT 3)
Juan Benet
juan@benet.ai

## 摘要ABSTRACT

> IPFS（InterPlanetary FileSystem，星际文件系统）是点对点分布式系统，追求连接所有相同文件系统的计算设备。在某些方面，IPFS类似于Web，但是IPFS可以被认作是一个单一的BitTorrent Swarm，使用Git仓库进行交换对象。换句话说，IPFS使用内容寻址超链接，提供了高效的基于内容的地址区块存储模型。这形成了一个通用的MerkleDAG，它可以构建版本文件系统、区块链、甚至一个永恒的Web的数据结构。IPFS由一个分布式哈希表（a distributed hashtable，DHT）,一个去中心化的区块交换，一个自证明命名空间（a self-certifying namespace）。IPFS没有单点故障，其中的节点也不需要互相信任。


## 1. 介绍INTRODUCTION
> 在构建全球分布式文件系统上已经进行过一些尝试。一些系统相当成功，而另外一些则完全失败了。在这些学术理论尝试中，AFS获得了广泛的成功，直到目前还在使用。另外的一些，则没有获得一样的成功。在理论研究之外，大多数成功的系统是适用于大媒体数据（音频和视频）的P2P文件共享系统的应用。最著名的有，Napster，KaZaA和BitTorrent，其中BitTorrent部署了大量文件分布式系统，有超过1亿的并发用户。即使今天，BitTorrent维持了大量的部署，每天有超过1000万的节点数。这些应用比理论文件系统有更多的用户和文件发布。尽管如此，这些应用并没有设计成一个基础设施，以此为基础建立扩展。然而，也有一些成功的改变，非通用的文件系统已经出现，提供了全球化、低延时、去中心发布。

> 可能对于大多数已存在的用户场景Http已经是一个足够好的系统。到目前为止，HTTP是有史以来最成功的“文件的分布式系统”。与浏览器相结合，HTTP产生了巨大的技术和社会影响。它已成为通过互联网传输文件的事实上的方式。然而，它未能利用过去十五年发明的数十种出色的文件分发技术。从某个视角，演进Web基础设施几乎是不可能的，因为涉及大量向后兼容性限制和大量团体在现有模型上的投入。但从另一个角度来看，自HTTP出现以来，新的协议已经出现并得到了广泛的应用。缺少的是升级设计：增强当前的HTTP Web，并在不降低用户体验的情况下引入新功能。

> 业界已经开始使用HTTP了很长时间，因为移动小文件相对便宜，即使对于拥有大量流量的小型组织也是如此。但我们正在进入一个新的数据分发时代，面临着新的挑战：（a）托管和分发PB级数据集，（b）跨组织计算大数据，（c）高容量高清晰度按需或实时媒体流，（d）大规模数据集的版本化和链接，（e）防止重要文件的意外消失等。其中许多可归结为大量数据，可随处访问。“受关键功能和带宽问题的压力，我们已经使用不同的数据分发协议放弃了HTTP。下一步是将它们作为Web本身的一部分。

> 正交于高效的数据分发，版本控制系统已设法开发重要的数据协作工作流程。Git是分布式源代码版本控制系统，它开发了许多有用的方法来建模和实现分布式数据操作。Git工具链提供了多种版本控制功能，大型文件分发系统严重缺乏。受Git启发的新解决方案正在出现，例如Camlistore[？]，一个个人文件存储系统，以及Dat [？]一个数据协作工具链和数据集包管理器。Git已经影响了分布式文件系统设计[9]，因为其内容为Merkle DAG数据模型提供了强大的文件分发策略。还有待探索的是，这种数据结构如何影响面向高吞吐量的文件系统的设计，以及它如何升级Web本身。

> 本文介绍了IPFS，一种新颖的P2P版本控制文件系统，旨在协调这些问题。IPFS综合了许多过去成功系统的学习经验。仔细的以界面为中心的集成产生的系统大于其各部分的总和。IPFS的核心原则是将所有数据建模为同一Merkle DAG的一部分。

> 这篇文章介绍了IPFS，一个P2P版本控制文件系统的新想法力图解决这些问题。IPFS综合学习了过去的一些成功的系统。基于接口紧密的集成产生的系统比各个部分的简单相加更强大。IPFFS的中心原则是把所有数据改造成同一个Merkle DAG的一部分。

## 2. 背景 BACKGROUND
> 这章节回顾了构成IPFS的成功P2P系统的重要特性。

### 2.1 分布式哈希表 Distribution Hash Tables，DHT
> DHTs广泛适用于P2P系统的元数据的调整及维护。例如，BitTorrent MainlineDHT记录了torrent swarm的节点集

#### 2.1.1 Kademlia DHT
> Kademlia是受欢迎的DHT，提供了：
1.在大网络中高效查询：查询节点的平均效率log2(n).（例如，对于1000万节点的网络只需要20跳）
2.低协调开销：优化了发送给其他节点的控制信息的数量
3.通过优选长期存在的节点来阻止大量攻击
4.广泛应用于P2P应用中，包括Guntella和BitTorrent，超过2000万节点的组网。

#### 2.1.2 Coral DSHT
> 虽然有些P2P文件系统直接在DHTs中存储数据块，这个"浪费存储和宽带，因为原本并不需要的数据必须被存储在节点上"。Coral DSHT通过三种特别重要的方式扩展了Kademlia：
1.Kademlia根据离key“最近”距离（使用XOR-distance）的节点ids中存储数据，这没有考虑应用数据的位置，忽略了拥有这些数据的“远”节点，同时不管他们是否需要而强制“最近”节点来存储。这浪费了有效存储和宽带。相反，Coral存储能存储这些数据块的节点的地址。
2.Coral将DHT API从get_value(key)改成get_any_values(key)(DSHT中的“sloppy”)。Coral用户仅需要一个正常运行的节点，而不需要整个列表。作为回报，Coral分发值的子集到“nearest”节点，避免热点（当一个键值受欢迎时，使周边所有nearest节点超负载）
3.还有，Coral依赖于区域和大小组成DSHTs的各个层级（称为clusters）。这使节点优先在自己的区域内查询节点，“查找临近的数据，而不是查找距离节点”，这极大减少查找的延时。

#### 2.1.3 S/Kademlia DHT
> S/Kademlia为了阻止恶意攻击扩展了Kademlia，在两种特别重要的方面：
1.S/Kademlia提供了保护NodeID生成及阻止女巫攻击的方案。要求节点产生PKI键值对，从中获得标识符，并且相互签名。其中的一个方案POW提高了女巫攻击的成本。
2.S/Kademlia节点通过独立的路径查询值，就是为了确保诚实节点能够在网络中存在大量恶意节点时能够互相连接。即使在一半恶意节点时，S/Kademlia也可以获得85%的成功率。

### 2.2 块交换Block Exchanges-BitTorrent
> BitTorrent是一个相当成功的P2P文件共享系统，成功的协调不信任节点（swarms）之间的网络，互相合作分发文件片段。从BitTorrent及其生态系统的关键特性中，ipfs得到的启发包括:
1. BitTorrent的数据交换协议采用类似于tit-for-tat策略，奖励相互贡献的节点，惩罚那些只获取他人的节点的资源。
2. BitTorrent节点跟踪文件的可用性，优先发送最稀有的碎片。 这会减轻种子负担，使非种子节点能够相互交易。
3. BitTorrent的标准tit-for-tat相对容易受到一些强占带宽共享策略的影响。PropShare是一种不同的对等带宽分配策略，可以更好地抵制强占策略，并提高群体的性能。

### 2.3 版本控制系统 Version Control Systems-Git
> 版本控制系统提供了模型文件的工具，可以随时间变化并有效地分发不同的版本。流行的版本控制系统Git提供了一个强大的Merkle DAG(Merkle Directed Acyclic Graph- 类似于但比MerkleTree更通用的结构。重复数据删除，不需要平衡，非叶节点包含数据。)对象模型，以分布式友好的方式捕获文件系统树的变化。
1.不可变对象表示文件（blob），目录（tree）和更改（commit）。
2.对象通过其内容的hast加密方式进行内容寻址。
3.链接其他嵌入的对象，形成Merkle DAG。 这提供了许多有用的完整性和工作流属性
4.大多数版本控制元数据（branches，tags等）只是指针引用，因此创建和更新成本低廉。
5.版本更改仅更新引用或添加对象。
6.将版本更改分发给其他用户只是传输对象和更新远程引用。

### 2.4 自证明文件系统 Self-Certified Filesystems-SFS
> SFS提出了两个的引人注目的实现：（a）分布式信任链和（b）平等共享全局命名空间。SFS引入了一种构建SelfCertified Filesystems的技术：使用以下方案寻址远程文件系统
        /sfs/<Location>:<HostID>
这里的Location是服务器网络地址，同时
        HostID = hash(public_key || Location)
因此，SFS文件系统的名称证明了他的服务器。用户可以验证服务器提供的公钥，协商共享密钥并保护所有流量。所有SFS实例共享一个全局命名空间，其中名称分配是加密的，而不是任何集中式主体的把控。

## 3. IPFS设计
> IPFS是一个分布式文件系统，它综合了以前的对等系统的成功思想，包括DHT，BitTorrent，Git和SFS。IPFS的贡献在于将经过验证的技术简化，发展和整合到一个整体的系统中，大于其各部分的总和。IPFS提供了一个用于编写和部署应用程序的新平台，以及一个用于分发和版本化大数据的新系统。 IPFS甚至可以改进网络本身。 

> IPFS是点对点的; 没有节点是特权。 IPFS节点将IPFS对象存储在本地存储中。节点间相互连接和转移对象。 这些对象可以表示为文件和其他数据结构。 IPFS协议分为一堆不同功能的子协议：

> 
1.身份 - 管理节点身份生成和验证。见3.1节。
2.网络 - 管理与其他节点的连接，使用各种底层网络协议。可配置。见3.2节。
3.路由 - 维护信息以定位特定的节点和对象。响应本地和远程查询。默认为DHT，可替换。见3.3节。
4.交换 - 一种新的块交换协议（BitSwap），用于管理有效的块分发。模仿市场，弱激励数据复制。交易策略可替换。在3.4节中描述。
5.对象 - 带有链接的内容寻址不可变对象的MerkleDAG。用于表示任意数据结构，例如文件层次结构和通信系统。在3.5节中描述。
6.文件 - 受Git启发的版本化文件系统层次结构。见3.6节。
7.命名 - 自我认证的可变名称系统。在3.7节中描述。
这些子系统不是独立的;它们是集成的并融合的。尽管如此，从下到上构建协议栈，分别描述它们是有益的。
注意：下面的数据结构和函数在Go语法中指定。

### 3.1身份 Identities
> 节点通过NodeId来标识，NodeId是公钥的加密哈希，使用S/Kademlia的静态加密拼图创建。 节点存储其公钥和私钥（使用密码加密）。用户可以在每次发布时自由地设置“新”节点标识，但是会丢失累积的网络收益。 被激励节点保持不变。

```
type NodeId Multihash
type Multihash []byte
// self-describing cryptographic hash digest
type PublicKey []byte
type PrivateKey []byte
// self-describing keys
type Node struct {
NodeId NodeID
PubKey PublicKey
PriKey PrivateKey
}
```

基于S/Kademlia方式产生的IPFS身份
```
difficulty = <integer parameter>
n = Node{}
do {
n.PubKey, n.PrivKey = PKI.genKeyPair()
n.NodeId = hash(n.PubKey)
p = count_preceding_zero_bits(hash(n.NodeId))
} while (p < difficulty)
```

> 第一次连接时，节点相互交换公钥，并且校验：hash(other.PublicKey)==other.NodeId.如果不是，连接终止。
关于加密函数的注意事项：
IPFS不是将系统锁定到一组特定的函数选择，而是支持自描述值。散列摘要值以multihash格式存储，其中包括指定所使用的散列函数的短标头和以字节为单位的摘要长度。 例如：

```
<function code><digest length><digest bytes>
```
> 这允许系统（a）为用例选择最佳功能（例如，更强的安全性和更快的性能），以及（b）随着功能选择的改变而发展。 自描述值允许兼容地使用不同的参数选择。

### 3.2  Network
> 
IPFS节点与网络中的数百个其他节点定期通信，可能跨越广泛的互联网。 IPFS网络堆栈功能：
•传输：IPFS可以使用任何传输协议，最适合WebRTC数据通道[？]（用于浏览器连接）或uTP（LEDBAT [14]）。
•可靠性：如果底层网络不提供IPFS，IPFS可以使用uTP（LEDBAT [14]）或SCTP [15]提供可靠性。
•连接性：IPFS还使用ICE NAT遍历技术[13]。
•完整性：可选择使用哈希校验和检查消息的完整性。
•真实性：可选择使用带有发件人公钥的HMAC检查邮件的真实性。

#### 3.2.1 节点寻址注意事项
> 
IPFS可以使用任何网络; 它不依赖或假设访问IP。这允许IPFS用于覆盖网络。IPFS将地址存储为multiaddr的字节字符串，供底层网络使用。 multiaddr提供了一种表达地址及其协议的方法，包括对封装的支持。 例如：
```
# an SCTP/IPv4 connection
/ip4/10.20.30.40/sctp/1234/
# an SCTP/IPv4 connection proxied over TCP/IPv4
/ip4/5.6.7.8/tcp/5678/ip4/1.2.3.4/sctp/1234/
```

### 3.3 Routing
> 
IPFS节点需要路由系统，该路由系统可以找到（a）其他Peer的网络地址，以及（b）可以为特定对象提供服务的Peer。 IPFS使用基于S/Kademlia和Coral的DSHT，2.1中讨论了这个。对象的大小和IPFS的使用模式类似于Coral和Mainline 因此IPFS DHT根据它们的大小对存储的值进行区分。对于值小（等于或小于1KB）的情况直接存储在DHT上。对于更大的值，DHT存储引用，它们是可以提供块服务的对等体的NodeId。
此DSHT的接口如下：

```
type IPFSRouting interface {
FindPeer(node NodeId)
// gets a particular peer’s network address
SetValue(key []bytes, value []bytes)
// stores a small metadata value in DHT
GetValue(key []bytes)
// retrieves small metadata value from DHT
ProvideValue(key Multihash)
// announces this node can serve a large value
FindValuePeers(key Multihash, min int)
// gets a number of peers serving a large value
}
```

> 注意：不同的用例将要求实质上不同的路由系统（例如，宽网络中的DHT，本地网络中的静态HT）。 因此，可以交换IPFS路由系统以满足用户的需求。 只要满足上面的接口，系统的其余部分将继续运行。

### 3.4 区块交换-BitSwap Protocol
> 
在IPFS中，通过使用BitTorrent启发的协议：BitSwap，与peers交换块来进行数据分发。与BitTorrent一样，BitSwap节点正在寻求获得一组块（want_list），并在交换中提供另一组块（have_list）。与BitTorrent不同，BitSwap不仅限于一个torrent中的块。BitSwap作为一个持久的市场运行，节点可以获取所需的块，无论这些块是哪些文件的一部分。 这些块可能来自文件系统中完全不相关的文件。 节点聚集在一起交易市场。

> 
虽然交易系统的概念意味着可以创建虚拟货币，但这需要全球分类账来跟踪货币的所有权和转移。这可以作为BitSwap策略实现，并将在未来的论文中进行探讨。
在基本情况下，BitSwap节点必须以块的形式相互提供直接值。当跨节点的块分布是互补的时，也就是说它们具有对方想要的内容，这非常有效。通常情况并非如此。在某些情况下，节点必须适用于其块。如果一个节点没有其peer想要的东西（或根本没有），它会寻找其peer想要的部分，其优先级低于节点自己想要的优先级。这激励节点缓存和传播稀有部分，即使他们不直接对它们感兴趣。

#### 3.4.1 BitSwap Credit
> 
协议还必须激励节点在不需要任何特定内容时做种子，因为它们可能有其他节点所需的块。因此，BitSwap节点积极地向其Peer发送块，期望偿还债务。 但必须防范水蛭（永不共享的免费加载节点）。 一个简单的信用类系统解决了这个问题：
1.Peers跟踪其他节点的收支（以字节验证）。
2.根据债务增加而下降的函数，peer在概率上向债务peer发送区块。
请注意，如果节点决定不发送给对等方，则该节点随后会忽略该Peer以获取ignore_cooldown超时。这可以防止发件人通过引起更多的骰子来尝试游戏概率。 （默认BitSwap为10秒）。

#### 3.4.2 BitSwap Strategy
> 
BitSwap Peers可能采用的不同策略会对整个交易所的表现产生了截然不同的影响。在BitTorrent中，虽然指定了标准策略（tit-for-tat），但也已经实施了各种其他策略，从BitTyrant（共享最不可能），到BitThief（利用漏洞，永不分享），到PropShare（按比例分享）。 BitSwap Peers可以类似地实施一系列策略（良好和恶意）。 那么，功能的选择应该旨在：
1.最大化节点和整个交易所的交易表现
2.阻止贪图者利用和降低交换
3.对其他未知策略有效并具有抵抗力
4.对受信任的peers宽容
对这些战略空间的探索是未来的工作。 在实践中起作用的一个功能选择是一个sigmoid，由债务比例缩放：
让节点与其对等体之间的负债比率为：

```
r = bytes_sent/(bytes_recv + 1)

//给定r，让发送给债务人的概率为：
P(send|r) = 1 − 1/(1 + exp1(6 − 3r))
```

> 正如您在图1中所看到的，当节点的负债率超过既定信用额度的两倍时，此功能会迅速下降。
debt ratio是衡量信任度的指标：对先前已成功交换大量数据的节点之间的债务宽容，对未知的不可信节点无情。 这（a）提供了对创建大量新节点（sybill攻击）的攻击者的抵抗，（b）保护以前成功的交换关系，即使其中一个节点暂时无法提供价值，以及（c）最终关闭已经恶化关系，直到他们改善。


#### 3.4.3 BitSwap Ledger
> 
BitSwap节点使分类账记录与其他节点的转账。这允许节点跟踪历史记录并避免篡改。激活连接时，BitSwap节点会交换其分类帐信息。 如果它不完全匹配，则从破坏或丢失的信用或债务开始重新初始化分类帐。恶意节点有可能故意丢失分类账本，从而达到消除债务的目的。节点不太可能产生足够的债务以保证也会失去应计信任;但是合作伙伴节点可以自由地将其视为不当行为，并且拒绝交易。

```
type Ledger struct {
owner NodeId
partner NodeId
bytes_sent int
bytes_recv int
timestamp Timestamp
}
```
节点可以自由保留分类帐历史记录，但不是必须的正确操作。只有当前分类帐条目才有用。节点也可以根据需要随意收集分类账，从较不实用的分类账开始：旧的（peers可能不再存在）和小。

#### 3.4.4 BitSwap Specification
> 
BitSwap节点遵循一个简单协议

```
// Additional state kept
type BitSwap struct {
ledgers map[NodeId]Ledger
// Ledgers known to this node, inc inactive
active map[NodeId]Peer
// currently open connections to other nodes
need_list []Multihash
// checksums of blocks this node needs
have_list []Multihash
// checksums of blocks this node has
}
type Peer struct {
nodeid NodeId
ledger Ledger
// Ledger between the node and this peer
last_seen Timestamp
// timestamp of last received message
want_list []Multihash
// checksums of all blocks wanted by peer
// includes blocks wanted by peer’s peers
}
// Protocol interface:
interface Peer {
open (nodeid :NodeId, ledger :Ledger);
send_want_list (want_list :WantList);
send_block (block :Block) -> (complete :Bool);
close (final :Bool);
}
```
> 
节点连接的全生命周期框架：
1.Open: 达成一致后，Peers发送Ledgers
2.Sending：peers 交换want_lists 和Blocks
3.Close：Peers关闭连接
4.Ignored：（特别的）如果一个节点的策略是避免发送，该Peer被忽略（在超时周期内）

```
Peer.open(NodeId, Ledger)
```

> 当连接时，节点使用Ledger初始化连接，该Ledger可以是从过去的连接存储的，也可以是新的连接。 然后，将带有Ledger的Open消息发送给Peer。
收到Open消息后，Peer选择是否激活连接。如果根据接收方的Ledger，发送方不是可信代理（传输低于零，或大量未偿还债务），接收方可以选择忽略该请求。这应该通过ignore_cooldown超时以概率方式完成，以便纠正错误并阻止攻击者。 如果激活连接，接收器将使用本地版本的Ledger初始化Peer对象并设置last_seen时间戳。然后，它将收到的Ledger与自己的Ledger进行比较。如果它们完全匹配，则连接已打开。如果它们不匹配，则对等方创建一个新的归零分类帐并发送它。

```
Peer.send_want_list(WantList)
```

> 当连接打开时，节点广播他们的want_list到所有连接的peers。完成这部分工作必须满足（a)打开连接，（b）在一个随机周期超时后，（c)在want_list发生改变，以及（d)在接收到一个新区块后。
在接收到want_list后，节点做好存储。然后校验是否存在想要的区块。如果有，依据BitSwap Strategy发送出去。


```
Peer.send_block(Block)
```
> 发送块很简单。 节点只是传输数据块。在接收到所有数据后，接收器计算Multihash校验和以验证它是否与预期数据匹配，并返回确认。在完成正确传输块之后，接收器将块从need_list移动到have_list，并且接收方和发送方都更新其分类账以反映传输的附加字节。 如果传输验证失败，则发送方发生故障或攻击接收方。接收方可以自由拒绝进一步的交易。请注意， BitSwap期望在可靠的传输通道上运行，因此传输错误--这可能会导致对诚实发件人的错误处罚--预计会在将数据提供给BitSwap之前被捕获.

```
Peer.close(Bool)
```
> 
close的最后参数表明拆除连接的意图是否是发送者。如果为false，接收方可以选择立即重新打开连接。这避免了过早关闭。应在两种情况下关闭对等连接：
•silence_wait超时已过期，但未收到来自对等方的任何消息（默认BitSwap使用30秒）。该节点发出Peer.close（false）。
•节点正在退出，BitSwap正在关闭。在这种情况下，节点发出Peer.close（true）。

> 收到消息后，收件人和发件人都会拆除连接，清除存储的任何状态。如果有用的话，可以存储Ledger以供将来使用。

> 注意事项：
•应忽略非活动连接上的Non-open消息。在send_block消息的情况下，接收器可以检查块以查看是否所需并且正确，如果是这样，请使用它。无论如何，所有这些无序消息都会触发来自接收器的close(flase)消息以强制重新初始化连接。


### 3.5 Object Merkle DAG
> DHT和BitSwap允许IPFS形成一个庞大的P2P系统，用于快速、稳健地存储和分发区块。 除此之外，IPFS还构建了一个MerkleDAG，一个有向无环图，其中对象之间的链接是嵌入源中的目标的加密哈希。这是Git数据结构的概括。 MerkleDAGs为IPFS提供了许多有用的属性，包括：
1.内容寻址：所有内容由其多哈校验的唯一标识，包括链接。
2.防篡改：所有内容都通过校验和进行验证。 如果数据被篡改或损坏，IPFS会检测到它。
3.重复数据删除：包含完全相同内容的所有对象都相同，并且只存储一次。这对于索引对象（例如gittrees和commits）或数据的公共部分特别有用。
IPFS对象格式是：

```
type IPFSLink struct {
Name string
// name or alias of this link
Hash Multihash
// cryptographic hash of target
Size int
// total size of target
}
type IPFSObject struct {
links []IPFSLink
// array of links
data []byte
// opaque content data
}
```

> IPFS Merkle DAG是一种非常灵活的数据存储方式。唯一的要求是对象引用是（a）内容寻址，（b）以上述格式编码。 IPFS授予应用程序对数据字段的完全控制权;应用程序可以使用他们选择的任何自定义数据格式，IPFS可能无法理解。单独的对象内链接表允许IPFS：
·在对象中列出所有对象引用，例如：

```
> ipfs ls /XLZ1625Jjn7SubMDgEyeaynFuR84ginqvzb
XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x 189458 less
XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5 19441 script
XLF4hwVHsVuZ78FZK6fozf8Jj9WEURMbCX4 5286 template

<object multihash> <object size> <link name>
```
> 
·解析查找字符串路径，例如foo/bar/baz。给定一个对象，IPFS将第一个路径组件解析为对象链接表中的哈希，获取第二个对象，并使用下一个组件重复上述操作。因此，无论数据格式是什么，字符串路径都可以遍历Merkle DAG。
·解析递归引用的所有对象：

```
> ipfs refs --recursive \
/XLZ1625Jjn7SubMDgEyeaynFuR84ginqvzb
XLLxhdgJcXzLbtsLRL1twCHA2NrURp4H38s
XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x
XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5
XLWVQDqxo9Km9zLyquoC9gAP8CL1gWnHZ7z
...
```

> 
原始数据字段和公共链接结构是在IPFS之上构造任意数据结构的必要组件。虽然很容易看出Git对象模型如何适合这个DAG，但请考虑以下其他潜在的数据结构：（a）键值存储（b）传统关系数据库（c）Linked Data triple storesLinked Data triple stores（d）链接文档发布系统linked document publishing systems（e）链接通信平台（f）加密货币区块链。 这些都可以在IPFS Merkle DAG之上建模，它允许任何这些系统使用IPFS作为更复杂应用程序的传输协议。

#### 3.5.1 Paths
> 可以使用字符串路径API遍历IPFS对象。路径可以像在传统UNIX文件系统和Web中一样工作。 Merkle DAG链接使其易于遍历。 请注意，IPFS中的完整路径具有以下形式：

```
# format
/ipfs/<hash-of-object>/<name-path-to-object>
# example
/ipfs/XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x/foo.txt
```
> /ipfs前缀允许在标准安装点安装到现有系统而不会发生冲突（安装点名称当然是可配置的）。 第二个路径组件（首先在IPFS中）是对象的哈希。情况总是如此，因为没有全局根。根对象有个不可能的任务：将处理分布式（并且可能是断开连接的）环境中的数百万个对象的一致性的。相反，我们使用内容寻址来模拟根。 所有对象始终可以通过哈希访问。 注意这意味着在路径<foo>/bar/baz中给出三个对象，所有人都可以访问最后一个对象：

```
/ipfs/<hash-of-foo>/bar/baz
/ipfs/<hash-of-bar>/baz
/ipfs/<hash-of-baz>
```

#### 3.5.2 本地对象Local Objects
> 
IPFS客户端需要一些本地存储和一个外部系统，用于存储和检索IPFS管理的对象的本地原始数据。存储类型取决于节点的用例。在大多数情况下，这只是磁盘空间的一部分（由本机文件系统管理，由kv存储系统，如leveldb，或直接由IPFS客户端管理）。在其他情况下，例如非持久性缓存，此存储只是RAM的一部分。
最终，IPFS中可用的所有块都在某个节点的本地存储中。当用户请求对象时，至少可以临时找到，下载和存储它们。 这为此后的一些可配置时间提供了快速查找。

#### 3.5.3 对象固定 Object Pinning
> 
希望确保特定对象生存的节点可以通过固定对象来实现。 这可确保对象保留在节点的本地存储中。 固定可以递归完成，也可以固定所有链接的后代对象。 然后指向的所有对象都存储在本地。 这对于保存文件（包括引用）特别有用。这也使IPFS成为一个永久链接的Web，而对象可以确保他们指向的其他对象是存在的。

#### 3.5.4 发布对象 Publishing Objects
> 
IPFS是全球分布的。 它旨在允许数百万用户的文件一起共存。具有内容哈希寻址的DHT允许以公平，安全和完全分布的方式发布对象。 任何人都可以通过简单地将其键添加到DHT，把自己加入到Peers，并向其他用户提供对象的路径来发布对象。 请注意，对象本质上是不可变的，就像在Git中一样。新版本的哈希值不同，因此是新对象。 跟踪版本是其他版本控制对象的工作。

#### 3.5.5 Object-level Cryptography
> IPFS可以处理对象级加密操作。加密或签名的对象包装在一个特殊的帧中，允许加密或验证原始字节。

```
type EncryptedObject struct {
Object []bytes
// raw object data encrypted
Tag []bytes
// optional tag for encryption groups
}
type SignedObject struct {
Object []bytes
// raw object data signed
Signature []bytes
// hmac signature
PublicKey []multihash
// multihash identifying key
}
```
> 
加密操作会更改对象的哈希值，从而定义不同的对象。IPFS自动验证签名，并可以使用用户指定的密钥链解密数据。 加密对象的链接也受到保护，没有解密密钥就无法进行遍历。可以在一个密钥下加密父对象，子对象在另一个密钥下加密或不加密。这可以确保链接共享对象的安全。

### 3.6 Files
> 
IPFS还定义了一组对象，用于在MerkleDAG之上对版本化文件系统进行建模。这个对象模型类似于Git：
1. block：可变大小的数据块。
2. list：块或其他列表的集合。
3. tree：区块，列表或其他树的集合。
4. commit：某棵树的版本历史中的快照。
我希望完全使用Git对象格式，但不得不离开以介绍在分布式文件系统中有用的某些功能，即（a）快速查找大小（对象中添加了聚合字节大小），（b）大文件重复数据删除（添加列表对象），以及（c）将commits嵌入到树中。尽管如此，IPFS文件对象与Git足够相似，可以实现两者之间的转换。此外，可以引入转换一组Git对象而不会丢失任何信息（unix文件权限等）。
注意：下面的文件对象格式使用JSON。请注意，虽然ipfs包含导入/导出到JSON，但此结构实际上是使用protobufs进行二进制编码，

#### 3.6.1 File Object: blob
> 
blob对象包含可寻址的数据单元，并使用文件表示。 IPFS块类似于Git blob或文件系统数据块。它们存储用户的数据。 请注意，IPFS文件可以由lists和blobs表示。 Blob没有链接。

```
{
"data": "some data here",
// blobs have no links
}
```

#### 3.6.2 File Object: list
> 
list对象表示由多个IPFS blob组成的大型或去重文件。 列表包含有序的blob或列表对象序列。
从某种意义上说，IPFS列表的功能类似于带有间接块的文件系统文件。由于列表可以包含其他列表，因此可以使用包括链接列表和平衡树的拓扑。同一节点出现在多个位置的定向图允许文件内部去重。 当然，循环是不可能的，因为哈希寻址强制执行。

```
{
"data": ["blob", "list", "blob"],
// lists have an array of object types as data
"links": [
{ "hash": "XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x",
"size": 189458 },
{ "hash": "XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5",
"size": 19441 },
{ "hash": "XLWVQDqxo9Km9zLyquoC9gAP8CL1gWnHZ7z",
"size": 5286 }
// lists have no names in links
]
}
```
#### 3.6.3 File Object: tree
> 
IPFS中的tree对象类似于Git中的tree对象：它表示一个目录，一个哈希名称的映射。哈希引用blobs，lists，其他trees或commits。 请注意，Merkle DAG已经实现了传统路径命名。

```
{
"data": ["blob", "list", "blob"],
// trees have an array of object types as data
"links": [
{ "hash": "XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x",
"name": "less", "size": 189458 },
{ "hash": "XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5",
"name": "script", "size": 19441 },
{ "hash": "XLWVQDqxo9Km9zLyquoC9gAP8CL1gWnHZ7z",
"name": "template", "size": 5286 }
// trees do have names
]
}
```

#### 3.6.4 File Object: commit
> 
IPFS中的commit对象表示任何对象的版本历史的快照。 它类似于Git，但可以指向任何类型的对象。 它还链接到作者对象。

```
{
"data": {
"type": "tree",
"date": "2014-09-20 12:44:06Z",
"message": "This is a commit message."
},
"links": [
{ "hash": "XLa1qMBKiSEEDhojb9FFZ4tEvLf7FEQdhdU",
"name": "parent", "size": 25309 },
{ "hash": "XLGw74KAy9junbh28x7ccWov9inu1Vo7pnX",
"name": "object", "size": 5198 },
{ "hash": "XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm",
"name": "author", "size": 109 }
]
}
```

#### 3.6.5 Version control
> 
commit对象表示一个对象的版本历史的特定快照。两个不同commit的对象（和子对象）的对比，透露了文件系统的两个版本之间的差异。只要单个commit和它引用的所有子对象可以访问，就可以检索所有先前版本，并且可以浏览文件系统变更的完整历史记录。这不属于MerkleDAG对象模型。IPFS用户可以使用Git版本控制工具的全部功能。 对象模型兼容，但不相同。可以（a）构建一个修改过的Git工具版本以适用于IPFS对象图，（b）构建一个挂载的FUSE文件系统，将IPFS树挂载为Git仓库，将Git文件系统读/写转换为IPFS格式。

#### 3.6.6 Filesystem Paths
> 
正如我们在Merkle DAG部分中看到的那样，可以使用字符串路径API遍历IPFS对象。IPFS文件对象的设计使IPFS挂载在UNIX文件系统上更简单。为了将tree表示为目录，tree下没有数据。并且commits可以表示为目录，也可以完全隐藏在文件系统中。

#### 3.6.7 Splitting Files into Lists and Blob
> 
版本控制和分发大型文件的主要挑战之一是找到将它们拆分为独立块的正确方法。IPFS提供以下备选方案，而不是假设它可以适用每一种类型的文件：
（a）在LBFS中使用Rabin指纹来选择合适的块边界。
（b）使用rsync滚动校验和算法来检测版本之间已更改的块。
（c）允许用户针对特定文件自定义块分割函数。

#### 3.6.8 Path Lookup Performance
> 
基于路径访问来遍历对象图。检索某个对象需要先在DHT中查找key，连接到peer及诶单并检索其块。 这是相当大的开销，特别是在查找包含许多组件的路径时。 这可以通过以下方式减轻：
•tree caching：由于所有对象都是散列寻址的，因此可以无限期地缓存它们。此外，tree往往很小，因此IPFS优先考虑将它们缓存在blob上。
•flattened tree：对于任何给定的tree，可以构造一个特殊的flattenedTree来列出从树可到达的所有对象。 flattened tree中的命名实际上是从原始树分开的带有斜杠的。

> Figure 2-Sample Object Graph
![Figure 2-Sample Object Graph](https://note.youdao.com/yws/api/personal/file/A850057732594BA78DD57184F7E71E2F?method=download&shareKey=e3f138fc44fa03dc28a5f107f6b9a4bf)


```
Figure 3: Sample Objects

> ipfs file-cat <ccc111-hash> --json
{
"data": {
"type": "tree",
"date": "2014-09-20 12:44:06Z",
"message": "This is a commit message."
},
"links": [
{ "hash": "<ccc000-hash>",
"name": "parent", "size": 25309 },
{ "hash": "<ttt111-hash>",
"name": "object", "size": 5198 },
{ "hash": "<aaa111-hash>",
"name": "author", "size": 109 }
]
}
> ipfs file-cat <ttt111-hash> --json
{
"data": ["tree", "tree", "blob"],
"links": [
{ "hash": "<ttt222-hash>",
"name": "ttt222-name", "size": 1234 },
{ "hash": "<ttt333-hash>",
"name": "ttt333-name", "size": 3456 },
{ "hash": "<bbb222-hash>",
"name": "bbb222-name", "size": 22 }
]
}
> ipfs file-cat <bbb222-hash> --json
{
"data": "blob222 data",
"links": []
}
```

> 
例如，上面的ttt111用flattened tree可以表示为：

```
{
"data":
["tree", "blob", "tree", "list", "blob" "blob"],
"links": [
{ "hash": "<ttt222-hash>", "size": 1234
"name": "ttt222-name" },
{ "hash": "<bbb111-hash>", "size": 123,
"name": "ttt222-name/bbb111-name" },
{ "hash": "<ttt333-hash>", "size": 3456,
"name": "ttt333-name" },
{ "hash": "<lll111-hash>", "size": 587,
"name": "ttt333-name/lll111-name"},
{ "hash": "<bbb222-hash>", "size": 22,
"name": "ttt333-name/lll111-name/bbb222-name" },
{ "hash": "<bbb222-hash>", "size": 22
"name": "bbb222-name" }
] }
```

### 3.7 IPNS: Naming and Mutable State
> 
到目前为止，IPFS堆栈形成了p2p块交换，构建了内容寻址的对象DAG。它用于发布和检索不可变对象。 它甚至可以跟踪这些对象的版本历史记录。但是，缺少一个关键组件：可变命名。没有它，新内容的所有通信必须在外带发送IPFS链接。 需要的是在同一路径上检索可变状态的某种方法。 值得说明的原因(如果最终需要可变数据),我们努力建立一个不可变的Merkle DAG。 考虑从Merkle DAG中放弃IPFS的属性：对象可以（a）通过其哈希检索，（b）检查完整性，（c）链接到其他人，以及（d）无限缓存。
在某种意义上：Objects are permanent
这些是高性能分布式系统的关键属性，其中数据在网络链路上移动的成本很高。对象内容寻址通过以下方式构建Web：（a）显着的带宽优化，（b）不可信内容服务，（c）永久链接，以及（d）对任何对象及其引用进行完全永久备份的能力。 
Merkle DAG，不可变的内容寻址对象，以及Naming，指向MerkleDAG的可变指针，采用了在许多成功的分布式系统中使用的二分法。 这些包括Git版本控制系统，其不可变对象和可变引用; 和Plan9[？]，UNIX的分布式继承者，具有可变的Fossil [？]和不可变的Venti [？]文件系统。 LBFS [？]也使用可变索引和不可变块。

#### 3.7.1 Self-Certified Names
> 
使用SFS的命名方案[12,11]为我们提供了一种在密码学全局命名空间中构造可自变的自认证名称的方法。 IPFS的方案如下：
1.回想一下IPFS中：NodeId = hash(node.PubKey)
2.我们分配每一个用户一个可变命名空间：/ipns/<NodeId>
3.用户可以将对象发布到此路径，并用私钥签名，比如说：
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm/
4.当其他用户检索对象时，他们可以检查签名是否与公钥和NodeId匹配。这验证了用户发布的Object的真实性，实现了可变的状态回溯。

> 请注意以下细节：
•ipns（InterPlanetary NameSpace）单独的前缀是为程序和人类读者建立一个易于识别的可变和不可变路径之间的区别。
•因为这不是内容寻址对象，所以发布它依赖于IPFS中唯一可变的状态分发系统，即路由系统。 该过程是（1）将对象发布为常规不可变IPFS对象，（2）在路由系统上将其散列作为元数据值发布：
routing.setValue(NodeId, <ns-object-hash>)
•发布的Object中的任何链接都充当命名空间中的子名称：
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm/
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm/docs
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm/docs/ipfs
•建议发布commit对象或具有版本历史记录的其他对象，以便客户端可以找到旧名称。这留给了用户选项(并不强制)。
请注意，当用户发布此Object时，无法以相同的方式再发布它

#### 3.7.2 Human Friendly Names
> 
虽然IPNS确实是一种分配和重新分配名称的方式，但它不是非常用户友好，因为它将长哈希值暴露为名称，这是众所周知难以记住的。 这些适用于URL，但不适用于多种离线传输。 因此，IPFS通过以下技术提高了IPNS的用户友好性。

> 
Peer Links
在SFS的鼓励下，用户可以将其他用户的对象直接链接到他们自己的对象（命名空间，主页等）。这样做的好处是还可以创建信任网（并支持旧的证书颁发机构模型）：

```
# Alice links to bob Bob
ipfs link /<alice-pk-hash>/friends/bob /<bob-pk-hash>
# Eve links to Alice
ipfs link /<eve-pk-hash/friends/alice /<alice-pk-hash>
# Eve also has access to Bob
/<eve-pk-hash/friends/alice/friends/bob
# access Verisign certified domains
/<verisign-pk-hash>/foo.com
```

> 
DNS TXT IPNS Records.
如果/ipns/<domain>是一个有效的域名,IPFS在DNS TXT记录表中查找Key的ipns。IPFS将该值解释为对象哈希或另一个IPNS路径：

```
# this DNS TXT record
ipfs.benet.ai. TXT "ipfs=XLF2ipQ4jD3U ..."
# behaves as symlink
ln -s /ipns/XLF2ipQ4jD3U /ipns/fs.benet.ai
```

> Proquint可发音标识符 Proquint Pronounceable Identifiers
一直有将二进制编码成可发音单词的方案。 IPNS支持Proquint [？]。 从而：

```
# this proquint phrase
/ipns/dahih-dolij-sozuk-vosah-luvar-fuluh
# will resolve to corresponding
/ipns/KhAwNprxYVxKqpDZ
```

> 名称缩短服务。Name Shortening Services.
必然会出现提供名称缩短服务，为用户提供名称空间。 这类似于我们今天看到的DNS和Web URL：

```
# User can get a link from
/ipns/shorten.er/foobar
# To her own namespace
/ipns/XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm
```

### 3.8 Using IPFS
> 
IPFS旨在以多种不同方式使用。 以下是我将要介绍的一些用例：
1.作为已挂载的全局文件系统，在/ipfs和/ipns下。
2.作为已安装的个人同步文件夹，可自动发布，发布和备份任何写入。
3.作为加密文件或数据共享系统。
4.作为所有软件的版本化软件包管理器。
5.作为虚拟机的根文件系统。
6.作为VM的引导文件系统（在管理程序下）。
7.作为数据库：应用程序可以直接写入MerkleDAG数据模型，并获得IPFS提供的所有版本控制，缓存和分发。
8.作为链接（和加密）通信平台。
9.作为完整性检查CDN的大文件（没有SSL）。
10.作为加密的CDN。
11.在网页上，作为网络CDN。
12.作为一个新的永久网络，链接不会消失。
> 
IPFS实现目标：
（a）在您自己的应用程序中导入的IPFS库。
（b）直接操纵对象的命令行工具。
（c）使用FUSE [？]或作为内核模块安装的文件系统。

## 4. 将来
> 
IPFS背后的思想是学术界和开源领域数十年成功的分布式系统研究的产物。IPFS综合了迄今为止最成功系统中的许多最佳创意。 除了BitSwap这是一种新颖的协议之外，IPFS的主要贡献在于系统的耦合和设计的综合。 
IPFS是对新的分散式互联网基础设施的雄心勃勃的愿景，可以在其上构建许多不同类型的应用程序。 至少，它可以用作全局，已安装，版本化的文件系统和命名空间，或者用作下一代文件共享系统。 在最好的情况下，它可以将网络推向新的视野，在那里发布有价值的信息并不会将其托管在发布者身上，而是在那些感兴趣的人身上，用户可以信任他们收到的内容而不信任他们从中收到的Peers，以及旧的 但重要的文件不会丢失。 期待IPFS将我们带入永恒网络。


# PHANTOM: A Scalable BlockDAG protocol

### Yonatan Sompolinsky and Aviv Zohar

### School of Engineering and Computer Science,

### The Hebrew University of Jerusalem, Israel

```
{yoni sompo,avivz}@cs.huji.ac.il
```
```
Abstract
In 2008 Satoshi Nakamoto invented the basis for what would come to be known as
blockchain technology. The core concept of this system is an open and anonymous network
of nodes, orminers, which together maintain a public ledger of transactions. The ledger takes
the form of a chain of blocks,the blockchain, where eachblockis a batch of new transactions
collected from users. One primary problem with Satoshi’s blockchain is its highly limited
scalability. The security of Satoshi’slongest chain rule, more generally known asthe Bitcoin
protocol, requires that all honest nodes be aware of each other’s blocks in real time. To this
end, the throughput is artificially suppressed so that each block fully propagates before the next
one is created, and that no “orphan blocks” that fork the chain be created spontaneously.
In this paper we present PHANTOM, a protocol for transaction confirmation that is secure
under any throughput that the network can support. PHANTOM thus does not suffer from
the security-scalability tradeoff which Satoshi’s protocol suffers from. PHANTOM utilizes a
Directed Acyclic Graph of blocks, akablockDAG, a generalization of Satoshi’s chain which
better suits a setup of fast or large blocks. PHANTOM uses a greedy algorithm on the
blockDAG to distinguish between blocks mined properly by honest nodes and those mined
by non-cooperating nodes that deviated from the DAG mining protocol. Using this distinction,
PHANTOM provides a full order on the blockDAG in a way that is eventually agreed upon
by all honest nodes.
```
#### 1. INTRODUCTION

A. The Bitcoin protocol

The Bitcoin protocol instructs miners how to create blocks of transactions. The body of a
block contains new transactions published by users, a proof-of-work puzzle, and a pointer to
one previous block. The latter rule implies that blocks naturally form in a tree structure. When
creating his next block, the miner is instructed to reference the tip of the longest chain within
the tree and ignore the rest of blocks (akaorphans).
Miners share and propagate a block immediately upon receiving or creating it, and reference
the latest block in the chain they observe. The security of Bitcoin relies on honest nodes being
sufficiently connected so that when one miner extends the chain with a new block, it propagates
in time to all honest nodes before the next one is created.


```
Fig. 1:An example of a block DAGG. Each block
references all blocks to which its miner was aware at
the time of its creation. The DAG terminology, applied
to blockHas an example, is as follows:
past(H) = {Genesis,C,D,E} – blocks which
H references directly or indirectly, and which were
provably created beforeH;
future(H) ={J,K,M}– blocks which referenceH
directly or indirectly, and which were provably created
afterH;
anticone(H) = {B,F,I,L}– the order between
these blocks andHis ambiguous.Deciding the order
betweenHand blocks inanticone(H)is the main
challenge of a DAG protocol.
tips(G) = {J,L,M}– leaf-blocks, namely, blocks
with in-degree 0; these will be referenced in the header
of the next block
```
In order to guarantee this property, the creation of blocks is regulated by the protocol to occur
once every 10 minutes. As a result, Bitcoin suffers from a highly restrictive throughput in the
order of 3-7 transactions per second (tps).

B. The PHANTOM protocol

In this work we present PHANTOM, a protocol that enjoys a very large transaction throughput
compared to Bitcoin. PHANTOM structures blocks in a Directed Acyclic Graph, ablockDAG.
Rather than extending a single chain, miners in PHANTOM are instructed to reference all blocks
in the graph (that were not previously referenced, i.e., leaf-blocks). An example of a blockDAG,
and the basic terminology, is provided in Figure 1 above.
The core challenge of a DAG protocol is how to order transactions embedded in it, so that in
case of two (or more) conflicting transactions, we can accept the one that arrived first (according
to the prescribed order) and reject the other(s).
To that end, PHANTOM relies on the interconnectivity of honest nodes (similarly to Bitcoin’s
assumption in slow rates). Since cooperating PHANTOM miners propagate their blocks as soon
as possible and reference all of their counterparts’ blocks, we should expect to see in the DAG a
well-connected cluster of blocks. In contrast, blocks that were mined by non-cooperating nodes
will appear as outliers and will easily be discerned. Indeed, deviation from PHANTOM’s mining
protocol comes in the form of (i) withholding a new block for a while, or\and (ii) creating a
new block that does not reference other blocks available at the time, both cases in which the
new block can be recognized and penalized.
Following this intuition, together with the assumption that honest nodes hold a majority of
the hashrate, we argue that thelargestset of blocks with good inter-connectivity was mined by


```
Fig. 2:An example of the largest 3-cluster of blocks within
a given DAG:A,B,C,D,F,G,I,J(coloured blue). It is
easy to verify that each of these blue blocks has at most 3
blue blocks in its anticone, and (a bit less easy) that this
is the largest set with this property. Setting PHANTOM’s
inter-connectivity parameter withk= 3means that at most
4 blocks are assumed to be created within each unit of
delay, so that typical anticone sizes should not exceed 3.
Blocks outside the largest 3-cluster,E,H,K(coloured red),
belong to the attacker (w.h.p.). For instance, blockEhas 6
blue blocks in its anticone (B,C,D,F,G,I); these blocks
didn’t referenceE, presumably becauseEwas withheld
from their miners. Similarly, blockKadmits 6 blue blocks
in its anticone (B,C,G,F,I,J); presumably, its malicious
miner received already some blocks from(B,C,D,G), but
violated the mining protocol by not referencing them.
```
honest nodes, with high probability. Accordingly, given a blockDAG, we would want to solve
the following optimization problem:

## Maximumk-cluster SubDAG (M CSk)

```
Input:DAGG= (C,E)
Output:A subsetS∗⊂Cof maximum size, s.t.|anticone(B)∩S∗|≤kfor allB∈S∗.
```
Here,anticone(B)is the set of blocks in the DAG which did not referenceB(directly or
indirectly via their predecessors) and were not referenced byB(directly or indirectly viaB’s
predecessors). The parameterkis related to an assumption that PHANTOM makes regarding
the network’s propagation delay; this is explained in detail in Section 4, following the formal
framework specified in Section 3.
In practice, the Maximum k-cluster SubDAG is NP hard (see problem [GT26] in [2]).
PHANTOM works therefore with a variant of this problem, using a greedy algorithm approach.
The algorithm will be given in Sections 2, and some interesting variants in Section 6.
Once the set of honest and dishonest blocks are properly recognized by the protocol, we order
the DAG in a way that favours the former set and penalizes the latter. Interestingly, the specific
way in which we order honest blocks is of no importance to the security of the protocol—any
arbitrary rule which respects the topology will enjoy the robustness properties of PHANTOM,
as we prove formally in Section 5.
However, the ordering rule does affect confirmation times. An arbitrary topological ordering
might take a long while before converging, especially if an active visible attack is taking place.
Thus, vanilla PHANTOM does not guarantee fast convergence time in all cases. In Section 7 we
elaborate on this, and discuss techniques and modifications that allow fast confirmation times.


C. Related work

Many suggestions to improve Bitcoin’s scalability have been proposed in recent years. These
proposals fall into two categories,on-chain scalingandoff-chain scaling. Roughly speaking, the
former includes protocols where all valid transactions are those that appear – as in Bitcoin –
inside blocks that are organized in some data structure (aka “the ledger”).

On-chain scaling.The protocols in this category may differ e.g. in how fast blocks are created,
how blocks are organized in the ledger (a chain, a tree, a DAG, etc.), which transactions in
the ledger are considered valid, and more. PHANTOM belongs to this line of works. Previous
works in this family of protocols includes GHOST [9], where a main chain of blocks is chosen
according to a greedy algorithm and not through the longest chain rule; Inclusive [5], where
any chain-selection rule is extended to an ordered DAG and transactions off the main chain are
added in a consistent manner; Bitcoin NG [1], where the ledger consists of slowkey blocks
(containing no transactions) and fastmicroblocksthat contain transactions. The sole purpose of
key blocks in Bitcoin NG is to define the miner that is eligible to create microblocks in that
epoch and confirm thus transactions at a high rate.
GHOST is still susceptible to some attacks, one of which was described in [3]. The DAG
in Iclusive adds throughput but not security to the main chain, hence suffers from the same
limitations as the underlying main chain selection rule. Key blocks in Bitcoin NG are still
generated slowly, thus confirmation times remain high.
Our work is most similar to the SPECTRE protocol [8]. SPECTRE enjoys both high throughput
and fast confirmation times. It uses the structure of the DAG as representing an abstract vote
regarding the order between each pair of blocks. One caveat of SPECTRE is that the output of
this pairwise ordering may not be extendable to a full linear ordering, due to possible Condorcet
cycles. PHANTOM solves this issue and provides a linear ordering over the blocks of the DAG.
As such, PHANTOM can support consensus regarding any general computation, also known as
Smart Contracts, which SPECTRE cannot. Indeed, in order for a computation or contract to be
processed correctly and consistently, the full order of events in the ledger is usually required,
and particularly the order of inputs to the contract.^1 PHANTOM’s linear ordering does not come
without cost—confirmation times are mush slower than those in SPECTRE. In Section 7 we
describe how the same system can simultaneously enjoy the best of both protocols.

Off-chain scaling.Another totally different approach keeps block creations infrequent and their
sizes small (so that propagation delay remains negligible), yet this slow chain is not used for
recording the entire economic activity. Instead, most of the transactions occur outside the chain,
with better scalability, and the chain itself is used for the purpose of resolving conflicts or
settling transactions. One example is Hybrid Consensus [6], improving over [4], which uses the
chain to select a rotating committee of nodes which in turn run a classic consensus protocol to
confirm transactions in the corresponding epoch. Another well known proposed solution in the
same category is the Lightning Network [7] (LN), where transactions are processed off-chain

(^1) Contracts that do not require such a strict ordering can indeed be served under SPECTRE as well.


over over a network of micropayment channels, and the blockchain is used only for settlement
of these channels.
Our work is orthogonal and complementary to these solutions, and can enhance their operation
by orders-of-magnitude. For instance, when the DAG is used to serve channel-settlement
transactions of LN, it allows for a much cheaper access (due to larger supply of blocks and
capacity) and much faster processing than if the LN were operating over a chain.

#### 2. THEPHANTOMPROTOCOL

In this section we describe the operation of the PHANTOM protocol. PHANTOM consists of
the following three-step procedure:

1) Using the structure of the DAG, we recognize in it a cluster of well-connected blocks; with
high probability, blocks that were mined honestly belong to this cluster and vice versa.
2) We extend the DAG’s natural partial ordering to a full topological ordering in a way that
favours blocks inside the selected cluster and penalizes those outside it.
3) The order over blocks induces an order over transactions; transactions in the same block are
ordered according to the order of their appearance in it. We iterate over all transactions in
this order, and accept each one that is consistent (according to the underlying Consistency
notion) with those approved so far.
The Consistency notion used in the last step depends on the specific application under
consideration. For instance, with regards to the Payments application, a transaction is consistent
with the history only if all of its inputs have been approved and no double spending transaction
has been approved before. Our work is agnostic to the definition of the Consistency rule. The
contribution of PHANTOM is its implementation of the first two steps described above, which
we now turn to describe.

A. Intuition

How can we distinguish between honest blocks (i.e., blocks mined by cooperating nodes) and
dishonest ones? Recall that the DAG mining protocol instructs a miner to acknowledge in its
new block the entire DAG it observes locally, by referencing the “tips” of the DAG. Thus, if
blockBwas mined at timetby an honest miner, then any block published before timet−D
was received by the miner and is therefore inB’s past set (i.e., referenced byB directly or
recursively via its predecessors; see illustration in Figure 1). Similarly, ifB’s miner is honest
then it publishedBimmediately, and so any honest block created after timet+Dbelongs to
B’s future set.
As a result, the set of honest blocks inB’s anticone – which we denoteanticoneh(B)–
is typically small, and consists only of blocks created in the interval[t−D,t+D].^2 In other
words, the probability that an honest blockB will suffer a large honest anticone is small:
Pr (|anticoneh(B)|> k)∈ O

#### (

```
e−C·k
```
#### )

```
, for some constantC > 0 (this stems from a bound
```
(^2) Note that, in contrast toanticoneh(B), an attacker can easily increase the size ofanticone(B), for any block
B, by creating many blocks that do not referenceBand that are kept secret so thatBcannot reference them.


on the Poisson distribution’s tail). We rely on this property and set PHANTOM’s parameterk
such that the latter probability is smaller thanδ, for some predefinedδ > 0 ; see discussion in
Section 4.
Following this intuition, the set of honest blocks (save perhaps a fractionδ thereof) is
guaranteed to form ak-cluster.

Definition 1.Given a DAGG= (C,E), a subsetS⊆ Cis called ak-cluster, if∀B ∈S:
|anticone(B)∩S|≤k.

Note that the attacker can easily createk-clusters as well, e.g., by structuring his blocks in a
single chain. Fortunately, we can leverage the fact that the group of honest miners possesses a
majority of the computational power, and look at thelargestk-cluster. We argue that the latter
represents, in most likelihood, blocks that were mined properly by cooperating nodes. Refer to
Figure 2 for an illustration of the largest 3 -cluster in a given blockDAG. Identifying this set in
general DAGs turns out to be computationally infeasible, and so in practice our protocol uses a
greedy algorithm which is good enough for our purposes.
We term this selection of ak-cluster acolouring of the DAG, and use the colours blue and
red as a convention for blocks inside and outside the chosen cluster, respectively.

B. Step #1: recognizing honest blocks

Algorithm 1 below selects ak-cluster in a greedy fashion. We denote byBLUEk(G)the set
of blocks that it returns. The algorithm operates as follows:

1) Given a DAGG, the algorithm recursively computes on the past set of each tip inG.^3 This
outputs ak-cluster for each tip.^4 (lines 4-5)
2) Then, it makes a greedy choice and picks the largest one among the outputted clusters.
(lines 6-7)
3) Finally, it tries to extend this set and add to it any block whose anticone is small enough
with respect to the set. (lines 8-10)
Intuitively, we first let the DAG inherit the colouring of its highest scoring tip, Bmax,
where the score of a block is defined as the number of blue blocks in its past:score(B) :=
|BLUEk(past(B))|. Then, we proceed to colour blocks inanticone(Bmax)in a way that
preserves thek-cluster property. This inheritance implies that the greedy algorithm operates
as a chain-selection rule—Bmax is the chain tip, the highest scoring tip inpast(Bmax) is
its predecessor in the chain, and so on. We denote this chain by Chn(G) = (genesis =
Chn 0 (G),Chn 1 (G),...,Chnh(G)). The reasoning behind this procedure is very similar to that
given in Section 1 in relation to the Maximumk-cluster SubDAG problem. They only differ in
that, instead of searching for the maximalk-cluster, we are hoping to maximize it via the tip

(^3) A tip is a leaf-block, that is, a block not referenced by other blocks. See Figure 1.
(^4) Observe that, for any blockB, the DAGpast(B)is fixed once and for all atB’s creation, and in particular
the setBLUEk(past(B))cannot be later modified. Thus, in an actual implementation of Algorithm 1, the sets
BLUEk(B)will have been computed already (by previous calls) and stored, and there will be no need to recompute
them.


Fig. 3:An example of a blockDAGGand the operation of the greedy algorithm to construct its blue set
BLUEk(G)set, under the parameterk= 3. The small circle near each blockXrepresents its score,
namely, the number of blue blocks in the DAGpast(X). The algorithm selects the chain greedily, starting
from the highest scoring tipM, then selecting its predecessorK(the highest scoring tip inpast(M)),
thenH,D(breaking theC,D,Etie arbitrarily), and finallyGenesis. For methodological reasons, we
add to this chain a hypothetical “virtual” blockV – a block whose past equals the entire current DAG.
Blocks in the chain(genesis,D,H,K,M,V)are marked with a light-blue shade. Using this chain, we
construct the DAG’s set of blue blocks,BLUEk(G). The set is constructed recursively, starting with an
empty one, as follows: In step 1 we visitDand addgenesisto the blue set (it is the only block in
past(D)). Next, in step 2, we visitHand add toBLUEk(G)blocks that are blue inpast(H), namely,
C,D,E. In step 3 we visitKand addH,I; note that blockBis inpast(K)but was not added to the
blue set, since it has 4 blue blocks in its anticone. In step 4 we visitMand addKto the blue set; again,
note thatF∈past(M)could not be added to the blue set due its large blue anticone. Finally, in step
5, we visit the blockvirtual(G) =V, and addMandLtoBLUEk(G), leavingJaway due its large
blue anticone.

with maximal cluster and then adding blocks from its anticone. Thus, the reader should think
of our algorithm (informally) as approximating the optimal solution to the Maximumk-cluster
SubDAG problem.
We demonstrate the operation of this algorithm in Figure 3. Another example appears in
Figure 4.
Note that the recursion halts because for any blockB∈G:|past(B)|<|G|.
Let us specify the order in which blocks inanticone(Bmax)should be visited, in line 8 of the
algorithm. We suggest inserting all blocks inanticone(Bmax)into alexicographical topological
priority queue, which we denotetopoqueue. The priority of a block is represented by the size


of its past set;^5 in case of ties, the block with lowest hash ID is chosen.

Remark. Our choice of ordering via topoqueue is not inherently significant, and many
alternative topological orderings can provide similar robustness properties. That said, it might
be the case that other rules provide faster convergence and confirmation times. We revisit this
issue in Section 7.

```
To summarize the function that the blue set satisfies, we state the following:
```
Proposition 2.LetG=Gpub∞ be the eventual DAG containing all blocks in history, and letB
be an arbitrary block inG.

- IfBwas created by an honest miner, the probability thatBwill not belong toBLUEk(G)
    decreases exponentially withk.
- IfBwas created by a malicious miner, and was withheld for a time interval of lengthT,
    the probability thatBwill belong toBLUEk(G)decreases exponentially withT.

```
The proof of this proposition follows from the proof of Claim 3 in Section 5.
```
C. Step #2: ordering blocks

Recall that the DAG is ordered partially via its topology. We now wish to extend this order
to a full order (akaa topological order). Our objective is to define a rule that gives precedence
to blocks in the blue set, that penalizes blocks outside this set by deferring their location in the
order, and that preserves the topological relations of blocks.
We propose the following procedure: Traverse the blue set according to some topological
order, and iteratively add blocks to the current last position inordk; when visiting blockB, first
check if there are blocks inpast(B)that haven’t been added yet, add such blocks to the order
(again, according to some topological order), and then addBto the order.
For example, a possible output of ord^3 on the blockDAG illustrated in Figure 2 is:
(A,D,C,G,B,F,I,E,J,H,K).
This procedure is formalized in Algorithm 2 below. The algorithm begins by initializing an
empty priority queue and an empty ordered list. Throughout, the list will represent the order in
which blocks were popped out from the queue. The algorithm favours blocks in the blue set by
adding to the queue all of the blue children of the current block. When a block is pushed into
the queue, all blocks in its past (that weren’t already treated) are pushed as well.
In the algorithm,topoqueueis the same lexicographical topological priority queue defined
in the preceding subsection. In addition, the queue should avoid duplicating elements, and so
topoqueue.push(C)should do nothing in caseCis already intopoqueue.
The intuition here is simple. The algorithm is intended to guarantee that a blockBcan precede
a blue blockConly ifBis blue, orBis referenced by a blue block. In this way, blocks that
were withheld by an attacker will not precede blocks that were mined properly and published
on time (which are represented roughly by the set of blue blocks).

(^5) This guarantees that a block cannot be popped out while a block in its past is still in the queue, sinceC∈
future(B) =⇒|past(C)|>|past(B)|.


Algorithm 1Selection of a blue set
Input:G– a block DAG,k– the propagation parameter
Output:BLUEk(G)– the dense-set ofG
1:functionCALC-BLUE(G,k)
2: ifB==genesisthen
3: return{genesis}
4: forB∈tips(G)do
5: BLUEk(B)←CALC-BLUE(past(B),k)
6: Bmax←arg max{|BLUEk(B)|:B∈tips(G)}(and break ties arbitrarily)
7: BLUEk(G)←BLUEk(Bmax)∪{Bmax}
8: forB∈anticone(Bmax)doin some topological ordering
9: if|anticone(B)∩BLUEk(G)|≤kthen
10: addBtoBLUEk(G)
11: returnBLUEk(G)

Algorithm 2Ordering of the DAG
Input:G– a block DAG,k– the propagation parameter
Output:ord(G)– an ordered list containing all ofG’s blocks
1:functionORDER(G,k)
2: initialize empty queuetopo queue
3: initialize empty ordered listL
4: BLUEk(G)←CALC-BLUE(G,k)
5: topoqueue.push(genesis)
6: whiletopoqueue 6 =∅do
7: B←topo queue.pop()
8: L.add(B)(Bis added to the end of the list)
9: for all C∈childrenB∩BLUEk(G)do
10: for all D∈past(C)∩anticone(b)\Ldo
11: topo queue.push(D)
12: topoqueue.push(C)
13: ord(G)←L
14: returnord(G)


D. Implications to transaction security

We now demonstrate how the above procedures of PHANTOM enable safe acceptance of
transactions. Consider a transactiontx∈B, whereBis a block in the blue set ofG. In order
to rendertxinvalid, a conflicting transactiontx ̄ must precede it in the order, and must therefore
be embedded in a blockC∈anticone(B)that precedesB.^6 The ordering procedure implies
that, forCto precedeB, it must either be a blue block or in the past set of a blue block. In both
cases,Ccould not have been withheld for too long, by the second guarantee of Proposition 2.
Thus, the recipient oftxcan wait for the blue set aroundBto become sufficiently robust to
reorgs, and then approvetx. In Section 5 we prove that robustness is indeed obtained, after
some waiting time.

#### 3. FORMALMODEL ANDSTATEMENT

In this section we describe our formal framework. While we introduce new notation and
terminology, the reader should keep in mind thatwe stick to Bitcoin’s model in almost every
respect—transactions, blocks, Proof-of-work, computationally bounded attacker, P2P propagation
of blocks, probabilistic security guarantees, etc. The “only” difference is that a block references
(possibly) several predecessors rather than a single one. While this has far reaching consequences
on how the ledger is to be interpreted, on the mining side things remain largely the same.

A. Network

We follow the model specified in [8]. The network of nodes (or miners) is denotedN,honest
denotes the set of nodes that abide to the mining protocol (as defined below), andmalicious
denotes the rest of the nodes. Honest nodes form a connected component inN’s topology, and
the communication delay diameter of the honest subnetwork isD: if an honest nodev∈ N
sends a message of sizebMB at timet, it arrives at all honest nodes by timet+Dthe latest.
The attacker is assumed to suffer no delays whatsoever on its outgoing or incoming links.
The real value ofDisa prioriunknown. The PHANTOM protocol assumes thatDis always
smaller than some constantDmax(both depend on the block sizeb). The parameterDmaxis not
hard-coded explicitly in the protocol, rather it influences another parameter,k=k(Dmax), which
is hard-coded and decided once and for all at the inception of the system. Roughly speaking,
k(Dmax)represents an upper bound on the number bound on the number of blocks that the
network creates in one unit of delay and that may not be referenced by one another. Section 4
discusses this parameter in more detail.

B. Mining framework

Proof-of-work.Nodes create blocks of transactions by solving Proof-of-work puzzles. Block
creation follows a Poisson process with parameterλ. For the sake of simplicity, we assume that

(^6) Or,tx ̄can appear inBbeforetx, but this is not an interesting scenario.


λis constant.^7 The computational power of nodev∈ Nis captured by 0 < αv< 1 , which
represents the probability that nodevwill be the creator of the next block in the system (at any
point in time; this is a memoryless process). The attackers’ computational power is less than
50%. Thus,

#### ∑

```
v∈Nαv= 1, and
```
#### ∑

v∈maliciousαv=:α <^0.^5.
Block references.Every block specifies its direct predecessors by referencing their ID in its
header (a block’s ID is obtained by applying a collision resistant hash to its header); the choice
of predecessors will be described in the next subsection. This results in a structure of a direct
acyclic graph (DAG) of blocks (as blocks can only reference blocks created before them), denoted
typicallyG= (C,E). Here,Crepresents blocks andErepresents the hash references. We will
frequently writeB∈Ginstead ofB∈C.

DAG topology.The topology of the blockDAG induces a natural partial ordering over blocks,
as follows: if there is a path in the DAG from blockCto blockBwe writeB∈past(C); in
this case,Cwas provably created afterBand thereforeBshould precedeCin the order.^8 A
node does not consider a block as valid until it receives its entire past set. The unique block
genesisis the block created at the inception of the system, and every valid block must have it
in its past set.
Similarly, the future set of a block,future(B), represents blocks that were provably created
after it:B ∈past(C) ⇐⇒ C∈future(B). In contrast to the past set, the future set of
a block keeps growing in time, as more blocks are created and are referencing it. To avoid
ambiguity, we writefuture(B)∩Gorfuture(B,G), and writefuture(B)only when the
context is clear or unimportant.
The setanticone(B)represents all blocks not inB’s future or past (excludingBas well).
These are blocks whose ordering with respect to B is not defined via the partial ordering
that the topology of the DAG induces. Formally, for two distinct blocks B,C ∈ G:C ∈
anticone(B,G) ⇐⇒ (B /∈past(C)∧C /∈past(B)) ⇐⇒ B∈anticone(C,g). Here too
we usually specify the context,anticone(B,G), because the anticone set can grow with time.
In Figure 1 above we illustrates this terminology.

DAG mining protocol.Gvtdenotes the block DAG that nodev∈Nobserves at timet. This DAG
represents the history of all (valid) block-messages received by the node.Goraclet :=∪v∈NGvt
denotes the block DAG of a hypothetical oracle node, andGpubt :=∪v∈honestGvt denotes the
block DAG containing all blocks that are visible to some honest node(s).
Atipof the DAG is a leaf-block, namely, a block with in-degree 0. The instructions to a
miner in the DAG paradigm are simple:

```
1) When creating or receiving a block, transmit it to all of one’s peers inN. Formally, this
implies that∀v,u∈honest:Gvt⊆Gut+D.
```
(^7) In practice,λmust occasionally be readjusted to account for shifting network conditions. PHANTOM can support
a retargeting mechanism similar to Bitcoin’s, e.g., readjust every time thatChn(G)grows by 2016 blocks.
(^8) Note that an edge in the DAG points back in time, from the new block to previously created blocks which it
references.


2) When creating a block, embed in its header a list containing the hash of all tips in the
locally-observed DAG. Formally, this implies that if blockB was created at timet, by
honest nodev, thenpast(B) =Gvt.^9
Since these are the only two mining rules in our system, a byzantine behaviour of the attacker
(which controls up toαof the mining power) amounts to an arbitrary deviation from one or
both of these instructions.

C. DAG client protocol

The DAG as described so far possibly embeds conflicting transactions. These are resolved on
the client level. Aclientcan be defined formally as a node inN which has no mining power.
Intuitively, it is any user of the system who is interested in reading and interpreting the current
state of the ledger.
In this work, a transactiontxis an arbitrary message that is embedded in a block. An
underlyingConsistencyrule takes as input a setTof transactions and returnsvalidorinvalid.
Our work is agnostic to the definition and operation of this rule, or to the characterization of the
transaction space. Instead, we focus on the following task: devising a protocol through which
all nodes agree on the order of all transactions in the system. Once such an order is agreed, one
can iterate over all transactions, in the prescribed order, and approve each transaction that is
consistent – according to the underlying rule – with those approved so far.Such an ordering rule
constitutes the client protocol, and is run by each client locally without any need to communicate
additional messages with other clients.
Formally, an ordering ruleordtakes as input a blockDAGGand outputs a linear order over
G’s blocks,ord(G) = (B 0 ,B 1 ,...,B|G|). Transactions in the same block are ordered according
to their appearance in it, and this convention allows us to talk henceforth on the order of blocks
only. With respect to a given ruleord, we writeB≺ord(G)Cif the index ofBprecedes that of
Cinord(G); we abbreviate and writeB≺GCor evenB≺Cwhen the context is understood.
For convenience, we use the same notationB≺GCwhenB∈GbutC /∈G.

D. Convergence of the order

The following definition captures the desired security of the protocol, in terms of the
probability that some order between two blocks will be reversed.

Definition 3.Fix a ruleord. LetB∈G=Gpubt 0. The functionRiskis defined by the probability
that a block that did not precedeBin timet 1 ≥t 0 will later come to precede it:Risk(B,t 1 ) :=

Pr

#### (

```
∃s > t 1 ,∃C∈Gpubs :B≺Gpubt 1 C∧C≺Gpubs B
```
#### )

#### .

In the definition above, the probability is taken over all random events in the network, including
block creation and propagation, as well as the attacker’s arbitrary (byzantine) behaviour. The
convergence property below guarantees that the order between a block and those succeeding it,

(^9) Technically it is more accurate to writepast(B) =Gvt\{B}, as a block does not belongs to its own past set.


or those not published yet, will not be reversed,w.h.p.This captures the security of the protocol,
as it provides honest nodes with (probabilistic) security guarantees regarding possible reorgs.

Property 1.An ordering ruleordisconvergingif∀t 0 > 0 andB∈Gpubt 0 :tlim
1 →∞

```
Risk(B,t 1 ) =
```
0 , even when a fractionαof the mining power is byzantine.

Remark. Property 1 essentially couples the Safety and Liveness properties required from
consensus protocols. as in Bitcoin and other protocols). Nevertheless, we avoid phrasing our results in these
terms, for the sake of clarity of presentation. The complication arises from the need to analyze
the system from the perspective of every nodeGvt, and not merely from the public ledger’s
hypothetical perspectiveGpubt ; this technicality is not unique to PHANTOM, and should be
regarded in any work that formalizes blockchain based consensus (unless propagation delays
are assumed to be negligible). We leave the task of bridging this gap to a later version.

The security threshold is the minimal hashing power that the attacker must acquire i order to
disrupt the protocol’s operation:

Definition 4.The security threshold of an ordering ruleord is defined as the maximalα
(attacker’s relative computational power) for which Property 1 holds true.

A protocol is scalable if it is safe to increase the block creation rateλwithout compromising
the security, that is, if the security threshold does not deteriorate asλincreases (this can be
phrased also in terms of increasing the block sizebrather thanλ).

E. Main result

Our goal in this paper is to describe formally the ordering procedure of PHANTOM and to
prove that it is scalable in the above sense.

Theorem 5(PHANTOM scales).Given a block creation rateλ > 0 ,δ > 0 , andDmax> 0 , if
Dmaxis equal to or greater than the network’s propagation delay diameterD, then the security
threshold of PHANTOM, parameterized withk(Dmax,δ), is at least 1 / 2 ·(1−δ).

The parameterization of PHANTOM viak(Dmax,δ) is defined in the subsequent section
(see (1)). Theorem 5 encapsulates the main achievement of our work. We prove the theorem
formally in Section 5. Contrast this result to a theorem regarding the Bitcoin protocol, which
appears in several forms in previous work (e.g., [6], [9]):

Theorem 6(Bitcoin does not scale).Asλincreases, the security threshold of the Bitcoin protocol
goes to 0.

Finally, we note that even ifDmax6≥D, the system’s security does not immediately break
apart. Rather, the minimal power needed to attack the system goes from 50% to 0, deteriorating
at a rate that depends on the error gapD−Dmax.


#### 4. SCALABILITY AND NETWORK DELAYS

A. The propagation delay parameterDmax

The scalability of a distributed algorithm is closely tied to the assumptions it makes on the
underlying network, and specifically on its propagation delayD. The real value ofDis both
unknown and sensitive to shifting network conditions. For this reason, Bitcoin operates under
the assumption thatDis much smaller than 10 minutes, and sets the average block interval
time to 10 minutes. While this seems like an overestimation of the network’s propagation delay
under normal conditions (at least in 2018’s Internet terms), some safety margin must be taken,
to account for peculiar network conditions as well. Similarly, in PHANTOM we assume that
the unknownDis upper bounded by someDmaxwhich is known to the protocol. The protocol
does not explicitly encodeDmax, rather, it is parameterized withkwhich depends on it, as will
be described in the next subsection.
The use of ana prioriknown boundDmaxdistinguishes PHANTOM’s security model from
that of SPECTRE [8]. While the security of both protocols depends on the assumption that the
network’s propagation delayDis upper bounded by some constant, in SPECTRE the value of
such a constant need not be known or assumed by the protocol, whereas PHANTOM makes
explicit use of this parameter (viak) when ordering the DAG’s blocks. The fact that the order
between any two blocks becomes robust in PHANTOM, but not in SPECTRE, should be ascribed
to this added assumption; see further discussion in Section 7.

B. The anticone size parameterk

The parameterkis decided from the outset and hard-coded in the protocol. It is defined as
follows:

```
k(Dmax,δ) := (1)
```
```
min
```
#### 

#### 

#### 

```
kˆ∈N:
```
#### (

```
1 −e−^2 ·Dmax·λ
```
#### )− 1

#### ·

#### 

#### 

#### ∑∞

```
j=ˆk+
```
```
e−^2 ·Dmax·λ·
```
```
(2·Dmax·λ)j
j!
```
#### 

```
< δ
```
#### 

#### 

#### 

The motivation here is to devise a bound over the number of blocks created in parallel. Since
the block creation rate follows a Poisson process, for an arbitrary blockBcreated at timet, at
mostk(Dmax,δ)additional blocks were created in the time interval[t−Dmax,t+Dmax], with
probability of at least 1 −δ.^10
Observe that blocks created in the intervals[0,t−Dmax)and(t+Dmax,∞), by honest nodes,
belong toB’s past and future sets, respectively. Consequently, in principle,|anticone(B)|≤k
with probability of 1 −δat least. However, an attacker can artificially increaseB’s anticone by
creating blocks that do not reference it and by withholding his blocks so thatBcannot reference
them.

(^10) In more detail: The second multiplicand in the definition ofkbounds the probability that more thankblocks
were created in parallel toBin the time interval[t−Dmax,t+Dmax]. This term is divided by 1 −e−^2 ·Dmax·λ,
which is the probability that at least one block was created during this time, namely,B. Thus, conditioned on the
appearance ofB, at mostkblocks were created during this time interval, with probability of 1 −δat least.


C. Trade-offs

Theorem 5, and the parameterization of PHANTOM in (1), tie betweenk,Dmax,λ, andδ.
Striving for a better performance by modifying one parameter (e.g., increasingλto obtain larger
throughput and more frequent blocks) must be understood and considered against the effect on
all other parameters.

Increased block creation rate.Although the security threshold does not deteriorate asλis
increased,λcannot be increased indefinitely, or otherwise the network becomes congested. The
value ofλ should be set such that nodes that are expected to participate in the system can
support such a throughput. For instance, if nodes are required to maintain a bandwidth of at
least 1MBper second, and blocks are of sizeb= 1MB, then the block creation rate should
be set toλ= 1blocks per second (this is merely a back-of-the-envelope calculation, and in
practice other messages consume the bandwidth as well).

Higher security threshold.Theorem 5 states the security threshold in terms ofδ. Following (1)
we notice that tightening the security threshold – by choosing a lowerδ– requires increasing
k. A largekleads to slow confirmation times, as will be discussed shortly.^11

Larger safety margin.Similarly, ifDmaxis to be increased, one needs to increasekas well
in order to maintain the same security level (represented byδ).
As discussed in Subsection 4.1, it is better to overestimateDand choose a largeDmaxin order
remain on the safe side.^12 Recall that the security of Bitcoin’s chain depends on the assumption
thatD·λ 1 , namely, that w.h.p. at leastDseconds pass between consecutive blocks, so that
forks are rare. Thus, Bitcoin’s large safety margin overDsuppresses its throughput severely as
it requires selecting a very low block rateλ= 1/ 600 (one block per 10 minutes). This is not
the case with PHANTOM’s DAG, as the security of the DAG ordering does not rely on the
assumptionD·λ 1. Therefore, even if we overestimateD, we can still allow for very high
block creation rates while maintaining the same level of security. Consequently, PHANTOM
supports a very large throughput, and does not suffer from a security-scalability tradeoff.
That said, in PHANTOM there is still a tradeoff between a large safety margin and fast
convergence of the protocol. A gross overestimation ofDmax– resulting an increase ink–
would significantly slow down the waiting time for transaction settlement. Thus,Dmaxshould
be set to a reasonable level. In Section 7 we discuss how this tradeoff can be restricted to visible
conflicts only, and how applications such asPaymentscan enjoy very fast confirmation times
nonetheless.

5. PROOF
We now turn to prove that the order converges, and that an attacker (with less than 50%) is
unable to cause reorgs. We restate the theorem from Section 3:

(^11) The advanced reader should notice that although increasingλhas a similar negative effect onk, it has at the
same time a positive effect on confirmation times, and so a certainλwill be optimal as far as confirmation times are
concerned.
(^12) Several blockchain based projects do not do so, and consequently compromise the security threshold of their
system.


Fig. 4: An example of a DAG with an Hourglass blockK. Here, the delay parameter isk= 3. As
in Figure 3, the small circle near each block represents its score. The colouring of the DAG was done
according to Algorithm 1. The greedily selected chain of blocks of highest score is marked with light-blue
filling (note that this chain is not the longest one). BlockKhas the property that all blue blocks are
either inK’s past or in its future (in addition toKitself). It is thus an Hourglass block.

Theorem 5(PHANTOM scales) Given a block creation rateλ > 0 ,δ > 0 , andDmax> 0 , if
Dmaxis equal to or greater than the network’s propagation delay diameterD, then the security
threshold of PHANTOM, parameterized withk(Dmax,δ), is at least 1 / 2 ·(1−δ).
Our proof relies on the following lemma, which states that if some blockBhas the property
that its anticone contains no blue blocks, then all blue blocks in its past precede all blocks
outside its past. We call this the Hourglass property:

Lemma 7. If for someB̂∈G,BLUEk(G)∩anticone

#### (

#### B̂

#### )

```
=∅, then∀B ∈past
```
#### (

#### B̂

#### )

#### ∩

BLUEk(G)and∀C /∈past

#### (

#### B̂

#### )

```
,B≺GC. We then writeB̂∈Hourglass(G).
```
In Figure 4 we provide an example of an Hourglass block. The proof of this lemma is
straightforward from the operation of Algorithm 2:

Proof of Lemma 7.First note that ifBLUEk(G)∩anticone

#### (

#### B̂

#### )

```
=∅thenB̂∈BLUEk(G).
```
Indeed, Algorithm 1 defines a chain,Chn(G) (see Subsection 2.2). This chain necessarily

intersects some block inanticone

#### (

#### B̂

#### )

#### ∪

#### {

#### B̂

#### }

. And the intersection block must become blue

itself, by lines (6-7). Thus,BLUEk(G)∩anticone

#### (

#### B̂

#### )

```
=∅implies thatB̂∈Chn(G)and in
```
particularB̂∈BLUEk(G).
Now, Algorithm 2 pushes a block into the queue only after it has pushed already all blocks
in its past (lines 11 and 12). Therefore,topoqueuepops out blocks according to a topological

order. Now, there are no blue blocks inanticone

#### (

#### B̂

#### )

```
, hence ifCis a blue block then it must
```

belong tofuture

#### (

#### B̂

#### )

#### ∪

#### {

#### B̂

#### }

```
, in which case it must have been pushed to the queue afterB̂
```
was popped (unlessC=B̂). Thus, ifCis blue, any block inpast

#### (

#### B̂

#### )

```
was popped out before
```
C, and added to the ordered listLbefore it (line 8). In particular,B≺GC. On the other hand,
ifCis not blue, it was necessarily pushed into the queue only by belonging to the past set of
some blue blockB′(lines 9-11).B′was popped out afterB̂(unlessB=B̂), from the same
argument as in the previous case. Thus,Cwas pushed into the queue afterB̂was popped out,
which in turn must have occurred afterBwas popped out. In particular, in this case as well,
B≺GC.

Lemma 8.IfB̂is an Hourglass block in a DAGG, thenGinherits the order fromB̂:ord(G)∩

past

#### (

#### B̂

#### )

```
=ord(past
```
#### (

#### B̂

#### )

#### ).

Proof.In the proof of the previous lemma we have shown thatB̂∈Chn(G). By the recursive
operation of Algorithm 1 (lines 6-7), this implies thatGinherits the colouring ofB̂on its past:
BLUEk(G)∩past

#### (

#### B̂

#### )

```
=BLUEk
```
#### (

```
past
```
#### (

#### B̂

#### ))

#### .

```
Now, no block inanticone
```
#### (

#### B̂

#### )

```
is pushed intotopo queuebeforeB̂is popped out, because
```
blocks get pushed in only if they are blue (and no such block exists inanticone

#### (

#### B̂

#### )

```
) or if
```
a blue block in their future was pushed in—and this too only happens afterB̂is popped out.

Since the queue respects the topology, this means that all blocks inpast

#### (

#### B̂

#### )

```
(which must be
```
visited beforeB̂) are popped out before any block outsidepast

#### (

#### B̂

#### )

#### .

```
The fact that blocks inanticone
```
#### (

#### B̂

#### )

```
are not pushed into the queue before all ofpast
```
#### (

#### B̂

#### )

```
is
```
popped out, implies further that the order in which Algorithm 2 pushes blocks inpast

#### (

#### B̂

#### )

```
into
```
and out oftopoqueuedepends only onpast

#### (

#### B̂

#### )

```
, and not on the DAGG. Hence,ord(G)∩
```
past

#### (

#### B̂

#### )

```
=ord(past
```
#### (

#### B̂

#### )

#### ).

The proof of Theorem 5 uses the occurrence of Hourglass blocks to secure all blocks in their
past set.

Proof of Theorem 5. Let δ > 0 and assume that α < 1 / 2 · (1−δ). We need to
prove that ∀t 0 > 0 and B ∈ Gpubt 0 : lim
t 1 →∞

```
Risk(B,t 1 ) = 0, where Risk(B,t 1 ) :=
```
Pr

#### (

```
∃s > t 1 ,∃C∈Gpubs :B≺Gpubt 1 C∧C≺Gpubs B
```
#### )

, and assuming a fraction of at most 1 / 2 −δ
can deviate arbitrarily from the mining protocol.
Fixt 0 andB, and letτ(t 0 )be the first time aftert 0 where an honest blockB̂was published
such thatB̂is an Hourglass block and will forever remain so:

```
τ(t 0 ) := min
```
#### {

```
u≥t 0 :∃B̂∈Gpubu such thatB̂∈∩r≥sHourglass
```
#### (

```
Gpubr
```
#### )}

#### (2)


τ(t 0 ) is a random variable. By Lemma 9 it has finite expectation. In particular,

tlim
1 →∞

```
Pr (τ(t 0 )> t 1 ) = 0(e.g., by Markov’s Inequality).
Assume that such aτ(t 0 )arrives, i.e., that a blockB̂is created and remains permanently
```
an Hourglass block. Then, by Lemma 8, the order between all blocks in past

#### (

#### B̂

#### )

```
never
```
changes, and in particular the order betweenBand any other blockCnever changes. Thus,=
Risk(B,τ(t 0 )) = 0. Therefore, lim
t 1 →∞

```
Pr (τ(t 0 )> t 1 ) = 0implies lim
t 1 →∞
```
```
Risk(B,t 1 ) = 0.
```
Lemma 9.Lett 0 ≥ 0. The expected waiting time forτ(t 0 )(defined in (2)) is finite, and moreover
is upper bounded by a constant that does not depend ont 0.

Proof.LetE(t 0 )denote the event defined by the following conditions:

```
1) Some blockB̂was created at some timeu > t 0 by an honest node (i.e.,B̂∈Gpubu ) and
apart fromB̂no other block was created in the time interval[u−D,u+D].^13
2) For someT 1 , theklatest blocks inBLUEk
```
#### (

```
past
```
#### (

#### B̂

#### ))

```
, denotedLASTk
```
#### (

```
past
```
#### (

#### B̂

#### ))

#### ,

```
were created in the time interval[u−T 1 ,u], and dishonest miners created no blocks in this
time interval.
3) The score of B̂’s chain is forever higher than the score of any chain that does not
pass throughB̂:∀s ≥ u,∀C 1 ,C 2 ∈tips
```
#### (

```
Gpubs
```
#### )

: score(C 1 ) ≥ score(C 2 ) =⇒ B̂ ∈
Chn(past(C 1 )).
Claim 2 and 4 together imply the required result.
The following claim states that the first two conditions in the above definition guarantee that
as long asB̂is blue no block in its anticone is blue:

Claim 1.Assume that for some blockB̂ the first two conditions in the definition ofE(t 0 )
hold true. Then, as long asB̂is blue, all blue blocks haveB̂in their past:∀s≥u,∀B ∈
anticone

#### (

#### B̂

#### )

```
:B /∈BLUEk
```
#### (

```
Gpubs
```
#### )

```
∨B /̂∈BLUEk
```
#### (

```
Gpubs
```
#### )

#### .

Proof of Claim 1.LetB ∈ anticone

#### (

#### B̂

#### )

. IfB /̂ ∈ BLUEk

#### (

```
Gpubs
```
#### )

```
we’re done. Assume
```
therefore thatB̂∈BLUEk

#### (

```
Gpubs
```
#### )

. Any block that was published before timeu−Dbelongs

topast

#### (

#### B̂

#### )

. Any block that was created after timeu+D by an honest node belongs to

future

#### (

#### B̂

#### )

. As honest nodes did not create blocks in the interim, blocks inanticone

#### (

#### B̂

#### )

can only belong to the attacker. Now, since the attacker did not create new blocks while

blocks inLASTk

#### (

```
past
```
#### (

#### B̂

#### ))

```
were created, all attacker blocks that were created in the interval
```
[u−T 1 ,u]and that do not belong to past

#### (

#### B̂

#### )

```
(hence belong to anticone
```
#### (

#### B̂

#### )

```
) have all
```
blocks in LASTk

#### (

```
past
```
#### (

#### B̂

#### ))

```
in their anticone; formally: ∀C ∈ anticone
```
#### (

```
B,Ĝ oracleu
```
#### )

#### :

(^13) We cannot assume, however, that no block was published during this interval, because the attacker might decide
to publish during this interval blocks that he created earlier.


LASTk

#### (

```
past
```
#### (

#### B̂

#### ))

```
⊆anticone
```
#### (

```
C,Goracleu
```
#### )

. Thus, ifB̂ ∈ BLUEk

#### (

```
Gpubs
```
#### )

```
, any block in
```
anticone

#### (

#### B̂

#### )

```
suffers an anticone that is larger thank(it containsLASTk
```
#### (

```
past
```
#### (

#### B̂

#### ))

#### ∪

#### {

#### B̂

#### }

#### )

and is therefore not inBLUEk

#### (

```
Gpubs
```
#### )

. In particular,B /∈BLUEk

#### (

```
Gpubs
```
#### )

#### .

Claim 2.The waiting time for the eventE(t 0 )upper boundsτ(t 0 ).

Proof of Claim 2.The previous claim implies that a blockB̂that satisfies the first two conditions
is an Hourglass block inGpubs. The third condition implies thatB̂forever remains in the chain,
and in particular forever remains blue. It is thus an Hourglass block in all DAGsGpubs (s≥u).

Claim 3.If the first two conditions in the definition ofE(t 0 )hold true then the third one holds
true with a positive probability.

Proof of Claim 3. Part I:We begin by assuming that the attacker did not publish any block in

the time interval[0,u−Dmax); formally, we assume that∀C∈past

#### (

#### B̂

#### )

:C∈honest.
Let us compare the score of the honest chain to that of any chain that excludesB̂. This gap
is captured by the following definition: For a timer > 0 , define

```
X^1 r:= max
B:B/̂∈Chn(B)
```
```
{score(Chn(B))} (3)
```
```
X^2 r:= max
B:B̂∈Chn(B)
```
```
{score(Chn(B))} (4)
```
```
Xr:=Xr^1 −Xr^2. (5)
```
Let us focus first on the evolution of the processXrbetween time 0 and timeu−T 1. We refer to
the leadXu−T 1 that the attacker obtained at the end of this stage as “the premining gap”; see [8].
LetB 1 rbe the argmax ofXr^1 , and letCr^1 be the latest block inGpubr ∩BLUEk(past(Br 1 )),
namely, the latest honest block which is blue in the attacker chain. Recall that for now we are
assuming that all attacker blocks that were premined were kept secret until after timeu−Dmax.
Observe that at mostkblocks that were created by the attacker beforetime

#### (

```
Cr^1
```
#### )

can be in
BLUEk(past(Br 1 ))and can contribute to the score of the attacker’s chain. Thus, between
time

#### (

```
Cr^1
```
#### )

and timeu−T 1 – i.e., during the premining phase – the score of the attacker chain
grows only via the contribution of attacker blocks, and therefore at a rate ofα·λat most.^14
Part II:Let us now consider the growth rate of the honest chain’s score. LetBtmaxbe the tip

of the public chainChn

#### (

```
Gpubt
```
#### )

. The score of this chain is by definition the score of this block.

(^14) Notice that our analysis doesnotassume that the attacker creates its blocks in a single chain. We only claim
that the attacker’s highest scoring chain grows at a rate ofα·λat most, because every attacker block can increase
the attacker’s highest scoring chain by 1 at most. Creating a single chain is indeed the optimal attack on the attacker
side.


Now, by the choice ofk(Dmax,δ)(defined in 1), the probability of an arbitrary honest blockB
having too large of an honest anticone is small:^15

```
Pr
B∼arbitrary honest block inGpubt
```
#### (∣∣

```
∣anticoneh
```
#### (

```
B,Gpubt
```
#### )∣∣

```
∣> k(Dmax,δ)
```
#### )

```
< δ. (6)
```
This is because block creation follows a Poisson process, and because honest blocks that were
createdDmaxseconds before (after)Bbelong to its past (future), hence are not in its anticone.
Since any block that is blue in the honest chain contributes to its score, at any time interval, the
score of the public chain grows at a rate of(1−δ)·(1−α)·λat least. And at mostkhonest
blocks created in the premining stage contributed to the score of the attack chain.
Part III:From Part I and Part II we conclude that the random processXr−kis upper bounded
by the premining race analyzed in [8]. Therein it was shown that the probability distribution
overXu−T 1 −k(the process’s state at the end of the premining stage) is dominated by the
stationary probability distributionπof a reflecting random walk over the non-negative integers
with a bias of(1 1 −−δδ)··(1(1−−αα)) towards negative infinity. The stationary distribution exists because

α≤ 1 / 2 ·(1−δ)<(1−α)·(1−δ).^16
In particular, there is a positive probability that at timeu−T 1 the premining gap,Xu−T 1 , was
less thank, so thatXu−T 1 −k < 0.^17 Between timesu−T 1 anduthe honest network contributed

additionalk+ 1to the score of the public chain, namely, blocksLASTk

#### (

```
past
```
#### (

#### Bˆ

#### ))

#### ∪

#### {

#### B̂

#### }

#### .

During the same time the score of any secret chain(s) did not increase at all, per the second
condition in the definition ofE(t 0 ). Thus,Xu=Xu−T 1 −(k+ 1). And by the first assumption,
Xu+Dmax=Xu. All in all, with some positive probability,Xu+Dmax< 0.
Part IV:Let us turn to look at the evolution of(Xr)r≥u+Dmaxat the second stage. Assume
thatXu+Dmax< 0.

```
In Claim 1 we saw that for anyrsuch thatB̂∈BLUEk
```
#### (

```
Gpubr
```
#### )

```
, all blocks inB̂’s anticone
```
are red inGpubr. This implies that, as long asB̂is blue in the public DAG, only attacker blocks

contribute to the score of the attacker’s chain:B̂∈BLUEk

#### (

```
Gpubr
```
#### )

```
=⇒∀B∈anticone
```
#### (

#### B̂

#### )

#### :

BLUEk(past(B))\future

#### (

#### B̂

#### )

```
=∅(indeed, note that all honest blocks created after time
```
u+Dmaxbelong tofuture

#### (

#### B̂

#### )

```
). Consequently, the attacker’s best chain grows at a rate of
```
α·λat most, as this interval ([u+Dmax,∞)) as well, as long asB̂ ∈ BLUEk

#### (

```
Gpubr
```
#### )

#### .

We have already seen that at any time interval the honest chain’s score grows at a rate of
(1−δ)·(1−α)·λat least. Thus, starting at timeu+Dmax, the race between the honest chain’s

(^15) Below,anticoneh(B,G)denotes all blocks inanticone(B,G)created by honest nodes.
(^16) Note that we have multiplied(1−α)by(1−δ)to account for the rare events where there was a burst of block
creation events and where consequently some honest blocks did not contribute to the score of the public chain. Note
on the other hand that blocks created by honest nodes too do not contribute to the attacker chain’s score, even if they
are red inGpubr. This is because we argued that at mostkblocks that were created by the attacker beforetime
(
Cr^1
)
can be blue inBLUEk(past(Br 1 )), and this is regardless ofC^1 r’s status withinGpubr (we merely used the fact that
Cr^1 was created by an honest node, we didn’t use the assumption thatC^1 ris blue in the honest chain).
(^17) In fact, this probability is decreasing logarithmically askincreases.


score and the attacker chain’s score can be modeled as a random walk over all integers with a
bias of(1 1 −−δδ)·(1·(1−−αα))towards negative infinity.
Since the random walk begins at the negative location Xu+Dmax, this supposedly
implies that with a positive probability the walk will never return to the origin:
Pr (∀r≥u+Dmax:Xr<0) > 0. This in turn implies thatB̂ will forever remain a blue
block (and a chain block for that matter). However, to complete the analysis correctly, some
careful attention is required:
Part V:First, we must account for the fact that not all honest nodes observe all honest blocks
immediately, i.e., thatGvr might be a proper subset ofGpubr. This might give the attacker an
advantage, as he can create blocks, reveal them immediately to honest nodes, and these blocks
do not compete with honest blocks unseen yet by these nodes. This advantage can be accounted
for by assuming that the race begins only after the attacker was givenDmaxadditional seconds
to create blocks while the honest network sat idle; see [8]. This fact does not change our general
conclusion thatPr (∀r≥u+Dmax:Xr<0)> 0 , e.g., because there is a positive probability
that the attacker did not create any block during theseDmaxadditional seconds.
Secondly, observe that the honest chain grows according to a random process that is not
necessarily Poisson. Fortunately, a result from [9] shows that nevertheless we can treat the block
race as if the honest score grows according to a Poisson process, and that this assumption can
only increase the value ofXr; it is thus a worst case analysis.
Part VI:Finally, we alleviate our assumption that the attacker published no block during
the premining phase[0,u−Dmax). Instead, consider any attacker blockC that belongs to

past

#### (

#### B̂

#### )

. IfC∈BLUEk(past(BB))thenCcontributed to the score of the honest chain

which passes throughB̂and maybe to the attacker chain as well, so its existence does not change
the above analysis. Similarly, ifC /∈BLUEk(past(BB)), the fact that it was published has
no consequence whatsoever on the score of the honest chain, and so we can ignore it and apply
the same analysis as if it weren’t published.

Claim 4.The waiting time for the eventE(t 0 )is finite. Moreover, it is upper bounded by a
constant that does not depend ont 0.

Proof of Claim 4.LetB be an arbitrary honest block created intime(()B). The probability
that no other block was created in the interval( [time(B)−Dmax,time(B) +Dmax]is given by

```
1 −e−^2 ·Dmax·λ
```
#### )− 1

#### ·

#### (

#### 1 −

#### ∑∞

```
j=2e
```
```
− 2 ·Dmax·λ·(e−^2 ·Dmax·λ)j
j!
```
#### )

. An arbitrary blockB satisfies the

first condition in the definition ofE(t 0 )with a positive probability.
Given an arbitrary blockBsatisfying the above condition, letT 1 be the creation time of the
earliest block inLASTk(past(B)). The probability that the attacker did not create any block
in the interval[u−T 1 ,u−Dmax]is given by((1−α)·(1−δ))k. In particular, given the first
condition, the second one is satisfied with a positive probability.
Importantly, for any two blocksB 1 andB 2 created aftert 0 and that satisfy|time(B 1 )−
time(B 2 )|> 4 ·Dmax, the satisfaction of the first condition with respect toB 1 is independent
from its satisfaction with respect to B 2. Consequently, the expected waiting time for the
occurrence of a blockB̂ which satisfies the first two conditions in the definition of E(t 0 )


is finite (see, for instance, Chapter 10.11 in [10]). Moreover, while the precise expected
time E[Hourglass(t 0 )] may theoretically depend on t 0 , the above argument shows that
E[Hourglass(t 0 )]< const+ 4·d, whereconstdoes not depend ont 0.
This completes the proof of Claim 4 and of Lemma 9.

Theorem 5 guarantees that the probability of reorg with respect to a given blockBdiminishes:
Risk(B,t 1 )→ 0. However, it does not guarantee anything about the convergence rate, i.e., the
waiting time for at 1 that satisfiesRisk(B,t 1 )<  for someI> 0.^18 Following the analysis
in the proof of Claim 4, the waiting time for an Hourglass block can be upper bounded by a
constantin the order of magnitude ofO

#### (

```
eC·Dmax·λ
```
#### )

, for someC > 0 ; and after such a block is
created,the analysis implies thatRisk(B,t 1 )converges to 0 at an exponential rate, due to the
random walk dynamic.
However, the analysis used in the proof is not tight. It relies on rather infrequent events which
guarantee convergence in a straightforward way, namely, on the occurrence ofHourglassblocks.
In practice the network is likely to converge much faster. Moreover, it can be shown that when
there is no active attack, the convergence time is faster by orders-of-magnitude.

#### 6. VARIANTS

In Section 2 we described the PHANTOM’s greedy algorithm to mark blocks as blue or red
(Algorithm 1). In fact, similar algorithms can provide similar guarantees. We describe several
such variants below and explain the intuition behind them. All these variants can be thought of
as greedy approximations to the Maximumk-cluster SubDAG problem described in Section 1.

A. Choosing the maximizing tip

In our original version of the colouring procedure, we chose the tip which has the highest
score (Algorithm 1, line 6). Instead, the algorithm below chooses the tip for which the score of
the (virtual block of the) current DAG would be highest:

B. Adding blocks to the greedily chosen blue set

Another variant over the previous algorithms is the procedure to mark more blocks as blue,
after the best tip and its blue set were selected. Previously, we required that the candidate block
admit an anticone of size at mostkin the current set. Instead, we can require that its anticone
be counted only inside the originally chosen blue set. Thus, yet another variant is to replace
lines 8-9 with
Observe that the resulting blue set may contain blocks which have an anticone larger thank
within the set.
Yet another viable alternative is to further relax the condition (in lines 8-9) by counting the
anticone of the candidateBonly against the resulting chain of blue blocks,Chn(past(B))∪{B},
whereBis the selected tip:

(^18) presents the risk that the party confirming the transaction is willing to absorb.


Algorithm 3Selection of a blue set

Input:G– a block DAG,k– the propagation parameter
Output:BLUEk(G)– the dense-set ofG
1:functionCALC-BLUE(G,k)
2: ifB==genesisthen
3: return{genesis}
4: forB∈tips(G)do
5: BLUEk(B)←CALC-BLUE(past(B),k)
6: SB←BLUEk(B)∪{B}
7: forC∈anticone(B)in some arbitrary orderdo
8: if|anticone(C)∩SB|≤kthen
9: addCtoSB
10: returnarg max{|SB|:B∈tips(G)}(and break ties arbitrarily)

```
8:if|anticone(C)∩(BLUEk(B)∪{B})|≤kthen
9: addCtoSB
```
Compare this variant to Satoshi’s longest-chain rule, which can be describe as follows: each
block in a chainChnincreases its weight by 1, and we select the highest scoring chain. In
contrast, in our last variant, each block whose gap from a chainChnis at mostkincreases its
weight by 1, and we select the highest scoring chain.

C. Iterative elimination of blocks

We now introduce another variant based on an iterative method common in combinatorial
optimization. The algorithm works as follows: Given a block DAGG, we iteratively eliminate
from the blue set the block with largest blue anticone, and continue doing so until all blue blocks
admit a blue anticone of sizekat most:
We conjecture that Algorithm 1 can be replaced with each of the greedy algorithms described
in this section. To this end, suffice it to repeat the proof of Theorem 5 with respect to these
variants. We suspect that the same proof technique, namely, the use of Hourglass, would prove
useful.

#### 7. CONFIRMATION TIMES

As discussed in Section 5, the convergence rate ofRisk(B,t)is slow, at least theoretically
and under certain circumstances. Recall that the functionRisk(B,t)measures the probability

```
8:if|anticone(C)∩(Chn(past(B))∪{B})|≤kthen
9: addCtoSB
```

Algorithm 4Selection of a blue set

Input:G= (V,E)– a block DAG,k– the propagation parameter
Output:BLUEk(G)– the dense-set ofG
1:functionCALC-BLUE(G,k)
2: BLUEk(G)←V
3: while∃B∈BLUEk(G)with|anticone(B)∩BLUEk(G)|> kdo
4: C←argBmax{|anticone(B)∩BLUEk(G)|}(with arbitrary tie-breaking)
5: removeCfromBLUEk(G)
6: returnBLUEk(G)

that a certain block that did not precedeBat timetwill later come to precede it. Recall further
that throughput this work we used arbitrary topological orderings (over blue blocks). In light if
this, it would be interesting to seek for an ordering rule (over blue blocks) that would converge
faster. We suspect this is not a trivial task, and leave its full investigation to future work.
The primary factor to the fact that PHANTOM cannot guarantee fast confirmation times is
that membership in the blue set takes time to finalize. The waiting time for such finalization can
be further increased if an attacker manages to balance the decision betweenB∈BLUEk(G)
andB /∈BLUEk(G). Observe however that if a certain transactiontx∈Badmits no conflicts
inanticone(B), thentxcan be accepted even before the decision regardingBis finalized.

A. Combining SPECTRE and PHANTOM

SPECTRE is a DAG based protocol that can support large transaction throughput (similarly
toPHANTOM) and very fast confirmations. SPECTRE doesnotoutput a linear order over
blocks. Rather, every blockBadmits a vote regarding the pairwise ordering of any two blocks
CandD, and the output is the majority vote regarding each pair. In SPECTRE, cycles of the
sort “BprecedesC,CprecedesD,DprecedesB” may form.
Furthermore, if an active balancing attack is taking place, forsomepairs of blocks(B,C)the
SPECTRE relation might not become robust (in which caseRisk(B,t)→ 0 is not guaranteed).
Instead, a published blockBis guaranteed to robustly precede any blockCthat was published
later thanB, unlessCwas published shortly afterB. We call this propertyWeak Liveness. In
the context of the Payments application, this fact can only harm a user that signed and published
two conflicting payments at approximately the same time.^19
SincePHANTOM does guarantee (strong) Liveness and a liner ordering, it is interesting
to inquire whether we can enjoy the best of both worlds. We provide below a partial answer to
this question.
Consider the following procedure: Given a blockDAGG,
1) mark blocks as blue or red according to PHANTOM’s colouring procedure

(^19) In contrast, usually any user can engage with a smart contract and introduce conflicts inputs. Thus, Weak Liveness
might potentially harm the usability of SPECTRE to the Smart Contracts application.


2) run SPECTRE on the subDAGBLUEk(G); this determines the pairwise ordering between
any twoblueblocksBandC
3) for any blue blockBand red blockC, ifC∈past(B)then determine thatCprecedes
B, otherwise determine thatBprecedesC
4) decide the pairwise ordering of any two red blocks in some arbitrary way that respects the
topology (B∈past(C)⇒BprecedesC)
Intuitively, we run SPECTRE on the set of blue blocks (as determined by PHANTOM), and
complemented the pairwise ordering in a way that both penalizes red blocks and respects the
topology. We argue that this protocol enjoys SPECTRE’s fast confirmation times and at the
same time inherits the (regular) Liveness property of PHANTOM. To see the latter, observe that
a Hourglass block would have a similar effect in SPECTRE—all blocks in its future will vote
according to its vote. In particular, the pairwise relation of all previous blocks becomes as robust
as the Hourglass block.
Note that this incremental improvement over vanilla SPECTRE is only possible because
we allowed the protocol to assume something on the network’s topology, namely, that the
communication delay diameter is upper bounded byDmax.

B. Summary

In summary, it is possible to achieve both fast confirmation times and Liveness by combining
PHANTOM and SPECTRE. It is of yet unclear whether we can further achieve a linear ordering
without compromising the fast confirmation times. Hopefully, we will provide answers to these
questions in future work.

#### REFERENCES

[1] Ittay Eyal, Adem Efe Gencer, Emin Gun Sirer, and Robbert Van Renesse. Bitcoin-ng: A scalable blockchain ̈
protocol. In13th USENIX Symposium on Networked Systems Design and Implementation (NSDI 16), pages
45–59, 2016.
[2] Michael R Garey and David S Johnson.Computers and intractability, volume 29. wh freeman New York, 2002.
[3] Aggelos Kiayias and Giorgos Panagiotakos. On trees, chains and fast transactions in the blockchain. Cryptology
ePrint Archive, Report 2016/545, 2016.
[4] Eleftherios Kokoris Kogias, Philipp Jovanovic, Nicolas Gailly, Ismail Khoffi, Linus Gasser, and Bryan Ford.
Enhancing bitcoin security and performance with strong consistency via collective signing. In25th USENIX
Security Symposium (USENIX Security 16), pages 279–296. USENIX Association, 2016.
[5] Yoad Lewenberg, Yonatan Sompolinsky, and Aviv Zohar. Inclusive block chain protocols. InInternational
Conference on Financial Cryptography and Data Security, pages 528–547. Springer, 2015.
[6] Rafael Pass and Elaine Shi. Hybrid consensus: Efficient consensus in the permissionless model, 2016.
[7] Joseph Poon and Thaddeus Dryja. The bitcoin lightning network: Scalable off-chain instant payments.Technical
Report (draft), 2015.
[8] Yonatan Sompolinsky, Yoad Lewenberg, and Aviv Zohar. Spectre: A fast and scalable cryptocurrency protocol.
IACR Cryptology ePrint Archive, 2016:1159, 2016.
[9] Yonatan Sompolinsky and Aviv Zohar. Secure high-rate transaction processing in bitcoin. InInternational
Conference on Financial Cryptography and Data Security, pages 507–527. Springer, 2015.
[10] David Williams.Probability with martingales. Cambridge university press, 1991.

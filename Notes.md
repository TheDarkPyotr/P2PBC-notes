## Content Distribution Network (CDN)
A **Content Distribution Network (CDN)** is a distributed network of servers that are designed to deliver web content to end-users with high availability and performance. The CDN works by caching web content, such as images, videos, and static HTML pages, on servers located around the world. When a user requests a particular piece of content, the CDN will route the request to the nearest server that has a cached copy of the content, reducing latency and improving response time.

CDNs typically consist of three main components: **edge servers, a caching infrastructure, and a global network**. Edge servers are distributed geographically to bring the content as close to the end-users as possible. The caching infrastructure is used to store and manage the content, and the global network is responsible for routing traffic to the appropriate edge server.

As an example, let's consider a popular e-commerce website that sells products globally. The website's static content, such as images, product descriptions, and videos, can be cached on a CDN. When a user requests a product page, the CDN will route the request to the nearest edge server that has a cached copy of the page. This reduces the latency and improves the user experience. The CDN can also provide security features like DDoS protection and SSL/TLS encryption to improve website security.

Some popular CDN providers include Amazon CloudFront, Akamai, Cloudflare, and Fastly. These providers offer a wide range of features and configurations to meet the needs of various websites and applications.

# BitTorrent Protocol

BitTorrent is a **content distribution network**, not based entirely on P2P paradigm. The problem is how distribute the contents to hundres of thousends of simultaneous users.
The *efficient content distribution* is developed using **file swarming**, later discussed, which is based on the idea that *a peer makes whatever portion of the file that is downloaded immediately available for sharing*.
Some issues have to been addressed:
- Flash crowd phenomena: large number of request in small time period implies the ability to manage a high degree of unexpected traffic
- CDN (Content Delivery Network): data and services replication on different mirror servers to optimize bandwidth usage

The basic idea is to import the CDN mechanism in a P2P environment, without the intervention of a centralized server: differently from a P2P system, it does not perform all the functionsof a typical P2P system like *searching*. 

The initial file reques are served by a centralized server but further requests served by peers which have already received and replicated the files. 
The server distribute two further pieces: in parallel, so previous pieces are distributed. When all pieces are distributed among peers, the server is no more inolved in the file dsitribution. 
The distribution and recomposition on request of those piaces of file allows to avoid an unexpected and high load on a single node/server. 
![[Pasted image 20230311093749.png]]
In summary, more nodes can serve thc ontent, not only one server: there is the need to detect which node is currently providing the content by introducing the **tracker** that taking trace of who is currently providing the content. In the example, *JOE* connects to the tracker announcing the content he store and the tracker update the information *"JOE provide the content x"*. 
If *BOB* want to download the red content:
1. Open a connection ti the tracker and learn that *JOE* is providing the content
2. Establish a direct connection with *JOE*
3. *JOE* sends the content, from now on the distributon of the file goes on the directly between peers
4. *BOB* then announces the tracker that is can also provide the red content
![[Pasted image 20230311094106.png | 800]]

### Glossary
- **Descriptor**: The file descriptor `.torrent` are managed by centralized server and not fully by the nodes. The descriptor includes the reference to a *tracker*.
- **Tracker**: entity which coordinates the peers sharing the file
- **Swarm**: set of peers collaborating to the distribution of the same file coordinate by the same tracker
- **Seeder**: peer which own all the parts of the file, having reconsturcted the entire content. At starting time a single seeder S exists
- **Leecher**: is a peer which has some part of no part of the file and downloads the file from the seeders and/or from other leechers. 
![[Pasted image 20230311094156.png]]

## Protocol overview
![[Pasted image 20230311094220.png | 700]]

In the pictued schema, the **peer Seeder** is going to share a file by:
1. Upload the `.torrent` on a Torrent Server
2. Opens a connection to the **tracker** and informs it of iwn own existence:for the moment, it's the only peer which own the file
The **peer Peer** is going to download the file by:
3. Retrieve the *file description (`.torrent`)* and opens it through the BitTorrent client
4. Open a conenction to the tracker and informs it of itw own existence and recieve from the tracker a list of **peers of the swarm**
5. (and step `6`): opens a set of connections with other peers of the **Swarm** so ask other peer about the parts they own, declaring its interest in some part of the file. Continues by exchaing the information with the peers in the swarm.
If **Peer** remains online when it has finished the file download, it goes on distributing the file, so becoming a **Seeder**. 

Here a summarized schema on the process explained:
![[Pasted image 20230311094713.png]]

#toreview n.b: *chocking alg* to avoid free riding prob. (see later lectures)

### Pieces and suboieces (blocks)
The content is split into chucks called **pieces** of a choosen dimension between $256KB$ and $2MB$: later those pieces are divided into **blocks** of $16KB$. 
The hash of **each piece** in $SHA-1$ is stored in the `.torrent` file descriptor: pieces are the *smaller unit of retransmission* so when the pieces are retrieved from seeders, the hash in file descriptor and the computed one are compared to verify the correctness of the piece to determine if retransmission is needed. 
![[Pasted image 20230311094937.png | 700]]

## Publishing
Torrent publishing is a process of distributing large files over the internet using a peer-to-peer (P2P) network. The process involves breaking the file into small pieces and distributing them to multiple users or peers. Each peer downloads and shares small pieces of the file with other peers, allowing for faster and more efficient downloading.

To publish a file on a torrent network, the following actions are typically taken:

1.  Create a Torrent File: The first step is to create a small metadata file known as a torrent file. The torrent file contains information about the file being distributed, such as its name, size, and checksums of each piece. It also contains a list of tracker servers that are used to coordinate the sharing of pieces between peers.
	![[Pasted image 20230311095142.png]]
    
2.  Upload the Torrent File: The torrent file is then uploaded to a torrent website or tracker server. The tracker server acts as a central coordinator between peers, keeping track of which peers have which pieces of the file.
    
3.  Share the Torrent File: Once the torrent file is uploaded, it can be shared with others through various means such as email, social media, or file sharing platforms. Peers can then download the torrent file and open it using a torrent client.
    
4.  Download the File: After opening the torrent file with a torrent client, the client connects to the tracker server and begins downloading the file by requesting pieces from other peers. As pieces are downloaded, the torrent client verifies their integrity using the checksums in the torrent file.
    
5.  Seed the File: Once the file has been fully downloaded, the torrent client can continue to upload or "seed" the file to other peers. This helps to improve the download speeds of other peers and contributes to the overall health of the torrent network.

The `.torrent` file descriptor are automatically generated by ad-hoc tools like *maketorrent* or by the BItTorrent client: some informations are required by the client like set-up info, address of a known tracker etc. To serialize the content it's used the **Bencode**: it's a serialization code similar to JSON but only used in `.torrent` files. The bencode serialization provide only 4 datatypes: `String, Integer, List, Dictionaries` and a set of simple rules, here listed:

1.  Strings: A string in Bencode is a sequence of bytes. It is encoded as follows: `<length>:<string>`. Here, `<length>` is the length of the string in bytes, and `<string>` is the actual string.

For example, the string "hello" would be encoded as `5:hello`.

2.  Integers: An integer in Bencode is a number encoded as follows: `i<integer>e`. Here, `<integer>` is the actual integer value.

For example, the integer 42 would be encoded as `i42e`.

3.  Lists: A list in Bencode is a collection of other Bencode data types, including strings, integers, lists, and dictionaries. It is encoded as follows: `l<item1><item2>...<itemN>e`. Here, `<item1>...<itemN>` are the individual items in the list.

For example, a list containing the integers 1, 2, and 3 would be encoded as `li1ei2ei3ee`.

4.  Dictionaries: A dictionary in Bencode is a collection of key-value pairs, where the keys are strings and the values can be any Bencode data type. It is encoded as follows: `d<key1><value1>...<keyN><valueN>e`. Here, `<key1>...<keyN>` are the keys in the dictionary, and `<value1>...<valueN>` are their corresponding values.

For example, a dictionary containing the key "name" with the value "Alice" would be encoded as `d4:name5:Alicee`.


### Peer Bootstrap
In the boostrapping phase, a peer downloads the `.torrent` of the file it wants to download from the tracker: the peer retrieve the tracker's `URL` and connects to it by issuing an `HTTP GET` request. Then sends to the tracker the informationa bout its identity (*identifier, port, etc*), **the number of the peers of the swarm it needs** and other minor informations.

The **tracker** return a *random list* of **50 peers** already in the torrent, providing information for each peer like *peer ID, peer IP, peer Port*. 
**Tracker** are used for boostrap and as a **certification authority** but are not inolved in file distribution itself: if a `.torrent` file is certified by the server, there is no way to corrupt the torrent because each $SHA-1$ in the `.torrent` is used from peers to verify the integrity of the pieces received from the P2P Network.
A peer connects to some peers returned by the tracker open at most *40* outgoing connections so remaining peers keps as a pool of peers to connect if needed. 

A node uses only part of its bandwidth to boostrapping while another part is left free to accept incoming connections. 

#### Why blocks?: Pipelining
The idea to divide **pieces** into **blocks** derive from the fact that the BitTorrent protocol works on TCP Protocol so the underlying **Slow Start Mechanism** of TCP enter in action: if data are not regularly sent, TCP  brings the peers connection to a slower speed than normal. 
It's crucial to have always something to transfer on a given connection to sustain the transfer rate constant.
Each peer for each connection have always some blocks (*typically 5 blocks*) ready to sent for each connection.

The mechanism that allows to determine the order in which the pieces are selected is called **Peer Selection**: this mechanism is fundamental because an inefficient policy may end up in a situation where each peer has all identical set of easily avaiable pieces and none of the missing ones.
Several policies that provide efficiency exists:

#### 1. Strict Priority
![[Pasted image 20230311101310.png]]
if a subpiece of a piece has been required, all the other subpieces of the same piece are required before requiring another block. It favors a fast assemblage of the pieces: only the complete pieces may be exchanged with other peers so each peer tries to assemble in the fastest way complete pieces. 

#### 2. Rarest First
Each peer knows the pieces owned by the others so it have a **local view** (*by gossip algorithms*) of the **availability of each piece**: this allows to form the **Rarest Piece Set**  that contains the pieces with the *minimal availability*. 
The algorithm select a piece among the rarest, as pictured:
![[Pasted image 20230311101518.png | 600]]


![[Pasted image 20230311101953.png | 700]]
Imagine that there are a set of peers, the edges indicates the ownership of a piace: the pink piece is the rarest. So the upload rate is higher of the seed and because the pink piece is highly requested, in exchange everyone want to download from us so we can download faster from others. It's likely that the peers that own the rarest piace will be insweted in the upload list of several peers. 

One **advantage** is that it can increase download speeds because the more peers that hold a piece of the file, the faster the download. P2P networks can also prevent rarest piece missing, which can make it impossible to assemble the entire file if a seeder disconnects from the swarm. Additionally, using a P2P network can decrease pressure on the seeder because new downloaders can download parts of the file from other peers. This guarantees that the entire file is replicated and the download load on the seeder is decreased.


#### 3. Random First Piece
Initially, a lecher does not own any piece and cannot offer anything to the other peers of the swarm: it is important that a **peer acquires a piece as soon as possible, to start the negotiation** of the pieces with other peers (***unchoking algorithm***). 
In the *rarest first* politics is not initially exploited because the download of a rare peer may be slow the process, because:
1. less piece availability, larger request 
2. larger probability to choose a slow peer 
3. sharing of upload bandwidth of the owner of the rare piece

This introduces the *Random First Piece Policy* at least for the **first 4 pieces to download**: they are chosen at random. This allows a **fast starting** and, after that, the *rarest first policy* is exploited. 

#### 4. Endgame
The last piece is downloaded using the **Endgame** policy: in the final phase the peers are characterized by low bandwidth, slow down when transfer is going to end.

The key idea of this policy is that the last pieces of the file to download are required to **all the peers that own the file**: to avoid bandwidth waste, when the piece is received from the peer which has requested it, the download started in parallel are cancelled.
There is only a little waste of bandwidth, since the end game is executed for a small period of time.

![[Pasted image 20230311102701.png | 800]]

## Free riders problem
The *free riders* in a general definition are "*individuals hiding his/her preference for a common good, so avoiding to pay its price and giving to others the burden to pay the good*". So if there are other individuals interested in the common good, the free rider is aware that he/she can benefit of the common good without paying for it.

In BitTorrent, the *free riders* are **peers that do not put their bandwidth at disposal of the community**. The solution to this problem in P2P context is complex due to an absence of centralized entity that may control the nodes and it's not possible to impose a given behaviour to the BitTorrent clients, because it's possible to modify the of the client by reverse engineering. 
The used approach is to based on a *dynamic monitoring of the connection*: the **reprocity approach** states that *a client obtains a good service if and only if it gives a good service to the community*, implemented by ***Choking/Unchoking Algorithm***. 

### Choking Algorithm
Each peer, periodially, *evaluate its neighbour* 's download speed in the previous round and decides which neighbours it's going to *"choke"*. 
The decision on which peers to *unchock* is taken every 10 seconds by each peer ans it's based on:
1. The download rate of the last $X$ seconds
Because the use of TCP slow-start mechanism, rapidly choking and unchocking is bas thus the timeframe of 10 second is suitable to match the timeframe of the slow-start mechanism. 
If our upload rate is high more peers will allow us to download from them. This means that **we can get a higher download rate if we are a good uploader. This is the most important feature of the BitTorrent protocol**.

Here is a pictured scenario:
![[Pasted image 20230311103905.png]]

BitTorrent divides the time in **round** (*generally 10 seconds*): for each round, decides **to whom it wants to send data (upload)**. Each connection with a peer in the neighbor set is controlled by the following variables:
- `interested/uninterested`: want/do not want a piece from you? (download) 
- `chocked/unchocked`: want/do not want to send data to you? (upload)
The connections are **bidirectional** so each connection has associated a pair of the previous listed variables. 

Each peer maintains for each remote peer it is connected the following state:
- ***am_choking***: the local peer is choking the remote peer. it does not want to send data to the remote peer because the local peer is not satisfied about how this peer collaborates 
- ***am_interested***: the local peer is interested in at least one piece on the remote peer
- ***peer_choking***: the remote peer is choking the local peer.  The remote peer does not want to send data to the local peer, because it is not satisfied by the collaboration with the local peer
- ***peer_interested***: the remote peer is interested in at least one piece of the local peer

So the *local peer* can receive data from the *remoe peer* if:
1. the local peer is **interested** in the remote peer, and
2. the remote peer **unchoked** the local peer
![[Pasted image 20230311104352.png | 700]]

### Optimistic Unchoking
The previous algorithm does not take into account the newly entered peers because initially have nothing to trade and not obtain any interesting pieces. To overcome this limitation, BItTorrent adopt **optimistic unchoking**: one random selected peer is unchocked so the current download rate from that peer is ignoreted and allow newcomers to download resources.

## BitTorrent at a glance
![[Pasted image 20230311104808.png]]
Horizontal arrows represents the exchange of messages in P2P fashion (as discussed till now). 

### Two-way handshake
Once a TCP connection is established with one of the peer received by the tracker, a client perform a two-way handshake. The header of the message contains:
![[Pasted image 20230311105046.png]]

- name length:”0x13” 
- protocol name: “BitTorrent protocol” 
- “Reserved”: used to signal extensions to the basic protocol. \X00 if no extension 
- SHA-1: hash of the torrent file's infohash that contains the hash of the content the peer is looking for.
- peerID: the client random ID

The peer which receives a TCP connection request **may refuse the connection** based on two checks:
- if the hash does not correspond to **any hash of the files it is participating to download**. The tracker has returned the hash of each pieces so if the hash is not present, the connection is not allowed.
- if the ID of the peer does not correspond to those received by the tracker so the ID is not present in the *swarm* group (*returned by the tracker*,)
![[Pasted image 20230311105313.png | 600]]

### PEER WIRE PROTOCOL
Then, after the handshake, the first thing that peer exchange is the `<bitfield>`: it's a bitmask in which if bit $i$ is set to 1 the peer has the piece $i$, 0 otherwise. 
![[Pasted image 20230311105716.png | 700]]
The *message exchanged* are:
- **Interested**: sent from a peer A to another peer B.  It indicates that A is interested in any of the B’s pieces and contains the piece index. 
- **Unchoke /(Choke)**: sent from B to A if B unchokes A. It notifies that the request of download coming from A is accepted.
- **`Request<index, begin, length>`**: this message is sent from peer A to peer B to *request a subpiece (block)* with `<index>` index and starting with an offset `<begin>` within the piece, of length `<length>`. A requested can be sent **only after receiving Unchoke message from B**.
- **`Piece<index><begin><block>`**: only one message is used to *send pieces*. The message is sent from peer A to B to send a block to peer B: the block of index `<index>`, starting with offset begin within the piece and payload  `<block>`.  
- **Have**: notifies that a new piece is completely downloaded and is available for sharing.  It contains information useful for monitoring the swarm: who has what. When a peer receives a Have message, it updates its view of the swarm.
- **Not Interested** indicates that the sender is not interested in that piece anymore. May be sent after receiving a HAVE message
- **Cancel**: sent from a peer to another to indicate that it already got a piece and it is not interested in it anymore. It's sed in end game mode only: sent from peer A to peer B to cancel a request already sent to peer;  for the piece with index starting with an offset within the piece of a length.

## BitTorrent Mainline DHT

Bittorrent Mainline DHT allows to **decentralizes the tracker services** that usually represent the bottleneck. This allows to o provide a “trackerless” peer discovery mechanism to locate peers belonging to a swarm by asking to the DHT, instead of the tracker.

Each node of the DHT **stores a part of the tracker information**: now each peer implement two different functionalities:
1. a client/server listening on a TCP port that implements the BitTorrent wire protocol
2. a client/server listening on a UDP port implementing the distributed hash table protocol.
So the DHT plays the role of the tracker and **stores the content's infohash (*key*)** and **the list of the peers in the swarm (*value*)**. 

The protocol messages includes basic messages like:
- `PING`: probe a node’s availability or announce one's existence
- `GET_PEERS(H)`: look for peers belonging to the swarm for the content with InfoHash H. The contacted node may
	- store peers for H, replies with a PEER message containing them 
	- have no info for H: replies with the identifiers of the 8 peers closest to H
- `ANNOUNCE_PEER`:a peer announces it belongs to a swarm.
- `FIND_PEER (target_id)`: request for nodes which are close to the node with node ID target_id


### Example
![[Pasted image 20230311111416.png]]
For example Alice wants to download the piece of content stored by Bob and identified by id=14. Alice only knows the peers 6,7,13.
Following the [[Chord]] idea of contacting the clostest node (with XOR distance to the content as in [[Kademlia]]), the clostest to the result node is peer 13.
Now 13 does not known who has piece 14, so it sends back to Alice the clostest peer to it, being peer 15.
Peer 15 knows that peer 4 has the content, and sends the contact information to Alice, which can then use the BitTorrent TCP protocol to exchange the content with peer 4.

After downloading the piece, Alice sends(broadcasts) an **Announce** message to tell all the peers it stores piece 14 and can exchange it.

![[Pasted image 20230311111432.png]]
The announce message has key 14 and value (11(peerid),info=(ipaddr,port)).
All the other nodes will add it to their "tracker"informations.

The internet uses [[NAT]] to face the scarcity of IP address, this is a problem for Peer to Peer network.



# Cryptographic toolbox for DHT and Blockchains
The following tools are used for the development of DHT and blockchains: they include hash function, cryptographic hash functions and digital signatures. More advanced tools are accumulators and zero-knowledge principles. 

# Hash function
It's a function that convrts a binaty string of arbitrary length to a binary string of fixed length. The **input** is described by its length counted in bits, normally a maximum length but even zero length is permitted.
The **output** represent the compression and usually have a fixed length among 128-160-256-512 bit. 

There are noticeable differences among non-cryptographic and cryptographic one hashing function: in case of 8-bit block parity (*as shown in picture*), it's easy to find a collision by inverting any even numer of bits in $m$ that are in the same column and the parity will not change. 
![[Pasted image 20230315112141.png | 700]]

One of the main property desiderable from cryptographic hash function is the minimum probability to find a value that determine a collision. 

Another example is the **CRC - CyclIc Redundancy Check**: the CRC is the remainder in a long division calculation. In the past has been used to detect burst errors. 

The **hash collision** are inevitable since the codomain of the hasing function is smaller than the domain. It follows the **Pigeon Principle** for which if $n$ items are put into $m$ containrs, with $n \ge m$ then at least one container must contain more than one item. 
The **hash security** implies that it's very hard to *find* collisions because would require a huge amount of computational power. 
To bound the probability of having a collision, it's necessary to determine the maximum number of guesses to certainl find a collision by a brute force attack. 

![[Pasted image 20230315113050.png]]
Using a brute force approach, picking $2^{256} + 1$ distinct values in the domain and compute the hashes of each of them, hecking if any two output are equal: for the pigeon principle at lest a collision will be founded.
The maximum number of guesses required is:
- $O(2^{n})$ time complexity
- O(1)
- n = len(H)

The birthday paradox show the correlation between the size of the domain and the maximum number of samples required: specific for the paradox, in a room with n people, the value of n to have the probability that two share a birthday becomes larger than 50% under the hypothesis of:
- a year of 365 days
- all days equally probable
- -> n = 23 ($\sqrt(365)$)


The paradox show a connection with hashing function H with $n$ possible outputs: there are $2^{n}$ possible different hashes.
If H is applied to k random inputs, what must be the value of k so that the prob. that at least a pair of $x,y$ satisfy $H(y) = H(x) = 0.5$?
If hashing about $2^{n/2}$ vìrandom values then you expect to find a collision because $\sqrt(2^{n}) = 2^{n/2}$ .

### Cryptographic hash functions
Must respect other property thhan non-cryptographic one:
1. **Adversarial collision resistance**:
2. **One-way function (Pre-image resistance)**
3. **Second pre-image resistance**: given $M$ and thus $h=H(M)$ it's hard to find another value $M'$ that $H(M') = h$. 
	![[Pasted image 20230315114245.png]]
4. **Hiding**: 
5. **Puzzle-friendliness**: 


#### Attacking hash functions
The approaches followed for attacking an hash function:
- Cryptanalysis involves exploiting logical weaknesses in the algorithm
- Performing *brute force* attack: in cryptography it corresponds to an exhaustive search that might be used when it's not possible exploit other mechanism. 


### Hiding
To avoid that the attacker is able to guess the input domain, a solution is to pick a **random integer R** of 256 bits from a distribution with high min entropy (*no particular value is more likely then others*) and append R to the original input. The input space becomes extremely hard to enumerate ($2^{257}$ possibilities).
The property can be formalized by:
*a hash function $H$ is said to be hiding if wehn a secret value R is chosen froma probability distribution that has **high min-entropy**  then given $H(R || x)$ 
it is infeasible to find $x$. -> $x$ is the **nounce** in blockchain corresponding mechanism.*

## Commitment scheme
It allows to commit to a value and reveal it later: seal a value in an envelope and put that envelope out on the table where everyone can see it. 
![[Pasted image 20230315115544.png | 750]]
#slides






# Data structure for DHT & Blockchains

In the following lecture the following data structure will be addressed (*also used for DHT and IPFS*):
1. Hash Pointer
2. Bloom filters
3. Merkle trees
4. Trie
5. Patricia Merkle Trie

## 1. Hash Pointers
An *hash pointer* is a pointer to where the information is stored using a cryptographic hash on the information itself. Starting from a given pointer we can ask to get the information back or verify that it hasn't changed. 
![[Pasted image 20230317155927.png]]

The key idea is to build data structures with hash pointers embedded: in the **blockchain** view, a list is linked with hash pointers by computing the hash to a block, hash again the entire block with its own hash pointer:
![[Pasted image 20230317160103.png]]

Is someone *tampers* the $k^{th}$ block of the chain, the hash of block $k+1$ is not going to match up because the **hash is collision resistant** so an adversary cannot tamper the data so that its hash is the same of the data before tampering (*as shown in the following picture*). 
![[Pasted image 20230317160241.png]]

Some **use cases** are useful applying this schema, like:
- **tamper-evident log**, a basic data structure in Bitcoin and Ethereum 
- in PoW-based blockchain, the block contains also the proof that PoW has been succesfully executed 
- if data is changed, PoW has to be re-executed for all the blocks 
- computationally infeasible

## 2. Bloom filter
Consider the set $S = \{s_{1}, s_{2}, ..., s_{n}\}$ of $n$ elements with $n$ very large. Usually we want to define an efficient data structure that support queries like "$k$ is an element of $S$" also called as **membership queries**: a given function $f$ returns value $true$ or $false$ according to the presence of $k$ in the given set $S$. 

The Bloom filter differently allows to provide an *approximate solution* to the set membership problem: 
![[Pasted image 20230317160722.png]]
The problem is translated into choosing a representation of elements in $S$ such that the result of the qery is computed *efficiently* and the space for the representaiton of the element is reduced. The result may be approximated to save space and arises the possibility of returning  **false positives**. 

#### Building a Bloom Filter
Given a set  $S = \{s_{1}, s_{2}, ..., s_{n}\}$ of $n$ elements, a vector $B$ of $m >> n$ (*generally $m > n*k$*) bits, $b_{i} \in \{0,1\}$. Choose $k$ ***independent hash functions*** $h_{1},...,h_{k}$ such that: 
	$\forall i. h_{i}: S \rightarrow [1..m]$, $h_{i}$ returns a value *uniformely distributed* in the range $[1..m]$ .
Construct the filter $B[1..m]$ such thath:
	$\forall x \in S: B[h_{j}(x)] = 1, \forall j = 1,2, ..., k$
Note that a bit in $B$ may be target for more than 1 element:
![[Pasted image 20230317161513.png]]
#### Look up
To verify if $y$ belongs to the set $S$ mapped on the bloom filter, apply $k$ hash functions to $y$ so we can determine that $y \in S$ if $B[h_{i}(y)]=1, \forall i = 1...k$: if at least a bit is equal to $0$, the element *does not belong to the set*. 
The problem of the **false positives** can occur if all bits are positive because another element or some combination of other elements could have set the same bits. 

###### Probability of false positives
Compute the probability that, after all the $n$ elements are mapped to the vector, a specificbit of the filter (*of size $m$*) has still value $0$:
		$p^{'} = (1 - \frac{1}{m})^{kn} \approx e^{\frac{-kn}{m}}$
The approximation is derived from the definition of $e$:
		$\lim_{x\to\infty} (1-\frac{1}{x})^{-x} = e$
A percentage of $e^{\frac{-kn}{m}}$ bits are $0$, after its construction. If we consider an element *not belonging* to the set, by applying the $k$ functions, a false positive is obtained if all the $k$ hash functions applied to that element return a value equals to $1$ and this happen with probability of:
			$(1 - e^{\frac{-kn}{m}})^{k}$
The **probability of false positives** depends on two parameters:
	- $\frac{m}{n}$: number of bits exploited for each element of the set
	- $k$: number of hash functions
If the first parameter is set:
- decreasing k increases the number of 0 and hence the probability to have a false positive should decrease,
- increasing k increases the precision of the method. Hence the probability of false positive should decrease.
Fixed the ratio $\frac{m}{n}$, the probability of false positives first decrease, then increases, when considering increasing values of $k$.
![[Pasted image 20230317163638.png]]
Let us now suppose that $k$ is fixed, the probability of false positives *exponentially decreases* when $m$ increases (*$m$ number of bits in the filter*).
For ***low values** of $\frac{m}{n}$* (a few bits for each element), the probablity is higher for large values of $k$:
![[Pasted image 20230317163622.png]]

![[Pasted image 20230317163655.png]]
A bloom filter becomes effective when $m =  c*n$, with $c$ constant value. 

##### Other operations
- ***Union***:-Given two Bloom Filters, B1 e B2 representing, respectively, the set S1 and S2 through the same number of bits and the same number of hash functions, the Bloom filter representing $S1 \cup S2$ is obtained by computing the bitwise OR bit of B1 and B2
- ***Delete***: note that it is not possible to set to $0$ all the elements indexed by the output of the hash functions, because of the conflicts. This operation is supported in a different version called *Counting Bloom Filter* in which each entry is a counter (*instead of a single bit*): at insertion time the counter in incremented while it's decremented at deletion.
- ***Intersection***: given two Bloom Filters B1 and B2 representing respectively, the sets S1 and S2 through the same number of bits and the same number of hash functions, the intersection of the Bloom filters obtained by computing the bitwise and of B1 and B2 and approximates $S1  \cap S2$. 

##### Bloom Filter usages
The set $S$ mapped on a bloom filter may be a set of ***Bitcoin addresses***: light-weight mobile nodes do not store the blockchain and build only the bloom filter with the addresses they are interested in. They send the $BF$ to a full node allowing bandwidth saving and privacy. The full node receives  a block $B$ of the blockhain and uses $BF$ to check if some address of $B$ belongs to $BF$. 
![[Pasted image 20230317164352.png]]
  In ***Ethereum*** is used to summarize events generated by a smart contract by finding all the tokens sold by a specific user in a block of 500 transanction. Instead of parsing all transaction, query a Bloom Filter (*or ***log bloom***) in the header of a block for the presence of that users and perform a search in the block only if a match is found.


## 3. Merkle Hash Tree
A merkle hash tree it's a data structure that allows to *summarize* a big quantity of data with the goal of verifying the correctness of the content. It's a **complete binary tree** of hashes built starting from a initial set of data:
- $i^{th}$ leaf stores the has of $h_{i}$ of $f_{i}$ (*$i^{th}$ *file*)
- an internal node contains the concatenations of the hashes of the sons of the node
- the last has tored in the root is called ***Merkle Root Hash***. 
 ![[Pasted image 20230317164834.png]]
Suppose have $n=8$ files $f_{1}...f_{8}$ and a collision resistant hash function $H$: go on hashing every two adjacent hashes by applying $H(x,y) = H(x|y)$  where $|$ indicate concatenation. 
A ***Collision-Resistant Hash Function*** for **Merkle Hash Tree (MHT)** takes $n$ input $(x_{1}...x_{n})$ and outputs a *Merkle root hash* $h = MHT(x_{1},...,x_{n})$.
The $MHT$ has an important proof-property: if a *verifier (Alice)* only known the Merkle root has $h$, the *prover (Bob)* can give Alice one of the values $x_{i}$ and convince ALice that is was the $i^{th}$ input used to compute $h$: to convince her, Bob gives Alice an associated ***Merkle proof*** without shoging all the other inputs. So if a merkle proof says that $x_{i}$ was the $t^{th}$ input used to computed $h$, no attacker can come up with another Merkle proof that says a differnet $x_{i} \neq x_{j}$ was the $i^{th}$ input used in $MHT$. 

#### Merkle Proof
The key idea is to have a given file $f_{i}$ and to download a small part of the Merkle tree called *Merkle Proof* of **Membership proof**: it contains the subset of hashes of the Merkle tree, that, together with $f_{i}$, allow to recomput the root hash. A proof is given for a specific *leaf node* and is comprises the leaf's sivlings, and the siblings of teach node on the path from the leaf to the root. 
The proof enables you to verify that $f_{i}$ was not modified by checking if the root hash obtained by the proof matches the Merkle root. 
![[Pasted image 20230317165901.png]]
To verify the proof, *"fill the blanks"* by computing the missing hashes in the dotted lines, then check if the computed Merkle root you kept locally is equel to the Merkle root obtain from the computation. 

###### Merkle Proof Consistency Theorem
*It is unfeasible to output a Merkle root h and two “inconsistent” proofs $\pi_{i}$ and $\pi_{i}'$ for two different inputs $x_{i}$ and $x_{i}'$ at the $i^{th}$ leaf in the tree of size $n$.
- ***Proof (by intuition)***:
	if the Merkle proof is verified, you were able to recompute the root hash by using $f_{i}$ as the i-th input and the Merkle proof as the remaining inputs. If the proof verification had yielded the same hash but with a different file  as the i-th input, this would yield a collision in the underlying hash function H used to build the tree. *Is not possible if H is collision resistant*.

#### Rationale behind large merkle proofs
The main reason to not simply use the has of the files to check their integrity is that the hashes $h_{i}$ are much smaller than the files themselves: can be done by storing the hash $h_{i} = H(f_{i})$ of every file, rather than the Merkle root so when you download a file, only compute the hash $y_{i}=H(f_{i})$ and check thath $h_{i} = y_{i}$. 
The merkle proof solution is suitable in case of a huge number of files as in *certificate transparency project* which stores millions of digital certificates for HTTPS websites. The Merkle Tree may have hundres of millions of leaves and it's designed to scale to billions by mantaining a $O(log(n))$ size. 

##### Merkle Tress in BitTorrent
In BitTorrent can help ensuring that *data blocks received from other peers are received undamaged and unaltered*: a trusted third party, like the site indexing the `.torrent` stores the *root hash* and the *total size of the file and the piece size*. So a **peer** receives a piece and a *Merkle proof* for it, calculate the has of that piece, request the root hash from the trusted site and by combining this information the client can recalculates the root has of the tree and caompares it to the root hash it received from the trusted source. 

##### Merkle Tress in Bitcoin
In a transaction scenarios between Alice and Bob, if Alice need to verify if the transaction has been inserted in the blockchain and don't want to download the entire blockchain, can use different solution:
- *Insicure-polling*: Alice simply asks the nodes storing the full blockchain if Bob's transaction has been inserted in the blockchain. Bob can lie by sending a fake transaction.
- *Merkle-based solution*: build a Merkle tree of the transactions in a block, a full node sends a Merkle Proof to Alice so she can finds the root of the Merkle tree in the block headers. In this way she must download only the block headers and not the entire blockchain. 

##### Merkle Tress Complexity Summary
$n$ is the number of data items, $m$ hash size
- size of the Merkle tree: $O(n)$
- size of the Merkle root: $O(m)$ 
- size of the authentication path: $O(m log(n))$ 
- $log(n)$ hashes are sufficient for checking each data block

## 4. Trie
The Trie data structure is a tree-like data structure that is commonly used to store and retrieve strings efficiently. In a Trie, each node represents a prefix or a complete string, and each edge represents a single character.

The key operations of a Trie are inserting a string, searching for a string, and deleting a string. The complexity of these operations depends on the length of the strings being processed and the size of the character set.
![[Pasted image 20230317173742.png]]

**Insertion** in a Trie involves traversing the tree until the end of the string is reached, and then adding a new node for each character that does not already have a corresponding child node. If the string is already in the Trie, no new nodes are added. The time complexity of insertion is $O(m)$, where $m$ is the length of the string being inserted.

***Searching*** in a Trie involves traversing the tree from the root to the node corresponding to the last character of the string. If the path does not exist in the Trie, the search fails. The time complexity of searching is $O(m)$, where m is the length of the string being searched for.

**Deleting** a string from a Trie involves first searching for the string, and then removing the nodes that correspond to the string's characters. If the string is not in the Trie, no nodes are removed. The time complexity of deletion is also $O(m)$, where m is the length of the string being deleted.

The space complexity of a Trie is determined by the number of unique strings that are stored in it. The worst-case space complexity is $O(n*m)$, where n is the number of strings and m is the length of the longest string. However, in practice, the space complexity of a Trie is usually much less than this, since many strings share common prefixes and can be stored using shared nodes.

### Patricia Trie
Patricia stands for "*Practical Algorithm To Retrieve Information Coded In Alphanumeric*". It compress long one-child branches into single node. In the following pictures, on the left a standard Trie, on the right the associated Patricia Trie:
![[Pasted image 20230317172227.png]]



## 5. Patricia Merkle Trie
The **Patricia Merkle Trie** is a combination of both:
- Patricia Tries: to enable faster search of data, stores keys, while grouping their common path in a node 
- Merkle Tree: to maintain data integrity and tamper proof validation
It is organized as a tree with every node hashed in the sense of *Merkle proof*, grouping common paths in the sense of *Patricia tries*. 

Regarding the Patricia Trie, it is not limited to string representation but can be used to store (`key,value`) pairs. Keys are string represented by the trie and the vaue will be stored in the which is at the end of a key path. 
![[Pasted image 20230317172816.png | 600]]

- **Branch node**: a node that has more than one child node. It stores links to child nodes, may also store a value.
- **Extension node**: a non-leaf node representing a sequence of nodes that has only one child. It sotes a key value that represents the combined nodes and a link to the next node.
- **Leaf Node/Value Node**: similar to an extension node. Instead of link, it stores a value. 
![[Pasted image 20230317173027.png]]

Key-value mapping represented in the Patricia trie `(BIN, 10) (BINARY, 8) (MOVE, 30) (MOVIE, 55) (TREE, 20) (TRIE, 48)`. 
On another example:
![[Pasted image 20230317173115.png]]
It's possible to ***"Merkelizing"*** the patricia trie by making the tree cryptographically secure, pairing each node with its hash. The root node becomes a cryptographic fingerprint of the entire data structure. The hash may be used for *lookup* in a database and to refenrence child nodes, as shown here:
![[Pasted image 20230317173304.png]]

The **Ethereum blockchain** store:
- the state of the contracts on the blockchain 
- a set of transactions in blocks
The state is a combination of `key/value` pair, may be that:
- `Key` is the address of an account, `value` is the account balance 
- `Key` is the address of a transaction, `value` is the transferred amount
Ethereum uses Merkle Patricia tries to store all the information on the blockchain.

The nodes of the Patricia Trie are *multi-element, structured records*, while hash function is applied to binary string  so it requires some kind of ***"serialization"*** to encode structured records as byte array (*like JSON, but more efficient*). 
Different techniques are used to encode a variety of data structures; two main techniques used by Ethereum are:
- ***Hex Prefix Encoding (HP)***: HP encoding is used for encoding/decoding Keys (Paths). 
- ***Recursive Length Prefix (RLP)***:  RLP is used for encoding/decoding Values, corresponding to keys (which may be structured values
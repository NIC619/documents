### 什麼是Proof of Stake

Proof of Stake （PoS）是公有鏈共識演算法的一類，乙太坊接下來的 Casper 演算法是其中之一。它和比特幣、目前的以太坊及許多其他區塊鏈背後的Proof of Work有著相似的作用，但在安全性及能源使用效率上有著更顯著的優點。

總的來說，PoS演算法大致如下。區塊鏈紀錄著一組 **validators** （驗證者），所有持有該區塊鏈數位貨幣的使用者（即以太坊的以太幣）都可藉由一個特殊交易將他們的數位貨幣鎖進一個存庫來成為validator。創造及許可一個新的區塊的過程則藉由一個目前validator可以參與的共識演算法來進行。

目前有許多種的共識演算法及許多種獎賞參與共識的validator的方式，因此有許多不同種類的PoS。從一個演算法的角度，主要分為兩種： 以鏈為基本的PoS 及 BFT（Byzantine Fault Tolerance）相似的PoS。

在以鏈為基本的PoS中，演算法在每個時間區間（例如每十秒為一區間）裡以偽隨機的方式選擇validator，並賦予其創造一個區塊的權利。而這個區塊如同區塊鏈，必須指向之前的區塊（通常是指向最長鏈的最新區塊）。所以隨時間拉長，區塊將會匯集並指向單一條持續增長的鏈。

在BFT相似的PoS中，validator被隨機指派來_提議_（propose）新的區塊，但用多輪投票的方式來決定區塊是否有效，每一輪每個validator送出投票（vote）給特定的區塊。在過程結束之後，validator會在一個區塊是否會成為鏈的一部分的這個議題上達成永久的共識。

### PoS在和PoW相比下有哪些優點？

[A Proof of Stake Design Philosophy](https://medium.com/@VitalikButerin/a-proof-of-stake-design-philosophy-506585978d51)裡有更完整的論述。 

簡短來說：
* 不需要為了鏈的安全兒**消耗大量電力**，（估計以太坊和比特幣每天都會各消耗一百萬美元在電力及硬體成本上）。
* 因為不需要消耗大量電力，所以**沒有發行等量的貨幣的需要**來提供加入網路的動機。理論上甚至可以有淨量為負值的發行量--以銷毀一部分的交易費的方式來達成。
* PoS提供了更多利用賽局理論的技術的發揮空間，來**降低中心化節點組織的形成**，及若組織已形成，可能的方法來防止他們進一步傷害網路（例如PoW中的[selfish mining](https://www.cs.cornell.edu/~ie53/publications/btcProcFC.pdf)）。
* **減少中心化造成的風險**，因為要擴展（scale）所要花的成本相比之下問題小得多。相比一百萬貨幣，一千萬的貨幣會讓你肯定獲得十倍多的報酬，且不需要以失衡的方式，如PoW中當你資本多到一定程度，便可負擔更好的規模生產設備。
* 可以利用經濟上的懲罰來**大大地拉高不同51%攻擊方式在PoS中所需要的成本**，引用Vlad Zamfir的話--"想像--參與51%攻擊則你的ASIC工廠就會被燒毀"。

### PoS和傳統的BFT研究怎麼結合？

BFT研究中有一些重要的結論是可以應用到各種共識演算法中的，包含傳統的演算法例如PBFT，同時也可用在任何PoS中，如果搭配恰當的數學模型進考量，甚至可以用在PoW中。

這些重要的結論包含：

* [**CAP 理論**](https://en.wikipedia.org/wiki/CAP_theorem) - "如果網路發生阻斷（partition）時，你只能選擇資料的一致性或可用性，無法兩者兼得"。論點很直覺：如果網路因阻斷而分隔為二，在其中一邊我送出一筆交易--"將我的十元給A"；在另一半我送出另一筆交易--"將我的十元給B"，則此時系統要不是 1）不可用，如這兩筆交易至少會有一筆交易不會被接受，要不 2）資料不一致，如一半看到的是A多了十元而另一半則看到B多了十元。要注意的是，CAP理論和擴展性（scalability）是無關的，他在分片（sharded）或非分片的系統皆適用。

* [**FLP impossibility**](http://the-paper-trail.org/blog/a-brief-tour-of-flp-impossibility/) - 在非同步（asynchronous）的環境中（即兩個正常運作節點間的網路延遲沒有保證的上限），只要有一個惡意的節點存在，就沒有演算法能在有限的時間內達成共識。但值得注意的是，["Las Vegas" algorithms](https://en.wikipedia.org/wiki/Las_Vegas_algorithm)不在其中，該演算法在每一輪皆有一定機率達成共識，而隨著時間增加，機率會越趨近於1。而這也是許多成功的共識演算法會採用的解決辦法。

* 容錯的上限 - 由[ DLS paper](http://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf) 我們可以得到以下結論： 1）在部分非同步（partially asynchronous）的網路環境中（即網路延遲有上限，但我們無法事先知道上限是多少），協議可以容忍最多 1/3 的拜占庭故障（Byzantine fault）。 2）在非同步（asynchronous）的網路環境中，具確定性的協議（deterministic protocol）無法容忍任何錯誤，但這篇論文並沒有提及[ randomized algorithms](http://link.springer.com/chapter/10.1007%2F978-3-540-77444-0_7) 在這種情況可以容忍最多1/3的拜占庭故障。 3）在同步（synchronous）的網路環境中（即網路延遲有上限且上限是已知的），協議可以容忍 100% 的拜占庭故障，但當超過 1/2 的節點為惡意節點時，會有一些限制條件。要注意的是，我們考慮的是"具認證特性的拜占庭模型（authenticated Byzantine）"，而不是"一般的拜占庭模型"；具認證特性指的是將如今已經過大量研究且成本低廉的公私鑰加密機制應用在我們的演算法中。

PoW [已被 Andrew Miller 及他人嚴格的分析過](https://socrates1024.s3.amazonaws.com/consensus.pdf)為一個仰賴同步的網路環境的模型。我們可以將網路設定為有接近無窮數量的節點，而每一個節點擁有非常小的算力且在一定時間內有非常小的機率可以產生區塊。在這個設定中，假如沒有網路延遲存在，則此協議有 50% 的容錯率。經過觀察，以太坊有約 46% 而比特幣擁有約 49.5% 的容錯率，但如果網路延遲和產生區塊時間相當時，容錯率會降低至 33% ，若網路延遲趨近無限，則容錯率趨近零。

PoS則是包含在BFT共識模型中，因為所有validator皆是已知且系統會記錄validator的數量。PoS的研究一般可分為兩個路線，一個是同步的網路環境模型，一個是部分非同步的網路環境模型。以鏈為基本的PoS演算法幾乎都仰賴同步的網路環境模型，而它們的安全性分析也都可用這些模型以相似於 [PoW](http://nakamotoinstitute.org/static/docs/anonymous-byzantine-consensus.pdf) 的方式來分析證明。另一個路線將部分同步網路環境中的傳統BFT演算法和PoS做連結，但解釋比較複雜，在後面的章節會有更深入的細節。

PoW演素法和以鏈為基本的PoS演算法選擇資料的可用性而非資料的一致性，但BFT類的共識演算法更傾向於選擇資料一致性。[Tendermint](https://github.com/tendermint/tendermint) 很清楚地選擇一致性，Casper使用混合的模型，此模型偏好資料的可用性，但盡可能的確保資料的一致性，它讓鏈上的應用和使用者在任何時間都能知道當前的資料一致性有多大的保證。

Ittay Eyal 和 Emin Gun Sirer 的 [selfish mining 研究](https://bitcoinmagazine.com/articles/selfish-mining-a-25-attack-against-the-bitcoin-network-1383578440) 的結論 - 在不同網路環境的模型中，比特幣挖礦的激勵兼容性（incentive compatibility）分別受到 25% 及 33% 的限制，即只有在 25% 或 33% 的礦工同謀不可能發生的前提下，挖礦的機制才是激勵兼容的（即礦工按照正常的方式挖礦--有多少算力獲得多少報酬）。這個結論和傳統的共識演算法的結論無關，因為傳統共識演算法的結論並沒有牽涉到激勵兼容性。

### 
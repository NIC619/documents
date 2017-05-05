### 什麼是Proof of Stake

Proof of Stake （PoS）是公有鏈共識演算法的一類，乙太坊接下來的 Casper 演算法是其中之一。它和比特幣、目前的以太坊及許多其他區塊鏈背後的Proof of Work有著相似的作用，但在安全性及能源使用效率上有著更顯著的優點。

總的來說，PoS演算法大致如下。區塊鏈紀錄著一組 **validators** （驗證者），所有持有該區塊鏈數位貨幣的使用者（即以太坊的以太幣）都可藉由一個特殊交易將他們的數位貨幣鎖進一個存庫來成為validator。創造及許可一個新的區塊的過程則藉由一個目前validator可以參與的共識演算法來進行。

目前有許多種的共識演算法及許多種獎賞參與共識的validator的方式，因此有許多不同種類的PoS。從一個演算法的角度，主要分為兩種： 以鏈為基本的PoS 及 BFT（Byzantine Fault Tolerance）相似的PoS。

在以鏈為基本的PoS中，演算法在每個時間區間（例如每十秒為一區間）裡以偽隨機的方式選擇validator，並賦予其創造一個區塊的權利。而這個區塊如同區塊鏈，必須指向之前的區塊（通常是指向最長鏈的最新區塊）。所以隨時間拉長，區塊將會匯集並指向單一條持續增長的鏈。

在BFT相似的PoS中，validator被隨機指派來_提議_（propose）新的區塊，但用多輪投票的方式來決定區塊是否有效，每一輪每個validator送出投票（vote）給特定的區塊。在過程結束之後，validator會在一個區塊是否會成為鏈的一部分的這個議題上達成永久的共識。

### PoS在和PoW相比下有哪些優點？

詳細說明可見：[A Proof of Stake Design Philosophy](https://medium.com/@VitalikButerin/a-proof-of-stake-design-philosophy-506585978d51)裡有更完整的論述。 

簡短來說：
* 不需要為了鏈的安全兒**消耗大量電力**，（估計以太坊和比特幣每天都會各消耗一百萬美元在電力及硬體成本上）。
* 因為不需要消耗大量電力，所以**沒有發行等量的貨幣的需要**來提供加入網路的動機。理論上甚至可以有淨量為負值的發行量--以銷毀一部分的交易費的方式來達成。
* PoS提供了更多利用賽局理論的技術的發揮空間，來**降低中心化節點組織的形成**，及若組織已形成，可能的方法來防止他們進一步傷害網路（例如PoW中的[selfish mining](https://www.cs.cornell.edu/~ie53/publications/btcProcFC.pdf)）。
* **減少中心化造成的風險**，因為要擴展（scale）所要花的成本相比之下問題小得多。相比一百萬貨幣，一千萬的貨幣會讓你肯定獲得十倍多的報酬，且不需要以失衡的方式，如PoW中當你資本多到一定程度，便可負擔更好的規模生產設備。
* 可以利用經濟上的懲罰來**大大地拉高不同51%攻擊方式在PoS中所需要的成本**，引用Vlad Zamfir的話--"想像一參與51%攻擊則你的ASIC工廠就會被燒毀"。

### PoS和傳統的BFT研究怎麼結合？

BFT研究中有一些重要的結論是可以應用到各種共識演算法中的，包含傳統的演算法例如PBFT，同時也可用在任何PoS中，如果搭配恰當的數學模型進考量，甚至可以用在PoW中。

這些重要的結論包含：

* [**CAP 理論**](https://en.wikipedia.org/wiki/CAP_theorem) - "如果網路發生阻斷（partition）時，你只能選擇資料的一致性或可用性，無法兩者兼得"。論點很直覺：如果網路因阻斷而分隔為二，在其中一邊我送出一筆交易--"將我的十元給A"；在另一半我送出另一筆交易--"將我的十元給B"，則此時系統要不是 1）不可用，如這兩筆交易至少會有一筆交易不會被接受，要不 2）資料不一致，如一半看到的是A多了十元而另一半則看到B多了十元。要注意的是，CAP理論和擴展性（scalability）是無關的，他在分片（sharded）或非分片的系統皆適用。

* [**FLP impossibility**](http://the-paper-trail.org/blog/a-brief-tour-of-flp-impossibility/) - 在非同步（asynchronous）的環境中（即兩個正常運作節點間的網路延遲沒有保證的上限），只要有一個惡意的節點存在，就沒有演算法能在有限的時間內達成共識。但值得注意的是，["Las Vegas" algorithms](https://en.wikipedia.org/wiki/Las_Vegas_algorithm)不在其中，該演算法在每一輪皆有一定機率達成共識，而隨著時間增加，機率會越趨近於1。而這也是許多成功的共識演算法會採用的解決辦法。

* 容錯的上限 - 由[ DLS paper](http://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf) 我們可以得到以下結論： 1）在部分非同步（partially synchronous）的網路環境中（即網路延遲有上限，但我們無法事先知道上限是多少），協議可以容忍最多 1/3 的拜占庭故障（Byzantine fault）。 2）在非同步（asynchronous）的網路環境中，具確定性的協議（deterministic protocol）無法容忍任何錯誤，但這篇論文並沒有提及[ randomized algorithms](http://link.springer.com/chapter/10.1007%2F978-3-540-77444-0_7) 在這種情況可以容忍最多1/3的拜占庭故障。 3）在同步（synchronous）的網路環境中（即網路延遲有上限且上限是已知的），協議可以容忍 100% 的拜占庭故障，但當超過 1/2 的節點為惡意節點時，會有一些限制條件。要注意的是，我們考慮的是"具認證特性的拜占庭模型（authenticated Byzantine）"，而不是"一般的拜占庭模型"；具認證特性指的是將如今已經過大量研究且成本低廉的公私鑰加密機制應用在我們的演算法中。

PoW [已被 Andrew Miller 及他人嚴格的分析過](https://socrates1024.s3.amazonaws.com/consensus.pdf)為一個仰賴同步的網路環境的模型。我們可以將網路設定為有接近無窮數量的節點，而每一個節點擁有非常小的算力且在一定時間內有非常小的機率可以產生區塊。在這個設定中，假如沒有網路延遲存在，則此協議有 50% 的容錯率。經過觀察，以太坊有約 46% 而比特幣擁有約 49.5% 的容錯率，但如果網路延遲和產生區塊時間相當時，容錯率會降低至 33% ，若網路延遲趨近無限，則容錯率趨近零。

PoS則是包含在BFT共識模型中，因為所有validator皆是已知且系統會記錄validator的數量。PoS的研究一般可分為兩個路線，一個是同步的網路環境模型，一個是部分非同步的網路環境模型。以鏈為基本的PoS演算法幾乎都仰賴同步的網路環境模型，而它們的安全性分析也都可用這些模型以相似於 [PoW](http://nakamotoinstitute.org/static/docs/anonymous-byzantine-consensus.pdf) 的方式來分析證明。另一個路線將部分同步網路環境中的傳統BFT演算法和PoS做連結，但解釋比較複雜，在後面的章節會有更深入的細節。

PoW演素法和以鏈為基本的PoS演算法選擇資料的可用性而非資料的一致性，但BFT類的共識演算法更傾向於選擇資料一致性。[Tendermint](https://github.com/tendermint/tendermint) 很清楚地選擇一致性，Casper使用混合的模型，此模型偏好資料的可用性，但盡可能的確保資料的一致性，它讓鏈上的應用和使用者在任何時間都能知道當前的資料一致性有多大的保證。

Ittay Eyal 和 Emin Gun Sirer 的 [selfish mining 研究](https://bitcoinmagazine.com/articles/selfish-mining-a-25-attack-against-the-bitcoin-network-1383578440) 的結論 - 在不同網路環境的模型中，比特幣挖礦的激勵兼容性（incentive compatibility）分別受到 25% 及 33% 的限制，即只有在 25% 或 33% 的礦工同謀不可能發生的前提下，挖礦的機制才是激勵兼容的（即礦工按照正常的方式挖礦--有多少算力獲得多少報酬）。這個結論和傳統的共識演算法的結論無關，因為傳統共識演算法的結論並沒有牽涉到激勵兼容性。

### 什麼是"零機會成本"問題及該如何解決這個問題？
在許多之前（以鏈為基本的）PoS演算法，包含Peercoin，只有對產生區塊給予相對應的獎賞但並沒有懲罰。這在出現多條相競爭（即分叉）的鏈的情況時，會有非預期的影響，因為每個validator皆有動機在每一條相互競爭中的鏈上都產生區塊（以下將下注和產生區塊視為相同意思）來確保會獲得獎賞，如下：

![](https://raw.githubusercontent.com/vbuterin/diagrams/master/possec.png)

在PoW中，這麼做會導致礦工的算力被分散，導致獲利下降：

![](https://github.com/vbuterin/diagrams/blob/master/powsec.png?raw=true)

結果就會造成如果每個參與者都是狹義上經濟理性（narrowly economically rational）的話，則即便在沒有任何攻擊者的情況下，區塊鏈本身也沒辦法達成共識，因為每個validator都在每條鏈上下注。如果有攻擊者，攻擊者只需要贏過那些執行利他行為（altruistic）的節點（即只會在單一條鏈下注的節點）即可，不需要贏過一般經濟理性的節點。相反的在PoW，攻擊者必須要同時贏過利他節點和經濟理性節點（至少是可行的威脅：可參考SchellingCoin的 [P + epsilon 攻擊](https://blog.ethereum.org/2015/01/28/p-epsilon-attack/)）。

有些人會認為下注者有動機按照規則來下注且只下注在最長的鏈上，好讓他們的投資能夠保值，然而這個論點忽略了這個動機受制於公地悲劇理論（[tragedy of the commons](https://en.wikipedia.org/wiki/Tragedy_of_the_commons)）：每個下注者可能只會有 1% 的機會成為關鍵的（pivotal）角色（即他的決定會影響一個攻擊的成敗），所以用來買通他們的賄絡金額只需要是他們總賭注金額的 1% 。因此，全部的賄賂金額只需要下注金額總額的 0.5-1% 。此外，這個（本段開頭的）論點同時暗示著任何"不可能失敗"的情況都不是一個穩定的平衡，因為不可能失敗的情況代表每個下注者成為關鍵角色的機會是零，即只要賄賂金額超過 0% 都能讓下注者有動機參與攻擊。

有兩種方式可以解決這個問題。第一個以 ["Slasher"](https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm/))為名稱大略地描述，並進一步由 [Iddo Bentov ](https://arxiv.org/pdf/1406.5694.pdf)開發。當validator同時在不同條分叉的鏈上下注（產生區塊）的情況發生時，將證據紀錄進區塊鏈中並以此銷毀validator的下注資本。這讓動機結構改變如下：

![](https://github.com/vbuterin/diagrams/blob/master/slasher1sec.png?raw=true)

注意，這個演算法要能執行，validator是哪些人需要事先就知道。否則validator可以任意選擇要下注的鏈：當A鏈可下注就下注A鏈，當B鏈可下注就下注B鏈，當兩條都可以下注就下注最長的鏈。所以事先確定validator的名單可以避免這種情況發生。但這也有缺點存在，包括要求節點需要頻繁地上線來獲得安全可信的鏈的狀態，並且讓medium-range的validator共謀攻擊（例如連續三十個validator中有25個預謀發起攻擊來回復過去19個區塊）有可能發生，因為validator事先知道什麼時候會輪到他產生區塊。如果這些風險是可接受的那就沒太大問題。

第二個方式單純地懲罰在錯的鏈產生區塊的validator。也就是當有兩個互相競爭的A和B鏈，如果有一個validator在B上面產生區塊，則他可在B鏈上獲得 R 的獎賞，但這個區塊的標頭（header）資料會被記錄在A鏈上（在Casper中叫做"dunkle"）且他在A鏈上會受到 F 的罰金（ F 可能等於 R ）。這會將結構改變成：

![](https://github.com/vbuterin/diagrams/blob/master/slasher2sec.png?raw=true)

直覺來說，我們可以把PoW的經濟模型複製到這來用。在PoW中，在錯的鏈上產生區塊同樣有懲罰，但這個懲罰並不顯而易見：礦工額外的電力或硬體成本的花費（因為要同時在兩條鏈上花費運算）。這個機制（第二個方式）的一個缺點是它將些微的風險加注到validator身上（因為validator要承擔在錯的鏈上產生區塊的成本），不過這個效應會隨者時間慢慢減退，但另一方面，優點是它不需要事先知道validator有誰。

###上一章節介紹以鏈為基本的PoS如何解決"零機會成本"問題，那BFT相似的PoS又是怎麼運作的呢？

BFT類型（部分同步）的PoS演算法允許validator藉由送出遵守兩類規則的簽名訊息的方式來對區塊進行"投票"，這兩類規則分別是：

* **終局條件（Finality conditions）** - 規則用來決定某雜湊值是否可被視為不可更改的（finalized）。
* **刪砍條件（Slashing condition）** - 規則用來決定某個validator是否有足夠理由懷疑他作弊（例如同時下注多個相衝突的區塊）。如果有validator觸發其中任何一條規則，他們的資本將全數被刪去。

以下舉兩個例子來說明不同刪砍條件的發生場景，下面的" 2/3 的validator"代表"全數validator的資本總和的 2/3"，其他的比例亦同。在這些例子中，"PREPARE" 和 "COMMIT" 可單純地理解為兩種validator可送的簽名訊息（`MESSAGE`）。

1. 如果`MESSAGE`包含--`["COMMIT", HASH1, view]` 和 `["COMMIT", HASH2, view]`，`view`為相同但`HASH1`和`HASH2`不同且皆由同一個validator所簽名，則validator資本被刪去。
2. 如果`MESSAGE`包含--`["COMMIT", HASH, view1]`，則**除非** `view1 == -1` 或同時存在其他包含`["PREPARE", HASH, view1. view2]`的簽名訊息（其中 `view2 < view1`）且訊息是由 2/3 的validator所簽，則對`"COMMIT"`簽名的validator的資本被刪去。

合適的刪砍條件需要有兩個重要的要求：

* **可咎責的安全性（Accountable safety）** - 如果相互衝突的`HASH1` 和 `HASH2`（即兩者有不同的雜湊值，但在鏈上兩者都不接在彼此的後方）都被認定為不可更改，則至少有 1/3 的validator肯定違反了某些刪砍條件。
* **Plausible liveness** - 除非至少 1/3 的validator違反了某些刪砍條件，否則必定存在某些合法的訊息是 2/3 的validator可以簽的，且這些訊息會讓某些（雜湊）值變成不可更改的。即除非至少 1/3 的validator違規，否則一定可以產生新的合法的區塊來讓鏈延續。

如果我們有一組刪砍條件可以達成這兩個要求，則我們便可以提供共識參與者足夠的動機並開始從經濟面上終局（economic finality）的特性中獲得成果。

###一般來說，什麼是"經濟面上終局的特性"？

經濟面上終局特性的想法是--當區塊被認定為不可更改，或更一般來說有一種訊息獲得足夠數量的簽名，則唯一讓鏈在未來收入另一個相衝突的區塊的方法是有一大群的人願意燒掉一大堆錢。如果一個節點看到某個區塊符合經濟面上終局的特性，則他有經濟面上非常大的保證這個區塊會成為鏈的歷史資料的一部分（且這條鏈也是大家都認可的）。

達成經濟面上終局有兩種方法：

1. 一個區塊可被視為經濟面上的不可更改，如果有足夠多的validator對以下形式的聲明進行簽名--"當區塊B沒有被收入則我會失去X的資本"。這讓使用者獲得了如下的保證--(I) 區塊B是鏈的一部分，或是(II)若validator想騙他們，讓他們相信區塊B是有被收入的，則validator會損失一大筆錢。

2. 一個區塊可被視為經濟面上的不可更改，如果有足夠多的validator簽名表達支持收入區塊B，且有數學能證明 __當不同於區塊B的某區塊B'也以同樣方式被收入__ ，則這些validator會損失一大筆錢。如果使用者看到這個被收入的區塊，並驗證了鏈的有效性（validate the chain），且藉由有效性（validity）和不可更改性（finality）他們可以在發生分叉的鏈之中做出選擇，則他們能獲得如下的保證--(I) 區塊B是鏈的一部分，或是(II)若validator同時也參與了另一條互相競爭且亦符合終局條件的鏈，則validator會損失一大筆錢。

兩種達成終局（finality）的方法分別繼承自"零機會成本問題"的兩個解決方法： 藉由懲罰錯誤（如`"COMMIT"`不合法的區塊）來達成終局 及 藉由懲罰不明確性（如`"COMMIT"`兩個衝突區塊）來達成終局。第一個方法的主要優點是輕量級使用者也能驗證且比較顯而易見，第二個方法的主要優點是(I)比較容易瞭解為何誠實的validator不會被懲罰及(II)干擾因素（griefing factors）對誠實的validator比較有利（即相比於不誠實者干擾誠實者所要付出的成本，誠實者干擾不誠實者的成本是比較低的）。

Casper遵循第二種方法，不過是可以透過增加鏈上的機制，讓validator可以自願選擇是否要多新增對第一種方法的聲明（即"當區塊B沒有被收入則我會失去X的資本"）的簽名，此舉可讓更多輕量級使用者增加效率。


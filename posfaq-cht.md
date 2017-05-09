### 什麼是Proof of Stake

Proof of Stake （PoS）是公有鏈共識演算法的一類，乙太坊接下來的 Casper 演算法是其中之一。它和比特幣、目前的以太坊及許多其他區塊鏈背後的Proof of Work有著相似的作用，但在安全性及能源使用效率上有著更顯著的優點。

總的來說，PoS演算法大致如下。區塊鏈紀錄著一組 **validators** （驗證者），所有持有該區塊鏈數位貨幣的使用者（即以太坊的以太幣）都可藉由一個特殊交易將他們的數位貨幣鎖進一個存庫來成為validator。創造及許可一個新的區塊的過程則藉由一個目前validator可以參與的共識演算法來進行。

目前有許多種的共識演算法及許多種獎賞參與共識的validator的方式，因此有許多不同種類的PoS。從一個演算法的角度，主要分為兩種： 鏈型態的PoS 及 BFT（Byzantine Fault Tolerance）相似的PoS。

在鏈型態的PoS中，演算法在每個時間區間（例如每十秒為一區間）裡以偽隨機的方式選擇validator，並賦予其創造一個區塊的權利。而這個區塊如同區塊鏈，必須指向之前的區塊（通常是指向最長鏈的最新區塊）。所以隨時間拉長，區塊將會匯集並指向單一條持續增長的鏈。

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

PoS則是包含在BFT共識模型中，因為所有validator皆是已知且系統會記錄validator的數量。PoS的研究一般可分為兩個路線，一個是同步的網路環境模型，一個是部分非同步的網路環境模型。鏈型態的PoS演算法幾乎都仰賴同步的網路環境模型，而它們的安全性分析也都可用這些模型以相似於 [PoW](http://nakamotoinstitute.org/static/docs/anonymous-byzantine-consensus.pdf) 的方式來分析證明。另一個路線將部分同步網路環境中的傳統BFT演算法和PoS做連結，但解釋比較複雜，在後面的章節會有更深入的細節。

PoW演素法和鏈型態的PoS演算法選擇資料的可用性而非資料的一致性，但BFT類的共識演算法更傾向於選擇資料一致性。[Tendermint](https://github.com/tendermint/tendermint) 很清楚地選擇一致性，Casper使用混合的模型，此模型偏好資料的可用性，但盡可能的確保資料的一致性，它讓鏈上的應用和使用者在任何時間都能知道當前的資料一致性有多大的保證。

Ittay Eyal 和 Emin Gun Sirer 的 [selfish mining 研究](https://bitcoinmagazine.com/articles/selfish-mining-a-25-attack-against-the-bitcoin-network-1383578440) 的結論 - 在不同網路環境的模型中，比特幣挖礦的激勵兼容性（incentive compatibility）分別受到 25% 及 33% 的限制，即只有在 25% 或 33% 的礦工同謀不可能發生的前提下，挖礦的機制才是激勵兼容的（即礦工按照正常的方式挖礦--有多少算力獲得多少報酬）。這個結論和傳統的共識演算法的結論無關，因為傳統共識演算法的結論並沒有牽涉到激勵兼容性。

### 什麼是"零風險成本"問題及該如何解決這個問題？
在許多之前（鏈型態的）PoS演算法，包含Peercoin，只有對產生區塊給予相對應的獎賞但並沒有懲罰。這在出現多條相競爭（即分叉）的鏈的情況時，會有非預期的影響，因為每個validator皆有動機在每一條相互競爭中的鏈上都產生區塊（以下將下注和產生區塊視為相同意思）來確保會獲得獎賞，如下：

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

###上一章節介紹鏈型態的PoS如何解決"零風險成本"問題，那BFT相似的PoS又是怎麼運作的呢？

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

兩種達成終局（finality）的方法分別繼承自"零風險成本問題"的兩個解決方法： 藉由懲罰錯誤（如`"COMMIT"`不合法的區塊）來達成終局 及 藉由懲罰不明確性（如`"COMMIT"`兩個衝突區塊）來達成終局。第一個方法的主要優點是輕量級使用者也能驗證且比較顯而易見，第二個方法的主要優點是 (I)比較容易瞭解為何誠實的validator不會被懲罰及 (II)干擾因素（griefing factors）對誠實的validator比較有利（即相比於不誠實者干擾誠實者所要付出的成本，誠實者干擾不誠實者的成本是比較低的）。

Casper遵循第二種方法，不過是可以透過增加鏈上的機制，讓validator可以自願選擇是否要多新增對第一種方法的聲明（即"當區塊B沒有被收入則我會失去X的資本"）的簽名，此舉可讓更多輕量級使用者增加效率。

###所以這和BFT的理論有什麼關聯？

傳統的BFT理論在 safety 和 liveness 上和我們有相似的要求。首先，傳統BFT理論要求當超過 2/3 的validator是誠實的時候，safety必須要被達成。嚴格來說這是比較容易實現的模型，傳統的BFT理論嘗試證明"如果共識機制無法達成safety，則我們知道至少有 1/3 的validator是惡意的"；而我們的模型則是嘗試證明"如果共識機制無法達成safety，則我們知道至少有 1/3的validator是惡意的，而且我們知道是哪些validator，即便你在出現問題的當下不在線上"。從liveness的角度，我們的模型比較容易達成，因為我們不需要證明**共識會被達成**，我們只需要證明機制**沒有卡住**。

不過幸運的是，額外的"可咎責的"（accountable）特性需求其實不難實現；事實上，只要協議有正確的防禦機制（protocol armor），我們都可以將任何不管是部分同步或是非同步的傳統BFT演算法轉換成可咎責的演算法。這個證明基本上歸結於一個事實--拜占庭故障（fault）可以被窮舉並分類，而這每一類要不是 (I)可咎責的（如果你`"COMMIT"`了這類的訊息，則你會被逮到，且我們可以為此建立一條刪砍條件的規則），要不就是 (II)無法被分辨是網路延遲還是故障（注意，即便是太早送出訊息這種故障，也沒辦法被分辨出來）。

###什麼是弱主觀性（weak subjectivity）？

很重要的一點是，利用存款（deposit，也就是validator下注的資本）來確保"風險成本不為零"的機制確實改變了安全模型（security model）。假設 (I)存款會被鎖住一個月，時間到之後可以提走， (II)有個 51% 攻擊嘗試反轉（revert）長達 10 天的交易量，則這些攻擊者產生的區塊會被寫入鏈裡當作證據且validator會被懲罰。然而，假設攻擊變成長達 40 天，則雖然這些攻擊產生的區塊可以再被寫入鏈裡，但validator早已能把錢提走而不會受到懲罰。為了要解決這個問題，我們需要一個"反轉限制（revert limit）"的規則，也就是當反轉所影響的區塊總時間長度超過存款鎖住的期限，則節點可以拒絕接受這些區塊（在上例，即拒絕影響超過一個月的反轉區外）。這表示節點現在多了兩項要求：

1. 當節點第一次連上並要同步鏈的資料的時候，他們必須藉由鏈外的方式來驗證最新的狀態，即透過朋友節點們或各個 Block Explorer 等等的方式。如果他們得到的都是同一條一樣的鏈，則可以確定這條是正確的。注意，只有在出現鏈分叉長度超過反轉限制時（在上例，即得到兩條在過去超過一個月的區塊皆不相同的鏈）才需要這種採用這種鏈外的交際驗證（social authentication）。

2. 節點每隔一段"反轉限制"的時間就必須要上線同步。如果沒定時同步，則需要再透過一次鏈外的交際驗證。

這種鏈外的交際驗證事實上範圍有限。如果攻擊者要利用這個管道來攻擊，他們必須要說服社群裡一大部分的人，讓他們相信攻擊者的鏈才是有效的；或是改為說服新加入社群的人--新加入的人可能會在下載軟體時一並收到最近一次的檢查點（checkpoint，即反轉限制的臨界點），但如果攻擊者能竄改這個檢查點的紀錄，則他們要能直接竄改整個軟體也不再是件難事，而且沒有單純的密碼經濟學的驗證方式能解決這個問題。當一個節點連接上了，只要他夠頻繁地上線，他就能以單純密碼經濟學上安全的模型來確保連接上正確的鏈，而不需要額外的鏈外交際驗證。

另外，這種交際驗證如果需要，也可以直接加入進使用者使用的過程中：如 (I)[BIP 70](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki)的交易就要求交易裡要加入最近一段時間的某個區塊的雜湊值，使用者的軟體會在交易成立前，確保使用者和商家是在同一條鏈上（也可以透過其他鏈上的互動方式）。 或(II)，採用Jeff Coleman的 [universal hash time](https://www.youtube.com/watch?v=phXohYF0xGo)。採用UHT的話，則如果攻擊要能成功，攻擊者必須在被攻擊鏈繼續增長的**同時**，暗中產生另一條鏈（即攻擊者沒辦法事先或在事後短時間內產生一條相抗衡的鏈），代表這需要大多數的validator共謀了一段非常長的時間。

###在POS裡可以用經濟上的方式懲罰審查（censorship）行為嗎？

審查行為比起交易反轉要更難去證明。區塊鏈本身無法分辨"使用者 A 嘗試送出交易 X 但被審查過濾掉了"或是"使用者 A 送出交易 X 但因為交易費不夠而沒被收入區塊裡"或是"使用者 A 從未送出交易 X "。但仍有一些方法可以對抗審查行為。

第一個是使用停機問題（halting problem）。這個方法比較弱的版本是將協議設計成圖靈完備，使得validator無法知道一筆交易會不會在花費他大量的運算後因為出現預期外的行為而出錯，這同時也讓validator面臨潛在的DoS攻擊。這也是當初[ the DAO 軟分叉](http://hackingdistributed.com/2016/07/05/eth-is-more-resilient-to-censorship/)沒有實行的原因。

比較強的版本則是讓交易在未來中短期的時間內觸發特定的效果。使用者可以送出多筆相互關聯交易及可預期的第三方的資訊來導致未來事件的發生，validator要等到交易都被收入區塊（並確認為不可更改）才能知道發生了什麼事件，但此時要阻止交易又已經太遲。即便過濾掉所有相關的交易，validator想阻止的事件還是會發生。validator可以試著過濾掉所有交易，或是過濾掉沒有附上相關證明（證明交易不會導致任何非預期的情況發生）的交易，但這麼做會擋掉非常多類型的交易以至於讓整個系統失靈，validator的存款（deposit）價值也會跟著該數位貨幣的價值崩盤而下降。

第二個方法（[經由 Adam Back 的描述](https://www.reddit.com/r/Bitcoin/comments/4j7pfj/adam_backs_clever_mechanism_to_prevent_miners/d34t9xa)），是要求交易都經過[timelock-encrypted](https://www.gwern.net/Self-decrypting%20files)加密。所以validator只能在不知道交易的內容的情況下將交易收入區塊，直到之後某個時間點交易內容被揭露，但此時要過濾掉交易已經太遲。但是validator可以只收入有附上解密內容的密碼學證明（如 zkSNARK ）的交易；這雖然會強迫使用者必須要去下載新的使用軟體，不過有心人士可以直接提供相關軟體讓人使用下載，而這在賽局理論的模型中，使用者是有動機去配合的。

或許在PoS中比較好的做法是使用者安裝軟體更新來執行硬分叉，將惡意的validator移除，而這和安裝軟體更新來讓交易更容易通過審查相比，並沒有多難。總之，這個方法（指第二個方法）雖然會降低和鏈溝通互動的速度（注意，採用這個方法必須是強制的才會有效，否則惡意的validator只需要過濾加密過的交易並收入沒加密的交易即可），卻也是比較適度且有效的。

第三個方法是在鏈分叉發生時，將偵測審查行為發生的機制加入進鏈選擇的考量。原理很簡單，節點觀察著網路及交易，如果他們發現某筆交易帶有夠多的手續費卻遲遲未被收入，就給沒有收入這筆交易的鏈較低的評分。如果所有的節點都遵守這規則，則最終較弱勢的鏈也會因為收入了這個交易而讓其他誠實的節點都轉而加入這條鏈。這個方法主要的缺點是，離線的節點還是紀錄者強勢的（有審查機制的）鏈，如果在他們重新上線之前審查行為就結束了，則會造成上線的（誠實的）節點間的分歧。因此這個方法比較適合被用來當作緊急情況如硬分叉發生時的一個節點協調的工具，如果是用在幾乎每天都會發生的鏈分叉的選擇考量則不太合適。

###validator是怎麼選出的？什麼又是stake grinding？

在任何鏈型態的PoS演算法中，都需要一個機制來隨機選出哪個validator可以產生下個區塊。例如，假設目前活躍中的validator包含 資本為 40 ehter 的 Alice、資本為 30 ether 的 Bob、資本為 20 ether 的 Charlie 及資本為 10 ether 的 David，則你希望他們各自被選出的機率分別為 40%、30%、20%及10%（當然在實際情況中，你會希望選出來的是一連串無限的候選人而不是一個，這樣當前面一位沒出現，後面一位就可以遞補，但這不影響根本的問題）。在非鏈形態的演算法中，還是因為不同原因而需要隨機性（randomness）。

"Stake grinding"是一類validator試圖透過一些計算或其他方式來影響隨機性的攻擊。例如：

1. 在 [Peercoin](https://bitcointalk.org/index.php?topic=131901.0) 中，validator可以搜尋各種參數的組合並找到特定的參數來增加他們產生有效區塊的機率。

2. 在一個目前已經不使用的方式裡，第 N+1 個區塊的隨機性取決於第 N 個區塊裡的簽章。這讓validator可以重複產生新的簽章直到他們找到一個特別的簽章來讓他們能預測並掌握下一個區塊，以藉此控制系統。

3. 在 NXT 中，第 N+1 個區塊的隨機性取決於產生第 N 個區塊的validator。這讓validator可以藉由跳過一個產生區塊的機會來操縱區塊隨機性。雖然這麼做的機會成本是一個區塊獎賞的價值，但有時候新產生的隨機種子可以讓validator在未來數十個區塊中獲得高於平均區塊獎賞的獎勵。[這裏](http://vitalik.ca/files/randomness.html)有更詳細的分析。

(1)和(2)很容易解決；一般的做法是要求validator事先存款來避免validator一直改變身份（address）來找到可以影響隨機性的值，並避免使用可以輕易被操縱的訊息如讓一個區塊的簽章。有幾個主要的策略來解決(3)。第一個是利用 [secret sharing](https://en.wikipedia.org/wiki/Secret_sharing) 或是 [deterministic threshold signatures](https://eprint.iacr.org/2002/081.pdf) 並要求validator一同產生隨機值。這些方法在大多數validator（看各種應用不同，33% - 50% 的validator合謀就可以干預，使得協議對維持liveness 的機率假設剩下 67% ）沒有同謀的時候都足夠穩固。

第二個方法是使用密碼學的方式--validator事先 commit 一些訊息（如公布 `sha3(x)`），接著在區塊內公佈 `x` 值，最後將 `x` 值和其他人的隨機值加在一起。理論上針對這個方法有兩種潛在的攻擊。

1. 在commit時操縱 `x` 值。但因為結果會將許多人的 `x` 值一起加入考量，其中只需要一個人是誠實的，隨機性的分佈就會呈常態分佈，所以這攻擊不太可行。

2. 選擇性地不公開區塊。這種攻擊的機會成本是一個區塊獎賞，而且這個方法頂多只能讓一個人看到下一個區塊的validator是誰，所以最多可能的獲益也是一個區塊獎賞。唯一的例外是，如果一個validator跳過，則遞補上來的validator和下一個區塊的validator會是同樣一個validator，可以增加一條跳過區塊的罰則來降低其發生的機率。

第三個方法是使用 [Iddo Bentov 的 "majority beacon"](https://arxiv.org/pdf/1406.5694.pdf)，藉由之前產生的（也用 beacon 方式產生的） N 個隨機數字中的每個 bit 值的多數決來產生新的隨機數字（即，如果大多數數字的第一個 bit 為 1 ，則新的數字的第一個 bit 為 1 ，否則為 0 ）。攻擊的成本會是 `~C * sqrt(N)` ，其中 C 是攻擊其他 beacon 產生的隨機數字的成本。總之，有許多 stake grinding 的解決方法存在，這個問題比較像是 [differential cryptanalysis](https://en.wikipedia.org/wiki/Differential_cryptanalysis) 而不是 [the halting problem](https://en.wikipedia.org/wiki/Halting_problem) - 一個 PoS 設計者最終會明瞭且會知道如何克服的痛點，但不是根本且無法彌補的缺陷。

###What would the equivalent of a 51% attack against Casper look like?

###針對Casper對等的 51% 算力攻擊會是怎麼樣的攻擊？

The most basic form of "51% attack" is a simple **finality reversion**: validators that already finalized block A then finalize some competing block A', thereby breaking the blockchain's finality guarantee. In this case, there now exist two incompatible finalized histories, creating a split of the blockchain, that full nodes would be willing to accept, and so it is up to the community to coordinate out of band to focus on one of the branches and ignore the other(s).

51% 攻擊最基本的形式就是**finality reversion**：validator確立區塊A的紀錄為不可更動後又將另一個區塊A'也列為不可更動，打破區塊鏈不可更動特性的保證。在這個情況中，並存著兩個彼此不相容的歷史紀錄導致鏈產生分叉，這需仰賴社群以鏈外的方式進行協調來決定應該選擇哪條鏈，而哪條該被捨棄。

This coordination could take place on social media, through private channels between block explorer providers, businesses and exchanges, various online discussion forms, and the like. The principle according to which the decision would be made is "whichever one was finalized first is the real one". Another alternative is to rely on "market consensus": both branches would be briefly being traded on exchanges for a very short period of time, until network effects rapidly make one branch much more valuable with the others. In this case, the "first finalized chain wins" principle would be a Schelling point for what the market would choose. It's very possible that a combination of both approaches will get used in practice.

協調的管道有很多種，如社群媒體、block explorer及交易所間的溝通、線上論壇等等。決定該選哪條鏈的原則是 "哪條先出現就選哪條" 。另一個方式是讓市場機制去決定：在很短的時間裡，兩條分叉都可以在交易所中交易，直到其中一條因為價值更高而勝出。在這種情況中，"哪條先出現就選哪條" 的原則會是市場機制的 [Schelling point](https://zh.wikipedia.org/wiki/谢林点)，即參與者會因為覺得其他人也會選擇先出現的那條而傾向選擇先出現的那條，所有人在沒有溝通的情況下按照這個傾向選擇了先出現的那條鏈。所以實際中，這兩種方式並用是非常有可能的。

Once there is consensus on which chain is real, users (ie. validators and light and full node operators) would be able to manually insert the winning block hash into their client software through a special option in the interface, and their nodes would then ignore all other chains. No matter which chain wins, there exists evidence that can immediately be used to destroy at least 1/3 of the validators' deposits.

當該選擇哪條鏈的共識達成時，使用者（即validator、light node及full node）就可以手動地將勝出的區塊的雜湊值藉由特殊的選項寫入軟體中，之後他們的節點就會忽略其他（不包含該區塊雜湊值）的鏈。之後不管是哪條鏈被選擇，都有證據可以用來懲罰至少 1/3 的違規validator。

Another kind of attack is liveness denial: instead of trying to revert blocks, a cartel of >=34% of validators could simply refuse to finalize any more blocks. In this case, blocks would never finalize. Casper uses a hybrid chain/BFT-style consensus, and so the blockchain would still grow, but it would have a much lower level of security. If no blocks are finalized for some long period of time (eg. 1 day), then there are several options:

另外一種攻擊是阻斷 liveness：一個由超過 34% validator組成的集團可以拒絕將任何區塊變成不可更動的。在這種情況下，將沒有區塊能被變成不可更動。Casper採用混合（鏈 + BFT型態）的共識，因此鏈還是會持續增長，但安全性會大大降低。如果很長一段時間（例如一天）都沒有區塊被變成不可更動，則有以下幾種選項：

1. The protocol can include an automatic feature to rotate the validator set. Blocks under the new validator set would finalize, but clients would get an indication that the new finalized blocks are in some sense suspect, as it's very possible that the old validator set will resume operating and finalize some other blocks. Clients could then manually override this warning once it's clear that the old validator set is not coming back online. There would be a protocol rule that under such an event all old validators that did not try to participate in the consensus process take a large penalty to their deposits.

1. 可以採用一個自動化的功能來輪轉validator名單，瓦解集團的佔比。在新的validator名單中區塊可以被變為不可更動，但使用者會收到提醒告訴他們這些不可更動的區塊還是不可全信的，因為很有可能舊的一組validator會重新奪回控制權並將其他的區塊變為不可更動。使用者如果確信舊的一組validator不會再上線，就可以手動覆蓋過這些警告。會有規則規定在這種情況發生時，所有舊的validator如果沒有再參與共識過程則會受到相當大筆的罰款。

2. A hard fork is used to add in new validators and delete the attackers' balances.

2. 採用硬分叉的方式移除攻擊者的存款，並增加新的validator。

In case (2), the fork would once again be coordinated via social consensus and possibly via market consensus (ie. the branch with the old and new validator set briefly both being traded on exchanges). In the latter case, there is a strong argument that the market would want to choose the branch where "the good guys win", as such a chain has validators that have demonstrated their goodwill (or at least, their alignment with the interest of the users) and so is a more useful chain for application developers.

在第二種方法中，分叉還是一樣要由鏈外的共識來協調，且可能會由市場共識的方式（即兩條擁有不一樣validator組成的鏈短暫的並存在交易市場上）。如果是藉由市場共識的方式，有個有力的論點是--市場會傾向選擇 "好人勝出" 的鏈，這條鏈的validator展現了他們的誠意（或至少他們和使用者的利益是並存的），因此也是一條對應用開發者較有用的鏈。

Note that there is a spectrum of response strategies here between social coordination and in-protocol automation, and it is generally considered desirable to push as far toward automated resolution as possible so as to minimize the risk of simultaneous 51% attacks and attacks on the social layer (and market consensus tools such as exchanges). One can imagine an implementation of (1) where nodes automatically accept a switch to a new validator set if they do not see a new block being committed for a long enough time, which would reduce the need for social coordination but at the cost of requiring those nodes that do not wish to rely on social coordination to remain constantly online. In either case, a solution can be designed where attackers take a large hit to their deposits.

在選擇攻擊應對的策略時，在交際協調和機制內自動化之間其實有如一道光譜，並不是非黑即白，通常設計越可能往自動化的解法越理想，因為這可以降低當 51% 攻擊和社交層面（包含市場共識如交易所）的攻擊同時發生時的風險。可以想像一個措施，採用 1 的方式使節點在超過一定時間都沒有區塊被 commit 時自動更換validator名單，這會降低交際協調的需要但節點也因此要更頻繁地保持上線。但不管是哪種方式，攻擊者都要損失一大筆的存款。

A more insidious kind of attack is a censorship attack, where >= 34% of validators refuse to finalize blocks that contain certain kinds of transactions that they do not like, but otherwise the blockchain keeps going and blocks keep getting finalized. This could range from a mild censorship attack which only censors to interfere with a few specific applications (eg. selectively censoring transactions in something like Raiden or the lightning network is a fairly easy way for a cartel to steal money) to an attack that blocks all transactions.

另外一種比較不容易發覺的攻擊是審查攻擊--當超過 34% 的validator拒絕將含有某些（他們不喜歡的）特定交易的區塊變為不可更動，除此之外鏈的運作都正常。攻擊的範圍從輕微的，干擾特定應用（如過濾 Raiden 或閃電網路的交易是較簡單偷錢的方式）的攻擊到阻擋所有交易的大範圍攻擊。

There are two sub-cases. The first is where the attacker has 34-67% of the stake. Here, we can program validators to refuse to finalize or build on blocks that they subjectively believe are clearly censoring transactions, which turns this kind of attack into a more standard liveness attack. The more dangerous case is where the attacker has more than 67% of the stake. Here, the attacker can freely block any transactions they wish to block and refuse to build on any blocks that do contain such transactions.

其中又分成兩個類別，第一個是攻擊者掌控 34%-67% 的下注資本。我們可以將validator設計成拒絕將他們主觀認定為正在過濾交易的區塊變成不可更動或接在其後，這讓這種攻擊變為一個標準的 liveness 攻擊。比較危險的情況是當攻擊者掌控超過 67% 的下注資本，攻擊者可以任意的阻擋他們不喜歡的交易並拒絕接在包含這些交易的區塊後面。

There are two lines of defense. First, because Ethereum is Turing-complete it is [naturally somewhat resistant to censorship](http://hackingdistributed.com/2016/07/05/eth-is-more-resilient-to-censorship/) as censoring transactions that have a certain effect is in some ways similar to solving the halting problem. Because there is a gas limit, it is not literally impossible, though the "easy" ways to do it do open up denial-of-service attack vulnerabilities.

面對這個攻擊有兩道防線。第一，因為以太坊具有圖靈完備特性，[在本質上就具有抵抗審查的能力](http://hackingdistributed.com/2016/07/05/eth-is-more-resilient-to-censorship/)，因為審查交易的過程在某種程度上相似於解決停機問題（halting problem）。但因為區塊有 gas 限制，所以審查並不是不可能，不過用"簡單"的方式來進行審查會讓攻擊者反而有被DoS的風險。

This resistance [is not perfect](https://pdaian.com/blog/on-soft-fork-security/), and there are ways to improve it. The most interesting approach is to add in-protocol features where transactions can automatically schedule future events, as it would be extremely difficult to try to foresee what the result of executing scheduled events and the events resulting from those scheduled events would be ahead of time. Validators could then use obfuscated sequences of scheduled events to deposit their ether, and dilute the attacker to below 33%.

單純具有這個抵抗能力[還不夠好](https://pdaian.com/blog/on-soft-fork-security/)，還有其他方式可以加強抵抗審查的能力。最有趣的方式是增加一個機制內的功能--讓交易能自動規劃未來的事件，因為預測這些事件的執行結果或是這些事件又再產生的事件是很難的。validator可以藉由混淆事件規劃的順序來下注存款，藉此稀釋攻擊者的佔比到低於 33% 。

Second, one can introduce the notion of an "active fork choice rule", where part of the process for determining whether not a given chain is valid is trying to interact with it and verifying that it is not trying to censor you. The most effective way to do this would be for nodes to repeatedly send a transaction to schedule depositing their ether and then cancel the deposit at the last moment. If nodes detect censorship, they could then follow through with the deposit, and so temporarily join the validator pool en masse, diluting the attacker to below 33%. If the validator cartel censors their attempts to deposit, then nodes running this "active fork choice rule" would not recognize the chain as valid; this would collapse the censorship attack into a liveness denial attack, at which point it can be resolved through the same means as other liveness denial attacks.

第二，引進 "active fork choice rule" 的概念--當面臨鏈分叉情況，其中一個選擇鏈的考量是和鏈進行互動並藉此驗證該鏈是否有在過濾你的交易。最有效的方式是節點重複地送出一筆交易來規劃下注存款並在最後一刻取消。如果節點偵測到審查機制，就不取消並暫時一起加入成為validator之一，將攻擊者的佔比稀釋到 33% 。如果集團過濾掉他們的存款交易的話，則採用這個 "active fork choice rule" 的節點就不會選擇這條鏈。這會讓審查攻擊轉變為 liveness 攻擊，此時就可以藉由解決 liveness 攻擊的方式來處理。
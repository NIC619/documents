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

很重要的一點是，利用存款（deposit，也就是validator下注的資本）來確保"風險成本不為零"的機制確實改變了PoS的安全模型（security model）。假設 (I)存款會被鎖住一個月，時間到之後可以提走， (II)有個 51% 攻擊嘗試反轉（revert）長達 10 天的交易量，則這些攻擊者產生的區塊會被寫入鏈裡當作證據且validator會被懲罰。然而，假設攻擊變成長達 40 天，則雖然這些攻擊產生的區塊可以再被寫入鏈裡，但validator早已能把錢提走而不會受到懲罰。為了要解決這個問題，我們需要一個"反轉限制（revert limit）"的規則，也就是當反轉所影響的區塊總時間長度超過存款鎖住的期限，則節點可以拒絕接受這些區塊（在上例，即拒絕影響超過一個月的反轉區外）。這表示節點現在多了兩項要求：

1. 當節點第一次連上並要同步鏈的資料的時候，他們必須藉由鏈外的方式來驗證最新的狀態，即透過朋友節點們或各個 Block Explorer 等等的方式。如果他們得到的都是同一條一樣的鏈，則可以確定這條是正確的。注意，只有在出現鏈分叉長度超過反轉限制時（在上例，即得到兩條在過去超過一個月的區塊皆不相同的鏈）才需要這種採用這種鏈外的交際驗證（social authentication）。

2. 節點每隔一段"反轉限制"的時間就必須要上線同步。如果沒定時同步，則需要再透過一次鏈外的交際驗證來保證狀態的可信度。

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

可以採用一個自動化的功能來輪轉validator名單，瓦解集團的佔比。在新的validator名單中區塊可以被變為不可更動，但使用者會收到提醒告訴他們這些不可更動的區塊還是不可全信的，因為很有可能舊的一組validator會重新奪回控制權並將其他的區塊變為不可更動。使用者如果確信舊的一組validator不會再上線，就可以手動覆蓋過這些警告。會有規則規定在這種情況發生時，所有舊的validator如果沒有再參與共識過程則會受到相當大筆的罰款。

2. A hard fork is used to add in new validators and delete the attackers' balances.

採用硬分叉的方式移除攻擊者的存款，並增加新的validator。

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

###That sounds like a lot of reliance on out-of-band social coordination; is that not dangerous?

###聽起來似乎很仰賴鏈外的社交協調，這樣難道不危險嗎？

Attacks against Casper are extremely expensive; as we will see below, attacks against Casper cost as much, if not more, than the cost of buying enough mining power in a proof of work chain to permanently 51% attack it over and over again to the point of uselessness. Hence, the recovery techniques described above will only be used in very extreme circumstances; in fact, advocates of proof of work also generally express willingness to use social coordination in similar circumstances by, for example, [changing the proof of work algorithm](https://news.bitcoin.com/bitcoin-developers-changing-proof-work-algorithm/). Hence, it is not even clear that the need for social coordination in proof of stake is larger than it is in proof of work.

攻擊 Casper 代價非常高。以下我們將會講到，攻擊 Casper 的代價至少和買礦機持續不斷的對PoW鏈發動 51% 攻擊直到無效果為止的代價一樣。因此，上面段落所描述的復原方法只有在非常極端的情形才會用到。事實上，PoW的提倡者亦表達在某些相似情況採用社交協調的意願，例如[改變PoW的算法](https://news.bitcoin.com/bitcoin-developers-changing-proof-work-algorithm/)。所以PoS需要的社交協調是否會比PoW所需要的社交協調還多還無明確的結果。

In reality, we expect the amount of social coordination required to be near-zero, as attackers will realize that it is not in their benefit to burn such large amounts of money to simply take a blockchain offline for one or two days.

在現實中，我們預期用到社交協調的方式的次數會接近零次，因為攻擊者會瞭解到單純為了讓區塊鏈停擺一兩天要花費這麼大量的錢是不符合利益的。

###Doesn't MC => MR mean that all consensus algorithms with a given security level are equally efficient (or in other words, equally wasteful)?

###邊際成本趨近邊際收益不就表示所有具有一定高度安全層級的共識演算法都一樣有效（或一樣地浪費）？

This is an argument that many have raised, perhaps best explained by [Paul Sztorc in this article](http://www.truthcoin.info/blog/pow-cheapest/). Essentially, if you create a way for people to earn $100, then people will be willing to spend anywhere up to $99.9 (including the cost of their own labor) in order to get it; marginal cost approaches marginal revenue. Hence, the theory goes, any algorithm with a given block reward will be equally "wasteful" in terms of the quantity of socially unproductive activity that is carried out in order to try to get the reward.

許多人都提出過這個論點，而解釋最清楚的就屬[Paul Sztorc 的這篇文章(http://www.truthcoin.info/blog/pow-cheapest/)。其中的重點是，如果你創造一個有 100 元獎賞的機會，則大家為了得到它會願意花費最高到 99.9 元（包含自己付出的勞力），此時邊際成本趨近邊際效益。因此，這個理論說--任何提供區塊獎賞的演算法，其中為了獲取獎賞而進行對社會無效益的活動的數量都是一樣的，即它們都一樣地浪費資源。

There are three flaws with this:

1. It's not enough to simply say that marginal cost approaches marginal revenue; one must also posit a plausible mechanism by which someone can actually expend that cost. For example, if tomorrow I announce that every day from then on I will give $100 to a randomly selected one of a given list of ten people (using my laptop's /dev/urandom as randomness), then there is simply no way for anyone to send $99 to try to get at that randomness. Either they are not in the list of ten, in which case they have no chance no matter what they do, or they are in the list of ten, in which case they don't have any reasonable way to manipulate my randomness so they're stuck with getting the expected-value $10 per day.

這個理論有三個盲點：

單純地說邊際成本趨近邊際效益是不夠的，必須還要假設一個有人可以真的花費那些成本的機制存在。例如，假設我明天宣布之後每一天我都會隨機地從一個十人名單中挑出一個人並給予他 100 元，然而沒有人有辦法花費 99 元來取得其中的隨機值。他們要不是不在名單中不管怎樣都拿不到獎賞，要不就是在名單中但沒有任何有效的方法取得我的隨機值，只有期望值為平均每天 10 元的獎賞。

2. MC => MR does NOT imply total cost approaches total revenue. For example, suppose that there is an algorithm which pseudorandomly selects 1000 validators out of some very large set (each validator getting a reward of $1), you have 10% of the stake so on average you get 100, and at a cost of $1 you can force the randomness to reset (and you can repeat this an unlimited number of times). Due to the [central limit theorem](https://en.wikipedia.org/wiki/Central_limit_theorem), the standard deviation of your reward is $10, and based on [other known results in math](http://math.stackexchange.com/questions/89030/expectation-of-the-maximum-of-gaussian-random-variables) the expected maximum of N random samples is slightly under M + S * sqrt(2 * log(N)) where M is the mean and S is the standard deviation. Hence the reward for making additional trials (ie. increasing N) drops off sharply, eg. with 0 re-trials your expected reward is $100, with one re-trial it's $105.5, with two it's $108.5, with three it's $110.3, with four it's $111.6, with five it's $112.6 and with six it's $113.5. Hence, after five retrials it stops being worth it. As a result, an economically motivated attacker with ten percent of stake will inefficiently spend $5 to get an additional revenue of $13, though the total revenue is $113. If the exploitable mechanisms only expose small opportunities, the economic loss will be small; it is decidedly NOT the case that a single drop of exploitability brings the entire flood of PoW-level economic waste rushing back in. This point will also be very relevant in our below discussion on capital lockup costs.

邊際成本趨近邊際效益不表示總成本趨近總收益。例如，假設存在一個演算法利用偽隨機（pseudo-randomly）的方式從一大群validator中選擇 1000 位validator(一個validator獲得 1 元獎賞)，你資本佔總資本的 10% 所以你平均會獲得 100 元，且你可以用花費 1 元來（無限次地）重設隨機值。因為 [central limit theorem](https://en.wikipedia.org/wiki/Central_limit_theorem)，你可獲得的獎賞的標準差是 10 元，又因為[其他已知的數學結論](http://math.stackexchange.com/questions/89030/expectation-of-the-maximum-of-gaussian-random-variables)， N 個隨機抽樣中最大的期望值約略小於 ` M + S * sqrt(2 * log(N))` ，其中 M 是中間值且 S 是標準差。因此增加重設隨機值的次數（即增加 N ）所獲得的獎賞會快速地下降，例如如果完全不嘗試重設隨機值你的期望獲利是 100 元，如果嘗試一次是 105.5 元，兩次是 108.5 元，三次是 110.3 元，四次是 111.6 元，五次是 112.6 元，六次是 113.5元（只增加 0.9 元的獲利）。因此嘗試超過五次之後就不值得再繼續嘗試。所以一個由經濟因素所驅動的攻擊者，如果他總資本佔 10% ，則他會花費 5 元嘗試重設隨機值來獲得額外的 13 元的獲利，即便這麼做很沒效率。如果一個機制可被有心人士利用，但被利用的機率不高，則損失不會多，但這不適用於出現一個漏洞而導致全部 PoW 資源投入造成浪費的情況。而這點也和下一章節要介紹的 capital lockup costs 非常相關。

3. Proof of stake can be secured with much lower total rewards than proof of work.

PoS 要變得安全穩固所需要的獎賞比起 PoW 的獎賞少的非常多。

###What about capital lockup costs?

Locking up X ether in a deposit is not free; it entails a sacrifice of optionality for the ether holder. Right now, if I have 1000 ether, I can do whatever I want with it; if I lock it up in a deposit, then it's stuck there for months, and I do not have, for example, the insurance utility of the money being there to pay for sudden unexpected expenses. I also lose some freedom to change my token allocations away from ether within that timeframe; I could simulate selling ether by shorting an amount equivalent to the deposit on an exchange, but this itself carries costs including exchange fees and paying interest. Some might argue: isn't this capital lockup inefficiency really just a highly indirect way of achieving the exact same level of economic inefficiency as exists in proof of work? The answer is no, for both reasons (2) and (3) above.

將 X 的 ether 鎖進存庫是有代價的，例如犧牲選擇其他選項的機會。如果我有 1000 ether ，我想怎麼使用就怎麼使用，但如果我將它鎖在存庫裡數個月，而且沒有保險來支付可能的意外支出的時候該怎麼辦。在這段時間我同時也失去從 ether 轉移成其他代幣的自由。我可以透過賣空相等數量 ether 的方式來模擬賣掉我的 ether，但其中隱含了交易手費續和利息。有些人可能會認為： captital lockup 造成的經濟上的不便和 PoW 造成的經濟層面上無效率的程度不是一樣嗎？答案是否定的，如同上一章節的 理由(2) 和 理由(3)。

Let us start with (3) first. Consider a model where proof of stake deposits are infinite-term, ASICs last forever, ASIC technology is fixed (ie. no Moore's law) and electricity costs are zero. Let's say the equilibrium interest rate is 5% per annum. In a proof of work blockchain, I can take $1000, convert it into a miner, and the miner will pay me $50 in rewards per year forever. In a proof of stake blockchain, I would buy $1000 of coins, deposit them (ie. losing them forever), and get $50 in rewards per year forever. So far, the situation looks completely symmetrical (technically, even here, in the proof of stake case my destruction of coins isn't fully socially destructive as it makes others' coins worth more, but we can leave that aside for the moment). The cost of a "Maginot-line" 51% attack (ie. buying up more hardware than the rest of the network) increases by $1000 in both cases.

我們先從 理由(3) 開始講起。考慮一種模型：PoS存款是無限期的、ASICs可以永久持續運作、ASIC技術固定（即不適用摩爾定律）而且電力花費是零，並假設穩定的利率是每年 5%。在 PoW 鏈裡，我花 1000 元買了礦機，礦機每年給我 50 元的利潤直到永遠。在 PoS鏈中，我存 1000 元（因為存款是無限期的所以這筆錢當作花掉）並獲得每年 50 元的利潤直到永遠。到目前為止，兩種情況看起來是等價的（雖然技術上來說，在PoS中當我的存款因違規被銷毀，會間接造成其他人的存款價值升高，但這邊我們先不討論）。任何人發動 "Maginot-line" 51% 攻擊（即硬體買得比網路其他人加起來還多）的成本在兩種情況都會再加上 1000 元。

Now, let's perform the following changes to our model in turn:

1. Moore's law exists, ASICs depreciate by 50% every 2.772 years (that's a continuously-compounded 25% per annum; picked to make the numbers simpler). If I want to retain the same "pay once, get money forever" behavior, I can do so: I would put $1000 into a fund, where $167 would go into an ASIC and the remaining $833 would go into investments at 5% interest; the $41.67 dividends per year would be just enough to keep renewing the ASIC hardware (assuming technological development is fully continuous, once again to make the math simpler). Rewards would go down to $8.33 per year; hence, 83.3% of miners will drop out until the system comes back into equilibrium with me earning $50 per year, and so the Maginot-line cost of an attack on PoW given the same rewards drops by a factor of 6.

現在假設模型依序做以下的變更：

摩爾定律適用，ASICs 每 2.772 年貶值一次（方便計算，大約是持續每年降低 25%）。如果我要繼續保持 "付一次，永遠都能持續拿回錢" 的模式，我可以：將 1000 變成一筆資金，其中167會花費在 ASICs 上，833 元花在 5% 報酬率的投資。每年 41.67 元的利潤剛好足夠花在更新 ASICs設備上（方便計算，假設技術發展是穩定連續的）。這時利潤會降低至每年 8.33 元，因此 83.3% 的礦工會放棄直到系統回到每年有 50 元利潤的穩定狀態，所以在利潤這麼低的 PoW鏈發動 Maginot-line 51% 攻擊的成本會大幅地縮小（至少六倍）。

2. Electricity plus maintenance makes up 1/3 of mining costs. We estimate the 1/3 from recent mining statistics: one of Bitfury's new data centers consumes [0.06 joules per gigahash](http://www.coindesk.com/bitfury-details-100-million-georgia-data-center/), or 60 J/TH or 0.000017 kWh/TH, and if we assume the entire Bitcoin network has similar efficiencies we get 27.9 kWh per second given [1.67 million TH/s total Bitcoin hashpower](http://bitcoinwatch.com/). Electricity in China costs [$0.11 per kWh](http://www.statista.com/statistics/477995/global-prices-of-electricity-by-select-country/), so that's about $3 per second, or $260,000 per day. Bitcoin block rewards plus fees are $600 per BTC * 13 BTC per block * 144 blocks per day = $1.12m per day. Thus electricity itself would make up 23% of costs, and we can back-of-the-envelope estimate maintenance at 10% to give a clean 1/3 ongoing costs, 2/3 fixed costs split. This means that out of your $1000 fund, only $111 would go into the ASIC, $55 would go into paying ongoing costs, and $833 would go into hardware investments; hence the Maginot-line cost of attack is 9x lower than in our original setting.

電力加上硬體維護組成大約 1/3 的挖礦成本。1/3 是從最近的數據估計而來的：Bitfury 的新資料中心 [每 gigahash 電力消耗為 0.06 焦耳 ](http://www.coindesk.com/bitfury-details-100-million-georgia-data-center/)，或 每 terahash 60 焦耳、每terahash 0.000017千瓦小時。如果假設比特幣網路的平均消耗皆如此的話，以[比特幣總共算立約 1.67 million TH/s](http://bitcoinwatch.com/) 來算每秒約消耗 27.9 千瓦小時。中國電力成本為[每千瓦小時 0.11 元 ](http://www.statista.com/statistics/477995/global-prices-of-electricity-by-select-country/)，約為每秒 3 元或每天 260000 元。比特幣區塊獎賞加上手續費為 600 * 13 * 144 = 1.12m，一天 112 萬元。因此電力組成約 23% 的成本，而且我們可以用簡單的計算預估硬體維護成本為 10% 。這表示你的 1000 元資金裡，111 元要拿來付 ASICs，55 元要拿來付持續性的花費如電力和維護等，而 833 元會花在投資上，因此現在要發動 Maginot-line 51% 攻擊的成本已經是一開始的九倍小了。

3. Deposits are temporary, not permanent. Sure, if I voluntarily keep staking forever, then this changes nothing. However, I regain some of the optionality that I had before; I could quit within a medium timeframe (say, 4 months) at any time. This means that I would be willing to put more than $1000 of ether in for the $50 per year gain; perhaps in equilibrium it would be something like $3000. Hence, the cost of the Maginot line attack on PoS increases by a factor of three, and so on net PoS gives 27x more security than PoW for the same cost.

事實上存款是短暫而非永遠（被視為拿不回來）的。當然你自願永遠存著的話也行。然而不同的是，我現在至少多了一些空間選項，我可以在任何時間內結束並等待一段時間（如四個月）。這表示我願意花更多的錢來獲得更多的利潤（因為投資不會再被當作拿不回來的錢），或許是 3000 元。因此，在 PoS 鏈上發動 Maginot line 51% 攻擊的成本增加為原本的三倍，和 PoW 比起來是 27 倍的安全性。

The above included a large amount of simplified modeling, however it serves to show how multiple factors stack up heavily in favor of PoS in such a way that PoS gets more bang for its buck in terms of security. The meta-argument for why this [perhaps suspiciously multifactorial argument](http://lesswrong.com/lw/kpj/multiple_factor_explanations_should_not_appear/) leans so heavily in favor of PoS is simple: in PoW, we are working directly with the laws of physics. In PoS, we are able to design the protocol in such a way that it has the precise properties that we want - in short, we can optimize the laws of physics in our favor. The "hidden trapdoor" that gives us (3) is the change in the security model, specifically the introduction of weak subjectivity.

以上包含了很多簡化過的計算，但它的目的是為了指出經過許多因素考量，都顯示出 PoS 擁有更高的安全性。而這個[看似可疑的多重因素論點](http://lesswrong.com/lw/kpj/multiple_factor_explanations_should_not_appear/)為何這麼強烈凸顯 PoS 的優點的主要理由是：在 PoW 中，我們操作利用物理特性；而在 PoS 中，我們可以設計出一套準確具有我們想要的特性的機制 - 簡而言之，我們可以用我們的方式來優化 "物理特性"。這個讓我們能得到 理由(3) 的 "隱藏的機關" 即是在安全模型上的改變，特別是弱主觀性（weak subjectivity）這個性質的出現。

Now, we can talk about the marginal/total distinction. In the case of capital lockup costs, this is very important. For example, consider a case where you have $100,000 of ether. You probably intend to hold a large portion of it for a long time; hence, locking up even $50,000 of the ether should be nearly free. Locking up $80,000 would be slightly more inconvenient, but $20,000 of breathing room still gives you a large space to maneuver. Locking up $90,000 is more problematic, $99,000 is very problematic, and locking up all $100,000 is absurd, as it means you would not even have a single bit of ether left to pay basic transaction fees. Hence, your marginal costs increase quickly. We can show the difference between this state of affairs and the state of affairs in proof of work as follows:

接者我們可以討論 邊際成本/效益 和 總成本/效益 的不同。在 capital lockup costs 的情況中這非常重要。例如假設一個情況，你有價值 100000 的 ether，你會傾向長久持有絕大部分的錢。因此鎖住其中的 50000 應該不會有什麼問題，鎖住 80000 可能會有一點不方便雖然 20000 還是有夠充足可以利用，鎖住 90000 就有點問題了，99000 問題就大了，全鎖住那你可能是瘋了，因為你會連交易費都出不起。因此你的邊際成本快速的增加，我們可以用以下方式比較 PoS 情況和 PoW 情況的不同：

![](https://blog.ethereum.org/wp-content/uploads/2014/07/liquidity.png)

Hence, the total cost of proof of stake is potentially much lower than the marginal cost of depositing 1 more ETH into the system multiplied by the amount of ether currently deposited.

可以看到，在 PoS 現有的總存款中再存入 1 ether 的總成本是遠低於邊際成本的。

Note that this component of the argument unfortunately does not fully translate into reduction of the "safe level of issuance". It does help us because it shows that we can get substantial proof of stake participation even if we keep issuance very low; however, it also means that a large portion of the gains will simply be borne by validators as economic surplus.

注意，這部分的論點可惜地並沒有完全解釋到 "safe level of issuance"。但其顯示出即使我們將貨幣發行量控制地很低，我們還是能獲得可觀的 PoS 參與數，雖然這也代表很大一部分的發行量會被用來當作獎賞驗證者。

###Will exchanges in proof of stake pose a similar centralization risk to pools in proof of work?

###在 PoS 的礦池會有類似 PoW 礦池中心化的風險嗎？

From a centralization perspective, in both [Bitcoin](https://blockchain.info/pools) and [Ethereum](https://etherscan.io/stats/miner?range=7&blocktype=blocks) it's the case that roughly three pools are needed to coordinate on a 51% attack (4 in Bitcoin, 3 in Ethereum at the time of this writing). In PoS, if we assume 30% participation including all exchanges, then [three exchanges](https://etherscan.io/accounts) would be enough to make a 51% attack; if participation goes up to 40% then the required number goes up to eight. However, exchanges will not be able to participate with all of their ether; the reason is that they need to accomodate withdrawals.

從 [Bitcoin](https://blockchain.info/pools) 和 [Ethereum](https://etherscan.io/stats/miner?range=7&blocktype=blocks) 來看，大約需要三個礦池聯合才能發動 51% 攻擊（ 在寫這篇文章的時候，Bitcoin 約需四個、Ethereum 約需三個）。假設在 PoS 包含所有礦池、交易所共有 30% 的參與率，則 [三個礦池、交易所](https://etherscan.io/accounts) 就足夠發動 51% 攻擊；如果參與率提高到 40% 則需要八個。另外礦池、交易所也不能將所有資本投入攻擊中，因為他們需要保留部分金錢來應付客戶。

Additionally, pooling in PoS is discouraged because it has a much higher trust requirement - a proof of stake pool can pretend to be hacked, destroy its participants' deposits and claim a reward for it. On the other hand, the ability to earn interest on one's coins without oneself running a node, even if trust is required, is something that many may find attractive; all in all, the centralization balance is an empirical question for which the answer is unclear until the system is actually running for a substantial period of time. With sharding, we expect pooling incentives to reduce further, as (i) there is even less concern about variance, and (ii) in a sharded model, transaction verification load is proportional to the amount of capital that one puts in, and so there are no direct infrastructure savings from pooling.

再加上組成 PoS 的礦池的動機是較低的，因為會需要較高的信任成本 - 礦池不偷錢，但是可以假裝被駭導致資本全被沒收，且同時扮成檢舉者領檢舉獎金。不過另一方面，不需要自己跑一個就能用自己的錢賺利潤仍是很有吸引力的，即便需要信任對方。總之，中心化和去中心化間的平衡是需要經驗才能找到答案的，也只有等到系統真的上線一段時間了才有辦法得到答案。但配上 sharding 之後，我們預計會更近一步降低中心化，因為 (i) 需要考量的變化更少且 (ii) 在 sharding 模型中，驗證交易的負擔和所投入的資本成正比，所以並不會因為組成礦池而有任何直接的成本支出的節省。

A final point is that centralization is less harmful in proof of stake than in proof of work, as there are much cheaper ways to recover from successful 51% attacks; one does not need to switch to a new mining algorithm.

最後一點是，中心化在 PoS 中比其在 PoW 中的負面影響更小，因為從 51% 攻擊中復原容易且便宜多了，也不必要換到新的挖礦演算法。

###Can proof of stake be used in private/consortium chains?

###PoS 能被用在私有鏈或聯盟鏈嗎？

Generally, yes; any proof of stake algorithm can be used as a consensus algorithm in private/consortium chain settings. The only change is that the way the validator set is selected would be different: it would start off as a set of trusted users that everyone agrees on, and then it would be up to the validator set to vote on adding in new validators.

一般來說是可以的。任何 PoS 演算法都可以被私有鏈或聯盟鏈用做一個共識演算法。唯一不同是成為驗證者的方式：一開始會由一群大家同意且信任的使用者成為驗證者，接著再由這群驗證者透過投票去加入新的驗證者。
### Casper The Friendly Finality Gadget

這份文件描述以太坊上的權益證明機制（Proof of Stake） - Casper 可能的實現方式。這個提議主要希望能達成具有 (1)高度安全的 finality 及 (2)低成本共識演算法的權益證明機制，同時又不會對現有的工作量證明機制（Proof of Work）的以太坊區塊鏈造成太劇烈的影響。我們將會介紹演算法的運作、如何在部分同步（partially synchronous）環境裡的容錯理論模型中達成 safety 及 liveness 兩個重要特性，接著介紹不同賽局理論裡激勵獎賞的考量、系統如何從超過 1/3 節點下線的情況下復原、藉由鏈下輕量的合作方式從 各種 51% 攻擊中復原。我們將會逐步描述越加複雜的演算法，從核心概念開始，到驗證者輪流方式及經濟激勵獎賞。

### 背景

權益證明機制被視為同時具有巨大潛力及爭議、以安全密碼經濟學、取代工作量證明機制的公有鏈共識機制已經有很長一段時間。工作量證明機制以計算量來衡量經濟共識，而權益證明機制簡單來說則是以[*虛擬挖礦](#)取代實體的 CPU、GPU、ASIC挖礦，其經濟共識則是以系統內 commit 的經濟資源多寡來衡量。

但是早期的權益證明機制受到"[*無成本風險（nothing at stake）](#)"的問題所困擾，這個問題描述的是：如果直接複製工作量證明機制的共識來建立權益證明機制的話，出來的成果會是，當鏈出現分叉時，理性的參與者會選擇兩條鏈都接受，最後導致沒有一條統一的鏈。不像在工作量證明機中，_外部_的資源只能用在其中一條分叉；在簡單的權益證明機制中出現分叉的話，代表兩條分叉上皆有等量的經濟資本的投入，所以驗證者可以用他在 A 鏈上的資本驗證支持 A 鏈，同時又以他在 B 鏈上的資本驗證支持 B 鏈。

工作量證明機制
<img src="https://raw.githubusercontent.com/vbuterin/diagrams/master/powsec.png" width="400px"></img>  
權益證明機制
<img src="https://raw.githubusercontent.com/vbuterin/diagrams/master/possec.png" width="400px"></img>

Casper 建構在 2014 年早期開始的 [Slasher](#) 概念，其概念嘗試透過偵測明顯的 equivocation 行為（一個拜占庭容錯理論中常見的術語，描述的是同時送出相抵觸的訊息來支持不同條分叉）並以經濟上的懲罰來遏止驗證者這麼做。

<img src="https://raw.githubusercontent.com/vbuterin/diagrams/master/slasher1sec.png" width="400px"></img> 

這個概念能解決無成本風險（但有極弱的同步網路環境的假設，後面會介紹的 weak subjectivity）的問題並確保這種權益證明機制至少和工作量證明機制一樣安全。不過我們還能再更進一步 - 很快 Vlad Zamfir 就發現以懲罰為基礎的權益證明機制遠比以獎賞為基礎的權益證明機制還安全，因為這兩者本身並不是對稱的特質。因為獎賞是由協議本身產生，所以其數量受到限制（想像協議設定獎賞為無上限或隨意給，則獎賞的價值將會大大降低）。然而懲罰則遠高得多，甚至可以高到等於協議的全部資本。

這帶來了 _economic finality_ 的概念：

>  如果互相衝突的區塊或狀態（state）也變為不可更改，則存在有證據可以用來對造成這個衝突的相關參與者罰與 X 元的罰金。在這個條件成立下我們可以說一個區塊或狀態可被視為不可更改。其中 X 被稱為終局機制（finality mechanism）裡的 _cryptoeconomic security margin_ 。

但是這種以懲罰為基礎的權益證明機制的形式會遇到另一個問題：“被卡住”的可能：

![](https://cdn-images-1.medium.com/max/800/1*ftuBRQnM8v1kC0Lnvsh3zQ.jpeg)

設計不良的演算法可能會導致沒有區塊能被 finalized ，而且沒有人會因此被懲罰。要設計一個具有精巧的終局特質又同時不會卡住的演算法是個困難的挑戰 - 但至少這個問題已經在拜占庭容錯理論中被研究了數十年了。

像是 PBFT、Paxos 和 [*HoneyBadger BFT](#) 這些演算法都試著完成相似的目標 - 在數群節點（有時稱做程序）間達成共識。一個早期嘗試定義這個問題的方式是利用拜占庭將軍問題：一群將軍試著要擬定出決策來攻打一座城市，但這些將軍中存在某些叛徒。進攻計劃有兩個目標：

A. 所有忠誠的將軍會採用相同的決策
B. [*只有少數的叛徒將沒辦法讓忠誠的將軍採用危險的決策](#)

在現實生活中的共識演算法，要擬定的決策為：operation 執行的順序。

在我們的例子中，我們的目標並不只是在單一輪的共識中決定某個值的結果，而是在持續增長的鏈上不斷出現新的一輪共識。在一個區塊鏈中，每個區塊都包含前一個區塊的雜湊值。所以本質上區塊是和全部的祖先區塊都鏈結在一起。在一個新的區塊上達成共識同時也代表對同一條，整條區塊鏈達成共識。因此，共識演算法不只是必須避免同時在兩個相衝突的區塊都達成共識，還要避免在一個和祖先區塊相衝突的區塊上達成共識。即要在一個區塊上達成共識，該區塊不只要和最新的這一區塊能相容，還要和它所有祖先區塊都相容。

![](https://cdn-images-1.medium.com/max/800/1*ARu6mWJ2_oWXZR0UB13hkQ.jpeg)

接著我們會先介紹 "Minimal Slashing Conditions"，一個具有上述特質，理論上同時也能在其他情況中用來取代 PBFT 的系統。

### Minimal Slashing Conditions

這個演算法有個假設前提是有一個 **proposal mechanism** ，來建立新的區塊且有辦法以 deteministic 的方式找到一條鏈最新的區塊。鏈可能會很完美的一個接一個新增區塊，也有可能發生短暫分叉，或甚至是長時間分叉導致最新的區塊不斷在不同分叉間變動。

我們不需要設計優良的 proposal mechanism 來確保能達成 safety 特質。不管 proposal mechanism 表現得多差，只要超過 2/3 的節點都遵循協議，相衝突的檢查點（checkpoint）就不會都被 finalize 。但 liveness 特質就有可能會受影響。

我們刻意不把 proposal mechanism 具體描述出來，有可能是用獨裁的方式、或 round robin，或是像我們的 hybrid Casper，沿用原本的工作量證明機制來產生新的區塊。

每隔一百個區塊的區塊會被稱做檢查點，兩個檢查點之間的週期稱做一個 **epoch**。我們假設會存在一組驗證者： V<sub>1</sub> ... V<sub>n</sub>，每個驗證者都有不同 size： S(V<sub>1</sub>) ... S(V<sub>n</sub>)，每個驗證著都必須將押一筆以太幣，其多寡決定該驗證者的 size。

驗證者可以送出兩類型的訊息：

    [PREPARE, epoch, hash, epoch_source, hash_source]
    [COMMIT, epoch, hash]

在第 `n` 個 epoch，驗證者等 proposal mechanism 提出新的檢查點區塊（假設其雜湊值為 `H`），其中 `epoch_source` 和 `hash_source` 要是最近一個有超過 2/3 `PREPARE` 的 epoch 和 該 epoch 的雜湊值（`sum_{v in PREPSET} S(V) >= sum_{v in ALL_VALIDATORS} S(V) * 2/3`，即送出PREPARE的驗證者的 size 總和要超過全體驗證者 size 總和的 2/3，接下說的比率指的都是驗證者的 size 而不是人數）。當超過 2/3 的驗證者送出以下的 PREPARE 訊息時： epoch 為 `n` 、雜湊值為 `H` 且 有一樣的 `epoch_source` 和 `hash_source`，驗證者才可以接著送出  epoch 為 `n` 、雜湊值為 `H` 的 COMMIT 訊息。如果超過 2/3 的驗證者送出了這樣的 COMMIT 訊息，則該檢查點可被視為 finalized。

雖然我們說驗證者應該要按照以上規則執行，但在很多情況裡是沒有辦法強制他們這麼做的。例如當 proposal mechanism 自己發生分叉導致在 epoch `n` 有相衝突的檢查點 C1 和 C2 出現。假設一個驗證者先看到 C1 ，五秒後才看到 C2，則根據剛剛的規則，他應該要對 C1 做 PREPARE，但如果他改為對 C2 做 PREPARE，這是沒辦法被察覺的。因為我們沒辦法確定他到底是先看到哪一個檢查點。

但我們可以做的是找出一些有明顯證據能證明驗證者亂來的特定情況，嚴厲的懲罰他們，甚至可以把他們押的以太幣全數沒收。

所以我們可以訂出一組規則 - "slashing conditions"，當驗證者觸發其中的條件時會受到懲罰。這些條件如下：

[*複製minimal slashing condition的規則](#)

接著，我們證明這個系統的兩個特質：

1. * **可咎責的安全性（Accountable safety）** - 如果相互衝突的雜湊值都被 finalized，則至少有 1/3 的驗證者肯定違反了某些刪砍條件。
2. * **Plausible liveness** - 除非至少 1/3 的驗證者違反了某些刪砍條件，否則必定存在某些合法的訊息是 2/3 的驗證者可以送出，並用來 finalize 檢查點的。即除非至少 1/3 的驗證者違規，否則一定可以讓新的檢查點被 finalize。

[這裏](https://medium.com/@pirapira/formal-methods-on-some-pos-stuff-e309775c2ab8)有以 Isabelle 寫的正規化驗證的證明。而證明大致如下：

假設兩個相衝突的雜湊值 C1 和 C2 被 finalized 了。則這表示在某個 epoch e1，C1 有 2/3 的 PREPARE 和 2/3 的 COMMIT；在某個 epoch e2，C2 也有 2/3 的 PREPARE 和 2/3 的 COMMIT（如果少了其中的 2/3 PREPARE，那就是違反 `PREPARE_REQ` 的刪砍條件）。首先先假設簡單的狀況： e1 = e2。此時有 1/3 的驗證者同時 PREPARE 了 C1 和 C2 而違反了 `NO_DBL_PREPARE` 的條件。

接著考慮 e2 > e1。因為要符合 `PREPARE_REQ`，所以 C2 的 2/3 PREPARE 表示在某個更之前的 epoch e2'(e2' < e2) 同樣也有 2/3 的 PREPARE（因為 PREPARE 的 epoch_source 也要有超過 2/3 的 PREPARE）。而同樣地 e2' 的 2/3 PREPARE 又表示還有 epoch e2'' 也有超過 2/3 PREPARE ...，直到以下其中一個情況：

(i) e2\* = e1，這裏 C1 有 2/3 PREPARE，C2 的祖先（即 e2\* 的檢查點）也有 2/3 PREPARE，因此有 1/3 的驗證者會因為違反 `NO_DBL_PREPARE` 而被懲罰。
(ii) e2\* < e1，這裏 C1 有 2/3 COMMIT，C2 有 2/3 PREPARE，但 C2 的 source epoch e2\* 比 e1 還早，代表有 1/3 的驗證者會因為違反 `PREPARE_COMMIT_CONSISTENCY`而被懲罰。

plausible liveness 的特質更容易證明。假設 (i) P 是擁有 2/3 PREPARE epoch中的最新的 epoch，且 (ii) M 是最新的 epoch。由 (i) 可知，比 P 還新的 epoch 都沒有合法的 COMMIT（因為都沒有超過 2/3 的 PREPARE），所以 2/3 的驗證者可以對 M+1 的 epoch 做 PREPARE（其中epoch_source 為 P） 而不會被懲罰，之後也可以安全地 COMMIT 該值。

### Hybrid fork choice rule
notes on vlad's take on consensus safety

## Consensus safety, estimate safety

### Consensus safety
如果兩個節點不可能會做出不一樣的決定（decision），則稱共識（consensus）是安全（safe）的。

區塊鏈有 consensus safety 嗎？採用 PoW 共識的區塊鏈沒有 finality，但隨著 confirmation 數量越多，歷史越難被推翻，也就越安全。你也可以自己新增決定規則（decision rule）例如 - 超過一百個 confirmation 的區塊紀錄就不會被推翻 - 來達成 consensus safety。

透過新增決定規則來達成 consensus safety 可以將 blockchain safety 和 consensus safety 做連結。在區塊鏈裡，consensus safety 是有同步環境的假設存在的，即當延遲太嚴重會造成大量區塊被推翻，致使 consensus safety 無法達成。不過至少節點在衡量目前他認知中的共識的安全度是不需考量同步或非同步環境的。

### Estimate safety
我們將節點“衡量目前他認知中的共識的安全度”稱為估計（estimate），例如 - 在最長鏈的區塊的雜湊值是“0x123123123...” - 是一個估計。如果協議（protocol）在未來的狀態，節點計算出不同的估計的機率趨近於零，則這個估計是安全的。其中協議的未來狀態指的是協議從初始狀態經過演變而可能產生的各種狀態（例如一個狀態可以是拜占庭節點數量小於總數的 1/3 ）。

### Consensus safety and Estimate safety
決定規則只有在 e1 是判定是安全的時候才會選擇（decide on）e1這個估計。

如果兩個遵守規則的節點收到相同的訊息（same set of messages），則他們的估計會是一樣的。

如果節點 A 在收到節點 B 的 view（即 set of messages）之後沒有改變他的估計，且反之亦然時，則 A 和 B 的估計是一樣的。

如果(1)節點 A 在某個狀態下的估計 e1 是安全的，且(2)節點 B 在另外某個狀態下的估計 e2 是安全的，且(3)兩者的 view 的聯集存在這些狀態之中，則 e1 = e2。

“All estimate safety decision rules (for some context) have consensus safety (for that context)”
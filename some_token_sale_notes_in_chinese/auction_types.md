## 1. English Auction
又稱為 **open ascending price auction**。可說是最常見的拍賣競標方式。買家彼此出價競爭，由最高出價者勝出。
* 價格是逐漸升高的。

## 2. Dutch Auction
又稱為 **open descending price auction**。賣家先設定最高賣價，由此遞減，直到全數賣完。另外也有人將統一售價方式的 Dutch Auction 稱為modified Dutch Auction。例如一家公司預計出售1000股，A、B 和 C 各以不同單價和不同數量買完1000股，則一種方式是以先後順序、各自的標價售出給A、B 和 C；另外一種方式則是三人都以最低者或最後一筆成交交易的出價購買，例如假設三人裡平均單價出價最低或最後一個成交（買完剩下股份）的是 C，則三人最後的購買價格都是 C 的出價（平均單價）而非自己一開始的出價。
* 價格是逐漸降低的。

## 3. Sealed first-price Auction
又稱為 **blind auction** 或 **first-price sealed-bid auction**。每一位潛在買家同時、將自己的競標價以保密的方式送出，最後同時揭露，出價最高的人成交。

## 4. Vickery Auction
又稱為 **sealed-bid second-price auction**。和 sealed first-price auction 一樣，只是出價最高的人不是以該價格購買，而是以第二高的出價購買。
* 通常用在自動化的競標場景。

___

## Reverse Auction
和一般常見的拍賣相反，買家和賣家的處境交換。在一般的拍賣中，買家彼此競爭，賣家尋找最高的買價；而在 reverse auction 中，賣家彼此競爭，買家尋找最低的買價。

## Reverse Dutch Auction
和 Dutch Auction 相反（廢話），只是買家和賣家的處境交換。商品的需求 > 1，賣家（們）從最低價開始出售，往上增加，直到買家的上限為止，或是直到商品的需求都被滿足。如果最低售價的賣家都賣完了還有需求，由第二低售價的賣家繼續滿足商品需求。
[source](https://blog.ethereum.org/2016/01/15/privacy-on-the-blockchain/)

## 完美的隱私
透過黑箱（black box）obfuscation來保護執行邏輯
* 完美的 black box obfuscation 已被[數學聲明不可能](https://www.iacr.org/archive/crypto2001/21390001.pdf)

或藉由比較弱的版本：[indistinguishabliity obfuscation](https://eprint.iacr.org/2013/451.pdf)。讓其他人無法分辨結果是從哪個程式執行得到的，例如以下兩個程式：
```
y = 0
y = sign(privkey, 0) - sign(privkey, 0)
```
由結果來看都是零，但第二個程式的執行包含了私鑰的使用和簽章的動作。

文中舉了一個例子：可以建立一個合約，其中的資料都經過私鑰加密，傳進來的請求不分讀寫也都要經過加密。
* 但問題是：不能從合約程式裡推出私鑰嗎？

另外一個問題是，這種程式計算太複雜。
  

## 尋找次佳的解法
1. ### Secure Multi-Party Computation
SMPC比obfuscation簡單（例子包含[MIT的Enigma](http://www.enigma.co/enigma_full.pdf)和[secret sharing DAO](https://blog.ethereum.org/2014/12/26/secret-sharing-daos-crypto-2-0/)）且有效率多了。但缺點是：
* 必須相信這些multi-party，而且即便當下是可信的，未來這些資料還是可以被出賣，重點是沒人能知道有人破解了計算。所以V覺得SMPC比較適合用在私鏈。
* 其計算的 overhead 還是很高（其中的 **degree reduction** 步驟需要網路每個節點彼此間傳遞訊息，O(n^2)複雜度。隨然有最近技術能將複雜度降到O(n)但對網路的負擔還是不小）

2. ### zkSNARKs
TODO: [ZCASH](https://z.cash/blog/snark-explain.html)

## 較低階的解法
1. 用public key來達成pseudo-anonymity
2. 使用one-time account來達成匿名的目的，但不小心使用還是有可能會讓有心人士可以連結帳號彼此之間的關係。
3. 將多筆交易合併成一筆，如 CoinJoin。讓其他人不知道你的錢是要轉給哪個帳戶。

## Ring Signature
在不揭露身份的前提下，證明你的私鑰是簽章的一部分。升級版的linkable ring signature則是在同一把私鑰重複簽名時可以被偵測到。
    

## Secret Sharing and Encryption
區塊鏈本身不提供隱私性，但提供真實性的驗證。所以要以區塊鏈當作某事物的證明但又不想失去隱私最簡單的作法就是加密再提供相關人士解密私鑰。如果每次都只想分享部分的資訊，可以用 deterministic wallet，將不同部分分別用不同的 child key 來加密。

另一個方法是 secret sharing，設定 n 個人中 只要有 m 個合作就能組出 secret。

## future of privacy
另外也可以在交易裡用修改一堆不相關資料的方式來混淆他人，讓你修改的目標不易被得知。但同時這樣做成本代價很高，所以效果有限，也因此這也是區塊鏈隱私議題目前的一個挑戰：總有一些資訊是可以被觀察到、並被做連結，甚至最後暴露出身份的。

另一個問題則是，這些提高隱私的技術會給開發者帶給不少麻煩。
distributed randomness 的問題包含(1)答案提前揭露：使得最後的隨機值能被操縱，(2)攻擊者以不揭露或其他方式改變影響隨機值。
這裏紀錄一些基礎密碼學的技術和嘗試解決這些問題的技術的介紹。

## [Schnorr (multi-) Signature](https://en.wikipedia.org/wiki/Schnorr_signature) 

## Publicly Verifiable Secret Sharing(PVSS)
___
類似threshold signature或multy party computation的概念，在一組總數為 n 的團體中，只要其中任何 m 個誠實成員一起就能完成目標如簽章、計算或還原等。 publicly verifiable 則是利用crypto signature的不可否認特性來確保成員提供的資料正確性是可驗證的。
* O(n^3) 複雜度

## RandShare
___
假設一個群：G、其generator：g、
假設 m of n：從 n 個參與者中的 m 個誠實參與者可以還原出 secret。每個參與者（server）都各自產生一個 m 次多項式，常數項為secret，將多項式帶入不同值傳給其他不同的 n-1 組server，由這些server其中 m 個就可還原出多項式並計算出secret。最後搜集所有server的secret並XOR起來得到隨機值。註1：在傳給其他 n-1 組server時，不只有傳多項式帶入的值（假設為Xi），還要廣播一組A1-Am（其中Ai = g^Xi）來當作Xi值的驗證。註2：m = f + 1，f是假設的惡意參與者數量。
* 有 BFT 惡意節點數量的限制
* 非同步環境

## RandHound
＿＿＿
RandHound藉由將server分成更小的組別的方式來改善RandShare無法scale的問題，將複雜度從 O(n^3) 將為O(n*c^2)，其中c為組別中的成員數量（為一常數），藉由鴿籠原理來保證至少有一組回傳的結果是安全可信的（每組平均誠實的server數(a)是誠實server總數(h)除以組數(m)，藉由設定復原secret的threshold值來確保至少m組中有一組是可信的）。

### RandHound - thread model
＿＿＿
* client和server總數為 3f+1 ，其中最多有 f 個拜占庭節點（如同 BFT 的設定）
* 攻擊者的攻擊形式為影響隨機值、DoS攻擊，client 亦可為惡意
* 假設client只能使用RandHound一次，server會為每個請求紀錄一個相關的組態檔（a seesion configuration file）來避免重複請求

client會將請求、收到和送出的每個訊息都記錄下來（一份transcript），任何人都可以藉由執行這份transcript來驗證結果。總共分為七步驟，包含三次client和server間的來回訊息傳遞：

1. 產生一個偽隨機的值來將server分組。
2. share distribution(server)：由組內成員數來決定recover的threshold（成員數的1/3 + 1），生成多項式、commitment、加密過的share和proof，回傳給client。
3. secret commitment(client)：用server公鑰、proof和剛剛產生的commitment對收到的shared secret做驗證commitment，搜集通過驗證的shared secret並產生相對應的Schnorr commit和challenge，最後送出給server。
4. secret acknowledgment(server)：驗證送來的secret數量超過threshold，計算Schnorr response並回傳給client。
5. decryption request(client)：client將Schnorr response合併，列出沒收到的server commit或response，傳給所有server。
6. share decryption(server)：檢查client送來的Schnorr signature，計算是否有超過 2f+1 的server簽名並驗證這些server的secret。通過驗證後將share解密回傳給client
7. raddomness recovery(client)：驗證解密後的share，用 Lagragnge interpolation 復原出secret，最後合併secret來產生隨機值。

## RandHerd

改變client、server溝通方式，將複雜度從 O(n) 再降到 O(logn)。將server分成小組，每輪會有一個小組的領導擔任該輪的領導，由它負責（即 RandHound 裡 client 的角色）。每輪由產生的統一 Schnorr 簽章當作隨機值。

會有一個 configuration file 紀錄所有 server 的相關資訊、公鑰及這些公鑰共同組成的統一公鑰。一開始會需要 (1)隨機選擇一個領導，(2)執行一次 RandHound 來決定分組，(3)之後如果有領導缺席或被要求更換，則按照順序來決定小組領導或總領導的順位，(4)小組計算出小組公鑰再告知總領導來計算統一公鑰，(5)總領導將總公鑰及所屬小組公鑰送給每個 server 做確認，(5)所有 server 確認完後簽名送回，只有收到超過門檻數量的簽名時，setup 才算完成。

* 領導選擇的方式有可能是潛在的攻擊目標
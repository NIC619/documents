## PVSS

## RandShare

## RandHound

RandHound藉由將server分組的方式來改善RandShare無法scale的問題，將複雜度從 O(n^3) 將為O(n*c^2)，其中c為組別中的成員數量（為一常數），藉由鴿籠原理來保證至少有一組回傳的結果是安全可信的（每組平均誠實的server數(a)是誠實server總數(h)除以組數(m)，藉由設定復原secret的threshold值來確保至少m組中有一組是可信的）。

### thread model

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
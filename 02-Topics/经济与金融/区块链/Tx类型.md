> 本文由[简悦 SimpRead](http://ksria.com/simpread/) 转码，原文地址 [medium.com](https://medium.com/@wilsonhuang/mastering-bitcoin-%E7%AD%86%E8%A8%98-standard-transactions-undone-bfb9b4ed0ed8) 

最近因為接了央行的案子，要用 Blockchain 處理票券交割的業務邏輯，需要大量使用 Multi-Signature，於是就回來翻這本書，順便複習 Bitcoin 裡面的五種 Standard Transactions 並寫個筆記。
很簡單的定義，就是 Bitcoin Protocol 裡面會有一個 function，叫做 isStandard()，這裡面定義了五種 Tx(Transaction for short) 的 types。這應該是 Bitcoin network 裡面的礦工 (miner) 在收 Tx 時，會判斷此交易是否為有效的，是否有效會視餘額是否正常，Tx format 是否正常等等，再來就是看是不是 standard transactions，如果都通過且是標準交易的話，才會收進 區塊(block)，否則一律丟掉不管。其餘細節可以去看 Bitcoin Core Client 裡面的 code 是怎麼實作的。
1.  **Pay-to-Public-Key-Hash** (P2PKH，最常見的交易)
2.  **Pay-to-Public-Key** (P2PK)
3.  **Multi-Signature** (MultiSig，多重簽章交易，最多 15 個 keys)
4.  Data Output (OP_RETURN，可以填客製化的資料上 Bitcoin Blockchain)
5.  **Pay-to-Script-Hash** (P2SH)

最常使用到的交易類型，Tx 裡的 locking script 放著 public key 的 hash，也就是所謂常見的 Bitcoin Address。通常要花 Bitcoin 就是要解鎖 Tx 裡的 Output 的 locking script，P2PKH 的 unlocking script 就是一個當初的收錢的人的 Digital Signature 與其 Public Key。Mastering Bitcoin 這邊給了一個例子，又是 Alice 和 Bob，來！
Alice 在 Bob 的咖啡廳，要付錢給 Bob，她付了 0.015 BTC 給咖啡廳的 address，而這筆 Tx 的 output 就會出現一個 locking script 長這樣：
```
OP_DUP OP_HASH160 <Cafe Public Key Hash> OP_EQUAL OP_CHECKSIG
```
這裡的 `<Cafe Public Key Hash>` 放的就是咖啡廳的 address，但是格式是沒有經過 Base58Check encoding 的，大部分應該是使用 hexadecimal encoding 來記錄，而不是平常我們常看到用 1 開頭的 bitcoin address 呈現。
而上面的 locking script 如果要解鎖，也就是 Bob 想移動或花用的話就會需要下列的 unlocking script：
```
<Cafe Signature> <Cafe Public Key>
```
照著順序使 unlocking script 和 locking script 合併的話會呈現下列的樣子：
```
<Cafe Signature> <Cafe Public Key> OP_DUP OP_HASH160 <Cafe Public Key Hash> OP_EQUAL OP_CHECKSIG
```
整個 script 會使用 stack 的方式作運算，最後結果如果是 True，就能移動這筆錢，反之則無法，所以關鍵就在於 unlocking script 給的是否真的是 Cafe 的 Private Key 的 Signature，詳下步驟如下圖：
![](https://miro.medium.com/v2/resize:fit:1050/1*SgImtvpeXVZuv0K13R4b7Q.png)
可以看到每一次都會將一個物件 push 進 stack，按照順序作運算。第一個操作的運算就是使用 DUP 把前一個複製一次，所以出現兩個 Public Key。
![](https://miro.medium.com/v2/resize:fit:1050/1*wQoG4sVyTyBZ3x8qNGFTFQ.png)
再來就是將 top 的 Public Key 作 HASH160，使其變成 Public Key Hash，接著 EQUALVERIFY 會看看是否相同，若相同返回 True 繼續，接著是 CHECKSIG，檢查這個 Signature 是否跟 Public Key 是同一組，若是則剩下 True 表示成功 unlocking，可以花費這筆錢了。
這個就是更簡單的交易格式了，相較 P2PKH 省略了很多步驟，最常在 coinbase tx 裡面出現，是比較早期的交易型態，舊的挖礦軟體還會繼續用。它的 locking script 像這樣：
```
<Public Key A> OP_CHECKSIG
```
如果你 P2PKH 有看懂的話應該很好猜，unlocking script 就是這樣：
```
<Signature from Private Key A>
```
有效的 combined script 就是：
```
<Signature from Private Key A> <Public Key A> OP_CHECKSIG
```
驗證過程就跟上面幾乎一模一樣，就不再解釋了。應該會有人跟我有一樣疑問，這麼簡單為何還需要 P2PKH？我猜可能是某種安全性問題吧？不然為何要更新成都用 P2PKH，之後如果有詳細解答再附上。
這就是重頭戲了，多重簽章交易，是我覺得一個滿潮的比特幣名詞，應用也非常廣，舉凡各種限制權限的業務邏輯，或是交易之前要幾個人同意，這種條件都很常用到。
創立時決定最多 N 把 Public Key 被記錄在上面 (N 最大為 15)，而 unlocking 時至少要有 M 把 Private Key 的 signature 才能動錢，而 M 會小於等於 N。
假設五把鑰匙，需要至少三把才能啟動，就叫做 3-of-5 Multi-Signature Tx，也可叫做「5 取 3 多重簽章交易」。N 的那些 Public Key 是哪些會被記錄在 Multi-Signature 的 locking script 上面，像這樣：
```
M <Public Key 1> <Public Key 2> ... <Public Key N> N OP_CHECKMULTISIG
```
例如 2-of-3 的 multi-signature 就會是：
```
2 <Public Key A> <Public Key B> <Public Key C> 3 OP_CHECKMULTISIG
```
unlocking script 則是：
```
OP_0 <Signature B> <Signature C>
```
值得注意的是，Signature 雖然只要放指定數量即可，但是順序卻不能亂，如果 locking script 的順序是 B 放在 C 前面，你 unlocking 這邊 C 就不能放在 B 前面，否則會 invalid，礦工不會把你的 Tx 收進 block 裡。
而 OP_0 雖然有點多餘，但還是要放的，因為當初 implement CHECKMULTISIG 時的 bug，具體是什麼 bug 我還不是非常懂，日後補上。
所以拼起來就會是：
```
OP_0 <Signature B> <Signature C> 2 <Public Key A> <Public Key B> <Public Key C> 3 OP_CHECKMULTISIG
```
一樣按照 Stack Operation 來做驗證，如果最後是 True 就生效。
比特幣的分散式時間戳帳本，也就是區塊鏈，不只有支付的用途，也有很多開發者想在上面做出更多應用，例如數位憑證的服務，股票證書甚至智能合約，就是主要因為 Bitcoin 有 Data Output 這類的交易，可以將某些資訊放在 Tx 的 output 裡 (最多 20-byte)，但是這個 output 之後就不能再拿來支付了。
把自己想要放的資訊放進 bitcoin blockchain 的作法，有一群人反對，也有一群人支持，反對通常是因為這樣會讓 blockchain 無意義的膨脹，沒辦法支付的 UTXO 會一直存在 memory pool 裡面，迫使 Bitcoin full node 需要更大的 RAM 去存這些不能被花的 Tx，非常不明智。而支持這樣作法的人是因為他們可以為 Bitcoin 帶來更多種應用，讓 blockchain technology 有更 powerful 的服務場景，而不是只有支付，所以希望多做這方面的實驗。
而 Bitcoin Core Client 在 version 0.9，雙方有新的妥協，就是 OP_RETURN，它可以讓開發者放上 40 bytes 的 non-payment data 到 Tx 的 output 裡面，它不像原本的 fake UTXO，它可以被認出來，就不用放到 UTXO set 裡面。OP_RETURN 的 outputs 會被永久存在 blockchain 上，做永久保存，這樣還是會增加 blockchain 的 size，但是因為不會放進 UTXO set 裡面，還是省下不少昂貴的 RAM。
OP_RETURN 的 scripts 長這樣：
```
OP_RETURN <data>
```
data 能夠放 40 bytes，通常都是用 hash 呈現，例如說 SHA256 的結果 (32 bytes)，不少應用會放 prefix 在前面以分辨自己的服務。
要再強調一次的是 OP_RETURN 並沒有 unlocking script，OP_RETURN 是 provably un-spendable，所以通常 OP_RETURN 的 output 的 bitcoin amount 會是 zero，假設你放錢進去，就代表你在浪費錢，因為之後根本領不出來。然後 OP_RETURN 放在 input 也是耍白痴，礦工並不會收這筆交易，會在 isStandard() 這邊被判斷為 invalid，而且一個交易的 outputs 只能有一個 OP_RETURN，而且可以存在在其他四種交易裡面。
最後一個，需要最多篇幅解釋，在 2012 年冬天登場的全新 type Tx，可以簡化複雜的 Tx scripts。書中用了一家公司叫做 Mohammed 的情境當做例子解釋，大意是說這家公司幾乎所有的服務都是 base on 2-of-5 Multi-Signature Tx，因為其中除了需要客戶 (Partner) 同意之外，可能也需要 Attorney 們同意才能動錢，所以最常出現的 locking script 就會像：
```
2 <Mohammed's Public Key> <Partner1 Public Key> <Partner2 Public Key> <Partner3 Public Key> <Attorney Public Key> 5 OP_CHECKMULTISIG
```
經典的 5 取 3 多重簽章腳本，這樣就衍生了兩個問題，第一個是不能指望客戶們都對 Multi-Signature 瞭若指掌，所以他們就需要使用特製的 Wallet。第二個是這樣 Tx 的大小至少會比正常的 Tx 大五倍之多，負擔非常的重，想像一堆 Public Key 塞在 UTXO 裡面，解鎖時又用不到一半的 keys，非常浪費空間，RAM 也很貴。
所以 P2SH 就是為了解決這件事情而誕生的，複雜的 Tx scripts 可以使用簡單的 digital fingerprint(數位指紋) 來替代，就是一串 hash。想要花這筆錢時，unlocking script 必須附上一串 redeem script 是 match 這段 digital fingerprint 的，所以 P2SH 全部的意思其實是：
> pay to a script matching this hash, a script which will be presented later when this output is spent
![](https://miro.medium.com/v2/resize:fit:1050/1*c-mFcADNgB_BZmSJYM2PJg.png)
上圖我們可以想像得出來，使用 P2SH 的 locking script 明顯比沒使用的短上許多，如果還是看不出來的話，可以看看下列的真實情況：
```
2 04C16B8698A9ABF84250A7C3EA7EEDEF9897D1C8C6ADF47F06CF73370D74DCCA01CDCA79DCC5C395D7EEC6984D83F1F50C900A24DD47F569FD4193AF5DE762C587 04A2192968D8655D6A935BEAF2CA23E3FB87A3495E7AF308EDF08DAC3C1FCBFC2C75B4B0F4D0B1B70CD2423657738C0C2B1D5CE65C97D78D0E34224858008E8B49 
047E63248B75DB7379BE9CDA8CE5751D16485F431E46117B9D0C1837C9D5737812F393DA7D4420D7E1A9162F0279CFC10F1E8E8F3020DECDBC3C0DD389D9977965 
0421D65CBD7149B255382ED7F78E946580657EE6FDA162A187543A9D85BAAA93A4AB3A8F
044DADA618D087227440645ABE8A35DA8C5B73997AD343BE5C2AFD94A5 
043752580AFA1ECED3C68D446BCAB69AC0BA7DF50D56231BE0AABF1FDEEC78A6A4 
5 OP_CHECKMULTISIG
```
這就是原本的 Multi-Signature 的 locking script，如果使用 P2SH 的話就可以變成 20-byte 的 cryptographic hash，使用 SHA256 和 RIPEMD160 algorithm ，就能產出下列 script：
```
54c557e07dde5bb6cb791c7a540e0a4796f5e97e
```
是不是短多了？所以 P2SH 的 output 的 locking script 就能變成：
```
OP_HASH160 54c557e07dde5bb6cb791c7a540e0a4796f5e97e OP_EQUAL
```
Mohammed 的客戶們只需要知道比較短的 script 並且把它放進 locking script 裡面即可，不佔空間又簡單，解決了上述的兩個問題，而 Mohammed 只要自己保存 redeem script 和其中兩把 private key 即可。其實比較麻煩的地方就只是要另外存當初 digital fingerprint 的原樣而已，那串 redeem script 要自己保管好，花錢時要再拿出來使用。
## **Pay-to-Script-Hash Addresses**
甚至 P2SH 可以把 script hash 直接 encode 成 address，詳細定義在 BIP0013 (Bitcoin Improvement Proposal 0013)，把 20-byte 的 hash 直接用 Base58Check encode 成 address，而原本 P2SH 的 addresses 就是用 prefix “3” 的版本做開頭，再做一次 Base58Check 的話會導致出來的 address 會變成 prefix “3”，也就是 3 開頭的 address，即 **39RF6JqABiHdYHkfChV6USGMe6Nsr66Gzw**。所以 Mohammed 就可以直接把這串 address 當做普通的 address 丟給客戶說，你們就照平常的方式送錢到這地址即可，其餘你們都不用煩惱，這樣就是非常棒的處理方式了。
## Benefits of Pay-to-Script-Hash
書中還總結了一下 P2SH 的各項優點：
1.  複雜的 locking script 變成只有 20 bytes 的 digital fingerprint。
2.  Script Hash 可以 encode 成 address，讓傳送者可以不需要有複雜的軟體才能使用。
3.  P2SH 把創建 script 的負擔從傳送者 (客戶) 轉移到接收者(Mohammed)。
4.  P2SH 把儲存容量的負擔從 UTXO set 轉移到 Blockchain 上面，因為在 Tx 的 outputs 裡面記錄的 locking sciprt 變短了。
5.  P2SH 把資料儲存的負擔從當下移到未來，只有在再次動錢的時候會需要花比較多時間。
6.  P2SH 把手續費的負擔轉移到接收者身上。
Redeem Script and isStandard Validation
---------------------------------------
在 Bitcoin Core Client version 0.9.2 之前，因為 isStandard() 的限制，P2SH 只能用 standard tx 裡面的 P2PK、P2PKH、Multi-Sig，而 version 0.9.2 之後，只要是 valid 的 script 都可以被做成 P2SH，但是還是不能把 P2SH 放在 P2SH 裡面，不能接受 recursive 的用法，當然也不能使用 OP_RETURN，因為 OP_RETURN 無法被 redeem。
而有一個需要警告的事情是，script hash 是沒辦法知道 redeem script 長什麼樣子的，而且 redeem script 也不會放在 blockchain 上面，要自己保管好。所以如果你不小心用了一個 invalid 的 redeem script 並且做了 P2SH，很有可能你的錢就上 blockchain 了，而你的 redeem script 因為是 invalid，變成你永遠沒辦法再使用那筆錢，因為根本無法有效的 redeem，這個要特別小心。
好大的篇幅簡單講解了 Bitcoin Blockchain 常出現的交易類型，其中有許多術語如果不懂的可以 Google 一下，我之後有空會再多寫幾篇，有問題和錯誤可以直接在下面留言發問，感謝！
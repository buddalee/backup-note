# Node.js Party 2018 共筆

-- 
// TODO @ 2018.10.18 19:43

## PART 1

今日開題
> Embracing parallelism

什麼是 Embracing parallelism?

> 拼字應該可能大概是 Embarrassingly Parallel。
> https://en.wikipedia.org/wiki/Embarrassingly_parallel

例子 1: 今天兩個 GET request 進到 server 會互相影響嗎 (去掉硬體因素)？

答案是不會，在 application layer 上兩邊互不影響

例子 2: 今天兩人互相轉帳會互相影響嗎？又或者 BC 同時轉帳給 A 呢？

在 db layer 會，但在 application layer 上不影響。

> 當兩個進程進行的任務互不干擾時，AS 可以自由的處理各自的任務，但當兩個任務可能有交集時，需要透過一些手段來確保任務都是成功的，
> 
> 以 B、C 同時轉帳給 A 這個例子來說
> 假設 A 初始的 Balance 是 100，B 是 50，C 是 50，B、C 將所有的 Balance 轉移給 A，任務處理的邏輯是「計算交易雙方的結餘，並將最後的結餘寫回 DB」，那麼
>
> -----------------------------------------------------------> 時間
> 進程1
>       讀取 A 和 B 的結餘 (100, 50) -------> 計算結餘 (150, 0)  ---> 寫回 DB (A:150, B: 0, C: 0)
> 進程2
>       讀取 A 和 C 的結餘 (100, 50) ---> 計算結餘 (150, 0)  ---> 寫回 DB (A:150, B: 50, C: 0)
>                                                              
> 為了讓最後的結果是正確的，一般會在 DB Layer 對資料上鎖，確保一個進程在處理交易時，另一個進程無法進行操作。

### Concurrency 是什麼？

簡單來說，就是今天你一個 thread 可以做出這樣的操作：

```
【 [ task1 ] [   task2   ]  [     task3    ] [ task4 ] 】
```

### Worker Models

我們平常會用 workers 來處理什麼任務？

如 [PHP FPM](http://php.net/manual/en/install.fpm.php) (我 php 不熟，大概查了一下，可參考[這篇](https://segmentfault.com/q/1010000000256516)、[那篇](https://www.jianshu.com/p/c9a028c834ff)，再麻煩大神補充)
或是 Redis Master/Slave 架構(聽說現在要改叫 Farmer/Worker 比較 politically correct)

![workers](https://image.slidesharecdn.com/kaistworkshop-171128071143/95/speeding-up-distributed-machine-learning-using-codes-63-638.jpg?cb=1512701589)
PS 隨便找的圖，能表達概念就好

### PHP-FPM 的缺點

PHP-FPM 會開啟多個 Worker，並於接收到請求時轉發給 Worker 進行處理，再把處理完的結果送回。

一般 Worker 運行時會佔用到 CPU 的核心數，假如 Worker 的數量等同於 CPU 核心數， Worker 在進行 IO 時，該 CPU 就會是閒置的。

對一些追求效能的人來說，CPU 的閒置 = 浪費，所以會嘗試多開幾個 Worker，讓某個 Worker 進行 IO 時，其它 Worker 可以佔用空閒下來的 CPU 資源；但作業系統為了「公平」，會盡可能讓多個需要 CPU 資源的 Worker 輪流使用 CPU，這個從一個 Worker 切換成另一個 Worker 的過程稱為 Context Switch，並帶來一些運算資源的浪費。

### NodeJS 的特性

NodeJS 標榜自己開發時有 single thread + concurrency 的特性，實作方式就是

1. 今天來了三個需要 async 處理的 request (r1, r2, r3)
2. NodeJS 首先處理 r1，把 r1 放進 thread 中執行
3. 執行到一半 r1 需要 async 計算了 (Db operation or something else)，NodeJS 把 r1 丟到 background 做自己的 async 計算
4. 現在 thread 是空的了，於是 NodeJS 把 r2 放進 thread 中執行
5. 執行到一半 r2 需要 async 計算了， NodeJS 把 r2 丟到 background 做自己的 async 計算
6. 換 r3...
...
7. r1 在 background 執行完 async 的計算，NodeJS 把 r1 放回 thread 執行剩下的工作
8. r2 此時也執行完 async 了 (其實 r1, r2, r3 async 計算結束的順序不固定)，如果此時 thread 已經跑完 r1 的計算就會把 r2 放進去，以此類推....


這種方式的優點是開發簡單快速，但有一個致命的缺點，那就是 thread 在切換 task 時會有一個很短的時間空擋，我們稱之為 context switching ，而這累積起來是相當 expensive 的！

所以 NodeJS 的解決方式是：增加 worker。假如今天一台機器有 4 個 core (因為現在 cpu 都可以一個當兩個用，因此這裡是指 virtual core)，那 NodeJS 就會依此開 4 個 worker (4 個 thread) ，因此就大大減少了 context switching。

這樣還有什麼好處？好處是服務很 ROBUST，今天就算一個 worker 被 memory leaking 搞掛，還有其他 worker 可以補上。

> 這邊我認為 Triton 指的是，Node.js 為了避免 PHP-FPM Context Switching 的問題，可以只開啟等同於 CPU 核心數的 Worker，如此每個 Worker 都可以分配到一個完整的 virtual core，從而避免 Context Switching 帶來的消耗，而 IO 造成的 CPU 浪費，則是透過 async 的機制，在 IO 時先處理其它 request 來解決。(假如 Worker 的數量少於 CPU 核心數，應該更不會發生頻繁的 Context Switching 才是)

壞處是什麼呢？ 舉一個經典的例子：

## Postgres V.S. MySQL

Postgres 的方式是 process & shared memory
MySQL 是每一次操作都會開一個 thread

## 所以壞處是？


因為雖然很多 worker 可以執行，但每個 worker 都是獨立 memory 且有自己的 connection pool，因此一台 4 core 的機器就有 4 個 worker、 4 個 connection pool，但 database 的資源 (connection) 有限，你今天多加一個機器就多了 4 個 worker、 4 個 connection pool ，呈倍數增長，很容易搞爆 db ，因此很多頂尖企業級的 server 不會用 NodeJS。


## 那 JAVA, GoLang 呢？


而至於為什麼 Java, GoLang 不會這樣呢？舉 JAVA 為例，JAVA 一台機器就一個 process ，而底下會有很多 thread 幫他做事，要注意，**thread 是共用 memory 的!** ，因此新增一台機器只會多一個 process 不會如 NodeJS 一樣成倍數成長，因此穩定度就成了大企業的首選。

情境：
1. 今天來了 3 個 request (r1, r2, r3)
2. process 指定 r1 去 thread 1
3. process 指定 r2 去 thread 2
4. process 指定 r3 去 thread 3...


Q: 那 Facebook 不是還用 php ?
A: 人家有錢這樣搞啊



## 大魔王 while(1) 出現

但不管是哪個方法，如果有人在程式中加上 while(1) 都一定爆炸，
先看 JAVA ，當第一個會去操作 while(1) 的 request 進來後，
process 指定他去 thread 1，然後 thread 1 就爆炸了，
再來下一個同樣的 request 近來， process 指定去 thread 2 ， thread 2 也爆炸了....
不用多久全部都爆炸了。

講者之前遇過這樣的情境，一個小時內瘋狂重開機器都沒有用，因為只要一開就被 `while(1)` 搞爆掉，
最後改掉 code 才解決。

而 NodeJS 狀況也一樣，有 `while(1)` 最終都會把每個 worker 都搞爆。

結論：現在這個時代，機器比人便宜，把一個機器開大的成本比再雇用一個人便宜太多了，因此通常出問題的都是在 stupid coder 而不是機器的問題


---

## PART 2 System Throttling

今天如果遇到突發高流量事件（如下）怎辦？
```
流量
|
|           --------
|          |       |
|          |       |
|          |       |
|          |       |
|----------        ------------
|
----------------------------------- 時間
```

答：如果事前知道的話就開大機器啊，

但如果就是突發呢？

答：那就 auto-scaling 啊

但 auto-scaling 第一速度可能跟不上爆發的流量、第二也不知到流量會高到什麼程度，
因此較聰明的方法就是做 **System throttling**。

> 另一個關鍵是，相對於 APP Scale Out 的速度，DB Scale Out 慢到不可能在遇到突發高流量時即時產生實例。

實務上有兩種做法： Rate limiting 以及 Access Token

### Rate limiting

簡單直接暴力，超過的一概不理，但缺點是什麼？

範例：假如你今天興致勃勃地在 Facebook 上寫文章，
寫了一長串的文字送出去後被 server 發現超過那個 limiting rate 了，
很抱歉，不但發文失敗連帶著心血結晶的文章也一併付諸流水。

因此這顯然不是個好方法。

### Access Token

所以另一種是 Access Token ，今天只要有來的人我就發一個 access token ，
並保證在時間內一定給你正確的操作回覆，
而那些沒拿到 access token 的人就連門都進不來，
所以不期不待就不會受傷害，
也保證持有期待的人能夠美夢成真。

#### Redis

那 Access Token 存在哪裡？ 存在 redis 也有問題，
今天如果有超乎 redis 能力的人次湧入，
那 redis 可能 100 個 request 只成功執行 20 個也很糟糕，
資源也被浪費。

### Read/Write Lock v.s. Message Passing Interface

在平行化的情境底下，常常需要存取同一項資源，很可能遇到 Race Condition，因此許多的實現會透過 Lock 或 Channel 的型式來保護資源的一致性。

RW Lock 是一種很常見的保護機制，通常和 Read Lock 搭配使用，主要是透過鎖來鎖定特定資源，避免一組資源於寫入時被另一項寫入干擾，或於讀取時取得寫入到一半的結果。也就是任何一線程要讀取某個資源時，必需將該資源加上 Read Lock，要變更某項資源時，要將該資源以 RW Lock 鎖上，由於 RW Lock 會排斥 RW Lock 或 Read Lock，而 Read Lock 僅排斥 RW Lock 不排斥 Read Lock，因此同一資源可以同時被多個線程讀取，或被一個線程變更資源 (同時其它線程不能讀取資源)。

RDBMS 的操作中也使用了 Read Lock 或 RW Lock，只是現代的 DB 為了加速處理，除了使用不同層級的 Lock 外，也引入了 MVCC 等其它機制來加速數據的處理。

有別於透過鎖定同一資源來保證一致，Messaging Passing 透過一個「訊息通道」進行資源的傳遞，這個訊息通道保證資源的完整性，並且只有一個線程可以接收到資源並加以處理，從而確保多重寫入的狀況。在寫多的情境底下，這個模式可以有效避免等待 RW Lock 造成的時間浪費。

### Bloom Filter

可以參考 https://zh.wikipedia.org/wiki/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8

鑑於在爆發場景底下，用 Redis 當第一道防線仍有可能失效，因此一般會在本地記憶體中再放一份資料，減少 as 再向 redis 確認狀態的消耗。於這次的例子中，我們要確認持有 Access Token 的人是否在 N 分鐘內曾經訪問過伺服器。

因此我們在本地端儲存一個 BloomFilter 的 Instance (大小可能是 1M)，並於每次 Request 進來時，比對 BloomFilter 中是否存在這筆記錄，若「可能存在」，就通過保護層，並進行 Request 的處理。

又由於多個進程 (可能是跨主機的) 沒辦法共用 Bloom Filter 的 Instance，因此應用會以一定頻率向 Redis 更新/取得最新的狀態表。

由於這個應用只考慮 N 分鐘內的訪問記錄，N+1 分鐘以上的記錄不考慮，因此 Triton 採用的方案是，產生 N 個 Instance，每個 Instance 只記錄一分鐘的內容，並於檢測 Access Token 時，將 N 個 Instance 的記錄組合成一個加以判斷。


---

### 其他重點


* 在考量上， througput 比 latency 重要。試想今天一個服務可能原本因為某個修改，導致 latency 從 200ms -> 400ms ，User 可能有點會有點不高興。但如果影響到 throughput 使得原本同時 10000 人能上線的手遊變成同時只能 2000 人上線...哪個比較嚴重就很明顯了。

* throttling 第三招 -> 強制 APP 閃退 XD||

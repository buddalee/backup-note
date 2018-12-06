###### tags: `DStudt4TW .NET Conf 2018`

# Message Queue Based RPC - Andrew Wu

* [Sample Code](https://github.com/andrew0928/Meetup/tree/master/20180929.dotNetConf/MQRPCDEMO)
* [講者的部落格](https://www.facebook.com/andrew.blog.0928/)

------
# Agenda

> 筆記, 請從這裡開始

## Message Queue

- enqueue 產出號碼的速度
- dequeue 叫號的速度
- worker 處理job的能力
- queued job 緩衝區容量

## Architecture Design

[天瓏網路書店-建構微服務｜設計細微化的系統 (Building Microservices)](https://www.tenlong.com.tw/products/9789864760787)
推薦閱讀第四章

- Orchestration
中央集權
容易造成只有CRUD的服務

- Choreography

## Queue RPC

![](https://i.imgur.com/xDyHJVk.jpg)

透過correlation id串接非同步的message



可靠度優先，效能不是第一考量

- 允許執行順序與送出的不同




[ThreadPool 實作 #3. AutoResetEvent / ManualResetEvent — 安德魯的部落格](https://columns.chicken-house.net/2007/12/17/threadpool-%E5%AF%A6%E4%BD%9C-3-autoresetevent-manualresetevent/)
[13 common RabbitMQ mistakes - CloudAMQP](https://www.cloudamqp.com/blog/2018-01-19-part4-rabbitmq-13-common-errors.html)
[微服務架構 系列文章導讀 — 安德魯的部落格](https://columns.chicken-house.net/2018/03/25/interview01-transaction/#%E5%89%8D%E8%A8%80-%E5%BE%AE%E6%9C%8D%E5%8B%99%E6%9E%B6%E6%A7%8B-%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E5%B0%8E%E8%AE%80)

------

# Q & A
> 發問區 & 意見回饋
------


## 1. Question 
hihi 大大你好

針對 https://github.com/andrew0928/Meetup/blob/master/20180929.dotNetConf/MQRPCDEMO/MQRPC.Core/RpcServerBase.cs#L165

想請教一下， 這個的意思是不是  server run起來之後 就會block在這一行 ?

> 是的。這行有兩個目的:
> 1. 等待 StopWorkers() 這指令被呼叫
> 2. 原本的 wait() 只會 block 住 thread, 但是不會 async return, 用 task 包裝起來配合 C# async method, 可以讓 caller 提前放開去做其他事情。



## 2. 另外因為 server run起來是靠console 所以應該是只會是初始化的那一個 Thread 停在這邊?
其他的Request的部分 是透過RabbitMQ做的 應該是不會被這個block住?
那這樣是不是也可以用AutoResetEvent就好了? 

因為Set 放行的話 看起來就直接Stop server了

> 是，只有起始的 main thread 會卡在這裡。
> 這邊其實也是靠 Auto/Manual Reset Event 物件來控制的沒錯。只是這裡要等的不是 "單一" 事件。
> 照邏輯來看，要跳出來的條件好幾個，按照順序 (每一項都要滿足才能結束):
> 1. server 已經進入 stopping 狀態 (已呼叫 StopWorkers())
> 2. 處理中的 message 都已經處理完畢
> 因為這些關係，(1) 及 (2) 都有自己的 wait 物件。所以這些羅及統一包裝在 WaitUntilShutdown() 內。


## 3. 
https://github.com/andrew0928/Meetup/blob/master/20180929.dotNetConf/MQRPCDEMO/MQRPC.Core/RpcServerBase.cs#L173,L194
我有注意到 StopWorkers() 會Set event, 然後就會WaitUntilShutdown()

但是
StartWorkersAsync 在SetEvent後 也會跑WaitUntilShutdown() 不曉得這邊為何需要重複跑兩次呢? 感謝大大幫忙釋疑~

> 實際狀況下，Start / Stop worker 不見得會在同一個 thread 下執行。因此我的設計是: 無論是 Start 或是 Stop, 我都用 async 模式
> 設計成 await result 會等到整個 server 生命週期結束為止。我把這兩個 method 的 sequence diagram 圈出來，你看圖可以發現兩個是等
> 同一個時間，一起 EXIT 的。
> 
![](https://i.imgur.com/eVqUQXM.png)










--------

> 聊天室，歡迎在下方喇賽!
> 

插頭香


# [Distributed transactions microservices icebergs](http://www.grahamlea.com/2016/08/distributed-transactions-microservices-icebergs/)

## 必要條件
* `commander` 管理重複執行的行程
* `retry` 重複嘗試
* `等冪` 可以被執行兩次，但是執行兩次後造成的side effect應該等同一次

## Async Procedure
* 由client觸發
* 第一層api layer接收後，進行儲存至queue
* 第一層api layer回應給client
* 其他services收到queue message後，進行重複檢查並執行交易
* 交易結束後刪除message

## Eventual Consistency
* 由兩次transaction產生最終一致性
* client不會被知會到結果已經一致，但隨時間推移，應會取得最後理想的結果

## Notice
* retry代表不正常，需要被記錄。
* retry需要被定義次數，但是要看工作context決定
* 每個服務邏輯都為非同步，接觸微服務的interface需要有能力面對非同步狀態business logic的能力。

  Ex: 在A服務上，某task定義為完成，但是B服務可能還沒標示為完成。interface需要如何去處理？

## Related

### [Compensating Transactions](https://dzone.com/articles/transactions-in-microservices)
* Client對A，B兩方同時做交易，如果發覺A方交易失敗，即向B方要求回復
* 如果在B進行交易前，A就失敗了要如何處理？
  * Store state: 一個儲存區保存此次交易情形
  * Routing Slip: A，B通過queue通知彼此是否失敗，若對應一方收到失敗message，則進行回復
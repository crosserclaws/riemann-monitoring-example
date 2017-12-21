# Index

閱讀 `riemann.config` 檔，首先要去注意 index 的使用。 相同 service, host 的監控項，放進 index 之後， index 會總是只保留最新的一筆。也因此，我在實作 vip 監控時，其實是把 index 也做為 temporary cache 來使用。一些 vip 監控的「vip列表」，我用外部的 `virtualIP.py` 這個檔案，將「vip列表」、「vip與平台的對應關系」都送進 Riemann 。然後，Riemann 就會將這些資訊暫存在 index 裡。

## write into index
```
(let [index (default :ttl 300 (index))]
  (streams index))
```
凡是來自於 `streams` 的 event 都要放進 index 裡，如果沒有設定 time to live 的話，預設的 time to live 會設定成 300 秒。

## read from index
```
;; 定義函數
(defn event2hostgroup
  "find out all the hostgroups that contains the host of current event.

  HOSTGROUP_INFO event describes hostgroup/host as :tags/:host field
  => { :service \"HOSTGROUP_INFO\",
       :host \"owl-docker\"
       :tags [\"hostgroup1\", \"hostgroup2\"]}"
  [e]
  (:tags (riemann.index/lookup (:index @core) (:host e) "HOSTGROUP_INFO")))

;; 使用函數於主程式 
(when-let [groupv (f/event2hostgroup e)]
...
)
```

`event2hostgroup` 這個函數，它會透過 `riemann.index/lookup` 去讀取 index 裡的資料，讀取資料時，傳入的參數是 host 和 service 。在這個函數裡，service 是指定為 "HOSTGROUP_INFO" 。也就是說，外部的 `virtualIP.py` 寫入的 "HOSTGROUP_INFO" 事件，會被這個函數所讀取。

在 Riemann 裡，每一個事件 (event) ，都是使用 clojure 的 map 來表現。在註解裡寫了，這種事件，它是用來描述「主機」與「主機群組」的對應關系。其中「主機」在 vip 監控的例子，就是 vip ，「主機群組」就是 vip 對應的平台。「主機」會存放在 event 的 host 欄位。「主機群組」會存放在 event 的 tags 欄位。

```
(:tags ...)
```
這一段 code snippet 的作用是：將事件的 tags 欄位的值取出。事件 (event) 的 tags 欄位的值在此處會是一個 vector of strings。

特別注意：這段 code snippet 的寫法是應用了 clojure 的 keyword as function 寫法。

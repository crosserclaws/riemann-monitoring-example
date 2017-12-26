# virtual ip 探測 event 的 trigger

```
(subscribe "hostgroup"
  (where (not (state "expired"))
     (where (service "service.lvs.httping.vip")
       (smap f/add-contact 
         (by [:host :service]
           (f/all 3 pos?
             (changed :all {:init false}
               prn)))))))
```

範例的程式碼每一行依序為：
1. 訂閱 hostgroup 這個 channel 裡的事件
2. 如果該事件的狀態 (state) 不是 "expired" 就往下一層送
3. 如果該事件的服務 (service) 恰好是　"service.lvs.httping.vip" 就往下一層送
4. 對於每一個事件，呼叫 `f/add-contact` 函數，加上連絡資訊
5. 對事件做分流，分流的條件為： 相同主機 (host)、相同服務 (service) 的，才放在同一個串流 (stream)
6. 對每個串流裡的事件，如果連續三個都是正數，就往下一層送。
7. 偵測得到的事件，如果該事件的 all 欄位有發生變化，就往下一層送。(初始的 all 欄位值視為 false )
8. 對事件做印出 (prn) 的動作。
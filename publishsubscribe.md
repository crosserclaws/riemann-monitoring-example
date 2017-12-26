# publish/subscribe

publish/subscribe 是 Riemann 裡，用來做出類似事件的通道 (channel) 的語法。

```
(streams
  (smap (fn [e] e)
        (publish "hostgroup")))
        
(subscribe "hostgroup"
  (where (not (state "expired"))
         prn))
```

上述的範例：

## publish (送進 channel )
對於系統流入的每個事件 (event) ，先對它做一個 smap 操作，不過剛好在此處的操作是 identity 操作，即不對事件做任何修改。操作完之後，就將事件送進 "hostgroup" 這個 channel 。

## subscribe (從 channel 提取事件)
對於 "hostgroup" 這個 channel 流入的每個事件 (event)，檢查它的狀態，如果不是 `expired` 的話，就將它印出。
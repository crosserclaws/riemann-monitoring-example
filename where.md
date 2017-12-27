# where/where*

Riemann 的 where 語意，與事件 (event) 的模型綁在一起。

## 事件
事件 \(event\) 在 Riemann 中是用 clojure 語言的 immutable map 來表現。

| 基本欄位 | 意義 |
| --- | --- |
| host | 主機名稱。 e.g. "api1", "foo.com" |
| service | 服務名稱。e.g. "API port 8000 reqs/sec" |
| state | 任何少於 255bytes 的字串。e.g. "ok", "warning", "critical" |
| time | 時間。使用 unix epoch seconds |
| description | 自由形式的字串 |
| tags | 字串的 list 。例如：\["rate", "fooproduct", "transient"\] |
| metric | 事件的數值。e.g. the number of reqs/sec |
| ttl | 單位為秒。這個事件「有效」的期間。逾時的話，事件可能會被從索引移除。 |

### 與 Open-Falcon 的對應關系
1. Riemann event 的 `host` 對應 Open-Falcon 的 `endpoint`
2. Riemann event 的 `service` 對應 Open-Falcon 的 `metric` 或是 `metric` + `tags`
3. Riemann event 的 `tags` 也是對應 Open-Falcon 的 `tags`
4. Riemann event 的 `ttl`  是處理 Open-Falcon 的 nodata 模組處理的問題。

## where 範例
```
(where (not (state "expired"))
       prn)
```

如果 `where` 得到的事件裡的 `state` 欄位，它的值不是 `expired` 就印出。

# where* 
where* 相對於 where 是跟事件，比較沒有綁得這麼緊的版本。如果需要做比較通用的「事件過濾器」，就要考慮使用 `where*`

解說 where* 的範例前，首先要介紹 clojure 的 lambda 語法。

## clojure lambda
```
#(:all %)
```
跟下例是等價的
```
(fn [e]
  (:all e))
```
意思就是，如果輸入的引數是一個 map ，就將 all 欄位的值取出。

## where* 範例
```
(where* #(:all %)
        #(owl/send->owl % vip-rule)
        (else prn))
```
`where*` 的條件判斷是看「是否 event 進入條件判斷式 #(:all %) 之後可以得到 true 值」

如果得到 true ，執行 `#(owl/send->owl % vip-rule)`
如果得到 false ，執行 `prn`
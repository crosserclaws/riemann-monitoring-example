# where

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

## where 範例
```
(where (not (state "expired"))
       prn)
```

如果 `where` 得到的事件裡的 `state` 欄位，它的值不是 `expired` 就印出。
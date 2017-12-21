#smap

`smap` 是 Riemann 裡最常用的事件轉換器。如果要做一些簡單的 Riemann 應用。只用 `streams` 和 `smap` 大概就可以做出來了。

```
(smap f & children)
Streaming map. Calls children with (f event), whenever (f event) is non-nil. Prefer this to (adjust f) and (combine f). Example:

(smap :metric prn) ; prints the metric of each event. (smap #(assoc % :state “ok”) index) ; Indexes each event with state “ok”
```
## example 1
```
(streams
  (smap (fn [e]
          e)
        (publish "hostgroup")))
```
上述的範例：
1. `streams` 收到事件之後，交給 `smap` 處理。
2. `smap` 將這個事件用「函數 `f`」 加以處理，處理完之後如果非 nil 就交給「子串流 `children`」。此處的函數為 identity function ，不做任何事，傳回原本的事件
3. 此處的子串流為 `(publish "hostgroup")` 。意即，將事件送到名稱為 "hostgroup" 的「通道 `channel`」

## example 2
```
(streams
  ; tags event with a group name and publish it to the group channel.
  (smap (fn [e]
          (assoc e :hostgroup "foo")
        (publish "hostgroup")))
```
上述的範例：
1. `streams` 收到事件之後，交給 `smap` 處理。
2. `smap` 將這個事件用「函數 `f`」 加以處理，處理完之後如果非 nil 就交給「子串流 `children`」。此處的函數會將事件做增加 :hostgroup 欄位的動作，且增加的 :hostgroup 欄位為 "foo"
3. 此處的子串流為 `(publish "hostgroup")` 。意即，將事件送到名稱為 "hostgroup" 的「通道 `channel`」

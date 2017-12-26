# clojure 語法 when, when-let, assoc

## 下列的函數是 smap 裡定義的函數
該函數會接受一個事件 (event) 做為輸入的引數 (input argument)。先用該事件去查詢，這個事件對應的「主機群組」(hostgroup) ，視查詢出來的主機群組，做對應的修改。

```
(fn [e]
  (when-let [groupv (f/event2hostgroup e)]
  ; groupv can be nil, [], [hostgroup_OWL hostgroup_CC]
    (when (and (not= "HOSTGROUP_INFO" (:service e))
               (not= "CONTACT_INFO" (:service e)) 
               (not= "ACTIVE_HOSTGROUP_INFO" (:service e)) 
               (seq groupv))
              ; not empty, pass on event to a group channel.
          (assoc e :hostgroup groupv))))
```

### when-let
```
(when-let [x (f ....)]
  body
)
```
`when-let` 會讓 x 得到 `(f ...)` 求值後的傳回值。如果 x 為 true ，才做 body

### when
```
(when (and X Y Z)
  body
)
```
`when` 會先求值 `(and X Y Z)` 。如果得到的值為 true ，才做 body。

其中 `(and X Y Z)` ，必須是 X, Y, Z 每個都傳回 true ，才會傳回 true 。

### (seq ...)
```
(seq groupv)
```
這個其實是一個 clojure idiom 。當 groupv 是一個 clojure 的 collection 時， `(seq groupv)` 和 `(not (empty? groupv))` 是等價的。

### assoc 
```
(assoc e :hostgroup groupv)
```
如果 e 的值是 `{:service "AA"}`, groupv 的值是 `["Q" "P"]`
上述的傳回值就是 `{:service "AA" :hostgroup ["Q" "P"]}`
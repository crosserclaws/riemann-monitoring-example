#Streams

在 vip 監控裡，只有使用兩次 `streams` 。
```
(streams
  ; tags event with a group name and publish it to the group channel.
  (smap ...))

;; Enable the function that when user disable vip,
;;  the corresponding vip alert will be recovered.
(streams
  (expired
    ...
    ))
```

`streams` 就是系統的輸入。系統每次得到新的事件 (event) ， streams 函數就可以得到 event 。典型的作法， event 就是透過 streams 開始做層層的串接、層層的處理。

系統得到的事件通常有兩類：
1. 外部事件： 比方說，外部的 python script 又送了事件進來。
2. 內部事件： 比方說，某個已經送進來的事件的 time to live 到期了。就會自動產生「過期」的事件。
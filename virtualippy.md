#VirtualIP.py 簡述

virtualIP.py 可以接受三種不同的 input argument。每一個 argument 會恰好對應一個 cronjob 。這三個 cronjob 做的事情，都是去打 BossAPI ，並且將 BossAPI 裡，和 virtual ip 相關的資訊、負責人、平台相關的資訊，都送進 Riemann 的 index 裡做**緩存**。

如果之後做重構，要放棄 Riemann 做為 online judge 的引擎的話，那就會需要一個類似 Redis 的儲存機制，用來取代目前用 Riemann 的 index 達成的緩存功能。

## 簡述原始碼 vip2platform
```
    if sys.argv[1] == "vip2platform":
        # get data from Boss API
        res = json.loads(platform_get_ip(boss_account, boss_key))
        # use group-by to re-arrange
        out = get_vip_platform(res)
        # send to riemann
        send2riemann(hostgroup_fmt(out), "HOSTGROUP_INFO", 3600)
```
內容就是三個 statement ，分別是做 ETL (extract, transform, load)
1. 將資料從 BossAPI 取出，這個是 Extract
`res = json.loads(platform_get_ip(boss_account, boss_key))`
2. 將 BossAPI 取出的資料做重新整理，這個是 Transform
`out = get_vip_platform(res)`
3. 將整理完的資料，送進 Riemann 。( 以便緩存在 Riemann 的 index ) 這個是 Load
`send2riemann(hostgroup_fmt(out), "HOSTGROUP_INFO", 3600)`

## 簡述原始碼 vip2contact
```
    if sys.argv[1] == "vip2contact":
        res = json.loads(platform_all(boss_account, boss_key))
        out = get_platform_contact(res, boss_account, boss_key)
        send2riemann(contact_fmt(out), "CONTACT_INFO", 86400)
```
前兩個函數分別都打不同的 BossAPI 。第一個函數打的 BossAPI 可以取回所有的平台名稱。第二個函數會對每一個平台打取得對應的負責人名稱的 BossAPI 。(所以第二個函數會一口氣打多次的 API )

## 送進 Riemann 的資料結構
本質上，送進 Riemann 的資料，就是 key/value pair 的對應關系。但是，這種對應關系，因為要送進 Riemann ，就把它變成 Riemann 的事件。

| 事件欄位 | vip2platform | vip-active | vip2contact |
| --- | --- | --- | --- |
| host | vip | vip | 平台名稱|
| service | "HOSTGROUP_INFO" | "ACTIVE_HOSTGROUP_INFO" | "CONTACT_INFO" |
| tags | [平台1, 平台2, ...] | [平台1, 平台2, ...] | [負責人1, 負責人2, ...] |
| ttl | 3600 | 600 | 86400 |

之後會緩存在 Riemann 的 index 裡的資料，key 是二維度的 key ，分別由 `host`, `service` 兩個欄位來表現。而 value 則是用 `tags` 欄位來表現。 

此處會特別選用 `host` 和 `service` 來做存放 key 是因為 Riemann 的內建函數 `riemann.index/lookup` 就是使用 `host`, `service` 來做查表。



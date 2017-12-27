#python script 與 Riemann 的溝通

1. VirtuaIP.py 送「控制用事件 (control event)」 進入 Rieamnn 並且做緩存。
```
VirtualIP    ---> service = "HOSTGROUP_INFO"            --->    Riemann (資料進入Riemann之後，放進 index 做緩存。)
                             或 "CONTACT_INFO" 
                             或 "ACTIVE_HOSTGROUP_INFO"
```                      

2. probe.py 從 Riemann 取得 virtual ip 清單，並且做 httping 的動作
```
probe.py   <---   從 Riemann 的 ACTIVE_HOSTGROUP_INFO 提取的 active vip 資料  <----       Riemann 
```

3. probe.py 送「監控用事件 (vip monitor event)」 進入 Riemann 。讓 Riemann 來做告警的判斷。
```
probe.py    --->  service = "service.lvs.httping.vip"   --->    Riemann (做告警的判斷)             
```
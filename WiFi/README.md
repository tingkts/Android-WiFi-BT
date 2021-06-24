




[Background knowledge](./elementary%20knowledge.md)

</br>



Call hierarchy :

![](./assets/drawio/call%20hierachy.png)



</br>




iw command : &emsp; [⥂](./iw%20--help.txt)
```shell
iw dev wlan0 scan

iw dev wlan0 interface add ap0 type __ap
iw dev wlan0 interface add wlan1 type __ap


	[  841.305796] R0: [iw][09:54:31.322238]  wlan: [1846:E :VOS] vos_get_context: vos context pointer is null
	[  841.315243] R0: [iw][09:54:31.331688]  wlan: [1846:E :HDD] wlan_hdd_get_dfs_mode: 21101: ACS dfs mode is NONE
	[  841.325329] R0: [VosMCThread][09:54:31.341774]  wlan: [251:E :WDA] wma_unified_vdev_create_send: ID = 2 VAP Addr = 0a:3a:88:a6:c6:da
	[  841.337617] R0: [VosMCThread][09:54:31.354058]  wlan: [251:E :WDA] invalid rate code, ignore.
	[  841.348145] R0: [iw][09:54:31.364587]  wlan: [1846:E :HDD] wlan_hdd_get_classAstats: Unable to retrieve Class A statistics
	[  841.359903] R0: [VosMCThread][09:54:31.376346]  wlan: [251:E :WDA] Invalid wda_cli_set pdev command/Not yet implemented 0x34
	[  841.359961] R0: [iw][09:54:31.376407]  wlan: [1846:E :HDD] wlan_hdd_get_classAstats: Unable to retrieve Class A statistics


```

wifi interface of STA, P2P, AP
```shell
[wifi.concurrent.interface]: [wlan1]
[wifi.direct.interface]: [p2p0]
[wifi.interface]: [wlan0]


wlan0     Link encap:Ethernet  HWaddr 08:3a:88:22:c6:da  Driver ar6k_wlan       // STA
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3000
          RX bytes:0 TX bytes:0

p2p0      Link encap:Ethernet  HWaddr 0a:3a:88:a5:c6:da  Driver ar6k_wlan       // P2P
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3000
          RX bytes:0 TX bytes:0

wlan1     Link encap:Ethernet  HWaddr 0a:3a:88:a6:c6:da  Driver ar6k_wlan       // AP
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3000
          RX bytes:0 TX bytes:0
```

wifi service :
```shell
pixel:/ # ps -A | grep -i wifi
wifi           304     1   45052   7964 binder_thread_read  0 S android.hardware.wifi@1.0-service
wifi          9618     1   38352   5336 do_epoll_wait       0 S wificond


pixel:/ # getprop | grep -i wifi
    [init.svc.vendor.wifi_hal_legacy]: [running]
    [init.svc.wificond]: [running]
    [log.tag.WifiHAL]: [D]
    [ro.boot.wificountrycode]: [CN]
    [ro.boottime.vendor.wifi_hal_legacy]: [10178485750]
    [ro.boottime.wificond]: [11040210750]
    [ro.wifi.channels]: []
    [sys.wifitracing.started]: [1]
```






</br>
</br>

AP
* * *

Create AP interface (ifcae) :
```
// frameworks/opt/net/wifi/service/java/com/android/server/wifi/
H A D	HostapdHal.java	358 status = iHostapdV1_1.addAccessPoint_1_1(ifaceParams1_1, nwParams);

	if (hostapd_get_iface(interfaces_, iface_params.V1_0.ifaceName.c_str())) {

// chip mode

  struct ChipMode {
    /**
     * Id that can be used to put the chip in this mode.
     */
    ChipModeId id;

    /**
     * A list of the possible interface combinations that the chip can have
     * while in this mode.
     */
    vec<ChipIfaceCombination> availableCombinations;
  };


  struct ChipIfaceCombination {
    vec<ChipIfaceCombinationLimit> limits;
  };

  struct ChipIfaceCombinationLimit {
    vec<IfaceType> types; // Each IfaceType must occur at most once.
    uint32_t maxIfaces;
  };

    // e.g.

   0 = {IWifiChip$ChipMode@13676} "{.id = 0, .availableCombinations = [{.limits = [{.types = [0], .maxIfaces = 1}, {.types = [2], .maxIfaces = 1}]}]}"
   1 = {IWifiChip$ChipMode@13677} "{.id = 1, .availableCombinations = [{.limits = [{.types = [1], .maxIfaces = 1}]}]}"
  availableModes = {ArrayList@13670}  size = 2
  chip = {IWifiChip$Proxy@13671} "android.hardware.wifi@1.3::IWifiChip@Proxy"
  chipId = 0

  {chipInfo={chipId=0, availableModes=[{.id = 0, .availableCombinations = [{.limits = [{.types = [0], .maxIfaces = 1}, {.types = [2], .maxIfaces = 1}]}]}, {.id = 1, .availableCombinations = [{.limits = [{.types = [1], .maxIfaces = 1}]}]}], currentModeIdValid=false, currentModeId=0, ifaces[1].length=0, ifaces[0].length=0, ifaces[2].length=0, ifaces[3].length=0), chipModeId=1, interfacesToBeRemovedFirst=[])


    IFaceType			ChipMode id 0				ChipMode id 1

    STA	  0				1					0

    AP	  1				0					1

    P2P	  2				1					0

    NAN	  3				0					0

                               	     STA/P2P				     AP only



// hostapd config file :

    05-18 03:48:20.294  2590  2590 I hostapd : lyt: wlan0 config=/data/vendor/wifi/hostapd/hostapd_wlan0.conf
```


> The IP assigned by Android hotspot is in the range of 192.168.43.*.

</br>
</br>

******

</br>

### Wi-Fi Country Code

- [Qualcomm WiFi country code / CSDN kv110](https://blog.csdn.net/kv110/article/details/104419721)

- [高通Android7.1 WIFI国家码问题 / CSDN VaderZhang](https://blog.csdn.net/zhangdaxia2/article/details/90545175)

- [[RK3399/RK3328/RK3288][Android7.1.1] WIFI:国家码导致部分信道不支持的问题(无法扫描到wifi) / CSDN 风之空响](https://blog.csdn.net/weixin_39966398/article/details/103310505)

- [命令設定wifi國家碼](https://www.itread01.com/content/1541806476.html)


</br>

### Wi-Fi Direct &ensp;[ 🔗 ](https://www.youtube.com/results?search_query=android+wifi+direct)

- [Wi-Fi P2P API / developer.android.com](https://developer.android.com/guide/topics/connectivity/wifip2p)

  － [WiFi Direct详解（p2p使能，扫描，连接流程）基于Android8.1.0 / CSDN 二十岁了还没有去过星巴克](https://blog.csdn.net/qq_43804080/article/details/92133306?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162433174516780265464072%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162433174516780265464072&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-92133306.first_rank_v2_pc_rank_v29&utm_term=android+wifi+direct&spm=1018.2226.3001.4187)

  － [Android平台Wifi_Direct使用 / CSDN a220315410](https://blog.csdn.net/a220315410/article/details/9114653?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162433174516780265464072%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162433174516780265464072&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-6-9114653.first_rank_v2_pc_rank_v29&utm_term=android+wifi+direct&spm=1018.2226.3001.4187)

  － [android中wifidirect的操作学习 / CSDN mockingbirds](https://blog.csdn.net/mockingbirds/article/details/40794865?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162433174516780265464072%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162433174516780265464072&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-7-40794865.first_rank_v2_pc_rank_v29&utm_term=android+wifi+direct&spm=1018.2226.3001.4187)


&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
&ensp;&ensp;&ensp;&ensp;[↪](https://so.csdn.net/so/search?q=android%20wifi%20direct&t=&u=)


- [如何测试wifi direct的传输速度 / CSDN myxmu](https://blog.csdn.net/myxmu/article/details/12249751)





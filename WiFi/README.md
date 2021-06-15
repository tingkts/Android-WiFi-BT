




[Background knowledge](./elementary%20knowledge.md)


iw command :
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

沒錯,Android hotspot所assign IP range是192.168.43.X,這是正確的

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

</br>
</br>






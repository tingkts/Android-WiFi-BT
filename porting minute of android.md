### Source tree

&ensp; [Platform: imx8mp andorid10  &ensp; ⥂](https://github.com/tingkts/Android-WiFi-BT/blob/main/source.md)


</br>
</br>

### Vendor resources

&ensp; Vendor need to provide the following resources to support porting :


&ensp;&ensp; - Porting guide

&ensp;&ensp; - Sources or Modules (Binary release) of __***Driver***__, __***Firmware***__, __***HIDL/HAL***__, and additional optional __***configuration files***__.






</br>
</br>

### Issues

__**Wi-Fi**__

```
▸ randomization MAC

  => need to diable randomization MAC of all modes, STA, AP, P2P
```


```
AP mode

▸ iface of AP

  => AP's iface name is wlan1
  
     ` iw dev wlan0 interface add wlan1 type __ap `

▸ AP channel

  => 2.4G/5G has its own channel set of country code by config file, and need to diable config_wifi_softap_acs_supported if driver doesn't support ACS (Auto Channel Selection).

```

__**BT**__

☞&ensp;libbt-vendor.so&ensp;⚭&ensp;hardware\qcom\bt\msm8992\libbt-vendor

```
▸ load firmware fail

  => check with vendor, vendor update FW files or give the patch of libbt-vendor to correct FW loading issue.
```


```
▸ power on fail in rfkill
▸ opcode 0xc03 response timeout

    opcode 0xc03 :
        #define HCI_GRP_HOST_CONT_BASEBAND_CMDS (0x03 << 10) /* 0x0C00 */
        /* Commands of HCI_GRP_HOST_CONT_BASEBAND_CMDS */
        #define HCI_RESET (0x0003 | HCI_GRP_HOST_CONT_BASEBAND_CMDS)


  => add rkfill node in dts

     bt_rfkill {
         compatible = "fsl,mxc_bt_rfkill";
         bt-power-gpios = <&gpio3 14 GPIO_ACTIVE_HIGH>;
         reset-delay-us = <2000>;
         reset-post-delay-ms = <40>;
         status ="okay";
     };
```




☞&ensp;libbuletooth.so&ensp;⚭&ensp;/system/bt

    ▸ opcode HCI_BLE_READ_MAXIMUM_DATA_LENGTH (0x202F), responds "illegal command"

      => check with vewndor, vendor confirm opcode of response "illegal command" means Bluetttoh Controller (impl. by vendor BT firmware) doesn't support this opcode, so it's able to skip this opcode in BT stack of /system/bt.


☞&ensp;android.hardware.bluetooth@1.0-service&ensp;⚭&ensp;hardware\interfaces\bluetooth\1.0\default

    ▸ HCI unknown packet type 254

      => check with vendor, vendor update NVM firmware file to diable IBS (In-band Sleep).

        Here’s the definition of the values used for IBS:

            #define HCI_IBS_SLEEP_IND 0xFE
            #define HCI_IBS_WAKE_IND 0xFD
            #define HCI_IBS_WAKE_ACK 0xFC





</br>
</br>
</br>

<p align="right">Based on porting QCA9377 in imx8mp android 10</p>

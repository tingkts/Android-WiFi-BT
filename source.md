
Kernel

    vendor/nxp-opensource/kernel_imx/drivers/net/wireless/qcacld-2.0_qca9377   // QCA9377 Wi-Fi/BT ddriver

Firmware

    vendor/nxp/imx-firmware    // optional in the bsp path, but must put in device path of vendor/firmware in nxp i.MX* platform

HIDL

    // Wi-Fi
    hardware/interfaces/wifi
    hardware/interfaces/wifi/hostapd
    hardware/interfaces/wifi/supplicant

    // BT
    hardware/interfaces/bluetooth
    hardware/interfaces/bluetooth/a2dp
    hardware/interfaces/bluetooth/audio

legacy HAL

    // Wi-Fi
    hardware/qcom/wlan

    // BT
    hardware/qcom/bt/msm8992/libbt-vendor


Service

    // Wi-Fi
    external/wpa_supplicant_8/wpa_supplicant
    external/wpa_supplicant_8/hostapd

    // BT
    packages/apps/Bluetooth


Framework

    // Wi-Fi
    frameworks/opt/net/wifi
    frameworks/base/wifi/

    // BT
    frameworks/base/core/java/android/bluetooth
    frameworks/base/services/core/java/com/android/server/BluetoothManagerService.java
    frameworks/base/services/core/java/com/android/server/BluetoothService.java

UI

    packages/apps/Settings
    frameworks/base/packages/SettingsLib


</br>
</br>
</br>

<p align="right">Based on porting QCA9377 in imx8mp andorid10</p>

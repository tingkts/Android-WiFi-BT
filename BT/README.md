## BT module hierarchy


&emsp;

&emsp;<img src="./diagram/BT%20module%20hierachy.png" width="60%" height="60%" alt="BT module hierachy"/>




<br/>
<br/>
<br/>

## HCI Packet

### opcode :

&emsp;<img src="./diagram/hci%20packet%20-%20opcode.png" width="30%" height="30%" alt="hci packet - opcode"/>

&emsp;[system/bt/stack/include/hcidefs.h](https://android.googlesource.com/platform/system/bt/+/refs/tags/android-10.0.0_r41/stack/include/hcidefs.h)

&emsp;

### vendor opcode :

~~~c
typedef enum {
    BT_VND_OP_POWER_CTRL,
    BT_VND_OP_FW_CFG,
    BT_VND_OP_SCO_CFG,
    BT_VND_OP_USERIAL_OPEN,
    BT_VND_OP_USERIAL_CLOSE,
    BT_VND_OP_GET_LPM_IDLE_TIMEOUT,
    BT_VND_OP_LPM_SET_MODE,
    BT_VND_OP_LPM_WAKE_SET_STATE,
    BT_VND_OP_SET_AUDIO_STATE,
    BT_VND_OP_EPILOG,
    BT_VND_OP_A2DP_OFFLOAD_START,
    BT_VND_OP_A2DP_OFFLOAD_STOP,
} bt_vendor_opcode_t;
~~~

&emsp;
### hci packet type :
~~~c
enum HciPacketType {
  HCI_PACKET_TYPE_UNKNOWN = 0,
  HCI_PACKET_TYPE_COMMAND = 1,
  HCI_PACKET_TYPE_ACL_DATA = 2,
  HCI_PACKET_TYPE_SCO_DATA = 3,
  HCI_PACKET_TYPE_EVENT = 4
};
~~~

&emsp;
### hci packet format :


&emsp;<img src="./diagram/hci%20packet%20format.png" width="70%" height="70%" alt="hci package format"/>




<br/>
<br/>
<br/>

## HCI Module


### ◤ system/bt



- HCI command / response event :

    [system/bt/device/src/controller.cc](https://android.googlesource.com/platform/system/bt/+/refs/tags/android-10.0.0_r41/device/src/controller.cc)
    ```java
        response = AWAIT_COMMAND(
            packet_factory->make_ble_read_suggested_default_data_length());
        packet_parser->parse_ble_read_suggested_default_data_length_response(
            response, &ble_suggested_default_data_length);
        }
    ```

    - Make command packet :

        [system/bt/hci/src/hci_packet_factory.cc](https://android.googlesource.com/platform/system/bt/+/refs/tags/android-10.0.0_r41/hci/src/hci_packet_factory.cc)
        ```java
        static BT_HDR* make_ble_read_maximum_data_length(void) {
            return make_command_no_params(HCI_BLE_READ_MAXIMUM_DATA_LENGTH);
        }

        static BT_HDR* make_command_no_params(uint16_t opcode) {
            return make_command(opcode, 0, NULL);
        }

        static BT_HDR* make_command(uint16_t opcode, size_t parameter_size,
                                    uint8_t** stream_out) {
            BT_HDR* packet = make_packet(HCI_COMMAND_PREAMBLE_SIZE + parameter_size);

            uint8_t* stream = packet->data;
            UINT16_TO_STREAM(stream, opcode);
            UINT8_TO_STREAM(stream, parameter_size);

            if (stream_out != NULL) *stream_out = stream;

            return packet;
        }

        static BT_HDR* make_packet(size_t data_size) {
            BT_HDR* ret = (BT_HDR*)buffer_allocator->alloc(sizeof(BT_HDR) + data_size);
            CHECK(ret);
            ret->event = 0;
            ret->offset = 0;
            ret->layer_specific = 0;
            ret->len = data_size;
            return ret;
        }
        ```

    - Parser event packet :

        [system/bt/hci/src/hci_packet_parser.cc](https://android.googlesource.com/platform/system/bt/+/refs/tags/android-10.0.0_r41/hci/src/hci_packet_parser.cc)

        ```java
        static void parse_ble_read_maximum_data_length_response(
                BT_HDR* response, uint16_t* ble_supported_max_tx_octets,
                uint16_t* ble_supported_max_tx_time, uint16_t* ble_supported_max_rx_octets,
                uint16_t* ble_supported_max_rx_time) {
            uint8_t* stream = read_command_complete_header(
                response, HCI_BLE_READ_MAXIMUM_DATA_LENGTH, 8 /* bytes after */);
            STREAM_TO_UINT16(*ble_supported_max_tx_octets, stream);
            STREAM_TO_UINT16(*ble_supported_max_tx_time, stream);
            STREAM_TO_UINT16(*ble_supported_max_rx_octets, stream);
            STREAM_TO_UINT16(*ble_supported_max_rx_time, stream);
            buffer_allocator->free(response);
        }

        static uint8_t* read_command_complete_header(BT_HDR* response,
                                                    command_opcode_t expected_opcode,
                                                    size_t minimum_bytes_after) {
            uint8_t* stream = response->data + response->offset;

            // Read the event header
            uint8_t event_code;
            uint8_t parameter_length;
            STREAM_TO_UINT8(event_code, stream);
            STREAM_TO_UINT8(parameter_length, stream);

            const size_t parameter_bytes_we_read_here = 4;

            // Check the event header values against what we expect
            CHECK(event_code == HCI_COMMAND_COMPLETE_EVT);
            CHECK(parameter_length >=
                    (parameter_bytes_we_read_here + minimum_bytes_after));

            // Read the command complete header
            command_opcode_t opcode;
            uint8_t status;
            STREAM_SKIP_UINT8(stream);  // skip the number of hci command packets field
            STREAM_TO_UINT16(opcode, stream);

            // Check the command complete header values against what we expect
            if (expected_opcode != NO_OPCODE_CHECKING) {
                CHECK(opcode == expected_opcode);
            }

            // Assume the next field is the status field
            STREAM_TO_UINT8(status, stream);
            if (status != HCI_SUCCESS) {
                LOG_ERROR(LOG_TAG, "%s: return status - 0x%x", __func__, status);
                return NULL;
            }
            return stream;
        }
        ```


<br/>






### ◤ hardware/interface/bt

- android.hardware.bluetooth@1.0

&emsp;&emsp; <img src="./diagram/android.hardware.bluetooth%401.0-service.png" width="70%" height="70%" alt="android.hardware.bluetooth@1.0"/>


- hci event callback

&emsp;&emsp;<img src="./diagram/hci%20event%20callback.png" width="70%" height="70%" alt="hci event callback"/>




<br/>
<br/>
<br/>

## Miscellaneous

### Bluetooth abbreviation
- [Bluetooth terminology](https://www.google.com/search?q=BLuetooth%E8%A1%93%E8%AA%9E&oq=BLuetooth%E8%A1%93%E8%AA%9E&aqs=chrome..69i57j0i333.12274j0j7&sourceid=chrome&ie=UTF-8)
- [What's Bluetooth LE (BLE)](https://www.google.com/search?q=Bluetooth+LE+%E6%98%AF%E4%BB%80%E9%BA%BC&oq=Bluetooth+LE+%E6%98%AF%E4%BB%80%E9%BA%BC&aqs=chrome..69i57j33i160.30570j0j7&sourceid=chrome&ie=UTF-8)

### Bluetooth HCI
- [蓝牙HCI command/event/acl/sco格式介绍_朝气蓬勃-CSDN博客](https://blog.csdn.net/XiaoXiaoPengBo/article/details/107638914)
- [如何分析HCI的Command Packet和Event Packet包_daydayupfromnowon的专栏-CSDN博客](https://blog.csdn.net/daydayupfromnowon/article/details/6324227)


### Bluetooth Protocol/Profile
- [Bluetooth GATT](https://www.google.com/search?q=bluetooth+GATT&rlz=1C1GCEU_zh-TWTW892TW892&oq=bluetooth+GATT&aqs=chrome..69i57j0l9.4520j0j7&sourceid=chrome&ie=UTF-8)
- [What's the HSP, HFP, A2DP, AVRCP](https://blog.witsper.com/tips/bluetooth-profile/)


### Bluetooth subsystem
- [bluetooth btif](https://www.google.com/search?q=bluetooth+btif&oq=bluetooth+btif&aqs=chrome..69i57j0i5i30j0i8i10i30j0i8i30l2.6237j0j7&client=ms-android-oppo-rev1&sourceid=chrome-mobile&ie=UTF-8)
- [android5.1 藍芽子系統介紹（一）Android下bluedroid、bluetooth apk介紹 - IT閱讀](https://www.itread01.com/content/1548699142.html)


### Bluetooth scan &nbsp;  [⥂](https://www.google.com/search?q=android+%E8%97%8D%E7%89%99+scan%E6%B5%81%E7%A8%8B&rlz=1C1GCEU_zh-TWTW892TW892&oq=android+%E8%97%8D%E7%89%99+scan%E6%B5%81%E7%A8%8B+&aqs=chrome..69i57j0i333l2.27403j0j7&sourceid=chrome&ie=UTF-8)
- [經典藍牙inquiry與inquiry scan - 台部落](https://www.twblogs.net/a/5c25c2ebbd9eee16b3db7d97)
- [Android 9.0 蓝牙扫描流程_一个Android菜鸟的博客-CSDN博客](https://blog.csdn.net/qq_43804080/article/details/105711347)
- [GATT scan的流程 - 雪山飞燕 - 博客园](https://www.cnblogs.com/libs-liu/p/9166075.html)


### Bluetooth PAN &nbsp;  [⥂](https://www.google.com/search?q=%E4%BB%80%E9%BA%BC%E6%98%AFbluetooth+pan&rlz=1C1GCEU_zh-TWTW892TW892&sxsrf=ALeKk017XgrcIYWH3tJRkGCoU3-biKkjHQ%3A1623997669263&ei=5TzMYIehD6W0mAWYq424Dw&oq=%E4%BB%80%E9%BA%BC%E6%98%AFbluetooth+pan&gs_lcp=Cgdnd3Mtd2l6EAM6BwgAEEcQsAM6BwgjELACECc6BggAEAcQHjoECAAQDToECCMQJzoCCAA6CAgAEAcQChAeOgUIABDLAToFCAAQzQJQ7CdYn1tgt2NoA3ACeACAAVqIAYEHkgECMTSYAQCgAQGqAQdnd3Mtd2l6yAECwAEB&sclient=gws-wiz&ved=0ahUKEwjHp8ThxqDxAhUlGqYKHZhVA_cQ4dUDCA4&uact=5)

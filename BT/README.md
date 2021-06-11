## BT module hierarchy

![BT module hierachy](./diagram/BT%20module%20hierachy.png "BT module hierachy")


&emsp;
<br/>






## HCI Packet

### opcode :



&emsp;![opcode](./diagram/hci%20packet%20-%20opcode.png "hci packet - opcode")

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

![hci package format](./diagram/hci%20packet%20format.png "hci package format")




&emsp;
<br/>

## HCI Module



<b><font size=5>◤ </font></b>
<u><b><font size=3>system/bt</font></b></u>
<b><font size=4> :</font></b>



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




<b><font size=5>◤ </font></b>
<u><b><font size=3>hardware/interface/bt</font></b></u>
<b><font size=4> :</font></b>






&emsp;
&emsp;
<br/>


## Miscellaneous

### BT abbreviation
- [Bluetooth terminology](https://www.google.com/search?q=BLuetooth%E8%A1%93%E8%AA%9E&oq=BLuetooth%E8%A1%93%E8%AA%9E&aqs=chrome..69i57j0i333.12274j0j7&sourceid=chrome&ie=UTF-8)
- [What's Bluetooth LE (BLE)](https://www.google.com/search?q=Bluetooth+LE+%E6%98%AF%E4%BB%80%E9%BA%BC&oq=Bluetooth+LE+%E6%98%AF%E4%BB%80%E9%BA%BC&aqs=chrome..69i57j33i160.30570j0j7&sourceid=chrome&ie=UTF-8)

### BT HCI
- [蓝牙HCI command/event/acl/sco格式介绍_朝气蓬勃-CSDN博客](https://blog.csdn.net/XiaoXiaoPengBo/article/details/107638914)
- [如何分析HCI的Command Packet和Event Packet包_daydayupfromnowon的专栏-CSDN博客](https://blog.csdn.net/daydayupfromnowon/article/details/6324227)

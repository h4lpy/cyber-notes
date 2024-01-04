Universal Serial Bus (USB) drives are common flash storage device found in forensics investigations. They allow the removal and exfiltration of potentially sensitive data or the delivery of malware.

## Registry Keys

USB registry keys exist within the `SYSTEM` hive.

### USBSTOR

The registry key `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` holds information about external drives like USB, hard drives. etc.

![](/images/usbstor.png)

USBSTOR holds information such as:

- Model
- Version name
- Windows-assigned serial number
- Last connected timestamp

Opening the `SYSTEM` registry hive, we see subkeys under USBSTOR which are generated after connecting a USB to the system:

![](/images/usbstor_subkey.png)

This reveals a randomly-generated key when expanded, which corresponds to the device's assigned serial number by the USB:

![](/images/usbstor_serial.png)

Clicking on this reveals a lot of information about the device:

![](/images/usbstor_info.png)

From the above, we can see the **Friendly Name** of the device which can be useful in establishing the purpose or the drive or its owner. Similarly, the **container ID** can be used in contextual analysis from different data sources.

Expanding further and drilling down on the `Properties` key and expanding `{83da...}`, we see additional subkeys with timestamp information:

![](/images/usbstor_properties.png)

The key `0064` holds the timestamp of when the device was last connected to the system (in UTC):

![](/images/usbstor_connected_timestamp.png)

![](/images/usbstor_connected_timestamp_data.png)

Similarly, key `0066` contains the timestamp of when the USB was disconnected from the system:

![](/images/usbstor_disconnected_timestamp.png)

## USB

The registry key `HKLM\SYSTEM\CurrentControlSet\Enum\USB` contains information regarding all devices connected through USB ports, such as keyboards, adapters, etc. It exhibits a similar hierarchy as `USBSTOR`:

![](/images/usb_hierarchy.png)

![](/images/usb_data.png)

From the above, this is a Bluetooth adapter as confirmed by the `Service`value.

## Event Logs

Windows logs can also offer substantial value in USB device investigations and provide additional contest into how these devices are utilised on the system in question.

There are multiple data sources where context can be derived, most notably:

1. Partition
2. Kernel-PnP
3. NTFS

In **event viewer**, these logs are located in **Application and Service Logs -> Microsoft -> Windows**:

![](/images/usb_event_logs.png)

### Partition

![](/images/usb_parition_source.png)


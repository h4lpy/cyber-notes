Universal Serial Bus (USB) drives are common flash storage device found in forensics investigations. They allow the removal and exfiltration of potentially sensitive data or the delivery of malware.

## USBSTOR

The registry key `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` holds information about external drives like USB, hard drives. etc.

![](/images/usbstor.png)

USBSTOR holds information such as:

- Model
- Version name
- Windows-assigned serial number
- Last connected timestamp

Opening the `SYSTEM` registry hive, we see subkeys under USBSTOR which are generated after connecting a USB to the system:

![](/images/usbstor_subkey.png)


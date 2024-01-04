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

Looking at these logs, we see event ID 1006. Within the **details** tab, in addition to manufacturer, model, capacity, and serial number, we also see the corresponding timestamp identified in `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` which denotes the time of the USB connection. 

![](/images/usb_parition_diagnostic.png)

### Kernel-PnP

![](/images/kernel-pnp_source.png)

Firstly, looking at **event ID 400**, this is logged when an external device, such as a USB is configured/connected to the system. 

![](/images/kernel-pnp_event_400.png)

The GUID can be cross-referenced with to determine which USB was connected, as well as the timestamp information, and additional metadata:

![](/images/kernel-pnp_event_400_guid.png)

![](/images/kernel-pnp_event_400_timestamp.png)

### NTFS

![](/images/usb_ntfs_logs.png)

In NTFS operational logs, event ID 142 can be used around the connected timestamp to identify the **disk drive letter** that was given to the USB. 

This can be used in conjunction with file paths with this drive letter in further investigations to understand the files that were transferred to/from the USB device.

![](/images/usb_ntfs_event_412.png)

Using the above example, the drive was assigned the `E:` disk letter. Any file paths associated with this disk letter during its connection can be valuable context to a forensic investigation.

## Shellbags

Shellbags are created when a user interacts with the shell, the UI for accessing the OS, and the filesystem itself. They contain information about the state of a folder, such as its size, position, and items that it contains. This information is then stored in order to display the folder in the same state when it is accessed again - persistent configuration.

Shellbags can be useful in USB investigations, for example, if a user has accessed a folder containing sensitive files in a USB, the shellbag for the folder may contain information about the name and location of those documents. This can allow an investigator to the user if they are attempting to cover their tracks by deleting/moving files, and can sometimes may be the only record of this activity.

Shellbag locations:

- `NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU`
- `NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags`
- `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags`
- `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`

To explore shellbags, we can use [ShellbagExplorer](=https://ericzimmerman.github.io/#!index.md) by Eric Zimmerman.

![](/images/usb_shellbags_eztools.png)

![](/images/usb_shellbags_active.png)

Opening ShellbagExplorer **as admin** and selecting either `NTUSER.dat` or `USRCLASS.dat` gives the above view.

Using our previous example, we are interested in `E:`. Expanding this find a directory called `Secret_Project_LD`:

![](/images/usb_shellbags_e_Drive.png)

![](/images/usb_shellbags_secret_project.png)

From this view, we confirm this is a directory and we are provided with timestamps when it was first accessed, last accessed, etc.

Shellbags are stored within `NTUSER.dat` which is a user-centric hive, meaning that each user has their own `NTUSER.dat` file. Whichever user's `NTUSER.dat` file we view will correspond to their activity.

Shellbags also include Zip files if explorer through the Explorer and **not password-protected**. We can expand this and see the contents:

![](/images/usb_shellbags_zip.png)

![](/images/usb_shellbags_zip_expand.png)

These files are only stored in shellbags if visited via Explorer, thus giving us evidence of access.

## Jumplists

Jumplists - added in Windows 7 - provide users with quick access to recently accessed application files and common tasks. In an investigation, they can be used to determine files accessed, the corresponding programs used, and the timestamps of these activities. Most importantly, jumplists persist 

Jumplists can provide evidence of historical activity on the system (file creation, access, and modification), but also enables investigators to construct a timeline of user activities.
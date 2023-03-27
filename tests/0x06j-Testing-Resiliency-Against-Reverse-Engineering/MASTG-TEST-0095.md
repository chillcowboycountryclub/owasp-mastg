---
masvs_v1_id:
- MSTG-RESILIENCE-10
masvs_v2_id:
- MASVS-CRYPTO-2
platform: ios
title: Testing Device Binding
masvs_v1_levels:
- R
---

## Overview

## Static Analysis

To test for device binding you can look for:

- MAC addresses: there are several ways to find the MAC address. When you use `CTL_NET` (a network subsystem) or `NET_RT_IFLIST` (getting the configured interfaces) or when the mac-address gets formatted, you'll often see formatting code for printing, such as `"%x:%x:%x:%x:%x:%x"`.
- UDID usage: `[[[UIDevice currentDevice] identifierForVendor] UUIDString];` and `UIDevice.current.identifierForVendor?.uuidString` in Swift3.
- Any Keychain- or filesystem-based binding, which isn't protected by `SecAccessControlCreateFlags` or and doesn't use protection classes, such as `kSecAttrAccessibleAlways` and `kSecAttrAccessibleAlwaysThisDeviceOnly`.

## Dynamic Analysis

There are several ways to test the application binding.

**Using A Simulator:**

Take the following steps when you want to verify app-binding in a simulator:

1. Run the application on a simulator.
2. Make sure you can raise the trust in the application instance (e.g., authenticate in the app).
3. Retrieve the data from the Simulator:
    - Because simulators use UUIDs to identify themselves, you can make locating the storage easier by creating a debug point and executing `po NSHomeDirectory()` on that point, which will reveal the location of the simulator's stored contents. You can also execute `find ~/Library/Developer/CoreSimulator/Devices/ | grep <appname>` for the suspected plist file.
    - Go to the directory indicated by the given command's output.
    - Copy all three found folders (Documents, Library, tmp).
    - Copy the contents of the Keychain. Since iOS 8, this has been in `~/Library/Developer/CoreSimulator/Devices/<Simulator Device ID>/data/Library/Keychains`.
4. Start the application on another simulator and find its data location as described in step 3.
5. Stop the application on the second simulator. Overwrite the existing data with the data copied in step 3.
6. Can you continue in an authenticated state? If so, then binding may not be working properly.

We are saying that the binding "may" not be working because not everything is unique in simulators.

**Using Two Jailbroken Devices:**

Take the following steps when you want to verify app-binding with two jailbroken devices:

1. Run the app on your jailbroken device.
2. Make sure you can raise the trust in the application instance (e.g., authenticate in the app).
3. Retrieve the data from the jailbroken device:
    - You can SSH into your device and extract the data (as with a simulator, either use debugging or `find /private/var/mobile/Containers/Data/Application/ |grep <name of app>`). The directory is in `/private/var/mobile/Containers/Data/Application/<Application uuid>`.
    - SSH into the directory indicated by the given command's output or use SCP (`scp <ipaddress>:/<folder_found_in_previous_step> targetfolder`) to copy the folders and it's data. You can use an FTP client like Filezilla as well.
    - Retrieve the data from the keychain, which is stored in `/private/var/Keychains/keychain-2.db`, which you can retrieve using [Keychain-dumper](0x08a-Testing-Tools.md#keychain-dumper).
4. Install the application on the second jailbroken device.
5. Overwrite the application data extracted during step 3. The Keychain data must be added manually.
6. Can you continue in an authenticated state? If so, then binding may not be working properly.
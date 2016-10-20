---
title: Smart Cards for Windows Service
description: "Windows Server Security"
ms.custom: na
ms.prod: windows-server-threshold
ms.reviewer: na
ms.suite: na
ms.technology: security-smart-cards
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 95140782-f765-428d-af3c-73fa67f0cc9b
author: coreyp-at-msft
ms.author: coreyp
manager: dongill
ms.date: 10/12/2016
---
# Smart Cards for Windows Service

>Applies To: Windows Server&reg; 2016, Windows Server&reg; 2012 R2, Windows Server&reg; 2012

This topic for the IT professional and smart card developers describes how the Smart Cards for Windows service (formerly called Smart Card Resurce Manager) manages readers and application interactions.

The Smart Cards for Windows service provides the basic infrastructure for all other smart card components as it manages smart card readers and application interactions on the computer. It is fully compliant with the specifications set by the PC/SC Workgroup. For information about these specifications, see the [PC/SC Workgroup library](http://www.pcscworkgroup.com/specifications/specdownloadV1.php).

The Smart Cards for Windows service runs in the context of a local service, and it is implemented as a shared service of the services host (svchost) process. The Smart Cards for Windows service, Scardsvr, has the following service description:

```
<serviceData
    dependOnService="PlugPlay"
    description="@%SystemRoot%\System32\SCardSvr.dll,-5"
    displayName="@%SystemRoot%\System32\SCardSvr.dll,-1"
    errorControl="normal"
    group="SmartCardGroup"
    imagePath="%SystemRoot%\system32\svchost.exe -k LocalServiceAndNoImpersonation"
    name="SCardSvr"
    objectName="NT AUTHORITY\LocalService"
    requiredPrivileges="SeCreateGlobalPrivilege,SeChangeNotifyPrivilege"
    sidType="unrestricted"
    start="demand"
    type="win32ShareProcess"
    >
  <failureActions resetPeriod="900">
       <actions>
          <action
              delay="120000"
              type="restartService"
          />
          <action
              delay="300000"
              type="restartService"
          />
          <action
               delay="0"
              type="none"
          />
      </actions>
  </failureActions>
  <securityDescriptor name="ServiceXSecurity"/>
</serviceData>

  <registryKeys buildFilter="">
      <registryKey keyName="HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SCardSvr\Parameters">
      <registryValue
          name="ServiceDll"
          value="%SystemRoot%\System32\SCardSvr.dll"
          valueType="REG_EXPAND_SZ"
          />
      <registryValue
          name="ServiceMain"
          value="CalaisMain"
          valueType="REG_SZ"
          />
      <registryValue
          name="ServiceDllUnloadOnStop"
          value="1"
          valueType="REG_DWORD"
          />
      </registryKey>
  </registryKeys>

```

> [!NOTE]
> For winscard.dll to be invoked as the proper class installer, the INF file for a smart card reader must specify the following for **Class** and **ClassGUID**:
> 
> `Class=SmartCardReader`
> 
> `ClassGuid={50DD5230-BA8A-11D1-BF5D-0000F805F530}`

By default, the service is configured for manual mode. Creators of smart card reader drivers must configure their INFs so that they start the service automatically and winscard.dll files call a predefined entry point to start the service during installation. The entry point is defined as part of the **SmartCardReader** class, and it is not called directly. If a device advertises itself as part of this class, the entry point is automatically invoked to start the service when the device is inserted. Using this method ensures that the service is enabled when it is needed, but it is also disabled for users who do not use smart cards.

When the service is started, it performs several functions:

1.  It registers itself for service notifications.

2.  It registers itself for Plug and Play (PnP) notifications related to device removal and additions.

3.  It initializes its data cache and a global event that signals that the service has started.

> [!NOTE]
> For smart card implementations, consider sending all communications in Windows operating systems with smart card readers through the Smart Cards for Windows service. This provides an interface to track, select, and communicate with all drivers that declare themselves members of the smart card reader device group.

The Smart Cards for Windows service categorizes each smart card reader slot as a unique reader, and each slot is also managed separately, regardless of the device's physical characteristics. The Smart Cards for Windows service handles the following high-level actions:

-   Device introduction

-   Reader initialization

-   Notifying clients of new readers

-   Serializing access to readers

-   Smart card access

-   Tunneling of reader-specific commands




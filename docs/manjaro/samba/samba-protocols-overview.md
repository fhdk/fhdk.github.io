---
title: 'SMB protocols overview'
date: '06:07 20-12-2024'
taxonomy:
    category:
        - docs
---

## Protocol availability
- client min protocol – This setting controls the minimum protocol version that the client will attempt to use.
- client max protocol – The value of the parameter (a string) is the highest protocol level that will be supported by the client.

## Available values
* SMB2: Re-implementation of the SMB protocol. Used by Windows Vista and later versions of Windows. SMB2 has sub protocols available:
  * SMB2_02: The earliest SMB2 version.
  * SMB2_10: Windows 7 SMB2 version. (By default SMB2 selects the SMB2_10 variant.)
  * SMB2_22: Early Windows 8 SMB2 version.
  * SMB2_24: Windows 8 beta SMB2 version.
  * SMB3: The same as SMB2. Used by Windows 8. SMB3 has sub protocols available.
* SMB3_00: Windows 8 SMB3 version. (mostly the same as SMB2_24)
  * SMB3_02: Windows 8.1 SMB3 version.
  * SMB3_10: early Windows 10 technical preview SMB3 version.
  * SMB3_11: Windows 10 technical preview SMB3 version (maybe final). By default SMB3 selects the SMB3_11 variant.

!!!! This combination provides the optimal function  
!!!! client min version = SMB2  
!!!! client max version = SMB 3  

Due to the exploitation by ransomware SMBv1 has been disabled on a Linux or Unix samba server to avoid security issues.

If you have a very old system - e.g. a router or a NAS which is no longer accessible NT1 can be assigned as protocol version.
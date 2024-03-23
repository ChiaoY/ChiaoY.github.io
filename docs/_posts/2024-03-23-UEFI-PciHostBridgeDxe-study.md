---
layout: post
title:  "UEFI PciHostBridgeDxe Study"
date:   2024-03-23
categories: BIOS
tags: BIOS, UEFI
---



# Introduction
PCI bus subsystem is a important subsystem for modern computers, either PC or 
HPC uses it to connect many important devices. For example, storage, NIC, graphic 
cards are using PCI buses to connect to processors. It also important for BIOS 
engineers to unerstand the stracture of the PCI bus to debug the issues related
to PCI devices. There are two most important DXE drivers for PCI enumeration in
UEFI, PciHostBridgeDxe and PciBusDxe.

Here are the location of these DXEs.
```
MdeModulePkg/Bus/Pci/PciHostBridgeDxe/PciHostBridgeDxe.inf
MdeModulePkg/Bus/Pci/PciBusDxe/PciBusDxe.inf
```

# PciHostBridgeDxe

## PciHostBridgeDxe.inf
From this INF file we can know that the entry point for this driver is 
`PciHostBridgeDxe()`
```c
[Defines]
  INF_VERSION                    = 0x00010005
  BASE_NAME                      = PciHostBridgeDxe
  FILE_GUID                      = 128FB770-5E79-4176-9E51-9BB268A17DD1
  MODULE_TYPE                    = DXE_DRIVER
  VERSION_STRING                 = 1.0
  ENTRY_POINT                    = InitializePciHostBridge

[Sources]
  PciHostBridge.h
  PciRootBridge.h
  PciHostBridge.c
  PciRootBridgeIo.c
  PciHostResource.h

[Packages]
  MdePkg/MdePkg.dec
  MdeModulePkg/MdeModulePkg.dec

[LibraryClasses]
  UefiDriverEntryPoint
  UefiBootServicesTableLib
  DebugLib
  DxeServicesTableLib
  DevicePathLib
  BaseMemoryLib
  BaseLib
  PciSegmentLib
  UefiLib
  PciHostBridgeLib
  TimerLib

[Protocols]
  gEfiCpuIo2ProtocolGuid                          ## CONSUMES
  gEfiDevicePathProtocolGuid                      ## BY_START
  gEfiPciRootBridgeIoProtocolGuid                 ## BY_START
  gEfiPciHostBridgeResourceAllocationProtocolGuid ## BY_START
  gEdkiiIoMmuProtocolGuid                         ## SOMETIMES_CONSUMES

[Depex]
  gEfiCpuIo2ProtocolGuid AND
  gEfiCpuArchProtocolGuid
```

## PciHostBridge.c

### InitializePciHostBridge()

Step 1: Get the Host bridge
```c
  /*
  Chiao :
  PciHostBridgeGetRootBridges:
  This function is implemented differently between platforms
  It return all the root bridge instances in an array and get the number of the 
  root bridges
  For OvmfPkgX64.dsc, it references OvmfPkg/Library/PciHostBridgeLib/PciHostBridgeLib.c
  */
  RootBridges = PciHostBridgeGetRootBridges (&RootBridgeCount);
  if ((RootBridges == NULL) || (RootBridgeCount == 0)) {
    return EFI_UNSUPPORTED;
  }

  /*
  Chiao: from edk2/MdePkg/Include/Protocol/CpuIo2.h
  This protocol provides an I/O abstraction for a system processor. This protocol
  is used by a PCI root bridge I/O driver to perform memory-mapped I/O and I/O transactions.
  The I/O or memory primitives can be used by the consumer of the protocol to materialize
  bus-specific configuration cycles, such as the transitional configuration address and data
  ports for PCI. Only drivers that require direct access to the entire system should use this
  protocol.
  */
  Status = gBS->LocateProtocol (&gEfiCpuIo2ProtocolGuid, NULL, (VOID **)&mCpuIo);
  ASSERT_EFI_ERROR (Status);

  //
  // Most systems in the world including complex servers have only one Host Bridge.
  //
  HostBridge = AllocateZeroPool (sizeof (PCI_HOST_BRIDGE_INSTANCE));
  ASSERT (HostBridge != NULL);

  HostBridge->Signature    = PCI_HOST_BRIDGE_SIGNATURE;
  HostBridge->CanRestarted = TRUE;
  // Chiao: Use doubly linked list to store the host bridges
  InitializeListHead (&HostBridge->RootBridges);
  ResourceAssigned = FALSE;

  //
  // Create Root Bridge Device Handle in this Host Bridge
  //
  // Chiao: Iterate through all root bridges
  for (Index = 0; Index < RootBridgeCount; Index++) {
    //
    // Create Root Bridge Handle Instance
    //
    RootBridge = CreateRootBridge (&RootBridges[Index]);
    ASSERT (RootBridge != NULL);
    if (RootBridge == NULL) {
      continue;
    }
    //
    // Make sure all root bridges share the same ResourceAssigned value.
    //
    if (Index == 0) {
      ResourceAssigned = RootBridges[Index].ResourceAssigned;
    } else {
      ASSERT (ResourceAssigned == RootBridges[Index].ResourceAssigned);
    }

    if (RootBridges[Index].Io.Base <= RootBridges[Index].Io.Limit) {
      //
      // Base and Limit in PCI_ROOT_BRIDGE_APERTURE are device address.
      // For GCD resource manipulation, we need to use host address.
      //
      HostAddress = TO_HOST_ADDRESS (
                      RootBridges[Index].Io.Base,
                      RootBridges[Index].Io.Translation
                      );

      Status = AddIoSpace (
                 HostAddress,
                 RootBridges[Index].Io.Limit - RootBridges[Index].Io.Base + 1
                 );
      ASSERT_EFI_ERROR (Status);
      if (ResourceAssigned) {
        Status = gDS->AllocateIoSpace (
                        EfiGcdAllocateAddress,
                        EfiGcdIoTypeIo,
                        0,
                        RootBridges[Index].Io.Limit - RootBridges[Index].Io.Base + 1,
                        &HostAddress,
                        gImageHandle,
                        NULL
                        );
        ASSERT_EFI_ERROR (Status);
      }
    }

```
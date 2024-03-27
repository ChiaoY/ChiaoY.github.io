---
layout: post
title:  "UEFI PciHostBridgeDxe Study"
date:   2024-03-23
categories: BIOS
tags: BIOS, UEFI
---



## Introduction
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

## PciHostBridgeDxe

### PciHostBridgeDxe.inf
From this INF file we can know that the entry point for this driver is 
`PciHostBridgeDxe()`
```=
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

### PciRootBridge.h
```c=
// PCI_ROOT_BRIDGE_INSTANCE stuct
typedef struct {
  UINT32                             Signature;
  LIST_ENTRY                         Link;
  EFI_HANDLE                         Handle;
  UINT64                             AllocationAttributes;
  UINT64                             Attributes;
  UINT64                             Supports;
  PCI_RES_NODE                       ResAllocNode[TypeMax];
  PCI_ROOT_BRIDGE_APERTURE           Bus;
  PCI_ROOT_BRIDGE_APERTURE           Io;
  PCI_ROOT_BRIDGE_APERTURE           Mem;
  PCI_ROOT_BRIDGE_APERTURE           PMem;
  PCI_ROOT_BRIDGE_APERTURE           MemAbove4G;
  PCI_ROOT_BRIDGE_APERTURE           PMemAbove4G;
  BOOLEAN                            DmaAbove4G;
  BOOLEAN                            NoExtendedConfigSpace;
  VOID                               *ConfigBuffer;
  EFI_DEVICE_PATH_PROTOCOL           *DevicePath;
  CHAR16                             *DevicePathStr;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL    RootBridgeIo;

  BOOLEAN                            ResourceSubmitted;
  LIST_ENTRY                         Maps;
} PCI_ROOT_BRIDGE_INSTANCE;

// PCI_HOST_BRIDGE_INSTANCE struct
typedef struct {
  UINT32                             Signature;
  LIST_ENTRY                         Link;
  EFI_HANDLE                         Handle;
  UINT64                             AllocationAttributes;
  UINT64                             Attributes;
  UINT64                             Supports;
  PCI_RES_NODE                       ResAllocNode[TypeMax];
  PCI_ROOT_BRIDGE_APERTURE           Bus;
  PCI_ROOT_BRIDGE_APERTURE           Io;
  PCI_ROOT_BRIDGE_APERTURE           Mem;
  PCI_ROOT_BRIDGE_APERTURE           PMem;
  PCI_ROOT_BRIDGE_APERTURE           MemAbove4G;
  PCI_ROOT_BRIDGE_APERTURE           PMemAbove4G;
  BOOLEAN                            DmaAbove4G;
  BOOLEAN                            NoExtendedConfigSpace;
  VOID                               *ConfigBuffer;
  EFI_DEVICE_PATH_PROTOCOL           *DevicePath;
  CHAR16                             *DevicePathStr;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL    RootBridgeIo;

  BOOLEAN                            ResourceSubmitted;
  LIST_ENTRY                         Maps;
} PCI_ROOT_BRIDGE_INSTANCE;
```
### PciHostBridgeLib.h
```c=
// PCI_ROOT_BRIDGE struct
typedef struct {
  UINT32                      Segment;               ///< Segment number.
  UINT64                      Supports;              ///< Supported attributes.
                                                     ///< Refer to EFI_PCI_ATTRIBUTE_xxx used by GetAttributes()
                                                     ///< and SetAttributes() in EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL.
  UINT64                      Attributes;            ///< Initial attributes.
                                                     ///< Refer to EFI_PCI_ATTRIBUTE_xxx used by GetAttributes()
                                                     ///< and SetAttributes() in EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL.
  BOOLEAN                     DmaAbove4G;            ///< DMA above 4GB memory.
                                                     ///< Set to TRUE when root bridge supports DMA above 4GB memory.
  BOOLEAN                     NoExtendedConfigSpace; ///< When FALSE, the root bridge supports
                                                     ///< Extended (4096-byte) Configuration Space.
                                                     ///< When TRUE, the root bridge supports
                                                     ///< 256-byte Configuration Space only.
  BOOLEAN                     ResourceAssigned;      ///< Resource assignment status of the root bridge.
                                                     ///< Set to TRUE if Bus/IO/MMIO resources for root bridge have been assigned.
  UINT64                      AllocationAttributes;  ///< Allocation attributes.
                                                     ///< Refer to EFI_PCI_HOST_BRIDGE_COMBINE_MEM_PMEM and
                                                     ///< EFI_PCI_HOST_BRIDGE_MEM64_DECODE used by GetAllocAttributes()
                                                     ///< in EFI_PCI_HOST_BRIDGE_RESOURCE_ALLOCATION_PROTOCOL.
  PCI_ROOT_BRIDGE_APERTURE    Bus;                   ///< Bus aperture which can be used by the root bridge.
  PCI_ROOT_BRIDGE_APERTURE    Io;                    ///< IO aperture which can be used by the root bridge.
  PCI_ROOT_BRIDGE_APERTURE    Mem;                   ///< MMIO aperture below 4GB which can be used by the root bridge.
  PCI_ROOT_BRIDGE_APERTURE    MemAbove4G;            ///< MMIO aperture above 4GB which can be used by the root bridge.
  PCI_ROOT_BRIDGE_APERTURE    PMem;                  ///< Prefetchable MMIO aperture below 4GB which can be used by the root bridge.
  PCI_ROOT_BRIDGE_APERTURE    PMemAbove4G;           ///< Prefetchable MMIO aperture above 4GB which can be used by the root bridge.
  EFI_DEVICE_PATH_PROTOCOL    *DevicePath;           ///< Device path.
} PCI_ROOT_BRIDGE;
```

### PciHostBridge.c

#### Steps

1. Get root bridges and their attributes, stores in `RootBridges` array.
2. Get CPU_IO_2_PROTOCOL
    1. TODO: Why
3. Create a doubly linked list `HostBridge->RootBridges` to store the configured root bridges
4. Iterate thorugh the root bridges and congure them
    1. Create root bridge device handle instance `RootBridge`
        1. TODO: PciBusDxe related
    2. Make sure all root bridges share the same `ResourceAssigned` value
        1. TODO: Why?
    3. IO space
        1. `AddIoSpace`: Add IO space to GCD
        2. `gDS->AllocateIoSpace`: If `ResourceAssigned`, Allocate IO resource from GCD of the processor
    4. Add all Mem/PMem to GCD
        1. `AddMemoryMappedIoSpace`: Add MMIO space to GCD
        2. `gDS->SetmemorySpaceAttributes`: Modifies the attributes for a memory region in the GCD of the processor.
        3. `gDS->AllocateMemorySpace`: If `ResourceAssigned`,  allocate memory from GCD of processor
5. Add the configured root bridge into the tail of the linked list `HostBridge->RootBridges`.
6. If NOT `ResourceAssigned`, expose `PciHostBridgeResourceAllocation` Protocal to host bridge
    - initializes a HostBridge structure with several function pointers like NotifyPhase, GetNextRootBridge, etc. These functions are defined in `PciHostBridgeDxe.c` and handle different aspects of resource allocation for the PCI host bridge.

5. Install DevicePath Protocol and PciRootBridgeIo Protocol for each root bridges
6. Free the root bridge instances array returned from PciHostBridgeGetRootBridges().


#### InitializePciHostBridge()
```clike=435
EFI_STATUS
EFIAPI
InitializePciHostBridge (
  IN EFI_HANDLE        ImageHandle,
  IN EFI_SYSTEM_TABLE  *SystemTable
  )
{
  EFI_STATUS                Status;
  PCI_HOST_BRIDGE_INSTANCE  *HostBridge;
  PCI_ROOT_BRIDGE_INSTANCE  *RootBridge;
  PCI_ROOT_BRIDGE           *RootBridges;
  UINTN                     RootBridgeCount;
  UINTN                     Index;
  PCI_ROOT_BRIDGE_APERTURE  *MemApertures[4];
  UINTN                     MemApertureIndex;
  BOOLEAN                   ResourceAssigned;
  LIST_ENTRY                *Link;
  UINT64                    HostAddress;

  /*
  Chiao :
  PciHostBridgeGetRootBridges:
  This function is implemented differently between platforms
  It return all the root bridge instances in an array and get the number of the 
  root bridges
  For OvmfPkgX64.dsc, it references OvmfPkg/Library/PciHostBridgeLib/PciHostBridgeLib.c
  This function fetches all of the data of each PCI root bridge and fills the each element
  of PCI_ROOT_BRIDGE into it. 
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
    // TODO: find why
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
      // Chiao: GCD (Global Coherency Domain) manages memory and IO resources seen
      //        by processor
      HostAddress = TO_HOST_ADDRESS (
                      RootBridges[Index].Io.Base,
                      RootBridges[Index].Io.Translation
                      );

      // Chiao: add IO space to GCD
      // TODO: difference between AddIoSpace and gDS->AllocateIoSpace
      Status = AddIoSpace (
                 HostAddress,
                 RootBridges[Index].Io.Limit - RootBridges[Index].Io.Base + 1
                 );
      ASSERT_EFI_ERROR (Status);
      if (ResourceAssigned) {
        // Chiao: Allocates nonexistent I/O, reserved I/O, or I/O resources from 
        //        the global coherency domain of the processor.
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

    //
    // Add all the Mem/PMem aperture to GCD
    // Mem/PMem shouldn't overlap with each other
    // Root bridge which needs to combine MEM and PMEM should only report
    // the MEM aperture in Mem
    //
    MemApertures[0] = &RootBridges[Index].Mem;
    MemApertures[1] = &RootBridges[Index].MemAbove4G;
    MemApertures[2] = &RootBridges[Index].PMem;
    MemApertures[3] = &RootBridges[Index].PMemAbove4G;

    for (MemApertureIndex = 0; MemApertureIndex < ARRAY_SIZE (MemApertures); MemApertureIndex++) {
      if (MemApertures[MemApertureIndex]->Base <= MemApertures[MemApertureIndex]->Limit) {
        //
        // Base and Limit in PCI_ROOT_BRIDGE_APERTURE are device address.
        // For GCD resource manipulation, we need to use host address.
        //
        HostAddress = TO_HOST_ADDRESS (
                        MemApertures[MemApertureIndex]->Base,
                        MemApertures[MemApertureIndex]->Translation
                        );
        Status = AddMemoryMappedIoSpace (
                   HostAddress,
                   MemApertures[MemApertureIndex]->Limit - MemApertures[MemApertureIndex]->Base + 1,
                   EFI_MEMORY_UC
                   );
        ASSERT_EFI_ERROR (Status);
        Status = gDS->SetMemorySpaceAttributes (
                        HostAddress,
                        MemApertures[MemApertureIndex]->Limit - MemApertures[MemApertureIndex]->Base + 1,
                        EFI_MEMORY_UC
                        );
        if (EFI_ERROR (Status)) {
          DEBUG ((DEBUG_WARN, "PciHostBridge driver failed to set EFI_MEMORY_UC to MMIO aperture - %r.\n", Status));
        }

        if (ResourceAssigned) {
          Status = gDS->AllocateMemorySpace (
                          EfiGcdAllocateAddress,
                          EfiGcdMemoryTypeMemoryMappedIo,
                          0,
                          MemApertures[MemApertureIndex]->Limit - MemApertures[MemApertureIndex]->Base + 1,
                          &HostAddress,
                          gImageHandle,
                          NULL
                          );
          ASSERT_EFI_ERROR (Status);
        }
      }
    }

    //
    // Insert Root Bridge Handle Instance
    //
    InsertTailList (&HostBridge->RootBridges, &RootBridge->Link);
  }

  //
  // When resources were assigned, it's not needed to expose
  // PciHostBridgeResourceAllocation protocol.
  //
  if (!ResourceAssigned) {
    HostBridge->ResAlloc.NotifyPhase          = NotifyPhase;
    HostBridge->ResAlloc.GetNextRootBridge    = GetNextRootBridge;
    HostBridge->ResAlloc.GetAllocAttributes   = GetAttributes;
    HostBridge->ResAlloc.StartBusEnumeration  = StartBusEnumeration;
    HostBridge->ResAlloc.SetBusNumbers        = SetBusNumbers;
    HostBridge->ResAlloc.SubmitResources      = SubmitResources;
    HostBridge->ResAlloc.GetProposedResources = GetProposedResources;
    HostBridge->ResAlloc.PreprocessController = PreprocessController;

    Status = gBS->InstallMultipleProtocolInterfaces (
                    &HostBridge->Handle,
                    &gEfiPciHostBridgeResourceAllocationProtocolGuid,
                    &HostBridge->ResAlloc,
                    NULL
                    );
    ASSERT_EFI_ERROR (Status);
  }

  for (Link = GetFirstNode (&HostBridge->RootBridges)
       ; !IsNull (&HostBridge->RootBridges, Link)
       ; Link = GetNextNode (&HostBridge->RootBridges, Link)
       )
  {
    RootBridge                            = ROOT_BRIDGE_FROM_LINK (Link);
    RootBridge->RootBridgeIo.ParentHandle = HostBridge->Handle;

    Status = gBS->InstallMultipleProtocolInterfaces (
                    &RootBridge->Handle,
                    &gEfiDevicePathProtocolGuid,
                    RootBridge->DevicePath,
                    &gEfiPciRootBridgeIoProtocolGuid,
                    &RootBridge->RootBridgeIo,
                    NULL
                    );
    ASSERT_EFI_ERROR (Status);
  }

  PciHostBridgeFreeRootBridges (RootBridges, RootBridgeCount);

  if (!EFI_ERROR (Status)) {
    mIoMmuEvent = EfiCreateProtocolNotifyEvent (
                    &gEdkiiIoMmuProtocolGuid,
                    TPL_CALLBACK,
                    IoMmuProtocolCallback,
                    NULL,
                    &mIoMmuRegistration
                    );
  }

  return Status;
}

```
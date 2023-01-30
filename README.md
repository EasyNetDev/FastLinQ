# ql4xxxx

This driver is based on original Marvell FastLinQ QL4XXXX 8.70.12.0 which can be compiled until Linux Kernel 5.15.x.
Because I notice that the Linux QED/QEDE driver is crashing each 2 reboots, I wanted to try this driver, but I couldn't compile it on Kernel 6.1.x.

With this patched driver I was able to compile it up to Linux 6.1.0 without issues and without crashes. Is the same code but adapted to new Linux Kernel API, like moving out functions or structures outside the main structures and they must be used coded in driver.
Examples:
1. `scsi_cmnd` moved out the commands `scsi_done` and the structure `SCp` must be implemented as `scsi_pointer` in driver.
2. Functions that are obsolete for long time and they were only wrappers to other functions, now they are gone and the code must be adapted to the new functions.

### Change log
#### QEDE driver
##### qede-8.70.12.0/src/qede_main.c 
1. Changed `netif_napi_add` to `netif_napi_add_weight`. Starting from Linux >=6.1 `netif_napi_add` has only 3 parameters.

In Linux 6.1 `netif_napi_add` changed the number of parameters from 4 to 3, so when you are compiling the driver the compiler is complaining about too many paramenters for `netif_napi_add` functions. Acctually `netif_napi_add` is a wrapper to `netif_napi_add_weight` function.

#### QEDF driver
##### qedf-8.70.12.0/qedf.h
1. Changed the the code compatibility up to 5.15 for old calls: `#define CMD_SCSI_STATUS(Cmnd)` is needed only up to 5.15.
2. Starting from Linux 5.16.0 `scsi_pointer` is not part of `scsi_cmnd anymore`. Needs to be implemented separate in the driver code.
3. Starting from Linux 5.16.0 `device_attribute` is obsolete. Need to move to `attribute_group`.

##### qedf-8.70.12.0/qedf_attr.c

1. Move from `device_attribute` to `attribute_group`.
2. `scsi_cmnd->request` moved out from `scsi_cmnd`. Added code for `request` for Linux >=5.16.
3. Added code for `scsi_pointer` for Linux >=5.16.
4. From Linux >=5.16 `scsi_cmnd` moved out the command `scsi_done`. Added code to fix it.

##### qedf-8.70.12.0/qedf_main.c

1. Added code for `scsi_pointer` for Linux >=5.16.
2. Move out from `device_attribute` to `attribute_group` for Linux >=5.16.
3. Move out from `pci_alloc_consistent` to `dma_alloc_coherent`. Function `pci_alloc_consistent` was a wrapper to `dma_alloc_coherent` and it was obsolete. For Linux >=5.18.
4. Move out from `pci_free_consistent` to `dma_free_coherent`. Function `pci_free_consistent` was a wrapper to `dma_free_coherent` and it was obsolete. For Linux >=5.18.

#### QEDI driver
##### qedf-8.70.12.0/qedi.h
1. Starting from Linux 5.16.0 `scsi_pointer` is not part of `scsi_cmnd anymore`. Needs to be implemented separate in the driver code.
2. Starting from Linux 5.16.0 `device_attribute` is obsolete. Need to move to `attribute_group`.

##### qedf-8.70.12.0/qedi_iscsi.c
1. Move out from `device_attribute` to `attribute_group` for Linux >=5.16.

##### qedf-8.70.12.0/qedi_main.c
1. Added code for `scsi_pointer` for Linux >=5.16.
2. `scsi_cmnd->request` moved out from `scsi_cmnd`. Added code for `request` for Linux >=5.16.
3. Change from `random32` / `prandom_u32` to `get_random_u32` for Linux >=5.16.
4. Added code for `scsi_pointer` for Linux >=5.16.
5. Move out from `pci_alloc_consistent` to `dma_alloc_coherent`. Function `pci_alloc_consistent` was a wrapper to `dma_alloc_coherent` and it was obsolete. For Linux >=5.18.
6. Move out from `pci_free_consistent` to `dma_free_coherent`. Function `pci_free_consistent` was a wrapper to `dma_free_coherent` and it was obsolete. For Linux >=5.18.

##### qedf-8.70.12.0/qedi_sysfs.c
1. Starting from Linux 5.16.0 `device_attribute` is obsolete. Need to move to `attribute_group`.

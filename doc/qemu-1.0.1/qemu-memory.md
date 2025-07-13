# QEMU memory model

## 1. RAMList

## Types
- RAM: a RAM region is simply a range of host memory that can be made available to the guest
- MMIO: a range of guest memory that is implemented by host callbacks; each read or write causes a callback to be called on the host
- container: a container simply includes other memory regions, each at a different offset
- alias: a subsection of another region


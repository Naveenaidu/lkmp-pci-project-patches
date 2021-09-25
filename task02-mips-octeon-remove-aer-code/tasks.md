### Remove redundant AER code from arch and driver code

Remove the generic parts of the AER code in arch/mips/pci/pci-octeon.c
`pcibios_plat_dev_init()`.  Clearing the status registers should already be
done by the PCI core (this needs to be verified).

 Most PCI hardware errors (device not responding, device removed, etc)
 cause config and MMIO reads to return ~0.  We should add a #define or
 similar common way to check for that value so it's easier to see where
 drivers do this.  See this
 [thread](https://lore.kernel.org/linux-pci/20190822200551.129039-1-helgaas@kernel.org/).
    

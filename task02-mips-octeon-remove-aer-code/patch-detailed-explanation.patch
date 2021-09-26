From 2bd9d93e5a12d25cdad7de5313e6765e1104cea6 Mon Sep 17 00:00:00 2001
From: Naveen Naidu <naveennaidu479@gmail.com>
Date: Wed, 22 Sep 2021 00:41:26 +0530
Subject: [PATCH] MIPS: OCTEON: Remove redundant AER code from pcibios_plat_dev_init()
To: helgaas@kernel.org

pcibios_plat_dev_init() is used to perform platform specific device
initialization when pci_enable_device() is called.

Octeon's pcibios_plat_dev_init() implementation regarding AER (Advance
Error Reporting) code does the following:
  1. Clears the Uncorrectable/Correctable error status register
  2. Clears the root status registers
  3. Enables the PCIe normal error reporting
  4. Enables reporting of all correctable and uncorrectable errors
  5. ECRC Generation enable
  6. Enable Root Port's interrupt in response to all error messages

But most of the above work are generic and is already handled by the PCI
core.

The execution stack is as follows:
  1. PCI Bus enumeration starts
  2. pcibios_plat_dev_init() is called to enable a device
  3. The PCI port service driver is started, which initializes the AER
     port driver.

PCI Bus Enumeration
===================

The execution stack, which leads to clearing of AER registers is:

  pci_scan_slot()
    pci_scan_single_device()
      pci_init_capabilities()
        pci_aer_init()
           pci_aer_clear_status()
             pci_aer_raw_clear_status()  <--- This is where the PCI_ERR_ROOT_STATUS,
                                              PCI_ERR_COR_STATUS and PCI_ERR_UNCOR_STATUS
                                              are cleared

As can be seen above during the PCI Bus Enumeration we clear the AER
registers. i.e the Task 1 and Task 2 which Octeon does are already done
during the Bus enumeration

PCI Port Service Driver is Initialized
=====================================

The execution stack, which leads to enabling of error reporting is:

  aer_probe()
    aer_enable_rootport()
       set_downstream_devices_error_reporting()
         set_device_error_reporting()
           pci_enable_pcie_error_reporting() <-- Enables the PCIe normal error reporting
           pcie_set_ecrc_checking()          <-- ECRC Generation enable
       pci_write_config_dword(pdev, aer + PCI_ERR_ROOT_COMMAND ...) <-- Enable RP's interrupt to all
                                                                         error messages

As you can see from above, when AER port serive driver is initialized,
we do the Task 3, 5 and 6

To summarize:

  - PCI Bus Enumeration
      * Clears all the AER status registers (Task 1 and 2)

  - AER Port service driver
      * Enables PCIe normal error reporting (Task 3)
      * ECRC Generation enable (Task 5)
      * Enable Root Port's interrupt in response to all error messages (Task 6)

This means that Octeon's pcibios_plat_dev_init() is doing a lot of
redundant work  pertaining to AER since these are already handled by the
PCI core. The only necessary task which Octeon code should do is the
Task 4, which is "Enables reporting of all correctable and uncorrectable
errors".

This patch removes the redundant code and keeps only the necessary task.

Signed-off-by: Naveen Naidu <naveennaidu479@gmail.com>
---
 arch/mips/pci/pci-octeon.c | 40 +-------------------------------------
 1 file changed, 1 insertion(+), 39 deletions(-)

diff --git a/arch/mips/pci/pci-octeon.c b/arch/mips/pci/pci-octeon.c
index fc29b85cfa92..3d742ae8e6a6 100644
--- a/arch/mips/pci/pci-octeon.c
+++ b/arch/mips/pci/pci-octeon.c
@@ -114,54 +114,16 @@ int pcibios_plat_dev_init(struct pci_dev *dev)
 		pci_write_config_word(dev, PCI_BRIDGE_CONTROL, config);
 	}
 
-	/* Enable the PCIe normal error reporting */
-	config = PCI_EXP_DEVCTL_CERE; /* Correctable Error Reporting */
-	config |= PCI_EXP_DEVCTL_NFERE; /* Non-Fatal Error Reporting */
-	config |= PCI_EXP_DEVCTL_FERE;	/* Fatal Error Reporting */
-	config |= PCI_EXP_DEVCTL_URRE;	/* Unsupported Request */
-	pcie_capability_set_word(dev, PCI_EXP_DEVCTL, config);
-
 	/* Find the Advanced Error Reporting capability */
 	pos = pci_find_ext_capability(dev, PCI_EXT_CAP_ID_ERR);
 	if (pos) {
-		/* Clear Uncorrectable Error Status */
-		pci_read_config_dword(dev, pos + PCI_ERR_UNCOR_STATUS,
-				      &dconfig);
-		pci_write_config_dword(dev, pos + PCI_ERR_UNCOR_STATUS,
-				       dconfig);
 		/* Enable reporting of all uncorrectable errors */
 		/* Uncorrectable Error Mask - turned on bits disable errors */
 		pci_write_config_dword(dev, pos + PCI_ERR_UNCOR_MASK, 0);
-		/*
-		 * Leave severity at HW default. This only controls if
-		 * errors are reported as uncorrectable or
-		 * correctable, not if the error is reported.
-		 */
-		/* PCI_ERR_UNCOR_SEVER - Uncorrectable Error Severity */
-		/* Clear Correctable Error Status */
-		pci_read_config_dword(dev, pos + PCI_ERR_COR_STATUS, &dconfig);
-		pci_write_config_dword(dev, pos + PCI_ERR_COR_STATUS, dconfig);
+
 		/* Enable reporting of all correctable errors */
 		/* Correctable Error Mask - turned on bits disable errors */
 		pci_write_config_dword(dev, pos + PCI_ERR_COR_MASK, 0);
-		/* Advanced Error Capabilities */
-		pci_read_config_dword(dev, pos + PCI_ERR_CAP, &dconfig);
-		/* ECRC Generation Enable */
-		if (config & PCI_ERR_CAP_ECRC_GENC)
-			config |= PCI_ERR_CAP_ECRC_GENE;
-		/* ECRC Check Enable */
-		if (config & PCI_ERR_CAP_ECRC_CHKC)
-			config |= PCI_ERR_CAP_ECRC_CHKE;
-		pci_write_config_dword(dev, pos + PCI_ERR_CAP, dconfig);
-		/* PCI_ERR_HEADER_LOG - Header Log Register (16 bytes) */
-		/* Report all errors to the root complex */
-		pci_write_config_dword(dev, pos + PCI_ERR_ROOT_COMMAND,
-				       PCI_ERR_ROOT_CMD_COR_EN |
-				       PCI_ERR_ROOT_CMD_NONFATAL_EN |
-				       PCI_ERR_ROOT_CMD_FATAL_EN);
-		/* Clear the Root status register */
-		pci_read_config_dword(dev, pos + PCI_ERR_ROOT_STATUS, &dconfig);
-		pci_write_config_dword(dev, pos + PCI_ERR_ROOT_STATUS, dconfig);
 	}
 
 	return 0;
-- 
2.25.1


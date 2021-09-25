Tasks:

### Add some device config to AER logging

Some PCIe errors are caused by device misconfiguration, e.g., an
upstream bridge configured with MPS too large for an endpoint.  Maybe
logging some device configuration registers (Device Control, Link
Control, etc) as part of the AER logging would help diagnosis?
Screen reader support enabled.
    
### Initialize `info->id` in the DPC path

In the `dpc_process_error()` path, `info->id` isn't initialized before being
passed to `aer_print_error()`.  In the corresponding AER path, it is
initialized in `aer_isr_one_error().`

### Fix failure to clear AER status register

We have many reports of AER message spew, including this.  I think the
problem is that we don't clear the AER status register correctly as
outlined [here](https://bugzilla.kernel.org/show_bug.cgi?id=109691)

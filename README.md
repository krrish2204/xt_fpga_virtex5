# xt_fpga_virtex5

<b> Xtables Driver for DPI Hardware Accelerators on Xilinx Virtex5 ML507 Boards </b>

BACKGROUND

Xilinx Virtex5 ML507 BSP (Board Support Package) contains a PowerPC 440 CPU design. Whole board support package with PowerPC 440 CPU is eligible to run an Open Source Linux distribution maintained in https://github.com/Xilinx/linux-xlnx repository. Apart from standard CPU capabilities, one may use custom hardware accelerators running in parallel. That co-processing may decrease the time and power that CPU uses.

PURPOSE

The aim of project is to enable co-processing of CPU and DPI (deep packet inspection) hardware accelerators on Xilinx Virtex5 ML507 boards. For that aim, the project contains the source codes for both Xtables kernel module and iptables-linked userspace tool.

ABBREVIATIONS
 * Host: An FPGA board that is supposed to run Linux on it.
 * Remote Terminal: A remote computer accessing the Linux console on host via a serial connection. Remote terminal and builder machine can be the same.
 * Builder Machine: A computer cross-compiling Linux Kernel Image and synthesizing the board support package. It is also responsible for programming the board (host) using a JTAG cable. Remote terminal and builder machine can be the same.

HARDWARE/SYSTEM REQUIREMENTS
 * Host machine = Xilinx Virtex5 ML507 board
 * RS232 cable to connect Linux console on host
 * COM3 connectivity (USB - COM3 converters also work) on a remote terminal to be linked with RS232 cable.
 * JTAG cable to enable board programming by builder machine.
 * Ethernet cable to connect the host to local network.
 * A high-speed local network connection between terminal and host machines.
 * A CF card to store initial ramdisk for host machine
 
 
SOFTWARE REQUIREMENTS FOR BUILDER MACHINE
 * Cross-compiler toolchain to build a kernel for host processor (PowerPC 440)
 * ELDK toolchain to design, synthesize and program board support packages and hardware accelerators for host.
 * Linux Kernel source code maintained by Xilinx which can be fetched from https://github.com/Xilinx/linux-xlnx repository. 
    * Checking out <b>3.14.2</b> tag and using it may be critical for module-kernel compliance.


HARDWARE DESIGN REQUIREMENTS FOR ML507 BOARD
 * A DPI hardware accelerator for ML507 must be present and synthesizable.
    * Its family must be <b>"xlnx,dpi-accelerator-1.00.a"</b> to comply with the hardware.
 * A Board Support Package for Virtex5 ML507 must be generated by EDK toolchain
    * Also DPI hardware accelerator in it must be located inside.
    * It needs to include another DMA (Direct Memory Access) controller.
    * That DMA controller must be connected to DPI hardware accelerator to support fast network packet transfer.
    
 
SOFTWARE REQUIREMENTS FOR ML507 BOARD
  * A <b>3.14.2</b> kernel image built on builder machine by cross-compiler toolchain must be ready.
  * An initial ramdisk must be ready (or compiled by toolchain). It should contain following components:
    * A busybox that is compatible with kernel version (<b>3.14.2</b>)
    * userspace iptables with version <b>1.4.21</b>
    * iperf to analyze network performance


HOW TO USE THIS PROJECT?
  * After all requirements are completed, userspace tool and kernel module must be compiled.
    * Change INST_DIR, KERNEL_DIR and CC parameters in kernel module makefile according to system-specific paths.
    * Change INST_DIR, SYSROOT, CFLAGS and CC parameters in userspace tool makefile according to system-specific paths.
  * Deliver kernel module <b>xt_fpga.ko</b> and shared userspace tool <b>libxt_fpga.so</b> to host via FTP connection.
  * Put <b>libxt_fpga.so</b> inside <b>/lib/xtables/</b> folder.
  * Load <b>xt_fpga.ko</b> with insmod command. 
    * (Example) If module is in current directory, run:
      * insmod xt_fpga.ko
  
  
EXAMPLES:
  * You can enter a filter rule using the following iptables command:
     * iptables -I INPUT -m fpga --filter -j REJECT 
  * You can test if the filter is successful by ensuring that such ping packets. Blocked signatures must be rejected, others must not to succed.
     * ping \<ip_address\> -c 1 -p \<string_in_hex_format\>
  
  

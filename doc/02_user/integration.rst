.. _core-integration:

Core Integration
================

The main module is named ``cve2_top`` and can be found in ``cve2_top.sv``.
Note that the core logic is split-out from the register file and RAMs under ``cve2_top``.

Below, the instantiation template is given and the parameters and interfaces are described.

Instantiation Template
----------------------

.. code-block:: verilog

  cve2_top #(
      .MHPMCounterNum   ( 0                                ),
      .MHPMCounterWidth ( 40                               ),
      .RV32E            ( 0                                ),
      .RV32M            ( cve2_pkg::RV32MFast              ),
      .RndCnstLfsrSeed  ( cve2_pkg::RndCnstLfsrSeedDefault ),
      .RndCnstLfsrPerm  ( cve2_pkg::RndCnstLfsrPermDefault ),
      .DmHaltAddr       ( 32'h1A110800                     ),
      .DmExceptionAddr  ( 32'h1A110808                     )
  ) u_top (
      // Clock and reset
      .clk_i                  (),
      .rst_ni                 (),
      .test_en_i              (),
      .scan_rst_ni            (),
      .ram_cfg_i              (),

      // Configuration
      .hart_id_i              (),
      .boot_addr_i            (),

      // Instruction memory interface
      .instr_req_o            (),
      .instr_gnt_i            (),
      .instr_rvalid_i         (),
      .instr_addr_o           (),
      .instr_rdata_i          (),
      .instr_rdata_intg_i     (),
      .instr_err_i            (),

      // Data memory interface
      .data_req_o             (),
      .data_gnt_i             (),
      .data_rvalid_i          (),
      .data_we_o              (),
      .data_be_o              (),
      .data_addr_o            (),
      .data_wdata_o           (),
      .data_wdata_intg_o      (),
      .data_rdata_i           (),
      .data_rdata_intg_i      (),
      .data_err_i             (),

      // Interrupt inputs
      .irq_software_i         (),
      .irq_timer_i            (),
      .irq_external_i         (),
      .irq_fast_i             (),
      .irq_nm_i               (),

      // Debug interface
      .debug_req_i            (),
      .crash_dump_o           (),

      // Special control signals
      .fetch_enable_i         (),
      .alert_minor_o          (),
      .alert_major_internal_o (),
      .alert_major_bus_o      (),
      .core_sleep_o           ()
  );

Parameters
----------

+------------------------------+---------------------+------------+-----------------------------------------------------------------------+
| Name                         | Type/Range          | Default    | Description                                                           |
+==============================+=====================+============+=======================================================================+
| ``MHPMCounterNum``           | int (0..10)         | 0          | Number of performance monitor event counters                          |
+------------------------------+---------------------+------------+-----------------------------------------------------------------------+
| ``MHPMCounterWidth``         | int (64..1)         | 40         | Bit width of performance monitor event counters                       |
+------------------------------+---------------------+------------+-----------------------------------------------------------------------+
| ``RV32E``                    | bit                 | 0          | RV32E mode enable (16 integer registers only)                         |
+------------------------------+---------------------+------------+-----------------------------------------------------------------------+
| ``RV32M``                    | cve2_pkg::rv32m_e   | RV32MFast  | M(ultiply) extension select:                                          |
|                              |                     |            | "cve2_pkg::RV32MNone": No M-extension                                 |
|                              |                     |            | "cve2_pkg::RV32MSlow": Slow multi-cycle multiplier, iterative divider |
|                              |                     |            | "cve2_pkg::RV32MFast": 3-4 cycle multiplier, iterative divider        |
|                              |                     |            | "cve2_pkg::RV32MSingleCycle": 1-2 cycle multiplier, iterative divider |
+------------------------------+---------------------+------------+-----------------------------------------------------------------------+
| ``DmHaltAddr``               | int                 | 0x1A110800 | Address to jump to when entering Debug Mode                           |
+------------------------------+---------------------+------------+-----------------------------------------------------------------------+
| ``DmExceptionAddr``          | int                 | 0x1A110808 | Address to jump to when an exception occurs while in Debug Mode       |
+------------------------------+---------------------+------------+-----------------------------------------------------------------------+

Any parameter marked *EXPERIMENTAL* when enabled is not verified to the same standard as the rest of the Ibex core.

Note that Ibex uses SystemVerilog enum parameters e.g. for ``RV32M``.
This is well supported by most tools but some care is needed when overriding these parameters at the top level:

* Synopsys VCS does not support overriding enum and string parameters at the top level via command line.
  As a workaround, SystemVerilog defines are used in Ibex top level files simulated with VCS.
  These defines can be set via command line.

* Yosys does not support overriding enum parameters at the top level by setting enum names.
  Instead, the enum values need to be used.

Interfaces
----------

+----------------------------+-------------------------+-----+----------------------------------------+
| Signal(s)                  | Width                   | Dir | Description                            |
+============================+=========================+=====+========================================+
| ``clk_i``                  | 1                       | in  | Clock signal                           |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``rst_ni``                 | 1                       | in  | Active-low asynchronous reset          |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``test_en_i``              | 1                       | in  | Test input, enables clock and allows   |
|                            |                         |     | test control of reset.                 |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``scan_rst_ni``            | 1                       | in  | Test controlled reset.  If DFT not     |
|                            |                         |     | used, tie off to 1.                    |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``hart_id_i``              | 32                      | in  | Hart ID, usually static, can be read   |
|                            |                         |     | from :ref:`csr-mhartid` CSR            |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``boot_addr_i``            | 32                      | in  | First program counter after reset      |
|                            |                         |     | = ``boot_addr_i``,                     |
|                            |                         |     | see :ref:`exceptions-interrupts`       |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``instr_*``                | Instruction fetch interface, see :ref:`instruction-fetch`              |
+----------------------------+------------------------------------------------------------------------+
| ``data_*``                 | Load-store unit interface, see :ref:`load-store-unit`                  |
+----------------------------+------------------------------------------------------------------------+
| ``irq_*``                  | Interrupt inputs, see :ref:`exceptions-interrupts`                     |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``debug_*``                | Debug interface, see :ref:`debug-support`                              |
+----------------------------+------------------------------------------------------------------------+
| ``crash_dump_o``           | A set of signals that can be captured on reset to aid crash debugging. |
+----------------------------+------------------------------------------------------------------------+
| ``double_fault_seen_o``    | A double fault was observed, see :ref:`double-fault-detect`            |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``fetch_enable_i``         | 4                       | in  | Allow the core to fetch instructions.  |
|                            |                         |     | If this bit is set low, the core will  |
|                            |                         |     | pause fetching new instructions. A     |
|                            |                         |     | multi-bit encoding scheme is used. See |
|                            |                         |     | `FetchEnableOn` / `FetchEnableOff` in  |
|                            |                         |     | :file:`rtl/cve2_pkg.sv`                |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``core_sleep_o``           | 1                       | out | Core in WFI with no outstanding data   |
|                            |                         |     | or instruction accesses. Deasserts     |
|                            |                         |     | if an external event (interrupt or     |
|                            |                         |     | debug req) wakes the core up           |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``alert_minor_o``          | 1                       | out | Core has detected a fault which it can |
|                            |                         |     | safely recover from. Can be used by a  |
|                            |                         |     | system to log errors over time and     |
|                            |                         |     | detect tampering / attack. This signal |
|                            |                         |     | is a pulse, one cycle per alert.       |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``alert_major_internal_o`` | 1                       | out | Core has detected an internal fault    |
|                            |                         |     | which cannot be recovered from. Can be |
|                            |                         |     | used by a system to reset the core and |
|                            |                         |     | possibly  take other remedial action.  |
|                            |                         |     | This signal is a pulse, but might be   |
|                            |                         |     | set for multiple cycles per alert.     |
+----------------------------+-------------------------+-----+----------------------------------------+
| ``alert_major_bus_o``      | 1                       | out | Core has detected a bus fault          |
|                            |                         |     | which cannot be recovered from. Can be |
|                            |                         |     | used by a system to reset the core and |
|                            |                         |     | possibly  take other remedial action.  |
|                            |                         |     | This signal is a pulse, but might be   |
|                            |                         |     | set for multiple cycles per alert.     |
+----------------------------+-------------------------+-----+----------------------------------------+

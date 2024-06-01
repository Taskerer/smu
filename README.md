# ryzen_nb_smu
A fight against Ryzen NorthBridge SMU

Further usage and other information is still under research.

PR and discussions are welcomed.

# Background
Some how, I deterimined the method of commiunication between processor and SMU on northbridge of Ryzen processor, so it would be possible to change some power and frequency related settings in OS. Especially for Mobile Raven Ridge platform.

# Utils
## kmod_pp
Kernel Module to dump and modify pptable.

(Not working, under research)

TODO: Determine why we can't dump pptable
## smu_test.c
A userspace tool to make smu service request

# Communication Protcol
The registers, RAM and ROM of SMU is wrapped by a couple of register in NorthBridge PCI Config Space.

To communicate with SMU, the procossor need send a service request to SMU by write values to SMU registers.

There are 8 32-bit registers related with service register, MSG, 6×ARG and REP.

MSG is the ID of service request, 6*ARGS are arguments, REP is respond of service request.

## Progress of a service request
- Clear the REP register
- Write the ARGs
- Write the MSG
- Poll the REP until it changed, means the request is proceeded
- Read back ARGs
- Check the REP if the request proceeded sucessfuly

## A table of request ID for MP1
| Name | ID | Note |
| :------| :------ | :------ |
| TestMessage | 0x1 |  |
| GetSmuVersion | 0x2 |  |
| GetBiosIfVersion | 0x3 |  |
| GetNameString | 0x4 |  |
| EnableSmuFeatures | 0x5 |  |
| DisableSmuFeatures | 0x6 |  |
| DramLogSetDramAddrHigh | 0x7 |  |
| DramLogSetDramAddrLow | 0x8 |  |
| DramLogSetDramSize | 0x9 |  |
| DxioTestMessage | 0xA |  |
| ReadCoreCacWeightRegister | 0xB |  |
| SleepEntry | 0xC |  |
| SetGbeStatus | 0xD |  |
| PowerUpGfx | 0xE |  |
| PowerUpSata | 0xF |  |
| PowerDownSata | 0x10 |  |
| DisableSataController | 0x11 |  |
| SetBiosDramAddrHigh | 0x12 |  |
| SetBiosDramAddrLow | 0x13 |  |
| SetToolsDramAddrHigh | 0x14 |  |
| SetToolsDramAddrLow | 0x15 |  |
| TransferTableSmu2Dram | 0x16 |  |
| TransferTableDram2Smu | 0x17 |  |
| PowerSourceAC | 0x18 |  |
| PowerSourceDC | 0x19 |  |
| SetSustainedPowerLimit | 0x1A |  |
| SetFastPPTLimit | 0x1B |  |
| SetSlowPPTLimit | 0x1C |  |
| SetSlowPPTTimeConstant | 0x1D |  |
| SetStapmTimeConstant | 0x1E |  |
| SetTctlMax | 0x1F |  |
| SetVrmCurrentLimit | 0x20 |  |
| SetVrmSocCurrentLimit | 0x21 |  |
| SetVrmMaximumCurrentLimit | 0x22 |  |
| SetVrmSocMaximumCurrentLimit | 0x23 |  |
| SetPSI0CurrentLimit | 0x24 |  |
| SetPSI0SocCurrentLimit | 0x25 |  |
| SetProchotDeassertionRampTime | 0x26 |  |
| UpdateSkinTempError | 0x27 |  |
| SetGpuApertureLow | 0x28 |  |
| SetGpuApertureHigh | 0x29 |  |
| StartGpuLink | 0x2A |  |
| StopGpuLink | 0x2B |  |
| UsbD3Entry | 0x2C |  |
| UsbD3Exit | 0x2D |  |
| UsbInit | 0x2E |  |
| AcBtcStartCal | 0x2F |  |
| AcBtcStopCal | 0x30 |  |
| AcBtcEndCal | 0x31 |  |
| DcBtc | 0x32 |  |
| BtcRestoreOnS3Resume | 0x33 |  |
| SetGpuDeviceId | 0x34 |  |
| SetUlvVidOffset | 0x35 |  |
| DisablePSI | 0x36 |  |
| EnablePostCode | 0x37 |  |
| UsbConfigUpdate | 0x38 |  |
| SetupUSB31ControllerTrap | 0x39 |  |
| SetVddOffVid | 0x3A |  |
| SetVminFrequency | 0x3B |  |
| SetFrequencyMax | 0x3C |  |
| SetGfxclkOverdriveByFreqVid | 0x3D |  |
| PowerGateXgbe | 0x3E |  |
| OC_Disable | 0x3F |  |
| OC_VoltageMax | 0x40 |  |
| OC_FrequencyMax | 0x41 |  |
| EnableCC6Filter | 0x42 |  |
| GetSustainedPowerAndThmLimit | 0x43 |  |
| SetSoftMaxCCLK | 0x44 |  |
| SetSoftMinCCLK | 0x45 |  |
| SetSoftMaxGfxClk | 0x46 |  |
| SetSoftMinGfxClk | 0x47 |  |
| SetSoftMaxSocclkByFreq | 0x48 |  |
| SetSoftMinSocclkByFreq | 0x49 |  |
| SetSoftMaxFclkByFreq | 0x4A |  |
| SetSoftMinFclkByFreq | 0x4B |  |
| SetSoftMaxVcn | 0x4C |  |
| SetSoftMinVcn | 0x4D |  |
| SetSoftMaxLclk | 0x4E |  |
| SetSoftMinLclk | 0x4F |  |
| Message_Count | 0x50 |  |

## About "Get" Commands
You can get SMU answer from reading arguments after command. Then decode them or it will be clear result

# A table of REP code
| Name | ID |
| :------| :------ |
| OK | 0x1 |
| Failed | 0xFF |
| UnknownCmd | 0xFE |
| CmdRejectedPrereq | 0xFD |
| CmdRejectedBusy | 0xFC |

# A table of SMU Feature ID
| Name             | Bit = args    | Note                                                                                                 |
| ---------------- | ------------- | ---------------------------------------------------------------------------------------------------- |
| CCLK_CONTROLLER  | 0 = 1         | CPU Clock controller - CPU will always work at 1600MHz                                               |
| FAN_CONTROLLER   | 1 = 2         | Fan will be always 4400 RPM or more                                                                  |
| DATA_CALCULATION | 2 = 4         | Disable or Enable  CPU power states. No more changing powers (besides STAPM) and voltages are static |
| PPT              | 3 = 8         | Disable or Enable Slow and Fast Power Limits                                                         |
| TDC              | 4 = 10        | Disable or Enable TDC Current limits                                                                 |
| THERMAL          | 5 = 20        | CPU, SoC and iGPU temperature controller disabling                                                   |
| FIT              | 6 = 40        | Monitors reliability and failure predictions                                                         |
| EDC              | 7 = 80        | Nothing happens                                                                                      |
| PLL_POWER_DOWN   | 8 = 100       | Power down state of phase-locked loops to save power                                                 |
| ULV              | 9 = 200       | ULV voltage for power saving mode enablement                                                         |
| VDDOFF           | 10 = 400      | Voltage rail off DEBUG PStates voltage control                                                       |
| VCN_DPM          | 11 = 800      | Locking max VCN clock to 400 MHz while disabled                                                      |
| ACP_DPM          | 12 = 1000     | Locking NB-Azalia (audio over iGPU) to 400 MHz wh disabl                                             |
| ISP_DPM          | 13 = 2000     | Locking Image Signal Processor to 100 MHz if disabled                                                |
| FCLK_DPM         | 14 = 4000     | FCLK clock locking? After disabling                                                                  |
| SOCCLK_DPM       | 15 = 8000     | Locking max SoC clock to 200 MHz while disabled                                                      |
| MP0CLK_DPM       | 16 = 10000    | MP0 clocks group clock locking? After disabling                                                      |
| LCLK_DPM         | 17 = 20000    | Data Latch clock locking? After disabling                                                            |
| SHUBCLK_DPM      | 18 = 40000    | System hub clock locking? After disabling                                                            |
| DCEFCLK_DPM      | 19 = 80000    | Nothing happens                                                                                      |
| GFX_DPM          | 20 = 100000   | Locking max iGPU clock to 400 MHz while disabled                                                     |
| DS_GFXCLK        | 21 = 200000   | Allow change GFX_CLK from SMU                                                                        |
| DS_SOCCLK        | 22 = 400000   | Allow change SOC_CLK from SMU                                                                        |
| DS_LCLK          | 23 = 800000   | Allow change LCLK from SMU                                                                           |
| DS_DCEFCLK       | 24 = 1000000  | Allow change DCEFCLK from SMU                                                                        |
| DS_SHUBCLK       | 25 = 2000000  | Allow change SystemHUB_CLK from SMU                                                                  |
| RM               | 26 = 4000000  | Resource management engine?                                                                          |
| S0i2             | 27 = 8000000  | Pstate S0i2 for low power idle                                                                       |
| WHISPER_MODE     | 28 = 10000000 | Whisper event start-stop                                                                             |
| DS_FCLK          | 29 = 20000000 | Allow change FCLK from SMU                                                                           |
| DS_SMNCLK        | 30 = 40000000 | Allow change SystemManagementNetworkCLK from SMU                                                     |
| DS_MP1CLK        | 31 = 80000000 | Allow change MP1 Clocks group from SMU                                                               |
| DS_MP0CLK        | 32 = 0,1      | Allow change MP0 Clocks group from SMU                                                               |
| MGCG             | 33 = 0,2      | MemoryGenericClockGating?                                                                            |
| DS_FUSE_SRAM     | 34 = 0,4      | Deep sleep strate for fuse SRAM from SMU                                                             |
| GFX_CKS          | 35 = 0,8      | DISABLING WILL DISABLE THE GPU!                                                                      |
| PSI0             | 36 = 0,10     | PSI0 Currents group                                                                                  |
| PROCHOT          | 37 = 0,20     | Disabling will disable PROCHOT and CPU CAN WORK ON 400 MHZ!                                          |
| CPUOFF           | 38 = 0,40     | Disabling will disable C-States                                                                      |
| STAPM            | 39 = 0,80     | Disabling will disable STAPM                                                                         |
| CORE_CSTATES     | 40 = 0,100    | Disabling will disable C-States                                                                      |
| GFX_DUTY_CYCLE   | 41 = 0,200    | Duty Cycle for iGPU control                                                                          |
| AA_MODE          | 42 = 0,400    | Enabling will give some performance boost                                                            |
| LIVMIN           | 43 = 0,800    | Minimum live voltage for RV2                                                                         |
| RLC_PACE         | 44 = 0,1000   | RLC (Run Length Coding) pacing for RV2                                                               |

-These can be enabled with EnableSmuFeatures (0x5) or disabled with DisableSmuFeatures (0x6)   

## Example
To enable feature set command to 0x5 and in arguments send value from table args (after "="). For examle to enable STAPM feature you need to send MP1 command 0x5 with arguments 0,80 (send 0 to zero argument and 80 to first, "," is a separator beetwen arguments

# PPtable
PowerPlay Table

A table to storage power management information

See AMDGPU Powerplay for more information

Now no way to get RX Vega Mobile Power Play from system

# Firmware
Grab from BIOS, reverse engineering in progesss.(Help wanted)

# Credit
- [zamaudio/smutool](https://github.com/zamaudio/smutool)
- Linux Kernel - amdgpu
- Internet
- My ThinkPad E485 with R7-2700U

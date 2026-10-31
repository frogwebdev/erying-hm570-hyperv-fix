# Erying HM570: fixing boot loops when starting Hyper-V

[Русский](README.md) | English

Tested on Erying HM570 with a Tiger Lake-H engineering sample, CPUID `806D0`, BIOS `THM570106` dated June 14, 2022, and Windows 11 Pro 25H2 `26200.9711`.

**Symptom:** Windows boots with VMX enabled, but enters a reboot loop when the Hyper-V hypervisor starts.

## If Windows boots

Open **Command Prompt as administrator** and run:

```cmd
bcdedit /set {current} hypervisorloadoptions DISABLEHARDWAREMBEC
```

If hypervisor startup was previously disabled, enable it again:

```cmd
bcdedit /set {current} hypervisorlaunchtype auto
```

Restart the computer. **Keep VMX and VT-d enabled in BIOS.**

## If Windows is stuck in a reboot loop

Open **Advanced options → Troubleshoot → Advanced options → Command Prompt**.

List the boot entries:

```cmd
bcdedit /enum osloader
```

Find the installed Windows entry using its `description` field. Do not select `Windows Recovery Environment`.

Replace `{GUID}` with that entry's identifier:

```cmd
bcdedit /set {GUID} hypervisorloadoptions DISABLEHARDWAREMBEC
```

Restart the computer and select that Windows entry.

## When installing Windows again

The setting must be reapplied **to the boot entry of Windows installed on the SSD before its first boot with the hypervisor**.

To automate this, the installation script must apply the setting after creating the boot configuration and before restarting. Changing only the USB installer's boot configuration does not configure Windows on the SSD. Automated installation was not tested as part of this verification.

## Verification

In PowerShell:

```powershell
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
```

The expected result is `True`.

Ubuntu startup in WSL2 was also verified:

```powershell
wsl -l -v
```

The distribution was `Running` with version `2`.

After applying the setting, Hyper-V and Ubuntu in WSL2 were confirmed working with VMX, VT-d, and Kernel DMA Protection enabled. No BIOS flashing or Windows reinstallation was required.

`DISABLEHARDWAREMBEC` disables the hypervisor's use of hardware MBEC.

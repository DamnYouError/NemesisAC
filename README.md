# Nemesis – Kernel-Based Anti-Tamper & Integrity Framework

Nemesis is a kernel-assisted anti-tamper and integrity verification framework designed to detect advanced usermode and kernelmode manipulation techniques. It focuses on stack integrity, execution validation, hardware trust signals, and hypervisor detection, while providing detailed diagnostics through a multi-level kernel logging system.

> ⚠️ Educational / Research Use Only  
> This project is not intended to bypass or interfere with commercial anti-cheat systems. Testing against games must be done with their anti-cheat disabled or in insecure mode.

---

## Features

### Thread & Execution Integrity
- Attached thread detection
- APC & DPC stack walking
- NMI stack walking via ISR `iretq`
- Return address exception hooking detection
- Removal of thread CID table entry detection

### Memory & Image Integrity
- Process module `.text` section integrity checks
- System module `.text` integrity checks
- Various driver and module image integrity checks
- Chained `.data` pointer detection *(experimental / iffy)*

### Kernel & Driver Validation
- Driver dispatch routine validation
- HalDispatch and HalPrivateDispatch routine validation
- Win32kBase DxgInterface routine validation
- System module device object verification

### Handle & Object Monitoring
- Handle stripping detection via object callbacks
- Process handle table enumeration

### Virtualization & Hook Detection
- Hypervisor detection
- EPT hook detection

### Hardware & Platform Security
- Extraction of various hardware identifiers
- Malicious PCI device detection via configuration space scanning

### Security Hardening
- Dynamic import resolving and encryption

---

## Architecture

> 🚧 TODO  
A detailed architecture overview will be added, covering kernel ↔ usermode communication, request dispatching, validation pipelines, and diagnostics.

---

## Planned Features

There is a long list of additional features planned. Whether they are implemented depends on time and feasibility.

Pull requests are accepted, but **must meet strict quality standards**:
- Clean, maintainable kernel-safe code
- Thorough testing on virtual machines and bare metal
- Verification with Driver Verifier enabled

Low-effort or unsafe kernel code will not be accepted.

---

## Example

An example recording of Nemesis running with **CS2** is available.

**Notes**
- VAC was disabled during testing
- If testing with a Steam game, launch it in **insecure mode**

The video demonstrates:
- Kernel VERBOSE-level logs in DebugView
- Usermode application console output
- Additional performance benchmarking metrics
---

## Known Issues

See the **Issues** tab for currently known problems.

If you encounter bugs:
- Open a new issue
- Provide logs (INFO or VERBOSE preferred)
- Include OS version and hardware details

---

## Tested Windows Versions

- Windows 10 22H2
- Windows 11 22H2

---

## Build Instructions

### Requirements
- Visual Studio
- Windows Driver Kit (WDK)

---

## Enable Test Signing Mode

The driver is not signed, so test signing must be enabled.

1. Open **Command Prompt as Administrator**
2. Run:
   ```
   bcdedit -set TESTSIGNING on
   bcdedit /debug on
   ```
3. Restart Windows

---

## Building the Project

1. Clone the repository:
   ```
   git clone https://github.com/DamnYouError/Nemesis-AC.git
   ```
2. Open the project in Visual Studio
3. Select the appropriate configuration:
   - `Release - No Server - Win10`
   - `Release - No Server - Win11`
4. Build the solution

### If You Encounter Build Issues

Ensure the driver project settings are:

- **Inf2Cat → General**
  - Use Local Time: Yes
- **C/C++**
  - Treat Warnings As Errors: No
  - Spectre Mitigation: Disabled

---

## Installing and Running Nemesis

1. Copy the built driver:
   ```
   ac\x64\Release - No Server\driver.sys
   ```
   to:
   ```
   C:\Windows\System32\Drivers\
   ```

2. (Optional) Rename the driver

3. Open **OSR Driver Loader**
   - Select the driver
   - Do NOT start the service
   - Set **Service Start** to **System** (required)
   - Click **Register Service**
   - Do NOT click **Start Service**

4. Restart Windows

5. Launch the target application (e.g. Notepad, CS2)
   - If using a game, ensure its anti-cheat is disabled

6. Inject the usermode DLL:
   ```
   ac\x64\Release - No Server\user.dll
   ```
   Use an injector of your choice (e.g. Process Hacker)

> The server component is not required.

---

## Kernel Debug Logging

Nemesis supports four logging levels:

```c
#define LOG_ERROR_LEVEL
#define LOG_WARNING_LEVEL
#define LOG_INFO_LEVEL
#define LOG_VERBOSE_LEVEL
```

### Logging Masks

| Level    | Mask |
|---------|------|
| ERROR   | 0x3  |
| WARNING | 0x7  |
| INFO    | 0xF  |
| VERBOSE | 0x1F |

Higher levels include all lower levels.

---

## Configuring Debug Print Filter

### Creating the Registry Key

1. Open **Registry Editor**
2. Navigate to:
   ```
   HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager
   ```
3. Right-click **Session Manager** → New → Key  
   Name it:
   ```
   Debug Print Filter
   ```
4. Inside it, create a **DWORD (32-bit)** value named:
   ```
   DEFAULT
   ```

---

### Setting the Mask

1. Double-click `DEFAULT`
2. Enter the desired value (see table above)
3. Click OK
4. Restart Windows

---

## Filtering Debug Output

### WinDbg

With WinDbg attached:
```
.ofilter Nemesis*
```

### DebugView

1. Click **Edit → Filter/Highlight**
2. Set **Include String** to:
   ```
   Nemesis*
   ```

---

## Final Notes

Nemesis is a research-driven kernel project intended for learning, experimentation, and defensive security exploration. Expect breaking changes, aggressive validation logic, and an evolving architecture.

Contributions are welcome — **quality over quantity**.

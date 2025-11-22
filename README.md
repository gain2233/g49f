# 🔓 ZIP Cracker - Advanced Password Recovery Tool

<div align="center">

![Python Version](https://img.shields.io/badge/python-3.7+-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey.svg)

**A powerful, multi-threaded ZIP password cracking tool with support for dictionary attacks, mask attacks, CRC32 collision attacks, and AES encryption.**

[Features](#-features) • [Installation](#-installation) • [Usage](#-usage) • [Examples](#-examples) • [Advanced](#-advanced-usage) • [FAQ](#-faq)

</div>

---

## 📋 Table of Contents

- [Features](#-features)
- [How It Works](#-how-it-works)
- [Installation](#-installation)
- [Quick Start](#-quick-start)
- [Usage Guide](#-usage-guide)
  - [Dictionary Attack](#1-dictionary-attack)
  - [Mask Attack](#2-mask-attack)
  - [CRC32 Collision Attack](#3-crc32-collision-attack)
- [Advanced Usage](#-advanced-usage)
- [Performance Optimization](#-performance-optimization)
- [Technical Details](#-technical-details)
- [Troubleshooting](#-troubleshooting)
- [FAQ](#-faq)
- [Contributing](#-contributing)
- [Legal Disclaimer](#-legal-disclaimer)
- [License](#-license)

---

## ✨ Features

### 🎯 Core Features

- **🔐 Multiple Encryption Support**
  - Traditional ZipCrypto encryption
  - AES-128/192/256 encryption (with pyzipper)
  - Pseudo-encryption detection and automatic fix

- **💪 Multiple Attack Modes**
  - Dictionary attack (single file or directory)
  - Mask attack (custom pattern-based)
  - CRC32 collision attack (for small files)
  - Built-in numeric dictionary (1-6 digits)

- **⚡ High Performance**
  - Multi-threaded architecture
  - Dynamic thread pool optimization
  - Memory-efficient chunk processing
  - Automatic CPU core detection

- **📊 Real-Time Monitoring**
  - Live progress tracking
  - Speed calculation (passwords/second)
  - Estimated time remaining
  - Current password display

- **🛡️ Robust Error Handling**
  - Thread-safe operations
  - Graceful error recovery
  - Detailed error messages
  - Corrupted file detection

---

## 🔍 How It Works

### Attack Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                    ZIP File Analysis                         │
│  • Check encryption status                                   │
│  • Detect pseudo-encryption                                  │
│  • Identify encryption type (ZipCrypto/AES)                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Pseudo-Encryption Check                         │
│  • Attempt automatic fix                                     │
│  • If successful → Extract files (No password needed!)       │
│  • If failed → Proceed to password cracking                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                CRC32 Collision Attack                        │
│  • Analyze small files (≤6 bytes)                           │
│  • Attempt content recovery via CRC32                        │
│  • Optional user confirmation                                │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Password Cracking Phase                         │
│                                                              │
│  [Dictionary Attack]    [Mask Attack]    [Numeric Attack]   │
│         │                    │                  │            │
│         ▼                    ▼                  ▼            │
│   User dictionary      Pattern-based      1-6 digits        │
│   Built-in dict        brute-force        auto-generated    │
│   Directory scan       Custom charset                       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│          Multi-threaded Verification                         │
│  • Thread pool executor                                      │
│  • Parallel password testing                                 │
│  • Automatic resource management                             │
│  • First-success termination                                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│               Success & Extraction                           │
│  • Password verification                                     │
│  • Automatic file extraction                                 │
│  • Output directory creation                                 │
│  • Success report                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Installation

### Prerequisites

- Python 3.7 or higher
- pip package manager

### Basic Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/zip-cracker.git
cd zip-cracker

# Basic usage (ZipCrypto support only)
python zip_cracker.py
```

### Full Installation (with AES support)

```bash
# Install pyzipper for AES encryption support
pip install pyzipper

# Or install all dependencies
pip install -r requirements.txt
```

### Create requirements.txt

```txt
pyzipper>=0.3.6
```

### Verify Installation

```bash
python zip_cracker.py --help
```

---

## 🎯 Quick Start

### 1. Simple Password Recovery

```bash
# Use default dictionary and numeric attack
python zip_cracker.py protected.zip
```

### 2. Custom Dictionary

```bash
# Use your own password list
python zip_cracker.py protected.zip passwords.txt
```

### 3. Mask Attack

```bash
# Try all 4-digit PIN codes
python zip_cracker.py protected.zip -m "?d?d?d?d"
```

### 4. Custom Output Directory

```bash
# Extract to specific folder
python zip_cracker.py protected.zip -o my_files
```

---

## 📚 Usage Guide

### 1. Dictionary Attack

Dictionary attack tries passwords from a text file or directory of files.

#### **Basic Syntax**

```bash
python zip_cracker.py <zip_file> [dictionary]
```

#### **Examples**

**Using built-in dictionary:**
```bash
python zip_cracker.py secret.zip
# Tries: password_list.txt → 1-6 digit numbers
```

**Using custom dictionary:**
```bash
python zip_cracker.py secret.zip rockyou.txt
```

**Using directory of dictionaries:**
```bash
python zip_cracker.py secret.zip ./dictionaries/
# Processes all files in directory alphabetically
```

#### **Creating Your Dictionary**

Create a text file with one password per line:

```txt
password123
admin
qwerty
12345678
letmein
```

#### **Recommended Dictionaries**

- **RockYou** - 14 million passwords
- **SecLists** - Curated security lists
- **CrackStation** - 1.5 billion passwords

---

### 2. Mask Attack

Mask attack uses patterns to generate passwords, perfect for when you know the password format.

#### **Mask Syntax**

| Placeholder | Character Set | Example |
|-------------|---------------|---------|
| `?d` | Digits (0-9) | 0123456789 |
| `?l` | Lowercase (a-z) | abcdefghijklmnopqrstuvwxyz |
| `?u` | Uppercase (A-Z) | ABCDEFGHIJKLMNOPQRSTUVWXYZ |
| `?s` | Symbols | !@#$%^&*()... |
| `??` | Literal `?` | ? |
| Any other | Literal character | abc123 |

#### **Basic Syntax**

```bash
python zip_cracker.py <zip_file> -m "mask_pattern"
```

#### **Examples**

**4-digit PIN code:**
```bash
python zip_cracker.py bank.zip -m "?d?d?d?d"
# Tries: 0000, 0001, 0002, ..., 9999
# Total: 10,000 combinations
```

**Password format: "pass" + 4 digits:**
```bash
python zip_cracker.py data.zip -m "pass?d?d?d?d"
# Tries: pass0000, pass0001, ..., pass9999
```

**Mixed case + digits:**
```bash
python zip_cracker.py file.zip -m "?u?l?l?l?d?d"
# Tries: Aaaa00, Aaaa01, ..., Zzzz99
# Total: 26 × 26 × 26 × 26 × 10 × 10 = 45,697,600 combinations
```

**Complex pattern:**
```bash
python zip_cracker.py secure.zip -m "Admin?d?d?d?d!"
# Tries: Admin0000!, Admin0001!, ..., Admin9999!
```

**Literal question mark:**
```bash
python zip_cracker.py test.zip -m "what???"
# Tries: what???
```

#### **Combination Calculator**

| Mask | Combinations | Est. Time (10k/s) |
|------|--------------|-------------------|
| `?d?d?d?d` | 10,000 | 1 second |
| `?d?d?d?d?d?d` | 1,000,000 | 1.7 minutes |
| `?l?l?l?l` | 456,976 | 46 seconds |
| `?u?l?l?l?d?d` | 45,697,600 | 1.3 hours |
| `?l?l?l?l?l?l?l?l` | 208,827,064,576 | 7.6 months |

---

### 3. CRC32 Collision Attack

For files ≤6 bytes, the tool can recover content using CRC32 checksums without the password!

#### **How It Works**

1. Detects small files in the ZIP
2. Extracts their CRC32 checksum
3. Brute-forces all possible contents
4. Matches CRC32 to recover original data

#### **Example**

```bash
python zip_cracker.py data.zip

# Output:
# [!] File "pin.txt" in "data.zip" is 4 bytes. Attempt CRC32 collision attack? (y/n) y
# [+] CRC32 value for pin.txt is: 123456789
# [+] Starting CRC32 collision attack...
# [*] Success! Content of pin.txt: 1234
```

#### **Limitations**

- Only works for files ≤6 bytes
- Uses printable ASCII characters
- Time increases exponentially with size

---

## 🔧 Advanced Usage

### Combining Options

```bash
# Custom dictionary + custom output
python zip_cracker.py file.zip passwords.txt -o extracted_files

# Mask attack + custom output
python zip_cracker.py file.zip -m "?d?d?d?d" -o pins
```

### Processing Multiple Files

Create a batch script:

```bash
#!/bin/bash
for zip in *.zip; do
    echo "Cracking $zip..."
    python zip_cracker.py "$zip" rockyou.txt
done
```

### Windows Batch Script

```batch
@echo off
for %%f in (*.zip) do (
    echo Cracking %%f...
    python zip_cracker.py "%%f" passwords.txt
)
```

---

## ⚡ Performance Optimization

### Thread Count

The tool automatically adjusts threads based on CPU cores:

```python
Threads = min(128, CPU_cores × 4)
```

**Example:**
- 4-core CPU → 16 threads
- 8-core CPU → 32 threads
- 32-core CPU → 128 threads (max)

### Memory Optimization

Dictionary files are loaded in chunks (1,000,000 passwords) to minimize memory usage:

```python
# Good for large dictionaries (>100MB)
python zip_cracker.py file.zip huge_dictionary.txt
```

### Speed Tips

1. **Use SSD storage** - Faster I/O operations
2. **Close background apps** - More CPU resources
3. **Use mask attack** - More efficient than large dictionaries
4. **Try CRC attack first** - Instant for small files
5. **Start with common passwords** - Order matters

### Expected Performance

| Hardware | Speed (passwords/sec) |
|----------|----------------------|
| 4-core CPU @ 2.5GHz | 5,000 - 10,000 |
| 8-core CPU @ 3.5GHz | 15,000 - 25,000 |
| 16-core CPU @ 4.0GHz | 30,000 - 50,000 |

*Note: Actual speed depends on encryption type and password complexity*

---

## 🔬 Technical Details

### Architecture

```
┌─────────────────────────────────────────────┐
│            Main Application                 │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │    ZipEncryptionChecker             │   │
│  │  • Encryption detection             │   │
│  │  • Pseudo-encryption fix            │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │        CRCCracker                   │   │
│  │  • Small file analysis              │   │
│  │  • CRC32 collision attack           │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │      AttackEngine                   │   │
│  │  ┌──────────────────────────────┐   │   │
│  │  │   PasswordCracker            │   │   │
│  │  │  • Password verification     │   │   │
│  │  │  • File extraction           │   │   │
│  │  └──────────────────────────────┘   │   │
│  │  ┌──────────────────────────────┐   │   │
│  │  │   AttackStatus               │   │   │
│  │  │  • Thread-safe state         │   │   │
│  │  │  • Progress tracking         │   │   │
│  │  └──────────────────────────────┘   │   │
│  │  ┌──────────────────────────────┐   │   │
│  │  │   ProgressDisplay            │   │   │
│  │  │  • Real-time updates         │   │   │
│  │  │  • Speed calculation         │   │   │
│  │  └──────────────────────────────┘   │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │    DictionaryGenerator              │   │
│  │  • File loading (chunked)           │   │
│  │  • Numeric generation               │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │       MaskParser                    │   │
│  │  • Pattern parsing                  │   │
│  │  • Charset expansion                │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### Encryption Support

| Encryption Type | Without pyzipper | With pyzipper |
|----------------|------------------|---------------|
| ZipCrypto (Legacy) | ✅ Supported | ✅ Supported |
| AES-128 | ❌ Not supported | ✅ Supported |
| AES-192 | ❌ Not supported | ✅ Supported |
| AES-256 | ❌ Not supported | ✅ Supported |
| Pseudo-encryption | ✅ Auto-fix | ✅ Auto-fix |

### Thread Safety

All shared state is protected by locks:

```python
class AttackStatus:
    def __init__(self):
        self.lock = threading.Lock()
    
    def add_tried_password(self, password: str):
        with self.lock:
            self.tried_passwords.append(password)
```

---

## 🐛 Troubleshooting

### Common Issues

#### "pyzipper not found" Warning

**Problem:** AES-encrypted ZIPs can't be cracked

**Solution:**
```bash
pip install pyzipper
```

#### "Permission denied" Error

**Problem:** Can't create output directory

**Solution:**
```bash
# Use a different output directory
python zip_cracker.py file.zip -o ~/Desktop/extracted

# Or run with appropriate permissions
sudo python zip_cracker.py file.zip
```

#### Very Slow Performance

**Problem:** Speed is under 1,000 passwords/second

**Possible Causes:**
1. AES encryption (slower than ZipCrypto)
2. Very complex ZIP structure
3. Slow hard drive (use SSD)
4. Too many background processes

**Solutions:**
```bash
# Close unnecessary programs
# Move ZIP to SSD
# Use mask attack instead of huge dictionary
```

#### "BadZipFile" Error

**Problem:** Corrupted or invalid ZIP file

**Solution:**
```bash
# Test ZIP integrity first
unzip -t file.zip

# Try to repair
zip -FF file.zip --out fixed.zip
python zip_cracker.py fixed.zip
```

#### Memory Error with Large Dictionaries

**Problem:** Out of memory with huge password lists

**Solution:**
The tool automatically chunks large files, but if you still have issues:
```bash
# Split large dictionary
split -l 1000000 huge_dict.txt smaller_dict_

# Process each part
python zip_cracker.py file.zip smaller_dict_aa
python zip_cracker.py file.zip smaller_dict_ab
```

---

## ❓ FAQ

### General Questions

**Q: Is this tool legal to use?**
A: Yes, for your own files or with explicit permission. See [Legal Disclaimer](#-legal-disclaimer).

**Q: How long will it take to crack my ZIP?**
A: Depends on password complexity and length. A 4-digit PIN takes seconds, an 8-character random password could take years.

**Q: Can this crack any ZIP password?**
A: Technically yes, but practically no. Strong passwords (12+ random characters) are computationally infeasible to crack.

**Q: Does it work on RAR, 7z, or other formats?**
A: No, this tool is specifically designed for ZIP files only.

### Technical Questions

**Q: Why use pyzipper instead of standard zipfile?**
A: Python's built-in `zipfile` doesn't support AES encryption, which is commonly used in modern ZIP files.

**Q: How many threads should I use?**
A: The tool automatically optimizes this. Default is `CPU_cores × 4`, capped at 128.

**Q: Can I pause and resume?**
A: Not currently. The tool must complete the attack or be stopped.

**Q: Does it support GPU acceleration?**
A: No. ZIP encryption is designed to be sequential and doesn't benefit much from GPU acceleration.

### Attack Strategy

**Q: Which attack mode should I use?**
A: 
- **Dictionary** - If you have password lists or know common patterns
- **Mask** - If you know the password format (e.g., "pass" + 4 digits)
- **CRC32** - Automatic for small files
- **Numeric** - Good starting point if password might be a simple number

**Q: What's the most efficient approach?**
A:
1. Try CRC32 attack (automatic for small files)
2. Use built-in dictionary + numeric (default)
3. Try mask attack if you know format
4. Use large dictionaries as last resort

**Q: How do I create a good mask?**
A: Think about the password pattern:
- Birth year? `19?d?d` or `20?d?d`
- Name + numbers? `John?d?d?d`
- Common format? `?u?l?l?l?d?d?d?d` (Capital + 3 lowercase + 4 digits)

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

### Reporting Bugs

1. Check if the issue already exists
2. Provide detailed information:
   - Python version
   - Operating system
   - Full error message
   - Steps to reproduce

### Suggesting Features

Open an issue with:
- Clear description of the feature
- Use cases
- Expected behavior

### Code Contributions

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Setup

```bash
# Clone your fork
git clone https://github.com/yourusername/zip-cracker.git
cd zip-cracker

# Install development dependencies
pip install -r requirements-dev.txt

# Run tests
python -m pytest tests/

# Check code style
flake8 zip_cracker.py
```

---

## ⚖️ Legal Disclaimer

### Important Notice

This tool is provided for **educational and legitimate purposes only**.

### ✅ Acceptable Use

- Recovering passwords for your own files
- Authorized security testing
- Educational research
- Forensic investigations with proper authorization

### ❌ Prohibited Use

- Unauthorized access to others' files
- Any illegal activities
- Circumventing security without permission
- Violating terms of service

### Responsibility

**Users are solely responsible for their actions.** The authors and contributors:
- Assume no liability for misuse
- Do not endorse illegal activities
- Provide this tool "as is" without warranties

### Legal Consequences

Unauthorized password cracking may violate:
- Computer Fraud and Abuse Act (USA)
- Computer Misuse Act (UK)
- Similar laws in other jurisdictions

**Always obtain proper authorization before use.**

---

## 📄 License

This project is licensed under the MIT License - see below for details:

```
MIT License

Copyright (c) 2025 Asaotomo (Hx0 Team)

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

## 🙏 Acknowledgments

- **Hx0 Team** - Original concept and development
- **pyzipper** - AES encryption support
- **Python Community** - Excellent libraries and tools
- **Security Researchers** - Cryptographic knowledge sharing

---

## 📞 Contact & Support

### Getting Help

- **Issues:** [GitHub Issues](https://github.com/yourusername/zip-cracker/issues)
- **Discussions:** [GitHub Discussions](https://github.com/yourusername/zip-cracker/discussions)
- **Email:** your.email@example.com

### Stay Updated

- ⭐ Star this repository to show support
- 👁️ Watch for updates and new features
- 🔀 Fork to create your own version

---

## 📊 Project Stats

![GitHub Stars](https://img.shields.io/github/stars/yourusername/zip-cracker?style=social)
![GitHub Forks](https://img.shields.io/github/forks/yourusername/zip-cracker?style=social)
![GitHub Issues](https://img.shields.io/github/issues/yourusername/zip-cracker)
![GitHub Pull Requests](https://img.shields.io/github/issues-pr/yourusername/zip-cracker)

---

<div align="center">

**Made with ❤️ by Hx0 Team**

If this tool helped you, consider giving it a ⭐!

[Report Bug](https://github.com/yourusername/zip-cracker/issues) • [Request Feature](https://github.com/yourusername/zip-cracker/issues) • [Contribute](https://github.com/yourusername/zip-cracker/pulls)

</div>

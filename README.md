# WPair - CVE-2025-36911 Fast Pair Vulnerability Scanner

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white)](https://www.python.org/)

**WPair** is a defensive security research tool that demonstrates CVE-2025-36911 vulnerability in Google's Fast Pair protocol.

## ⚠️  SECURITY RESEARCH TOOL - AUTHORIZED USE ONLY ⚠️

This tool is provided for:
- Security research and education
- Testing devices you **OWN**
- Authorized penetration testing with **written permission**

**Unauthorized access to computer systems is ILLEGAL.**

Violators will be prosecuted under applicable laws including:
- Computer Fraud and Abuse Act (USA)
- Computer Misuse Act (UK)
- Similar legislation in your jurisdiction

By using this tool, you agree to use it responsibly and legally.

---

## What is CVE-2025-36911?

CVE-2025-36911 (also known as "WhisperPair") is a vulnerability in Google's Fast Pair protocol that affects millions of Bluetooth audio devices worldwide.

**Impact:**
- **Unauthorized Bluetooth pairing** without user consent
- **Microphone access** via Hands-Free Profile (HFP)
- **Persistent device tracking** via Account Key injection

**CVSS Score:** 8.1 (High)

**Affected Devices:** JBL, Sony, Google Pixel Buds, Anker, Nothing, OnePlus, Beats, Bose, Jabra, Xiaomi, and many others.

---

## Installation

### From PyPI (when published)

```bash
pip install wpair
```

### From Source

```bash
git clone https://github.com/wpair/wpair-cli.git
cd wpair-cli
pip install -e ".[dev]"
```

---

## Usage

### Scan for Fast Pair Devices

```bash
wpair scan --timeout 30
```

### Test a Device for Vulnerability

```bash
wpair test AA:BB:CC:DD:EE:FF
```

**Output:**
- `VULNERABLE` - Device is affected by CVE-2025-36911
- `PATCHED` - Device has been updated with security fix
- `ERROR` - Test inconclusive

### Exploit (Authorized Testing ONLY)

```bash
wpair exploit AA:BB:CC:DD:EE:FF --confirm
```

**⚠️  WARNING:** Only use on devices you own or have explicit permission to test!

---

## Features

| Feature | Description |
|---------|-------------|
| **BLE Scanner** | Discovers Fast Pair devices broadcasting the 0xFE2C service UUID |
| **Vulnerability Tester** | Non-invasive check if device is patched against CVE-2025-36911 |
| **Exploit Demonstration** | Full proof-of-concept for authorized security testing |

---

## Development

### Setup Development Environment

```bash
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -e ".[dev]"
```

### Run Tests

```bash
pytest
```

### Code Formatting

```bash
black wpair/
ruff check wpair/
```

### Type Checking

```bash
mypy wpair/
```

---

## Credits

### Original Research Team - KU Leuven, Belgium

| Researcher | Affiliation |
|------------|-------------|
| Sayon Duttagupta | COSIC Group |
| Nikola Antonijević | COSIC Group |
| Bart Preneel | COSIC Group |
| Seppe Wyns | DistriNet Group |
| Dave Singelée | DistriNet Group |

**Funding:** Flemish Government Cybersecurity Research Program (VOEWICS02)

**Resources:**
- [WhisperPair Research Paper](https://whisperpair.eu)
- [WIRED Coverage](https://www.wired.com/story/google-fast-pair-bluetooth-audio-accessories-vulnerability-patches/)

---

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

---

## Disclaimer

This application is an independent implementation created for security research purposes. The original KU Leuven researchers discovered and disclosed the vulnerability but have not released any code and are not affiliated with this project. Their inclusion in credits is solely to acknowledge their research contribution.

**Built for the security research community.**

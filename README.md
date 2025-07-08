# WiFiPasswordTester
Java CLI tool to test WiFi passwords using netsh &amp; wordlist

# WiFiPasswordTester

A powerful, easy-to-use Java CLI tool for testing Wi-Fi passwords using Windows `netsh` and a customizable wordlist.

---

## 🚦 Overview

**WiFiPasswordTester** is a console-based Java application that helps you test the security of Wi-Fi networks by performing a dictionary (brute-force) attack using a wordlist. It scans available Wi-Fi networks on Windows, lets you select a target SSID, and then attempts to connect using each password in your list. This tool is designed for **cybersecurity education** and **authorized penetration testing** only.

---

## ✨ Features

- 🔍 **Scan**: Detects nearby Wi-Fi networks using `netsh wlan show networks`
- 🎯 **Select**: User-friendly network selection menu
- 📖 **Wordlist Support**: Reads passwords from a `.txt` file (one password per line)
- 🏹 **Automated Testing**: Attempts connection with each password in the wordlist
- ✅ **Success/Failure Feedback**: Instantly notifies on successful connection
- 🧹 **Clean Up**: Dynamically creates and deletes temporary Wi-Fi profiles for each attempt
- 🖥️ **CLI-Based**: Simple, efficient, and scriptable for quick testing
- 🛡️ **Safe**: Never stores or leaks any tested passwords

---

## 🛠️ Requirements

- **Java JDK 8 or higher**
- **Windows OS only** (uses `netsh`; not compatible with macOS/Linux)
- **Administrator Command Prompt** (recommended for best compatibility)
- A plain text wordlist file (e.g., `wordlist.txt`) with one password per line

---

## 🚀 Quick Start

1. **Clone or Download** this repository.

2. **Compile:**
   ```bash
   javac WiFiPasswordTester.java
   ```

3. **Run:**
   ```bash
   java WiFiPasswordTester
   ```

4. **Follow the prompts:**
   - Select the Wi-Fi network by number
   - Enter the wordlist file path (`wordlist.txt` by default)
   - Watch as it cycles through passwords and displays results

---

## 📁 Example Wordlist (`wordlist.txt`)

```
12345678
password123
iloveyou
mypassword
admin123
```

---

## ⚠️ Legal & Ethical Notice

> **This tool is for EDUCATIONAL and ETHICAL TESTING purposes ONLY.**
>
> - Use **only** on networks you own or have explicit permission to test.
> - **DO NOT** attempt unauthorized access to networks.  
> - Unauthorized use is **illegal** and may result in criminal prosecution.

---

## 📝 License

Released under the **MIT License**. See [`LICENSE`](LICENSE) for details.

---

## 👤 Author

**Muhammad Arslan** (`CosmicHackerX`)  
Cybersecurity enthusiast, network defender, and ethical hacker.

---

## 🛣️ Roadmap

- **Version 2:** Swing GUI interface (coming soon)
- **Version 3:** Double-clickable `.jar` with enhanced UX (coming soon)

---

## 💬 Feedback & Contributions

Pull requests, bug reports, and feature suggestions are welcome!  
If you find this project useful, give it a ⭐ on GitHub.

---

Stay ethical. Hack responsibly.

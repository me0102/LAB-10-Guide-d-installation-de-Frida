# LAB-10-Guide-d-installation-de-Frida

# 🔬 Lab Report — Frida Mobile Security Framework

> **Student:** Bouchra Ouzanzoul  
> **Program:** 2nd Year Engineering Cycle — Cybersecurity & Telecommunications (ENSA Marrakech)  
> **Course:** Mobile Application Security  
> **Date:** April 27, 2026

---

## 📋 Table of Contents

- [Objectives](#-objectives)
- [Environment & Prerequisites](#-environment--prerequisites)
- [Lab Walkthrough](#-lab-walkthrough)
  - [1. Installing the Frida Client](#1-installing-the-frida-client)
  - [2. Deploying frida-server on Android](#2-deploying-frida-server-on-android)
  - [3. Connection Tests & Minimal Injections](#3-connection-tests--minimal-injections)
- [Advanced Exploration — Security Analysis](#-advanced-exploration--security-analysis)
  - [Interactive Console](#interactive-console)
  - [Instrumentation & Hooking](#instrumentation--hooking)
- [Troubleshooting](#-troubleshooting)
- [Conclusion](#-conclusion)

---

## 🎯 Objectives

The goal of this lab was to set up a complete **dynamic analysis environment** for Android applications using the Frida framework. Key steps included:

- Installing the Frida client on a Windows workstation
- Deploying and configuring `frida-server` on an Android emulator
- Validating the environment through JavaScript injection (Java and native)
- Exploring instrumentation capabilities for security analysis (network, files, local storage)

---

## 🖥️ Environment & Prerequisites

| Component | Details |
|-----------|---------|
| **Host Machine** | Windows 10 (v10.0.26200) |
| **Target Device** | Android Emulator (x86_64 architecture) |
| **Python** | 3.9.0 |
| **ADB** | Android Debug Bridge v1.0.41 |
| **Frida** | 17.9.1 |

---

## 🔧 Lab Walkthrough

### 1. Installing the Frida Client

Installation was done via the Python package manager `pip`:

```bash
pip install frida-tools
frida --version
# → 17.9.1
```

Both the CLI executable and the Python library were confirmed at version **17.9.1**.

---

### 2. Deploying frida-server on Android

#### Step 1 — Identify the emulator architecture

```bash
adb shell getprop ro.product.cpu.abi
# → x86_64
```

#### Step 2 — Push and configure the binary

```bash
# Transfer the server binary
adb push frida-server-17.9.1-android-x86_64 /data/local/tmp/frida-server

# Make it executable
adb shell chmod 755 /data/local/tmp/frida-server
```

#### Step 3 — Launch frida-server in the background

```bash
adb shell "nohup /data/local/tmp/frida-server &"
```

#### Step 4 — Forward communication ports

```bash
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

---

### 3. Connection Tests & Minimal Injections

#### Listing active processes

```bash
frida-ps -D emulator-5556
```

#### Java injection — `hello.js`

Injected into Chrome (`com.android.chrome`). After issuing `%resume`, the console output confirmed:

```
[+] Frida Java.perform OK
```

#### Native hook — `hello_native.js`

Attempted interception of the `recv` function from a system library. The script loaded successfully:

```
[+] Script loaded
```

---

## 🔍 Advanced Exploration — Security Analysis

### Interactive Console

Using the interactive Frida console on the Chrome process, the following critical information was extracted:

- **Architecture:** x64
- **Loaded modules:** Sensitive libraries like `libcrypto.so` and `libssl.so` identified via:
  ```javascript
  Process.enumerateModules()
  ```
- **Network functions:** Memory addresses of `connect`, `send`, and `recv` located in `libc.so`

---

### Instrumentation & Hooking

Several advanced scripts were deployed to monitor common attack vectors:

| Vector | Hook Target | Observed Data |
|--------|------------|---------------|
| **File System** | `open` / `read` syscalls | File descriptors accessed in real time |
| **Local Storage** | `SharedPreferencesImpl` class | Keys and values read/written by the app |
| **Database** | `SQLiteDatabase.execSQL` | SQLite queries executed at runtime |

---

## 🛠️ Troubleshooting

### Problem: `unable to access process`

**Symptom:** Injection failed with privilege errors on first attempts.

**Diagnosis:** `frida-server` was running as the `shell` user, which lacks the permissions required to instrument third-party processes.

**Fix:**

```bash
# Restart ADB as root
adb root

# Restart frida-server
adb shell "nohup /data/local/tmp/frida-server &"

# Specify the target device explicitly to resolve ambiguity (multiple emulators detected)
frida -D emulator-5556 -n com.android.chrome -l script.js
```

Using the `-D emulator-5556` flag also resolved the ambiguity caused by multiple emulator instances being detected by ADB.

---

## ✅ Conclusion

This lab validated the complete dynamic instrumentation chain with Frida. The ability to intercept both **high-level Java methods** and **native system functions** is a major asset for:

- Vulnerability analysis
- Reverse engineering
- Bypassing protection mechanisms such as **Root Detection** and **SSL Pinning**

This hands-on experience demonstrates how Frida serves as a powerful tool in the mobile cybersecurity analyst's toolkit.

---

<div align="center">
  <sub>ENSA Marrakech · Cybersecurity & Telecommunications · 2026</sub>
</div>

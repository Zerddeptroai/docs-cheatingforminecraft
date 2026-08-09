# APK Reverse Engineering: Quick Reference Guide

## 1. APK Structure at a Glance

```
MyApp.apk (ZIP Archive)
├── META-INF/                 # Digital Signature & Certificates
│   ├── MANIFEST.MF
│   ├── CERT.RSA
│   └── CERT.SF
│
├── AndroidManifest.xml       # App Configuration (permissions, activities)
│                             # Binary format - use apktool to decode
│
├── classes.dex               # Primary Compiled Java Code (1.8 MB typical)
├── classes2.dex              # Additional code if multidex (158 KB)
├── classes3.dex              # More code if needed (9.2 KB)
│
├── resources.arsc            # Compiled Resources (strings, layouts, drawables)
│
├── res/                       # Raw Resources (XML, images, animations)
│   ├── drawable/            # Images & drawable resources
│   ├── layout/               # UI layout files
│   ├── values/               # Strings, colors, dimensions
│   ├── menu/                 # Menu definitions
│   └── ...
│
└── lib/                      # Native Libraries (C/C++ compiled code)
    ├── arm64-v8a/           # 64-bit ARM (most modern Android)
    │   └── libipjngysad.so   # Native library (890 KB)
    ├── armeabi-v7a/         # 32-bit ARM (older devices)
    │   └── libipjngysad.so
    └── x86_64/              # Intel 64-bit (emulators/tablets)
        └── libipjngysad.so
```

---

## 2. Quick Analysis Checklist

### Initial Assessment (5 minutes)

```
□ 1. Extract APK
      unzip -q myapp.apk -d extracted/

□ 2. Check file sizes
      du -sh extracted/*/
      
□ 3. Verify structure
      ls -la extracted/

□ 4. Check for native code
      find extracted/lib -name "*.so"
      
□ 5. Look for multidex
      ls -l extracted/classes*.dex | wc -l
```

### Manifest Analysis (10 minutes)

```
□ 1. Decode manifest
      apktool d myapp.apk
      
□ 2. Extract package name
      grep 'package=' myapp/AndroidManifest.xml
      
□ 3. List permissions
      grep 'uses-permission' myapp/AndroidManifest.xml
      
□ 4. Find entry point
      grep -A 5 'android.intent.action.MAIN' myapp/AndroidManifest.xml
      
□ 5. Check exported components
      grep 'android:exported="true"' myapp/AndroidManifest.xml
```

### String Extraction (10 minutes)

```
□ 1. Extract from DEX
      python3 extract_strings_from_dex.py classes.dex
      
□ 2. Search for suspicious patterns
      grep -i "http\|api\|key\|secret\|password" strings.txt
      
□ 3. Extract from native library
      python3 extract_strings_from_so.py lib/arm64-v8a/lib.so
      
□ 4. Look for hardcoded credentials
      grep -i "username\|password\|token" strings.txt
```

### Java Code Review (30 minutes)

```
□ 1. Convert DEX to JAR
      d2j-dex2jar.sh classes.dex
      
□ 2. Decompile JAR
      java -jar cfr.jar classes-dex2jar.jar --outputdir src/
      
□ 3. Find main activity
      find src -name "*MainActivity.java"
      
□ 4. Trace network calls
      grep -r "HttpClient\|Socket\|URL" src/
      
□ 5. Find JNI calls
      grep -r "System.loadLibrary\|native " src/
```

### Native Code Analysis (30+ minutes)

```
□ 1. Check architecture
      readelf -h lib/arm64-v8a/lib.so
      
□ 2. List exported functions
      readelf -s lib/arm64-v8a/lib.so | grep Java_
      
□ 3. Open in Ghidra
      ghidraRun &
      # File → Open → Select .so file
      
□ 4. Analyze with Radare2
      r2 lib/arm64-v8a/lib.so
      aa  (analyze all)
      afl (list functions)
      
□ 5. Extract strings from native code
      python3 extract_strings_from_so.py lib/arm64-v8a/lib.so
```

---

## 3. Essential Command Reference

### APK Manipulation

```bash
# Extract APK (it's a ZIP file)
unzip -q myapp.apk -d extracted/

# Repackage APK
cd extracted/
zip -r ../modified.apk *
cd ..

# Sign APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore keystore.jks modified.apk alias

# Install APK
adb install -r myapp.apk
```

### Decompilation Tools

```bash
# Decode APK (resources + manifest)
apktool d myapp.apk
apktool b myapp -o myapp_rebuilt.apk

# Convert DEX to JAR
d2j-dex2jar.sh classes.dex

# Decompile JAR to Java
java -jar cfr.jar classes-dex2jar.jar --outputdir src/
java -jar procyon.jar classes-dex2jar.jar -o src/

# Direct DEX decompile
apk-string-extractor myapp.apk
radare2 classes.dex
```

### Binary Analysis

```bash
# ELF header analysis
readelf -h lib/arm64-v8a/lib.so        # Header info
readelf -s lib/arm64-v8a/lib.so        # Symbol table
readelf -l lib/arm64-v8a/lib.so        # Program headers
readelf -S lib/arm64-v8a/lib.so        # Section headers

# Disassembly
objdump -d lib/arm64-v8a/lib.so        # Full disassembly
objdump -d lib/arm64-v8a/lib.so > dis.txt

# String extraction
strings lib/arm64-v8a/lib.so
nm lib/arm64-v8a/lib.so                # Symbol names
```

### Dynamic Analysis

```bash
# Frida setup
frida-ps -U                            # List running apps
frida -U -f com.app.name -l hook.js    # Launch with hook
frida -U com.app.name -l hook.js       # Attach to running app

# ADB
adb shell am start -n com.app/.MainActivity
adb shell pm dump com.app              # App info
adb logcat                              # Monitor logs
adb shell tcpdump -i any -w traffic.pcap
```

---

## 4. File Format Quick Reference

### DEX File Header (First 32 bytes)

```
Offset  Length  Description
0x00    8       Magic: "dex\n035\0" (or similar version)
0x08    4       Checksum (adler32)
0x0C    20      SHA-1 digest
0x20    4       File size (little-endian)
0x24    4       Header size (0x70)
```

### ELF File Header (First 64 bytes)

```
Offset  Length  Description
0x00    4       Magic: 0x7F 0x45 0x4C 0x46 (ELF)
0x04    1       Class: 1 (32-bit) or 2 (64-bit)
0x05    1       Data: 1 (little-endian) or 2 (big-endian)
0x06    1       Version: 1
0x07    1       OS/ABI
0x08    8       ABI version (padding)
0x10    2       Type: 1 (relocatable), 2 (executable), 3 (shared)
0x12    2       Machine: 0x03 (Intel 80386), 0xB7 (ARM64)
0x14    4       Version
0x18    8       Entry point address (64-bit)
0x20    8       Program header offset
0x28    8       Section header offset
```

### JNI Function Naming

```
Pattern: Java_[PackageName]_[ClassName]_[MethodName]

Example: Java_com_app_Native_Init
         ^^^^  ^^^_^^^  ^^^^^^ ^^^^
         |     |        |      |
         |     |        |      Method name
         |     |        Class name
         |     Package name
         Prefix
```

---

## 5. Common Patterns to Look For

### Pattern 1: Network Communication

```java
// Decompiled code pattern
HttpClient client = new HttpClient();
HttpGet request = new HttpGet("http://api.example.com/data");
HttpResponse response = client.execute(request);

// What to search for
grep -r "HttpClient\|HttpGet\|HttpPost\|Socket\|URL" src/
```

### Pattern 2: JNI/Native Code Calls

```java
// Decompiled code pattern
static {
    System.loadLibrary("libname");
}

public native String someFunction();
private native void nativeMethod(String arg);

// What to search for
grep -r "System.loadLibrary\|native \|JNI\|Jni" src/
```

### Pattern 3: String Obfuscation

```java
// Common patterns
String encoded = "VGVzdA==";  // Base64 encoded
byte[] decoded = Base64.decode(encoded);

String xored = "xyz";  // XOR encoded
for (char c : xored.toCharArray()) c ^= 0xFF;

// What to search for
grep -r "Base64\|decode\|encode\|xor\|crypt" src/
```

### Pattern 4: Data Exfiltration

```java
// Common patterns
SharedPreferences prefs = getSharedPreferences("data", 0);
String data = prefs.getString("user_id", "");

Intent intent = new Intent();
intent.setData(Uri.parse("content://..."));
startActivity(intent);

// What to search for
grep -r "SharedPreferences\|ContentProvider\|Intent" src/
```

---

## 6. Threat Indicators Checklist

### High Priority Findings

```
⚠️ CRITICAL:
  □ Hardcoded API keys / credentials
  □ Calls to Runtime.exec() (code execution)
  □ Certificate pinning disabled
  □ Unencrypted data storage
  □ Exported components without permission checks

⚠️ HIGH:
  □ Network communication without HTTPS
  □ Unnecessary permissions requested
  □ Suspicious JNI functions
  □ Command injection vulnerabilities
  □ Insecure data deserialization

⚠️ MEDIUM:
  □ Outdated libraries with known vulnerabilities
  □ Debug symbols left in release build
  □ Weak cryptographic algorithms
  □ Information disclosure in error messages
```

---

## 7. Tool Comparison Matrix

| Task | Tool | Pros | Cons |
|------|------|------|------|
| **Decode APK** | apktool | Handles all resources | Requires Java |
| | jadx | Direct DEX→Java | May fail on obfuscation |
| **Analyze Java** | CFR | Modern Java features | Command-line only |
| | JD-GUI | Visual interface | Outdated, slower |
| | Procyon | Accurate decompilation | Less active maintenance |
| **Reverse ELF** | Ghidra | Powerful, free | Learning curve |
| | IDA Pro | Industry standard | Very expensive |
| | Radare2 | Command-line scripting | Complex CLI |
| **Dynamic Hook** | Frida | Real-time instrumentation | Requires Frida server |
| | Xposed | Framework-wide hooks | Needs rooted device |
| | strace | System call tracing | Low-level only |

---

## 8. Common Mistakes & Fixes

### Mistake 1: "Cannot read resources after extracting"

**Problem:** Resources are binary/compressed
```bash
# Wrong: trying to read binary files
cat resources.arsc

# Right: use apktool
apktool d myapp.apk
```

### Mistake 2: "DEX file is unreadable"

**Problem:** DEX is bytecode, not source
```bash
# Wrong: trying to read binary DEX
cat classes.dex

# Right: convert to readable format
d2j-dex2jar.sh classes.dex
java -jar cfr.jar classes-dex2jar.jar --outputdir src/
```

### Mistake 3: "Symbols not found in native library"

**Problem:** Library is stripped (common in production)
```bash
# Wrong: expecting all symbols
readelf -s lib.so | grep Function_Name

# Right: use Ghidra/Radare2 to find code
ghidra lib.so
```

### Mistake 4: "JNI function names don't match"

**Problem:** Obfuscation in package/class names
```
// App package might be obfuscated
Java_a_b_c_Init  // Not "Java_com_app_Native_Init"

// Solution: Search for pattern
grep -r "Java_" lib.so | strings
```

---

## 9. Timeline: Analysis Time Estimates

| Phase | Duration | Complexity |
|-------|----------|-----------|
| Extraction & Exploration | 5 min | Low |
| Manifest Analysis | 10 min | Low |
| String Extraction | 10 min | Low |
| Java Decompilation | 5 min | Low |
| Java Code Review | 30-60 min | Medium |
| Native Code Disassembly | 10 min | Low |
| Native Code Analysis | 60-180 min | High |
| Dynamic Testing | 30-120 min | Medium |
| **Total (Basic)** | **70 min** | |
| **Total (Thorough)** | **360+ min** | |

---

## 10. Real-World RE Workflow

```
START
  │
  ├─→ Extract APK
  │   └─→ Verify structure
  │
  ├─→ Quick Win: String Search
  │   ├─→ Found credentials? → ALERT
  │   ├─→ Found URLs? → Investigate
  │   └─→ No findings → Continue
  │
  ├─→ Manifest Analysis
  │   ├─→ Suspicious permissions? → Investigate
  │   ├─→ Exported components? → Investigate
  │   └─→ Looks normal? → Continue
  │
  ├─→ Java Code Review
  │   ├─→ Network calls found? → Monitor dynamically
  │   ├─→ JNI calls found? → Analyze native code
  │   ├─→ Suspicious logic? → Deep dive
  │   └─→ Nothing suspicious? → Verify with dynamic
  │
  ├─→ Native Code Analysis (if applicable)
  │   ├─→ Stripped? → Use Ghidra
  │   ├─→ Large? → Focus on JNI functions
  │   └─→ Suspicious patterns? → Deep analysis
  │
  ├─→ Dynamic Analysis (Optional)
  │   ├─→ Hook network calls
  │   ├─→ Monitor file access
  │   ├─→ Verify hypotheses
  │   └─→ Capture evidence
  │
  └─→ Report Findings
```

---

## 11. Useful One-Liners

```bash
# Find all URLs in APK
unzip -p myapp.apk classes.dex | strings | grep -E 'https?://'

# Count methods in all DEX files
for f in extracted/classes*.dex; do
  echo -n "$f: "
  python3 -c "
import struct
with open('$f','rb') as f:
    d=f.read()
    print(struct.unpack('<I',d[0x58:0x5C])[0])
  "
done

# List all JNI functions in .so
objdump -T extracted/lib/arm64-v8a/lib.so | grep Java_

# Find all network-related strings
strings extracted/classes*.dex extracted/lib/*/lib.so | \
  grep -iE 'http|url|socket|ssl|tls|dns|domain'

# Extract strings with addresses
python3 -c "
with open('extracted/lib/arm64-v8a/lib.so','rb') as f:
    d=f.read()
    off=0
    while off<len(d):
        if 32<=d[off]<=126:
            s=b''
            while off<len(d) and 32<=d[off]<=126:
                s+=bytes([d[off]])
                off+=1
            if len(s)>5:
                print(hex(off-len(s)),':',s.decode())
        else:
            off+=1
"

# Monitor APK installation
adb logcat | grep -i "install\|pm\|dalvik"
```

---

## 12. Reference Links (One per tool)

| Tool | Official Link |
|------|---|
| **Ghidra** | https://ghidra-sre.org/ |
| **Frida** | https://frida.re/ |
| **apktool** | https://ibotpeaches.github.io/Apktool/ |
| **Radare2** | https://rada.re/ |
| **dex2jar** | https://github.com/ThexXTURBOXx/dex2jar |
| **CFR** | https://www.benf.org/other/cfr/ |
| **JADX** | https://github.com/skylot/jadx |
| **Android SDK** | https://developer.android.com/studio |

---

*Use this as a quick reference while analyzing APKs. Print or bookmark for easy access during RE sessions.*

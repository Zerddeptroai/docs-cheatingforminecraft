# Practical APK Reverse Engineering Guide
## Step-by-Step Walkthroughs

---

## Part 1: Setting Up Your Environment

### Required Tools Installation

#### Ubuntu/Debian
```bash
# Update package manager
sudo apt update

# Install Java (needed for most tools)
sudo apt install openjdk-11-jdk

# Install Python 3
sudo apt install python3 python3-pip

# Install Ghidra (Java-based disassembler)
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_10.1.2_build/ghidra_10.1.2_PUBLIC_20220125.zip
unzip ghidra_10.1.2_PUBLIC_20220125.zip
cd ghidra_10.1.2_PUBLIC && ./ghidraRun &

# Install apktool
wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.7.0.jar
mv apktool_2.7.0.jar apktool.jar
alias apktool='java -jar /path/to/apktool.jar'

# Install dex2jar
wget https://github.com/ThexXTURBOXx/dex2jar/releases/download/v2.1-20230506-181904/dex-tools-v2.1-20230506-181904.zip
unzip dex-tools-v2.1-20230506-181904.zip

# Install Radare2
sudo apt install radare2 radare2-dev radare2-doc

# Install Frida (dynamic instrumentation)
pip3 install frida frida-tools
```

#### macOS
```bash
brew install java
brew install radare2
brew install apktool
pip3 install frida frida-tools
```

### Verify Installation
```bash
java -version
apktool --version
radare2 -v
frida --version
```

---

## Part 2: Basic APK Analysis Workflow

### Walkthrough 1: Extract and Explore APK Structure

**Goal:** Understand what's inside an APK

**Step 1: Extract APK**
```bash
# APK is just a ZIP file
unzip -q myapp.apk -d extracted_app/

# Verify extraction
ls -la extracted_app/
```

**Expected Output:**
```
extracted_app/
├── AndroidManifest.xml     (App configuration)
├── classes.dex             (Main Java bytecode)
├── classes2.dex            (Additional bytecode)
├── resources.arsc          (Compiled resources)
├── res/                    (Raw resources)
│   ├── drawable/
│   ├── layout/
│   ├── values/
│   └── ...
├── lib/                    (Native libraries)
│   ├── arm64-v8a/
│   ├── armeabi-v7a/
│   └── x86_64/
└── META-INF/              (Signing certificates)
    ├── MANIFEST.MF
    ├── CERT.RSA
    └── CERT.SF
```

**Step 2: Explore Each Component**

```bash
# List all files with sizes
find extracted_app -type f -exec du -h {} \; | sort -hr | head -20

# Find all executable files
find extracted_app -type f -name "*.so"

# Find all configuration files
find extracted_app -type f -name "*.xml"

# Count methods in DEX files
cd extracted_app
python3 << 'EOF'
import struct
import os

def count_methods_in_dex(filename):
    with open(filename, 'rb') as f:
        data = f.read()
    # DEX header offsets
    # methods_size at offset 0x58-0x5B (little-endian)
    methods_size = struct.unpack('<I', data[0x58:0x5C])[0]
    return methods_size

for dex_file in ['classes.dex', 'classes2.dex', 'classes3.dex']:
    if os.path.exists(dex_file):
        count = count_methods_in_dex(dex_file)
        print(f"{dex_file}: {count} methods")
EOF
```

**Key Insights:**
- Larger DEX files = more functionality
- Multiple .so files = native code for different architectures
- Resource size indicates UI complexity

### Walkthrough 2: Analyze AndroidManifest.xml

**Goal:** Understand app configuration and permissions

**Step 1: Decode Manifest**
```bash
apktool d myapp.apk

# Now you have decoded manifest
cat myapp/AndroidManifest.xml
```

**Step 2: Parse Key Information**

Look for:
```xml
<!-- App package and version -->
<manifest package="com.example.myapp" android:versionCode="1" android:versionName="1.0">

<!-- Required permissions -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

<!-- Main application -->
<application android:label="@string/app_name" ...>
    
    <!-- Activities (screens) -->
    <activity android:name=".MainActivity" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
    
    <!-- Services (background) -->
    <service android:name=".MyService" />
    
    <!-- Broadcast receivers -->
    <receiver android:name=".MyReceiver" />
    
    <!-- Content providers -->
    <provider android:name=".MyProvider" android:authorities="com.example.myapp.provider" />
</application>
```

**Step 3: Extract Information**
```bash
# Get package name
grep 'package=' myapp/AndroidManifest.xml

# Get all permissions
grep 'uses-permission' myapp/AndroidManifest.xml

# Get all activities
grep 'android:name' myapp/AndroidManifest.xml | grep activity

# Get targeted API level
grep 'android:targetSdkVersion' myapp/AndroidManifest.xml
```

**Interesting Findings:**
- ⚠️ INTERNET permission = network communication
- ⚠️ WRITE_EXTERNAL_STORAGE = file access
- ⚠️ android:exported="true" = can be called from other apps
- ⚠️ Service without explicit launch = hidden background process

### Walkthrough 3: Extract and Analyze Strings

**Goal:** Find hardcoded strings (URLs, API keys, messages)

**Step 1: Extract from DEX Files**
```bash
# Install apk-string-extractor (or use strings if available)
cd extracted_app

# Try to extract readable strings
python3 << 'EOF'
import sys
import struct

def extract_strings_from_dex(filename):
    with open(filename, 'rb') as f:
        data = f.read()
    
    # Find string pool section
    # DEX string pool usually contains visible strings
    result = []
    current = b''
    
    for byte in data:
        if 32 <= byte <= 126 or byte in [10, 13, 9]:  # Printable ASCII or whitespace
            current += bytes([byte])
        else:
            if len(current) > 4:
                try:
                    decoded = current.decode('ascii', errors='ignore').strip()
                    if decoded and not decoded.startswith('\x00'):
                        result.append(decoded)
                except:
                    pass
            current = b''
    
    return result

for dex in ['classes.dex', 'classes2.dex', 'classes3.dex']:
    try:
        strings = extract_strings_from_dex(dex)
        print(f"\n=== Strings from {dex} ===")
        # Look for interesting patterns
        for s in strings:
            if any(x in s.lower() for x in ['http', 'api', 'key', 'token', 'secret', 'password', 'auth']):
                print(f"[INTERESTING] {s}")
    except:
        pass
EOF
```

**Step 2: Search for Specific Patterns**
```bash
# Search for URLs
python3 << 'EOF'
import re
import os

def find_urls_in_file(filename):
    with open(filename, 'rb') as f:
        content = f.read().decode('utf-8', errors='ignore')
    
    urls = re.findall(r'https?://[^\s<>"{}|\\^`\[\]]*', content)
    return urls

# Check all DEX files
for dex in ['classes.dex', 'classes2.dex', 'classes3.dex']:
    if os.path.exists(dex):
        urls = find_urls_in_file(dex)
        if urls:
            print(f"URLs in {dex}:")
            for url in set(urls):
                print(f"  {url}")
EOF

# Search for sensitive keywords
grep -a -i "password\|api_key\|secret\|token" extracted_app/classes.dex 2>/dev/null | od -c | head -20
```

**Step 3: Analyze Native Library Strings**
```bash
# Use Ghidra to analyze .so file
# Or extract strings programmatically
python3 << 'EOF'
# Read binary file
with open('extracted_app/lib/arm64-v8a/libipjngysad.so', 'rb') as f:
    data = f.read()

# Extract null-terminated strings
strings = []
current = b''
for byte in data:
    if 32 <= byte <= 126:
        current += bytes([byte])
    elif current:
        try:
            s = current.decode('ascii')
            if len(s) > 4:
                strings.append(s)
        except:
            pass
        current = b''

# Print interesting strings
print("Extracted strings from .so file:")
for s in sorted(set(strings)):
    if any(x in s.lower() for x in ['http', 'api', 'func', 'java', 'native', 'init']):
        print(f"  {s}")
EOF
```

---

## Part 3: Java Code Analysis

### Walkthrough 4: Decompile DEX to Java

**Goal:** Read the actual Java source code

**Step 1: Convert DEX to JAR**
```bash
# Using dex2jar
d2j-dex2jar.sh classes.dex
# Output: classes-dex2jar.jar

# Also convert multi-dex
d2j-dex2jar.sh classes2.dex
d2j-dex2jar.sh classes3.dex
```

**Step 2: Decompile JAR to Java**
```bash
# Option 1: Use CFR
java -jar cfr.jar classes-dex2jar.jar --outputdir src/

# Option 2: Use Procyon
java -jar procyon.jar -o src/ classes-dex2jar.jar

# Option 3: Use JD-GUI (GUI tool)
jd-gui classes-dex2jar.jar &
```

**Step 3: Analyze Decompiled Code**
```bash
# Find main activity
find src/ -name "*MainActivity*"

# Look for network communication
grep -r "HttpClient\|HttpConnection\|Socket\|URL" src/ | head -20

# Look for JNI calls
grep -r "System.loadLibrary\|native " src/ | head -10

# Look for suspicious patterns
grep -r "Runtime.exec\|ProcessBuilder" src/
```

**Example Findings:**
```java
// Native library loading
static {
    System.loadLibrary("ipjngysad");
}

// JNI method calls
public native String AtsStrings();
public native void Init();
public native void setClick();
public native void setLogin();

// Network call
HttpGet request = new HttpGet("http://api.example.com/data");
```

### Walkthrough 5: Trace Function Calls

**Goal:** Understand program flow

**Step 1: Find Entry Points**
```bash
# Main activity
grep -r "class.*MainActivity" src/

# Main function
grep -r "onCreate" src/ | head -5

# Application class
grep -r "class.*extends Application" src/
```

**Step 2: Trace Execution Path**
```java
// Example: MainActivity.java
public class MainActivity extends AppCompatActivity {
    
    static {
        // Load native library on app start
        System.loadLibrary("ipjngysad");
    }
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Initialize native code
        Init();
        
        // Setup click listeners
        findViewById(R.id.button).setOnClickListener(v -> {
            setClick();
            // Calls native function
        });
    }
}
```

**Step 3: Build Call Graph**
```
MainActivity.onCreate()
    ├── System.loadLibrary("ipjngysad")
    │   └── Loads native library
    ├── Init()
    │   └── Native function
    └── setOnClickListener()
        └── setClick()
            └── Native function
```

---

## Part 4: Native Code Analysis

### Walkthrough 6: Analyze ARM64 Library

**Goal:** Understand native C++ implementation

**Step 1: Load in Ghidra**
```bash
# Open Ghidra
ghidraRun &

# Open Project → New → Select working directory
# File → Open → Select libipjngysad.so
# Let it analyze (takes a few minutes)
```

**Step 2: Examine Functions**
```
In Ghidra:
1. Window → Symbol Tree
2. Find "Java_com_app_Native_*" functions
3. Double-click to view disassembly
4. Use "Define" menu to analyze
```

**Step 3: Manual Analysis with Radare2**

```bash
# Open with Radare2
r2 extracted_app/lib/arm64-v8a/libipjngysad.so

# Radare2 commands
aa                          # Analyze all
afl                         # List functions
pdf @ Java_com_app_Native_Init    # Print function disassembly
iz                          # Print strings
ix                          # Print imports
px 100 @ 0x1000             # Print hex at address

# Find JNI functions
?/x Java_com              # Search for string
```

**Example Output:**
```asm
; Function: Java_com_app_Native_Init
0x00040000      55             push rbp
0x00040001      4889e5         mov rbp, rsp
0x00040004      4883ec20       sub rsp, 0x20
0x00040008      48897df8       mov qword [rbp - 8], rdi
0x0004000c      488975f0       mov qword [rbp - 0x10], rsi
; ... more instructions

; Typically looks like:
; Setup stack frame
; Save arguments
; Call other functions
; Return result
```

### Walkthrough 7: Reverse Engineer String Handling

**Goal:** Understand obfuscation and string manipulation

**Step 1: Find String Functions**
```bash
# In Ghidra, search for "string" operations
# Look for functions like:
# - strcpy, strcat, strlen
# - memcpy, memset
# - malloc, free (memory allocation)
```

**Step 2: Analyze Algorithm**
```c
// Typical pattern: String decoding/deobfuscation
char* decode_string(const char* encoded) {
    char* result = malloc(strlen(encoded));
    
    for (int i = 0; encoded[i]; i++) {
        result[i] = encoded[i] ^ 0xAA;  // XOR with key
    }
    
    return result;
}
```

**Step 3: Replicate in Python**
```python
# If you find the algorithm, recreate it
def decode_string(encoded_bytes, key=0xAA):
    decoded = bytearray()
    for byte in encoded_bytes:
        decoded.append(byte ^ key)
    return decoded.decode('ascii', errors='ignore')

# Test
test_bytes = bytes([0xE3, 0xF2, 0xF4, 0xF4, 0xF6])  # "hello" XOR 0xAA
print(decode_string(test_bytes))  # Output: "hello"
```

---

## Part 5: Dynamic Analysis with Frida

### Walkthrough 8: Hook Native Functions

**Goal:** Intercept and monitor function calls at runtime

**Step 1: Setup Frida**
```bash
# Install Frida on PC
pip3 install frida frida-tools

# Install Frida server on Android device
adb push frida-server /data/local/tmp/
adb shell chmod +x /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &

# Verify connection
frida-ps -U
```

**Step 2: Create Hook Script**
```javascript
// hook.js
console.log("[*] Script loaded");

// Hook Java_com_app_Native_Init
var Java_Init = Module.findExportByName("libipjngysad.so", 
    "Java_com_app_Native_Init");

if (Java_Init) {
    Interceptor.attach(Java_Init, {
        onEnter: function(args) {
            console.log("[+] Java_com_app_Native_Init called");
            console.log("    JNIEnv: " + args[0]);
            console.log("    jobject: " + args[1]);
        },
        onLeave: function(retval) {
            console.log("[+] Java_com_app_Native_Init returned: " + retval);
        }
    });
} else {
    console.log("[-] Function not found");
}

// Hook String manipulation
var memcpy = Module.findExportByName(null, "memcpy");
if (memcpy) {
    Interceptor.attach(memcpy, {
        onEnter: function(args) {
            var dest = args[0];
            var src = args[1];
            var size = args[2].toInt32();
            
            console.log("[*] memcpy: dest=" + dest + 
                       " src=" + src + " size=" + size);
            
            // Read data being copied
            if (size < 256) {
                var data = src.readByteArray(size);
                console.log("    Data: " + data);
            }
        }
    });
}
```

**Step 3: Run Hook Script**
```bash
# Launch app on device
adb shell am start -n com.example.myapp/.MainActivity

# Run Frida hook
frida -U -f com.example.myapp -l hook.js --no-pause

# Keep running
# App functions are monitored in real-time
```

**Step 4: Analyze Output**
```
[*] Script loaded
[+] Java_com_app_Native_Init called
    JNIEnv: 0x7abc123456
    jobject: 0x7def789012
[*] memcpy: dest=0x1000 src=0x2000 size=128
    Data: [0x48, 0x65, 0x6c, 0x6c, 0x6f ...]  // "Hello..."
```

### Walkthrough 9: Monitor Network Requests

**Goal:** Capture API calls and network traffic

**Step 1: Hook Network Functions**
```javascript
// network_hook.js
console.log("[*] Network monitoring loaded");

// Hook send function
var libc = Module.findExportByName(null, "libc.so");
var send = Module.findExportByName(null, "send");

if (send) {
    Interceptor.attach(send, {
        onEnter: function(args) {
            var socket = args[0].toInt32();
            var data = args[1];
            var len = args[2].toInt32();
            
            if (len < 4096) {
                try {
                    var buffer = data.readByteArray(len);
                    var str = buffer.toString();
                    console.log("[*] send(" + socket + "): " + str);
                } catch(e) {
                    console.log("[*] send(" + socket + "): [binary data]");
                }
            }
        }
    });
}

// Hook recv function
var recv = Module.findExportByName(null, "recv");
if (recv) {
    Interceptor.attach(recv, {
        onLeave: function(retval) {
            console.log("[*] recv: " + retval + " bytes");
        }
    });
}
```

**Step 2: Intercept HTTPS (requires certificate pinning bypass)**
```javascript
// ssl_hook.js - Bypass certificate pinning
var SSL_CTX_set_verify = Module.findExportByName(null, 
    "SSL_CTX_set_verify");

if (SSL_CTX_set_verify) {
    Interceptor.attach(SSL_CTX_set_verify, {
        onEnter: function(args) {
            console.log("[*] SSL_CTX_set_verify called");
            // Modify verification mode to accept all certs
        }
    });
}
```

---

## Part 6: Practical Challenges

### Challenge 1: Find Hidden API Endpoint

**Scenario:** App communicates with backend API, but URL is not visible in strings

**Solution:**
```bash
# 1. Check decompiled code for URL construction
grep -r "https\|http" src/ --include="*.java"

# 2. Look for string building
grep -r "StringBuilder\|concat" src/ --include="*.java"

# 3. Search for base64 encoding
grep -r "Base64\|decode" src/ --include="*.java"

# 4. Hook network functions to see actual URLs
# (See Walkthrough 9)

# 5. Analyze with Burp Suite
#    - Intercept HTTPS traffic
#    - Monitor all requests
```

### Challenge 2: Decrypt Obfuscated String

**Scenario:** Found encoded string, need to decrypt it

**Steps:**
```python
# 1. Extract encoded string from binary
encoded = bytes.fromhex("E3F2F4F4F60x...")

# 2. Try common encodings
import base64
print(base64.b64decode(encoded))  # Try base64
print(encoded.decode('utf-8', errors='ignore'))  # Try UTF-8

# 3. Try XOR with different keys
for key in range(256):
    decoded = bytes(b ^ key for b in encoded)
    if b'\x00' not in decoded:
        try:
            print(f"Key {key}: {decoded.decode()}")
        except:
            pass

# 4. Try ROT13
import codecs
print(codecs.encode(encoded, 'rot_13'))

# 5. If nothing works, hook the decryption function with Frida
```

### Challenge 3: Identify Command & Control Server

**Scenario:** Find where app communicates with malicious server

**Steps:**
```bash
# 1. Extract all strings (see Walkthrough 3)
python3 extract_strings.py

# 2. Look for suspicious domains
grep -r "\.onion\|\.cc\|\.tk\|\.ml" extracted/

# 3. Check DNS queries with Frida
# Hook getaddrinfo function

# 4. Monitor network traffic
adb shell tcpdump -i any -w /data/local/tmp/traffic.pcap
# Copy traffic.pcap and analyze in Wireshark

# 5. Check configuration files
grep -r "server\|host\|domain" extracted/res/
```

---

## Part 7: Building Your Own RE Tools

### Simple DEX Analysis Tool

```python
#!/usr/bin/env python3
"""
Simple DEX file analyzer
"""

import struct
import sys

def analyze_dex(filename):
    with open(filename, 'rb') as f:
        data = f.read()
    
    # DEX magic and version
    magic = data[0:8]
    print(f"[*] Magic: {magic.hex()}")
    
    # DEX file size
    file_size = struct.unpack('<I', data[32:36])[0]
    print(f"[*] File Size: {file_size} bytes")
    
    # Number of strings
    string_ids_offset = struct.unpack('<I', data[0x34:0x38])[0]
    string_ids_size = struct.unpack('<I', data[0x38:0x3C])[0]
    print(f"[*] Strings: {string_ids_size}")
    
    # Number of types
    type_ids_size = struct.unpack('<I', data[0x40:0x44])[0]
    print(f"[*] Types: {type_ids_size}")
    
    # Number of methods
    method_ids_size = struct.unpack('<I', data[0x58:0x5C])[0]
    print(f"[*] Methods: {method_ids_size}")
    
    # Number of classes
    class_defs_size = struct.unpack('<I', data[0x60:0x64])[0]
    print(f"[*] Classes: {class_defs_size}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python3 dex_analyzer.py <dex_file>")
        sys.exit(1)
    
    analyze_dex(sys.argv[1])
```

**Usage:**
```bash
python3 dex_analyzer.py classes.dex
```

---

## Part 8: Common Mistakes & Best Practices

### Mistakes to Avoid

❌ **Blindly trusting tool output**
- Tools can have bugs
- Always verify with manual analysis

❌ **Ignoring obfuscation**
- App might be intentionally hard to read
- Look for suspicious code patterns

❌ **Not checking all DEX files**
- Multi-dex apps split code across files
- Must analyze all DEX files

❌ **Skipping Android manifest**
- Critical information about permissions
- Lists all exported components

### Best Practices

✅ **Start with automated tools**
- apktool for resources
- dex2jar for Java code
- Ghidra for native code

✅ **Verify with multiple tools**
- Run dex2jar and JADX on same file
- Compare results

✅ **Document findings**
- Keep notes on function purposes
- Map out data flow

✅ **Use version control**
- Track changes to extracted files
- Helps with complex analysis

✅ **Test your theories**
- Use Frida to verify hypotheses
- Hook functions to confirm behavior

---

## References & Resources

### Essential Links
- **Ghidra User Guide:** https://ghidra-sre.org/
- **Frida Documentation:** https://frida.re/docs/
- **Radare2 Book:** https://book.rada.re/
- **Android Internals:** https://www.oreilly.com/library/view/android-internals/9781491954936/

### Useful Repositories
- **Frida Scripts:** https://github.com/frida/frida-tools
- **Android Security Tools:** https://github.com/SEC7ION/APKHunt

### Learning Platforms
- **HackTheBox Mobile Challenges:** https://www.hackthebox.com/
- **TryHackMe Mobile RE:** https://tryhackme.com/
- **OWASP Mobile Top 10:** https://owasp.org/www-project-mobile-top-10/

---

*This guide is for educational purposes. Always ensure you have authorization before analyzing any application.*

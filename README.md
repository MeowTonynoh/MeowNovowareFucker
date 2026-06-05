# Meow Novoware Fucker

Forensic analysis tool to detect the **Novoware** cheat client on Minecraft through RAM memory scanning and NTFS journal analysis.

## ⚠️ Important Notice

**VirusTotal and other antiviruses may flag it as a false positive** — the tool reads process memory and downloads an external component, behaviors that some AVs consider suspicious. The tool is completely safe and the source code is available for verification.

---

## Usage

Download and run the `.exe` file. The program offers three options accessible through the main menu:

**[1] Scan Memory Processes** — RAM memory forensic analysis

**[2] Journal Trace** — NTFS journal forensic analysis (powered by spokwn's JournalTrace)

**[3] Exit**

---

## How It Works

### Option 1: Memory Process Scanner

This function performs a forensic RAM memory analysis to detect whether Novoware is currently running.

#### Phase 1: Process Discovery

**Java process enumeration**
- Identifies all `java.exe` and `javaw.exe` processes active on the system
- Collects metadata: PID, process name, uptime
- Requests handle with `PROCESS_VM_READ` and `PROCESS_QUERY_INFORMATION` permissions
- If no Java process is found, the user is prompted to start Minecraft first

#### Phase 2: Memory Mapping

**Memory region mapping**
- Enumerates all process memory regions via `VirtualQueryEx`
- Filters only committed regions (`MEM_COMMIT`)
- Selects regions with readable protection attributes: `PAGE_READONLY`, `PAGE_READWRITE`, `PAGE_EXECUTE_READ`, `PAGE_EXECUTE_READWRITE`
- Excludes guarded regions with `PAGE_GUARD` flag
- Ignores regions larger than 100MB to optimize performance

#### Phase 3: Pattern Matching

**Signature search in memory**
- Reads memory in 50MB chunks via `ReadProcessMemory`
- Performs byte-by-byte pattern matching using **private cheat signatures**
- Signatures are exclusive to Novoware and kept private to prevent bypass — they are designed to detect even **self-destructed** instances of the client

#### Phase 4: Classification

**Results:**
- **CLEAN** — No signature detected → legitimate process
- **FUCKED** — One or more signatures found → Novoware client detected in memory

A final summary shows total processes scanned, clean count, and detected count.

---

### Option 2: Journal Trace

This function integrates **JournalTrace** by [spokwn](https://github.com/spokwn/JournalTrace) to perform forensic analysis of the **Windows NTFS journal**, which records file system activity at a low level.

#### What is JournalTrace?

JournalTrace is an external forensic tool that parses the NTFS change journal (`$UsnJrnl`) to reconstruct file system activity. It can surface traces of files that were created, modified, or deleted — even if the cheat has already removed itself from disk.

#### How it works

**Automatic component download**
- Downloads `JournalTrace.exe` (latest release) from spokwn's official GitHub repository
- Saves to temporary directory: `%TEMP%\JournalTraceTool\`
- **If `JournalTrace.exe` already exists in the temp folder, the download is skipped** — no redundant re-downloads

**Overlay hint**
- After launching JournalTrace, the tool renders an animated **fade overlay message** directly on the JournalTrace window: `now filter with .jar to find the replace`
- The overlay uses GDI (`DrawText`, `SetTextColor`) with a pulsing brightness animation for ~7 seconds, then fades out automatically

**NTFS Journal Analysis**

JournalTrace parses the raw NTFS journal and returns a full list of file activity. To find Novoware traces:

> 💡 **Tip**: Filter results by `.jar` and check the dates — this is how you spot if Novoware was replaced or self-destructed.

**Execution**
- Launches JournalTrace with elevated privileges (UAC prompt via `runas`)
- Opens a separate window for detailed output
- The main tool waits for JournalTrace to finish before returning to the menu

---

## System Requirements

### Required Administrative Privileges

The tool requires elevated permissions for:

**Memory Scanner:**
- Access to external process memory
- Direct RAM reading via Windows APIs

**Journal Trace:**
- Raw NTFS journal access
- JournalTrace execution with elevated privileges

### Behavior without Admin

If run without administrative privileges:
- A warning is shown with the option to continue anyway (`Y/N`)
- **Running as administrator is strongly recommended** — without it, memory reads for certain processes may silently fail

---

## Detection Methodology

### Why is memory scanning reliable?

**Private signatures** — detection patterns are exclusive to Novoware and kept private to prevent bypass

**Self-destruct detection** — signatures are designed to catch Novoware even when it attempts to clean itself from memory

**Runtime detection** — detects the cheat only if currently loaded in memory, no false positives from residual files

### Why is Journal Trace analysis important?

**NTFS persistence** — the Windows journal records file activity at the filesystem level, even files that have been deleted leave traces

**Self-destruct bypass** — even if Novoware removes itself from disk, the NTFS journal retains evidence of its presence

---

## Security and Privacy

- **The tool does NOT send data online** — all memory analysis is local
- **The tool does NOT modify memory** — read-only mode only
- **The tool does NOT collect personal information**
- The only external download is JournalTrace, fetched directly from the official [spokwn/JournalTrace](https://github.com/spokwn/JournalTrace) GitHub repository

---

## Contacts

**Creator**: Tonynoh

💬 Discord: `tonyboy90_`

📎 GitHub: [MeowTonynoh](https://github.com/MeowTonynoh)

🎥 YouTube: tonynoh-07

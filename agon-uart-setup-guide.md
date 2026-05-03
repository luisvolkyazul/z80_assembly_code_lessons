# Agon Light 2 UART1 Serial Terminal Setup Guide
## For use with `interrupts_UART_1.asm` lessons

### Prerequisites
- Agon Light 2 with `interrupts_UART_1.asm` compiled and running
- FTDI232 USB-to-TTL adapter (3.3V compatible, **set jumper to 3.3V**)
- 3 female-female dupont wires
- MX Linux PC

---

### 1. Wiring (FTDI232 ↔ Agon Light 2 GPIO)
⚠️ **CRITICAL: Never use 5V mode on FTDI, this will damage the Agon Light 2**
| FTDI Pin | Agon Light 2 GPIO Pin | Function |
|----------|------------------------|----------|
| TX (Orange) | Pin 18 (PC1) | UART1 RxD (Agon receives data from FTDI) |
| RX (Yellow) | Pin 17 (PC0) | UART1 TxD (Agon sends data to FTDI) |
| GND (Black) | Pin 3 or 5 | Ground |
| *Leave VCC (Red) disconnected* | - | Do not power Agon from FTDI |

Agon Light 2 GPIO pin numbering: 34-pin header, viewed from front component side up (Olimex standard). PC0 = Pin 17, PC1 = Pin 18.

---

### 2. MX Linux Setup
#### Install terminal software
```bash
sudo apt update && sudo apt install picocom
```

#### Grant serial port access
Add your user to the `dialout` group (required for serial port permissions):
```bash
sudo usermod -aG dialout $USER
```
*Log out and back in for this change to take effect.*

#### Identify your FTDI device
Plug in the FTDI adapter, then run:
```bash
ls /dev/ttyUSB*
```
Typical device name is `/dev/ttyUSB0` (note this for connection).
I am using /dev/ttyUSB1 for the FTDI232 since my Agon Light 2 microcomputer is on /dev/ttyUSB0.

---

### 3. Connect to Agon via Serial
Use picocom with parameters matching `interrupts_UART_1.asm` config:
- Baud rate: 31250 (alternate: 9600, see asm comment)
- Data bits: 8
- Stop bits: 1
- Parity: None
- Flow control: None

Run:
```bash
picocom -b 31250 -d 8 -p n -s 1 /dev/ttyUSB0
```
Note: Baud rate of 9600 did not work well for me. This 31250 worked fine.

---

### 4. Verify Operation
1. Ensure `interrupts_UART_1.asm` is running on the Agon (shows "Count:" on VGA output)
2. Type keystrokes in the picocom terminal on MX Linux
3. The Agon screen will update the byte count and display received characters

*Note: The Agon only receives data via UART1, so picocom will not show output from the Agon.*

---

### 5. Picocom Quick Reference
| Command | Action |
|---------|--------|
| `Ctrl+A` then `Ctrl+X` | Exit picocom |
| `Ctrl+A` then `Ctrl+T` | Show current settings |
| `Ctrl+A` then `Ctrl+L` | Clear screen |

---

### 6. Troubleshooting
| Issue | Fix |
|-------|-----|
| No `/dev/ttyUSB0` found | Check FTDI is plugged in, run `lsusb` to verify detection |
| Permission denied | Confirm you're in `dialout` group, or run temporarily with `sudo picocom ...` |
| No response from Agon | Check 3.3V jumper on FTDI, verify wiring, try 9600 baud, confirm asm is running |
| Garbled characters | Mismatched baud rate, check FTDI connection |

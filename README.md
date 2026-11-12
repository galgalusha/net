# IOS Setup

1. Install ish shell
2. Open ish and run: `apk add python3 nano`
3. Run `echo bla > udp.py`
4. Run `nano udp.py` 


# Sending UDP Datagram

```python
from socket import *

sock = socket(AF_INET, SOCK_DGRAM)
sock.sendto(b"Hello", ("192.168.28.135", 5151))
```


# UDP Chat

```python
import socket
import threading

DEFAULT_PORT = 5151

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Touch the socket with a dummy connect/disconnect or wait until first sendto
# Note: On Linux/iOS, recvfrom() on a completely uninitialized socket before any sendto()
# may fail with EINVAL because no local port has been assigned yet.
# Sending a 0-byte packet to localhost assigns an ephemeral port immediately without bind():
sock.sendto(b"", ("127.0.0.1", DEFAULT_PORT))

def listen():
    while True:
        try:
            data, addr = sock.recvfrom(1024)
            if data:  # Ignore the initial 0-byte self-kick
                msg = data.decode("utf-8", errors="replace")
                print(f"\n[{addr[0]}:{addr[1]}] {msg}")
        except OSError:
            break

threading.Thread(target=listen, daemon=True).start()

print(f"My ephemeral port: {sock.getsockname()[1]}")
print("Send format: <IP> <message> (e.g., 192.168.28.135 hello world)")

try:
    while True:
        line = input().strip()
        if not line:
            continue

        parts = line.split(maxsplit=1)
        if len(parts) < 2:
            print("Format error. Use: <IP> <message>")
            continue

        target_ip, msg = parts
        try:
            sock.sendto(msg.encode("utf-8"), (target_ip, DEFAULT_PORT))
        except socket.gaierror:
            print(f"Invalid IP address: {target_ip}")
        except OSError as e:
            print(f"Send failed: {e}")
except (KeyboardInterrupt, EOFError):
    print("\nExiting.")
finally:
    sock.close()

```

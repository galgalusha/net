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

PORT = 5151

# Socket 1: Dedicated listener on port 5151
recv_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
recv_sock.bind(("0.0.0.0", PORT))

# Socket 2: Dedicated unbound sender (routes cleanly over Wi-Fi on iOS)
send_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

def listen():
    while True:
        try:
            data, addr = recv_sock.recvfrom(1024)
            msg = data.decode("utf-8", errors="replace")
            print(f"\n[{addr[0]}:{addr[1]}] {msg}")
        except OSError:
            break

threading.Thread(target=listen, daemon=True).start()

print(f"Listening on port {PORT}.")
print("Send format: <IP> <message> (e.g., 192.168.28.135 hello)")

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
            send_sock.sendto(msg.encode("utf-8"), (target_ip, PORT))
        except (socket.gaierror, OSError) as e:
            print(f"Send failed: {e}")
except (KeyboardInterrupt, EOFError):
    print("\nExiting.")
finally:
    recv_sock.close()
    send_sock.close()

```

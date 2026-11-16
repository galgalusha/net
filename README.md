# Sending UDP Datagram

```python
from socket import *

sock = socket(AF_INET, SOCK_DGRAM)
sock.sendto(b"Hello", ("192.168.28.135", 5151))
```

# IOS Setup

1. Install ish shell
2. Open ish and run: `apk add python3 nano`
3. Run `echo > udp.py`
4. Run `nano udp.py`
5. paste the code
6. Click CTRL+O to save and CTRL+X to exit nano
7. Run `python3 udp.py`

<img width="238" height="335" alt="image" src="https://github.com/user-attachments/assets/ad523ea0-1309-4aa2-b47a-33d50cfcd347" />

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

# HTTP

```py
import socket
from urllib.parse import quote_plus

HOST = "192.168.1.100"
PORT = 8080

name = input("Name: ")
animal = input("Favorite animal: ")
body = f"name={quote_plus(name)}&animal={quote_plus(animal)}"

req = (
    f"POST / HTTP/1.1\r\n"
    f"Host: {HOST}\r\n"
    f"Content-Type: application/x-www-form-urlencoded\r\n"
    f"Content-Length: {len(body.encode())}\r\n"
    f"Connection: close\r\n\r\n"
    f"{body}"
)

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))
s.sendall(req.encode())
print(s.recv(4096).decode())
s.close()
```

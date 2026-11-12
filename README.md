# Sending UDP Datagram

```python
from socket import *

sock = socket(AF_INET, SOCK_DGRAM)
sock.sendto(b"Hello", ("192.168.28.135", 5151))
```


# UDP Chat

```python
from socket import *
from threading import *

PORT = 5151

sock = socket(AF_INET, SOCK_DGRAM)
sock.bind(("0.0.0.0", PORT))

def listen():
    while True:
          data, addr = sock.recvfrom(PORT )
          msg = data.decode("utf-8", errors="replace")
          print(f"\n[{addr[0]}:{addr[1]}] {msg}")

Thread(target=listen, daemon=True).start()

print(f"Listening on port {PORT}.")
print("Send format: <IP> <message> (e.g., 192.168.28.0 hello world)")


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
        sock.sendto(msg.encode("utf-8"), (target_ip, PORT))
    except gaierror:
        print(f"Invalid IP address: {target_ip}")
    except OSError as e:
        print(f"Send failed: {e}")

```

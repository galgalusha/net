# IOS Setup

1. Install ish shell
2. Open ish and run: `apk add python3 nano`
3. Run `echo > udp.py`
4. Run `nano udp.py`
5. paste the code
6. Click CTRL+O to save and CTRL+X to exit nano
7. Run `python3 udp.py`

<img width="238" height="335" alt="image" src="https://github.com/user-attachments/assets/ad523ea0-1309-4aa2-b47a-33d50cfcd347" />

# Android Setup
Install `Pydroid 3`

# Sending UDP Datagram

```python
from socket import *

sock = socket(AF_INET, SOCK_DGRAM)
sock.sendto(b"Hello", ("192.168.28.135", 5151))
```

# UDP Chat (Send and Receive)

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

# HTTP GET
```py
HOST = '192.168.28.103'
PATH = '/'
BUFFER_SIZE = 1000

import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, 80))
sock.sendall(f"GET {PATH} HTTP/1.1\r\nHost: {HOST}\r\nConnection: close\r\n\r\n".encode())

response = b""
while chunk := sock.recv(BUFFER_SIZE):
    response += chunk
sock.close()

print(response.decode("utf-8"))
```
# HTTP POST

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

# HTTP Server (for lecturer leptop)
```py
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import parse_qs

HTML_FORM = """<!DOCTYPE html>
<html>
<body style="font-family: sans-serif; margin: 40px;">
  <h2>Networks Lab - POST Test</h2>
  <form method="POST" action="/">
    Your name: <input type="text" name="name" required /> <br><br>
    Your favorite animal: <input type="text" name="animal" required /> <br><br>
    <input type="submit" value="Submit" />
  </form>
</body>
</html>
"""

class LabHandler(BaseHTTPRequestHandler):
    def log_message(self, format, *args):
        # Silence default noisy HTTP request logs
        pass

    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/html; charset=utf-8")
        self.end_headers()
        self.wfile.write(HTML_FORM.encode("utf-8"))

    def do_POST(self):
        content_length = int(self.headers.get("Content-Length", 0))
        raw_body = self.rfile.read(content_length).decode("utf-8", errors="replace")
        fields = parse_qs(raw_body)

        user_agent = self.headers.get("User-Agent", "")
        is_browser = "Mozilla/" in user_agent

        name = fields.get("name", ["(missing)"])[0]
        animal = fields.get("animal", ["(missing)"])[0]
        source = "REAL BROWSER" if is_browser else "PYTHON SCRIPT"

        print(f"[{self.client_address[0]}] [{source}] Name: {name} | Animal: {animal}")
        if not is_browser and user_agent:
            print(f"   -> User-Agent: {user_agent}")

        self.send_response(200)
        self.send_header("Content-Type", "text/plain; charset=utf-8")
        self.end_headers()
        self.wfile.write(f"Received from {source}: {name} likes {animal}\n".encode("utf-8"))

if __name__ == "__main__":
    server = HTTPServer(("0.0.0.0", 8080), LabHandler)
    print("Listening on port 8080...")
    server.serve_forever()
```

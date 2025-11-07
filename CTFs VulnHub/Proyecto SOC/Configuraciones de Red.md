1 - Snort ademas de ser nuestro principal punto de entrada debe tener la capacidad de actuar enrunando los paquetes entre las redes que se vincula a el por lo que sera necesario activar un propiedad en su configuración conocida como port forwarding que permitirá el enrrutado de paquetes entre interfaces. 

```
Revisamos si esta activo

cat /proc/sys/net/ipv4/ip_forward    # 0 || 1

nano /etc/sysctl.conf     #Editamos el fichero 
net.ipv4.ip_forward = 1   #Lo activa permanente
sysctl -p                 #Aplicar la configuracion 
```


Corrosion2
Configuramos la ruta de salida para que tenga acceso a la red externa simulando asi la salida a Internet que seria la red 10.10.11.0/24
```
nmcli connection show

nmcli -t -f NAME connection show

nmcli connection show "Wired connection 1"

nmcli connection modify ens33 +ipv4.routes "10.10.11.0/24 10.10.12.129"
```

Muy similar lo haríamos en el SOC a diferencia de que este corre un sistema Amazon Linux que no tiene nmcli por lo que se configura de la siguiente manera. 
Nos creamos un fichero de ruta con el nombre de la interface que tendrá la salida
y le agregaremos la ruta
```
sudo nano /usr/local/sbin/add-static-route.sh

#!/bin/bash
sleep 5
ip route replace 10.10.11.0/24 via 10.10.12.129 dev eth0

sudo chmod +x /usr/local/sbin/add-static-route.sh

sudo nano /etc/systemd/system/static-route.service

[Unit]
Description=Add static route at boot
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/add-static-route.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target


sudo systemctl daemon-reload
sudo systemctl restart static-route.service
sudo systemctl status static-route.service

```
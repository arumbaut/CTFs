1 - Instalar Suricata en Ubuntu Server
```
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```

2- Poner la interface donde se registraran los paquetes en modo promiscuo
```
sudo ip link set vmnet80 promisc on
```

3 - Descargar y extraer el conjunto de reglas de Emerging Threats Suricata
```
mkdir -p /etc/suricata/rules/
cd /tmp/ && curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz && sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 755 /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules

```

4 - Modifique la configuración de Suricata en el archivo/etc/suricata/suricata.yaml
```
HOME_NET: "<UBUNTU_IP>"  #IP que se van a monitorear
EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules
rule-files:
- "*.rules"

# Global stats configuration
stats:
enabled: Yes

# Linux high speed capture support
af-packet:
  - interface: eth0
```

5- Reiniciar el servicio de Suricata
```
sudo systemctl restart suricata
```

Instalar Wazuh
Lo haremos descargandonos una Maquina Virtual de su sitio oficial que ya esta preconfigurada y lo ejecuaremos en nuestra red.
Luego haremos un despliegue de agentes en nuestras maquinas a monitorer incluido nuestro IDS para registrar los logs que este genere para porder generar las alertas. Mas configuraciones [[Wazuh Configuraciones]]

Para que funcione correcamente nuestra topoligia debemos hacer unos ajustes en la configuracion de rede de los diferetes actores que intervienen lo que esa explicado en 
[[Configuraciones de Red]]

Resolución de las maquinas en las carpetas [Corrosion2] y
[[Corrosion1]]
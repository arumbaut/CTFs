Instalar Suricata 

Nos apoyamos en este repo de github

http://github.com/0xrajneesh/Suricata-IDS-Home-Lab/blob/main/installing-suricata.md

Nos instalamos el agente de Wazuh en la maquina donde instalamos el suricata para enviar estos logs a Wazuh para poder gestionarlos. Configuramos el envio de los logs a Wazuh

Hay que tner la interface de Suricata en modo promiscuo para que inspeccione toda la red, y ademas con un port mirrorin enviar todo el trafico a la interface de suricata

```
sudo ip link set vmnet80 promisc on

```

Wazuh da un error al leer el eve.json de suricata por lo cual hay que configurarle un parametro para que lo lea correctamente
```
nano /var/ossec/etc/internal_options.conf

# Maximum number of fields in a decoder (order tag) [32..1024]
#analysisd.decoder_order_size=255  Valor por defecto

analysisd.decoder_order_size=1024

```


```
nano /var/ossec/etc/ossec.conf

 <!-- Log analysis -->
 <ossec_config>
  <localfile>
    <log_format>journald</log_format>
    <location>journald</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/ossec/logs/active-responses.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/dpkg.log</location>
  </localfile>

 <!-- Agregamos los log que queremos monitorizar de suricata -->	
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>

</ossec_config>

Cerramos y reiniciamos el agente 

systemctl restart wazuh-agent

El directorio /var/log/suricata contiene los diferentes logs que almacena suricata pero eve.json es como una acumulacion de todos

Para ver un tipo de log en tiempo real utilizamos

tail -f /var/log/suricata/suricata.log
tail -f /var/log/suricata/fast.log



```
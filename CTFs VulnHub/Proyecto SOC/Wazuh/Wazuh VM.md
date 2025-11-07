
Para la instalacion de la maquina virtual 
Link 
https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html

Para cambiar la pass de admin

Acceder a esta direccion como root
```
cd /usr/share/wazuh-indexer/plugins/opensearch-security/tools/
```

Utilizar este comando. Tienes que cumplir bien los crierios de pass si no da problemas para el cambio 
```
bash wazuh-passwords-tool.sh -u USER -p PASS
```

Reiniciar el sericio para que funcione todo correctamente
```
systemctl restart filebeat.service

systemctl status filebeat.service
```

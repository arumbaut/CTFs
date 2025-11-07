File Integrity

Primero habilitamos en el servidor la File Integrity

```
nano /var/ossec/etc/ossec.conf

  <logall>yes</logall>
  
systemctl restart wazuh-manager.service  

```

En los clientes configuramos los ficheros que queremos monitorizar

```
nano /var/ossec/etc/ossec.conf

En la seccion
  <!-- File integrity monitoring -->
  
    <disabled>no</disabled>
    
Estructura para monitorizar un directori especifico
<directories>/etc,/usr/bin,/usr/sbin</directories>  
<directories check_all="yes" report_changes="yes" realtime="yes">/root</directories>  


systemctl status wazuh-agent
```
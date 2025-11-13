Utilizar samba para Dumpear la SAM
Intalamos esta herramienta para poder utilizar e identificar si samba esta firmado o no

Instalacion
```
python3 -m venv ~/cme-venv
source ~/cme-venv/bin/activate
cd cme-venv/

git clone https://github.com/byt3bl33d3r/CrackMapExec.git
cd CrackMapExec/
pip install -e .

deactivate 

sudo tee /usr/local/bin/cme > /dev/null <<'EOF'

Se agrega dentro del archivo cme
#!/usr/bin/env bash
# Ruta al entorno virtual
VENV="$HOME/Tools/cme-env"

# Comprobar que el binario exista
if [ ! -x "$VENV/bin/cme" ]; then
  echo "Error: no se encontró $VENV/bin/cme o no es ejecutable." >&2
  exit 1
fi

# Ejecutar directamente el binario de cme dentro del venv
exec "$VENV/bin/cme" "$@"

Le damos permisos
sudo chmod +x /usr/local/bin/cme
```

Ya podremos ejecutarlo desde nuestra terminal sin necesidad de correr en enviroment de python


```
Nos da si el SMB esta firmado o no ademas que enumera los equipos de la red por samba. Funciona si esta desactivado el firewal de windows

cme smb 10.10.11.0/24
SMB         10.10.11.131    445    PC-ALEX          [*] Windows 10.0 Build 19041 x64 (name:PC-ALEX) (domain:savicorp.local) (signing:False) (SMBv1:False)
SMB         10.10.11.130    445    DC-COMPANY       [*] Windows Server 2016 Essentials 14393 x64 (name:DC-COMPANY) (domain:savicorp.local) (signing:True) (SMBv1:True)


Teniendo usurio y password de un usurio pordemos ejecutar este comando y si el usuario tiene Privilegios elevados nos porndra  (Pwn3d!) de lo contrario no tiene privilegios


$cme smb 10.10.11.0/24 -u "alex" -p "P@ssword1"
SMB         10.10.11.130    445    DC-COMPANY       [*] Windows Server 2016 Essentials 14393 x64 (name:DC-COMPANY) (domain:savicorp.local) (signing:True) (SMBv1:True)
SMB         10.10.11.131    445    PC-ALEX          [*] Windows 10.0 Build 19041 x64 (name:PC-ALEX) (domain:savicorp.local) (signing:False) (SMBv1:False)
SMB         10.10.11.130    445    DC-COMPANY       [+] savicorp.local\alex:P@ssword1 
SMB         10.10.11.131    445    PC-ALEX          [+] savicorp.local\alex:P@ssword1 (Pwn3d!)


cme smb 10.10.11.0/24 -u "alex" -p "P@ssword1"
SMB         10.10.11.130    445    DC-COMPANY       [*] Windows Server 2016 Essentials 14393 x64 (name:DC-COMPANY) (domain:savicorp.local) (signing:True) (SMBv1:True)
SMB         10.10.11.131    445    PC-ALEX          [*] Windows 10.0 Build 19041 x64 (name:PC-ALEX) (domain:savicorp.local) (signing:False) (SMBv1:False)
SMB         10.10.11.131    445    PC-ALEX          [+] savicorp.local\alex:P@ssword1 
SMB         10.10.11.130    445    DC-COMPANY       [+] savicorp.local\alex:P@ssword1 
Running CME against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00

```

Si tenemos un usuario con privilegios podemos ejecutar varios tecnicas como dumpear la SAM ademas de hacer ejecución de código en una maquina que el usuario que obtenido tenga privilegios de administrador, como lo hacemos ---->

[[Ejecucion de codigo mediante SAMBA]]

Buscar informacion
https://pwncat.readthedocs.io/en/latest/

Instalarlo

```
sudo apt update
sudo apt install -y pipx python3-venv

# Asegura que pipx ponga sus binarios en PATH
python3 -m pipx ensurepath

# Cierra y vuelve a abrir la terminal, o:
source ~/.profile 2>/dev/null || true

# Instala pwncat-cs en su propio entorno gestionado por pipx
pipx install pwncat-cs

# Comprobar
command -v pwncat-cs
pwncat-cs --help

```

Se ejecuta similar a netcat
```
pwncat-cs -lp 443

Luego damos un back y ya estamos en la shell del objetivo 
dentro de esta tenemos una ayuda help que nos brinda varias cosas como enumeracion con el comando 
run enumerate.
```
Buscar en la salida de ayuda de algun comando

Coincidencia exacta (línea completa), salida combinada stdout+stderr, búsqueda literal
```
gobuster dir -h 2>&1 | grep -xF -- 'Usage: gobuster dir [options]'

```

Igual que anterior, pero insensible a mayúsculas
```
gobuster dir -h 2>&1 | grep -ixF -- 'usage: gobuster dir [options]'

```
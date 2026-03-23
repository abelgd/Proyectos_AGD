# Writeup — Diario del Atacante

**Categoría:** Forense | **Dificultad:** Media | **Puntos:** 50
**Flag:** `CTF{ch4rm_h1st0ry_n3v3r_l135}`

---

## Fase 1 — Identificar el archivo

El archivo `History` tiene la extensión `.txt`. Al abrirlo con un editor muestra `SQLite format 3` al inicio.

**Solución:** Renombrar a `History.db` y abrir con **DB Browser for SQLite**.

![Imagen1](img/archivo-db.png)

> Si no se dispone de la herramienta DB Browser for SQLite descargar desde la siguiente url: https://sqlitebrowser.org/dl/

---

## Fase 2 — Explorar la tabla `urls`

Abrimos la herramienta de DB Browser for SQLite y clickamos en opendatabase y seleccionamos el archivo `History.db`

![Imagen2](img/open-db.png)

A continuacion, con la base de datos cargada, hacemos click en pestaña **"Hoja de datos"** → tabla **`urls`**.

![Imagen3](img/hoja-urls.png)

Mirando los registros se observa actividad totalmente normal pero la fila 14 llama la atención inmediatamente: el título de la página dice literalmente **"CODIGO SECRETO:"** seguido de una cadena en Base64.

![Imagen4](img/cod14.png)
![Imagen5](img/codigo.png)

---

## Fase 3 — Decodificar la flag

La cadena `Q1RGe2NoNHJtX2gxc3QwcnlfbjN2M3JfbDEzNX0=` tiene el padding `=` característico de Base64.

Desde la pag web https://www.base64decode.org/ pegamos la cadena y nos da el resultado inmediatamente:

![Imagen6](img/flag.png)

**Resultado:**
```
CTF{ch4rm_h1st0ry_n3v3r_l135}
```

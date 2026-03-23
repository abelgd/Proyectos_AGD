# Write-up: Sombra caída - CTF Challenge

## Descripción

Se nos presenta un escenario donde el agente "SOMBRA-7" ha desertado con datos confidenciales. Nuestro objetivo es descifrar la contraseña de 6 caracteres que protege su equipo cifrado utilizando dos piezas de evidencia:

1. Un mensaje interceptado (cifrado)
2. Un pergamino con un poema

---

## Reconocimiento inicial

### Archivo 1: Comunicación interceptada

```
Yr cevzre irefb rf yn yynir. 
Yb dhf ivrar qrfchéf rf ry pnzvab.
Frvf yrgenf, han ibyhagnq.
```

**Observaciones:**
- Texto aparentemente en español, pero con letras alteradas
- No tiene sentido gramatical directo
- Pista del analista menciona: *"Trece pasos hacia la verdad, trece pasos de vuelta"*

### Archivo 2: El Pergamino

```
La noche cerrada sin luna que alumbrar,
Apunta al cielo en días de calma total,
Cubre prados frescos al alba del rocío,
Destruye con calor que quema sin piedad,
Ilumina el mundo cuando amanece el día,
Tiñe de otoño las hojas del nogal.
```

**Observaciones:**
- Poema de 6 versos
- Cada verso describe algo diferente
- Firmado con "Sello: S7" (SOMBRA-7)

---

## Solución

### Paso 1: Descifrar el mensaje ROT13

La pista clave está en la frase del analista: *"Trece pasos hacia la verdad, trece pasos de vuelta"*

Esto sugiere el cifrado **ROT13** (Rotate 13), un cifrado César con desplazamiento de 13 posiciones.

Usamos una herramienta online como [rot13.com](https://rot13.com) o el comando bash:

```bash
echo "Yr cevzre irefb rf yn yynir." | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**Resultado descifrado:**

```
La primer verso es la llave.
Lo que viene después es el camino.
Seis letras, una voluntad.
```

**Interpretación:**
-  El primer elemento de cada verso es importante
-  Son 6 letras en total (6 versos)
-  Estas letras forman la contraseña

---

### Paso 2: Analizar el pergamino

Ahora que sabemos que debemos extraer algo del pergamino, analizamos cada verso:

#### Verso 1: "La noche cerrada sin luna que alumbrar"
- Describe: **Oscuridad total, ausencia de luz**
- Concepto: **NEGRO**

#### Verso 2: "Apunta al cielo en días de calma total"
- Describe: **El cielo despejado**
- Concepto: **AZUL**

#### Verso 3: "Cubre prados frescos al alba del rocío"
- Describe: **Prados, hierba, vegetación**
- Concepto: **VERDE**

#### Verso 4: "Destruye con calor que quema sin piedad"
- Describe: **Fuego, llamas**
- Concepto: **ROJO**

#### Verso 5: "Ilumina el mundo cuando amanece el día"
- Describe: **El sol al amanecer**
- Concepto: **AMARILLO**

#### Verso 6: "Tiñe de otoño las hojas del nogal"
- Describe: **Hojas secas de otoño**
- Concepto: **MARRÓN**

---

### Paso 3: Extraer el acróstico

Siguiendo las instrucciones del mensaje descifrado: *"La primer verso es la llave"*

Tomamos la **primera letra** de cada color identificado:

```
Negro    → N
Azul     → A
Verde    → V
Rojo     → R
Amarillo → A
Marrón   → M
```

**Contraseña obtenida:** `NAVRAM`

---

### Paso 4: Construir la FLAG

Formato indicado: La FLAG sirve para acceder al equipo del agente.

**FLAG:** `FLAG{NAVRAM}`

---

## Herramientas utilizadas

- **ROT13 Decoder**: https://rot13.com
- **Comando bash**: `tr 'A-Za-z' 'N-ZA-Mn-za-m'`
- Análisis manual de texto

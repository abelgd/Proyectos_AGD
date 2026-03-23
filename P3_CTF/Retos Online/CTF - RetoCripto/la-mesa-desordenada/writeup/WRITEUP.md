# Write-Up — La mesa desordenada

## Descripción
Se proporcionan dos archivos: `README.md` con el contexto de una
investigación criminal de un varón de 38 años de Lebrija (Sevilla)
sospechoso de narcotráfico, y `E-01.png`, una fotografía recuperada
en el registro domiciliario. El propio README indica que ambos archivos
pueden contener información relevante pendiente de análisis forense.

## Solución
El README nos pone sobre la pista: tanto la imagen como el propio
documento pueden contener información útil. Al examinar `E-01.png`,
entre los objetos de la mesa se aprecia una nota manuscrita con una
lista de la compra. Tomando la inicial de cada elemento en orden vertical:

**F**ruta · **L**echuga · **A**gua · **G**arbanzos · **S**al · **H**uevos
· **O**reo · **P**an de molde · **P**atatas · **I**sotónica · **N**atillas · **G**el

## Flag
```
flag{shopping}
```
# Vidas Marinas (Costa Rica Azul)

Sitio estático de exploración de la vida marina por profundidad. Los datos viven en el arreglo `marineData` dentro de `es/index.html` y `en/index.html` (contenido duplicado por idioma).

---

## Cómo clonar el repositorio y trabajar en equipo

### Clonar

```bash
git clone https://github.com/MauricioBuzz98/vidas_marinas.git
cd vidas_marinas
```

### Flujo de trabajo con ramas

Cada colaborador trabaja en **su propia rama**; no se comitea directo en la rama principal de integración.

1. Asegurarte de tener la última base:
   ```bash
   git fetch origin
   git checkout main
   git pull origin main
   ```

2. Crear una rama con el formato acordado:
   ```bash
   git checkout -b "fix/<pequeña-descripción>/<id-task-hubstaff>"
   ```

   ```text
   fix/<pequeña-descripción>/<id-task-hubstaff>
   ```
   Ejemplos:
   - `fix/agregar-vineta-manati/1234567`
   - `fix/corregir-texto-epipelagico/1234568`

3. Trabajar, hacer commits claros y subir la rama:
   ```bash
   git push -u origin fix/agregar-vineta-manati/1234567
   ```

4. Abrir un **merge request** o **pull request** hacia la rama principal y revisar antes de integrar.

### Notas

- Los cambios en datos o copy suelen requerir **actualizar en paralelo** `es/index.html` y `en/index.html` para no desalinear idiomas.
- Las imágenes se referencian con el nombre de archivo; el despliegue actual usa la base `https://esencialcostarica.com/costaricaazul/uploads/`.

---

## Estructura de `marineData`

`marineData` es un arreglo de objetos. El script recorre los ítems **ordenados por `depth`**. La rama de renderizado es:

1. Si tiene `zone: true` → se dibuja un **bloque de zona** (título, opcional `zoneInfo`, descripción).
2. Si no, si tiene `monte: true` → se dibuja un **monte submarino** (risco, textos, profundidades).
3. Si no, si `item.image` es **verdadero** (cadena no vacía) → se dibuja una **ficha de especie o mensaje** con imagen.

**Importante:** un ítem que no sea `zone` ni `monte` y tenga `image: ""` **no se renderiza** (el código solo entra en la rama de ítems con imagen si `item.image` existe y no es vacío).

### Tipos de ítems y cuándo usarlos

| Tipo | Cómo marcarlo | Uso |
|------|---------------|-----|
| **Zona batimétrica** | `zone: true` | Encabezados de sección (p. ej. “Zona epipelágica”): título en `name`, datos técnicos en `zoneInfo`, narrativa en `info` (admite HTML). Puede ir con `image: ""`. |
| **Monte submarino** | `monte: true` | Formaciones grandes (cima en `depth`, base opcional en `base`, texto en `info`, `ubicacion` opcional, `image` del risco). |
| **Especie u objeto con ilustración** | Sin `zone` ni `monte`; `image` con nombre de archivo | Lo habitual: peces, invertebrados, etc. `depth` = profundidad de referencia. |
| **Mensaje o cartela especial** | `mensaje: true` + `image` (obligatorio para que aparezca) | Textos didácticos o piezas con estilo `marine-item-mensaje` (carga de imagen prioritaria). Puede combinarse con `monte: true` en casos como el domo térmico. |

### Propiedades comunes (referencia)

- **`depth`**: número; determina orden vertical y posición aproximada.
- **`name`**: título. Si el nombre encaja con el patrón `Nombre (Subtítulo)`, el paréntesis se muestra como subtítulo.
- **`image`**: solo el nombre del archivo en el CDN; cadena **no vacía** salvo en ítems `zone` puros.
- **`info`**: descripción; HTML permitido donde ya se use en el proyecto.
- **`size_cm`**: rango en cm, p. ej. `"5-10"`; afecta el tamaño visual de la ficha.
- **`conservation_status`**: ver tabla bilingüe más abajo; cadena vacía `""` si no aplica.
- **`endemica`**: `0` o `1`; con `1` se muestra la insignia de endemismo (texto en español o inglés según el `index.html` editado).
- **`slug`**: identificador kebab-case (opcional en datos; conviene mantenerlo alineado entre ES/EN por si se usa en futuros enlaces o integraciones).

### Propiedades solo de **zona** (`zone: true`)

- **`zoneInfo`**: línea de metadatos (rango de profundidad, temperatura, presión, etc.).

### Propiedades solo de **monte** (`monte: true`)

- **`base`**: profundidad de la base en metros, o `""` si no se muestra.
- **`ubicacion`**: texto de ubicación con icono de mapa.
- **`ocultar_cima_monte`**: si es verdadero, no se muestra la línea de cima/profundidad (útil p. ej. en el domo térmico).
- **`monte_sombra_suave`**: modifica el estilo del risco (sombra suave).

---

## Estados de conservación (ES y EN)

El estilo (color de la etiqueta e imagen) lo calcula `getConservationClass` buscando **subcadenas** en el texto (en minúsculas). Conviene usar **una de las cadenas canónicas** de cada fila para ES y EN, según el archivo que edites.

| Clase CSS (resultado) | Español (sugerido en `es/index.html`) | Inglés (sugerido en `en/index.html`) |
|------------------------|----------------------------------------|--------------------------------------|
| Vulnerable | Vulnerable | Vulnerable |
| En peligro crítico / Critically Endangered | En peligro crítico de extinción o En peligro crítico | Critically endangered (también reconoce "Critically Endangered") |
| En peligro / Endangered | En peligro de extinción | Endangered |
| Amenazada / Threatened | Amenazada | Threatened (diferente de *Endangered*, que aplica a “en peligro de extinción” / *Endangered*) |
| Casi amenazado / Near threatened | Casi amenazado | Near threatened |
| Especie invasora | Especie Invasora | Invasive Species (cualquier texto con “invasora” o “invasive”) |
| Preocupación menor / Least concern | Especie de preocupación menor | Least concern (cualquier texto con “preocupación menor” o “least concern”) |

**Orden lógico en el código:** primero se detecta *Vulnerable*; luego *en peligro crítico*; después *en peligro* (cubre “En peligro de extinción”); luego *amenazada/threatened*; etc. Evita mezclar términos que contengan “en peligro” con otros niveles de forma ambigua; si no coincide ningún patrón, el estado se muestra como texto pero **sin** clase de color especial (`conservationClass` nulo).

Para especies **sin** categoría UICN o equivalente, deja `conservation_status: ""`.

---

## Archivos clave

- `es/index.html` — versión en español (`marineData` y lógica de la página).
- `en/index.html` — versión en inglés; mantener estructura de ítems y profundidades coherente con la ES al añadir especies.

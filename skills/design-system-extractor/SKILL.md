---
name: design-system-extractor
description: Examina una carpeta /design del proyecto para extraer la identidad visual y generar un manual de construcción (DESIGN_SYSTEM.md) que permita crear nuevas vistas desde cero con el mismo estilo y stack técnico. Usa esta skill cuando el usuario quiera analizar mockups, vistas HTML o capturas para extraer patrones visuales, crear guías de estilo, documentar un design system o unificar identidad visual. También cuando mencione carpetas /design, screen.png, mockups, o diga "revisa mis diseños", "sigue este estilo", "hazme una vista igual", o "crea una pantalla nueva para mi proyecto". No es solo documentación — es la receta para replicar el diseño.
---

# Design System Extractor

Genera un `DESIGN_SYSTEM.md` analizando la carpeta `/design` del proyecto (cada módulo tiene `code.html` + `screen.png`). El resultado es un manual de construcción que permite crear vistas nuevas idénticas en estilo y stack.

## Principio rector: Fidelidad al stack

Las nuevas vistas usan EXACTAMENTE las mismas tecnologías, librerías, CDNs (mismas URLs y versiones), fuentes e iconos del proyecto. No se introduce nada nuevo ni se reemplaza nada existente. El DESIGN_SYSTEM.md documenta un **stack obligatorio** y una lista de **tecnologías prohibidas**.

## Flujo de trabajo

### Paso 1: Descubrimiento

```bash
find /design -type f \( -name "code.html" -o -name "screen.png" \) | sort
```
Informar al usuario cuántos módulos hay y cuáles son. Si la estructura varía, adaptar.

### Paso 2: Análisis de cada módulo

**Del código (`code.html`) extraer con valores exactos:**

1. **Stack técnico** — Cada `<link>` y `<script>` del `<head>` (URLs textuales), `tailwind.config` completo si aplica, Google Fonts con pesos, librería de iconos con parámetros, estilos globales en `<style>`. Todo esto es el stack obligatorio que se copia idéntico.

2. **Tokens** — Colores (nombre semántico + hex + rol), tipografía (familia + peso + tamaño por elemento), espaciado (escala usada), bordes (grosor, color, radius por componente), sombras (valores exactos por intensidad), iconografía (librería, estilo, tamaño).

3. **Componentes** — Para cada uno (botones, cards, inputs, nav, footer, badges, etc.): estructura HTML con clases exactas, variantes, estados interactivos (hover/focus/active/disabled), snippet copiable funcional.

4. **Secciones** — Cada tipo de sección (hero, productos, testimonios, features, footer): layout (grid/flex, columnas), composición interna (qué componentes y en qué orden), decoración (fondos con opacidad, orbs, separadores).

5. **Layout general** — Estructura de página, ancho máximo, paddings responsive, patrón de navegación.

**De la captura (`screen.png`) complementar:**
Personalidad/tono visual, jerarquía (qué atrae primero), densidad de info, detalles decorativos que dan carácter. **Si código e imagen difieren, la imagen manda.**

### Paso 3: Síntesis (si hay múltiples módulos)

Identificar patrones consistentes (= el sistema), inconsistencias (señalar cuál debería ser el estándar), y destilar la filosofía visual en 2-3 frases concretas con valores (no "diseño moderno" sino "paleta turquesa-coral-mostaza, esquinas rounded-2xl/3xl, Montserrat bold para títulos + Lexend para body").

### Paso 4: Generar DESIGN_SYSTEM.md

Estructura del documento:

```
# Design System — [Nombre]

> [Filosofía visual: 2-3 frases con valores concretos]

## 1. Stack obligatorio

### Dependencias
| Tipo | Nombre | URL exacta |
[Tabla con TODAS las dependencias]

### Bloque <head> completo (copiar tal cual a cada vista nueva)
```html
[<head> íntegro del proyecto]
```

### Tailwind config (o equivalente)
```javascript
[Config completo]
```

### Estilos base
```css
[Estilos globales]
```

### Tecnologías prohibidas
[Lista de lo que NO debe introducirse: otros frameworks, otras fuentes, otros iconos, etc.]

## 2. Tokens de diseño

### Colores
| Nombre | Clase/variable | Hex | Uso |

### Tipografía
| Rol | Familia | Peso | Tamaño | Cuándo usarla |

### Espaciado, bordes, sombras, iconografía
[Tablas con valores exactos]

## 3. Blueprints de componentes

Para cada componente:
- Snippet HTML copiable con clases reales
- Variantes y estados
- Notas de uso

[Botones, Cards, Header, Footer, Badges, Forms, etc.]

## 4. Blueprints de secciones

Para cada tipo de sección:
- Layout y composición
- Qué componentes usa
- Esqueleto HTML con comentarios

[Hero, Productos, Testimonios, Features, Footer, etc.]

## 5. Receta para vista nueva

1. Copiar boilerplate del <head> exacto
2. Verificar stack — solo tecnologías del proyecto
3. Armar esqueleto: Header → Secciones → Footer
4. Elegir secciones de blueprints
5. Rellenar con componentes de blueprints
6. Usar solo tokens documentados
7. Revisar contra reglas de oro

## 6. Reglas de oro

[5-10 reglas ultra-específicas, cada una con:]
- ✅ Qué hacer (valores exactos / clases)
- ❌ Qué evitar

## 7. Anti-patrones

### Violaciones de stack (las más graves)
[Qué tecnologías no introducir y por qué]

### Violaciones de estilo
[Qué decisiones visuales evitar]
```

### Paso 5: Validación

Preguntar al usuario: ¿refleja la identidad visual? ¿Falta algún componente? ¿Ajustar reglas?

## Criterios de calidad

El documento está completo si:
1. Un dev nuevo puede crear una vista solo con este documento
2. Los snippets funcionan al copiar y pegar con el boilerplate
3. 3 devs crearían vistas que se ven del mismo proyecto
4. No hay frases ambiguas sin valores exactos
5. Queda claro qué tecnologías se usan y cuáles están prohibidas

## Notas

- Con un solo módulo, se usa como única fuente de verdad.
- Si hay inconsistencias entre módulos, sugerir el estándar más completo/frecuente.
- Guardar donde indique el usuario, o junto a /design por defecto.
- Si se necesita funcionalidad no cubierta por el stack, resolver con CSS/JS vanilla o lo que el proyecto ya tenga — nunca agregar dependencias sin autorización.

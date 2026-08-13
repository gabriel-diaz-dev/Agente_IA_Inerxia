# INFORME DE HALLAZGOS — Análisis del dataset de 4.017 llamadas

> Fecha: 13 de agosto de 2026. Fuente: `prueba-2-retencion-7x-1-04-08-26-report.xlsx` (4.017 registros, 2.843 completadas, 1.174 fallidas).

---

## Metodología

1. **Análisis programático** (Python + openpyxl): detección de patrones de malas prácticas, inconsistencias y falta de protocolo sobre las transcripciones y campos de análisis de las 4.017 filas.
2. **Muestra profunda**: revisión manual de las transcripciones completas de los casos más representativos de cada patrón.
3. **Acción**: cada hallazgo se mapea a una mejora existente del prompt v3 (ya cubierta) o a una mejora nueva añadida (`[MEJORA P11]`-`[MEJORA P15]`).

---

## Estadísticas generales

| Métrica | Valor |
|---|---|
| Llamadas totales | 4.017 |
| Completadas | 2.843 (70,8%) |
| Fallidas | 1.174 (29,2%) |
| `user_hangup` | 1.985 (49,4% del total) |
| `agent_hangup` | 673 |
| Voicemail alcanzado | 880 |
| Sentiment negativo con interacción | 87 |
| `objetivo_cumplido=Yes` sin `recargo=Yes` (falsos positivos) | 269 |
| `interes_mostrado=Yes` sin recarga | 121 |
| Llamadas < 15 s con interacción | 939 |

---

## Hallazgos detectados

| # | Hallazgo | Ocurrencias | Severidad | Mejora asociada |
|---|---|---|---|---|
| H1 | **Fuga de variables internas en voz**: el agente dice "Resultado: NO RECARGÓ...", "Resumen: ..." al cliente | 16 | CRÍTICA | **P11 (nueva)** |
| H2 | **"¿Sigues en línea?" repetido ≥2 veces** en la misma llamada | 612 | ALTA | E3 (existente) |
| H3 | "¿Sigues en línea?" repetido ≥3 veces | 344 | ALTA | E3 (existente) |
| H4 | **Voicemail sin mensaje con marca** ("lo llamaremos más adelante" o vacío) | 923 de 926 | ALTA | V2 (existente) |
| H5 | **Agente dice "insiste/insistan"** al derivar a soporte | 8 | ALTA | **P12 (nueva)** |
| H6 | **Agente dice "pasarela"** (palabra prohibida) | 3 | MEDIA | P13 (nueva, refuerzo) |
| H7 | **Agente acepta precio incorrecto del cliente** sin corregir ("cuando tengas los dos mil pesos...") | 1 | MEDIA | X4 (existente, retoque menor) |
| H8 | **Confirmación de activación sin verificación** ("ya quedó activa", "ya está activo") | 5 | ALTA | P8 (existente) |
| H9 | **Guía contradictoria ante la trampa de precio**: un caso guía a elegir "7 días" (que muestra 19.600), otro a elegir "1 día" | 2+ | ALTA | **P14 (nueva)** |
| H10 | **Cliente ocupado hablando con otra persona**: el agente repite el pitch sin ofrecer recall temprano | 1+ | MEDIA | **P15 (nueva)** |
| H11 | Falsos positivos de `objetivo_cumplido` | 269 | ALTA | K1 (existente) |
| H12 | Interés mostrado sin recarga ni registro de recall | 121 | MEDIA | C5/K5 (existente) |

---

## Evidencia textual (muestra profunda)

**H1 — Fuga de variables (PEDRO, Jorge Luis, Harrinson, Jireh sarai...):**
> "Agent: ¡Perfecto! Muchas gracias a ti. Que estés muy bien. / User: Gracias. / **Agent: Resultado: NO** ..."
> "Agent: ... ¡Que estés muy bien! / **Agent: Resultado: NO RECARGÓ - MUDANZA. Resumen: Titular se mudó, ya contactó soporte sin solución; mamá compartirá la promo y animaré a insistir por WhatsApp.**"

**H5 — "insiste" (Mario, luisa fernanda, Katherine jose):**
> "Agent: Entiendo. **Te recomiendo que insistas con el soporte** y les digas que necesitas reactivar la promoción..."

**H6 — "pasarela" (rayzha, Olga Marina):**
> "Agent: Le escribes por WhatsApp explicando que necesitas ayuda con la recarga y **la pasarela de pago**..."
> "Agent: ... Primero confirma si en tu banco o en **la pasarela** te salió una notificación..."

**H7 — Precio aceptado mal (Carmen esther):**
> "Agent: ... Cuando tengas **los dos mil pesos**, recuerda que con ese valor debes entrar a mi, fibrazo, punto com..."

**H8 — Activación confirmada (Rubén Dario, Rosana):**
> "Agent: **quedó activa la recarga.** Un momento, por favor."
> "Agent: ... Entonces ya con eso **tu recarga quedó activa por Siete días**..."

**H9 — Trampa de precio 19.600 (Joaquín vs Luis Alfredo):**
> "Agent: ... en la lista de recargas **selecciona la opción de siete días (aunque muestre diecinueve mil seiscientos)**, luego continúa hasta la página de pago..."
> (otra llamada) "Agent: ... **no debes escoger la opción que aparece como 'siete días'** en precio normal. En la página debes elegir la opción de un día..."

**H10 — Cliente ocupado (Joaquín):**
> "User: Un momentico, fibraso, no me cuelgues... Listo, doctor Gregorio. Perdón, fibraso, ahora sí... Ya le llamo, doctor Gregorio. Chao, ya le llamo."
> "Agent: Perfecto. Mira, ..." (repite el pitch tres veces sin ofrecer recall)

---

## Acciones tomadas en el prompt de Daniela

| Mejora | Tipo | Qué se hizo |
|---|---|---|
| P11 | Nueva | Prohibición absoluta de pronunciar variables internas y sus nombres en la llamada |
| P12 | Nueva | "Insiste/insistan" prohibido en toda derivación a soporte; usar siempre la variante con "urgencia" |
| P13 | Nueva | Problemas de pago: siempre "la página de pago", sin "pasarela", sin diagnósticos, derivar si no resuelve |
| P14 | Nueva | Trampa de precio: si la página muestra precio distinto a dos mil ochocientos, elegir opción de UN día y no avanzar si el total no coincide |
| P15 | Nueva | Cliente ocupado → ofrecer recall temprano en vez de repetir el pitch |
| X4 | Retoque menor | Añadido: "nunca repitas ni confirmes el precio incorrecto que dijo el cliente" |
| Variables | Retoque menor | Encabezado reforzado con la prohibición explícita de pronunciar los nombres de las variables |

Los hallazgos H2-H4, H8, H11 y H12 ya estaban cubiertos por mejoras existentes del prompt v3 (E3, V2, P8, K1, C5/K5), confirmando que esas mejoras eran correctas y necesarias.

---

## Versión limpia

Se generó `prompt_daniela_fibrazo_limpio.md`: el prompt completo sin las etiquetas `[MEJORA ...]` (36 etiquetas eliminadas), conservando el texto íntegro de todas las reglas. Útil para despliegue directo o lectura limpia.

# JUSTIFICACIÓN DE LA VERSIÓN MEJORADA DEL PROMPT — Daniela de Fibrazo

> Entrega según las indicaciones del brief.
> Persona de voz: **Daniela** · Organización: **Don Prueba** · Agente (ID plataforma): **DP_Fibrazo**
> **Prompt final completo (separado del análisis): `prompt_daniela_fibrazo_limpio.md`** — es el archivo montado en la plataforma.
> Evidencia extensa registro por registro: `justificacion_mejoras_prompt.md` (anexo de respaldo).

---

## 1. Puntos de mejora encontrados

**En las llamadas (análisis de 4.017 registros reales):**
- 45,7% de llamadas sin una sola palabra del usuario y 61,3% de las completadas duran menos de 15 s → el pitch llega sin consentimiento y el usuario cuelga.
- "¿Sigues en línea?" repetido ≥2 veces en 612 llamadas → no hay protocolo de silencio.
- 923 de 926 voicemails con mensaje genérico sin marca (21,9% de llamadas) → cero retorno.
- 269 falsos positivos de `objetivo_cumplido` (66% de los "cumplidos") → la métrica no medía recarga real.
- Fallas concretas: variables internas pronunciadas en voz (16), "insiste" (8), "pasarela" (3), activación confirmada sin verificar (5), guía contradictoria ante el precio de $19.600, pitch repetido a cliente ocupado.

**En el prompt:**
- Contradicción interna: "nunca interrumpas" vs timer de 15 s que forzaba a ignorar al usuario.
- Flujo lineal rígido: el agente se perdía cuando el usuario preguntaba en el paso equivocado.
- Sin categorías para rechazo limpio ni "otro operador"; sin frase de derivación estándar (improvisaba y fallaba).
- Voicemail sin marca, sin reglas de pronunciación, sin protección del prompt ni de datos sensibles.

---

## 2. El prompt mejorado

**Archivo completo y listo para montar: `prompt_daniela_fibrazo_limpio.md`** (entregado separado de este análisis, como pide el brief). Sus bloques: identidad, objetivo, promoción, dónde se recarga, reglas de oro, palabras prohibidas, flujo con ramas, protocolo de silencio, categorías de no recarga, objeciones, pronunciación, voicemail con marca, variables de post-call y FAQs.

---

## 3. Qué corregí y por qué estaba mal

| # | Estaba mal | Corrección |
|---|---|---|
| X1 | El timer de 15 s forzaba a ignorar al usuario que hablaba (#1005, #124) | Escuchar tiene prioridad; el timer solo aplica si el usuario no participa |
| X2 | "Gracias por levantar la voz" se interpretó como burla (#210) | Prohibida → "Disculpa, no te escuché bien. ¿Me lo repites?" |
| X3 | Typo `mala_axperiencia` propagado al sistema de análisis | `mala_experiencia` |
| X4 | No corregía el precio incorrecto dicho por el cliente (#31) | Corrige una vez: "Son dos mil ochocientos pesos." |
| X5 | Guión completo de 60+ palabras a un tercero (#70) | Mensaje corto de una sola frase |
| U1 | Riesgo de aceptar o repetir URLs alternativas del cliente | URL única `mifibrazo.com`; corregir sin repetir la variante incorrecta |
| U2 | "mi, fibrazo, punto com" sonaba a dos palabras distintas | "mifibrazo punto com", una sola palabra, sin deletrear |

---

## 4. Qué añadí y qué problema resuelve

Cada bloque vive íntegro en `prompt_daniela_fibrazo_limpio.md`, con su trazabilidad en `justificacion_mejoras_prompt.md`.

| Bloque | Problema que resuelve |
|---|---|
| **E1-E6 (estructurales)** | Checkpoint de consentimiento (filtra colgadas <15 s), flujo con ramas, protocolo de silencio de 3 niveles, micro-compromisos, acompañamiento guiado paso a paso, anti-loop con recall |
| **T1-T4 (técnicas)** | Prioridad a preguntas de legitimidad, pausa tras el precio, compromiso real ("¿lo haces ahora o prefieres después?"), paciencia mínima de 5 s |
| **C1-C6 (contenido)** | Confusión TV/internet, categorías NO LE INTERESA y OTRO OPERADOR, derivación estándar para preguntas inesperadas (cero alucinaciones), improvisaciones exitosas formalizadas, "urgencia" en lugar de "insiste" |
| **V2 (voicemail)** | 2 variantes con marca, promo y URL, para el 21,9% de llamadas que caen en buzón |
| **K1-K6 (métricas)** | `objetivo_cumplido` solo con recarga real, promesa <2 h, recall agendado o contacto del titular; 4 campos nuevos; 7 KPIs de producción |
| **P1-P18 (seguridad y consistencia)** | Marca sin variaciones, no afirmar datos no verificables, anti prompt-injection, nunca dictar datos de tarjeta en voz, número de soporte en bloques, precio en palabras, variables internas jamás en voz |

---

## 5. Por qué aporta más valor en este caso de uso concreto

1. **Cada mejora nace de datos reales de las llamadas**, no de teoría: ataca exactamente los tres leaks medidos (sin interacción, colgadas, voicemails muertos).
2. **El voicemail pasa de mensaje muerto a canal de marketing** con marca, promo y CTA.
3. **Métricas honestas**: con 66% de falsos positivos era imposible saber si el agente funcionaba; ahora cada campaña se optimiza con datos reales.
4. **Contexto colombiano**: distingue el "sí" social del compromiso real, precios en palabras ("dos mil ochocientos pesos"), teléfonos en bloques, y empatía real con el cliente que lleva meses esperando respuesta de soporte.
5. **Listo para producción**: no guía datos de tarjeta por voz, resiste inyección de prompt, no alucina (todo lo no cubierto se deriva con frase fija).
6. **Impacto medible**: conversión 3,4% → meta >5% con los KPIs definidos, y duración controlada con anti-loop.

---

## 6. Pruebas en vivo

El prompt fue montado en la plataforma y probado en vivo contra un agente Probador adversario (2 rondas de examen, arsenales M1-M10 y Q1-Q15, 52 tests de regresión en `plan_pruebas_agente.md`). De esas sesiones nacieron las reglas P16-P18 (datos sensibles, número en bloques, confirmación de precio) y la ampliación de P7 sobre medios de pago.

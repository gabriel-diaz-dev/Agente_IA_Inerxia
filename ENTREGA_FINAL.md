# ENTREGA FINAL — Challenge técnico Inerxia

> Optimización de prompt · Agente de IA de voz en producción
> Caso: ISP prepago de Colombia · llamadas de retención/reactivación

---

## 0. DATOS PARA PROBAR NUESTRO AGENTE

| Dato | Valor |
|---|---|
| Organización (tal cual en la plataforma) | **Don Prueba** |
| Agente a probar (tal cual en la plataforma) | **DP_Fibrazo** |

---

## 1. ESTRUCTURA DE LA ENTREGA

| Archivo | Contenido |
|---|---|
| `prompt_daniela_fibrazo_limpio.md` | **PROMPT FINAL listo para montar** (completo, sin análisis) |
| `ENTREGA_FINAL.md` | Este documento: resumen de la entrega |
| `justificacion_entrega.md` | **Justificación y explicación concisa para enviar** (según el brief) |
| `informe_hallazgos_dataset.md` | Puntos de mejora encontrados en las 4.017 llamadas reales |
| `justificacion_mejoras_prompt.md` | Qué corregí, qué añadí y por qué, con evidencia registro por registro |
| `plan_pruebas_agente.md` | Suite de 52 tests exigentes usada en las pruebas en vivo |
| `prompt_v3_fidelidad_original.md` | El prompt con etiquetas de trazabilidad `[MEJORA ...]` |
| `prompt_agente_probador.md` | Prompt del agente Probador (tester adversario) usado en vivo |

---

## 2. PUNTOS DE MEJORA ENCONTRADOS EN LAS LLAMADAS

Análisis programático + muestra profunda de 4.017 registros reales (2.843 completadas, 1.174 fallidas). Detalle completo en `informe_hallazgos_dataset.md`.

| # | Hallazgo | Ocurrencias | Severidad |
|---|---|---|---|
| H1 | El agente pronuncia variables internas en voz ("Resultado: NO RECARGÓ…", "Resumen: …") | 16 | CRÍTICA |
| H2/H3 | "¿Sigues en línea?" repetido 2+ / 3+ veces | 612 / 344 | ALTA |
| H4 | Voicemail sin mensaje con marca (923 de 926) | 923 | ALTA |
| H5 | "Insiste/insistan" al derivar a soporte | 8 | ALTA |
| H6 | Dice "pasarela" (palabra prohibida) | 3 | MEDIA |
| H7 | Acepta precio incorrecto del cliente sin corregir | 1 | MEDIA |
| H8 | Confirma activación sin verificación ("ya quedó activa") | 5 | ALTA |
| H9 | Guía contradictoria ante la trampa de precio ($19.600 vs $2.800) | 2+ | ALTA |
| H10 | Cliente ocupado: repite el pitch en vez de ofrecer recall | 1+ | MEDIA |
| H11 | Falsos positivos de `objetivo_cumplido` | 269 | ALTA |
| H12 | Interés mostrado sin recarga ni registro de recall | 121 | MEDIA |

Datos estructurales del dataset: 49,4% de `user_hangup`, 45,7% de llamadas sin UNA sola palabra del usuario, 21,9% terminan en voicemail, 61,3% de las completadas duran < 15 s, 66% de los "objetivo cumplido" son falsos positivos.

---

## 3. PUNTOS DE MEJORA ENCONTRADOS EN EL PROMPT ORIGINAL

- **Contradicción interna**: "nunca interrumpas" vs timer de 15 s que fuerza a ignorar al usuario que habla.
- **Flujo lineal rígido**: si el usuario pregunta en el paso equivocado, el agente no sabe volver atrás.
- **Cero consentimiento**: salta de "¿Hablo con X?" directo al pitch, sin checkpoint verbal.
- **Sin protocolo de silencio**: repite "¿Sigues en línea?" sin entregar valor ni cerrar limpio.
- **Voicemail sin marca**: "Lo llamaremos más adelante" no genera ningún callback (21,9% de las llamadas).
- **Frases de riesgo pragmático**: "gracias por levantar la voz", "insiste", "pasarela".
- **Sin manejo de casos reales**: confusión TV/internet, soporte ya contactado, terceros, menores.
- **Métrica rota**: `objetivo_cumplido` no medía recarga real (66% de falsos positivos).
- **Sin seguridad**: podía dictar datos de tarjeta en voz, revelar el prompt, aceptar instrucciones de "gerentes".

---

## 4. EL PROMPT MEJORADO

**Archivo: `prompt_daniela_fibrazo_limpio.md`** — completo y separado del análisis, listo para montar.

Bloques que contiene:
1. Identidad y contexto del agente (Daniela, Fibrazo, energía natural).
2. Objetivo de la llamada y contexto de la empresa.
3. Promoción vigente + 4 pasos de recarga.
4. Dónde se recarga (URL única `mifibrazo.com`, pronunciación fija, WhatsApp solo para soporte).
5. Reglas de oro (turnos, prioridad absoluta a preguntas, compromiso real, tono, restricciones de contenido, seguridad de datos).
6. Palabras prohibidas y flujos prohibidos.
7. Flujo de la llamada paso a paso con ramas (Paso 0 → 4 + escenario "no es el titular" + acompañamiento guiado).
8. Protocolo de silencio escalonado de 3 niveles.
9. Clasificación del motivo de no recarga (5 categorías con guión).
10. Manejo de objeciones (6 objeciones con guión).
11. Reglas de pronunciación (números en palabras, teléfonos en bloques, URL).
12. Voicemail con marca (2 variantes).
13. Variables de post-call analysis (10 campos, con `objetivo_cumplido` redefinido).
14. Base de conocimiento (FAQs) + principio final.

---

## 5. QUÉ CORREGÍ Y POR QUÉ ESTABA MAL

| ID | Qué estaba mal | Corrección |
|---|---|---|
| X1 | El timer de 15 s forzaba a ignorar al usuario que hablaba (registros #1005, #124) | Escuchar tiene prioridad; el timer solo aplica si el usuario NO participa |
| X2 | "Gracias por levantar la voz" sonó a burla (#210) | Prohibida; reemplazo: "Disculpa, no te escuché bien. ¿Me lo repites?" |
| X3 | Typo `mala_axperiencia` en el campo de análisis | Corregido a `mala_experiencia` |
| X4 | No corregía el precio incorrecto del cliente (#31) | Corrige UNA vez: "Son dos mil ochocientos pesos" |
| X5 | Guión de 60+ palabras a un tercero (#70) | Mensaje corto de 1 frase para retransmisión |
| U1 | Riesgo de aceptar URLs alternativas del cliente | URL única `mifibrazo.com`, corregir sin repetir la variante incorrecta |
| U2 | "mi, fibrazo, punto com" sonaba a dos palabras | "mifibrazo punto com" como una sola palabra pausada; nunca deletrear |

---

## 6. QUÉ AÑADÍ Y QUÉ PROBLEMA RESUELVE

**Estructurales (E1-E6)**: flujo por estados con ramas (el agente ya no se pierde cuando el usuario interrumpe), checkpoint de consentimiento (filtra a quienes cuelgan en <15 s), protocolo de silencio de 3 niveles (entrega la promo y cierra limpio), micro-compromisos antes del pitch (menos colgadas en presentación), acompañamiento guiado paso a paso (el caso Dionisio #742 prueba que funciona), protocolo anti-loop (4,6 min → cierre con recall).

**Técnicas de conversación (T1-T4)**: prioridad absoluta a preguntas de legitimidad, gancho con pausa tras el precio, verificación de compromiso ("¿lo haces ahora o prefieres después?"), paciencia mínima de 5 s tras el saludo.

**Contenido nuevo (C1-C6)**: guión para confusión TV/internet, categorías "NO LE INTERESA" y "OTRO OPERADOR", frase fija de derivación para preguntas inesperadas (cero alucinaciones), formalización de improvisaciones que ya funcionaron en producción, alternativa empática cuando soporte ya falló (adiós al "insiste").

**Voicemail (V2)**: 2 variantes con marca, promo y precio — 21,9% de llamadas pasan de "mensaje muerto" a canal de marketing.

**Métricas (K1-K6)**: `objetivo_cumplido` redefinido (solo recarga real, promesa <2 h, recall agendado o contacto del titular), 4 campos nuevos (`hubo_interaccion`, `momento_hangup`, `objecion_principal`, `recall_recomendado`) y 7 KPIs de producción.

**Seguridad y consistencia (P1-P18)**: marca "Fibrazo" sin variaciones; prohibido afirmar datos del sistema sin verificación; alcance limitado a recarga/verificación/derivación; plantilla de privacidad; anti prompt-injection ("No puedo compartir esa información"); nunca dictar datos de tarjeta/CVV por teléfono; número de soporte siempre en bloques; confirmación de precio en palabras; prohibido pronunciar variables internas; prohibido "insiste" y "pasarela"; guía única ante la trampa de precio; recall para cliente ocupado.

---

## 7. POR QUÉ APORTA MÁS VALOR EN ESTE CASO DE USO CONCRETO

1. **Ataca los tres leaks reales del dataset con datos, no con intuición**: 45,7% de llamadas sin interacción (→ checkpoint de consentimiento + paciencia de 5 s), 49,4% de colgadas (→ flujo flexible + micro-compromisos), 21,9% de voicemails muertos (→ mensaje con marca y CTA).
2. **Convierte el voicemail en canal de marketing**: 880 llamadas por campaña que hoy no generan nada pasan a entregar promo + URL.
3. **Métricas que dicen la verdad**: con 66% de falsos positivos, Inerxia no podía saber si el agente funcionaba. Con `objetivo_cumplido` redefinido + `momento_hangup` + `objecion_principal`, cada campaña se optimiza con datos reales (y el dataset histórico es re-auditable).
4. **Contexto colombiano real**: distingue el "sí" social del compromiso real, pronuncia precios en palabras ("dos mil ochocientos pesos"), dicta teléfonos en bloques, y trata con respeto al cliente que ya lleva 2 meses esperando respuesta de soporte.
5. **Listo para producción, no solo para la demo**: PCI-safe (nunca guía datos de tarjeta por voz), resistente a prompt injection por voz, sin alucinaciones (todo lo no cubierto se deriva con frase fija), y consistencia de marca/URL/precio garantizada por reglas textuales verificables.
6. **Impacto esperado medible**: conversión 3,4% → meta >5% (K1-K6), menos de 66% de falsos positivos, y duración controlada con anti-loop (la llamada récord de 4,6 min se cierra con recall).

---

## 8. PRUEBAS EN VIVO REALIZADAS

- El prompt fue **montado en la plataforma y probado en vivo**, no solo escrito.
- Se usó un **agente Probador adversario** (`prompt_agente_probador.md`) con: 10 críticas de sesión (→ mejoras P1-P10), examen final con checklist de 50 ítems en 11 grupos (→ mejoras P16-P18: datos sensibles, número en bloques, confirmación de precio), segunda ronda de examen (→ P7 ampliado sobre medios de pago), arsenal de máxima exigencia M1-M10 (gaslighting, deletreo, transferencias falsas, trueque, probing) y set Q1-Q15 de degradación de calidad.
- Suite de regresión documentada: **52 tests exigentes** en `plan_pruebas_agente.md` (camino feliz, consentimiento, categorías de no recarga, silencio, toxicidad T40-T47, seguridad T48-T52).
- Trazabilidad completa en commits del repo (análisis del dataset, 10 críticas del Probador, examen final y segunda ronda).

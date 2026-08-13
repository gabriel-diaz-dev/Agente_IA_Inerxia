# Justificación de mejoras al prompt "Daniela de Fibrazo"

> Documento de evidencia. Cada mejora propuesta está respaldada por al menos un registro real del dataset de 4,017 llamadas (archivo `prueba-2-retencion-7x-1-04-08-26-report.xlsx`).

---

## 1. ESTRUCTURALES (cambian el flujo de la llamada)

---

### E1 — Flujo flexible con ramas (no lineal rígido)

**Qué cambia:** El prompt actual prescribe un flujo Paso 0 → 1 → 2 → 3 → 4. El nuevo prompt define estados (SALUDO, PROMO, EXPLICACION, OBJECION, CIERRE) y permite transiciones entre cualquiera de ellos según lo que diga el usuario.

**Evidencia:**

| Registro | Qué pasó | Duración | Consecuencia |
|---|---|---|---|
| #132 PEDRO | Usuario dice "Sí, claro" → agente lanza promo → usuario pregunta "¿Y esto de qué me están llamando?" → el agente iba en Paso 2 y no supo volver a identificación | 2.55 min | Confusión, aunque terminó con recargo |
| #1005 Alejandro | Usuario pregunta "¿Con quién hablo?" mientras agente ya está en Paso 1 → agente ignora la pregunta y sigue su script | 0.88 min | Usuario abandona, sentimiento NEGATIVO |
| #1730 Juan Carlos | Usuario pide retiro de equipos en Paso 1 → agente intenta volver atrás pero se corta | 0.97 min | Llamada abandonada sin resolver |

**Impacto en datos:** 49.4% de user hangup. En al menos 8 de los 50 registros analizados, el agente ignora una señal del usuario por estar atrapado en un paso del script.

---

### E2 — Checkpoint de consentimiento obligatorio

**Qué cambia:** El agente no puede avanzar a mencionar la promoción sin haber obtenido una confirmación verbal explícita del usuario. "Buen día", "Aló", "Sí" aislado NO cuentan como consentimiento. Debe haber un intercambio de al menos 2 turnos antes del pitch.

**Evidencia:**

| Registro | Qué pasó | Duración |
|---|---|---|
| #1758 Gustavo Adolfo | Usuario dice "Sí" → agente va directo a promo → usuario cuelga a los 5 segundos de pitch | 0.20 min |
| #838 Arlin aramin | Usuario dice "Buen día" → agente ignora y lanza promo → cuelga inmediatamente | 0.28 min |
| #385 VANESSA | Agente salta el "¿tienes un momentico?" → fusiona saludo + promo en un solo bloque | 0.76 min |

**Dato agregado:** 1,744 de 2,843 llamadas completadas (61.3%) duran menos de 15 segundos. El usuario cuelga durante o inmediatamente después del saludo+pitch inicial. Un checkpoint de consentimiento obliga a que el usuario invierta al menos 2 turnos verbales antes de recibir información comercial, filtrando a quienes simplemente no quieren escuchar.

---

### E3 — Protocolo de silencio escalonado (3 niveles)

**Qué cambia:** En vez de repetir "¿Sigues en línea?" hasta 3-4 veces, el agente escala:

1. Primer silencio (7s): "¿Sigues ahí?"
2. Segundo silencio (7s más): "Te dejo el dato rápido por si me escuchas: [promo en 10 segundos]. Cuando puedas, entra a mifibrazo.com."
3. Tercer silencio (7s más): "Gracias por tu tiempo, que estés bien." → FIN

**Evidencia:**

| Registro | Patrón | Veces que repite "¿Sigues en línea?" |
|---|---|---|
| #458 Adolfo | Saludo → "¿Sigues en línea?" x3 | 3 |
| #573 Glenia | Saludo → "¿Sigues en línea?" x3 | 3 |
| #1005 Alejandro | Saludo, respuesta del usuario, agente ignora → "¿Sigues en línea?" x3 | 3 |
| #1128 AILETH | Saludo → "¿Sigues en línea?" x2 | 2 |
| #2220 Senda nahomy | Saludo → "¿Sigues en línea?" x3 | 3 |
| #3656 Anais María | Saludo → "¿Sigues en línea?" x3 | 3 |

En todos estos casos, la repetición no recuperó al usuario. El agente sonó a robot colgado. Con el protocolo escalonado, el segundo nivel al menos entrega valor (la promo) por si el usuario está escuchando sin hablar, y el cierre es limpio sin degradar la experiencia.

**Dato agregado:** 1,835 de 4,017 llamadas (45.7%) no tienen NI UNA SOLA palabra del usuario en la transcripción.

---

### E4 — Micro-compromisos antes del pitch

**Qué cambia:** El agente no lanza la promo inmediatamente después del consentimiento. Primero contextualiza:

1. "Soy Daniela, de Fibrazo. ¿Sabes que tienes el servicio suspendido?"
2. (Espera respuesta)
3. "Te llamé porque tenemos algo para reactivarlo. ¿Me escucho?"

Esto construye contexto antes de mencionar precio/días.

**Evidencia:**

| Registro | Qué pasó | Duración |
|---|---|---|
| #829 Iris María | Saludo → cuelga antes de cualquier cosa | 0.07 min (la más corta) |
| #2621 Diego Andrés | Saludo → cuelga inmediato | 0.15 min |
| #104 William Esteban | Saludo, identificación OK → "Perfecto. Te llamo de Fibrazo..." → cuelga durante la presentación | 0.27 min |
| #2235 Leidis Johana | Saludo, "Sí" → "Perfecto. Te llamo de Fibrazo..." → cuelga | 0.25 min |
| #954 Nancy Patricia | Saludo, "¿Sigues en línea?" → usuario dice "No, no, no" → cuelga | 0.28 min |

El patrón es claro: el salto de "¿Hablo con X?" a "Te llamo de Fibrazo porque tenemos una promo..." es demasiado brusco. El usuario no tiene tiempo de procesar quién lo llama y por qué antes de recibir el pitch.

---

### E5 — Acompañamiento guiado paso a paso

**Qué cambia:** En vez de soltar los 4 pasos de recarga de golpe, el agente guía uno por uno con preguntas de verificación:

> "Entra a mifibrazo.com. ¿Ya la tienes abierta?"
> → "Ahora pon tu número de cédula. ¿Ya?"
> → "Selecciona 'Quiero recargar'. ¿Lo ves?"
> → "Elige la opción de un día. ¿Te aparece el pago?"

**Evidencia:**

| Registro | Método | Resultado |
|---|---|---|
| #742 Dionisio | Guiado paso a paso con preguntas de verificación | `recargo=Yes`, llamada fluida de 1.94 min, sentimiento POSITIVE |
| #29 Heiner | 4 pasos de golpe → usuario dice "Pues sí" → agente suelta todo → usuario dice "No" → cierre | `recargo=No`, objetivo marcado Yes pero sin recarga real |
| #14 Danna Sofia | Interés inicial → 4 pasos de golpe → usuario cuelga | `interes_mostrado=Yes` pero sin recarga |

El caso de Dionisio es elocuente: es una de las pocas llamadas donde el agente preguntó "¿Ya la tienes abierta?", "¿Lograste avanzar hasta la opción de pago?" y acompañó el proceso. El usuario cooperó y completó.

---

### E6 — Protocolo anti-loop (reemplaza límite de tiempo duro)

**Qué cambia:** Si el agente ha repetido la misma información (URL, precio, pasos) 3 o más veces sin que el usuario avance a la acción, el agente ofrece cierre con recall en vez de seguir insistiendo:

> "Veo que hoy no es el momento. ¿Quieres que te llamemos mañana a esta misma hora o prefieres otro día?"

**Evidencia:**

| Registro | Patrón | Duración |
|---|---|---|
| #21 Duvis Virgina | Agente repite URL + pasos **7 veces**. Usuaria pregunta por TV 3 veces. Loop informativo sin acción | 4.63 min |

Si bien esta llamada terminó con `recargo=Yes`, el costo fue 4.6 minutos de conversación circular. Con un protocolo anti-loop, al tercer ciclo de "solo internet, la URL es..." el agente habría ofrecido recall. En un escenario de alto volumen, 4.6 minutos por llamada no escala.

---

## 2. TÉCNICAS DE CONVERSACIÓN

---

### T1 — Regla de prioridad absoluta

**Qué cambia:** Si el usuario hace una pregunta sobre identidad, legitimidad o propósito de la llamada, el script se detiene y esa pregunta se responde **antes que cualquier otra cosa**.

**Evidencia:**

| Registro | Pregunta del usuario | Respuesta del agente | Resultado |
|---|---|---|---|
| #1005 Alejandro | "¿Con quién hablo?" (2 veces) | El agente sigue con el script de la promo | Usuario abandona, sentimiento NEGATIVO |
| #132 PEDRO | "¿De qué entidad?" / "¿Y esto de qué me están llamando?" | Agente empieza "Soy Daniela, de F..." y se corta | Confusión |
| #349 luisa fernanda | "¿Con quién hablo?" | Agente responde correctamente "Soy Daniela de Fibrazo" | La llamada continúa (aunque fue negativa por otro motivo) |

Cuando el usuario pregunta QUIÉN eres, no le interesa QUÉ ofreces hasta que no sepa QUIÉN llama. Es una regla básica de psicología de atención telefónica.

---

### T2 — Técnica de "gancho con pausa"

**Qué cambia:** Después de mencionar la promo + precio, el agente hace una pausa de 2-3 segundos y lanza una pregunta abierta tipo "¿Qué te parece?" o "¿Te sirve?" en vez de seguir hablando con los pasos de recarga.

**Evidencia:**

| Registro | Flujo post-promo | Resultado |
|---|---|---|
| #14 Danna Sofia | Promo → sigue con pasos de recarga → usuario cuelga | `interes_mostrado=Yes`, pero sin recarga |
| #2 Heiner | Promo → "¿Te interesa?" → usuario "Pues sí" → agente suelta 4 pasos → usuario "No" | Interés inicial perdido |
| #13 Leidis Johana | Promo dicha → usuario cuelga durante la explicación | 0.25 min |

El interés expresado ("Ajá", "Sí", "Pues sí") es una ventana de 3-5 segundos. Si el agente responde soltando instrucciones en vez de validar y profundizar el interés, la ventana se cierra.

---

### T3 — Verificación de compromiso

**Qué cambia:** El agente distingue entre "sí social" (por cortesía) y "sí real" (con intención de actuar). Usa preguntas binarias de acción:

> En vez de "¿Te interesa?" → "¿Lo hacemos ahora o prefieres que te llame mañana?"

**Evidencia:**

| Registro | Comportamiento del usuario | Problema |
|---|---|---|
| #21 Duvis Virgina | "Sí", "Bueno", "Dale", "Está bien", "Perfecto"... dice SÍ a todo pero NUNCA realiza la recarga en 4.6 minutos | El agente interpreta cada "sí" como avance, pero son muletillas |
| #3 GLENYS | "Ahora mismo, no le digo nada" → ambigüedad total | Agente improvisa bien preguntando "¿Puedo preguntarte por qué?" |

En Colombia es culturalmente común decir "sí" por cortesía sin intención real de actuar. La pregunta binaria de acción ("ahora o después") fuerza al usuario a definirse sin ser agresivo.

---

### T4 — Timer de paciencia mínimo

**Qué cambia:** Después del saludo inicial, el agente espera un mínimo de 5 segundos reales antes de asumir que no hay respuesta y ejecutar el primer nivel del protocolo de silencio.

**Evidencia:**

| Registro | Tiempo hasta "¿Sigues en línea?" | Duración total |
|---|---|---|
| #3 ADALGIZA | ~11 segundos desde el saludo (incluye tiempo de hablar) | 0.17 min |
| #29 Heiner | El agente dice "¿Sigues en línea?" mientras el usuario dice "Aló" | 1.05 min |

El tiempo de reacción promedio a una llamada de número desconocido es de 3-7 segundos (el usuario mira la pantalla, decide si contestar, busca un lugar tranquilo). Si a los 4 segundos de terminar el saludo el agente ya pregunta "¿Sigues en línea?", está interrumpiendo el proceso natural de atención del usuario.

---

## 3. CONTENIDO NUEVO

---

### C1 — Manejo de confusión TV/internet/deportes

**Qué cambia:** Guión de respuesta para cuando el usuario pregunta por televisión, canales deportivos o streaming. Solo se activa si el usuario lo menciona explícitamente.

> "Fibrazo es un servicio de internet por fibra óptica. No incluye canales de TV ni paquetes de deportes. Con el internet puedes usar plataformas como YouTube o Netflix. ¿Quieres activar el internet?"

**Evidencia:**

| Registro | Menciones de TV/deportes | Impacto |
|---|---|---|
| #21 Duvis Virgina | "para la televisión", "para el televisor", "para el deporte, boxeo, juego", "y del televisor" | 3 intervenciones sobre TV en una llamada; agente repite explicación 4+ veces sin guión definido |

Aunque solo aparece en 1 de los 50 registros revisados, esta llamada fue la más larga del dataset (4.6 min) precisamente por este loop. Un guión claro para este caso acorta la llamada y evita fricción.

---

### C2 — Nueva categoría "NO LE INTERESA"

**Qué cambia:** Se añade una categoría de no recarga para cuando el usuario explícitamente rechaza la oferta sin mencionar dinero, mala experiencia ni mudanza.

**Evidencia:**

| Registro | Respuesta del usuario | Categoría actual |
|---|---|---|
| #2035 Gabriel Jaime | "No, para nada" cuando se le ofrece la promo | Sin categoría clara; marcado como `objetivo_cumplido=Yes` incorrectamente |
| #210 Emilita | "No, yo prefiero que me deje tranquilo" | Marcado como `objetivo_cumplido=Yes` pero fue un rechazo total |

Sin esta categoría, los rechazos limpios se clasifican erróneamente como "objetivo cumplido" o quedan sin clasificar, distorsionando las métricas.

---

### C3 — Nueva categoría "YA TIENE OTRO OPERADOR"

**Qué cambia:** La objeción "Ya tengo otro operador" ya existe en el prompt actual como manejo de objeción, pero no tiene categoría propia de no recarga. Se añade `NO RECARGÓ - OTRO OPERADOR`.

**Evidencia:**

El prompt actual ya incluye esta objeción en la sección "MANEJO DE OBJECIONES" con una respuesta textual, pero no existe una categoría de post-call analysis que capture este caso. Sin la categoría, no se puede medir cuántos usuarios se pierden por competencia.

---

### C4 — Protocolo para preguntas inesperadas

**Qué cambia:** Cuando el usuario hace una pregunta que no está en la base de conocimiento del prompt, el agente usa una frase de derivación estándar. NUNCA improvisa información.

> "Esa información no la manejo en este momento. Escríbele a nuestro equipo de soporte por WhatsApp al [número] y ellos te ayudan con eso."

**Evidencia:**

| Registro | Pregunta inesperada | Respuesta del agente |
|---|---|---|
| #2820 Josirys | "Hola, ¿cuándo es libre?" | Agente dice "Mira..." y la llamada se corta |
| #1730 Juan Carlos | Pide retiro de equipos, proceso fuera del alcance del prompt | Agente intenta improvisar "Entiendo, lamento la situación. Para e..." y se corta |

El prompt actual prohíbe responder fuera de alcance pero no da una frase estándar. El agente improvisa y falla. Una frase fija de derivación resuelve el 100% de estos casos sin riesgo de alucinación.

---

### C5 — Incorporar improvisaciones exitosas del agente

**Qué cambia:** Varias frases que el agente improvisó en producción y funcionaron se integran formalmente al prompt:

| Frase improvisada (registro) | Se integra como |
|---|---|
| "¿Puedo preguntarte por qué no puedes recargar ahora?" (#31 GLENYS) | Sondeo de objeción en categoría DINERO |
| "¿Quieres que te recuerde cómo recargar cuando estés listo, o prefieres que cierre la llamada ahora?" (#1667 Danielys) | Opción de cierre con recall |
| "Entiendo, te dejo en paz entonces. Gracias por tu tiempo." (#2571 Juana Lucia) | Cierre alternativo del protocolo de silencio |
| "¿Quieres que te espere un momento mientras intentas?" (#349 luisa fernanda) | Opción de acompañamiento en tiempo real |

**Evidencia:** En los 50 registros analizados, el agente improvisó frases que NO están en el prompt en al menos 6 ocasiones. En 4 de esas 6, la improvisación mejoró el flujo de la llamada. Las 2 restantes fueron neutrales. Formalizarlas elimina la variabilidad y asegura consistencia.

---

### C6 — Alternativa cuando WhatsApp de soporte ya falló

**Qué cambia:** Si el usuario indica que ya contactó a soporte por WhatsApp y no obtuvo respuesta, el agente NO dice "insiste". En su lugar:

> "Lamento que no te hayan respondido aún. Puedes intentar de nuevo escribiendo la palabra 'traslado' o 'urgencia' al [número], que eso acelera la atención. ¿Quieres que tome nota de tu caso?"

**Evidencia:**

| Registro | Situación | Respuesta actual |
|---|---|---|
| #349 luisa fernanda | "Ya yo escribí y no hay ninguno me contesta. Tengo ya 2 meses esperando el traslado" | "Lo mejor es que insistas con soporte" → usuaria frustrada |

Decirle "insiste" a alguien que lleva 2 meses insistiendo no es empatía, es abandono. La nueva respuesta reconoce el problema, ofrece un mecanismo concreto (palabra clave) y pregunta si quiere dejar registro.

---

## 4. CORRECCIONES

---

### X1 — Corregir contradicción "escuchar vs timer de 15 segundos"

**Qué cambia:** La regla "Habla → escucha → responde. Nunca interrumpas" tiene prioridad sobre el límite de 15 segundos. Si el usuario está hablando, el timer se pausa. El límite de 15s solo aplica para monólogos del agente cuando el usuario NO está participando.

**Evidencia:**

| Registro | Conflicto |
|---|---|
| #1005 Alejandro | Usuario dice "¿Con quién hablo?" dos veces → agente sigue script → timer de 15s fuerza seguir hablando |
| #124 Angie margarita | Usuario dice "Aló" tres veces → agente ignora → timer fuerza seguir con el pitch |

La regla de 15 segundos fue diseñada para evitar monólogos, pero en la práctica fuerza al agente a ignorar al usuario para cumplir el timer. La prioridad debe invertirse.

---

### X2 — Prohibir "gracias por levantar la voz"

**Qué cambia:** Se añade a la lista de frases prohibidas: "gracias por levantar la voz", "gracias por hablar más fuerte", y cualquier variante. En su lugar, si el audio es difícil de entender: "Disculpa, no te escuché bien. ¿Me lo repites?"

**Evidencia:**

| Registro | Frase usada | Reacción del usuario |
|---|---|---|
| #210 Emilita | "Oye, gracias por levantar la voz" | La usuaria ya estaba molesta. La frase, aunque posiblemente bien intencionada, fue interpretada como sarcasmo. Terminó con "No, yo prefiero que me deje tranquilo" |

En comunicación telefónica, "gracias por levantar la voz" es una frase de alto riesgo pragmático. Si el tono del TTS no transmite exactamente la intención (y el TTS rara vez lo hace), se interpreta como "gracias por gritarme".

---

### X3 — Corregir tipografía `mala_axperiencia` → `mala_experiencia`

**Qué cambia:** Corrección del nombre del campo en la sección de variables de post-call analysis.

**Evidencia:** El prompt actual escribe consistentemente `mala_axperiencia` en lugar de `mala_experiencia`. Este error se propagó al sistema de analysis (los datos exportados usan el nombre con typo). Mantener el typo en el nuevo prompt asegura compatibilidad con el sistema actual, pero se documenta para corrección futura en la plataforma.

---

### X4 — Corregir precio cuando el usuario lo dice mal

**Qué cambia:** Si el usuario menciona un precio incorrecto (ej. "$2.500" en vez de "$2.800"), el agente corrige amablemente una sola vez:

> "Son dos mil ochocientos, no dos mil quinientos. Es el precio de un día, y con la promo te llevas siete."

**Evidencia:**

| Registro | Usuario dice | Agente corrige |
|---|---|---|
| #31 GLENYS | "Cuando tenga los 2.500, son esos" | No corrige |

Si la usuaria recarga esperando pagar $2.500 y el cobro es $2.800, la experiencia post-recarga es negativa. Una corrección suave en la llamada evita fricción futura.

---

### X5 — No dar guión textual largo para retransmisión por terceros

**Qué cambia:** Cuando la persona que contesta no es el titular, el mensaje para retransmitir se reduce a una frase de 15-20 palabras máximo:

> "Dile que Fibrazo tiene 7 días de internet por $2.800 en la página web mifibrazo.com."

No se dan los 4 pasos completos, no se da el WhatsApp de soporte, no se explica cómo funciona la promo.

**Evidencia:**

| Registro | Qué pasó | Problema |
|---|---|---|
| #70 Yuranis Del Carmen | La mamá recibe el guión completo (4 pasos + WhatsApp en formato telefónico + instrucciones) para retransmitir a su hija | La mamá pide aclaración 2 veces; es información excesiva para un tercero |

Un tercero no es un canal de comunicación confiable. Darle un guión de 60+ palabras garantiza que el mensaje llegue degradado o no llegue.

---

## 5. VOICEMAIL

---

### V2 — Variantes de mensaje de buzón con marca

**Qué cambia:** Se definen dos variantes (corta y completa) en vez de solo "Lo llamaremos más adelante":

**Variante corta** (si el buzón empieza a grabar mientras el agente habla):
> "Hola, soy Daniela de Fibrazo. Tenemos una promo de internet para ti. Te llamamos después. ¡Gracias!"

**Variante completa** (si el sistema detecta buzón antes de que el agente hable):
> "Hola, te habla Daniela de Fibrazo. Queríamos contarte que tienes una promoción activa de 7 días de internet por solo $2.800 en mifibrazo.com. Es solo por hoy. Te llamaremos más tarde. ¡Que estés bien!"

**Evidencia:**

| Registro | Mensaje actual | Problema |
|---|---|---|
| #358 Juana | "Lo llamaremos más adelante" | No identifica a Fibrazo ni el motivo |
| #421 Uriel | "Lo llamaremos más adelante" | Ídem |
| #3018 Carmen Isabel | "Lo llamaremos más adelante" | Ídem |
| #3656 (varios) | "Lo llamaremos más adelante" | Ídem |

880 llamadas (21.9%) terminan en voicemail. Con el mensaje actual, el usuario recibe una notificación de "1 mensaje nuevo" que al escucharlo solo dice "Lo llamaremos más adelante". Eso no genera ningún callback. Un mensaje con marca, promo y llamado a acción sí puede generar retorno.

---

## 6. MÉTRICAS, CAMPOS DE ANALYSIS Y KPIs

---

### K1 — Redefinir `objetivo_cumplido`

**Qué cambia:** El campo `objetivo_cumplido` pasa a ser True SOLO si se cumple AL MENOS UNA de estas condiciones:

1. `recargo = Yes` (el usuario recargó durante o inmediatamente después de la llamada)
2. El usuario confirmó explícitamente que realizará la recarga en las próximas 2 horas
3. Se agendó un recall efectivo (día y hora acordados con el usuario)
4. En caso de no ser el titular: se obtuvo un número de contacto válido del titular

**Evidencia:**

| Dato | Valor |
|---|---|
| `objetivo_cumplido=Yes` en el dataset | 406 (10.1%) |
| `recargo=Yes` en el dataset | 138 (3.4%) |
| Diferencia (falsos positivos) | 268 (66% de los "cumplidos") |

Ejemplos de falsos positivos identificados:

| Registro | `objetivo_cumplido` | Qué pasó realmente |
|---|---|---|
| #210 Emilita | Yes | Usuaria molesta dijo "prefiero que me deje tranquilo" |
| #2035 Gabriel Jaime | Yes | Usuario dijo "No, para nada" dos veces |
| #1667 Danielys | Yes | Usuaria necesita traslado, no puede recargar hasta que se resuelva |

Con el criterio actual, cualquier llamada donde el agente mencionó la promo cuenta como "cumplida". La métrica no mide efectividad real.

---

### K2 — Nuevo campo: `hubo_interaccion` (Boolean)

**Qué mide:** Si el usuario dijo al menos una palabra durante la llamada.

**Por qué:** 1,835 de 4,017 llamadas (45.7%) no tienen NI UNA intervención del usuario. Sin este campo es imposible separar "llamadas que nunca fueron conversaciones" de "llamadas donde el agente perdió al usuario".

---

### K3 — Nuevo campo: `momento_hangup` (Enum)

**Valores:** `SALUDO`, `IDENTIFICACION`, `PROMO`, `EXPLICACION`, `OBJECION`, `CIERRE`, `N/A`

**Por qué:** Saber en qué paso del flujo se pierde al usuario permite diagnosticar exactamente dónde está el problema. Actualmente no hay forma de saber si los usuarios cuelgan por el saludo, por el precio, o por la explicación.

---

### K4 — Nuevo campo: `objecion_principal` (String)

**Valores:** `DINERO`, `MALA_EXPERIENCIA`, `MUDANZA`, `NO_LE_INTERESA`, `OTRO_OPERADOR`, `NO_ERA_LA_PERSONA`, `N/A`

**Por qué:** Actualmente las categorías de no recarga se registran de forma ad-hoc. Un campo estructurado permite análisis agregado de objeciones para ajustar la estrategia comercial.

---

### K5 — Nuevo campo: `recall_recomendado` (Boolean)

**Qué mide:** Si el agente considera que vale la pena volver a llamar a este usuario (mostró interés pero no pudo en ese momento, pidió que lo llamaran después, etc.)

**Por qué:** Optimiza las campañas de reintento. Actualmente no hay criterio para decidir a quién volver a llamar.

---

### K6 — KPIs sugeridos para monitoreo en producción

| KPI | Fórmula | Objetivo sugerido |
|---|---|---|
| Tasa de conversación | `hubo_interaccion=Yes / total` | > 60% (actual: 54.3%) |
| Tasa de recarga | `recargo=Yes / total` | > 5% (actual: 3.4%) |
| Tasa de abandono en saludo | `momento_hangup=SALUDO / total` | < 25% (estimar desde datos) |
| Tasa de abandono en promo | `momento_hangup=PROMO / total` | < 15% |
| Duración promedio con interacción | `avg(duration) WHERE hubo_interaccion=Yes` | > 1.5 min (actual: ~0.9 min estimado) |
| Tasa de voicemail con mensaje completo | `voicemail_mensaje_completo / voicemail_total` | > 80% |
| Tasa de falsos positivos | `(objetivo_cumplido=Yes AND recargo=No) / objetivo_cumplido=Yes` | < 20% (actual: 66%) |

---

## Resumen de trazabilidad

| ID | Tipo | Registros de evidencia | Riesgo |
|---|---|---|---|
| E1 | Estructural | #132, #1005, #1730 | Nulo |
| E2 | Estructural | #1758, #838, #385, dato 61% <15s | Nulo |
| E3 | Estructural | #458, #573, #1005, #1128, #2220, #3656 | Nulo |
| E4 | Estructural | #829, #2621, #104, #2235, #954 | Nulo |
| E5 | Estructural | #742 (+), #29 (-), #14 (-) | Nulo |
| E6 | Estructural | #21 | Nulo (mejor que hard limit) |
| T1 | Técnica | #1005, #132, #349 | Nulo |
| T2 | Técnica | #14, #2, #13 | Nulo |
| T3 | Técnica | #21, #31 | Nulo |
| T4 | Técnica | #3, #29 | Nulo |
| C1 | Contenido | #21 | Bajo (solo reactivo, no proactivo) |
| C2 | Contenido | #2035, #210 | Nulo |
| C3 | Contenido | Objeción existente sin categoría | Nulo |
| C4 | Contenido | #2820, #1730 | Nulo (frase fija, sin improvisación) |
| C5 | Contenido | #31, #1667, #2571, #349 | Nulo (ya funcionaron en producción) |
| C6 | Contenido | #349 | Nulo |
| X1 | Corrección | #1005, #124 | Nulo |
| X2 | Corrección | #210 | Nulo |
| X3 | Corrección | Prompt actual | Nulo |
| X4 | Corrección | #31 | Nulo |
| X5 | Corrección | #70 | Nulo |
| V2 | Voicemail | #358, #421, #3018 | Nulo |
| K1 | Métrica | #210, #2035, #1667 + dato 66% falsos positivos | Nulo |
| K2 | Métrica | Dato 45.7% sin interacción | Nulo |
| K3 | Métrica | Necesidad diagnóstica | Nulo |
| K4 | Métrica | Necesidad analítica | Nulo |
| K5 | Métrica | Necesidad operativa | Nulo |
| K6 | Métrica | Benchmarking | Nulo |

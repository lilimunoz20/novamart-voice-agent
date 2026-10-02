# NovaMart — Lili, agente de voz conversacional

Agente de voz para atención a clientes construido sobre la plataforma **ElevenLabs Agents**, como parte del *Partner Deployment Engineer Lab*.

**Autora:** Liliana Gabriela Muñoz Rojas
**Partner:** Ts4 Strategy / Skala Global
**Octubre 2026**

---

## Qué es

**NovaMart** es una tienda en línea ficticia de electrónica y artículos para el hogar.
**Lili** es su asistente de voz: atiende cinco intenciones, habla español e inglés, y protege las acciones privilegiadas con una verificación de identidad estructural.

🔗 **Demo en vivo:** [enlace al sitio publicado]

---

## Las cinco intenciones

| Intención | Qué hace |
|---|---|
| Consulta de pedidos | Estado, artículos y fecha estimada vía `lookup_order` |
| Devoluciones | Explica la política y toma los datos para iniciar una |
| Productos | Responde comparaciones y recomienda del catálogo |
| Cambio de dirección | Acción privilegiada, solo tras verificar identidad |
| Escalación | Pasa a una persona cuando corresponde |

---

## Decisión de arquitectura principal

### El candado de verificación es estructural, no una regla del prompt

El brief exige que la verificación se aplique **estructuralmente**, no por instrucción. La implementación:

- La herramienta `update_shipping_address` existe **únicamente** en el nodo privilegiado del workflow. No está asignada a ningún otro nodo.
- Las dos entradas a ese nodo usan **expression edges**: leen directamente el resultado de `verify_identity`, no la interpretación del modelo.
- El contador de intentos vive en una variable dinámica (`update_state`), no en el prompt.

**Capa adicional:** `update_shipping_address` está duplicada para esta versión con el número de cuenta **amarrado a la cuenta verificada**. Aunque alguien lograra manipular la conversación para indicar otra cuenta, el cambio siempre se aplica sobre la que pasó la verificación.

### Workflow en lugar de Procedures

La plataforma ofrece dos paradigmas. Se eligió **workflow** porque el brief exige ruteo determinista sobre el resultado de la herramienta — en *procedures*, ese disparador lo evalúa el modelo. Los agentes de referencia del enablement pack usan workflow.

---

## Números

| | |
|---|---|
| Nodos del workflow | 19 |
| Conexiones | 36 |
| Conexiones deterministas (expression / result) | 10 |
| Reglas del prompt | 42 |
| Tests automáticos | 37 |
| Criterios de evaluación | 6 |
| Campos de recolección de datos | 11 |

---

## Más allá del brief

- **Multilingüe** español e inglés, con detección de idioma dominante
- **Manejo de spanglish**: no cambia de idioma por palabras sueltas
- **Política de menores**: detecta a quien se identifica como menor y pide un adulto antes de cualquier acción sobre una cuenta
- **Regla de dominio**: declina temas ajenos a NovaMart sin ofrecer conversación
- **Read-back de confirmación** antes de ejecutar el cambio de dirección
- **Verificación persistente** dentro de la misma llamada
- **Escalación agnóstica al canal**: intenta siempre y cae a un fallback natural si falla
- **Audio tags selectivos** solo en momentos con carga emocional real

---

## Archivos

| Archivo | Qué es |
|---|---|
| `index.html` | Sitio de NovaMart con el widget de voz embebido |
| `diagrama-candado.html` | Diagrama del candado de verificación |
| `guia-pruebas-lili.md` | Guía de 34 escenarios de prueba manual |

---

## Nota

NovaMart es una tienda ficticia creada para demostración. Los endpoints de herramientas provienen del enablement pack del programa y no se usan en ningún despliegue real.

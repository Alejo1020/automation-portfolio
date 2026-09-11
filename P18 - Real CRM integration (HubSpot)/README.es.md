# P18 — Integración real con CRM (HubSpot)

[🇬🇧 English](./README.md) | 🇪🇸 Español

## Problema

Hasta el proyecto 17, el pipeline de leads vivía en Google Sheets. Eso funciona para una demo, pero ningún cliente serio va a mover su operación de ventas fuera de su CRM real (HubSpot, Salesforce, Pipedrive). Si una automatización no puede hablar con el CRM que el negocio ya usa, la venta se cae ahí mismo.

## Solución

**Flujo:** Webhook → Claude Haiku califica el lead → parsea/limpia la respuesta → busca el contacto en HubSpot por email → crea o actualiza el contacto → si el lead es caliente, crea un Deal en una etapa custom del pipeline, asociado al contacto → responde al webhook con el resultado.

```
[Webhook: lead nuevo]
        ↓
[Claude Haiku 4.5] → califica lead (1-10), clasifica frío/tibio/caliente
        ↓
[Parsear respuesta] → limpia el JSON de Claude, lo une con los datos del lead
        ↓
[HubSpot: Buscar Contacto por email]
        ↓
   ┌────┴────┐
 existe?   nuevo?
   ↓          ↓
[Actualizar] [Crear]
   └────┬────┘
     [Merge]
        ↓
[IF: ¿lead caliente?]
   ┌────┴────┐
  sí          no
   ↓          ↓
[Crear Deal] │
asociado al   │
contacto      │
   └────┬─────┘
        ↓
[Respond to Webhook]
status, categoría, score, deal_creado
```

Un lead entra por webhook (WhatsApp, un formulario, cualquier origen externo), Claude lo califica, y termina en HubSpot como Contact — actualizado si ya existía, creado si no. Si el lead se califica como "caliente", se crea automáticamente un Deal en una etapa custom del pipeline ("Lead Caliente - Nuevo"), ya vinculado al contacto, listo para que un vendedor lo trabaje sin tocar una hoja de cálculo.

## Resultado de negocio

100% de los leads entrantes quedan en el CRM sin captura manual. Los leads calientes (score 8-9+ en las pruebas) generan una oportunidad de venta en el pipeline en segundos, en vez de horas o días de triage manual y captura de datos.

## Cómo se vende esto

*"Conecto tu sistema de captación de leads —WhatsApp, un formulario, lo que uses— directo a tu CRM, sin que nadie tenga que copiar y pegar nada. La IA califica el lead al instante y, si es una oportunidad real, aparece en tu pipeline de ventas listo para trabajar."*

**Cliente objetivo:** negocios pequeños/medianos que ya usan HubSpot (u otro CRM) y hoy dependen de captura manual de leads que llegan por WhatsApp, formularios o anuncios.

**Rango de precio:** $300-$600 USD por la integración inicial (un CRM, lógica de calificación, creación de deals), $50-150/mes por mantenimiento o para extenderlo a más fuentes de leads.

## Tech Stack

n8n · Claude API (Haiku 4.5) · HubSpot API (Private App) · Cloudflare Tunnel (exposición de webhook)

## Aprendizajes técnicos clave

- **Los Stage IDs de pipelines de HubSpot no son el nombre visible.** Se generan al crear la etapa — se obtienen vía `GET /crm/v3/pipelines/deals` o directo de la UI de configuración tras guardar.
- **La asociación Deal↔Contact vía API** usa `associations: [{ to: {id}, types: [{associationCategory: "HUBSPOT_DEFINED", associationTypeId: 3}] }]` dentro del mismo POST que crea el Deal. Ojo: el campo `num_associated_contacts` en la respuesta puede mostrar `0` aunque la asociación sí se haya guardado correctamente — verificar en la UI de HubSpot, no solo en el body de la respuesta.
- **Claude a veces envuelve el JSON en markdown** (```json) pese a instrucción explícita de no hacerlo. Siempre sanitizar con `.replace(/```json/g, '').replace(/```/g, '').trim()` antes de `JSON.parse()` — nunca confiar 100% en la instrucción del prompt.
- **El nodo Merge en modo "Append"** (no "Choose Branch") es la forma correcta de unificar dos ramas de un IF donde solo una ejecuta por corrida.
- **Después de un Merge con ramas condicionales que convergen**, preferir referencias explícitas a nodos (`$('NombreNodo')`) en vez de `$json` genérico antes de un nodo de respuesta final — `$json` ahí depende de cuál rama corrió al final, algo fácil de asumir mal.

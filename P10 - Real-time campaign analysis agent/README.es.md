# P10 — Agente de análisis de campañas con acceso a datos en tiempo real

[🇬🇧 English](README.md) | 🇪🇸 Español

## Problema

El dueño de un negocio con campañas activas no tiene tiempo de abrir dashboards ni esperar un reporte semanal. Necesita respuestas inmediatas a preguntas puntuales — "¿cómo va la campaña X?", "¿dónde estoy perdiendo plata?" — en el momento que se le ocurren, no cuando llega el reporte del lunes.

## Solución

Un agente conversacional de IA que recibe preguntas en lenguaje natural vía webhook, decide por sí solo si necesita consultar los datos reales de campañas, y responde en texto plano listo para WhatsApp.

**Arquitectura:**

```
Webhook (POST /analisis-campanas)
        │
        ▼
   AI Agent ── Chat Model: Claude Sonnet 5
        │  └── Tool: Google Sheets (lectura de datos de campañas)
        ▼
Respond to Webhook (texto plano)
```

El agente no sigue un flujo lineal fijo: razona sobre la pregunta y decide por sí mismo si llama la tool de Google Sheets o responde directo, basándose únicamente en la descripción de la tool.

## Resultado de negocio

Validado con dos tipos de pregunta distintos, sin reprogramar nada:

- **Pregunta de ranking** ("¿cuál campaña tiene mejor CTR?") — el agente calculó el CTR de cada campaña a partir de clicks/impresiones crudos y las ordenó correctamente.
- **Pregunta de optimización** ("¿qué campaña está desperdiciando más presupuesto?") — el agente derivó el costo por conversión (una métrica que no está directamente en los datos) y dio una recomendación concreta de reasignación.

Esto demuestra que el agente generaliza entre tipos de pregunta, en lugar de responder un solo caso predefinido.

## Cómo se vende

*"En vez de otro dashboard que tenés que abrir y leer, esto es un analista de IA al que le escribís por WhatsApp — responde con datos reales de tus campañas, al instante, todos los días."*

- **Cliente objetivo:** dueños de negocio pequeños/medianos con campañas en Meta Ads (concesionarios, clínicas, e-commerce) que revisan resultados de forma reactiva, no proactiva.
- **Rango de precio:** $150–$400 USD de setup + $30–$80 USD/mes de mantenimiento, según la complejidad de la fuente de datos (Sheets vs. API en vivo de la plataforma de ads).
- **Posicionamiento:** es la base técnica para sistemas multi-agente (P17) y el diferenciador entre "automatización básica en n8n" y "AI Automation Engineer".

## Stack técnico

n8n · Claude API (Sonnet 5) · Google Sheets · Cloudflare Tunnel (exposición local del webhook)

## Aprendizajes técnicos clave

1. El nodo AI Agent de n8n (LangChain) siempre devuelve la respuesta final en el campo fijo `$json.output` — no es configurable.
2. Sin instrucciones explícitas de formato, Claude por defecto usa markdown (tablas, negritas) y a veces expone su razonamiento intermedio ("espera, recalculo...") en la respuesta final — ambas cosas hubo que restringirlas explícitamente en el System Message para un output listo para WhatsApp.
3. La Tool Description es el mecanismo real de control: el agente decide cuándo llamar una tool basado en ese texto, no en el prompt del usuario — hay que ser preciso y específico ahí.
4. Claude Sonnet 5 es el punto óptimo costo/razonamiento para tool calling con decisiones multi-paso — confirmado en pruebas con una autocorrección visible en un cálculo de CTR.
5. Con `Respond to Webhook` en modo `Text`, el nodo trigger Webhook se puede reemplazar después por un trigger de Twilio WhatsApp sin tocar el resto del flujo — al agente no le importa ni sabe cuál es el canal de origen.

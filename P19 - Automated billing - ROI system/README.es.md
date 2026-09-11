# P19 — Sistema de Facturación/ROI Automático

[🇬🇧 English](README.md) | 🇪🇸 Español

## Problema

Un negocio de servicios que corre automatización (captación, calificación, agendamiento de leads) genera resultados operativos, pero nada de eso se traduce automáticamente en el único número que realmente le importa al dueño del negocio: **cuánta plata generó el sistema y cuánto costó operarlo.**

Un cliente que paga por automatización no quiere escuchar "el chatbot respondió 200 mensajes". Quiere escuchar: "el sistema generó $X en deals cerrados, costó $Y en operación, ROI de Z%". Ese es el reporte que justifica la mensualidad — y la evidencia que le permite a un freelancer subir precios con datos, no con promesas.

## Solución

Un workflow de n8n que convierte datos crudos de deals cerrados en un reporte financiero en lenguaje simple, entregado automáticamente según un horario fijo.

```
Schedule Trigger (semanal)
  → Get Rows (deals cerrados, cargados en Google Sheets)
  → Code (suma ingreso, resta costo operativo fijo, calcula ROI%)
  → HTTP Request → Claude Sonnet 5 (convierte números en resumen ejecutivo)
  → Edit Fields (extrae el texto de Claude + reconstruye datos referenciando el nodo)
  → Append Row (histórico de ROI)
  → Gmail (envía el reporte en HTML al dueño del negocio)
```

Los deals se cargan manualmente en un Google Sheet en vez de traerse de un CRM — una decisión de alcance deliberada, para entregar un sistema funcionando ya, con un camino limpio para conectar un CRM (ej. HubSpot) más adelante sin tocar el resto del flujo.

## Resultado de Negocio

- Reporte completamente automático una vez cargado un deal — cero trabajo manual de reporting después de eso
- Probado con números reales: $1,650 en ingresos, $50 en costo operativo, ROI de 3,200%, 2 deals
- Claude Sonnet detectó por sí solo un riesgo real de negocio — concentración del 100% de ingresos en un solo cliente — algo que un modelo más económico (Haiku) no señaló en la misma prueba
- Construye un histórico buscable y fechado de resultados en Sheets — un trackrecord que le podés mostrar al cliente en cualquier momento

## Cómo Se Vende

*"Además de automatizar tu captación de leads, te entrego un reporte financiero semanal: cuánto generó el sistema, cuánto costó operarlo, y una recomendación en lenguaje simple — no un dashboard técnico que nadie lee. Podés ver en cualquier momento si la automatización se está pagando sola, y yo tengo evidencia concreta para seguir optimizando tu cuenta."*

Se vende como add-on de $200-400/mes sobre cualquier paquete existente, o como parte de un paquete completo ($2,500-4,500+/mes) — es la pieza que convierte "tengo un bot" en "tengo un sistema con ROI medible".

## Stack Técnico

n8n (self-hosted, Docker) · Claude API (Sonnet 5) · Google Sheets · Gmail

## Aprendizajes Técnicos Clave

1. **`WEBHOOK_URL` también rompe OAuth, no solo webhooks.** n8n usa esa misma variable de entorno para armar el redirect de cualquier callback público, incluyendo el de Google OAuth2 — aunque OAuth no necesite exponerse a internet. Si la variable apunta a la URL del tunnel y el token de una credencial expira, la reautenticación falla con `redirect_uri_mismatch`, porque esa URL del tunnel nunca se registró en Google Cloud Console. Fix permanente: setear `N8N_EDITOR_BASE_URL=http://localhost:5678` en el contenedor — esto separa `WEBHOOK_URL` (solo webhooks externos, ej. Twilio/Telegram) del flujo de editor/OAuth (siempre localhost). Se confirma que funcionó cuando el log de arranque muestra `Editor is now accessible via: http://localhost:5678`.
2. **Decisión de alcance, no atajo:** entregar con carga manual en Google Sheets en vez de esperar a dominar una integración de CRM fue la decisión correcta — un sistema simple funcionando hoy supera a uno perfecto que todavía se está aprendiendo.
3. Reconfirma un patrón ya conocido de n8n: `$json` se resetea después de cualquier nodo HTTP Request. Se recuperaron los datos previos acá con `$('Code').item.json.campo` tras la llamada a Claude.
4. Una comparación directa Haiku vs Sonnet en la misma tarea confirmó el criterio de selección de modelo: Sonnet para contenido cara al cliente donde el insight importa, Haiku solo para clasificación simple.

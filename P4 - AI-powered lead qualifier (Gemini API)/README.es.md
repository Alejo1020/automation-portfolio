# Calificador de Leads con IA (Gemini API)

[🇬🇧 English](./README.md) | 🇪🇸 Español

## Problema

Los equipos de ventas pierden tiempo revisando manualmente cada lead que llega por WhatsApp o formularios web para decidir a cuál llamar primero. Los leads calientes se enfrían mientras esperan en la misma fila que los fríos y de bajo interés.

## Solución

Un workflow de n8n que recibe un lead vía webhook, usa IA para analizar el mensaje y clasificarlo (score 1-10, frío/tibio/caliente, con razón), registra cada lead en Google Sheets, responde de inmediato con el resultado de la calificación a quien llamó al webhook, y manda una alerta por WhatsApp solo cuando el lead es caliente.

**Arquitectura:**

```
Webhook (recibe el lead)
  → HTTP Request (Gemini API califica el lead)
  → Edit Fields (limpia markdown de la respuesta de IA)
  → Code (parsea el JSON, estructura datos del lead + score)
  → Append row in sheet (registra en Google Sheets)
     ├─→ Respond to Webhook (devuelve el resultado a quien llamó)
     └─→ If (categoria == "caliente")
           └─→ HTTP Request (alerta WhatsApp vía Twilio)
```

El webhook siempre responde de inmediato con el resultado de la calificación, sin importar la temperatura del lead. La notificación de WhatsApp corre en paralelo y solo se dispara para leads calientes — esto evita bloquear la respuesta y evita gastar notificaciones innecesarias.

## Resultado de negocio

- Tiempo de respuesta a lead caliente: horas → minutos
- 100% de leads calificados sin revisión manual
- Cero leads calientes perdidos por seguimiento lento
- Cero notificaciones desperdiciadas en leads fríos/tibios

## Cómo se vende

- Fee de setup único para armar e integrar el sistema al CRM/WhatsApp/formulario actual del cliente.
- Fee mensual de mantenimiento para ajuste de prompt, monitoreo y soporte.
- Pitch: *"Tu equipo de ventas ya no revisa leads uno por uno — el sistema te avisa por WhatsApp apenas entra uno caliente, con la razón exacta de por qué lo es."*
- Clientes objetivo: inmobiliarias, concesionarios y cualquier negocio con alto volumen de leads vía Meta Ads.

## Stack técnico

n8n (self-hosted) · Gemini API (free tier, fase de pruebas) · Google Sheets · Twilio WhatsApp Sandbox

## Aprendizajes técnicos clave

- Los nombres de modelo de IA cambian sin aviso — siempre verificar que el modelo vigente siga disponible antes de asumir que un nombre documentado funciona.
- El orden del nodo "Respond to Webhook" importa mucho. Conectarlo muy temprano en el flujo devuelve datos crudos/sin procesar a quien llamó. Debe ir después de tener los datos limpios, y antes de cualquier rama condicional (como un IF hacia WhatsApp) que no siempre se ejecuta — si no, el webhook puede hacer timeout para los leads que no toman esa rama.
- Un solo nodo puede conectarse a varios nodos en paralelo — "Append row in sheet" se conecta simultáneamente a "Respond to Webhook" y a "If", así la respuesta a quien llamó nunca depende de la rama condicional de WhatsApp.
- Twilio exige que "From" y "To" compartan el mismo prefijo de canal (`whatsapp:`) — un desajuste de canal devuelve el error 21910.
- "Retry On Fail" es esencial para APIs externas de IA — el free tier de Gemini se satura bajo carga, y sin retry automático cualquier pico de demanda rompe el flujo en producción.
- Vale la pena probar ambas ramas de un condicional (no solo el "camino feliz") — el bug de timeout del webhook solo se veía con leads fríos, no con calientes.

# P9 — Chatbot de ventas con RAG

🇬🇧 [Read in English](./README.md)

## Problema

Los negocios de servicios con alto volumen de preguntas repetitivas (precios, horarios, tratamientos, ubicación) pierden tiempo de personal respondiendo lo mismo una y otra vez por WhatsApp — y la demora en responder cuesta leads, sobre todo fuera de horario.

## Solución

Un chatbot de WhatsApp que responde preguntas de clientes usando RAG (Retrieval-Augmented Generation) apoyado en una base de conocimiento viva, para que el dueño del negocio pueda actualizar precios/horarios sin tocar código.

**Arquitectura:**

```
Mensaje de WhatsApp (Twilio)
        │
        ▼
   [ Webhook ]
        │
        ▼
[ Google Sheets ]  ← base de conocimiento (tratamientos, precios, horarios, ubicación, FAQ)
        │
        ▼
   [ Aggregate ]  ← junta todas las filas en un solo bloque de contexto
        │
        ▼
[ Gemini / LLM ]  ← responde SOLO con el contexto entregado; deriva a un
        │            asesor humano si la respuesta no está en los datos
        ▼
  [ Edit Fields ]  ← extrae la respuesta + restaura el número del remitente
        │
        ▼
[ API de Twilio ]  ← envía la respuesta de vuelta por WhatsApp
```

La base de conocimiento vive en una sola pestaña de Google Sheets. No se usa base de datos vectorial — para un catálogo chico y estático (10-20 filas), pasar el dataset completo como contexto en cada request es más simple, más barato, y más fácil de mantener para un dueño de negocio no técnico que gestionar embeddings.

## Resultado de negocio

- Respuesta instantánea, 24/7, sin gastar tiempo de personal en preguntas repetitivas
- Cero alucinaciones de precios o tratamientos — el modelo está limitado a la base de conocimiento y explícitamente instruido para derivar a un humano cuando no sabe
- Base de conocimiento editable por el dueño directo en Sheets, sin necesitar un desarrollador para actualizarla

## Stack técnico

n8n · API de WhatsApp de Twilio · Google Sheets · API de Gemini (Google AI Studio) · Cloudflare Tunnel

## Aprendizajes técnicos clave

- **RAG sin base de datos vectorial:** para catálogos chicos y estáticos, empaquetar todas las filas en un solo bloque de contexto (con un nodo Aggregate) es más simple y barato que usar embeddings/búsqueda vectorial — la complejidad extra no se justifica hasta que el dataset crece considerablemente.
- **Desfase de autenticación en el Webhook:** el nodo Webhook de entrada debe tener Authentication en `None` cuando el que llama es Twilio — Twilio no manda credenciales en el request entrante, así que Basic Auth rechaza todos los mensajes sin avisar por qué.
- **El `+` se pierde en la decodificación de URL:** Twilio manda el número del remitente como `x-www-form-urlencoded`, donde el `+` se interpreta como espacio al decodificar. Solución: `$json.body.From.replace(' ', '+')` antes de usar el número más adelante en el flujo.
- **Cada nodo HTTP Request necesita su propia credencial seleccionada explícitamente** — no se hereda de otros nodos aunque sea el mismo tipo de auth; esto causó un 401 silencioso en la llamada de salida a Twilio.
- **Registro de webhook desactualizado en n8n self-hosted:** un workflow publicado y activo puede dejar de recibir tráfico real de webhook ("unknown webhook" en los logs) después de varias ediciones del nodo, aunque el CLI lo reporte como activo. Solución: borrar y recrear el nodo Webhook desde cero, guardar, reactivar, y reiniciar el contenedor — un simple toggle o reactivación por CLI no siempre alcanza.

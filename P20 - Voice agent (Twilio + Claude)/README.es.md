# P20 — Agente de Voz con Twilio + Claude

[🇬🇧 English](./README.md) | 🇪🇸 Español

## Problema

Los negocios pierden leads y ventas cada vez que una llamada no se contesta — fuera de horario, en horas pico, o simplemente porque el equipo está ocupado. Cada llamada perdida es un cliente potencial que probablemente llama a la competencia. Contratar recepción 24/7 no es viable para la mayoría de las pymes.

## Solución

Un agente de voz que atiende llamadas telefónicas reales las 24 horas:

- **Sostiene una conversación natural** en español, con Claude Haiku 4.5 (elegido por su baja latencia, crítica en voz en tiempo real).
- **Recuerda el contexto** dentro de la misma llamada (nombre del que llama, lo que ya preguntó) usando Google Sheets como historial liviano, indexado por `CallSid`.
- **Decide su propia acción** en cada turno: seguir conversando, despedirse y colgar, o transferir a un humano — todo desde un único llamado a Claude que devuelve JSON estructurado (`{texto, accion}`).
- **Construido enteramente sobre n8n + Twilio + Claude**, sin infraestructura de telefonía adicional.

### Arquitectura

```
Llamada entra → Webhook 1 → Say + Gather (saluda, escucha)
     ↓ (usuario habla, Twilio transcribe)
Webhook 2 → Lee historial (Sheets) → Arma mensajes → Claude Haiku
     ↓
Parsea {texto, accion} → Arma TwiML según acción → Guarda turno en Sheets → Responde a Twilio
     ↓
continuar → vuelve a escuchar (loop) | despedir → cuelga | transferir → marca a un humano
```

**Por qué Twilio + n8n en vez de una plataforma de voz-IA dedicada:** control total sobre la lógica de conversación, sin sobrecosto de plataforma más allá de las tarifas propias de Twilio, y se integra directo al mismo stack n8n/Claude usado en el resto de este portafolio.

## Resultado de negocio

Prueba en vivo verificada de punta a punta: el agente saludó, entendió una pregunta hablada sobre horarios de atención, y — en un turno posterior — recordó correctamente el nombre del usuario sin que se lo repitieran, confirmado en el log de conversación guardado en Google Sheets.

*"Tu negocio nunca más pierde una llamada. Este agente contesta 24/7, entiende lo que le preguntan, recuerda el contexto de la conversación, y sabe cuándo pasarte la llamada a vos porque el cliente ya está listo para cerrar — o cuándo despedirse solo porque la consulta ya quedó resuelta. No es un IVR de menú numérico: es una conversación real."*

## Stack técnico

n8n · Claude API (Haiku 4.5) · Twilio Programmable Voice · Google Sheets · Cloudflare Tunnel

## Aprendizajes técnicos clave

1. **Twilio espera TwiML (XML), no JSON** como respuesta del webhook — usar `Respond With: Text` (nunca `JSON`) más un header explícito `Content-Type: text/xml`, o Twilio tira `Invalid JSON in Response Body`.
2. **Arquitectura de loop de conversación:** dos webhooks (entrada de llamada + transcripción) en ping-pong hasta que una acción de hangup o transferencia cierra el bucle.
3. **Memoria por llamada sin usar un nodo Memory:** Google Sheets filtrado por `CallSid` como session key — mismo patrón conceptual que un nodo Memory de n8n, construido a mano para control total sobre qué se guarda.
4. **Las lecturas vacías de Google Sheets no se propagan por default** en n8n — hay que activar `Always Output Data` en el nodo de lectura, y filtrar ese item vacío en el siguiente paso, o el HTTP Request a Claude falla con `messages.0.role: Field required`.
5. **Escapar comillas en un system prompt embebido en un body JSON:** las comillas dobles internas deben escaparse con `\"` o el propio parseo JSON del nodo se rompe antes de que el request llegue a la API.
6. **Restricciones de la cuenta trial de Twilio:** gratis, sin tarjeta, pero limitada a 5 números destino verificados — esto restringe probar transferencias `<Dial>` reales a un tercero sin un segundo número verificado.

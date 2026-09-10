# Notificador de Leads en Tiempo Real por WhatsApp

🇬🇧 [Read in English](README.md)

## Problema

Los negocios que capturan leads por formularios web (inmobiliarias, concesionarios, clínicas, firmas de abogados) suelen tardar horas —a veces días— en hacer seguimiento, porque alguien tiene que revisar manualmente un correo o CRM. Estudios de tiempo de respuesta muestran que contactar a un lead en los primeros 5 minutos aumenta drásticamente la conversión, mientras que responder después de 30 minutos hace que la conversión caiga fuertemente.

## Solución

Una automatización que recibe cualquier lead entrante (formulario web, landing page, Facebook Ads) vía webhook y notifica instantáneamente por WhatsApp al vendedor asignado —con nombre, teléfono, mensaje y origen del lead— listo para llamar en segundos.

**Arquitectura:**

```
[Webhook] → [Edit Fields] → [HTTP Request: Twilio WhatsApp API]
   ↓               ↓                      ↓
Recibe         Arma el texto         Envía la
el lead        para WhatsApp         notificación
```

1. **Webhook** — captura los datos del lead entrante (nombre, teléfono, mensaje, origen).
2. **Edit Fields** — formatea los datos en un solo mensaje listo para WhatsApp y separa el número de teléfono destino.
3. **HTTP Request** — llama a la API de Twilio para enviar el mensaje de WhatsApp al vendedor en tiempo real.

## Resultado de negocio

- Tiempo de respuesta al lead: de horas → segundos
- Cero leads perdidos por "se me olvidó revisar el correo"
- Escala de 1 a 100+ leads/día sin trabajo manual adicional


 *"¿Cuántos leads están perdiendo porque nadie los contactó a tiempo? Este sistema le avisa a tu equipo de ventas por WhatsApp en el segundo que entra un lead nuevo."*


## Stack técnico

- n8n (self-hosted, Docker)
- API de WhatsApp de Twilio (sandbox para pruebas / WhatsApp Business API para producción)
- Trigger basado en Webhook (compatible con cualquier formulario o landing page)

## Aprendizajes técnicos clave

- **Restricción del sandbox de WhatsApp:** el sandbox de Twilio solo entrega mensajes a números que hayan mandado manualmente el mensaje `join <código>` primero. Es una limitación exclusiva de testing — producción requiere un número de WhatsApp Business API verificado para mensajear a cualquier destinatario libremente.
- **El Content Type del Body importa:** la API de Twilio espera `Form-Urlencoded`, no JSON. Mandar JSON causa fallos silenciosos ("To phone number is required") incluso con el campo lleno en la UI del nodo, si el toggle "Send Body" está apagado.
- **Resolución DNS intermitente (`ENOTFOUND api.twilio.com`):** se resolvió limpiando la caché DNS local y cambiando al DNS público de Google (8.8.8.8 / 8.8.4.4) — era un problema de red local, no de configuración de n8n o Twilio.

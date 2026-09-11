[🇬🇧 English](README.md) | 🇪🇸 Español

# P11 — Sistema de Nutrición de Leads Automatizada

## Problema

Un lead califica caliente, tibio o frío (ver P4) — pero si nadie le escribe en las siguientes horas, se enfría y se pierde. La mayoría de negocios pequeños (concesionarias, clínicas, inmobiliarias, firmas legales) no tienen capacidad humana para hacer seguimiento sistemático a cada lead. El follow-up depende de que un vendedor se acuerde, y en la práctica, no se acuerda. Leads calientes que no se contactan en las primeras horas caen dramáticamente en probabilidad de conversión — y los tibios/fríos simplemente se olvidan.

## Solución

Sistema automatizado que ejecuta secuencias de nutrición diferenciadas por categoría de lead, generando mensajes de WhatsApp personalizados con IA según el contexto (nombre, categoría, punto de la secuencia):

- **Caliente:** contacto inmediato (0h), seguimiento a las 4h, y a las 24h — tono de urgencia, empuja a cerrar.
- **Tibio:** 24h, 72h, 7 días — tono educativo, genera confianza sin presionar.
- **Frío:** 7, 21 y 45 días — presencia suave, mantiene el contacto vivo sin saturar.

**Arquitectura (n8n):**

```
Schedule Trigger (cada hora)
  → Get rows in sheet (lee leads con estado = activo)
  → Code: evalúa categoría + etapa vs. horas transcurridas
  → Claude API (genera mensaje personalizado por lead)
  → Code: extrae el texto del mensaje
  → Twilio (envía WhatsApp)
  → Edit Fields (rescata row_number y siguiente etapa)
  → Update row in sheet (avanza etapa, marca completado si aplica)
```

**Base de datos:** Google Sheet dedicada (`Leads_Nutricion`) con columnas: nombre, teléfono, categoría, fecha_ingreso, etapa_nutricion, estado, ultima_actualizacion.

## Resultado de negocio

Sistema probado end-to-end con 10 leads de prueba distribuidos en las 3 categorías y distintas etapas. Resultados:
- Identificó correctamente a quién le tocaba mensaje según tiempo transcurrido (8 de 10 leads activos dispararon mensaje; 2 quedaron correctamente excluidos — uno por estar detenido, otro por no cumplir aún el umbral).
- Generó mensajes con tono claramente diferenciado por categoría.
- Envió los 8 mensajes por WhatsApp vía Twilio sin errores.
- Actualizó las 8 filas correspondientes, avanzando etapa y marcando `completado` a los que ya agotaron su secuencia (3 leads).

Cero intervención manual en el ciclo evaluar → generar → enviar → actualizar.


*"¿Cuántos leads se te enfrían porque nadie alcanza a escribirles a tiempo? Este sistema hace seguimiento automático a cada lead según qué tan interesado está — mensaje inmediato si es caliente, educación progresiva si es tibio, presencia constante sin fastidiar si es frío. Todo generado con IA, personalizado por nombre y contexto, sin que nadie tenga que acordarse de escribir. Se integra directo con tu WhatsApp Business."*


## Stack técnico

n8n · Claude Sonnet 5 (API de Anthropic) · Twilio (sandbox de WhatsApp) · Google Sheets

## Aprendizajes técnicos clave

1. **Twilio + Body Content Type:** debe ser explícitamente "Form Urlencoded", no JSON — si no, Twilio ignora los parámetros y tira error "To phone number is required" aunque el campo esté bien lleno.
2. **Reset de JSON en cadenas de HTTP Requests:** cuando dos HTTP Request nodes están en cadena (Claude → Twilio), cualquier dato necesario del primero se pierde en el segundo. Solución: nodo Edit Fields intermedio que rescate explícitamente vía `$('NodeName').item.json.campo` antes de que se necesite — no reaccionar después del error.
3. **Matching por row_number en Google Sheets Update:** más confiable que columnas con posibles duplicados (ej. teléfono en datos de prueba), pero row_number debe mapearse explícitamente como columna con su valor, no solo seleccionarse como matching column — de lo contrario el nodo no sabe qué fila tocar.
4. **Modelo confirmado:** `claude-sonnet-5` vía Header Auth (`anthropic-version: 2023-06-01`, `content-type: application/json`) — no confundir con `claude-sonnet-4-6`, que es la generación anterior.

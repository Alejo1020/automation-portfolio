🇬🇧 [Read in English](README.md)

# Clínica Renova — Automatización End-to-End de un Negocio de Servicios

**Tipo:** Sistema de automatización con IA (n8n + Claude Sonnet 5)
**Tipo de cliente:** Clínicas estéticas/médicas, consultorios dentales, negocios de servicios
**Complejidad:** Alta — 6 bloques integrados, ciclo de vida completo del lead

---

## Problema

Los negocios de servicios como una clínica estética pierden dinero en las costuras entre sistemas: leads que entran por WhatsApp o formulario web y nadie califica a tiempo, citas que se agendan mal o se duplican, leads tibios que se enfrían por falta de seguimiento, y un dueño que se entera del estado real del negocio —facturación, ROI, errores del sistema— solo cuando ya es tarde, sin visibilidad diaria ni alertas automáticas.

## Solución

Un sistema único de 6 bloques que orquesta todo el ciclo de vida del lead, de punta a punta, sin intervención manual:

1. **Entrada + Calificación** — Dos canales (WhatsApp vía Twilio + formulario web) convergen en un flujo normalizado que Claude Sonnet 5 califica automáticamente (score, categoría, razón).
2. **Agendamiento** — Leads calientes/tibios pasan a un AI Agent con herramientas propias (consultar agenda, agendar cita) que propone y confirma horario sin doble-booking, respetando el horario real del negocio.
3. **Nutrición** — Leads fríos reciben un mensaje de valor generado por IA (sin presión de venta) vía WhatsApp, con seguimiento registrado.
4. **Reporting diario** — Cada mañana a las 8am, el dueño recibe un email con las métricas del día anterior y un resumen ejecutivo de IA que señala qué requiere atención (ej. leads calientes sin agendar).
5. **Facturación/ROI semanal** — Cada lunes, el sistema calcula ingreso vs. costo operativo, y Claude interpreta el resultado en lenguaje de negocio, con causa probable y acción recomendada si el ROI es negativo.
6. **Manejo de errores** — Un workflow independiente detecta automáticamente cualquier fallo en cualquier bloque y alerta por email de inmediato, para que ningún lead se pierda por un error silencioso.

## Arquitectura

```
Bloque 1: Entrada + Calificación
  Webhook WhatsApp ─┐
                     ├─> Merge → Normalizar → Claude califica lead → Guardar en Sheet
  Webhook Formulario ─┘

Bloque 2: Agendamiento (leads calientes/tibios)
  AI Agent (Claude Sonnet 5 + tools: consultar agenda, agendar cita)

Bloque 3: Nutrición (leads fríos)
  Claude genera mensaje de valor → Envío WhatsApp → Registro en Sheet

Bloque 4: Reporting Diario (Schedule Trigger, 8am)
  Leer 3 Sheets → Merge → Consolidar métricas → Resumen Claude → Email

Bloque 5: Facturación/ROI Semanal (Schedule Trigger, Lunes 8am)
  Leer citas → Calcular ROI → Claude interpreta → Email

Bloque 6: Manejo de Errores (workflow independiente)
  Error Trigger → Armar mensaje de alerta → Email
```

## Resultado de Negocio

- Cero intervención manual entre "llega un lead" y "el dueño recibe un reporte accionable de su negocio"
- Visibilidad diaria y semanal que hoy la mayoría de negocios de servicios de este tamaño no tiene
- Sistema autodiagnosticado: si algo falla, el dueño se entera en minutos, no en días
- Arquitectura reutilizable: el mismo esqueleto (calificar → agendar/nutrir → reportar → facturar → vigilar) sirve para firma legal, inmobiliaria, consultorio, spa — cualquier negocio de servicios con agenda

## Pitch de Venta

> "No te vendo un chatbot. Te instalo el sistema nervioso de tu negocio: cada lead que te escribe se califica solo, se agenda solo si está listo, se nutre solo si no lo está, y cada semana te llega a tu correo un resumen que te dice exactamente qué está funcionando y qué necesita tu atención — sin que tengas que abrir una sola hoja de cálculo."

Encaja directo en un paquete de precio medio-alto ($1.200–2.500+/mes): son los 4 módulos de servicio (contenido/mensajería/leads/dashboard) funcionando integrados, no vendidos por separado.

## Stack Técnico

- **Orquestación:** n8n (self-hosted, Docker)
- **IA:** Claude Sonnet 5 (API de Anthropic) — usado para calificación de leads, razonamiento de agendamiento, generación de mensajes de nutrición, reporting ejecutivo e interpretación de ROI
- **Canales:** Twilio (WhatsApp), Formulario web (webhook)
- **Datos:** Google Sheets (leads, citas, registro de nutrición, histórico de ROI)
- **Notificaciones:** Gmail (reporte diario, reporte semanal de ROI, alertas de error)

## Aprendizajes Técnicos Clave

1. **Toggle Fixed vs. Expression** — En cualquier campo de texto de n8n (body de HTTP, system message de AI Agent), tanto el editor `fx` como el toggle "Fixed | Expression" deben estar independientemente en modo Expression. Tener uno sin el otro manda strings literales sin evaluar, de forma silenciosa.
2. **Leer un Sheet vacío detiene la ejecución** — Un nodo "Get row(s)" de Google Sheets sobre un Sheet vacío devuelve 0 items y, por default, n8n detiene todo el workflow ahí. Fix: activar "Always Output Data" en Settings del nodo.
3. **Los AI Agents corren una vez por cada item recibido** — Nunca conectar un nodo multi-fila (ej. una lista completa de citas) directo al input principal de un AI Agent — dispara ejecuciones duplicadas del agente (y costos duplicados de API) una vez por cada fila. Los datos que el agente necesita consultar van en un Tool que invoca bajo demanda, no en la cadena principal.
4. **Los nombres exactos de nodo importan** — n8n autonumera nodos con nombres duplicados (ej. "Edit Fields" vs. "Edit Fields1"). Referenciar el nombre equivocado causa un `undefined` silencioso sin error visible.
5. **Merge antes de consolidar ramas paralelas** — Cuando 3+ ramas paralelas (ej. varios Sheets leídos en paralelo) confluyen a un solo nodo Code, siempre insertar un nodo Merge (modo Append) antes. Referenciar cada nodo paralelo directo por nombre en el Code es poco confiable — solo una rama puede registrar como "ejecutada".
6. **Los prompts son iterativos** — Un prompt de IA rara vez funciona perfecto al primer intento. Dos o tres rondas de refinamiento después de ver el output real (ej. quitar una pregunta de seguimiento que Claude agregó y que no tiene sentido en un email automático) es normal, no señal de error.

---

*Construido como parte de un roadmap de portafolio de 29 proyectos de automatización, avanzando desde fundamentos de n8n hasta sistemas multi-agente con integración real del protocolo MCP.*

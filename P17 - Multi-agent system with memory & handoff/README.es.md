# P17 — Sistema multi-agente con memoria y handoff

[🇬🇧 English](./README.md) | 🇪🇸 Español

## Problema

Un negocio de servicios (clínica, concesionario, firma legal) recibe mensajes que necesitan atención muy distinta: preguntas de precios, agendamiento de citas, o reclamos. Un solo chatbot genérico maneja esto mal — no recuerda contexto entre temas y no puede ejecutar acciones reales (agendar, modificar, cancelar).

El manejo manual no escala y es caro. Un sistema multi-agente resuelve esto igual que lo haría un equipo humano: un "recepcionista" que rutea la conversación, y especialistas que atienden cada tipo de solicitud con su propio criterio y sus propias herramientas.

## Solución

**Arquitectura:**
```
Telegram Trigger → Normalizar Input → Orquestador (Claude Haiku 4.5, con memoria)
    ↓
Switch (handoff según clasificación)
    ├── Agente Ventas (Claude Sonnet 5, memoria)
    ├── Agente Agendamiento (Claude Sonnet 5, memoria + 3 tools: leer / crear / actualizar cita en Sheets)
    └── Agente Soporte (Claude Haiku 4.5, memoria, escala a humano)
    ↓
Telegram Send Message (respuesta unificada al cliente)
```

**Piezas clave:**
- **Orquestador** clasifica cada mensaje en `ventas` / `agendamiento` / `soporte` usando el historial de conversación, sin actuar como asistente conversacional (restricción de output explícita en el system prompt).
- **Agente de Agendamiento** con 3 herramientas de Google Sheets: primero consulta si el cliente ya tiene una cita (Get Rows), y según el resultado decide entre crear (Append) o actualizar (Update Row) — nunca asume.
- **Memoria por usuario**: cada uno de los 4 agentes usa `Simple Memory` con Session Key = `usuario_id` (chat.id de Telegram), así cada cliente tiene su hilo de contexto aislado.
- **Estrategia de modelo mixto por costo/complejidad**: Haiku 4.5 para clasificación y soporte (tareas simples), Sonnet 5 para ventas y agendamiento (razonamiento y uso de tools).

## Resultado de negocio

- Un cliente puede agendar, cambiar de opinión, y modificar su cita **en la misma conversación**, sin repetir datos — el sistema recuerda una cita existente y la actualiza en vez de duplicarla.
- El bot nunca inventa una confirmación falsa: si una herramienta falla, lo admite y ofrece alternativa (comportamiento verificado en pruebas).
- Arquitectura agnóstica de canal: hoy corre en Telegram, migrar a WhatsApp/Twilio es un swap de un solo nodo de entrada/salida — la lógica de agentes no cambia.

## Cómo se vende esto

**Pitch corto:** *"Un sistema de atención por WhatsApp/Telegram con 3 especialistas de IA (ventas, agendamiento y soporte) que se pasan la conversación entre sí sin que el cliente repita nada, y que agenda o modifica citas directamente en tu sistema — no solo responde preguntas, ejecuta acciones."*

**A quién:** clínicas, concesionarios, firmas legales — cualquier negocio con flujo de citas + consultas comerciales + reclamos ocasionales.

**Argumento de venta fuerte:** la arquitectura de modelo mixto (Haiku para lo simple, Sonnet para lo que requiere razonamiento con tools) demuestra conciencia de costo operativo, no solo "hacer que funcione" — eso diferencia a un automatizador junior de un AI Automation Engineer.

**Precio de referencia:** este tipo de sistema (multi-agente + integración de agenda) se ubica en el rango alto de un paquete de automatización — sugerido $400-700 USD de setup + retainer mensual por ajustes de prompts/mantenimiento.

## Tech Stack

n8n (self-hosted, Docker) · Claude API (Sonnet 5 + Haiku 4.5) · Telegram Bot API · Google Sheets · Cloudflare Tunnel

## Aprendizajes técnicos clave

- **`.item` vs `.first()`:** después de un nodo `Switch` (o cualquier nodo que ramifique/rompa el flujo lineal), el linking por posición de `.item` puede fallar. Usar `$('NombreNodo').first().json.campo` en cualquier nodo posterior a un Switch — incluidos los Session Key de Memory y los prompts de Agent.
- **Los nodos de Memory son sub-nodos**, no forman parte del flujo principal de datos — son especialmente sensibles al problema anterior.
- **Un orquestador con memoria puede "actuar" en vez de clasificar** si su contexto incluye turnos conversacionales de otros agentes — requiere una restricción de output explícita y dura en el system prompt, no alcanza con pedirlo una vez.
- **Un agente con tools puede alucinar el éxito de una acción** (ej. decir "reemplacé tu cita") si no tiene realmente la herramienta disponible — el fix real es dar la tool faltante y forzar un flujo estricto de leer → decidir → actuar, no solo ajustar el prompt.
- **Google Sheets Tool — Update Row** requiere que `columns.matchingColumns` quede seteado explícitamente — elegir la columna en el selector visual por sí solo no siempre alcanza.
- **Telegram Trigger requiere `WEBHOOK_URL`** como variable de entorno del contenedor Docker, apuntando a la URL pública activa del tunnel — cada vez que cambia la URL de Cloudflare Tunnel, hay que recrear el contenedor con la URL nueva.
- **Configurar `appendAttribution: false`** en el nodo Telegram Send Message para quitar el footer "sent automatically with n8n".

## Limitación conocida (documentada, no oculta)

Al momento de este cierre, la rama del Agente de Ventas todavía tiene un par de detalles sin resolver del build original — su expresión de Session Key de memoria y el texto del prompt no se actualizaron con el mismo fix `.item` → `.first()` aplicado al resto de ramas. Se deja explícito acá a propósito, como parte de mostrar profundidad de debugging en vez de presentar un sistema "perfecto".

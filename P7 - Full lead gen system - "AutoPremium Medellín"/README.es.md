# P7 — Sistema completo de lead gen para AutoPremium Medellín (concesionario ficticio)

[🇬🇧 English](./README.md) | 🇪🇸 Español

## Problema

Un concesionario recibe leads por múltiples canales (formulario web + WhatsApp) sin ninguna priorización. Los vendedores pierden tiempo revisando lead por lead en vez de atacar primero al que está listo para comprar. Los leads que no están listos (tibios/fríos) se contactan una vez y quedan abandonados — sin seguimiento, sin reactivación. El dueño no tiene visibilidad diaria de volumen ni calidad de leads sin entrar manualmente al Sheets.

## Solución

Dos subsistemas conectados, construidos como workflows separados de n8n que comparten el mismo backend de Google Sheets.

### P7a — Captación + Calificación
```
Webhook (Form) ──┐
                  ├─→ Merge (Append) ─→ Gemini 3.6 Flash (score + categoría)
Webhook (WhatsApp)┘        ↓
                    Limpiar JSON → Parsear → Guardar en Sheets
                            ↓
                    IF categoría = "caliente" → Alerta WhatsApp al vendedor
```
Captura leads de dos canales con formatos nativos distintos (JSON del formulario web, `x-www-form-urlencoded` de Twilio), normaliza ambos a un esquema común, envía el mensaje a Gemini para calificación (score 1-10) y categorización (frío/tibio/caliente), registra todo en Sheets, y dispara una alerta instantánea por WhatsApp solo para leads calientes.

### P7b — Seguimiento + Reporting
```
Schedule (9am diario) → Sheets → Filter (no caliente, sin contactar)
   → Gemini redacta mensaje corto de seguimiento → WhatsApp → marcar "contactado" en Sheets

Schedule (diario) → Sheets → Filter (leads de hoy)
   → Nodo Code calcula métricas (totales, por categoría, por canal) → Email de reporte al dueño
```
Reactiva automáticamente leads fríos/tibios que nunca recibieron seguimiento, y le manda al dueño del concesionario un resumen diario en HTML por correo sin que nadie tenga que abrir el spreadsheet.

## Resultado de negocio

- Cero leads tibios/fríos abandonados sin intento de seguimiento.
- Triage instantáneo: el vendedor se entera en el momento en que llega un lead caliente, en cualquiera de los dos canales.
- El dueño tiene visibilidad diaria (volumen, desglose por calidad, split por canal) sin esfuerzo manual de reporting.
- Totalmente demostrable de punta a punta: captación → calificación con IA → nutrición → reporting.


## Stack técnico

n8n · Gemini 3.6 Flash API (dev/pruebas) · Claude API (objetivo de producción) · Twilio WhatsApp · Google Sheets · Gmail · Cloudflare Tunnel

## Aprendizajes técnicos clave

1. **Multicanal ≠ mismo formato.** Cada fuente manda los datos como quiere: tu propio formulario → JSON (lo controlás vos), Twilio → `x-www-form-urlencoded` con campos fijos (te adaptás a su estándar). Regla: *si el emisor es tuyo, elegís el formato; si es un tercero, te adaptás al de ellos.*
2. **Normalizar antes de unificar.** Dos formatos de payload distintos no pueden entrar directo a un nodo `Merge` — cada canal necesita su propio `Edit Fields` que lo traduzca a un esquema común antes de converger.
3. **`Merge` con Append, no combinación por índice**, cuando las entradas llegan desincronizadas en el tiempo — Append simplemente apila los items sin importar el orden de llegada.
4. **Bug real de producción, no de laboratorio**: un prefijo `+57` duplicado (`+57+573205020526`) pasó porque cada canal dejaba el teléfono en un formato distinto antes de llegar al mismo nodo downstream. Las inconsistencias de formato se arreglan en la normalización, no con un parche en el nodo final.
5. **Un nodo deshabilitado deja pasar todo el input sin filtrar** — no bloquea nada. Un `If` deshabilitado parece un filtro funcionando hasta que revisás su output real; siempre verificá el estado del nodo cuando una lógica que "debería" filtrar no filtra.
6. **La salida de un LLM es texto, no datos.** Limpiar backticks de markdown + `JSON.parse()` es lo que convierte la salida del modelo en campos usables — pero cuando solo necesitás prosa (como un mensaje de seguimiento), saltate el paso de parseo por completo y pedí texto plano.
7. **`$('NodeName').item.json`** permite traer campos de cualquier nodo anterior del flujo, no solo el inmediatamente anterior — útil cuando la cadena incluye una llamada a IA en el medio.
8. **La matemática determinística no necesita un LLM.** Las métricas diarias (conteos, desgloses) se calculan con un nodo `Code` plano — no hay razón para gastar tokens de IA sumando números.
9. **Un mismo workflow puede alojar dos ramas independientes de `Schedule Trigger`** que nunca interactúan entre sí — forma limpia de mantener automatizaciones relacionadas (seguimiento + reporting) en un solo JSON sin que se interfieran.

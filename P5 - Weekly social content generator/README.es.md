# P5 — Generador de Contenido Semanal para Redes

🇬🇧 [Read in English](./README.md)

## Problema

Los negocios pequeños saben que necesitan estar activos en redes sociales, pero rara vez tienen el tiempo o la disciplina para producir contenido de forma constante. Resultado: cuentas abandonadas, pérdida de visibilidad y presencia de marca inconsistente.

## Solución

Un workflow de n8n que genera automáticamente 30 días de contenido listo para publicar (2 posts/día: Instagram + LinkedIn) a partir de un calendario de temas precargado, sin trabajo manual una vez configurado.

**Flujo:**

```
Schedule Trigger (diario)
        │
        ▼
Get row(s) in sheet  ──> compara la fecha de hoy con el calendario de contenido
        │
        ▼
HTTP Request (Gemini) ──> genera borradores de IG + LinkedIn en JSON
        │
        ▼
Code (JS)             ──> parsea el string JSON anidado de Gemini
        │
        ▼
Update row in sheet   ──> escribe ambos posts + estado, buscando por fecha
```

1. **Schedule Trigger** — se dispara una vez al día a una hora fija.
2. **Get row(s) in sheet** — busca en Google Sheets la fila donde `Fecha` coincide con hoy, usando matching por fecha en vez de número de fila para que el flujo no se desincronice si se salta un día.
3. **HTTP Request → Gemini API** (`gemini-flash-latest`, free tier) — envía el tema del día y devuelve dos borradores (post casual para IG, post profesional para LinkedIn) en JSON estructurado.
4. **Code** — parsea el string JSON anidado que devuelve Gemini y arma un objeto limpio.
5. **Update row in sheet** — sobreescribe la fila del día con `Post_IG`, `Post_LinkedIn` y `Estado = Generado`, buscando por fecha.

## Resultado de negocio

- 30 días de contenido para 2 plataformas = 60 piezas generadas sin trabajo manual diario.
- Tiempo estimado ahorrado: ~5–8 horas/mes que un negocio pequeño gastaría redactando posts.
- Cero intervención humana una vez cargado el calendario de temas.


## Stack técnico

n8n (self-hosted) · Google AI Studio — Gemini API (free tier, fase de pruebas) · Google Sheets

## Aprendizajes técnicos clave

1. **Matching por columna real (fecha), no por número de fila** — mismo patrón usado en el calificador de leads (P4); evita desincronización si el flujo se pausa o las filas se reordenan.
2. **El formato de fecha debe coincidir exacto** entre el Sheet y `$now.format('yyyy-MM-dd')` — un desfase devuelve cero filas silenciosamente, sin error claro.
3. **Los alias "latest" de modelo pueden resolver a versiones inesperadas** — `gemini-flash-latest` resolvió a `gemini-3.6-flash` en las pruebas. Siempre revisar `modelVersion` en la respuesta para confirmar qué está corriendo realmente.
4. **La respuesta de Gemini es un string JSON anidado dentro de JSON** — `candidates[0].content.parts[0].text` requiere un segundo `JSON.parse()` para extraer el contenido real de los posts.
5. **El modo "Using JSON"** en headers/body del nodo HTTP Request es más rápido y menos propenso a error que configurar campo por campo cuando el payload tiene estructura anidada.

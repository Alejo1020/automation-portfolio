# P12 — Dashboard Inteligente con IA que Interpreta Métricas y Recomienda Acciones

[🇬🇧 English](./README.md) | 🇪🇸 Español

## Problema

Los dueños de negocios de servicios (clínicas, concesionarios, inmobiliarias) recolectan métricas semanales — leads, ventas, ingresos, gasto en marketing, satisfacción del cliente, tickets de soporte — pero la mayoría no tiene el tiempo ni la expertise analítica para interpretarlas. Una hoja de cálculo llena de números no les dice qué hacer. El resultado: decisiones lentas basadas en intuición en vez de datos, y problemas que pasan desapercibidos hasta que ya afectaron los ingresos.

## Solución

Un sistema automático semanal que lee el histórico de métricas del negocio, hace que un LLM razone sobre la tendencia completa (no solo el último dato) y entrega un diagnóstico en lenguaje simple con acciones concretas — directo al correo del dueño.

**Arquitectura:**

```
Schedule Trigger (Lunes 8am)
      │
      ▼
Google Sheets — Trae 8 semanas de métricas
      │
      ▼
Aggregate — junta todas las filas en un array
      │
      ▼
HTTP Request — Claude API (Sonnet)
  Analiza la tendencia completa, devuelve JSON estructurado:
  { diagnostico, alerta_principal, recomendaciones[] }
      │
      ▼
Edit Fields — extrae el texto de la respuesta de Claude
      │
      ▼
Code — parsea el JSON string en campos separados
      │
      ▼
Google Sheets — guarda en historial de análisis
      │
      ▼
Gmail — envía reporte HTML formateado al dueño
```

**Cómo funciona:**
1. Cada lunes a las 8am, el flujo trae las últimas 8 semanas de métricas del negocio desde Google Sheets.
2. Todas las filas se agregan en un solo array para que la IA detecte *tendencias*, no analice semanas aisladas.
3. Claude API recibe el dataset completo y devuelve un diagnóstico estructurado: tendencia general, el problema más urgente, y 3 acciones concretas — en JSON limpio, sin texto de relleno.
4. La respuesta se parsea y se guarda en una hoja de historial, construyendo un registro de consultoría IA a lo largo del tiempo.
5. Un correo HTML formateado entrega el análisis directo al dueño del negocio — sin dashboard que abrir, sin números que interpretar.

## Resultado de Negocio

Usando 8 semanas de datos realistas de un negocio de servicios, el sistema identificó una correlación no obvia que un dashboard estándar pasaría por alto: caídas en satisfacción del cliente y picos en tickets de soporte (semanas 5 y 8) precedieron consistentemente caídas de ventas la semana siguiente. Este tipo de insight, que cruza variables y razona sobre tendencia, requiere analizar múltiples métricas simultáneamente — justo lo que una tabla o gráfico estático no puede mostrar por sí solo.

## Cómo Se Vende

**Pitch:** *"Esto no es un dashboard — es un analista de negocio que nunca duerme. Cada semana, sin mover un dedo, el dueño recibe qué está pasando en su negocio, qué es lo más urgente, y qué hacer al respecto — el mismo nivel de razonamiento que pagaría por un consultor, entregado automáticamente y de forma constante."*

**Cliente objetivo:** Negocios de servicios pequeños/medianos que ya trackean métricas (clínicas, concesionarios, inmobiliarias, agencias) pero no tienen el tiempo ni un analista interno para interpretarlas semana a semana.

**Rango de precio:** $150–$400 USD/mes como add-on recurrente a un retainer de automatización existente, o $300–$600 USD como suscripción independiente de "analista de negocio con IA", dependiendo de la complejidad de los datos y frecuencia de reporte.

## Tech Stack

- n8n (orquestación del flujo)
- Claude API (Claude Sonnet — análisis y recomendaciones)
- Google Sheets (fuente de datos + historial de análisis)
- Gmail (entrega del reporte)

## Aprendizajes Técnicos Clave

- **Body de HTTP Request con expresión JSON:** cuando el body se arma con `JSON.stringify()` dentro de una expresión de n8n, `Specify Body` debe estar en **"Using JSON"**, nunca en "Using Fields Below". Esta última opción trata toda la expresión stringificada como el *nombre* de un solo campo, rompiendo el request silenciosamente sin un error claro.
- **Las credenciales de Header Auth solo soportan un par de header.** Headers adicionales requeridos (`anthropic-version`, `content-type` para la API de Anthropic) deben agregarse manualmente en la sección "Send Headers" del nodo — no pueden vivir dentro de la credencial misma.
- **`$json` se resetea después de un nodo HTTP Request.** Los datos de nodos anteriores no son accesibles directamente después de que retorna la llamada; la respuesta debe extraerse del `$json` actual (ej. `$json.content[0].text` para el formato de respuesta de Claude), no referenciarse vía `$('Nodo Anterior')`.
- El formato de respuesta de Claude (`content[0].text`) difiere estructuralmente del de Gemini (`candidates[0].content.parts[0].text`) — vale la pena tener esto anotado como referencia al alternar entre proveedores en distintos proyectos.

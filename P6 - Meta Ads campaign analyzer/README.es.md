[🇬🇧 English](./README.md) | 🇪🇸 Español

# P6 — Analizador de Campañas Meta Ads con IA

Workflow de n8n que lee métricas de campañas de Meta Ads, calcula indicadores reales de eficiencia (CTR, CPA), y usa IA para generar un análisis comparativo con recomendaciones accionables — entregado automáticamente por correo.

## Problema

Los dueños de negocios que corren campañas de Meta Ads revisan el gasto total y el número de leads, pero rara vez calculan el CPA o CTR por campaña — y casi nunca comparan campañas entre sí semana a semana. Resultado: el presupuesto sigue fluyendo hacia campañas ineficientes simplemente porque nadie tiene el tiempo (ni el hábito analítico) para detectarlo.

## Solución

**Arquitectura:**

```
Trigger Manual (→ Schedule Trigger en producción)
        ↓
Google Sheets — Lee datos de campañas (simula la API de Meta Ads)
        ↓
Edit Fields — Calcula CTR y CPA por campaña
        ↓
Aggregate — Une todas las campañas en un solo array
        ↓
HTTP Request — Gemini analiza todas las campañas juntas
        ↓
Edit Fields — Extrae el texto limpio del análisis + fecha
        ↓
Google Sheets — Agrega al historial
        ↓
Gmail — Envía reporte HTML con formato profesional
```

Cada semana, el workflow trae los datos de campañas, calcula las métricas de eficiencia que realmente importan (no solo el gasto bruto), y manda todo a un modelo de IA en una sola llamada agrupada para que pueda razonar entre campañas — identificando cuál está ganando, cuál está quemando presupuesto, y por qué. El análisis queda registrado en una hoja de historial y se entrega como un correo HTML con diseño profesional, listo para reenviar a un cliente.

## Resultado de negocio

En el escenario de prueba (5 campañas simuladas de un concesionario de autos), el análisis identificó una **diferencia de 4.6x en costo por adquisición** entre la campaña más eficiente y la menos eficiente ($14.545 vs $67.778). Reasignar ese presupuesto hacia la campaña eficiente representa el potencial de casi duplicar el volumen de leads sin gasto adicional.

## Cómo se vende esto

- **Cliente objetivo:** negocios pequeños/medianos que corren Meta Ads sin un analista de marketing dedicado (concesionarios, clínicas, inmobiliarias, servicios locales).
- **Setup fee:** $150–300 USD (integración con la cuenta real de Meta Ads, Sheets y correo del cliente).
- **Recurrente mensual:** $80–150 USD/mes (mantenimiento + ajuste de prompt según categoría del negocio).
- **Pitch:** *"No te vendo un dashboard más. Te doy un analista de marketing que revisa tus campañas cada semana y te dice, en lenguaje simple, dónde estás quemando plata."*
- Upsell natural hacia un sistema completo de lead gen (captación multicanal + calificación con IA + reporting).

## Tech Stack

n8n (self-hosted) · Gemini API (free tier, solo pruebas) · Google Sheets · Gmail

## Aprendizajes técnicos clave

- **Aggregate antes de comparar:** para que la IA compare ítems entre sí (no que los analice uno por uno), hay que unirlos primero en un solo array. Saltarse este paso hace que el modelo pierda todo el contexto entre campañas.
- **JSON.stringify anidado en expresiones:** mezclar comillas de JSON literal con expresiones `{{ }}` rompe el parser. La solución robusta es construir todo el body como objeto JS y serializarlo una sola vez al final (`JSON.stringify({...})`), en vez de armar el JSON a mano como texto.
- **Los rate limits del free tier son reales y bajos:** 20 requests/día en Gemini Flash. Cada error de sintaxis cuenta igual como request gastado — la precisión antes de ejecutar importa más que iterar a prueba y error.
- **La nomenclatura de modelos cambia rápido:** siempre verificar la versión de modelo vigente antes de fijarla en el código; los alias de modelos del free tier se deprecan o renombran sin mucho aviso.
- **Credenciales fuera del JSON:** usar credenciales de Header Auth en vez de incrustar la API key en la URL. Cualquier JSON que compartas (portafolio, GitHub, cliente) debe poder circular sin regalar acceso a la cuenta.
- **La métrica derivada es el producto, no el dato crudo.** Cualquiera puede exportar spend y conversiones de Meta Ads. El valor que se vende acá es el cálculo de CTR/CPA + comparación razonada + recomendación accionable — la parte que un dueño de negocio no tiene tiempo ni skill analítico de hacer solo.

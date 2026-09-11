# P3 — Dashboard básico de métricas (Google Sheets)

🇬🇧 [Read in English](README.md)

## Problema

Negocios de servicios pequeños y medianos (agencias, concesionarios, consultoras) suelen llevar sus métricas de ventas en Excel manual o no las llevan. Los dueños no tienen visibilidad diaria de leads, ventas cerradas, ingresos o tasa de conversión sin pedirle a alguien que arme un reporte — normalmente 15-20 minutos de trabajo manual, repetidos una y otra vez.

## Solución

Un workflow en n8n que captura datos de actividad diaria y los consolida automáticamente en un Google Sheet vivo — una vista "Resumen" que siempre refleja los totales actuales, sin cálculo manual.

**Arquitectura:**

```
[Trigger Manual]
      ↓
[Generar Datos de Ejemplo]  → simula leads, ventas, ingreso (nodo Code)
      ↓
[Agregar Fila]  → registra el dato en "Metricas_Diarias" (log histórico crudo)
      ↓
[Leer Filas]  → lee todo el log histórico
      ↓
[Calcular Totales]  → suma/promedia leads, ventas, ingreso, tasa de conversión (nodo Code)
      ↓
[Actualizar Fila]  → sobreescribe la fila 2 de "Resumen" (snapshot siempre actualizado)
```

Dos pestañas en Google Sheets:
- **Metricas_Diarias** — log histórico de solo-agregar, una fila por ejecución
- **Resumen** — una sola fila viva, siempre sobreescrita, funciona como "el dashboard"

## Resultado de negocio

Elimina por completo el proceso manual de consolidar métricas diarias. El dueño abre una sola hoja y ve leads totales, ventas cerradas, ingreso acumulado y % de conversión — siempre actualizado, sin tiempo administrativo invertido en armarlo.

## Cómo se vende

- **Cliente objetivo:** negocios de servicios pequeños sin CRM ni hábito de reporting (agencias, concesionarios, clínicas, consultoras)
- **Pitch:** "Automated Sales Dashboard Setup" — automatización liviana y rápida de desplegar que reemplaza el mantenimiento manual de hojas de cálculo
- **Rango de precio:** $150-300 USD, entrega en 2-3 días
- **Upsell natural:** conecta directo con calificación de leads con IA y análisis de campañas — mismo cliente, mayor alcance

## Tech Stack

n8n (self-hosted, Docker) · Google Sheets API · JavaScript (nodos Code)

## Aprendizajes técnicos clave

- **Bug de mapeo Fixed vs. Expression:** al mapear manualmente los campos de "Values to Update" en el nodo Update de Google Sheets, cada campo arranca en modo valor literal/fijo. Copiar o escribir un valor directo (ej. "2") lo deja como constante hardcodeada en vez de referencia dinámica — cada campo debe pasarse explícitamente a modo Expression (`{{ $json.campo }}`) para traer el dato real del nodo anterior.
- **`row_number` como columna de match:** usar `row_number: 2` fijo (no expresión) es intencional acá — es lo que convierte la hoja en un "dashboard vivo" (siempre sobreescrito) en vez de un log histórico. Es una decisión de arquitectura deliberada, no un bug.
- **Patrón de dos hojas:** separar el log crudo (solo-agregar) del resumen (siempre sobreescrito) es un patrón reutilizable para cualquier entregable tipo dashboard — mantiene el histórico intacto mientras le da al cliente una vista de un solo vistazo.

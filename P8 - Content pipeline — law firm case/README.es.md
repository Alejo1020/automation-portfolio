# P8 — Pipeline de contenido semanal para firma legal (cliente ficticio)

[🇬🇧 English](README.md) | 🇪🇸 Español

## Problema
Las firmas de servicios profesionales (legal, contable, consultoría) necesitan presencia constante en redes y blog para captar leads orgánicos, pero generar contenido de calidad semanalmente consume horas de un socio o requiere contratar a alguien. Resultado: la firma no publica, o publica contenido genérico sin estrategia.

## Solución
Pipeline en n8n que automatiza la generación semanal de contenido desde un calendario editorial:
- Schedule Trigger dispara cada lunes 8am
- Lee temas pendientes de un Google Sheet (columna Estado vacía)
- Por cada tema, un solo llamado a Gemini genera 3 piezas: post de Instagram, post de LinkedIn y un artículo SEO de 800-1000 palabras
- Limpia y parsea la respuesta de la IA
- Actualiza el Sheet con el contenido generado, marcando Estado = "Generado"

**Arquitectura:**
```
Schedule Trigger → Get rows (temas pendientes) → Preparar Tema (Code)
→ Gemini API (HTTP Request) → Limpiar respuesta (Set) → Parsear + armar fila (Code)
→ Update Sheet
```

## Resultado de negocio
- 12 temas cargados → 12 semanas de contenido (36 piezas) generadas con 12 llamados a la API
- Prueba real: 2/12 filas procesadas sin errores, contenido coherente y listo para publicar
- Tiempo ahorrado estimado: generar este contenido manualmente tomaría 1-2 horas por tema; el pipeline lo hace en segundos

## Cómo se vende
Cliente objetivo: firmas de servicios profesionales pequeñas/medianas (legal, inmobiliaria, contable, clínicas) sin equipo de contenido interno.

Pitch: "Cargás tus temas una vez al mes — el sistema entrega 3 piezas de contenido listas para revisar, cada semana, sin que nadie del equipo escriba desde cero."

Rango de precio: $150-300 USD/mes como retainer de mantenimiento, o $400-600 USD como cobro único por implementación en la instancia del cliente.

## Stack Técnico
n8n · Google Gemini API (Flash) · Google Sheets · JavaScript (nodos Code)

## Aprendizajes técnicos clave
- Pedir las 3 piezas en un solo prompt (una respuesta JSON) en vez de 3 llamados separados ahorra cuota de API y mantiene coherencia de mensaje entre canales.
- El filtro de Google Sheets en n8n no tiene operador nativo "is empty" — dejar el campo Value en blanco logra el mismo resultado.
- Con varios nodos Code en un mismo flujo, renombrarlos explícitamente (ej. "Preparar Tema") es necesario para referenciar la salida de uno sin ambigüedad vía `$('NombreNodo')`.
- Buscar filas por una llave de negocio (Fecha) en vez de row_number es más resistente a inserciones/eliminaciones — mismo patrón usado en P3/P5.

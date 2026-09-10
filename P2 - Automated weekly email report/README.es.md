[🇬🇧 English](./README.md) | 🇪🇸 Español

# P2 — Reporte Semanal Automático de Leads por Email

## Problema

Los dueños de negocio que capturan leads (formularios, WhatsApp, ads) rara vez los revisan de forma consistente. La revisión manual en spreadsheets es inconsistente, consume tiempo, y los leads se pierden sin que nadie se dé cuenta.

## Solución

Un reporte semanal totalmente automático que llega al correo del dueño cada lunes en la mañana — sin trabajo manual.

**Flujo:** `Schedule Trigger (semanal, lun 8am) → Google Sheets (lee leads) → Code (formatea tabla HTML) → Gmail (envía reporte)`

1. **Schedule Trigger** se dispara cada lunes a las 8:00 AM.
2. Nodo **Google Sheets** lee todas las filas de leads de la hoja de seguimiento.
3. **Nodo Code** (JavaScript) transforma las filas crudas en una tabla HTML limpia con resumen de cantidad de leads.
4. Nodo **Gmail** envía el reporte formateado al destinatario configurado.

## Resultado de Negocio

Tiempo ahorrado estimado: ~45 min/semana de revisión manual (~3 horas/mes) por negocio, además de menos leads perdidos por seguimiento inconsistente. La generación y envío del reporte queda completamente automatizada una vez configurado.

## Cómo Se Vende

- **Cliente target:** negocios de servicios pequeños (clínicas, agencias, inmobiliarias, comercio local) que ya tienen un canal de captación de leads pero sin disciplina de reporting.
- **Rango de precio:** $50–100 USD de setup fee como entregable independiente.
- **Upsell:** se combina naturalmente con P1 (notificador de leads por WhatsApp) en un paquete "Sistema de Seguimiento de Leads" — mayor ticket, mismo stack base.

## Tech Stack

n8n (self-hosted, Docker/Ubuntu) · Google Sheets API · Gmail API · JavaScript (nodo Code)

## Aprendizajes Técnicos Clave

- Los nombres de las pestañas de Google Sheets se pueden renombrar después de configurar el workflow — la referencia cacheada `sheetName` (por `gid`) del nodo sigue resolviendo correctamente, pero siempre hay que reverificar que el nombre de pestaña mostrado coincide con lo esperado antes de correr en producción.
- El cuerpo del email HTML debe armarse completo en el nodo Code y pasarse como un solo string (`$json.htmlReport`) al campo Message del nodo Gmail — se necesitan estilos inline porque Gmail elimina el CSS externo.
- La configuración de credencial OAuth2 (Google Cloud Console → habilitar API → crear OAuth client → autorizar en n8n) es el mismo flujo que se reutiliza en todos los nodos de Google (Sheets, Gmail, Drive) — configurarla una vez desbloquea el patrón para futuros proyectos.

## Próximas Iteraciones (planeadas)

- Filtrar solo leads creados en los últimos 7 días, en vez de la hoja completa en cada corrida.
- Mejorar el estilo de la tabla HTML para un look más pulido y con marca.

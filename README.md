# Métricas PIA

Panel de métricas de uso de **Pía** (People IA), el asistente de RR.HH. de ADIPA en Slack.

- **Producción:** https://metricas-pia.vercel.app
- **Dominio objetivo:** `metricas-pia.adipa.ai`
- **Proyecto en Vercel:** `metricaspia`

## ¿Qué hace?

Muestra cuánto se usa Pía (conteo de mensajes) sin guardar el texto de las consultas. El panel lee los datos desde un Google Sheet a través de un webhook de n8n y los presenta con la marca ADIPA.

> Privacidad: Pía **no guarda el contenido** de las consultas. Solo se registran métricas de uso.

## Arquitectura

```
Slack (Pía)
   │  cada mensaje
   ▼
n8n cloud ──► Google Sheet "Metricas_PIA"
                 ├─ Log      (un registro por mensaje)
                 └─ Resumen  (agregados)
                      ▲
                      │ webhook (protegido con token)
                      │
        Función serverless (Vercel) ◄── guarda el token, el navegador nunca lo ve
                      ▲
                      │
        Panel web (Vercel) ◄── login con Google (SSO)
```

## Seguridad (3 capas)

1. **Token en el webhook de n8n:** el webhook rechaza llamadas sin token válido.
2. **Función serverless:** el token vive en variables de entorno de Vercel; el frontend solo llama a la función.
3. **Login con Google (SSO):** solo accede personal autorizado. Se eligió SSO (pedido de Nico) en lugar de una clave compartida.

## Variables de entorno

Configurar en Vercel (Settings → Environment Variables). **No subir valores al repositorio.**

| Variable | Descripción |
|---|---|
| `N8N_WEBHOOK_URL` | URL del webhook de n8n que entrega las métricas |
| `N8N_WEBHOOK_TOKEN` | Token que valida el webhook |
| `GOOGLE_CLIENT_ID` | Client ID de OAuth |
| `ALLOWED_DOMAIN` / lista de correos | Quién puede entrar (ajustar según implementación) |

> Ajustar los nombres a los que realmente use el código.

## Fuente de datos: Google Sheet `Metricas_PIA`

- **Log:** una fila por mensaje recibido por Pía (alimentada por nodos de n8n).
- **Resumen:** consolidado que consume el panel.

Si se cambian columnas o pestañas, hay que actualizar el flujo de n8n y el panel.

## Despliegue

1. Push a la rama principal → Vercel despliega automáticamente.
2. Verificar variables de entorno en Vercel.
3. Probar login con Google y carga de datos.

### Dominio propio (pendiente)

1. Ale crea el registro DNS para `metricas-pia.adipa.ai`.
2. Agregar el dominio en Vercel (Settings → Domains).
3. Agregar el nuevo origen en **Authorized JavaScript origins** y **redirect URIs** del Client ID de Google (lo hace Ale, porque administra Google Cloud).

## Desarrollo local

```bash
npm install
cp .env.example .env.local   # completar variables
npm run dev
```

> Ajustar comandos según el stack real del proyecto.

## Responsables

| Rol | Persona |
|---|---|
| Desarrollo / líder técnico | Paulo Vicencio |
| Contraparte de negocio (Talento y Bienestar) | Thiare Lizana |
| Google Cloud / Workspace / DNS | Alejandro Luzanto |

## Pendientes

- [ ] Conectar `metricas-pia.adipa.ai`

## Problemas comunes

- **El panel no carga datos:** revisar que el flujo de n8n esté activo y que el token de Vercel coincida con el del webhook.
- **Login de Google falla:** el dominio o URL no está en los orígenes autorizados del Client ID.
- **Faltan mensajes en el Log:** revisar ejecuciones fallidas en n8n.

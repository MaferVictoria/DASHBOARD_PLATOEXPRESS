# Dashboard Plato Express — Paid Media & Pipeline Comercial

## Qué hay en este repo
```
index.html              ← el dashboard completo (una sola página, sin build step)
api/google-ads-data.js  ← función serverless de Vercel para Google Ads (lista, falta configurar)
```

## Estado actual
- **Pipeline comercial:** en vivo, leyendo directo de tu Google Sheet "Plato - LEADS" (pestaña "Hoja 1").
- **Paid media (Google Ads):** en modo respaldo, usando la pestaña "Resumen mensual" del Sheet "Reporte Comercial - Plato Express" (datos mensuales). En cuanto conectes la API en vivo (ver abajo), el dashboard cambia solo, sin tocar código.

## 1. Desplegar en Vercel
1. Sube este contenido a un repo de GitHub.
2. En Vercel: **Add New → Project → Import** ese repo.
3. Framework preset: "Other" (es HTML estático, no necesita build).
4. Deploy. Con eso el dashboard ya funciona con el pipeline en vivo y Google Ads en modo respaldo.

## 2. Conectar Google Ads en vivo (cuando tengas las credenciales)
En Vercel → tu proyecto → **Settings → Environment Variables**, agrega:

| Variable | De dónde sale |
|---|---|
| `GOOGLE_ADS_DEVELOPER_TOKEN` | Google Ads → Herramientas → Centro de API |
| `GOOGLE_ADS_CLIENT_ID` | Google Cloud Console → credenciales OAuth (tipo **"Aplicación web"**, no "de escritorio") |
| `GOOGLE_ADS_CLIENT_SECRET` | mismo lugar que el Client ID |
| `GOOGLE_ADS_REFRESH_TOKEN` | se genera una vez vía OAuth Playground con el Client ID/Secret de arriba |
| `GOOGLE_ADS_CUSTOMER_ID` | ID de la cuenta de Google Ads a conectar (sin guiones) |
| `GOOGLE_ADS_LOGIN_CUSTOMER_ID` | sólo si la cuenta está bajo un MCC |

Si quieres, te guío paso a paso para sacar cada una de estas cuando llegue el momento — es la parte donde más se traba la gente la primera vez.

Una vez puestas las variables y redeployado, el dashboard detecta automáticamente que `/api/google-ads-data` responde y cambia solo a "🟢 Google Ads en vivo" (y de paso, los filtros de Hoy/Ayer/7 días se vuelven exactos, porque ahí sí hay dato diario).

## 3. Pendiente de tu lado: columna "Correo"
Para que el cálculo de **MQL** funcione, agrega una columna llamada exactamente **"Correo"** en el Sheet "Plato - LEADS" (Hoja 1), con el correo de cada lead. Mientras un lead no tenga correo capturado, cuenta como Lead pero no como MQL (el dashboard lo marca como "pendiente", no como error).

Dominios que **no** cuentan como correo de empresa (ajustable en el código, constante `PERSONAL_EMAIL_DOMAINS`): gmail, hotmail, outlook (incl. .com.mx), live, yahoo, icloud, msn, aol, protonmail, gmx, y el dominio interno de la agencia (`rockinmedia.com` / `rockinmedia.mx` — si el dominio real es otro, dímelo y lo corrijo). Si falta alguno más o hay que quitar uno, dímelo y lo ajusto.

## 5. Embudo actualizado

- **Lead:** toda fila del Sheet de leads.
- **MQL:** correo de dominio empresarial (no personal, no `rockinmedia`).
- **SQL** (fórmula final, ambas rutas exigen correo empresarial):
  1. ≥400 empleados **y** ≥1 turno, **o**
  2. 200-399 empleados **y** exactamente 1 turno.
- **Posible SQL** (entre MQL y SQL): cualquiera de estas dos rutas —
  1. 200-399 empleados **y** 2-5 turnos (sin importar el correo), **o**
  2. ≥200 empleados **y** sin correo empresarial.

Estas fórmulas se revisaron a propósito para que **nunca se traslapen** SQL y Posible SQL (antes sí podían coincidir en un mismo lead) — probado con los casos límite exactos (400 vs. 399 empleados, 1 vs. 2 turnos, con y sin correo).

## 6. Pestaña "Por plaza"

- Inversión: Sheet "Reporte Comercial", pestaña "Resumen mensual por plaza" — columna A (Mes), E (Inversión Nuevo León), H (Inversión CDMX y Estado de México).
- Leads: Sheet "Plato - LEADS", columna "Ciudad", agrupada en las mismas 2 plazas que separa la inversión (Nuevo León vs. CDMX+Estado de México juntos, ya que la inversión no se reporta por separado para esas dos ciudades).
- Muestra Gasto, Leads, CPL, CPA MQL y CPA SQL por plaza, más una gráfica de gastado vs. generado y una tabla mensual desglosada.

## 7. Otros detalles ya incorporados
- El agrupamiento mensual de leads usa la columna **"Mes"** del Sheet (no la fecha), tal cual lo pediste.
- El dashboard ahora tiene botones de "Mes específico" en el filtro de periodo (uno por cada mes con datos), además de los presets normales.
- La pestaña "Resumen mensual" del Sheet de Google Ads ya sólo trae 2026 — se quitó 2025 a propósito para que las gráficas se lean mejor.

## 4. Fuentes de datos (IDs, por si algún día cambian de nombre)
- Pipeline: Sheet `1QEZ_w30onOHOLN_VJ4pUt5e90LC9z4ZXILa5Qz7vrA8`, pestaña `Hoja 1`.
- Google Ads (respaldo): Sheet `1ToVcoGqYLd3PR6HghpyjQrVPCWEttkBbSxG2wKigwaI`, pestaña `Resumen mensual`.

Ambos Sheets deben mantenerse compartidos como "Cualquier persona con el enlace puede ver".

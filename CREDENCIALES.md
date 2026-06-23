# Credenciales del Proyecto Chile-Robot

> **IMPORTANTE:** Este archivo contiene claves reales. NO subir a repositorios publicos.
> Las claves de Firebase son de uso publico (client-side), pero la de Google Calendar esta restringida.

---

## Firebase Realtime Database

**Proyecto:** juanech-sa
**Plan:** Spark (gratuito)
**Region de la base de datos:** us-central1
**URL de la base de datos:** https://juanech-sa-default-rtdb.firebaseio.com
**Reglas:** Modo de prueba (lectura/escritura abierta hasta 2026-07-23)

### firebaseConfig completo (copiar y pegar tal cual)

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyAZ_6YiC9FJTwwkPPMhT7SpdOgyRW5nbsg",
  authDomain: "juanech-sa.firebaseapp.com",
  databaseURL: "https://juanech-sa-default-rtdb.firebaseio.com",
  projectId: "juanech-sa",
  storageBucket: "juanech-sa.firebasestorage.app",
  messagingSenderId: "310883397247",
  appId: "1:310883397247:web:be8090e93bdd43a429194a"
};
```

### CDN Scripts de Firebase (agregar en este orden exacto)

```html
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-database-compat.js"></script>
```

### Inicializacion de Firebase (despues de los scripts CDN)

```javascript
firebase.initializeApp(firebaseConfig);
const db = firebase.database();
```

---

## Google Calendar API

**API Key:** `AIzaSyBunB8Ugcu1GGflPEOJg9NkMlKP6w1qKs4`
**Restriccion de API:** Solo Google Calendar API
**Restriccion de aplicacion:** Ninguna (se puede agregar despues)
**Calendar ID:** `juaneo2006@gmail.com`
**Zona horaria:** `America/Santiago`

### Ejemplo de uso (endpoint completo)

```javascript
const CALENDAR_API_KEY = 'AIzaSyBunB8Ugcu1GGflPEOJg9NkMlKP6w1qKs4';
const CALENDAR_ID = 'juaneo2006@gmail.com';

// Obtener eventos de las proximas 2 semanas
const now = new Date();
const twoWeeksLater = new Date(now.getTime() + 14 * 24 * 60 * 60 * 1000);
const timeMin = now.toISOString();
const timeMax = twoWeeksLater.toISOString();

const url = `https://www.googleapis.com/calendar/v3/calendars/${encodeURIComponent(CALENDAR_ID)}/events?key=${CALENDAR_API_KEY}&timeMin=${timeMin}&timeMax=${timeMax}&singleEvents=true&orderBy=startTime&timeZone=America/Santiago`;

fetch(url)
  .then(r => r.json())
  .then(data => {
    // data.items es un array de eventos
    // Cada evento tiene: summary, start.dateTime (o start.date), end.dateTime
  });
```

### REQUISITO: El calendario debe ser publico

Para que la API Key funcione sin autenticacion OAuth, el calendario de Google debe estar configurado como publico:

1. Ir a https://calendar.google.com/calendar/r/settings
2. En la columna izquierda, hacer clic en el calendario "juaneo2006@gmail.com"
3. Buscar "Permisos de acceso para los eventos"
4. Activar "Compartir publicamente"
5. Guardar

**NOTA:** Si el calendario NO esta publico, la API devolvera error 404 "Not Found". Esto NO es un error de codigo, es un error de configuracion del calendario.

---

## APIs publicas sin autenticacion

### Dolar (USD/CLP) - mindicador.cl
```
GET https://mindicador.cl/api/dolar
```
No requiere key. Devuelve JSON con `serie[0].valor` (valor del dia).

### SpaceX (SPCX) - Yahoo Finance via proxy CORS
```
GET https://corsproxy.io/?url=https://query1.finance.yahoo.com/v8/finance/chart/SPCX?interval=1d&range=1d
```
Si `corsproxy.io` falla, alternativas:
- `https://api.allorigins.win/raw?url=...`
- `https://corsproxy.org/?url=...`

El JSON devuelto tiene la estructura:
```
chart.result[0].meta.regularMarketPrice  -> precio actual
chart.result[0].meta.previousClose       -> cierre anterior
```

# Prompt de trabajo — Chile-Robot: Expansion de funcionalidades

## ANTES DE EMPEZAR: Lee estos archivos obligatoriamente

1. Lee `PROJECT_CONTEXT.md` — contiene la descripcion completa del proyecto, stack tecnico, decisiones de diseno, archivos existentes y APIs.
2. Lee `CREDENCIALES.md` — contiene TODAS las claves, configuraciones y ejemplos de codigo listos para copiar y pegar. Las credenciales ya estan creadas y funcionando. NO necesitas crear ninguna cuenta, proyecto ni clave. Solo usa las que estan ahi.
3. Lee `index.html` — el codigo fuente actual completo. Necesitas entenderlo antes de modificarlo.

---

## Reglas de trabajo ESTRICTAS

1. **Trabaja de forma totalmente autonoma.** Implementa todo sin pedir autorizaciones ni confirmaciones. Si tienes una duda genuina de contexto o ambiguedad tecnica, pregunta de forma concisa y sigue adelante con lo demas.

2. **Cero errores, cero alucinaciones.** Todo el codigo debe funcionar al primer intento. NO inventes APIs que no existan. NO asumas endpoints sin verificar. NO alucines librerias, funciones, ni parametros. Si algo no es viable, dilo y propone alternativa real.

3. **Stack: HTML + CSS + JS vanilla UNICAMENTE.** No uses React, Vue, Angular, Tailwind, Bootstrap, jQuery ni ningun framework. La UNICA libreria externa permitida es Firebase SDK via CDN (los scripts estan en CREDENCIALES.md). Todo lo demas es codigo vanilla puro.

4. **Mantener la estetica EXACTA del proyecto.** Fondo negro puro (#000000), texto blanco (#fff), acento rojo (#ff3b30), inputs fondo #222, bordes #444. Revisa la seccion "Decisiones de diseno" en PROJECT_CONTEXT.md y seguila al pie de la letra. Las nuevas paginas/vistas deben verse IDENTICAS en estilo al index.html existente.

5. **Todo debe funcionar en GitHub Pages.** Solo archivos estaticos. Rutas relativas (no absolutas). Sin servidor backend. Sin npm, sin build step.

6. **No dejes codigo muerto, console.logs de debug, ni TODOs sin resolver.**

---

## CREDENCIALES (ya configuradas, NO crear nuevas)

Todas las credenciales estan en el archivo `CREDENCIALES.md`. Ahi encontraras:
- El `firebaseConfig` completo con todos los valores reales
- Los scripts CDN de Firebase con la version exacta a usar
- El codigo de inicializacion de Firebase
- La API Key de Google Calendar
- El Calendar ID
- Ejemplos de codigo funcional para cada API
- Las URLs exactas de las APIs de finanzas

**IMPORTANTE:** NO uses placeholders como `YOUR_API_KEY`. Usa los valores reales que estan en CREDENCIALES.md.

---

## TAREA 1: Control Remoto Movil (`remote.html`)

Crea el archivo `remote.html` en la raiz del proyecto (accesible en `https://juanech5.github.io/Chile-robot/remote.html`).

### Proposito
Esta pagina se usa EXCLUSIVAMENTE desde un celular para controlar remotamente la pagina principal (`index.html`) que esta abierta en un computador. La comunicacion entre ambas paginas es via Firebase Realtime Database.

### Diseno visual obligatorio
- **Mobile-first.** Disenar para pantallas de 375-430px de ancho.
- Debe verse como una app nativa, no como una "pagina web".
- Fondo negro (#000), texto blanco, acento rojo (#ff3b30).
- Fuente: `system-ui, -apple-system, BlinkMacSystemFont, sans-serif`
- Border-radius: 12px para cards, 6px para inputs/botones
- Inputs: fondo #222, borde #444, color blanco, placeholder #555
- Botones principales: fondo blanco, texto negro, font-weight 600
- Boton de peligro (apagar alarma): fondo #ff3b30, texto blanco
- Transiciones: 0.2s ease en todo

### Funcionalidad 1.1: Status de conexion (parte superior)
- Muestra si `index.html` esta abierta en algun computador
- Lee de Firebase la ruta `/chile-robot/status/online`
- Si `online === true`: circulo verde (8px) + texto "Conectado" + nombre del dispositivo (leer de `/chile-robot/status/device`)
- Si `online === false` o no existe: circulo rojo + texto "Sin conexion"
- Usa `firebase.database().ref('chile-robot/status').on('value', callback)` para escuchar cambios en tiempo real

### Funcionalidad 1.2: Configuracion de alarma
Replica el menu de configuracion de alarma que existe en `index.html`, adaptado para movil:

**Campos a incluir (en este orden exacto):**

1. **Hora de alarma** — `<input type="time">` con fondo #222, borde #444
2. **Toggle "Modo Despertador"** — Un switch/toggle visual. Cuando esta ON, se expande y muestra los campos 3-7. Cuando esta OFF, solo se muestra la hora.
3. **Selector de tono** — Un `<select>` dropdown (NO file upload) con estas 3 opciones exactas:
   - Opcion 1: texto "Lo-fi Alarm", value `lesiakower-lo-fi-alarm-clock-243766.mp3`
   - Opcion 2: texto "Alarm Clock Clasico", value `freesound_community-alarm-clock-90867.mp3`
   - Opcion 3: texto "Tono Incendio", value `Tono Incendio.mpeg`
   - Agrega un boton de play/preview al lado del select para escuchar el tono. Usa un elemento `<audio>` oculto y cambia su `src` al value seleccionado.
4. **Mensaje TTS** — `<input type="text" placeholder="Que quieres que diga?">` — Este texto se lee por voz cuando suena la alarma
5. **Pendientes/Recordatorios** — `<textarea placeholder="Que quieres que salga en pantalla?">` — Este texto aparece en pantalla pero NO se lee
6. **Link de Spotify** — `<input type="url" placeholder="Link de Spotify">` — Se abre en nueva pestana
7. **Comando Alexa** — `<input type="text" placeholder="Comando para Alexa">` — Se dice por TTS "Alexa, [comando]"

**Boton "Guardar alarma":** Al hacer clic, escribe todos estos valores en Firebase:

```javascript
firebase.database().ref('chile-robot/alarm').set({
  time: document.getElementById('alarm-time').value,        // "07:30"
  despertadorMode: toggleEstaActivo,                         // true/false
  tone: document.getElementById('tone-select').value,        // nombre del archivo
  ttsMessage: document.getElementById('tts-msg').value,      // texto
  reminders: document.getElementById('reminders').value,     // texto
  spotifyLink: document.getElementById('spotify-link').value,// URL
  alexaCommand: document.getElementById('alexa-cmd').value,  // texto
  stopAlarm: false                                           // siempre false al guardar
});
```

**Boton "Apagar alarma":** Boton grande, rojo (#ff3b30), siempre visible. Al hacer clic:
```javascript
firebase.database().ref('chile-robot/alarm/stopAlarm').set(true);
```

### Funcionalidad 1.3: Cambiar vista de la pantalla principal
En la parte inferior de la pagina, 3 botones/tabs para cambiar que se muestra en el computador:
- **Reloj** (icono de reloj o texto "Reloj") — vista por defecto
- **Finanzas** (icono de grafico o texto "$")
- **Calendario** (icono de calendario o texto "Cal")

Al seleccionar una vista:
```javascript
firebase.database().ref('chile-robot/view/current').set('clock'); // o 'finance' o 'calendar'
```

El tab activo debe tener estilo diferente (por ejemplo, texto blanco en el activo, texto gris en los inactivos, con una linea inferior roja en el activo).

### Estructura HTML basica de remote.html

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Chile-Robot Remote</title>
  <!-- Estilos inline en <style> -->
</head>
<body>
  <!-- 1. Status de conexion -->
  <!-- 2. Formulario de alarma -->
  <!-- 3. Boton apagar alarma -->
  <!-- 4. Tabs de vista (fijo abajo) -->

  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-database-compat.js"></script>
  <script>
    // Pegar firebaseConfig de CREDENCIALES.md
    // Inicializar Firebase
    // Logica del control remoto
  </script>
</body>
</html>
```

---

## TAREA 2: Vista de Finanzas (dentro de `index.html`)

Crear una vista/seccion dentro de `index.html` que muestre datos financieros. Esta vista NO es una pagina separada. Es un `<div>` que se muestra/oculta segun la vista activa.

### Como implementar el sistema de vistas en index.html

El `index.html` actualmente muestra el reloj siempre. Necesitas:

1. Envolver el reloj existente (el `<div class="clock-face">` y todo lo que lo rodea) en un contenedor:
```html
<div id="view-clock" class="view active">
  <!-- todo el reloj existente va aqui -->
</div>
```

2. Crear dos contenedores mas:
```html
<div id="view-finance" class="view" style="display:none;">
  <!-- vista de finanzas -->
</div>

<div id="view-calendar" class="view" style="display:none;">
  <!-- vista de calendario -->
</div>
```

3. Funcion para cambiar vista:
```javascript
function switchView(viewName) {
  document.querySelectorAll('.view').forEach(v => {
    v.style.display = 'none';
    v.classList.remove('active');
  });
  const target = document.getElementById('view-' + viewName);
  if (target) {
    target.style.display = 'flex'; // o 'block' segun el layout
    target.classList.add('active');
  }
}
```

4. Escuchar cambios de vista desde Firebase:
```javascript
firebase.database().ref('chile-robot/view/current').on('value', (snapshot) => {
  const view = snapshot.val() || 'clock';
  switchView(view);
});
```

### Contenido de la vista de finanzas

Tres "cards" con datos financieros, una debajo de otra, centradas en la pantalla:

#### Card 1: Dolar (USD/CLP)
- Titulo: "USD / CLP" en texto gris pequeno (#888)
- Valor: `$XXX,XX` en texto grande (3-4rem), blanco
- Subtitulo: "Fuente: mindicador.cl" + fecha de actualizacion en texto gris pequeno
- API: `fetch('https://mindicador.cl/api/dolar').then(r => r.json())`
- El valor esta en `data.serie[0].valor`
- Formatear con separador de miles chileno: `valor.toLocaleString('es-CL')`

#### Card 2: SpaceX (SPCX)
- Titulo: "SPCX" en texto gris + "NASDAQ" como badge
- Precio: en USD, texto grande
- Cambio porcentual: calcular `((precio_actual - cierre_anterior) / cierre_anterior * 100)`
  - Si es positivo: texto verde (#4cd964) con flecha "↑"
  - Si es negativo: texto rojo (#ff3b30) con flecha "↓"
- API via proxy CORS (ver CREDENCIALES.md para la URL exacta y estructura del JSON)
- **IMPORTANTE:** Yahoo Finance puede bloquear peticiones. Si falla, mostrar "Datos no disponibles" con un boton de reintentar. NO dejar la card vacia ni con error de JS en consola.

#### Card 3: Fondo BICE
- Titulo: "BICE Acciones Mercados Emergentes"
- Como BICE no tiene API publica, implementar asi:
  - Mostrar el nombre del fondo
  - Un boton "Ver en BICE" que abra `https://mercadosenlinea.biceinversiones.cl/www/fondosmutuos.html` en nueva pestana
  - Intentar scraping via proxy CORS: `fetch('https://corsproxy.io/?url=https://mercadosenlinea.biceinversiones.cl/www/fondosmutuos.html')`. Si funciona, parsear el HTML para extraer el valor. Si no funciona (lo mas probable), mostrar solo el boton.
  - NO usar iframe (no funcionara por X-Frame-Options).

#### Comportamiento general de la vista de finanzas
- Al entrar a la vista, cargar todos los datos inmediatamente
- Actualizar automaticamente cada 5 minutos: `setInterval(fetchFinanceData, 5 * 60 * 1000)`
- Boton de refresh manual (icono de recarga ↻) en la esquina superior derecha
- Cada card tiene borde sutil (#222), border-radius 12px, padding 20px
- Timestamp de ultima actualizacion en texto gris pequeno al final de cada card

### Estilos de las cards

```css
.finance-card {
  background: #000;
  border: 1px solid #222;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 16px;
  max-width: 500px;
  width: 100%;
}
.finance-card .label {
  color: #888;
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
.finance-card .value {
  color: #fff;
  font-size: 3rem;
  font-weight: 200;
  margin: 8px 0;
}
.finance-card .change.positive { color: #4cd964; }
.finance-card .change.negative { color: #ff3b30; }
.finance-card .timestamp {
  color: #555;
  font-size: 0.75rem;
}
```

---

## TAREA 3: Vista de Calendario (dentro de `index.html`)

Crear otra vista/seccion dentro de `index.html` que muestre un calendario de 2 semanas con eventos de Google Calendar.

### Fuente de datos
- Usar Google Calendar API v3 con la API Key de CREDENCIALES.md
- Calendar ID: `juaneo2006@gmail.com`
- El endpoint y ejemplo completo estan en CREDENCIALES.md

### Diseno del calendario
- **NO usar iframe de Google Calendar.** Construir una vista propia.
- Vista de 2 semanas: semana actual (lunes a domingo) + semana siguiente (lunes a domingo)
- 7 columnas (Lu, Ma, Mi, Ju, Vi, Sa, Do)
- 2 filas (una por semana)

#### Cada celda de dia muestra:
- Numero del dia (ej: "23") — grande si es hoy
- Lista de eventos con hora y titulo
- Si no hay eventos: vacio o texto sutil "—"

#### Dia actual:
- Resaltado con borde rojo (#ff3b30) o fondo rojo muy sutil (rgba(255,59,48,0.1))
- El numero del dia en rojo

#### Eventos:
- Mostrar como pastillas/chips con hora + titulo
- Fuente pequena (0.7-0.8rem)
- Fondo #222, border-radius 4px
- Si un evento es de todo el dia (tiene `start.date` en vez de `start.dateTime`), mostrar solo el titulo sin hora

### Logica de obtencion de eventos

```javascript
const CALENDAR_API_KEY = 'AIzaSyBunB8Ugcu1GGflPEOJg9NkMlKP6w1qKs4';
const CALENDAR_ID = encodeURIComponent('juaneo2006@gmail.com');

function fetchCalendarEvents() {
  // Calcular lunes de esta semana
  const now = new Date();
  const dayOfWeek = now.getDay(); // 0=domingo
  const mondayOffset = dayOfWeek === 0 ? -6 : 1 - dayOfWeek;
  const monday = new Date(now);
  monday.setDate(now.getDate() + mondayOffset);
  monday.setHours(0, 0, 0, 0);

  // Calcular domingo de la semana siguiente (14 dias desde el lunes)
  const endSunday = new Date(monday);
  endSunday.setDate(monday.getDate() + 13);
  endSunday.setHours(23, 59, 59, 999);

  const timeMin = monday.toISOString();
  const timeMax = endSunday.toISOString();

  const url = `https://www.googleapis.com/calendar/v3/calendars/${CALENDAR_ID}/events?key=${CALENDAR_API_KEY}&timeMin=${timeMin}&timeMax=${timeMax}&singleEvents=true&orderBy=startTime&timeZone=America/Santiago`;

  fetch(url)
    .then(r => {
      if (!r.ok) throw new Error('Calendar API error: ' + r.status);
      return r.json();
    })
    .then(data => {
      const events = data.items || [];
      renderCalendar(monday, events);
    })
    .catch(err => {
      console.error('Error fetching calendar:', err);
      // Mostrar mensaje de error en la vista
    });
}
```

### Como renderizar el calendario

```javascript
function renderCalendar(startMonday, events) {
  // Crear un mapa: fecha (YYYY-MM-DD) -> array de eventos
  const eventMap = {};
  events.forEach(event => {
    // Fecha del evento
    const dateStr = event.start.dateTime
      ? event.start.dateTime.split('T')[0]
      : event.start.date; // evento de todo el dia
    if (!eventMap[dateStr]) eventMap[dateStr] = [];
    eventMap[dateStr].push(event);
  });

  // Generar 14 dias a partir del lunes
  const today = new Date().toISOString().split('T')[0]; // YYYY-MM-DD

  for (let i = 0; i < 14; i++) {
    const date = new Date(startMonday);
    date.setDate(startMonday.getDate() + i);
    const dateStr = date.toISOString().split('T')[0];
    const dayNum = date.getDate();
    const isToday = dateStr === today;
    const dayEvents = eventMap[dateStr] || [];

    // Renderizar celda con dayNum, isToday, dayEvents
    // Cada evento: extraer hora de start.dateTime (ej: "14:30") y summary
  }
}
```

### Actualizacion automatica
- Cada 15 minutos: `setInterval(fetchCalendarEvents, 15 * 60 * 1000)`
- Al entrar a la vista de calendario, cargar inmediatamente

### Si la API devuelve error 404
Significa que el calendario NO esta publico. Mostrar un mensaje amigable:
```
"Calendario no accesible. Verifica que tu Google Calendar sea publico."
```
Con instrucciones breves de como hacerlo publico.

---

## TAREA 4: Modificaciones a `index.html`

El archivo `index.html` existente necesita estas modificaciones:

### 4.1 Agregar Firebase SDK
Al final del `<body>`, ANTES del `<script>` existente que tiene toda la logica, agregar:

```html
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-database-compat.js"></script>
```

### 4.2 Inicializar Firebase
Al inicio del script principal, agregar el firebaseConfig de CREDENCIALES.md e inicializar:

```javascript
// Firebase
const firebaseConfig = { /* copiar de CREDENCIALES.md */ };
firebase.initializeApp(firebaseConfig);
const db = firebase.database();
```

### 4.3 Publicar status de conexion
Cuando `index.html` carga, publicar que esta online:

```javascript
// Publicar status
const statusRef = db.ref('chile-robot/status');
const deviceName = navigator.userAgent.includes('Windows') ? 'Windows' :
                   navigator.userAgent.includes('Mac') ? 'Mac' :
                   navigator.userAgent.includes('Linux') ? 'Linux' : 'Desktop';
const browserName = navigator.userAgent.includes('Chrome') ? 'Chrome' :
                    navigator.userAgent.includes('Firefox') ? 'Firefox' :
                    navigator.userAgent.includes('Safari') ? 'Safari' : 'Browser';

statusRef.set({
  online: true,
  device: deviceName + ' - ' + browserName,
  lastSeen: Date.now()
});

// Cuando se cierra la pagina, marcar offline
statusRef.onDisconnect().set({
  online: false,
  device: deviceName + ' - ' + browserName,
  lastSeen: Date.now()
});
```

### 4.4 Escuchar configuracion de alarma desde Firebase
Cuando el control remoto envia una configuracion de alarma, `index.html` debe recibirla y aplicarla:

```javascript
db.ref('chile-robot/alarm').on('value', (snapshot) => {
  const alarm = snapshot.val();
  if (!alarm || !alarm.time) return;

  // Aplicar hora de alarma al input existente
  document.getElementById('alarm-time').value = alarm.time;

  // Aplicar modo despertador
  // (activar/desactivar el toggle de modo despertador segun alarm.despertadorMode)

  // Aplicar tono
  if (alarm.tone) {
    document.getElementById('alarm-sound').src = alarm.tone;
  }

  // Aplicar campos de texto
  // ttsMessage -> input de "que quieres que diga"
  // reminders -> textarea de "que quieres que salga"
  // spotifyLink -> input de link spotify
  // alexaCommand -> input de comando alexa

  // Re-setear la alarma con los nuevos valores
  // (llamar a la funcion que configura el setTimeout de la alarma)
});
```

**IMPORTANTE:** Asegurate de que los IDs de los elementos HTML coincidan. Lee el `index.html` actual para ver cuales son los IDs reales de cada input y adaptate a ellos.

### 4.5 Escuchar comando de apagar alarma

```javascript
db.ref('chile-robot/alarm/stopAlarm').on('value', (snapshot) => {
  if (snapshot.val() === true) {
    stopAlarm(); // la funcion que ya existe en index.html para detener la alarma
    // Resetear el flag
    db.ref('chile-robot/alarm/stopAlarm').set(false);
  }
});
```

### 4.6 Sistema de vistas
Implementar el sistema de vistas como se describio en la Tarea 2 (envolver reloj en `#view-clock`, crear `#view-finance` y `#view-calendar`, escuchar Firebase para cambiar vista).

### 4.7 Cambiar selector de tono
El `index.html` actual usa un file input para seleccionar el tono. Cambiarlo por un `<select>` dropdown con los 3 tonos del repositorio (igual que en remote.html). Los archivos de audio ya estan en el repositorio:
- `lesiakower-lo-fi-alarm-clock-243766.mp3`
- `freesound_community-alarm-clock-90867.mp3`
- `Tono Incendio.mpeg`

---

## Resumen de archivos

| Archivo | Accion | Que hacer |
|---------|--------|-----------|
| `index.html` | MODIFICAR | Agregar Firebase, sistema de vistas, recibir alarma remota, status online, vista finanzas, vista calendario, cambiar file input por select de tonos |
| `remote.html` | CREAR | Control remoto movil completo |
| `CREDENCIALES.md` | SOLO LEER | Contiene todas las claves y configuraciones |
| `PROJECT_CONTEXT.md` | SOLO LEER | Contiene contexto del proyecto y decisiones de diseno |

---

## Checklist de calidad (verifica TODOS estos puntos)

- [ ] `index.html` carga sin NINGUN error en la consola del navegador
- [ ] `remote.html` carga sin errores y se ve correctamente en viewport de 375px de ancho
- [ ] Firebase se conecta y sincroniza datos entre ambas paginas en tiempo real
- [ ] Los 3 tonos de alarma se pueden seleccionar y reproducir correctamente
- [ ] La alarma se puede configurar desde el control remoto y se aplica en index.html
- [ ] La alarma se puede apagar desde el control remoto
- [ ] El cambio de vista (reloj/finanzas/calendario) funciona desde el control remoto
- [ ] La vista de finanzas muestra al menos el valor del dolar (mindicador.cl)
- [ ] La vista de finanzas muestra SPCX (o "datos no disponibles" si Yahoo Finance falla)
- [ ] La vista de calendario muestra la estructura de 2 semanas con los dias correctos
- [ ] Si el calendario no esta publico, muestra mensaje de error amigable (no error de JS)
- [ ] Toda la estetica es coherente: fondo negro, colores del proyecto, tipografia sistema
- [ ] No hay console.log de debug en el codigo final
- [ ] No hay codigo muerto ni TODOs sin resolver
- [ ] Todas las rutas de archivos son relativas (funcionar en GitHub Pages en subdirectorio `/Chile-robot/`)
- [ ] El Wake Lock que ya existe en index.html sigue funcionando
- [ ] La funcionalidad de alarma existente (tanto modo simple como modo despertador) sigue funcionando igual
- [ ] El menu de configuracion con ESPACIO sigue funcionando (y sigue ignorando ESPACIO cuando el foco esta en un input/textarea)

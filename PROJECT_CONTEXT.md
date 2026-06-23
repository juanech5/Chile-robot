# Chile-Robot — Contexto del Proyecto

## Resumen

Chile-Robot es una aplicación web de reloj/despertador inteligente desplegada en GitHub Pages. Funciona como una pantalla de escritorio (computador) que muestra un reloj analógico minimalista con funciones avanzadas de alarma. Se controla remotamente desde un celular.

- **URL de producción:** https://juanech5.github.io/Chile-robot/
- **Repositorio:** https://github.com/juanech5/Chile-robot
- **Hosting:** GitHub Pages (solo archivos estáticos, no hay backend propio)
- **Backend en tiempo real:** Firebase Realtime Database (gratuito, para sincronización entre dispositivos)

## Estructura de archivos actual

```
Chile-robot/
├── index.html                              # Página principal (reloj + despertador)
├── lesiakower-lo-fi-alarm-clock-243766.mp3 # Tono de alarma: Lo-fi
├── freesound_community-alarm-clock-90867.mp3 # Tono de alarma: Clásico
├── Tono Incendio.mpeg                      # Tono de alarma: Incendio
└── PROJECT_CONTEXT.md                      # Este archivo
```

## Stack técnico

- **HTML/CSS/JS puro** — No hay frameworks, bundlers ni dependencias npm. Todo es vanilla.
- **Estilo:** Fondo negro (#000), texto blanco, acentos en rojo (#ff3b30). Fuente del sistema (system-ui). Diseño minimalista, premium, tipo Apple Watch.
- **GitHub Pages** como hosting estático.
- **Firebase Realtime Database** para comunicación entre dispositivos (pendiente de implementar).

## Funcionalidad actual de index.html

### Reloj analógico
- Esfera con 12 marcas horarias (4 principales en 12/3/6/9, 8 estándar)
- Manecilla de hora, minuto y segundero (rojo)
- Actualización fluida cada 50ms

### Sistema de alarma (dos modos)

**Modo Simple (toggle OFF):**
- Solo suena el tono de alarma en loop
- Modal de "ALARMA SONANDO" con botón apagar

**Modo Despertador (toggle ON):**
Secuencia al activarse:
1. Suena el tono de alarma 2 veces completas (no loop)
2. Consulta el clima de Las Condes via `wttr.in/Las+Condes?format=j1`
3. TTS (Text-to-Speech, Web Speech API, lang: es-ES) dice: fecha, clima en °C, y el mensaje personalizado del usuario
4. Los recordatorios/pendientes aparecen en un panel lateral derecho con texto grande (2rem) — NO se leen por TTS
5. TTS dice "Alexa, [comando]" para activar Alexa por voz (opcional)
6. Abre link de Spotify en nueva pestaña (opcional)

### Pantalla de alarma sonando (fullscreen)
- Layout de 2 columnas: izquierda (hora 6rem, fecha, clima, botón apagar) + derecha (pendientes grandes)
- El panel derecho solo se muestra si hay pendientes configurados

### Configuración (modal)
- Se abre con ESPACIO (no cierra si el foco está en un input/textarea)
- Selector de hora
- Toggle "Modo Despertador" que expande panel con:
  - Selector de archivo de audio (tono)
  - Campo TTS: "¿Qué quieres que diga?"
  - Textarea: "¿Qué quieres que salga en pantalla?"
  - Link de Spotify
  - Comando para Alexa

### Otras funcionalidades
- Wake Lock API para evitar suspensión de pantalla
- Indicador verde (punto) cuando Wake Lock está activo

## APIs externas en uso

| API | Propósito | Autenticación |
|-----|-----------|---------------|
| `wttr.in` | Clima Las Condes | Ninguna (gratuita, sin key) |
| Web Speech API | Text-to-Speech | Nativa del navegador |
| Wake Lock API | Evitar suspensión | Nativa del navegador |

## Decisiones de diseño

- **Fondo negro puro (#000000)** en toda la aplicación
- **Bordes y separadores:** tonos de gris muy oscuro (#222, #333)
- **Texto principal:** blanco (#fff)
- **Texto secundario:** gris (#666, #888)
- **Acento/peligro:** rojo iOS (#ff3b30)
- **Éxito:** verde (#4cd964 para Wake Lock, #1db954 para Spotify)
- **Inputs:** fondo #222, borde #444, placeholder #555
- **Botones principales:** blanco con texto negro
- **Sin emojis en la UI** (excepto 🎵 en el selector de archivo)
- **Tipografía:** system-ui stack, pesos 200-600
- **Transiciones:** 0.2-0.4s ease
- **Border-radius:** 6px para inputs/botones, 12px para cards/modals, 50px para pill buttons

## Archivos de audio disponibles

Los siguientes archivos ya están en el repositorio y deben usarse como opciones de tono:
1. `lesiakower-lo-fi-alarm-clock-243766.mp3` — "Lo-fi Alarm"
2. `freesound_community-alarm-clock-90867.mp3` — "Alarm Clock Clásico"  
3. `Tono Incendio.mpeg` — "Tono Incendio"

## Zona horaria

La aplicación opera en **America/Santiago** (Chile continental, UTC-3 / UTC-4).

## Notas importantes para desarrollo

- NO usar frameworks ni librerías externas de JS/CSS (excepto Firebase SDK via CDN)
- Todo debe funcionar en GitHub Pages (archivos estáticos solamente)
- La comunicación entre dispositivos se hará via Firebase Realtime Database
- Las Google APIs que se usen deben ser con API keys del tier gratuito
- Para datos financieros, usar solo APIs públicas y gratuitas (sin autenticación si es posible)
- SpaceX cotiza en NASDAQ bajo el ticker **SPCX** (IPO: 12 junio 2026)
- El calendario de Google requiere Google Calendar API con API key — el usuario NO tiene una aún

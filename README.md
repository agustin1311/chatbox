# 🤖 WhatsApp Promo Bot

Bot de WhatsApp que se conecta con **tu propio número** (escaneando un código QR, igual que WhatsApp Web) para:

- 💬 Responder automáticamente a tus clientes (chatbot con menú configurable)
- 📢 Enviar promociones/anuncios a todos tus contactos registrados que hayan aceptado recibirlas
- 📅 Agendar citas conversacionalmente (el cliente escribe, el bot pregunta servicio/fecha/hora y confirma)
- 🗂️ Guardar automáticamente en una base de datos a todo el que te escriba, como contacto
- 🖥️ Panel web simple para ver el estado de conexión y escanear el QR

---

## ⚠️ Léeme antes de usarlo (importante)

1. **Este proyecto usa `whatsapp-web.js`**, una librería no oficial que automatiza WhatsApp Web mediante un navegador headless. No es la API oficial de Meta. Funciona muy bien para negocios pequeños/medianos, pero:
   - Viola técnicamente los Términos de Servicio de WhatsApp, por lo que existe **riesgo de que tu número sea bloqueado**, especialmente si envías muchos mensajes muy rápido a gente que no te ha escrito antes.
   - Si necesitas algo 100% oficial y sin riesgo para producción a gran escala, la alternativa es migrar a la **WhatsApp Business Cloud API de Meta** (de pago por mensaje, requiere registro de negocio).
2. **No envíes promociones a números que nunca te han contactado o que no dieron su consentimiento.** Esto es spam, aumenta muchísimo el riesgo de bloqueo, y en muchos países es ilegal enviar publicidad no solicitada por WhatsApp/SMS sin opt-in (por ejemplo, normativas tipo GDPR en Europa o leyes locales de protección de datos). Este proyecto solo envía promociones a contactos marcados como `opted_in = true`.
3. El envío masivo (`/api/broadcast`) tiene un **límite de velocidad configurable** (`BROADCAST_RATE_PER_MINUTE` en `.env`) para reducir el riesgo de bloqueo. No lo subas a un número muy alto.
4. Usa preferiblemente un número de WhatsApp dedicado al negocio (no tu número personal principal), por si acaso.

---

## 📦 Requisitos

- Node.js 18 o superior
- Un número de WhatsApp activo en tu celular (para escanear el QR)
- En Linux, puede que necesites instalar dependencias del sistema para Chromium (Puppeteer). Ver sección de solución de problemas.

---

## 🚀 Instalación

```bash
git clone <la-url-de-tu-repo>
cd whatsapp-bot-project
npm install
cp .env.example .env
```

Edita `.env` y como mínimo cambia:
- `API_KEY` → pon una clave secreta tuya (protege tu API)
- `BUSINESS_NAME` → el nombre de tu negocio
- `BROADCAST_RATE_PER_MINUTE` → cuántos mensajes de promoción por minuto (recomendado: 6-10)

---

## ▶️ Uso

```bash
npm start
```

Esto va a:
1. Levantar el servidor web en `http://localhost:3000`
2. Mostrar un código QR en la terminal (y también en `http://localhost:3000` en el panel web)
3. Al escanear el QR desde tu celular (**WhatsApp > Configuración > Dispositivos vinculados > Vincular un dispositivo**), el bot queda conectado usando tu número

La sesión queda guardada localmente en la carpeta `.wwebjs_auth/`, así que no tienes que escanear el QR cada vez que reinicies el bot.

---

## 🧠 Personalizar el chatbot

Edita `data/bot-config.json` (no requiere tocar código) para cambiar:
- El mensaje de bienvenida
- Las opciones del menú
- El listado de promociones
- Los mensajes de agendamiento de citas
- El horario de atención

---

## 📡 API disponible

Todos los endpoints requieren el header `x-api-key: <tu API_KEY del .env>`.

| Método | Endpoint | Descripción |
|---|---|---|
| GET | `/api/status` | Estado de conexión de WhatsApp y QR actual |
| GET | `/api/contacts` | Lista todos los contactos guardados |
| POST | `/api/contacts` | Agrega un contacto manualmente `{ phone, name }` |
| DELETE | `/api/contacts/:phone` | Elimina un contacto |
| POST | `/api/contacts/:phone/opt-in` | Activa/desactiva promociones para ese contacto `{ optedIn: true|false }` |
| POST | `/api/send` | Envía un mensaje individual `{ phone, message }` |
| POST | `/api/broadcast` | Envía una promoción a todos los contactos con opt-in `{ title, message }` |
| GET | `/api/broadcasts` | Historial de envíos masivos |
| GET | `/api/appointments` | Lista de citas agendadas |
| POST | `/api/appointments/:id/cancel` | Cancela una cita |

### Ejemplo: enviar una promoción a todos tus contactos

```bash
curl -X POST http://localhost:3000/api/broadcast \
  -H "Content-Type: application/json" \
  -H "x-api-key: TU_API_KEY" \
  -d '{"title": "Promo julio", "message": "🎉 20% de descuento hoy usando el código JULIO20"}'
```

---

## 🗂️ Estructura del proyecto

```
whatsapp-bot-project/
├── src/
│   ├── index.js            # Servidor principal
│   ├── config.js           # Configuración desde .env
│   ├── db.js                # Almacenamiento en archivo JSON (data/bot.db.json)
│   ├── contactsRepo.js      # Acceso a datos de contactos
│   ├── appointmentsRepo.js  # Acceso a datos de citas
│   ├── chatbot.js           # Lógica conversacional del bot
│   ├── whatsapp.js          # Conexión con WhatsApp (whatsapp-web.js)
│   ├── broadcast.js         # Envío masivo de promociones con rate limiting
│   ├── middleware/auth.js   # Autenticación por API key
│   └── routes/api.js        # Endpoints REST
├── data/
│   └── bot-config.json      # Mensajes/menú del chatbot (editable sin código)
├── public/
│   └── index.html           # Panel web para ver el QR
├── .env.example
├── package.json
└── README.md
```

---

## 🛠️ Solución de problemas

**El QR no aparece / Chromium falla al iniciar (Linux):**
```bash
sudo apt-get install -y ca-certificates fonts-liberation libasound2 \
  libatk-bridge2.0-0 libatk1.0-0 libcups2 libdbus-1-3 libgbm1 \
  libnspr4 libnss3 libx11-xcb1 libxcomposite1 libxdamage1 libxrandr2 \
  xdg-utils libu2f-udev libvulkan1
```

**Se desconectó el bot:** vuelve a correr `npm start`, si no reconecta solo, borra la carpeta `.wwebjs_auth/` y escanea el QR de nuevo (tendrás que vincular el dispositivo otra vez desde tu celular).

**Quiero migrar a la API oficial de Meta más adelante:** el código está organizado para que solo tengas que reemplazar `src/whatsapp.js` por una implementación que use la Cloud API; el resto del proyecto (chatbot, citas, broadcast, contactos) no necesita cambios.

---

## 📄 Licencia

MIT — usa y modifica libremente este proyecto para tu negocio.

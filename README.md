# Mi red — código listo para desplegar

Plataforma local de **negocios, productos, servicios, eventos, lugares y promociones**.

Stack: **Node.js + Express** (backend) · **Firebase Auth + Firestore** · **Cloudinary** · **PayPal Subscriptions**.

---

## Estructura del proyecto

```
mi-red/
├── server.js           # API Express completa
├── package.json
├── firestore.rules     # Reglas: cliente NO escribe directo
├── .env.example        # Variables de entorno (solo referencia)
├── public/
│   └── app.html        # Frontend SPA responsive en español
└── README.md
```

---

## Errores que se corrigieron respecto a tu versión original

| Problema | Corrección |
|----------|------------|
| Rutas `PUT` y `DELETE` sin prefijo `/api` | Ahora son `/api/:collection/:id` |
| Faltaba `public/app.html` | Se incluye un frontend completo y funcional |
| `paypal-rest-sdk` en dependencies sin uso | Eliminado (se usa `fetch` nativo) |
| Query Firestore `status != "deleted"` sin índice | Filtrado en memoria para evitar error de índice |
| No había endpoint de estado Premium | Añadido `GET /api/premium/status` |
| Firebase config de ejemplo con valores dummy | Debes pegar tu config real en `app.html` |

---

## Paso a paso para que funcione al 100 %

### 1. Firebase Console (proyecto `mi-red-a687d`)

1. **Authentication → Sign-in method** → habilita **Correo electrónico/contraseña**.
2. **Authentication → Settings → Authorized domains** → agrega el dominio de Render cuando lo tengas (ej. `tu-app.onrender.com`).
3. **Firestore Database** → crea la base (modo producción).
4. **Firestore → Rules** → pega el contenido de `firestore.rules` y publica.
5. **Project settings → Service accounts → Generate new private key** → descarga el JSON.
6. **Project settings → Your apps** → copia la config web (`apiKey`, `authDomain`, etc.).

### 2. Editar `public/app.html` (Firebase config)

Abre `public/app.html` y busca el bloque:

```js
const firebaseConfig = {
  apiKey: "AIzaSyDummyReplaceWithYourRealKey",
  authDomain: "mi-red-a687d.firebaseapp.com",
  projectId: "mi-red-a687d",
  ...
};
```

Sustituye **todos** los valores por los de tu consola Firebase (son públicos, no son secretos).

### 3. Cloudinary

1. Console → API Keys → copia **Cloud name**, **API Key** y **API Secret**.
2. En Render usa:
   - `CLOUDINARY_CLOUD_NAME=olte8v49` (o el tuyo)
   - `CLOUDINARY_API_KEY=...`
   - `CLOUDINARY_API_SECRET=...`

### 4. PayPal (Sandbox primero)

1. [developer.paypal.com](https://developer.paypal.com) → crea app REST (Sandbox).
2. Copia **Client ID** y **Secret**.
3. Crea un **Producto** + **Plan de suscripción mensual de USD 5.00** → copia el `Plan ID`.
4. Crea un **Webhook** que apunte a:
   ```
   https://TU-SERVICIO.onrender.com/api/paypal/webhook
   ```
   Eventos recomendados: `BILLING.SUBSCRIPTION.ACTIVATED`, `CANCELLED`, `SUSPENDED`, `EXPIRED`.
5. Variables:
   ```
   PAYPAL_MODE=sandbox
   PAYPAL_CLIENT_ID=...
   PAYPAL_CLIENT_SECRET=...
   PAYPAL_PLAN_ID=P-...
   PAYPAL_WEBHOOK_ID=...
   ```

### 5. Subir a GitHub

```bash
cd mi-red
git init
git add .
git commit -m "Mi red — versión funcional"
# Crea el repo en GitHub y:
git remote add origin https://github.com/TU_USUARIO/mi-red.git
git branch -M main
git push -u origin main
```

**Nunca subas** el JSON de service account ni secretos a GitHub.

### 6. Desplegar en Render (plan gratuito)

1. Render → **New → Web Service** → conecta el repositorio.
2. Runtime: **Node**
3. Build command: `npm install`
4. Start command: `npm start`
5. **Environment** → agrega **todas** estas variables:

| Variable | Valor |
|----------|-------|
| `FIREBASE_SERVICE_ACCOUNT_JSON` | Pega el JSON **completo** de la service account |
| `CLOUDINARY_CLOUD_NAME` | `olte8v49` (o el tuyo) |
| `CLOUDINARY_API_KEY` | tu API Key |
| `CLOUDINARY_API_SECRET` | tu API Secret |
| `PAYPAL_MODE` | `sandbox` |
| `PAYPAL_CLIENT_ID` | ... |
| `PAYPAL_CLIENT_SECRET` | ... |
| `PAYPAL_PLAN_ID` | `P-...` |
| `PAYPAL_WEBHOOK_ID` | ... |
| `PUBLIC_APP_URL` | `https://TU-SERVICIO.onrender.com` |
| `ALLOWED_ORIGIN` | `https://TU-SERVICIO.onrender.com` |

6. Deploy. Cuando termine, abre:
   ```
   https://TU-SERVICIO.onrender.com/api/health
   ```
   Debes ver:
   ```json
   {
     "ok": true,
     "firebaseAdmin": true,
     "cloudinary": true,
     "paypalConfigured": true
   }
   ```

7. En Firebase → Authorized domains → agrega `TU-SERVICIO.onrender.com`.

### 7. Lista de comprobación

- [ ] `/api/health` → los tres flags en `true`
- [ ] Registrar cuenta de prueba
- [ ] Iniciar / cerrar sesión
- [ ] Recuperar contraseña
- [ ] Publicar un negocio y verlo en el listado
- [ ] Publicar producto, servicio, evento, lugar y promoción
- [ ] Subir imagen → aparece URL de Cloudinary
- [ ] Editar publicación propia
- [ ] Segundo usuario **no** puede editar/eliminar la ajena
- [ ] Rechazar ubicación → búsqueda por ciudad sigue funcionando
- [ ] PayPal Sandbox: aprobar suscripción → Premium activo
- [ ] Diseño correcto en móvil y escritorio

---

## Endpoints de la API

| Método | Ruta | Auth | Descripción |
|--------|------|------|-------------|
| GET | `/api/health` | No | Estado del servidor |
| GET | `/api/public/:collection` | No | Listado público (`?q=&city=&limit=`) |
| GET | `/api/mine` | Sí | Publicaciones del usuario |
| POST | `/api/:collection` | Sí | Crear |
| PUT | `/api/:collection/:id` | Sí | Actualizar (solo dueño) |
| DELETE | `/api/:collection/:id` | Sí | Soft-delete (solo dueño) |
| POST | `/api/cloudinary/signature` | Sí | Firma para subir imágenes |
| POST | `/api/paypal/create-subscription` | Sí | Crear suscripción |
| GET | `/api/paypal/subscription/:id` | Sí | Verificar suscripción |
| GET | `/api/premium/status` | Sí | ¿Tiene Premium? |
| POST | `/api/paypal/webhook` | No* | Eventos PayPal (*verifica firma) |

Colecciones válidas: `businesses`, `products`, `services`, `events`, `venues`, `advertisements`.

---

## Limitaciones conocidas (honestas)

- La búsqueda filtra en memoria sobre un lote limitado (máx. ~80 docs). Para catálogos grandes hace falta paginación con cursores e índices compuestos.
- “Cerca de ti” pide geolocalización pero **aún no ordena por distancia** ni muestra mapa.
- No hay carrito, reservas, reseñas ni cobro de campañas publicitarias.
- Al eliminar una publicación no se borran las imágenes de Cloudinary (haría falta API de borrado firmado).
- En plan gratuito de Render el servicio se duerme tras inactividad; el primer request puede tardar ~30 s.

---

## Soporte rápido de errores

| Síntoma | Causa probable |
|---------|----------------|
| `firebaseAdmin: false` | `FIREBASE_SERVICE_ACCOUNT_JSON` mal pegado o inválido |
| `cloudinary: false` | Faltan `CLOUDINARY_API_KEY` o `SECRET` |
| Error 401 al publicar | Token expirado o dominio no autorizado en Firebase |
| CORS error | `ALLOWED_ORIGIN` no coincide exactamente con la URL de la app |
| Imágenes no suben | Firma Cloudinary o límites del plan |
| Premium no se activa | Webhook no configurado o `PAYPAL_WEBHOOK_ID` incorrecto |

Con estos pasos y este código la aplicación queda **funcional de extremo a extremo**.

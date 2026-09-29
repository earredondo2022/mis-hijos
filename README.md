# Mis hijos — PWA para iPhone

App web instalable (PWA) para registrar los periodos en que mis dos hijos están conmigo y los eventos o gastos asociados (compras, útiles, consultas médicas, etc.). Funciona 100% en el teléfono: los datos se guardan en `localStorage` y nunca se envían a ningún servidor. GitHub Pages solo sirve los archivos de la app.

## Tecnologías

- **Lenguajes**: HTML5, CSS3 y JavaScript moderno (ES2020, sin TypeScript).
- **Sin frameworks ni librerías**: no usa React, Vue ni jQuery, ni dependencias npm.
- **Sin compilación**: no hay bundler, `package.json` ni paso de build. Los archivos se publican tal cual y basta con editar `index.html` y hacer push.
- **APIs del navegador que usa**:
  - `localStorage`: guarda los datos en el teléfono.
  - Service Worker y Cache API: permiten usar la app sin conexión.
  - Web App Manifest: permite instalarla en la pantalla de inicio.
  - Web Share API: exporta el respaldo con el menú Compartir de iOS.
  - `Intl.NumberFormat`: da formato a los montos en CLP.
  - `matchMedia('(prefers-color-scheme: dark)')`: detecta el tema claro u oscuro del sistema.

## Archivos

| Archivo | Función |
|---|---|
| `index.html` | Toda la app (HTML, CSS y JS en un solo archivo, sin dependencias externas) |
| `sw.js` | Service worker: permite abrir la app sin conexión (estrategia "red primero", con caché de respaldo) |
| `manifest.json` | Manifiesto PWA (nombre, íconos, modo standalone) |
| `icon-180.png` | Ícono para la pantalla de inicio de iOS (`apple-touch-icon`) |
| `icon-192.png`, `icon-512.png` | Íconos del manifiesto |
| `.gitignore` | Evita subir al repo los respaldos exportados (`respaldo-hijos-*.json`), que contienen datos reales |
| `README.md` | Este documento |

Los archivos de la app deben quedar en la **raíz** del repositorio.

## Funcionalidades (v1.1)

- **Periodos**: fecha y hora de llegada y de salida (`datetime-local`), con atajos +7 y +14 días. En un periodo nuevo, la llegada parte a las **20:00** del día elegido (o de hoy). Al tocar la salida vacía, también parte a las 20:00, del día de llegada. Ambas horas se pueden cambiar. Avisa si un periodo se superpone con otro. Hoy el régimen es de 14 días, pero puede cambiar a 7.
- **Calendario mensual**: la semana parte el lunes. Los días completos se ven en azul y los de llegada y salida en diagonal (días parciales). Los días con eventos llevan un punto amarillo.
- **Eventos**: fecha, tipo, para quién (ambos, hijo 1 o hijo 2), monto en CLP (opcional), N° de factura o boleta (opcional) y detalle libre. Los tipos por defecto son Compra de zapatos, Útiles escolares y Consulta médica, y se pueden agregar tipos libres.
- **Resumen mensual**: días con los niños y gastos por tipo y por hijo.
- **Respaldo**: exportar e importar un archivo JSON. Importar **reemplaza** los datos actuales, previa confirmación. Si pasan 14 días sin respaldar, la app muestra un aviso.
- **Ajustes**: nombres de los hijos, tipos de evento, tamaño de los datos guardados y botón para borrar todo.
- **Tema claro/oscuro**: botón 🌙/☀️ en el encabezado. Mientras no se toque, sigue el tema del sistema. Al tocarlo, fija el tema contrario y lo recuerda en ese teléfono (clave `localStorage` `calhijos:tema`).
- Solo texto, sin fotos de boletas, para ahorrar espacio.

## Reglas de negocio y supuestos

- Un periodo aplica a **ambos hijos** a la vez.
- Los "días conmigo" del mes se calculan como las horas reales dentro del mes divididas por 24. Un periodo que cruza de mes se reparte entre ambos meses.
- Los montos se guardan como enteros en CLP.

## Modelo de datos

Se guarda en la clave `localStorage` `calhijos:v1`, con el mismo formato que usa el archivo de respaldo:

```json
{
  "app": "calendario-hijos",
  "version": 1,
  "hijos": ["Hijo 1", "Hijo 2"],
  "tipos": ["Compra de zapatos", "Útiles escolares", "Consulta médica"],
  "periodos": [
    { "id": "uuid", "inicio": "2026-09-09T20:00", "fin": "2026-09-23T20:00", "nota": "" }
  ],
  "eventos": [
    { "id": "uuid", "fecha": "2026-09-15", "tipo": "Compra de zapatos", "hijo": "ambos",
      "monto": 35990, "factura": "B-1234", "detalle": "Zapatillas colegio" }
  ],
  "ultimoRespaldo": "2026-09-29T19:00:00.000Z"
}
```

- Las fechas de los periodos están en hora local, con formato `YYYY-MM-DDTHH:MM`.
- `hijo` puede ser `"ambos"`, `"0"` o `"1"`, que son índices del arreglo `hijos`.
- Al exportar se agrega `exportadoEn`.
- Al importar, `normalize()` valida el archivo y descarta los registros inválidos.

## Publicación en GitHub Pages

Publicada en **https://earredondo2022.github.io/mis-hijos/**, desde el repo público `earredondo2022/mis-hijos`, rama `main`, carpeta raíz.

Se creó con git y GitHub CLI (`gh`):

```powershell
git init -b main
git add index.html sw.js manifest.json icon-180.png icon-192.png icon-512.png
git commit -m "Primera versión de la PWA Mis hijos"
gh repo create earredondo2022/mis-hijos --public --source . --remote origin --push
gh api -X POST repos/earredondo2022/mis-hijos/pages -f "source[branch]=main" -f "source[path]=/"
```

Para ver el estado del despliegue (`built` = publicado):

```powershell
gh api repos/earredondo2022/mis-hijos/pages/builds/latest --jq ".status, .commit"
```

**Push por HTTPS:** este computador no tiene una llave SSH asociada a la cuenta, así que el push por SSH falla con `Permission denied (publickey)`. El remoto usa HTTPS, con `gh` como gestor de credenciales (`gh auth setup-git`).

**El repo debe ser público:** con GitHub Free, Pages solo publica desde repos públicos. Si se pasa a privado, el sitio se despublica. Aun con un plan pagado, el sitio seguiría siendo público. Que el repo sea público solo expone el código, no los datos, porque los datos nunca salen del teléfono.

**Instalar en el iPhone:** abrir la URL en Safari, tocar Compartir y luego "Agregar a inicio", dejando activado "Abrir como app web".

## Advertencias de iOS

- **Borrar el ícono de la app borra sus datos.** Hay que exportar un respaldo antes y guardarlo en Archivos o iCloud Drive.
- **Usar solo el ícono instalado**, no una pestaña de Safari. Comprobado en iPhone 11: la app instalada parte vacía, con su almacenamiento separado del de Safari. Para pasar los datos de Safari a la app hay que exportar un respaldo en Safari e importarlo en la app.
- Safari puede borrar el almacenamiento de sitios web tras unos 7 días sin uso. Entiendo que las apps instaladas en la pantalla de inicio están exentas, pero hay que verificarlo en la documentación de WebKit.
- **El `localStorage` está atado al origen** (`TU-USUARIO.github.io`), así que:
  - Si cambia la URL o el dominio, la app empieza vacía y hay que exportar antes e importar después.
  - Otras apps publicadas en el mismo `github.io` comparten el origen. Por eso la clave lleva un prefijo propio (`calhijos:v1`).
- Límite aproximado de `localStorage`: unos 5 MB, cifra aproximada que varía según el navegador. Sobra para texto.

## Estado de pruebas

- **Probado en Chromium** con una pantalla simulada de iPhone: flujo de periodo, evento, cálculo de días y guardado.
- **Probado en iPhone 11 real**:
  - Abrir la app desde GitHub Pages en Safari y registrar periodos.
  - Exportar, borrar los datos e importar el respaldo: funciona.
  - Instalarla en la pantalla de inicio: abre como app, a pantalla completa.
- **Pendiente en iPhone real**:
  - Hora 20:00 por defecto en periodos nuevos.
  - Botón de tema claro/oscuro, y que recuerde el tema al volver a abrir la app.
  - Abrir la app sin conexión (modo avión).
  - Ciclo completo: exportar, borrar la app, reinstalar e importar.

## Actualizaciones

Basta con hacer push a `main`. La app toma la versión nueva la próxima vez que se abre con internet. Si se agregan archivos, hay que sumarlos a `ASSETS` en `sw.js` y subir la versión de `CACHE` (por ejemplo `calhijos-v2`).

## Ideas para siguientes versiones

- Sugerir el siguiente periodo según el régimen (7 o 14 días).
- Exportar los eventos a CSV.
- Importar combinando datos en lugar de reemplazarlos.

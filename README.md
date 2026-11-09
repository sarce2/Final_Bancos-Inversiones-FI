# Panel de Fondos

Dos páginas estáticas que muestran los saldos de las cinco empresas, leyéndolos de la
planilla de Google Sheets. No necesitan servidor, base de datos ni cuenta de Claude:
cualquiera con el link las abre desde la computadora o el celular.

- **`index.html`** — la portada: el resumen por empresa. Totales, ranking, bancos vs
  inversiones y dónde está la plata en cada una.
- **`detalle.html`** — el detalle fondo por fondo, con una pestaña por empresa.

Cada una tiene al pie un enlace a la otra.

## Archivos del repositorio

| Archivo        | Para qué sirve                                            | ¿Se toca? |
|----------------|-----------------------------------------------------------|-----------|
| `index.html`   | Resumen por empresa (portada).                             | No        |
| `detalle.html` | Detalle fondo por fondo.                                   | No        |
| `datos.json`   | Números de respaldo, por si la planilla no responde.       | Opcional  |
| `.nojekyll`    | Le dice a GitHub Pages que publique los archivos tal cual. | No        |
| `diagnostico.html` | Prueba las conexiones y dice qué falla. No muestra saldos. | No   |
| `robots.txt`   | Pide a los buscadores que no indexen las páginas.          | No        |

Los cinco tienen que quedar juntos en la raíz del repositorio.

**La URL de la planilla ya está puesta en los dos archivos y no hay que tocar nada.**

---

## Subir a GitHub Pages

1. En GitHub: **New repository**. Nombre, por ejemplo, `panel-fondos`.
2. **Add file → Upload files** y arrastrá los cinco archivos de esta carpeta.
   Si `.nojekyll` no aparece en el selector de archivos, activá "mostrar archivos
   ocultos" (en Mac, `Cmd + Shift + .`).
3. **Commit changes**.
4. **Settings → Pages**. En *Source* elegí **Deploy from a branch**;
   en *Branch*, `main` y carpeta `/ (root)`. **Save**.
5. Esperá uno o dos minutos. Los links quedan en:

```
https://TU-USUARIO.github.io/panel-fondos/               ← resumen por empresa
https://TU-USUARIO.github.io/panel-fondos/detalle.html   ← detalle fondo por fondo
```

### Antes de publicar, leé esto

En las cuentas gratuitas de GitHub, **GitHub Pages sólo funciona con repositorios
públicos**. Público quiere decir que cualquiera que llegue al link ve los saldos, y que
`datos.json` también se puede descargar directo. Las páginas piden no ser indexadas
(`robots.txt` y la etiqueta `noindex`), pero eso no es una contraseña.

Si los saldos no deberían ser públicos, conviene **Cloudflare Pages** en lugar de
GitHub: en el plan gratuito permite poner un candado por correo (Cloudflare Access),
y el que entra recibe un código de un solo uso en su email. Se suben los mismos
archivos, sin cambiar nada.

---

## Actualizar los números

No hay que volver a tocar nada: cargás los saldos en las pestañas de cada fondo de la
planilla y las páginas los reflejan solas al recargarlas (leen la planilla sin caché
cada vez que se abren).

### Con o sin tipo de cambio

Las páginas funcionan de las dos maneras y se dan cuenta solas:

- **Sin tipo de cambio** (la fila `tc` de la hoja `DATOS` vacía, que es como está hoy):
  cada monto se muestra en su moneda original y nada se convierte. El gráfico se parte
  en dos rankings, uno por moneda, y donde una empresa tiene las dos se muestra
  `Gs. 42.000.000 + US$ 2.000`. En el detalle fondo por fondo los totales de arriba
  aparecen separados, uno en guaraníes y otro en dólares.
- **Con tipo de cambio** (cargando la celda **B4 de la hoja `Consolidado`**): vuelven los
  porcentajes del total, el ranking único y, en el detalle, el total consolidado en guaraníes.

No hay nada que tocar en los archivos para pasar de un modo al otro: depende sólo de si
la planilla trae cotización o no.

### Cómo encuentran los datos

La URL publicada que está configurada apunta al documento completo, así que Google
devuelve la primera hoja (`Consolidado`). Las páginas se dan cuenta, buscan en el
documento publicado la pestaña **`DATOS`**, y leen esa. El identificador de esa hoja
queda guardado en el navegador, así que a partir de la segunda visita van derecho.

Todo esto pasa solo y sin avisos. Lo único importante: **la planilla tiene que seguir
publicada en la web** (Archivo → Compartir → Publicar en la web). Si alguna vez la
despublicás y volvés a publicarla, el link puede cambiar; en ese caso pegá el nuevo en
la línea `var ORIGEN_CSV = ...`, que está cerca del principio del script, **en los dos
archivos**.

> Compartir con "cualquiera con el link puede ver" **no** sirve: esa otra URL devuelve
> un redirect al login y el navegador la bloquea.

### Respaldo: `datos.json`

Si Google no responde, las páginas muestran los números de `datos.json` con un aviso.
Hoy ese archivo está en cero. Para que el respaldo sirva de algo, cada tanto podés
reemplazarlo por uno con los últimos saldos:

```
actualizado   fecha/hora ISO 8601 del último cambio (se muestra arriba)
fx.rate       guaraníes por dólar
fx.source     de dónde salió la cotización (se muestra)
fx.asOf       fecha/hora de esa cotización
fondos[]      un objeto por empresa
  id          identificador corto, sin espacios (sirve para el link directo: ...#eco-1)
  nombre      cómo se muestra, sin la moneda entre paréntesis
  moneda      "USD" o "PYG" — es lo que va entre paréntesis después del nombre
  bancos[]        filas: { label, amount, currency }
  inversiones[]   filas: { label, amount, currency }
```

`currency` puede ser distinta de la moneda de la empresa: una fila en dólares dentro de
una empresa en guaraníes se suma bien igual. Las filas con `label` vacío y `amount` 0 se
muestran en gris como lugares libres.

---

## Formato de la hoja `DATOS`

La hoja se llena sola con fórmulas desde las pestañas de cada fondo; no se edita a mano.
Primero tres filas de configuración (`tc`, `fuente`, `actualizado`), después la fila de
encabezados y las filas de datos:

```
fondo | moneda_fondo | seccion | nombre | moneda | monto
```

- `seccion` tiene que decir `banco` o `inversion`; cualquier otra fila se ignora.
- Si agregás más de tres filas por sección, las páginas las muestran todas.
- El nombre puede venir con la moneda entre paréntesis — `Eco 1 (Gs.)` —: se separa sola.
- Si le cambiás el nombre a la pestaña `DATOS`, cambiá también la línea `var HOJA = "DATOS";`
  en los dos archivos.

---

## Si algo no funciona

**Abrí primero `diagnostico.html`** (por ejemplo `https://TU-USUARIO.github.io/panel-fondos/diagnostico.html`).
Prueba una por una las direcciones que usa el panel, marca con un tilde o una cruz cuál
falla, y si encuentra la hoja `DATOS` dentro del documento publicado te da la URL exacta
para pegar en `ORIGEN_CSV`, con un botón para copiarla.

### Qué hace la página si no llega a la hoja `DATOS`

Lee igual la hoja `Consolidado`, que es la que Google devuelve cuando la URL apunta al
documento entero, y muestra el total de bancos e inversiones de cada empresa con un
aviso. Ojo: esos números vienen **convertidos a guaraníes con el tipo de cambio de la
celda B4**, y no incluyen el detalle banco por banco. Para ver cada moneda por separado
hay que publicar la hoja `DATOS`.


| Qué ves | Qué pasó |
|---|---|
| Aviso amarillo: *"no se pudo leer la planilla"* y todo en cero | La planilla dejó de estar publicada, o cambió el link. Republicá en Archivo → Compartir → Publicar en la web. |
| Aviso: *"la URL devolvió una página HTML"* | Se usó el link de "compartir" en vez del de "publicar en la web". |
| Aviso: *"no encontré la hoja DATOS"* | Se publicó sólo una hoja y no es la `DATOS`. Publicá el documento completo, o la hoja `DATOS`, y pegá esa URL en `ORIGEN_CSV` en los dos archivos. |
| Error 404, o el enlace del pie no va a ningún lado | Pages todavía está armando el sitio (tarda un par de minutos), o falta subir alguno de los dos archivos. |
| Abrís un archivo con doble clic y no carga nada | Normal: el navegador bloquea la lectura de archivos locales. Tiene que estar publicado. |

Abajo de todo, las dos páginas dicen siempre de dónde salieron los números que estás
viendo ("planilla de Google · hoja DATOS" o "archivo datos.json").

El resumen se adapta solo al modo oscuro del celular o de la computadora.

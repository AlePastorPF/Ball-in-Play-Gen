# URBA — Ball in Play / Secuencias Largas

Dashboard de performance (Ball in Play, secuencias largas y clasificación
Verde/Rojo por jugada), clonado del proyecto original de San Isidro Club y
personalizado con la identidad de URBA.

**El sitio se reconstruye y publica solo.** Subís una planilla `.csv` nueva a
`data/`, GitHub Actions la procesa y el sitio en GitHub Pages queda
actualizado en 1–2 minutos. No hay que tocar HTML ni JS a mano.

---

## Estado actual de este clon

- **Carpeta `data/`: vacía.** Todavía no tiene ningún partido cargado — el
  dashboard va a mostrar "Todavía no hay partidos cargados" hasta que subas
  la primera planilla.
- **Carpeta `assets/logos/`**: solo tiene `URBA.png` (el escudo que me
  pasaste). Los escudos de los rivales se van agregando a medida que
  aparecen en las planillas (ver más abajo).
- **`scripts/build_dashboard.py` → `CLUB_KEY_MAP`**: está vacío,
  listo para completarse con los rivales reales apenas tengas la primera
  planilla.
- **Login**: usuario `URBA`, contraseña `Condor1899` (elegida por mí en base
  al escudo — cóndor + año de fundación 1899. Se cambia fácil, ver abajo).
- **Colores**: rojo `#8F0D06` y dorado `#A9781F`, tomados directamente del
  escudo que subiste.

---

## Cómo está armado el proyecto

```
.
├── data/                          ← una planilla .csv por partido (vacío por ahora)
├── assets/logos/                  ← escudo de URBA + escudos de rivales (a completar)
├── template/dashboard_template.html   ← el dashboard (HTML/CSS/JS), sin datos
├── scripts/build_dashboard.py     ← procesa data/ + assets/logos/ y arma el HTML final
├── requirements.txt
└── .github/workflows/build-and-deploy.yml   ← automatización (GitHub Actions)
```

Cuando corre, `build_dashboard.py`:
1. Lee todos los `.csv` de `data/` (mismo formato de siempre: Session Start
   Date, Event, Session Name, Session Start, Session End, Tag Description,
   Tag Notes, Tag Start, Tag End, Tag Duration (secs), Partido, Resultado,
   Rueda, Etapa).
2. Calcula todo lo que ves en el dashboard: Ball in Play por partido,
   secuencias >60s, clasificación Verde/Rojo por jugada (desde la columna
   *Tag Notes*), KPIs, correlaciones, distribución de duración, etc.
3. Toma los escudos de `assets/logos/`, los reduce a miniatura y los
   incrusta como base64.
4. Inyecta todo eso en `template/dashboard_template.html` y escribe el
   resultado en `dist/index.html`.
5. GitHub Actions publica `dist/` en GitHub Pages (rama `gh-pages`).

`dist/` **no se versiona** (está en `.gitignore`): se genera de cero en cada
build.

---

## Puesta en marcha (una sola vez)

### 1. Crear el repositorio

- En GitHub: **New repository** → nombre sugerido `urba-ball-in-play-dashboard` → **Public**
- Subí **todo** el contenido de esta carpeta manteniendo la estructura
  (`data/`, `assets/`, `template/`, `scripts/`, `.github/`, `requirements.txt`,
  `.gitignore`, `README.md`) — por drag & drop en la web de GitHub, o:
  ```bash
  cd urba-ball-in-play-dashboard   # esta carpeta
  git init
  git add .
  git commit -m "Proyecto inicial del dashboard de URBA"
  git branch -M main
  git remote add origin https://github.com/TU_USUARIO/urba-ball-in-play-dashboard.git
  git push -u origin main
  ```

  ⚠️ Ojo con `.github/workflows/build-and-deploy.yml`: al subir por drag &
  drop en la web, asegurate de que quede exactamente en esa ruta (carpeta
  `.github`, con el punto, y adentro `workflows`). Si GitHub no la detecta al
  arrastrar la carpeta completa, creá el archivo manualmente con **Add file →
  Create new file** y escribí la ruta completa en el nombre.

### 2. Activar GitHub Pages (primera vez)

- **Settings → Pages → Source: GitHub Actions** → guardar
- Pestaña **Actions** → confirmá que corre **"Build and deploy dashboard"**
  (se dispara solo con el push del paso 1). Si no arranca, entrá al workflow
  y usá **Run workflow**.
- Cuando termine en verde, va a existir una rama nueva llamada **`gh-pages`**
  con el sitio ya construido.
- Volvé a **Settings → Pages → Source: Deploy from a branch** → elegí
  **`gh-pages` / `/ (root)`** → **Save**.

  (Se usa la rama `gh-pages` en vez del deploy nativo de Actions porque este
  último tiene un bug conocido de GitHub que a veces deja el despliegue
  trabado en "Queued" por horas. Publicar a una rama lo evita por completo.)

- El sitio queda en: `https://TU_USUARIO.github.io/urba-ball-in-play-dashboard/`

---

## Uso día a día: agregar el primer partido (y los siguientes)

1. Exportá la planilla del partido en el formato de siempre.
2. Subila a la carpeta `data/` (drag & drop en GitHub web, o `git add` + `commit` + `push`).
3. Listo. El push dispara el workflow automáticamente, reconstruye el
   dashboard con el partido incluido, y en 1–2 minutos el sitio está
   actualizado.

El número de fecha se toma del campo **Session Name** dentro de la planilla
(ej. "Fecha 1"), así que el nombre del archivo es libre.

## Agregar el escudo de un rival nuevo

1. Subí la imagen (`.png` o `.jpg`) a `assets/logos/`.
2. Nombrá el archivo con el mismo nombre que aparece en la columna
   **Partido** de la planilla (sin el " 2"/" 3" de segundos equipos).
3. Si el nombre del archivo no coincide exactamente con el de la columna
   *Partido* (por ejemplo, un acrónimo como "LPRC" que en realidad debería
   apuntar a un archivo `La_Plata.png`), agregá esa equivalencia en el
   diccionario `CLUB_KEY_MAP` al principio de `scripts/build_dashboard.py`:
   ```python
   CLUB_KEY_MAP = {
       "LPRC": "La_Plata",   # "Partido" en la planilla -> nombre de archivo en assets/logos/
   }
   ```
4. Commit + push. Si no subís logo para un club, el dashboard simplemente no
   muestra su escudo (no rompe nada).

---

## Cambiar el usuario/contraseña del login

Están hardcodeados en dos líneas de `template/dashboard_template.html`
(buscá `AUTH_USER` y `AUTH_PASS`). **Esto no es seguridad real** — es un
filtro simple en el navegador; cualquiera que vea el código fuente de la
página puede leer la contraseña. Sirve para que el link no ande circulando
libremente, no para proteger datos sensibles.

---

## Probarlo en tu computadora antes de subir (opcional)

```bash
pip install -r requirements.txt
python scripts/build_dashboard.py
# abrí dist/index.html en el navegador
```

---

## Notas de datos (heredadas del proyecto original)

- La duración de cada secuencia se toma de la columna **"Tag Duration (secs)"**
  (con Tag End − Tag Start solo como respaldo si esa celda está vacía).
  Algunas planillas pueden tener timestamps de Start/End superpuestos entre
  filas consecutivas; calcular por diferencia en esos casos infla el Ball in
  Play, por eso se prioriza la columna de duración.
- La clasificación Verde/Rojo por jugada sale de la columna **Tag Notes**
  (ej. "Verde - Penal", "Rojo - Try"). Las secuencias sin esa etiqueta se
  muestran como "Sin clasificar".
- La tipografía "Mark Pro" no está disponible en CDNs públicos por ser
  comercial; el dashboard usa Manrope + Inter como reemplazo visualmente
  cercano.

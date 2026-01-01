# Postventa MVP

Prototipo de seguimiento y **trazabilidad operacional** para vehículos en postventa (concesionaria).  
Incluye una API con **FastAPI** y una interfaz web en **HTML/JS** (Kanban + búsqueda + línea de tiempo).

## ¿Qué problema resuelve?

- Reduce el tiempo de ubicación de vehículos (búsqueda rápida por placa)
- Registra movimientos por ubicación con timestamps (llegada/salida) para medir **tiempos por zona**
- Aporta visibilidad operativa (tablero en tiempo real + timeline)

## Reglas clave

- **Una orden puede tener máximo 1 movimiento activo** (sin `ts_salida`) a la vez. El endpoint de mover cierra cualquier movimiento activo y crea el nuevo en una transacción.
- Los timestamps se guardan en UTC en backend y se muestran en hora local en el navegador.

## UX (modo demo)

- Selector de **rol** y nombre (opcional) en la interfaz. Cada movimiento registra `actor` para trazabilidad.
- Autocompletado de placa (`/api/placas`) para acelerar búsqueda.
- Los botones rápidos sugieren el "siguiente paso" según la ubicación actual (configurable en el frontend).
- Si alguien se equivoca al mover, se puede **deshacer el último movimiento** desde el mensaje verde (endpoint `/api/deshacer`).

## Requisitos

- Python 3.10 o superior
- pip (gestor de paquetes Python)
- SQLite (incluido con Python)  
- (Opcional) Node.js y Rust si vas a empaquetar un cliente con Tauri

## Instalación y ejecución

1. Clona o descarga este repositorio y entra al directorio `postventa-mvp`.
2. Crea un entorno virtual e instala dependencias:

   ```bash
   python -m venv .venv
   # Windows:
   .venv\Scripts\activate
   # macOS/Linux:
   # source .venv/bin/activate

   pip install -r requirements.txt
   ```

3. Ejecuta el servidor en modo desarrollo:

   ```bash
   uvicorn app.main:app --reload
   ```

4. Abre [http://127.0.0.1:8000](http://127.0.0.1:8000) en tu navegador.  
   Podrás buscar placas, ver el tablero Kanban y la línea de tiempo de cada orden.

## Solución rápida a errores (tablero no carga / no deja crear órdenes)

Si el tablero falla o al crear órdenes aparece error, casi siempre es por un `postventa.db` viejo (SQLite no migra esquemas automáticamente).

1) Detén el servidor (CTRL+C)
2) Elimina la base local:

```bash
python scripts/reset_db.py
```

3) Vuelve a levantar:

```bash
python -m uvicorn app.main:app --reload
```

## Endpoints principales

- `GET /api/tablero` → tablero Kanban por ubicación (solo movimientos activos)
- `GET /api/orden/{orden_id}/timeline` → timeline con actor y duración
- `POST /api/mover/{orden_id}/{ubicacion}?actor=...` → mueve y registra actor
- `POST /api/deshacer/{orden_id}?actor=...` → deshace el último movimiento (reabre el anterior)
- `POST /api/finalizar/{orden_id}?actor=...` → finaliza (cierra movimiento activo)
- `GET /api/placas?q=...` → autocomplete de placas
- `GET /api/ubicaciones` → lista de ubicaciones (para mantener UI sincronizada)
- `GET /api/reportes/tiempos?start=...&end=...` → tiempos por ubicación (movimientos cerrados)

5. Si deseas usar la base de datos de SQLite desde línea de comandos, puedes ejecutar las sentencias SQL en `sql/schema.sql` para crear el esquema y en `sql/seed_ubicacion.sql` para insertar las ubicaciones por defecto.

## Estructura de carpetas

- `app/`: código Python de la API y modelos.
- `sql/`: definición del esquema y datos de catálogo.
- `static/`: archivos HTML/JS para la interfaz.
- `requirements.txt`: lista de dependencias de Python.
- `README.md`: este archivo con instrucciones.


## Si ves 'Failed to fetch'
- Verifica que el backend esté corriendo: abre `http://127.0.0.1:8000/api/health`.
- Si estás sirviendo la UI con VS Code Live Server u otro puerto, abre la UI con:
  `http://127.0.0.1:5500/?api=http://127.0.0.1:8000`
  (ajusta el puerto 5500 al que uses).

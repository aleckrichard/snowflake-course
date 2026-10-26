# ❄️ Snowflake Data Loading & Practices

Este repositorio contiene apuntes, scripts de código SQL y comandos prácticos sobre **Carga de Datos en Snowflake** (*Data Loading*), incluyendo la gestión de *Stages*, automatización de cargas incrementales y manejo de errores.

---

## 📌 Contenido del Repositorio

- **Gestión de Stages**: Tipos de stages (User, Table, Named) y comandos de administración (`PUT`, `GET`, `LIST`, `REMOVE`).
- **Comandos `COPY INTO`**: Carga masiva e incremental de archivos CSV.
- **Manejo de Errores (`ON_ERROR`)**: Estrategias para omisión de filas o archivos completos durante la ingesta.

---

## 🛠️ Requisitos Previos

1. Cuenta activa de **Snowflake** (Free Trial o Empresarial).
2. Cliente de línea de comandos **SnowSQL** o acceso a **Snowsight UI**.
3. Archivos de datos de prueba (`.csv`).

---

## 🚀 Guía Rápida de Uso

### 1. Subir archivos a un Stage (SnowSQL)
```sql
PUT file:///ruta/local/tus_datos.csv @~ OVERWRITE = TRUE;
```

### 2. Cargar datos a una Tabla
```sql
COPY INTO MI_TABLA
FROM @~
FILE_FORMAT = (TYPE = CSV, SKIP_HEADER = 1, ON_ERROR = 'CONTINUE');
```

---

## 📝 Documentación Completa

Para revisar los apuntes teóricos explicados paso a paso, consulta el archivo:
📄 [`snowflake_file_loading_notes.md`](./snowflake_file_loading_notes.md)

---

## 🤝 Contribuciones
¡Las contribuciones y sugerencias son bienvenidas! Si encuentras algún error o quieres agregar más ejemplos prácticos, siéntete libre de abrir un *Pull Request* o crear un *Issue*.
# 📚 Apuntes de Snowflake: Carga de Archivos y Manejo de Errores

## 1. Tipos de Stages en Snowflake
Los *stages* son ubicaciones de almacenamiento donde se guardan los archivos de datos antes de cargarlos a las tablas.

*   **User Stage (`@~`):** Espacio de almacenamiento privado asignado a cada usuario. Ideal para pruebas individuales.
*   **Table Stage (`@%<nombre_tabla>`):** Stage interno asociado directamente a una tabla específica. 
*   **Named Internal Stage (ej. `@sales`):** Stage creado explícitamente (`CREATE STAGE`) que puede ser compartido por múltiples usuarios y tablas.

---

## 2. Comandos Principales de Gestión de Archivos (SnowSQL)

*   **Subir archivos al Stage (`PUT`):**
    Sube archivos desde la máquina local al stage de Snowflake. *(Nota: Snowflake comprime automáticamente los archivos al subirlos con `PUT`, agregando la extensión `.gz`).*
    ```sql
    PUT file:///Users/alex/Projects/snowflake/customers_sample_data.csv @~ OVERWRITE = TRUE;
    ```

*   **Listar contenido del Stage (`LIST`):**
    Permite ver qué archivos se encuentran almacenados:
    *   De mi user: `LIST @~;`
    *   De un table stage específico: `LIST @%emp_new_test;`

*   **Descargar archivos del Stage (`GET`):**
    Descarga un archivo desde el stage de Snowflake hacia la máquina local (incluyendo la extensión `.gz` por la compresión automática):
    ```sql
    GET @~/customers_sample_data.csv.gz file:///Users/alex/Downloads/;
    ```

*   **Eliminar archivos del Stage (`REMOVE`):**
    ```sql
    REMOVE @~/customers_sample_data.csv.gz;
    ```

---

## 3. Comandos de Carga Básica (`COPY INTO`)

El comando `COPY INTO` transfiere los datos desde un stage hacia una tabla de Snowflake.

*   **Desde un Table Stage hacia su tabla:**
    ```sql
    COPY INTO EMP_NEW_TEST 
    FROM @%emp_new_test 
    FILE_FORMAT = (TYPE = CSV, SKIP_HEADER = 1);
    ```

*   **Desde el User Stage hacia una tabla:**
    ```sql
    COPY INTO EMPLOYEE_USER_STAGE 
    FROM @~ 
    FILE_FORMAT = (TYPE = CSV, SKIP_HEADER = 1);
    ```

---

## 4. Cargas Masivas e Incrementales (*Daily Incremental Loads*)

*   **Funcionamiento:** Si se cargan lotes diarios (ej. `sales_01.csv`, `sales_02.csv`, `sales_03.csv`) ejecutando el mismo comando `COPY INTO` diariamente, **Snowflake registra en su metadata los archivos ya procesados y solo carga los nuevos**, ignorando automáticamente los anteriores.

### Ejemplo práctico completo con Named Stage:
```sql
-- 1. Crear la tabla destino
CREATE TABLE SALES_TRANSACTIONS (
    TRANSACTION_ID NUMBER, 
    CUSTOMER_ID NUMBER, 
    PRODUCT_ID NUMBER, 
    QUANTITY NUMBER, 
    AMOUNT NUMBER(10,2), 
    TRANSACTION_DATE DATE
);

-- 2. Crear un Stage nombrado
CREATE STAGE SALES;

-- 3. Subir el archivo local al stage
PUT file:///Users/alex/Projects/snowflake/sales_transactions.csv @sales;

-- 4. Copiar los datos del stage a la tabla
COPY INTO SALES_TRANSACTIONS 
FROM @sales 
FILE_FORMAT = (TYPE = CSV, SKIP_HEADER = 1);
```

---

## 5. Manejo de Errores en la Carga (`ON_ERROR`)

Por defecto, si Snowflake encuentra un error en una fila (por ejemplo, intentar insertar un valor de texto `STRING` en una columna de tipo entero `INT`), **todo el archivo falla y se cancela la carga**. Para cambiar este comportamiento, se utiliza el parámetro **`ON_ERROR`**:

### Escenario A: Continuar cargando y saltar solo las filas con errores
Permite que Snowflake cargue todas las filas válidas y omita únicamente las que contengan errores de formato o tipos de datos:
```sql
COPY INTO SALES 
FROM @sales_stage 
FILE_FORMAT = (
    TYPE = CSV 
    SKIP_HEADER = 1 
    ON_ERROR = 'CONTINUE'
);
```
* **Resultado:** Si de 1,000,000 de registros hay 5 con errores, Snowflake carga exitosamente 999,995 registros y solo rechaza los 5 defectuosos, permitiendo que la operación y los reportes de negocio continúen.

### Escenario B: Omitir (saltar) todo el archivo si se encuentra un error
Si por políticas de calidad de datos se requiere que el archivo completo sea rechazado ante la mínima inconsistencia:
```sql
COPY INTO SALES 
FROM @sales_stage 
FILE_FORMAT = (
    TYPE = CSV 
    SKIP_HEADER = 1 
    ON_ERROR = 'SKIP_FILE'
);
```
* **Resultado:** Si el archivo contiene tan solo un registro con error, Snowflake aborta la inserción del archivo completo y pasa al siguiente.
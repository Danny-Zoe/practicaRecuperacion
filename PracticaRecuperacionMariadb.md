## TECNOLOGICO NACIONAL DE MEXICO
## INSTITUTO TECNOLOGICO DE MILPA ALTA II
### **Práctica 1: Recuperación de una base de datos en MariaDB**
**Gonzalez Garcia Daniela Zoe**
No. Control: **241190095**
Ingenieria en Sistemas Computacionales
Docente: Ing. Diego Alberto Bollas Paredes
26/11/2026 

---

## Objetivo

Hacer un respaldo de una base de datos, simular un desastre (borrarla por error) y **recuperarla** desde la terminal usando solo MariaDB.

- **Parte A:** respaldo y restauración con `mariadb-dump`.
- **Parte B:** recuperar también los datos posteriores al respaldo usando el binlog.

## Requisitos

- MariaDB instalado y en ejecución.
- Una terminal con acceso a un usuario administrador de MariaDB (en los ejemplos, `root`).
- Una carpeta de trabajo:

```bash
mkdir practica1 && cd practica1
```

> En versiones anteriores a MariaDB 10.5 los comandos se llaman `mysql`, `mysqldump` y `mysqlbinlog`.

---

# PARTE A: Respaldo y restauración

## Paso 1. Crear la base de datos y los datos

Entrar al cliente:

```bash
mariadb -u root -p
```

Ejecutar:

```sql
CREATE DATABASE tienda;
USE tienda;

CREATE TABLE clientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(100)
);

INSERT INTO clientes (nombre, email) VALUES
('Ana Torres', 'ana@correo.com'),
('Luis Perez', 'luis@correo.com'),
('Maria Gomez', 'maria@correo.com'),
('Carlos Ruiz', 'carlos@correo.com'),
('Sofia Diaz', 'sofia@correo.com');

SELECT COUNT(*) FROM clientes;
```

**Resultado:**
![Imagen 1](Imagenes/Imagen%201.png)

Salir:

```sql
EXIT;
```

## Paso 2. Hacer el respaldo

Desde la terminal:

```bash
mariadb-dump -u root -p --databases tienda > tienda_backup.sql
```

Comprobar que el archivo se creó:

```bash
ls -lh tienda_backup.sql
```

**Resultado:**
![Imagen 2](Imagenes/Imagen%202.png)

## Paso 3. Simular el desastre

```bash
mariadb -u root -p
```

```sql
DROP DATABASE tienda;
SHOW DATABASES;
```

**Resultado:**

![Imagen 3](Imagenes/Imagen%203.png)

Salir con `EXIT;`

## Paso 4. Restaurar el respaldo

```bash
mariadb -u root -p < tienda_backup.sql
```

## Paso 5. Verificar la recuperación

```bash
mariadb -u root -p -e "SELECT * FROM tienda.clientes;"
```

**Resultado:**


**Parte A completada.**
![Imagen 4](Imagenes/Imagen%204.png)
---

# PARTE B: Recuperar datos posteriores al respaldo con el binlog

En la Parte A, todo lo que se hubiera guardado **después** del respaldo se habría perdido. El **binlog** registra todos los cambios y permite recuperarlos.

## Paso 1. Activar el binlog

Editar el archivo de configuración (en Debian/Ubuntu):

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

(En otras distribuciones puede ser `/etc/my.cnf.d/server.cnf`.)

Debajo de la sección `[mariadbd]` agregar:

```ini
log_bin = /var/lib/mysql/mariadb-bin
server_id = 1
```

**Resultado:**
![Imagen 5](Imagenes/Imagen%205.png)

Guardar y reiniciar:

```bash
sudo systemctl restart mariadb
```

Verificar:

```bash
mariadb -u root -p -e "SHOW VARIABLES LIKE 'log_bin';"
```

**Resultado:**
  ![Imagen 6](Imagenes/Imagen%206.png)


## Paso 2. Hacer un respaldo que guarde la posición del binlog

```bash
mariadb-dump -u root -p --databases tienda --flush-logs --master-data=2 > tienda_pitr.sql
```

Buscar la posición guardada:

```bash
grep "CHANGE MASTER" tienda_pitr.sql
```

**Resultado:**
![Imagen 7](Imagenes/Imagen%207.png) 


Anota el **archivo** y la **posición**.

## Paso 3. Agregar datos nuevos (posteriores al respaldo)

```bash
mariadb -u root -p
```

```sql
USE tienda;
INSERT INTO clientes (nombre, email) VALUES
('Pedro Nunez', 'pedro@correo.com'),
('Laura Vega', 'laura@correo.com');

SELECT COUNT(*) FROM clientes;
```

**Resultado:**
 ![Imagen 8](Imagenes/Imagen%208.png)


```sql
SELECT NOW();
```

Anotamos la hora: `2026-11-26 19:08:57 `
![Imagen 9](Imagenes/Imagen%209.png)

## Paso 4. Simular el error

Esperar otros 5 segundos y ejecutar:

```sql
DELETE FROM clientes;
SELECT COUNT(*) FROM clientes;
```

**Resultado:**
![Imagen 10](Imagenes/Imagen%2010.png)

Salir con `EXIT;`

## Paso 5. Restaurar el respaldo

```bash
mariadb -u root -p < tienda_pitr.sql
```

En este punto solo hay 5 clientes (faltan Pedro y Laura).

## Paso 6. Aplicar el binlog hasta antes del error

Usar la posición y el archivo que anotaste, y la hora anotada en el Paso 3:

```bash
sudo mariadb-binlog --start-position=330 --stop-datetime="2026-11-24 14:31:20" \
    /var/lib/mysql/mariadb-bin.000002 | mariadb -u root -p
```
**Resultado:**
![Imagen 11](Imagenes/Imagen%2011.png)
## Paso 7. Verificar

```bash
mariadb -u root -p -e "SELECT * FROM tienda.clientes;"
```

**Resultado:**
![Imagen 12](Imagenes/Imagen%2012.png)

**Parte B completada:** se recuperó todo excepto el `DELETE` accidental.

## Conclusion




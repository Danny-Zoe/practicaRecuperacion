# TECNOLOGICO NACIONAL DE MEXICO
## INSTITUTO TECNOLOGICO DE MILPA ALTA II
### **Cuestionario 1: Recuperación de una base de datos en MariaDB**
**Gonzalez Garcia Daniela Zoe**
No. Control: **241190095**
Ingenieria en Sistemas Computacionales
Docente: Ing. Diego Alberto Bollas Paredes
26/11/2026

## Responde las siguientes preguntas

1. ¿Qué datos se habrían perdido si solo hubieras usado el respaldo de la Parte A?
**Se habrían perdido los cambios realizados después del respaldo, como registros nuevos, modificaciones y eliminaciones de datos.**

2. ¿Para qué sirve la opción --master-data=2?
**Sirve para guardar en el respaldo la posición del Binary Log (archivo y número de posición), necesaria para identificar desde dónde continuar la recuperación de los cambios.**

3. ¿Por qué es necesario activar el binlog antes de que ocurra el error?
**Porque el Binary Log registra las operaciones realizadas en la base de datos, si no está activado antes del error, no será posible recuperar los cambios registrados durante ese tiempo.**

4. ¿Por qué se esperaron 5 segundos antes y después de anotar la hora?
**Para dejar un pequeño intervalo de tiempo que facilite identificar los cambios registrados en el Binary Log y ubicar con mayor precisión el momento del error.**
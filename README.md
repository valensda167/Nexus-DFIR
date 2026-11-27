# Nexus DFIR 0.1

¿Cómo comprobarías que una copia conserva exactamente el contenido del archivo original? Dos documentos pueden tener el mismo nombre y tamaño aunque una cifra haya cambiado. Nexus DFIR permite seleccionar un archivo local, calcular su huella SHA-256 y compararla con una referencia. Es una aplicación educativa de escritorio escrita en Java Swing.

## Fundamento teórico

**Integridad digital.** En este proyecto interesa saber si los bytes de un archivo coinciden con los de una versión de referencia. En informática forense y respuesta a incidentes (DFIR, por sus siglas en inglés), esta comprobación ayuda a verificar las copias que se examinan. Una investigación completa también documenta de dónde salió la evidencia, quién la manejó y cómo se conservó. [Guía forense del NIST, SP 800-86](https://csrc.nist.gov/pubs/sp/800/86/final)

**Huella SHA-256.** Una función hash transforma una secuencia de bytes en un resumen. SHA-256 produce 256 bits, representados por 64 caracteres hexadecimales. Los mismos bytes siempre producen la misma huella. Un cambio tan pequeño como añadir un salto de línea suele modificar muchos caracteres del resultado. Encontrar dos contenidos distintos con la misma huella es computacionalmente inviable en la práctica. [Estándar de funciones hash del NIST, FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)

**Contenido y metadatos.** Nexus muestra nombre, ruta, tamaño y fechas para identificar el archivo examinado. Calcula la huella leyendo los bytes del contenido. Por eso puedes renombrar un archivo y conservar su SHA-256, o sustituir una letra por otra y obtener una huella distinta aunque el tamaño siga igual.

## Cómo interpretar la pantalla

| Qué haces | Qué puedes concluir |
| --- | --- |
| Comparas Nexus con PowerShell sobre el mismo archivo actual | Una coincidencia confirma que ambas herramientas obtuvieron el mismo resultado en esas lecturas. |
| Comparas una copia con una huella que guardaste del original | **COINCIDE** aporta evidencia de que la copia conserva los mismos bytes, siempre que la referencia se haya conservado correctamente. |
| Obtienes **NO COINCIDE** | Las huellas difieren. Comprueba que elegiste la versión correcta y copiaste la referencia completa. Para localizar qué cambió necesitarías comparar ambos archivos. |

La confianza depende del origen de la referencia. Si una persona puede sustituir tanto el archivo como la huella publicada, ambos valores pueden coincidir. Para comprobar una descarga, conviene obtener la huella por un canal confiable del editor. Determinar quién creó un archivo o si contiene código dañino requiere otras comprobaciones.

## Qué hace el programa al seleccionar un archivo

1. Lee los atributos del sistema de archivos antes de empezar.
2. Abre el archivo para lectura y calcula SHA-256 por bloques de hasta 8192 bytes. Así puede procesar archivos grandes con una cantidad pequeña de memoria.
3. Vuelve a consultar el tamaño y la fecha de modificación. Si detecta una diferencia, interrumpe el análisis. Esta comprobación ayuda a detectar cambios durante la lectura, aunque un cambio que conserve ambos atributos podría pasar inadvertido.
4. Muestra los metadatos y el hash calculado. Al pulsar **Verificar SHA-256**, compara el resultado guardado con los 64 caracteres hexadecimales introducidos. Acepta letras mayúsculas o minúsculas. Si editas el archivo, selecciónalo de nuevo para actualizar el cálculo.

## Abrir en NetBeans
1. Descomprimir el ZIP.
2. File > Open Project y seleccionar la carpeta Nexus-DFIR que contiene pom.xml.
3. Seleccionar JDK 17 o 21. Maven puede descargar complementos de compilación en el primer inicio.
4. Run Project (F6). Si pide clase principal, elegir nexusdfir.NexusDFIR.

## Probar
Ejecuta `mvn test` desde la carpeta que contiene `pom.xml` o usa **Test** en el menú contextual del proyecto en NetBeans.

### Práctica con un cambio de un byte

1. En PowerShell, crea un archivo temporal de tres bytes. La última línea muestra su ruta.

   ```powershell
   $muestra = New-TemporaryFile
   [System.IO.File]::WriteAllBytes($muestra.FullName, [byte[]](97, 98, 99))
   $muestra.FullName
   ```

2. En Nexus, pulsa **Seleccionar archivo** y abre esa ruta. Los bytes corresponden al texto `abc`. Pega la huella de referencia en **SHA-256 esperado** y pulsa **Verificar SHA-256**. El resultado esperado es **COINCIDE**.

   ```text
   ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
   ```

3. En la misma consola, cambia la última letra de `abc` a `abd`.

   ```powershell
   [System.IO.File]::WriteAllBytes($muestra.FullName, [byte[]](97, 98, 100))
   ```

4. Vuelve a seleccionar el archivo, pega la **misma referencia de `abc`** y verifica. El tamaño sigue siendo de tres bytes. El resultado esperado es **NO COINCIDE** porque cambió el contenido.

## Estructura
- ArchivoAnalizado: modelo con atributos y getters.
- AnalizadorForense: metadatos básicos, SHA-256 y comparación.
- NexusDFIR: interfaz gráfica Swing y punto de entrada.
- AnalizadorForenseTest: pruebas automatizadas con JUnit 5.

## Alcance
El programa solo abre el archivo para leer su contenido y metadatos del sistema de archivos; no extrae EXIF ni otros metadatos internos. Leer un archivo puede actualizar su fecha de acceso según el sistema. La fecha de creación depende del sistema de archivos. La comparación corresponde al contenido leído en el momento del análisis, no a cambios posteriores; vuelve a analizar para verificar de nuevo. La coincidencia de un hash no garantiza que el archivo sea seguro o auténtico sin una referencia confiable. Este prototipo no es una herramienta de adquisición forense certificada ni reemplaza una cadena de custodia. Trabaja con archivos propios o autorizados, preferentemente copias.

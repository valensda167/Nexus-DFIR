# Nexus DFIR 0.1

Prototipo educativo de análisis de archivos locales en Java Swing. No requiere dependencias externas en tiempo de ejecución.

## Abrir en NetBeans
1. Descomprimir el ZIP.
2. File > Open Project y seleccionar la carpeta Nexus-DFIR que contiene pom.xml.
3. Seleccionar JDK 17 o 21. Maven puede descargar complementos de compilación en el primer inicio.
4. Run Project (F6). Si pide clase principal, elegir nexusdfir.NexusDFIR.

## Probar
Ejecuta `mvn test` desde la carpeta que contiene `pom.xml` o usa **Test** en el menú contextual del proyecto en NetBeans.

Crear un archivo de texto con exactamente `abc` (sin salto de línea y en UTF-8 sin BOM), seleccionarlo, y comparar contra:
`ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad`
También puedes probar con cualquier copia de un archivo propio. Cambia el contenido de la copia, analízala de nuevo y compara con el hash anterior.

## Estructura
- ArchivoAnalizado: modelo con atributos y getters.
- AnalizadorForense: metadatos básicos, SHA-256 y comparación.
- NexusDFIR: interfaz gráfica Swing y punto de entrada.
- AnalizadorForenseTest: pruebas automatizadas con JUnit 5.

## Alcance
El programa solo abre el archivo para leer su contenido y metadatos del sistema de archivos; no extrae EXIF ni otros metadatos internos. Leer un archivo puede actualizar su fecha de acceso según el sistema. La fecha de creación depende del sistema de archivos. La comparación corresponde al contenido leído en el momento del análisis, no a cambios posteriores; vuelve a analizar para verificar de nuevo. La coincidencia de un hash no garantiza que el archivo sea seguro o auténtico sin una referencia confiable. Este prototipo no es una herramienta de adquisición forense certificada ni reemplaza una cadena de custodia. Trabaja con archivos propios o autorizados, preferentemente copias.

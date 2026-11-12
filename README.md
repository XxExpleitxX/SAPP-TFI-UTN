# SAPP — Sistema de Automatización de Procesamiento de Pedidos

Material de reproducibilidad del **trabajo final de carrera** «SAPP — Sistema de Automatización de Procesamiento de Pedidos».

**Autores:** Juan Darío Sinatra · Matías Martínez · Emanuel Ortíz
**Carrera:** Ingeniería en Informática
**Institución:** Universidad Tecnológica Nacional — Facultad Regional Mendoza
**Año:** 2026

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22148608.svg)](https://doi.org/10.5281/zenodo.22148608)

---

## Qué es este repositorio

Contiene los artefactos necesarios para reproducir las dos rondas de validación reportadas en el Capítulo 5 del trabajo: los flujos de automatización exportados, el corpus de documentos de prueba, el protocolo con sus resultados, la línea de base del proceso manual, la verificación independiente y las evidencias de ejecución.

El corpus está compuesto por veinte documentos comerciales **generados de forma sintética** mediante un script en Python, conforme a una especificación que fijó diez ítems por documento, rubros diversos y datos de cliente ficticios. No se trata de pedidos de organizaciones reales.

## Por qué hay dos flujos publicados

En `Workflow/` conviven dos archivos y la distinción es deliberada.

`SAPP_workflow_v1_medido.json` es el flujo **sin corregir**, en el estado exacto sobre el cual se obtuvieron los resultados de la ronda ampliada de agosto de 2026. Se publica así para que esos resultados sean verificables: contiene los cinco defectos que el trabajo documenta. **No debe tomarse como versión de referencia para su uso.**

`SAPP_workflow_v2.json` es el flujo **corregido**, con las siete correcciones aplicadas y verificadas en la segunda ronda de octubre de 2026. Es la versión que corresponde usar como punto de partida.

La diferencia entre ambos archivos es, en sí misma, evidencia: muestra qué se cambió y qué no.

## Contenido

| Carpeta | Contenido |
|---|---|
| `Workflow/` | Los dos flujos de n8n exportados en formato JSON |
| `Corpus/` | Los 20 documentos del corpus, más la versión sin capa de texto empleada en el caso CP-07 |
| `Validacion/` | Protocolo con ambas rondas y verificación independiente del evaluador externo |
| `Linea_de_base/` | Planillas de transcripción manual y grabación de pantalla de uno de los casos |
| `Evidencias/` | Capturas del entorno de ejecución referenciadas en el Anexo C |
| `Diagramas/` | Diagrama de componentes, modelo de datos y diagrama de flujo |

## El sistema

Flujo que recibe pedidos comerciales por correo electrónico, interpreta su contenido mediante un modelo de lenguaje y persiste los datos extraídos en planillas de cálculo.

**Cuatro capas:** ingesta y clasificación · procesamiento en dos pipelines paralelos (A para PDF adjunto, B para texto en el cuerpo) · persistencia · notificación.

**Entorno:** n8n 2.10.3 autoalojado en Docker · Gmail, Google Drive y Google Sheets vía OAuth2.

**Modelo de lenguaje:** la ronda ampliada se ejecutó sobre Mistral Cloud, con `mistral-medium` en el Pipeline A y el valor predeterminado del nodo en el Pipeline B. La segunda ronda se ejecutó sobre Google Gemini, con `gemini-3.6-flash` y temperatura cero en ambos pipelines. El motivo del cambio se explica más abajo.

## Cómo reproducir la validación

1. Importar en una instancia de n8n 2.10.3 el flujo que corresponda: `SAPP_workflow_v1_medido.json` para reproducir la ronda ampliada, `SAPP_workflow_v2.json` para partir de la versión corregida.
2. Configurar las credenciales localmente. **El archivo exportado no contiene secretos**: n8n conserva solo referencias, que deben recrearse en la instancia propia.
3. Ajustar los identificadores de carpeta de Drive. En el flujo v1 figuran en `Upload file`, `Move file`, `Move file1`, `Search files and folders` y `Search files and folders1`; en el v2 los dos nodos de búsqueda ya no existen.
4. Configurar la credencial del proveedor de modelo y verificar que el identificador declarado siga disponible en su catálogo. Ver la advertencia sobre disponibilidad más abajo.
5. Consultar la hoja **Corpus y valores esperados** para saber qué documento corresponde a cada caso.
6. Enviar los documentos de `Corpus/` uno por vez, según la hoja **Instrucciones**.
7. Registrar cada corrida en la hoja **Registro de ejecuciones**, indicando la versión del flujo en la columna correspondiente.

> Los valores esperados deben definirse **antes** de procesar cada documento. Definirlos después de observar la salida invalida la medición.

### Advertencia sobre disponibilidad

La reproducción exacta de la ronda ampliada **no es posible**. El identificador de modelo con el que operó el Pipeline B dejó de figurar en el catálogo del proveedor y la interfaz de ese proveedor dejó de responder a las llamadas de generación. La configuración medida resultó irrecuperable a menos de dos meses de las mediciones.

Es una limitación de fondo y no un contratiempo administrativo: la reproducibilidad de un sistema cuyo componente central es un servicio de terceros depende de condiciones que quien publica no controla.

## Resultados

### Ronda ampliada — agosto de 2026

32 casos computados, ejecutados en cuatro jornadas (12, 14, 17 y 27 de agosto) con la misma versión del flujo: 14 por el Pipeline A y 18 por el Pipeline B.

| Métrica | Pipeline A | Pipeline B | Total |
|---|---|---|---|
| Casos | 14 | 18 | 32 |
| Sin defecto detectado | 11 | 12 | 23 (71,9 %) |
| Precisión extremo a extremo | 66,7 % | 94,0 % | 77,7 % |
| Precisión sobre campos persistidos | 100,0 % | 94,0 % | 97,0 % |

Tiempo de procesamiento: mediana 20,9 s, rango 14,5 – 36,0 s.

Se computa un caso con defecto cuando presenta discrepancia en al menos un campo verificado, o cuando el sistema exhibió un comportamiento anómalo documentado, aun habiendo persistido correctamente los campos que escribió. Los nueve casos con defecto son CP-07, CP-08, CP-10, CP-13, CP-17, CP-18, CP-19, CP-28 y CP-31.

Se reportan dos métricas de precisión porque miden cosas distintas: la primera toma como denominador los campos que el esquema declara extraer; la segunda, solo los que el flujo escribe en la planilla. La diferencia corresponde a un defecto de la capa de persistencia.

**El sistema no derivó por sí mismo ningún caso a revisión humana.** El 71,9 % mide los casos procesados sin defecto detectado por una verificación externa, no la capacidad del sistema para discriminar los casos que requieren intervención. Esa carencia es uno de los hallazgos del trabajo.

### Segunda ronda — octubre de 2026

Diez casos, seleccionados y declarados antes de ejecutar: los ocho que habían manifestado defecto más dos controles de extracción.

**No es una nueva estimación de prevalencia** y no es comparable con la ronda anterior en términos agregados: la muestra está deliberadamente enriquecida en casos defectuosos. Los resultados se reportan defecto por defecto. Las métricas de ambas rondas se registran por separado en el protocolo y no se agregan.

| Defecto | Estado |
|---|---|
| 1 — confirmación ante correos que no son pedidos | **cerrado**, verificado en CP-17 y CP-18 |
| 2 — unidad extraída y no persistida | **cerrado**, CP-01 y CP-05 pasan de 20/30 a 30/30 |
| 3 — extracción no reproducible | **no verificable** en las condiciones originales |
| 4 — planilla de destino no identificada de forma unívoca | **cerrado**, verificado en los diez casos |
| 5 — sin correspondencia verificable | **mitigado parcialmente**, solo en el Pipeline B |

### Línea de base manual

Diez documentos transcritos manualmente por un integrante del equipo que no participó de la ronda automatizada. Cada uno tiene su contraparte procesada por el sistema, de modo que la comparación es **pareada sobre documentos idénticos**.

| Métrica | Manual | Sistema |
|---|---|---|
| Tiempo medio | 124,3 s | 20,4 s |
| Mediana | 121,5 s | 20,6 s |
| Coeficiente de variación | 12,9 % | 5,6 % |

Reducción media: **83,4 %** sobre el tiempo de procesamiento del flujo (rango por documento: 81,1 % – 86,2 %). Factor de aceleración: 6,1.

Computando además el intervalo de consulta del disparador, la reducción extremo a extremo cae al 35,3 %. El contraste de H1 se sostiene sobre el tiempo de procesamiento y no sobre la latencia extremo a extremo; el intervalo de consulta es un parámetro de configuración y no una propiedad del diseño.

### Evaluación independiente

Ocho casos (25 % de los computados) verificados a ciegas, sin acceso a los valores esperados ni al resultado de ninguna verificación previa. Se realizaron tres evaluaciones: la primera verificación, una segunda por un integrante del equipo, y una réplica por una evaluadora externa, hermana de uno de los autores, sin participación en el trabajo ni vínculo con la institución.

| Par de evaluadores | Acuerdo observado | Kappa de Cohen |
|---|---|---|
| Primera verificación – evaluador del equipo | 93,8 % | 0,64 |
| Primera verificación – evaluadora externa | 88,8 % | 0,38 |
| Evaluador del equipo – evaluadora externa | 95,0 % | 0,78 |

La concordancia con la evaluadora externa resulta **inferior** a la obtenida entre los dos evaluadores del equipo. El dato se publica porque cuantifica la limitación que el propio trabajo declaraba: la concordancia entre evaluadores que comparten el interés en el resultado sobrestima la fiabilidad de la verificación.

El contraste más informativo es el tercero. Los dos evaluadores que no definieron el criterio de cómputo concuerdan más entre sí que cualquiera de ellos con la primera verificación, porque ambos computaron como error la ausencia de mayúscula inicial, que el criterio declarado excluye de forma expresa. La exclusión de la capitalización no resulta evidente para quien examina el material sin conocer la decisión.

### Contraste de hipótesis

**H1 — reducción del tiempo de al menos 50 %: se sostiene.** Los diez pares superan el umbral, sin excepciones.

**H2 — reducción de la tasa de error de al menos 2 puntos: se sostiene con margen estrecho.** Bajo el criterio declarado antes de medir, el proceso manual registró 12 discrepancias sobre 100 artículos y el sistema 8.

La conclusión es sensible al tratamiento de las tildes: excluyéndolas, el manual registraría 6 y el sistema 8, y H2 no se sostendría. Se adopta el criterio declarado a priori por razones de procedimiento; ambos cómputos se reportan. **Las seis discrepancias que separan un cómputo del otro pertenecen todas al proceso manual.**

## Defectos identificados

Cinco defectos, todos en la capa de orquestación y persistencia.

| # | Defecto | Componente |
|---|---|---|
| 1 | Emite confirmación ante correos que no son pedidos | `Code in JavaScript1` |
| 2 | El campo de unidad se extrae y no se persiste | `Append row in sheet1` |
| 3 | Extracción no reproducible entre ejecuciones | nodos de modelo, sin temperatura fijada |
| 4 | La escritura no identifica de forma unívoca la planilla de destino | `Search files and folders1` |
| 5 | Sin correspondencia verificable entre lo procesado y lo comunicado | transversal |

El quinto agrupa siete casos con el mismo comportamiento: el sistema produce un resultado incorrecto, incompleto o duplicado y prosigue sin señal de anomalía.

En sentido favorable, el modelo **no fabrica los campos ausentes** ni inventa cantidades cuando el documento no las especifica. Se verificó en la segunda ronda incluso ante un documento sin texto extraíble.

### Hallazgos de la segunda ronda

La aplicación de las correcciones produjo cuatro resultados que la especificación no anticipaba.

Dos de ellos dieron lugar a correcciones adicionales: el nodo de corte no evitaba la creación de la planilla de destino, y la rama de rechazo no retiraba la etiqueta del mensaje. Un tercero fue un defecto preexistente que la reestructuración hizo manifiesto.

El cuarto es el más relevante: **el clasificador binario incorporado para resolver el falso positivo introduce el modo de falla inverso**. Un pedido legítimo sin cantidades numéricas se descarta por completo en lugar de registrarse como incompleto. La corrección resuelve un extremo y abre el otro.

## Limitaciones

- La línea de base se cronometró con un único operador, integrante del equipo, que además elaboró el script generador del corpus.
- El corpus es sintético y de estructura homogénea: mide el desempeño del sistema ante una estructura conocida, no ante la variabilidad de documentos reales.
- El contraste de hipótesis no emplea pruebas de inferencia estadística. La comparación es pareada sobre diez documentos y describe el conjunto evaluado; la generalización a otras poblaciones de pedidos requiere replicación en contextos organizacionales diversos.
- Bajo la configuración de credenciales evaluada (proyecto en modo de prueba), el token expira a los siete días y exige reautorización manual. La publicación de la aplicación, especificada como corrección, resultó inaplicable por exigir un proceso de verificación que los permisos requeridos vuelven obligatorio.
- La segunda ronda se ejecutó sobre un proveedor de modelo distinto del original, de modo que la precisión de extracción de ambas rondas no es comparable.

## Nota sobre los datos

Los documentos del corpus fueron generados con fines de evaluación. Nombres de clientes, domicilios y datos comerciales son ficticios y no corresponden a personas ni organizaciones reales.

## Cómo citar

    Sinatra, J. D., Martínez, M. y Ortíz, E. (2026). SAPP — Sistema de
    Automatización de Procesamiento de Pedidos [Trabajo final de carrera].
    Universidad Tecnológica Nacional, Facultad Regional Mendoza.
    https://doi.org/10.5281/zenodo.22148608

## Licencia

- **Código y flujo de automatización:** MIT
- **Corpus y documentación:** CC BY 4.0

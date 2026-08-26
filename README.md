# SAPP — Sistema de Automatización de Procesamiento de Pedidos

Material de reproducibilidad del trabajo final de carrera **«SAPP — Sistema de Automatización de Procesamiento de Pedidos»**.

**Autores:** Juan Darío Sinatra · Matías Martínez · Emanuel Ortíz  
**Carrera:** Ingeniería en Informática  
**Institución:** Universidad Tecnológica Nacional — Facultad Regional Mendoza  
**Año:** 2026

---

## Qué es este repositorio

Contiene los artefactos necesarios para reproducir la validación reportada en el Capítulo 5 de la tesis: el flujo de automatización exportado, el corpus de documentos de prueba, el protocolo de validación con sus resultados, la línea de base del proceso manual y las evidencias de ejecución.

Se publica en cumplimiento del criterio de reproducibilidad: toda afirmación empírica del trabajo debe poder verificarse contra el material que la sustenta.

## Contenido

| Carpeta | Contenido |
|---|---|
| `Workflow/` | Flujo de n8n exportado en formato JSON |
| `Corpus/` | Los 20 documentos comerciales que conforman el corpus de evaluación |
| `Validacion/` | Protocolo de validación: corpus con valores esperados, registro de las 28 ejecuciones, línea de base manual y métricas |
| `Linea_de_base/` | Planillas de transcripción manual y registros de cronometraje |
| `Evidencias/` | Capturas del entorno de ejecución referenciadas en el Anexo C |
| `Diagramas/` | Diagrama de componentes y modelo de datos en resolución completa |

## El sistema

Flujo de automatización que recibe pedidos comerciales por correo electrónico, interpreta su contenido mediante un modelo de lenguaje y persiste los datos extraídos en planillas de cálculo.

**Arquitectura en cuatro capas:**

1. **Ingesta** — recepción del correo y clasificación según exista o no un documento adjunto
2. **Procesamiento inteligente** — dos subsistemas paralelos: Pipeline A para documentos PDF adjuntos, Pipeline B para pedidos redactados en el cuerpo del mensaje
3. **Persistencia** — creación de la planilla de destino y escritura de los ítems extraídos
4. **Salida** — notificación de confirmación al remitente

**Entorno de ejecución:**

- n8n 2.10.3 autoalojado en Docker
- Modelo de lenguaje: Mistral Cloud (`mistral-medium`)
- Servicios externos: Gmail, Google Drive y Google Sheets, vía OAuth2

## Cómo reproducir la validación

1. Importar el flujo de `Workflow/` en una instancia de n8n 2.10.3.
2. Configurar las credenciales localmente. **El archivo exportado no contiene secretos**: n8n solo conserva referencias a credenciales, que deben recrearse en la instancia propia. Se requieren credenciales OAuth2 de Gmail, Google Drive y Google Sheets, y una clave de API de Mistral Cloud.
3. Ajustar los identificadores de las carpetas de Drive en los nodos `Upload file`, `Move file`, `Move file1`, `Search files and folders` y `Search files and folders1`.
4. Consultar la hoja **Corpus y valores esperados** del protocolo para conocer qué documento corresponde a cada caso.
5. Enviar los documentos de `Corpus/` uno por vez, según el procedimiento descrito en la hoja **Instrucciones**.
6. Registrar cada corrida en la hoja **Registro de ejecuciones**.

> Los valores esperados deben definirse **antes** de procesar cada documento. Definirlos después de observar la salida del sistema invalida la medición.

## Resultados reportados

### Ronda automatizada — 17 de agosto de 2026

| Métrica | Valor |
|---|---|
| Casos computados | 27 |
| Casos sin escalamiento | 23 |
| Tasa de procesamiento autónomo | 85,2 % |
| Precisión de extracción | 78,8 % |
| Precisión sobre campos persistidos | 97,3 % |
| Tiempo de procesamiento (mediana) | 20,9 s |
| Tiempo de procesamiento (rango) | 15,9 – 36,0 s |

Se reportan dos métricas de precisión porque miden cosas distintas: la primera toma como denominador los campos que el esquema declara extraer, y la segunda únicamente los que el flujo escribe efectivamente en la planilla. La diferencia entre ambas corresponde a un defecto de la capa de persistencia documentado en la tesis.

### Línea de base manual — 21 de agosto de 2026

Diez documentos del corpus fueron transcritos manualmente por un integrante del equipo que no participó de la ronda automatizada ni conocía los documentos. Cada uno cuenta con su contraparte procesada por el sistema, de modo que la comparación es **pareada sobre documentos idénticos**.

| Métrica | Manual | Sistema |
|---|---|---|
| Tiempo medio | 124,3 s | 20,4 s |
| Mediana | 121,5 s | 20,6 s |
| Desvío estándar | 16,1 s | 1,1 s |
| Coeficiente de variación | 12,9 % | 5,5 % |

Reducción media del tiempo de procesamiento: **83,4 %** (rango por documento: 81,1 % – 86,2 %). Factor de aceleración: 6,1.

### Contraste de hipótesis

**H1 — reducción del tiempo en al menos un 50 %: se sostiene.** Los diez pares superan el umbral, sin excepciones.

**H2 — reducción de la tasa de error en al menos 2 puntos porcentuales: no se sostiene.** Sobre 100 artículos de cada proceso, el manual registró 6 discrepancias de contenido y el sistema 8. La diferencia opera en favor del proceso manual.

El resultado admite una lectura cualitativa que la comparación agregada oscurece: el operador humano no cometió ningún error de cantidad pero sí seis errores ortográficos; el sistema no omitió una sola tilde pero registró dos errores de cantidad. Los errores difieren en naturaleza y en severidad operativa.

## Defectos identificados

La validación localizó cuatro defectos, todos en la capa de orquestación y persistencia. **Ninguno fue corregido en la versión aquí publicada**: el flujo se entrega en el estado sobre el cual se realizaron las mediciones, para que los resultados sean verificables.

| # | Defecto | Componente |
|---|---|---|
| 1 | Emite confirmación de pedido ante correos que no lo son | `Code in JavaScript1` |
| 2 | El campo de unidad se extrae y no se persiste | `Append row in sheet1` |
| 3 | Extracción no reproducible entre ejecuciones | nodos de modelo, sin temperatura fijada |
| 4 | La escritura no identifica de forma unívoca la planilla de destino | `Search files and folders1` |

En sentido favorable, se verificó que el modelo **no fabrica los campos ausentes** en el documento de origen: ante un campo requerido por el esquema pero no consignado en el pedido, devuelve el valor vacío en lugar de generar un dato plausible.

## Limitaciones de esta validación

- La verificación campo a campo fue realizada por un único evaluador, integrante del equipo, que además definió los valores esperados. No se dispone de una estimación de concordancia entre evaluadores.
- La línea de base se cronometró con un único operador de referencia, integrante del equipo y no un administrativo con experiencia en carga de pedidos. No se estimó la variabilidad entre personas.
- Cinco casos del corpus diseñado no fueron ejecutados por requerir condiciones de preparación no disponibles.
- El corpus es de estructura homogénea y fue elaborado por el equipo; no se evaluó el comportamiento ante documentos de proveedores reales con maquetación heterogénea.
- No se aplicaron pruebas de inferencia estadística: el corpus no constituye una muestra aleatoria de una población de pedidos comerciales.

## Nota sobre los datos

El corpus está compuesto por documentos elaborados por el equipo de investigación con fines de evaluación. Los nombres de clientes, domicilios y datos comerciales son ficticios y no corresponden a personas ni organizaciones reales.

## Licencia

- **Código y flujo de automatización:** MIT
- **Corpus y documentación:** CC BY 4.0

Se permite su uso con fines educativos y de investigación citando la fuente.

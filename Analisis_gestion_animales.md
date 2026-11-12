# Tarea 1 — Análisis formal del proyecto

**Programa ejecutable:** `app.py` (`datos/`, `validaciones/`, `interfaz/`, `exportacion/`)  
**Documentación formal:** carpeta [`docs/`](docs/README.md) (requisitos RF/RNF, CU, UML, manuales, pruebas)  
**Guías y cuadernos:** este documento + notebooks actualizados en la raíz del proyecto (`gestion_animales_tkinter_V5_actualizado.ipynb`, `arquitectura_refugio_animal_v5.ipynb`)  
**Enunciado oficial:** `Enunciado_oficial_Tarea_1.pdf`  
**Fecha de entrega (enunciado):** 12 de noviembre de 2026  

Este documento presenta un análisis formal del sistema desarrollado, describiendo su alcance funcional, diseño de componentes, relación con el enunciado, uso del temario de la asignatura y criterios de mantenimiento. La versión revisada refleja el estado actual del repositorio y no se limita a una interpretación preliminar del proyecto.

**Arranque de la aplicación:** `cd tarea1` → `python app.py` (o `python main.py`)  
**Extras respecto al enunciado mínimo:** ficha clínica, foto/documentos (importar/pegar, máx. 5 MB), exportación PDF, ficha POO en la GUI, **especies personalizadas** (JSON + `EspeciePersonalizada`), **persistencia del inventario** (`inventario.json`) y tests con informe en disco.

**Cuaderno ≠ programa.** El notebook `.ipynb` cumple un propósito didáctico (CRUD + ficha POO con scrollbars + demo de `EspeciePersonalizada`). La aplicación de uso real y completa, con foto, clínica, PDF, *Gestionar especies* y estilos, se ejecuta desde `tarea1/app.py`. El repositorio actual incluye versiones V5 actualizadas y la diferencia entre cuaderno y aplicación queda documentada en [README.md](README.md).

**Estado verificado:** la suite de pruebas del proyecto se ejecuta correctamente con `python -m unittest discover -s tests -v` y finaliza con resultado satisfactorio (112 tests, OK).

---

## Resumen ejecutivo

La aplicación desarrollada consiste en un gestor de animales de refugio con interfaz gráfica en Tkinter, basado en un inventario de tipo diccionario persistido en JSON. El sistema permite crear, consultar, actualizar y eliminar animales, así como gestionar información clínica, adjuntos físicos, imágenes y exportación de fichas en formato PDF.

Además, el proyecto incorpora una capa de programación orientada a objetos para representar el comportamiento específico de cada especie, manteniendo la estructura del dominio en diccionarios, tal y como exige el enunciado. Esta separación entre modelo de datos, validación y presentación permite un diseño modular, reutilizable y fácilmente extensible.

En términos de arquitectura, el proyecto se organiza en capas funcionales: datos, validación, interfaz gráfica y exportación. La capa de persistencia guarda el inventario principal y el catálogo de especies personalizadas en archivos JSON independientes, evitando mezclar dos tipos de información distintos pero relacionados.

## Objetivos del proyecto

El sistema tiene como objetivo principal proporcionar una herramienta de gestión de animales en un refugio, con capacidad para:

- registrar animales con datos básicos y clínicos;
- mantener la información actualizada mediante operaciones CRUD;
- visualizar el inventario en una interfaz gráfica clara y operativa;
- gestionar especies nativas y personalizadas sin reescribir código Python;
- conservar la información entre ejecuciones mediante persistencia JSON;
- exportar fichas de animales en formato PDF para su uso documental;
- servir como demostración del uso de POO y de interfaces gráficas en la asignatura.

## Alcance funcional y arquitectura general

El proyecto implementa una arquitectura por capas, con separación clara entre lógica de dominio, validación, interfaz de usuario y exportación de datos. La capa central está formada por el inventario de animales y las funciones de acceso al mismo; la capa de validación protege la consistencia de los registros; la capa de interfaz presenta la información al usuario y recoge sus entradas; la capa de exportación genera informes que pueden usarse como documentación del animal.

La arquitectura refleja una buena práctica docente: la lógica del CRUD no depende de la GUI, lo que facilita la comprobación automática, la reutilización y la extensibilidad del sistema.

## 0. Estado del proyecto y validación

Se ha verificado el estado actual del repositorio mediante ejecución directa de la batería de pruebas del proyecto. La comprobación realizada con la herramienta estándar de Python confirma que el sistema funciona de manera consistente en la versión actual.

- Comando ejecutado: `python -m unittest discover -s tests -v`
- Resultado: 112 pruebas ejecutadas, todas correctas.

Este dato es relevante porque permite contrastar el análisis documental con la implementación efectiva, evitando que el documento quede desalineado respecto al código real del proyecto.

---

## 1. Relación con el enunciado

| Requisito del enunciado | Cómo se cumple en el proyecto |
| --- | --- |
| Temática libre | Refugio / inventario de animales (Perro, Gato, Conejo, Loro, Hámster) |
| Al menos un diccionario complejo | `animales`: ID → ficha con varios campos |
| Funcionalidades CRUD | Crear, leer, actualizar y borrar |
| Interfaz con `tkinter` | Clase `GestorAnimales` + widgets `tk` / `ttk` |
| Presentación con posibles cambios | Ver sección 8 (preguntas de refactorización) |

**Dato clave:** el CRUD trabaja con **diccionarios** (como pide el enunciado). No hay base de datos relacional. Además, el inventario se **persiste** en `datos/inventario.json` y el catálogo de especies personalizadas en `datos/especies_personalizadas.json` (archivos separados).

---

## 2. Idea general del proyecto

Aplicación de escritorio CRUD sobre un inventario de animales, ampliada con ficha clínica, adjuntos y PDF.

| Operación | Método de lógica | Método de interfaz |
| --- | --- | --- |
| Create | `crear_animal()` | `_crear()` |
| Read | `leer_animal()` | `_seleccionar()` / `_actualizar_tabla()` |
| Update | `actualizar_animal()` | `_actualizar()` |
| Delete | `borrar_animal()` | `_borrar()` |
| Clínica / adjuntos | `adjuntos.py` + listas en el `dict` | `DialogoFichaClinica` |
| Exportar | `exportar_fichas_pdf()` | `_exportar_pdf()` |

Resumen en una frase:

> Es una aplicación CRUD con Tkinter sobre diccionarios persistidos en JSON: separamos datos, validación e interfaz; además hay ficha clínica, especies dinámicas, pegado de adjuntos (límite 5 MB) y exportación a PDF.

---

## 3. Mapa con el temario (Tema 2 y Tema 3)

### 3.1 Tema 2 — POO y estructuras

El enunciado exige **diccionario + CRUD + Tkinter**. La ficha vive en un `dict`; además hay una **capa POO** paralela para demostrar el Tema 2 sin romper el enunciado.

| Idea del Tema 2 | En este proyecto |
| --- | --- |
| Diccionarios | Inventario `dict[int, dict]`; ID = identidad |
| Herencia + ABC | `AnimalRefugio` (abstracta) → `Perro`, `Gato`, `Conejo`, `Loro`, `Hamster` en `datos/modelos.py` |
| Getters / setters | `@property` + `@xxx.setter` (nombre, edad, peso, color, observaciones) |
| Decoradores | `@abstractmethod`, `@property`, `@classmethod` |
| Polimorfismo | `hacer_sonido()` / `tipo_alimentacion()` / `cuidados_especificos()`; la GUI muestra `generar_ficha()` sin `if` por especie |
| Puente dict ↔ objeto | `datos/adaptador_poo.py` (`registro_a_animal`, `animal_a_campos`) |
| Ficha en Tkinter | Panel «Información del animal seleccionado»: dict → adaptador → objeto → `generar_ficha()`; scrollbars en comportamiento / alimentación / cuidados |
| Especies dinámicas | Nativas (clases fijas) + personalizadas (`EspeciePersonalizada` + `especies_personalizadas.json`) vía *Gestionar especies* |
| Validación en dos vías | `validar_datos()` protege el CRUD (`dict`); los setters de `AnimalRefugio` encapsulan el objeto POO (detalle en la guía didáctica §3.7) |

**Idea clave:** *«El CRUD trabaja con diccionarios, como pide el enunciado; la herencia y las properties son una capa didáctica conectada por un adaptador.»*

### 3.2 Tema 3 — Interfaces gráficas aplicadas

| Lección Tema 3 | Uso en el cuaderno |
| --- | --- |
| Gestores de geometría | `pack()` en bloques principales; `grid()` dentro de `LabelFrame` |
| Variables de control | `StringVar` enlazadas a `Entry` / `Combobox` |
| Frame / LabelFrame | Formulario, filtros y zona de tabla |
| Button / Entry | Acciones CRUD y campos de texto |
| Eventos genéricos | `bind("<KeyRelease>")`, `<<ComboboxSelected>>`, `<<TreeviewSelect>>` |
| Cajas de diálogo | `showerror`, `showinfo`, `askyesno` |
| Widgets TTK | Controles `ttk` homogéneos |
| Combobox | Especie del formulario y filtro |
| Text | Observaciones multilínea |
| Treeview | Tabla con columnas e `iid` = ID |

---

## 4. Sección 1 — Datos iniciales

```python
ESPECIES = ("Perro", "Gato", "Conejo", "Loro", "Hámster")
ESPECIES_NATIVAS = ESPECIES  # alias: las nativas son exactamente estas cinco

animales = {
    1: {"nombre": "Luna", "especie": "Perro", "edad": 4, "peso": 18.5, ...},
    2: {...},
}
```

**Qué hace**

- `ESPECIES` / `ESPECIES_NATIVAS` (definidas en `datos/constantes.py`) son la tupla de las cinco especies con clase Python propia.  
  El catálogo completo que ve el usuario (nativas + personalizadas activas) lo gestiona `datos/gestor_especies.py` a través de `CatalogoEspecies.nombres_activos_para_alta()`.
- `animales` usa un **ID numérico** como clave y un diccionario como valor.

**¿Por qué un diccionario y no una lista?**

| Criterio | Diccionario por ID | Lista |
| --- | --- | --- |
| Acceso por ID | `animales[3]` en O(1) | Hay que buscar el elemento |
| Actualizar / borrar | Directo con la clave | Recorrer o conocer el índice |
| Similitud con BD | El ID actúa como clave primaria | Índice frágil si se reordena |

**Pregunta típica:** *¿Y si quiero añadir otro loro?*  
→ Añadir una entrada con el siguiente ID libre (o usar `crear_animal`).

---

## 5. Sección 2 — Funciones CRUD puras

Corazón lógico del proyecto, **separado de la interfaz**.

### `validar_datos(datos)`

1. Limpia espacios con `.strip()`.
2. Obliga nombre, especie (del catálogo activo vía `get_catalogo()`) y color.
3. Edad: entero entre 0 y 30 (rechaza `bool`).
4. Peso: `float` finito entre `PESO_MIN_KG` (0.01) y `PESO_MAX_KG` (100) kg.
5. Si falla → `ValueError` con mensaje claro.
6. Si va bien → diccionario limpio y normalizado (incluye campos clínicos: `foto`, `tratamientos`, `vacunaciones`, `documentos`).

La validación de especie consulta `catalogo.especie_permitida_en_alta(especie)`. Al actualizar un registro existente, también se admite la misma especie aunque esté desactivada (`especie_previa`), para no romper registros ya guardados.

### `crear_animal` / `leer_animal` / `actualizar_animal` / `borrar_animal`

- Reciben el inventario como **argumento** (no dependen de una global oculta).
- `leer_animal` usa `deepcopy` para devolver una copia profunda y no modificar el registro original.
- `crear_animal` calcula `max(inventario, default=0) + 1` → ID único.
- `borrar_animal` usa `.pop()`, llama a `borrar_adjuntos_animal` para limpiar archivos del disco y devuelve el registro eliminado.
- `actualizar_animal` fusiona los datos nuevos con los campos clínicos existentes antes de validar, para que editar los datos básicos no borre tratamientos o vacunaciones.

**Punto fuerte:** estas funciones se pueden probar sin abrir ninguna ventana (separación de responsabilidades).

---

## 6. Sección 3 — Prueba rápida

Usa `inventario_prueba = {}` independiente:

**Crear → Leer → Actualizar (edad 2→3) → Borrar (quedan 0)**

Es una mini prueba manual. En un proyecto real se usaría `unittest` o `pytest`; aquí basta para verificar el flujo antes de la GUI.

---

## 7. Sección 4 — Interfaz gráfica (`GestorAnimales`)

### 7.1 Variables de control

```python
self.nombre_var = tk.StringVar()
especies_alta = self.catalogo.nombres_activos_para_alta()
self.especie_var = tk.StringVar(value=especies_alta[0])
```

Enlazan widgets y datos: al escribir en un `Entry`, la variable se actualiza; al hacer `set()`, el campo se rellena.

El valor inicial de `especie_var` procede de `CatalogoEspecies.nombres_activos_para_alta()`, que devuelve las cinco especies nativas más las personalizadas activas que haya en `especies_personalizadas.json`.

### 7.2 Layout (_crear_controles y métodos _montar_*)

_crear_controles es el único método que llama el constructor. En lugar de construir toda la interfaz en un bloque único, delega cada zona a un método auxiliar privado con nombre _montar_*. Esto hace que _crear_controles sea un índice legible de la pantalla y que cada método sea responsable de una sola zona:

| Método auxiliar | Zona que construye |
| --- | --- |
| _montar_cabecera(contenedor) | Logo del refugio (efugio.png/.jpg) y títulos |
| _montar_formulario(contenedor) | LabelFrame «Datos del animal»: campos Entry, Combobox, Text de observaciones, panel de foto y botones de acción (Crear, Actualizar, Borrar, Limpiar, Exportar PDF, Ficha clínica / adjuntos, Gestionar especies) |
| _montar_filtros(contenedor) | LabelFrame «Buscar y filtrar»: campo de búsqueda por nombre y Combobox de especie |
| _montar_ficha_poo(contenedor) | LabelFrame «Información del animal seleccionado»: bloques de datos generales, comportamiento, alimentación y cuidados (con scrollbars) |
| _montar_tabla(contenedor) | Treeview del inventario con barras de desplazamiento vertical y horizontal |

Todos los bloques usan pack() para apilarse verticalmente en el contenedor principal. Los LabelFrame internos usan grid() para alinear sus controles en cuadrícula.

**Regla de oro:** no mezclar pack() y grid() en el **mismo** contenedor. Aquí el padre usa pack y cada LabelFrame interno usa grid.

**Ventaja de esta organización:** encontrar o modificar una zona concreta de la GUI no requiere leer todo el método; basta con localizar el _montar_ correspondiente. Los tests también pueden verificar zonas de forma independiente.

### 7.3 Treeview

- Columnas: id, nombre, especie, edad, peso, color, observaciones.
- `show="headings"` → sin columna fantasma del árbol.
- `selectmode="browse"` → una sola fila seleccionable.
- `iid=str(animal_id)` → el ID viaja con la fila.

### 7.4 Eventos (`bind`)

| Evento | Efecto |
| --- | --- |
| `<KeyRelease>` en búsqueda | Refresco en tiempo real |
| `<<ComboboxSelected>>` | Filtro por especie |
| `<<TreeviewSelect>>` | Carga la fila en el formulario |

### 7.5 Acciones `_crear` / `_actualizar` / `_borrar`

Patrón común:

```python
try:
    ...  # operación CRUD
except ValueError as error:
    messagebox.showerror("...", str(error))
    return
```

El borrado pide confirmación con `askyesno`.

### 7.6 `_actualizar_tabla`

Borra filas (`get_children` + `delete`) y reinserta aplicando:

- Filtro de texto en el nombre (`.casefold()`, parcial).
- Filtro de especie o “Todas”.
- Barra de estado: total / visibles.

---

## 8. Sección 5 — Ejecución

```python
raiz = tk.Tk()
aplicacion = GestorAnimales(raiz, animales)
raiz.mainloop()
```

Crea la ventana, pasa el inventario y arranca el bucle de eventos.

---

## 9. Preguntas de refactorización (y respuestas)

### 1. ¿Cómo añadirías una especie personalizada?

Conejo es ya una especie nativa en V5: está en `ESPECIES = ("Perro", "Gato", "Conejo", "Loro", "Hámster")` (`datos/constantes.py`) y tiene su propia clase Python en `datos/modelos.py`. No se puede volver a crear como personalizada.

Si queremos añadir **una especie nueva** (p. ej. Tortuga), se hace sin tocar el código Python, a través de la funcionalidad de gestión de especies de la interfaz:

1. En la GUI, pulsar **Gestionar especies** → **Añadir**. El diálogo pide nombre, descripción del sonido, tipo de alimentación y cuidados.
2. `CatalogoEspecies.anadir()` (`datos/gestor_especies.py`) guarda el registro en `datos/especies_personalizadas.json` (escritura atómica).
3. `registro_a_animal()` (`datos/adaptador_poo.py`) detecta que la especie no está en `CLASES_POR_ESPECIE` y construye un objeto `EspeciePersonalizada` (`datos/modelos.py`) con los atributos que vienen del catálogo, usando el mismo contrato `AnimalRefugio` que las nativas.
4. Los combos de alta y filtro se refrescan automáticamente mediante `_refrescar_desplegables_especies()`.

Desde ese momento la especie personalizada aparece en el `Combobox` del formulario, acepta registros y genera ficha POO. No es necesario tocar ningún archivo Python.

### 2. ¿Cómo guardar datos entre sesiones?

**Ya está hecho con JSON**, en dos archivos distintos:

1. **Inventario:** `datos/persistencia_inventario.py` → `inventario.json` (carga al arrancar; guarda tras CRUD/clínica).
2. **Especies personalizadas:** `datos/gestor_especies.py` → `especies_personalizadas.json`.

Evoluciones posibles: CSV (más plano) o **SQLite** (tabla `animales` con `PRIMARY KEY`).

### 3. ¿Cómo separarías el código en archivos?

Ya está hecho en `tarea1/`:

| Carpeta / archivo | Contenido |
| --- | --- |
| `datos/constantes.py` + `inventario.py` | `ESPECIES`, inventario inicial, CRUD |
| `datos/persistencia_inventario.py` | Carga/guarda `inventario.json` |
| `datos/gestor_especies.py` | Catálogo + `especies_personalizadas.json` |
| `validaciones/reglas.py` | `validar_datos` |
| `interfaz/gestor.py` | `GestorAnimales` |
| `app.py` / `main.py` | Arranque (`cargar_inventario` + `Tk` + `mainloop`) |

Ejecución: `python app.py` desde `tarea1/`.

### 4. ¿Cómo añadirías el campo "chip" (sí/no)?

Actualmente el campo **chip** no existe en el proyecto: es un cambio hipotético para ilustrar qué puntos del código habría que tocar.

**Lo que ya existe y se vería afectado:**

| Punto afectado | Qué habría que hacer |
| --- | --- |
| `validaciones/reglas.py` → `validar_datos()` | Añadir la lectura y validación del campo `chip` (`bool`); incluirlo en el diccionario limpio que devuelve la función |
| Formulario (`_montar_formulario` en `interfaz/gestor.py`) | Añadir `ttk.Checkbutton` enlazado a un `BooleanVar`; actualizar `_datos_del_formulario` y `_seleccionar` |
| Tabla `Treeview` (`_montar_tabla`) | Incluir `chip` en la tupla `columnas` y en `encabezados` / `anchos` |
| Ficha POO (`generar_ficha()` en `datos/modelos.py`) | Si se quiere mostrar el chip en la ficha POO, añadirlo en la sección `generales` de `AnimalRefugio.generar_ficha()` |
| Persistencia | El campo se guarda automáticamente en `datos/inventario.json` porque `guardar_inventario` serializa el `dict` completo; solo habría que añadir `chip` a `asegurar_campos_clinicos` (`datos/ficha.py`) para garantizar retrocompatibilidad con registros antiguos |
| Tests | Actualizar `tests/test_validaciones.py` y `tests/test_datos.py` para cubrir los casos con/sin chip |

### 5. ¿Por qué el CRUD no usa solo clases?

El enunciado pide **diccionarios**. Por eso el inventario sigue siendo `dict`. Las clases (`AnimalRefugio` y subclases) existen en `datos/modelos.py` con herencia y `@property`; el adaptador las construye cuando hace falta (p. ej. resumen polimórfico en la GUI).

### 6. ¿Qué pasa si dos animales tienen el mismo nombre?

No hay conflicto: el identificador único es el **ID**, no el nombre.

### 7. ¿Cómo harías estadísticas?

Botón que abra un `Toplevel` con totales por especie (`Counter`), medias de edad/peso y, opcionalmente, un `Canvas` o matplotlib.

### 8. ¿Por qué `_actualizar_tabla` borra y reinserta todo?

Con filtros, muchas filas dejan de ser visibles. Reconstruir la vista es simple y seguro. En inventarios pequeños el coste es imperceptible. A gran escala: `tabla.item(iid, values=...)`.

### 9. ¿Y si el peso se escribe con coma?

Normalizar en el formulario, por ejemplo:

```python
texto_peso = self.peso_var.get().replace(",", ".")
```

### 10. ¿Confirmación antes de salir?

```python
raiz.protocol("WM_DELETE_WINDOW", self._confirmar_salida)
```

y en `_confirmar_salida` un `askyesno`.

---

## 10. Consejos para explicar el proyecto

1. Empieza por lo general (CRUD + separación datos/GUI).  
2. Nombra principios: validación con excepciones, variables de control, gestores de geometría.  
3. Ante un cambio: lee el código, formula el plan y luego edita.  
4. Si algo no está implementado: describe el enfoque (“se haría así…”).  
5. Domina `pack` / `grid` / `place` (aquí no se usa `place`).

---

## 11. Conclusiones

El proyecto desarrollado cumple plenamente el objetivo de integrar en una sola aplicación una gestión CRUD de animales de refugio, con interfaz de usuario, persistencia de datos, validación, tratamiento de adjuntos y exportación de información. La solución se enmarca correctamente dentro del enunciado de la asignatura, ya que combina un almacenamiento basado en diccionarios con una capa POO demostrativa y una GUI desarrollada en Tkinter.

La principal fortaleza del sistema es su organización modular. La separación entre inventario, validación, interfaz y exportación permite ampliar el proyecto sin afectar a las demás capas. Asimismo, la gestión de especies personalizadas demuestra una capacidad de extensión real, evitando un diseño rígido y reforzando la reutilización del código.

Otra aportación notable es la persistencia del inventario y de las especies personalizadas en JSON, lo que convierte la aplicación en una herramienta útil más allá del entorno didáctico. El uso de validaciones concretas, mensajes de error controlados y funciones puras de manipulación del inventario contribuye a la robustez general del sistema.

En síntesis, el proyecto representa una solución completa y bien estructurada para la gestión de un refugio animal, con un equilibrio adecuado entre simplicidad didáctica y funcionalidad real. Su carácter modular, su dependencia de diccionarios frente a bases de datos relacionales y su conexión con POO y Tkinter lo convierten en un ejemplo muy apropiado para la entrega de la asignatura.

---

## 12. Checklist rápido de entrega

- [ ] Ejecutar la aplicación principal desde `python app.py` y comprobar CRUD, filtros, ficha clínica, importación de adjuntos y exportación PDF.
- [ ] Confirmar que el sistema arranca correctamente desde `main.py` como alias de ejecución.
- [ ] Ejecutar la batería de pruebas del proyecto y revisar el informe generado en `tests/resultados/`.
- [ ] Verificar que el inventario se persiste en `datos/inventario.json` y que las especies personalizadas quedan registradas en `datos/especies_personalizadas.json`.
- [ ] Comprobar la diferencia entre cuaderno didáctico y aplicación completa documentada en `README.md`.
- [ ] Revisar la documentación formal en `docs/` para requisitos, manuales y pruebas.
- [ ] Señalar las capas clave del sistema: datos, validación, interfaz y exportación.
- [ ] Explicar cómo se hace una especie personalizada sin modificar Python.
- [ ] Identificar la relación entre diccionarios del inventario y objetos POO del modelo.
- [ ] Ser capaz de explicar qué ocurre al seleccionar un animal y cómo se muestra su ficha en la GUI.

- [ ] Ejecutar `python app.py`: CRUD, filtros, ficha clínica, pegar/importar ≤ 5 MB, foto, exportar PDF.  
- [ ] Ejecutar `python ejecutar_tests.py` y abrir `tests/resultados/ultimo_informe.txt`.  
- [ ] (Opcional) `python probar_crud.py` para mostrar el CRUD sin GUI.  
- [ ] Provocar un error de validación y mostrar el `messagebox`.  
- [ ] Señalar módulos clave y diagramas en [docs/uml.md](docs/uml.md) / [docs/requisitos.md](docs/requisitos.md).  
- [ ] Tener respuesta lista para “añadir especie” y “dónde se guarda el inventario”.  
- [ ] Mostrar `inventario.json` ≠ `especies_personalizadas.json`.  
- [ ] Relacionar al menos 5 widgets con lecciones del Tema 3.  
- [ ] Usar el cuaderno solo como guion didáctico / documentación (no como sustituto de `app.py`).  
- [ ] Si alguien ejecuta el notebook: aclarar que la GUI completa es `python app.py`.  
- [ ] Conocer [docs/pruebas.md](docs/pruebas.md) y [docs/manual_usuario.md](docs/manual_usuario.md).

---

## 13. Archivos de esta carpeta

| Archivo / carpeta | Contenido |
| --- | --- |
| `app.py` / `main.py` | Arranque del programa GUI |
| `datos/`, `validaciones/`, `interfaz/` | Lógica de dominio, validación y interfaz |
| `exportacion/` | Generación de PDF de fichas |
| `probar_crud.py` | Prueba CRUD sin ventana |
| `ejecutar_tests.py` | Batería unittest + informe en `tests/resultados/` |
| `docs/` | Requisitos (RF/RNF), CU, UML, manuales, pruebas |
| `muestras/` | Fotos y PDF/DOCX de ejemplo para probar importación |
| `Enunciado_oficial_Tarea_1.pdf` | PDF del profesor |
| `Analisis_gestion_animales.md` | Esta guía (Markdown) |
| `README.md` | Índice del proyecto y diferencias entre cuaderno y programa |
| `gestion_animales_tkinter_V5_actualizado.ipynb` | Cuaderno didáctico actualizado |
| `arquitectura_refugio_animal_v5.ipynb` | Cuaderno de arquitectura v5 |
| `docs/Arquitectura_Refugio_Animal_v3.pdf` | Análisis de arquitectura en PDF |
| `generar_pdf.py` | Herramienta para regenerar PDFs de documentación |
| `adjuntos/` | Fotos/documentos importados por animal |
| `tests/resultados/` | Informes de ejecución de pruebas |

# 🍽️ Restaurante App — Semana 12

**Asignatura:** Programación Orientada a Objetos
**Estudiante:** Isabel Montaño
**Semana:** 12 — Utilización de colecciones para la mejora de rendimiento

---

## 📋 Tabla de contenido

1. [Descripción del sistema](#-descripción-del-sistema)
2. [Estructura del proyecto](#-estructura-del-proyecto)
3. [Responsabilidad de cada componente](#-responsabilidad-de-cada-componente)
4. [Arquitectura y flujo del sistema](#-arquitectura-y-flujo-del-sistema)
5. [Índices en memoria para mejorar el rendimiento](#-índices-en-memoria-para-mejorar-el-rendimiento-semana-12)
6. [Funcionamiento del stock](#-funcionamiento-del-stock)
7. [Relación Usuario–Producto mediante Venta](#-relación-usuarioproducto-mediante-venta)
8. [Persistencia de productos, usuarios y ventas](#-persistencia-de-productos-usuarios-y-ventas)
9. [Menú del sistema](#-menú-del-sistema)
10. [Instrucciones de ejecución](#-instrucciones-de-ejecución)
11. [Validaciones y manejo de excepciones](#-validaciones-y-manejo-de-excepciones)
12. [Pruebas realizadas](#-pruebas-realizadas)
13. [Reflexión](#-reflexión)

---

## 📖 Descripción del sistema

`restaurante_app` es un sistema de consola desarrollado en Python que administra **productos**, **usuarios** y **ventas** de un restaurante utilizando Programación Orientada a Objetos.

Esta versión, correspondiente a la **Semana 12**, parte de la base funcional de la Semana 11 (registro de productos con stock, usuarios, y la relación Usuario–Producto mediante `Venta`) y **no incorpora funcionalidades nuevas**. El objetivo de esta semana es exclusivamente de **rendimiento**: se analizaron las búsquedas, consultas y validaciones que ya existían en el sistema y se incorporaron estructuras auxiliares (`dict` y `set`) dentro del servicio `Restaurante` para evitar recorrer listas completas cuando existe una clave conocida, como el código de un producto o la identificación de un usuario.

| Elemento | ¿Qué representa en este proyecto? |
|------------|-------------------------------------|
| `Producto` | Cada plato/bebida del restaurante, con su stock disponible |
| `Usuario`  | La persona registrada que puede realizar una compra |
| `Venta`    | La relación entre un Usuario y un Producto vendido |
| `Restaurante` | Servicio que administra las colecciones, las reglas de negocio y, desde esta semana, los **índices auxiliares** de búsqueda |
| `ArchivoServicio` | Servicio que persiste las tres colecciones en archivos JSON |

---

## 🗂 Estructura del proyecto

restaurante_app/
│
├── datos/
│   ├── productos.json      # Productos y su stock actualizado
│   ├── usuarios.json       # Usuarios registrados
│   └── ventas.json         # Relación Usuario–Producto de cada venta
│
├── modelos/
│   ├── __init__.py
│   ├── producto.py         # Clase Producto (con stock)
│   ├── usuario.py          # Clase Usuario
│   └── venta.py            # Clase Venta
│
├── servicios/
│   ├── __init__.py
│   ├── archivo_servicio.py # Persistencia JSON de las 3 colecciones
│   └── restaurante.py      # Lógica de negocio + índices en memoria (mejora Semana 12)
│
├── main.py                 # Punto de arranque + menú interactivo
└── README.md                # Este archivo

**Vista jerárquica del paquete:**

    graph TD
        A[restaurante_app] --> DAT[datos/]
        A --> B[modelos/]
        A --> C[servicios/]
        A --> D[main.py]
        A --> E[README.md]

        DAT --> DAT1[productos.json]
        DAT --> DAT2[usuarios.json]
        DAT --> DAT3[ventas.json]

        B --> B1[__init__.py]
        B --> B2[producto.py]
        B --> B3[usuario.py]
        B --> B4[venta.py]

        C --> C1[__init__.py]
        C --> C2[archivo_servicio.py]
        C --> C3[restaurante.py]

        style A fill:#2c3e50,color:#fff
        style D fill:#e67e22,color:#fff
        style C2 fill:#c0392b,color:#fff
        style C3 fill:#2980b9,color:#fff
        style B2 fill:#27ae60,color:#fff
        style B3 fill:#27ae60,color:#fff
        style B4 fill:#8e44ad,color:#fff

---

## 🧩 Responsabilidad de cada componente

| Archivo | Responsabilidad |
|---|---|
| `modelos/producto.py` | Define `Producto`: código, nombre, categoría, precio y **stock** (validado para que nunca sea negativo). Incluye `vender(cantidad)` para descontar stock de forma controlada. |
| `modelos/usuario.py` | Define `Usuario`: identificación, nombre y correo, validados con `@property`. |
| `modelos/venta.py` | Define `Venta`: representa la relación entre un usuario y un producto vendido (`usuario_id`, `producto_codigo`, `cantidad`). |
| `servicios/restaurante.py` | Administra las **tres colecciones principales** (`_productos`, `_usuarios`, `_ventas`) y toda la lógica de negocio: registrar, buscar, actualizar, eliminar, `vender_producto()` y `consultar_ventas_usuario()`. **Desde la Semana 12**, incorpora además los **índices auxiliares** (`_productos_por_codigo`, `_usuarios_por_identificacion`, `_ventas_por_usuario`, `_categorias`) que aceleran esas mismas operaciones. |
| `servicios/archivo_servicio.py` | Centraliza la lectura y escritura de `productos.json`, `usuarios.json` y `ventas.json`. **No sufrió cambios esta semana.** |
| `main.py` | Punto de arranque. Contiene el menú, solicita datos por consola (`input()`) y coordina las llamadas al servicio `Restaurante`. **No sufrió cambios esta semana.** |
| `README.md` | Documentación del proyecto. |

> ⚠️ **Regla de arquitectura:** `main.py` **nunca** accede directamente a las listas ni a los índices internos del servicio. Toda operación se realiza a través de los métodos públicos de `Restaurante`.

---

## 🔄 Arquitectura y flujo del sistema

    flowchart TD
        U([Usuario del sistema]) -->|Selecciona una opción| M[main.py]
        M -->|Solicita datos con input| U
        M -->|Crea objetos Producto/Usuario| MOD[modelos/]
        M -->|Llama métodos del servicio| S[servicios/restaurante.py]
        S -->|Administra| L1[(list productos)]
        S -->|Administra| L2[(list usuarios)]
        S -->|Administra| L3[(list ventas)]
        S -->|Busca/valida vía| IDX[(indices dict/set)]
        IDX -->|Sincronizado con| L1
        IDX -->|Sincronizado con| L2
        IDX -->|Sincronizado con| L3
        S -->|Retorna resultado| M
        M -->|Solicita guardar| AS[servicios/archivo_servicio.py]
        AS -->|Escribe| J1[(productos.json)]
        AS -->|Escribe| J2[(usuarios.json)]
        AS -->|Escribe| J3[(ventas.json)]
        M -->|Muestra resultado| U

        style M fill:#e67e22,color:#fff
        style S fill:#2980b9,color:#fff
        style AS fill:#c0392b,color:#fff
        style MOD fill:#27ae60,color:#fff
        style L1 fill:#8e44ad,color:#fff
        style L2 fill:#8e44ad,color:#fff
        style L3 fill:#8e44ad,color:#fff
        style IDX fill:#f1c40f,color:#000

Al iniciar el programa, `ArchivoServicio` recupera las listas de productos, usuarios y ventas desde JSON, y `Restaurante` las recibe para construir sus colecciones principales. **A partir de esta semana**, inmediatamente después `Restaurante` reconstruye también sus índices en memoria a partir de esas mismas listas, antes de que el menú quede disponible.

---

## 🔍 Índices en memoria para mejorar el rendimiento (Semana 12)

La lógica de negocio no cambió: lo que cambió es **cómo** `Restaurante` busca internamente. Se mantienen las listas `_productos`, `_usuarios` y `_ventas` como colección principal (se siguen usando para listar, recorrer y persistir en JSON), y se agregaron estructuras auxiliares únicamente donde existía una clave de búsqueda frecuente:

| Estructura | Tipo | Reemplaza a este recorrido | Usada en |
|---|---|---|---|
| `_productos_por_codigo` | `dict[str, Producto]` | Recorrer toda la lista de productos comparando código | `buscar_producto`, `existe_codigo_producto`, `actualizar_producto`, `eliminar_producto`, `vender_producto` |
| `_usuarios_por_identificacion` | `dict[str, Usuario]` | Recorrer toda la lista de usuarios comparando identificación | `buscar_usuario`, `existe_identificacion_usuario`, `vender_producto` |
| `_ventas_por_usuario` | `dict[str, list[Venta]]` | Recorrer y filtrar toda la lista de ventas en cada consulta | `consultar_ventas_usuario` |
| `_categorias` | `set[str]` | Reconstruir la lista de categorías recorriendo todos los productos | `obtener_categorias` |

`_reconstruir_indices()` se ejecuta una sola vez, dentro de `__init__`, justo después de recibir las listas cargadas desde JSON. Cada operación que agrega, actualiza o elimina un producto, un usuario o una venta actualiza la lista principal **y** su índice correspondiente en la misma operación, para que ambos nunca queden desincronizados.

Con los datos reales de este repositorio (`datos/productos.json`, `datos/usuarios.json` y `datos/ventas.json`), al iniciar el programa los índices quedan así:

    _productos_por_codigo   -> {"0001": Pizza, "0002": Limonda}
    _usuarios_por_identificacion -> {"115051": Leo, "172021": Isabel}
    _ventas_por_usuario      -> {"172021": [Venta(0001, 3)], "115051": [Venta(0001, 2)]}
    _categorias              -> {"Comida rápida", "Bebida"}

> No se reemplazó ninguna lista por un diccionario: `Producto`, `Usuario` y `Venta` se siguen recorriendo y persistiendo como listas; los índices son estructuras adicionales que solo existen para acelerar la búsqueda por clave.

---

## 📦 Funcionamiento del stock

Cada `Producto` mantiene un atributo `stock` validado en su `@property.setter` (nunca puede ser negativo). Al vender, primero se comprueba que la cantidad solicitada sea mayor que cero y que exista stock suficiente; solo entonces se descuenta:

    Antes de vender    -> Limonda | Stock: 30
    Cantidad solicitada -> 5
    Después de vender  -> Limonda | Stock: 25
    Venta registrada correctamente

Si se intenta vender una cantidad mayor al stock disponible, la operación se rechaza y **no se modifica ningún dato**. Esta regla no cambió esta semana; lo único distinto es que ahora `buscar_producto()` y `buscar_usuario()` usan los índices en lugar de recorrer las listas.

---

## 🔗 Relación Usuario–Producto mediante Venta

    sequenceDiagram
        participant U as Usuario (consola)
        participant M as main.py
        participant R as Restaurante
        participant IDX as Indices (dict)
        participant P as Producto
        participant V as Venta

        U->>M: Selecciona "Vender producto"
        M->>R: buscar_usuario(identificacion)
        R->>IDX: _usuarios_por_identificacion.get(identificacion)
        M->>R: buscar_producto(codigo)
        R->>IDX: _productos_por_codigo.get(codigo)
        M->>R: vender_producto(codigo, identificacion, cantidad)
        R->>R: valida cantidad > 0 y stock suficiente
        R->>V: crea Venta(usuario_id, producto_codigo, cantidad)
        R->>IDX: _ventas_por_usuario[usuario_id].append(venta)
        R->>P: producto.vender(cantidad)
        R-->>M: True / False
        M-->>U: muestra resultado

**Flujo textual equivalente:**

    Usuario registrado (buscado por indice _usuarios_por_identificacion)
    ↓
    Producto existente (buscado por indice _productos_por_codigo)
    ↓
    Validar cantidad solicitada
    ↓
    Validar stock disponible
    ↓
    Crear Venta(usuario_id, producto_codigo, cantidad)
    ↓
    Agregar Venta a la coleccion _ventas y al indice _ventas_por_usuario
    ↓
    Disminuir stock del producto
    ↓
    Guardar ventas.json y productos.json

`consultar_ventas_usuario()` **ya no recorre** toda la colección de ventas: consulta directamente `_ventas_por_usuario` con la identificación del usuario. Por ejemplo, con los datos de este repositorio, consultar al usuario `172021` (Isabel) devuelve únicamente su venta de 3 unidades del producto `0001` (Pizza), sin revisar la venta de Leo (`115051`).

---

## 💾 Persistencia de productos, usuarios y ventas

    OBJETOS
    ↓
    convertir_a_diccionario()
    ↓
    lista de diccionarios
    ↓
    json.dump()
    ↓
    archivo JSON

    archivo JSON
    ↓
    json.load()
    ↓
    diccionarios
    ↓
    reconstrucción de objetos
    ↓
    reconstrucción de índices (_reconstruir_indices())

| Archivo | ¿Qué conserva? | ¿Cuándo se guarda? |
|---|---|---|
| `productos.json` | Productos y su stock actualizado | Registrar, actualizar, eliminar producto **y** después de cada venta |
| `usuarios.json` | Usuarios registrados | Registrar usuario |
| `ventas.json` | Relación usuario–producto–cantidad | Después de cada venta exitosa |

> ⚠️ Una sola operación puede modificar más de una colección: al vender, se registra una nueva `Venta` **y**, al mismo tiempo, se actualiza el stock del `Producto`. Por eso `vender_producto()` guarda ambos archivos. Esta parte no cambió; lo que se agregó es que, además de reconstruir los objetos, `Restaurante` reconstruye sus índices apenas termina de recibir las listas.

---

## 📋 Menú del sistema

    ========================================
               SISTEMA DE RESTAURANTE
    ========================================
    1. Registrar producto
    2. Buscar producto
    3. Actualizar producto
    4. Eliminar producto
    5. Listar productos
    6. Registrar usuario
    7. Listar usuarios
    8. Mostrar categorías
    9. Vender producto
    10. Consultar ventas de un usuario
    11. Listar todas las ventas
    12. Salir

El menú no cambió esta semana: las mismas 12 opciones ahora se resuelven usando los índices en memoria.

---

## ▶️ Instrucciones de ejecución

### Requisitos
- Python 3.9 o superior instalado.
- No requiere librerías externas (solo la biblioteca estándar).

### Pasos

1. Clonar o descargar este repositorio.
2. Ubicarse en la carpeta raíz del proyecto (donde está `restaurante_app/`).
3. Ejecutar el punto de arranque:

       cd restaurante_app
       python main.py

4. Usar el menú interactivo para registrar productos (con stock), usuarios, vender productos, consultar ventas por usuario, y listar todo.
5. Seleccionar la opción `12` para salir del programa.

> ⚠️ **Importante:** el único archivo que se debe ejecutar directamente es `main.py`. La carpeta `datos/` ya viene incluida en este repositorio con productos, usuarios y ventas de ejemplo, y se actualiza automáticamente cada vez que se guarda información.

---

## 🛡 Validaciones y manejo de excepciones

- `Producto`, `Usuario` y `Venta` usan `@property` para validar sus datos en el momento en que se asignan (código/nombre/categoría/correo no vacíos, precio y stock numéricos y no negativos, cantidad mayor que cero).
- `ArchivoServicio` controla, para cada una de las tres colecciones:
  - `FileNotFoundError`: si el archivo aún no existe, inicia con lista vacía.
  - `json.JSONDecodeError`: si el contenido no es JSON válido.
  - `PermissionError`: si no hay permisos de lectura/escritura.
  - `KeyError`: si un registro no tiene una clave esperada (se omite y se informa por consola).
  - `ValueError`: para datos inválidos al reconstruir un objeto.
- No se utiliza `except: pass` en ninguna parte del proyecto.
- No se permiten **códigos de producto** ni **identificaciones de usuario** duplicadas (ahora validado con `existe_codigo_producto`/`existe_identificacion_usuario` contra los índices, en lugar de recorrer las listas).
- `vender_producto()` impide la venta si el usuario o el producto no existen, si la cantidad es inválida, o si el stock es insuficiente.

---

## 🧪 Pruebas realizadas

Las pruebas se realizaron sobre los datos reales incluidos en `datos/`: productos `0001` (Pizza, stock 18) y `0002` (Limonda, stock 30); usuarios `115051` (Leo) y `172021` (Isabel); y dos ventas previas ya registradas en `ventas.json`.

1. Se cargaron los tres archivos JSON al iniciar el programa: 2 productos, 2 usuarios y 2 ventas, sin errores de formato.
2. Se buscó el producto `0001` con `buscar_producto()` y se obtuvo Pizza correctamente, usando el índice `_productos_por_codigo`.
3. Se buscó el usuario `172021` con `buscar_usuario()` y se obtuvo Isabel correctamente, usando el índice `_usuarios_por_identificacion`.
4. Se consultó `consultar_ventas_usuario("172021")` y solo devolvió la venta de 3 unidades de Pizza de Isabel; se consultó `consultar_ventas_usuario("115051")` y solo devolvió la venta de 2 unidades de Pizza de Leo, confirmando que el índice `_ventas_por_usuario` separa correctamente por usuario.
5. Se llamó a `obtener_categorias()` y devolvió `{"Comida rápida", "Bebida"}` sin duplicados.
6. Se realizó una venta nueva de 5 unidades de Limonda (`0002`) al usuario Leo (`115051`): la venta se registró, el stock de Limonda bajó de 30 a 25, y la consulta de ventas de Leo pasó a mostrar 2 ventas en total.
7. Se cerró el programa y se volvió a ejecutar: productos, usuarios y ventas se recuperaron correctamente desde los archivos JSON, y los índices se reconstruyeron automáticamente en `_reconstruir_indices()` antes de mostrar el menú, con los mismos resultados de búsqueda que antes de cerrar.
8. Se intentó vender una cantidad mayor al stock disponible: la operación fue rechazada y no se modificaron los datos ni los índices.

---

## 💭 Reflexión

Incorporar la operación de venta como una nueva colección (`Venta`) permitió representar de forma explícita la relación entre un usuario y un producto, y conservar un historial de lo ocurrido; eso ya se había trabajado en la Semana 11. Esta semana, en cambio, la reflexión pasa por otro lado: no toda mejora en un sistema orientado a objetos consiste en agregar funcionalidades nuevas, sino en revisar las operaciones que ya existen y preguntarse cuáles se repiten con frecuencia y podrían resolverse de forma más directa. Usar un diccionario cuando existe una clave única, y un `set` cuando solo interesa la pertenencia o la unicidad de un valor, no reemplaza a las listas: las complementa. Las listas siguen siendo necesarias porque son las que permiten recorrer, listar y persistir los objetos tal como son; los índices solo existen para que, cuando se conoce la clave, no haga falta recorrer nada. Probar el sistema con datos reales (dos productos, dos usuarios y dos ventas ya existentes) permitió confirmar que los índices no solo funcionan en teoría: separan correctamente las ventas de cada usuario y encuentran cada producto o usuario sin depender del orden en que fueron cargados desde el JSON. Mantener ambas estructuras sincronizadas en cada registro, actualización, eliminación o venta refuerza además que el rendimiento no es una mejora aislada: depende de que toda la lógica de negocio quede escrita con disciplina para no dejar ninguna estructura desactualizada.
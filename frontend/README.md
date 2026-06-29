# Frontend - Gestión de Productos

Frontend en Java que genera una página estática HTML para gestión de productos.

## Características

- Registro de usuarios
- Visualización de productos (solo lectura, no se puede comprar)
- Interfaz moderna y responsiva
- Almacenamiento local de usuarios registrados (localStorage)

## Requisitos

- Java 17 o superior
- Maven 3.6 o superior

## Build

Para generar la página estática:

```bash
mvn clean compile exec:java
```

Esto generará los archivos estáticos en el directorio `output/`:
- `index.html` - Página principal
- `styles.css` - Estilos CSS
- `script.js` - Funcionalidad JavaScript

## Ejecutar

Abra el archivo `output/index.html` en su navegador web.

## Estructura del Proyecto

```
front/
├── pom.xml                          # Configuración Maven
├── README.md                        # Este archivo
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── eval3/
│       │           └── frontend/
│       │               └── StaticPageGenerator.java  # Generador de páginas
│       └── resources/
└── output/                          # Directorio generado con archivos estáticos
    ├── index.html
    ├── styles.css
    └── script.js
```

## Funcionalidades

### Registro de Usuarios
- Formulario de registro con validación
- Almacenamiento en localStorage
- Validación de contraseñas
- Prevención de duplicados

### Visualización de Productos
- Grid de productos con información detallada
- 8 productos de ejemplo pre-cargados
- Diseño responsivo
- Solo lectura (no se pueden realizar compras)

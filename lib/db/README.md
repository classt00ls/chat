# Estructura de la base de datos

Este directorio contiene todos los archivos relacionados con la base de datos del proyecto. A continuación, se explica el propósito de cada archivo y directorio.

## Archivos principales

### `schema.ts`

Este archivo define el esquema de la base de datos utilizando Drizzle ORM. Define todas las tablas, sus columnas, tipos de datos, relaciones y restricciones.

Las tablas principales definidas son:
- `User`: Almacena información de los usuarios (email, contraseña).
- `Chat`: Almacena los chats creados por los usuarios.
- `Message`: Almacena los mensajes dentro de los chats.
- `Vote`: Almacena los votos (positivos/negativos) para los mensajes.
- `Document`: Almacena documentos creados por los usuarios.
- `Suggestion`: Almacena sugerencias relacionadas con los documentos.

Cada tabla tiene definidos sus tipos utilizando `InferSelectModel` de Drizzle, lo que facilita el tipado en TypeScript.

### `queries.ts`

Este archivo contiene todas las funciones que interactúan con la base de datos. Estas funciones encapsulan las operaciones CRUD (Crear, Leer, Actualizar, Eliminar) para cada entidad.

Algunas de las funciones principales son:
- `getUser`: Obtiene un usuario por su email.
- `createUser`: Crea un nuevo usuario con email y contraseña (hasheada).
- `saveChat`: Guarda un nuevo chat.
- `getChatsByUserId`: Obtiene todos los chats de un usuario.
- `saveMessages`: Guarda mensajes en un chat.
- `getMessagesByChatId`: Obtiene todos los mensajes de un chat.

Todas estas funciones utilizan Drizzle ORM para construir y ejecutar consultas SQL de manera segura y con tipado.

### `migrate.ts`

Este archivo contiene el código para ejecutar las migraciones de la base de datos. Las migraciones son cambios en el esquema de la base de datos que se aplican de manera secuencial.

El script:
1. Lee la variable de entorno `POSTGRES_URL` para conectarse a la base de datos.
2. Establece una conexión a PostgreSQL.
3. Ejecuta todas las migraciones encontradas en la carpeta `migrations`.
4. Registra el tiempo que tarda en completarse.

Este script se ejecuta con el comando `pnpm db:migrate`.

## Directorio `migrations/`

Este directorio contiene todos los archivos de migración generados por Drizzle Kit. Cada archivo representa un cambio en el esquema de la base de datos.

Los archivos de migración se nombran con un prefijo numérico (por ejemplo, `0000_`, `0001_`) para indicar el orden en que deben aplicarse.

Dentro de este directorio también hay una carpeta `meta/` que contiene metadatos sobre las migraciones, como instantáneas del esquema en diferentes puntos del tiempo.

## Cómo se utiliza

1. **Definir el esquema**: Modificar `schema.ts` para definir nuevas tablas o modificar las existentes.
2. **Generar migraciones**: Ejecutar `pnpm db:generate` para generar archivos de migración basados en los cambios en el esquema.
3. **Aplicar migraciones**: Ejecutar `pnpm db:migrate` para aplicar las migraciones a la base de datos.
4. **Interactuar con la base de datos**: Utilizar las funciones definidas en `queries.ts` para interactuar con la base de datos desde la aplicación.

## Conexión a la base de datos

La conexión a la base de datos se establece utilizando la variable de entorno `POSTGRES_URL`. Esta URL debe tener el formato:

```
postgres://usuario:contraseña@host:puerto/nombre_db?sslmode=require
```

La conexión se establece en `queries.ts` utilizando la biblioteca `postgres` y luego se envuelve con Drizzle ORM para proporcionar una interfaz tipada para interactuar con la base de datos. 
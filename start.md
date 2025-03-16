# Guía de inicio para el proyecto de chat

## 1. Configuración de la autenticación

Para hacer funcionar el sistema de autenticación en este proyecto, hemos tenido que realizar los siguientes pasos:

### 1.1. Entender el sistema de autenticación

El proyecto utiliza NextAuth.js (ahora conocido como Auth.js) para manejar la autenticación de usuarios. La autenticación se basa en:

- **Proveedor de credenciales**: Utiliza email y contraseña para autenticar usuarios.
- **Base de datos PostgreSQL**: Almacena la información de los usuarios.
- **Drizzle ORM**: Gestiona las consultas a la base de datos.
- **bcrypt**: Hashea las contraseñas para almacenarlas de forma segura.

### 1.2. Configurar las variables de entorno

Creamos un archivo `.env.local` en la raíz del proyecto con las siguientes variables:

```
AUTH_SECRET="un_secreto_aleatorio"
POSTGRES_URL="postgres://usuario:contraseña@host:puerto/nombre_db?sslmode=require"
```

Donde:
- `AUTH_SECRET`: Es un secreto aleatorio utilizado por NextAuth para cifrar cookies y tokens.
- `POSTGRES_URL`: Es la URL de conexión a la base de datos PostgreSQL.

### 1.3. Crear las tablas en la base de datos

Para que la autenticación funcione, necesitamos crear las tablas definidas en el esquema (`lib/db/schema.ts`). Esto se hace ejecutando:

```bash
pnpm db:migrate
```

Este comando ejecuta el script `lib/db/migrate.ts` que aplica todas las migraciones definidas en la carpeta `lib/db/migrations`.

Las tablas principales que se crean son:
- `User`: Almacena información de los usuarios (email, contraseña hasheada).
- `Chat`: Almacena los chats de los usuarios.
- `Message`: Almacena los mensajes de los chats.
- Y otras tablas relacionadas con la funcionalidad del chat.

### 1.4. Flujo de autenticación

Una vez configurado, el sistema permite:

- **Registro de usuarios**: Los usuarios pueden registrarse proporcionando un email y contraseña.
- **Inicio de sesión**: Los usuarios pueden iniciar sesión con sus credenciales.
- **Protección de rutas**: El middleware de NextAuth protege las rutas según la configuración.

### 1.5. Problemas comunes y soluciones

- **Error de conexión a la base de datos**: Asegúrate de que la URL de PostgreSQL es correcta y que la base de datos está accesible.
- **Tablas no creadas**: Ejecuta `pnpm db:migrate` para crear las tablas necesarias.
- **Variables de entorno no cargadas**: Verifica que el archivo `.env.local` existe y tiene las variables correctas. 
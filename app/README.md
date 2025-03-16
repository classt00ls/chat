# Arquitectura Serverless de Next.js en este Proyecto

Este documento explica cómo se implementa la arquitectura serverless de Next.js en este proyecto, detallando la estructura de carpetas y archivos clave.

## Estructura App Router

Este proyecto utiliza el App Router de Next.js, que se basa en una estructura de carpetas para definir rutas y comportamientos. La carpeta `app` es la raíz de esta estructura.

### Grupos de Rutas

El proyecto utiliza grupos de rutas (route groups) para organizar el código:

- `(auth)`: Contiene todo lo relacionado con la autenticación.
- `(chat)`: Contiene todo lo relacionado con la funcionalidad de chat.

Estos grupos no afectan a las URL finales, pero permiten organizar el código de manera lógica.

## Componentes del Servidor (Server Components)

Next.js 14 utiliza React Server Components (RSC) por defecto. Esto significa que la mayoría de los componentes se ejecutan en el servidor, lo que mejora el rendimiento y la seguridad.

Ejemplos de Server Components en este proyecto:
- `app/layout.tsx`: Define el layout principal de la aplicación.
- `app/(chat)/page.tsx`: Página principal que se renderiza en el servidor.

## Acciones del Servidor (Server Actions)

Las Server Actions permiten ejecutar código en el servidor desde el cliente de manera segura. En este proyecto se utilizan para:

### En la autenticación (`app/(auth)/actions.ts`):
```typescript
export const login = async (
  _: LoginActionState,
  formData: FormData,
): Promise<LoginActionState> => {
  // Código para iniciar sesión
};

export const register = async (
  _: RegisterActionState,
  formData: FormData,
): Promise<RegisterActionState> => {
  // Código para registrar usuarios
};
```

### En el chat (`app/(chat)/actions.ts`):
```typescript
export const createChat = async () => {
  // Código para crear un nuevo chat
};

export const deleteChat = async (id: string) => {
  // Código para eliminar un chat
};
```

## API Routes (Rutas de API)

Next.js permite crear API endpoints serverless mediante la estructura de carpetas. En este proyecto:

### Estructura de API Routes:
- `app/(auth)/api/auth/[...nextauth]/route.ts`: Maneja las rutas de autenticación.
- `app/(chat)/api/chat/route.ts`: Maneja las solicitudes de chat.
- `app/(chat)/api/files/upload/route.ts`: Maneja la subida de archivos.
- `app/(chat)/api/history/route.ts`: Maneja el historial de chats.
- `app/(chat)/api/vote/route.ts`: Maneja los votos en mensajes.

Cada uno de estos archivos exporta funciones como `GET`, `POST`, etc., que se ejecutan como funciones serverless cuando se accede a la ruta correspondiente.

## Middleware

El middleware (`middleware.ts` en la raíz) se ejecuta antes de cada solicitud y se utiliza principalmente para la autenticación:

```typescript
import NextAuth from 'next-auth';
import { authConfig } from '@/app/(auth)/auth.config';

export default NextAuth(authConfig).auth;

export const config = {
  matcher: ['/', '/:id', '/api/:path*', '/login', '/register'],
};
```

Este middleware verifica si el usuario está autenticado antes de permitir el acceso a ciertas rutas.

## Autenticación Serverless

La autenticación se implementa utilizando NextAuth.js, que se integra perfectamente con la arquitectura serverless:

- `app/(auth)/auth.config.ts`: Define la configuración de autenticación.
- `app/(auth)/auth.ts`: Implementa los proveedores y callbacks de autenticación.
- `app/(auth)/api/auth/[...nextauth]/route.ts`: Expone los endpoints de autenticación.

## Conexión a la Base de Datos

La conexión a la base de datos también sigue un enfoque serverless:

```typescript
// lib/db/queries.ts
const client = postgres(process.env.POSTGRES_URL!);
const db = drizzle(client);
```

Esto permite que cada función serverless establezca su propia conexión a la base de datos cuando sea necesario, en lugar de mantener conexiones persistentes.

## Optimizaciones de Rendimiento

El proyecto utiliza varias optimizaciones para mejorar el rendimiento en un entorno serverless:

- **Partial Prerendering (PPR)**: Habilitado en `next.config.ts` con `ppr: true`.
- **Edge Runtime**: Algunas rutas utilizan el Edge Runtime para menor latencia.
- **Streaming**: Implementado en componentes de chat para mostrar respuestas incrementalmente.

## Despliegue Serverless

Este proyecto está diseñado para desplegarse en Vercel, donde cada ruta y API endpoint se convierte automáticamente en una función serverless. Esto proporciona:

- Escalado automático
- Despliegue global
- Sin servidor que mantener
- Pago por uso

## Conclusión

La arquitectura serverless de Next.js en este proyecto permite una aplicación altamente escalable y de alto rendimiento, con una clara separación entre el código del cliente y del servidor. La estructura de carpetas del App Router facilita la organización del código y la implementación de funcionalidades como autenticación, API routes y server actions. 
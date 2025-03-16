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

## Análisis Detallado de los Archivos `actions.ts`

### ¿Qué son los archivos `actions.ts`?

Los archivos `actions.ts` son una implementación de Server Actions de Next.js, una característica que permite ejecutar funciones en el servidor directamente desde componentes del cliente. Estos archivos se identifican por la directiva `'use server';` al inicio, que indica a Next.js que todas las funciones exportadas deben ejecutarse en el servidor.

### ¿Por qué se definen en esos lugares?

1. **Colocación estratégica por contexto**:
   - `app/(auth)/actions.ts`: Se coloca en el grupo de rutas de autenticación porque contiene acciones específicas de autenticación.
   - `app/(chat)/actions.ts`: Se coloca en el grupo de rutas de chat porque contiene acciones específicas de la funcionalidad de chat.

2. **Separación de responsabilidades**:
   - Esta estructura sigue el principio de separación de responsabilidades, agrupando las acciones según su función.
   - Facilita el mantenimiento y la escalabilidad del código.

3. **Optimización de carga**:
   - Al colocar las acciones cerca de los componentes que las utilizan, Next.js puede optimizar la carga de código.

### ¿Cómo funcionan estas acciones?

#### Estructura de una Server Action

Las Server Actions siguen un patrón común:

1. **Directiva `'use server'`**: Indica que la función se ejecuta en el servidor.
2. **Función asíncrona**: Permite operaciones asíncronas como acceso a base de datos.
3. **Parámetros tipados**: Utilizan TypeScript para definir tipos de entrada y salida.
4. **Manejo de errores**: Incluyen bloques try/catch para manejar errores.
5. **Retorno de estado**: Devuelven objetos que indican el resultado de la operación.

#### Ejemplo detallado: `login` en `app/(auth)/actions.ts`

```typescript
export const login = async (
  _: LoginActionState,
  formData: FormData,
): Promise<LoginActionState> => {
  try {
    // 1. Validación de datos con Zod
    const validatedData = authFormSchema.parse({
      email: formData.get('email'),
      password: formData.get('password'),
    });

    // 2. Intento de inicio de sesión con NextAuth
    await signIn('credentials', {
      email: validatedData.email,
      password: validatedData.password,
      redirect: false,
    });

    // 3. Retorno de estado exitoso
    return { status: 'success' };
  } catch (error) {
    // 4. Manejo de errores específicos
    if (error instanceof z.ZodError) {
      return { status: 'invalid_data' };
    }

    // 5. Manejo de errores generales
    return { status: 'failed' };
  }
};
```

#### Ejemplo detallado: `updateChatVisibility` en `app/(chat)/actions.ts`

```typescript
export async function updateChatVisibility({
  chatId,
  visibility,
}: {
  chatId: string;
  visibility: VisibilityType;
}) {
  // Llamada directa a la función de base de datos
  await updateChatVisiblityById({ chatId, visibility });
}
```

### Integración con componentes del cliente

Los componentes del cliente utilizan estas acciones de varias formas:

1. **Formularios con `action`**:
   ```tsx
   <form action={login}>
     {/* Campos del formulario */}
   </form>
   ```

2. **Hook `useFormState`** (para manejar el estado del formulario):
   ```tsx
   const [state, formAction] = useFormState(login, { status: 'idle' });
   ```

3. **Hook `useFormStatus`** (para mostrar estados de carga):
   ```tsx
   const { pending } = useFormStatus();
   ```

4. **Llamada directa** (para acciones no vinculadas a formularios):
   ```tsx
   const handleClick = async () => {
     await updateChatVisibility({ chatId, visibility: 'public' });
   };
   ```

## Ejemplos Concretos de Uso de Server Actions

Veamos ejemplos reales de cómo se utilizan las Server Actions en este proyecto, desde su definición hasta su invocación:

### Ejemplo 1: Formulario de Login

#### 1. Definición de la acción en `app/(auth)/actions.ts`:
```typescript
'use server';

import { z } from 'zod';
import { signIn } from './auth';

const authFormSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export interface LoginActionState {
  status: 'idle' | 'in_progress' | 'success' | 'failed' | 'invalid_data';
}

export const login = async (
  _: LoginActionState,
  formData: FormData,
): Promise<LoginActionState> => {
  try {
    const validatedData = authFormSchema.parse({
      email: formData.get('email'),
      password: formData.get('password'),
    });

    await signIn('credentials', {
      email: validatedData.email,
      password: validatedData.password,
      redirect: false,
    });

    return { status: 'success' };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { status: 'invalid_data' };
    }
    return { status: 'failed' };
  }
};
```

#### 2. Uso en el componente de cliente `app/(auth)/login/page.tsx`:
```typescript
'use client';

import { useRouter } from 'next/navigation';
import { useActionState, useEffect, useState } from 'react';
import { toast } from '@/components/toast';
import { AuthForm } from '@/components/auth-form';
import { SubmitButton } from '@/components/submit-button';
import { login, type LoginActionState } from '../actions';

export default function Page() {
  const router = useRouter();
  const [email, setEmail] = useState('');
  const [isSuccessful, setIsSuccessful] = useState(false);

  // Integración con useActionState para manejar el estado del formulario
  const [state, formAction] = useActionState<LoginActionState, FormData>(
    login,
    { status: 'idle' }
  );

  // Efecto para manejar los cambios de estado
  useEffect(() => {
    if (state.status === 'failed') {
      toast({
        type: 'error',
        description: 'Invalid credentials!',
      });
    } else if (state.status === 'success') {
      setIsSuccessful(true);
      router.refresh();
    }
  }, [state.status, router]);

  // Función que maneja el envío del formulario
  const handleSubmit = (formData: FormData) => {
    setEmail(formData.get('email') as string);
    formAction(formData);  // Aquí se llama a la Server Action
  };

  return (
    <div className="flex h-dvh w-screen items-center justify-center">
      <AuthForm action={handleSubmit} defaultEmail={email}>
        <SubmitButton isSuccessful={isSuccessful}>Sign in</SubmitButton>
      </AuthForm>
    </div>
  );
}
```

#### 3. Componente de formulario `components/auth-form.tsx`:
```typescript
import Form from 'next/form';
import { Input } from './ui/input';
import { Label } from './ui/label';

export function AuthForm({
  action,
  children,
  defaultEmail = '',
}: {
  action: (formData: FormData) => void | Promise<void>;
  children: React.ReactNode;
  defaultEmail?: string;
}) {
  return (
    <Form action={action} className="flex flex-col gap-4">
      <div className="flex flex-col gap-2">
        <Label htmlFor="email">Email Address</Label>
        <Input
          id="email"
          name="email"
          type="email"
          required
          defaultValue={defaultEmail}
        />
      </div>
      <div className="flex flex-col gap-2">
        <Label htmlFor="password">Password</Label>
        <Input
          id="password"
          name="password"
          type="password"
          required
        />
      </div>
      {children}
    </Form>
  );
}
```

#### 4. Flujo completo:
1. El usuario introduce su email y contraseña en el formulario.
2. Al hacer clic en "Sign in", se llama a `handleSubmit`.
3. `handleSubmit` llama a `formAction` con los datos del formulario.
4. `formAction` es una versión vinculada de la Server Action `login`.
5. La Server Action `login` se ejecuta en el servidor:
   - Valida los datos con Zod
   - Intenta iniciar sesión con NextAuth
   - Devuelve un estado (`success`, `failed`, etc.)
6. El componente recibe el estado y actúa en consecuencia:
   - Muestra un mensaje de error si falla
   - Redirige al usuario si tiene éxito

### Ejemplo 2: Actualización de visibilidad de chat

#### 1. Definición de la acción en `app/(chat)/actions.ts`:
```typescript
'use server';

import { updateChatVisiblityById } from '@/lib/db/queries';
import { VisibilityType } from '@/components/visibility-selector';

export async function updateChatVisibility({
  chatId,
  visibility,
}: {
  chatId: string;
  visibility: VisibilityType;
}) {
  await updateChatVisiblityById({ chatId, visibility });
}
```

#### 2. Uso en un componente de cliente `components/chat-settings.tsx`:
```typescript
'use client';

import { useState } from 'react';
import { VisibilitySelector, VisibilityType } from './visibility-selector';
import { updateChatVisibility } from '@/app/(chat)/actions';

export function ChatSettings({ chatId, initialVisibility }: { 
  chatId: string;
  initialVisibility: VisibilityType;
}) {
  const [visibility, setVisibility] = useState<VisibilityType>(initialVisibility);
  
  // Función que maneja el cambio de visibilidad
  const handleVisibilityChange = async (newVisibility: VisibilityType) => {
    setVisibility(newVisibility);
    
    // Llamada directa a la Server Action
    await updateChatVisibility({ 
      chatId, 
      visibility: newVisibility 
    });
  };
  
  return (
    <div className="p-4 border rounded">
      <h3 className="text-lg font-medium">Chat Settings</h3>
      <div className="mt-4">
        <label className="block text-sm font-medium">Visibility</label>
        <VisibilitySelector 
          value={visibility} 
          onChange={handleVisibilityChange} 
        />
      </div>
    </div>
  );
}
```

#### 3. Flujo completo:
1. El componente `ChatSettings` se renderiza con la visibilidad inicial.
2. El usuario selecciona una nueva opción de visibilidad.
3. Se llama a `handleVisibilityChange` con la nueva visibilidad.
4. `handleVisibilityChange` actualiza el estado local y llama directamente a la Server Action.
5. La Server Action `updateChatVisibility` se ejecuta en el servidor:
   - Llama a la función de base de datos para actualizar la visibilidad
6. La operación se completa en el servidor sin necesidad de una respuesta explícita.

### Ejemplo 3: Generación de título para un chat

#### 1. Definición de la acción en `app/(chat)/actions.ts`:
```typescript
'use server';

import { generateText, Message } from 'ai';
import { myProvider } from '@/lib/ai/providers';

export async function generateTitleFromUserMessage({
  message,
}: {
  message: Message;
}) {
  const { text: title } = await generateText({
    model: myProvider.languageModel('title-model'),
    system: `Generate a short title based on the user's message`,
    prompt: JSON.stringify(message),
  });

  return title;
}
```

#### 2. Uso en un componente de cliente `components/new-chat.tsx`:
```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { Message } from 'ai';
import { generateTitleFromUserMessage } from '@/app/(chat)/actions';

export function NewChat() {
  const router = useRouter();
  const [message, setMessage] = useState('');
  const [isGenerating, setIsGenerating] = useState(false);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!message.trim()) return;
    
    setIsGenerating(true);
    
    try {
      // Crear objeto de mensaje
      const userMessage: Message = {
        id: Date.now().toString(),
        role: 'user',
        content: message,
      };
      
      // Llamar a la Server Action para generar un título
      const title = await generateTitleFromUserMessage({ message: userMessage });
      
      // Usar el título generado (por ejemplo, para crear un nuevo chat)
      console.log(`Chat creado con título: ${title}`);
      
      // Redirigir o actualizar la UI
      router.push(`/chat/${title}`);
    } catch (error) {
      console.error('Error al generar título:', error);
    } finally {
      setIsGenerating(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="flex flex-col gap-4">
      <textarea
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Escribe tu mensaje..."
        className="p-2 border rounded"
      />
      <button 
        type="submit" 
        disabled={isGenerating}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        {isGenerating ? 'Generando...' : 'Enviar'}
      </button>
    </form>
  );
}
```

#### 3. Flujo completo:
1. El usuario escribe un mensaje en el textarea.
2. Al hacer clic en "Enviar", se llama a `handleSubmit`.
3. `handleSubmit` crea un objeto de mensaje y llama a la Server Action.
4. La Server Action `generateTitleFromUserMessage` se ejecuta en el servidor:
   - Utiliza la API de IA para generar un título basado en el mensaje
   - Devuelve el título generado
5. El componente recibe el título y lo utiliza para crear un nuevo chat.

### Consideraciones importantes

1. **Serialización**: Todos los datos que se pasan a las Server Actions deben ser serializables. Por ejemplo, no se pueden pasar funciones o clases complejas.

2. **Progresividad**: Las Server Actions funcionan incluso si JavaScript está deshabilitado en el cliente cuando se usan con `<form>`, proporcionando una experiencia progresiva.

3. **Optimizaciones automáticas**: Next.js optimiza automáticamente las Server Actions:
   - Genera un ID único para cada acción
   - Crea un endpoint temporal para la acción
   - Maneja la serialización/deserialización de datos

4. **Seguridad**: Las Server Actions son seguras porque:
   - El código sensible nunca se envía al cliente
   - Las validaciones siempre se ejecutan en el servidor
   - Se pueden implementar comprobaciones de CSRF automáticamente

### Ventajas de este enfoque

1. **Seguridad**: El código sensible se ejecuta solo en el servidor.
2. **Rendimiento**: Reduce la cantidad de JavaScript enviado al cliente.
3. **Simplicidad**: No requiere configurar endpoints API separados.
4. **Tipado**: Proporciona tipado completo entre cliente y servidor.
5. **Progresividad**: Funciona incluso sin JavaScript en el cliente (para formularios).

### Consideraciones técnicas

1. **Serialización**: Los datos pasados a y desde Server Actions deben ser serializables.
2. **Validación**: Es importante validar los datos de entrada en el servidor (usando Zod en este caso).
3. **Idempotencia**: Las acciones deben diseñarse para ser idempotentes cuando sea posible.
4. **Tamaño del paquete**: Las acciones contribuyen al tamaño del paquete de JavaScript del cliente.

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
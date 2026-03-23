# React Native com Expo — Referência Completa

Referência consolidada sobre React Native com Expo SDK 52+, cobrindo fundamentos, navegação com Expo Router, gerenciamento de estado global, integração com backend, telas de lista e formulários, internacionalização, autenticação JWT com controle de acesso por role e distribuição para Android e iOS.

---

## Sumário

1. [Fundamentos do React Native com Expo](#1-fundamentos-do-react-native-com-expo)
   - [Configuração do Projeto](#configuração-do-projeto)
   - [Estrutura de Pastas](#estrutura-de-pastas)
   - [Componentes Essenciais](#componentes-essenciais)
   - [Estilização com StyleSheet e NativeWind](#estilização-com-stylesheet-e-nativewind)
2. [Navegação com Expo Router](#2-navegação-com-expo-router)
   - [Conceito de File-Based Routing](#conceito-de-file-based-routing)
   - [Layouts e Grupos de Rotas](#layouts-e-grupos-de-rotas)
   - [Parâmetros e Navegação Programática](#parâmetros-e-navegação-programática)
   - [Rotas Protegidas (Auth Guard)](#rotas-protegidas-auth-guard)
3. [Estado Global com Zustand](#3-estado-global-com-zustand)
   - [Conceitos e Configuração](#conceitos-e-configuração)
   - [Store de Autenticação](#store-de-autenticação)
   - [Persistência com AsyncStorage](#persistência-com-asyncstorage)
4. [Integração com Backend](#4-integração-com-backend)
   - [Configuração do Axios](#configuração-do-axios)
   - [Interceptors e Refresh de Token](#interceptors-e-refresh-de-token)
   - [TanStack Query para Cache e Sincronização](#tanstack-query-para-cache-e-sincronização)
   - [Repository Pattern](#repository-pattern)
5. [Telas de Lista e Formulários](#5-telas-de-lista-e-formulários)
   - [Listas com FlatList e Paginação](#listas-com-flatlist-e-paginação)
   - [Formulários com React Hook Form e Zod](#formulários-com-react-hook-form-e-zod)
   - [Feedback Visual ao Usuário](#feedback-visual-ao-usuário)
6. [Internacionalização (i18n)](#6-internacionalização-i18n)
   - [Configuração do i18next](#configuração-do-i18next)
   - [Traduções e Troca de Idioma](#traduções-e-troca-de-idioma)
   - [Mensagens de Validação Localizadas](#mensagens-de-validação-localizadas)
7. [Autenticação JWT e Controle de Acesso por Role](#7-autenticação-jwt-e-controle-de-acesso-por-role)
   - [Armazenamento Seguro do Token](#armazenamento-seguro-do-token)
   - [Decodificação e Roles do JWT](#decodificação-e-roles-do-jwt)
   - [Fluxo Completo de Autenticação](#fluxo-completo-de-autenticação)
   - [Guard de Rotas por Role](#guard-de-rotas-por-role)
8. [Compilação e Distribuição](#8-compilação-e-distribuição)
   - [EAS Build — Configuração Inicial](#eas-build--configuração-inicial)
   - [Build para Android](#build-para-android)
   - [Publicação na Google Play Store](#publicação-na-google-play-store)
   - [Build para iOS](#build-para-ios)
   - [Publicação na Apple App Store](#publicação-na-apple-app-store)
   - [OTA Updates com EAS Update](#ota-updates-com-eas-update)

---

## 1. Fundamentos do React Native com Expo

### Configuração do Projeto

**Pré-requisitos:**
- Node.js 18+
- Expo CLI (`npm install -g expo-cli`)
- EAS CLI (`npm install -g eas-cli`)
- Para iOS: macOS com Xcode 15+
- Para Android: Android Studio com SDK 34+

**Criar novo projeto com Expo SDK 52:**

```bash
# Criar projeto com template TypeScript
npx create-expo-app@latest meu-app --template

# Escolher: "Navigation (TypeScript)" — inclui Expo Router pré-configurado

cd meu-app

# Instalar dependências base do projeto
npx expo install expo-secure-store expo-constants expo-device

# Iniciar servidor de desenvolvimento
npx expo start
```

**Instalar todas as dependências da referência:**

```bash
# Navegação (já inclusa no template, confirmar versão)
npx expo install expo-router

# Estado global
npm install zustand

# Persistência
npx expo install @react-native-async-storage/async-storage

# HTTP e cache
npm install axios
npm install @tanstack/react-query

# Formulários e validação
npm install react-hook-form zod @hookform/resolvers

# i18n
npm install i18next react-i18next
npx expo install expo-localization

# JWT decode
npm install jwt-decode

# UI Components (opcional mas recomendado)
npm install @rneui/themed @rneui/base
# OU
npm install react-native-paper
```

---

### Estrutura de Pastas

O Expo Router exige que as rotas fiquem dentro de `app/`. A estrutura recomendada:

```
meu-app/
├── app/                          # Rotas (Expo Router)
│   ├── _layout.tsx               # Layout raiz (providers globais)
│   ├── index.tsx                 # Tela inicial (redireciona conforme auth)
│   ├── (auth)/                   # Grupo: telas públicas
│   │   ├── _layout.tsx
│   │   ├── login.tsx
│   │   └── register.tsx
│   ├── (app)/                    # Grupo: telas protegidas
│   │   ├── _layout.tsx           # Tab Navigator + Auth Guard
│   │   ├── home.tsx
│   │   ├── profile.tsx
│   │   └── (admin)/              # Sub-grupo: apenas ROLE_ADMIN
│   │       ├── _layout.tsx
│   │       └── dashboard.tsx
│   └── +not-found.tsx
├── src/
│   ├── components/               # Componentes reutilizáveis
│   │   ├── common/
│   │   └── forms/
│   ├── hooks/                    # Custom hooks
│   ├── services/                 # Comunicação com API
│   │   ├── api.ts                # Instância Axios
│   │   └── repositories/
│   ├── stores/                   # Zustand stores
│   ├── i18n/                     # Configuração i18next
│   │   └── locales/
│   │       ├── pt-BR.json
│   │       └── en-US.json
│   ├── types/                    # TypeScript types/interfaces
│   └── utils/                    # Funções utilitárias
├── assets/
├── app.json
└── package.json
```

---

### Componentes Essenciais

React Native usa componentes nativos em vez de elementos HTML:

```tsx
// src/components/common/Card.tsx
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ViewStyle,
} from 'react-native';

interface CardProps {
  title: string;
  subtitle?: string;
  onPress?: () => void;
  style?: ViewStyle;
  children?: React.ReactNode;
}

export function Card({ title, subtitle, onPress, style, children }: CardProps) {
  const Container = onPress ? TouchableOpacity : View;

  return (
    <Container style={[styles.card, style]} onPress={onPress} activeOpacity={0.8}>
      <Text style={styles.title}>{title}</Text>
      {subtitle && <Text style={styles.subtitle}>{subtitle}</Text>}
      {children}
    </Container>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginVertical: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3, // Android
  },
  title: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1a1a1a',
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
});
```

**Componentes nativos mais usados:**

| Componente         | Equivalente Web      | Uso Principal                    |
|--------------------|----------------------|----------------------------------|
| `View`             | `div`                | Container, layouts               |
| `Text`             | `p`, `span`, `h1`   | Todo texto visível               |
| `Image`            | `img`                | Imagens locais e remotas         |
| `TextInput`        | `input`              | Campos de texto                  |
| `TouchableOpacity` | `button` (com estilo)| Botões e áreas clicáveis         |
| `Pressable`        | `button`             | Botões com estados de pressão    |
| `ScrollView`       | `div` com overflow   | Scroll simples                   |
| `FlatList`         | `ul` virtualizado    | Listas performáticas             |
| `Modal`            | `dialog`             | Modais nativos                   |
| `ActivityIndicator`| spinner CSS          | Indicadores de carregamento      |

---

### Estilização com StyleSheet e NativeWind

**StyleSheet nativo (padrão):**

```tsx
import { StyleSheet, Platform } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 16,
  },
  row: {
    flexDirection: 'row',     // padrão no RN é 'column'
    alignItems: 'center',
    justifyContent: 'space-between',
  },
  // Estilo condicional por plataforma
  shadow: {
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.15,
        shadowRadius: 6,
      },
      android: {
        elevation: 4,
      },
    }),
  },
});
```

**NativeWind (Tailwind CSS para React Native) — alternativa popular:**

```bash
npm install nativewind tailwindcss
npx tailwindcss init
```

```js
// tailwind.config.js
module.exports = {
  content: ['./app/**/*.{js,jsx,ts,tsx}', './src/**/*.{js,jsx,ts,tsx}'],
  presets: [require('nativewind/preset')],
  theme: { extend: {} },
  plugins: [],
};
```

```tsx
// Com NativeWind — classes Tailwind diretamente
import { View, Text } from 'react-native';

export function Header() {
  return (
    <View className="flex-row items-center justify-between px-4 py-3 bg-white shadow-sm">
      <Text className="text-xl font-bold text-gray-900">Meu App</Text>
    </View>
  );
}
```

---

## 2. Navegação com Expo Router

### Conceito de File-Based Routing

O Expo Router v4 usa **file-based routing** — o nome do arquivo define a rota, similar ao Next.js:

| Arquivo                    | Rota            |
|----------------------------|-----------------|
| `app/index.tsx`            | `/`             |
| `app/home.tsx`             | `/home`         |
| `app/profile/[id].tsx`     | `/profile/123`  |
| `app/(auth)/login.tsx`     | `/login`        |
| `app/(app)/home.tsx`       | `/home`         |

> Pastas entre parênteses `(nome)` são **grupos de rota** — organizam arquivos sem afetar a URL.

---

### Layouts e Grupos de Rotas

**Layout raiz — providers globais:**

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { I18nextProvider } from 'react-i18next';
import i18n from '@/src/i18n';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
      staleTime: 1000 * 60 * 5, // 5 minutos
    },
  },
});

export default function RootLayout() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <QueryClientProvider client={queryClient}>
        <I18nextProvider i18n={i18n}>
          <Stack screenOptions={{ headerShown: false }} />
        </I18nextProvider>
      </QueryClientProvider>
    </GestureHandlerRootView>
  );
}
```

**Grupo de autenticação:**

```tsx
// app/(auth)/_layout.tsx
import { Stack } from 'expo-router';

export default function AuthLayout() {
  return (
    <Stack>
      <Stack.Screen name="login" options={{ title: 'Entrar' }} />
      <Stack.Screen name="register" options={{ title: 'Cadastrar' }} />
    </Stack>
  );
}
```

**Tab Navigator para área logada:**

```tsx
// app/(app)/_layout.tsx
import { Tabs, Redirect } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';
import { useAuthStore } from '@/src/stores/authStore';

export default function AppLayout() {
  const { isAuthenticated } = useAuthStore();

  if (!isAuthenticated) {
    return <Redirect href="/login" />;
  }

  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#6366f1',
        tabBarInactiveTintColor: '#9ca3af',
        headerShown: true,
      }}
    >
      <Tabs.Screen
        name="home"
        options={{
          title: 'Início',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home-outline" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Perfil',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person-outline" size={size} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

---

### Parâmetros e Navegação Programática

```tsx
// app/(app)/products/[id].tsx
import { useLocalSearchParams, useRouter } from 'expo-router';
import { View, Text, TouchableOpacity } from 'react-native';

export default function ProductScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const router = useRouter();

  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Text>Produto ID: {id}</Text>

      {/* Navegar para trás */}
      <TouchableOpacity onPress={() => router.back()}>
        <Text>Voltar</Text>
      </TouchableOpacity>

      {/* Substituir tela atual */}
      <TouchableOpacity onPress={() => router.replace('/home')}>
        <Text>Ir para Home (sem histórico)</Text>
      </TouchableOpacity>
    </View>
  );
}
```

**Navegação tipada — utilitário recomendado:**

```tsx
// src/utils/navigation.ts
import { router } from 'expo-router';

export const navigate = {
  toLogin: () => router.replace('/(auth)/login'),
  toHome: () => router.replace('/(app)/home'),
  toProduct: (id: string) => router.push(`/(app)/products/${id}`),
  back: () => router.back(),
};
```

---

### Rotas Protegidas (Auth Guard)

```tsx
// src/components/common/ProtectedRoute.tsx
import { useEffect } from 'react';
import { router } from 'expo-router';
import { useAuthStore } from '@/src/stores/authStore';
import { LoadingScreen } from './LoadingScreen';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRole?: string;
}

export function ProtectedRoute({ children, requiredRole }: ProtectedRouteProps) {
  const { isAuthenticated, user, isLoading } = useAuthStore();

  useEffect(() => {
    if (!isLoading && !isAuthenticated) {
      router.replace('/(auth)/login');
    }
  }, [isAuthenticated, isLoading]);

  if (isLoading) return <LoadingScreen />;
  if (!isAuthenticated) return null;

  if (requiredRole && !user?.roles.includes(requiredRole)) {
    return <UnauthorizedScreen />;
  }

  return <>{children}</>;
}
```

---

## 3. Estado Global com Zustand

### Conceitos e Configuração

**Zustand** é a opção mais popular para estado global em React Native por ser simples, leve (~1KB) e sem boilerplate. Alternativas: Redux Toolkit (mais complexo, mas amplo ecossistema) e Jotai (atômico).

```bash
npm install zustand
```

**Store básica:**

```tsx
// src/stores/counterStore.ts
import { create } from 'zustand';

interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```

---

### Store de Autenticação

```tsx
// src/stores/authStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';
import * as SecureStore from 'expo-secure-store';
import { jwtDecode } from 'jwt-decode';

export interface User {
  id: string;
  name: string;
  email: string;
  roles: string[];
}

interface JwtPayload {
  sub: string;
  name: string;
  email: string;
  roles: string[];
  exp: number;
}

interface AuthState {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  // Actions
  login: (accessToken: string, refreshToken: string) => void;
  logout: () => void;
  setTokens: (accessToken: string, refreshToken: string) => void;
  initializeAuth: () => Promise<void>;
  hasRole: (role: string) => boolean;
  hasAnyRole: (roles: string[]) => boolean;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      isAuthenticated: false,
      isLoading: true,

      login: (accessToken: string, refreshToken: string) => {
        const payload = jwtDecode<JwtPayload>(accessToken);
        const user: User = {
          id: payload.sub,
          name: payload.name,
          email: payload.email,
          roles: payload.roles || [],
        };

        // Armazenar tokens de forma segura
        SecureStore.setItemAsync('accessToken', accessToken);
        SecureStore.setItemAsync('refreshToken', refreshToken);

        set({ user, accessToken, refreshToken, isAuthenticated: true });
      },

      logout: () => {
        SecureStore.deleteItemAsync('accessToken');
        SecureStore.deleteItemAsync('refreshToken');
        set({ user: null, accessToken: null, refreshToken: null, isAuthenticated: false });
      },

      setTokens: (accessToken: string, refreshToken: string) => {
        SecureStore.setItemAsync('accessToken', accessToken);
        SecureStore.setItemAsync('refreshToken', refreshToken);
        set({ accessToken, refreshToken });
      },

      initializeAuth: async () => {
        try {
          const accessToken = await SecureStore.getItemAsync('accessToken');
          const refreshToken = await SecureStore.getItemAsync('refreshToken');

          if (accessToken) {
            const payload = jwtDecode<JwtPayload>(accessToken);
            const isExpired = payload.exp * 1000 < Date.now();

            if (!isExpired) {
              const user: User = {
                id: payload.sub,
                name: payload.name,
                email: payload.email,
                roles: payload.roles || [],
              };
              set({ user, accessToken, refreshToken, isAuthenticated: true });
            } else {
              // Token expirado: limpar
              get().logout();
            }
          }
        } catch {
          get().logout();
        } finally {
          set({ isLoading: false });
        }
      },

      hasRole: (role: string) => {
        const { user } = get();
        return user?.roles.includes(role) ?? false;
      },

      hasAnyRole: (roles: string[]) => {
        const { user } = get();
        return roles.some((role) => user?.roles.includes(role)) ?? false;
      },
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => AsyncStorage),
      // Não persistir tokens no AsyncStorage (usar SecureStore para isso)
      partialize: (state) => ({ user: state.user }),
    }
  )
);
```

---

### Persistência com AsyncStorage

```tsx
// src/stores/settingsStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface SettingsState {
  language: string;
  theme: 'light' | 'dark' | 'system';
  notificationsEnabled: boolean;
  setLanguage: (lang: string) => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  toggleNotifications: () => void;
}

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      language: 'pt-BR',
      theme: 'system',
      notificationsEnabled: true,
      setLanguage: (language) => set({ language }),
      setTheme: (theme) => set({ theme }),
      toggleNotifications: () =>
        set((state) => ({ notificationsEnabled: !state.notificationsEnabled })),
    }),
    {
      name: 'settings-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

**Inicializar auth ao abrir o app:**

```tsx
// app/_layout.tsx (atualizado)
import { useEffect } from 'react';
import { useAuthStore } from '@/src/stores/authStore';

export default function RootLayout() {
  const initializeAuth = useAuthStore((state) => state.initializeAuth);

  useEffect(() => {
    initializeAuth();
  }, []);

  // ... resto do layout
}
```

---

## 4. Integração com Backend

### Configuração do Axios

```tsx
// src/services/api.ts
import axios, { AxiosInstance, InternalAxiosRequestConfig } from 'axios';
import Constants from 'expo-constants';

const BASE_URL = Constants.expoConfig?.extra?.apiUrl ?? 'http://localhost:8080/api';

export const api: AxiosInstance = axios.create({
  baseURL: BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json',
  },
});
```

**`app.json` — configurar a URL da API por ambiente:**

```json
{
  "expo": {
    "extra": {
      "apiUrl": "https://api.meuapp.com"
    }
  }
}
```

---

### Interceptors e Refresh de Token

```tsx
// src/services/interceptors.ts
import { AxiosError, InternalAxiosRequestConfig } from 'axios';
import { api } from './api';
import { useAuthStore } from '@/src/stores/authStore';

let isRefreshing = false;
let failedQueue: Array<{ resolve: Function; reject: Function }> = [];

const processQueue = (error: AxiosError | null, token: string | null = null) => {
  failedQueue.forEach(({ resolve, reject }) => {
    if (error) reject(error);
    else resolve(token);
  });
  failedQueue = [];
};

// Interceptor de requisição — adiciona Authorization header
api.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const { accessToken } = useAuthStore.getState();
    if (accessToken) {
      config.headers.Authorization = `Bearer ${accessToken}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Interceptor de resposta — trata 401 e faz refresh
api.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & { _retry?: boolean };

    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then((token) => {
          originalRequest.headers.Authorization = `Bearer ${token}`;
          return api(originalRequest);
        });
      }

      originalRequest._retry = true;
      isRefreshing = true;

      const { refreshToken, setTokens, logout } = useAuthStore.getState();

      try {
        const response = await api.post('/auth/refresh', { refreshToken });
        const { accessToken: newToken, refreshToken: newRefreshToken } = response.data;

        setTokens(newToken, newRefreshToken);
        processQueue(null, newToken);

        originalRequest.headers.Authorization = `Bearer ${newToken}`;
        return api(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError as AxiosError, null);
        logout();
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);
```

**Inicializar interceptors no layout raiz:**

```tsx
// app/_layout.tsx
import '@/src/services/interceptors'; // importar para registrar
```

---

### TanStack Query para Cache e Sincronização

O **TanStack Query** (React Query) é a melhor opção para gerenciar dados do servidor: cache automático, sincronização em background, estados de loading/error e paginação.

```tsx
// src/hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient, useInfiniteQuery } from '@tanstack/react-query';
import { productRepository } from '@/src/services/repositories/productRepository';

// Buscar lista (com paginação infinita)
export function useProducts() {
  return useInfiniteQuery({
    queryKey: ['products'],
    queryFn: ({ pageParam = 0 }) =>
      productRepository.findAll({ page: pageParam, size: 20 }),
    getNextPageParam: (lastPage) =>
      lastPage.last ? undefined : lastPage.number + 1,
    initialPageParam: 0,
  });
}

// Buscar por ID
export function useProduct(id: string) {
  return useQuery({
    queryKey: ['products', id],
    queryFn: () => productRepository.findById(id),
    enabled: !!id,
  });
}

// Criar produto
export function useCreateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: productRepository.create,
    onSuccess: () => {
      // Invalidar cache para refetch automático
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}

// Atualizar produto
export function useUpdateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<Product> }) =>
      productRepository.update(id, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
      queryClient.invalidateQueries({ queryKey: ['products', id] });
    },
  });
}
```

---

### Repository Pattern

```tsx
// src/types/product.ts
export interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
  active: boolean;
}

export interface PageResponse<T> {
  content: T[];
  number: number;
  size: number;
  totalElements: number;
  totalPages: number;
  last: boolean;
}

export interface CreateProductDTO {
  name: string;
  price: number;
  category: string;
}
```

```tsx
// src/services/repositories/productRepository.ts
import { api } from '../api';
import { Product, PageResponse, CreateProductDTO } from '@/src/types/product';

interface FindAllParams {
  page?: number;
  size?: number;
  search?: string;
}

export const productRepository = {
  findAll: async (params: FindAllParams): Promise<PageResponse<Product>> => {
    const { data } = await api.get('/products', { params });
    return data;
  },

  findById: async (id: string): Promise<Product> => {
    const { data } = await api.get(`/products/${id}`);
    return data;
  },

  create: async (dto: CreateProductDTO): Promise<Product> => {
    const { data } = await api.post('/products', dto);
    return data;
  },

  update: async (id: string, dto: Partial<CreateProductDTO>): Promise<Product> => {
    const { data } = await api.put(`/products/${id}`, dto);
    return data;
  },

  delete: async (id: string): Promise<void> => {
    await api.delete(`/products/${id}`);
  },
};
```

---

## 5. Telas de Lista e Formulários

### Listas com FlatList e Paginação

```tsx
// app/(app)/products/index.tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  FlatList,
  ActivityIndicator,
  TextInput,
  StyleSheet,
  RefreshControl,
} from 'react-native';
import { useRouter } from 'expo-router';
import { useProducts } from '@/src/hooks/useProducts';
import { Product } from '@/src/types/product';
import { ProductCard } from '@/src/components/ProductCard';

export default function ProductsScreen() {
  const router = useRouter();
  const [search, setSearch] = useState('');

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    isError,
    refetch,
    isRefetching,
  } = useProducts();

  // Achatar páginas em lista única
  const products = data?.pages.flatMap((page) => page.content) ?? [];

  const renderItem = ({ item }: { item: Product }) => (
    <ProductCard
      product={item}
      onPress={() => router.push(`/(app)/products/${item.id}`)}
    />
  );

  const renderFooter = () => {
    if (!isFetchingNextPage) return null;
    return (
      <View style={styles.footer}>
        <ActivityIndicator size="small" color="#6366f1" />
      </View>
    );
  };

  if (isLoading) {
    return (
      <View style={styles.centered}>
        <ActivityIndicator size="large" color="#6366f1" />
      </View>
    );
  }

  if (isError) {
    return (
      <View style={styles.centered}>
        <Text style={styles.errorText}>Erro ao carregar produtos.</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.searchInput}
        placeholder="Buscar produtos..."
        value={search}
        onChangeText={setSearch}
        returnKeyType="search"
      />

      <FlatList
        data={products}
        keyExtractor={(item) => item.id}
        renderItem={renderItem}
        ListFooterComponent={renderFooter}
        onEndReached={() => {
          if (hasNextPage && !isFetchingNextPage) fetchNextPage();
        }}
        onEndReachedThreshold={0.5}
        refreshControl={
          <RefreshControl refreshing={isRefetching} onRefresh={refetch} />
        }
        ListEmptyComponent={
          <View style={styles.centered}>
            <Text style={styles.emptyText}>Nenhum produto encontrado.</Text>
          </View>
        }
        contentContainerStyle={products.length === 0 && styles.emptyContainer}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f5f5f5' },
  centered: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  emptyContainer: { flexGrow: 1 },
  searchInput: {
    margin: 16,
    padding: 12,
    backgroundColor: '#fff',
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#e5e7eb',
    fontSize: 16,
  },
  footer: { padding: 16, alignItems: 'center' },
  errorText: { fontSize: 16, color: '#ef4444' },
  emptyText: { fontSize: 16, color: '#9ca3af' },
});
```

---

### Formulários com React Hook Form e Zod

```tsx
// src/schemas/productSchema.ts
import { z } from 'zod';

export const productSchema = z.object({
  name: z
    .string()
    .min(3, 'Nome deve ter no mínimo 3 caracteres')
    .max(100, 'Nome muito longo'),
  price: z
    .number({ invalid_type_error: 'Preço deve ser um número' })
    .positive('Preço deve ser maior que zero')
    .multipleOf(0.01, 'Máximo 2 casas decimais'),
  category: z.string().min(1, 'Selecione uma categoria'),
});

export type ProductFormData = z.infer<typeof productSchema>;
```

```tsx
// app/(app)/products/new.tsx
import React from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
  Alert,
  KeyboardAvoidingView,
  Platform,
} from 'react-native';
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { useRouter } from 'expo-router';
import { productSchema, ProductFormData } from '@/src/schemas/productSchema';
import { useCreateProduct } from '@/src/hooks/useProducts';

export default function NewProductScreen() {
  const router = useRouter();
  const createProduct = useCreateProduct();

  const {
    control,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<ProductFormData>({
    resolver: zodResolver(productSchema),
    defaultValues: {
      name: '',
      price: undefined,
      category: '',
    },
  });

  const onSubmit = async (data: ProductFormData) => {
    try {
      await createProduct.mutateAsync(data);
      Alert.alert('Sucesso', 'Produto criado com sucesso!', [
        { text: 'OK', onPress: () => router.back() },
      ]);
    } catch (error) {
      Alert.alert('Erro', 'Não foi possível criar o produto. Tente novamente.');
    }
  };

  return (
    <KeyboardAvoidingView
      style={{ flex: 1 }}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <ScrollView style={styles.container} keyboardShouldPersistTaps="handled">
        <Text style={styles.title}>Novo Produto</Text>

        {/* Campo: Nome */}
        <View style={styles.field}>
          <Text style={styles.label}>Nome *</Text>
          <Controller
            control={control}
            name="name"
            render={({ field: { onChange, onBlur, value } }) => (
              <TextInput
                style={[styles.input, errors.name && styles.inputError]}
                placeholder="Nome do produto"
                onBlur={onBlur}
                onChangeText={onChange}
                value={value}
                autoCapitalize="words"
              />
            )}
          />
          {errors.name && (
            <Text style={styles.errorText}>{errors.name.message}</Text>
          )}
        </View>

        {/* Campo: Preço */}
        <View style={styles.field}>
          <Text style={styles.label}>Preço *</Text>
          <Controller
            control={control}
            name="price"
            render={({ field: { onChange, onBlur, value } }) => (
              <TextInput
                style={[styles.input, errors.price && styles.inputError]}
                placeholder="0,00"
                onBlur={onBlur}
                onChangeText={(text) => onChange(parseFloat(text.replace(',', '.')))}
                value={value?.toString()}
                keyboardType="decimal-pad"
              />
            )}
          />
          {errors.price && (
            <Text style={styles.errorText}>{errors.price.message}</Text>
          )}
        </View>

        {/* Campo: Categoria */}
        <View style={styles.field}>
          <Text style={styles.label}>Categoria *</Text>
          <Controller
            control={control}
            name="category"
            render={({ field: { onChange, onBlur, value } }) => (
              <TextInput
                style={[styles.input, errors.category && styles.inputError]}
                placeholder="Categoria"
                onBlur={onBlur}
                onChangeText={onChange}
                value={value}
              />
            )}
          />
          {errors.category && (
            <Text style={styles.errorText}>{errors.category.message}</Text>
          )}
        </View>

        <TouchableOpacity
          style={[styles.button, isSubmitting && styles.buttonDisabled]}
          onPress={handleSubmit(onSubmit)}
          disabled={isSubmitting || createProduct.isPending}
        >
          <Text style={styles.buttonText}>
            {createProduct.isPending ? 'Salvando...' : 'Salvar Produto'}
          </Text>
        </TouchableOpacity>
      </ScrollView>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#fff' },
  title: { fontSize: 24, fontWeight: 'bold', marginBottom: 24, color: '#1a1a1a' },
  field: { marginBottom: 16 },
  label: { fontSize: 14, fontWeight: '600', marginBottom: 6, color: '#374151' },
  input: {
    borderWidth: 1,
    borderColor: '#d1d5db',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    backgroundColor: '#f9fafb',
  },
  inputError: { borderColor: '#ef4444' },
  errorText: { color: '#ef4444', fontSize: 12, marginTop: 4 },
  button: {
    backgroundColor: '#6366f1',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 8,
    marginBottom: 32,
  },
  buttonDisabled: { backgroundColor: '#a5b4fc' },
  buttonText: { color: '#fff', fontSize: 16, fontWeight: '600' },
});
```

---

### Feedback Visual ao Usuário

**Toast notifications com `react-native-toast-message`:**

```bash
npm install react-native-toast-message
```

```tsx
// app/_layout.tsx — adicionar Toast ao final
import Toast from 'react-native-toast-message';

export default function RootLayout() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      {/* ... providers */}
      <Stack screenOptions={{ headerShown: false }} />
      <Toast /> {/* deve ser o último elemento */}
    </GestureHandlerRootView>
  );
}
```

```tsx
// src/utils/toast.ts
import Toast from 'react-native-toast-message';

export const showToast = {
  success: (message: string, title?: string) =>
    Toast.show({ type: 'success', text1: title ?? 'Sucesso', text2: message }),

  error: (message: string, title?: string) =>
    Toast.show({ type: 'error', text1: title ?? 'Erro', text2: message }),

  info: (message: string, title?: string) =>
    Toast.show({ type: 'info', text1: title ?? 'Aviso', text2: message }),
};

// Uso:
// showToast.success('Produto criado com sucesso!');
// showToast.error('Erro ao conectar com o servidor');
```

**Skeleton Loading com `react-native-skeleton-placeholder`:**

```bash
npm install react-native-skeleton-placeholder
```

```tsx
// src/components/ProductCardSkeleton.tsx
import React from 'react';
import SkeletonPlaceholder from 'react-native-skeleton-placeholder';

export function ProductCardSkeleton() {
  return (
    <SkeletonPlaceholder borderRadius={8}>
      <SkeletonPlaceholder.Item padding={16} marginVertical={4}>
        <SkeletonPlaceholder.Item width={200} height={20} />
        <SkeletonPlaceholder.Item marginTop={8} width={100} height={16} />
      </SkeletonPlaceholder.Item>
    </SkeletonPlaceholder>
  );
}
```

---

## 6. Internacionalização (i18n)

### Configuração do i18next

```bash
npm install i18next react-i18next
npx expo install expo-localization
```

```tsx
// src/i18n/index.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import * as Localization from 'expo-localization';
import ptBR from './locales/pt-BR.json';
import enUS from './locales/en-US.json';

const resources = {
  'pt-BR': { translation: ptBR },
  'en-US': { translation: enUS },
};

// Detectar idioma do dispositivo
const deviceLanguage = Localization.getLocales()[0]?.languageTag ?? 'pt-BR';
const supportedLanguage = Object.keys(resources).includes(deviceLanguage)
  ? deviceLanguage
  : 'pt-BR';

i18n
  .use(initReactI18next)
  .init({
    resources,
    lng: supportedLanguage,
    fallbackLng: 'pt-BR',
    interpolation: {
      escapeValue: false,
    },
  });

export default i18n;
```

---

### Traduções e Troca de Idioma

```json
// src/i18n/locales/pt-BR.json
{
  "common": {
    "save": "Salvar",
    "cancel": "Cancelar",
    "loading": "Carregando...",
    "error": "Erro",
    "success": "Sucesso",
    "back": "Voltar",
    "confirm": "Confirmar",
    "delete": "Excluir"
  },
  "auth": {
    "login": "Entrar",
    "logout": "Sair",
    "email": "E-mail",
    "password": "Senha",
    "forgotPassword": "Esqueceu a senha?",
    "noAccount": "Não tem uma conta? Cadastre-se",
    "loginSuccess": "Login realizado com sucesso!",
    "loginError": "E-mail ou senha inválidos"
  },
  "products": {
    "title": "Produtos",
    "new": "Novo Produto",
    "name": "Nome",
    "price": "Preço",
    "category": "Categoria",
    "noResults": "Nenhum produto encontrado",
    "createSuccess": "Produto criado com sucesso!",
    "createError": "Erro ao criar produto"
  },
  "validation": {
    "required": "Campo obrigatório",
    "minLength": "Mínimo {{count}} caracteres",
    "maxLength": "Máximo {{count}} caracteres",
    "invalidEmail": "E-mail inválido",
    "minValue": "Valor mínimo: {{value}}",
    "passwordMismatch": "As senhas não coincidem"
  }
}
```

```json
// src/i18n/locales/en-US.json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "loading": "Loading...",
    "error": "Error",
    "success": "Success",
    "back": "Back",
    "confirm": "Confirm",
    "delete": "Delete"
  },
  "auth": {
    "login": "Sign In",
    "logout": "Sign Out",
    "email": "Email",
    "password": "Password",
    "forgotPassword": "Forgot password?",
    "noAccount": "No account? Sign up",
    "loginSuccess": "Login successful!",
    "loginError": "Invalid email or password"
  },
  "products": {
    "title": "Products",
    "new": "New Product",
    "name": "Name",
    "price": "Price",
    "category": "Category",
    "noResults": "No products found",
    "createSuccess": "Product created successfully!",
    "createError": "Error creating product"
  },
  "validation": {
    "required": "Required field",
    "minLength": "Minimum {{count}} characters",
    "maxLength": "Maximum {{count}} characters",
    "invalidEmail": "Invalid email",
    "minValue": "Minimum value: {{value}}",
    "passwordMismatch": "Passwords do not match"
  }
}
```

**Uso nos componentes:**

```tsx
import { useTranslation } from 'react-i18next';

export function ProductsScreen() {
  const { t } = useTranslation();

  return (
    <View>
      <Text>{t('products.title')}</Text>
      <Text>{t('validation.minLength', { count: 3 })}</Text>
    </View>
  );
}
```

**Troca de idioma com persistência:**

```tsx
// src/hooks/useLanguage.ts
import { useTranslation } from 'react-i18next';
import { useSettingsStore } from '@/src/stores/settingsStore';

export function useLanguage() {
  const { i18n } = useTranslation();
  const { language, setLanguage } = useSettingsStore();

  const changeLanguage = async (lang: string) => {
    await i18n.changeLanguage(lang);
    setLanguage(lang);
  };

  return { language, changeLanguage };
}
```

---

### Mensagens de Validação Localizadas

```tsx
// src/schemas/productSchema.ts (com i18n)
import { z } from 'zod';
import i18n from '@/src/i18n';

export const createProductSchema = () =>
  z.object({
    name: z
      .string()
      .min(3, i18n.t('validation.minLength', { count: 3 }))
      .max(100, i18n.t('validation.maxLength', { count: 100 })),
    price: z
      .number({ invalid_type_error: i18n.t('validation.required') })
      .positive(i18n.t('validation.minValue', { value: 0 })),
    category: z.string().min(1, i18n.t('validation.required')),
  });

// No formulário, recriar o schema ao mudar idioma:
const { i18n } = useTranslation();
const schema = useMemo(() => createProductSchema(), [i18n.language]);

const { control, handleSubmit } = useForm({
  resolver: zodResolver(schema),
});
```

---

## 7. Autenticação JWT e Controle de Acesso por Role

### Armazenamento Seguro do Token

O **Expo SecureStore** usa o Keychain (iOS) e KeyStore (Android) — armazenamento criptografado nativo:

```tsx
// src/utils/tokenStorage.ts
import * as SecureStore from 'expo-secure-store';

const ACCESS_TOKEN_KEY = 'jwt_access_token';
const REFRESH_TOKEN_KEY = 'jwt_refresh_token';

export const tokenStorage = {
  getAccessToken: () => SecureStore.getItemAsync(ACCESS_TOKEN_KEY),
  getRefreshToken: () => SecureStore.getItemAsync(REFRESH_TOKEN_KEY),

  saveTokens: async (accessToken: string, refreshToken: string) => {
    await Promise.all([
      SecureStore.setItemAsync(ACCESS_TOKEN_KEY, accessToken),
      SecureStore.setItemAsync(REFRESH_TOKEN_KEY, refreshToken),
    ]);
  },

  clearTokens: async () => {
    await Promise.all([
      SecureStore.deleteItemAsync(ACCESS_TOKEN_KEY),
      SecureStore.deleteItemAsync(REFRESH_TOKEN_KEY),
    ]);
  },
};
```

> **Atenção:** `AsyncStorage` não é seguro para tokens — não é criptografado. Use sempre o `expo-secure-store` para dados sensíveis.

---

### Decodificação e Roles do JWT

```tsx
// src/utils/jwt.ts
import { jwtDecode } from 'jwt-decode';

interface JwtPayload {
  sub: string;       // subject (user ID)
  name: string;
  email: string;
  roles: string[];   // ex: ["ROLE_USER", "ROLE_ADMIN"]
  exp: number;       // expiração (unix timestamp)
  iat: number;       // emissão
}

export function decodeToken(token: string): JwtPayload | null {
  try {
    return jwtDecode<JwtPayload>(token);
  } catch {
    return null;
  }
}

export function isTokenExpired(token: string): boolean {
  const payload = decodeToken(token);
  if (!payload) return true;
  return payload.exp * 1000 < Date.now();
}

export function getTokenRoles(token: string): string[] {
  return decodeToken(token)?.roles ?? [];
}
```

---

### Fluxo Completo de Autenticação

**Tela de Login:**

```tsx
// app/(auth)/login.tsx
import React from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  KeyboardAvoidingView,
  Platform,
} from 'react-native';
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useRouter } from 'expo-router';
import { useTranslation } from 'react-i18next';
import { useMutation } from '@tanstack/react-query';
import { useAuthStore } from '@/src/stores/authStore';
import { authRepository } from '@/src/services/repositories/authRepository';
import { showToast } from '@/src/utils/toast';

const loginSchema = z.object({
  email: z.string().email('E-mail inválido'),
  password: z.string().min(6, 'Senha deve ter no mínimo 6 caracteres'),
});

type LoginFormData = z.infer<typeof loginSchema>;

export default function LoginScreen() {
  const { t } = useTranslation();
  const router = useRouter();
  const login = useAuthStore((state) => state.login);

  const { control, handleSubmit, formState: { errors } } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const loginMutation = useMutation({
    mutationFn: authRepository.login,
    onSuccess: ({ accessToken, refreshToken }) => {
      login(accessToken, refreshToken);
      showToast.success(t('auth.loginSuccess'));
      router.replace('/(app)/home');
    },
    onError: () => {
      showToast.error(t('auth.loginError'));
    },
  });

  const onSubmit = (data: LoginFormData) => {
    loginMutation.mutate(data);
  };

  return (
    <KeyboardAvoidingView
      style={styles.container}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <View style={styles.form}>
        <Text style={styles.title}>{t('auth.login')}</Text>

        <Controller
          control={control}
          name="email"
          render={({ field: { onChange, onBlur, value } }) => (
            <TextInput
              style={[styles.input, errors.email && styles.inputError]}
              placeholder={t('auth.email')}
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              keyboardType="email-address"
              autoCapitalize="none"
              autoComplete="email"
            />
          )}
        />
        {errors.email && <Text style={styles.errorText}>{errors.email.message}</Text>}

        <Controller
          control={control}
          name="password"
          render={({ field: { onChange, onBlur, value } }) => (
            <TextInput
              style={[styles.input, errors.password && styles.inputError]}
              placeholder={t('auth.password')}
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              secureTextEntry
              autoComplete="password"
            />
          )}
        />
        {errors.password && <Text style={styles.errorText}>{errors.password.message}</Text>}

        <TouchableOpacity
          style={[styles.button, loginMutation.isPending && styles.buttonDisabled]}
          onPress={handleSubmit(onSubmit)}
          disabled={loginMutation.isPending}
        >
          <Text style={styles.buttonText}>
            {loginMutation.isPending ? t('common.loading') : t('auth.login')}
          </Text>
        </TouchableOpacity>
      </View>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  form: { flex: 1, justifyContent: 'center', padding: 24 },
  title: { fontSize: 28, fontWeight: 'bold', marginBottom: 32, color: '#1a1a1a' },
  input: {
    borderWidth: 1,
    borderColor: '#d1d5db',
    borderRadius: 8,
    padding: 14,
    fontSize: 16,
    marginBottom: 8,
    backgroundColor: '#f9fafb',
  },
  inputError: { borderColor: '#ef4444' },
  errorText: { color: '#ef4444', fontSize: 12, marginBottom: 8 },
  button: {
    backgroundColor: '#6366f1',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 16,
  },
  buttonDisabled: { backgroundColor: '#a5b4fc' },
  buttonText: { color: '#fff', fontSize: 16, fontWeight: '600' },
});
```

**Repository de autenticação:**

```tsx
// src/services/repositories/authRepository.ts
import { api } from '../api';

interface LoginCredentials {
  email: string;
  password: string;
}

interface AuthTokens {
  accessToken: string;
  refreshToken: string;
}

export const authRepository = {
  login: async (credentials: LoginCredentials): Promise<AuthTokens> => {
    const { data } = await api.post('/auth/login', credentials);
    return data;
  },

  refresh: async (refreshToken: string): Promise<AuthTokens> => {
    const { data } = await api.post('/auth/refresh', { refreshToken });
    return data;
  },

  logout: async (): Promise<void> => {
    await api.post('/auth/logout');
  },

  me: async () => {
    const { data } = await api.get('/auth/me');
    return data;
  },
};
```

---

### Guard de Rotas por Role

```tsx
// src/components/common/RoleGuard.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { useAuthStore } from '@/src/stores/authStore';

interface RoleGuardProps {
  roles: string[];             // qualquer um destes roles concede acesso
  children: React.ReactNode;
  fallback?: React.ReactNode;  // renderizado quando não autorizado
}

export function RoleGuard({ roles, children, fallback }: RoleGuardProps) {
  const hasAnyRole = useAuthStore((state) => state.hasAnyRole);

  if (!hasAnyRole(roles)) {
    return fallback ? (
      <>{fallback}</>
    ) : (
      <View style={styles.container}>
        <Text style={styles.title}>Acesso Negado</Text>
        <Text style={styles.subtitle}>
          Você não tem permissão para acessar este conteúdo.
        </Text>
      </View>
    );
  }

  return <>{children}</>;
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', justifyContent: 'center', padding: 24 },
  title: { fontSize: 20, fontWeight: 'bold', color: '#1a1a1a' },
  subtitle: { fontSize: 14, color: '#6b7280', marginTop: 8, textAlign: 'center' },
});
```

**Layout protegido por role:**

```tsx
// app/(app)/(admin)/_layout.tsx
import { Stack, Redirect } from 'expo-router';
import { useAuthStore } from '@/src/stores/authStore';

export default function AdminLayout() {
  const hasRole = useAuthStore((state) => state.hasRole);

  if (!hasRole('ROLE_ADMIN')) {
    return <Redirect href="/(app)/home" />;
  }

  return (
    <Stack>
      <Stack.Screen name="dashboard" options={{ title: 'Painel Administrativo' }} />
    </Stack>
  );
}
```

**Uso do RoleGuard em componentes:**

```tsx
// app/(app)/home.tsx
import { RoleGuard } from '@/src/components/common/RoleGuard';

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text>Bem-vindo!</Text>

      {/* Visível apenas para ROLE_ADMIN */}
      <RoleGuard roles={['ROLE_ADMIN']}>
        <TouchableOpacity onPress={() => router.push('/(app)/(admin)/dashboard')}>
          <Text>Painel Administrativo</Text>
        </TouchableOpacity>
      </RoleGuard>

      {/* Visível para ROLE_MANAGER ou ROLE_ADMIN */}
      <RoleGuard roles={['ROLE_MANAGER', 'ROLE_ADMIN']}>
        <TouchableOpacity>
          <Text>Relatórios</Text>
        </TouchableOpacity>
      </RoleGuard>
    </View>
  );
}
```

**Hook utilitário para uso de autenticação:**

```tsx
// src/hooks/useAuth.ts
import { useAuthStore } from '@/src/stores/authStore';
import { useMutation } from '@tanstack/react-query';
import { authRepository } from '@/src/services/repositories/authRepository';
import { showToast } from '@/src/utils/toast';
import { router } from 'expo-router';

export function useAuth() {
  const { user, isAuthenticated, login, logout, hasRole, hasAnyRole } = useAuthStore();

  const logoutMutation = useMutation({
    mutationFn: authRepository.logout,
    onSettled: () => {
      logout();
      showToast.info('Você saiu da sua conta.');
      router.replace('/(auth)/login');
    },
  });

  return {
    user,
    isAuthenticated,
    hasRole,
    hasAnyRole,
    isAdmin: hasRole('ROLE_ADMIN'),
    isManager: hasAnyRole(['ROLE_ADMIN', 'ROLE_MANAGER']),
    login,
    logout: () => logoutMutation.mutate(),
    isLoggingOut: logoutMutation.isPending,
  };
}
```

---

## 8. Compilação e Distribuição

### EAS Build — Configuração Inicial

O **EAS (Expo Application Services)** é a plataforma oficial da Expo para build na nuvem. Alternativa ao build local, especialmente útil para iOS sem macOS.

```bash
# 1. Instalar EAS CLI
npm install -g eas-cli

# 2. Login na conta Expo
eas login

# 3. Configurar EAS no projeto
eas build:configure
```

**Arquivo `eas.json` gerado:**

```json
{
  "cli": {
    "version": ">= 12.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      },
      "ios": {
        "simulator": false
      }
    },
    "production": {
      "autoIncrement": true,
      "android": {
        "buildType": "app-bundle"
      }
    }
  },
  "submit": {
    "production": {
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "internal"
      },
      "ios": {
        "appleId": "seu@email.com",
        "ascAppId": "1234567890",
        "appleTeamId": "ABCDE12345"
      }
    }
  }
}
```

**`app.json` — configuração da aplicação:**

```json
{
  "expo": {
    "name": "Meu App",
    "slug": "meu-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#6366f1"
    },
    "ios": {
      "supportsTablet": false,
      "bundleIdentifier": "com.empresa.meuapp",
      "buildNumber": "1"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#6366f1"
      },
      "package": "com.empresa.meuapp",
      "versionCode": 1,
      "permissions": []
    },
    "extra": {
      "apiUrl": "https://api.meuapp.com",
      "eas": {
        "projectId": "seu-project-id-aqui"
      }
    }
  }
}
```

---

### Build para Android

**Build de desenvolvimento (APK para testes internos):**

```bash
# Gerar APK para testes locais
eas build --platform android --profile development

# Build local (requer Android Studio configurado)
eas build --platform android --profile development --local
```

**Build de produção (AAB para Google Play):**

```bash
# Build na nuvem (recomendado)
eas build --platform android --profile production

# Acompanhar o status
eas build:list
```

**Configurar assinatura automática no EAS:**

```bash
# EAS gera e gerencia o keystore automaticamente
eas credentials --platform android
# Selecionar: "Set up a new keystore" → EAS gerencia por você
```

**Build local com Gradle (alternativo, sem EAS):**

```bash
# Gerar arquivos nativos
npx expo prebuild --platform android --clean

cd android

# APK debug
./gradlew assembleDebug

# AAB release (produção)
./gradlew bundleRelease

# APK release
./gradlew assembleRelease
```

---

### Publicação na Google Play Store

**Pré-requisitos:**
- Conta de desenvolvedor Google Play ($25 taxa única)
- App criado no Google Play Console
- Service Account JSON para upload automático

**Passo a passo:**

```bash
# 1. Fazer build de produção
eas build --platform android --profile production

# 2. Enviar para Google Play (track interno)
eas submit --platform android --profile production

# Ou enviar o .aab manualmente no Google Play Console
```

**Submissão via Google Play Console (manual):**

1. Acesse [play.google.com/console](https://play.google.com/console)
2. Crie o aplicativo → Preencha informações da loja
3. **Produção** → **Criar nova versão**
4. Faça upload do `.aab` gerado pelo EAS
5. Preencha notas da versão
6. Revise e envie para revisão

**Fluxo de tracks recomendado:**
```
Testes internos → Testes fechados (alpha) → Testes abertos (beta) → Produção
```

---

### Build para iOS

> **Requisito:** Conta Apple Developer ($99/ano) — obrigatória para distribuição.

**Build na nuvem com EAS (sem macOS necessário):**

```bash
# Build de produção para App Store
eas build --platform ios --profile production

# EAS solicitará credenciais Apple na primeira execução:
# - Apple ID
# - Certificado de distribuição (EAS gerencia automaticamente)
# - Provisioning Profile (EAS gerencia automaticamente)
```

**Build local (requer macOS + Xcode):**

```bash
# Gerar projeto nativo
npx expo prebuild --platform ios --clean

cd ios

# Abrir no Xcode
open MeuApp.xcworkspace

# Via linha de comando (archive para App Store)
xcodebuild -workspace MeuApp.xcworkspace \
           -scheme MeuApp \
           -configuration Release \
           -archivePath build/MeuApp.xcarchive \
           archive

# Exportar IPA
xcodebuild -exportArchive \
           -archivePath build/MeuApp.xcarchive \
           -exportPath build/ \
           -exportOptionsPlist ExportOptions.plist
```

**Configurar certificados no Xcode (build local):**

1. Xcode → **Signing & Capabilities**
2. Selecionar **Team** (conta Apple Developer)
3. Habilitar **Automatically manage signing**
4. Xcode gerencia certificados e profiles automaticamente

---

### Publicação na Apple App Store

**Via EAS Submit (automático):**

```bash
# Enviar para App Store Connect após build
eas submit --platform ios --profile production

# EAS pedirá: Apple ID, App Store Connect App ID, Team ID
```

**Via App Store Connect (manual):**

1. Acesse [appstoreconnect.apple.com](https://appstoreconnect.apple.com)
2. **Meus Apps** → **+** → **Novo App**
3. Preencha: Nome, bundle ID, SKU, idioma primário
4. Faça upload do `.ipa` via **Transporter** (app macOS) ou Xcode → **Organizer**
5. Preencha metadados: descrição, capturas, palavras-chave, categoria
6. **Enviar para Revisão** → aguardar ~24-48h

**Informações necessárias para App Store:**
- Ícone 1024×1024px (sem transparência)
- Capturas de tela para iPhone 6.9" e 6.5" (obrigatório)
- Descrição curta e longa
- URL de privacidade (obrigatório)
- Classificação de conteúdo

---

### OTA Updates com EAS Update

O **EAS Update** permite atualizar o código JavaScript sem passar por review das lojas — ideal para correções rápidas de bugs.

```bash
# Instalar dependência
npx expo install expo-updates
```

**`app.json` — habilitar updates:**

```json
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/seu-project-id",
      "enabled": true,
      "checkAutomatically": "ON_LOAD",
      "fallbackToCacheTimeout": 3000
    },
    "runtimeVersion": {
      "policy": "appVersion"
    }
  }
}
```

**Publicar update:**

```bash
# Publicar para o canal production
eas update --channel production --message "Correção de bug no formulário"

# Publicar para canal específico
eas update --channel staging --message "Nova funcionalidade em teste"
```

**Verificar e aplicar updates manualmente no app:**

```tsx
// src/hooks/useOTAUpdate.ts
import * as Updates from 'expo-updates';
import { useState } from 'react';

export function useOTAUpdate() {
  const [checking, setChecking] = useState(false);

  const checkForUpdate = async () => {
    if (__DEV__) return; // Não verificar em desenvolvimento

    setChecking(true);
    try {
      const update = await Updates.checkForUpdateAsync();
      if (update.isAvailable) {
        await Updates.fetchUpdateAsync();
        await Updates.reloadAsync(); // Reinicia o app com a nova versão
      }
    } catch (error) {
      console.error('Erro ao verificar update:', error);
    } finally {
      setChecking(false);
    }
  };

  return { checkForUpdate, checking };
}
```

**Resumo dos comandos EAS:**

```bash
# Builds
eas build --platform android --profile production
eas build --platform ios --profile production
eas build --platform all --profile production    # ambos simultaneamente

# Submissão às lojas
eas submit --platform android
eas submit --platform ios

# OTA Updates
eas update --channel production --message "Descrição"

# Listar builds
eas build:list

# Ver credenciais
eas credentials
```

---

## Referência Rápida de Versões

| Pacote                      | Versão Recomendada | Observação                              |
|-----------------------------|--------------------|-----------------------------------------|
| `expo`                      | `~52.0.0`          | SDK base                                |
| `expo-router`               | `~4.0.0`           | File-based routing                      |
| `react-native`              | `0.76.x`           | Incluso no Expo SDK 52                  |
| `zustand`                   | `^5.0.0`           | Estado global                           |
| `@tanstack/react-query`     | `^5.0.0`           | Cache e sincronização de servidor       |
| `axios`                     | `^1.7.0`           | Cliente HTTP                            |
| `react-hook-form`           | `^7.54.0`          | Formulários                             |
| `zod`                       | `^3.23.0`          | Validação e schemas                     |
| `i18next`                   | `^24.0.0`          | Internacionalização                     |
| `react-i18next`             | `^15.0.0`          | Bindings React para i18next             |
| `expo-secure-store`         | `~14.0.0`          | Armazenamento seguro de tokens          |
| `@react-native-async-storage/async-storage` | `^2.1.0` | Persistência não sensível      |
| `jwt-decode`                | `^4.0.0`           | Decodificação de JWT                    |
| `expo-localization`         | `~16.0.0`          | Locale do dispositivo                   |
| `eas-cli`                   | `latest`           | Build e distribuição                    |

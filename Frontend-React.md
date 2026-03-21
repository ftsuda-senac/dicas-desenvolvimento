# React 19 com TypeScript — Referência Completa

Referência consolidada sobre React 19 com TypeScript, cobrindo fundamentos, roteamento, gerenciamento de estado, integração com backend, formulários, internacionalização, datas e autenticação JWT.

---

## Sumário

1. [Fundamentos do React 19](#1-fundamentos-do-react-19)
   - [Configuração do Projeto](#configuração-do-projeto)
   - [JSX e TSX](#jsx-e-tsx)
   - [Componentes Funcionais](#componentes-funcionais)
   - [Props e Tipagem](#props-e-tipagem)
   - [Hooks Essenciais](#hooks-essenciais)
   - [Novidades do React 19](#novidades-do-react-19)
2. [Gerenciamento de Rotas com React Router](#2-gerenciamento-de-rotas-com-react-router)
   - [Configuração](#configuração-do-react-router)
   - [Rotas Protegidas](#rotas-protegidas)
   - [Navegação Programática](#navegação-programática)
3. [Estado Global](#3-estado-global)
   - [Redux Toolkit](#redux-toolkit)
   - [Zustand](#zustand)
4. [Integração com Backend](#4-integração-com-backend)
   - [Fetch API](#fetch-api)
   - [TanStack Query](#tanstack-query)
   - [RTK Query](#rtk-query)
5. [Formulários e Validação](#5-formulários-e-validação)
   - [React Hook Form](#react-hook-form)
   - [Validação com Zod](#validação-com-zod)
6. [Internacionalização com react-i18next](#6-internacionalização-com-react-i18next)
7. [Manipulação de Datas com date-fns](#7-manipulação-de-datas-com-date-fns)
8. [Autenticação JWT e Controle de Acesso por Role](#8-autenticação-jwt-e-controle-de-acesso-por-role)
   - [Estrutura do Token JWT](#estrutura-do-token-jwt)
   - [Contexto de Autenticação](#contexto-de-autenticação)
   - [Interceptor de Requisições](#interceptor-de-requisições)
   - [Componentes de Proteção por Role](#componentes-de-proteção-por-role)

---

## 1. Fundamentos do React 19

### Configuração do Projeto

```bash
# Criar projeto com Vite (recomendado para React 19)
npm create vite@latest meu-projeto -- --template react-ts
cd meu-projeto
npm install

# Instalar dependências principais abordadas neste guia
npm install react-router-dom
npm install @reduxjs/toolkit react-redux
npm install zustand
npm install @tanstack/react-query
npm install react-hook-form @hookform/resolvers zod
npm install react-i18next i18next i18next-browser-languagedetector i18next-http-backend
npm install date-fns
npm install jwt-decode
```

Estrutura recomendada de pastas:

```
src/
├── assets/
├── components/
│   ├── common/
│   └── layout/
├── features/
│   ├── auth/
│   ├── users/
│   └── products/
├── hooks/
├── i18n/
├── pages/
├── routes/
├── services/
├── store/
├── types/
└── utils/
```

---

### JSX e TSX

TSX é a extensão do JSX com suporte a TypeScript. Regras importantes:

- Todo componente deve retornar **um único elemento raiz** (ou `<>...</>` Fragment).
- Atributos HTML usam **camelCase**: `className`, `onClick`, `htmlFor`.
- Expressões JavaScript ficam dentro de `{}`.
- Listas precisam de `key` única.

```tsx
// src/components/common/CardProduto.tsx
type Produto = {
  id: number;
  nome: string;
  preco: number;
  disponivel: boolean;
};

function CardProduto({ id, nome, preco, disponivel }: Produto) {
  return (
    <article className="card-produto" aria-label={`Produto: ${nome}`}>
      <h2>{nome}</h2>
      <p>Preço: R$ {preco.toFixed(2)}</p>
      {disponivel ? (
        <span className="badge badge--disponivel">Disponível</span>
      ) : (
        <span className="badge badge--indisponivel">Indisponível</span>
      )}
    </article>
  );
}

// Renderizando uma lista de produtos
function ListaProdutos({ produtos }: { produtos: Produto[] }) {
  return (
    <section>
      {produtos.map((p) => (
        <CardProduto key={p.id} {...p} />
      ))}
    </section>
  );
}
```

---

### Componentes Funcionais

Componentes são funções que recebem props e retornam JSX. Com TypeScript, tipamos as props explicitamente.

```tsx
// Componente simples
function Saudacao({ nome }: { nome: string }) {
  return <h1>Olá, {nome}!</h1>;
}

// Com props opcionais e valores padrão
type BotaoProps = {
  label: string;
  variante?: 'primario' | 'secundario' | 'perigo';
  desabilitado?: boolean;
  onClick?: () => void;
};

function Botao({
  label,
  variante = 'primario',
  desabilitado = false,
  onClick,
}: BotaoProps) {
  return (
    <button
      className={`btn btn--${variante}`}
      disabled={desabilitado}
      onClick={onClick}
    >
      {label}
    </button>
  );
}

// Componente que aceita filhos (children)
type PainelProps = {
  titulo: string;
  children: React.ReactNode;
};

function Painel({ titulo, children }: PainelProps) {
  return (
    <div className="painel">
      <header className="painel__cabecalho">
        <h2>{titulo}</h2>
      </header>
      <div className="painel__corpo">{children}</div>
    </div>
  );
}
```

---

### Props e Tipagem

```tsx
// Tipos utilitários do TypeScript com React
import { ComponentProps, ReactNode, CSSProperties } from 'react';

// Estendendo props nativas de elementos HTML
type InputProps = ComponentProps<'input'> & {
  label: string;
  erro?: string;
};

function CampoTexto({ label, erro, id, ...rest }: InputProps) {
  return (
    <div className="campo">
      <label htmlFor={id}>{label}</label>
      <input id={id} {...rest} className={erro ? 'input--erro' : 'input'} />
      {erro && <span className="campo__erro" role="alert">{erro}</span>}
    </div>
  );
}

// Tipagem de eventos
function FormularioBusca() {
  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const dados = new FormData(e.currentTarget);
    console.log(dados.get('busca'));
  }

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    console.log(e.target.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="busca" onChange={handleChange} />
      <button type="submit">Buscar</button>
    </form>
  );
}
```

---

### Hooks Essenciais

#### useState

```tsx
import { useState } from 'react';

function Contador() {
  const [contagem, setContagem] = useState<number>(0);

  return (
    <div>
      <p>Contagem: {contagem}</p>
      <button onClick={() => setContagem((c) => c + 1)}>Incrementar</button>
      <button onClick={() => setContagem(0)}>Resetar</button>
    </div>
  );
}

// Com objeto no estado
type Filtros = {
  busca: string;
  categoria: string;
  ativo: boolean;
};

function FiltroProdutos() {
  const [filtros, setFiltros] = useState<Filtros>({
    busca: '',
    categoria: 'todos',
    ativo: true,
  });

  function atualizarFiltro<K extends keyof Filtros>(campo: K, valor: Filtros[K]) {
    setFiltros((prev) => ({ ...prev, [campo]: valor }));
  }

  return (
    <form>
      <input
        value={filtros.busca}
        onChange={(e) => atualizarFiltro('busca', e.target.value)}
        placeholder="Buscar..."
      />
    </form>
  );
}
```

#### useEffect

```tsx
import { useState, useEffect } from 'react';

function ListaUsuarios() {
  const [usuarios, setUsuarios] = useState<Usuario[]>([]);
  const [carregando, setCarregando] = useState(true);
  const [erro, setErro] = useState<string | null>(null);

  useEffect(() => {
    let cancelado = false; // evita atualização em componente desmontado

    async function buscarUsuarios() {
      try {
        setCarregando(true);
        const res = await fetch('/api/usuarios');
        if (!res.ok) throw new Error('Erro ao carregar usuários');
        const dados = await res.json();
        if (!cancelado) setUsuarios(dados);
      } catch (e) {
        if (!cancelado) setErro(e instanceof Error ? e.message : 'Erro desconhecido');
      } finally {
        if (!cancelado) setCarregando(false);
      }
    }

    buscarUsuarios();
    return () => { cancelado = true; };
  }, []); // array vazio = executa só na montagem

  if (carregando) return <p>Carregando...</p>;
  if (erro) return <p role="alert">Erro: {erro}</p>;
  return <ul>{usuarios.map((u) => <li key={u.id}>{u.nome}</li>)}</ul>;
}
```

#### useCallback e useMemo

```tsx
import { useState, useCallback, useMemo } from 'react';

function ListaFiltrada({ itens }: { itens: string[] }) {
  const [busca, setBusca] = useState('');

  // Memoriza o resultado filtrado — recalcula só quando busca ou itens mudam
  const itensFiltrados = useMemo(
    () => itens.filter((item) => item.toLowerCase().includes(busca.toLowerCase())),
    [itens, busca]
  );

  // Memoriza a função para evitar re-renders em componentes filhos
  const handleBusca = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setBusca(e.target.value);
  }, []);

  return (
    <div>
      <input value={busca} onChange={handleBusca} placeholder="Filtrar..." />
      <ul>
        {itensFiltrados.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### useRef

```tsx
import { useRef, useEffect } from 'react';

function CampoAutoFoco() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} type="text" />;
}

// Ref para armazenar valor mutável sem causar re-render
function Temporizador() {
  const intervaloRef = useRef<ReturnType<typeof setInterval> | null>(null);
  const [ativo, setAtivo] = useState(false);
  const [segundos, setSegundos] = useState(0);

  function iniciar() {
    setAtivo(true);
    intervaloRef.current = setInterval(() => setSegundos((s) => s + 1), 1000);
  }

  function parar() {
    setAtivo(false);
    if (intervaloRef.current) clearInterval(intervaloRef.current);
  }

  return (
    <div>
      <p>{segundos}s</p>
      <button onClick={iniciar} disabled={ativo}>Iniciar</button>
      <button onClick={parar} disabled={!ativo}>Parar</button>
    </div>
  );
}
```

#### useContext

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type Tema = 'claro' | 'escuro';

type TemaContexto = {
  tema: Tema;
  alternarTema: () => void;
};

const TemaContext = createContext<TemaContexto | null>(null);

function ProvedorTema({ children }: { children: ReactNode }) {
  const [tema, setTema] = useState<Tema>('claro');
  const alternarTema = () => setTema((t) => (t === 'claro' ? 'escuro' : 'claro'));

  return (
    <TemaContext.Provider value={{ tema, alternarTema }}>
      {children}
    </TemaContext.Provider>
  );
}

// Hook customizado para consumir o contexto com segurança
function useTema() {
  const ctx = useContext(TemaContext);
  if (!ctx) throw new Error('useTema deve ser usado dentro de ProvedorTema');
  return ctx;
}

function BotaoTema() {
  const { tema, alternarTema } = useTema();
  return (
    <button onClick={alternarTema}>
      Tema atual: {tema}
    </button>
  );
}
```

#### useReducer

```tsx
import { useReducer } from 'react';

type Estado = {
  itens: string[];
  carregando: boolean;
  erro: string | null;
};

type Acao =
  | { type: 'CARREGAR' }
  | { type: 'SUCESSO'; payload: string[] }
  | { type: 'ERRO'; payload: string }
  | { type: 'ADICIONAR'; payload: string };

const estadoInicial: Estado = { itens: [], carregando: false, erro: null };

function reducer(estado: Estado, acao: Acao): Estado {
  switch (acao.type) {
    case 'CARREGAR':
      return { ...estado, carregando: true, erro: null };
    case 'SUCESSO':
      return { ...estado, carregando: false, itens: acao.payload };
    case 'ERRO':
      return { ...estado, carregando: false, erro: acao.payload };
    case 'ADICIONAR':
      return { ...estado, itens: [...estado.itens, acao.payload] };
    default:
      return estado;
  }
}

function ComponenteComReducer() {
  const [estado, dispatch] = useReducer(reducer, estadoInicial);

  return (
    <div>
      {estado.carregando && <p>Carregando...</p>}
      {estado.erro && <p role="alert">{estado.erro}</p>}
      <button onClick={() => dispatch({ type: 'ADICIONAR', payload: 'novo item' })}>
        Adicionar
      </button>
    </div>
  );
}
```

---

### Novidades do React 19

#### Actions e useActionState

O React 19 introduz o conceito de **Actions**: funções assíncronas que gerenciam transições de estado de forma nativa, eliminando a necessidade de `useState` manual para loading/erro em muitos casos.

```tsx
import { useActionState } from 'react';

type Resultado = { sucesso: boolean; mensagem: string } | null;

async function salvarPerfil(estadoAnterior: Resultado, formData: FormData): Promise<Resultado> {
  const nome = formData.get('nome') as string;
  const email = formData.get('email') as string;

  try {
    const res = await fetch('/api/perfil', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ nome, email }),
    });

    if (!res.ok) throw new Error('Falha ao salvar');
    return { sucesso: true, mensagem: 'Perfil salvo com sucesso!' };
  } catch {
    return { sucesso: false, mensagem: 'Erro ao salvar o perfil.' };
  }
}

function FormularioPerfil() {
  const [resultado, formAction, isPending] = useActionState(salvarPerfil, null);

  return (
    <form action={formAction}>
      <input name="nome" placeholder="Nome" required />
      <input name="email" type="email" placeholder="E-mail" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Salvando...' : 'Salvar'}
      </button>
      {resultado && (
        <p className={resultado.sucesso ? 'mensagem--sucesso' : 'mensagem--erro'}>
          {resultado.mensagem}
        </p>
      )}
    </form>
  );
}
```

#### useOptimistic

Permite atualizar a UI de forma otimista antes de receber a resposta do servidor.

```tsx
import { useOptimistic, useState } from 'react';

type Mensagem = { id: number; texto: string; enviando?: boolean };

function Chat({ mensagensIniciais }: { mensagensIniciais: Mensagem[] }) {
  const [mensagens, setMensagens] = useState<Mensagem[]>(mensagensIniciais);
  const [otimistas, adicionarOtimista] = useOptimistic(
    mensagens,
    (estado, novaMensagem: Mensagem) => [...estado, novaMensagem]
  );

  async function enviar(formData: FormData) {
    const texto = formData.get('mensagem') as string;
    const tempId = Date.now();

    adicionarOtimista({ id: tempId, texto, enviando: true });

    const res = await fetch('/api/mensagens', {
      method: 'POST',
      body: JSON.stringify({ texto }),
      headers: { 'Content-Type': 'application/json' },
    });
    const salva = await res.json();
    setMensagens((prev) => [...prev, salva]);
  }

  return (
    <div>
      <ul>
        {otimistas.map((m) => (
          <li key={m.id} style={{ opacity: m.enviando ? 0.5 : 1 }}>
            {m.texto}
          </li>
        ))}
      </ul>
      <form action={enviar}>
        <input name="mensagem" />
        <button type="submit">Enviar</button>
      </form>
    </div>
  );
}
```

#### use()

O hook `use()` permite consumir Promises e Contextos de forma mais flexível, inclusive dentro de condicionais.

```tsx
import { use, Suspense } from 'react';

// Consumindo um contexto com use() (pode ser usado em condicionais)
function ComponenteCondicional({ mostrarTema }: { mostrarTema: boolean }) {
  if (mostrarTema) {
    const { tema } = use(TemaContext)!; // uso condicional, algo impossível com useContext
    return <p>Tema: {tema}</p>;
  }
  return <p>Sem tema</p>;
}

// Consumindo uma Promise com use() + Suspense
async function buscarUsuario(id: number): Promise<Usuario> {
  const res = await fetch(`/api/usuarios/${id}`);
  return res.json();
}

const promiseUsuario = buscarUsuario(1);

function DetalhesUsuario() {
  const usuario = use(promiseUsuario); // "suspende" automaticamente
  return <h1>Olá, {usuario.nome}</h1>;
}

function PaginaUsuario() {
  return (
    <Suspense fallback={<p>Carregando usuário...</p>}>
      <DetalhesUsuario />
    </Suspense>
  );
}
```

#### Metadados de Documento (Document Metadata)

React 19 permite renderizar `<title>`, `<meta>` e `<link>` diretamente nos componentes sem biblioteca externa.

```tsx
function PaginaProduto({ produto }: { produto: Produto }) {
  return (
    <>
      <title>{produto.nome} — Minha Loja</title>
      <meta name="description" content={`Compre ${produto.nome} com o melhor preço`} />
      <link rel="canonical" href={`https://minhaloja.com/produtos/${produto.id}`} />

      <main>
        <h1>{produto.nome}</h1>
        <p>R$ {produto.preco.toFixed(2)}</p>
      </main>
    </>
  );
}
```

---

## 2. Gerenciamento de Rotas com React Router

### Configuração do React Router

React Router v7 adota o modelo de configuração declarativa com `createBrowserRouter`.

```tsx
// src/routes/router.tsx
import { createBrowserRouter, RouterProvider, Outlet } from 'react-router-dom';
import { Suspense, lazy } from 'react';
import Layout from '../components/layout/Layout';
import RotaProtegida from './RotaProtegida';
import PaginaLogin from '../pages/PaginaLogin';
import PaginaNaoEncontrada from '../pages/PaginaNaoEncontrada';

// Lazy loading das páginas
const PaginaInicio = lazy(() => import('../pages/PaginaInicio'));
const PaginaProdutos = lazy(() => import('../pages/PaginaProdutos'));
const PaginaDetalhesProduto = lazy(() => import('../pages/PaginaDetalhesProduto'));
const PaginaAdmin = lazy(() => import('../pages/PaginaAdmin'));
const PaginaPerfil = lazy(() => import('../pages/PaginaPerfil'));

const router = createBrowserRouter([
  {
    path: '/',
    element: (
      <Suspense fallback={<div className="carregando-pagina">Carregando...</div>}>
        <Layout />
      </Suspense>
    ),
    errorElement: <PaginaNaoEncontrada />,
    children: [
      { index: true, element: <PaginaInicio /> },
      { path: 'produtos', element: <PaginaProdutos /> },
      { path: 'produtos/:id', element: <PaginaDetalhesProduto /> },
      {
        element: <RotaProtegida />,
        children: [
          { path: 'perfil', element: <PaginaPerfil /> },
          {
            element: <RotaProtegida rolesPermitidas={['ADMIN']} />,
            children: [
              { path: 'admin', element: <PaginaAdmin /> },
            ],
          },
        ],
      },
    ],
  },
  { path: '/login', element: <PaginaLogin /> },
]);

export function AppRouter() {
  return <RouterProvider router={router} />;
}
```

```tsx
// src/components/layout/Layout.tsx
import { Outlet, NavLink } from 'react-router-dom';
import { useAuth } from '../../features/auth/useAuth';

function Layout() {
  const { usuario, sair } = useAuth();

  return (
    <div className="app">
      <header className="cabecalho">
        <nav className="nav-principal" aria-label="Navegação principal">
          <NavLink to="/" end className={({ isActive }) => isActive ? 'nav-link nav-link--ativo' : 'nav-link'}>
            Início
          </NavLink>
          <NavLink to="/produtos" className={({ isActive }) => isActive ? 'nav-link nav-link--ativo' : 'nav-link'}>
            Produtos
          </NavLink>
          {usuario && (
            <NavLink to="/perfil" className={({ isActive }) => isActive ? 'nav-link nav-link--ativo' : 'nav-link'}>
              Perfil
            </NavLink>
          )}
          {usuario?.roles.includes('ADMIN') && (
            <NavLink to="/admin" className={({ isActive }) => isActive ? 'nav-link nav-link--ativo' : 'nav-link'}>
              Admin
            </NavLink>
          )}
        </nav>
        <div className="cabecalho__acoes">
          {usuario ? (
            <button onClick={sair} className="btn btn--secundario">Sair</button>
          ) : (
            <NavLink to="/login" className="btn btn--primario">Entrar</NavLink>
          )}
        </div>
      </header>

      <main className="conteudo-principal">
        <Outlet />
      </main>

      <footer className="rodape">
        <p>&copy; 2025 Minha Aplicação</p>
      </footer>
    </div>
  );
}

export default Layout;
```

---

### Rotas Protegidas

```tsx
// src/routes/RotaProtegida.tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';
import { useAuth } from '../features/auth/useAuth';

type RotaProtegidaProps = {
  rolesPermitidas?: string[];
};

function RotaProtegida({ rolesPermitidas }: RotaProtegidaProps) {
  const { usuario, carregando } = useAuth();
  const localizacao = useLocation();

  if (carregando) return <div className="carregando-pagina">Verificando autenticação...</div>;

  if (!usuario) {
    // Salva a URL de destino para redirecionar após login
    return <Navigate to="/login" state={{ from: localizacao }} replace />;
  }

  if (rolesPermitidas && !rolesPermitidas.some((r) => usuario.roles.includes(r))) {
    return <Navigate to="/" replace />;
  }

  return <Outlet />;
}

export default RotaProtegida;
```

---

### Navegação Programática

```tsx
import { useNavigate, useParams, useSearchParams, useLocation } from 'react-router-dom';

function ExemplosNavegacao() {
  const navigate = useNavigate();
  const { id } = useParams<{ id: string }>();
  const [searchParams, setSearchParams] = useSearchParams();
  const location = useLocation();

  // Recupera estado passado via navigate
  const state = location.state as { from?: Location } | null;

  const pagina = Number(searchParams.get('pagina') ?? '1');
  const busca = searchParams.get('busca') ?? '';

  function irParaProduto(produtoId: number) {
    navigate(`/produtos/${produtoId}`);
  }

  function voltarOuIrParaInicio() {
    if (state?.from) {
      navigate(state.from.pathname);
    } else {
      navigate('/');
    }
  }

  function atualizarBusca(valor: string) {
    setSearchParams({ busca: valor, pagina: '1' });
  }

  function proximaPagina() {
    setSearchParams({ busca, pagina: String(pagina + 1) });
  }

  return (
    <div>
      <button onClick={() => navigate(-1)}>Voltar</button>
      <button onClick={voltarOuIrParaInicio}>Início</button>
      <button onClick={() => irParaProduto(42)}>Ver Produto 42</button>
      <input
        value={busca}
        onChange={(e) => atualizarBusca(e.target.value)}
        placeholder="Buscar produtos..."
      />
      <button onClick={proximaPagina}>Próxima página ({pagina})</button>
    </div>
  );
}
```

---

## 3. Estado Global

### Redux Toolkit

Redux Toolkit (RTK) é a forma moderna e recomendada de usar Redux, eliminando o boilerplate excessivo.

```tsx
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import produtosReducer from '../features/products/produtosSlice';
import authReducer from '../features/auth/authSlice';
import { api } from '../services/api'; // RTK Query

export const store = configureStore({
  reducer: {
    produtos: produtosReducer,
    auth: authReducer,
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// Hooks tipados — use sempre estes em vez dos originais
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

```tsx
// src/features/products/produtosSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

type Produto = {
  id: number;
  nome: string;
  preco: number;
  categoria: string;
};

type ProdutosState = {
  itens: Produto[];
  carregando: boolean;
  erro: string | null;
  filtroCategoria: string;
};

// Thunk para busca assíncrona
export const buscarProdutos = createAsyncThunk(
  'produtos/buscarProdutos',
  async (categoria?: string, { rejectWithValue }) => {
    try {
      const url = categoria ? `/api/produtos?categoria=${categoria}` : '/api/produtos';
      const res = await fetch(url);
      if (!res.ok) throw new Error('Falha ao buscar produtos');
      return (await res.json()) as Produto[];
    } catch (e) {
      return rejectWithValue(e instanceof Error ? e.message : 'Erro desconhecido');
    }
  }
);

const produtosSlice = createSlice({
  name: 'produtos',
  initialState: {
    itens: [],
    carregando: false,
    erro: null,
    filtroCategoria: '',
  } as ProdutosState,
  reducers: {
    setFiltroCategoria(state, action: PayloadAction<string>) {
      state.filtroCategoria = action.payload;
    },
    adicionarProduto(state, action: PayloadAction<Produto>) {
      state.itens.push(action.payload);
    },
    removerProduto(state, action: PayloadAction<number>) {
      state.itens = state.itens.filter((p) => p.id !== action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(buscarProdutos.pending, (state) => {
        state.carregando = true;
        state.erro = null;
      })
      .addCase(buscarProdutos.fulfilled, (state, action) => {
        state.carregando = false;
        state.itens = action.payload;
      })
      .addCase(buscarProdutos.rejected, (state, action) => {
        state.carregando = false;
        state.erro = action.payload as string;
      });
  },
});

export const { setFiltroCategoria, adicionarProduto, removerProduto } = produtosSlice.actions;
export default produtosSlice.reducer;

// Selectors
export const selectProdutosFiltrados = (state: { produtos: ProdutosState }) => {
  const { itens, filtroCategoria } = state.produtos;
  if (!filtroCategoria) return itens;
  return itens.filter((p) => p.categoria === filtroCategoria);
};
```

```tsx
// src/main.tsx — Provider do Redux
import { Provider } from 'react-redux';
import { store } from './store';
import { AppRouter } from './routes/router';

function App() {
  return (
    <Provider store={store}>
      <AppRouter />
    </Provider>
  );
}
```

```tsx
// Usando Redux em um componente
import { useEffect } from 'react';
import { useAppDispatch, useAppSelector } from '../../store';
import { buscarProdutos, setFiltroCategoria, selectProdutosFiltrados } from './produtosSlice';

function PaginaProdutos() {
  const dispatch = useAppDispatch();
  const produtos = useAppSelector(selectProdutosFiltrados);
  const { carregando, erro, filtroCategoria } = useAppSelector((s) => s.produtos);

  useEffect(() => {
    dispatch(buscarProdutos());
  }, [dispatch]);

  return (
    <div>
      <select
        value={filtroCategoria}
        onChange={(e) => dispatch(setFiltroCategoria(e.target.value))}
      >
        <option value="">Todas as categorias</option>
        <option value="eletronicos">Eletrônicos</option>
        <option value="livros">Livros</option>
      </select>

      {carregando && <p>Carregando produtos...</p>}
      {erro && <p role="alert" className="erro">{erro}</p>}

      <ul>
        {produtos.map((p) => (
          <li key={p.id}>{p.nome} — R$ {p.preco.toFixed(2)}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

### Zustand

Zustand é uma biblioteca minimalista de gerenciamento de estado, sem boilerplate, ideal para projetos de médio porte.

```tsx
// src/store/useCarrinhoStore.ts
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

type ItemCarrinho = {
  id: number;
  nome: string;
  preco: number;
  quantidade: number;
};

type CarrinhoStore = {
  itens: ItemCarrinho[];
  total: number;
  adicionarItem: (produto: Omit<ItemCarrinho, 'quantidade'>) => void;
  removerItem: (id: number) => void;
  alterarQuantidade: (id: number, quantidade: number) => void;
  limparCarrinho: () => void;
};

export const useCarrinhoStore = create<CarrinhoStore>()(
  devtools(
    persist(
      (set, get) => ({
        itens: [],
        total: 0,

        adicionarItem(produto) {
          const itensAtuais = get().itens;
          const existente = itensAtuais.find((i) => i.id === produto.id);

          let novosItens: ItemCarrinho[];
          if (existente) {
            novosItens = itensAtuais.map((i) =>
              i.id === produto.id ? { ...i, quantidade: i.quantidade + 1 } : i
            );
          } else {
            novosItens = [...itensAtuais, { ...produto, quantidade: 1 }];
          }

          const novoTotal = novosItens.reduce(
            (acc, i) => acc + i.preco * i.quantidade,
            0
          );
          set({ itens: novosItens, total: novoTotal });
        },

        removerItem(id) {
          const novosItens = get().itens.filter((i) => i.id !== id);
          const novoTotal = novosItens.reduce(
            (acc, i) => acc + i.preco * i.quantidade,
            0
          );
          set({ itens: novosItens, total: novoTotal });
        },

        alterarQuantidade(id, quantidade) {
          if (quantidade <= 0) {
            get().removerItem(id);
            return;
          }
          const novosItens = get().itens.map((i) =>
            i.id === id ? { ...i, quantidade } : i
          );
          const novoTotal = novosItens.reduce(
            (acc, i) => acc + i.preco * i.quantidade,
            0
          );
          set({ itens: novosItens, total: novoTotal });
        },

        limparCarrinho() {
          set({ itens: [], total: 0 });
        },
      }),
      { name: 'carrinho-storage' } // persiste no localStorage
    )
  )
);
```

```tsx
// Usando o store Zustand em componentes
function BotaoCarrinho() {
  const totalItens = useCarrinhoStore((s) => s.itens.reduce((acc, i) => acc + i.quantidade, 0));

  return (
    <button className="btn-carrinho" aria-label={`Carrinho com ${totalItens} itens`}>
      Carrinho ({totalItens})
    </button>
  );
}

function CardProdutoComCarrinho({ produto }: { produto: Produto }) {
  const adicionarItem = useCarrinhoStore((s) => s.adicionarItem);

  return (
    <article className="card-produto">
      <h3>{produto.nome}</h3>
      <p>R$ {produto.preco.toFixed(2)}</p>
      <button onClick={() => adicionarItem(produto)}>Adicionar ao carrinho</button>
    </article>
  );
}

function Carrinho() {
  const { itens, total, removerItem, alterarQuantidade, limparCarrinho } = useCarrinhoStore();

  if (itens.length === 0) return <p>Carrinho vazio</p>;

  return (
    <div className="carrinho">
      {itens.map((item) => (
        <div key={item.id} className="carrinho__item">
          <span>{item.nome}</span>
          <input
            type="number"
            min="1"
            value={item.quantidade}
            onChange={(e) => alterarQuantidade(item.id, Number(e.target.value))}
          />
          <span>R$ {(item.preco * item.quantidade).toFixed(2)}</span>
          <button onClick={() => removerItem(item.id)}>Remover</button>
        </div>
      ))}
      <p className="carrinho__total">Total: R$ {total.toFixed(2)}</p>
      <button onClick={limparCarrinho} className="btn btn--perigo">Limpar carrinho</button>
    </div>
  );
}
```

---

## 4. Integração com Backend

### Fetch API

Padrão nativo do browser para requisições HTTP.

```tsx
// src/services/httpClient.ts
const BASE_URL = import.meta.env.VITE_API_URL ?? 'http://localhost:8080/api';

type OpcoesFetch = RequestInit & {
  params?: Record<string, string | number | boolean | undefined>;
};

async function request<T>(endpoint: string, opcoes: OpcoesFetch = {}): Promise<T> {
  const { params, ...rest } = opcoes;

  // Monta query string
  let url = `${BASE_URL}${endpoint}`;
  if (params) {
    const qs = new URLSearchParams(
      Object.entries(params)
        .filter(([, v]) => v !== undefined)
        .map(([k, v]) => [k, String(v)])
    );
    url += `?${qs}`;
  }

  // Injeta token JWT nos headers
  const token = localStorage.getItem('token');
  const headers: HeadersInit = {
    'Content-Type': 'application/json',
    ...(token ? { Authorization: `Bearer ${token}` } : {}),
    ...rest.headers,
  };

  const resposta = await fetch(url, { ...rest, headers });

  // Token expirado
  if (resposta.status === 401) {
    localStorage.removeItem('token');
    window.location.href = '/login';
  }

  if (!resposta.ok) {
    const erro = await resposta.json().catch(() => ({ mensagem: resposta.statusText }));
    throw new Error(erro.mensagem ?? 'Erro na requisição');
  }

  // 204 No Content
  if (resposta.status === 204) return null as T;
  return resposta.json() as Promise<T>;
}

export const http = {
  get: <T>(endpoint: string, opcoes?: OpcoesFetch) =>
    request<T>(endpoint, { ...opcoes, method: 'GET' }),

  post: <T>(endpoint: string, body: unknown, opcoes?: OpcoesFetch) =>
    request<T>(endpoint, { ...opcoes, method: 'POST', body: JSON.stringify(body) }),

  put: <T>(endpoint: string, body: unknown, opcoes?: OpcoesFetch) =>
    request<T>(endpoint, { ...opcoes, method: 'PUT', body: JSON.stringify(body) }),

  patch: <T>(endpoint: string, body: unknown, opcoes?: OpcoesFetch) =>
    request<T>(endpoint, { ...opcoes, method: 'PATCH', body: JSON.stringify(body) }),

  delete: <T>(endpoint: string, opcoes?: OpcoesFetch) =>
    request<T>(endpoint, { ...opcoes, method: 'DELETE' }),
};
```

---

### TanStack Query

TanStack Query (React Query) gerencia cache, refetch, loading e erro de chamadas ao servidor.

```tsx
// src/main.tsx — Configuração do QueryClient
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,   // 5 minutos
      gcTime: 10 * 60 * 1000,     // 10 minutos (antes chamado cacheTime)
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Provider store={store}>
        <AppRouter />
      </Provider>
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```tsx
// src/features/products/useProdutos.ts
import { useQuery, useMutation, useQueryClient, keepPreviousData } from '@tanstack/react-query';
import { http } from '../../services/httpClient';

type Produto = { id: number; nome: string; preco: number };
type ProdutoInput = Omit<Produto, 'id'>;

// Chaves de cache centralizadas
export const produtoKeys = {
  all: ['produtos'] as const,
  lists: () => [...produtoKeys.all, 'list'] as const,
  list: (filtros: Record<string, unknown>) => [...produtoKeys.lists(), filtros] as const,
  detail: (id: number) => [...produtoKeys.all, 'detail', id] as const,
};

// Query: listar com paginação
export function useProdutos(pagina = 1, busca = '') {
  return useQuery({
    queryKey: produtoKeys.list({ pagina, busca }),
    queryFn: () => http.get<{ itens: Produto[]; total: number }>(
      '/produtos',
      { params: { pagina, busca: busca || undefined } }
    ),
    placeholderData: keepPreviousData, // mantém dados anteriores durante refetch
  });
}

// Query: detalhe de um produto
export function useProduto(id: number) {
  return useQuery({
    queryKey: produtoKeys.detail(id),
    queryFn: () => http.get<Produto>(`/produtos/${id}`),
    enabled: !!id, // só executa se id for válido
  });
}

// Mutation: criar produto
export function useCriarProduto() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (dados: ProdutoInput) => http.post<Produto>('/produtos', dados),
    onSuccess: () => {
      // Invalida o cache da lista para recarregar
      queryClient.invalidateQueries({ queryKey: produtoKeys.lists() });
    },
    onError: (erro: Error) => {
      console.error('Erro ao criar produto:', erro.message);
    },
  });
}

// Mutation: atualizar com atualização otimista
export function useAtualizarProduto() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, ...dados }: Produto) =>
      http.put<Produto>(`/produtos/${id}`, dados),

    onMutate: async (produtoAtualizado) => {
      await queryClient.cancelQueries({ queryKey: produtoKeys.detail(produtoAtualizado.id) });
      const anterior = queryClient.getQueryData<Produto>(produtoKeys.detail(produtoAtualizado.id));
      queryClient.setQueryData(produtoKeys.detail(produtoAtualizado.id), produtoAtualizado);
      return { anterior };
    },

    onError: (_err, produto, context) => {
      // Reverte em caso de erro
      queryClient.setQueryData(produtoKeys.detail(produto.id), context?.anterior);
    },

    onSettled: (_, __, produto) => {
      queryClient.invalidateQueries({ queryKey: produtoKeys.detail(produto.id) });
    },
  });
}
```

```tsx
// Usando TanStack Query em componentes
function ListaProdutosQuery() {
  const [pagina, setPagina] = useState(1);
  const [busca, setBusca] = useState('');
  const { data, isLoading, isError, error, isFetching } = useProdutos(pagina, busca);
  const criarProduto = useCriarProduto();

  if (isLoading) return <p>Carregando...</p>;
  if (isError) return <p role="alert">Erro: {(error as Error).message}</p>;

  return (
    <div>
      <input value={busca} onChange={(e) => setBusca(e.target.value)} placeholder="Buscar..." />

      {isFetching && <span className="indicador-atualizando">Atualizando...</span>}

      <ul>
        {data?.itens.map((p) => (
          <li key={p.id}>{p.nome} — R$ {p.preco.toFixed(2)}</li>
        ))}
      </ul>

      <div className="paginacao">
        <button onClick={() => setPagina((p) => p - 1)} disabled={pagina === 1}>Anterior</button>
        <span>Página {pagina}</span>
        <button onClick={() => setPagina((p) => p + 1)}>Próxima</button>
      </div>

      <button
        onClick={() => criarProduto.mutate({ nome: 'Novo Produto', preco: 99.90 })}
        disabled={criarProduto.isPending}
      >
        {criarProduto.isPending ? 'Criando...' : 'Criar Produto'}
      </button>
    </div>
  );
}
```

---

### RTK Query

RTK Query é a solução de cache e fetching integrada ao Redux Toolkit, ideal quando já se usa Redux na aplicação.

```tsx
// src/services/api.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type { RootState } from '../store';

type Produto = { id: number; nome: string; preco: number; categoria: string };
type ProdutoInput = Omit<Produto, 'id'>;
type PaginaResposta<T> = { itens: T[]; total: number; pagina: number };

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: import.meta.env.VITE_API_URL ?? 'http://localhost:8080/api',
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) headers.set('Authorization', `Bearer ${token}`);
      return headers;
    },
  }),
  tagTypes: ['Produto', 'Usuario'],
  endpoints: (builder) => ({
    // Listar produtos
    listarProdutos: builder.query<PaginaResposta<Produto>, { pagina?: number; busca?: string }>({
      query: ({ pagina = 1, busca } = {}) => ({
        url: '/produtos',
        params: { pagina, ...(busca ? { busca } : {}) },
      }),
      providesTags: (result) =>
        result
          ? [
              ...result.itens.map(({ id }) => ({ type: 'Produto' as const, id })),
              { type: 'Produto', id: 'LIST' },
            ]
          : [{ type: 'Produto', id: 'LIST' }],
    }),

    // Buscar produto por ID
    buscarProduto: builder.query<Produto, number>({
      query: (id) => `/produtos/${id}`,
      providesTags: (_, __, id) => [{ type: 'Produto', id }],
    }),

    // Criar produto
    criarProduto: builder.mutation<Produto, ProdutoInput>({
      query: (dados) => ({ url: '/produtos', method: 'POST', body: dados }),
      invalidatesTags: [{ type: 'Produto', id: 'LIST' }],
    }),

    // Atualizar produto
    atualizarProduto: builder.mutation<Produto, Produto>({
      query: ({ id, ...dados }) => ({ url: `/produtos/${id}`, method: 'PUT', body: dados }),
      invalidatesTags: (_, __, { id }) => [{ type: 'Produto', id }],
    }),

    // Deletar produto
    deletarProduto: builder.mutation<void, number>({
      query: (id) => ({ url: `/produtos/${id}`, method: 'DELETE' }),
      invalidatesTags: (_, __, id) => [
        { type: 'Produto', id },
        { type: 'Produto', id: 'LIST' },
      ],
    }),
  }),
});

// Hooks gerados automaticamente pelo RTK Query
export const {
  useListarProdutosQuery,
  useBuscarProdutoQuery,
  useCriarProdutoMutation,
  useAtualizarProdutoMutation,
  useDeletarProdutoMutation,
} = api;
```

```tsx
// Usando RTK Query em componentes
function PaginaProdutosRTK() {
  const [pagina, setPagina] = useState(1);
  const { data, isLoading, isFetching } = useListarProdutosQuery({ pagina });
  const [deletar, { isLoading: deletando }] = useDeletarProdutoMutation();

  if (isLoading) return <p>Carregando...</p>;

  return (
    <div>
      {isFetching && <span>Atualizando...</span>}
      <table>
        <thead>
          <tr>
            <th>Nome</th>
            <th>Preço</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          {data?.itens.map((p) => (
            <tr key={p.id}>
              <td>{p.nome}</td>
              <td>R$ {p.preco.toFixed(2)}</td>
              <td>
                <button
                  onClick={() => deletar(p.id)}
                  disabled={deletando}
                  className="btn btn--perigo btn--pequeno"
                >
                  Excluir
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
      <div className="paginacao">
        <button onClick={() => setPagina((p) => p - 1)} disabled={pagina === 1}>Anterior</button>
        <span>Página {pagina}</span>
        <button onClick={() => setPagina((p) => p + 1)}>Próxima</button>
      </div>
    </div>
  );
}
```

---

## 5. Formulários e Validação

### React Hook Form

React Hook Form minimiza re-renders e integra facilmente com validadores externos.

```tsx
import { useForm, SubmitHandler, Controller } from 'react-hook-form';
```

### Validação com Zod

Zod é uma biblioteca de validação com inferência automática de tipos TypeScript.

```tsx
// src/features/products/schemas.ts
import { z } from 'zod';

export const schemaProduto = z.object({
  nome: z.string()
    .min(3, 'Nome deve ter pelo menos 3 caracteres')
    .max(100, 'Nome deve ter no máximo 100 caracteres'),
  preco: z.number({ invalid_type_error: 'Informe um preço válido' })
    .positive('Preço deve ser positivo')
    .multipleOf(0.01, 'Máximo de 2 casas decimais'),
  categoria: z.enum(['eletronicos', 'livros', 'roupas', 'alimentos'], {
    errorMap: () => ({ message: 'Selecione uma categoria válida' }),
  }),
  descricao: z.string().max(500, 'Máximo 500 caracteres').optional(),
  ativo: z.boolean().default(true),
});

export type ProdutoForm = z.infer<typeof schemaProduto>;

// Schema de login
export const schemaLogin = z.object({
  email: z.string().email('E-mail inválido'),
  senha: z.string().min(6, 'Senha deve ter pelo menos 6 caracteres'),
});

export type LoginForm = z.infer<typeof schemaLogin>;

// Schema com validação condicional e refinement
export const schemaRedefinirSenha = z
  .object({
    senhaAtual: z.string().min(6),
    novaSenha: z.string().min(8, 'Nova senha deve ter pelo menos 8 caracteres'),
    confirmarSenha: z.string(),
  })
  .refine((data) => data.novaSenha === data.confirmarSenha, {
    message: 'As senhas não coincidem',
    path: ['confirmarSenha'],
  });

export type RedefinirSenhaForm = z.infer<typeof schemaRedefinirSenha>;
```

```tsx
// src/features/products/FormularioProduto.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { schemaProduto, ProdutoForm } from './schemas';
import { useCriarProduto, useAtualizarProduto } from './useProdutos';

type Props = {
  produtoExistente?: ProdutoForm & { id: number };
  aoSalvar?: () => void;
};

function FormularioProduto({ produtoExistente, aoSalvar }: Props) {
  const criar = useCriarProduto();
  const atualizar = useAtualizarProduto();
  const editando = !!produtoExistente;

  const {
    register,
    handleSubmit,
    reset,
    formState: { errors, isSubmitting, isDirty },
  } = useForm<ProdutoForm>({
    resolver: zodResolver(schemaProduto),
    defaultValues: produtoExistente ?? {
      nome: '',
      preco: 0,
      categoria: 'eletronicos',
      descricao: '',
      ativo: true,
    },
  });

  const onSubmit: SubmitHandler<ProdutoForm> = async (dados) => {
    if (editando) {
      await atualizar.mutateAsync({ id: produtoExistente.id, ...dados });
    } else {
      await criar.mutateAsync(dados);
      reset();
    }
    aoSalvar?.();
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="formulario" noValidate>
      <div className="campo">
        <label htmlFor="nome">Nome *</label>
        <input
          id="nome"
          {...register('nome')}
          className={errors.nome ? 'input input--erro' : 'input'}
          aria-describedby={errors.nome ? 'nome-erro' : undefined}
          aria-invalid={!!errors.nome}
        />
        {errors.nome && (
          <span id="nome-erro" className="campo__erro" role="alert">
            {errors.nome.message}
          </span>
        )}
      </div>

      <div className="campo">
        <label htmlFor="preco">Preço *</label>
        <input
          id="preco"
          type="number"
          step="0.01"
          {...register('preco', { valueAsNumber: true })}
          className={errors.preco ? 'input input--erro' : 'input'}
          aria-invalid={!!errors.preco}
        />
        {errors.preco && (
          <span className="campo__erro" role="alert">{errors.preco.message}</span>
        )}
      </div>

      <div className="campo">
        <label htmlFor="categoria">Categoria *</label>
        <select
          id="categoria"
          {...register('categoria')}
          className={errors.categoria ? 'select select--erro' : 'select'}
        >
          <option value="eletronicos">Eletrônicos</option>
          <option value="livros">Livros</option>
          <option value="roupas">Roupas</option>
          <option value="alimentos">Alimentos</option>
        </select>
        {errors.categoria && (
          <span className="campo__erro" role="alert">{errors.categoria.message}</span>
        )}
      </div>

      <div className="campo">
        <label htmlFor="descricao">Descrição</label>
        <textarea id="descricao" {...register('descricao')} className="textarea" rows={3} />
        {errors.descricao && (
          <span className="campo__erro" role="alert">{errors.descricao.message}</span>
        )}
      </div>

      <div className="campo campo--checkbox">
        <input id="ativo" type="checkbox" {...register('ativo')} />
        <label htmlFor="ativo">Produto ativo</label>
      </div>

      <div className="formulario__acoes">
        <button
          type="submit"
          disabled={isSubmitting || !isDirty}
          className="btn btn--primario"
        >
          {isSubmitting ? 'Salvando...' : editando ? 'Atualizar' : 'Criar Produto'}
        </button>
        <button type="button" onClick={() => reset()} className="btn btn--secundario">
          Limpar
        </button>
      </div>
    </form>
  );
}
```

```tsx
// Formulário de login completo
import { useForm, SubmitHandler } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { useNavigate, useLocation } from 'react-router-dom';
import { schemaLogin, LoginForm } from '../schemas';
import { useAuth } from '../auth/useAuth';

function FormularioLogin() {
  const { entrar } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const destino = (location.state as any)?.from?.pathname ?? '/';

  const {
    register,
    handleSubmit,
    setError,
    formState: { errors, isSubmitting },
  } = useForm<LoginForm>({ resolver: zodResolver(schemaLogin) });

  const onSubmit: SubmitHandler<LoginForm> = async ({ email, senha }) => {
    try {
      await entrar(email, senha);
      navigate(destino, { replace: true });
    } catch {
      // Erro de credenciais — atribui ao campo senha sem revelar qual está errado
      setError('root', { message: 'E-mail ou senha inválidos' });
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="formulario formulario--login" noValidate>
      <h1>Entrar</h1>

      {errors.root && (
        <div className="alerta alerta--erro" role="alert">
          {errors.root.message}
        </div>
      )}

      <div className="campo">
        <label htmlFor="email">E-mail</label>
        <input id="email" type="email" {...register('email')} autoComplete="email" />
        {errors.email && <span className="campo__erro" role="alert">{errors.email.message}</span>}
      </div>

      <div className="campo">
        <label htmlFor="senha">Senha</label>
        <input id="senha" type="password" {...register('senha')} autoComplete="current-password" />
        {errors.senha && <span className="campo__erro" role="alert">{errors.senha.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting} className="btn btn--primario btn--bloco">
        {isSubmitting ? 'Entrando...' : 'Entrar'}
      </button>
    </form>
  );
}
```

---

## 6. Internacionalização com react-i18next

```typescript
// src/i18n/index.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

import ptBR from './locales/pt-BR.json';
import enUS from './locales/en-US.json';
import esES from './locales/es-ES.json';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      'pt-BR': { translation: ptBR },
      'en-US': { translation: enUS },
      'es-ES': { translation: esES },
    },
    fallbackLng: 'pt-BR',
    interpolation: { escapeValue: false }, // React já escapa XSS
    detection: {
      order: ['localStorage', 'navigator'],
      caches: ['localStorage'],
    },
  });

export default i18n;
```

```json
// src/i18n/locales/pt-BR.json
{
  "nav": {
    "inicio": "Início",
    "produtos": "Produtos",
    "perfil": "Perfil",
    "admin": "Administração",
    "entrar": "Entrar",
    "sair": "Sair"
  },
  "produto": {
    "titulo": "Produto",
    "lista": "Lista de Produtos",
    "nome": "Nome",
    "preco": "Preço",
    "categoria": "Categoria",
    "categorias": {
      "eletronicos": "Eletrônicos",
      "livros": "Livros",
      "roupas": "Roupas",
      "alimentos": "Alimentos"
    },
    "acoes": {
      "criar": "Criar Produto",
      "editar": "Editar",
      "excluir": "Excluir",
      "salvar": "Salvar",
      "cancelar": "Cancelar"
    },
    "mensagens": {
      "criado": "Produto criado com sucesso!",
      "atualizado": "Produto atualizado com sucesso!",
      "excluido": "Produto excluído.",
      "confirmarExclusao": "Deseja excluir o produto \"{{nome}}\"?"
    }
  },
  "paginacao": {
    "anterior": "Anterior",
    "proxima": "Próxima",
    "pagina": "Página {{pagina}} de {{total}}"
  },
  "erros": {
    "generico": "Ocorreu um erro. Tente novamente.",
    "naoAutorizado": "Você não tem permissão para acessar este recurso.",
    "naoEncontrado": "Página não encontrada."
  },
  "data": {
    "formato": "dd/MM/yyyy",
    "formatoHora": "dd/MM/yyyy HH:mm"
  }
}
```

```json
// src/i18n/locales/en-US.json
{
  "nav": {
    "inicio": "Home",
    "produtos": "Products",
    "perfil": "Profile",
    "admin": "Administration",
    "entrar": "Sign In",
    "sair": "Sign Out"
  },
  "produto": {
    "titulo": "Product",
    "lista": "Product List",
    "nome": "Name",
    "preco": "Price",
    "categoria": "Category",
    "categorias": {
      "eletronicos": "Electronics",
      "livros": "Books",
      "roupas": "Clothing",
      "alimentos": "Food"
    },
    "acoes": {
      "criar": "Create Product",
      "editar": "Edit",
      "excluir": "Delete",
      "salvar": "Save",
      "cancelar": "Cancel"
    },
    "mensagens": {
      "criado": "Product created successfully!",
      "atualizado": "Product updated successfully!",
      "excluido": "Product deleted.",
      "confirmarExclusao": "Do you want to delete the product \"{{nome}}\"?"
    }
  },
  "paginacao": {
    "anterior": "Previous",
    "proxima": "Next",
    "pagina": "Page {{pagina}} of {{total}}"
  },
  "erros": {
    "generico": "An error occurred. Please try again.",
    "naoAutorizado": "You are not authorized to access this resource.",
    "naoEncontrado": "Page not found."
  },
  "data": {
    "formato": "MM/dd/yyyy",
    "formatoHora": "MM/dd/yyyy hh:mm a"
  }
}
```

```tsx
// src/main.tsx — importar i18n antes do React
import './i18n';
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

```tsx
// Usando i18n em componentes
import { useTranslation } from 'react-i18next';

function ListaProdutosI18n() {
  const { t, i18n } = useTranslation();

  function mudarIdioma(idioma: string) {
    i18n.changeLanguage(idioma);
  }

  return (
    <div>
      <div className="seletor-idioma">
        <button onClick={() => mudarIdioma('pt-BR')}>PT</button>
        <button onClick={() => mudarIdioma('en-US')}>EN</button>
        <button onClick={() => mudarIdioma('es-ES')}>ES</button>
      </div>

      <h1>{t('produto.lista')}</h1>

      <table>
        <thead>
          <tr>
            <th>{t('produto.nome')}</th>
            <th>{t('produto.preco')}</th>
            <th>{t('produto.categoria')}</th>
            <th>{t('produto.acoes.editar')}</th>
          </tr>
        </thead>
      </table>
    </div>
  );
}

// Interpolação com variáveis
function ConfirmacaoExclusao({ nomeProduto }: { nomeProduto: string }) {
  const { t } = useTranslation();
  return (
    <p>{t('produto.mensagens.confirmarExclusao', { nome: nomeProduto })}</p>
  );
}

// Paginação com variáveis
function Paginacao({ pagina, total }: { pagina: number; total: number }) {
  const { t } = useTranslation();
  return (
    <div className="paginacao">
      <button>{t('paginacao.anterior')}</button>
      <span>{t('paginacao.pagina', { pagina, total })}</span>
      <button>{t('paginacao.proxima')}</button>
    </div>
  );
}

// Hook customizado para formatar valores conforme o locale
function useFormatacao() {
  const { i18n } = useTranslation();
  const locale = i18n.language;

  return {
    formatarMoeda: (valor: number) =>
      new Intl.NumberFormat(locale, { style: 'currency', currency: 'BRL' }).format(valor),
    formatarNumero: (valor: number) =>
      new Intl.NumberFormat(locale).format(valor),
    formatarData: (data: Date) =>
      new Intl.DateTimeFormat(locale).format(data),
  };
}
```

---

## 7. Manipulação de Datas com date-fns

date-fns é uma biblioteca modular — importe apenas o que usar para manter o bundle enxuto.

```tsx
// src/utils/datas.ts
import {
  format,
  formatDistanceToNow,
  formatRelative,
  parseISO,
  isValid,
  isBefore,
  isAfter,
  addDays,
  addMonths,
  addYears,
  subDays,
  startOfDay,
  endOfDay,
  startOfMonth,
  endOfMonth,
  differenceInDays,
  differenceInHours,
  isToday,
  isTomorrow,
  isYesterday,
  getYear,
  getMonth,
  eachDayOfInterval,
} from 'date-fns';
import { ptBR, enUS } from 'date-fns/locale';

const locales: Record<string, Locale> = {
  'pt-BR': ptBR,
  'en-US': enUS,
};

function getLocale(idioma: string): Locale {
  return locales[idioma] ?? ptBR;
}

// Formatar data para exibição
export function formatarData(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data;
  if (!isValid(d)) return '—';
  const locale = getLocale(idioma);
  return format(d, idioma === 'en-US' ? 'MM/dd/yyyy' : 'dd/MM/yyyy', { locale });
}

// Formatar data e hora
export function formatarDataHora(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data;
  if (!isValid(d)) return '—';
  const locale = getLocale(idioma);
  return format(d, idioma === 'en-US' ? 'MM/dd/yyyy hh:mm a' : 'dd/MM/yyyy HH:mm', { locale });
}

// "há 3 horas", "em 2 dias"
export function formatarRelativo(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data;
  if (!isValid(d)) return '—';
  return formatDistanceToNow(d, { addSuffix: true, locale: getLocale(idioma) });
}

// Exemplos de uso de manipulação
export const datasUtil = {
  // Intervalo para filtros
  ultimosMeses: (meses: number) => ({
    inicio: startOfDay(addMonths(new Date(), -meses)),
    fim: endOfDay(new Date()),
  }),

  // Dias de um mês para calendário
  diasDoMes: (ano: number, mes: number) => {
    const inicio = startOfMonth(new Date(ano, mes));
    const fim = endOfMonth(new Date(ano, mes));
    return eachDayOfInterval({ start: inicio, end: fim });
  },

  // Validações de negócio
  prazoVencido: (dataVencimento: Date | string): boolean => {
    const d = typeof dataVencimento === 'string' ? parseISO(dataVencimento) : dataVencimento;
    return isBefore(d, startOfDay(new Date()));
  },

  diasRestantes: (dataFutura: Date | string): number => {
    const d = typeof dataFutura === 'string' ? parseISO(dataFutura) : dataFutura;
    return differenceInDays(d, new Date());
  },
};
```

```tsx
// Componente de data com i18n integrado
import { useTranslation } from 'react-i18next';
import { formatarData, formatarDataHora, formatarRelativo } from '../../utils/datas';
import { isToday, isYesterday, isTomorrow, parseISO } from 'date-fns';

type Props = {
  data: string; // ISO 8601 do backend
  formato?: 'data' | 'dataHora' | 'relativo';
};

function DataFormatada({ data, formato = 'data' }: Props) {
  const { i18n } = useTranslation();
  const idioma = i18n.language;

  const texto = (() => {
    switch (formato) {
      case 'dataHora': return formatarDataHora(data, idioma);
      case 'relativo': return formatarRelativo(data, idioma);
      default: return formatarData(data, idioma);
    }
  })();

  return (
    <time dateTime={data} title={formatarDataHora(data, idioma)}>
      {texto}
    </time>
  );
}

// Hook para usar datas em componentes
function useDatas() {
  const { i18n } = useTranslation();
  const idioma = i18n.language;

  return {
    formatar: (data: Date | string) => formatarData(data, idioma),
    formatarHora: (data: Date | string) => formatarDataHora(data, idioma),
    relativo: (data: Date | string) => formatarRelativo(data, idioma),
  };
}

// Exemplo: exibindo prazo de entrega
function PrazoEntrega({ dataEntrega }: { dataEntrega: string }) {
  const { formatar } = useDatas();
  const { t } = useTranslation();
  const data = parseISO(dataEntrega);

  let label = formatar(data);
  let className = 'prazo';

  if (isToday(data)) {
    label = 'Hoje';
    className = 'prazo prazo--hoje';
  } else if (isTomorrow(data)) {
    label = 'Amanhã';
    className = 'prazo prazo--amanha';
  } else if (isYesterday(data)) {
    label = 'Ontem';
    className = 'prazo prazo--vencido';
  }

  return <span className={className}>{label}</span>;
}
```

---

## 8. Autenticação JWT e Controle de Acesso por Role

### Estrutura do Token JWT

O backend emite um JWT com o payload contendo o usuário e suas roles:

```json
{
  "sub": "usuario@email.com",
  "nome": "Maria Silva",
  "roles": ["USER", "GERENTE"],
  "iat": 1700000000,
  "exp": 1700086400
}
```

```typescript
// src/types/auth.ts
export type Role = 'USER' | 'GERENTE' | 'ADMIN';

export type UsuarioJWT = {
  sub: string;      // email / identificador único
  nome: string;
  roles: Role[];
  iat: number;      // issued at
  exp: number;      // expiration
};

export type Usuario = {
  email: string;
  nome: string;
  roles: Role[];
};

export type RespostaLogin = {
  token: string;
  refreshToken?: string;
};
```

---

### Contexto de Autenticação

```tsx
// src/features/auth/AuthContext.tsx
import { createContext, useContext, useState, useEffect, useCallback, ReactNode } from 'react';
import { jwtDecode } from 'jwt-decode';
import type { Usuario, UsuarioJWT, RespostaLogin } from '../../types/auth';
import { http } from '../../services/httpClient';

type AuthContexto = {
  usuario: Usuario | null;
  token: string | null;
  carregando: boolean;
  entrar: (email: string, senha: string) => Promise<void>;
  sair: () => void;
  temRole: (role: string) => boolean;
  temAlgumaRole: (roles: string[]) => boolean;
};

const AuthContext = createContext<AuthContexto | null>(null);

const TOKEN_KEY = 'auth_token';

function tokenValido(token: string): boolean {
  try {
    const { exp } = jwtDecode<UsuarioJWT>(token);
    return Date.now() < exp * 1000;
  } catch {
    return false;
  }
}

function usuarioDo(token: string): Usuario | null {
  try {
    const { sub, nome, roles } = jwtDecode<UsuarioJWT>(token);
    return { email: sub, nome, roles };
  } catch {
    return null;
  }
}

export function ProvedorAuth({ children }: { children: ReactNode }) {
  const [token, setToken] = useState<string | null>(null);
  const [usuario, setUsuario] = useState<Usuario | null>(null);
  const [carregando, setCarregando] = useState(true);

  // Restaura sessão ao iniciar
  useEffect(() => {
    const tokenSalvo = localStorage.getItem(TOKEN_KEY);
    if (tokenSalvo && tokenValido(tokenSalvo)) {
      setToken(tokenSalvo);
      setUsuario(usuarioDo(tokenSalvo));
    } else {
      localStorage.removeItem(TOKEN_KEY);
    }
    setCarregando(false);
  }, []);

  const entrar = useCallback(async (email: string, senha: string) => {
    const resposta = await http.post<RespostaLogin>('/auth/login', { email, senha });
    localStorage.setItem(TOKEN_KEY, resposta.token);
    setToken(resposta.token);
    setUsuario(usuarioDo(resposta.token));
  }, []);

  const sair = useCallback(() => {
    localStorage.removeItem(TOKEN_KEY);
    setToken(null);
    setUsuario(null);
  }, []);

  const temRole = useCallback(
    (role: string) => usuario?.roles.includes(role as any) ?? false,
    [usuario]
  );

  const temAlgumaRole = useCallback(
    (roles: string[]) => roles.some((r) => usuario?.roles.includes(r as any)),
    [usuario]
  );

  return (
    <AuthContext.Provider value={{ usuario, token, carregando, entrar, sair, temRole, temAlgumaRole }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth deve ser usado dentro de ProvedorAuth');
  return ctx;
}
```

---

### Interceptor de Requisições

O cliente HTTP já injeta o token (veja seção 4 — Fetch API). Para lidar com refresh automático de token:

```typescript
// src/services/httpClient.ts (versão com refresh token)
let refreshPromise: Promise<string> | null = null;

async function refreshToken(): Promise<string> {
  const refreshTk = localStorage.getItem('refresh_token');
  if (!refreshTk) throw new Error('Sem refresh token');

  const res = await fetch(`${BASE_URL}/auth/refresh`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ refreshToken: refreshTk }),
  });

  if (!res.ok) {
    localStorage.removeItem('auth_token');
    localStorage.removeItem('refresh_token');
    window.location.href = '/login';
    throw new Error('Sessão expirada');
  }

  const { token } = await res.json();
  localStorage.setItem('auth_token', token);
  return token;
}

async function requestComRefresh<T>(endpoint: string, opcoes: OpcoesFetch = {}): Promise<T> {
  const token = localStorage.getItem('auth_token');
  const headers: HeadersInit = {
    'Content-Type': 'application/json',
    ...(token ? { Authorization: `Bearer ${token}` } : {}),
  };

  const res = await fetch(`${BASE_URL}${endpoint}`, { ...opcoes, headers });

  if (res.status === 401) {
    // Evita múltiplas chamadas de refresh simultâneas
    if (!refreshPromise) {
      refreshPromise = refreshToken().finally(() => { refreshPromise = null; });
    }

    const novoToken = await refreshPromise;
    const novasHeaders: HeadersInit = {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${novoToken}`,
    };
    const retentativa = await fetch(`${BASE_URL}${endpoint}`, { ...opcoes, headers: novasHeaders });

    if (!retentativa.ok) throw new Error('Erro após refresh de token');
    if (retentativa.status === 204) return null as T;
    return retentativa.json() as Promise<T>;
  }

  if (!res.ok) {
    const erro = await res.json().catch(() => ({ mensagem: res.statusText }));
    throw new Error(erro.mensagem ?? 'Erro na requisição');
  }

  if (res.status === 204) return null as T;
  return res.json() as Promise<T>;
}
```

---

### Componentes de Proteção por Role

```tsx
// src/features/auth/AcessoRestrito.tsx
import { ReactNode } from 'react';
import { useAuth } from './useAuth';

type Props = {
  roles?: string[];              // Requer pelo menos uma dessas roles
  todasAsRoles?: string[];       // Requer todas essas roles
  fallback?: ReactNode;          // O que exibir quando sem permissão
  children: ReactNode;
};

/**
 * Renderiza `children` apenas se o usuário tiver a(s) role(s) exigida(s).
 * Exibe `fallback` (ou nada) caso contrário.
 *
 * Exemplos:
 *   <AcessoRestrito roles={['ADMIN']}>...</AcessoRestrito>
 *   <AcessoRestrito roles={['ADMIN', 'GERENTE']} fallback={<p>Sem acesso</p>}>...</AcessoRestrito>
 *   <AcessoRestrito todasAsRoles={['USER', 'GERENTE']}>...</AcessoRestrito>
 */
export function AcessoRestrito({ roles, todasAsRoles, fallback = null, children }: Props) {
  const { usuario, temRole, temAlgumaRole } = useAuth();

  if (!usuario) return <>{fallback}</>;

  const temPermissao =
    (!roles || temAlgumaRole(roles)) &&
    (!todasAsRoles || todasAsRoles.every(temRole));

  return temPermissao ? <>{children}</> : <>{fallback}</>;
}

// Hook para uso em lógica de componentes
export function usePermissao(roles: string[]): boolean {
  const { temAlgumaRole } = useAuth();
  return temAlgumaRole(roles);
}
```

```tsx
// src/features/auth/useAuth.ts — reexporta para conveniência
export { useAuth } from './AuthContext';
export { AcessoRestrito, usePermissao } from './AcessoRestrito';
```

```tsx
// Usando em componentes

// No Layout — botões condicionais
function MenuNavegacao() {
  const { usuario } = useAuth();
  const { t } = useTranslation();

  return (
    <nav>
      <NavLink to="/">{t('nav.inicio')}</NavLink>
      <NavLink to="/produtos">{t('nav.produtos')}</NavLink>

      <AcessoRestrito roles={['USER', 'GERENTE', 'ADMIN']}>
        <NavLink to="/perfil">{t('nav.perfil')}</NavLink>
      </AcessoRestrito>

      <AcessoRestrito roles={['GERENTE', 'ADMIN']}>
        <NavLink to="/relatorios">Relatórios</NavLink>
      </AcessoRestrito>

      <AcessoRestrito roles={['ADMIN']}>
        <NavLink to="/admin">{t('nav.admin')}</NavLink>
      </AcessoRestrito>
    </nav>
  );
}

// Em uma tabela de produtos — ações condicionais por role
function TabelaProdutos({ produtos }: { produtos: Produto[] }) {
  const podeDeletar = usePermissao(['ADMIN']);
  const podeEditar = usePermissao(['ADMIN', 'GERENTE']);
  const { t } = useTranslation();

  return (
    <table>
      <thead>
        <tr>
          <th>{t('produto.nome')}</th>
          <th>{t('produto.preco')}</th>
          {podeEditar && <th>Ações</th>}
        </tr>
      </thead>
      <tbody>
        {produtos.map((p) => (
          <tr key={p.id}>
            <td>{p.nome}</td>
            <td>R$ {p.preco.toFixed(2)}</td>
            {podeEditar && (
              <td className="acoes">
                <button className="btn btn--secundario btn--pequeno">
                  {t('produto.acoes.editar')}
                </button>
                {podeDeletar && (
                  <button className="btn btn--perigo btn--pequeno">
                    {t('produto.acoes.excluir')}
                  </button>
                )}
              </td>
            )}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// Página com verificação de role via hook
function PaginaRelatorios() {
  const { usuario } = useAuth();
  const { t } = useTranslation();

  // Verificação granular por role dentro da página
  const podeExportarCSV = usePermissao(['ADMIN', 'GERENTE']);
  const podeVerDadosFinanceiros = usePermissao(['ADMIN']);

  return (
    <div>
      <h1>Relatórios</h1>

      <section>
        <h2>Vendas por Período</h2>
        {/* acessível para GERENTE e ADMIN */}
      </section>

      <AcessoRestrito
        roles={['ADMIN']}
        fallback={
          <div className="alerta alerta--info">
            <p>{t('erros.naoAutorizado')}</p>
          </div>
        }
      >
        <section>
          <h2>Dados Financeiros</h2>
          {/* somente ADMIN */}
        </section>
      </AcessoRestrito>

      {podeExportarCSV && (
        <button className="btn btn--secundario">Exportar CSV</button>
      )}
    </div>
  );
}
```

```tsx
// App.tsx — montagem completa dos provedores
import './i18n';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { Provider } from 'react-redux';
import { store } from './store';
import { ProvedorAuth } from './features/auth/AuthContext';
import { AppRouter } from './routes/router';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { staleTime: 5 * 60 * 1000, retry: 1, refetchOnWindowFocus: false },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Provider store={store}>
        <ProvedorAuth>
          <AppRouter />
        </ProvedorAuth>
      </Provider>
    </QueryClientProvider>
  );
}

export default App;
```

---

## CSS — Exemplo de Estrutura Sem Lib de Estilos

```css
/* src/index.css — Reset e variáveis globais */

*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --cor-primaria: #2563eb;
  --cor-primaria-hover: #1d4ed8;
  --cor-secundaria: #6b7280;
  --cor-perigo: #dc2626;
  --cor-perigo-hover: #b91c1c;
  --cor-sucesso: #16a34a;
  --cor-aviso: #d97706;
  --cor-info: #0ea5e9;

  --cor-texto: #111827;
  --cor-texto-fraco: #6b7280;
  --cor-fundo: #f9fafb;
  --cor-fundo-card: #ffffff;
  --cor-borda: #e5e7eb;

  --raio-borda: 6px;
  --sombra-card: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);
  --fonte-base: 'Inter', system-ui, -apple-system, sans-serif;
  --tamanho-fonte: 1rem;
  --espacamento: 1rem;
}

body {
  font-family: var(--fonte-base);
  font-size: var(--tamanho-fonte);
  color: var(--cor-texto);
  background-color: var(--cor-fundo);
  line-height: 1.6;
}

/* Layout principal */
.app { display: flex; flex-direction: column; min-height: 100vh; }
.conteudo-principal { flex: 1; max-width: 1200px; width: 100%; margin: 0 auto; padding: 2rem 1rem; }

/* Cabeçalho */
.cabecalho {
  background-color: var(--cor-fundo-card);
  border-bottom: 1px solid var(--cor-borda);
  padding: 0 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 64px;
  position: sticky;
  top: 0;
  z-index: 100;
}

.nav-principal { display: flex; gap: 0.5rem; }

.nav-link {
  padding: 0.5rem 0.75rem;
  border-radius: var(--raio-borda);
  text-decoration: none;
  color: var(--cor-texto-fraco);
  font-weight: 500;
  transition: color 0.15s, background-color 0.15s;
}

.nav-link:hover { color: var(--cor-primaria); background-color: #eff6ff; }
.nav-link--ativo { color: var(--cor-primaria); background-color: #eff6ff; }

/* Botões */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: var(--raio-borda);
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.15s, opacity 0.15s;
  text-decoration: none;
}

.btn:disabled { opacity: 0.5; cursor: not-allowed; }
.btn--bloco { width: 100%; }
.btn--pequeno { padding: 0.25rem 0.625rem; font-size: 0.75rem; }

.btn--primario { background-color: var(--cor-primaria); color: white; }
.btn--primario:hover:not(:disabled) { background-color: var(--cor-primaria-hover); }

.btn--secundario {
  background-color: transparent;
  color: var(--cor-secundaria);
  border: 1px solid var(--cor-borda);
}
.btn--secundario:hover:not(:disabled) { background-color: var(--cor-fundo); }

.btn--perigo { background-color: var(--cor-perigo); color: white; }
.btn--perigo:hover:not(:disabled) { background-color: var(--cor-perigo-hover); }

/* Cards */
.card-produto {
  background-color: var(--cor-fundo-card);
  border: 1px solid var(--cor-borda);
  border-radius: var(--raio-borda);
  padding: 1.25rem;
  box-shadow: var(--sombra-card);
}

/* Formulários */
.formulario { display: flex; flex-direction: column; gap: 1.25rem; max-width: 480px; }

.campo { display: flex; flex-direction: column; gap: 0.375rem; }

.campo label {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--cor-texto);
}

.input, .select, .textarea {
  padding: 0.5rem 0.75rem;
  border: 1px solid var(--cor-borda);
  border-radius: var(--raio-borda);
  font-size: 1rem;
  color: var(--cor-texto);
  background-color: var(--cor-fundo-card);
  width: 100%;
  transition: border-color 0.15s, box-shadow 0.15s;
}

.input:focus, .select:focus, .textarea:focus {
  outline: none;
  border-color: var(--cor-primaria);
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15);
}

.input--erro, .select--erro { border-color: var(--cor-perigo); }
.input--erro:focus, .select--erro:focus { box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.15); }

.campo__erro { font-size: 0.75rem; color: var(--cor-perigo); }

.campo--checkbox {
  flex-direction: row;
  align-items: center;
  gap: 0.5rem;
}

.campo--checkbox input { width: auto; }

/* Alertas */
.alerta {
  padding: 0.875rem 1rem;
  border-radius: var(--raio-borda);
  font-size: 0.875rem;
}

.alerta--erro { background-color: #fee2e2; color: #991b1b; border: 1px solid #fca5a5; }
.alerta--sucesso { background-color: #dcfce7; color: #166534; border: 1px solid #86efac; }
.alerta--info { background-color: #e0f2fe; color: #075985; border: 1px solid #7dd3fc; }
.alerta--aviso { background-color: #fef3c7; color: #92400e; border: 1px solid #fcd34d; }

/* Badges */
.badge {
  display: inline-block;
  padding: 0.125rem 0.5rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
}

.badge--disponivel { background-color: #dcfce7; color: #166534; }
.badge--indisponivel { background-color: #fee2e2; color: #991b1b; }

/* Tabelas */
table { width: 100%; border-collapse: collapse; }
thead { background-color: var(--cor-fundo); }
th {
  padding: 0.75rem 1rem;
  text-align: left;
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--cor-texto-fraco);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  border-bottom: 1px solid var(--cor-borda);
}
td {
  padding: 0.875rem 1rem;
  border-bottom: 1px solid var(--cor-borda);
  font-size: 0.875rem;
}
tr:hover td { background-color: var(--cor-fundo); }

/* Paginação */
.paginacao { display: flex; align-items: center; gap: 0.5rem; margin-top: 1rem; }

/* Carregando */
.carregando-pagina {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 200px;
  color: var(--cor-texto-fraco);
}

/* Responsivo */
@media (max-width: 768px) {
  .conteudo-principal { padding: 1rem; }
  .cabecalho { height: 56px; }
  .nav-link { padding: 0.375rem 0.5rem; font-size: 0.875rem; }
  table { font-size: 0.8125rem; }
  th, td { padding: 0.625rem 0.75rem; }
}
```

---

## Referência Rápida de Versões

| Biblioteca | Versão | Finalidade |
|---|---|---|
| `react` / `react-dom` | 19.x | Framework UI |
| `react-router-dom` | 7.x | Roteamento |
| `@reduxjs/toolkit` | 2.x | Estado global + RTK Query |
| `react-redux` | 9.x | Integração Redux com React |
| `zustand` | 5.x | Estado global leve |
| `@tanstack/react-query` | 5.x | Cache e fetching de dados |
| `react-hook-form` | 7.x | Gerenciamento de formulários |
| `zod` | 3.x | Validação com inferência de tipos |
| `@hookform/resolvers` | 3.x | Integração RHF + Zod |
| `react-i18next` / `i18next` | 15.x / 24.x | Internacionalização |
| `date-fns` | 4.x | Manipulação de datas |
| `jwt-decode` | 4.x | Decodificação de JWT |

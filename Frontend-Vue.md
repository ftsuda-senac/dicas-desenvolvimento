# Vue 3.5 com TypeScript — Referência Completa

Referência consolidada sobre Vue 3.5 com TypeScript e Composition API, cobrindo fundamentos, roteamento com Vue Router, gerenciamento de estado com Pinia, integração com backend, formulários com VeeValidate e Zod, internacionalização, manipulação de datas e autenticação JWT com controle de acesso por roles.

---

## Sumário

1. [Fundamentos do Vue 3.5](#1-fundamentos-do-vue-35)
   - [Configuração do Projeto](#configuração-do-projeto)
   - [Single File Components e script setup](#single-file-components-e-script-setup)
   - [Reatividade — ref, reactive, computed](#reatividade--ref-reactive-computed)
   - [Props, Emits e defineModel](#props-emits-e-definemodel)
   - [Watch e watchEffect](#watch-e-watcheffect)
   - [Composables — Hooks Reutilizáveis](#composables--hooks-reutilizáveis)
   - [Diretivas Essenciais](#diretivas-essenciais)
   - [Slots e Provide/Inject](#slots-e-provideinject)
   - [Ciclo de Vida](#ciclo-de-vida)
2. [Gerenciamento de Rotas com Vue Router 4](#2-gerenciamento-de-rotas-com-vue-router-4)
   - [Configuração do Router](#configuração-do-router)
   - [Navigation Guards](#navigation-guards)
   - [Navegação Programática](#navegação-programática)
3. [Estado Global com Pinia](#3-estado-global-com-pinia)
   - [Definindo Stores](#definindo-stores)
   - [Stores com Composition API](#stores-com-composition-api)
4. [Integração com Backend](#4-integração-com-backend)
   - [Serviço HTTP com Fetch](#serviço-http-com-fetch)
   - [Composable useFetch](#composable-usefetch)
   - [TanStack Query para Vue](#tanstack-query-para-vue)
5. [Formulários e Validação](#5-formulários-e-validação)
   - [VeeValidate com Zod](#veevalidate-com-zod)
   - [Formulários com defineModel](#formulários-com-definemodel)
6. [Internacionalização com Vue I18n](#6-internacionalização-com-vue-i18n)
7. [Manipulação de Datas com date-fns](#7-manipulação-de-datas-com-date-fns)
8. [Autenticação JWT e Controle de Acesso por Role](#8-autenticação-jwt-e-controle-de-acesso-por-role)
   - [Estrutura do Token JWT](#estrutura-do-token-jwt)
   - [Store de Autenticação com Pinia](#store-de-autenticação-com-pinia)
   - [Interceptor de Requisições](#interceptor-de-requisições)
   - [Guards de Rota por Role](#guards-de-rota-por-role)
   - [Diretiva e Componente de Acesso por Role](#diretiva-e-componente-de-acesso-por-role)

---

## 1. Fundamentos do Vue 3.5

### Configuração do Projeto

```bash
# Criar projeto com Vite + Vue 3 + TypeScript
npm create vue@latest
# Escolher: TypeScript ✔ | Vue Router ✔ | Pinia ✔ | Vitest ✔ | ESLint ✔

cd meu-projeto
npm install

# Instalar dependências principais abordadas neste guia
npm install vue-router pinia
npm install vee-validate @vee-validate/zod zod
npm install vue-i18n@9
npm install date-fns
npm install jwt-decode
npm install @tanstack/vue-query
```

Estrutura recomendada de pastas:

```
src/
├── assets/
├── components/
│   ├── common/
│   └── layout/
├── composables/
├── directives/
├── features/
│   ├── auth/
│   ├── produtos/
│   └── admin/
├── i18n/
│   └── locales/
│       ├── pt-BR.json
│       └── en-US.json
├── router/
├── services/
├── stores/
├── types/
├── utils/
├── App.vue
└── main.ts
```

```typescript
// src/main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import { VueQueryPlugin } from '@tanstack/vue-query'
import App from './App.vue'
import { router } from './router'
import { i18n } from './i18n'
import { vAcessoRestrito } from './directives/acesso-restrito'
import './assets/styles.css'

const app = createApp(App)

app.use(createPinia())
app.use(router)
app.use(i18n)
app.use(VueQueryPlugin, {
  queryClientConfig: {
    defaultOptions: {
      queries: {
        staleTime: 5 * 60 * 1000,
        retry: 1,
        refetchOnWindowFocus: false,
      },
    },
  },
})

app.directive('acesso-restrito', vAcessoRestrito)

app.mount('#app')
```

---

### Single File Components e script setup

Um **Single File Component (SFC)** reúne template, lógica e estilo num único arquivo `.vue`. O `<script setup>` é o modo Composition API mais conciso — tudo declarado nele fica automaticamente disponível no template.

```vue
<!-- src/components/common/CardProduto.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import type { Produto } from '@/types/produto'

// Props tipadas com defineProps
const props = defineProps<{
  produto: Produto
  destacado?: boolean
}>()

// Emits tipados com defineEmits
const emit = defineEmits<{
  editar:  [produto: Produto]
  excluir: [id: number]
}>()

const { t, n } = useI18n()

const badgeClass = computed(() =>
  props.produto.ativo ? 'badge badge--sucesso' : 'badge badge--erro'
)

const precoFormatado = computed(() =>
  n(props.produto.preco, 'currency')
)
</script>

<template>
  <article class="card-produto" :class="{ 'card-produto--destacado': destacado }">
    <h2 class="card-produto__titulo">{{ produto.nome }}</h2>
    <p class="card-produto__preco">{{ precoFormatado }}</p>
    <p class="card-produto__categoria">
      {{ t(`produto.categorias.${produto.categoria}`) }}
    </p>
    <span :class="badgeClass">
      {{ produto.ativo ? t('produto.ativo') : t('produto.inativo') }}
    </span>
    <div class="card-produto__acoes">
      <button class="btn btn--secundario btn--pequeno" @click="emit('editar', produto)">
        {{ t('produto.acoes.editar') }}
      </button>
      <button class="btn btn--perigo btn--pequeno" @click="emit('excluir', produto.id)">
        {{ t('produto.acoes.excluir') }}
      </button>
    </div>
  </article>
</template>
```

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { onMounted } from 'vue'
import { useAuthStore } from '@/stores/auth'
import NavbarComponent from '@/components/layout/NavbarComponent.vue'

const { locale } = useI18n()
const auth = useAuthStore()

onMounted(() => {
  const idiomaSalvo = localStorage.getItem('idioma') ?? 'pt-BR'
  locale.value = idiomaSalvo
  auth.restaurarSessao()
})
</script>

<template>
  <div class="app">
    <NavbarComponent />
    <main class="conteudo-principal">
      <RouterView />
    </main>
    <footer class="rodape">
      <p>&copy; 2025 Minha Aplicação</p>
    </footer>
  </div>
</template>
```

---

### Reatividade — ref, reactive, computed

```vue
<script setup lang="ts">
import { ref, reactive, computed, readonly } from 'vue'

// ref — para valores primitivos ou qualquer valor único
const contagem = ref(0)
const nome     = ref('')
const ativo    = ref(true)

// Acessa e modifica com .value no script; sem .value no template
contagem.value++

// reactive — para objetos/arrays (acesso direto sem .value)
const filtros = reactive({
  busca:     '',
  categoria: 'todos',
  ativo:     true,
  pagina:    1,
})

// Atualizar propriedade do reactive
filtros.busca = 'notebook'

// computed — derivado reativo, recalcula automaticamente
const totalCaracteres = computed(() => nome.value.length)
const podeSalvar      = computed(() => nome.value.length >= 3 && ativo.value)

// computed com getter e setter
const nomeCompleto = computed({
  get: () => `${filtros.busca} ${filtros.categoria}`,
  set: (valor: string) => {
    const [busca, categoria] = valor.split(' ')
    filtros.busca     = busca
    filtros.categoria = categoria
  },
})

// readonly — expõe estado sem permitir mutação externa
const contagemPublica = readonly(contagem)
</script>

<template>
  <div>
    <!-- No template, ref é desembrulhado automaticamente (sem .value) -->
    <p>Contagem: {{ contagem }}</p>
    <p>Pode salvar: {{ podeSalvar }}</p>

    <input v-model="nome" placeholder="Digite seu nome" class="input" />
    <input v-model="filtros.busca" placeholder="Buscar..." class="input" />

    <button class="btn btn--primario" @click="contagem++">Incrementar</button>
    <button class="btn btn--secundario" @click="contagem = 0">Resetar</button>
  </div>
</template>
```

---

### Props, Emits e defineModel

```vue
<!-- Componente filho com defineModel (Vue 3.4+, estável no 3.5) -->
<!-- src/components/common/CampoTexto.vue -->
<script setup lang="ts">
// defineModel cria um v-model bidirecional automaticamente
const modelo = defineModel<string>({ default: '' })

// v-model nomeado para múltiplos modelos
const modeloNome  = defineModel<string>('nome',  { required: true })
const modeloEmail = defineModel<string>('email', { default: '' })

// Props adicionais além do model
const props = defineProps<{
  label:        string
  placeholder?: string
  erro?:        string
  obrigatorio?: boolean
  tipo?:        'text' | 'email' | 'password' | 'number'
}>()

const id = `campo-${Math.random().toString(36).slice(2, 9)}`
</script>

<template>
  <div class="campo">
    <label :for="id">
      {{ label }}
      <span v-if="obrigatorio" class="campo__obrigatorio" aria-hidden="true">*</span>
    </label>
    <input
      :id="id"
      v-model="modelo"
      :type="tipo ?? 'text'"
      :placeholder="placeholder"
      class="input"
      :class="{ 'input--erro': erro }"
      :aria-describedby="erro ? `${id}-erro` : undefined"
      :aria-invalid="!!erro"
    />
    <span v-if="erro" :id="`${id}-erro`" class="campo__erro" role="alert">
      {{ erro }}
    </span>
  </div>
</template>
```

```vue
<!-- Uso do defineModel no componente pai -->
<script setup lang="ts">
import { ref } from 'vue'
import CampoTexto from '@/components/common/CampoTexto.vue'

const email = ref('')
const senha = ref('')
</script>

<template>
  <!-- v-model simples -->
  <CampoTexto v-model="email" label="E-mail" tipo="email" obrigatorio />
  <CampoTexto v-model="senha" label="Senha"  tipo="password" obrigatorio />
  <p>Email digitado: {{ email }}</p>
</template>
```

```vue
<!-- Props com valores padrão (desestruturação reativa — Vue 3.5) -->
<script setup lang="ts">
// Desestruturação reativa de props, estável no Vue 3.5
// As variáveis mantêm reatividade mesmo desestruturadas
const {
  titulo,
  subtitulo = 'Sem subtítulo',
  itens = [],
  carregando = false,
} = defineProps<{
  titulo:      string
  subtitulo?:  string
  itens?:      string[]
  carregando?: boolean
}>()
</script>

<template>
  <div>
    <h1>{{ titulo }}</h1>
    <p>{{ subtitulo }}</p>
    <div v-if="carregando" class="carregando-pagina">
      <div class="spinner"></div>
    </div>
    <ul v-else>
      <li v-for="item in itens" :key="item">{{ item }}</li>
    </ul>
  </div>
</template>
```

---

### Watch e watchEffect

```vue
<script setup lang="ts">
import { ref, watch, watchEffect, onWatcherCleanup } from 'vue'

const busca   = ref('')
const pagina  = ref(1)
const usuario = ref({ nome: '', email: '' })

// watch — reage a uma fonte específica
watch(busca, (novoValor, valorAnterior) => {
  console.log(`Busca: "${valorAnterior}" → "${novoValor}"`)
  pagina.value = 1 // reseta paginação ao buscar
})

// watch com múltiplas fontes
watch([busca, pagina], ([novaBusca, novaPagina]) => {
  console.log(`Buscando "${novaBusca}" na página ${novaPagina}`)
})

// watch profundo — detecta mudanças em objetos aninhados
watch(
  usuario,
  (novo) => console.log('Usuário alterado:', novo),
  { deep: true }
)

// watch imediato — executa ao montar e em cada mudança
watch(
  busca,
  (valor) => buscarProdutos(valor),
  { immediate: true }
)

// watchEffect — rastreia dependências automaticamente
// Vue 3.5: onWatcherCleanup substitui o callback de limpeza
watchEffect(async () => {
  const termoBusca = busca.value  // dependência rastreada automaticamente
  const controller = new AbortController()

  // onWatcherCleanup é chamado antes da próxima execução ou ao desmontar
  onWatcherCleanup(() => controller.abort())

  try {
    const res = await fetch(`/api/produtos?busca=${termoBusca}`, {
      signal: controller.signal,
    })
    // processar resultado
  } catch (e) {
    if ((e as Error).name !== 'AbortError') console.error(e)
  }
})

async function buscarProdutos(termo: string) {
  // implementação
}
</script>
```

---

### Composables — Hooks Reutilizáveis

Composables são funções que encapsulam lógica reativa e podem ser reutilizadas em qualquer componente.

```typescript
// src/composables/usePaginacao.ts
import { ref, computed } from 'vue'
import { useRouter, useRoute } from 'vue-router'

export function usePaginacao(totalPaginas: () => number) {
  const router = useRouter()
  const route  = useRoute()

  const pagina = ref(Number(route.query.pagina ?? 1))

  const podeVoltar  = computed(() => pagina.value > 1)
  const podeAvancar = computed(() => pagina.value < totalPaginas())

  function irPara(p: number) {
    pagina.value = p
    router.push({ query: { ...route.query, pagina: p } })
  }

  function anterior() { if (podeVoltar.value)  irPara(pagina.value - 1) }
  function proxima()  { if (podeAvancar.value) irPara(pagina.value + 1) }

  return { pagina, podeVoltar, podeAvancar, irPara, anterior, proxima }
}
```

```typescript
// src/composables/useConfirmacao.ts
import { ref } from 'vue'

type OpcoeConfirmacao = {
  titulo:   string
  mensagem: string
  labelConfirmar?: string
  labelCancelar?:  string
}

export function useConfirmacao() {
  const visivel  = ref(false)
  const opcoes   = ref<OpcoeConfirmacao | null>(null)
  let resolvePromise: ((valor: boolean) => void) | null = null

  function confirmar(config: OpcoeConfirmacao): Promise<boolean> {
    opcoes.value  = config
    visivel.value = true
    return new Promise((resolve) => { resolvePromise = resolve })
  }

  function aceitar()  { resolvePromise?.(true);  fechar() }
  function cancelar() { resolvePromise?.(false); fechar() }
  function fechar()   { visivel.value = false; opcoes.value = null }

  return { visivel, opcoes, confirmar, aceitar, cancelar }
}
```

```typescript
// src/composables/useLocalStorage.ts
import { ref, watch } from 'vue'

export function useLocalStorage<T>(chave: string, valorPadrao: T) {
  const armazenado = localStorage.getItem(chave)
  const dados = ref<T>(armazenado ? JSON.parse(armazenado) : valorPadrao)

  watch(
    dados,
    (novo) => localStorage.setItem(chave, JSON.stringify(novo)),
    { deep: true }
  )

  function remover() {
    localStorage.removeItem(chave)
    dados.value = valorPadrao
  }

  return { dados, remover }
}

// Uso: const { dados: preferencias } = useLocalStorage('prefs', { tema: 'claro' })
```

```typescript
// src/composables/useTemplateRef.ts — useTemplateRef (Vue 3.5)
import { useTemplateRef, onMounted } from 'vue'

// Dentro de um componente com <input ref="campoRef" />
export function useFocoAutomatico() {
  const campoRef = useTemplateRef<HTMLInputElement>('campoRef')

  onMounted(() => {
    campoRef.value?.focus()
  })

  return { campoRef }
}
```

---

### Diretivas Essenciais

```vue
<script setup lang="ts">
import { ref } from 'vue'

const mostrar    = ref(true)
const lista      = ref(['Maçã', 'Banana', 'Laranja'])
const cor        = ref('primaria')
const estilo     = ref({ fontWeight: 'bold', color: '#1d4ed8' })
const disabled   = ref(false)
const url        = ref('https://exemplo.com')
const mensagem   = ref('Olá <strong>Mundo</strong>')
</script>

<template>
  <!-- v-if / v-else-if / v-else — renderização condicional (remove do DOM) -->
  <div v-if="mostrar">Visível</div>
  <div v-else>Oculto</div>

  <!-- v-show — alterna display CSS (permanece no DOM) -->
  <div v-show="mostrar">Alterna visibilidade</div>

  <!-- v-for com :key obrigatório -->
  <ul>
    <li v-for="(item, index) in lista" :key="item">
      {{ index + 1 }}. {{ item }}
    </li>
  </ul>

  <!-- v-for em objeto -->
  <dl>
    <template v-for="(valor, chave) in { nome: 'Ana', cargo: 'Dev' }" :key="chave">
      <dt>{{ chave }}</dt>
      <dd>{{ valor }}</dd>
    </template>
  </dl>

  <!-- v-bind (ou :) — vinculação de atributos -->
  <div :class="`btn--${cor}`">Classe dinâmica</div>
  <div :class="{ 'btn--ativo': mostrar, 'btn--desabilitado': disabled }">Classes condicionais</div>
  <div :style="estilo">Estilo inline</div>
  <a :href="url">Link dinâmico</a>
  <button :disabled="disabled">Botão</button>

  <!-- v-model — sincronização bidirecional -->
  <input v-model.trim="lista[0]"   class="input" />   <!-- .trim remove espaços -->
  <input v-model.number="lista[1]" class="input" />   <!-- .number converte para número -->
  <input v-model.lazy="lista[2]"   class="input" />   <!-- .lazy sincroniza no blur -->

  <!-- v-on (ou @) — eventos -->
  <button @click="mostrar = !mostrar">Toggle</button>
  <form @submit.prevent="console.log('enviado')">   <!-- .prevent chama preventDefault() -->
    <input @keydown.enter="console.log('Enter')" />  <!-- modificadores de tecla -->
    <input @keydown.ctrl.s.prevent="console.log('Ctrl+S')" />
  </form>

  <!-- v-html — renderiza HTML (cuidado com XSS) -->
  <div v-html="mensagem"></div>

  <!-- v-once — renderiza uma vez, sem reatividade -->
  <p v-once>Valor imutável: {{ lista[0] }}</p>
</template>
```

---

### Slots e Provide/Inject

```vue
<!-- src/components/common/Painel.vue -->
<script setup lang="ts">
defineProps<{
  titulo:   string
  carregando?: boolean
}>()
</script>

<template>
  <div class="painel">
    <header class="painel__cabecalho">
      <!-- slot nomeado para o cabeçalho -->
      <slot name="cabecalho">
        <h2>{{ titulo }}</h2>
      </slot>
      <!-- slot nomeado para ações do cabeçalho -->
      <div class="painel__acoes-cabecalho">
        <slot name="acoes" />
      </div>
    </header>

    <div class="painel__corpo">
      <div v-if="carregando" class="carregando-pagina">
        <div class="spinner"></div>
      </div>
      <!-- slot padrão para o conteúdo -->
      <slot v-else />
    </div>

    <footer v-if="$slots.rodape" class="painel__rodape">
      <slot name="rodape" />
    </footer>
  </div>
</template>
```

```vue
<!-- Usando o componente Painel com slots -->
<template>
  <Painel titulo="Produtos" :carregando="carregando">
    <template #cabecalho>
      <h1>Lista de Produtos</h1>
    </template>
    <template #acoes>
      <button class="btn btn--primario">Novo Produto</button>
    </template>

    <!-- conteúdo do slot padrão -->
    <p>Total: {{ lista.length }} produtos</p>

    <template #rodape>
      <p>Última atualização: agora</p>
    </template>
  </Painel>
</template>
```

```typescript
// Provide/Inject — compartilha dados sem prop drilling
// src/composables/useTema.ts
import { provide, inject, ref, type InjectionKey } from 'vue'

type Tema = 'claro' | 'escuro'
type TemaContexto = { tema: Ref<Tema>; alternar: () => void }

export const TEMA_KEY: InjectionKey<TemaContexto> = Symbol('tema')

// No componente raiz ou pai
export function providerTema() {
  const tema = ref<Tema>('claro')
  const alternar = () => { tema.value = tema.value === 'claro' ? 'escuro' : 'claro' }
  provide(TEMA_KEY, { tema, alternar })
}

// Em qualquer componente filho (qualquer nível)
export function useTema() {
  const ctx = inject(TEMA_KEY)
  if (!ctx) throw new Error('useTema deve ser usado dentro do provedor de tema')
  return ctx
}
```

---

### Ciclo de Vida

```vue
<script setup lang="ts">
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
} from 'vue'

// onMounted — DOM disponível, ideal para fetch inicial
onMounted(() => {
  console.log('Componente montado')
  // buscar dados, inicializar bibliotecas DOM
})

// onUnmounted — limpeza de recursos
onUnmounted(() => {
  // cancelar timers, fechar conexões WebSocket, etc.
})

// onBeforeUnmount — antes da desmontagem (DOM ainda disponível)
onBeforeUnmount(() => {
  // última chance de salvar estado
})
</script>
```

---

## 2. Gerenciamento de Rotas com Vue Router 4

### Configuração do Router

```typescript
// src/router/index.ts
import {
  createRouter,
  createWebHistory,
  type RouteRecordRaw,
} from 'vue-router'
import { useAuthStore } from '@/stores/auth'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('@/components/layout/LayoutPrincipal.vue'),
    children: [
      {
        path: '',
        name: 'inicio',
        component: () => import('@/features/home/HomeView.vue'),
        meta: { title: 'Início' },
      },
      {
        path: 'produtos',
        children: [
          {
            path: '',
            name: 'produtos',
            component: () => import('@/features/produtos/ListaProdutosView.vue'),
            meta: { title: 'Produtos' },
          },
          {
            path: ':id',
            name: 'produto-detalhe',
            component: () => import('@/features/produtos/DetalheProdutoView.vue'),
            props: true,  // passa parâmetros de rota como props
          },
          {
            path: ':id/editar',
            name: 'produto-editar',
            component: () => import('@/features/produtos/FormularioProdutoView.vue'),
            props: true,
            meta: { requerAutenticacao: true, roles: ['ADMIN', 'GERENTE'] },
          },
        ],
      },
      {
        path: 'painel',
        meta: { requerAutenticacao: true },
        children: [
          {
            path: 'perfil',
            name: 'perfil',
            component: () => import('@/features/perfil/PerfilView.vue'),
            meta: { title: 'Meu Perfil' },
          },
          {
            path: 'admin',
            name: 'admin',
            component: () => import('@/features/admin/AdminView.vue'),
            meta: { title: 'Administração', roles: ['ADMIN'] },
          },
          {
            path: 'relatorios',
            name: 'relatorios',
            component: () => import('@/features/relatorios/RelatoriosView.vue'),
            meta: { title: 'Relatórios', roles: ['ADMIN', 'GERENTE'] },
          },
        ],
      },
    ],
  },
  {
    path: '/login',
    name: 'login',
    component: () => import('@/features/auth/LoginView.vue'),
    meta: { title: 'Entrar' },
  },
  {
    path: '/403',
    name: 'acesso-negado',
    component: () => import('@/components/common/AcessoNegadoView.vue'),
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'nao-encontrado',
    component: () => import('@/components/common/NaoEncontradoView.vue'),
  },
]

export const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) return savedPosition
    return { top: 0, behavior: 'smooth' }
  },
})
```

```typescript
// Extensão de tipagem para meta das rotas
// src/types/router.d.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    title?:                string
    requerAutenticacao?:   boolean
    roles?:                string[]
  }
}
```

```vue
<!-- src/components/layout/LayoutPrincipal.vue -->
<script setup lang="ts">
import { RouterLink, RouterView, useLink } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useI18n } from 'vue-i18n'
import SeletorIdioma from '@/components/common/SeletorIdioma.vue'

const auth = useAuthStore()
const { t } = useI18n()
</script>

<template>
  <div class="app">
    <header class="cabecalho">
      <nav class="nav-principal" aria-label="Navegação principal">
        <RouterLink to="/" class="nav-link" active-class="nav-link--ativo" exact-active-class="nav-link--ativo">
          {{ t('nav.inicio') }}
        </RouterLink>
        <RouterLink to="/produtos" class="nav-link" active-class="nav-link--ativo">
          {{ t('nav.produtos') }}
        </RouterLink>
        <RouterLink
          v-if="auth.autenticado"
          to="/painel/perfil"
          class="nav-link"
          active-class="nav-link--ativo"
        >
          {{ t('nav.perfil') }}
        </RouterLink>
        <RouterLink
          v-if="auth.temAlgumaRole(['ADMIN', 'GERENTE'])"
          to="/painel/relatorios"
          class="nav-link"
          active-class="nav-link--ativo"
        >
          Relatórios
        </RouterLink>
        <RouterLink
          v-if="auth.temRole('ADMIN')"
          to="/painel/admin"
          class="nav-link"
          active-class="nav-link--ativo"
        >
          {{ t('nav.admin') }}
        </RouterLink>
      </nav>

      <div class="cabecalho__acoes">
        <SeletorIdioma />
        <template v-if="auth.autenticado">
          <span class="usuario-nome">{{ auth.usuario?.nome }}</span>
          <button class="btn btn--secundario" @click="auth.sair()">
            {{ t('nav.sair') }}
          </button>
        </template>
        <RouterLink v-else to="/login" class="btn btn--primario">
          {{ t('nav.entrar') }}
        </RouterLink>
      </div>
    </header>

    <main class="conteudo-principal">
      <RouterView v-slot="{ Component, route }">
        <Transition name="fade" mode="out-in">
          <component :is="Component" :key="route.path" />
        </Transition>
      </RouterView>
    </main>

    <footer class="rodape">
      <p>&copy; 2025 Minha Aplicação</p>
    </footer>
  </div>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active { transition: opacity .15s ease; }
.fade-enter-from,
.fade-leave-to     { opacity: 0; }
</style>
```

---

### Navigation Guards

```typescript
// src/router/index.ts — adicionar guards globais após createRouter

router.beforeEach((to, from) => {
  // Atualizar título da aba
  document.title = to.meta.title ? `${to.meta.title} — Minha App` : 'Minha App'

  const auth = useAuthStore()

  // Verificar autenticação (herança: rota ou pai com meta.requerAutenticacao)
  const requerAuth = to.matched.some((r) => r.meta.requerAutenticacao)
  if (requerAuth && !auth.autenticado) {
    return { name: 'login', query: { returnUrl: to.fullPath } }
  }

  // Verificar roles (herança: pega a rota mais específica que define roles)
  const rolesExigidas = to.meta.roles
  if (rolesExigidas?.length && !auth.temAlgumaRole(rolesExigidas)) {
    return { name: 'acesso-negado' }
  }

  return true
})

// Guard dentro de um componente (per-route)
// src/features/admin/AdminView.vue
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

onBeforeRouteLeave((to, from) => {
  const confirmado = window.confirm('Tem certeza que deseja sair? Alterações não salvas serão perdidas.')
  if (!confirmado) return false
})

onBeforeRouteUpdate(async (to) => {
  const novoId = to.params.id
  // recarregar dados quando o id mudar sem desmontar o componente
})
```

---

### Navegação Programática

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route  = useRoute()

// Parâmetros e query como props (via props: true na rota)
const props = defineProps<{ id?: string }>()

// Ou lidos diretamente da rota
const id       = route.params.id as string
const pagina   = Number(route.query.pagina ?? 1)
const returnUrl = route.query.returnUrl as string | undefined

function irParaProduto(id: number) {
  router.push({ name: 'produto-detalhe', params: { id } })
}

function buscarPagina(p: number) {
  router.push({ query: { ...route.query, pagina: p } })
}

function voltarOuInicio() {
  if (returnUrl) {
    router.push(returnUrl)
  } else {
    router.push('/')
  }
}

function substituirSemHistorico() {
  // replace não adiciona à pilha de histórico
  router.replace({ name: 'inicio' })
}
</script>
```

---

## 3. Estado Global com Pinia

### Definindo Stores

```typescript
// src/stores/produtos.ts — Options Store (semelhante ao Options API)
import { defineStore } from 'pinia'
import type { Produto, PaginaResposta } from '@/types/produto'
import { produtosService } from '@/services/produtos.service'

export const useProdutosStore = defineStore('produtos', {
  state: (): {
    itens: Produto[]
    total: number
    pagina: number
    totalPaginas: number
    carregando: boolean
    salvando: boolean
    erro: string | null
    filtroBusca: string
  } => ({
    itens: [],
    total: 0,
    pagina: 1,
    totalPaginas: 1,
    carregando: false,
    salvando: false,
    erro: null,
    filtroBusca: '',
  }),

  getters: {
    produtosFiltrados: (state) =>
      state.filtroBusca
        ? state.itens.filter((p) =>
            p.nome.toLowerCase().includes(state.filtroBusca.toLowerCase())
          )
        : state.itens,

    totalAtivos: (state) => state.itens.filter((p) => p.ativo).length,
  },

  actions: {
    async carregar(pagina = 1, busca?: string) {
      this.carregando = true
      this.erro       = null
      try {
        const dados = await produtosService.listar(pagina, busca)
        this.itens        = dados.itens
        this.total        = dados.total
        this.pagina       = dados.pagina
        this.totalPaginas = dados.totalPaginas
      } catch (e) {
        this.erro = e instanceof Error ? e.message : 'Erro ao carregar produtos'
      } finally {
        this.carregando = false
      }
    },

    async criar(dados: Omit<Produto, 'id'>) {
      this.salvando = true
      this.erro     = null
      try {
        const novo = await produtosService.criar(dados)
        this.itens.push(novo)
        return novo
      } catch (e) {
        this.erro = e instanceof Error ? e.message : 'Erro ao criar produto'
        throw e
      } finally {
        this.salvando = false
      }
    },

    async atualizar(produto: Produto) {
      this.salvando = true
      try {
        const atualizado = await produtosService.atualizar(produto)
        const idx = this.itens.findIndex((p) => p.id === produto.id)
        if (idx !== -1) this.itens[idx] = atualizado
        return atualizado
      } catch (e) {
        this.erro = e instanceof Error ? e.message : 'Erro ao atualizar produto'
        throw e
      } finally {
        this.salvando = false
      }
    },

    async excluir(id: number) {
      await produtosService.excluir(id)
      this.itens = this.itens.filter((p) => p.id !== id)
    },

    setFiltro(busca: string) {
      this.filtroBusca = busca
    },
  },
})
```

---

### Stores com Composition API

```typescript
// src/stores/carrinho.ts — Setup Store (Composition API, mais flexível)
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { Produto } from '@/types/produto'

type ItemCarrinho = Produto & { quantidade: number }

export const useCarrinhoStore = defineStore(
  'carrinho',
  () => {
    // state como refs
    const itens  = ref<ItemCarrinho[]>([])
    const aberto = ref(false)

    // getters como computed
    const totalItens = computed(() =>
      itens.value.reduce((acc, i) => acc + i.quantidade, 0)
    )
    const totalValor = computed(() =>
      itens.value.reduce((acc, i) => acc + i.preco * i.quantidade, 0)
    )
    const vazio = computed(() => itens.value.length === 0)

    // actions como funções
    function adicionarItem(produto: Produto) {
      const idx = itens.value.findIndex((i) => i.id === produto.id)
      if (idx !== -1) {
        itens.value[idx].quantidade++
      } else {
        itens.value.push({ ...produto, quantidade: 1 })
      }
    }

    function removerItem(id: number) {
      itens.value = itens.value.filter((i) => i.id !== id)
    }

    function alterarQuantidade(id: number, quantidade: number) {
      if (quantidade <= 0) { removerItem(id); return }
      const item = itens.value.find((i) => i.id === id)
      if (item) item.quantidade = quantidade
    }

    function limpar()         { itens.value = [] }
    function toggleAberto()   { aberto.value = !aberto.value }

    return {
      itens, aberto,
      totalItens, totalValor, vazio,
      adicionarItem, removerItem, alterarQuantidade, limpar, toggleAberto,
    }
  },
  {
    persist: true,  // persiste no localStorage via pinia-plugin-persistedstate
  }
)
```

```vue
<!-- Usando stores Pinia nos componentes -->
<script setup lang="ts">
import { onMounted } from 'vue'
import { storeToRefs } from 'pinia'
import { useProdutosStore } from '@/stores/produtos'
import { useCarrinhoStore } from '@/stores/carrinho'
import CardProduto from '@/components/common/CardProduto.vue'

const produtosStore = useProdutosStore()
const carrinho      = useCarrinhoStore()

// storeToRefs mantém a reatividade ao desestruturar
const { produtosFiltrados, carregando, erro, pagina, totalPaginas } =
  storeToRefs(produtosStore)

// Ações não precisam de storeToRefs
const { carregar, setFiltro } = produtosStore

onMounted(() => carregar())
</script>

<template>
  <div class="pagina-produtos">
    <input
      class="input"
      :value="produtosStore.filtroBusca"
      @input="setFiltro(($event.target as HTMLInputElement).value)"
      placeholder="Buscar produtos..."
    />

    <div v-if="carregando" class="carregando-pagina">
      <div class="spinner"></div>
    </div>

    <div v-else-if="erro" class="alerta alerta--erro" role="alert">{{ erro }}</div>

    <ul v-else class="lista-produtos">
      <li v-for="produto in produtosFiltrados" :key="produto.id">
        <CardProduto
          :produto="produto"
          @editar="(p) => console.log('editar', p)"
          @excluir="produtosStore.excluir($event)"
        />
        <button class="btn btn--primario btn--pequeno" @click="carrinho.adicionarItem(produto)">
          Adicionar ao carrinho
        </button>
      </li>
    </ul>

    <div class="paginacao">
      <button class="btn btn--secundario" :disabled="pagina === 1" @click="carregar(pagina - 1)">
        Anterior
      </button>
      <span>Página {{ pagina }} de {{ totalPaginas }}</span>
      <button class="btn btn--secundario" :disabled="pagina === totalPaginas" @click="carregar(pagina + 1)">
        Próxima
      </button>
    </div>
  </div>
</template>
```

---

## 4. Integração com Backend

### Serviço HTTP com Fetch

```typescript
// src/services/http.ts
const BASE_URL = import.meta.env.VITE_API_URL ?? 'http://localhost:8080/api'

type OpcoesFetch = RequestInit & {
  params?: Record<string, string | number | boolean | undefined>
}

async function request<T>(endpoint: string, opcoes: OpcoesFetch = {}): Promise<T> {
  const { params, ...rest } = opcoes

  let url = `${BASE_URL}${endpoint}`
  if (params) {
    const qs = new URLSearchParams(
      Object.entries(params)
        .filter(([, v]) => v !== undefined)
        .map(([k, v]) => [k, String(v)])
    )
    url += `?${qs}`
  }

  const token = localStorage.getItem('auth_token')
  const headers: HeadersInit = {
    'Content-Type': 'application/json',
    ...(token ? { Authorization: `Bearer ${token}` } : {}),
    ...rest.headers,
  }

  const resposta = await fetch(url, { ...rest, headers })

  if (resposta.status === 401) {
    localStorage.removeItem('auth_token')
    window.location.href = '/login'
    throw new Error('Não autorizado')
  }

  if (!resposta.ok) {
    const erro = await resposta.json().catch(() => ({ mensagem: resposta.statusText }))
    throw new Error(erro.mensagem ?? 'Erro na requisição')
  }

  if (resposta.status === 204) return null as T
  return resposta.json() as Promise<T>
}

export const http = {
  get:    <T>(url: string, o?: OpcoesFetch) => request<T>(url, { ...o, method: 'GET' }),
  post:   <T>(url: string, body: unknown, o?: OpcoesFetch) =>
    request<T>(url, { ...o, method: 'POST', body: JSON.stringify(body) }),
  put:    <T>(url: string, body: unknown, o?: OpcoesFetch) =>
    request<T>(url, { ...o, method: 'PUT', body: JSON.stringify(body) }),
  patch:  <T>(url: string, body: unknown, o?: OpcoesFetch) =>
    request<T>(url, { ...o, method: 'PATCH', body: JSON.stringify(body) }),
  delete: <T>(url: string, o?: OpcoesFetch) => request<T>(url, { ...o, method: 'DELETE' }),
}
```

```typescript
// src/services/produtos.service.ts
import { http } from './http'
import type { Produto, PaginaResposta } from '@/types/produto'

export const produtosService = {
  listar:      (pagina = 1, busca?: string) =>
    http.get<PaginaResposta<Produto>>('/produtos', { params: { pagina, busca } }),

  buscarPorId: (id: number) =>
    http.get<Produto>(`/produtos/${id}`),

  criar:       (dados: Omit<Produto, 'id'>) =>
    http.post<Produto>('/produtos', dados),

  atualizar:   (produto: Produto) =>
    http.put<Produto>(`/produtos/${produto.id}`, produto),

  excluir:     (id: number) =>
    http.delete<void>(`/produtos/${id}`),
}
```

---

### Composable useFetch

```typescript
// src/composables/useAsyncData.ts
import { ref, shallowRef } from 'vue'

type OpcoesFetch<T> = {
  imediato?:      boolean
  valorInicial?:  T
  aoSucesso?:     (dados: T) => void
  aoErro?:        (erro: Error) => void
}

export function useAsyncData<T>(
  fn: () => Promise<T>,
  opcoes: OpcoesFetch<T> = {}
) {
  const { imediato = true, valorInicial, aoSucesso, aoErro } = opcoes

  const dados      = shallowRef<T | undefined>(valorInicial)
  const carregando = ref(false)
  const erro       = ref<string | null>(null)

  async function executar() {
    carregando.value = true
    erro.value       = null
    try {
      dados.value = await fn()
      aoSucesso?.(dados.value)
    } catch (e) {
      const msg    = e instanceof Error ? e.message : 'Erro desconhecido'
      erro.value   = msg
      aoErro?.(e instanceof Error ? e : new Error(msg))
    } finally {
      carregando.value = false
    }
  }

  if (imediato) executar()

  return { dados, carregando, erro, executar }
}

// Uso em componentes
// const { dados: produto, carregando, erro, executar: recarregar } =
//   useAsyncData(() => produtosService.buscarPorId(id.value))
```

---

### TanStack Query para Vue

```typescript
// src/features/produtos/useProdutosQuery.ts
import {
  useQuery,
  useMutation,
  useQueryClient,
  keepPreviousData,
} from '@tanstack/vue-query'
import { computed, type Ref } from 'vue'
import { produtosService } from '@/services/produtos.service'
import type { Produto } from '@/types/produto'

// Chaves de cache centralizadas
export const produtoKeys = {
  all:    () => ['produtos'] as const,
  lists:  () => [...produtoKeys.all(), 'list'] as const,
  list:   (f: object) => [...produtoKeys.lists(), f] as const,
  detail: (id: number) => [...produtoKeys.all(), 'detail', id] as const,
}

// Query reativa — rebusca quando pagina ou busca mudam
export function useProdutosQuery(pagina: Ref<number>, busca: Ref<string>) {
  return useQuery({
    queryKey: computed(() => produtoKeys.list({ pagina: pagina.value, busca: busca.value })),
    queryFn:  () => produtosService.listar(pagina.value, busca.value || undefined),
    placeholderData: keepPreviousData,
  })
}

export function useProdutoQuery(id: Ref<number>) {
  return useQuery({
    queryKey: computed(() => produtoKeys.detail(id.value)),
    queryFn:  () => produtosService.buscarPorId(id.value),
    enabled:  computed(() => !!id.value),
  })
}

export function useCriarProduto() {
  const client = useQueryClient()
  return useMutation({
    mutationFn: (dados: Omit<Produto, 'id'>) => produtosService.criar(dados),
    onSuccess:  () => client.invalidateQueries({ queryKey: produtoKeys.lists() }),
  })
}

export function useAtualizarProduto() {
  const client = useQueryClient()
  return useMutation({
    mutationFn: (produto: Produto) => produtosService.atualizar(produto),
    onSuccess:  (_, produto) => {
      client.invalidateQueries({ queryKey: produtoKeys.detail(produto.id) })
      client.invalidateQueries({ queryKey: produtoKeys.lists() })
    },
  })
}

export function useExcluirProduto() {
  const client = useQueryClient()
  return useMutation({
    mutationFn: (id: number) => produtosService.excluir(id),
    onSuccess:  () => client.invalidateQueries({ queryKey: produtoKeys.lists() }),
  })
}
```

```vue
<!-- Usando TanStack Query num componente -->
<script setup lang="ts">
import { ref } from 'vue'
import { useProdutosQuery, useCriarProduto } from './useProdutosQuery'
import CardProduto from '@/components/common/CardProduto.vue'

const pagina = ref(1)
const busca  = ref('')

const { data, isLoading, isError, error, isFetching } = useProdutosQuery(pagina, busca)
const criarProduto = useCriarProduto()
</script>

<template>
  <div>
    <input v-model="busca" class="input" placeholder="Buscar..." />

    <span v-if="isFetching" class="indicador-atualizando">Atualizando...</span>

    <div v-if="isLoading" class="carregando-pagina"><div class="spinner"></div></div>
    <div v-else-if="isError" class="alerta alerta--erro">{{ error?.message }}</div>
    <ul v-else class="lista-produtos">
      <li v-for="produto in data?.itens" :key="produto.id">
        <CardProduto :produto="produto" />
      </li>
    </ul>

    <div class="paginacao">
      <button class="btn btn--secundario" :disabled="pagina === 1" @click="pagina--">Anterior</button>
      <span>Página {{ pagina }}</span>
      <button class="btn btn--secundario" @click="pagina++">Próxima</button>
    </div>
  </div>
</template>
```

---

## 5. Formulários e Validação

### VeeValidate com Zod

```typescript
// src/features/produtos/schemas.ts
import { z } from 'zod'
import { toTypedSchema } from '@vee-validate/zod'

export const schemaProduto = z.object({
  nome: z
    .string()
    .min(3, 'Nome deve ter pelo menos 3 caracteres')
    .max(100, 'Máximo 100 caracteres'),
  preco: z
    .number({ invalid_type_error: 'Informe um preço válido' })
    .positive('Preço deve ser positivo')
    .multipleOf(0.01, 'Máximo 2 casas decimais'),
  categoria: z.enum(['eletronicos', 'livros', 'roupas', 'alimentos'], {
    errorMap: () => ({ message: 'Selecione uma categoria válida' }),
  }),
  descricao: z.string().max(500, 'Máximo 500 caracteres').optional(),
  ativo: z.boolean().default(true),
})

export type ProdutoForm = z.infer<typeof schemaProduto>

// Adapta o schema Zod para o VeeValidate
export const validacaoProduto = toTypedSchema(schemaProduto)

// Schema de login
export const schemaLogin = z.object({
  email: z.string().email('E-mail inválido'),
  senha: z.string().min(6, 'Senha deve ter pelo menos 6 caracteres'),
})
export const validacaoLogin = toTypedSchema(schemaLogin)
export type LoginForm = z.infer<typeof schemaLogin>

// Schema com validação cruzada
export const schemaRedefinirSenha = z
  .object({
    senhaAtual:     z.string().min(6),
    novaSenha:      z.string().min(8, 'Mínimo 8 caracteres'),
    confirmarSenha: z.string(),
  })
  .refine((d) => d.novaSenha === d.confirmarSenha, {
    message: 'As senhas não coincidem',
    path: ['confirmarSenha'],
  })
export const validacaoRedefinirSenha = toTypedSchema(schemaRedefinirSenha)
```

```vue
<!-- src/features/produtos/FormularioProduto.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useForm } from 'vee-validate'
import { useI18n } from 'vue-i18n'
import { useProdutosStore } from '@/stores/produtos'
import { validacaoProduto, type ProdutoForm } from './schemas'
import type { Produto } from '@/types/produto'

const props = defineProps<{
  produto?: Produto | null
}>()

const emit = defineEmits<{
  salvo:    [produto: Produto]
  cancelar: []
}>()

const { t }          = useI18n()
const produtosStore  = useProdutosStore()
const editando       = computed(() => !!props.produto)

// useForm com schema Zod
const { handleSubmit, resetForm, isSubmitting, meta, defineField, errors } = useForm<ProdutoForm>({
  validationSchema: validacaoProduto,
  initialValues: {
    nome:      props.produto?.nome      ?? '',
    preco:     props.produto?.preco     ?? 0,
    categoria: props.produto?.categoria ?? 'eletronicos',
    descricao: props.produto?.descricao ?? '',
    ativo:     props.produto?.ativo     ?? true,
  },
})

// defineField vincula o campo ao formulário mantendo reatividade
const [nome,      nomeAttr]      = defineField('nome')
const [preco,     precoAttr]     = defineField('preco')
const [categoria, categoriaAttr] = defineField('categoria')
const [descricao, descricaoAttr] = defineField('descricao')
const [ativo,     ativoAttr]     = defineField('ativo')

const onSubmit = handleSubmit(async (valores) => {
  try {
    if (editando.value && props.produto) {
      const atualizado = await produtosStore.atualizar({ ...valores, id: props.produto.id })
      emit('salvo', atualizado)
    } else {
      const novo = await produtosStore.criar(valores)
      emit('salvo', novo)
      resetForm()
    }
  } catch {
    // erro já tratado na store
  }
})
</script>

<template>
  <form class="formulario" novalidate @submit.prevent="onSubmit">
    <h2>{{ editando ? t('produto.acoes.editar') : t('produto.acoes.criar') }}</h2>

    <!-- Nome -->
    <div class="campo">
      <label for="nome">{{ t('produto.nome') }} *</label>
      <input
        id="nome"
        v-model="nome"
        v-bind="nomeAttr"
        class="input"
        :class="{ 'input--erro': errors.nome }"
        :aria-invalid="!!errors.nome"
        aria-describedby="nome-erro"
      />
      <span v-if="errors.nome" id="nome-erro" class="campo__erro" role="alert">
        {{ errors.nome }}
      </span>
    </div>

    <!-- Preço -->
    <div class="campo">
      <label for="preco">{{ t('produto.preco') }} *</label>
      <input
        id="preco"
        v-model.number="preco"
        v-bind="precoAttr"
        type="number"
        step="0.01"
        class="input"
        :class="{ 'input--erro': errors.preco }"
      />
      <span v-if="errors.preco" class="campo__erro" role="alert">{{ errors.preco }}</span>
    </div>

    <!-- Categoria -->
    <div class="campo">
      <label for="categoria">{{ t('produto.categoria') }} *</label>
      <select
        id="categoria"
        v-model="categoria"
        v-bind="categoriaAttr"
        class="select"
        :class="{ 'select--erro': errors.categoria }"
      >
        <option value="">Selecione...</option>
        <option value="eletronicos">{{ t('produto.categorias.eletronicos') }}</option>
        <option value="livros">{{ t('produto.categorias.livros') }}</option>
        <option value="roupas">{{ t('produto.categorias.roupas') }}</option>
        <option value="alimentos">{{ t('produto.categorias.alimentos') }}</option>
      </select>
      <span v-if="errors.categoria" class="campo__erro" role="alert">{{ errors.categoria }}</span>
    </div>

    <!-- Descrição -->
    <div class="campo">
      <label for="descricao">Descrição</label>
      <textarea
        id="descricao"
        v-model="descricao"
        v-bind="descricaoAttr"
        class="textarea"
        rows="3"
      ></textarea>
      <small class="campo__dica">{{ descricao?.length ?? 0 }}/500</small>
      <span v-if="errors.descricao" class="campo__erro" role="alert">{{ errors.descricao }}</span>
    </div>

    <!-- Ativo -->
    <div class="campo campo--checkbox">
      <input id="ativo" v-model="ativo" v-bind="ativoAttr" type="checkbox" />
      <label for="ativo">Produto ativo</label>
    </div>

    <div class="formulario__acoes">
      <button
        type="submit"
        class="btn btn--primario"
        :disabled="isSubmitting || !meta.dirty"
      >
        {{ isSubmitting ? 'Salvando...' : t('produto.acoes.salvar') }}
      </button>
      <button type="button" class="btn btn--secundario" @click="emit('cancelar')">
        {{ t('produto.acoes.cancelar') }}
      </button>
    </div>
  </form>
</template>
```

---

### Formulários com defineModel

```vue
<!-- Componente de campo reutilizável com v-model + VeeValidate -->
<!-- src/components/common/CampoValidado.vue -->
<script setup lang="ts">
import { useField } from 'vee-validate'

const props = defineProps<{
  name:         string
  label:        string
  tipo?:        string
  placeholder?: string
  obrigatorio?: boolean
}>()

const { value, errorMessage, meta } = useField<string>(() => props.name)
const invalido = computed(() => !!errorMessage.value && meta.touched)
const id = `campo-${props.name}`
</script>

<template>
  <div class="campo">
    <label :for="id">
      {{ label }}
      <span v-if="obrigatorio" class="campo__obrigatorio" aria-hidden="true">*</span>
    </label>
    <input
      :id="id"
      v-model="value"
      :type="tipo ?? 'text'"
      :placeholder="placeholder"
      class="input"
      :class="{ 'input--erro': invalido }"
      :aria-describedby="invalido ? `${id}-erro` : undefined"
      :aria-invalid="invalido"
    />
    <span v-if="invalido" :id="`${id}-erro`" class="campo__erro" role="alert">
      {{ errorMessage }}
    </span>
  </div>
</template>
```

---

## 6. Internacionalização com Vue I18n

```typescript
// src/i18n/index.ts
import { createI18n } from 'vue-i18n'
import ptBR from './locales/pt-BR.json'
import enUS from './locales/en-US.json'

export type Locale = 'pt-BR' | 'en-US'

export const i18n = createI18n({
  legacy: false,       // modo Composition API
  locale: 'pt-BR',
  fallbackLocale: 'pt-BR',
  globalInjection: true,  // disponibiliza $t, $n, $d globalmente
  messages: {
    'pt-BR': ptBR,
    'en-US': enUS,
  },
  numberFormats: {
    'pt-BR': {
      currency: { style: 'currency', currency: 'BRL', minimumFractionDigits: 2 },
      decimal:  { style: 'decimal',  minimumFractionDigits: 2, maximumFractionDigits: 2 },
      percent:  { style: 'percent',  minimumFractionDigits: 1 },
    },
    'en-US': {
      currency: { style: 'currency', currency: 'USD', minimumFractionDigits: 2 },
      decimal:  { style: 'decimal',  minimumFractionDigits: 2, maximumFractionDigits: 2 },
      percent:  { style: 'percent',  minimumFractionDigits: 1 },
    },
  },
  datetimeFormats: {
    'pt-BR': {
      curto:   { day: '2-digit', month: '2-digit', year: 'numeric' },
      longo:   { day: '2-digit', month: 'long', year: 'numeric' },
      completo:{ day: '2-digit', month: '2-digit', year: 'numeric', hour: '2-digit', minute: '2-digit' },
    },
    'en-US': {
      curto:   { month: '2-digit', day: '2-digit', year: 'numeric' },
      longo:   { month: 'long',    day: '2-digit', year: 'numeric' },
      completo:{ month: '2-digit', day: '2-digit', year: 'numeric', hour: '2-digit', minute: '2-digit' },
    },
  },
})
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
    "lista": "Lista de Produtos",
    "nome": "Nome",
    "preco": "Preço",
    "categoria": "Categoria",
    "ativo": "Ativo",
    "inativo": "Inativo",
    "semResultados": "Nenhum produto encontrado.",
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
      "confirmarExclusao": "Deseja excluir o produto \"{nome}\"?"
    }
  },
  "paginacao": {
    "anterior": "Anterior",
    "proxima": "Próxima",
    "pagina": "Página {pagina} de {total}"
  },
  "erros": {
    "generico": "Ocorreu um erro. Tente novamente.",
    "naoAutorizado": "Você não tem permissão para acessar este recurso.",
    "naoEncontrado": "Página não encontrada.",
    "sessaoExpirada": "Sua sessão expirou. Faça login novamente."
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
    "lista": "Product List",
    "nome": "Name",
    "preco": "Price",
    "categoria": "Category",
    "ativo": "Active",
    "inativo": "Inactive",
    "semResultados": "No products found.",
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
      "confirmarExclusao": "Do you want to delete the product \"{nome}\"?"
    }
  },
  "paginacao": {
    "anterior": "Previous",
    "proxima": "Next",
    "pagina": "Page {pagina} of {total}"
  },
  "erros": {
    "generico": "An error occurred. Please try again.",
    "naoAutorizado": "You are not authorized to access this resource.",
    "naoEncontrado": "Page not found.",
    "sessaoExpirada": "Your session has expired. Please sign in again."
  }
}
```

```vue
<!-- Usando Vue I18n nos componentes -->
<!-- src/components/common/SeletorIdioma.vue -->
<script setup lang="ts">
import { useI18n } from 'vue-i18n'

const { locale } = useI18n()

const idiomas = [
  { codigo: 'pt-BR', rotulo: 'PT' },
  { codigo: 'en-US', rotulo: 'EN' },
]

function mudar(codigo: string) {
  locale.value = codigo
  localStorage.setItem('idioma', codigo)
}
</script>

<template>
  <div class="seletor-idioma" role="navigation" aria-label="Seleção de idioma">
    <button
      v-for="idioma in idiomas"
      :key="idioma.codigo"
      class="btn btn--idioma"
      :class="{ 'btn--idioma-ativo': locale === idioma.codigo }"
      :lang="idioma.codigo"
      @click="mudar(idioma.codigo)"
    >
      {{ idioma.rotulo }}
    </button>
  </div>
</template>
```

```vue
<!-- Usando $t, $n e $d no template -->
<script setup lang="ts">
import { useI18n } from 'vue-i18n'
const { t, n, d } = useI18n()
</script>

<template>
  <!-- Tradução simples -->
  <h1>{{ t('produto.lista') }}</h1>

  <!-- Interpolação de variáveis -->
  <p>{{ t('paginacao.pagina', { pagina: 2, total: 10 }) }}</p>
  <p>{{ t('produto.mensagens.confirmarExclusao', { nome: 'Mouse Gamer' }) }}</p>

  <!-- Formatação de número (usa numberFormats do i18n) -->
  <p>{{ n(1299.90, 'currency') }}</p>

  <!-- Formatação de data (usa datetimeFormats do i18n) -->
  <time>{{ d(new Date(), 'completo') }}</time>
</template>
```

---

## 7. Manipulação de Datas com date-fns

```typescript
// src/utils/datas.ts
import {
  format,
  formatDistanceToNow,
  parseISO,
  isValid,
  isBefore,
  isAfter,
  addDays,
  addMonths,
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
  eachDayOfInterval,
} from 'date-fns'
import { ptBR, enUS } from 'date-fns/locale'

const locales: Record<string, Locale> = { 'pt-BR': ptBR, 'en-US': enUS }

export function getLocale(idioma: string): Locale {
  return locales[idioma] ?? ptBR
}

export function formatarData(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data
  if (!isValid(d)) return '—'
  return format(d, idioma === 'en-US' ? 'MM/dd/yyyy' : 'dd/MM/yyyy', {
    locale: getLocale(idioma),
  })
}

export function formatarDataHora(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data
  if (!isValid(d)) return '—'
  return format(d, idioma === 'en-US' ? 'MM/dd/yyyy hh:mm a' : 'dd/MM/yyyy HH:mm', {
    locale: getLocale(idioma),
  })
}

export function formatarRelativo(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data
  if (!isValid(d)) return '—'
  return formatDistanceToNow(d, { addSuffix: true, locale: getLocale(idioma) })
}

export const datasUtil = {
  prazoVencido:  (data: Date | string) =>
    isBefore(typeof data === 'string' ? parseISO(data) : data, startOfDay(new Date())),
  diasRestantes: (data: Date | string) =>
    differenceInDays(typeof data === 'string' ? parseISO(data) : data, new Date()),
  diasDoMes: (ano: number, mes: number) =>
    eachDayOfInterval({ start: startOfMonth(new Date(ano, mes)), end: endOfMonth(new Date(ano, mes)) }),
}
```

```typescript
// src/composables/useDatas.ts — composable que reage ao idioma
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import {
  formatarData,
  formatarDataHora,
  formatarRelativo,
  datasUtil,
} from '@/utils/datas'
import { parseISO, isToday, isTomorrow, isYesterday } from 'date-fns'

export function useDatas() {
  const { locale } = useI18n()

  const formatar      = (d: Date | string) => formatarData(d, locale.value)
  const formatarHora  = (d: Date | string) => formatarDataHora(d, locale.value)
  const relativo      = (d: Date | string) => formatarRelativo(d, locale.value)

  function prazoLabel(data: Date | string): string {
    const d = typeof data === 'string' ? parseISO(data) : data
    if (isToday(d))     return locale.value === 'en-US' ? 'Today'    : 'Hoje'
    if (isTomorrow(d))  return locale.value === 'en-US' ? 'Tomorrow' : 'Amanhã'
    if (isYesterday(d)) return locale.value === 'en-US' ? 'Yesterday (overdue)' : 'Ontem (vencido)'
    return formatar(d)
  }

  return { formatar, formatarHora, relativo, prazoLabel, datasUtil }
}
```

```vue
<!-- Componente de data reutilizável -->
<!-- src/components/common/DataFormatada.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useDatas } from '@/composables/useDatas'
import { datasUtil } from '@/utils/datas'

const props = defineProps<{
  valor:    string | Date | null | undefined
  formato?: 'data' | 'dataHora' | 'relativo' | 'prazo'
}>()

const { formatar, formatarHora, relativo, prazoLabel } = useDatas()

const texto = computed(() => {
  if (!props.valor) return '—'
  switch (props.formato) {
    case 'dataHora': return formatarHora(props.valor)
    case 'relativo': return relativo(props.valor)
    case 'prazo':    return prazoLabel(props.valor)
    default:         return formatar(props.valor)
  }
})

const vencido = computed(() =>
  props.formato === 'prazo' && props.valor
    ? datasUtil.prazoVencido(props.valor)
    : false
)

const iso = computed(() =>
  props.valor instanceof Date
    ? props.valor.toISOString()
    : props.valor ?? undefined
)
</script>

<template>
  <time
    :datetime="iso"
    :title="iso ? formatarHora(iso) : undefined"
    :class="{ 'prazo--vencido': vencido }"
  >
    {{ texto }}
  </time>
</template>
```

---

## 8. Autenticação JWT e Controle de Acesso por Role

### Estrutura do Token JWT

O backend emite um JWT com o payload:

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
export type Role = 'USER' | 'GERENTE' | 'ADMIN'

export type UsuarioJWT = {
  sub:   string
  nome:  string
  roles: Role[]
  iat:   number
  exp:   number
}

export type Usuario = {
  email: string
  nome:  string
  roles: Role[]
}

export type RespostaLogin = {
  token:         string
  refreshToken?: string
}
```

---

### Store de Autenticação com Pinia

```typescript
// src/stores/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { jwtDecode } from 'jwt-decode'
import { useRouter } from 'vue-router'
import { http } from '@/services/http'
import type { Usuario, UsuarioJWT, RespostaLogin, Role } from '@/types/auth'

const TOKEN_KEY   = 'auth_token'
const REFRESH_KEY = 'refresh_token'

export const useAuthStore = defineStore('auth', () => {
  const router = useRouter()

  const _usuario = ref<Usuario | null>(null)
  const _token   = ref<string | null>(null)

  // Getters
  const usuario     = computed(() => _usuario.value)
  const token       = computed(() => _token.value)
  const autenticado = computed(() => !!_usuario.value)

  function tokenValido(t: string): boolean {
    try {
      const { exp } = jwtDecode<UsuarioJWT>(t)
      return Date.now() < exp * 1000
    } catch { return false }
  }

  function extrairUsuario(t: string): Usuario | null {
    try {
      const { sub, nome, roles } = jwtDecode<UsuarioJWT>(t)
      return { email: sub, nome, roles }
    } catch { return null }
  }

  function restaurarSessao(): void {
    const t = localStorage.getItem(TOKEN_KEY)
    if (t && tokenValido(t)) {
      _token.value   = t
      _usuario.value = extrairUsuario(t)
    } else {
      limparStorage()
    }
  }

  function limparStorage(): void {
    localStorage.removeItem(TOKEN_KEY)
    localStorage.removeItem(REFRESH_KEY)
  }

  async function entrar(email: string, senha: string): Promise<void> {
    const resposta = await http.post<RespostaLogin>('/auth/login', { email, senha })
    localStorage.setItem(TOKEN_KEY, resposta.token)
    if (resposta.refreshToken) {
      localStorage.setItem(REFRESH_KEY, resposta.refreshToken)
    }
    _token.value   = resposta.token
    _usuario.value = extrairUsuario(resposta.token)
  }

  function sair(): void {
    limparStorage()
    _token.value   = null
    _usuario.value = null
    router.push('/login')
  }

  function temRole(role: Role | string): boolean {
    return _usuario.value?.roles.includes(role as Role) ?? false
  }

  function temAlgumaRole(roles: string[]): boolean {
    return roles.some((r) => temRole(r))
  }

  function temTodasAsRoles(roles: string[]): boolean {
    return roles.every((r) => temRole(r))
  }

  function getToken(): string | null {
    return localStorage.getItem(TOKEN_KEY)
  }

  async function renovarToken(): Promise<string> {
    const refreshToken = localStorage.getItem(REFRESH_KEY)
    if (!refreshToken) { sair(); throw new Error('Sem refresh token') }

    const resposta = await http.post<RespostaLogin>('/auth/refresh', { refreshToken })
    localStorage.setItem(TOKEN_KEY, resposta.token)
    _token.value   = resposta.token
    _usuario.value = extrairUsuario(resposta.token)
    return resposta.token
  }

  return {
    usuario, token, autenticado,
    restaurarSessao, entrar, sair,
    temRole, temAlgumaRole, temTodasAsRoles,
    getToken, renovarToken,
  }
})
```

---

### Interceptor de Requisições

```typescript
// src/services/http.ts — versão com refresh token automático
import { useAuthStore } from '@/stores/auth'

let renovando: Promise<string> | null = null

async function requestComRefresh<T>(endpoint: string, opcoes: OpcoesFetch = {}): Promise<T> {
  const auth    = useAuthStore()
  const token   = auth.getToken()
  const headers: HeadersInit = {
    'Content-Type': 'application/json',
    ...(token ? { Authorization: `Bearer ${token}` } : {}),
  }

  const url      = montarUrl(endpoint, opcoes.params)
  const { params, ...rest } = opcoes
  const resposta = await fetch(url, { ...rest, headers })

  if (resposta.status === 401) {
    // Deduplicar chamadas de refresh simultâneas
    if (!renovando) {
      renovando = auth.renovarToken().finally(() => { renovando = null })
    }

    try {
      const novoToken     = await renovando!
      const novasHeaders: HeadersInit = {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${novoToken}`,
      }
      const retentativa   = await fetch(url, { ...rest, headers: novasHeaders })
      if (!retentativa.ok) throw new Error('Falha após refresh')
      if (retentativa.status === 204) return null as T
      return retentativa.json() as Promise<T>
    } catch {
      auth.sair()
      throw new Error('Sessão expirada')
    }
  }

  if (!resposta.ok) {
    const erro = await resposta.json().catch(() => ({ mensagem: resposta.statusText }))
    throw new Error(erro.mensagem ?? 'Erro na requisição')
  }

  if (resposta.status === 204) return null as T
  return resposta.json() as Promise<T>
}

function montarUrl(endpoint: string, params?: Record<string, string | number | boolean | undefined>): string {
  const BASE_URL = import.meta.env.VITE_API_URL ?? 'http://localhost:8080/api'
  let url = `${BASE_URL}${endpoint}`
  if (params) {
    const qs = new URLSearchParams(
      Object.entries(params)
        .filter(([, v]) => v !== undefined)
        .map(([k, v]) => [k, String(v)])
    )
    url += `?${qs}`
  }
  return url
}
```

---

### Guards de Rota por Role

```typescript
// Já integrado no router/index.ts — recapitulando o guard completo:
// src/router/guards.ts
import type { NavigationGuard } from 'vue-router'
import { useAuthStore } from '@/stores/auth'

export const authGuard: NavigationGuard = (to) => {
  const auth = useAuthStore()

  // Verifica herança: qualquer rota pai ou a própria rota
  const requerAuth  = to.matched.some((r) => r.meta.requerAutenticacao)
  const rolesNecessarias = to.meta.roles

  if (requerAuth && !auth.autenticado) {
    return { name: 'login', query: { returnUrl: to.fullPath } }
  }

  if (rolesNecessarias?.length && !auth.temAlgumaRole(rolesNecessarias)) {
    return { name: 'acesso-negado' }
  }

  return true
}

// Registrando no router
router.beforeEach(authGuard)
```

---

### Diretiva e Componente de Acesso por Role

```typescript
// src/directives/acesso-restrito.ts
import type { Directive } from 'vue'
import { useAuthStore } from '@/stores/auth'

/**
 * Remove o elemento do DOM se o usuário não tiver a(s) role(s) exigida(s).
 *
 * Uso:
 *   <button v-acesso-restrito="'ADMIN'">Excluir</button>
 *   <button v-acesso-restrito="['ADMIN', 'GERENTE']">Editar</button>
 */
export const vAcessoRestrito: Directive<HTMLElement, string | string[]> = {
  mounted(el, binding) {
    const auth  = useAuthStore()
    const roles = Array.isArray(binding.value) ? binding.value : [binding.value]

    if (!auth.temAlgumaRole(roles)) {
      el.parentNode?.removeChild(el)
    }
  },
}
```

```vue
<!-- src/components/common/AcessoRestrito.vue -->
<!-- Componente alternativo baseado em slots -->
<script setup lang="ts">
import { computed } from 'vue'
import { useAuthStore } from '@/stores/auth'

const props = defineProps<{
  roles?:        string[]   // Qualquer uma dessas roles
  todasAsRoles?: string[]   // Todas essas roles
}>()

const auth = useAuthStore()

const temPermissao = computed(() => {
  const { roles = [], todasAsRoles = [] } = props

  if (!auth.autenticado) return false
  if (roles.length > 0 && !auth.temAlgumaRole(roles)) return false
  if (todasAsRoles.length > 0 && !auth.temTodasAsRoles(todasAsRoles)) return false

  return true
})
</script>

<template>
  <slot v-if="temPermissao" />
  <slot v-else name="sem-acesso" />
</template>
```

```vue
<!-- Usando nos componentes -->
<script setup lang="ts">
import { useAuthStore } from '@/stores/auth'
import AcessoRestrito from '@/components/common/AcessoRestrito.vue'

const auth = useAuthStore()
</script>

<template>
  <!-- Diretiva estrutural (simples, sem fallback) -->
  <button v-acesso-restrito="'ADMIN'" class="btn btn--perigo btn--pequeno">
    Excluir
  </button>

  <button v-acesso-restrito="['ADMIN', 'GERENTE']" class="btn btn--secundario btn--pequeno">
    Editar
  </button>

  <!-- Componente com slot de fallback -->
  <AcessoRestrito :roles="['ADMIN']">
    <section class="painel-admin">
      <h2>Configurações Administrativas</h2>
    </section>
    <template #sem-acesso>
      <div class="alerta alerta--info">
        Você não tem permissão para acessar esta área.
      </div>
    </template>
  </AcessoRestrito>

  <!-- Verificação direta via store (em lógica inline) -->
  <table>
    <thead>
      <tr>
        <th>Nome</th>
        <th>Preço</th>
        <th v-if="auth.temAlgumaRole(['ADMIN', 'GERENTE'])">Ações</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="produto in produtos" :key="produto.id">
        <td>{{ produto.nome }}</td>
        <td>{{ produto.preco }}</td>
        <td v-if="auth.temAlgumaRole(['ADMIN', 'GERENTE'])">
          <button class="btn btn--secundario btn--pequeno">Editar</button>
          <button
            v-if="auth.temRole('ADMIN')"
            class="btn btn--perigo btn--pequeno"
          >
            Excluir
          </button>
        </td>
      </tr>
    </tbody>
  </table>
</template>
```

```vue
<!-- Página de login completa -->
<!-- src/features/auth/LoginView.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useForm } from 'vee-validate'
import { useRouter, useRoute } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { useAuthStore } from '@/stores/auth'
import { validacaoLogin, type LoginForm } from '@/features/auth/schemas'

const { t }   = useI18n()
const auth    = useAuthStore()
const router  = useRouter()
const route   = useRoute()

const erroLogin = ref('')

const { handleSubmit, isSubmitting, defineField, errors } = useForm<LoginForm>({
  validationSchema: validacaoLogin,
})

const [email, emailAttr] = defineField('email')
const [senha, senhaAttr] = defineField('senha')

const onSubmit = handleSubmit(async (valores) => {
  erroLogin.value = ''
  try {
    await auth.entrar(valores.email, valores.senha)
    const returnUrl = (route.query.returnUrl as string | undefined) ?? '/'
    router.push(returnUrl)
  } catch {
    erroLogin.value = 'E-mail ou senha inválidos.'
  }
})
</script>

<template>
  <div class="pagina-login">
    <div class="card-login">
      <h1>{{ t('nav.entrar') }}</h1>

      <div v-if="erroLogin" class="alerta alerta--erro" role="alert">
        {{ erroLogin }}
      </div>

      <form class="formulario" novalidate @submit.prevent="onSubmit">
        <div class="campo">
          <label for="email">E-mail *</label>
          <input
            id="email"
            v-model="email"
            v-bind="emailAttr"
            type="email"
            class="input"
            :class="{ 'input--erro': errors.email }"
            autocomplete="email"
          />
          <span v-if="errors.email" class="campo__erro" role="alert">{{ errors.email }}</span>
        </div>

        <div class="campo">
          <label for="senha">Senha *</label>
          <input
            id="senha"
            v-model="senha"
            v-bind="senhaAttr"
            type="password"
            class="input"
            :class="{ 'input--erro': errors.senha }"
            autocomplete="current-password"
          />
          <span v-if="errors.senha" class="campo__erro" role="alert">{{ errors.senha }}</span>
        </div>

        <button
          type="submit"
          class="btn btn--primario btn--bloco"
          :disabled="isSubmitting"
        >
          {{ isSubmitting ? 'Entrando...' : t('nav.entrar') }}
        </button>
      </form>
    </div>
  </div>
</template>
```

---

## CSS — Estrutura sem Biblioteca de Estilos

```css
/* src/assets/styles.css */

*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --cor-primaria:       #1d4ed8;
  --cor-primaria-hover: #1e40af;
  --cor-secundaria:     #6b7280;
  --cor-perigo:         #dc2626;
  --cor-perigo-hover:   #b91c1c;
  --cor-sucesso:        #16a34a;
  --cor-aviso:          #d97706;
  --cor-info:           #0284c7;

  --cor-texto:          #111827;
  --cor-texto-fraco:    #6b7280;
  --cor-fundo:          #f9fafb;
  --cor-fundo-card:     #ffffff;
  --cor-borda:          #e5e7eb;

  --raio-borda:         6px;
  --sombra-card:        0 1px 3px rgba(0,0,0,.10), 0 1px 2px rgba(0,0,0,.06);
  --fonte-base:         'Inter', system-ui, -apple-system, sans-serif;
}

body {
  font-family: var(--fonte-base);
  font-size: 1rem;
  color: var(--cor-texto);
  background-color: var(--cor-fundo);
  line-height: 1.6;
}

/* Layout */
.app                { display: flex; flex-direction: column; min-height: 100vh; }
.conteudo-principal { flex: 1; max-width: 1200px; width: 100%; margin: 0 auto; padding: 2rem 1rem; }

/* Cabeçalho */
.cabecalho {
  background: var(--cor-fundo-card);
  border-bottom: 1px solid var(--cor-borda);
  padding: 0 1rem;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: var(--sombra-card);
}

.nav-principal     { display: flex; gap: .5rem; align-items: center; }

.nav-link {
  padding: .5rem .75rem;
  border-radius: var(--raio-borda);
  text-decoration: none;
  color: var(--cor-texto-fraco);
  font-weight: 500;
  font-size: .9375rem;
  transition: color .15s, background-color .15s;
}
.nav-link:hover    { color: var(--cor-primaria); background: #eff6ff; }
.nav-link--ativo   { color: var(--cor-primaria); background: #eff6ff; }

.cabecalho__acoes  { display: flex; align-items: center; gap: .75rem; }
.usuario-nome      { font-size: .875rem; color: var(--cor-texto-fraco); }

/* Botões */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: .5rem;
  padding: .5rem 1rem;
  border: none;
  border-radius: var(--raio-borda);
  font-size: .875rem;
  font-weight: 600;
  cursor: pointer;
  text-decoration: none;
  transition: background-color .15s, opacity .15s;
  white-space: nowrap;
}
.btn:disabled                          { opacity: .5; cursor: not-allowed; }
.btn--bloco                            { width: 100%; }
.btn--pequeno                          { padding: .25rem .625rem; font-size: .75rem; }
.btn--primario                         { background: var(--cor-primaria); color: #fff; }
.btn--primario:hover:not(:disabled)    { background: var(--cor-primaria-hover); }
.btn--secundario                       { background: transparent; color: var(--cor-secundaria); border: 1px solid var(--cor-borda); }
.btn--secundario:hover:not(:disabled)  { background: var(--cor-fundo); }
.btn--perigo                           { background: var(--cor-perigo); color: #fff; }
.btn--perigo:hover:not(:disabled)      { background: var(--cor-perigo-hover); }
.btn--idioma                           { background: transparent; color: var(--cor-texto-fraco); border: 1px solid var(--cor-borda); padding: .25rem .5rem; font-size: .75rem; }
.btn--idioma-ativo                     { background: var(--cor-primaria); color: #fff; border-color: var(--cor-primaria); }

/* Cards */
.card-produto {
  background: var(--cor-fundo-card);
  border: 1px solid var(--cor-borda);
  border-radius: var(--raio-borda);
  padding: 1.25rem;
  box-shadow: var(--sombra-card);
  display: flex;
  flex-direction: column;
  gap: .5rem;
}
.card-produto--destacado { border-color: var(--cor-primaria); box-shadow: 0 0 0 2px rgba(29,78,216,.2); }
.card-produto__titulo    { font-size: 1.125rem; font-weight: 600; }
.card-produto__preco     { font-size: 1.25rem; font-weight: 700; color: var(--cor-primaria); }
.card-produto__acoes     { display: flex; gap: .5rem; margin-top: .5rem; }

/* Lista de produtos */
.lista-produtos { list-style: none; display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 1rem; }

/* Formulários */
.formulario            { display: flex; flex-direction: column; gap: 1.25rem; max-width: 480px; }
.campo                 { display: flex; flex-direction: column; gap: .375rem; }
.campo label           { font-size: .875rem; font-weight: 500; }
.campo__erro           { font-size: .75rem; color: var(--cor-perigo); }
.campo__dica           { font-size: .75rem; color: var(--cor-texto-fraco); text-align: right; }
.campo__obrigatorio    { color: var(--cor-perigo); margin-left: .25rem; }
.campo--checkbox       { flex-direction: row; align-items: center; gap: .5rem; }
.campo--checkbox input { width: auto; }

.input, .select, .textarea {
  padding: .5rem .75rem;
  border: 1px solid var(--cor-borda);
  border-radius: var(--raio-borda);
  font-size: 1rem;
  color: var(--cor-texto);
  background: var(--cor-fundo-card);
  width: 100%;
  transition: border-color .15s, box-shadow .15s;
}
.input:focus, .select:focus, .textarea:focus {
  outline: none;
  border-color: var(--cor-primaria);
  box-shadow: 0 0 0 3px rgba(29,78,216,.15);
}
.input--erro, .select--erro  { border-color: var(--cor-perigo); }
.input--erro:focus, .select--erro:focus { box-shadow: 0 0 0 3px rgba(220,38,38,.15); }

.formulario__acoes     { display: flex; gap: .75rem; }

/* Login */
.pagina-login { min-height: calc(100vh - 64px); display: flex; align-items: center; justify-content: center; }
.card-login   { background: var(--cor-fundo-card); border: 1px solid var(--cor-borda); border-radius: var(--raio-borda); padding: 2rem; box-shadow: var(--sombra-card); width: 100%; max-width: 400px; }
.card-login h1{ margin-bottom: 1.5rem; font-size: 1.5rem; }

/* Alertas */
.alerta          { padding: .875rem 1rem; border-radius: var(--raio-borda); font-size: .875rem; }
.alerta--erro    { background: #fee2e2; color: #991b1b; border: 1px solid #fca5a5; }
.alerta--sucesso { background: #dcfce7; color: #166534; border: 1px solid #86efac; }
.alerta--info    { background: #e0f2fe; color: #075985; border: 1px solid #7dd3fc; }
.alerta--aviso   { background: #fef3c7; color: #92400e; border: 1px solid #fcd34d; }

/* Badges */
.badge           { display: inline-block; padding: .125rem .5rem; border-radius: 9999px; font-size: .75rem; font-weight: 600; }
.badge--sucesso  { background: #dcfce7; color: #166534; }
.badge--erro     { background: #fee2e2; color: #991b1b; }
.badge--info     { background: #e0f2fe; color: #075985; }

/* Painel */
.painel                    { background: var(--cor-fundo-card); border: 1px solid var(--cor-borda); border-radius: var(--raio-borda); box-shadow: var(--sombra-card); overflow: hidden; }
.painel__cabecalho         { display: flex; align-items: center; justify-content: space-between; padding: 1rem 1.25rem; border-bottom: 1px solid var(--cor-borda); background: var(--cor-fundo); }
.painel__corpo             { padding: 1.25rem; }
.painel__rodape            { padding: .875rem 1.25rem; border-top: 1px solid var(--cor-borda); font-size: .875rem; color: var(--cor-texto-fraco); }
.painel__acoes-cabecalho   { display: flex; gap: .5rem; }

/* Tabelas */
table       { width: 100%; border-collapse: collapse; }
thead       { background: var(--cor-fundo); }
th {
  padding: .75rem 1rem;
  text-align: left;
  font-size: .75rem;
  font-weight: 600;
  color: var(--cor-texto-fraco);
  text-transform: uppercase;
  letter-spacing: .05em;
  border-bottom: 1px solid var(--cor-borda);
}
td          { padding: .875rem 1rem; border-bottom: 1px solid var(--cor-borda); font-size: .875rem; }
tr:hover td { background: var(--cor-fundo); }

/* Paginação */
.paginacao { display: flex; align-items: center; gap: .5rem; margin-top: 1rem; justify-content: center; }

/* Seletor de idioma */
.seletor-idioma { display: flex; gap: .25rem; }

/* Datas / Prazo */
.prazo--vencido { color: var(--cor-perigo); font-weight: 600; }

/* Estado vazio */
.estado-vazio { text-align: center; padding: 3rem 1rem; color: var(--cor-texto-fraco); }

/* Indicador de atualização */
.indicador-atualizando { font-size: .75rem; color: var(--cor-texto-fraco); font-style: italic; }

/* Carregando */
.carregando-pagina { display: flex; align-items: center; justify-content: center; min-height: 200px; }
.spinner {
  width: 36px; height: 36px;
  border: 3px solid var(--cor-borda);
  border-top-color: var(--cor-primaria);
  border-radius: 50%;
  animation: girar .7s linear infinite;
}
@keyframes girar { to { transform: rotate(360deg); } }

/* Rodapé */
.rodape { border-top: 1px solid var(--cor-borda); padding: 1rem; text-align: center; font-size: .875rem; color: var(--cor-texto-fraco); }

/* Responsivo */
@media (max-width: 768px) {
  .conteudo-principal { padding: 1rem; }
  .cabecalho          { height: 56px; padding: 0 .75rem; }
  .nav-link           { padding: .375rem .5rem; font-size: .875rem; }
  .lista-produtos     { grid-template-columns: 1fr; }
  .formulario         { max-width: 100%; }
  .formulario__acoes  { flex-direction: column; }
  table               { font-size: .8125rem; }
  th, td              { padding: .625rem .75rem; }
}

@media (max-width: 480px) {
  .cabecalho    { flex-wrap: wrap; height: auto; padding: .5rem; gap: .5rem; }
  .nav-principal{ flex-wrap: wrap; }
}
```

---

## Referência Rápida de Versões

| Biblioteca | Versão | Finalidade |
|---|---|---|
| `vue` | 3.5.x | Framework UI |
| `vue-router` | 4.x | Roteamento |
| `pinia` | 2.x | Estado global |
| `vee-validate` | 4.x | Validação de formulários |
| `@vee-validate/zod` | 4.x | Adaptador Zod para VeeValidate |
| `zod` | 3.x | Schema e inferência de tipos |
| `vue-i18n` | 9.x | Internacionalização |
| `@tanstack/vue-query` | 5.x | Cache e fetching de dados |
| `date-fns` | 4.x | Manipulação de datas |
| `jwt-decode` | 4.x | Decodificação de JWT |


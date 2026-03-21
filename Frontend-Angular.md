# Angular 21 — Referência Completa

Referência consolidada sobre Angular 21, cobrindo fundamentos da plataforma, Signals, roteamento, gerenciamento de estado com NgRx, integração com backend, formulários reativos, internacionalização, manipulação de datas e autenticação JWT com controle de acesso por roles.

---

## Sumário

1. [Fundamentos do Angular 21](#1-fundamentos-do-angular-21)
   - [Configuração do Projeto](#configuração-do-projeto)
   - [Componentes Standalone](#componentes-standalone)
   - [Signals](#signals)
   - [Novo Controle de Fluxo de Template](#novo-controle-de-fluxo-de-template)
   - [Carregamento Diferido com @defer](#carregamento-diferido-com-defer)
   - [Injeção de Dependências com inject()](#injeção-de-dependências-com-inject)
   - [Pipes Essenciais](#pipes-essenciais)
2. [Gerenciamento de Rotas](#2-gerenciamento-de-rotas)
   - [Configuração do Router](#configuração-do-router)
   - [Lazy Loading de Módulos e Componentes](#lazy-loading-de-módulos-e-componentes)
   - [Guards Funcionais](#guards-funcionais)
   - [Resolvers](#resolvers)
   - [Navegação Programática](#navegação-programática)
3. [Estado Global com NgRx](#3-estado-global-com-ngrx)
   - [NgRx Store com Signals](#ngrx-store-com-signals)
   - [NgRx SignalStore](#ngrx-signalstore)
4. [Integração com Backend](#4-integração-com-backend)
   - [HttpClient e provideHttpClient](#httpclient-e-providehttpclient)
   - [Resource API e httpResource](#resource-api-e-httpresource)
   - [NgRx Effects](#ngrx-effects)
5. [Formulários Reativos e Validação](#5-formulários-reativos-e-validação)
   - [Reactive Forms](#reactive-forms)
   - [Validadores Customizados](#validadores-customizados)
   - [Formulários Template-Driven](#formulários-template-driven)
6. [Internacionalização com @ngx-translate](#6-internacionalização-com-ngx-translate)
7. [Manipulação de Datas com date-fns](#7-manipulação-de-datas-com-date-fns)
8. [Autenticação JWT e Controle de Acesso por Role](#8-autenticação-jwt-e-controle-de-acesso-por-role)
   - [Estrutura do Token JWT](#estrutura-do-token-jwt)
   - [Serviço de Autenticação](#serviço-de-autenticação)
   - [Interceptor HTTP Funcional](#interceptor-http-funcional)
   - [Guards de Autenticação e Role](#guards-de-autenticação-e-role)
   - [Diretiva de Acesso por Role](#diretiva-de-acesso-por-role)

---

## 1. Fundamentos do Angular 21

### Configuração do Projeto

```bash
# Instalar Angular CLI globalmente
npm install -g @angular/cli

# Criar projeto (standalone por padrão no Angular 21)
ng new meu-projeto --routing --style=css
cd meu-projeto

# Instalar dependências principais abordadas neste guia
npm install @ngrx/store @ngrx/effects @ngrx/entity @ngrx/signals @ngrx/operators
npm install @ngx-translate/core @ngx-translate/http-loader
npm install date-fns
npm install jwt-decode

# Gerar componentes, serviços e guards (standalone por padrão)
ng generate component features/produtos/lista-produtos
ng generate service services/auth
ng generate guard guards/auth --functional
```

Estrutura recomendada de pastas:

```
src/
├── app/
│   ├── core/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   ├── models/
│   │   └── services/
│   ├── features/
│   │   ├── auth/
│   │   ├── produtos/
│   │   └── admin/
│   ├── shared/
│   │   ├── components/
│   │   ├── directives/
│   │   └── pipes/
│   ├── store/
│   ├── app.config.ts
│   ├── app.routes.ts
│   └── app.component.ts
├── assets/
│   └── i18n/
│       ├── pt-BR.json
│       └── en-US.json
└── styles.css
```

```typescript
// src/app/app.config.ts — configuração central da aplicação
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter, withComponentInputBinding, withViewTransitions } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import { provideStore } from '@ngrx/store';
import { provideEffects } from '@ngrx/effects';
import { provideStoreDevtools } from '@ngrx/store-devtools';
import { TranslateLoader, provideTranslateService } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { HttpClient } from '@angular/common/http';

import { routes } from './app.routes';
import { authInterceptor } from './core/interceptors/auth.interceptor';
import { produtosReducer } from './store/produtos/produtos.reducer';
import { ProdutosEffects } from './store/produtos/produtos.effects';
import { authReducer } from './store/auth/auth.reducer';

function httpLoaderFactory(http: HttpClient): TranslateHttpLoader {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),

    // Router
    provideRouter(
      routes,
      withComponentInputBinding(),  // parâmetros de rota injetados como inputs
      withViewTransitions()
    ),

    // HTTP
    provideHttpClient(withInterceptors([authInterceptor])),

    // Animações
    provideAnimationsAsync(),

    // NgRx Store
    provideStore({ produtos: produtosReducer, auth: authReducer }),
    provideEffects([ProdutosEffects]),
    provideStoreDevtools({ maxAge: 25, logOnly: false }),

    // i18n
    provideTranslateService({
      defaultLanguage: 'pt-BR',
      loader: {
        provide: TranslateLoader,
        useFactory: httpLoaderFactory,
        deps: [HttpClient],
      },
    }),
  ],
};
```

```typescript
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch(console.error);
```

---

### Componentes Standalone

Desde o Angular 17, componentes standalone são o padrão. Eles importam diretamente o que precisam, sem depender de `NgModule`.

```typescript
// src/app/features/produtos/card-produto/card-produto.component.ts
import { Component, input, output, computed } from '@angular/core';
import { CurrencyPipe } from '@angular/common';
import { TranslatePipe } from '@ngx-translate/core';
import { Produto } from '../../../core/models/produto.model';

@Component({
  selector: 'app-card-produto',
  standalone: true,
  imports: [CurrencyPipe, TranslatePipe],
  template: `
    <article class="card-produto">
      <h2 class="card-produto__titulo">{{ produto().nome }}</h2>
      <p class="card-produto__preco">
        {{ produto().preco | currency:'BRL':'symbol':'1.2-2':'pt-BR' }}
      </p>
      <p class="card-produto__categoria">
        {{ 'produto.categorias.' + produto().categoria | translate }}
      </p>
      <span class="badge" [class]="badgeClass()">
        {{ produto().ativo ? ('produto.ativo' | translate) : ('produto.inativo' | translate) }}
      </span>
      <div class="card-produto__acoes">
        <button class="btn btn--secundario btn--pequeno" (click)="editar.emit(produto())">
          {{ 'produto.acoes.editar' | translate }}
        </button>
        <button class="btn btn--perigo btn--pequeno" (click)="excluir.emit(produto().id)">
          {{ 'produto.acoes.excluir' | translate }}
        </button>
      </div>
    </article>
  `,
})
export class CardProdutoComponent {
  // Signal inputs (Angular 17+)
  produto = input.required<Produto>();
  destacado = input(false);

  // Signal outputs
  editar = output<Produto>();
  excluir = output<number>();

  // Computed signal derivado de inputs
  badgeClass = computed(() =>
    this.produto().ativo ? 'badge badge--sucesso' : 'badge badge--erro'
  );
}
```

```typescript
// src/app/app.component.ts
import { Component, OnInit } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { TranslateService } from '@ngx-translate/core';
import { NavbarComponent } from './shared/components/navbar/navbar.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, NavbarComponent],
  template: `
    <div class="app">
      <app-navbar />
      <main class="conteudo-principal">
        <router-outlet />
      </main>
      <footer class="rodape">
        <p>&copy; 2025 Minha Aplicação</p>
      </footer>
    </div>
  `,
})
export class AppComponent implements OnInit {
  private translate = inject(TranslateService);

  ngOnInit(): void {
    const idiomaSalvo = localStorage.getItem('idioma') ?? 'pt-BR';
    this.translate.use(idiomaSalvo);
  }
}
```

---

### Signals

Signals são a nova primitiva de reatividade do Angular, substituindo gradualmente o Zone.js. Representam valores reativos com rastreamento automático de dependências.

```typescript
// src/app/features/contador/contador.component.ts
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-contador',
  standalone: true,
  template: `
    <div class="painel">
      <h2>Contador: {{ contagem() }}</h2>
      <p>Dobro: {{ dobro() }}</p>
      <p>É par: {{ ePar() ? 'Sim' : 'Não' }}</p>
      <div class="painel__acoes">
        <button class="btn btn--primario" (click)="incrementar()">+1</button>
        <button class="btn btn--secundario" (click)="decrementar()">-1</button>
        <button class="btn btn--secundario" (click)="resetar()">Reset</button>
      </div>
    </div>
  `,
})
export class ContadorComponent {
  // Signal mutável
  contagem = signal(0);

  // Computed signals — recalculam quando dependências mudam
  dobro = computed(() => this.contagem() * 2);
  ePar = computed(() => this.contagem() % 2 === 0);

  // Effect — efeito colateral quando signals mudam
  constructor() {
    effect(() => {
      console.log(`Contagem alterada para: ${this.contagem()}`);
    });
  }

  incrementar() { this.contagem.update((c) => c + 1); }
  decrementar() { this.contagem.update((c) => c - 1); }
  resetar()     { this.contagem.set(0); }
}
```

```typescript
// Signals em um serviço — gerenciamento de estado reativo
import { Injectable, signal, computed } from '@angular/core';
import { Produto } from '../core/models/produto.model';

@Injectable({ providedIn: 'root' })
export class CarrinhoService {
  private _itens = signal<ItemCarrinho[]>([]);
  private _aberto = signal(false);

  // Leitura pública (somente leitura)
  readonly itens = this._itens.asReadonly();
  readonly aberto = this._aberto.asReadonly();

  readonly totalItens = computed(() =>
    this._itens().reduce((acc, i) => acc + i.quantidade, 0)
  );

  readonly totalValor = computed(() =>
    this._itens().reduce((acc, i) => acc + i.preco * i.quantidade, 0)
  );

  readonly vazio = computed(() => this._itens().length === 0);

  adicionarItem(produto: Produto): void {
    this._itens.update((itens) => {
      const existente = itens.find((i) => i.id === produto.id);
      if (existente) {
        return itens.map((i) =>
          i.id === produto.id ? { ...i, quantidade: i.quantidade + 1 } : i
        );
      }
      return [...itens, { ...produto, quantidade: 1 }];
    });
  }

  removerItem(id: number): void {
    this._itens.update((itens) => itens.filter((i) => i.id !== id));
  }

  alterarQuantidade(id: number, quantidade: number): void {
    if (quantidade <= 0) { this.removerItem(id); return; }
    this._itens.update((itens) =>
      itens.map((i) => (i.id === id ? { ...i, quantidade } : i))
    );
  }

  limpar(): void { this._itens.set([]); }
  toggleAberto(): void { this._aberto.update((v) => !v); }
}

type ItemCarrinho = Produto & { quantidade: number };
```

```typescript
// linkedSignal — signal derivado e mutável (Angular 19+)
import { Component, signal, linkedSignal } from '@angular/core';

@Component({
  selector: 'app-filtro-produtos',
  standalone: true,
  template: `
    <div>
      <select (change)="categoria.set($any($event.target).value)">
        <option value="todos">Todos</option>
        <option value="eletronicos">Eletrônicos</option>
        <option value="livros">Livros</option>
      </select>
      <input [value]="busca()" (input)="busca.set($any($event.target).value)"
             placeholder="Buscar em {{ categoria() }}..." />
    </div>
  `,
})
export class FiltroProdutosComponent {
  categoria = signal('todos');

  // linkedSignal reseta 'busca' sempre que 'categoria' mudar
  busca = linkedSignal({
    source: this.categoria,
    computation: () => '',
  });
}
```

---

### Novo Controle de Fluxo de Template

O Angular 17+ introduziu sintaxe nativa de controle de fluxo (`@if`, `@for`, `@switch`), substituindo `*ngIf`, `*ngFor` e `*ngSwitch`.

```typescript
@Component({
  selector: 'app-lista-produtos',
  standalone: true,
  imports: [CardProdutoComponent, CurrencyPipe, TranslatePipe],
  template: `
    <!-- @if substitui *ngIf -->
    @if (carregando()) {
      <div class="carregando-pagina">
        <div class="spinner" aria-label="Carregando..."></div>
      </div>
    } @else if (erro()) {
      <div class="alerta alerta--erro" role="alert">
        <p>{{ erro() }}</p>
        <button class="btn btn--secundario" (click)="recarregar()">Tentar novamente</button>
      </div>
    } @else if (produtos().length === 0) {
      <div class="estado-vazio">
        <p>{{ 'produto.semResultados' | translate }}</p>
      </div>
    } @else {
      <!-- @for substitui *ngFor, com @empty nativo -->
      <ul class="lista-produtos">
        @for (produto of produtosFiltrados(); track produto.id) {
          <li>
            <app-card-produto
              [produto]="produto"
              (editar)="abrirEdicao($event)"
              (excluir)="confirmarExclusao($event)"
            />
          </li>
        } @empty {
          <li class="estado-vazio">Nenhum produto encontrado para o filtro.</li>
        }
      </ul>

      <!-- @switch substitui *ngSwitch -->
      @switch (visualizacao()) {
        @case ('grade') { <span class="badge badge--info">Grade</span> }
        @case ('lista') { <span class="badge badge--info">Lista</span> }
        @default       { <span class="badge">Padrão</span> }
      }
    }
  `,
})
export class ListaProdutosComponent {
  // ... implementação na seção de NgRx
}
```

---

### Carregamento Diferido com @defer

`@defer` permite lazy loading declarativo de componentes no template.

```typescript
@Component({
  selector: 'app-pagina-produtos',
  standalone: true,
  imports: [FiltrosProdutosComponent],
  template: `
    <h1>{{ 'produto.lista' | translate }}</h1>

    <!-- Carrega o componente pesado apenas quando visível na tela -->
    @defer (on viewport) {
      <app-grafico-vendas />
    } @placeholder {
      <div class="placeholder-grafico">Carregando gráfico...</div>
    } @loading (minimum 500ms) {
      <div class="carregando-pagina"><div class="spinner"></div></div>
    } @error {
      <p class="alerta alerta--erro">Erro ao carregar o gráfico.</p>
    }

    <!-- Carrega quando o usuário interage (hover, focus, click) -->
    @defer (on interaction) {
      <app-modal-detalhe-produto />
    } @placeholder {
      <button class="btn btn--secundario">Ver detalhes</button>
    }

    <!-- Carrega após N segundos de idle -->
    @defer (on idle; prefetch on immediate) {
      <app-produtos-relacionados />
    }

    <!-- Carrega condicionalmente via signal/variável -->
    @defer (when mostrarDetalhes()) {
      <app-painel-detalhes />
    }
  `,
})
export class PaginaProdutosComponent {
  mostrarDetalhes = signal(false);
}
```

---

### Injeção de Dependências com inject()

A função `inject()` é a forma moderna de obter dependências, usável em construtores, factories e fora de classes.

```typescript
import { inject, DestroyRef } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

// Uso em um componente standalone
@Component({ /* ... */ })
export class ExemploComponent {
  private router    = inject(Router);
  private authSvc   = inject(AuthService);
  private destroyRef = inject(DestroyRef);

  ngOnInit(): void {
    // takeUntilDestroyed cancela a assinatura automaticamente
    this.authSvc.usuario$.pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe((usuario) => {
      if (!usuario) this.router.navigate(['/login']);
    });
  }
}

// Função utilitária reutilizável (inject fora de classe)
export function injectPaginacao() {
  const route  = inject(ActivatedRoute);
  const router = inject(Router);

  const pagina = signal(Number(route.snapshot.queryParamMap.get('pagina') ?? 1));

  function irParaPagina(p: number): void {
    router.navigate([], { queryParams: { pagina: p }, queryParamsHandling: 'merge' });
    pagina.set(p);
  }

  return { pagina: pagina.asReadonly(), irParaPagina };
}
```

---

### Pipes Essenciais

```typescript
// Pipes nativos — Angular importa por demanda nos componentes standalone
import {
  DatePipe,
  CurrencyPipe,
  DecimalPipe,
  PercentPipe,
  UpperCasePipe,
  LowerCasePipe,
  TitleCasePipe,
  SlicePipe,
  JsonPipe,
  AsyncPipe,
  KeyValuePipe,
} from '@angular/common';

// Exemplo de uso no template
@Component({
  standalone: true,
  imports: [DatePipe, CurrencyPipe, DecimalPipe, PercentPipe, AsyncPipe],
  template: `
    <p>{{ hoje | date:'dd/MM/yyyy HH:mm':'':'pt-BR' }}</p>
    <p>{{ preco | currency:'BRL':'symbol':'1.2-2':'pt-BR' }}</p>
    <p>{{ numero | number:'1.0-2':'pt-BR' }}</p>
    <p>{{ taxa | percent:'1.1-2':'pt-BR' }}</p>

    <!-- AsyncPipe: assina Observable/Promise e desinscreve automaticamente -->
    @if (usuario$ | async; as usuario) {
      <p>Olá, {{ usuario.nome }}</p>
    }
  `,
})
export class ExemploPipesComponent {
  hoje = new Date();
  preco = 1299.90;
  numero = 12345.678;
  taxa = 0.0756;
  usuario$ = inject(AuthService).usuario$;
}

// Pipe customizado com DatePipe + date-fns
import { Pipe, PipeTransform } from '@angular/core';
import { formatDistanceToNow, parseISO, isValid } from 'date-fns';
import { ptBR, enUS } from 'date-fns/locale';
import { TranslateService } from '@ngx-translate/core';

@Pipe({ name: 'dataRelativa', standalone: true, pure: false })
export class DataRelativaPipe implements PipeTransform {
  private translate = inject(TranslateService);

  transform(valor: string | Date | null | undefined): string {
    if (!valor) return '—';
    const data = typeof valor === 'string' ? parseISO(valor) : valor;
    if (!isValid(data)) return '—';
    const locale = this.translate.currentLang === 'en-US' ? enUS : ptBR;
    return formatDistanceToNow(data, { addSuffix: true, locale });
  }
}
```

---

## 2. Gerenciamento de Rotas

### Configuração do Router

```typescript
// src/app/app.routes.ts
import { Routes } from '@angular/router';
import { authGuard } from './core/guards/auth.guard';
import { roleGuard } from './core/guards/role.guard';
import { produtoResolver } from './core/resolvers/produto.resolver';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./features/home/home.component').then((m) => m.HomeComponent),
    title: 'Início',
  },
  {
    path: 'produtos',
    title: 'Produtos',
    children: [
      {
        path: '',
        loadComponent: () =>
          import('./features/produtos/lista-produtos/lista-produtos.component')
            .then((m) => m.ListaProdutosComponent),
      },
      {
        path: ':id',
        loadComponent: () =>
          import('./features/produtos/detalhe-produto/detalhe-produto.component')
            .then((m) => m.DetalheProdutoComponent),
        resolve: { produto: produtoResolver },
        title: (route) => `Produto ${route.paramMap.get('id')}`,
      },
    ],
  },
  {
    path: 'painel',
    canActivate: [authGuard],
    children: [
      {
        path: 'perfil',
        loadComponent: () =>
          import('./features/perfil/perfil.component').then((m) => m.PerfilComponent),
      },
      {
        path: 'admin',
        canActivate: [roleGuard(['ADMIN'])],
        loadComponent: () =>
          import('./features/admin/admin.component').then((m) => m.AdminComponent),
      },
      {
        path: 'relatorios',
        canActivate: [roleGuard(['ADMIN', 'GERENTE'])],
        loadComponent: () =>
          import('./features/relatorios/relatorios.component')
            .then((m) => m.RelatoriosComponent),
      },
    ],
  },
  {
    path: 'login',
    loadComponent: () =>
      import('./features/auth/login/login.component').then((m) => m.LoginComponent),
    title: 'Entrar',
  },
  {
    path: '403',
    loadComponent: () =>
      import('./shared/components/acesso-negado/acesso-negado.component')
        .then((m) => m.AcessoNegadoComponent),
  },
  {
    path: '**',
    loadComponent: () =>
      import('./shared/components/nao-encontrado/nao-encontrado.component')
        .then((m) => m.NaoEncontradoComponent),
  },
];
```

```typescript
// src/app/shared/components/navbar/navbar.component.ts
import { Component, inject } from '@angular/core';
import { RouterLink, RouterLinkActive } from '@angular/router';
import { TranslatePipe } from '@ngx-translate/core';
import { AuthService } from '../../../core/services/auth.service';
import { AcessoRestritoPipe } from '../../pipes/acesso-restrito.pipe';

@Component({
  selector: 'app-navbar',
  standalone: true,
  imports: [RouterLink, RouterLinkActive, TranslatePipe],
  template: `
    <header class="cabecalho">
      <nav class="nav-principal" aria-label="Navegação principal">
        <a routerLink="/" routerLinkActive="nav-link--ativo" [routerLinkActiveOptions]="{ exact: true }" class="nav-link">
          {{ 'nav.inicio' | translate }}
        </a>
        <a routerLink="/produtos" routerLinkActive="nav-link--ativo" class="nav-link">
          {{ 'nav.produtos' | translate }}
        </a>

        @if (auth.autenticado()) {
          <a routerLink="/painel/perfil" routerLinkActive="nav-link--ativo" class="nav-link">
            {{ 'nav.perfil' | translate }}
          </a>
        }
        @if (auth.temRole('GERENTE') || auth.temRole('ADMIN')) {
          <a routerLink="/painel/relatorios" routerLinkActive="nav-link--ativo" class="nav-link">
            Relatórios
          </a>
        }
        @if (auth.temRole('ADMIN')) {
          <a routerLink="/painel/admin" routerLinkActive="nav-link--ativo" class="nav-link">
            {{ 'nav.admin' | translate }}
          </a>
        }
      </nav>

      <div class="cabecalho__acoes">
        @if (auth.autenticado()) {
          <span class="usuario-nome">{{ auth.usuario()?.nome }}</span>
          <button class="btn btn--secundario" (click)="auth.sair()">
            {{ 'nav.sair' | translate }}
          </button>
        } @else {
          <a routerLink="/login" class="btn btn--primario">
            {{ 'nav.entrar' | translate }}
          </a>
        }
      </div>
    </header>
  `,
})
export class NavbarComponent {
  auth = inject(AuthService);
}
```

---

### Lazy Loading de Módulos e Componentes

```typescript
// Lazy loading de rotas filhas agrupadas em um arquivo de rotas
// src/app/features/produtos/produtos.routes.ts
import { Routes } from '@angular/router';

export const produtosRoutes: Routes = [
  { path: '', loadComponent: () => import('./lista-produtos/lista-produtos.component').then(m => m.ListaProdutosComponent) },
  { path: 'novo', loadComponent: () => import('./formulario-produto/formulario-produto.component').then(m => m.FormularioProdutoComponent) },
  { path: ':id', loadComponent: () => import('./detalhe-produto/detalhe-produto.component').then(m => m.DetalheProdutoComponent) },
  { path: ':id/editar', loadComponent: () => import('./formulario-produto/formulario-produto.component').then(m => m.FormularioProdutoComponent) },
];

// No app.routes.ts, carrega o grupo:
{
  path: 'produtos',
  loadChildren: () =>
    import('./features/produtos/produtos.routes').then((m) => m.produtosRoutes),
}
```

---

### Guards Funcionais

```typescript
// src/app/core/guards/auth.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth   = inject(AuthService);
  const router = inject(Router);

  if (auth.autenticado()) return true;

  // Salva URL destino para redirecionar após login
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url },
  });
};
```

```typescript
// src/app/core/guards/role.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export function roleGuard(rolesPermitidas: string[]): CanActivateFn {
  return () => {
    const auth   = inject(AuthService);
    const router = inject(Router);

    if (!auth.autenticado()) {
      return router.createUrlTree(['/login']);
    }

    if (rolesPermitidas.some((r) => auth.temRole(r))) {
      return true;
    }

    return router.createUrlTree(['/403']);
  };
}
```

---

### Resolvers

```typescript
// src/app/core/resolvers/produto.resolver.ts
import { inject } from '@angular/core';
import { ResolveFn, Router } from '@angular/router';
import { catchError, EMPTY } from 'rxjs';
import { ProdutosService } from '../services/produtos.service';
import { Produto } from '../models/produto.model';

export const produtoResolver: ResolveFn<Produto> = (route) => {
  const svc    = inject(ProdutosService);
  const router = inject(Router);
  const id     = Number(route.paramMap.get('id'));

  return svc.buscarPorId(id).pipe(
    catchError(() => {
      router.navigate(['**']);
      return EMPTY;
    })
  );
};

// No componente, o dado resolvido chega via input (withComponentInputBinding)
@Component({ /* ... */ })
export class DetalheProdutoComponent {
  // Angular injeta o resultado do resolver diretamente como input
  produto = input.required<Produto>();
}
```

---

### Navegação Programática

```typescript
import { Component, inject } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';

@Component({ standalone: true, template: `...` })
export class ExemploNavegacaoComponent {
  private router = inject(Router);
  private route  = inject(ActivatedRoute);

  // Parâmetros de rota como inputs (withComponentInputBinding)
  id = input<string>();

  irParaProduto(id: number): void {
    this.router.navigate(['/produtos', id]);
  }

  irComQueryParams(): void {
    this.router.navigate(['/produtos'], {
      queryParams: { pagina: 2, busca: 'notebook' },
      queryParamsHandling: 'merge',  // preserva outros query params
    });
  }

  voltarOuInicio(): void {
    const returnUrl = this.route.snapshot.queryParamMap.get('returnUrl');
    this.router.navigateByUrl(returnUrl ?? '/');
  }

  abrirEmNovaAba(id: number): void {
    const url = this.router.serializeUrl(
      this.router.createUrlTree(['/produtos', id])
    );
    window.open(url, '_blank');
  }
}
```

---

## 3. Estado Global com NgRx

### NgRx Store com Signals

```typescript
// src/app/core/models/produto.model.ts
export type Produto = {
  id: number;
  nome: string;
  preco: number;
  categoria: 'eletronicos' | 'livros' | 'roupas' | 'alimentos';
  ativo: boolean;
  descricao?: string;
};

export type PaginaResposta<T> = {
  itens: T[];
  total: number;
  pagina: number;
  totalPaginas: number;
};
```

```typescript
// src/app/store/produtos/produtos.actions.ts
import { createActionGroup, emptyProps, props } from '@ngrx/store';
import { Produto, PaginaResposta } from '../../core/models/produto.model';

export const ProdutosActions = createActionGroup({
  source: 'Produtos',
  events: {
    'Carregar Produtos':        props<{ pagina: number; busca?: string }>(),
    'Carregar Produtos Sucesso': props<{ dados: PaginaResposta<Produto> }>(),
    'Carregar Produtos Falha':  props<{ erro: string }>(),

    'Criar Produto':        props<{ produto: Omit<Produto, 'id'> }>(),
    'Criar Produto Sucesso': props<{ produto: Produto }>(),
    'Criar Produto Falha':  props<{ erro: string }>(),

    'Atualizar Produto':        props<{ produto: Produto }>(),
    'Atualizar Produto Sucesso': props<{ produto: Produto }>(),
    'Atualizar Produto Falha':  props<{ erro: string }>(),

    'Excluir Produto':        props<{ id: number }>(),
    'Excluir Produto Sucesso': props<{ id: number }>(),
    'Excluir Produto Falha':  props<{ erro: string }>(),

    'Selecionar Produto': props<{ id: number | null }>(),
    'Definir Filtro':     props<{ busca: string }>(),
  },
});
```

```typescript
// src/app/store/produtos/produtos.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { ProdutosActions } from './produtos.actions';
import { Produto } from '../../core/models/produto.model';

export type ProdutosState = {
  itens: Produto[];
  total: number;
  pagina: number;
  totalPaginas: number;
  carregando: boolean;
  salvando: boolean;
  erro: string | null;
  produtoSelecionadoId: number | null;
  filtroBusca: string;
};

const estadoInicial: ProdutosState = {
  itens: [],
  total: 0,
  pagina: 1,
  totalPaginas: 1,
  carregando: false,
  salvando: false,
  erro: null,
  produtoSelecionadoId: null,
  filtroBusca: '',
};

export const produtosReducer = createReducer(
  estadoInicial,

  on(ProdutosActions.carregarProdutos, (s) => ({ ...s, carregando: true, erro: null })),
  on(ProdutosActions.carregarProdutosSucesso, (s, { dados }) => ({
    ...s,
    carregando: false,
    itens: dados.itens,
    total: dados.total,
    pagina: dados.pagina,
    totalPaginas: dados.totalPaginas,
  })),
  on(ProdutosActions.carregarProdutosFalha, (s, { erro }) => ({
    ...s, carregando: false, erro,
  })),

  on(ProdutosActions.criarProduto, (s) => ({ ...s, salvando: true, erro: null })),
  on(ProdutosActions.criarProdutoSucesso, (s, { produto }) => ({
    ...s, salvando: false, itens: [...s.itens, produto],
  })),
  on(ProdutosActions.criarProdutoFalha, (s, { erro }) => ({
    ...s, salvando: false, erro,
  })),

  on(ProdutosActions.atualizarProdutoSucesso, (s, { produto }) => ({
    ...s,
    salvando: false,
    itens: s.itens.map((p) => (p.id === produto.id ? produto : p)),
  })),

  on(ProdutosActions.excluirProdutoSucesso, (s, { id }) => ({
    ...s, itens: s.itens.filter((p) => p.id !== id),
  })),

  on(ProdutosActions.selecionarProduto, (s, { id }) => ({
    ...s, produtoSelecionadoId: id,
  })),

  on(ProdutosActions.definirFiltro, (s, { busca }) => ({
    ...s, filtroBusca: busca,
  })),
);
```

```typescript
// src/app/store/produtos/produtos.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { ProdutosState } from './produtos.reducer';

const selectFeature = createFeatureSelector<ProdutosState>('produtos');

export const selectProdutos        = createSelector(selectFeature, (s) => s.itens);
export const selectCarregando      = createSelector(selectFeature, (s) => s.carregando);
export const selectSalvando        = createSelector(selectFeature, (s) => s.salvando);
export const selectErro            = createSelector(selectFeature, (s) => s.erro);
export const selectPagina          = createSelector(selectFeature, (s) => s.pagina);
export const selectTotalPaginas    = createSelector(selectFeature, (s) => s.totalPaginas);
export const selectFiltroBusca     = createSelector(selectFeature, (s) => s.filtroBusca);
export const selectProdutoSelecionado = createSelector(
  selectFeature,
  (s) => s.itens.find((p) => p.id === s.produtoSelecionadoId) ?? null
);
export const selectProdutosFiltrados = createSelector(
  selectProdutos,
  selectFiltroBusca,
  (produtos, busca) =>
    busca
      ? produtos.filter((p) => p.nome.toLowerCase().includes(busca.toLowerCase()))
      : produtos
);
```

```typescript
// src/app/store/produtos/produtos.effects.ts
import { inject } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, exhaustMap, map, of, switchMap } from 'rxjs';
import { ProdutosActions } from './produtos.actions';
import { ProdutosService } from '../../core/services/produtos.service';

export class ProdutosEffects {
  private actions$ = inject(Actions);
  private svc      = inject(ProdutosService);

  carregar$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProdutosActions.carregarProdutos),
      switchMap(({ pagina, busca }) =>
        this.svc.listar(pagina, busca).pipe(
          map((dados) => ProdutosActions.carregarProdutosSucesso({ dados })),
          catchError((e: Error) =>
            of(ProdutosActions.carregarProdutosFalha({ erro: e.message }))
          )
        )
      )
    )
  );

  criar$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProdutosActions.criarProduto),
      exhaustMap(({ produto }) =>
        this.svc.criar(produto).pipe(
          map((criado) => ProdutosActions.criarProdutoSucesso({ produto: criado })),
          catchError((e: Error) =>
            of(ProdutosActions.criarProdutoFalha({ erro: e.message }))
          )
        )
      )
    )
  );

  atualizar$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProdutosActions.atualizarProduto),
      exhaustMap(({ produto }) =>
        this.svc.atualizar(produto).pipe(
          map((atualizado) => ProdutosActions.atualizarProdutoSucesso({ produto: atualizado })),
          catchError((e: Error) =>
            of(ProdutosActions.atualizarProdutoFalha({ erro: e.message }))
          )
        )
      )
    )
  );

  excluir$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProdutosActions.excluirProduto),
      exhaustMap(({ id }) =>
        this.svc.excluir(id).pipe(
          map(() => ProdutosActions.excluirProdutoSucesso({ id })),
          catchError((e: Error) =>
            of(ProdutosActions.excluirProdutoFalha({ erro: e.message }))
          )
        )
      )
    )
  );
}
```

```typescript
// Usando NgRx no componente com selectSignal
import { Component, OnInit, inject, signal, computed } from '@angular/core';
import { Store } from '@ngrx/store';
import { ProdutosActions } from '../../../store/produtos/produtos.actions';
import {
  selectProdutosFiltrados,
  selectCarregando,
  selectErro,
  selectPagina,
  selectTotalPaginas,
} from '../../../store/produtos/produtos.selectors';

@Component({
  selector: 'app-lista-produtos',
  standalone: true,
  imports: [CardProdutoComponent, TranslatePipe],
  template: `
    <div class="pagina-produtos">
      <div class="pagina-produtos__cabecalho">
        <h1>{{ 'produto.lista' | translate }}</h1>
        <input
          class="input"
          [value]="filtroBusca()"
          (input)="filtrar($any($event.target).value)"
          placeholder="Buscar produtos..."
        />
      </div>

      @if (carregando()) {
        <div class="carregando-pagina"><div class="spinner"></div></div>
      } @else if (erro()) {
        <div class="alerta alerta--erro" role="alert">{{ erro() }}</div>
      } @else {
        <ul class="lista-produtos">
          @for (produto of produtos(); track produto.id) {
            <li>
              <app-card-produto
                [produto]="produto"
                (excluir)="excluir($event)"
              />
            </li>
          } @empty {
            <li class="estado-vazio">Nenhum produto encontrado.</li>
          }
        </ul>

        <div class="paginacao">
          <button class="btn btn--secundario" (click)="mudarPagina(pagina() - 1)" [disabled]="pagina() === 1">
            Anterior
          </button>
          <span>Página {{ pagina() }} de {{ totalPaginas() }}</span>
          <button class="btn btn--secundario" (click)="mudarPagina(pagina() + 1)" [disabled]="pagina() === totalPaginas()">
            Próxima
          </button>
        </div>
      }
    </div>
  `,
})
export class ListaProdutosComponent implements OnInit {
  private store = inject(Store);

  // selectSignal converte seletores NgRx em Signals
  produtos     = this.store.selectSignal(selectProdutosFiltrados);
  carregando   = this.store.selectSignal(selectCarregando);
  erro         = this.store.selectSignal(selectErro);
  pagina       = this.store.selectSignal(selectPagina);
  totalPaginas = this.store.selectSignal(selectTotalPaginas);
  filtroBusca  = signal('');

  ngOnInit(): void {
    this.store.dispatch(ProdutosActions.carregarProdutos({ pagina: 1 }));
  }

  filtrar(busca: string): void {
    this.filtroBusca.set(busca);
    this.store.dispatch(ProdutosActions.definirFiltro({ busca }));
  }

  mudarPagina(p: number): void {
    this.store.dispatch(ProdutosActions.carregarProdutos({ pagina: p }));
  }

  excluir(id: number): void {
    this.store.dispatch(ProdutosActions.excluirProduto({ id }));
  }
}
```

---

### NgRx SignalStore

NgRx SignalStore é uma API mais enxuta, sem actions/reducers/effects separados, ideal para estados locais ou de features.

```typescript
// src/app/features/carrinho/carrinho.store.ts
import { signalStore, withState, withComputed, withMethods, patchState } from '@ngrx/signals';
import { rxMethod } from '@ngrx/signals/rxjs-interop';
import { computed, inject } from '@angular/core';
import { pipe, switchMap, tap } from 'rxjs';
import { tapResponse } from '@ngrx/operators';
import { ProdutosService } from '../../core/services/produtos.service';
import { Produto } from '../../core/models/produto.model';

type ItemCarrinho = Produto & { quantidade: number };

type CarrinhoState = {
  itens: ItemCarrinho[];
  aberto: boolean;
  carregando: boolean;
};

export const CarrinhoStore = signalStore(
  { providedIn: 'root' },

  withState<CarrinhoState>({
    itens: [],
    aberto: false,
    carregando: false,
  }),

  withComputed(({ itens }) => ({
    totalItens: computed(() => itens().reduce((a, i) => a + i.quantidade, 0)),
    totalValor: computed(() => itens().reduce((a, i) => a + i.preco * i.quantidade, 0)),
    vazio:      computed(() => itens().length === 0),
  })),

  withMethods((store) => ({
    adicionarItem(produto: Produto): void {
      patchState(store, (s) => {
        const existente = s.itens.find((i) => i.id === produto.id);
        const novosItens = existente
          ? s.itens.map((i) => i.id === produto.id ? { ...i, quantidade: i.quantidade + 1 } : i)
          : [...s.itens, { ...produto, quantidade: 1 }];
        return { itens: novosItens };
      });
    },

    removerItem(id: number): void {
      patchState(store, (s) => ({ itens: s.itens.filter((i) => i.id !== id) }));
    },

    alterarQuantidade(id: number, quantidade: number): void {
      if (quantidade <= 0) {
        patchState(store, (s) => ({ itens: s.itens.filter((i) => i.id !== id) }));
        return;
      }
      patchState(store, (s) => ({
        itens: s.itens.map((i) => (i.id === id ? { ...i, quantidade } : i)),
      }));
    },

    limpar(): void { patchState(store, { itens: [] }); },
    toggleAberto(): void { patchState(store, (s) => ({ aberto: !s.aberto })); },
  }))
);

// Uso nos componentes
@Component({ /* ... */ })
export class CarrinhoComponent {
  carrinho = inject(CarrinhoStore);

  // Acesso direto aos signals
  // this.carrinho.itens()     — array de itens
  // this.carrinho.totalItens() — computed
  // this.carrinho.adicionarItem(produto) — método
}
```

---

## 4. Integração com Backend

### HttpClient e provideHttpClient

```typescript
// src/app/core/services/produtos.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';
import { Produto, PaginaResposta } from '../models/produto.model';

@Injectable({ providedIn: 'root' })
export class ProdutosService {
  private http = inject(HttpClient);
  private url  = `${environment.apiUrl}/produtos`;

  listar(pagina = 1, busca?: string): Observable<PaginaResposta<Produto>> {
    let params = new HttpParams().set('pagina', pagina);
    if (busca) params = params.set('busca', busca);
    return this.http.get<PaginaResposta<Produto>>(this.url, { params });
  }

  buscarPorId(id: number): Observable<Produto> {
    return this.http.get<Produto>(`${this.url}/${id}`);
  }

  criar(produto: Omit<Produto, 'id'>): Observable<Produto> {
    return this.http.post<Produto>(this.url, produto);
  }

  atualizar(produto: Produto): Observable<Produto> {
    return this.http.put<Produto>(`${this.url}/${produto.id}`, produto);
  }

  excluir(id: number): Observable<void> {
    return this.http.delete<void>(`${this.url}/${id}`);
  }
}
```

```typescript
// src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api',
};

// src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://api.minhaapp.com/api',
};
```

---

### Resource API e httpResource

O Angular 19+ introduziu a Resource API, que integra nativa e reativamente Signals com requisições HTTP.

```typescript
// src/app/features/produtos/detalhe-produto/detalhe-produto.component.ts
import { Component, inject, input, resource, computed } from '@angular/core';
import { httpResource } from '@angular/core'; // Angular 19+
import { environment } from '../../../../environments/environment';
import { Produto } from '../../../core/models/produto.model';
import { DataRelativaPipe } from '../../../shared/pipes/data-relativa.pipe';
import { CurrencyPipe } from '@angular/common';

@Component({
  selector: 'app-detalhe-produto',
  standalone: true,
  imports: [CurrencyPipe, DataRelativaPipe],
  template: `
    @if (produtoResource.isLoading()) {
      <div class="carregando-pagina"><div class="spinner"></div></div>
    } @else if (produtoResource.error()) {
      <div class="alerta alerta--erro">Erro ao carregar produto.</div>
    } @else if (produtoResource.value(); as produto) {
      <article class="card-produto">
        <h1>{{ produto.nome }}</h1>
        <p>{{ produto.preco | currency:'BRL':'symbol':'1.2-2':'pt-BR' }}</p>
        <p>Criado {{ produto.criadoEm | dataRelativa }}</p>
      </article>
    }
  `,
})
export class DetalheProdutoComponent {
  id = input.required<number>();

  // httpResource: busca automática quando 'id' muda
  produtoResource = httpResource<Produto>(() =>
    `${environment.apiUrl}/produtos/${this.id()}`
  );
}

// resource() com lógica customizada
import { resource } from '@angular/core';
import { firstValueFrom } from 'rxjs';
import { ProdutosService } from '../../../core/services/produtos.service';

@Component({ /* ... */ })
export class OutroProdutoComponent {
  private svc = inject(ProdutosService);
  pagina = signal(1);
  busca  = signal('');

  // Rebusca automaticamente quando pagina() ou busca() mudam
  listaProdutos = resource({
    request: () => ({ pagina: this.pagina(), busca: this.busca() }),
    loader: ({ request }) =>
      firstValueFrom(this.svc.listar(request.pagina, request.busca)),
  });

  // Acesso ao estado
  // listaProdutos.isLoading()
  // listaProdutos.value()
  // listaProdutos.error()
  // listaProdutos.reload() — força nova busca
}
```

---

### NgRx Effects

Effects isolam efeitos colaterais (chamadas HTTP, localStorage, etc.) do reducer.

```typescript
// Operadores úteis em Effects
import { switchMap, mergeMap, exhaustMap, concatMap } from 'rxjs';
// switchMap  — cancela a anterior ao receber nova ação (ex: busca ao digitar)
// mergeMap   — executa em paralelo (ex: upload de múltiplos arquivos)
// exhaustMap — ignora novas ações enquanto a atual está em andamento (ex: login)
// concatMap  — enfileira em ordem (ex: pedidos sequenciais)

// Effect com notificação de sucesso
import { ToastService } from '../../core/services/toast.service';

export class ProdutosEffects {
  private toast = inject(ToastService);

  notificarSucesso$ = createEffect(
    () =>
      this.actions$.pipe(
        ofType(
          ProdutosActions.criarProdutoSucesso,
          ProdutosActions.atualizarProdutoSucesso,
          ProdutosActions.excluirProdutoSucesso
        ),
        tap((action) => {
          const mensagem =
            'produto' in action
              ? 'Produto salvo com sucesso!'
              : 'Produto excluído.';
          this.toast.mostrar(mensagem, 'sucesso');
        })
      ),
    { dispatch: false } // não despacha nova action
  );
}
```

---

## 5. Formulários Reativos e Validação

### Reactive Forms

Formulários reativos oferecem controle total via TypeScript, facilitando testes e validações complexas.

```typescript
// src/app/core/validators/validadores.ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Validador de preço positivo com precisão
export function precoValido(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const valor = control.value;
    if (valor === null || valor === undefined || valor === '') return null;
    if (isNaN(valor) || valor <= 0) return { precoInvalido: true };
    if (!/^\d+(\.\d{1,2})?$/.test(String(valor))) return { precisaoInvalida: true };
    return null;
  };
}

// Validador de CNPJ
export function cnpjValido(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const valor = control.value?.replace(/\D/g, '');
    if (!valor || valor.length !== 14) return { cnpjInvalido: true };
    if (/^(\d)\1{13}$/.test(valor)) return { cnpjInvalido: true };
    // algoritmo de validação omitido por brevidade
    return null;
  };
}

// Validador de confirmação de senha
export function senhasIguais(campo1: string, campo2: string): ValidatorFn {
  return (group: AbstractControl): ValidationErrors | null => {
    const s1 = group.get(campo1)?.value;
    const s2 = group.get(campo2)?.value;
    return s1 === s2 ? null : { senhasNaoConferem: true };
  };
}

// Validador assíncrono (ex: verificar email único no backend)
import { AsyncValidatorFn } from '@angular/forms';
import { HttpClient } from '@angular/common/http';
import { map, catchError, of, debounceTime, distinctUntilChanged, switchMap } from 'rxjs';

export function emailUnico(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl) => {
    if (!control.value) return of(null);
    return of(control.value).pipe(
      debounceTime(400),
      distinctUntilChanged(),
      switchMap((email) =>
        http.get<{ disponivel: boolean }>(`/api/usuarios/verificar-email?email=${email}`).pipe(
          map((res) => (res.disponivel ? null : { emailEmUso: true })),
          catchError(() => of(null))
        )
      )
    );
  };
}
```

```typescript
// src/app/features/produtos/formulario-produto/formulario-produto.component.ts
import { Component, OnInit, inject, input, output, computed } from '@angular/core';
import {
  FormBuilder,
  FormGroup,
  Validators,
  ReactiveFormsModule,
  AbstractControl,
} from '@angular/forms';
import { TranslatePipe } from '@ngx-translate/core';
import { Produto } from '../../../core/models/produto.model';
import { precoValido } from '../../../core/validators/validadores';
import { Store } from '@ngrx/store';
import { ProdutosActions } from '../../../store/produtos/produtos.actions';
import { selectSalvando } from '../../../store/produtos/produtos.selectors';

@Component({
  selector: 'app-formulario-produto',
  standalone: true,
  imports: [ReactiveFormsModule, TranslatePipe],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()" class="formulario" novalidate>
      <h2>{{ editando() ? ('produto.acoes.editar' | translate) : ('produto.acoes.criar' | translate) }}</h2>

      <!-- Nome -->
      <div class="campo">
        <label for="nome">{{ 'produto.nome' | translate }} *</label>
        <input
          id="nome"
          formControlName="nome"
          class="input"
          [class.input--erro]="campoInvalido('nome')"
          [attr.aria-describedby]="campoInvalido('nome') ? 'nome-erro' : null"
          [attr.aria-invalid]="campoInvalido('nome')"
        />
        @if (campoInvalido('nome')) {
          <span id="nome-erro" class="campo__erro" role="alert">
            @if (form.get('nome')?.hasError('required')) { Campo obrigatório. }
            @else if (form.get('nome')?.hasError('minlength')) { Mínimo 3 caracteres. }
            @else if (form.get('nome')?.hasError('maxlength')) { Máximo 100 caracteres. }
          </span>
        }
      </div>

      <!-- Preço -->
      <div class="campo">
        <label for="preco">{{ 'produto.preco' | translate }} *</label>
        <input
          id="preco"
          type="number"
          step="0.01"
          formControlName="preco"
          class="input"
          [class.input--erro]="campoInvalido('preco')"
        />
        @if (campoInvalido('preco')) {
          <span class="campo__erro" role="alert">
            @if (form.get('preco')?.hasError('required')) { Campo obrigatório. }
            @else if (form.get('preco')?.hasError('precoInvalido')) { Preço deve ser positivo. }
            @else if (form.get('preco')?.hasError('precisaoInvalida')) { Máximo 2 casas decimais. }
          </span>
        }
      </div>

      <!-- Categoria -->
      <div class="campo">
        <label for="categoria">{{ 'produto.categoria' | translate }} *</label>
        <select id="categoria" formControlName="categoria" class="select"
                [class.select--erro]="campoInvalido('categoria')">
          <option value="">Selecione...</option>
          <option value="eletronicos">{{ 'produto.categorias.eletronicos' | translate }}</option>
          <option value="livros">{{ 'produto.categorias.livros' | translate }}</option>
          <option value="roupas">{{ 'produto.categorias.roupas' | translate }}</option>
          <option value="alimentos">{{ 'produto.categorias.alimentos' | translate }}</option>
        </select>
        @if (campoInvalido('categoria')) {
          <span class="campo__erro" role="alert">Selecione uma categoria.</span>
        }
      </div>

      <!-- Descrição -->
      <div class="campo">
        <label for="descricao">Descrição</label>
        <textarea id="descricao" formControlName="descricao" class="textarea" rows="3"></textarea>
        <small class="campo__dica">
          {{ form.get('descricao')?.value?.length ?? 0 }}/500
        </small>
      </div>

      <!-- Ativo -->
      <div class="campo campo--checkbox">
        <input id="ativo" type="checkbox" formControlName="ativo" />
        <label for="ativo">Produto ativo</label>
      </div>

      <div class="formulario__acoes">
        <button
          type="submit"
          class="btn btn--primario"
          [disabled]="form.invalid || form.pristine || salvando()"
        >
          {{ salvando() ? 'Salvando...' : ('produto.acoes.salvar' | translate) }}
        </button>
        <button type="button" class="btn btn--secundario" (click)="cancelar.emit()">
          {{ 'produto.acoes.cancelar' | translate }}
        </button>
      </div>
    </form>
  `,
})
export class FormularioProdutoComponent implements OnInit {
  private fb    = inject(FormBuilder);
  private store = inject(Store);

  produto = input<Produto | null>(null);
  cancelar = output<void>();
  salvo    = output<void>();

  editando = computed(() => !!this.produto());
  salvando = this.store.selectSignal(selectSalvando);

  form!: FormGroup;

  ngOnInit(): void {
    const p = this.produto();
    this.form = this.fb.group({
      nome:      [p?.nome ?? '',  [Validators.required, Validators.minLength(3), Validators.maxLength(100)]],
      preco:     [p?.preco ?? null, [Validators.required, precoValido()]],
      categoria: [p?.categoria ?? '', Validators.required],
      descricao: [p?.descricao ?? '', Validators.maxLength(500)],
      ativo:     [p?.ativo ?? true],
    });
  }

  campoInvalido(campo: string): boolean {
    const ctrl = this.form.get(campo);
    return !!(ctrl?.invalid && (ctrl.dirty || ctrl.touched));
  }

  onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    const dados = this.form.getRawValue();
    const produto = this.produto();

    if (produto) {
      this.store.dispatch(ProdutosActions.atualizarProduto({ produto: { ...dados, id: produto.id } }));
    } else {
      this.store.dispatch(ProdutosActions.criarProduto({ produto: dados }));
    }
    this.salvo.emit();
  }
}
```

---

### Validadores Customizados

```typescript
// Formulário de redefinição de senha com validação cruzada
import { Component, inject } from '@angular/core';
import { FormBuilder, Validators, ReactiveFormsModule } from '@angular/forms';
import { senhasIguais } from '../../../core/validators/validadores';

@Component({
  selector: 'app-redefinir-senha',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()" class="formulario" novalidate>
      <div class="campo">
        <label for="senhaAtual">Senha atual *</label>
        <input id="senhaAtual" type="password" formControlName="senhaAtual" class="input"
               [class.input--erro]="campoInvalido('senhaAtual')" autocomplete="current-password" />
        @if (campoInvalido('senhaAtual')) {
          <span class="campo__erro" role="alert">Informe a senha atual.</span>
        }
      </div>

      <div class="campo">
        <label for="novaSenha">Nova senha *</label>
        <input id="novaSenha" type="password" formControlName="novaSenha" class="input"
               [class.input--erro]="campoInvalido('novaSenha')" autocomplete="new-password" />
        @if (campoInvalido('novaSenha')) {
          <span class="campo__erro" role="alert">
            @if (form.get('novaSenha')?.hasError('required')) { Campo obrigatório. }
            @else if (form.get('novaSenha')?.hasError('minlength')) { Mínimo 8 caracteres. }
          </span>
        }
      </div>

      <div class="campo">
        <label for="confirmarSenha">Confirmar nova senha *</label>
        <input id="confirmarSenha" type="password" formControlName="confirmarSenha" class="input"
               [class.input--erro]="campoInvalido('confirmarSenha') || form.hasError('senhasNaoConferem')"
               autocomplete="new-password" />
        @if (form.hasError('senhasNaoConferem') && form.get('confirmarSenha')?.touched) {
          <span class="campo__erro" role="alert">As senhas não coincidem.</span>
        }
      </div>

      <button type="submit" class="btn btn--primario btn--bloco" [disabled]="form.invalid">
        Redefinir senha
      </button>
    </form>
  `,
})
export class RedefinirSenhaComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group(
    {
      senhaAtual:     ['', [Validators.required]],
      novaSenha:      ['', [Validators.required, Validators.minLength(8)]],
      confirmarSenha: ['', Validators.required],
    },
    { validators: senhasIguais('novaSenha', 'confirmarSenha') }
  );

  campoInvalido(campo: string): boolean {
    const ctrl = this.form.get(campo);
    return !!(ctrl?.invalid && (ctrl.dirty || ctrl.touched));
  }

  onSubmit(): void {
    if (this.form.invalid) { this.form.markAllAsTouched(); return; }
    // chamar serviço
  }
}
```

---

### Formulários Template-Driven

```typescript
// src/app/features/auth/login/login.component.ts
import { Component, inject } from '@angular/core';
import { FormsModule, NgForm } from '@angular/forms';
import { Router } from '@angular/router';
import { AuthService } from '../../../core/services/auth.service';
import { TranslatePipe } from '@ngx-translate/core';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [FormsModule, TranslatePipe],
  template: `
    <div class="pagina-login">
      <div class="card-login">
        <h1>{{ 'nav.entrar' | translate }}</h1>

        @if (erroLogin) {
          <div class="alerta alerta--erro" role="alert">{{ erroLogin }}</div>
        }

        <form #loginForm="ngForm" (ngSubmit)="onSubmit(loginForm)" class="formulario" novalidate>
          <div class="campo">
            <label for="email">E-mail *</label>
            <input
              id="email"
              type="email"
              name="email"
              ngModel
              required
              email
              #emailCtrl="ngModel"
              class="input"
              [class.input--erro]="emailCtrl.invalid && emailCtrl.touched"
              autocomplete="email"
            />
            @if (emailCtrl.invalid && emailCtrl.touched) {
              <span class="campo__erro" role="alert">
                @if (emailCtrl.hasError('required')) { Campo obrigatório. }
                @else if (emailCtrl.hasError('email')) { E-mail inválido. }
              </span>
            }
          </div>

          <div class="campo">
            <label for="senha">Senha *</label>
            <input
              id="senha"
              type="password"
              name="senha"
              ngModel
              required
              minlength="6"
              #senhaCtrl="ngModel"
              class="input"
              [class.input--erro]="senhaCtrl.invalid && senhaCtrl.touched"
              autocomplete="current-password"
            />
            @if (senhaCtrl.invalid && senhaCtrl.touched) {
              <span class="campo__erro" role="alert">
                @if (senhaCtrl.hasError('required')) { Campo obrigatório. }
                @else if (senhaCtrl.hasError('minlength')) { Mínimo 6 caracteres. }
              </span>
            }
          </div>

          <button
            type="submit"
            class="btn btn--primario btn--bloco"
            [disabled]="carregando"
          >
            {{ carregando ? 'Entrando...' : ('nav.entrar' | translate) }}
          </button>
        </form>
      </div>
    </div>
  `,
})
export class LoginComponent {
  private auth   = inject(AuthService);
  private router = inject(Router);

  erroLogin = '';
  carregando = false;

  async onSubmit(form: NgForm): Promise<void> {
    if (form.invalid) { form.control.markAllAsTouched(); return; }

    this.carregando = true;
    this.erroLogin  = '';

    try {
      const { email, senha } = form.value;
      await this.auth.entrar(email, senha);
      const returnUrl = new URLSearchParams(window.location.search).get('returnUrl') ?? '/';
      this.router.navigateByUrl(returnUrl);
    } catch {
      this.erroLogin = 'E-mail ou senha inválidos.';
    } finally {
      this.carregando = false;
    }
  }
}
```

---

## 6. Internacionalização com @ngx-translate

```json
// src/assets/i18n/pt-BR.json
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
      "confirmarExclusao": "Deseja excluir o produto \"{{ nome }}\"?"
    }
  },
  "paginacao": {
    "anterior": "Anterior",
    "proxima": "Próxima",
    "pagina": "Página {{ pagina }} de {{ total }}"
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
// src/assets/i18n/en-US.json
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
      "confirmarExclusao": "Do you want to delete the product \"{{ nome }}\"?"
    }
  },
  "paginacao": {
    "anterior": "Previous",
    "proxima": "Next",
    "pagina": "Page {{ pagina }} of {{ total }}"
  },
  "erros": {
    "generico": "An error occurred. Please try again.",
    "naoAutorizado": "You are not authorized to access this resource.",
    "naoEncontrado": "Page not found.",
    "sessaoExpirada": "Your session has expired. Please sign in again."
  }
}
```

```typescript
// Usando ngx-translate em componentes e serviços
import { Component, inject, signal } from '@angular/core';
import { TranslatePipe, TranslateService } from '@ngx-translate/core';

@Component({
  selector: 'app-seletor-idioma',
  standalone: true,
  imports: [TranslatePipe],
  template: `
    <div class="seletor-idioma" role="navigation" aria-label="Seleção de idioma">
      @for (idioma of idiomas; track idioma.codigo) {
        <button
          class="btn btn--idioma"
          [class.btn--idioma-ativo]="idiomaAtual() === idioma.codigo"
          (click)="mudar(idioma.codigo)"
          [attr.lang]="idioma.codigo"
        >
          {{ idioma.rotulo }}
        </button>
      }
    </div>
  `,
})
export class SeletorIdiomaComponent {
  private translate = inject(TranslateService);

  idiomas = [
    { codigo: 'pt-BR', rotulo: 'PT' },
    { codigo: 'en-US', rotulo: 'EN' },
    { codigo: 'es-ES', rotulo: 'ES' },
  ];

  idiomaAtual = signal(this.translate.currentLang);

  mudar(codigo: string): void {
    this.translate.use(codigo);
    this.idiomaAtual.set(codigo);
    localStorage.setItem('idioma', codigo);
  }
}

// Tradução em serviços (fora de templates)
@Injectable({ providedIn: 'root' })
export class ToastService {
  private translate = inject(TranslateService);

  mostrarErroPadrao(): void {
    const msg = this.translate.instant('erros.generico');
    this.mostrar(msg, 'erro');
  }

  // Interpolação com variáveis
  confirmarExclusao(nome: string): void {
    const msg = this.translate.instant('produto.mensagens.confirmarExclusao', { nome });
    console.log(msg); // "Deseja excluir o produto "Mouse Gamer"?"
  }

  mostrar(mensagem: string, tipo: 'sucesso' | 'erro' | 'info'): void {
    // implementação de toast
  }
}

// Pipe para tradução com interpolação no template
// Uso: {{ 'produto.mensagens.confirmarExclusao' | translate:{ nome: produto.nome } }}
// Ou:  <p [innerHTML]="'chave.html' | translate"></p>
```

---

## 7. Manipulação de Datas com date-fns

```typescript
// src/app/core/utils/datas.util.ts
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
} from 'date-fns';
import { ptBR, enUS, es } from 'date-fns/locale';

const localeMap: Record<string, Locale> = {
  'pt-BR': ptBR,
  'en-US': enUS,
  'es-ES': es,
};

export function getLocale(idioma: string): Locale {
  return localeMap[idioma] ?? ptBR;
}

export function formatarData(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data;
  if (!isValid(d)) return '—';
  const locale = getLocale(idioma);
  const padrao = idioma === 'en-US' ? 'MM/dd/yyyy' : 'dd/MM/yyyy';
  return format(d, padrao, { locale });
}

export function formatarDataHora(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data;
  if (!isValid(d)) return '—';
  const locale = getLocale(idioma);
  const padrao = idioma === 'en-US' ? 'MM/dd/yyyy hh:mm a' : 'dd/MM/yyyy HH:mm';
  return format(d, padrao, { locale });
}

export function formatarRelativo(data: Date | string, idioma = 'pt-BR'): string {
  const d = typeof data === 'string' ? parseISO(data) : data;
  if (!isValid(d)) return '—';
  return formatDistanceToNow(d, { addSuffix: true, locale: getLocale(idioma) });
}

export const datasUtil = {
  prazoVencido: (data: Date | string): boolean => {
    const d = typeof data === 'string' ? parseISO(data) : data;
    return isBefore(d, startOfDay(new Date()));
  },
  diasRestantes: (data: Date | string): number => {
    const d = typeof data === 'string' ? parseISO(data) : data;
    return differenceInDays(d, new Date());
  },
  intervaloMes: (ano: number, mes: number) => ({
    inicio: startOfMonth(new Date(ano, mes)),
    fim: endOfMonth(new Date(ano, mes)),
  }),
  diasDoMes: (ano: number, mes: number): Date[] => {
    const { inicio, fim } = datasUtil.intervaloMes(ano, mes);
    return eachDayOfInterval({ start: inicio, end: fim });
  },
};
```

```typescript
// Pipes de data integrados com i18n
import { Pipe, PipeTransform, inject } from '@angular/core';
import { TranslateService } from '@ngx-translate/core';
import { formatarData, formatarDataHora, formatarRelativo } from '../utils/datas.util';
import { parseISO, isToday, isTomorrow, isYesterday } from 'date-fns';

@Pipe({ name: 'dataFormatada', standalone: true, pure: false })
export class DataFormatadaPipe implements PipeTransform {
  private translate = inject(TranslateService);

  transform(valor: string | Date | null | undefined, formato: 'data' | 'dataHora' | 'relativo' = 'data'): string {
    if (!valor) return '—';
    const idioma = this.translate.currentLang;

    switch (formato) {
      case 'dataHora': return formatarDataHora(valor, idioma);
      case 'relativo': return formatarRelativo(valor, idioma);
      default:         return formatarData(valor, idioma);
    }
  }
}

@Pipe({ name: 'prazoEntrega', standalone: true, pure: false })
export class PrazoEntregaPipe implements PipeTransform {
  transform(valor: string | Date | null | undefined): string {
    if (!valor) return '—';
    const data = typeof valor === 'string' ? parseISO(valor) : valor;
    if (isToday(data))     return 'Hoje';
    if (isTomorrow(data))  return 'Amanhã';
    if (isYesterday(data)) return 'Ontem (vencido)';
    return formatarData(data);
  }
}

// Serviço de datas com acesso ao idioma atual
@Injectable({ providedIn: 'root' })
export class DatasService {
  private translate = inject(TranslateService);

  get idioma(): string { return this.translate.currentLang ?? 'pt-BR'; }

  formatar(data: Date | string): string     { return formatarData(data, this.idioma); }
  formatarHora(data: Date | string): string { return formatarDataHora(data, this.idioma); }
  relativo(data: Date | string): string     { return formatarRelativo(data, this.idioma); }
}

// Uso nos templates
// <time [attr.datetime]="produto.criadoEm">{{ produto.criadoEm | dataFormatada }}</time>
// <time>{{ produto.criadoEm | dataFormatada:'relativo' }}</time>
// <span [class.prazo--vencido]="datasUtil.prazoVencido(pedido.prazo)">
//   {{ pedido.prazo | prazoEntrega }}
// </span>
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
// src/app/core/models/auth.model.ts
export type Role = 'USER' | 'GERENTE' | 'ADMIN';

export type UsuarioJWT = {
  sub: string;
  nome: string;
  roles: Role[];
  iat: number;
  exp: number;
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

export type CredenciaisLogin = {
  email: string;
  senha: string;
};
```

---

### Serviço de Autenticação

```typescript
// src/app/core/services/auth.service.ts
import { Injectable, inject, signal, computed } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { firstValueFrom } from 'rxjs';
import { jwtDecode } from 'jwt-decode';
import { environment } from '../../../environments/environment';
import {
  Usuario,
  UsuarioJWT,
  RespostaLogin,
  CredenciaisLogin,
  Role,
} from '../models/auth.model';

const TOKEN_KEY   = 'auth_token';
const REFRESH_KEY = 'refresh_token';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private http   = inject(HttpClient);
  private router = inject(Router);

  private _usuario = signal<Usuario | null>(null);
  private _token   = signal<string | null>(null);

  // API pública somente leitura
  readonly usuario    = this._usuario.asReadonly();
  readonly token      = this._token.asReadonly();
  readonly autenticado = computed(() => !!this._usuario());
  readonly nomeUsuario = computed(() => this._usuario()?.nome ?? '');

  constructor() {
    this.restaurarSessao();
  }

  private restaurarSessao(): void {
    const token = localStorage.getItem(TOKEN_KEY);
    if (token && this.tokenValido(token)) {
      this._token.set(token);
      this._usuario.set(this.extrairUsuario(token));
    } else {
      this.limparStorage();
    }
  }

  private tokenValido(token: string): boolean {
    try {
      const { exp } = jwtDecode<UsuarioJWT>(token);
      return Date.now() < exp * 1000;
    } catch {
      return false;
    }
  }

  private extrairUsuario(token: string): Usuario | null {
    try {
      const { sub, nome, roles } = jwtDecode<UsuarioJWT>(token);
      return { email: sub, nome, roles };
    } catch {
      return null;
    }
  }

  private limparStorage(): void {
    localStorage.removeItem(TOKEN_KEY);
    localStorage.removeItem(REFRESH_KEY);
  }

  async entrar(email: string, senha: string): Promise<void> {
    const resposta = await firstValueFrom(
      this.http.post<RespostaLogin>(`${environment.apiUrl}/auth/login`, { email, senha })
    );
    localStorage.setItem(TOKEN_KEY, resposta.token);
    if (resposta.refreshToken) {
      localStorage.setItem(REFRESH_KEY, resposta.refreshToken);
    }
    this._token.set(resposta.token);
    this._usuario.set(this.extrairUsuario(resposta.token));
  }

  sair(): void {
    this.limparStorage();
    this._token.set(null);
    this._usuario.set(null);
    this.router.navigate(['/login']);
  }

  temRole(role: Role | string): boolean {
    return this._usuario()?.roles.includes(role as Role) ?? false;
  }

  temAlgumaRole(roles: string[]): boolean {
    return roles.some((r) => this.temRole(r));
  }

  temTodasAsRoles(roles: string[]): boolean {
    return roles.every((r) => this.temRole(r));
  }

  getToken(): string | null {
    return localStorage.getItem(TOKEN_KEY);
  }

  async renovarToken(): Promise<string> {
    const refreshToken = localStorage.getItem(REFRESH_KEY);
    if (!refreshToken) {
      this.sair();
      throw new Error('Sem refresh token');
    }

    const resposta = await firstValueFrom(
      this.http.post<RespostaLogin>(`${environment.apiUrl}/auth/refresh`, { refreshToken })
    );

    localStorage.setItem(TOKEN_KEY, resposta.token);
    this._token.set(resposta.token);
    this._usuario.set(this.extrairUsuario(resposta.token));
    return resposta.token;
  }
}
```

---

### Interceptor HTTP Funcional

```typescript
// src/app/core/interceptors/auth.interceptor.ts
import { HttpInterceptorFn, HttpRequest, HttpHandlerFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, switchMap, throwError, BehaviorSubject, filter, take } from 'rxjs';
import { AuthService } from '../services/auth.service';

let renovando = false;
const tokenAtualizado$ = new BehaviorSubject<string | null>(null);

function adicionarToken(req: HttpRequest<unknown>, token: string): HttpRequest<unknown> {
  return req.clone({
    setHeaders: { Authorization: `Bearer ${token}` },
  });
}

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);

  // Não intercepta chamadas de login/refresh
  if (req.url.includes('/auth/')) return next(req);

  const token = auth.getToken();
  const reqComToken = token ? adicionarToken(req, token) : req;

  return next(reqComToken).pipe(
    catchError((erro: HttpErrorResponse) => {
      if (erro.status !== 401) return throwError(() => erro);

      // Evita múltiplas chamadas de refresh simultâneas
      if (renovando) {
        return tokenAtualizado$.pipe(
          filter((t) => t !== null),
          take(1),
          switchMap((novoToken) => next(adicionarToken(req, novoToken!)))
        );
      }

      renovando = true;
      tokenAtualizado$.next(null);

      return new Promise<string>((res, rej) =>
        auth.renovarToken().then(res).catch(rej)
      ).then((novoToken) => {
        renovando = false;
        tokenAtualizado$.next(novoToken);
        return next(adicionarToken(req, novoToken));
      }).catch((erroRefresh) => {
        renovando = false;
        auth.sair();
        return throwError(() => erroRefresh);
      }) as any;
    })
  );
};
```

---

### Guards de Autenticação e Role

```typescript
// Já definidos na seção 2 — Guards Funcionais
// Recapitulando o uso integrado:

// src/app/app.routes.ts
{
  path: 'painel',
  canActivate: [authGuard],             // exige login
  children: [
    { path: 'perfil', loadComponent: () => import('./features/perfil/perfil.component').then(m => m.PerfilComponent) },
    { path: 'admin', canActivate: [roleGuard(['ADMIN'])], loadComponent: () => import('./features/admin/admin.component').then(m => m.AdminComponent) },
    { path: 'relatorios', canActivate: [roleGuard(['ADMIN', 'GERENTE'])], loadComponent: () => import('./features/relatorios/relatorios.component').then(m => m.RelatoriosComponent) },
  ],
},
```

---

### Diretiva de Acesso por Role

```typescript
// src/app/shared/directives/acesso-restrito.directive.ts
import { Directive, inject, input, OnInit, TemplateRef, ViewContainerRef, effect } from '@angular/core';
import { AuthService } from '../../core/services/auth.service';

/**
 * Exibe o elemento apenas se o usuário tiver a role exigida.
 *
 * Uso:
 *   <button *appAcessoRestrito="'ADMIN'">Excluir</button>
 *   <div *appAcessoRestrito="['ADMIN', 'GERENTE']; else semAcesso">Conteúdo</div>
 *   <ng-template #semAcesso><p>Sem permissão</p></ng-template>
 */
@Directive({
  selector: '[appAcessoRestrito]',
  standalone: true,
})
export class AcessoRestritoDirective implements OnInit {
  private templateRef = inject(TemplateRef<unknown>);
  private vcr         = inject(ViewContainerRef);
  private auth        = inject(AuthService);

  appAcessoRestrito     = input<string | string[]>('');
  appAcessoRestritoElse = input<TemplateRef<unknown> | null>(null);

  ngOnInit(): void {
    this.atualizar();
  }

  private atualizar(): void {
    const roles = this.appAcessoRestrito();
    const rolesArray = Array.isArray(roles) ? roles : [roles];
    const temPermissao = this.auth.temAlgumaRole(rolesArray);

    this.vcr.clear();
    if (temPermissao) {
      this.vcr.createEmbeddedView(this.templateRef);
    } else {
      const elseTpl = this.appAcessoRestritoElse();
      if (elseTpl) this.vcr.createEmbeddedView(elseTpl);
    }
  }
}
```

```typescript
// src/app/shared/components/acesso-restrito/acesso-restrito.component.ts
// Alternativa como componente com ng-content
import { Component, inject, input, computed } from '@angular/core';
import { AuthService } from '../../../core/services/auth.service';

@Component({
  selector: 'app-acesso-restrito',
  standalone: true,
  template: `
    @if (temPermissao()) {
      <ng-content />
    } @else {
      <ng-content select="[slot=sem-acesso]" />
    }
  `,
})
export class AcessoRestritoComponent {
  private auth = inject(AuthService);

  roles        = input<string[]>([]);       // Qualquer uma dessas roles
  todasAsRoles = input<string[]>([]);       // Todas essas roles

  temPermissao = computed(() => {
    const r  = this.roles();
    const tr = this.todasAsRoles();

    if (r.length > 0 && !this.auth.temAlgumaRole(r)) return false;
    if (tr.length > 0 && !this.auth.temTodasAsRoles(tr)) return false;
    if (r.length === 0 && tr.length === 0) return this.auth.autenticado();
    return true;
  });
}
```

```typescript
// Usando nos componentes
@Component({
  standalone: true,
  imports: [AcessoRestritoDirective, AcessoRestritoComponent, RouterLink, TranslatePipe],
  template: `
    <nav>
      <!-- Diretiva estrutural -->
      <a routerLink="/" class="nav-link">Início</a>

      <ng-template #semAcesso>
        <span class="nav-link nav-link--desabilitado">Admin (sem acesso)</span>
      </ng-template>

      <a *appAcessoRestrito="'ADMIN'; else semAcesso" routerLink="/painel/admin" class="nav-link">
        Admin
      </a>

      <a *appAcessoRestrito="['ADMIN', 'GERENTE']" routerLink="/painel/relatorios" class="nav-link">
        Relatórios
      </a>
    </nav>

    <!-- Tabela com ações condicionais -->
    <table>
      <tbody>
        @for (produto of produtos(); track produto.id) {
          <tr>
            <td>{{ produto.nome }}</td>
            <td>
              <!-- Componente de acesso restrito -->
              <app-acesso-restrito [roles]="['ADMIN', 'GERENTE']">
                <button class="btn btn--secundario btn--pequeno">Editar</button>
              </app-acesso-restrito>

              <app-acesso-restrito [roles]="['ADMIN']">
                <button class="btn btn--perigo btn--pequeno">Excluir</button>
                <div slot="sem-acesso"><!-- vazio para ADMIN --></div>
              </app-acesso-restrito>
            </td>
          </tr>
        }
      </tbody>
    </table>

    <!-- Usando signals do AuthService diretamente -->
    @if (auth.temRole('ADMIN')) {
      <section class="painel-admin">
        <h2>Painel Administrativo</h2>
      </section>
    }
  `,
})
export class PaginaAdminComponent {
  auth     = inject(AuthService);
  produtos = signal<Produto[]>([]);
}
```

---

## CSS — Estrutura sem Biblioteca de Estilos

```css
/* src/styles.css */

*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --cor-primaria:        #1d4ed8;
  --cor-primaria-hover:  #1e40af;
  --cor-secundaria:      #6b7280;
  --cor-perigo:          #dc2626;
  --cor-perigo-hover:    #b91c1c;
  --cor-sucesso:         #16a34a;
  --cor-aviso:           #d97706;
  --cor-info:            #0284c7;

  --cor-texto:           #111827;
  --cor-texto-fraco:     #6b7280;
  --cor-fundo:           #f9fafb;
  --cor-fundo-card:      #ffffff;
  --cor-borda:           #e5e7eb;

  --raio-borda:          6px;
  --sombra-card:         0 1px 3px rgba(0,0,0,.10), 0 1px 2px rgba(0,0,0,.06);
  --fonte-base:          'Inter', system-ui, -apple-system, sans-serif;
  --espaco:              1rem;
}

body {
  font-family: var(--fonte-base);
  font-size: 1rem;
  color: var(--cor-texto);
  background-color: var(--cor-fundo);
  line-height: 1.6;
}

/* Layout */
.app                   { display: flex; flex-direction: column; min-height: 100vh; }
.conteudo-principal    { flex: 1; max-width: 1200px; width: 100%; margin: 0 auto; padding: 2rem 1rem; }

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

.nav-principal         { display: flex; gap: .5rem; align-items: center; }

.nav-link {
  padding: .5rem .75rem;
  border-radius: var(--raio-borda);
  text-decoration: none;
  color: var(--cor-texto-fraco);
  font-weight: 500;
  font-size: .9375rem;
  transition: color .15s, background-color .15s;
}
.nav-link:hover        { color: var(--cor-primaria); background: #eff6ff; }
.nav-link--ativo       { color: var(--cor-primaria); background: #eff6ff; }
.nav-link--desabilitado{ color: var(--cor-borda); cursor: not-allowed; }

.cabecalho__acoes      { display: flex; align-items: center; gap: .75rem; }
.usuario-nome          { font-size: .875rem; color: var(--cor-texto-fraco); }

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
.btn:disabled          { opacity: .5; cursor: not-allowed; }
.btn--bloco            { width: 100%; }
.btn--pequeno          { padding: .25rem .625rem; font-size: .75rem; }

.btn--primario         { background: var(--cor-primaria); color: #fff; }
.btn--primario:hover:not(:disabled) { background: var(--cor-primaria-hover); }

.btn--secundario       { background: transparent; color: var(--cor-secundaria); border: 1px solid var(--cor-borda); }
.btn--secundario:hover:not(:disabled) { background: var(--cor-fundo); }

.btn--perigo           { background: var(--cor-perigo); color: #fff; }
.btn--perigo:hover:not(:disabled) { background: var(--cor-perigo-hover); }

.btn--idioma           { background: transparent; color: var(--cor-texto-fraco); border: 1px solid var(--cor-borda); padding: .25rem .5rem; font-size: .75rem; }
.btn--idioma-ativo     { background: var(--cor-primaria); color: #fff; border-color: var(--cor-primaria); }

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
.card-produto__titulo  { font-size: 1.125rem; font-weight: 600; }
.card-produto__preco   { font-size: 1.25rem; font-weight: 700; color: var(--cor-primaria); }
.card-produto__acoes   { display: flex; gap: .5rem; margin-top: .5rem; }

/* Lista de produtos */
.lista-produtos        { list-style: none; display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 1rem; }

/* Formulários */
.formulario            { display: flex; flex-direction: column; gap: 1.25rem; max-width: 480px; }
.campo                 { display: flex; flex-direction: column; gap: .375rem; }
.campo label           { font-size: .875rem; font-weight: 500; }
.campo__erro           { font-size: .75rem; color: var(--cor-perigo); }
.campo__dica           { font-size: .75rem; color: var(--cor-texto-fraco); text-align: right; }
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
.input--erro, .select--erro { border-color: var(--cor-perigo); }
.input--erro:focus, .select--erro:focus { box-shadow: 0 0 0 3px rgba(220,38,38,.15); }

.formulario__acoes     { display: flex; gap: .75rem; }

/* Card login */
.pagina-login          { min-height: calc(100vh - 64px); display: flex; align-items: center; justify-content: center; }
.card-login            { background: var(--cor-fundo-card); border: 1px solid var(--cor-borda); border-radius: var(--raio-borda); padding: 2rem; box-shadow: var(--sombra-card); width: 100%; max-width: 400px; }
.card-login h1         { margin-bottom: 1.5rem; font-size: 1.5rem; }

/* Alertas */
.alerta                { padding: .875rem 1rem; border-radius: var(--raio-borda); font-size: .875rem; }
.alerta--erro          { background: #fee2e2; color: #991b1b; border: 1px solid #fca5a5; }
.alerta--sucesso       { background: #dcfce7; color: #166534; border: 1px solid #86efac; }
.alerta--info          { background: #e0f2fe; color: #075985; border: 1px solid #7dd3fc; }
.alerta--aviso         { background: #fef3c7; color: #92400e; border: 1px solid #fcd34d; }

/* Badges */
.badge                 { display: inline-block; padding: .125rem .5rem; border-radius: 9999px; font-size: .75rem; font-weight: 600; }
.badge--sucesso        { background: #dcfce7; color: #166534; }
.badge--erro           { background: #fee2e2; color: #991b1b; }
.badge--info           { background: #e0f2fe; color: #075985; }
.badge--aviso          { background: #fef3c7; color: #92400e; }

/* Tabelas */
table                  { width: 100%; border-collapse: collapse; }
thead                  { background: var(--cor-fundo); }
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
td                     { padding: .875rem 1rem; border-bottom: 1px solid var(--cor-borda); font-size: .875rem; }
tr:hover td            { background: var(--cor-fundo); }

/* Paginação */
.paginacao             { display: flex; align-items: center; gap: .5rem; margin-top: 1rem; justify-content: center; }

/* Seletor de idioma */
.seletor-idioma        { display: flex; gap: .25rem; }

/* Prazo */
.prazo--vencido        { color: var(--cor-perigo); font-weight: 600; }
.prazo--hoje           { color: var(--cor-aviso); font-weight: 600; }
.prazo--amanha         { color: var(--cor-sucesso); }

/* Estado vazio */
.estado-vazio          { text-align: center; padding: 3rem 1rem; color: var(--cor-texto-fraco); }

/* Carregando */
.carregando-pagina     { display: flex; align-items: center; justify-content: center; min-height: 200px; }
.spinner {
  width: 36px; height: 36px;
  border: 3px solid var(--cor-borda);
  border-top-color: var(--cor-primaria);
  border-radius: 50%;
  animation: girar .7s linear infinite;
}
@keyframes girar { to { transform: rotate(360deg); } }

/* Rodapé */
.rodape                { border-top: 1px solid var(--cor-borda); padding: 1rem; text-align: center; font-size: .875rem; color: var(--cor-texto-fraco); }

/* Responsivo */
@media (max-width: 768px) {
  .conteudo-principal  { padding: 1rem; }
  .cabecalho           { height: 56px; padding: 0 .75rem; }
  .nav-link            { padding: .375rem .5rem; font-size: .875rem; }
  .lista-produtos      { grid-template-columns: 1fr; }
  .formulario          { max-width: 100%; }
  table                { font-size: .8125rem; }
  th, td               { padding: .625rem .75rem; }
  .formulario__acoes   { flex-direction: column; }
}

@media (max-width: 480px) {
  .cabecalho           { flex-wrap: wrap; height: auto; padding: .5rem; gap: .5rem; }
  .nav-principal       { flex-wrap: wrap; }
}
```

---

## Referência Rápida de Versões

| Biblioteca | Versão | Finalidade |
|---|---|---|
| `@angular/core` | 21.x | Framework principal |
| `@angular/router` | 21.x | Roteamento |
| `@angular/forms` | 21.x | Formulários reativos e template-driven |
| `@angular/common/http` | 21.x | HttpClient |
| `@ngrx/store` | 19.x | Estado global (Store) |
| `@ngrx/effects` | 19.x | Efeitos colaterais assíncronos |
| `@ngrx/signals` | 19.x | SignalStore |
| `@ngrx/entity` | 19.x | Coleções normalizadas |
| `@ngrx/operators` | 19.x | tapResponse e outros operadores |
| `@ngx-translate/core` | 16.x | Internacionalização |
| `@ngx-translate/http-loader` | 16.x | Carregador de traduções via HTTP |
| `date-fns` | 4.x | Manipulação de datas |
| `jwt-decode` | 4.x | Decodificação de JWT |

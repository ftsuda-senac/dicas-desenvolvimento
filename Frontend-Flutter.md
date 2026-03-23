# Flutter 3 — Referência Completa

Referência consolidada sobre Flutter 3 com Dart, cobrindo fundamentos, navegação, gerenciamento de estado global, integração com backend, telas de lista e formulários, internacionalização, autenticação JWT com controle de acesso por role e distribuição para Android e iOS.

---

## Sumário

1. [Fundamentos do Flutter](#1-fundamentos-do-flutter)
   - [Configuração do Projeto](#configuração-do-projeto)
   - [Estrutura de Pastas](#estrutura-de-pastas)
   - [Widgets Essenciais](#widgets-essenciais)
   - [Fundamentos do Dart](#fundamentos-do-dart)
2. [Navegação com GoRouter](#2-navegação-com-gorouter)
   - [Configuração do GoRouter](#configuração-do-gorouter)
   - [Rotas Aninhadas e Parâmetros](#rotas-aninhadas-e-parâmetros)
   - [Rotas Protegidas](#rotas-protegidas)
   - [Navegação Programática](#navegação-programática)
3. [Estado Global com Riverpod](#3-estado-global-com-riverpod)
   - [Conceitos Básicos](#conceitos-básicos)
   - [Providers Principais](#providers-principais)
   - [AsyncNotifier para Estado Assíncrono](#asyncnotifier-para-estado-assíncrono)
4. [Integração com Backend](#4-integração-com-backend)
   - [Configuração do Dio](#configuração-do-dio)
   - [Interceptors e Tratamento de Erros](#interceptors-e-tratamento-de-erros)
   - [Repository Pattern](#repository-pattern)
5. [Telas de Lista e Formulários](#5-telas-de-lista-e-formulários)
   - [Listas com ListView e Paginação](#listas-com-listview-e-paginação)
   - [Formulários e Validação](#formulários-e-validação)
   - [Feedback Visual ao Usuário](#feedback-visual-ao-usuário)
6. [Internacionalização (i18n)](#6-internacionalização-i18n)
   - [Configuração do flutter_localizations](#configuração-do-flutter_localizations)
   - [Mensagens de Validação Localizadas](#mensagens-de-validação-localizadas)
7. [Autenticação JWT e Controle de Acesso por Role](#7-autenticação-jwt-e-controle-de-acesso-por-role)
   - [Armazenamento Seguro do Token](#armazenamento-seguro-do-token)
   - [Decodificação e Roles do JWT](#decodificação-e-roles-do-jwt)
   - [Provider de Autenticação](#provider-de-autenticação)
   - [Guard de Rotas por Role](#guard-de-rotas-por-role)
8. [Compilação e Distribuição](#8-compilação-e-distribuição)
   - [Build para Android](#build-para-android)
   - [Publicação na Google Play Store](#publicação-na-google-play-store)
   - [Build para iOS](#build-para-ios)
   - [Publicação na Apple App Store](#publicação-na-apple-app-store)

---

## 1. Fundamentos do Flutter

### Configuração do Projeto

Flutter exige a instalação do Flutter SDK e, para iOS, do Xcode (macOS obrigatório).

```bash
# Verificar instalação
flutter doctor

# Criar novo projeto
flutter create meu_app
cd meu_app

# Instalar dependências deste guia
flutter pub add go_router
flutter pub add flutter_riverpod riverpod_annotation
flutter pub add dev:riverpod_generator dev:build_runner

flutter pub add dio
flutter pub add flutter_secure_storage
flutter pub add jwt_decoder

flutter pub add intl
# flutter_localizations faz parte do SDK, não precisa de pub add separado

# Rodar o app
flutter run
```

> **Dica:** use `flutter pub get` após editar `pubspec.yaml` manualmente. Use `dart run build_runner watch` em paralelo ao desenvolvimento para regenerar código do Riverpod automaticamente.

---

### Estrutura de Pastas

```
lib/
├── main.dart
├── app.dart                  # MaterialApp + GoRouter + ProviderScope
├── core/
│   ├── constants/
│   ├── errors/
│   ├── extensions/
│   ├── network/
│   │   ├── dio_client.dart
│   │   └── api_interceptor.dart
│   └── storage/
│       └── secure_storage.dart
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── auth_repository.dart
│   │   │   └── auth_repository.g.dart
│   │   ├── domain/
│   │   │   ├── auth_state.dart
│   │   │   └── user_model.dart
│   │   └── presentation/
│   │       ├── login_screen.dart
│   │       └── auth_provider.dart
│   ├── produtos/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── ...
├── l10n/
│   ├── app_en.arb
│   └── app_pt.arb
└── routes/
    └── app_router.dart
```

---

### Widgets Essenciais

No Flutter tudo é widget. Os dois tipos fundamentais são:

- **StatelessWidget** — imutável, redesenhado somente quando o pai muda.
- **StatefulWidget** — possui estado interno que pode ser atualizado com `setState`.

Com Riverpod, a maioria das telas usa `ConsumerWidget` (equivalente a `StatelessWidget`) ou `ConsumerStatefulWidget`.

```dart
// lib/features/produtos/presentation/card_produto.dart
import 'package:flutter/material.dart';
import '../domain/produto_model.dart';

class CardProduto extends StatelessWidget {
  final Produto produto;
  final VoidCallback onTap;

  const CardProduto({super.key, required this.produto, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: ListTile(
        leading: produto.imagemUrl != null
            ? Image.network(produto.imagemUrl!, width: 56, fit: BoxFit.cover)
            : const Icon(Icons.inventory_2),
        title: Text(produto.nome),
        subtitle: Text('R\$ ${produto.preco.toStringAsFixed(2)}'),
        trailing: produto.disponivel
            ? const Icon(Icons.check_circle, color: Colors.green)
            : const Icon(Icons.cancel, color: Colors.red),
        onTap: onTap,
      ),
    );
  }
}
```

---

### Fundamentos do Dart

```dart
// Tipos básicos e null safety
String nome = 'Flutter';
int versao = 3;
double preco = 19.99;
bool ativo = true;
String? descricao;           // nullable — pode ser null

// Records (Dart 3+)
(String, int) par = ('Flutter', 3);
({String nome, int idade}) pessoa = (nome: 'Ana', idade: 25);

// Sealed classes para estados (Dart 3+)
sealed class ResultadoAuth {}
class Autenticado extends ResultadoAuth { final String token; Autenticado(this.token); }
class NaoAutenticado extends ResultadoAuth {}
class ErroAuth extends ResultadoAuth { final String mensagem; ErroAuth(this.mensagem); }

// Pattern matching com switch expressions (Dart 3+)
String descreveResultado(ResultadoAuth resultado) => switch (resultado) {
  Autenticado(token: var t) => 'Token: $t',
  NaoAutenticado() => 'Não autenticado',
  ErroAuth(mensagem: var m) => 'Erro: $m',
};

// async/await
Future<List<Produto>> buscarProdutos() async {
  final response = await dio.get('/produtos');
  return (response.data as List).map((e) => Produto.fromJson(e)).toList();
}
```

---

## 2. Navegação com GoRouter

GoRouter é a solução de roteamento oficial recomendada pelo Flutter team. Suporta deep linking, navegação declarativa, guards e rotas aninhadas.

### Configuração do GoRouter

```dart
// lib/routes/app_router.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../features/auth/presentation/auth_provider.dart';
import '../features/auth/presentation/login_screen.dart';
import '../features/produtos/presentation/produtos_list_screen.dart';
import '../features/produtos/presentation/produto_detalhe_screen.dart';
import '../features/admin/presentation/admin_screen.dart';

part 'app_router.g.dart';

@riverpod
GoRouter appRouter(AppRouterRef ref) {
  final authState = ref.watch(authStateProvider);

  return GoRouter(
    initialLocation: '/produtos',
    debugLogDiagnostics: true,
    redirect: (context, state) {
      final logado = authState.valueOrNull?.token != null;
      final naLogin = state.matchedLocation == '/login';

      if (!logado && !naLogin) return '/login';
      if (logado && naLogin) return '/produtos';
      return null;
    },
    routes: [
      GoRoute(
        path: '/login',
        name: 'login',
        builder: (context, state) => const LoginScreen(),
      ),
      ShellRoute(
        builder: (context, state, child) => MainScaffold(child: child),
        routes: [
          GoRoute(
            path: '/produtos',
            name: 'produtos',
            builder: (context, state) => const ProdutosListScreen(),
            routes: [
              GoRoute(
                path: ':id',
                name: 'produto-detalhe',
                builder: (context, state) {
                  final id = state.pathParameters['id']!;
                  return ProdutoDetalheScreen(id: id);
                },
              ),
            ],
          ),
          GoRoute(
            path: '/admin',
            name: 'admin',
            builder: (context, state) => const AdminScreen(),
          ),
        ],
      ),
    ],
    errorBuilder: (context, state) => Scaffold(
      body: Center(child: Text('Página não encontrada: ${state.error}')),
    ),
  );
}
```

```dart
// lib/app.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import 'routes/app_router.dart';

class App extends ConsumerWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);

    return MaterialApp.router(
      title: 'Meu App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      routerConfig: router,
      localizationsDelegates: const [
        AppLocalizations.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      supportedLocales: const [Locale('pt', 'BR'), Locale('en')],
    );
  }
}
```

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'app.dart';

void main() {
  runApp(const ProviderScope(child: App()));
}
```

---

### Rotas Aninhadas e Parâmetros

```dart
// Navegação para rota com parâmetro
context.goNamed('produto-detalhe', pathParameters: {'id': produto.id.toString()});

// Navegação com query parameters
context.goNamed('produtos', queryParameters: {'categoria': 'eletronicos'});

// Lendo parâmetros na tela de destino
class ProdutoDetalheScreen extends StatelessWidget {
  final String id;
  const ProdutoDetalheScreen({super.key, required this.id});

  @override
  Widget build(BuildContext context) {
    // id disponível diretamente via construtor
    return Scaffold(appBar: AppBar(title: Text('Produto $id')));
  }
}
```

---

### Rotas Protegidas

O `redirect` no GoRouter (veja seção anterior) já protege rotas globalmente. Para proteção por role, veja a [seção 7](#7-autenticação-jwt-e-controle-de-acesso-por-role).

---

### Navegação Programática

```dart
// Ir para rota (substitui o histórico)
context.go('/produtos');

// Empilhar rota (mantém botão voltar)
context.push('/produtos/42');

// Voltar com resultado
context.pop(resultado);

// Fora de contexto de widget (via GlobalKey ou GoRouter diretamente)
final router = ref.read(appRouterProvider);
router.go('/login');
```

---

## 3. Estado Global com Riverpod

Riverpod 2.x é a solução de gerenciamento de estado mais popular no ecossistema Flutter. É type-safe, testável e não depende do `BuildContext`.

> **Alternativa popular:** **Bloc/Cubit** (`flutter_bloc`) — preferido em equipes que valorizam separação rígida de eventos e estados. Use Riverpod para produtividade e Bloc quando precisar de rastreabilidade explícita de eventos.

### Conceitos Básicos

```dart
// Todo provider deve ser anotado com @riverpod
// O código .g.dart é gerado automaticamente pelo build_runner

import 'package:riverpod_annotation/riverpod_annotation.dart';
part 'exemplos_provider.g.dart';

// Provider simples (valor imutável)
@riverpod
String saudacao(SaudacaoRef ref) => 'Olá, Flutter!';

// Provider que depende de outro
@riverpod
String saudacaoCompleta(SaudacaoCompletaRef ref) {
  final base = ref.watch(saudacaoProvider);
  return '$base Com Riverpod.';
}
```

```dart
// Consumindo providers em um widget
import 'package:flutter_riverpod/flutter_riverpod.dart';

class MinhaTelaWidget extends ConsumerWidget {
  const MinhaTelaWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final saudacao = ref.watch(saudacaoCompletaProvider);
    return Text(saudacao);
  }
}
```

---

### Providers Principais

```dart
// lib/features/carrinho/presentation/carrinho_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../domain/item_carrinho.dart';
part 'carrinho_provider.g.dart';

// Notifier — estado mutável síncrono
@riverpod
class Carrinho extends _$Carrinho {
  @override
  List<ItemCarrinho> build() => [];

  void adicionar(ItemCarrinho item) {
    state = [...state, item];
  }

  void remover(String produtoId) {
    state = state.where((i) => i.produtoId != produtoId).toList();
  }

  double get total => state.fold(0, (soma, item) => soma + item.subtotal);
}
```

```dart
// Consumindo o Notifier
class CarrinhoScreen extends ConsumerWidget {
  const CarrinhoScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final itens = ref.watch(carrinhoProvider);
    final carrinho = ref.read(carrinhoProvider.notifier);

    return ListView.builder(
      itemCount: itens.length,
      itemBuilder: (context, index) {
        final item = itens[index];
        return ListTile(
          title: Text(item.nome),
          trailing: IconButton(
            icon: const Icon(Icons.delete),
            onPressed: () => carrinho.remover(item.produtoId),
          ),
        );
      },
    );
  }
}
```

---

### AsyncNotifier para Estado Assíncrono

```dart
// lib/features/produtos/presentation/produtos_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../data/produto_repository.dart';
import '../domain/produto_model.dart';
part 'produtos_provider.g.dart';

@riverpod
class ProdutosList extends _$ProdutosList {
  @override
  Future<List<Produto>> build() async {
    final repo = ref.watch(produtoRepositoryProvider);
    return repo.listarTodos();
  }

  Future<void> recarregar() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() {
      final repo = ref.read(produtoRepositoryProvider);
      return repo.listarTodos();
    });
  }
}
```

```dart
// Consumindo AsyncValue com tratamento de estados
class ProdutosListScreen extends ConsumerWidget {
  const ProdutosListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final produtosAsync = ref.watch(produtosListProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Produtos')),
      body: produtosAsync.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.error_outline, size: 48, color: Colors.red),
              const SizedBox(height: 8),
              Text('Erro ao carregar produtos'),
              TextButton(
                onPressed: () => ref.invalidate(produtosListProvider),
                child: const Text('Tentar novamente'),
              ),
            ],
          ),
        ),
        data: (produtos) => ListView.builder(
          itemCount: produtos.length,
          itemBuilder: (context, index) => CardProduto(
            produto: produtos[index],
            onTap: () => context.goNamed(
              'produto-detalhe',
              pathParameters: {'id': produtos[index].id.toString()},
            ),
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(produtosListProvider.notifier).recarregar(),
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
```

---

## 4. Integração com Backend

### Configuração do Dio

Dio é o cliente HTTP mais popular no Flutter, com suporte nativo a interceptors, cancelamento de requisições, upload/download com progresso e timeout configurável.

```dart
// lib/core/network/dio_client.dart
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../storage/secure_storage.dart';
part 'dio_client.g.dart';

const _baseUrl = 'https://api.meuapp.com.br/api/v1';

@riverpod
Dio dioClient(DioClientRef ref) {
  final dio = Dio(BaseOptions(
    baseUrl: _baseUrl,
    connectTimeout: const Duration(seconds: 10),
    receiveTimeout: const Duration(seconds: 30),
    headers: {'Content-Type': 'application/json'},
  ));

  dio.interceptors.add(ApiInterceptor(ref));
  dio.interceptors.add(LogInterceptor(
    requestBody: true,
    responseBody: true,
    logPrint: (obj) => debugPrint(obj.toString()),
  ));

  return dio;
}
```

---

### Interceptors e Tratamento de Erros

```dart
// lib/core/network/api_interceptor.dart
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../storage/secure_storage.dart';

class ApiInterceptor extends Interceptor {
  final Ref ref;
  ApiInterceptor(this.ref);

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    final storage = ref.read(secureStorageProvider);
    final token = await storage.lerToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Token expirado: limpar sessão e redirecionar para login
      final storage = ref.read(secureStorageProvider);
      await storage.limpar();
      // Notificar o provider de auth (ver seção 7)
    }
    handler.next(err);
  }
}
```

```dart
// lib/core/errors/api_exception.dart

class ApiException implements Exception {
  final String mensagem;
  final int? statusCode;

  ApiException({required this.mensagem, this.statusCode});

  factory ApiException.fromDioException(dynamic error) {
    if (error is DioException) {
      return switch (error.type) {
        DioExceptionType.connectionTimeout ||
        DioExceptionType.receiveTimeout =>
          ApiException(mensagem: 'Tempo de conexão esgotado. Tente novamente.'),
        DioExceptionType.badResponse => ApiException(
            mensagem: error.response?.data?['message'] ?? 'Erro no servidor',
            statusCode: error.response?.statusCode,
          ),
        _ => ApiException(mensagem: 'Sem conexão com a internet.'),
      };
    }
    return ApiException(mensagem: 'Erro inesperado.');
  }

  @override
  String toString() => mensagem;
}
```

---

### Repository Pattern

```dart
// lib/features/produtos/domain/produto_model.dart
class Produto {
  final int id;
  final String nome;
  final double preco;
  final bool disponivel;
  final String? imagemUrl;

  const Produto({
    required this.id,
    required this.nome,
    required this.preco,
    required this.disponivel,
    this.imagemUrl,
  });

  factory Produto.fromJson(Map<String, dynamic> json) => Produto(
        id: json['id'],
        nome: json['nome'],
        preco: (json['preco'] as num).toDouble(),
        disponivel: json['disponivel'],
        imagemUrl: json['imagemUrl'],
      );

  Map<String, dynamic> toJson() => {
        'nome': nome,
        'preco': preco,
        'disponivel': disponivel,
      };
}
```

```dart
// lib/features/produtos/data/produto_repository.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../../core/errors/api_exception.dart';
import '../../../core/network/dio_client.dart';
import '../domain/produto_model.dart';
part 'produto_repository.g.dart';

@riverpod
ProdutoRepository produtoRepository(ProdutoRepositoryRef ref) {
  final dio = ref.watch(dioClientProvider);
  return ProdutoRepository(dio);
}

class ProdutoRepository {
  final Dio _dio;
  ProdutoRepository(this._dio);

  Future<List<Produto>> listarTodos({int pagina = 0, int tamanho = 20}) async {
    try {
      final response = await _dio.get(
        '/produtos',
        queryParameters: {'page': pagina, 'size': tamanho},
      );
      final lista = response.data['content'] as List;
      return lista.map((e) => Produto.fromJson(e)).toList();
    } on DioException catch (e) {
      throw ApiException.fromDioException(e);
    }
  }

  Future<Produto> buscarPorId(int id) async {
    try {
      final response = await _dio.get('/produtos/$id');
      return Produto.fromJson(response.data);
    } on DioException catch (e) {
      throw ApiException.fromDioException(e);
    }
  }

  Future<Produto> criar(Produto produto) async {
    try {
      final response = await _dio.post('/produtos', data: produto.toJson());
      return Produto.fromJson(response.data);
    } on DioException catch (e) {
      throw ApiException.fromDioException(e);
    }
  }

  Future<Produto> atualizar(int id, Produto produto) async {
    try {
      final response = await _dio.put('/produtos/$id', data: produto.toJson());
      return Produto.fromJson(response.data);
    } on DioException catch (e) {
      throw ApiException.fromDioException(e);
    }
  }

  Future<void> excluir(int id) async {
    try {
      await _dio.delete('/produtos/$id');
    } on DioException catch (e) {
      throw ApiException.fromDioException(e);
    }
  }
}
```

---

## 5. Telas de Lista e Formulários

### Listas com ListView e Paginação

```dart
// lib/features/produtos/presentation/produtos_list_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import 'produtos_provider.dart';
import '../domain/produto_model.dart';

class ProdutosListScreen extends ConsumerStatefulWidget {
  const ProdutosListScreen({super.key});

  @override
  ConsumerState<ProdutosListScreen> createState() => _ProdutosListScreenState();
}

class _ProdutosListScreenState extends ConsumerState<ProdutosListScreen> {
  final _scrollController = ScrollController();
  int _pagina = 0;
  bool _carregandoMais = false;

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  void _onScroll() {
    final posicao = _scrollController.position;
    if (posicao.pixels >= posicao.maxScrollExtent - 200 && !_carregandoMais) {
      _carregarMais();
    }
  }

  Future<void> _carregarMais() async {
    setState(() { _carregandoMais = true; });
    await ref.read(produtosListProvider.notifier).carregarMais(++_pagina);
    if (mounted) setState(() { _carregandoMais = false; });
  }

  @override
  Widget build(BuildContext context) {
    final produtosAsync = ref.watch(produtosListProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Produtos'),
        actions: [
          IconButton(
            icon: const Icon(Icons.add),
            onPressed: () => context.push('/produtos/novo'),
          ),
        ],
      ),
      body: RefreshIndicator(
        onRefresh: () async {
          setState(() { _pagina = 0; });
          ref.invalidate(produtosListProvider);
        },
        child: produtosAsync.when(
          loading: () => const Center(child: CircularProgressIndicator()),
          error: (e, _) => _ErroWidget(mensagem: e.toString(),
              onRetry: () => ref.invalidate(produtosListProvider)),
          data: (produtos) => produtos.isEmpty
              ? const Center(child: Text('Nenhum produto encontrado.'))
              : ListView.builder(
                  controller: _scrollController,
                  itemCount: produtos.length + (_carregandoMais ? 1 : 0),
                  itemBuilder: (context, index) {
                    if (index == produtos.length) {
                      return const Padding(
                        padding: EdgeInsets.all(16),
                        child: Center(child: CircularProgressIndicator()),
                      );
                    }
                    final produto = produtos[index];
                    return _ItemProduto(
                      produto: produto,
                      onTap: () => context.goNamed(
                        'produto-detalhe',
                        pathParameters: {'id': produto.id.toString()},
                      ),
                    );
                  },
                ),
        ),
      ),
    );
  }
}

class _ItemProduto extends StatelessWidget {
  final Produto produto;
  final VoidCallback onTap;
  const _ItemProduto({required this.produto, required this.onTap});

  @override
  Widget build(BuildContext context) {
    final tema = Theme.of(context);
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 6),
      child: ListTile(
        title: Text(produto.nome, style: tema.textTheme.titleMedium),
        subtitle: Text('R\$ ${produto.preco.toStringAsFixed(2)}'),
        trailing: Chip(
          label: Text(produto.disponivel ? 'Disponível' : 'Indisponível'),
          backgroundColor: produto.disponivel
              ? Colors.green.shade100 : Colors.red.shade100,
        ),
        onTap: onTap,
      ),
    );
  }
}

class _ErroWidget extends StatelessWidget {
  final String mensagem;
  final VoidCallback onRetry;
  const _ErroWidget({required this.mensagem, required this.onRetry});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.error_outline, size: 64, color: Colors.red),
          const SizedBox(height: 12),
          Text(mensagem, textAlign: TextAlign.center),
          const SizedBox(height: 16),
          ElevatedButton.icon(
            onPressed: onRetry,
            icon: const Icon(Icons.refresh),
            label: const Text('Tentar novamente'),
          ),
        ],
      ),
    );
  }
}
```

---

### Formulários e Validação

```dart
// lib/features/produtos/presentation/produto_form_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import 'package:go_router/go_router.dart';
import '../data/produto_repository.dart';
import '../domain/produto_model.dart';

class ProdutoFormScreen extends ConsumerStatefulWidget {
  final Produto? produto; // null = criação, não-null = edição

  const ProdutoFormScreen({super.key, this.produto});

  @override
  ConsumerState<ProdutoFormScreen> createState() => _ProdutoFormScreenState();
}

class _ProdutoFormScreenState extends ConsumerState<ProdutoFormScreen> {
  final _formKey = GlobalKey<FormState>();
  late final TextEditingController _nomeCtrl;
  late final TextEditingController _precoCtrl;
  bool _disponivel = true;
  bool _salvando = false;

  bool get _modoEdicao => widget.produto != null;

  @override
  void initState() {
    super.initState();
    _nomeCtrl = TextEditingController(text: widget.produto?.nome);
    _precoCtrl = TextEditingController(
      text: widget.produto?.preco.toStringAsFixed(2),
    );
    _disponivel = widget.produto?.disponivel ?? true;
  }

  @override
  void dispose() {
    _nomeCtrl.dispose();
    _precoCtrl.dispose();
    super.dispose();
  }

  Future<void> _salvar() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() { _salvando = true; });

    final l10n = AppLocalizations.of(context)!;
    final repo = ref.read(produtoRepositoryProvider);
    final novoProduto = Produto(
      id: widget.produto?.id ?? 0,
      nome: _nomeCtrl.text.trim(),
      preco: double.parse(_precoCtrl.text.replaceAll(',', '.')),
      disponivel: _disponivel,
    );

    try {
      if (_modoEdicao) {
        await repo.atualizar(novoProduto.id, novoProduto);
      } else {
        await repo.criar(novoProduto);
      }

      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(SnackBar(
          content: Text(l10n.produtoSalvoComSucesso),
          backgroundColor: Colors.green,
        ));
        context.pop(true); // retorna true para indicar sucesso
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(SnackBar(
          content: Text(e.toString()),
          backgroundColor: Colors.red,
        ));
      }
    } finally {
      if (mounted) setState(() { _salvando = false; });
    }
  }

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;

    return Scaffold(
      appBar: AppBar(
        title: Text(_modoEdicao ? l10n.editarProduto : l10n.novoProduto),
      ),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            TextFormField(
              controller: _nomeCtrl,
              decoration: InputDecoration(
                labelText: l10n.nomeProduto,
                hintText: l10n.nomeProdutoHint,
                prefixIcon: const Icon(Icons.inventory_2),
                border: const OutlineInputBorder(),
              ),
              textCapitalization: TextCapitalization.words,
              validator: (valor) {
                if (valor == null || valor.trim().isEmpty) {
                  return l10n.campoObrigatorio;
                }
                if (valor.trim().length < 3) {
                  return l10n.nomeMinCaracteres(3);
                }
                return null;
              },
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _precoCtrl,
              decoration: InputDecoration(
                labelText: l10n.precoProduto,
                prefixIcon: const Icon(Icons.attach_money),
                prefixText: 'R\$ ',
                border: const OutlineInputBorder(),
              ),
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
              inputFormatters: [
                FilteringTextInputFormatter.allow(RegExp(r'^\d+[,.]?\d{0,2}')),
              ],
              validator: (valor) {
                if (valor == null || valor.isEmpty) return l10n.campoObrigatorio;
                final numero = double.tryParse(valor.replaceAll(',', '.'));
                if (numero == null || numero <= 0) return l10n.precoInvalido;
                return null;
              },
            ),
            const SizedBox(height: 16),
            SwitchListTile(
              title: Text(l10n.disponivel),
              subtitle: Text(_disponivel ? l10n.produtoDisponivel : l10n.produtoIndisponivel),
              value: _disponivel,
              onChanged: (v) => setState(() { _disponivel = v; }),
            ),
            const SizedBox(height: 32),
            FilledButton(
              onPressed: _salvando ? null : _salvar,
              child: _salvando
                  ? const SizedBox(
                      height: 20, width: 20,
                      child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white))
                  : Text(l10n.salvar),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### Feedback Visual ao Usuário

```dart
// Utilitários para feedback — lib/core/extensions/feedback_extensions.dart
import 'package:flutter/material.dart';

extension FeedbackContext on BuildContext {
  void mostrarSucesso(String mensagem) {
    ScaffoldMessenger.of(this).showSnackBar(SnackBar(
      content: Row(children: [
        const Icon(Icons.check_circle, color: Colors.white),
        const SizedBox(width: 8),
        Expanded(child: Text(mensagem)),
      ]),
      backgroundColor: Colors.green.shade700,
      behavior: SnackBarBehavior.floating,
    ));
  }

  void mostrarErro(String mensagem) {
    ScaffoldMessenger.of(this).showSnackBar(SnackBar(
      content: Row(children: [
        const Icon(Icons.error_outline, color: Colors.white),
        const SizedBox(width: 8),
        Expanded(child: Text(mensagem)),
      ]),
      backgroundColor: Colors.red.shade700,
      behavior: SnackBarBehavior.floating,
    ));
  }

  Future<bool?> confirmarExclusao(String nome) => showDialog<bool>(
        context: this,
        builder: (ctx) => AlertDialog(
          title: const Text('Confirmar exclusão'),
          content: Text('Deseja realmente excluir "$nome"?'),
          actions: [
            TextButton(onPressed: () => Navigator.pop(ctx, false), child: const Text('Cancelar')),
            FilledButton(
              onPressed: () => Navigator.pop(ctx, true),
              style: FilledButton.styleFrom(backgroundColor: Colors.red),
              child: const Text('Excluir'),
            ),
          ],
        ),
      );
}
```

---

## 6. Internacionalização (i18n)

### Configuração do flutter_localizations

**1. pubspec.yaml:**

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.19.0

flutter:
  generate: true
```

**2. l10n.yaml (na raiz do projeto):**

```yaml
arb-dir: lib/l10n
template-arb-file: app_pt.arb
output-localization-file: app_localizations.dart
```

**3. lib/l10n/app_pt.arb:**

```json
{
  "@@locale": "pt",
  "novoProduto": "Novo Produto",
  "editarProduto": "Editar Produto",
  "nomeProduto": "Nome",
  "nomeProdutoHint": "Ex: Notebook Dell Inspiron",
  "precoProduto": "Preço",
  "disponivel": "Disponível para venda",
  "produtoDisponivel": "Produto disponível no estoque",
  "produtoIndisponivel": "Produto fora de estoque",
  "salvar": "Salvar",
  "produtoSalvoComSucesso": "Produto salvo com sucesso!",
  "campoObrigatorio": "Campo obrigatório",
  "nomeMinCaracteres": "Mínimo de {min} caracteres",
  "@nomeMinCaracteres": {
    "placeholders": {
      "min": { "type": "int" }
    }
  },
  "precoInvalido": "Informe um preço válido maior que zero",
  "emailInvalido": "Informe um e-mail válido",
  "senhaMinCaracteres": "A senha deve ter pelo menos {min} caracteres",
  "@senhaMinCaracteres": {
    "placeholders": {
      "min": { "type": "int" }
    }
  },
  "entrar": "Entrar",
  "sair": "Sair",
  "email": "E-mail",
  "senha": "Senha",
  "bemVindo": "Bem-vindo, {nome}!",
  "@bemVindo": {
    "placeholders": {
      "nome": { "type": "String" }
    }
  },
  "erroConexao": "Sem conexão com o servidor. Tente novamente.",
  "credenciaisInvalidas": "E-mail ou senha incorretos."
}
```

**4. lib/l10n/app_en.arb:**

```json
{
  "@@locale": "en",
  "novoProduto": "New Product",
  "editarProduto": "Edit Product",
  "nomeProduto": "Name",
  "nomeProdutoHint": "Ex: Dell Inspiron Notebook",
  "precoProduto": "Price",
  "disponivel": "Available for sale",
  "produtoDisponivel": "Product available in stock",
  "produtoIndisponivel": "Product out of stock",
  "salvar": "Save",
  "produtoSalvoComSucesso": "Product saved successfully!",
  "campoObrigatorio": "Required field",
  "nomeMinCaracteres": "Minimum of {min} characters",
  "@nomeMinCaracteres": {
    "placeholders": {
      "min": { "type": "int" }
    }
  },
  "precoInvalido": "Enter a valid price greater than zero",
  "emailInvalido": "Enter a valid email",
  "senhaMinCaracteres": "Password must be at least {min} characters",
  "@senhaMinCaracteres": {
    "placeholders": {
      "min": { "type": "int" }
    }
  },
  "entrar": "Sign In",
  "sair": "Sign Out",
  "email": "Email",
  "senha": "Password",
  "bemVindo": "Welcome, {nome}!",
  "@bemVindo": {
    "placeholders": {
      "nome": { "type": "String" }
    }
  },
  "erroConexao": "Cannot connect to server. Please try again.",
  "credenciaisInvalidas": "Incorrect email or password."
}
```

Execute para gerar o código:

```bash
flutter gen-l10n
# ou durante desenvolvimento:
flutter run  # a geração ocorre automaticamente
```

---

### Mensagens de Validação Localizadas

```dart
// Usando l10n dentro de um validator (requer BuildContext)
validator: (valor) {
  final l10n = AppLocalizations.of(context)!;
  if (valor == null || valor.isEmpty) return l10n.campoObrigatorio;
  if (!valor.contains('@')) return l10n.emailInvalido;
  return null;
},

// Formatação de datas e moedas com intl
import 'package:intl/intl.dart';

final formatoData = DateFormat('dd/MM/yyyy', 'pt_BR');
final formatoMoeda = NumberFormat.currency(locale: 'pt_BR', symbol: 'R\$');

Text(formatoData.format(DateTime.now()));
Text(formatoMoeda.format(199.90));  // R$ 199,90
```

---

## 7. Autenticação JWT e Controle de Acesso por Role

### Armazenamento Seguro do Token

`flutter_secure_storage` armazena dados criptografados: Keychain (iOS) e EncryptedSharedPreferences (Android).

```dart
// lib/core/storage/secure_storage.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
part 'secure_storage.g.dart';

@riverpod
SecureStorage secureStorage(SecureStorageRef ref) => SecureStorage();

class SecureStorage {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(encryptedSharedPreferences: true),
    iOptions: IOSOptions(accessibility: KeychainAccessibility.first_unlock),
  );

  static const _keyToken = 'auth_token';
  static const _keyRefreshToken = 'auth_refresh_token';

  Future<void> salvarToken(String token, {String? refreshToken}) async {
    await _storage.write(key: _keyToken, value: token);
    if (refreshToken != null) {
      await _storage.write(key: _keyRefreshToken, value: refreshToken);
    }
  }

  Future<String?> lerToken() => _storage.read(key: _keyToken);
  Future<String?> lerRefreshToken() => _storage.read(key: _keyRefreshToken);

  Future<void> limpar() async {
    await _storage.delete(key: _keyToken);
    await _storage.delete(key: _keyRefreshToken);
  }
}
```

---

### Decodificação e Roles do JWT

```dart
// lib/features/auth/domain/user_model.dart
import 'package:jwt_decoder/jwt_decoder.dart';

enum Role { admin, gerente, usuario }

class UsuarioLogado {
  final String token;
  final String email;
  final String nome;
  final List<Role> roles;
  final DateTime expiracao;

  UsuarioLogado({
    required this.token,
    required this.email,
    required this.nome,
    required this.roles,
    required this.expiracao,
  });

  bool get tokenExpirado => JwtDecoder.isExpired(token);

  bool temRole(Role role) => roles.contains(role);

  bool get isAdmin => temRole(Role.admin);

  factory UsuarioLogado.fromToken(String token) {
    final payload = JwtDecoder.decode(token);

    // Adapte o campo de roles ao padrão do seu backend
    // Spring Security costuma usar "roles" ou "authorities"
    final rawRoles = (payload['roles'] as List?)
            ?.map((r) => r.toString().replaceFirst('ROLE_', '').toLowerCase())
            .toList() ??
        [];

    final roles = rawRoles.map((r) {
      return switch (r) {
        'admin' => Role.admin,
        'gerente' => Role.gerente,
        _ => Role.usuario,
      };
    }).toList();

    final exp = payload['exp'] as int;

    return UsuarioLogado(
      token: token,
      email: payload['sub'] ?? payload['email'] ?? '',
      nome: payload['nome'] ?? payload['name'] ?? '',
      roles: roles,
      expiracao: DateTime.fromMillisecondsSinceEpoch(exp * 1000),
    );
  }
}
```

---

### Provider de Autenticação

```dart
// lib/features/auth/presentation/auth_provider.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../../core/network/dio_client.dart';
import '../../../core/storage/secure_storage.dart';
import '../domain/user_model.dart';
part 'auth_provider.g.dart';

@riverpod
class AuthState extends _$AuthState {
  @override
  Future<UsuarioLogado?> build() async {
    // Tenta restaurar sessão ao iniciar o app
    final storage = ref.read(secureStorageProvider);
    final token = await storage.lerToken();
    if (token == null) return null;
    try {
      final usuario = UsuarioLogado.fromToken(token);
      if (usuario.tokenExpirado) {
        await storage.limpar();
        return null;
      }
      return usuario;
    } catch (_) {
      await storage.limpar();
      return null;
    }
  }

  Future<void> login(String email, String senha) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final dio = ref.read(dioClientProvider);
      final response = await dio.post('/auth/login', data: {
        'username': email,
        'password': senha,
      });

      final token = response.data['token'] as String;
      final refreshToken = response.data['refreshToken'] as String?;

      final storage = ref.read(secureStorageProvider);
      await storage.salvarToken(token, refreshToken: refreshToken);

      return UsuarioLogado.fromToken(token);
    });
  }

  Future<void> logout() async {
    final storage = ref.read(secureStorageProvider);
    await storage.limpar();
    state = const AsyncData(null);
  }
}
```

---

### Guard de Rotas por Role

```dart
// lib/routes/app_router.dart — adicionar redirect por role
redirect: (context, state) {
  final usuario = authState.valueOrNull;
  final logado = usuario != null && !usuario.tokenExpirado;
  final naLogin = state.matchedLocation == '/login';

  if (!logado && !naLogin) return '/login';
  if (logado && naLogin) return '/produtos';

  // Proteção de rota /admin apenas para admins
  if (state.matchedLocation.startsWith('/admin') && !(usuario?.isAdmin ?? false)) {
    return '/produtos'; // ou '/acesso-negado'
  }

  return null;
},
```

```dart
// Widget de proteção por role — usar dentro de telas
// lib/features/auth/presentation/role_guard.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../domain/user_model.dart';
import 'auth_provider.dart';

class RoleGuard extends ConsumerWidget {
  final Role roleNecessario;
  final Widget child;
  final Widget? semPermissao;

  const RoleGuard({
    super.key,
    required this.roleNecessario,
    required this.child,
    this.semPermissao,
  });

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final usuario = ref.watch(authStateProvider).valueOrNull;
    if (usuario != null && usuario.temRole(roleNecessario)) return child;
    return semPermissao ?? const SizedBox.shrink();
  }
}
```

```dart
// Uso do RoleGuard em telas
RoleGuard(
  roleNecessario: Role.admin,
  child: IconButton(
    icon: const Icon(Icons.delete),
    onPressed: () => _excluirProduto(),
  ),
)
```

```dart
// Tela de Login completa
// lib/features/auth/presentation/login_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import 'auth_provider.dart';

class LoginScreen extends ConsumerStatefulWidget {
  const LoginScreen({super.key});

  @override
  ConsumerState<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends ConsumerState<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailCtrl = TextEditingController();
  final _senhaCtrl = TextEditingController();
  bool _senhaVisivel = false;

  @override
  void dispose() {
    _emailCtrl.dispose();
    _senhaCtrl.dispose();
    super.dispose();
  }

  Future<void> _login() async {
    if (!_formKey.currentState!.validate()) return;
    await ref.read(authStateProvider.notifier).login(
      _emailCtrl.text.trim(),
      _senhaCtrl.text,
    );

    if (mounted) {
      final authState = ref.read(authStateProvider);
      if (authState.hasError) {
        final l10n = AppLocalizations.of(context)!;
        ScaffoldMessenger.of(context).showSnackBar(SnackBar(
          content: Text(authState.error.toString()),
          backgroundColor: Colors.red,
        ));
      }
      // GoRouter redireciona automaticamente via redirect
    }
  }

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    final authState = ref.watch(authStateProvider);

    return Scaffold(
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24),
            child: Form(
              key: _formKey,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  const FlutterLogo(size: 80),
                  const SizedBox(height: 32),
                  TextFormField(
                    controller: _emailCtrl,
                    decoration: InputDecoration(
                      labelText: l10n.email,
                      prefixIcon: const Icon(Icons.email),
                      border: const OutlineInputBorder(),
                    ),
                    keyboardType: TextInputType.emailAddress,
                    textInputAction: TextInputAction.next,
                    validator: (v) {
                      if (v == null || v.isEmpty) return l10n.campoObrigatorio;
                      if (!v.contains('@')) return l10n.emailInvalido;
                      return null;
                    },
                  ),
                  const SizedBox(height: 16),
                  TextFormField(
                    controller: _senhaCtrl,
                    decoration: InputDecoration(
                      labelText: l10n.senha,
                      prefixIcon: const Icon(Icons.lock),
                      border: const OutlineInputBorder(),
                      suffixIcon: IconButton(
                        icon: Icon(_senhaVisivel
                            ? Icons.visibility_off : Icons.visibility),
                        onPressed: () =>
                            setState(() { _senhaVisivel = !_senhaVisivel; }),
                      ),
                    ),
                    obscureText: !_senhaVisivel,
                    textInputAction: TextInputAction.done,
                    onFieldSubmitted: (_) => _login(),
                    validator: (v) {
                      if (v == null || v.isEmpty) return l10n.campoObrigatorio;
                      if (v.length < 6) return l10n.senhaMinCaracteres(6);
                      return null;
                    },
                  ),
                  const SizedBox(height: 24),
                  FilledButton(
                    onPressed: authState.isLoading ? null : _login,
                    child: authState.isLoading
                        ? const SizedBox(
                            height: 20, width: 20,
                            child: CircularProgressIndicator(
                                strokeWidth: 2, color: Colors.white))
                        : Text(l10n.entrar),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 8. Compilação e Distribuição

### Build para Android

**Pré-requisitos:** Android Studio, Java 17+, conta na Google Play (se publicar).

**1. Configurar assinatura (obrigatório para publicação):**

```bash
# Gerar keystore (faça uma vez e guarde com segurança)
keytool -genkey -v -keystore ~/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload
```

**2. android/key.properties** (nunca commitar no git):

```properties
storePassword=sua_senha_keystore
keyPassword=sua_senha_chave
keyAlias=upload
storeFile=/Users/voce/upload-keystore.jks
```

**3. android/app/build.gradle** — adicionar configuração de assinatura:

```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    // ...
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
        }
    }
}
```

**4. Gerar o bundle (recomendado para Play Store):**

```bash
# App Bundle (formato preferido pela Google Play)
flutter build appbundle --release

# APK universal (para distribuição direta)
flutter build apk --release

# APK por ABI (menor tamanho por dispositivo)
flutter build apk --split-per-abi --release

# Arquivos gerados:
# build/app/outputs/bundle/release/app-release.aab
# build/app/outputs/flutter-apk/app-release.apk
```

**5. Verificar e testar antes de enviar:**

```bash
# Testar o build release em dispositivo físico
flutter install --release

# Analisar tamanho do app
flutter build appbundle --analyze-size
```

---

### Publicação na Google Play Store

1. Acesse [Google Play Console](https://play.google.com/console).
2. Crie o aplicativo (nome, idioma padrão, tipo: app ou jogo).
3. Preencha a **ficha da loja**: descrição, ícones (512×512 PNG), screenshots.
4. Configure **classificação indicativa** (questionário IARC).
5. Em **Produção > Criar versão**, faça upload do `.aab`.
6. Defina o **número de versão** em `pubspec.yaml`:
   ```yaml
   version: 1.0.0+1  # formato: nome+código (versionName+versionCode)
   ```
7. Aguarde a revisão (normalmente 1–3 dias para o primeiro envio).

> **Dica:** use **Testes internos** e **Testes fechados (alfa/beta)** antes de publicar em Produção.

---

### Build para iOS

> **Requisito obrigatório:** macOS com Xcode 15+ instalado. Não é possível compilar para iOS no Windows ou Linux.

**Pré-requisitos:**
- Conta de desenvolvedor Apple (gratuita para testes, paga USD 99/ano para publicação).
- Dispositivo ou simulador iOS configurado no Xcode.

**1. Configurar Bundle Identifier e versão em Xcode:**

```bash
open ios/Runner.xcworkspace
# Em Runner > General: defina Bundle Identifier e Version
```

**2. Configurar assinatura no Xcode:**
- **Runner > Signing & Capabilities > Team**: selecione sua conta Apple.
- Marque **Automatically manage signing** para desenvolvimento.
- Para distribuição: crie manualmente os **Provisioning Profiles** em developer.apple.com.

**3. Build para dispositivo físico (teste):**

```bash
flutter run --release
```

**4. Build para distribuição (App Store):**

```bash
# Gerar arquivo .ipa para App Store
flutter build ipa --release

# Arquivo gerado:
# build/ios/ipa/NomeApp.ipa
```

**5. Upload via Xcode ou Transporter:**

```bash
# Via linha de comando (xcrun altool ou notarytool)
xcrun altool --upload-app --type ios \
  --file build/ios/ipa/NomeApp.ipa \
  --username "seu@apple.id" \
  --password "@keychain:AC_PASSWORD"
```

Ou abra o **Xcode > Product > Archive** e use o **Distributor** integrado.

---

### Publicação na Apple App Store

1. Acesse [App Store Connect](https://appstoreconnect.apple.com).
2. Em **Meus Apps > +**, crie o app (Bundle ID deve coincidir com o Xcode).
3. Preencha a **ficha do app**: nome, subtítulo, descrição, palavras-chave.
4. Adicione **screenshots** para cada tamanho de tela exigido (use o simulador no Xcode).
5. Defina **classificação indicativa**.
6. Em **TestFlight**, distribua para testadores internos e externos antes do lançamento.
7. Faça upload do `.ipa` via Xcode ou Transporter.
8. Submeta para **Revisão da App Store** (prazo típico: 1–2 dias).

---

### Resumo de Comandos de Build

| Plataforma       | Comando                                      | Arquivo gerado                        |
|------------------|----------------------------------------------|---------------------------------------|
| Android (bundle) | `flutter build appbundle --release`          | `build/.../app-release.aab`           |
| Android (APK)    | `flutter build apk --release`                | `build/.../app-release.apk`           |
| iOS (arquivo)    | `flutter build ipa --release`                | `build/ios/ipa/*.ipa`                 |
| Web              | `flutter build web --release`                | `build/web/`                          |
| Windows          | `flutter build windows --release`            | `build/windows/runner/Release/`       |

---

### Automatizando com CI/CD

Para automatizar builds e publicações, use o **Fastlane** (integra com Flutter, Play Store e App Store):

```bash
# Instalar Fastlane
gem install fastlane

# Inicializar no projeto
cd android && fastlane init   # para Android
cd ios && fastlane init       # para iOS
```

Alternativas populares: **Codemagic** (suporte nativo ao Flutter, gratuito para projetos open source) e **GitHub Actions** com as actions oficiais do Flutter.

```yaml
# .github/workflows/build.yml (exemplo GitHub Actions)
name: Build Flutter
on:
  push:
    branches: [main]

jobs:
  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.27.x'
          channel: 'stable'
      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs
      - run: flutter build appbundle --release
```

---

> **Referências:**
> - [flutter.dev](https://flutter.dev) — documentação oficial
> - [pub.dev](https://pub.dev) — repositório de pacotes Dart/Flutter
> - [GoRouter](https://pub.dev/packages/go_router) — navegação declarativa
> - [Riverpod](https://riverpod.dev) — gerenciamento de estado
> - [Dio](https://pub.dev/packages/dio) — cliente HTTP
> - [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) — armazenamento seguro

# Flutter 3 — Referência Completa

Referência consolidada sobre Flutter 3 com Dart, cobrindo fundamentos, navegação, gerenciamento de estado global, integração com backend, telas de lista e formulários, internacionalização, autenticação JWT com controle de acesso por role e distribuição para Android e iOS.

---

## Sumário

1. [Fundamentos do Flutter](#1-fundamentos-do-flutter)
   - [Configuração do Projeto](#configuração-do-projeto)
     - [Instalação no Windows](#instalação-no-windows)
     - [Instalação no macOS](#instalação-no-macos)
   - [Gerenciamento de Versões — FVM](#gerenciamento-de-versões--fvm)
     - [Instalação do FVM no Windows](#instalação-do-fvm-no-windows)
     - [Instalação do FVM no macOS / Linux](#instalação-do-fvm-no-macos--linux)
   - [Configuração das IDEs](#configuração-das-ides)
   - [Estrutura de Pastas](#estrutura-de-pastas)
   - [Arquivos de Configuração](#arquivos-de-configuração)
   - [Widgets Essenciais](#widgets-essenciais)
     - [Estrutura de Tela](#estrutura-de-tela) — Scaffold, AppBar, SafeArea, Drawer
     - [Layout e Posicionamento](#layout-e-posicionamento) — Column, Row, Stack, Container, Padding, SizedBox, Center, Align, Wrap
     - [Expansão e Flexibilidade](#expansão-e-flexibilidade) — Expanded, Flexible, Spacer
     - [Exibição de Conteúdo](#exibição-de-conteúdo) — Text, Icon, Image, Card, ListTile, Chip
     - [Listas e Rolagem](#listas-e-rolagem) — ListView, GridView, SingleChildScrollView, RefreshIndicator
     - [Interação e Botões](#interação-e-botões) — GestureDetector, InkWell, IconButton, FilledButton, ElevatedButton, TextButton, OutlinedButton, FloatingActionButton
     - [Formulários e Inputs](#formulários-e-inputs) — TextField, TextFormField, Switch, Checkbox, Radio, DropdownButtonFormField
     - [Feedback Visual](#feedback-visual-widgets) — CircularProgressIndicator, SnackBar, AlertDialog, BottomSheet
   - [Fundamentos do Dart](#fundamentos-do-dart)
   - [Layouts de Tela Completos](#layouts-de-tela-completos)
     - [Cabeçalho + Corpo Rolável + Rodapé Fixo](#cabeçalho--corpo-rolável--rodapé-fixo)
     - [Duas Colunas — Sidebar + Conteúdo](#duas-colunas--sidebar--conteúdo)
     - [Header + 2 Colunas + Footer Responsivo](#header--2-colunas--footer-responsivo)
     - [Lista com Busca e Filtros](#lista-com-busca-e-filtros)
     - [Grade de Itens — Grid Responsivo](#grade-de-itens--grid-responsivo)
     - [Hero — Transição de Elemento Compartilhado](#hero--transição-de-elemento-compartilhado)
     - [Tela de Perfil](#tela-de-perfil)
     - [Dashboard com Métricas](#dashboard-com-métricas)
     - [Navegação por Abas — TabBar](#navegação-por-abas--tabbar)
2. [Navegação](#2-navegação)
   - [GoRouter](#gorouter)
     - [Configuração do GoRouter](#configuração-do-gorouter)
     - [Rotas Aninhadas e Parâmetros](#rotas-aninhadas-e-parâmetros)
     - [Rotas Protegidas](#rotas-protegidas)
     - [Navegação Programática](#navegação-programática)
     - [Deep Linking](#deep-linking)
   - [flutter_modular](#flutter_modular)
     - [Configuração do flutter_modular](#configuração-do-flutter_modular)
     - [Módulos e Rotas](#módulos-e-rotas)
     - [Guards — Rotas Protegidas](#guards--rotas-protegidas)
     - [Injeção de Dependências](#injeção-de-dependências)
     - [Navegação Programática — flutter_modular](#navegação-programática--flutter_modular)
     - [Deep Linking — flutter_modular](#deep-linking--flutter_modular)
3. [Estado Global com Riverpod](#3-estado-global-com-riverpod)
   - [Conceitos Básicos](#conceitos-básicos)
   - [Providers Principais](#providers-principais)
   - [AsyncNotifier para Estado Assíncrono](#asyncnotifier-para-estado-assíncrono)
4. [Integração com Backend](#4-integração-com-backend)
   - [Separação de Responsabilidades](#separação-de-responsabilidades)
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
9. [WebView no Flutter](#9-webview-no-flutter)
   - [Instalação e Configuração](#instalação-e-configuração)
   - [WebViewController — Controlando o WebView](#webviewcontroller--controlando-o-webview)
   - [Carregamento de Conteúdo](#carregamento-de-conteúdo)
   - [NavigationDelegate — Interceptando Navegação](#navigationdelegate--interceptando-navegação)
   - [Avaliação de JavaScript](#avaliação-de-javascript)
   - [Canais JavaScript — Comunicação Flutter ↔ JS](#canais-javascript--comunicação-flutter--js)
   - [Indicador de Progresso e Tratamento de Erros](#indicador-de-progresso-e-tratamento-de-erros)
   - [Tela Completa com WebView](#tela-completa-com-webview)
10. [HTML + CSS × Flutter — Tabela de Equivalências](#10-html--css--flutter--tabela-de-equivalências)
    - [Elementos HTML → Widgets Flutter](#elementos-html--widgets-flutter)
      - [Estrutura e Layout](#estrutura-e-layout)
      - [Texto e Tipografia](#texto-e-tipografia)
      - [Mídia](#mídia)
      - [Formulários e Inputs](#formulários-e-inputs-1)
      - [Tabelas](#tabelas)
    - [CSS → Flutter](#css--flutter)
      - [Modelo de Caixa](#modelo-de-caixa-box-model)
      - [Display e Flexbox](#display-e-flexbox)
      - [Posicionamento](#posicionamento)
      - [Aparência Visual](#aparência-visual)
      - [Tipografia](#tipografia)
      - [Animações e Transições](#animações-e-transições)
    - [Padrões de Tela — HTML × Flutter lado a lado](#padrões-de-tela--html--flutter-lado-a-lado)
    - [Requisições HTTP Assíncronas — Fetch API × Flutter](#requisições-http-assíncronas--fetch-api--flutter)

---

## 1. Fundamentos do Flutter

### Configuração do Projeto

Flutter exige a instalação do Flutter SDK e, para iOS, do Xcode (macOS obrigatório).

#### Instalação no Windows

**Pré-requisitos:**
- Windows 10 (versão 1903+) ou Windows 11, 64 bits
- [Git for Windows](https://git-scm.com/download/win) 2.x+
- PowerShell 5.0+ (já incluído no Windows 10/11)
- ~2,5 GB para o Flutter SDK + ~8 GB para o Android Studio e Android SDK

**Opção 1 — via `winget` (recomendado):**

```powershell
# Instalar Flutter SDK
winget install Google.Flutter

# Instalar Android Studio (inclui o Android SDK)
winget install Google.AndroidStudio
```

Após a instalação, feche e reabra o terminal para que o `PATH` seja atualizado.

**Opção 2 — instalação manual:**

1. Baixe o SDK em [flutter.dev/install/windows](https://flutter.dev/install/windows).
2. Extraia o `.zip` em `C:\dev\flutter` — evite caminhos com espaços ou caracteres especiais (ex.: não use `C:\Program Files`).
3. Adicione `C:\dev\flutter\bin` ao `PATH`:
   - *Painel de Controle > Sistema > Configurações avançadas do sistema > Variáveis de Ambiente*
   - Em *Variáveis do usuário*, edite `Path` e adicione a entrada `C:\dev\flutter\bin`

   Ou via PowerShell (permanente para o usuário atual):

   ```powershell
   $novo = "C:\dev\flutter\bin"
   $atual = [System.Environment]::GetEnvironmentVariable("Path", "User")
   [System.Environment]::SetEnvironmentVariable("Path", "$atual;$novo", "User")
   ```

**Configurar o Android SDK:**

1. Instale o [Android Studio](https://developer.android.com/studio).
2. No primeiro lançamento, conclua o assistente de configuração (baixa o Android SDK automaticamente).
3. Instale os plugins **Flutter** e **Dart**: *File > Settings > Plugins > Marketplace*.
4. Aceite as licenças do Android:

```powershell
flutter doctor --android-licenses
```

**Habilitar suporte a Windows Desktop (opcional):**

```powershell
flutter config --enable-windows-desktop

# Criar projeto com suporte a Windows
flutter create --platforms=windows meu_app

# Rodar no Windows
flutter run -d windows
```

**Verificar a instalação:**

```powershell
flutter doctor -v
```

Resultado esperado após configuração completa:

```
[✓] Flutter (Channel stable, 3.x.x, on Microsoft Windows 11)
[✓] Windows Version (Windows 10 ou superior)
[✓] Android toolchain - develop for Android devices
[✓] Android Studio (version 2024.x)
[✓] VS Code (version 1.x.x)
[✓] Connected device (x available)
```

> **Atenção:** iOS **não pode** ser compilado no Windows. Para publicar na App Store é obrigatório macOS com Xcode.

---

#### Instalação no macOS

```bash
# Via Homebrew (recomendado)
brew install --cask flutter

# Instalar CocoaPods (necessário para compilar iOS)
sudo gem install cocoapods

# Ativar ferramentas de linha de comando do Xcode (instale o Xcode pela App Store antes)
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch

# Verificar instalação
flutter doctor -v
```

> **Dica:** em vez de instalar o Flutter SDK diretamente, prefira o FVM (seção seguinte) para gerenciar versões por projeto.

---

#### Verificação e Primeiro Projeto

```bash
# Verificar instalação e dependências
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

### Gerenciamento de Versões — FVM

FVM (Flutter Version Manager) permite instalar e alternar entre versões do Flutter SDK, tanto globalmente quanto por projeto — útil quando diferentes projetos exigem versões distintas do SDK.

**Pré-requisitos:**

| Plataforma | Requisito mínimo |
|---|---|
| Windows | Git for Windows 2.x+ · um dos métodos: Chocolatey, Scoop **ou** Dart SDK no `PATH` |
| macOS | Homebrew **ou** Dart SDK no `PATH` |
| Linux | Homebrew (Linuxbrew) **ou** Dart SDK no `PATH` |

O Dart SDK já vem incluído no Flutter SDK. Se o Flutter ainda não estiver instalado, use Chocolatey/Scoop/Homebrew (sem pré-requisito adicional) ou instale o [Dart SDK standalone](https://dart.dev/get-dart) antes de usar `dart pub global`.

---

#### Instalação do FVM no Windows

**Opção 1 — via Chocolatey (não exige Dart pré-instalado):**

```powershell
# 1. Instalar Chocolatey — execute o PowerShell como Administrador
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = `
  [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# 2. Instalar FVM
choco install fvm

# 3. Verificar
fvm --version
```

**Opção 2 — via Scoop (não exige Dart pré-instalado, sem necessidade de Administrador):**

```powershell
# 1. Instalar Scoop (se ainda não tiver)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex

# 2. Adicionar o bucket extras e instalar FVM
scoop bucket add extras
scoop install fvm

# 3. Verificar
fvm --version
```

**Opção 3 — via `dart pub global` (requer Dart ou Flutter já instalado):**

```powershell
dart pub global activate fvm
```

O executável `fvm.bat` é instalado em `%LOCALAPPDATA%\Pub\Cache\bin`. Esse diretório precisa estar no `PATH`:

```powershell
# Verificar se já está no PATH
$env:Path -split ";" | Where-Object { $_ -like "*Pub\Cache\bin*" }

# Se não estiver, adicionar permanentemente para o usuário atual
$pubBin = "$env:LOCALAPPDATA\Pub\Cache\bin"
$atual = [System.Environment]::GetEnvironmentVariable("Path", "User")
[System.Environment]::SetEnvironmentVariable("Path", "$atual;$pubBin", "User")

# Reabrir o terminal e verificar
fvm --version
```

> **Nota:** ao instalar o Flutter via `winget install Google.Flutter`, o `flutter doctor` costuma adicionar `%LOCALAPPDATA%\Pub\Cache\bin` ao `PATH` automaticamente. Nesse caso, `dart pub global activate fvm` já deixa o `fvm` disponível sem configuração manual.

---

#### Instalação do FVM no macOS / Linux

**Via Homebrew (recomendado — não exige Dart pré-instalado):**

```bash
brew tap leoafarias/fvm
brew install fvm

# Verificar
fvm --version
```

**Via `dart pub global` (requer Dart ou Flutter instalado):**

```bash
dart pub global activate fvm

# Verificar se ~/.pub-cache/bin está no PATH
echo $PATH | tr ':' '\n' | grep pub-cache

# Se não estiver, adicionar ao shell (zsh)
echo 'export PATH="$PATH:$HOME/.pub-cache/bin"' >> ~/.zshrc
source ~/.zshrc

# Para bash
echo 'export PATH="$PATH:$HOME/.pub-cache/bin"' >> ~/.bashrc
source ~/.bashrc

# Verificar
fvm --version
```

---

**Versão global (padrão para o sistema):**

```bash
# Listar todas as versões disponíveis para download
fvm releases

# Instalar uma versão específica
fvm install 3.32.0

# Definir a versão global padrão do sistema
fvm global 3.32.0

# Listar versões já instaladas localmente
fvm list
```

**Versão local (fixada por projeto):**

Execute dentro do diretório do projeto. O FVM cria o arquivo `.fvm/fvm_config.json` com a versão fixada — commite esse arquivo para que toda a equipe use a mesma versão.

```bash
# Usar a versão estável mais recente
fvm use stable

# Usar uma versão específica
fvm use 3.32.0

# Usar o canal beta
fvm use beta
```

**Comandos úteis:**

```bash
# Verificar diagnóstico do FVM no projeto atual
fvm doctor

# Remover uma versão instalada
fvm remove 3.24.0

# Ver versão do Flutter gerenciada pelo FVM no projeto
fvm flutter --version
fvm dart --version
```

**Rodar o app com FVM:**

Em projetos com versão fixada, use sempre o prefixo `fvm` para garantir o SDK correto:

```bash
fvm flutter pub get
fvm flutter run                                              # dispositivo padrão
fvm flutter run -d chrome                                    # Web
fvm flutter run -d emulator-5554                            # emulador Android específico
fvm flutter build apk --release
fvm flutter build appbundle --release
fvm flutter build ipa --release
fvm flutter test
fvm dart run build_runner build --delete-conflicting-outputs
fvm dart run build_runner watch
```

**Integração com VS Code** — adicionar em `.vscode/settings.json`:

```json
{
  "dart.flutterSdkPath": ".fvm/flutter_sdk"
}
```

> **Dica:** adicione `.fvm/flutter_sdk` ao `.gitignore` (o link simbólico do SDK não deve ser commitado) e commite `.fvm/fvm_config.json` para que todos no time usem a mesma versão do Flutter.

---

### Configuração das IDEs

#### Android Studio

1. Instale os plugins **Flutter** e **Dart**: *File > Settings > Plugins > Marketplace*.
2. Para abrir um projeto Flutter: *File > Open* — selecione a **raiz do projeto** (onde está o `pubspec.yaml`). Não abra a subpasta `android/`.
3. Se o Flutter SDK não for detectado automaticamente, configure o caminho em:
   *File > Settings > Languages & Frameworks > Flutter > Flutter SDK path*
   - **Sem FVM:** caminho global do SDK (ex.: `~/flutter` ou `C:\flutter`)
   - **Com FVM:** `.fvm/flutter_sdk` na raiz do projeto
4. Para criar e gerenciar emuladores Android: *Tools > Device Manager > +*.

#### VS Code

1. Instale a extensão **Flutter** (inclui a extensão Dart automaticamente).
2. Abra a **pasta raiz** do projeto: *File > Open Folder*.
3. Com FVM, configure `.vscode/settings.json`:
   ```json
   { "dart.flutterSdkPath": ".fvm/flutter_sdk" }
   ```
4. Atalhos úteis:
   - `F5` — iniciar debug
   - `Ctrl+F5` — rodar sem debug
   - `Ctrl+Shift+P > Flutter: Select Device` — trocar dispositivo alvo

#### Xcode (iOS — macOS obrigatório)

> **Importante:** abra sempre o arquivo `.xcworkspace`, **nunca** o `.xcodeproj`. O `.xcworkspace` carrega as dependências CocoaPods.

```bash
# Abrir o workspace correto
open ios/Runner.xcworkspace

# Se os pods não estiverem instalados ou estiverem desatualizados
cd ios && pod install && cd ..
```

Após abrir no Xcode:

1. Selecione o target **Runner** na barra lateral esquerda.
2. Em *Signing & Capabilities*, defina o **Team** (conta Apple Developer).
3. Confirme o **Bundle Identifier** (ex.: `com.empresa.meuapp`).
4. Em *General > Minimum Deployments*, verifique a versão mínima do iOS.

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

### Arquivos de Configuração

Principais arquivos de configuração de um projeto Flutter:

**Raiz do projeto:**

| Arquivo | Função |
|---|---|
| `pubspec.yaml` | Metadados, dependências, assets e fontes — equivalente ao `package.json` |
| `pubspec.lock` | Versões exatas das dependências resolvidas (commitar no git) |
| `analysis_options.yaml` | Regras do analisador Dart (lint) |
| `.fvm/fvm_config.json` | Versão do Flutter fixada por projeto — commitar no git |
| `.fvm/flutter_sdk/` | Link simbólico para o SDK gerenciado pelo FVM — **não commitar** |
| `.flutter-plugins` | Plugins nativos registrados (gerado automaticamente) |
| `.flutter-plugins-dependencies` | Grafo de dependências entre plugins (gerado automaticamente) |
| `.gitignore` | Arquivos ignorados pelo git |
| `.vscode/settings.json` | Configurações do projeto para o VS Code (SDK path) |

**Android (`android/`):**

| Arquivo | Função |
|---|---|
| `app/build.gradle` | Build config: versão mínima do Android, compileSdk, assinatura, minificação |
| `app/src/main/AndroidManifest.xml` | Permissões, activities, deep links, metadados do app |
| `gradle/wrapper/gradle-wrapper.properties` | Versão do Gradle |
| `local.properties` | Caminhos locais do SDK Android — **não commitar** |
| `key.properties` | Credenciais de assinatura (keystore) — **não commitar** |

**iOS (`ios/`):**

| Arquivo | Função |
|---|---|
| `Runner.xcworkspace` | Projeto Xcode com CocoaPods integrados — **sempre abra este arquivo** |
| `Runner/Info.plist` | Permissões, Bundle ID, nome do app, configurações do sistema |
| `Podfile` | Dependências CocoaPods e versão mínima do iOS |
| `Podfile.lock` | Versões exatas dos pods — commitar no git |

**Web (`web/`):**

| Arquivo | Função |
|---|---|
| `index.html` | Ponto de entrada HTML |
| `manifest.json` | Configuração do PWA (ícone, nome, cor de tema) |

**CI/CD:**

| Arquivo | Função |
|---|---|
| `.github/workflows/*.yml` | Pipelines GitHub Actions |
| `fastlane/Fastfile` | Automação de build e publicação com Fastlane |

---

### Widgets Essenciais

No Flutter tudo é widget. Os dois tipos fundamentais são:

- **StatelessWidget** — imutável, redesenhado somente quando o pai muda.
- **StatefulWidget** — possui estado interno que pode ser atualizado com `setState`.

Com Riverpod, a maioria das telas usa `ConsumerWidget` (equivalente a `StatelessWidget`) ou `ConsumerStatefulWidget`.

Os widgets são organizados abaixo por categoria de uso:

- [Estrutura de Tela](#estrutura-de-tela) — Scaffold, AppBar, SafeArea, Drawer, BottomNavigationBar
- [Layout e Posicionamento](#layout-e-posicionamento) — Column, Row, Stack, Container, Padding, SizedBox, Center, Align, Wrap
- [Expansão e Flexibilidade](#expansão-e-flexibilidade) — Expanded, Flexible, Spacer
- [Exibição de Conteúdo](#exibição-de-conteúdo) — Text, RichText, Icon, Image, Card, ListTile, Divider, Chip
- [Listas e Rolagem](#listas-e-rolagem) — ListView, GridView, SingleChildScrollView, RefreshIndicator
- [Interação e Botões](#interação-e-botões) — GestureDetector, InkWell, IconButton, ElevatedButton, FilledButton, TextButton, OutlinedButton, FloatingActionButton
- [Formulários e Inputs](#formulários-e-inputs) — TextField, TextFormField, Checkbox, Switch, Radio, DropdownButton, Form
- [Feedback Visual](#feedback-visual-widgets) — CircularProgressIndicator, LinearProgressIndicator, SnackBar, AlertDialog, BottomSheet

---

#### Estrutura de Tela

**Scaffold**

Fornece a estrutura visual padrão de uma tela Material Design. Gerencia AppBar, body, FAB, Drawer e SnackBars.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `appBar` | `PreferredSizeWidget?` | Barra superior da tela |
| `body` | `Widget?` | Conteúdo principal |
| `floatingActionButton` | `Widget?` | Botão de ação flutuante |
| `floatingActionButtonLocation` | `FloatingActionButtonLocation?` | Posição do FAB |
| `drawer` | `Widget?` | Menu lateral esquerdo |
| `endDrawer` | `Widget?` | Menu lateral direito |
| `bottomNavigationBar` | `Widget?` | Barra de navegação inferior |
| `bottomSheet` | `Widget?` | Painel fixo na base |
| `backgroundColor` | `Color?` | Cor de fundo |
| `resizeToAvoidBottomInset` | `bool` | Redimensiona ao abrir teclado (padrão: `true`) |

```dart
Scaffold(
  appBar: AppBar(title: const Text('Produtos')),
  body: const Center(child: Text('Conteúdo')),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: const Icon(Icons.add),
  ),
  bottomNavigationBar: BottomNavigationBar(
    items: const [
      BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Início'),
      BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Perfil'),
    ],
  ),
)
```

---

**AppBar**

Barra superior padrão do Material Design.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `title` | `Widget?` | Título central ou alinhado à esquerda |
| `leading` | `Widget?` | Widget à esquerda (ex.: botão voltar) |
| `actions` | `List<Widget>?` | Widgets à direita (ícones de ação) |
| `bottom` | `PreferredSizeWidget?` | Widget abaixo da barra (ex.: TabBar) |
| `backgroundColor` | `Color?` | Cor de fundo |
| `foregroundColor` | `Color?` | Cor do texto e ícones |
| `elevation` | `double?` | Sombra |
| `centerTitle` | `bool?` | Centraliza o título |
| `automaticallyImplyLeading` | `bool` | Adiciona botão voltar automaticamente (padrão: `true`) |

```dart
AppBar(
  title: const Text('Produtos'),
  centerTitle: false,
  actions: [
    IconButton(icon: const Icon(Icons.search), onPressed: () {}),
    IconButton(icon: const Icon(Icons.more_vert), onPressed: () {}),
  ],
)
```

---

**SafeArea**

Insere o conteúdo dentro dos limites seguros da tela, evitando sobreposição com notch, barra de status e gestos do sistema.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `child` | `Widget` | Widget filho protegido |
| `top` | `bool` | Aplica inset superior (padrão: `true`) |
| `bottom` | `bool` | Aplica inset inferior (padrão: `true`) |
| `left` | `bool` | Aplica inset esquerdo (padrão: `true`) |
| `right` | `bool` | Aplica inset direito (padrão: `true`) |
| `minimum` | `EdgeInsets` | Inset mínimo garantido |

```dart
// Tela de login sem AppBar — SafeArea protege o conteúdo
Scaffold(
  body: SafeArea(
    child: Padding(
      padding: const EdgeInsets.all(24),
      child: Column(children: [...]),
    ),
  ),
)
```

> **Quando usar:** sempre que a tela não tiver AppBar, ou em telas de splash, onboarding e login onde o Scaffold não gerencia os insets automaticamente.

---

**Drawer**

Painel deslizante lateral para navegação.

```dart
Drawer(
  child: ListView(
    padding: EdgeInsets.zero,
    children: [
      DrawerHeader(
        decoration: const BoxDecoration(color: Colors.indigo),
        child: Text('Menu', style: TextStyle(color: Colors.white, fontSize: 24)),
      ),
      ListTile(
        leading: const Icon(Icons.home),
        title: const Text('Início'),
        onTap: () => context.go('/home'),
      ),
    ],
  ),
)
```

---

#### Layout e Posicionamento

**Column**

Organiza filhos na vertical. Não rola — use `ListView` quando o conteúdo puder ultrapassar a tela.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `children` | `List<Widget>` | Lista de filhos |
| `mainAxisAlignment` | `MainAxisAlignment` | Alinhamento no eixo vertical |
| `crossAxisAlignment` | `CrossAxisAlignment` | Alinhamento no eixo horizontal |
| `mainAxisSize` | `MainAxisSize` | `max` (ocupa toda altura) ou `min` (altura dos filhos) |

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,   // centraliza verticalmente
  crossAxisAlignment: CrossAxisAlignment.stretch, // estica horizontalmente
  children: [
    const Text('Título'),
    const SizedBox(height: 16),
    ElevatedButton(onPressed: () {}, child: const Text('Ação')),
  ],
)
```

| `MainAxisAlignment` | Efeito |
|---|---|
| `start` | Agrupa no início |
| `center` | Centraliza |
| `end` | Agrupa no fim |
| `spaceBetween` | Espaço igual entre filhos |
| `spaceAround` | Espaço ao redor de cada filho |
| `spaceEvenly` | Espaço igual entre e nas bordas |

---

**Row**

Organiza filhos na horizontal. Mesmas propriedades de `Column`, com eixos invertidos.

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [
    Text('Nome'),
    Chip(label: Text('Disponível')),
  ],
)
```

---

**Stack**

Sobrepõe widgets uns sobre os outros (semelhante a `position: absolute` no CSS).

| Propriedade | Tipo | Descrição |
|---|---|---|
| `children` | `List<Widget>` | Filhos (o último fica no topo) |
| `alignment` | `AlignmentGeometry` | Alinhamento padrão dos filhos não posicionados |
| `fit` | `StackFit` | Como filhos não posicionados preenchem o Stack |
| `clipBehavior` | `Clip` | Se corta o conteúdo que extrapola |

**Positioned** — usado dentro de `Stack` para posicionamento absoluto:

```dart
Stack(
  children: [
    Image.network(produto.imagemUrl!),
    Positioned(
      top: 8,
      right: 8,
      child: Chip(label: Text('NOVO')),
    ),
    Positioned.fill(
      child: Material(
        color: Colors.transparent,
        child: InkWell(onTap: onTap),
      ),
    ),
  ],
)
```

---

**Container**

Widget de propósito geral: combina decoração, dimensionamento, padding, margin e transformações.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `child` | `Widget?` | Conteúdo interno |
| `width` / `height` | `double?` | Dimensões fixas |
| `padding` | `EdgeInsetsGeometry?` | Espaço interno |
| `margin` | `EdgeInsetsGeometry?` | Espaço externo |
| `color` | `Color?` | Cor de fundo (exclusivo com `decoration`) |
| `decoration` | `BoxDecoration?` | Bordas, sombra, gradiente, border-radius |
| `alignment` | `AlignmentGeometry?` | Alinha o filho dentro do container |
| `constraints` | `BoxConstraints?` | Limites de tamanho (min/max largura e altura) |
| `transform` | `Matrix4?` | Transformação geométrica (rotação, escala) |

```dart
Container(
  width: double.infinity,
  padding: const EdgeInsets.all(16),
  margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 8)],
    border: Border.all(color: Colors.grey.shade200),
  ),
  child: Text('Conteúdo decorado'),
)
```

> **Dica:** prefira `Padding` quando só precisar de espaçamento, e `DecoratedBox` quando só precisar de decoração — `Container` é conveniente mas mais custoso.

---

**Padding**

Aplica espaço interno ao redor de um único filho. Mais eficiente que `Container` quando só há necessidade de padding.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `padding` | `EdgeInsetsGeometry` | Espaçamento interno |
| `child` | `Widget?` | Widget filho |

```dart
// Construtores EdgeInsets mais usados
EdgeInsets.all(16)                          // todos os lados iguais
EdgeInsets.symmetric(horizontal: 16, vertical: 8)
EdgeInsets.only(top: 8, bottom: 24)
EdgeInsets.fromLTRB(16, 8, 16, 24)         // left, top, right, bottom

Padding(
  padding: const EdgeInsets.symmetric(horizontal: 24),
  child: Text('Com padding lateral'),
)
```

---

**SizedBox**

Define dimensões fixas ou atua como espaçador.

```dart
// Espaçador vertical entre widgets em Column
const SizedBox(height: 16)

// Espaçador horizontal em Row
const SizedBox(width: 8)

// Forçar dimensão em um filho
SizedBox(
  width: double.infinity, // ocupa toda a largura disponível
  height: 48,
  child: FilledButton(onPressed: () {}, child: const Text('Salvar')),
)

// Widget invisível (remove um filho condicionalmente sem afetar o layout)
SizedBox.shrink()
```

---

**Center**

Centraliza seu filho horizontal e verticalmente dentro do espaço disponível.

```dart
Center(
  child: CircularProgressIndicator(),
)
```

---

**Align**

Alinha o filho em uma posição específica dentro do espaço disponível.

```dart
Align(
  alignment: Alignment.bottomRight,
  child: FloatingActionButton(onPressed: () {}, child: Icon(Icons.add)),
)
```

---

**Wrap**

Distribui filhos em linhas (ou colunas), quebrando automaticamente ao atingir o limite de espaço. Útil para chips e tags.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `direction` | `Axis` | Eixo principal (`horizontal` padrão) |
| `spacing` | `double` | Espaço entre filhos no eixo principal |
| `runSpacing` | `double` | Espaço entre linhas |
| `alignment` | `WrapAlignment` | Alinhamento dos filhos |

```dart
Wrap(
  spacing: 8,
  runSpacing: 4,
  children: categorias.map((c) => Chip(label: Text(c))).toList(),
)
```

---

#### Expansão e Flexibilidade

**Expanded**

Ocupa todo o espaço restante disponível no eixo principal de `Row`, `Column` ou `Flex`. Equivalente a `flex: 1` com `flexFit: FlexFit.tight`.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `flex` | `int` | Proporção relativa de espaço (padrão: `1`) |
| `child` | `Widget` | Widget filho |

```dart
Row(
  children: [
    const Icon(Icons.search),
    const SizedBox(width: 8),
    Expanded(             // campo de texto ocupa o restante da linha
      child: TextField(decoration: InputDecoration(hintText: 'Buscar...')),
    ),
  ],
)

// Múltiplos Expanded com proporções diferentes
Row(
  children: [
    Expanded(flex: 2, child: Container(color: Colors.blue)),  // 2/3 da largura
    Expanded(flex: 1, child: Container(color: Colors.red)),   // 1/3 da largura
  ],
)
```

---

**Flexible**

Similar ao `Expanded`, mas o filho pode ser menor que o espaço atribuído (`FlexFit.loose` por padrão). Não força o filho a preencher todo o espaço.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `flex` | `int` | Proporção relativa (padrão: `1`) |
| `fit` | `FlexFit` | `loose` (pode ser menor) ou `tight` (igual ao Expanded) |
| `child` | `Widget` | Widget filho |

```dart
Row(
  children: [
    Flexible(
      child: Text(
        'Texto longo que pode ser truncado sem causar overflow',
        overflow: TextOverflow.ellipsis,
      ),
    ),
    const SizedBox(width: 8),
    const Chip(label: Text('Tag')),
  ],
)
```

> **Expanded vs Flexible:** use `Expanded` quando o filho deve sempre preencher o espaço alocado; use `Flexible` quando o filho pode ser menor (ex.: texto que precisa apenas do espaço necessário, sem forçar expansão).

---

**Spacer**

Atalho para `Expanded(child: SizedBox())`. Preenche espaço vazio em `Row` ou `Column`.

```dart
Row(
  children: [
    Text('Esquerda'),
    const Spacer(),       // empurra o próximo widget para a direita
    Text('Direita'),
  ],
)
```

---

#### Exibição de Conteúdo

**Text**

| Propriedade | Tipo | Descrição |
|---|---|---|
| `data` | `String` | Texto exibido (argumento posicional) |
| `style` | `TextStyle?` | Estilo tipográfico |
| `textAlign` | `TextAlign?` | Alinhamento horizontal do texto |
| `maxLines` | `int?` | Número máximo de linhas |
| `overflow` | `TextOverflow?` | Comportamento ao ultrapassar (`ellipsis`, `clip`, `fade`) |
| `softWrap` | `bool?` | Permite quebra de linha (padrão: `true`) |

```dart
Text(
  'Nome do produto muito longo',
  style: Theme.of(context).textTheme.titleMedium?.copyWith(
    fontWeight: FontWeight.bold,
    color: Colors.indigo,
  ),
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)

// Estilos tipográficos do tema Material 3
// textTheme.displayLarge / displayMedium / displaySmall
// textTheme.headlineLarge / headlineMedium / headlineSmall
// textTheme.titleLarge / titleMedium / titleSmall
// textTheme.bodyLarge / bodyMedium / bodySmall
// textTheme.labelLarge / labelMedium / labelSmall
```

---

**Icon**

| Propriedade | Tipo | Descrição |
|---|---|---|
| `icon` | `IconData` | Ícone (ex.: `Icons.home`) |
| `size` | `double?` | Tamanho em pixels lógicos (padrão: `24`) |
| `color` | `Color?` | Cor do ícone |

```dart
const Icon(Icons.favorite, size: 32, color: Colors.red)
```

---

**Image**

| Construtor | Uso |
|---|---|
| `Image.network(url)` | Imagem remota via URL |
| `Image.asset(path)` | Imagem do pacote (pubspec.yaml) |
| `Image.file(file)` | Imagem do sistema de arquivos |
| `Image.memory(bytes)` | Imagem de bytes em memória |

Propriedades comuns:

| Propriedade | Tipo | Descrição |
|---|---|---|
| `width` / `height` | `double?` | Dimensões |
| `fit` | `BoxFit?` | Como preencher o espaço (`cover`, `contain`, `fill`, `fitWidth`, `fitHeight`) |
| `loadingBuilder` | `ImageLoadingBuilder?` | Widget exibido enquanto carrega |
| `errorBuilder` | `ImageErrorWidgetBuilder?` | Widget exibido em caso de erro |

```dart
Image.network(
  produto.imagemUrl!,
  width: 80,
  height: 80,
  fit: BoxFit.cover,
  loadingBuilder: (_, child, progress) =>
      progress == null ? child : const CircularProgressIndicator(),
  errorBuilder: (_, __, ___) => const Icon(Icons.broken_image, size: 80),
)
```

---

**Card**

Container com elevação e bordas arredondadas padrão Material Design.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `child` | `Widget?` | Conteúdo interno |
| `elevation` | `double?` | Altura da sombra |
| `color` | `Color?` | Cor de fundo |
| `margin` | `EdgeInsetsGeometry?` | Margem externa |
| `shape` | `ShapeBorder?` | Forma (ex.: `RoundedRectangleBorder`) |

```dart
Card(
  elevation: 2,
  margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 6),
  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
  child: ListTile(
    leading: const Icon(Icons.inventory_2),
    title: Text(produto.nome),
    subtitle: Text('R\$ ${produto.preco.toStringAsFixed(2)}'),
    trailing: produto.disponivel
        ? const Icon(Icons.check_circle, color: Colors.green)
        : const Icon(Icons.cancel, color: Colors.red),
    onTap: onTap,
  ),
)
```

---

**ListTile**

Widget de linha pré-formatado para listas, com suporte a ícone, título, subtítulo e widget à direita.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `title` | `Widget?` | Texto principal |
| `subtitle` | `Widget?` | Texto secundário |
| `leading` | `Widget?` | Widget à esquerda (ícone, avatar) |
| `trailing` | `Widget?` | Widget à direita (ícone, chip, switch) |
| `onTap` | `VoidCallback?` | Ação ao tocar |
| `onLongPress` | `VoidCallback?` | Ação ao pressionar longo |
| `selected` | `bool` | Destaca o tile como selecionado |
| `enabled` | `bool` | Habilita ou desabilita interação |
| `contentPadding` | `EdgeInsetsGeometry?` | Padding interno customizado |

---

**Chip**

Elemento compacto para tags, filtros e seleção.

```dart
// Chip simples
Chip(
  label: const Text('Flutter'),
  avatar: const Icon(Icons.code, size: 18),
  onDeleted: () {},           // exibe ícone X
  deleteIcon: const Icon(Icons.close, size: 16),
)

// FilterChip para seleção múltipla
FilterChip(
  label: const Text('Eletrônicos'),
  selected: categoriaSelecionada,
  onSelected: (valor) => setState(() { categoriaSelecionada = valor; }),
)

// ActionChip como botão compacto
ActionChip(
  label: const Text('Compartilhar'),
  avatar: const Icon(Icons.share, size: 18),
  onPressed: () {},
)
```

---

**Divider**

Linha separadora horizontal.

```dart
const Divider()                                  // linha com espessura padrão
const Divider(height: 1, color: Colors.grey)     // linha mais fina
const VerticalDivider(width: 1)                  // separador vertical em Row
```

---

#### Listas e Rolagem

**ListView**

Lista rolável de widgets. Para listas longas, prefira `ListView.builder` para renderização eficiente sob demanda.

| Construtor | Quando usar |
|---|---|
| `ListView(children: [...])` | Listas curtas com poucos itens conhecidos |
| `ListView.builder(...)` | Listas longas ou com dados dinâmicos |
| `ListView.separated(...)` | Lista com separador entre itens |

Propriedades comuns:

| Propriedade | Tipo | Descrição |
|---|---|---|
| `itemCount` | `int?` | Número de itens (`builder` e `separated`) |
| `itemBuilder` | `IndexedWidgetBuilder` | Constrói cada item pelo índice |
| `separatorBuilder` | `IndexedWidgetBuilder` | Constrói o separador (`separated`) |
| `controller` | `ScrollController?` | Controla a posição de rolagem |
| `padding` | `EdgeInsetsGeometry?` | Padding ao redor da lista |
| `scrollDirection` | `Axis` | Direção: `vertical` (padrão) ou `horizontal` |
| `physics` | `ScrollPhysics?` | Comportamento da física de rolagem |
| `shrinkWrap` | `bool` | Lista ocupa apenas o espaço necessário |

```dart
// Lista com separador
ListView.separated(
  itemCount: produtos.length,
  separatorBuilder: (_, __) => const Divider(height: 1),
  itemBuilder: (context, index) => ListTile(
    title: Text(produtos[index].nome),
  ),
)

// Lista horizontal de cards
SizedBox(
  height: 180,
  child: ListView.builder(
    scrollDirection: Axis.horizontal,
    itemCount: destaques.length,
    itemBuilder: (context, index) => Padding(
      padding: const EdgeInsets.only(left: 16),
      child: CardDestaque(item: destaques[index]),
    ),
  ),
)
```

---

**GridView**

Grade rolável de widgets com layout em colunas.

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,          // colunas
    mainAxisSpacing: 12,        // espaço entre linhas
    crossAxisSpacing: 12,       // espaço entre colunas
    childAspectRatio: 3 / 4,    // proporção largura/altura de cada célula
  ),
  itemCount: produtos.length,
  itemBuilder: (context, index) => CardProduto(produto: produtos[index]),
)
```

---

**SingleChildScrollView**

Torna um único widget rolável. Útil para formulários e conteúdos que podem ser maiores que a tela.

```dart
SingleChildScrollView(
  padding: const EdgeInsets.all(16),
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.stretch,
    children: [...],  // formulário ou conteúdo longo
  ),
)
```

---

**RefreshIndicator**

Adiciona o gesto "arrastar para atualizar" (pull-to-refresh) em widgets roláveis.

```dart
RefreshIndicator(
  onRefresh: () async {
    await ref.refresh(produtosListProvider.future);
  },
  child: ListView.builder(...),
)
```

---

#### Interação e Botões

**GestureDetector**

Detecta gestos em qualquer widget. Não exibe feedback visual — prefira `InkWell` para Material Design.

```dart
GestureDetector(
  onTap: () {},
  onDoubleTap: () {},
  onLongPress: () {},
  onHorizontalDragEnd: (details) {},
  child: Container(color: Colors.blue, width: 100, height: 100),
)
```

---

**InkWell**

Detecta toques com efeito ripple visual Material Design. Deve estar dentro de `Material`.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `onTap` | `VoidCallback?` | Ação ao tocar |
| `onLongPress` | `VoidCallback?` | Ação ao pressionar longo |
| `onDoubleTap` | `VoidCallback?` | Ação ao tocar duas vezes |
| `borderRadius` | `BorderRadius?` | Arredonda o ripple |
| `splashColor` | `Color?` | Cor do efeito ripple |

```dart
InkWell(
  onTap: onTap,
  borderRadius: BorderRadius.circular(12),
  child: Padding(
    padding: const EdgeInsets.all(12),
    child: Text('Toque aqui'),
  ),
)
```

---

**IconButton**

Botão com um único ícone, com área de toque mínima garantida pelo Material Design (48×48 dp).

| Propriedade | Tipo | Descrição |
|---|---|---|
| `icon` | `Widget` | Ícone exibido |
| `onPressed` | `VoidCallback?` | Ação ao tocar (`null` desabilita) |
| `tooltip` | `String?` | Texto exibido ao pressionar longo |
| `color` | `Color?` | Cor do ícone |
| `iconSize` | `double?` | Tamanho do ícone |
| `style` | `ButtonStyle?` | Personalização via `IconButton.styleFrom` |
| `visualDensity` | `VisualDensity?` | Ajuste da área de toque |

```dart
// Na AppBar
IconButton(
  icon: const Icon(Icons.search),
  tooltip: 'Buscar',
  onPressed: () => context.push('/busca'),
)

// Botão de exclusão com confirmação
IconButton(
  icon: const Icon(Icons.delete_outline),
  color: Colors.red,
  onPressed: () async {
    final confirmar = await context.confirmarExclusao(produto.nome);
    if (confirmar == true) await _excluir();
  },
)
```

---

**Botões de Ação**

O Material 3 define uma hierarquia clara de botões:

| Widget | Hierarquia | Quando usar |
|---|---|---|
| `FilledButton` | Primário | Ação principal da tela (Salvar, Confirmar) |
| `ElevatedButton` | Primário alternativo | Ação importante com destaque sutil |
| `OutlinedButton` | Secundário | Ação secundária (Cancelar, Voltar) |
| `TextButton` | Terciário | Ação de baixo destaque (links, ações opcionais) |

Propriedades comuns:

| Propriedade | Tipo | Descrição |
|---|---|---|
| `onPressed` | `VoidCallback?` | Ação ao tocar (`null` desabilita) |
| `child` | `Widget` | Conteúdo (geralmente `Text` ou `Row` com ícone) |
| `style` | `ButtonStyle?` | Personalização via `XButton.styleFrom(...)` |

```dart
// Botão primário — ocupa toda a largura
SizedBox(
  width: double.infinity,
  child: FilledButton(
    onPressed: _salvando ? null : _salvar,
    child: _salvando
        ? const SizedBox(
            height: 20, width: 20,
            child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white))
        : const Text('Salvar'),
  ),
)

// Botão com ícone
FilledButton.icon(
  onPressed: () {},
  icon: const Icon(Icons.save),
  label: const Text('Salvar'),
)

// Personalização com styleFrom
ElevatedButton(
  onPressed: () {},
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.red,
    foregroundColor: Colors.white,
    padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
  ),
  child: const Text('Excluir'),
)

// Linha de ações típica de formulário
Row(
  children: [
    Expanded(
      child: OutlinedButton(onPressed: () => context.pop(), child: const Text('Cancelar')),
    ),
    const SizedBox(width: 12),
    Expanded(
      child: FilledButton(onPressed: _salvar, child: const Text('Confirmar')),
    ),
  ],
)
```

---

**FloatingActionButton**

Botão de ação flutuante, posicionado pelo `Scaffold`.

```dart
FloatingActionButton(
  onPressed: () => context.push('/produtos/novo'),
  tooltip: 'Novo produto',
  child: const Icon(Icons.add),
)

// Versão estendida com rótulo
FloatingActionButton.extended(
  onPressed: () {},
  icon: const Icon(Icons.add),
  label: const Text('Novo Produto'),
)
```

---

#### Formulários e Inputs

**TextField / TextFormField**

`TextFormField` é o `TextField` integrado ao `Form` para validação automática.

| Propriedade | Tipo | Descrição |
|---|---|---|
| `controller` | `TextEditingController?` | Controla o valor do texto |
| `decoration` | `InputDecoration?` | Aparência (label, hint, ícone, borda) |
| `keyboardType` | `TextInputType?` | Tipo de teclado |
| `textInputAction` | `TextInputAction?` | Botão de ação do teclado (`next`, `done`, `search`) |
| `obscureText` | `bool` | Oculta o texto (senha) |
| `maxLines` | `int?` | `1` (padrão) ou `null` para multilinha |
| `inputFormatters` | `List<TextInputFormatter>?` | Filtros de entrada |
| `onChanged` | `ValueChanged<String>?` | Callback a cada alteração |
| `onFieldSubmitted` | `ValueChanged<String>?` | Callback ao pressionar Enter/Done |
| `validator` | `FormFieldValidator<String>?` | Apenas em `TextFormField` |
| `focusNode` | `FocusNode?` | Controla o foco programaticamente |
| `autofocus` | `bool` | Foca automaticamente ao exibir |
| `readOnly` | `bool` | Campo somente leitura |
| `enabled` | `bool?` | Habilita ou desabilita o campo |

```dart
TextFormField(
  controller: _emailCtrl,
  decoration: const InputDecoration(
    labelText: 'E-mail',
    hintText: 'nome@exemplo.com',
    prefixIcon: Icon(Icons.email),
    border: OutlineInputBorder(),
    // Variações de borda:
    // border: UnderlineInputBorder()  — apenas linha inferior
    // border: InputBorder.none        — sem borda
  ),
  keyboardType: TextInputType.emailAddress,
  textInputAction: TextInputAction.next,    // avança para o próximo campo
  validator: (v) {
    if (v == null || v.isEmpty) return 'Campo obrigatório';
    if (!v.contains('@')) return 'E-mail inválido';
    return null;
  },
)
```

---

**Form e GlobalKey**

Agrupa `TextFormField`s para validação e reset coletivos.

```dart
final _formKey = GlobalKey<FormState>();

Form(
  key: _formKey,
  child: Column(children: [
    TextFormField(validator: ...),
    TextFormField(validator: ...),
    ElevatedButton(
      onPressed: () {
        if (_formKey.currentState!.validate()) {
          // todos os campos são válidos
          _formKey.currentState!.save();   // dispara onSaved de cada campo
        }
      },
      child: const Text('Enviar'),
    ),
  ]),
)
```

---

**Switch, Checkbox e Radio**

```dart
// Switch — ativa/desativa uma opção
SwitchListTile(
  title: const Text('Disponível para venda'),
  subtitle: Text(_ativo ? 'Produto ativo' : 'Produto inativo'),
  value: _ativo,
  onChanged: (v) => setState(() { _ativo = v; }),
)

// Checkbox
CheckboxListTile(
  title: const Text('Aceito os termos'),
  value: _termos,
  onChanged: (v) => setState(() { _termos = v ?? false; }),
  controlAffinity: ListTileControlAffinity.leading,
)

// Radio — seleção exclusiva em grupo
Column(
  children: Categoria.values.map((c) => RadioListTile<Categoria>(
    title: Text(c.nome),
    value: c,
    groupValue: _categoriaSelecionada,
    onChanged: (v) => setState(() { _categoriaSelecionada = v; }),
  )).toList(),
)
```

---

**DropdownButtonFormField**

Seleção a partir de uma lista suspensa, integrado ao `Form`.

```dart
DropdownButtonFormField<String>(
  value: _estadoSelecionado,
  decoration: const InputDecoration(
    labelText: 'Estado',
    border: OutlineInputBorder(),
  ),
  items: estados.map((e) => DropdownMenuItem(value: e, child: Text(e))).toList(),
  onChanged: (v) => setState(() { _estadoSelecionado = v; }),
  validator: (v) => v == null ? 'Selecione um estado' : null,
)
```

---

#### Feedback Visual Widgets

**CircularProgressIndicator / LinearProgressIndicator**

```dart
// Indeterminado (carregamento de duração desconhecida)
const CircularProgressIndicator()
const LinearProgressIndicator()

// Determinado (progresso conhecido de 0.0 a 1.0)
CircularProgressIndicator(value: _progresso)
LinearProgressIndicator(value: _progresso)

// Centralizado em uma tela de carregamento
const Center(child: CircularProgressIndicator())

// Inline em botão durante operação assíncrona
SizedBox(
  height: 20, width: 20,
  child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white),
)
```

---

**SnackBar**

Mensagem temporária exibida na parte inferior da tela.

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: const Text('Produto salvo com sucesso!'),
    backgroundColor: Colors.green,
    behavior: SnackBarBehavior.floating,   // flutua sobre o conteúdo
    duration: const Duration(seconds: 3),
    action: SnackBarAction(
      label: 'Desfazer',
      onPressed: () {},
    ),
  ),
)
```

---

**AlertDialog**

Caixa de diálogo modal para confirmações e alertas.

```dart
final confirmar = await showDialog<bool>(
  context: context,
  barrierDismissible: false,   // impede fechar ao tocar fora
  builder: (ctx) => AlertDialog(
    icon: const Icon(Icons.warning, color: Colors.orange),
    title: const Text('Confirmar exclusão'),
    content: Text('Deseja excluir "${produto.nome}"? Esta ação não pode ser desfeita.'),
    actions: [
      TextButton(
        onPressed: () => Navigator.pop(ctx, false),
        child: const Text('Cancelar'),
      ),
      FilledButton(
        onPressed: () => Navigator.pop(ctx, true),
        style: FilledButton.styleFrom(backgroundColor: Colors.red),
        child: const Text('Excluir'),
      ),
    ],
  ),
);

if (confirmar == true) await _excluir();
```

---

**BottomSheet**

Painel que desliza a partir da base da tela.

```dart
// Modal (bloqueia interação com o fundo)
showModalBottomSheet(
  context: context,
  isScrollControlled: true,     // permite altura maior que 50% da tela
  shape: const RoundedRectangleBorder(
    borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
  ),
  builder: (ctx) => Padding(
    padding: EdgeInsets.only(bottom: MediaQuery.of(ctx).viewInsets.bottom),
    child: FiltrosSheet(),
  ),
)
```

---

```dart
// lib/features/produtos/presentation/card_produto.dart — exemplo combinando widgets
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

### Layouts de Tela Completos

Exemplos de composições de tela recorrentes em apps Flutter, do mais simples ao mais composto. Todos pressupõem `MaterialApp` com `ThemeData` e `GoRouter` configurados.

---

#### Cabeçalho + Corpo Rolável + Rodapé Fixo

Padrão mais comum em telas de detalhe: AppBar fixa no topo, conteúdo rolável no meio e área de ação fixa no rodapé.

```dart
// lib/features/produtos/presentation/produto_detalhe_screen.dart
class ProdutoDetalheScreen extends ConsumerWidget {
  final String id;
  const ProdutoDetalheScreen({super.key, required this.id});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final produtoAsync = ref.watch(produtoPorIdProvider(int.parse(id)));

    return produtoAsync.when(
      loading: () => const Scaffold(body: Center(child: CircularProgressIndicator())),
      error: (e, _) => Scaffold(body: Center(child: Text('Erro: $e'))),
      data: (produto) => Scaffold(
        appBar: AppBar(title: Text(produto.nome)),
        body: Column(
          children: [
            // Banner fixo abaixo da AppBar
            Container(
              width: double.infinity,
              color: Theme.of(context).colorScheme.primaryContainer,
              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
              child: Row(
                children: [
                  const Icon(Icons.local_shipping, size: 16),
                  const SizedBox(width: 6),
                  Text('Frete grátis acima de R\$ 200',
                      style: Theme.of(context).textTheme.bodySmall),
                ],
              ),
            ),

            // Corpo rolável ocupa o espaço restante
            Expanded(
              child: SingleChildScrollView(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    ClipRRect(
                      borderRadius: BorderRadius.circular(12),
                      child: Image.network(
                        produto.imagemUrl ?? '',
                        height: 280,
                        width: double.infinity,
                        fit: BoxFit.cover,
                        errorBuilder: (_, __, ___) => Container(
                          height: 280,
                          color: Colors.grey.shade100,
                          child: const Icon(Icons.inventory_2, size: 80, color: Colors.grey),
                        ),
                      ),
                    ),
                    const SizedBox(height: 16),
                    Text(produto.nome,
                        style: Theme.of(context).textTheme.headlineSmall),
                    const SizedBox(height: 6),
                    Text(
                      'R\$ ${produto.preco.toStringAsFixed(2)}',
                      style: Theme.of(context).textTheme.titleLarge?.copyWith(
                            color: Theme.of(context).colorScheme.primary,
                            fontWeight: FontWeight.bold,
                          ),
                    ),
                    const SizedBox(height: 16),
                    const Divider(),
                    Text('Descrição',
                        style: Theme.of(context).textTheme.titleMedium),
                    const SizedBox(height: 8),
                    Text(produto.descricao ?? ''),
                    const SizedBox(height: 80), // espaço para o rodapé não cobrir o conteúdo
                  ],
                ),
              ),
            ),

            // Rodapé fixo — ação principal
            SafeArea(
              child: Container(
                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
                decoration: BoxDecoration(
                  color: Theme.of(context).colorScheme.surface,
                  boxShadow: const [BoxShadow(color: Colors.black12, blurRadius: 8, offset: Offset(0, -2))],
                ),
                child: Row(
                  children: [
                    // Preço resumido
                    Column(
                      mainAxisSize: MainAxisSize.min,
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Text('Total', style: TextStyle(fontSize: 11, color: Colors.grey)),
                        Text(
                          'R\$ ${produto.preco.toStringAsFixed(2)}',
                          style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 18),
                        ),
                      ],
                    ),
                    const SizedBox(width: 16),
                    Expanded(
                      child: FilledButton.icon(
                        onPressed: produto.disponivel
                            ? () => ref.read(carrinhoProvider.notifier).adicionar(produto)
                            : null,
                        icon: const Icon(Icons.shopping_cart),
                        label: Text(produto.disponivel ? 'Adicionar ao Carrinho' : 'Indisponível'),
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

#### Duas Colunas — Sidebar + Conteúdo

Layout de gestão para tablets: menu lateral fixo à esquerda e área de conteúdo à direita.

```dart
// lib/features/admin/presentation/admin_layout.dart
class AdminLayout extends StatefulWidget {
  final Widget conteudo;
  const AdminLayout({super.key, required this.conteudo});

  @override
  State<AdminLayout> createState() => _AdminLayoutState();
}

class _AdminLayoutState extends State<AdminLayout> {
  int _indiceMenu = 0;

  static const _itensMenu = [
    (icone: Icons.dashboard,    rotulo: 'Dashboard',     rota: '/admin'),
    (icone: Icons.inventory_2,  rotulo: 'Produtos',      rota: '/admin/produtos'),
    (icone: Icons.people,       rotulo: 'Clientes',      rota: '/admin/clientes'),
    (icone: Icons.receipt_long, rotulo: 'Pedidos',       rota: '/admin/pedidos'),
    (icone: Icons.bar_chart,    rotulo: 'Relatórios',    rota: '/admin/relatorios'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Row(
        children: [
          // Sidebar fixa
          NavigationRail(
            selectedIndex: _indiceMenu,
            onDestinationSelected: (i) {
              setState(() => _indiceMenu = i);
              context.go(_itensMenu[i].rota);
            },
            labelType: NavigationRailLabelType.selected,
            leading: Padding(
              padding: const EdgeInsets.symmetric(vertical: 16),
              child: Icon(Icons.store, color: Theme.of(context).colorScheme.primary, size: 28),
            ),
            destinations: _itensMenu
                .map((m) => NavigationRailDestination(
                      icon: Icon(m.icone),
                      label: Text(m.rotulo),
                    ))
                .toList(),
          ),
          const VerticalDivider(width: 1),
          // Conteúdo principal — ocupa o restante
          Expanded(child: widget.conteudo),
        ],
      ),
    );
  }
}
```

> **Quando usar:** `NavigationRail` é a solução recomendada pelo Material 3 para navegação lateral. Para sidebars mais largas com submenus, use `Drawer` com `ListView` customizada.

---

#### Header + 2 Colunas + Footer Responsivo

Tela de checkout com indicador de progresso no topo, formulário + resumo em colunas (tablet) ou empilhado (celular) e barra de ação fixa no rodapé.

```dart
// lib/features/checkout/presentation/checkout_screen.dart
class CheckoutScreen extends StatelessWidget {
  const CheckoutScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final largura = MediaQuery.sizeOf(context).width;
    final isTablet = largura > 768;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Checkout'),
        // Header — indicador de etapas
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(72),
          child: Padding(
            padding: const EdgeInsets.fromLTRB(16, 0, 16, 12),
            child: Row(
              children: [
                _EtapaCheckout(numero: '1', rotulo: 'Endereço', ativa: true),
                const Expanded(child: Divider()),
                _EtapaCheckout(numero: '2', rotulo: 'Pagamento', ativa: false),
                const Expanded(child: Divider()),
                _EtapaCheckout(numero: '3', rotulo: 'Confirmação', ativa: false),
              ],
            ),
          ),
        ),
      ),

      // Corpo — 2 colunas em tablet, 1 coluna em celular
      body: isTablet
          ? Row(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Expanded(
                  flex: 3,
                  child: SingleChildScrollView(
                    padding: const EdgeInsets.all(24),
                    child: _FormularioEndereco(),
                  ),
                ),
                const VerticalDivider(width: 1),
                SizedBox(
                  width: 320,
                  child: SingleChildScrollView(
                    padding: const EdgeInsets.all(24),
                    child: _ResumoPedido(),
                  ),
                ),
              ],
            )
          : SingleChildScrollView(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  _FormularioEndereco(),
                  const SizedBox(height: 24),
                  _ResumoPedido(),
                  const SizedBox(height: 80),
                ],
              ),
            ),

      // Footer fixo — ações de navegação
      bottomNavigationBar: SafeArea(
        child: Padding(
          padding: const EdgeInsets.fromLTRB(16, 8, 16, 12),
          child: Row(
            children: [
              OutlinedButton(
                onPressed: () => context.pop(),
                child: const Text('Voltar'),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: FilledButton(
                  onPressed: () => context.push('/checkout/pagamento'),
                  child: const Text('Continuar para Pagamento'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class _EtapaCheckout extends StatelessWidget {
  final String numero;
  final String rotulo;
  final bool ativa;

  const _EtapaCheckout({required this.numero, required this.rotulo, required this.ativa});

  @override
  Widget build(BuildContext context) {
    final cor = ativa
        ? Theme.of(context).colorScheme.primary
        : Theme.of(context).colorScheme.outlineVariant;

    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        CircleAvatar(
          radius: 14,
          backgroundColor: cor,
          child: Text(numero,
              style: TextStyle(
                  color: ativa ? Colors.white : Theme.of(context).colorScheme.outline,
                  fontSize: 12,
                  fontWeight: FontWeight.bold)),
        ),
        const SizedBox(height: 4),
        Text(rotulo, style: TextStyle(color: cor, fontSize: 11)),
      ],
    );
  }
}
```

---

#### Lista com Busca e Filtros

Barra de busca integrada na AppBar, chips de filtro horizontal e lista com resultado vazio tratado.

```dart
// lib/features/produtos/presentation/produtos_busca_screen.dart
class ProdutosBuscaScreen extends ConsumerStatefulWidget {
  const ProdutosBuscaScreen({super.key});

  @override
  ConsumerState<ProdutosBuscaScreen> createState() => _ProdutosBuscaScreenState();
}

class _ProdutosBuscaScreenState extends ConsumerState<ProdutosBuscaScreen> {
  final _buscaCtrl = TextEditingController();
  String _busca = '';
  String? _categoriaFiltro;

  static const _categorias = ['Eletrônicos', 'Roupas', 'Alimentos', 'Casa'];

  @override
  void dispose() {
    _buscaCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final produtosAsync = ref.watch(produtosListProvider);

    return Scaffold(
      appBar: AppBar(
        title: TextField(
          controller: _buscaCtrl,
          autofocus: true,
          decoration: InputDecoration(
            hintText: 'Buscar produtos...',
            border: InputBorder.none,
            suffixIcon: _busca.isNotEmpty
                ? IconButton(
                    icon: const Icon(Icons.clear),
                    onPressed: () {
                      _buscaCtrl.clear();
                      setState(() => _busca = '');
                    },
                  )
                : null,
          ),
          onChanged: (v) => setState(() => _busca = v),
        ),
        // Chips de categoria — rolagem horizontal
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(52),
          child: SizedBox(
            height: 52,
            child: ListView(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
              children: [
                // Chip "Todos"
                Padding(
                  padding: const EdgeInsets.only(right: 8),
                  child: FilterChip(
                    label: const Text('Todos'),
                    selected: _categoriaFiltro == null,
                    onSelected: (_) => setState(() => _categoriaFiltro = null),
                  ),
                ),
                // Chips por categoria
                ..._categorias.map((cat) => Padding(
                      padding: const EdgeInsets.only(right: 8),
                      child: FilterChip(
                        label: Text(cat),
                        selected: _categoriaFiltro == cat,
                        onSelected: (_) => setState(() {
                          _categoriaFiltro = _categoriaFiltro == cat ? null : cat;
                        }),
                      ),
                    )),
              ],
            ),
          ),
        ),
      ),

      body: produtosAsync.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, _) => Center(child: Text('Erro ao carregar: $e')),
        data: (todos) {
          final filtrados = todos.where((p) {
            final buscaOk = _busca.isEmpty ||
                p.nome.toLowerCase().contains(_busca.toLowerCase());
            final catOk = _categoriaFiltro == null || p.categoria == _categoriaFiltro;
            return buscaOk && catOk;
          }).toList();

          if (filtrados.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.search_off, size: 64, color: Colors.grey.shade400),
                  const SizedBox(height: 12),
                  Text(
                    'Nenhum resultado para "$_busca"',
                    style: Theme.of(context).textTheme.bodyLarge,
                  ),
                  const SizedBox(height: 8),
                  TextButton(
                    onPressed: () {
                      _buscaCtrl.clear();
                      setState(() {
                        _busca = '';
                        _categoriaFiltro = null;
                      });
                    },
                    child: const Text('Limpar filtros'),
                  ),
                ],
              ),
            );
          }

          return ListView.separated(
            itemCount: filtrados.length,
            separatorBuilder: (_, __) => const Divider(height: 1, indent: 72, endIndent: 16),
            itemBuilder: (_, i) {
              final p = filtrados[i];
              return ListTile(
                contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
                leading: ClipRRect(
                  borderRadius: BorderRadius.circular(8),
                  child: Image.network(p.imagemUrl ?? '', width: 48, height: 48, fit: BoxFit.cover,
                      errorBuilder: (_, __, ___) =>
                          Container(width: 48, height: 48, color: Colors.grey.shade100,
                              child: const Icon(Icons.inventory_2))),
                ),
                title: Text(p.nome),
                subtitle: Text(p.categoria ?? ''),
                trailing: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  crossAxisAlignment: CrossAxisAlignment.end,
                  children: [
                    Text('R\$ ${p.preco.toStringAsFixed(2)}',
                        style: const TextStyle(fontWeight: FontWeight.bold)),
                    if (!p.disponivel)
                      const Text('Indisponível',
                          style: TextStyle(color: Colors.red, fontSize: 11)),
                  ],
                ),
                onTap: () => context.goNamed('produto-detalhe',
                    pathParameters: {'id': p.id.toString()}),
              );
            },
          );
        },
      ),
    );
  }
}
```

---

#### Grade de Itens — Grid Responsivo

Grid com número de colunas adaptável à largura da tela e card com imagem, nome e preço.

```dart
// lib/features/produtos/presentation/produtos_grid_screen.dart
class ProdutosGridScreen extends ConsumerWidget {
  const ProdutosGridScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final produtosAsync = ref.watch(produtosListProvider);
    final largura = MediaQuery.sizeOf(context).width;

    final colunas = switch (largura) {
      > 1200 => 4,
      > 800  => 3,
      > 500  => 2,
      _      => 2,
    };

    return Scaffold(
      appBar: AppBar(
        title: const Text('Catálogo'),
        actions: [
          IconButton(icon: const Icon(Icons.filter_list), onPressed: () {}),
          IconButton(icon: const Icon(Icons.view_list), onPressed: () {}),
        ],
      ),
      body: produtosAsync.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, _) => Center(child: Text('Erro: $e')),
        data: (produtos) => GridView.builder(
          padding: const EdgeInsets.all(12),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: colunas,
            mainAxisSpacing: 12,
            crossAxisSpacing: 12,
            childAspectRatio: 0.68,
          ),
          itemCount: produtos.length,
          itemBuilder: (_, i) => _CardGrid(produto: produtos[i]),
        ),
      ),
    );
  }
}

class _CardGrid extends StatelessWidget {
  final Produto produto;
  const _CardGrid({required this.produto});

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: () => context.goNamed('produto-detalhe',
            pathParameters: {'id': produto.id.toString()}),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Imagem com overlay de indisponível
            Expanded(
              flex: 3,
              child: Stack(
                fit: StackFit.expand,
                children: [
                  Image.network(
                    produto.imagemUrl ?? '',
                    fit: BoxFit.cover,
                    errorBuilder: (_, __, ___) => Container(
                      color: Colors.grey.shade100,
                      child: const Icon(Icons.inventory_2, size: 40, color: Colors.grey),
                    ),
                  ),
                  if (!produto.disponivel)
                    Container(
                      color: Colors.black54,
                      alignment: Alignment.center,
                      child: const Text('Indisponível',
                          style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
                    ),
                  // Badge de categoria
                  if (produto.categoria != null)
                    Positioned(
                      top: 6,
                      left: 6,
                      child: Container(
                        padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
                        decoration: BoxDecoration(
                          color: Theme.of(context).colorScheme.primaryContainer,
                          borderRadius: BorderRadius.circular(4),
                        ),
                        child: Text(produto.categoria!,
                            style: const TextStyle(fontSize: 10, fontWeight: FontWeight.bold)),
                      ),
                    ),
                ],
              ),
            ),
            // Informações
            Expanded(
              flex: 2,
              child: Padding(
                padding: const EdgeInsets.all(8),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(produto.nome,
                        maxLines: 2,
                        overflow: TextOverflow.ellipsis,
                        style: Theme.of(context).textTheme.bodyMedium),
                    const Spacer(),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text(
                          'R\$ ${produto.preco.toStringAsFixed(2)}',
                          style: TextStyle(
                            fontWeight: FontWeight.bold,
                            color: Theme.of(context).colorScheme.primary,
                          ),
                        ),
                        Icon(
                          Icons.add_circle_outline,
                          size: 20,
                          color: produto.disponivel
                              ? Theme.of(context).colorScheme.primary
                              : Colors.grey,
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

#### Hero — Transição de Elemento Compartilhado

`Hero` anima a transição de um widget entre duas rotas, criando a ilusão de que o elemento "voa" de uma tela para outra. O `tag` deve ser único e idêntico nas duas rotas.

```dart
// Tela de lista — widget de origem
class _CardComHero extends StatelessWidget {
  final Produto produto;
  const _CardComHero({required this.produto});

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: () => context.push('/produtos/${produto.id}'),
        child: Column(
          children: [
            Hero(
              tag: 'produto-imagem-${produto.id}', // tag único por produto
              child: Image.network(produto.imagemUrl!,
                  height: 160, width: double.infinity, fit: BoxFit.cover),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: Text(produto.nome),
            ),
          ],
        ),
      ),
    );
  }
}

// Tela de detalhe — SliverAppBar com a mesma tag no Hero
class ProdutoDetalheHeroScreen extends StatelessWidget {
  final Produto produto;
  const ProdutoDetalheHeroScreen({super.key, required this.produto});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 300,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: Text(produto.nome),
              background: Hero(
                tag: 'produto-imagem-${produto.id}', // mesma tag — Flutter anima
                child: Image.network(produto.imagemUrl!, fit: BoxFit.cover),
              ),
            ),
          ),
          SliverPadding(
            padding: const EdgeInsets.all(16),
            sliver: SliverList(
              delegate: SliverChildListDelegate([
                Text('R\$ ${produto.preco.toStringAsFixed(2)}',
                    style: Theme.of(context).textTheme.headlineSmall?.copyWith(
                        fontWeight: FontWeight.bold,
                        color: Theme.of(context).colorScheme.primary)),
                const SizedBox(height: 16),
                Text(produto.descricao ?? ''),
              ]),
            ),
          ),
        ],
      ),
    );
  }
}
```

> **Atenção:** use `context.push()` (não `context.go()`) para manter a rota anterior na pilha e garantir a animação de retorno. O Hero funciona tanto com rotas nomeadas do GoRouter quanto com `Navigator.push`.

---

#### Tela de Perfil

Layout com `SliverAppBar` expansível contendo avatar e nome, seguida de cards de informação e ações.

```dart
// lib/features/perfil/presentation/perfil_screen.dart
class PerfilScreen extends ConsumerWidget {
  const PerfilScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final usuario = ref.watch(authStateProvider).valueOrNull;
    if (usuario == null) return const SizedBox.shrink();

    final inicial = usuario.nome.isNotEmpty ? usuario.nome[0].toUpperCase() : 'U';

    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // Header expansível com gradiente
          SliverAppBar(
            expandedHeight: 220,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              background: Container(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topLeft,
                    end: Alignment.bottomRight,
                    colors: [
                      Theme.of(context).colorScheme.primary,
                      Theme.of(context).colorScheme.tertiary,
                    ],
                  ),
                ),
                child: SafeArea(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const SizedBox(height: 16),
                      CircleAvatar(
                        radius: 44,
                        backgroundColor: Colors.white,
                        child: Text(inicial,
                            style: TextStyle(
                                fontSize: 36,
                                fontWeight: FontWeight.bold,
                                color: Theme.of(context).colorScheme.primary)),
                      ),
                      const SizedBox(height: 10),
                      Text(usuario.nome,
                          style: const TextStyle(
                              color: Colors.white,
                              fontSize: 20,
                              fontWeight: FontWeight.bold)),
                      Text(usuario.email,
                          style: const TextStyle(color: Colors.white70, fontSize: 13)),
                    ],
                  ),
                ),
              ),
            ),
          ),

          SliverPadding(
            padding: const EdgeInsets.all(16),
            sliver: SliverList(
              delegate: SliverChildListDelegate([
                // Card — informações da conta
                Card(
                  child: Column(
                    children: [
                      ListTile(
                        leading: const Icon(Icons.badge_outlined),
                        title: const Text('Função'),
                        trailing: Chip(
                          label: Text(usuario.roles.map((r) => r.name).join(', ')),
                        ),
                      ),
                      const Divider(height: 1),
                      ListTile(
                        leading: const Icon(Icons.email_outlined),
                        title: const Text('E-mail'),
                        trailing: Text(usuario.email,
                            style: const TextStyle(color: Colors.grey)),
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 12),

                // Card — ações de navegação
                Card(
                  child: Column(
                    children: [
                      ListTile(
                        leading: const Icon(Icons.edit_outlined),
                        title: const Text('Editar perfil'),
                        trailing: const Icon(Icons.chevron_right),
                        onTap: () => context.push('/perfil/editar'),
                      ),
                      const Divider(height: 1),
                      ListTile(
                        leading: const Icon(Icons.lock_outline),
                        title: const Text('Alterar senha'),
                        trailing: const Icon(Icons.chevron_right),
                        onTap: () => context.push('/perfil/senha'),
                      ),
                      const Divider(height: 1),
                      ListTile(
                        leading: const Icon(Icons.notifications_outlined),
                        title: const Text('Notificações'),
                        trailing: const Icon(Icons.chevron_right),
                        onTap: () => context.push('/perfil/notificacoes'),
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 24),

                // Botão de logout
                SizedBox(
                  width: double.infinity,
                  child: OutlinedButton.icon(
                    onPressed: () async {
                      await ref.read(authStateProvider.notifier).logout();
                    },
                    icon: const Icon(Icons.logout, color: Colors.red),
                    label: const Text('Sair da conta',
                        style: TextStyle(color: Colors.red)),
                    style: OutlinedButton.styleFrom(
                        side: const BorderSide(color: Colors.red)),
                  ),
                ),
                const SizedBox(height: 16),
              ]),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

#### Dashboard com Métricas

Grade 2×2 de cards de resumo seguida de lista de atividades recentes, tudo dentro de um `ListView` único com `RefreshIndicator`.

```dart
// lib/features/dashboard/presentation/dashboard_screen.dart
class DashboardScreen extends ConsumerWidget {
  const DashboardScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final usuario = ref.watch(authStateProvider).valueOrNull;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard'),
        actions: [
          IconButton(
              icon: const Icon(Icons.notifications_outlined), onPressed: () {}),
        ],
      ),
      body: RefreshIndicator(
        onRefresh: () async => ref.invalidate(produtosListProvider),
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            // Saudação
            Text(
              'Bom dia${usuario != null ? ', ${usuario.nome.split(' ').first}' : ''}!',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 4),
            Text('Aqui está o resumo de hoje.',
                style: Theme.of(context).textTheme.bodyMedium?.copyWith(color: Colors.grey)),
            const SizedBox(height: 20),

            // Grade de métricas 2×2
            GridView.count(
              shrinkWrap: true,
              physics: const NeverScrollableScrollPhysics(),
              crossAxisCount: 2,
              mainAxisSpacing: 12,
              crossAxisSpacing: 12,
              childAspectRatio: 1.5,
              children: const [
                _CardMetrica(titulo: 'Vendas hoje',   valor: 'R\$ 4.280', icone: Icons.trending_up,    cor: Colors.green),
                _CardMetrica(titulo: 'Pedidos',       valor: '34',        icone: Icons.receipt_long,   cor: Colors.blue),
                _CardMetrica(titulo: 'Clientes',      valor: '128',       icone: Icons.people_outline, cor: Colors.orange),
                _CardMetrica(titulo: 'Estoque baixo', valor: '7 itens',   icone: Icons.warning_amber,  cor: Colors.red),
              ],
            ),
            const SizedBox(height: 24),

            // Cabeçalho da seção de atividades
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text('Atividades recentes',
                    style: Theme.of(context).textTheme.titleMedium),
                TextButton(
                  onPressed: () => context.push('/pedidos'),
                  child: const Text('Ver todos'),
                ),
              ],
            ),
            const SizedBox(height: 8),

            // Lista de atividades dentro de Card
            Card(
              child: ListView.separated(
                shrinkWrap: true,
                physics: const NeverScrollableScrollPhysics(),
                itemCount: _atividadesMock.length,
                separatorBuilder: (_, __) => const Divider(height: 1),
                itemBuilder: (_, i) {
                  final a = _atividadesMock[i];
                  return ListTile(
                    leading: CircleAvatar(
                      backgroundColor: a.cor.withOpacity(0.12),
                      child: Icon(a.icone, color: a.cor, size: 20),
                    ),
                    title: Text(a.descricao),
                    subtitle: Text(a.tempo,
                        style: const TextStyle(fontSize: 12, color: Colors.grey)),
                    trailing: a.valor != null
                        ? Text(a.valor!,
                            style: TextStyle(
                                color: a.cor, fontWeight: FontWeight.bold))
                        : null,
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class _CardMetrica extends StatelessWidget {
  final String titulo;
  final String valor;
  final IconData icone;
  final Color cor;

  const _CardMetrica(
      {required this.titulo,
      required this.valor,
      required this.icone,
      required this.cor});

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(14),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Flexible(
                    child: Text(titulo,
                        style: Theme.of(context).textTheme.bodySmall,
                        overflow: TextOverflow.ellipsis)),
                Icon(icone, color: cor, size: 18),
              ],
            ),
            const Spacer(),
            Text(valor,
                style: Theme.of(context)
                    .textTheme
                    .titleLarge
                    ?.copyWith(fontWeight: FontWeight.bold)),
          ],
        ),
      ),
    );
  }
}

// Dados de exemplo
class _AtividadeMock {
  final String descricao, tempo;
  final String? valor;
  final IconData icone;
  final Color cor;
  const _AtividadeMock(
      {required this.descricao,
      required this.tempo,
      this.valor,
      required this.icone,
      required this.cor});
}

const _atividadesMock = [
  _AtividadeMock(descricao: 'Pedido #4821 aprovado', tempo: 'Há 5 min', valor: '+R\$ 320', icone: Icons.check_circle, cor: Colors.green),
  _AtividadeMock(descricao: 'Novo cliente cadastrado', tempo: 'Há 18 min', icone: Icons.person_add, cor: Colors.blue),
  _AtividadeMock(descricao: 'Pedido #4820 cancelado', tempo: 'Há 42 min', valor: '-R\$ 85', icone: Icons.cancel, cor: Colors.red),
  _AtividadeMock(descricao: 'Produto "Notebook" atualizado', tempo: 'Há 1h', icone: Icons.edit, cor: Colors.orange),
];
```

---

#### Navegação por Abas — TabBar

Múltiplas abas dentro da mesma tela com `DefaultTabController`.

```dart
// lib/features/catalogo/presentation/catalogo_screen.dart
class CatalogoScreen extends StatelessWidget {
  const CatalogoScreen({super.key});

  static const _abas = [
    (rotulo: 'Todos',        icone: Icons.grid_view,              filtro: null),
    (rotulo: 'Disponíveis',  icone: Icons.check_circle_outline,   filtro: 'disponivel'),
    (rotulo: 'Promoções',    icone: Icons.local_offer_outlined,   filtro: 'promocao'),
  ];

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: _abas.length,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Catálogo'),
          actions: [
            IconButton(icon: const Icon(Icons.search), onPressed: () {}),
          ],
          bottom: TabBar(
            tabs: _abas
                .map((a) => Tab(text: a.rotulo, icon: Icon(a.icone)))
                .toList(),
          ),
        ),
        body: TabBarView(
          children: _abas
              .map((a) => _ListaAba(filtro: a.filtro))
              .toList(),
        ),
      ),
    );
  }
}

class _ListaAba extends ConsumerWidget {
  final String? filtro;
  const _ListaAba({this.filtro});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final produtosAsync = ref.watch(produtosListProvider);

    return produtosAsync.when(
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (e, _) => Center(child: Text('Erro: $e')),
      data: (todos) {
        final lista = switch (filtro) {
          'disponivel' => todos.where((p) => p.disponivel).toList(),
          'promocao'   => todos.where((p) => p.emPromocao).toList(),
          _            => todos,
        };

        if (lista.isEmpty) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(Icons.inventory_2_outlined, size: 64, color: Colors.grey.shade400),
                const SizedBox(height: 12),
                const Text('Nenhum produto nesta categoria.'),
              ],
            ),
          );
        }

        return RefreshIndicator(
          onRefresh: () => ref.refresh(produtosListProvider.future),
          child: ListView.builder(
            padding: const EdgeInsets.symmetric(vertical: 8),
            itemCount: lista.length,
            itemBuilder: (_, i) => CardProduto(
              produto: lista[i],
              onTap: () => context.goNamed('produto-detalhe',
                  pathParameters: {'id': lista[i].id.toString()}),
            ),
          ),
        );
      },
    );
  }
}
```

---

## 2. Navegação

O Flutter não tem uma solução única de navegação — a escolha depende do tamanho do projeto e das necessidades de injeção de dependências:

| | GoRouter | flutter_modular |
|---|---|---|
| Mantenedor | Flutter team (Google) | Flutterando (comunidade) |
| Escopo | Somente navegação | Navegação + injeção de dependências |
| Integração oficial | Sim | Não |
| Estado global | Via Riverpod / Bloc externo | DI embutido (+ Bloc / MobX) |
| Modularização | Manual por feature | `Module` nativo |
| Curva de aprendizado | Baixa | Média |
| Indicado para | Apps novos, equipes já com Riverpod | Apps grandes com DI centralizada |

---

### GoRouter

Solução de roteamento oficial recomendada pelo Flutter team. Suporta deep linking, navegação declarativa, guards e rotas aninhadas. Integra diretamente com Riverpod (ver [seção 3](#3-estado-global-com-riverpod)).

```bash
flutter pub add go_router
```

#### Configuração do GoRouter

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

#### Rotas Aninhadas e Parâmetros

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

#### Rotas Protegidas

O `redirect` no GoRouter (veja seção anterior) já protege rotas globalmente. Para proteção por role, veja a [seção 7](#7-autenticação-jwt-e-controle-de-acesso-por-role).

---

#### Navegação Programática

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

#### Deep Linking

Deep linking permite abrir o app diretamente em uma tela específica a partir de uma URL externa — seja por um link em e-mail, notificação push, QR code ou outro app. O GoRouter trata deep links nativamente: basta que o path da URL corresponda a uma rota registrada.

Existem dois tipos de deep link:

| Tipo | Exemplo de URL | Quando usar |
|---|---|---|
| **Custom scheme** | `meuapp://produtos/42` | Links internos, testes, integrações simples |
| **App Links (Android) / Universal Links (iOS)** | `https://meuapp.com/produtos/42` | Links compartilháveis, SEO, fallback para web |

---

**Configuração no Android**

Edite `android/app/src/main/AndroidManifest.xml` e adicione um `<intent-filter>` dentro da `<activity>` principal:

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<activity
    android:name=".MainActivity"
    android:exported="true"
    ...>

    <!-- Custom scheme: meuapp://... -->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="meuapp" />
    </intent-filter>

    <!-- App Links (HTTPS): https://meuapp.com/... -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:scheme="https"
            android:host="meuapp.com" />
    </intent-filter>
</activity>
```

> **App Links (HTTPS):** exige que o arquivo `/.well-known/assetlinks.json` esteja publicado no seu domínio com o fingerprint SHA-256 do certificado de assinatura do app. Sem isso, o Android abre o navegador em vez do app.

---

**Configuração no iOS**

Edite `ios/Runner/Info.plist` para registrar o custom scheme:

```xml
<!-- ios/Runner/Info.plist -->
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>meuapp</string>
        </array>
    </dict>
</array>
```

Para Universal Links (HTTPS), adicione o domínio associado em *Xcode > Runner > Signing & Capabilities > Associated Domains*:

```
applinks:meuapp.com
```

E publique o arquivo `/.well-known/apple-app-site-association` no seu servidor:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAM_ID.com.empresa.meuapp",
        "paths": ["/produtos/*", "/pedidos/*"]
      }
    ]
  }
}
```

---

**Configuração no GoRouter**

O GoRouter já intercepta deep links automaticamente quando `MaterialApp.router` é usado — nenhuma configuração adicional é necessária no Dart. Basta garantir que os paths das rotas coincidam com o path da URL recebida:

```dart
// lib/routes/app_router.dart
GoRouter(
  initialLocation: '/produtos',
  // deepLinkBuilder é opcional — útil para transformar a URL antes do roteamento
  redirect: (context, state) {
    final logado = authState.valueOrNull?.token != null;
    final naLogin = state.matchedLocation == '/login';
    if (!logado && !naLogin) return '/login?redirect=${state.uri}';
    if (logado && naLogin) return '/produtos';
    return null;
  },
  routes: [
    GoRoute(
      path: '/produtos',
      name: 'produtos',
      builder: (_, __) => const ProdutosListScreen(),
      routes: [
        GoRoute(
          path: ':id',
          name: 'produto-detalhe',
          builder: (_, state) {
            final id = state.pathParameters['id']!;
            return ProdutoDetalheScreen(id: id);
          },
        ),
      ],
    ),
    GoRoute(
      path: '/pedidos/:numero',
      name: 'pedido-detalhe',
      builder: (_, state) {
        final numero = state.pathParameters['numero']!;
        // Query parameters opcionais: ?origem=notificacao
        final origem = state.uri.queryParameters['origem'];
        return PedidoDetalheScreen(numero: numero, origem: origem);
      },
    ),
  ],
)
```

**Mapeamento de URLs para rotas:**

| URL recebida | Rota ativada | Parâmetros |
|---|---|---|
| `meuapp://produtos` | `/produtos` | — |
| `meuapp://produtos/42` | `/produtos/:id` | `id = "42"` |
| `https://meuapp.com/pedidos/PED-001?origem=email` | `/pedidos/:numero` | `numero = "PED-001"`, `origem = "email"` |

---

**Redirecionamento pós-autenticação**

Quando um deep link chega para uma rota protegida e o usuário não está autenticado, salve a URL alvo para redirecionar após o login:

```dart
// lib/routes/app_router.dart
redirect: (context, state) {
  final logado = authState.valueOrNull?.token != null;
  final naLogin = state.matchedLocation == '/login';

  if (!logado && !naLogin) {
    // preserva a URL de destino como query parameter
    final destino = Uri.encodeComponent(state.uri.toString());
    return '/login?redirect=$destino';
  }
  if (logado && naLogin) {
    // após login, redireciona para a URL original ou para /produtos
    final redirect = state.uri.queryParameters['redirect'];
    return redirect != null ? Uri.decodeComponent(redirect) : '/produtos';
  }
  return null;
},
```

---

**Testando Deep Links**

**Android — via ADB:**

```bash
# Custom scheme
adb shell am start \
  -a android.intent.action.VIEW \
  -d "meuapp://produtos/42" \
  com.empresa.meuapp

# HTTPS (App Link)
adb shell am start \
  -a android.intent.action.VIEW \
  -d "https://meuapp.com/produtos/42" \
  com.empresa.meuapp
```

**iOS — via xcrun:**

```bash
# Custom scheme (simulador)
xcrun simctl openurl booted "meuapp://produtos/42"

# Universal Link (simulador)
xcrun simctl openurl booted "https://meuapp.com/produtos/42"
```

**Flutter — durante o desenvolvimento:**

```bash
# Acionar deep link no dispositivo/emulador conectado
flutter run
# Em outro terminal:
flutter attach  # se já estiver rodando
```

> **Dica:** use o pacote `uni_links` ou `app_links` se precisar capturar deep links de forma programática (ex.: para analytics) ou processar links recebidos enquanto o app já está em primeiro plano.

---

### flutter_modular

Pacote da comunidade (Flutterando) que combina **navegação declarativa** e **injeção de dependências** em um único sistema modular. Cada feature é encapsulada em um `Module` que registra suas próprias rotas e binds, tornando o app naturalmente extensível e com acoplamento reduzido entre features.

```bash
flutter pub add flutter_modular
```

#### Configuração do flutter_modular

O ponto de entrada usa `ModularApp`, que injeta o sistema de módulos acima do `MaterialApp`:

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';
import 'app_module.dart';
import 'app_widget.dart';

void main() {
  runApp(ModularApp(module: AppModule(), child: const AppWidget()));
}
```

```dart
// lib/app_widget.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';

class AppWidget extends StatelessWidget {
  const AppWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Meu App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      routerConfig: Modular.routerConfig,
    );
  }
}
```

O módulo raiz (`AppModule`) registra as rotas de primeiro nível, delega features para submódulos e expõe binds globais:

```dart
// lib/app_module.dart
import 'package:flutter_modular/flutter_modular.dart';
import 'core/network/dio_client.dart';
import 'core/guards/auth_guard.dart';
import 'features/auth/auth_module.dart';
import 'features/produtos/produtos_module.dart';
import 'features/admin/admin_module.dart';

class AppModule extends Module {
  @override
  void exportedBinds(Injector i) {
    // binds disponíveis em todos os submódulos
    i.addSingleton<DioClient>(DioClient.new);
  }

  @override
  void routes(RouteManager r) {
    r.module('/auth', module: AuthModule());
    r.module('/produtos', module: ProdutosModule(), guards: [AuthGuard()]);
    r.module('/admin', module: AdminModule(), guards: [AuthGuard(), AdminGuard()]);
    r.redirect('/', to: '/produtos/');
  }
}
```

#### Módulos e Rotas

Cada feature tem seu próprio `Module` com rotas e dependências encapsuladas. O módulo não sabe de nada fora de si — apenas recebe binds exportados do pai via `exportedBinds`.

```dart
// lib/features/produtos/produtos_module.dart
import 'package:flutter_modular/flutter_modular.dart';
import 'data/produto_repository.dart';
import 'presentation/produtos_list_screen.dart';
import 'presentation/produto_detalhe_screen.dart';
import 'presentation/produto_form_screen.dart';

class ProdutosModule extends Module {
  @override
  void binds(Injector i) {
    // singleton scoped ao módulo — destruído quando o módulo for descarregado
    i.addSingleton<ProdutoRepository>(ProdutoRepository.new);
  }

  @override
  void routes(RouteManager r) {
    r.child('/', child: (_) => const ProdutosListScreen());
    r.child('/:id', child: (_) => ProdutoDetalheScreen(
      id: Modular.args.params['id']!,
    ));
    r.child('/novo', child: (_) => const ProdutoFormScreen());
    r.child('/:id/editar', child: (_) => ProdutoFormScreen(
      id: Modular.args.params['id'],
    ));
  }
}
```

```dart
// lib/features/auth/auth_module.dart
import 'package:flutter_modular/flutter_modular.dart';
import 'data/auth_repository.dart';
import 'presentation/login_screen.dart';

class AuthModule extends Module {
  @override
  void binds(Injector i) {
    i.addSingleton<AuthRepository>(AuthRepository.new);
  }

  @override
  void routes(RouteManager r) {
    r.child('/', child: (_) => const LoginScreen());
  }
}
```

**Lendo parâmetros e query strings na tela de destino:**

```dart
// path parameter — /produtos/:id
final id = Modular.args.params['id'];

// query parameter — /produtos?categoria=eletronicos
final categoria = Modular.args.queryParams['categoria'];

// dados passados programaticamente (não visíveis na URL)
final dados = Modular.args.data as MeuObjeto?;
```

#### Guards — Rotas Protegidas

`RouteGuard` intercepta a navegação antes de ativar a rota. Se `canActivate` retornar `false`, redireciona para `redirectTo`:

```dart
// lib/core/guards/auth_guard.dart
import 'package:flutter_modular/flutter_modular.dart';
import '../storage/secure_storage.dart';

class AuthGuard extends RouteGuard {
  AuthGuard() : super(redirectTo: '/auth/');

  @override
  Future<bool> canActivate(String path, ModularRoute route) async {
    final storage = Modular.get<SecureStorage>();
    final token = await storage.lerToken();
    return token != null;
  }
}
```

```dart
// Guard de role — verifica permissão de admin
class AdminGuard extends RouteGuard {
  AdminGuard() : super(redirectTo: '/produtos/');

  @override
  Future<bool> canActivate(String path, ModularRoute route) async {
    final repo = Modular.get<AuthRepository>();
    final roles = await repo.obterRoles();
    return roles.contains('ROLE_ADMIN');
  }
}
```

Guards podem ser aplicados no `AppModule` (para módulos inteiros) ou em rotas individuais dentro de um módulo:

```dart
// AppModule — guard no módulo inteiro
r.module('/admin', module: AdminModule(), guards: [AuthGuard(), AdminGuard()]);

// Dentro de um módulo — guard em rota específica
r.child('/configuracoes', child: (_) => const ConfigScreen(), guards: [AdminGuard()]);
```

#### Injeção de Dependências

O sistema de DI do flutter_modular gerencia o ciclo de vida das dependências por módulo, eliminando a necessidade de providers externos para infraestrutura:

```dart
// Tipos de registro disponíveis no Injector
@override
void binds(Injector i) {
  i.addSingleton<DioClient>(DioClient.new);               // instância única, vive enquanto o módulo existir
  i.addLazySingleton<RelatorioService>(RelatorioService.new); // singleton criado na primeira chamada
  i.add<ProdutoBloc>(ProdutoBloc.new);                    // nova instância a cada Modular.get<>()
}
```

```dart
// Obtendo dependências em qualquer ponto do app
final repo = Modular.get<ProdutoRepository>();

// Em um widget StatelessWidget
class ProdutosListScreen extends StatelessWidget {
  ProdutoRepository get _repo => Modular.get();

  @override
  Widget build(BuildContext context) { ... }
}

// Em um StatefulWidget ou serviço
class ProdutoBloc {
  final _repo = Modular.get<ProdutoRepository>();
}
```

> **flutter_modular + Riverpod:** é possível usar ambos juntos — flutter_modular gerencia DI de infraestrutura (repositórios, clientes HTTP) e Riverpod gerencia o estado reativo de UI. No entanto, em muitos projetos o flutter_modular é combinado com **Bloc** ou **MobX** para estado local, dispensando o Riverpod.

#### Navegação Programática — flutter_modular

```dart
// Navegar substituindo o histórico (equivalente a context.go no GoRouter)
Modular.to.navigate('/produtos/');

// Empilhar rota — mantém botão voltar (equivalente a context.push)
Modular.to.pushNamed('/produtos/42');

// Empilhar e aguardar resultado
final resultado = await Modular.to.pushNamed<bool>('/produtos/novo');

// Voltar
Modular.to.pop();

// Voltar com resultado
Modular.to.pop(true);

// Voltar até uma rota específica
Modular.to.popUntil(ModalRoute.withName('/produtos/'));

// Passar dados sem expor na URL
Modular.to.pushNamed('/confirmacao', arguments: pedido);
// Na tela de destino:
final pedido = Modular.args.data as Pedido;
```

#### Deep Linking — flutter_modular

O flutter_modular integra com `MaterialApp.router` via `Modular.routerConfig` e trata automaticamente os deep links recebidos **na inicialização** do app. A configuração nativa no Android e iOS é idêntica à descrita na [seção de Deep Linking do GoRouter](#deep-linking).

Para capturar deep links enquanto o app já está aberto (foreground / background), o flutter_modular não fornece esse mecanismo por conta própria — é necessário combinar com o pacote **`app_links`**.

| Cenário | Quem trata |
|---|---|
| App **fechado**, aberto via link | `Modular.routerConfig` (automático) |
| App **aberto/retomado**, link chegando | `app_links` (listener manual) |

**Instalação do app_links:**

```bash
flutter pub add app_links
```

```yaml
# pubspec.yaml
dependencies:
  app_links: ^6.3.3
```

---

**Integração com AppWidget**

O padrão recomendado é transformar `AppWidget` em `StatefulWidget` e inicializar o listener de deep links em `initState`:

```dart
// lib/app_widget.dart
import 'dart:async';
import 'package:app_links/app_links.dart';
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';

class AppWidget extends StatefulWidget {
  const AppWidget({super.key});

  @override
  State<AppWidget> createState() => _AppWidgetState();
}

class _AppWidgetState extends State<AppWidget> {
  final _appLinks = AppLinks();
  StreamSubscription<Uri>? _linkSubscription;

  @override
  void initState() {
    super.initState();
    _initDeepLinks();
  }

  Future<void> _initDeepLinks() async {
    // 1. Link inicial — app foi ABERTO via deep link (executado uma única vez)
    final initialUri = await _appLinks.getInitialLink();
    if (initialUri != null) {
      _navegarParaUri(initialUri);
    }

    // 2. Links chegando com o app já em execução (foreground / background)
    _linkSubscription = _appLinks.uriLinkStream.listen(
      _navegarParaUri,
      onError: (err) => debugPrint('Erro ao processar deep link: $err'),
    );
  }

  void _navegarParaUri(Uri uri) {
    // Custom scheme: meuapp://produtos/42
    //   uri.host = "produtos", uri.path = "/42"  →  /produtos/42
    // HTTPS: https://meuapp.com/produtos/42
    //   uri.path = "/produtos/42"                 →  /produtos/42
    final basePath = uri.scheme.startsWith('http')
        ? uri.path
        : '/${uri.host}${uri.path}';

    final pathFinal = uri.queryParameters.isNotEmpty
        ? '$basePath?${uri.query}'
        : basePath;

    Modular.to.navigate(pathFinal);
  }

  @override
  void dispose() {
    _linkSubscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Meu App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      routerConfig: Modular.routerConfig,
    );
  }
}
```

> **Por que cancelar o subscription em `dispose`?** O `AppWidget` vive durante toda a sessão do app, mas em testes ou hot restart o `dispose` é chamado — deixar o stream aberto causaria múltiplos listeners ativos e navegações duplicadas.

---

**Tratamento de deep link com autenticação**

Quando o link aponta para uma rota protegida e o usuário não está autenticado, o `AuthGuard` do flutter_modular já intercepta e redireciona para o login. Para redirecionar de volta após o login, passe o path original como dado:

```dart
void _navegarParaUri(Uri uri) {
  final path = uri.scheme.startsWith('http')
      ? uri.path
      : '/${uri.host}${uri.path}';

  final estaLogado = Modular.get<AuthRepository>().isAuthenticated();

  if (!estaLogado) {
    // redireciona para login passando o destino original como dado
    Modular.to.navigate('/auth/', arguments: path);
    return;
  }

  Modular.to.navigate(path);
}
```

```dart
// lib/features/auth/presentation/login_screen.dart
// após login bem-sucedido:
final destino = Modular.args.data as String?;
Modular.to.navigate(destino ?? '/produtos/');
```

---

**Mapeamento de URLs para módulos e rotas:**

| URL recebida | Módulo ativado | Rota no módulo |
|---|---|---|
| `meuapp://produtos/` | `ProdutosModule` | `/` |
| `meuapp://produtos/42` | `ProdutosModule` | `/:id` (id = "42") |
| `meuapp://produtos/42?origem=email` | `ProdutosModule` | `/:id` + query `origem` |
| `meuapp://admin/` | `AdminModule` (passa pelo `AdminGuard`) | `/` |
| `https://meuapp.com/produtos/42` | `ProdutosModule` | `/:id` (id = "42") |

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

### Separação de Responsabilidades

O padrão consolidado no ecossistema Flutter é a **arquitetura em três camadas por feature**, refletida diretamente na estrutura de pastas adotada neste guia:

```
feature/produtos/
├── data/           ← integração com backend
│   └── produto_repository.dart
├── domain/         ← modelos e contratos
│   └── produto_model.dart
└── presentation/   ← tela e estado de UI
    ├── produtos_list_screen.dart
    └── produtos_provider.dart
```

**`data/`** contém o `Repository`. Responsabilidades: fazer a chamada HTTP, converter JSON → modelo e traduzir exceções de rede para erros de domínio. Nunca conhece widgets, `BuildContext` ou `SnackBar`.

**`domain/`** contém modelos de dados e, opcionalmente, interfaces abstratas do repositório. Apenas Dart puro — sem dependência de `dio` ou `flutter`. Isso permite trocar a implementação HTTP sem alterar nada acima.

**`presentation/`** divide-se em dois arquivos com papéis distintos:

- **Provider** (`produtos_provider.dart`) — ponte entre repository e tela: chama o repository, gerencia os estados `loading / data / error` via `AsyncNotifier`. Não constrói widgets nem chama `showSnackBar` diretamente.
- **Screen** (`produtos_list_screen.dart`) — renderiza o estado lido do provider via `ref.watch`, exibe `SnackBar`, `AlertDialog` e navega com `context.go`. Não instancia Dio nem faz chamadas HTTP diretamente.

**Tabela de responsabilidades:**

| Responsabilidade | Camada | Justificativa |
|---|---|---|
| Chamada HTTP | `data/repository` | Isolada e testável com mock |
| Conversão JSON ↔ modelo | `domain/model` | Sem dependência de framework |
| Tratamento de erro de rede | `data/repository` + `core/errors` | Centralizado no interceptor |
| Estado de carregamento/erro | `presentation/provider` | Riverpod gerencia `AsyncValue` |
| Exibir SnackBar / Dialog | `presentation/screen` | Requer `BuildContext` |
| Navegação após ação | `presentation/screen` | Requer `context.go` |
| Regra de negócio complexa | `domain/` (use case separado) | Testável sem UI e sem rede |

---

**O erro mais comum: efeito colateral de UI dentro do provider.** A violação mais frequente é chamar `showSnackBar` ou `context.go` de dentro do `AsyncNotifier`. O padrão correto é: o provider expõe o estado de erro; a tela observa com `ref.listen` e exibe o feedback.

```dart
// ❌ errado — provider conhece SnackBar
Future<void> salvar(Produto p) async {
  await repo.criar(p);
  ScaffoldMessenger.of(context).showSnackBar(...); // context não pertence aqui
}

// ✅ correto — provider expõe estado; tela reage
// No provider:
Future<void> salvar(Produto p) async {
  state = const AsyncLoading();
  state = await AsyncValue.guard(() => repo.criar(p));
}

// Na tela:
ref.listen(produtosListProvider, (_, next) {
  if (next is AsyncError) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(next.error.toString())),
    );
  }
});
```

---

**Quando adicionar uma camada `domain/use_case`:** a estrutura de três camadas é suficiente para a maioria dos casos. Use cases valem a pena quando a mesma lógica de negócio é compartilhada entre features, quando há orquestração entre dois ou mais repositories (ex.: criar pedido + atualizar estoque + enviar notificação) ou quando a regra é complexa o suficiente para justificar testes unitários próprios. Fora desses cenários, extrair use cases prematuramente só aumenta o número de arquivos sem benefício real.

---

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

---

## 9. WebView no Flutter

`webview_flutter` é o pacote oficial do Flutter team para exibir páginas web dentro do app. Usa `WKWebView` no iOS e `WebView` (Chromium) no Android.

### Instalação e Configuração

```bash
flutter pub add webview_flutter
```

**Android — `android/app/build.gradle`:**

```groovy
android {
    defaultConfig {
        minSdkVersion 20   // mínimo exigido pelo webview_flutter
    }
}
```

**iOS — `ios/Runner/Info.plist`** (quando carregar conteúdo de domínios HTTP sem TLS):

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

> **Dica:** em produção, prefira HTTPS e remova `NSAllowsArbitraryLoads`. Use `NSExceptionDomains` para liberar apenas os domínios necessários.

---

### WebViewController — Controlando o WebView

O `WebViewController` é o ponto central de controle. Deve ser criado **fora** do método `build` e reutilizado durante o ciclo de vida do widget.

```dart
import 'package:webview_flutter/webview_flutter.dart';

// Criação — geralmente em initState ou como campo do State
final controller = WebViewController()
  ..setJavaScriptMode(JavaScriptMode.unrestricted) // habilita JavaScript
  ..setBackgroundColor(Colors.white)
  ..loadRequest(Uri.parse('https://flutter.dev'));

// Exibição
WebViewWidget(controller: controller)
```

Métodos e propriedades principais do `WebViewController`:

| Método / Propriedade | Descrição |
|---|---|
| `setJavaScriptMode(mode)` | `JavaScriptMode.unrestricted` ou `disabled` |
| `setBackgroundColor(color)` | Cor de fundo antes da página carregar |
| `setUserAgent(ua)` | Sobrescreve o User-Agent |
| `loadRequest(uri, {headers})` | Carrega uma URL |
| `loadHtmlString(html, {baseUrl})` | Carrega HTML diretamente |
| `loadFlutterAsset(key)` | Carrega arquivo dos assets do Flutter |
| `loadFile(absoluteFilePath)` | Carrega arquivo do sistema de arquivos |
| `reload()` | Recarrega a página atual |
| `goBack()` | Navega para a página anterior |
| `goForward()` | Navega para a próxima página |
| `canGoBack()` | `Future<bool>` — verifica se pode voltar |
| `canGoForward()` | `Future<bool>` — verifica se pode avançar |
| `currentUrl()` | `Future<String?>` — URL atual |
| `getTitle()` | `Future<String?>` — título da página atual |
| `scrollTo(x, y)` | Rola para posição absoluta |
| `scrollBy(x, y)` | Rola relativamente à posição atual |
| `getScrollPosition()` | `Future<Offset>` — posição atual de rolagem |
| `runJavaScript(js)` | Executa JS sem retorno |
| `runJavaScriptReturningResult(js)` | Executa JS e retorna o resultado |
| `addJavaScriptChannel(name, onReceived)` | Cria canal de comunicação JS → Flutter |
| `setNavigationDelegate(delegate)` | Define o delegate de navegação |
| `enableZoom(flag)` | Habilita ou desabilita zoom por gesto |
| `clearCache()` | Limpa o cache do WebView |
| `clearLocalStorage()` | Limpa o localStorage |

---

### Carregamento de Conteúdo

```dart
// URL remota com cabeçalhos customizados
controller.loadRequest(
  Uri.parse('https://meuapp.com.br/relatorio'),
  headers: {'Authorization': 'Bearer $token'},
);

// HTML inline — útil para conteúdo gerado no app (recibos, termos)
controller.loadHtmlString('''
  <!DOCTYPE html>
  <html lang="pt-BR">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
      body { font-family: sans-serif; padding: 16px; }
      h1 { color: #3F51B5; }
    </style>
  </head>
  <body>
    <h1>Recibo #${pedido.id}</h1>
    <p>Total: R\$ ${pedido.total.toStringAsFixed(2)}</p>
  </body>
  </html>
''');

// Asset local — arquivo em assets/web/index.html registrado no pubspec.yaml
controller.loadFlutterAsset('assets/web/index.html');
```

> **Dica:** ao carregar HTML com `loadHtmlString`, passe `baseUrl` para que recursos relativos (imagens, CSS) sejam resolvidos corretamente: `loadHtmlString(html, baseUrl: Uri.parse('https://meuapp.com'))`.

---

### NavigationDelegate — Interceptando Navegação

O `NavigationDelegate` permite interceptar e cancelar navegações, além de reagir a eventos do ciclo de vida da página.

```dart
controller.setNavigationDelegate(
  NavigationDelegate(
    // Intercepta toda tentativa de navegação
    onNavigationRequest: (NavigationRequest request) {
      // Bloquear links externos — abrir no navegador do sistema
      if (!request.url.startsWith('https://meuapp.com')) {
        launchUrl(Uri.parse(request.url)); // pacote url_launcher
        return NavigationDecision.prevent;
      }
      return NavigationDecision.navigate;
    },

    onPageStarted: (String url) {
      setState(() => _carregando = true);
    },

    onPageFinished: (String url) {
      setState(() => _carregando = false);
    },

    onProgress: (int progresso) {
      setState(() => _progresso = progresso / 100);
    },

    onWebResourceError: (WebResourceError error) {
      debugPrint('Erro WebView: ${error.description} (${error.errorCode})');
    },

    // iOS — permissão para abrir links em nova aba
    onUrlChange: (UrlChange change) {
      debugPrint('URL mudou para: ${change.url}');
    },
  ),
);
```

| Callback | Quando dispara |
|---|---|
| `onNavigationRequest` | Antes de cada navegação; retorne `prevent` para cancelar |
| `onPageStarted` | Quando a página começa a carregar |
| `onPageFinished` | Quando a página terminou de carregar |
| `onProgress` | A cada atualização de progresso (0–100) |
| `onWebResourceError` | Quando um recurso falha ao carregar |
| `onUrlChange` | Quando a URL muda (ex.: SPA com hash routing) |
| `onHttpError` | Quando a resposta HTTP tem código de erro (4xx/5xx) |

---

### Avaliação de JavaScript

```dart
// Executar JS sem retorno — manipular o DOM, disparar eventos
await controller.runJavaScript(
  "document.querySelector('#botao-aceitar').click();",
);

// Executar JS com retorno — ler dados da página
final titulo = await controller.runJavaScriptReturningResult(
  'document.title',
) as String;

// Ler valor de um campo de formulário
final email = await controller.runJavaScriptReturningResult(
  "document.getElementById('email').value",
) as String;

// Injetar CSS dinamicamente
await controller.runJavaScript('''
  var style = document.createElement("style");
  style.textContent = "body { background: #F5F5F5 !important; }";
  document.head.appendChild(style);
''');

// Passar dados do Flutter para a página via JS
final jsonDados = jsonEncode({'usuario': 'Ana', 'role': 'admin'});
await controller.runJavaScript('window.receberDados($jsonDados)');
```

> **Atenção:** `runJavaScriptReturningResult` retorna `Object` (pode ser `String`, `num` ou `bool`). Faça o cast explicitamente. Strings são retornadas **com aspas** no Android — use `json.decode` para normalizar: `json.decode(resultado as String)`.

---

### Canais JavaScript — Comunicação Flutter ↔ JS

Para receber dados enviados pela página web, registre um `JavaScriptChannel`. O nome do canal fica disponível como objeto global dentro da página.

```dart
// Flutter — registrar o canal antes de carregar a URL
controller
  ..addJavaScriptChannel(
    'FlutterChannel',               // nome acessível no JS como window.FlutterChannel
    onMessageReceived: (JavaScriptMessage mensagem) {
      final dados = jsonDecode(mensagem.message);
      _processarPagamento(dados);
    },
  )
  ..loadRequest(Uri.parse('https://meuapp.com/checkout'));
```

```html
<!-- Página web — enviando mensagem para o Flutter -->
<script>
  function confirmarPedido() {
    const payload = JSON.stringify({
      pedidoId: 42,
      total: 199.90,
      metodoPagamento: 'cartao'
    });

    // Disponível em Android e iOS
    if (window.FlutterChannel) {
      window.FlutterChannel.postMessage(payload);
    }
  }
</script>
<button onclick="confirmarPedido()">Confirmar Pedido</button>
```

> **Múltiplos canais:** registre quantos canais precisar, cada um com um nome distinto. Isso separa responsabilidades: um canal para autenticação, outro para eventos de navegação, outro para dados de formulário.

---

### Indicador de Progresso e Tratamento de Erros

```dart
// lib/features/webview/presentation/webview_screen.dart
class WebViewScreen extends StatefulWidget {
  final String url;
  final String titulo;
  const WebViewScreen({super.key, required this.url, required this.titulo});

  @override
  State<WebViewScreen> createState() => _WebViewScreenState();
}

class _WebViewScreenState extends State<WebViewScreen> {
  late final WebViewController _controller;
  double _progresso = 0;
  bool _carregando = true;
  String? _erro;

  @override
  void initState() {
    super.initState();
    _controller = WebViewController()
      ..setJavaScriptMode(JavaScriptMode.unrestricted)
      ..setNavigationDelegate(NavigationDelegate(
        onProgress: (p) => setState(() => _progresso = p / 100),
        onPageStarted: (_) => setState(() {
          _carregando = true;
          _erro = null;
        }),
        onPageFinished: (_) => setState(() => _carregando = false),
        onWebResourceError: (error) {
          // Filtra erros de sub-recursos (iframes, imagens) — reporta só o erro principal
          if (error.isForMainFrame ?? true) {
            setState(() {
              _carregando = false;
              _erro = 'Não foi possível carregar a página.\n${error.description}';
            });
          }
        },
      ))
      ..loadRequest(Uri.parse(widget.url));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.titulo),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              setState(() { _erro = null; _carregando = true; });
              _controller.reload();
            },
          ),
        ],
        // Barra de progresso abaixo da AppBar
        bottom: _carregando
            ? PreferredSize(
                preferredSize: const Size.fromHeight(3),
                child: LinearProgressIndicator(value: _progresso == 0 ? null : _progresso),
              )
            : null,
      ),
      body: _erro != null
          ? Center(
              child: Padding(
                padding: const EdgeInsets.all(24),
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Icon(Icons.wifi_off, size: 64, color: Colors.grey),
                    const SizedBox(height: 16),
                    Text(_erro!, textAlign: TextAlign.center),
                    const SizedBox(height: 24),
                    FilledButton.icon(
                      onPressed: () {
                        setState(() { _erro = null; _carregando = true; });
                        _controller.reload();
                      },
                      icon: const Icon(Icons.refresh),
                      label: const Text('Tentar novamente'),
                    ),
                  ],
                ),
              ),
            )
          : WebViewWidget(controller: _controller),
    );
  }
}
```

---

### Tela Completa com WebView

Exemplo com barra de navegação (voltar/avançar), controle de progresso, interceptação de links externos e canal JavaScript para receber eventos da página.

```dart
// lib/features/webview/presentation/webview_completo_screen.dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:url_launcher/url_launcher.dart';
import 'package:webview_flutter/webview_flutter.dart';

class WebViewCompletoScreen extends StatefulWidget {
  final String url;
  final String titulo;
  final String dominioPrincipal; // ex.: 'meuapp.com.br'

  const WebViewCompletoScreen({
    super.key,
    required this.url,
    required this.titulo,
    required this.dominioPrincipal,
  });

  @override
  State<WebViewCompletoScreen> createState() => _WebViewCompletoScreenState();
}

class _WebViewCompletoScreenState extends State<WebViewCompletoScreen> {
  late final WebViewController _controller;
  double _progresso = 0;
  bool _carregando = true;
  bool _podeVoltar = false;
  bool _podeAvancar = false;
  String _tituloAtual = '';

  @override
  void initState() {
    super.initState();
    _tituloAtual = widget.titulo;

    _controller = WebViewController()
      ..setJavaScriptMode(JavaScriptMode.unrestricted)
      ..setBackgroundColor(Colors.white)
      ..addJavaScriptChannel(
        'AppChannel',
        onMessageReceived: (msg) => _tratarMensagem(msg.message),
      )
      ..setNavigationDelegate(NavigationDelegate(
        onNavigationRequest: (req) {
          // Links externos abrem no navegador
          if (!req.url.contains(widget.dominioPrincipal)) {
            launchUrl(Uri.parse(req.url), mode: LaunchMode.externalApplication);
            return NavigationDecision.prevent;
          }
          return NavigationDecision.navigate;
        },
        onProgress: (p) => setState(() => _progresso = p / 100),
        onPageStarted: (_) => setState(() => _carregando = true),
        onPageFinished: (_) async {
          final titulo = await _controller.getTitle();
          final podeVoltar = await _controller.canGoBack();
          final podeAvancar = await _controller.canGoForward();
          if (mounted) {
            setState(() {
              _carregando = false;
              _tituloAtual = titulo ?? widget.titulo;
              _podeVoltar = podeVoltar;
              _podeAvancar = podeAvancar;
            });
          }
        },
        onWebResourceError: (error) {
          if (error.isForMainFrame ?? true) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text('Erro ao carregar: ${error.description}'),
                backgroundColor: Colors.red,
                action: SnackBarAction(
                  label: 'Tentar novamente',
                  onPressed: _controller.reload,
                ),
              ),
            );
          }
        },
      ))
      ..loadRequest(Uri.parse(widget.url));
  }

  void _tratarMensagem(String mensagem) {
    try {
      final dados = jsonDecode(mensagem) as Map<String, dynamic>;
      final tipo = dados['tipo'] as String?;

      switch (tipo) {
        case 'login_sucesso':
          final token = dados['token'] as String;
          // Salvar token, atualizar estado de auth, etc.
          debugPrint('Login via WebView: $token');
        case 'fechar':
          Navigator.of(context).pop(dados['resultado']);
        default:
          debugPrint('Mensagem desconhecida: $mensagem');
      }
    } catch (e) {
      debugPrint('Erro ao processar mensagem JS: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_tituloAtual, overflow: TextOverflow.ellipsis),
        bottom: _carregando
            ? PreferredSize(
                preferredSize: const Size.fromHeight(3),
                child: LinearProgressIndicator(
                  value: _progresso == 0 ? null : _progresso,
                  backgroundColor: Colors.transparent,
                ),
              )
            : null,
      ),
      body: WebViewWidget(controller: _controller),

      // Barra de navegação inferior
      bottomNavigationBar: SafeArea(
        child: Container(
          height: 48,
          decoration: BoxDecoration(
            color: Theme.of(context).colorScheme.surface,
            border: Border(top: BorderSide(color: Colors.grey.shade200)),
          ),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              IconButton(
                icon: const Icon(Icons.arrow_back_ios, size: 20),
                onPressed: _podeVoltar ? _controller.goBack : null,
                tooltip: 'Voltar',
              ),
              IconButton(
                icon: const Icon(Icons.arrow_forward_ios, size: 20),
                onPressed: _podeAvancar ? _controller.goForward : null,
                tooltip: 'Avançar',
              ),
              IconButton(
                icon: const Icon(Icons.refresh, size: 20),
                onPressed: _controller.reload,
                tooltip: 'Recarregar',
              ),
              IconButton(
                icon: const Icon(Icons.open_in_browser, size: 20),
                tooltip: 'Abrir no navegador',
                onPressed: () async {
                  final url = await _controller.currentUrl();
                  if (url != null) {
                    launchUrl(Uri.parse(url), mode: LaunchMode.externalApplication);
                  }
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Uso da tela:**

```dart
// Abrir uma URL qualquer dentro do app
context.push(
  '/webview',
  extra: const WebViewCompletoScreen(
    url: 'https://meuapp.com.br/politica-privacidade',
    titulo: 'Política de Privacidade',
    dominioPrincipal: 'meuapp.com.br',
  ),
);

// Abrir conteúdo HTML gerado dinamicamente
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (_) {
      final screen = WebViewScreen(
        url: '',       // não usado
        titulo: 'Recibo',
      );
      // controller.loadHtmlString(html) chamado em initState
      return screen;
    },
  ),
);
```

**Configuração do GoRouter para WebView:**

```dart
GoRoute(
  path: '/webview',
  name: 'webview',
  builder: (context, state) {
    final args = state.extra as WebViewCompletoScreen;
    return args;
  },
),
```

---

---

## 10. HTML + CSS × Flutter — Tabela de Equivalências

Referência rápida para quem vem do desenvolvimento web e quer mapear conceitos HTML/CSS para os widgets e propriedades Flutter correspondentes.

---

### Elementos HTML → Widgets Flutter

#### Estrutura e Layout

| HTML | Flutter | Observação |
|---|---|---|
| `<body>` | `Scaffold.body` | Área principal de conteúdo da tela |
| `<header>` | `AppBar` | Barra superior com título e ações |
| `<footer>` | `Scaffold.bottomNavigationBar` / `SafeArea` | Rodapé fixo na base |
| `<nav>` | `Drawer`, `BottomNavigationBar`, `NavigationRail`, `TabBar` | Depende do padrão de navegação |
| `<aside>` | `Drawer`, `NavigationRail` | Menu lateral |
| `<main>`, `<section>`, `<article>` | `Column`, `Container`, `Padding` | Sem semântica equivalente; use pelo layout |
| `<div>` | `Container`, `SizedBox`, `Padding`, `Column`, `Row` | Widget mais próximo depende do estilo aplicado |
| `<span>` | `Text` inline dentro de `RichText` | Para texto misto de estilos, use `TextSpan` |
| `<hr>` | `Divider` / `VerticalDivider` | Linha separadora horizontal ou vertical |

```dart
// <div class="card"> com padding, borda e sombra
Container(
  padding: const EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 8)],
    border: Border.all(color: Colors.grey.shade200),
  ),
  child: Text('Conteúdo'),
)
```

---

#### Texto e Tipografia

| HTML | Flutter | Observação |
|---|---|---|
| `<h1>` | `Text(style: textTheme.displayLarge)` | Título maior |
| `<h2>` | `Text(style: textTheme.headlineLarge)` | |
| `<h3>` | `Text(style: textTheme.headlineMedium)` | |
| `<h4>` | `Text(style: textTheme.titleLarge)` | |
| `<h5>` | `Text(style: textTheme.titleMedium)` | |
| `<h6>` | `Text(style: textTheme.titleSmall)` | |
| `<p>` | `Text(style: textTheme.bodyMedium)` | Parágrafo comum |
| `<strong>`, `<b>` | `Text(style: TextStyle(fontWeight: FontWeight.bold))` | |
| `<em>`, `<i>` | `Text(style: TextStyle(fontStyle: FontStyle.italic))` | |
| `<small>` | `Text(style: textTheme.bodySmall)` | |
| `<a>` | `TextButton`, `GestureDetector`, `InkWell` | Link clicável |
| `<br>` | `SizedBox(height: 8)` ou `\n` no texto | Quebra de linha |
| `<ul>` / `<ol>` + `<li>` | `ListView`, `Column` + `ListTile` | Lista de itens |

```dart
// <p>Texto com <strong>negrito</strong> e <em>itálico</em></p>
RichText(
  text: TextSpan(
    style: Theme.of(context).textTheme.bodyMedium,
    children: [
      const TextSpan(text: 'Texto com '),
      const TextSpan(text: 'negrito', style: TextStyle(fontWeight: FontWeight.bold)),
      const TextSpan(text: ' e '),
      const TextSpan(text: 'itálico', style: TextStyle(fontStyle: FontStyle.italic)),
    ],
  ),
)

// <a href="/rota">Clique aqui</a>
TextButton(
  onPressed: () => context.go('/rota'),
  child: const Text('Clique aqui'),
)
```

---

#### Mídia

| HTML | Flutter | Observação |
|---|---|---|
| `<img src="url">` | `Image.network(url)` | Imagem remota |
| `<img src="assets/...">` | `Image.asset(path)` | Imagem local (declarada no pubspec.yaml) |
| `<video>` | pacote `video_player` | `pub add video_player` |
| `<audio>` | pacote `audioplayers` | `pub add audioplayers` |
| `<iframe>` | `WebViewWidget` | Seção 9 deste guia |
| `<svg>` | pacote `flutter_svg` | `pub add flutter_svg` |
| `<canvas>` | `CustomPaint` + `Canvas` | Desenho de baixo nível |

```dart
// <img src="foto.jpg" width="80" height="80" style="object-fit:cover; border-radius:50%">
ClipOval(
  child: Image.network(
    'https://exemplo.com/foto.jpg',
    width: 80,
    height: 80,
    fit: BoxFit.cover,
    errorBuilder: (_, __, ___) => const Icon(Icons.person, size: 80),
  ),
)
```

---

#### Formulários e Inputs

| HTML | Flutter | Observação |
|---|---|---|
| `<form>` | `Form` + `GlobalKey<FormState>` | Agrupa campos para validação |
| `<input type="text">` | `TextField` / `TextFormField` | Campo de texto genérico |
| `<input type="email">` | `TextFormField(keyboardType: TextInputType.emailAddress)` | |
| `<input type="password">` | `TextFormField(obscureText: true)` | |
| `<input type="number">` | `TextFormField(keyboardType: TextInputType.number)` | |
| `<input type="tel">` | `TextFormField(keyboardType: TextInputType.phone)` | |
| `<input type="checkbox">` | `Checkbox` / `CheckboxListTile` | |
| `<input type="radio">` | `Radio` / `RadioListTile` | |
| `<input type="range">` | `Slider` | |
| `<input type="date">` | `showDatePicker()` + `TextFormField` para exibir | |
| `<input type="time">` | `showTimePicker()` + `TextFormField` para exibir | |
| `<input type="file">` | pacotes `file_picker` / `image_picker` | |
| `<input type="search">` | `TextField` com `suffixIcon: Icon(Icons.search)` | |
| `<textarea>` | `TextFormField(maxLines: null)` | Campo multilinha |
| `<select>` | `DropdownButtonFormField` | Lista suspensa |
| `<button type="submit">` | `FilledButton` / `ElevatedButton` | Ação primária |
| `<button type="reset">` | `OutlinedButton` + `_formKey.currentState!.reset()` | |
| `<label>` | `InputDecoration.labelText` / `Text` antes do campo | |
| `<fieldset>` + `<legend>` | `Card` + `Column` com título `Text` | Agrupamento visual |
| `<datalist>` | `Autocomplete<String>` | Sugestões conforme digitação |

```dart
// <input type="date"> com seletor nativo
TextFormField(
  controller: _dataCtrl,
  readOnly: true,
  decoration: const InputDecoration(
    labelText: 'Data de nascimento',
    suffixIcon: Icon(Icons.calendar_today),
    border: OutlineInputBorder(),
  ),
  onTap: () async {
    final data = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime(1900),
      lastDate: DateTime.now(),
    );
    if (data != null) {
      _dataCtrl.text = '${data.day}/${data.month}/${data.year}';
    }
  },
)

// <input type="range" min="0" max="100">
Slider(
  value: _volume,
  min: 0,
  max: 100,
  divisions: 10,
  label: _volume.round().toString(),
  onChanged: (v) => setState(() => _volume = v),
)
```

---

#### Tabelas

| HTML | Flutter | Observação |
|---|---|---|
| `<table>` | `DataTable` | Tabela Material com ordenação |
| `<table>` (layout) | `Table` widget | Para layout tabular sem dados |
| `<tr>` | `DataRow` | |
| `<td>` | `DataCell` | |
| `<th>` | `DataColumn` | |

```dart
// <table> simples com cabeçalho
DataTable(
  columns: const [
    DataColumn(label: Text('Nome')),
    DataColumn(label: Text('Preço'), numeric: true),
    DataColumn(label: Text('Status')),
  ],
  rows: produtos.map((p) => DataRow(cells: [
    DataCell(Text(p.nome)),
    DataCell(Text('R\$ ${p.preco.toStringAsFixed(2)}')),
    DataCell(Icon(p.disponivel ? Icons.check : Icons.close,
        color: p.disponivel ? Colors.green : Colors.red)),
  ])).toList(),
)
```

---

### CSS → Flutter

#### Modelo de Caixa (Box Model)

| CSS | Flutter | Observação |
|---|---|---|
| `padding: 16px` | `Padding(padding: EdgeInsets.all(16))` | |
| `padding: 8px 16px` | `EdgeInsets.symmetric(vertical: 8, horizontal: 16)` | |
| `padding: 4px 8px 12px 16px` | `EdgeInsets.fromLTRB(16, 4, 8, 12)` | top→vertical, LTRB = left,top,right,bottom |
| `margin: 16px` | `Container(margin: EdgeInsets.all(16))` | |
| `width: 100%` | `width: double.infinity` | |
| `height: 100%` | `height: double.infinity` | |
| `max-width: 600px` | `ConstrainedBox(constraints: BoxConstraints(maxWidth: 600))` | |
| `min-height: 48px` | `ConstrainedBox(constraints: BoxConstraints(minHeight: 48))` | |
| `box-sizing: border-box` | Padrão no Flutter | Todo widget inclui padding no tamanho |

---

#### Display e Flexbox

| CSS | Flutter | Observação |
|---|---|---|
| `display: flex` (linha) | `Row` | Eixo principal horizontal |
| `display: flex` (coluna) | `Column` | Eixo principal vertical |
| `display: grid` | `GridView` | Grade de itens |
| `flex-direction: row` | `Row` | |
| `flex-direction: column` | `Column` | |
| `justify-content: flex-start` | `mainAxisAlignment: MainAxisAlignment.start` | |
| `justify-content: center` | `mainAxisAlignment: MainAxisAlignment.center` | |
| `justify-content: flex-end` | `mainAxisAlignment: MainAxisAlignment.end` | |
| `justify-content: space-between` | `mainAxisAlignment: MainAxisAlignment.spaceBetween` | |
| `justify-content: space-around` | `mainAxisAlignment: MainAxisAlignment.spaceAround` | |
| `justify-content: space-evenly` | `mainAxisAlignment: MainAxisAlignment.spaceEvenly` | |
| `align-items: flex-start` | `crossAxisAlignment: CrossAxisAlignment.start` | |
| `align-items: center` | `crossAxisAlignment: CrossAxisAlignment.center` | |
| `align-items: flex-end` | `crossAxisAlignment: CrossAxisAlignment.end` | |
| `align-items: stretch` | `crossAxisAlignment: CrossAxisAlignment.stretch` | |
| `flex: 1` | `Expanded(child: ...)` | Ocupa todo o espaço restante |
| `flex-grow: 1` com tamanho mínimo | `Flexible(child: ...)` | Pode ser menor que o espaço disponível |
| `gap: 16px` | `SizedBox(width: 16)` / `SizedBox(height: 16)` | Espaçador entre filhos |
| `flex-wrap: wrap` | `Wrap(spacing: 8, runSpacing: 4)` | Quebra automaticamente em múltiplas linhas |
| `order` | Ordem dos filhos na lista `children` | Primeiro filho = mais à esquerda/cima |

```dart
// display: flex; justify-content: space-between; align-items: center
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [Text('Esquerda'), Text('Direita')],
)

// display: flex; gap: 12px
Row(
  children: [
    Text('A'),
    const SizedBox(width: 12), // gap
    Text('B'),
    const SizedBox(width: 12),
    Text('C'),
  ],
)
```

---

#### Posicionamento

| CSS | Flutter | Observação |
|---|---|---|
| `position: relative` | `Stack` | Contêiner para posicionamento absoluto |
| `position: absolute; top: 8px; right: 8px` | `Positioned(top: 8, right: 8, child: ...)` dentro de `Stack` | |
| `position: absolute; inset: 0` | `Positioned.fill(child: ...)` | Ocupa todo o Stack |
| `position: fixed` | Widget dentro do `Scaffold` fora do `body` | Ex.: `bottomNavigationBar`, FAB |
| `position: sticky` | `SliverAppBar(pinned: true)` | Cabeçalho fixo ao rolar |
| `z-index` | Ordem em `Stack.children` | Último filho fica no topo |
| `overflow: hidden` | `ClipRRect`, `ClipRect`, `ClipOval` | Corta o conteúdo |
| `overflow: scroll` | `SingleChildScrollView`, `ListView` | Conteúdo rolável |

---

#### Aparência Visual

| CSS | Flutter | Observação |
|---|---|---|
| `background-color: #fff` | `color: Colors.white` em `Container` | |
| `background: linear-gradient(...)` | `BoxDecoration(gradient: LinearGradient(...))` | |
| `border: 1px solid #ddd` | `BoxDecoration(border: Border.all(color: Colors.grey.shade300))` | |
| `border-radius: 12px` | `BoxDecoration(borderRadius: BorderRadius.circular(12))` | |
| `border-radius: 16px 16px 0 0` | `BorderRadius.vertical(top: Radius.circular(16))` | |
| `box-shadow: 0 4px 8px rgba(0,0,0,.1)` | `BoxDecoration(boxShadow: [BoxShadow(blurRadius: 8, color: Colors.black12)])` | |
| `opacity: 0.5` | `Opacity(opacity: 0.5, child: ...)` | |
| `visibility: hidden` | `Visibility(visible: false, child: ...)` | Mantém espaço |
| `display: none` | `if (condicao) widget` ou `Visibility(visible: false, maintainSize: false)` | Remove do layout |
| `cursor: pointer` | `InkWell`, `GestureDetector`, `MouseRegion(cursor: SystemMouseCursors.click)` | |
| `clip-path: circle(50%)` | `ClipOval(child: ...)` | Recorte circular |

```dart
// background: linear-gradient(135deg, #6366f1, #8b5cf6)
Container(
  decoration: const BoxDecoration(
    gradient: LinearGradient(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
      colors: [Color(0xFF6366F1), Color(0xFF8B5CF6)],
    ),
  ),
)

// border-radius: 16px 16px 0 0  (arredonda só o topo)
Container(
  decoration: const BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
  ),
)
```

---

#### Tipografia

| CSS | Flutter | Observação |
|---|---|---|
| `color: #333` | `TextStyle(color: Color(0xFF333333))` | |
| `font-size: 16px` | `TextStyle(fontSize: 16)` | Unidade em pixels lógicos |
| `font-weight: bold` / `700` | `TextStyle(fontWeight: FontWeight.bold)` | |
| `font-weight: 300` | `TextStyle(fontWeight: FontWeight.w300)` | w100 a w900 |
| `font-style: italic` | `TextStyle(fontStyle: FontStyle.italic)` | |
| `text-decoration: underline` | `TextStyle(decoration: TextDecoration.underline)` | |
| `text-decoration: line-through` | `TextStyle(decoration: TextDecoration.lineThrough)` | |
| `letter-spacing: 1.5px` | `TextStyle(letterSpacing: 1.5)` | |
| `line-height: 1.5` | `TextStyle(height: 1.5)` | Multiplicador sobre `fontSize` |
| `text-align: center` | `Text('', textAlign: TextAlign.center)` | |
| `text-align: right` | `Text('', textAlign: TextAlign.end)` | |
| `text-overflow: ellipsis` | `Text('', overflow: TextOverflow.ellipsis, maxLines: 1)` | |
| `text-transform: uppercase` | `text.toUpperCase()` | Aplicado na string Dart |
| `white-space: nowrap` | `Text('', softWrap: false)` | |
| `word-break: break-all` | `Text('', overflow: TextOverflow.visible, softWrap: true)` | |

---

#### Animações e Transições

| CSS | Flutter | Observação |
|---|---|---|
| `transition: all 300ms ease` | `AnimatedContainer(duration: Duration(milliseconds: 300))` | Anima propriedades de layout automaticamente |
| `transition: opacity 200ms` | `AnimatedOpacity(duration: Duration(milliseconds: 200))` | |
| `transition: color 200ms` | `TweenAnimationBuilder` | Interpolação manual de cor |
| `@keyframes` + `animation` | `AnimationController` + `Tween` | Animação explícita com controle total |
| `transform: scale(1.1)` | `Transform.scale(scale: 1.1, child: ...)` | |
| `transform: translateX(20px)` | `Transform.translate(offset: Offset(20, 0), child: ...)` | |
| `transform: rotate(45deg)` | `Transform.rotate(angle: pi / 4, child: ...)` | Ângulo em radianos |

```dart
// transition: width 300ms ease, background-color 300ms
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: _expandido ? 200 : 80,
  color: _expandido ? Colors.indigo : Colors.grey,
  child: ...,
)

// transition: opacity 200ms
AnimatedOpacity(
  duration: const Duration(milliseconds: 200),
  opacity: _visivel ? 1.0 : 0.0,
  child: Text('Conteúdo'),
)
```

---

### Padrões de Tela — HTML × Flutter lado a lado

#### Cabeçalho de Página

```html
<!-- HTML -->
<header class="sticky top-0 z-10 bg-white shadow">
  <div class="flex justify-between items-center px-4 h-14">
    <button>☰</button>
    <h1>Título</h1>
    <button>🔍</button>
  </div>
</header>
```

```dart
// Flutter
AppBar(
  leading: IconButton(icon: const Icon(Icons.menu), onPressed: () {}),
  title: const Text('Título'),
  actions: [IconButton(icon: const Icon(Icons.search), onPressed: () {})],
)
```

---

#### Card com Imagem

```html
<!-- HTML -->
<div class="rounded-xl shadow overflow-hidden">
  <img src="..." class="w-full h-48 object-cover">
  <div class="p-4">
    <h3>Nome</h3>
    <p class="text-gray-500">Descrição</p>
    <span class="font-bold text-indigo-600">R$ 99,90</span>
  </div>
</div>
```

```dart
// Flutter
Card(
  clipBehavior: Clip.antiAlias,
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Image.network(url, height: 192, width: double.infinity, fit: BoxFit.cover),
      Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Nome', style: Theme.of(context).textTheme.titleMedium),
            const SizedBox(height: 4),
            Text('Descrição', style: TextStyle(color: Colors.grey.shade600)),
            const SizedBox(height: 8),
            Text('R\$ 99,90',
                style: TextStyle(
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary)),
          ],
        ),
      ),
    ],
  ),
)
```

---

#### Formulário de Login

```html
<!-- HTML -->
<form>
  <label>E-mail</label>
  <input type="email" placeholder="nome@exemplo.com">
  <label>Senha</label>
  <input type="password" placeholder="••••••••">
  <button type="submit">Entrar</button>
  <a href="/esqueci">Esqueci minha senha</a>
</form>
```

```dart
// Flutter
Form(
  key: _formKey,
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.stretch,
    children: [
      TextFormField(
        decoration: const InputDecoration(
          labelText: 'E-mail',
          hintText: 'nome@exemplo.com',
          border: OutlineInputBorder(),
        ),
        keyboardType: TextInputType.emailAddress,
        validator: (v) => (v == null || !v.contains('@')) ? 'E-mail inválido' : null,
      ),
      const SizedBox(height: 16),
      TextFormField(
        decoration: const InputDecoration(
          labelText: 'Senha',
          hintText: '••••••••',
          border: OutlineInputBorder(),
        ),
        obscureText: true,
        validator: (v) => (v == null || v.length < 6) ? 'Senha muito curta' : null,
      ),
      const SizedBox(height: 24),
      FilledButton(
        onPressed: () {
          if (_formKey.currentState!.validate()) _entrar();
        },
        child: const Text('Entrar'),
      ),
      const SizedBox(height: 12),
      TextButton(
        onPressed: () => context.go('/esqueci-senha'),
        child: const Text('Esqueci minha senha'),
      ),
    ],
  ),
)
```

---

#### Barra de Navegação Inferior

```html
<!-- HTML -->
<nav class="fixed bottom-0 flex justify-around bg-white border-t">
  <a href="/">🏠 Início</a>
  <a href="/busca">🔍 Buscar</a>
  <a href="/perfil">👤 Perfil</a>
</nav>
```

```dart
// Flutter
Scaffold(
  body: ...,
  bottomNavigationBar: NavigationBar(
    selectedIndex: _indice,
    onDestinationSelected: (i) => setState(() => _indice = i),
    destinations: const [
      NavigationDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: 'Início'),
      NavigationDestination(icon: Icon(Icons.search), label: 'Buscar'),
      NavigationDestination(icon: Icon(Icons.person_outline), selectedIcon: Icon(Icons.person), label: 'Perfil'),
    ],
  ),
)
```

> **`NavigationBar` vs `BottomNavigationBar`:** `NavigationBar` é o componente Material 3 recomendado (suporta badges e indicador de seleção animado). `BottomNavigationBar` é o predecessor Material 2, ainda funcional mas legado.

---

---

### Requisições HTTP Assíncronas — Fetch API × Flutter

#### Conceitos equivalentes

| Fetch API (JavaScript) | Flutter (Dart) | Observação |
|---|---|---|
| `fetch(url)` | `http.get(Uri.parse(url))` / `dio.get(url)` | Retorna uma `Future` (equivalente a `Promise`) |
| `Promise` | `Future<T>` | Representação de valor assíncrono futuro |
| `.then()` / `.catch()` | `await` + `try/catch` | Dart usa async/await nativo; `.then()` também existe |
| `async function` | `Future<T> nome() async {}` | Toda função assíncrona retorna `Future` |
| `await fetch(url)` | `await http.get(...)` | Mesma sintaxe de espera |
| `response.json()` | `jsonDecode(response.body)` | Decodificação manual em Dart |
| `response.ok` | `response.statusCode >= 200 && < 300` | Verificação manual de status HTTP |
| `response.status` | `response.statusCode` | Código numérico da resposta |
| `AbortController` | `CancelToken` (Dio) | Cancelamento de requisição |
| `Headers` | `Map<String, String>` em `headers:` | Cabeçalhos como mapa Dart |
| `FormData` | `MultipartRequest` (http) / `FormData` (Dio) | Upload de arquivos e formulários |

---

#### GET — buscar dados

```js
// JavaScript — Fetch API
async function buscarProdutos() {
  const response = await fetch('https://api.exemplo.com/produtos');
  if (!response.ok) throw new Error(`Erro: ${response.status}`);
  const dados = await response.json();
  return dados;
}
```

```dart
// Flutter — pacote http
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<List<Map<String, dynamic>>> buscarProdutos() async {
  final response = await http.get(
    Uri.parse('https://api.exemplo.com/produtos'),
  );
  if (response.statusCode < 200 || response.statusCode >= 300) {
    throw Exception('Erro: ${response.statusCode}');
  }
  final List<dynamic> dados = jsonDecode(response.body);
  return dados.cast<Map<String, dynamic>>();
}
```

```dart
// Flutter — Dio (equivalente mais direto ao fetch)
import 'package:dio/dio.dart';

final dio = Dio();

Future<List<dynamic>> buscarProdutos() async {
  final response = await dio.get('https://api.exemplo.com/produtos');
  return response.data as List<dynamic>; // já decodificado
}
```

---

#### POST — enviar dados com corpo JSON

```js
// JavaScript — Fetch API
async function criarProduto(produto) {
  const response = await fetch('https://api.exemplo.com/produtos', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(produto),
  });
  if (!response.ok) throw new Error(`Erro: ${response.status}`);
  return response.json();
}
```

```dart
// Flutter — pacote http
Future<Map<String, dynamic>> criarProduto(Map<String, dynamic> produto) async {
  final response = await http.post(
    Uri.parse('https://api.exemplo.com/produtos'),
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode(produto),
  );
  if (response.statusCode < 200 || response.statusCode >= 300) {
    throw Exception('Erro: ${response.statusCode}');
  }
  return jsonDecode(response.body) as Map<String, dynamic>;
}
```

```dart
// Flutter — Dio
Future<Map<String, dynamic>> criarProduto(Map<String, dynamic> produto) async {
  final response = await dio.post(
    'https://api.exemplo.com/produtos',
    data: produto, // serialização automática
  );
  return response.data as Map<String, dynamic>;
}
```

---

#### PUT e DELETE

```js
// JavaScript
await fetch(`/produtos/${id}`, { method: 'PUT', body: JSON.stringify(dados), headers: {...} });
await fetch(`/produtos/${id}`, { method: 'DELETE' });
```

```dart
// Flutter — Dio
await dio.put('/produtos/$id', data: dados);
await dio.delete('/produtos/$id');
```

---

#### Tratamento de erros

```js
// JavaScript
try {
  const res = await fetch('/produtos');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const dados = await res.json();
} catch (e) {
  console.error('Falha na requisição:', e.message);
}
```

```dart
// Flutter — http
try {
  final res = await http.get(Uri.parse('/produtos'));
  if (res.statusCode != 200) throw Exception('HTTP ${res.statusCode}');
  final dados = jsonDecode(res.body);
} on http.ClientException catch (e) {
  // sem conectividade ou URL inválida
  debugPrint('Erro de rede: $e');
} catch (e) {
  debugPrint('Erro: $e');
}
```

```dart
// Flutter — Dio (exceções tipadas)
try {
  final res = await dio.get('/produtos');
} on DioException catch (e) {
  switch (e.type) {
    case DioExceptionType.connectionTimeout:
    case DioExceptionType.receiveTimeout:
      debugPrint('Timeout');
    case DioExceptionType.badResponse:
      debugPrint('HTTP ${e.response?.statusCode}');
    case DioExceptionType.connectionError:
      debugPrint('Sem conexão');
    default:
      debugPrint('Erro: ${e.message}');
  }
}
```

---

#### Requisição com cabeçalho de autenticação

```js
// JavaScript
const token = localStorage.getItem('token');
const res = await fetch('/perfil', {
  headers: { 'Authorization': `Bearer ${token}` },
});
```

```dart
// Flutter — http
final token = await storage.read(key: 'token'); // flutter_secure_storage
final res = await http.get(
  Uri.parse('https://api.exemplo.com/perfil'),
  headers: {'Authorization': 'Bearer $token'},
);
```

```dart
// Flutter — Dio com interceptor (equivalente a um middleware global)
dio.interceptors.add(
  InterceptorsWrapper(
    onRequest: (options, handler) async {
      final token = await storage.read(key: 'token');
      options.headers['Authorization'] = 'Bearer $token';
      handler.next(options);
    },
  ),
);
// A partir daqui, todas as requisições do `dio` incluem o token automaticamente.
```

---

#### Estado de carregamento na UI

```js
// JavaScript — React
const [loading, setLoading] = useState(false);
const [dados, setDados] = useState([]);

async function carregar() {
  setLoading(true);
  try {
    const res = await fetch('/produtos');
    setDados(await res.json());
  } finally {
    setLoading(false);
  }
}
```

```dart
// Flutter — StatefulWidget
bool _carregando = false;
List<dynamic> _dados = [];

Future<void> carregar() async {
  setState(() => _carregando = true);
  try {
    final res = await http.get(Uri.parse('/produtos'));
    setState(() => _dados = jsonDecode(res.body) as List<dynamic>);
  } finally {
    setState(() => _carregando = false);
  }
}

// Na build:
_carregando
    ? const CircularProgressIndicator()
    : ListView.builder(itemCount: _dados.length, ...)
```

```dart
// Flutter — Riverpod FutureProvider (mais idiomático)
final produtosProvider = FutureProvider<List<Produto>>((ref) async {
  final res = await ref.watch(dioProvider).get('/produtos');
  return (res.data as List).map(Produto.fromJson).toList();
});

// Na widget (ConsumerWidget):
ref.watch(produtosProvider).when(
  loading: () => const CircularProgressIndicator(),
  error: (e, _) => Text('Erro: $e'),
  data: (lista) => ListView.builder(itemCount: lista.length, ...),
)
```

---

> **Referências:**
> - [flutter.dev](https://flutter.dev) — documentação oficial
> - [pub.dev](https://pub.dev) — repositório de pacotes Dart/Flutter
> - [GoRouter](https://pub.dev/packages/go_router) — navegação declarativa
> - [Riverpod](https://riverpod.dev) — gerenciamento de estado
> - [Dio](https://pub.dev/packages/dio) — cliente HTTP
> - [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) — armazenamento seguro
> - [webview_flutter](https://pub.dev/packages/webview_flutter) — WebView oficial do Flutter team

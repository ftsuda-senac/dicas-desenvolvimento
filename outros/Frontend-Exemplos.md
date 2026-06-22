# Frontend — Exemplos Avançados

Exemplos completos de telas comuns em Flutter, React Native e React Web, com implementações equivalentes entre as três plataformas.

---

## Criação de Projeto em Branco

### Flutter 3.x

```bash
# Pré-requisito: Flutter SDK instalado (https://docs.flutter.dev/get-started/install)
flutter --version

# Criar projeto
flutter create --org com.exemplo meu_app
cd meu_app

# Rodar
flutter run
```

**Estrutura gerada:**

```
meu_app/
├── lib/
│   └── main.dart          ← ponto de entrada
├── test/
├── android/
├── ios/
├── web/
├── pubspec.yaml           ← dependências
└── analysis_options.yaml
```

**Selecionar plataformas específicas (opcional):**

```bash
# Apenas Android e iOS
flutter create --platforms=android,ios meu_app

# Com Web e Desktop
flutter create --platforms=android,ios,web,windows meu_app
```

---

### React Native + Expo (SDK 53)

```bash
# Pré-requisito: Node.js 18+ instalado
node --version

# Criar projeto com template em branco + TypeScript
npx create-expo-app@latest meu-app --template blank-typescript
cd meu-app

# Rodar
npx expo start
# Pressionar: a (Android) | i (iOS) | w (Web)
```

**Estrutura gerada:**

```
meu-app/
├── app.json               ← configuração do Expo
├── App.tsx                ← ponto de entrada
├── assets/
├── package.json           ← dependências
└── tsconfig.json
```

**Com Expo Router (navegação baseada em arquivos):**

```bash
npx create-expo-app@latest meu-app --template tabs
cd meu-app

npx expo start
```

```
meu-app/
├── app/                   ← rotas baseadas em arquivos
│   ├── (tabs)/
│   │   ├── index.tsx      ← tela inicial
│   │   └── _layout.tsx    ← layout das abas
│   └── _layout.tsx        ← layout raiz
├── components/
├── constants/
├── app.json
├── package.json
└── tsconfig.json
```

---

### React Web (Vite + React 19 + TypeScript)

```bash
# Pré-requisito: Node.js 18+ instalado
node --version

# Criar projeto com Vite
npm create vite@latest meu-app -- --template react-ts
cd meu-app

# Instalar dependências
npm install

# Rodar
npm run dev
```

**Estrutura gerada:**

```
meu-app/
├── public/
├── src/
│   ├── App.tsx            ← componente raiz
│   ├── main.tsx           ← ponto de entrada
│   ├── App.css
│   └── index.css
├── index.html             ← HTML host
├── package.json           ← dependências
├── tsconfig.json
└── vite.config.ts
```

**Com React Router (para SPAs com rotas):**

```bash
npm create vite@latest meu-app -- --template react-ts
cd meu-app
npm install
npm install react-router-dom

npm run dev
```

**Alternativa — Next.js 15 (SSR + App Router):**

```bash
npx create-next-app@latest meu-app --typescript --tailwind --app --src-dir --eslint
cd meu-app

npm run dev
```

```
meu-app/
├── src/
│   └── app/
│       ├── layout.tsx     ← layout raiz
│       ├── page.tsx       ← rota /
│       └── globals.css
├── public/
├── package.json
├── next.config.ts
└── tsconfig.json
```

---

### Comparação Rápida

| | Flutter | React Native + Expo | React Web (Vite) |
|---|---|---|---|
| Comando | `flutter create meu_app` | `npx create-expo-app@latest meu-app` | `npm create vite@latest meu-app` |
| Linguagem | Dart | TypeScript | TypeScript |
| Ponto de entrada | `lib/main.dart` | `App.tsx` | `src/main.tsx` |
| Dependências | `pubspec.yaml` | `package.json` | `package.json` |
| Dev server | `flutter run` | `npx expo start` | `npm run dev` |
| Hot reload | Automático | Automático (Fast Refresh) | Automático (HMR Vite) |
| Plataformas | Android, iOS, Web, Desktop | Android, iOS, Web | Web (SPA) |

---

## Sumário

1. [Player de Vídeo](#1-player-de-vídeo)
   - [Flutter — better_player_plus](#flutter--better_player_plus)
   - [React Native — react-native-video](#react-native--react-native-video)
   - [React Web — Video.js 8](#react-web--videojs-8)
2. [Mapas com Geolocalização](#2-mapas-com-geolocalização)
   - [Flutter — google_maps_flutter + geolocator](#flutter--google_maps_flutter--geolocator)
   - [React Native — react-native-maps + expo-location](#react-native--react-native-maps--expo-location)
   - [React Web — Leaflet + Geolocation API](#react-web--leaflet--geolocation-api)
3. [Autenticação OAuth2/OIDC com Controle de Acesso](#3-autenticação-oauth2oidc-com-controle-de-acesso)
   - [Flutter — flutter_appauth + GoRouter](#flutter--flutter_appauth--gorouter)
   - [Flutter (alternativo) — flutter_web_auth_2 + flutter_modular](#flutter-alternativo--flutter_web_auth_2--flutter_modular)
   - [React Native — expo-auth-session + React Navigation](#react-native--expo-auth-session--react-navigation)
   - [React Web — oidc-client-ts + React Router](#react-web--oidc-client-ts--react-router)
4. [Layout Responsivo — Home com Sidebar, Cards e Scroll Infinito](#4-layout-responsivo--home-com-sidebar-cards-e-scroll-infinito)
   - [Flutter — GoRouter](#flutter--gorouter)
   - [Flutter (alternativo) — Modular](#flutter-alternativo--modular)
   - [React Native — Expo Router + Drawer](#react-native--expo-router--drawer)
   - [React Web — Bootstrap + Font Awesome](#react-web--bootstrap--font-awesome)
5. [Formulários Complexos com Validação e Máscaras](#5-formulários-complexos-com-validação-e-máscaras)
   - [Flutter — Form + mask_text_input_formatter](#flutter--form--mask_text_input_formatter)
   - [React Native — React Hook Form + react-native-mask-input](#react-native--react-hook-form--react-native-mask-input)
   - [React Web — Bootstrap Forms + validação customizada](#react-web--bootstrap-forms--validação-customizada)
6. [Chat em Tempo Real (WebSocket)](#6-chat-em-tempo-real-websocket)
   - [Flutter — web_socket_channel](#flutter--web_socket_channel)
   - [React Native — WebSocket API](#react-native--websocket-api)
   - [React Web — WebSocket API + Bootstrap](#react-web--websocket-api--bootstrap)
7. [Notificações Push](#7-notificações-push)
   - [Flutter — firebase_messaging + flutter_local_notifications](#flutter--firebase_messaging--flutter_local_notifications)
   - [React Native — expo-notifications](#react-native--expo-notifications)
   - [React Web — Notification API + Service Worker](#react-web--notification-api--service-worker)
8. [CRUD Completo com Tabela de Dados](#8-crud-completo-com-tabela-de-dados)
   - [Flutter — DataTable + AlertDialog](#flutter--datatable--alertdialog)
   - [React Native — FlatList + Modal](#react-native--flatlist--modal)
   - [React Web — Bootstrap Table + react-bootstrap Modal](#react-web--bootstrap-table--react-bootstrap-modal)
9. [Tema Claro/Escuro](#9-tema-claroescuro)
   - [Flutter — ThemeData + Riverpod](#flutter--themedata--riverpod)
   - [React Native — useColorScheme + Context](#react-native--usecolorscheme--context)
   - [React Web — Bootstrap data-bs-theme + localStorage](#react-web--bootstrap-data-bs-theme--localstorage)
10. [Câmera e Galeria](#10-câmera-e-galeria)
    - [Flutter — image_picker + image_cropper](#flutter--image_picker--image_cropper)
    - [React Native — expo-camera + expo-image-picker](#react-native--expo-camera--expo-image-picker)
    - [React Web — MediaDevices API + Canvas](#react-web--mediadevices-api--canvas)
11. [Dashboard com Gráficos](#11-dashboard-com-gráficos)
    - [Flutter — fl_chart](#flutter--fl_chart)
    - [React Native — react-native-chart-kit](#react-native--react-native-chart-kit)
    - [React Web — Chart.js + Bootstrap](#react-web--chartjs--bootstrap)
12. [QR Code — Leitor e Gerador](#12-qr-code--leitor-e-gerador)
    - [Flutter — mobile_scanner + qr_flutter](#flutter--mobile_scanner--qr_flutter)
    - [React Native — expo-camera + react-native-qrcode-svg](#react-native--expo-camera--react-native-qrcode-svg)
    - [React Web — html5-qrcode + qrcode](#react-web--html5-qrcode--qrcode)
13. [Armazenamento Offline e Sincronização](#13-armazenamento-offline-e-sincronização)
    - [Flutter — sqflite + connectivity_plus](#flutter--sqflite--connectivity_plus)
    - [React Native — expo-sqlite + NetInfo](#react-native--expo-sqlite--netinfo)
    - [React Web — IndexedDB + navigator.onLine](#react-web--indexeddb--navigatoronline)
14. [Internacionalização (i18n)](#14-internacionalização-i18n)
    - [Flutter — flutter_localizations + intl](#flutter--flutter_localizations--intl)
    - [React Native — expo-localization + i18next](#react-native--expo-localization--i18next)
    - [React Web — i18next + react-i18next + Bootstrap](#react-web--i18next--react-i18next--bootstrap)
15. [Autenticação Biométrica](#15-autenticação-biométrica)
    - [Flutter — local_auth](#flutter--local_auth)
    - [React Native — expo-local-authentication](#react-native--expo-local-authentication)
    - [React Web — Web Authentication API (WebAuthn)](#react-web--web-authentication-api-webauthn)
16. [Bluetooth — Integração com Beacons](#16-bluetooth--integração-com-beacons)
    - [Pré-requisitos e Infraestrutura](#pré-requisitos-e-infraestrutura)
    - [Flutter — flutter_beacon](#flutter--flutter_beacon)
    - [React Native — react-native-ble-plx + beacons](#react-native--react-native-ble-plx--beacons)
    - [React Web — Web Bluetooth API](#react-web--web-bluetooth-api-1)

---

## 1. Player de Vídeo

Tela de player com controles customizados, suporte a múltiplas resoluções, picture-in-picture e tratamento de erros.

### Flutter — better_player_plus

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  better_player_plus: ^0.3.0
```

```bash
flutter pub get
```

**2. Configuração Android — `android/app/src/main/AndroidManifest.xml`**

```xml
<manifest>
  <!-- Permissão para streaming via rede -->
  <uses-permission android:name="android.permission.INTERNET" />

  <application>
    <activity
      android:name=".MainActivity"
      android:supportsPictureInPicture="true"
      android:configChanges="screenSize|smallestScreenSize|screenLayout|orientation" />
  </application>
</manifest>
```

**3. Configuração iOS — `ios/Runner/Info.plist`**

```xml
<dict>
  <!-- Permite reprodução em background -->
  <key>UIBackgroundModes</key>
  <array>
    <string>audio</string>
  </array>
</dict>
```

**4. Modelo de dados**

```dart
// lib/features/video/domain/video_item.dart
class VideoItem {
  final String titulo;
  final String url;
  final String? thumbnailUrl;
  final Duration? duracao;
  final Map<String, String>? resolucoes;

  const VideoItem({
    required this.titulo,
    required this.url,
    this.thumbnailUrl,
    this.duracao,
    this.resolucoes,
  });
}
```

**5. Tela do player**

```dart
// lib/features/video/presentation/video_player_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:better_player_plus/better_player_plus.dart';
import '../domain/video_item.dart';

class VideoPlayerScreen extends StatefulWidget {
  final VideoItem video;
  const VideoPlayerScreen({super.key, required this.video});

  @override
  State<VideoPlayerScreen> createState() => _VideoPlayerScreenState();
}

class _VideoPlayerScreenState extends State<VideoPlayerScreen> {
  late BetterPlayerController _controller;

  @override
  void initState() {
    super.initState();
    _controller = BetterPlayerController(
      BetterPlayerConfiguration(
        autoPlay: true,
        fit: BoxFit.contain,
        allowedScreenSleep: false,
        controlsConfiguration: const BetterPlayerControlsConfiguration(
          enableSkips: true,
          skipForwardIcon: Icons.forward_10,
          skipBackIcon: Icons.replay_10,
          enablePip: true,
          enablePlaybackSpeed: true,
          enableSubtitles: true,
          enableQualities: true,
          enableOverflowMenu: true,
        ),
        deviceOrientationsOnFullScreen: [
          DeviceOrientation.landscapeLeft,
          DeviceOrientation.landscapeRight,
        ],
        deviceOrientationsAfterFullScreen: [
          DeviceOrientation.portraitUp,
        ],
      ),
      betterPlayerDataSource: _buildDataSource(),
    );

    _controller.addEventsListener(_onPlayerEvent);
  }

  BetterPlayerDataSource _buildDataSource() {
    final video = widget.video;

    if (video.resolucoes != null && video.resolucoes!.isNotEmpty) {
      return BetterPlayerDataSource(
        BetterPlayerDataSourceType.network,
        video.url,
        resolutions: video.resolucoes,
        notificationConfiguration: BetterPlayerNotificationConfiguration(
          showNotification: true,
          title: video.titulo,
          imageUrl: video.thumbnailUrl,
        ),
      );
    }

    return BetterPlayerDataSource(
      BetterPlayerDataSourceType.network,
      video.url,
      notificationConfiguration: BetterPlayerNotificationConfiguration(
        showNotification: true,
        title: video.titulo,
        imageUrl: video.thumbnailUrl,
      ),
    );
  }

  void _onPlayerEvent(BetterPlayerEvent event) {
    if (event.betterPlayerEventType == BetterPlayerEventType.exception) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Erro ao reproduzir o vídeo.'),
          backgroundColor: Colors.red,
        ),
      );
    }
  }

  @override
  void dispose() {
    _controller.removeEventsListener(_onPlayerEvent);
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        title: Text(widget.video.titulo),
        backgroundColor: Colors.black,
        foregroundColor: Colors.white,
      ),
      body: Column(
        children: [
          // Player
          AspectRatio(
            aspectRatio: 16 / 9,
            child: BetterPlayer(controller: _controller),
          ),

          // Informações do vídeo
          Expanded(
            child: Container(
              width: double.infinity,
              color: Colors.grey.shade900,
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    widget.video.titulo,
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  if (widget.video.duracao != null)
                    Padding(
                      padding: const EdgeInsets.only(top: 8),
                      child: Text(
                        _formatarDuracao(widget.video.duracao!),
                        style: TextStyle(color: Colors.grey.shade400),
                      ),
                    ),
                  const SizedBox(height: 16),

                  // Controles extras
                  Row(
                    children: [
                      _BotaoAcao(
                        icone: Icons.speed,
                        rotulo: 'Velocidade',
                        onTap: () => _controller
                            .setOverriddenAspectRatio(16 / 9),
                      ),
                      const SizedBox(width: 16),
                      _BotaoAcao(
                        icone: Icons.picture_in_picture_alt,
                        rotulo: 'PiP',
                        onTap: () =>
                            _controller.enablePictureInPicture(
                                _controller.betterPlayerGlobalKey!),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  String _formatarDuracao(Duration d) {
    final h = d.inHours;
    final m = d.inMinutes.remainder(60).toString().padLeft(2, '0');
    final s = d.inSeconds.remainder(60).toString().padLeft(2, '0');
    return h > 0 ? '$h:$m:$s' : '$m:$s';
  }
}

class _BotaoAcao extends StatelessWidget {
  final IconData icone;
  final String rotulo;
  final VoidCallback onTap;

  const _BotaoAcao({
    required this.icone,
    required this.rotulo,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(8),
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
        child: Column(
          children: [
            Icon(icone, color: Colors.white, size: 24),
            const SizedBox(height: 4),
            Text(rotulo,
                style: TextStyle(color: Colors.grey.shade300, fontSize: 12)),
          ],
        ),
      ),
    );
  }
}
```

**6. Tela de lista de vídeos com navegação**

```dart
// lib/features/video/presentation/video_list_screen.dart
import 'package:flutter/material.dart';
import '../domain/video_item.dart';
import 'video_player_screen.dart';

class VideoListScreen extends StatelessWidget {
  const VideoListScreen({super.key});

  static const _videos = [
    VideoItem(
      titulo: 'Big Buck Bunny',
      url: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
      thumbnailUrl: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/BigBuckBunny.jpg',
      duracao: Duration(minutes: 9, seconds: 56),
      resolucoes: {
        '480p': 'https://sample-videos.com/video321/mp4/480/big_buck_bunny_480p_1mb.mp4',
        '720p': 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
      },
    ),
    VideoItem(
      titulo: 'Elephant Dream',
      url: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4',
      thumbnailUrl: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/ElephantsDream.jpg',
      duracao: Duration(minutes: 10, seconds: 53),
    ),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Vídeos')),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: _videos.length,
        itemBuilder: (context, i) {
          final video = _videos[i];
          return Card(
            clipBehavior: Clip.antiAlias,
            margin: const EdgeInsets.only(bottom: 16),
            child: InkWell(
              onTap: () => Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (_) => VideoPlayerScreen(video: video),
                ),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Thumbnail com duração
                  Stack(
                    alignment: Alignment.bottomRight,
                    children: [
                      Image.network(
                        video.thumbnailUrl ?? '',
                        height: 200,
                        width: double.infinity,
                        fit: BoxFit.cover,
                        errorBuilder: (_, __, ___) => Container(
                          height: 200,
                          color: Colors.grey.shade800,
                          child: const Icon(Icons.play_circle_outline,
                              size: 64, color: Colors.white54),
                        ),
                      ),
                      if (video.duracao != null)
                        Container(
                          margin: const EdgeInsets.all(8),
                          padding: const EdgeInsets.symmetric(
                              horizontal: 6, vertical: 2),
                          decoration: BoxDecoration(
                            color: Colors.black87,
                            borderRadius: BorderRadius.circular(4),
                          ),
                          child: Text(
                            _formatarDuracao(video.duracao!),
                            style: const TextStyle(
                                color: Colors.white, fontSize: 12),
                          ),
                        ),
                    ],
                  ),
                  Padding(
                    padding: const EdgeInsets.all(12),
                    child: Text(video.titulo,
                        style: Theme.of(context).textTheme.titleMedium),
                  ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }

  String _formatarDuracao(Duration d) {
    final h = d.inHours;
    final m = d.inMinutes.remainder(60).toString().padLeft(2, '0');
    final s = d.inSeconds.remainder(60).toString().padLeft(2, '0');
    return h > 0 ? '$h:$m:$s' : '$m:$s';
  }
}
```

---

### React Native — react-native-video

**1. Dependências**

```bash
npx expo install react-native-video
npx expo install expo-screen-orientation
npx expo install react-native-safe-area-context
```

**2. Tipos**

```typescript
// src/types/video.ts
export interface VideoItem {
  id: string;
  titulo: string;
  url: string;
  thumbnailUrl?: string;
  duracao?: number; // segundos
}
```

**3. Componente do player**

```tsx
// src/features/video/VideoPlayerScreen.tsx
import React, { useRef, useState, useCallback } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  ActivityIndicator,
  StatusBar,
  Dimensions,
} from 'react-native';
import Video, {
  OnLoadData,
  OnProgressData,
  OnBufferData,
  VideoRef,
} from 'react-native-video';
import { VideoItem } from '../../types/video';

interface Props {
  video: VideoItem;
  onVoltar: () => void;
}

export function VideoPlayerScreen({ video, onVoltar }: Props) {
  const playerRef = useRef<VideoRef>(null);
  const [pausado, setPausado] = useState(false);
  const [carregando, setCarregando] = useState(true);
  const [progresso, setProgresso] = useState(0);
  const [duracaoTotal, setDuracaoTotal] = useState(0);
  const [erro, setErro] = useState<string | null>(null);
  const [controlesVisiveis, setControlesVisiveis] = useState(true);
  const [fullscreen, setFullscreen] = useState(false);

  const { width } = Dimensions.get('window');
  const alturaPlayer = (width * 9) / 16;

  const onLoad = useCallback((data: OnLoadData) => {
    setDuracaoTotal(data.duration);
    setCarregando(false);
  }, []);

  const onProgress = useCallback((data: OnProgressData) => {
    setProgresso(data.currentTime);
  }, []);

  const onBuffer = useCallback((data: OnBufferData) => {
    setCarregando(data.isBuffering);
  }, []);

  const onError = useCallback(() => {
    setErro('Erro ao reproduzir o vídeo.');
    setCarregando(false);
  }, []);

  const avancar = useCallback((segundos: number) => {
    playerRef.current?.seek(progresso + segundos);
  }, [progresso]);

  const formatarTempo = (s: number): string => {
    const h = Math.floor(s / 3600);
    const m = String(Math.floor((s % 3600) / 60)).padStart(2, '0');
    const seg = String(Math.floor(s % 60)).padStart(2, '0');
    return h > 0 ? `${h}:${m}:${seg}` : `${m}:${seg}`;
  };

  const porcentagem = duracaoTotal > 0 ? progresso / duracaoTotal : 0;

  if (erro) {
    return (
      <View style={styles.erroContainer}>
        <Text style={styles.erroTexto}>{erro}</Text>
        <TouchableOpacity onPress={onVoltar} style={styles.botaoVoltar}>
          <Text style={styles.botaoVoltarTexto}>Voltar</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <StatusBar hidden={fullscreen} />

      {/* Player */}
      <TouchableOpacity
        activeOpacity={1}
        onPress={() => setControlesVisiveis(!controlesVisiveis)}
        style={{ width, height: alturaPlayer }}
      >
        <Video
          ref={playerRef}
          source={{ uri: video.url }}
          style={StyleSheet.absoluteFill}
          resizeMode="contain"
          paused={pausado}
          onLoad={onLoad}
          onProgress={onProgress}
          onBuffer={onBuffer}
          onError={onError}
          repeat={false}
        />

        {/* Loading */}
        {carregando && (
          <View style={styles.overlay}>
            <ActivityIndicator size="large" color="#fff" />
          </View>
        )}

        {/* Controles */}
        {controlesVisiveis && !carregando && (
          <View style={styles.overlay}>
            <View style={styles.controlesLinha}>
              <TouchableOpacity onPress={() => avancar(-10)}>
                <Text style={styles.iconeControle}>⏪ 10s</Text>
              </TouchableOpacity>

              <TouchableOpacity onPress={() => setPausado(!pausado)}>
                <Text style={styles.iconePrincipal}>
                  {pausado ? '▶️' : '⏸️'}
                </Text>
              </TouchableOpacity>

              <TouchableOpacity onPress={() => avancar(10)}>
                <Text style={styles.iconeControle}>10s ⏩</Text>
              </TouchableOpacity>
            </View>
          </View>
        )}
      </TouchableOpacity>

      {/* Barra de progresso */}
      <View style={styles.barraContainer}>
        <View style={[styles.barraProgresso, { flex: porcentagem }]} />
        <View style={[styles.barraRestante, { flex: 1 - porcentagem }]} />
      </View>
      <View style={styles.tempoContainer}>
        <Text style={styles.tempoTexto}>{formatarTempo(progresso)}</Text>
        <Text style={styles.tempoTexto}>{formatarTempo(duracaoTotal)}</Text>
      </View>

      {/* Informações */}
      <View style={styles.info}>
        <Text style={styles.titulo}>{video.titulo}</Text>
        {video.duracao && (
          <Text style={styles.duracao}>
            Duração: {formatarTempo(video.duracao)}
          </Text>
        )}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#000' },
  overlay: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(0,0,0,0.4)',
    justifyContent: 'center',
    alignItems: 'center',
  },
  controlesLinha: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 40,
  },
  iconeControle: { color: '#fff', fontSize: 16 },
  iconePrincipal: { fontSize: 48 },
  barraContainer: { flexDirection: 'row', height: 3, backgroundColor: '#333' },
  barraProgresso: { backgroundColor: '#e53935' },
  barraRestante: { backgroundColor: '#555' },
  tempoContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingHorizontal: 16,
    paddingVertical: 4,
  },
  tempoTexto: { color: '#aaa', fontSize: 12 },
  info: { padding: 16 },
  titulo: { color: '#fff', fontSize: 18, fontWeight: 'bold' },
  duracao: { color: '#aaa', marginTop: 8, fontSize: 14 },
  erroContainer: {
    flex: 1,
    backgroundColor: '#000',
    justifyContent: 'center',
    alignItems: 'center',
  },
  erroTexto: { color: '#e53935', fontSize: 16, marginBottom: 16 },
  botaoVoltar: {
    paddingHorizontal: 24,
    paddingVertical: 12,
    backgroundColor: '#333',
    borderRadius: 8,
  },
  botaoVoltarTexto: { color: '#fff', fontSize: 14 },
});
```

**4. Tela de lista de vídeos**

```tsx
// src/features/video/VideoListScreen.tsx
import React from 'react';
import {
  View,
  Text,
  FlatList,
  Image,
  TouchableOpacity,
  StyleSheet,
} from 'react-native';
import { VideoItem } from '../../types/video';

const VIDEOS: VideoItem[] = [
  {
    id: '1',
    titulo: 'Big Buck Bunny',
    url: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
    thumbnailUrl:
      'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/BigBuckBunny.jpg',
    duracao: 596,
  },
  {
    id: '2',
    titulo: 'Elephant Dream',
    url: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4',
    thumbnailUrl:
      'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/ElephantsDream.jpg',
    duracao: 653,
  },
];

interface Props {
  onSelecionar: (video: VideoItem) => void;
}

export function VideoListScreen({ onSelecionar }: Props) {
  const formatarTempo = (s: number): string => {
    const m = String(Math.floor(s / 60)).padStart(2, '0');
    const seg = String(Math.floor(s % 60)).padStart(2, '0');
    return `${m}:${seg}`;
  };

  return (
    <FlatList
      data={VIDEOS}
      keyExtractor={(item) => item.id}
      contentContainerStyle={styles.lista}
      renderItem={({ item }) => (
        <TouchableOpacity
          style={styles.card}
          onPress={() => onSelecionar(item)}
        >
          <View>
            <Image
              source={{ uri: item.thumbnailUrl }}
              style={styles.thumbnail}
              resizeMode="cover"
            />
            {item.duracao && (
              <View style={styles.badgeDuracao}>
                <Text style={styles.badgeTexto}>
                  {formatarTempo(item.duracao)}
                </Text>
              </View>
            )}
          </View>
          <Text style={styles.titulo}>{item.titulo}</Text>
        </TouchableOpacity>
      )}
    />
  );
}

const styles = StyleSheet.create({
  lista: { padding: 16 },
  card: {
    backgroundColor: '#1a1a1a',
    borderRadius: 12,
    marginBottom: 16,
    overflow: 'hidden',
  },
  thumbnail: { width: '100%', height: 200 },
  badgeDuracao: {
    position: 'absolute',
    bottom: 8,
    right: 8,
    backgroundColor: 'rgba(0,0,0,0.85)',
    paddingHorizontal: 6,
    paddingVertical: 2,
    borderRadius: 4,
  },
  badgeTexto: { color: '#fff', fontSize: 12 },
  titulo: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600',
    padding: 12,
  },
});
```

---

### React Web — Video.js 8

**1. Dependências**

```bash
npm install video.js@8
```

**2. Tipos**

```typescript
// src/types/video.ts
export interface VideoItem {
  id: string;
  titulo: string;
  url: string;
  tipo: string; // 'video/mp4', 'application/x-mpegURL', etc.
  thumbnailUrl?: string;
  duracao?: number;
  resolucoes?: { label: string; src: string; type: string }[];
}
```

**3. Componente do player — wrapper do Video.js**

```tsx
// src/components/VideoPlayer.tsx
import { useEffect, useRef } from 'react';
import videojs from 'video.js';
import type Player from 'video.js/dist/types/player';
import 'video.js/dist/video-js.css';

interface VideoPlayerProps {
  url: string;
  tipo: string;
  poster?: string;
  onReady?: (player: Player) => void;
  onError?: (error: unknown) => void;
}

export function VideoPlayer({
  url,
  tipo,
  poster,
  onReady,
  onError,
}: VideoPlayerProps) {
  const videoRef = useRef<HTMLDivElement>(null);
  const playerRef = useRef<Player | null>(null);

  useEffect(() => {
    if (!videoRef.current) return;

    const videoElement = document.createElement('video-js');
    videoElement.classList.add('vjs-big-play-centered', 'vjs-16-9');
    videoRef.current.appendChild(videoElement);

    const player = videojs(videoElement, {
      controls: true,
      autoplay: false,
      preload: 'auto',
      fluid: true,
      responsive: true,
      playbackRates: [0.5, 1, 1.25, 1.5, 2],
      poster,
      sources: [{ src: url, type: tipo }],
      controlBar: {
        pictureInPictureToggle: true,
        skipButtons: { forward: 10, backward: 10 },
      },
      userActions: {
        hotkeys: true,
      },
    });

    player.ready(() => onReady?.(player));
    player.on('error', () => onError?.(player.error()));

    playerRef.current = player;

    return () => {
      if (playerRef.current && !playerRef.current.isDisposed()) {
        playerRef.current.dispose();
        playerRef.current = null;
      }
    };
  }, [url, tipo, poster, onReady, onError]);

  return (
    <div data-vjs-player>
      <div ref={videoRef} />
    </div>
  );
}
```

**4. Tela do player**

```tsx
// src/features/video/VideoPlayerPage.tsx
import { useCallback, useState } from 'react';
import type Player from 'video.js/dist/types/player';
import { VideoPlayer } from '../../components/VideoPlayer';
import { VideoItem } from '../../types/video';
import styles from './VideoPlayerPage.module.css';

interface Props {
  video: VideoItem;
  onVoltar: () => void;
}

export function VideoPlayerPage({ video, onVoltar }: Props) {
  const [erro, setErro] = useState<string | null>(null);

  const onReady = useCallback(
    (player: Player) => {
      // Adicionar resolucoes como fontes alternativas
      if (video.resolucoes) {
        const sources = video.resolucoes.map((r) => ({
          src: r.src,
          type: r.type,
          label: r.label,
        }));
        player.src(sources);
      }
    },
    [video.resolucoes],
  );

  const onError = useCallback(() => {
    setErro('Erro ao reproduzir o vídeo.');
  }, []);

  const formatarTempo = (s: number): string => {
    const h = Math.floor(s / 3600);
    const m = String(Math.floor((s % 3600) / 60)).padStart(2, '0');
    const seg = String(Math.floor(s % 60)).padStart(2, '0');
    return h > 0 ? `${h}:${m}:${seg}` : `${m}:${seg}`;
  };

  return (
    <div className={styles.container}>
      <header className={styles.header}>
        <button onClick={onVoltar} className={styles.botaoVoltar}>
          ← Voltar
        </button>
      </header>

      {erro ? (
        <div className={styles.erroContainer}>
          <p className={styles.erroTexto}>{erro}</p>
          <button onClick={onVoltar} className={styles.botaoVoltar}>
            Voltar
          </button>
        </div>
      ) : (
        <>
          <div className={styles.playerWrapper}>
            <VideoPlayer
              url={video.url}
              tipo={video.tipo}
              poster={video.thumbnailUrl}
              onReady={onReady}
              onError={onError}
            />
          </div>

          <div className={styles.info}>
            <h1 className={styles.titulo}>{video.titulo}</h1>
            {video.duracao && (
              <p className={styles.duracao}>
                Duração: {formatarTempo(video.duracao)}
              </p>
            )}
          </div>
        </>
      )}
    </div>
  );
}
```

**5. Estilos**

```css
/* src/features/video/VideoPlayerPage.module.css */
.container {
  min-height: 100vh;
  background: #0a0a0a;
  color: #fff;
}

.header {
  padding: 16px;
}

.botaoVoltar {
  background: #222;
  color: #fff;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
}

.botaoVoltar:hover {
  background: #333;
}

.playerWrapper {
  max-width: 960px;
  margin: 0 auto;
}

.info {
  max-width: 960px;
  margin: 0 auto;
  padding: 24px 16px;
}

.titulo {
  font-size: 1.5rem;
  font-weight: 700;
  margin: 0;
}

.duracao {
  color: #aaa;
  margin-top: 8px;
  font-size: 0.875rem;
}

.erroContainer {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 50vh;
}

.erroTexto {
  color: #e53935;
  font-size: 1rem;
  margin-bottom: 16px;
}
```

**6. Tela de lista de vídeos**

```tsx
// src/features/video/VideoListPage.tsx
import { VideoItem } from '../../types/video';
import styles from './VideoListPage.module.css';

const VIDEOS: VideoItem[] = [
  {
    id: '1',
    titulo: 'Big Buck Bunny',
    url: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
    tipo: 'video/mp4',
    thumbnailUrl:
      'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/BigBuckBunny.jpg',
    duracao: 596,
  },
  {
    id: '2',
    titulo: 'Elephant Dream',
    url: 'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4',
    tipo: 'video/mp4',
    thumbnailUrl:
      'https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/images/ElephantsDream.jpg',
    duracao: 653,
  },
];

interface Props {
  onSelecionar: (video: VideoItem) => void;
}

export function VideoListPage({ onSelecionar }: Props) {
  const formatarTempo = (s: number): string => {
    const m = String(Math.floor(s / 60)).padStart(2, '0');
    const seg = String(Math.floor(s % 60)).padStart(2, '0');
    return `${m}:${seg}`;
  };

  return (
    <div className={styles.container}>
      <h1 className={styles.header}>Vídeos</h1>
      <div className={styles.grid}>
        {VIDEOS.map((video) => (
          <button
            key={video.id}
            className={styles.card}
            onClick={() => onSelecionar(video)}
          >
            <div className={styles.thumbnailWrapper}>
              <img
                src={video.thumbnailUrl}
                alt={video.titulo}
                className={styles.thumbnail}
              />
              {video.duracao && (
                <span className={styles.badge}>
                  {formatarTempo(video.duracao)}
                </span>
              )}
            </div>
            <p className={styles.titulo}>{video.titulo}</p>
          </button>
        ))}
      </div>
    </div>
  );
}
```

```css
/* src/features/video/VideoListPage.module.css */
.container {
  min-height: 100vh;
  background: #0a0a0a;
  color: #fff;
  padding: 24px;
}

.header {
  font-size: 1.5rem;
  margin-bottom: 24px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 24px;
}

.card {
  background: #1a1a1a;
  border: none;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  text-align: left;
  transition: transform 0.2s;
}

.card:hover {
  transform: translateY(-2px);
}

.thumbnailWrapper {
  position: relative;
}

.thumbnail {
  width: 100%;
  height: 200px;
  object-fit: cover;
  display: block;
}

.badge {
  position: absolute;
  bottom: 8px;
  right: 8px;
  background: rgba(0, 0, 0, 0.85);
  color: #fff;
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 4px;
}

.titulo {
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  padding: 12px;
  margin: 0;
}
```

---

## 2. Mapas com Geolocalização

Tela de mapa com marcadores, localização do usuário em tempo real e busca de endereço.

### Flutter — google_maps_flutter + geolocator

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  google_maps_flutter: ^2.9.0
  geolocator: ^13.0.0
  geocoding: ^3.0.0
  permission_handler: ^11.3.0
```

```bash
flutter pub get
```

**2. Configuração Android — `android/app/src/main/AndroidManifest.xml`**

```xml
<manifest>
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

  <application>
    <meta-data
      android:name="com.google.android.geo.API_KEY"
      android:value="SUA_API_KEY_AQUI" />
  </application>
</manifest>
```

**3. Configuração iOS — `ios/Runner/AppDelegate.swift`**

```swift
import GoogleMaps

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GMSServices.provideAPIKey("SUA_API_KEY_AQUI")
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

**`ios/Runner/Info.plist`:**

```xml
<dict>
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>Precisamos da sua localização para mostrar no mapa.</string>
  <key>NSLocationAlwaysUsageDescription</key>
  <string>Precisamos da sua localização para navegação contínua.</string>
</dict>
```

**4. Serviço de localização**

```dart
// lib/core/services/location_service.dart
import 'package:geolocator/geolocator.dart';
import 'package:geocoding/geocoding.dart';

class LocationService {
  Future<bool> verificarPermissao() async {
    bool serviceEnabled = await Geolocator.isLocationServiceEnabled();
    if (!serviceEnabled) return false;

    LocationPermission permissao = await Geolocator.checkPermission();
    if (permissao == LocationPermission.denied) {
      permissao = await Geolocator.requestPermission();
      if (permissao == LocationPermission.denied) return false;
    }
    if (permissao == LocationPermission.deniedForever) return false;

    return true;
  }

  Future<Position> obterLocalizacaoAtual() async {
    return await Geolocator.getCurrentPosition(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
      ),
    );
  }

  Stream<Position> rastrearLocalizacao() {
    return Geolocator.getPositionStream(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
        distanceFilter: 10,
      ),
    );
  }

  Future<String?> obterEndereco(double lat, double lng) async {
    try {
      final placemarks = await placemarkFromCoordinates(lat, lng);
      if (placemarks.isEmpty) return null;
      final p = placemarks.first;
      return '${p.street}, ${p.subLocality}, ${p.locality} - ${p.administrativeArea}';
    } catch (_) {
      return null;
    }
  }

  Future<Location?> buscarCoordenadas(String endereco) async {
    try {
      final locations = await locationFromAddress(endereco);
      return locations.isNotEmpty ? locations.first : null;
    } catch (_) {
      return null;
    }
  }
}
```

**5. Tela do mapa**

```dart
// lib/features/mapa/presentation/mapa_screen.dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'package:geolocator/geolocator.dart';
import '../../../core/services/location_service.dart';

class MapaScreen extends StatefulWidget {
  const MapaScreen({super.key});

  @override
  State<MapaScreen> createState() => _MapaScreenState();
}

class _MapaScreenState extends State<MapaScreen> {
  final _locationService = LocationService();
  final _buscaCtrl = TextEditingController();
  GoogleMapController? _mapCtrl;
  StreamSubscription<Position>? _positionSub;

  LatLng _posicaoAtual = const LatLng(-23.5505, -46.6333); // São Paulo
  final Set<Marker> _marcadores = {};
  String? _enderecoAtual;
  bool _carregando = true;
  bool _rastreando = false;

  @override
  void initState() {
    super.initState();
    _inicializar();
  }

  Future<void> _inicializar() async {
    final permitido = await _locationService.verificarPermissao();
    if (!permitido) {
      if (mounted) {
        setState(() => _carregando = false);
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Permissão de localização negada.')),
        );
      }
      return;
    }

    final posicao = await _locationService.obterLocalizacaoAtual();
    final latlng = LatLng(posicao.latitude, posicao.longitude);
    final endereco = await _locationService.obterEndereco(
        posicao.latitude, posicao.longitude);

    if (mounted) {
      setState(() {
        _posicaoAtual = latlng;
        _enderecoAtual = endereco;
        _carregando = false;
        _marcadores.add(Marker(
          markerId: const MarkerId('usuario'),
          position: latlng,
          infoWindow: InfoWindow(
            title: 'Você está aqui',
            snippet: endereco ?? '',
          ),
          icon: BitmapDescriptor.defaultMarkerWithHue(
              BitmapDescriptor.hueAzure),
        ));
      });
      _mapCtrl?.animateCamera(CameraUpdate.newLatLngZoom(latlng, 15));
    }
  }

  void _toggleRastreamento() {
    if (_rastreando) {
      _positionSub?.cancel();
      setState(() => _rastreando = false);
    } else {
      _positionSub = _locationService.rastrearLocalizacao().listen(
        (posicao) async {
          final latlng = LatLng(posicao.latitude, posicao.longitude);
          final endereco = await _locationService.obterEndereco(
              posicao.latitude, posicao.longitude);
          if (mounted) {
            setState(() {
              _posicaoAtual = latlng;
              _enderecoAtual = endereco;
              _marcadores.removeWhere(
                  (m) => m.markerId == const MarkerId('usuario'));
              _marcadores.add(Marker(
                markerId: const MarkerId('usuario'),
                position: latlng,
                infoWindow: InfoWindow(
                  title: 'Você está aqui',
                  snippet: endereco ?? '',
                ),
                icon: BitmapDescriptor.defaultMarkerWithHue(
                    BitmapDescriptor.hueAzure),
              ));
            });
            _mapCtrl?.animateCamera(CameraUpdate.newLatLng(latlng));
          }
        },
      );
      setState(() => _rastreando = true);
    }
  }

  Future<void> _buscarEndereco() async {
    final texto = _buscaCtrl.text.trim();
    if (texto.isEmpty) return;

    final location = await _locationService.buscarCoordenadas(texto);
    if (location == null) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Endereço não encontrado.')),
        );
      }
      return;
    }

    final latlng = LatLng(location.latitude, location.longitude);
    setState(() {
      _marcadores.add(Marker(
        markerId: MarkerId('busca-${DateTime.now().millisecondsSinceEpoch}'),
        position: latlng,
        infoWindow: InfoWindow(title: texto),
      ));
    });
    _mapCtrl?.animateCamera(CameraUpdate.newLatLngZoom(latlng, 15));
    _buscaCtrl.clear();
    FocusScope.of(context).unfocus();
  }

  @override
  void dispose() {
    _positionSub?.cancel();
    _buscaCtrl.dispose();
    _mapCtrl?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          // Mapa
          GoogleMap(
            initialCameraPosition: CameraPosition(
              target: _posicaoAtual,
              zoom: 14,
            ),
            onMapCreated: (ctrl) => _mapCtrl = ctrl,
            markers: _marcadores,
            myLocationEnabled: true,
            myLocationButtonEnabled: false,
            zoomControlsEnabled: false,
            onLongPress: (latlng) async {
              final endereco = await _locationService.obterEndereco(
                  latlng.latitude, latlng.longitude);
              setState(() {
                _marcadores.add(Marker(
                  markerId: MarkerId(
                      'pin-${DateTime.now().millisecondsSinceEpoch}'),
                  position: latlng,
                  infoWindow: InfoWindow(
                    title: 'Marcador',
                    snippet: endereco ?? '${latlng.latitude}, ${latlng.longitude}',
                  ),
                ));
              });
            },
          ),

          // Barra de busca
          Positioned(
            top: MediaQuery.paddingOf(context).top + 8,
            left: 16,
            right: 16,
            child: Material(
              elevation: 4,
              borderRadius: BorderRadius.circular(12),
              child: TextField(
                controller: _buscaCtrl,
                decoration: InputDecoration(
                  hintText: 'Buscar endereço...',
                  prefixIcon: const Icon(Icons.search),
                  suffixIcon: IconButton(
                    icon: const Icon(Icons.send),
                    onPressed: _buscarEndereco,
                  ),
                  border: InputBorder.none,
                  contentPadding: const EdgeInsets.symmetric(
                      horizontal: 16, vertical: 14),
                ),
                textInputAction: TextInputAction.search,
                onSubmitted: (_) => _buscarEndereco(),
              ),
            ),
          ),

          // Endereço atual
          if (_enderecoAtual != null)
            Positioned(
              bottom: 100,
              left: 16,
              right: 16,
              child: Material(
                elevation: 4,
                borderRadius: BorderRadius.circular(12),
                child: Padding(
                  padding: const EdgeInsets.all(12),
                  child: Row(
                    children: [
                      const Icon(Icons.location_on, color: Colors.red),
                      const SizedBox(width: 8),
                      Expanded(
                        child: Text(_enderecoAtual!,
                            style: const TextStyle(fontSize: 13)),
                      ),
                    ],
                  ),
                ),
              ),
            ),

          // Loading
          if (_carregando)
            const Center(child: CircularProgressIndicator()),
        ],
      ),

      // Botões de ação
      floatingActionButton: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          FloatingActionButton.small(
            heroTag: 'rastrear',
            onPressed: _toggleRastreamento,
            backgroundColor: _rastreando ? Colors.red : null,
            child: Icon(_rastreando ? Icons.gps_off : Icons.gps_fixed),
          ),
          const SizedBox(height: 8),
          FloatingActionButton.small(
            heroTag: 'centralizar',
            onPressed: () => _mapCtrl?.animateCamera(
                CameraUpdate.newLatLngZoom(_posicaoAtual, 15)),
            child: const Icon(Icons.my_location),
          ),
          const SizedBox(height: 8),
          FloatingActionButton.small(
            heroTag: 'limpar',
            onPressed: () => setState(() => _marcadores
                .removeWhere((m) => m.markerId != const MarkerId('usuario'))),
            child: const Icon(Icons.layers_clear),
          ),
        ],
      ),
    );
  }
}
```

---

### React Native — react-native-maps + expo-location

**1. Dependências**

```bash
npx expo install react-native-maps expo-location
```

**2. Tela do mapa**

```tsx
// src/features/mapa/MapaScreen.tsx
import React, { useEffect, useRef, useState, useCallback } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  ActivityIndicator,
  Alert,
  Keyboard,
} from 'react-native';
import MapView, { Marker, Region, LatLng } from 'react-native-maps';
import * as Location from 'expo-location';

interface MarkerData {
  id: string;
  coordinate: LatLng;
  title: string;
  description?: string;
}

export function MapaScreen() {
  const mapRef = useRef<MapView>(null);
  const [regiao, setRegiao] = useState<Region>({
    latitude: -23.5505,
    longitude: -46.6333,
    latitudeDelta: 0.01,
    longitudeDelta: 0.01,
  });
  const [marcadores, setMarcadores] = useState<MarkerData[]>([]);
  const [enderecoAtual, setEnderecoAtual] = useState<string | null>(null);
  const [busca, setBusca] = useState('');
  const [carregando, setCarregando] = useState(true);
  const [rastreando, setRastreando] = useState(false);
  const locationSubRef = useRef<Location.LocationSubscription | null>(null);

  // Obter localização inicial
  useEffect(() => {
    (async () => {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        Alert.alert('Permissão negada', 'Ative a localização nas configurações.');
        setCarregando(false);
        return;
      }

      const pos = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.High,
      });
      const { latitude, longitude } = pos.coords;

      const endereco = await reverseGeocode(latitude, longitude);
      setEnderecoAtual(endereco);

      const novaRegiao: Region = {
        latitude,
        longitude,
        latitudeDelta: 0.01,
        longitudeDelta: 0.01,
      };
      setRegiao(novaRegiao);
      setMarcadores([
        {
          id: 'usuario',
          coordinate: { latitude, longitude },
          title: 'Você está aqui',
          description: endereco ?? undefined,
        },
      ]);
      mapRef.current?.animateToRegion(novaRegiao, 500);
      setCarregando(false);
    })();
  }, []);

  const reverseGeocode = async (
    lat: number,
    lng: number,
  ): Promise<string | null> => {
    try {
      const results = await Location.reverseGeocodeAsync({
        latitude: lat,
        longitude: lng,
      });
      if (results.length === 0) return null;
      const r = results[0];
      return [r.street, r.district, r.city, r.region]
        .filter(Boolean)
        .join(', ');
    } catch {
      return null;
    }
  };

  const geocode = async (
    endereco: string,
  ): Promise<LatLng | null> => {
    try {
      const results = await Location.geocodeAsync(endereco);
      if (results.length === 0) return null;
      return {
        latitude: results[0].latitude,
        longitude: results[0].longitude,
      };
    } catch {
      return null;
    }
  };

  // Buscar endereço
  const buscarEndereco = useCallback(async () => {
    const texto = busca.trim();
    if (!texto) return;

    const coord = await geocode(texto);
    if (!coord) {
      Alert.alert('Não encontrado', 'Endereço não localizado.');
      return;
    }

    setMarcadores((prev) => [
      ...prev,
      {
        id: `busca-${Date.now()}`,
        coordinate: coord,
        title: texto,
      },
    ]);
    mapRef.current?.animateToRegion(
      {
        ...coord,
        latitudeDelta: 0.01,
        longitudeDelta: 0.01,
      },
      500,
    );
    setBusca('');
    Keyboard.dismiss();
  }, [busca]);

  // Rastreamento em tempo real
  const toggleRastreamento = useCallback(async () => {
    if (rastreando) {
      locationSubRef.current?.remove();
      locationSubRef.current = null;
      setRastreando(false);
      return;
    }

    const sub = await Location.watchPositionAsync(
      { accuracy: Location.Accuracy.High, distanceInterval: 10 },
      async (pos) => {
        const { latitude, longitude } = pos.coords;
        const endereco = await reverseGeocode(latitude, longitude);
        setEnderecoAtual(endereco);

        setMarcadores((prev) => [
          ...prev.filter((m) => m.id !== 'usuario'),
          {
            id: 'usuario',
            coordinate: { latitude, longitude },
            title: 'Você está aqui',
            description: endereco ?? undefined,
          },
        ]);
        mapRef.current?.animateToRegion(
          {
            latitude,
            longitude,
            latitudeDelta: 0.01,
            longitudeDelta: 0.01,
          },
          300,
        );
      },
    );
    locationSubRef.current = sub;
    setRastreando(true);
  }, [rastreando]);

  // Adicionar marcador ao pressionar longo
  const onLongPress = useCallback(
    async (e: { nativeEvent: { coordinate: LatLng } }) => {
      const { latitude, longitude } = e.nativeEvent.coordinate;
      const endereco = await reverseGeocode(latitude, longitude);
      setMarcadores((prev) => [
        ...prev,
        {
          id: `pin-${Date.now()}`,
          coordinate: { latitude, longitude },
          title: 'Marcador',
          description: endereco ?? `${latitude.toFixed(5)}, ${longitude.toFixed(5)}`,
        },
      ]);
    },
    [],
  );

  const limparMarcadores = useCallback(() => {
    setMarcadores((prev) => prev.filter((m) => m.id === 'usuario'));
  }, []);

  const centralizar = useCallback(() => {
    mapRef.current?.animateToRegion(regiao, 500);
  }, [regiao]);

  return (
    <View style={styles.container}>
      {/* Mapa */}
      <MapView
        ref={mapRef}
        style={StyleSheet.absoluteFill}
        initialRegion={regiao}
        showsUserLocation
        showsMyLocationButton={false}
        onLongPress={onLongPress}
      >
        {marcadores.map((m) => (
          <Marker
            key={m.id}
            coordinate={m.coordinate}
            title={m.title}
            description={m.description}
            pinColor={m.id === 'usuario' ? '#2196F3' : '#F44336'}
          />
        ))}
      </MapView>

      {/* Barra de busca */}
      <View style={styles.buscaContainer}>
        <TextInput
          style={styles.buscaInput}
          placeholder="Buscar endereço..."
          value={busca}
          onChangeText={setBusca}
          onSubmitEditing={buscarEndereco}
          returnKeyType="search"
        />
        <TouchableOpacity onPress={buscarEndereco} style={styles.buscaBotao}>
          <Text style={styles.buscaBotaoTexto}>🔍</Text>
        </TouchableOpacity>
      </View>

      {/* Endereço atual */}
      {enderecoAtual && (
        <View style={styles.enderecoContainer}>
          <Text style={styles.enderecoTexto}>📍 {enderecoAtual}</Text>
        </View>
      )}

      {/* Botões de ação */}
      <View style={styles.botoesContainer}>
        <TouchableOpacity
          style={[styles.fab, rastreando && styles.fabAtivo]}
          onPress={toggleRastreamento}
        >
          <Text style={styles.fabTexto}>{rastreando ? '📡' : '🛰️'}</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.fab} onPress={centralizar}>
          <Text style={styles.fabTexto}>📌</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.fab} onPress={limparMarcadores}>
          <Text style={styles.fabTexto}>🗑️</Text>
        </TouchableOpacity>
      </View>

      {/* Loading */}
      {carregando && (
        <View style={styles.loading}>
          <ActivityIndicator size="large" color="#2196F3" />
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  buscaContainer: {
    position: 'absolute',
    top: 56,
    left: 16,
    right: 16,
    flexDirection: 'row',
    backgroundColor: '#fff',
    borderRadius: 12,
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.15,
    shadowRadius: 4,
  },
  buscaInput: {
    flex: 1,
    paddingHorizontal: 16,
    paddingVertical: 12,
    fontSize: 15,
  },
  buscaBotao: {
    justifyContent: 'center',
    paddingHorizontal: 16,
  },
  buscaBotaoTexto: { fontSize: 18 },
  enderecoContainer: {
    position: 'absolute',
    bottom: 100,
    left: 16,
    right: 16,
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 12,
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.15,
    shadowRadius: 4,
  },
  enderecoTexto: { fontSize: 13, color: '#333' },
  botoesContainer: {
    position: 'absolute',
    bottom: 32,
    right: 16,
    gap: 8,
  },
  fab: {
    width: 48,
    height: 48,
    borderRadius: 24,
    backgroundColor: '#fff',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.2,
    shadowRadius: 4,
  },
  fabAtivo: { backgroundColor: '#FFCDD2' },
  fabTexto: { fontSize: 20 },
  loading: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(255,255,255,0.6)',
  },
});
```

---

### React Web — Leaflet + Geolocation API

**1. Dependências**

```bash
npm install leaflet react-leaflet
npm install -D @types/leaflet
```

**2. Componente do mapa**

```tsx
// src/features/mapa/MapaPage.tsx
import { useCallback, useEffect, useRef, useState } from 'react';
import {
  MapContainer,
  TileLayer,
  Marker,
  Popup,
  useMapEvents,
  useMap,
} from 'react-leaflet';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import styles from './MapaPage.module.css';

// Fix default marker icons (Leaflet + bundlers)
import markerIcon2x from 'leaflet/dist/images/marker-icon-2x.png';
import markerIcon from 'leaflet/dist/images/marker-icon.png';
import markerShadow from 'leaflet/dist/images/marker-shadow.png';

L.Icon.Default.mergeOptions({
  iconRetinaUrl: markerIcon2x,
  iconUrl: markerIcon,
  shadowUrl: markerShadow,
});

const ICON_USUARIO = new L.Icon({
  iconUrl: markerIcon,
  iconRetinaUrl: markerIcon2x,
  shadowUrl: markerShadow,
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  className: 'marker-usuario',
});

interface MarkerData {
  id: string;
  lat: number;
  lng: number;
  titulo: string;
  descricao?: string;
}

// Centralizar mapa programaticamente
function FlyTo({ center, zoom }: { center: [number, number]; zoom: number }) {
  const map = useMap();
  useEffect(() => {
    map.flyTo(center, zoom, { duration: 0.8 });
  }, [map, center, zoom]);
  return null;
}

// Capturar cliques no mapa
function MapClickHandler({
  onLongPress,
}: {
  onLongPress: (lat: number, lng: number) => void;
}) {
  useMapEvents({
    contextmenu(e) {
      onLongPress(e.latlng.lat, e.latlng.lng);
    },
  });
  return null;
}

export function MapaPage() {
  const [centro, setCentro] = useState<[number, number]>([-23.5505, -46.6333]);
  const [zoom, setZoom] = useState(14);
  const [marcadores, setMarcadores] = useState<MarkerData[]>([]);
  const [enderecoAtual, setEnderecoAtual] = useState<string | null>(null);
  const [busca, setBusca] = useState('');
  const [carregando, setCarregando] = useState(true);
  const [rastreando, setRastreando] = useState(false);
  const watchIdRef = useRef<number | null>(null);

  // Geocoding via Nominatim (OpenStreetMap)
  const reverseGeocode = async (
    lat: number,
    lng: number,
  ): Promise<string | null> => {
    try {
      const res = await fetch(
        `https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lng}&format=json&accept-language=pt-BR`,
      );
      const data = await res.json();
      return data.display_name ?? null;
    } catch {
      return null;
    }
  };

  const geocode = async (
    endereco: string,
  ): Promise<{ lat: number; lng: number } | null> => {
    try {
      const res = await fetch(
        `https://nominatim.openstreetmap.org/search?q=${encodeURIComponent(endereco)}&format=json&limit=1&accept-language=pt-BR`,
      );
      const data = await res.json();
      if (data.length === 0) return null;
      return { lat: parseFloat(data[0].lat), lng: parseFloat(data[0].lon) };
    } catch {
      return null;
    }
  };

  // Localização inicial
  useEffect(() => {
    if (!navigator.geolocation) {
      setCarregando(false);
      return;
    }

    navigator.geolocation.getCurrentPosition(
      async (pos) => {
        const { latitude, longitude } = pos.coords;
        const endereco = await reverseGeocode(latitude, longitude);
        setEnderecoAtual(endereco);
        setCentro([latitude, longitude]);
        setZoom(15);
        setMarcadores([
          {
            id: 'usuario',
            lat: latitude,
            lng: longitude,
            titulo: 'Você está aqui',
            descricao: endereco ?? undefined,
          },
        ]);
        setCarregando(false);
      },
      () => {
        setCarregando(false);
      },
      { enableHighAccuracy: true },
    );
  }, []);

  // Buscar endereço
  const buscarEndereco = useCallback(async () => {
    const texto = busca.trim();
    if (!texto) return;

    const coord = await geocode(texto);
    if (!coord) {
      alert('Endereço não encontrado.');
      return;
    }

    setMarcadores((prev) => [
      ...prev,
      {
        id: `busca-${Date.now()}`,
        lat: coord.lat,
        lng: coord.lng,
        titulo: texto,
      },
    ]);
    setCentro([coord.lat, coord.lng]);
    setZoom(15);
    setBusca('');
  }, [busca]);

  // Rastreamento em tempo real
  const toggleRastreamento = useCallback(() => {
    if (rastreando) {
      if (watchIdRef.current !== null) {
        navigator.geolocation.clearWatch(watchIdRef.current);
        watchIdRef.current = null;
      }
      setRastreando(false);
      return;
    }

    const id = navigator.geolocation.watchPosition(
      async (pos) => {
        const { latitude, longitude } = pos.coords;
        const endereco = await reverseGeocode(latitude, longitude);
        setEnderecoAtual(endereco);
        setCentro([latitude, longitude]);
        setMarcadores((prev) => [
          ...prev.filter((m) => m.id !== 'usuario'),
          {
            id: 'usuario',
            lat: latitude,
            lng: longitude,
            titulo: 'Você está aqui',
            descricao: endereco ?? undefined,
          },
        ]);
      },
      undefined,
      { enableHighAccuracy: true },
    );
    watchIdRef.current = id;
    setRastreando(true);
  }, [rastreando]);

  // Adicionar marcador com clique direito
  const onContextMenu = useCallback(async (lat: number, lng: number) => {
    const endereco = await reverseGeocode(lat, lng);
    setMarcadores((prev) => [
      ...prev,
      {
        id: `pin-${Date.now()}`,
        lat,
        lng,
        titulo: 'Marcador',
        descricao: endereco ?? `${lat.toFixed(5)}, ${lng.toFixed(5)}`,
      },
    ]);
  }, []);

  const limparMarcadores = useCallback(() => {
    setMarcadores((prev) => prev.filter((m) => m.id === 'usuario'));
  }, []);

  return (
    <div className={styles.container}>
      {/* Mapa */}
      <MapContainer
        center={centro}
        zoom={zoom}
        className={styles.mapa}
        zoomControl={false}
      >
        <TileLayer
          attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        />
        <FlyTo center={centro} zoom={zoom} />
        <MapClickHandler onLongPress={onContextMenu} />

        {marcadores.map((m) => (
          <Marker
            key={m.id}
            position={[m.lat, m.lng]}
            icon={m.id === 'usuario' ? ICON_USUARIO : undefined}
          >
            <Popup>
              <strong>{m.titulo}</strong>
              {m.descricao && <br />}
              {m.descricao && <span>{m.descricao}</span>}
            </Popup>
          </Marker>
        ))}
      </MapContainer>

      {/* Barra de busca */}
      <div className={styles.buscaContainer}>
        <input
          type="text"
          placeholder="Buscar endereço..."
          value={busca}
          onChange={(e) => setBusca(e.target.value)}
          onKeyDown={(e) => e.key === 'Enter' && buscarEndereco()}
          className={styles.buscaInput}
        />
        <button onClick={buscarEndereco} className={styles.buscaBotao}>
          🔍
        </button>
      </div>

      {/* Endereço atual */}
      {enderecoAtual && (
        <div className={styles.enderecoContainer}>
          <span>📍 {enderecoAtual}</span>
        </div>
      )}

      {/* Botões de ação */}
      <div className={styles.botoesContainer}>
        <button
          className={`${styles.fab} ${rastreando ? styles.fabAtivo : ''}`}
          onClick={toggleRastreamento}
          title={rastreando ? 'Parar rastreamento' : 'Rastrear localização'}
        >
          {rastreando ? '📡' : '🛰️'}
        </button>
        <button
          className={styles.fab}
          onClick={() => {
            setCentro([...centro]);
            setZoom(15);
          }}
          title="Centralizar"
        >
          📌
        </button>
        <button
          className={styles.fab}
          onClick={limparMarcadores}
          title="Limpar marcadores"
        >
          🗑️
        </button>
      </div>

      {/* Loading */}
      {carregando && (
        <div className={styles.loading}>
          <div className={styles.spinner} />
        </div>
      )}
    </div>
  );
}
```

**3. Estilos**

```css
/* src/features/mapa/MapaPage.module.css */
.container {
  position: relative;
  width: 100%;
  height: 100vh;
}

.mapa {
  width: 100%;
  height: 100%;
  z-index: 0;
}

.buscaContainer {
  position: absolute;
  top: 16px;
  left: 16px;
  right: 16px;
  max-width: 480px;
  display: flex;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  z-index: 1000;
  overflow: hidden;
}

.buscaInput {
  flex: 1;
  border: none;
  padding: 12px 16px;
  font-size: 15px;
  outline: none;
}

.buscaBotao {
  background: none;
  border: none;
  padding: 0 16px;
  cursor: pointer;
  font-size: 18px;
}

.buscaBotao:hover {
  background: #f5f5f5;
}

.enderecoContainer {
  position: absolute;
  bottom: 100px;
  left: 16px;
  right: 16px;
  max-width: 480px;
  background: #fff;
  border-radius: 12px;
  padding: 12px 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  z-index: 1000;
  font-size: 13px;
  color: #333;
}

.botoesContainer {
  position: absolute;
  bottom: 32px;
  right: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  z-index: 1000;
}

.fab {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border: none;
  background: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  font-size: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

.fab:hover {
  background: #f0f0f0;
}

.fabAtivo {
  background: #ffcdd2;
}

.loading {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.6);
  z-index: 1000;
}

.spinner {
  width: 48px;
  height: 48px;
  border: 4px solid #e0e0e0;
  border-top-color: #2196f3;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Estilo do marcador do usuário */
:global(.marker-usuario) {
  filter: hue-rotate(200deg);
}
```

---

## Equivalência entre Plataformas

### Player de Vídeo

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Biblioteca | `better_player_plus` | `react-native-video` | Video.js 8 |
| Controles nativos | `BetterPlayerControlsConfiguration` | Implementação manual | Configuração do player |
| Múltiplas resoluções | `resolutions` no DataSource | Múltiplos `source` | `player.src([...])` |
| Picture-in-Picture | `enablePip: true` | API nativa por plataforma | `pictureInPictureToggle` |
| Velocidade de reprodução | `enablePlaybackSpeed: true` | `rate` prop | `playbackRates` |
| Fullscreen | Rotação automática | `expo-screen-orientation` | API nativa do Video.js |
| Tratamento de erros | `BetterPlayerEventType.exception` | `onError` callback | `player.on('error')` |

### Mapas com Geolocalização

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Mapa | `google_maps_flutter` | `react-native-maps` | `react-leaflet` (Leaflet) |
| Localização | `geolocator` | `expo-location` | `navigator.geolocation` |
| Geocoding | `geocoding` (Google) | `expo-location` | Nominatim (OpenStreetMap) |
| Permissões | `Geolocator.requestPermission()` | `Location.requestForegroundPermissionsAsync()` | Prompt automático do navegador |
| Rastreamento | `Geolocator.getPositionStream()` | `Location.watchPositionAsync()` | `geolocation.watchPosition()` |
| Marcadores dinâmicos | `Set<Marker>` + `setState` | Estado + componente `<Marker>` | Estado + componente `<Marker>` |
| Busca de endereço | `locationFromAddress()` | `Location.geocodeAsync()` | Nominatim REST API |
| Adicionar marcador | `onLongPress` no `GoogleMap` | `onLongPress` no `MapView` | `contextmenu` (clique direito) |
| API key obrigatória | Sim (Google Maps) | Sim (Google Maps) | Não (OpenStreetMap é gratuito) |

---

## 3. Autenticação OAuth2/OIDC com Controle de Acesso

Login e logout via OAuth2/OIDC (ex: Keycloak, Auth0, Google), refresh token automático, route guard por role (`COMUM`, `ADMIN`) e propagação do usuário logado para as telas protegidas.

### Flutter — flutter_appauth + GoRouter

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_appauth: ^7.0.0
  flutter_secure_storage: ^9.2.0
  jwt_decoder: ^2.0.1
  go_router: ^14.0.0
  flutter_riverpod: ^2.5.0
```

```bash
flutter pub get
```

**2. Configuração Android — `android/app/build.gradle`**

```groovy
android {
    defaultConfig {
        manifestPlaceholders += [
            'appAuthRedirectScheme': 'com.exemplo.app'
        ]
    }
}
```

**3. Configuração iOS — `ios/Runner/Info.plist`**

```xml
<dict>
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
      <key>CFBundleURLSchemes</key>
      <array>
        <string>com.exemplo.app</string>
      </array>
    </dict>
  </array>
</dict>
```

**4. Modelo do usuário**

```dart
// lib/core/auth/user_model.dart
class AppUser {
  final String id;
  final String nome;
  final String email;
  final List<String> roles;

  const AppUser({
    required this.id,
    required this.nome,
    required this.email,
    required this.roles,
  });

  bool get isAdmin => roles.contains('ADMIN');
  bool get isComum => roles.contains('COMUM');
  bool hasRole(String role) => roles.contains(role);
}
```

**5. Configuração OIDC**

```dart
// lib/core/auth/oidc_config.dart
abstract final class OidcConfig {
  static const String issuer = 'https://auth.exemplo.com/realms/meu-realm';
  static const String clientId = 'meu-app-mobile';
  static const String redirectUrl = 'com.exemplo.app://callback';
  static const String postLogoutRedirectUrl = 'com.exemplo.app://logout';
  static const List<String> scopes = ['openid', 'profile', 'email', 'roles'];

  // Endpoints (discovery automático via issuer, mas pode fixar)
  static const String discoveryUrl =
      '$issuer/.well-known/openid-configuration';
}
```

**6. Serviço de autenticação**

```dart
// lib/core/auth/auth_service.dart
import 'package:flutter_appauth/flutter_appauth.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:jwt_decoder/jwt_decoder.dart';
import 'oidc_config.dart';
import 'user_model.dart';

class AuthService {
  final _appAuth = const FlutterAppAuth();
  final _storage = const FlutterSecureStorage();

  static const _keyAccessToken = 'access_token';
  static const _keyRefreshToken = 'refresh_token';
  static const _keyIdToken = 'id_token';

  // --- Login ---
  Future<AppUser?> login() async {
    final result = await _appAuth.authorizeAndExchangeCode(
      AuthorizationTokenRequest(
        OidcConfig.clientId,
        OidcConfig.redirectUrl,
        issuer: OidcConfig.issuer,
        scopes: OidcConfig.scopes,
        promptValues: ['login'],
      ),
    );

    if (result == null) return null;

    await _salvarTokens(
      accessToken: result.accessToken!,
      refreshToken: result.refreshToken!,
      idToken: result.idToken!,
    );

    return _decodificarUsuario(result.accessToken!);
  }

  // --- Logout ---
  Future<void> logout() async {
    final idToken = await _storage.read(key: _keyIdToken);

    try {
      await _appAuth.endSession(EndSessionRequest(
        idTokenHint: idToken,
        postLogoutRedirectUrl: OidcConfig.postLogoutRedirectUrl,
        issuer: OidcConfig.issuer,
      ));
    } catch (_) {
      // Logout local mesmo se o servidor falhar
    }

    await _limparTokens();
  }

  // --- Refresh ---
  Future<AppUser?> refreshSession() async {
    final refreshToken = await _storage.read(key: _keyRefreshToken);
    if (refreshToken == null) return null;

    try {
      final result = await _appAuth.token(TokenRequest(
        OidcConfig.clientId,
        OidcConfig.redirectUrl,
        issuer: OidcConfig.issuer,
        refreshToken: refreshToken,
        scopes: OidcConfig.scopes,
      ));

      if (result == null) return null;

      await _salvarTokens(
        accessToken: result.accessToken!,
        refreshToken: result.refreshToken ?? refreshToken,
        idToken: result.idToken!,
      );

      return _decodificarUsuario(result.accessToken!);
    } catch (_) {
      await _limparTokens();
      return null;
    }
  }

  // --- Sessão ativa ---
  Future<AppUser?> recuperarSessao() async {
    final accessToken = await _storage.read(key: _keyAccessToken);
    if (accessToken == null) return null;

    if (JwtDecoder.isExpired(accessToken)) {
      return refreshSession();
    }

    return _decodificarUsuario(accessToken);
  }

  // --- Access token para requisições ---
  Future<String?> getAccessToken() async {
    final token = await _storage.read(key: _keyAccessToken);
    if (token == null) return null;

    if (JwtDecoder.isExpired(token)) {
      final user = await refreshSession();
      if (user == null) return null;
      return await _storage.read(key: _keyAccessToken);
    }

    return token;
  }

  // --- Helpers ---
  AppUser _decodificarUsuario(String accessToken) {
    final payload = JwtDecoder.decode(accessToken);

    // Keycloak: roles em realm_access.roles
    // Auth0: roles em custom claim
    final realmAccess = payload['realm_access'] as Map<String, dynamic>?;
    final roles = (realmAccess?['roles'] as List<dynamic>?)
            ?.map((r) => r.toString())
            .toList() ??
        [];

    return AppUser(
      id: payload['sub'] as String,
      nome: payload['name'] as String? ??
          payload['preferred_username'] as String? ??
          '',
      email: payload['email'] as String? ?? '',
      roles: roles,
    );
  }

  Future<void> _salvarTokens({
    required String accessToken,
    required String refreshToken,
    required String idToken,
  }) async {
    await Future.wait([
      _storage.write(key: _keyAccessToken, value: accessToken),
      _storage.write(key: _keyRefreshToken, value: refreshToken),
      _storage.write(key: _keyIdToken, value: idToken),
    ]);
  }

  Future<void> _limparTokens() async {
    await Future.wait([
      _storage.delete(key: _keyAccessToken),
      _storage.delete(key: _keyRefreshToken),
      _storage.delete(key: _keyIdToken),
    ]);
  }
}
```

> **2FA e MFA:** o fluxo OAuth2 Authorization Code delega toda a autenticação — incluindo segundo fator — para o identity provider (Keycloak, Auth0, etc.) dentro do navegador do sistema. O `authorizeAndExchangeCode` abre um **Chrome Custom Tab** (Android) ou **ASWebAuthenticationSession** (iOS), que é um processo independente do app. Quando o IdP solicita o código 2FA, o usuário pode sair para o app autenticador (Google Authenticator, SMS, etc.) e voltar ao navegador sem perder a sessão — o `Future` simplesmente aguarda o callback de redirect. O app não precisa implementar nenhuma lógica adicional para 2FA.

**7. Provider de autenticação (Riverpod)**

```dart
// lib/core/auth/auth_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'auth_service.dart';
import 'user_model.dart';

// Estado de autenticação
sealed class AuthState {}
class AuthLoading extends AuthState {}
class Autenticado extends AuthState {
  final AppUser usuario;
  Autenticado(this.usuario);
}
class NaoAutenticado extends AuthState {}

// Notifier
class AuthNotifier extends Notifier<AuthState> {
  late final _service = AuthService();

  @override
  AuthState build() {
    _inicializar();
    return AuthLoading();
  }

  Future<void> _inicializar() async {
    final usuario = await _service.recuperarSessao();
    state = usuario != null ? Autenticado(usuario) : NaoAutenticado();
  }

  Future<void> login() async {
    state = AuthLoading();
    final usuario = await _service.login();
    state = usuario != null ? Autenticado(usuario) : NaoAutenticado();
  }

  Future<void> logout() async {
    await _service.logout();
    state = NaoAutenticado();
  }

  Future<String?> getAccessToken() => _service.getAccessToken();
}

final authProvider = NotifierProvider<AuthNotifier, AuthState>(
  AuthNotifier.new,
);
```

**8. Router com guards por role**

```dart
// lib/routes/app_router.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../core/auth/auth_provider.dart';
import '../core/auth/user_model.dart';
import '../features/home/home_screen.dart';
import '../features/admin/admin_screen.dart';
import '../features/perfil/perfil_screen.dart';
import '../features/auth/login_screen.dart';
import '../features/public/sobre_screen.dart';

final routerProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authProvider);

  return GoRouter(
    initialLocation: '/',
    debugLogDiagnostics: true,

    // Redirect global — roda antes de cada navegação
    redirect: (context, state) {
      final destino = state.matchedLocation;
      final estaLogado = authState is Autenticado;

      // Rotas públicas — não precisam de autenticação
      const rotasPublicas = ['/', '/sobre', '/login'];
      final ehPublica = rotasPublicas.contains(destino);

      // Se não está logado e tenta acessar rota protegida → login
      if (!estaLogado && !ehPublica) {
        return '/login?redirect=$destino';
      }

      // Se está logado e tenta acessar login → home
      if (estaLogado && destino == '/login') {
        return '/home';
      }

      // Guard por role — rotas /admin exigem ADMIN
      if (estaLogado && destino.startsWith('/admin')) {
        final usuario = (authState as Autenticado).usuario;
        if (!usuario.isAdmin) {
          return '/home'; // ou /acesso-negado
        }
      }

      return null; // sem redirect
    },

    routes: [
      // --- Rotas públicas ---
      GoRoute(
        path: '/',
        builder: (_, __) => const SobreScreen(),
      ),
      GoRoute(
        path: '/sobre',
        builder: (_, __) => const SobreScreen(),
      ),
      GoRoute(
        path: '/login',
        builder: (_, state) => LoginScreen(
          redirectUrl: state.uri.queryParameters['redirect'],
        ),
      ),

      // --- Rotas autenticadas (COMUM ou ADMIN) ---
      GoRoute(
        path: '/home',
        builder: (_, __) => const HomeScreen(),
      ),
      GoRoute(
        path: '/perfil',
        builder: (_, __) => const PerfilScreen(),
      ),

      // --- Rotas somente ADMIN ---
      GoRoute(
        path: '/admin',
        builder: (_, __) => const AdminScreen(),
      ),
      GoRoute(
        path: '/admin/usuarios',
        builder: (_, __) => const AdminScreen(), // substituir pela tela real
      ),
    ],

    // Tela de erro / 404
    errorBuilder: (_, state) => Scaffold(
      body: Center(
        child: Text('Página não encontrada: ${state.matchedLocation}'),
      ),
    ),
  );
});
```

**9. Tela de login**

```dart
// lib/features/auth/login_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../../core/auth/auth_provider.dart';

class LoginScreen extends ConsumerStatefulWidget {
  final String? redirectUrl;
  const LoginScreen({super.key, this.redirectUrl});

  @override
  ConsumerState<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends ConsumerState<LoginScreen> {
  bool _carregando = false;
  String? _erro;

  Future<void> _fazerLogin() async {
    setState(() {
      _carregando = true;
      _erro = null;
    });

    try {
      await ref.read(authProvider.notifier).login();

      if (mounted) {
        final authState = ref.read(authProvider);
        if (authState is Autenticado) {
          context.go(widget.redirectUrl ?? '/home');
        } else {
          setState(() => _erro = 'Login cancelado ou falhou.');
        }
      }
    } catch (e) {
      if (mounted) setState(() => _erro = 'Erro ao conectar: $e');
    } finally {
      if (mounted) setState(() => _carregando = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Padding(
            padding: const EdgeInsets.all(32),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.lock_outline, size: 72, color: Colors.indigo),
                const SizedBox(height: 24),
                Text('Bem-vindo',
                    style: Theme.of(context).textTheme.headlineMedium),
                const SizedBox(height: 8),
                Text('Faça login para continuar',
                    style: Theme.of(context).textTheme.bodyLarge),
                const SizedBox(height: 48),
                SizedBox(
                  width: double.infinity,
                  child: FilledButton.icon(
                    onPressed: _carregando ? null : _fazerLogin,
                    icon: _carregando
                        ? const SizedBox(
                            width: 20,
                            height: 20,
                            child: CircularProgressIndicator(
                                strokeWidth: 2, color: Colors.white))
                        : const Icon(Icons.login),
                    label: Text(_carregando ? 'Conectando...' : 'Entrar com SSO'),
                  ),
                ),
                if (_erro != null) ...[
                  const SizedBox(height: 16),
                  Text(_erro!,
                      style: const TextStyle(color: Colors.red),
                      textAlign: TextAlign.center),
                ],
                const SizedBox(height: 24),
                TextButton(
                  onPressed: () => context.go('/sobre'),
                  child: const Text('Continuar sem login'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**10. Tela protegida com dados do usuário**

```dart
// lib/features/home/home_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../../core/auth/auth_provider.dart';

class HomeScreen extends ConsumerWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authProvider);

    return switch (authState) {
      Autenticado(usuario: final user) => Scaffold(
          appBar: AppBar(
            title: const Text('Home'),
            actions: [
              if (user.isAdmin)
                IconButton(
                  icon: const Icon(Icons.admin_panel_settings),
                  tooltip: 'Admin',
                  onPressed: () => context.go('/admin'),
                ),
              IconButton(
                icon: const Icon(Icons.person),
                tooltip: 'Perfil',
                onPressed: () => context.go('/perfil'),
              ),
              IconButton(
                icon: const Icon(Icons.logout),
                tooltip: 'Sair',
                onPressed: () async {
                  await ref.read(authProvider.notifier).logout();
                  if (context.mounted) context.go('/');
                },
              ),
            ],
          ),
          body: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Olá, ${user.nome}!',
                    style: Theme.of(context).textTheme.headlineSmall),
                const SizedBox(height: 8),
                Text(user.email,
                    style: Theme.of(context).textTheme.bodyLarge),
                const SizedBox(height: 16),
                Wrap(
                  spacing: 8,
                  children: user.roles
                      .map((r) => Chip(label: Text(r)))
                      .toList(),
                ),
              ],
            ),
          ),
        ),
      _ => const Scaffold(
          body: Center(child: CircularProgressIndicator()),
        ),
    };
  }
}
```

**11. Dio interceptor com refresh automático**

```dart
// lib/core/network/auth_interceptor.dart
import 'package:dio/dio.dart';
import '../auth/auth_service.dart';

class AuthInterceptor extends Interceptor {
  final AuthService _authService;

  AuthInterceptor(this._authService);

  @override
  void onRequest(
      RequestOptions options, RequestInterceptorHandler handler) async {
    final token = await _authService.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      final user = await _authService.refreshSession();
      if (user != null) {
        final token = await _authService.getAccessToken();
        err.requestOptions.headers['Authorization'] = 'Bearer $token';
        final retryResponse = await Dio().fetch(err.requestOptions);
        return handler.resolve(retryResponse);
      }
    }
    handler.next(err);
  }
}
```

**12. main.dart**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'routes/app_router.dart';

void main() {
  runApp(const ProviderScope(child: App()));
}

class App extends ConsumerWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(routerProvider);

    return MaterialApp.router(
      title: 'Meu App',
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      routerConfig: router,
    );
  }
}
```

---

### Flutter (alternativo) — flutter_web_auth_2 + flutter_modular

Abordagem alternativa usando `flutter_web_auth_2` para o fluxo OAuth2 Authorization Code + PKCE via WebView do sistema, `flutter_modular` para injeção de dependências e route guards, e `flutter_secure_storage` para persistência de tokens.

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_modular: ^6.3.0
  flutter_web_auth_2: ^4.0.0
  flutter_secure_storage: ^9.2.0
  jwt_decoder: ^2.0.1
  dio: ^5.7.0
  crypto: ^3.0.0
```

```bash
flutter pub get
```

**2. Configuração Android — `android/app/build.gradle`**

```groovy
android {
    defaultConfig {
        // Scheme usado no redirect URI
        manifestPlaceholders += [
            'appAuthRedirectScheme': 'com.exemplo.app'
        ]
    }
}
```

**`android/app/src/main/AndroidManifest.xml`:**

```xml
<manifest>
  <application>
    <activity
      android:name="com.linusu.flutter_web_auth_2.CallbackActivity"
      android:exported="true">
      <intent-filter android:label="flutter_web_auth_2">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="com.exemplo.app" android:host="callback" />
      </intent-filter>
    </activity>
  </application>
</manifest>
```

**3. Configuração iOS — `ios/Runner/Info.plist`**

```xml
<dict>
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
      <key>CFBundleURLSchemes</key>
      <array>
        <string>com.exemplo.app</string>
      </array>
    </dict>
  </array>
</dict>
```

**4. Modelo do usuário** (mesmo do exemplo anterior)

```dart
// lib/core/auth/user_model.dart
class AppUser {
  final String id;
  final String nome;
  final String email;
  final List<String> roles;

  const AppUser({
    required this.id,
    required this.nome,
    required this.email,
    required this.roles,
  });

  bool get isAdmin => roles.contains('ADMIN');
  bool get isComum => roles.contains('COMUM');
  bool hasRole(String role) => roles.contains(role);
}
```

**5. Configuração OIDC**

```dart
// lib/core/auth/oidc_config.dart
abstract final class OidcConfig {
  static const String issuer = 'https://auth.exemplo.com/realms/meu-realm';
  static const String clientId = 'meu-app-mobile';
  static const String redirectUri = 'com.exemplo.app://callback';
  static const String callbackUrlScheme = 'com.exemplo.app';
  static const List<String> scopes = ['openid', 'profile', 'email', 'roles'];

  // Endpoints derivados do issuer
  static String get authorizationEndpoint => '$issuer/protocol/openid-connect/auth';
  static String get tokenEndpoint => '$issuer/protocol/openid-connect/token';
  static String get endSessionEndpoint => '$issuer/protocol/openid-connect/logout';
}
```

**6. Serviço de autenticação com flutter_web_auth_2**

```dart
// lib/core/auth/auth_service.dart
import 'dart:convert';
import 'dart:math';
import 'package:crypto/crypto.dart';
import 'package:dio/dio.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:flutter_web_auth_2/flutter_web_auth_2.dart';
import 'package:jwt_decoder/jwt_decoder.dart';
import 'oidc_config.dart';
import 'user_model.dart';

class AuthService {
  final _storage = const FlutterSecureStorage();
  final _dio = Dio();

  static const _keyAccessToken = 'access_token';
  static const _keyRefreshToken = 'refresh_token';
  static const _keyIdToken = 'id_token';

  // --- PKCE helpers ---
  String _gerarCodeVerifier() {
    final random = Random.secure();
    final bytes = List<int>.generate(32, (_) => random.nextInt(256));
    return base64UrlEncode(bytes).replaceAll('=', '');
  }

  String _gerarCodeChallenge(String codeVerifier) {
    final digest = sha256.convert(utf8.encode(codeVerifier));
    return base64UrlEncode(digest.bytes).replaceAll('=', '');
  }

  // --- Login ---
  Future<AppUser?> login() async {
    final codeVerifier = _gerarCodeVerifier();
    final codeChallenge = _gerarCodeChallenge(codeVerifier);
    final state = _gerarCodeVerifier(); // reutiliza como nonce

    final authUrl = Uri.parse(OidcConfig.authorizationEndpoint).replace(
      queryParameters: {
        'response_type': 'code',
        'client_id': OidcConfig.clientId,
        'redirect_uri': OidcConfig.redirectUri,
        'scope': OidcConfig.scopes.join(' '),
        'state': state,
        'code_challenge': codeChallenge,
        'code_challenge_method': 'S256',
        'prompt': 'login',
      },
    );

    // Abre o navegador do sistema para login
    final resultUrl = await FlutterWebAuth2.authenticate(
      url: authUrl.toString(),
      callbackUrlScheme: OidcConfig.callbackUrlScheme,
    );

    final uri = Uri.parse(resultUrl);
    final code = uri.queryParameters['code'];
    final returnedState = uri.queryParameters['state'];

    if (code == null || returnedState != state) return null;

    // Trocar code por tokens
    final tokenResponse = await _dio.post(
      OidcConfig.tokenEndpoint,
      options: Options(contentType: Headers.formUrlEncodedContentType),
      data: {
        'grant_type': 'authorization_code',
        'client_id': OidcConfig.clientId,
        'redirect_uri': OidcConfig.redirectUri,
        'code': code,
        'code_verifier': codeVerifier,
      },
    );

    final tokens = tokenResponse.data as Map<String, dynamic>;
    await _salvarTokens(
      accessToken: tokens['access_token'],
      refreshToken: tokens['refresh_token'],
      idToken: tokens['id_token'],
    );

    return _decodificarUsuario(tokens['access_token']);
  }

  // --- Logout ---
  Future<void> logout() async {
    final idToken = await _storage.read(key: _keyIdToken);

    if (idToken != null) {
      try {
        final logoutUrl = Uri.parse(OidcConfig.endSessionEndpoint).replace(
          queryParameters: {
            'id_token_hint': idToken,
            'post_logout_redirect_uri': OidcConfig.redirectUri,
          },
        );

        await FlutterWebAuth2.authenticate(
          url: logoutUrl.toString(),
          callbackUrlScheme: OidcConfig.callbackUrlScheme,
        );
      } catch (_) {
        // Logout local mesmo se o servidor falhar
      }
    }

    await _limparTokens();
  }

  // --- Refresh ---
  Future<AppUser?> refreshSession() async {
    final refreshToken = await _storage.read(key: _keyRefreshToken);
    if (refreshToken == null) return null;

    try {
      final response = await _dio.post(
        OidcConfig.tokenEndpoint,
        options: Options(contentType: Headers.formUrlEncodedContentType),
        data: {
          'grant_type': 'refresh_token',
          'client_id': OidcConfig.clientId,
          'refresh_token': refreshToken,
        },
      );

      final tokens = response.data as Map<String, dynamic>;
      await _salvarTokens(
        accessToken: tokens['access_token'],
        refreshToken: tokens['refresh_token'] ?? refreshToken,
        idToken: tokens['id_token'],
      );

      return _decodificarUsuario(tokens['access_token']);
    } catch (_) {
      await _limparTokens();
      return null;
    }
  }

  // --- Sessão ativa ---
  Future<AppUser?> recuperarSessao() async {
    final accessToken = await _storage.read(key: _keyAccessToken);
    if (accessToken == null) return null;

    if (JwtDecoder.isExpired(accessToken)) {
      return refreshSession();
    }

    return _decodificarUsuario(accessToken);
  }

  // --- Access token para requisições ---
  Future<String?> getAccessToken() async {
    final token = await _storage.read(key: _keyAccessToken);
    if (token == null) return null;

    if (JwtDecoder.isExpired(token)) {
      final user = await refreshSession();
      if (user == null) return null;
      return await _storage.read(key: _keyAccessToken);
    }

    return token;
  }

  // --- Estado atual ---
  AppUser _decodificarUsuario(String accessToken) {
    final payload = JwtDecoder.decode(accessToken);
    final realmAccess = payload['realm_access'] as Map<String, dynamic>?;
    final roles = (realmAccess?['roles'] as List<dynamic>?)
            ?.map((r) => r.toString())
            .toList() ??
        [];

    return AppUser(
      id: payload['sub'] as String,
      nome: payload['name'] as String? ??
          payload['preferred_username'] as String? ??
          '',
      email: payload['email'] as String? ?? '',
      roles: roles,
    );
  }

  Future<void> _salvarTokens({
    required String accessToken,
    required String refreshToken,
    required String idToken,
  }) async {
    await Future.wait([
      _storage.write(key: _keyAccessToken, value: accessToken),
      _storage.write(key: _keyRefreshToken, value: refreshToken),
      _storage.write(key: _keyIdToken, value: idToken),
    ]);
  }

  Future<void> _limparTokens() async {
    await Future.wait([
      _storage.delete(key: _keyAccessToken),
      _storage.delete(key: _keyRefreshToken),
      _storage.delete(key: _keyIdToken),
    ]);
  }
}
```

> **2FA e MFA:** assim como no exemplo com `flutter_appauth`, o `FlutterWebAuth2.authenticate()` abre o navegador do sistema (Chrome Custom Tab / ASWebAuthenticationSession) que é um processo independente do app. A sessão do navegador persiste enquanto o usuário sai para buscar o código 2FA no app autenticador. Ao retornar e completar o 2FA, o IdP redireciona para o callback e o `Future` resolve normalmente. Todo o fluxo de segundo fator é gerenciado pelo identity provider — o app não precisa de lógica adicional.

**7. Store de autenticação (ChangeNotifier para uso com Modular)**

```dart
// lib/core/auth/auth_store.dart
import 'package:flutter/foundation.dart';
import 'auth_service.dart';
import 'user_model.dart';

enum AuthStatus { loading, autenticado, naoAutenticado }

class AuthStore extends ChangeNotifier {
  final AuthService _service;

  AuthStore(this._service) {
    _inicializar();
  }

  AuthStatus _status = AuthStatus.loading;
  AppUser? _usuario;

  AuthStatus get status => _status;
  AppUser? get usuario => _usuario;
  bool get isAutenticado => _status == AuthStatus.autenticado;

  Future<void> _inicializar() async {
    _usuario = await _service.recuperarSessao();
    _status = _usuario != null
        ? AuthStatus.autenticado
        : AuthStatus.naoAutenticado;
    notifyListeners();
  }

  Future<void> login() async {
    _status = AuthStatus.loading;
    notifyListeners();

    _usuario = await _service.login();
    _status = _usuario != null
        ? AuthStatus.autenticado
        : AuthStatus.naoAutenticado;
    notifyListeners();
  }

  Future<void> logout() async {
    await _service.logout();
    _usuario = null;
    _status = AuthStatus.naoAutenticado;
    notifyListeners();
  }

  Future<String?> getAccessToken() => _service.getAccessToken();
}
```

**8. Guard de autenticação**

```dart
// lib/core/auth/auth_guard.dart
import 'dart:async';
import 'package:flutter_modular/flutter_modular.dart';
import 'auth_store.dart';

class AuthGuard extends RouteGuard {
  AuthGuard() : super(redirectTo: '/login/');

  @override
  FutureOr<bool> canActivate(String path, ModularRoute route) {
    final authStore = Modular.get<AuthStore>();
    return authStore.isAutenticado;
  }
}
```

**9. Guard por role**

```dart
// lib/core/auth/role_guard.dart
import 'dart:async';
import 'package:flutter_modular/flutter_modular.dart';
import 'auth_store.dart';

class RoleGuard extends RouteGuard {
  final List<String> rolesPermitidas;

  RoleGuard({required this.rolesPermitidas})
      : super(redirectTo: '/acesso-negado/');

  @override
  FutureOr<bool> canActivate(String path, ModularRoute route) {
    final authStore = Modular.get<AuthStore>();
    if (!authStore.isAutenticado) return false;

    final usuario = authStore.usuario;
    if (usuario == null) return false;

    return rolesPermitidas.any((role) => usuario.hasRole(role));
  }
}
```

**10. Módulo raiz com rotas e guards**

```dart
// lib/app_module.dart
import 'package:flutter_modular/flutter_modular.dart';
import 'core/auth/auth_service.dart';
import 'core/auth/auth_store.dart';
import 'core/auth/auth_guard.dart';
import 'core/auth/role_guard.dart';
import 'core/network/dio_client.dart';
import 'features/public/sobre_screen.dart';
import 'features/auth/login_screen.dart';
import 'features/home/home_screen.dart';
import 'features/perfil/perfil_screen.dart';
import 'features/admin/admin_module.dart';
import 'features/public/acesso_negado_screen.dart';

class AppModule extends Module {
  @override
  void binds(Injector i) {
    i.addSingleton<AuthService>(AuthService.new);
    i.addSingleton<AuthStore>(() => AuthStore(i.get<AuthService>()));
    i.addSingleton<DioClient>(() => DioClient(i.get<AuthStore>()));
  }

  @override
  void routes(RouteManager r) {
    // --- Rotas públicas ---
    r.child('/', child: (_) => const SobreScreen());
    r.child('/sobre/', child: (_) => const SobreScreen());
    r.child('/login/', child: (_) => const LoginScreen());
    r.child('/acesso-negado/', child: (_) => const AcessoNegadoScreen());

    // --- Rotas autenticadas (qualquer role) ---
    r.child(
      '/home/',
      child: (_) => const HomeScreen(),
      guards: [AuthGuard()],
    );
    r.child(
      '/perfil/',
      child: (_) => const PerfilScreen(),
      guards: [AuthGuard()],
    );

    // --- Rotas somente ADMIN (módulo separado) ---
    r.module(
      '/admin',
      module: AdminModule(),
      guards: [AuthGuard(), RoleGuard(rolesPermitidas: ['ADMIN'])],
    );
  }
}
```

**11. Módulo admin com rotas internas**

```dart
// lib/features/admin/admin_module.dart
import 'package:flutter_modular/flutter_modular.dart';
import 'presentation/admin_dashboard_screen.dart';
import 'presentation/admin_usuarios_screen.dart';

class AdminModule extends Module {
  @override
  void routes(RouteManager r) {
    r.child('/', child: (_) => const AdminDashboardScreen());
    r.child('/usuarios/', child: (_) => const AdminUsuariosScreen());
  }
}
```

**12. Tela de login**

```dart
// lib/features/auth/login_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';
import '../../core/auth/auth_store.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _authStore = Modular.get<AuthStore>();
  bool _carregando = false;
  String? _erro;

  Future<void> _fazerLogin() async {
    setState(() {
      _carregando = true;
      _erro = null;
    });

    try {
      await _authStore.login();

      if (mounted && _authStore.isAutenticado) {
        Modular.to.navigate('/home/');
      } else {
        setState(() => _erro = 'Login cancelado ou falhou.');
      }
    } catch (e) {
      if (mounted) setState(() => _erro = 'Erro ao conectar: $e');
    } finally {
      if (mounted) setState(() => _carregando = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Padding(
            padding: const EdgeInsets.all(32),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.lock_outline, size: 72, color: Colors.indigo),
                const SizedBox(height: 24),
                Text('Bem-vindo',
                    style: Theme.of(context).textTheme.headlineMedium),
                const SizedBox(height: 8),
                Text('Faça login para continuar',
                    style: Theme.of(context).textTheme.bodyLarge),
                const SizedBox(height: 48),
                SizedBox(
                  width: double.infinity,
                  child: FilledButton.icon(
                    onPressed: _carregando ? null : _fazerLogin,
                    icon: _carregando
                        ? const SizedBox(
                            width: 20, height: 20,
                            child: CircularProgressIndicator(
                                strokeWidth: 2, color: Colors.white))
                        : const Icon(Icons.login),
                    label: Text(_carregando ? 'Conectando...' : 'Entrar com SSO'),
                  ),
                ),
                if (_erro != null) ...[
                  const SizedBox(height: 16),
                  Text(_erro!,
                      style: const TextStyle(color: Colors.red),
                      textAlign: TextAlign.center),
                ],
                const SizedBox(height: 24),
                TextButton(
                  onPressed: () => Modular.to.navigate('/sobre/'),
                  child: const Text('Continuar sem login'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**13. Tela protegida com dados do usuário**

```dart
// lib/features/home/home_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';
import '../../core/auth/auth_store.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final authStore = Modular.get<AuthStore>();

    return AnimatedBuilder(
      animation: authStore,
      builder: (context, _) {
        final usuario = authStore.usuario;
        if (usuario == null) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }

        return Scaffold(
          appBar: AppBar(
            title: const Text('Home'),
            actions: [
              if (usuario.isAdmin)
                IconButton(
                  icon: const Icon(Icons.admin_panel_settings),
                  tooltip: 'Admin',
                  onPressed: () => Modular.to.navigate('/admin/'),
                ),
              IconButton(
                icon: const Icon(Icons.person),
                tooltip: 'Perfil',
                onPressed: () => Modular.to.pushNamed('/perfil/'),
              ),
              IconButton(
                icon: const Icon(Icons.logout),
                tooltip: 'Sair',
                onPressed: () async {
                  await authStore.logout();
                  Modular.to.navigate('/');
                },
              ),
            ],
          ),
          body: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Olá, ${usuario.nome}!',
                    style: Theme.of(context).textTheme.headlineSmall),
                const SizedBox(height: 8),
                Text(usuario.email,
                    style: Theme.of(context).textTheme.bodyLarge),
                const SizedBox(height: 16),
                Wrap(
                  spacing: 8,
                  children: usuario.roles
                      .map((r) => Chip(label: Text(r)))
                      .toList(),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}
```

**14. Dio client com interceptor de token**

```dart
// lib/core/network/dio_client.dart
import 'package:dio/dio.dart';
import '../auth/auth_store.dart';

class DioClient {
  final Dio dio;
  final AuthStore _authStore;

  DioClient(this._authStore)
      : dio = Dio(BaseOptions(baseUrl: 'https://api.exemplo.com')) {
    dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) async {
        final token = await _authStore.getAccessToken();
        if (token != null) {
          options.headers['Authorization'] = 'Bearer $token';
        }
        handler.next(options);
      },
      onError: (error, handler) async {
        if (error.response?.statusCode == 401) {
          // Tenta refresh e retry
          final novoToken = await _authStore.getAccessToken();
          if (novoToken != null) {
            error.requestOptions.headers['Authorization'] =
                'Bearer $novoToken';
            final retryResponse = await Dio().fetch(error.requestOptions);
            return handler.resolve(retryResponse);
          }
          // Refresh falhou — logout
          await _authStore.logout();
        }
        handler.next(error);
      },
    ));
  }
}
```

**15. main.dart**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';
import 'app_module.dart';

void main() {
  runApp(
    ModularApp(
      module: AppModule(),
      child: const App(),
    ),
  );
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Meu App',
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      routerConfig: Modular.routerConfig,
    );
  }
}
```

**Comparação entre as duas abordagens Flutter:**

| Aspecto | flutter_appauth + GoRouter | flutter_web_auth_2 + flutter_modular |
|---|---|---|
| Fluxo OAuth2 | Biblioteca dedicada (AppAuth) | Manual via WebView do sistema + Dio |
| PKCE | Automático | Implementação manual (`sha256` + `base64Url`) |
| Token exchange | Automático | POST manual para `token_endpoint` |
| Navegação | GoRouter (declarativo) | flutter_modular (declarativo + DI) |
| Route guard | `GoRouter.redirect` global | Classes `RouteGuard` por rota |
| Guard por role | Verificação inline no redirect | Classe `RoleGuard` separada e reutilizável |
| Injeção de dependências | Riverpod providers | Modular `Injector` (binds) |
| Estado de auth | Riverpod `NotifierProvider` | `ChangeNotifier` + `AnimatedBuilder` |
| Modularização | Arquivo único de rotas | Módulos separados (`AdminModule`, etc.) |
| Controle sobre o fluxo | Menor (AppAuth abstrai) | Maior (cada etapa é explícita) |

---

### React Native — expo-auth-session + React Navigation

**1. Dependências**

```bash
npx expo install expo-auth-session expo-crypto expo-web-browser expo-secure-store
npm install @react-navigation/native @react-navigation/native-stack
npx expo install react-native-screens react-native-safe-area-context
npm install jwt-decode
```

**2. Configuração OIDC**

```typescript
// src/core/auth/oidcConfig.ts
export const OIDC_CONFIG = {
  issuer: 'https://auth.exemplo.com/realms/meu-realm',
  clientId: 'meu-app-mobile',
  scopes: ['openid', 'profile', 'email', 'roles'],
  redirectUri: 'com.exemplo.app://callback',
} as const;
```

**3. Modelo do usuário**

```typescript
// src/core/auth/types.ts
export interface AppUser {
  id: string;
  nome: string;
  email: string;
  roles: string[];
}

export type AuthState =
  | { status: 'loading' }
  | { status: 'autenticado'; usuario: AppUser }
  | { status: 'nao_autenticado' };

export type UserRole = 'COMUM' | 'ADMIN';
```

**4. Serviço de autenticação**

```typescript
// src/core/auth/authService.ts
import * as AuthSession from 'expo-auth-session';
import * as SecureStore from 'expo-secure-store';
import * as WebBrowser from 'expo-web-browser';
import { jwtDecode } from 'jwt-decode';
import { OIDC_CONFIG } from './oidcConfig';
import { AppUser } from './types';

WebBrowser.maybeCompleteAuthSession();

const KEYS = {
  accessToken: 'access_token',
  refreshToken: 'refresh_token',
  idToken: 'id_token',
} as const;

interface JwtPayload {
  sub: string;
  name?: string;
  preferred_username?: string;
  email?: string;
  realm_access?: { roles: string[] };
  exp: number;
}

function decodificarUsuario(accessToken: string): AppUser {
  const payload = jwtDecode<JwtPayload>(accessToken);
  return {
    id: payload.sub,
    nome: payload.name ?? payload.preferred_username ?? '',
    email: payload.email ?? '',
    roles: payload.realm_access?.roles ?? [],
  };
}

function tokenExpirado(token: string): boolean {
  try {
    const { exp } = jwtDecode<{ exp: number }>(token);
    return Date.now() >= exp * 1000;
  } catch {
    return true;
  }
}

// Descobrir endpoints via issuer
async function getDiscovery() {
  return AuthSession.fetchDiscoveryAsync(OIDC_CONFIG.issuer);
}

export async function login(): Promise<AppUser | null> {
  const discovery = await getDiscovery();
  const request = new AuthSession.AuthRequest({
    clientId: OIDC_CONFIG.clientId,
    redirectUri: OIDC_CONFIG.redirectUri,
    scopes: [...OIDC_CONFIG.scopes],
    responseType: AuthSession.ResponseType.Code,
    usePKCE: true,
    prompt: AuthSession.Prompt.Login,
  });

  const result = await request.promptAsync(discovery);

  if (result.type !== 'success' || !result.params.code) return null;

  const tokenResult = await AuthSession.exchangeCodeAsync(
    {
      clientId: OIDC_CONFIG.clientId,
      redirectUri: OIDC_CONFIG.redirectUri,
      code: result.params.code,
      extraParams: { code_verifier: request.codeVerifier! },
    },
    discovery,
  );

  await salvarTokens(tokenResult);
  return decodificarUsuario(tokenResult.accessToken);
}

export async function logout(): Promise<void> {
  const idToken = await SecureStore.getItemAsync(KEYS.idToken);
  const discovery = await getDiscovery();

  try {
    if (discovery.endSessionEndpoint && idToken) {
      await WebBrowser.openAuthSessionAsync(
        `${discovery.endSessionEndpoint}?id_token_hint=${idToken}&post_logout_redirect_uri=${OIDC_CONFIG.redirectUri}`,
        OIDC_CONFIG.redirectUri,
      );
    }
  } catch {
    // Logout local mesmo se falhar
  }

  await limparTokens();
}

export async function refreshSession(): Promise<AppUser | null> {
  const refreshToken = await SecureStore.getItemAsync(KEYS.refreshToken);
  if (!refreshToken) return null;

  try {
    const discovery = await getDiscovery();
    const tokenResult = await AuthSession.refreshAsync(
      {
        clientId: OIDC_CONFIG.clientId,
        refreshToken,
        scopes: [...OIDC_CONFIG.scopes],
      },
      discovery,
    );

    await salvarTokens(tokenResult);
    return decodificarUsuario(tokenResult.accessToken);
  } catch {
    await limparTokens();
    return null;
  }
}

export async function recuperarSessao(): Promise<AppUser | null> {
  const accessToken = await SecureStore.getItemAsync(KEYS.accessToken);
  if (!accessToken) return null;

  if (tokenExpirado(accessToken)) {
    return refreshSession();
  }

  return decodificarUsuario(accessToken);
}

export async function getAccessToken(): Promise<string | null> {
  const token = await SecureStore.getItemAsync(KEYS.accessToken);
  if (!token) return null;

  if (tokenExpirado(token)) {
    const user = await refreshSession();
    if (!user) return null;
    return SecureStore.getItemAsync(KEYS.accessToken);
  }

  return token;
}

async function salvarTokens(result: AuthSession.TokenResponse) {
  await Promise.all([
    SecureStore.setItemAsync(KEYS.accessToken, result.accessToken),
    result.refreshToken
      ? SecureStore.setItemAsync(KEYS.refreshToken, result.refreshToken)
      : Promise.resolve(),
    result.idToken
      ? SecureStore.setItemAsync(KEYS.idToken, result.idToken)
      : Promise.resolve(),
  ]);
}

async function limparTokens() {
  await Promise.all([
    SecureStore.deleteItemAsync(KEYS.accessToken),
    SecureStore.deleteItemAsync(KEYS.refreshToken),
    SecureStore.deleteItemAsync(KEYS.idToken),
  ]);
}
```

> **2FA e MFA:** o `promptAsync` do `expo-auth-session` abre o navegador do sistema, onde o identity provider gerencia todo o fluxo de autenticação — incluindo segundo fator. O usuário pode alternar entre o navegador e o app autenticador para copiar o código 2FA sem perder a sessão. O `Promise` retornado por `promptAsync` aguarda o redirect de callback, independentemente de quanto tempo o usuário leve na etapa de 2FA.

**5. Context de autenticação**

```tsx
// src/core/auth/AuthContext.tsx
import React, { createContext, useContext, useEffect, useState, useCallback } from 'react';
import * as authService from './authService';
import { AppUser, AuthState } from './types';

interface AuthContextType {
  state: AuthState;
  usuario: AppUser | null;
  login: () => Promise<void>;
  logout: () => Promise<void>;
  hasRole: (role: string) => boolean;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState<AuthState>({ status: 'loading' });

  useEffect(() => {
    authService.recuperarSessao().then((usuario) => {
      setState(
        usuario
          ? { status: 'autenticado', usuario }
          : { status: 'nao_autenticado' },
      );
    });
  }, []);

  const handleLogin = useCallback(async () => {
    setState({ status: 'loading' });
    const usuario = await authService.login();
    setState(
      usuario
        ? { status: 'autenticado', usuario }
        : { status: 'nao_autenticado' },
    );
  }, []);

  const handleLogout = useCallback(async () => {
    await authService.logout();
    setState({ status: 'nao_autenticado' });
  }, []);

  const usuario =
    state.status === 'autenticado' ? state.usuario : null;

  const hasRole = useCallback(
    (role: string) => usuario?.roles.includes(role) ?? false,
    [usuario],
  );

  return (
    <AuthContext.Provider
      value={{ state, usuario, login: handleLogin, logout: handleLogout, hasRole }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth deve estar dentro de AuthProvider');
  return ctx;
}
```

**6. Navegação com route guard**

```tsx
// src/navigation/AppNavigator.tsx
import React from 'react';
import { ActivityIndicator, View } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { useAuth } from '../core/auth/AuthContext';
import { LoginScreen } from '../features/auth/LoginScreen';
import { HomeScreen } from '../features/home/HomeScreen';
import { PerfilScreen } from '../features/perfil/PerfilScreen';
import { AdminScreen } from '../features/admin/AdminScreen';
import { SobreScreen } from '../features/public/SobreScreen';

export type RootStackParamList = {
  Sobre: undefined;
  Login: { redirect?: string };
  Home: undefined;
  Perfil: undefined;
  Admin: undefined;
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export function AppNavigator() {
  const { state, hasRole } = useAuth();

  if (state.status === 'loading') {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  const autenticado = state.status === 'autenticado';

  return (
    <NavigationContainer>
      <Stack.Navigator>
        {/* Rotas públicas — sempre disponíveis */}
        <Stack.Screen name="Sobre" component={SobreScreen} />

        {!autenticado ? (
          // Não autenticado — só mostra login
          <Stack.Screen
            name="Login"
            component={LoginScreen}
            options={{ headerShown: false }}
          />
        ) : (
          // Autenticado — rotas protegidas
          <>
            <Stack.Screen name="Home" component={HomeScreen} />
            <Stack.Screen name="Perfil" component={PerfilScreen} />

            {/* Rota somente ADMIN */}
            {hasRole('ADMIN') && (
              <Stack.Screen name="Admin" component={AdminScreen} />
            )}
          </>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

**7. Tela de login**

```tsx
// src/features/auth/LoginScreen.tsx
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, ActivityIndicator, StyleSheet } from 'react-native';
import { useAuth } from '../../core/auth/AuthContext';

export function LoginScreen() {
  const { login } = useAuth();
  const [carregando, setCarregando] = useState(false);
  const [erro, setErro] = useState<string | null>(null);

  const handleLogin = async () => {
    setCarregando(true);
    setErro(null);
    try {
      await login();
    } catch (e) {
      setErro('Erro ao conectar. Tente novamente.');
    } finally {
      setCarregando(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.icone}>🔒</Text>
      <Text style={styles.titulo}>Bem-vindo</Text>
      <Text style={styles.subtitulo}>Faça login para continuar</Text>

      <TouchableOpacity
        style={[styles.botao, carregando && styles.botaoDesabilitado]}
        onPress={handleLogin}
        disabled={carregando}
      >
        {carregando ? (
          <ActivityIndicator color="#fff" />
        ) : (
          <Text style={styles.botaoTexto}>Entrar com SSO</Text>
        )}
      </TouchableOpacity>

      {erro && <Text style={styles.erro}>{erro}</Text>}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 32,
    backgroundColor: '#fff',
  },
  icone: { fontSize: 64, marginBottom: 24 },
  titulo: { fontSize: 28, fontWeight: 'bold', marginBottom: 8 },
  subtitulo: { fontSize: 16, color: '#666', marginBottom: 48 },
  botao: {
    width: '100%',
    backgroundColor: '#4338ca',
    padding: 16,
    borderRadius: 12,
    alignItems: 'center',
  },
  botaoDesabilitado: { opacity: 0.6 },
  botaoTexto: { color: '#fff', fontSize: 16, fontWeight: '600' },
  erro: { color: '#e53935', marginTop: 16, textAlign: 'center' },
});
```

**8. Tela protegida com dados do usuário**

```tsx
// src/features/home/HomeScreen.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { useAuth } from '../../core/auth/AuthContext';
import { RootStackParamList } from '../../navigation/AppNavigator';

type Nav = NativeStackNavigationProp<RootStackParamList>;

export function HomeScreen() {
  const { usuario, logout, hasRole } = useAuth();
  const navigation = useNavigation<Nav>();

  if (!usuario) return null;

  return (
    <View style={styles.container}>
      <Text style={styles.saudacao}>Olá, {usuario.nome}!</Text>
      <Text style={styles.email}>{usuario.email}</Text>

      <View style={styles.roles}>
        {usuario.roles.map((role) => (
          <View key={role} style={styles.chip}>
            <Text style={styles.chipTexto}>{role}</Text>
          </View>
        ))}
      </View>

      <View style={styles.acoes}>
        <TouchableOpacity
          style={styles.botao}
          onPress={() => navigation.navigate('Perfil')}
        >
          <Text style={styles.botaoTexto}>Meu Perfil</Text>
        </TouchableOpacity>

        {hasRole('ADMIN') && (
          <TouchableOpacity
            style={[styles.botao, styles.botaoAdmin]}
            onPress={() => navigation.navigate('Admin')}
          >
            <Text style={styles.botaoTexto}>Painel Admin</Text>
          </TouchableOpacity>
        )}

        <TouchableOpacity
          style={[styles.botao, styles.botaoLogout]}
          onPress={logout}
        >
          <Text style={styles.botaoTexto}>Sair</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24 },
  saudacao: { fontSize: 24, fontWeight: 'bold' },
  email: { fontSize: 16, color: '#666', marginTop: 4 },
  roles: { flexDirection: 'row', gap: 8, marginTop: 16 },
  chip: {
    backgroundColor: '#e0e7ff',
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 16,
  },
  chipTexto: { color: '#4338ca', fontSize: 13, fontWeight: '600' },
  acoes: { marginTop: 32, gap: 12 },
  botao: {
    backgroundColor: '#4338ca',
    padding: 16,
    borderRadius: 12,
    alignItems: 'center',
  },
  botaoAdmin: { backgroundColor: '#7c3aed' },
  botaoLogout: { backgroundColor: '#dc2626' },
  botaoTexto: { color: '#fff', fontSize: 16, fontWeight: '600' },
});
```

**9. App.tsx**

```tsx
// App.tsx
import React from 'react';
import { AuthProvider } from './src/core/auth/AuthContext';
import { AppNavigator } from './src/navigation/AppNavigator';

export default function App() {
  return (
    <AuthProvider>
      <AppNavigator />
    </AuthProvider>
  );
}
```

---

### React Web — oidc-client-ts + React Router

**1. Dependências**

```bash
npm install oidc-client-ts react-oidc-context react-router-dom
```

**2. Configuração OIDC**

```typescript
// src/core/auth/oidcConfig.ts
import { WebStorageStateStore } from 'oidc-client-ts';

export const oidcConfig = {
  authority: 'https://auth.exemplo.com/realms/meu-realm',
  client_id: 'meu-app-web',
  redirect_uri: `${window.location.origin}/callback`,
  post_logout_redirect_uri: window.location.origin,
  response_type: 'code',
  scope: 'openid profile email roles',
  automaticSilentRenew: true,
  userStore: new WebStorageStateStore({ store: window.localStorage }),
};
```

**3. Modelo e utilitários**

```typescript
// src/core/auth/types.ts
export interface AppUser {
  id: string;
  nome: string;
  email: string;
  roles: string[];
}

export type UserRole = 'COMUM' | 'ADMIN';
```

```typescript
// src/core/auth/parseUser.ts
import { User } from 'oidc-client-ts';
import { AppUser } from './types';

export function parseUser(oidcUser: User): AppUser {
  const profile = oidcUser.profile;
  const realmAccess = (profile as Record<string, unknown>).realm_access as
    | { roles: string[] }
    | undefined;

  return {
    id: profile.sub,
    nome: (profile.name ?? profile.preferred_username ?? '') as string,
    email: (profile.email ?? '') as string,
    roles: realmAccess?.roles ?? [],
  };
}
```

> **2FA e MFA:** no React Web, o `signinRedirect` do `oidc-client-ts` redireciona o navegador inteiro para a página do identity provider. O 2FA acontece nativamente na mesma aba — o usuário pode abrir outra aba ou app para buscar o código sem perder a sessão. Ao completar, o IdP redireciona de volta para `/callback`. Não há lógica adicional necessária.

**4. Hook de autenticação**

```typescript
// src/core/auth/useAppAuth.ts
import { useAuth } from 'react-oidc-context';
import { useMemo, useCallback } from 'react';
import { parseUser } from './parseUser';
import { AppUser } from './types';

export function useAppAuth() {
  const auth = useAuth();

  const usuario: AppUser | null = useMemo(
    () => (auth.user ? parseUser(auth.user) : null),
    [auth.user],
  );

  const hasRole = useCallback(
    (role: string) => usuario?.roles.includes(role) ?? false,
    [usuario],
  );

  const accessToken = auth.user?.access_token ?? null;

  return {
    isLoading: auth.isLoading,
    isAuthenticated: auth.isAuthenticated,
    usuario,
    accessToken,
    hasRole,
    login: () => auth.signinRedirect(),
    logout: () =>
      auth.signoutRedirect({ post_logout_redirect_uri: window.location.origin }),
  };
}
```

**5. Componente de route guard**

```tsx
// src/core/auth/ProtectedRoute.tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAppAuth } from './useAppAuth';
import { UserRole } from './types';

interface Props {
  children: React.ReactNode;
  roles?: UserRole[];
}

export function ProtectedRoute({ children, roles }: Props) {
  const { isLoading, isAuthenticated, hasRole } = useAppAuth();
  const location = useLocation();

  if (isLoading) {
    return (
      <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100vh' }}>
        <p>Carregando...</p>
      </div>
    );
  }

  if (!isAuthenticated) {
    return <Navigate to={`/login?redirect=${encodeURIComponent(location.pathname)}`} replace />;
  }

  if (roles && roles.length > 0 && !roles.some((r) => hasRole(r))) {
    return <Navigate to="/acesso-negado" replace />;
  }

  return <>{children}</>;
}
```

**6. Router com rotas públicas e protegidas**

```tsx
// src/routes/AppRouter.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { ProtectedRoute } from '../core/auth/ProtectedRoute';
import { LoginPage } from '../features/auth/LoginPage';
import { CallbackPage } from '../features/auth/CallbackPage';
import { HomePage } from '../features/home/HomePage';
import { PerfilPage } from '../features/perfil/PerfilPage';
import { AdminPage } from '../features/admin/AdminPage';
import { SobrePage } from '../features/public/SobrePage';
import { AcessoNegadoPage } from '../features/public/AcessoNegadoPage';
import { Layout } from '../components/Layout';

export function AppRouter() {
  return (
    <BrowserRouter>
      <Routes>
        <Route element={<Layout />}>
          {/* Rotas públicas */}
          <Route path="/" element={<SobrePage />} />
          <Route path="/sobre" element={<SobrePage />} />
          <Route path="/login" element={<LoginPage />} />
          <Route path="/callback" element={<CallbackPage />} />
          <Route path="/acesso-negado" element={<AcessoNegadoPage />} />

          {/* Rotas autenticadas — qualquer role */}
          <Route
            path="/home"
            element={
              <ProtectedRoute>
                <HomePage />
              </ProtectedRoute>
            }
          />
          <Route
            path="/perfil"
            element={
              <ProtectedRoute>
                <PerfilPage />
              </ProtectedRoute>
            }
          />

          {/* Rotas somente ADMIN */}
          <Route
            path="/admin"
            element={
              <ProtectedRoute roles={['ADMIN']}>
                <AdminPage />
              </ProtectedRoute>
            }
          />
          <Route
            path="/admin/usuarios"
            element={
              <ProtectedRoute roles={['ADMIN']}>
                <AdminPage />
              </ProtectedRoute>
            }
          />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

**7. Página de login**

```tsx
// src/features/auth/LoginPage.tsx
import { useSearchParams } from 'react-router-dom';
import { useAppAuth } from '../../core/auth/useAppAuth';
import styles from './LoginPage.module.css';

export function LoginPage() {
  const { login, isAuthenticated } = useAppAuth();
  const [params] = useSearchParams();
  const redirect = params.get('redirect') ?? '/home';

  if (isAuthenticated) {
    window.location.href = redirect;
    return null;
  }

  return (
    <div className={styles.container}>
      <div className={styles.card}>
        <span className={styles.icone}>🔒</span>
        <h1 className={styles.titulo}>Bem-vindo</h1>
        <p className={styles.subtitulo}>Faça login para continuar</p>
        <button className={styles.botao} onClick={() => login()}>
          Entrar com SSO
        </button>
        <a href="/sobre" className={styles.link}>
          Continuar sem login
        </a>
      </div>
    </div>
  );
}
```

```css
/* src/features/auth/LoginPage.module.css */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 80vh;
}
.card {
  text-align: center;
  max-width: 400px;
  padding: 48px 32px;
}
.icone { font-size: 64px; }
.titulo { font-size: 1.75rem; margin: 24px 0 8px; }
.subtitulo { color: #666; margin-bottom: 48px; }
.botao {
  width: 100%;
  padding: 14px;
  background: #4338ca;
  color: #fff;
  border: none;
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
}
.botao:hover { background: #3730a3; }
.link { display: block; margin-top: 24px; color: #4338ca; }
```

**8. Callback OIDC**

```tsx
// src/features/auth/CallbackPage.tsx
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from 'react-oidc-context';

export function CallbackPage() {
  const auth = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if (!auth.isLoading && auth.isAuthenticated) {
      const redirect = sessionStorage.getItem('auth_redirect') ?? '/home';
      sessionStorage.removeItem('auth_redirect');
      navigate(redirect, { replace: true });
    }
  }, [auth.isLoading, auth.isAuthenticated, navigate]);

  return (
    <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '80vh' }}>
      <p>Processando login...</p>
    </div>
  );
}
```

**9. Página protegida com dados do usuário**

```tsx
// src/features/home/HomePage.tsx
import { useNavigate } from 'react-router-dom';
import { useAppAuth } from '../../core/auth/useAppAuth';
import styles from './HomePage.module.css';

export function HomePage() {
  const { usuario, logout, hasRole } = useAppAuth();
  const navigate = useNavigate();

  if (!usuario) return null;

  return (
    <div className={styles.container}>
      <h1>Olá, {usuario.nome}!</h1>
      <p className={styles.email}>{usuario.email}</p>

      <div className={styles.roles}>
        {usuario.roles.map((role) => (
          <span key={role} className={styles.chip}>
            {role}
          </span>
        ))}
      </div>

      <div className={styles.acoes}>
        <button onClick={() => navigate('/perfil')} className={styles.botao}>
          Meu Perfil
        </button>

        {hasRole('ADMIN') && (
          <button
            onClick={() => navigate('/admin')}
            className={`${styles.botao} ${styles.admin}`}
          >
            Painel Admin
          </button>
        )}

        <button
          onClick={() => logout()}
          className={`${styles.botao} ${styles.logout}`}
        >
          Sair
        </button>
      </div>
    </div>
  );
}
```

```css
/* src/features/home/HomePage.module.css */
.container { padding: 24px; max-width: 600px; }
.email { color: #666; margin-top: 4px; }
.roles { display: flex; gap: 8px; margin-top: 16px; }
.chip {
  background: #e0e7ff;
  color: #4338ca;
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 13px;
  font-weight: 600;
}
.acoes { margin-top: 32px; display: flex; flex-direction: column; gap: 12px; }
.botao {
  padding: 14px;
  border: none;
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 600;
  color: #fff;
  background: #4338ca;
  cursor: pointer;
}
.botao:hover { opacity: 0.9; }
.admin { background: #7c3aed; }
.logout { background: #dc2626; }
```

**10. Layout com header condicional**

```tsx
// src/components/Layout.tsx
import { Outlet, useNavigate } from 'react-router-dom';
import { useAppAuth } from '../core/auth/useAppAuth';
import styles from './Layout.module.css';

export function Layout() {
  const { isAuthenticated, usuario, logout, hasRole } = useAppAuth();
  const navigate = useNavigate();

  return (
    <>
      <header className={styles.header}>
        <nav className={styles.nav}>
          <a href="/" className={styles.logo}>MeuApp</a>

          <div className={styles.links}>
            <a href="/sobre">Sobre</a>

            {isAuthenticated ? (
              <>
                <a href="/home">Home</a>
                {hasRole('ADMIN') && <a href="/admin">Admin</a>}
                <span className={styles.usuario}>{usuario?.nome}</span>
                <button onClick={() => logout()} className={styles.btnLogout}>
                  Sair
                </button>
              </>
            ) : (
              <button
                onClick={() => navigate('/login')}
                className={styles.btnLogin}
              >
                Entrar
              </button>
            )}
          </div>
        </nav>
      </header>

      <main>
        <Outlet />
      </main>
    </>
  );
}
```

```css
/* src/components/Layout.module.css */
.header { background: #1e1b4b; color: #fff; padding: 0 24px; }
.nav { display: flex; justify-content: space-between; align-items: center; height: 56px; max-width: 1200px; margin: 0 auto; }
.logo { color: #fff; font-weight: 700; font-size: 1.25rem; text-decoration: none; }
.links { display: flex; align-items: center; gap: 20px; }
.links a { color: #c7d2fe; text-decoration: none; font-size: 0.875rem; }
.links a:hover { color: #fff; }
.usuario { color: #a5b4fc; font-size: 0.875rem; }
.btnLogin {
  background: #4338ca; color: #fff; border: none; padding: 8px 16px;
  border-radius: 8px; cursor: pointer; font-size: 0.875rem;
}
.btnLogout {
  background: transparent; color: #fca5a5; border: 1px solid #fca5a5;
  padding: 6px 12px; border-radius: 8px; cursor: pointer; font-size: 0.8rem;
}
.btnLogout:hover { background: rgba(252, 165, 165, 0.1); }
```

**11. Fetch com token automático**

```typescript
// src/core/auth/authFetch.ts
import { useAppAuth } from './useAppAuth';
import { useCallback } from 'react';

export function useAuthFetch() {
  const { accessToken, logout } = useAppAuth();

  const authFetch = useCallback(
    async (url: string, options: RequestInit = {}): Promise<Response> => {
      const headers = new Headers(options.headers);
      if (accessToken) {
        headers.set('Authorization', `Bearer ${accessToken}`);
      }

      const response = await fetch(url, { ...options, headers });

      if (response.status === 401) {
        logout();
        throw new Error('Sessão expirada');
      }

      return response;
    },
    [accessToken, logout],
  );

  return authFetch;
}
```

**12. main.tsx**

```tsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { AuthProvider } from 'react-oidc-context';
import { oidcConfig } from './core/auth/oidcConfig';
import { AppRouter } from './routes/AppRouter';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <AuthProvider {...oidcConfig}>
      <AppRouter />
    </AuthProvider>
  </StrictMode>,
);
```

---

### Equivalência — Autenticação OAuth2/OIDC

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Biblioteca OIDC | `flutter_appauth` | `expo-auth-session` | `oidc-client-ts` + `react-oidc-context` |
| Armazenamento seguro | `flutter_secure_storage` | `expo-secure-store` | `localStorage` (via `WebStorageStateStore`) |
| Decodificação JWT | `jwt_decoder` | `jwt-decode` | `oidc-client-ts` (automático) |
| Refresh token | `FlutterAppAuth.token()` manual | `AuthSession.refreshAsync()` manual | `automaticSilentRenew: true` (automático) |
| Route guard | `GoRouter.redirect` global | Renderização condicional no Navigator | Componente `<ProtectedRoute>` |
| Guard por role | Verificação no `redirect` | Condição no `<Stack.Screen>` | Prop `roles` no `<ProtectedRoute>` |
| Estado de auth | Riverpod `NotifierProvider` | React Context + `useState` | `react-oidc-context` + hook custom |
| Dados do usuário | `ref.watch(authProvider)` | `useAuth().usuario` | `useAppAuth().usuario` |
| Token em requisições | `Dio` interceptor com refresh | `getAccessToken()` manual | Hook `useAuthFetch()` |
| Redirect após login | Query param `?redirect=` | Troca de stack do Navigator | `sessionStorage` + `navigate()` |
| Logout | `endSession` + limpar storage | `WebBrowser` + limpar SecureStore | `signoutRedirect()` |

| Rota | Acesso | Flutter | React Native | React Web |
|---|---|---|---|---|
| `/` `/sobre` | Público | Sem redirect | Sempre no Navigator | Sem `<ProtectedRoute>` |
| `/home` `/perfil` | Autenticado | Redirect para `/login` | Stack condicional | `<ProtectedRoute>` |
| `/admin` `/admin/*` | Somente ADMIN | `usuario.isAdmin` no redirect | `hasRole('ADMIN')` no Screen | `<ProtectedRoute roles={['ADMIN']}>` |

---

## 4. Layout Responsivo — Home com Sidebar, Cards e Scroll Infinito

Layout completo de aplicação com:

- **Cabeçalho:** logotipo à esquerda, avatar do usuário logado ou botão "Entrar" à direita.
- **Área principal (Home):** grid responsivo de cards (2 colunas mobile retrato, 3 colunas tablet/mobile paisagem, 4 colunas desktop). Cada card tem thumbnail, título, descrição e badges. Inicia em skeleton e suporta scroll infinito.
- **Rodapé:** barra de navegação inferior com 4 botões no mobile; rodapé ilustrativo em tablet/desktop.
- **Sidebar:** overlay deslizante no mobile; fixa com ícone + título reduzido no tablet; fixa com toggle expandido/recolhido no desktop.
- **Rotas Flutter:** versões com GoRouter e com Modular.
- **React Web:** Bootstrap CSS + Font Awesome via classes; `react-bootstrap` apenas para componentes dinâmicos (Offcanvas, Collapse).

### Estrutura de breakpoints adotada

| Classificação | Flutter (`MediaQuery`) | React Native (`Dimensions`) | Bootstrap |
|---|---|---|---|
| Mobile retrato | `width < 600` | `width < 600` | `< 768px` (xs/sm) |
| Tablet / Mobile paisagem | `600 ≤ width < 1024` | `600 ≤ width < 1024` | `768px–1199px` (md/lg) |
| Desktop | `width ≥ 1024` | `width ≥ 1024` | `≥ 1200px` (xl+) |

---

### Flutter — GoRouter

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  go_router: ^14.0.0
  flutter_riverpod: ^2.5.0
  cached_network_image: ^3.4.0
  shimmer: ^3.0.0
```

```bash
flutter pub get
```

**2. Modelo de dados**

```dart
// lib/features/home/domain/content_item.dart
class ContentItem {
  final String id;
  final String titulo;
  final String descricao;
  final String thumbnailUrl;
  final List<String> badges;

  const ContentItem({
    required this.id,
    required this.titulo,
    required this.descricao,
    required this.thumbnailUrl,
    this.badges = const [],
  });
}
```

**3. Serviço de conteúdo simulado (com paginação)**

```dart
// lib/features/home/data/content_service.dart
import '../domain/content_item.dart';

class ContentService {
  Future<List<ContentItem>> buscarConteudos({
    required int pagina,
    int porPagina = 10,
  }) async {
    // Simula latência de rede
    await Future.delayed(const Duration(milliseconds: 800));

    return List.generate(
      porPagina,
      (i) {
        final index = (pagina - 1) * porPagina + i + 1;
        return ContentItem(
          id: 'item-$index',
          titulo: 'Conteúdo $index',
          descricao: 'Descrição breve do item $index para demonstrar o layout do card.',
          thumbnailUrl: 'https://picsum.photos/seed/item$index/400/300',
          badges: ['Tag A', 'Tag B'],
        );
      },
    );
  }
}
```

**4. Provider do estado da Home**

```dart
// lib/features/home/presentation/home_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../data/content_service.dart';
import '../domain/content_item.dart';

class HomeState {
  final List<ContentItem> itens;
  final bool carregando;
  final bool temMais;
  final int paginaAtual;

  const HomeState({
    this.itens = const [],
    this.carregando = false,
    this.temMais = true,
    this.paginaAtual = 0,
  });

  HomeState copyWith({
    List<ContentItem>? itens,
    bool? carregando,
    bool? temMais,
    int? paginaAtual,
  }) {
    return HomeState(
      itens: itens ?? this.itens,
      carregando: carregando ?? this.carregando,
      temMais: temMais ?? this.temMais,
      paginaAtual: paginaAtual ?? this.paginaAtual,
    );
  }
}

class HomeNotifier extends Notifier<HomeState> {
  late final _service = ContentService();

  @override
  HomeState build() {
    carregarMais();
    return const HomeState(carregando: true);
  }

  Future<void> carregarMais() async {
    if (state.carregando || !state.temMais) return;

    state = state.copyWith(carregando: true);
    final proximaPagina = state.paginaAtual + 1;

    final novosItens = await _service.buscarConteudos(pagina: proximaPagina);

    state = state.copyWith(
      itens: [...state.itens, ...novosItens],
      carregando: false,
      paginaAtual: proximaPagina,
      temMais: novosItens.length >= 10,
    );
  }
}

final homeProvider = NotifierProvider<HomeNotifier, HomeState>(
  HomeNotifier.new,
);
```

**5. Widget de skeleton do card**

```dart
// lib/features/home/presentation/widgets/card_skeleton.dart
import 'package:flutter/material.dart';
import 'package:shimmer/shimmer.dart';

class CardSkeleton extends StatelessWidget {
  const CardSkeleton({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: Shimmer.fromColors(
        baseColor: Colors.grey.shade300,
        highlightColor: Colors.grey.shade100,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Container(height: 140, color: Colors.white),
            Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Container(
                    height: 16,
                    width: double.infinity,
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(4),
                    ),
                  ),
                  const SizedBox(height: 8),
                  Container(
                    height: 12,
                    width: 180,
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(4),
                    ),
                  ),
                  const SizedBox(height: 12),
                  Row(
                    children: List.generate(
                      2,
                      (_) => Container(
                        height: 20,
                        width: 48,
                        margin: const EdgeInsets.only(right: 8),
                        decoration: BoxDecoration(
                          color: Colors.white,
                          borderRadius: BorderRadius.circular(10),
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**6. Widget do card de conteúdo**

```dart
// lib/features/home/presentation/widgets/content_card.dart
import 'package:flutter/material.dart';
import 'package:cached_network_image/cached_network_image.dart';
import '../../domain/content_item.dart';

class ContentCard extends StatelessWidget {
  final ContentItem item;

  const ContentCard({super.key, required this.item});

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          CachedNetworkImage(
            imageUrl: item.thumbnailUrl,
            height: 140,
            width: double.infinity,
            fit: BoxFit.cover,
            placeholder: (_, __) => Container(
              height: 140,
              color: Colors.grey.shade200,
              child: const Center(child: CircularProgressIndicator(strokeWidth: 2)),
            ),
            errorWidget: (_, __, ___) => Container(
              height: 140,
              color: Colors.grey.shade300,
              child: const Icon(Icons.broken_image, size: 48, color: Colors.grey),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(12),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  item.titulo,
                  style: Theme.of(context).textTheme.titleSmall,
                  maxLines: 1,
                  overflow: TextOverflow.ellipsis,
                ),
                const SizedBox(height: 4),
                Text(
                  item.descricao,
                  style: Theme.of(context).textTheme.bodySmall,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
                if (item.badges.isNotEmpty) ...[
                  const SizedBox(height: 8),
                  Wrap(
                    spacing: 6,
                    runSpacing: 4,
                    children: item.badges
                        .map((b) => Chip(
                              label: Text(b, style: const TextStyle(fontSize: 10)),
                              materialTapTargetSize: MaterialTapTargetSize.shrinkWrap,
                              visualDensity: VisualDensity.compact,
                              padding: EdgeInsets.zero,
                            ))
                        .toList(),
                  ),
                ],
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

**7. Helper de responsividade**

```dart
// lib/core/responsive.dart
import 'package:flutter/material.dart';

enum ScreenType { mobile, tablet, desktop }

ScreenType getScreenType(BuildContext context) {
  final width = MediaQuery.sizeOf(context).width;
  if (width < 600) return ScreenType.mobile;
  if (width < 1024) return ScreenType.tablet;
  return ScreenType.desktop;
}

int getGridColumns(BuildContext context) {
  return switch (getScreenType(context)) {
    ScreenType.mobile => 2,
    ScreenType.tablet => 3,
    ScreenType.desktop => 4,
  };
}
```

**8. Sidebar**

```dart
// lib/features/shell/presentation/widgets/app_sidebar.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../../../../core/responsive.dart';

class SidebarItem {
  final IconData icone;
  final String titulo;
  final String tituloReduzido;
  final String rota;

  const SidebarItem({
    required this.icone,
    required this.titulo,
    required this.tituloReduzido,
    required this.rota,
  });
}

const _itens = [
  SidebarItem(icone: Icons.home, titulo: 'Início', tituloReduzido: 'Início', rota: '/'),
  SidebarItem(icone: Icons.explore, titulo: 'Explorar', tituloReduzido: 'Explorar', rota: '/explorar'),
  SidebarItem(icone: Icons.bookmark, titulo: 'Salvos', tituloReduzido: 'Salvos', rota: '/salvos'),
  SidebarItem(icone: Icons.settings, titulo: 'Configurações', tituloReduzido: 'Config', rota: '/config'),
];

class AppSidebar extends StatelessWidget {
  final bool expandida;
  final VoidCallback? onToggle;

  const AppSidebar({super.key, this.expandida = true, this.onToggle});

  @override
  Widget build(BuildContext context) {
    final screenType = getScreenType(context);
    final mostrarExpandida = screenType == ScreenType.tablet ? false : expandida;
    final largura = mostrarExpandida ? 240.0 : 72.0;

    return AnimatedContainer(
      duration: const Duration(milliseconds: 200),
      width: largura,
      color: Theme.of(context).colorScheme.surfaceContainerLow,
      child: Column(
        children: [
          if (screenType == ScreenType.desktop)
            Padding(
              padding: const EdgeInsets.symmetric(vertical: 8),
              child: IconButton(
                icon: Icon(mostrarExpandida ? Icons.menu_open : Icons.menu),
                onPressed: onToggle,
                tooltip: mostrarExpandida ? 'Recolher menu' : 'Expandir menu',
              ),
            ),
          const Divider(height: 1),
          Expanded(
            child: ListView(
              children: _itens.map((item) {
                final selecionado =
                    GoRouterState.of(context).matchedLocation == item.rota;

                if (mostrarExpandida) {
                  return ListTile(
                    leading: Icon(item.icone),
                    title: Text(item.titulo),
                    selected: selecionado,
                    onTap: () => context.go(item.rota),
                  );
                }

                return Tooltip(
                  message: item.titulo,
                  child: InkWell(
                    onTap: () => context.go(item.rota),
                    child: Container(
                      padding: const EdgeInsets.symmetric(vertical: 12),
                      decoration: selecionado
                          ? BoxDecoration(
                              color: Theme.of(context)
                                  .colorScheme
                                  .primaryContainer
                                  .withAlpha(80),
                              border: Border(
                                left: BorderSide(
                                  color: Theme.of(context).colorScheme.primary,
                                  width: 3,
                                ),
                              ),
                            )
                          : null,
                      child: Column(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          Icon(item.icone,
                              color: selecionado
                                  ? Theme.of(context).colorScheme.primary
                                  : null),
                          const SizedBox(height: 4),
                          Text(
                            item.tituloReduzido,
                            style: TextStyle(
                              fontSize: 10,
                              color: selecionado
                                  ? Theme.of(context).colorScheme.primary
                                  : null,
                            ),
                            textAlign: TextAlign.center,
                          ),
                        ],
                      ),
                    ),
                  ),
                );
              }).toList(),
            ),
          ),
        ],
      ),
    );
  }
}
```

**9. Cabeçalho**

```dart
// lib/features/shell/presentation/widgets/app_header.dart
import 'package:flutter/material.dart';

class AppHeader extends StatelessWidget implements PreferredSizeWidget {
  final bool logado;
  final String? avatarUrl;
  final VoidCallback? onMenuTap;
  final VoidCallback? onEntrarTap;
  final VoidCallback? onAvatarTap;

  const AppHeader({
    super.key,
    required this.logado,
    this.avatarUrl,
    this.onMenuTap,
    this.onEntrarTap,
    this.onAvatarTap,
  });

  @override
  Size get preferredSize => const Size.fromHeight(kToolbarHeight);

  @override
  Widget build(BuildContext context) {
    return AppBar(
      leading: onMenuTap != null
          ? IconButton(icon: const Icon(Icons.menu), onPressed: onMenuTap)
          : null,
      title: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(Icons.widgets, color: Theme.of(context).colorScheme.primary),
          const SizedBox(width: 8),
          const Text('MeuApp', style: TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
      actions: [
        if (logado)
          Padding(
            padding: const EdgeInsets.only(right: 12),
            child: GestureDetector(
              onTap: onAvatarTap,
              child: CircleAvatar(
                radius: 16,
                backgroundImage:
                    avatarUrl != null ? NetworkImage(avatarUrl!) : null,
                child: avatarUrl == null ? const Icon(Icons.person, size: 18) : null,
              ),
            ),
          )
        else
          Padding(
            padding: const EdgeInsets.only(right: 12),
            child: FilledButton.tonal(
              onPressed: onEntrarTap,
              child: const Text('Entrar'),
            ),
          ),
      ],
    );
  }
}
```

**10. Rodapé — barra inferior mobile e rodapé desktop/tablet**

```dart
// lib/features/shell/presentation/widgets/app_bottom_bar.dart
import 'package:flutter/material.dart';

class AppBottomBar extends StatelessWidget {
  final int indexSelecionado;
  final ValueChanged<int> onItemTap;

  const AppBottomBar({
    super.key,
    required this.indexSelecionado,
    required this.onItemTap,
  });

  @override
  Widget build(BuildContext context) {
    return NavigationBar(
      selectedIndex: indexSelecionado,
      onDestinationSelected: onItemTap,
      destinations: const [
        NavigationDestination(icon: Icon(Icons.home), label: 'Início'),
        NavigationDestination(icon: Icon(Icons.explore), label: 'Explorar'),
        NavigationDestination(icon: Icon(Icons.bookmark), label: 'Salvos'),
        NavigationDestination(icon: Icon(Icons.person), label: 'Perfil'),
      ],
    );
  }
}

class AppFooter extends StatelessWidget {
  const AppFooter({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: double.infinity,
      padding: const EdgeInsets.symmetric(vertical: 16),
      color: Theme.of(context).colorScheme.surfaceContainerLow,
      child: Text(
        '© 2025 MeuApp. Todos os direitos reservados.',
        textAlign: TextAlign.center,
        style: Theme.of(context).textTheme.bodySmall,
      ),
    );
  }
}
```

**11. Shell — layout principal responsivo**

```dart
// lib/features/shell/presentation/app_shell.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../core/responsive.dart';
import 'widgets/app_header.dart';
import 'widgets/app_sidebar.dart';
import 'widgets/app_bottom_bar.dart';

final sidebarExpandidaProvider = StateProvider<bool>((_) => true);

class AppShell extends ConsumerWidget {
  final Widget child;

  const AppShell({super.key, required this.child});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final screenType = getScreenType(context);
    final sidebarExpandida = ref.watch(sidebarExpandidaProvider);

    return Scaffold(
      appBar: AppHeader(
        logado: true,
        onMenuTap: screenType == ScreenType.mobile
            ? () => Scaffold.of(context).openDrawer()
            : null,
        onAvatarTap: () {},
      ),
      drawer: screenType == ScreenType.mobile
          ? Drawer(
              child: SafeArea(
                child: AppSidebar(expandida: true),
              ),
            )
          : null,
      body: Row(
        children: [
          if (screenType != ScreenType.mobile)
            AppSidebar(
              expandida: sidebarExpandida,
              onToggle: () => ref
                  .read(sidebarExpandidaProvider.notifier)
                  .state = !sidebarExpandida,
            ),
          Expanded(
            child: Column(
              children: [
                Expanded(child: child),
                if (screenType != ScreenType.mobile) const AppFooter(),
              ],
            ),
          ),
        ],
      ),
      bottomNavigationBar: screenType == ScreenType.mobile
          ? AppBottomBar(
              indexSelecionado: 0,
              onItemTap: (i) {
                final rotas = ['/', '/explorar', '/salvos', '/perfil'];
                if (i < rotas.length) {
                  // Navegação via GoRouter seria feita aqui
                }
              },
            )
          : null,
    );
  }
}
```

**12. Tela Home com grid e scroll infinito**

```dart
// lib/features/home/presentation/home_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../core/responsive.dart';
import 'home_provider.dart';
import 'widgets/content_card.dart';
import 'widgets/card_skeleton.dart';

class HomeScreen extends ConsumerStatefulWidget {
  const HomeScreen({super.key});

  @override
  ConsumerState<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends ConsumerState<HomeScreen> {
  final _scrollCtrl = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollCtrl.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollCtrl.position.pixels >=
        _scrollCtrl.position.maxScrollExtent - 200) {
      ref.read(homeProvider.notifier).carregarMais();
    }
  }

  @override
  void dispose() {
    _scrollCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final state = ref.watch(homeProvider);
    final colunas = getGridColumns(context);

    // Exibição inicial: grid de skeletons
    if (state.itens.isEmpty && state.carregando) {
      return GridView.builder(
        padding: const EdgeInsets.all(16),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: colunas,
          mainAxisSpacing: 16,
          crossAxisSpacing: 16,
          childAspectRatio: 0.75,
        ),
        itemCount: colunas * 3,
        itemBuilder: (_, __) => const CardSkeleton(),
      );
    }

    final totalItens = state.itens.length + (state.carregando ? colunas : 0);

    return GridView.builder(
      controller: _scrollCtrl,
      padding: const EdgeInsets.all(16),
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: colunas,
        mainAxisSpacing: 16,
        crossAxisSpacing: 16,
        childAspectRatio: 0.75,
      ),
      itemCount: totalItens,
      itemBuilder: (_, i) {
        if (i >= state.itens.length) {
          return const CardSkeleton();
        }
        return ContentCard(item: state.itens[i]);
      },
    );
  }
}
```

**13. Telas placeholder**

```dart
// lib/features/explorar/explorar_screen.dart
import 'package:flutter/material.dart';

class ExplorarScreen extends StatelessWidget {
  const ExplorarScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Center(child: Text('Explorar — em construção'));
  }
}

// lib/features/salvos/salvos_screen.dart
import 'package:flutter/material.dart';

class SalvosScreen extends StatelessWidget {
  const SalvosScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Center(child: Text('Salvos — em construção'));
  }
}

// lib/features/config/config_screen.dart
import 'package:flutter/material.dart';

class ConfigScreen extends StatelessWidget {
  const ConfigScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Center(child: Text('Configurações — em construção'));
  }
}
```

**14. Rotas com GoRouter**

```dart
// lib/routes/app_router.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import '../features/shell/presentation/app_shell.dart';
import '../features/home/presentation/home_screen.dart';
import '../features/explorar/explorar_screen.dart';
import '../features/salvos/salvos_screen.dart';
import '../features/config/config_screen.dart';

final routerProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    initialLocation: '/',
    routes: [
      ShellRoute(
        builder: (_, __, child) => AppShell(child: child),
        routes: [
          GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
          GoRoute(path: '/explorar', builder: (_, __) => const ExplorarScreen()),
          GoRoute(path: '/salvos', builder: (_, __) => const SalvosScreen()),
          GoRoute(path: '/config', builder: (_, __) => const ConfigScreen()),
        ],
      ),
    ],
    errorBuilder: (_, state) => Scaffold(
      body: Center(child: Text('Página não encontrada: ${state.matchedLocation}')),
    ),
  );
});
```

**15. main.dart**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'routes/app_router.dart';

void main() {
  runApp(const ProviderScope(child: App()));
}

class App extends ConsumerWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(routerProvider);

    return MaterialApp.router(
      title: 'MeuApp',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      routerConfig: router,
    );
  }
}
```

**Estrutura de arquivos:**

```
lib/
├── main.dart
├── core/
│   └── responsive.dart
├── routes/
│   └── app_router.dart
├── features/
│   ├── shell/
│   │   └── presentation/
│   │       ├── app_shell.dart
│   │       └── widgets/
│   │           ├── app_header.dart
│   │           ├── app_sidebar.dart
│   │           └── app_bottom_bar.dart
│   ├── home/
│   │   ├── domain/
│   │   │   └── content_item.dart
│   │   ├── data/
│   │   │   └── content_service.dart
│   │   └── presentation/
│   │       ├── home_screen.dart
│   │       ├── home_provider.dart
│   │       └── widgets/
│   │           ├── content_card.dart
│   │           └── card_skeleton.dart
│   ├── explorar/
│   │   └── explorar_screen.dart
│   ├── salvos/
│   │   └── salvos_screen.dart
│   └── config/
│       └── config_screen.dart
```

---

### Flutter (alternativo) — Modular

Mesmos componentes visuais do exemplo anterior (model, skeleton, card, header, sidebar, bottom bar, home screen, telas placeholder). As diferenças estão na configuração de rotas, injeção de dependências e no shell.

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_modular: ^6.3.0
  cached_network_image: ^3.4.0
  shimmer: ^3.0.0
```

```bash
flutter pub get
```

**2. Store da Home (ChangeNotifier em vez de Riverpod)**

```dart
// lib/features/home/presentation/home_store.dart
import 'package:flutter/foundation.dart';
import '../data/content_service.dart';
import '../domain/content_item.dart';

class HomeStore extends ChangeNotifier {
  final ContentService _service;

  HomeStore(this._service) {
    carregarMais();
  }

  List<ContentItem> itens = [];
  bool carregando = false;
  bool temMais = true;
  int _paginaAtual = 0;

  Future<void> carregarMais() async {
    if (carregando || !temMais) return;

    carregando = true;
    notifyListeners();

    final proximaPagina = _paginaAtual + 1;
    final novosItens = await _service.buscarConteudos(pagina: proximaPagina);

    itens = [...itens, ...novosItens];
    _paginaAtual = proximaPagina;
    temMais = novosItens.length >= 10;
    carregando = false;
    notifyListeners();
  }
}
```

**3. Store do sidebar (toggle expandido/recolhido)**

```dart
// lib/features/shell/presentation/sidebar_store.dart
import 'package:flutter/foundation.dart';

class SidebarStore extends ChangeNotifier {
  bool _expandida = true;

  bool get expandida => _expandida;

  void toggle() {
    _expandida = !_expandida;
    notifyListeners();
  }
}
```

**4. Shell adaptado para Modular**

```dart
// lib/features/shell/presentation/app_shell_modular.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';
import '../../../core/responsive.dart';
import 'sidebar_store.dart';
import 'widgets/app_header.dart';
import 'widgets/app_sidebar.dart';
import 'widgets/app_bottom_bar.dart';

class AppShellModular extends StatelessWidget {
  const AppShellModular({super.key});

  @override
  Widget build(BuildContext context) {
    final sidebarStore = Modular.get<SidebarStore>();
    final screenType = getScreenType(context);

    return AnimatedBuilder(
      animation: sidebarStore,
      builder: (context, _) {
        return Scaffold(
          appBar: AppHeader(
            logado: true,
            onMenuTap: screenType == ScreenType.mobile
                ? () => Scaffold.of(context).openDrawer()
                : null,
            onAvatarTap: () {},
          ),
          drawer: screenType == ScreenType.mobile
              ? Drawer(
                  child: SafeArea(child: AppSidebar(expandida: true)),
                )
              : null,
          body: Row(
            children: [
              if (screenType != ScreenType.mobile)
                AppSidebar(
                  expandida: sidebarStore.expandida,
                  onToggle: sidebarStore.toggle,
                ),
              Expanded(
                child: Column(
                  children: [
                    const Expanded(child: RouterOutlet()),
                    if (screenType != ScreenType.mobile) const AppFooter(),
                  ],
                ),
              ),
            ],
          ),
          bottomNavigationBar: screenType == ScreenType.mobile
              ? AppBottomBar(
                  indexSelecionado: 0,
                  onItemTap: (i) {
                    final rotas = ['/', '/explorar/', '/salvos/', '/perfil/'];
                    if (i < rotas.length) Modular.to.navigate(rotas[i]);
                  },
                )
              : null,
        );
      },
    );
  }
}
```

**5. Home Screen adaptada para Modular**

```dart
// lib/features/home/presentation/home_screen_modular.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';
import '../../../core/responsive.dart';
import 'home_store.dart';
import 'widgets/content_card.dart';
import 'widgets/card_skeleton.dart';

class HomeScreenModular extends StatefulWidget {
  const HomeScreenModular({super.key});

  @override
  State<HomeScreenModular> createState() => _HomeScreenModularState();
}

class _HomeScreenModularState extends State<HomeScreenModular> {
  final _scrollCtrl = ScrollController();
  late final HomeStore _store;

  @override
  void initState() {
    super.initState();
    _store = Modular.get<HomeStore>();
    _scrollCtrl.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollCtrl.position.pixels >=
        _scrollCtrl.position.maxScrollExtent - 200) {
      _store.carregarMais();
    }
  }

  @override
  void dispose() {
    _scrollCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final colunas = getGridColumns(context);

    return AnimatedBuilder(
      animation: _store,
      builder: (context, _) {
        if (_store.itens.isEmpty && _store.carregando) {
          return GridView.builder(
            padding: const EdgeInsets.all(16),
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: colunas,
              mainAxisSpacing: 16,
              crossAxisSpacing: 16,
              childAspectRatio: 0.75,
            ),
            itemCount: colunas * 3,
            itemBuilder: (_, __) => const CardSkeleton(),
          );
        }

        final totalItens =
            _store.itens.length + (_store.carregando ? colunas : 0);

        return GridView.builder(
          controller: _scrollCtrl,
          padding: const EdgeInsets.all(16),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: colunas,
            mainAxisSpacing: 16,
            crossAxisSpacing: 16,
            childAspectRatio: 0.75,
          ),
          itemCount: totalItens,
          itemBuilder: (_, i) {
            if (i >= _store.itens.length) return const CardSkeleton();
            return ContentCard(item: _store.itens[i]);
          },
        );
      },
    );
  }
}
```

**6. Módulo raiz**

```dart
// lib/app_module.dart
import 'package:flutter_modular/flutter_modular.dart';
import 'features/home/data/content_service.dart';
import 'features/home/presentation/home_store.dart';
import 'features/shell/presentation/sidebar_store.dart';
import 'features/shell/presentation/app_shell_modular.dart';
import 'features/home/presentation/home_screen_modular.dart';
import 'features/explorar/explorar_screen.dart';
import 'features/salvos/salvos_screen.dart';
import 'features/config/config_screen.dart';

class AppModule extends Module {
  @override
  void binds(Injector i) {
    i.addSingleton<ContentService>(ContentService.new);
    i.addSingleton<HomeStore>(() => HomeStore(i.get<ContentService>()));
    i.addSingleton<SidebarStore>(SidebarStore.new);
  }

  @override
  void routes(RouteManager r) {
    r.child(
      '/',
      child: (_) => const AppShellModular(),
      children: [
        ChildRoute('/', child: (_) => const HomeScreenModular()),
        ChildRoute('/explorar/', child: (_) => const ExplorarScreen()),
        ChildRoute('/salvos/', child: (_) => const SalvosScreen()),
        ChildRoute('/config/', child: (_) => const ConfigScreen()),
      ],
    );
  }
}
```

**7. main.dart**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_modular/flutter_modular.dart';
import 'app_module.dart';

void main() {
  runApp(
    ModularApp(
      module: AppModule(),
      child: const App(),
    ),
  );
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'MeuApp',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      routerConfig: Modular.routerConfig,
    );
  }
}
```

---

### React Native — Expo Router + Drawer

**1. Dependências**

```bash
npx create-expo-app@latest meu-app --template tabs
cd meu-app

npx expo install @react-navigation/drawer react-native-gesture-handler react-native-reanimated
npx expo install expo-image
```

**2. Tipos**

```typescript
// src/types/content.ts
export interface ContentItem {
  id: string;
  titulo: string;
  descricao: string;
  thumbnailUrl: string;
  badges: string[];
}
```

**3. Hook de responsividade**

```typescript
// src/hooks/useResponsive.ts
import { useWindowDimensions } from 'react-native';

export type ScreenType = 'mobile' | 'tablet' | 'desktop';

export function useResponsive() {
  const { width } = useWindowDimensions();

  const screenType: ScreenType =
    width < 600 ? 'mobile' : width < 1024 ? 'tablet' : 'desktop';

  const gridColumns = screenType === 'mobile' ? 2 : screenType === 'tablet' ? 3 : 4;

  return { screenType, gridColumns, width };
}
```

**4. Serviço de conteúdo simulado**

```typescript
// src/services/contentService.ts
import { ContentItem } from '../types/content';

export async function buscarConteudos(
  pagina: number,
  porPagina = 10,
): Promise<ContentItem[]> {
  await new Promise((r) => setTimeout(r, 800));

  return Array.from({ length: porPagina }, (_, i) => {
    const index = (pagina - 1) * porPagina + i + 1;
    return {
      id: `item-${index}`,
      titulo: `Conteúdo ${index}`,
      descricao: `Descrição breve do item ${index} para demonstrar o layout do card.`,
      thumbnailUrl: `https://picsum.photos/seed/item${index}/400/300`,
      badges: ['Tag A', 'Tag B'],
    };
  });
}
```

**5. Componente skeleton**

```tsx
// src/components/CardSkeleton.tsx
import React, { useEffect, useRef } from 'react';
import { View, Animated, StyleSheet } from 'react-native';

export function CardSkeleton() {
  const opacity = useRef(new Animated.Value(0.3)).current;

  useEffect(() => {
    const animation = Animated.loop(
      Animated.sequence([
        Animated.timing(opacity, {
          toValue: 1,
          duration: 800,
          useNativeDriver: true,
        }),
        Animated.timing(opacity, {
          toValue: 0.3,
          duration: 800,
          useNativeDriver: true,
        }),
      ]),
    );
    animation.start();
    return () => animation.stop();
  }, [opacity]);

  return (
    <Animated.View style={[styles.card, { opacity }]}>
      <View style={styles.thumbnail} />
      <View style={styles.body}>
        <View style={styles.linhaGrande} />
        <View style={styles.linhaPequena} />
        <View style={styles.badgesRow}>
          <View style={styles.badge} />
          <View style={styles.badge} />
        </View>
      </View>
    </Animated.View>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    overflow: 'hidden',
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  thumbnail: { height: 120, backgroundColor: '#e0e0e0' },
  body: { padding: 12 },
  linhaGrande: {
    height: 14,
    backgroundColor: '#e0e0e0',
    borderRadius: 4,
    marginBottom: 8,
  },
  linhaPequena: {
    height: 10,
    width: '60%',
    backgroundColor: '#e0e0e0',
    borderRadius: 4,
    marginBottom: 12,
  },
  badgesRow: { flexDirection: 'row', gap: 6 },
  badge: {
    height: 18,
    width: 48,
    backgroundColor: '#e0e0e0',
    borderRadius: 9,
  },
});
```

**6. Componente do card**

```tsx
// src/components/ContentCard.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { Image } from 'expo-image';
import { ContentItem } from '../types/content';

interface Props {
  item: ContentItem;
}

export function ContentCard({ item }: Props) {
  return (
    <View style={styles.card}>
      <Image
        source={{ uri: item.thumbnailUrl }}
        style={styles.thumbnail}
        contentFit="cover"
        transition={300}
      />
      <View style={styles.body}>
        <Text style={styles.titulo} numberOfLines={1}>
          {item.titulo}
        </Text>
        <Text style={styles.descricao} numberOfLines={2}>
          {item.descricao}
        </Text>
        {item.badges.length > 0 && (
          <View style={styles.badgesRow}>
            {item.badges.map((b) => (
              <View key={b} style={styles.badge}>
                <Text style={styles.badgeTexto}>{b}</Text>
              </View>
            ))}
          </View>
        )}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    overflow: 'hidden',
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  thumbnail: { height: 120 },
  body: { padding: 12 },
  titulo: { fontSize: 14, fontWeight: '600', marginBottom: 4 },
  descricao: { fontSize: 12, color: '#666', marginBottom: 8 },
  badgesRow: { flexDirection: 'row', gap: 6, flexWrap: 'wrap' },
  badge: {
    backgroundColor: '#e8eaf6',
    paddingHorizontal: 8,
    paddingVertical: 2,
    borderRadius: 10,
  },
  badgeTexto: { fontSize: 10, color: '#3949ab' },
});
```

**7. Hook de scroll infinito**

```typescript
// src/hooks/useInfiniteContent.ts
import { useState, useCallback } from 'react';
import { ContentItem } from '../types/content';
import { buscarConteudos } from '../services/contentService';

export function useInfiniteContent() {
  const [itens, setItens] = useState<ContentItem[]>([]);
  const [carregando, setCarregando] = useState(false);
  const [temMais, setTemMais] = useState(true);
  const [pagina, setPagina] = useState(0);
  const [carregamentoInicial, setCarregamentoInicial] = useState(true);

  const carregarMais = useCallback(async () => {
    if (carregando || !temMais) return;
    setCarregando(true);

    const proximaPagina = pagina + 1;
    const novos = await buscarConteudos(proximaPagina);

    setItens((prev) => [...prev, ...novos]);
    setPagina(proximaPagina);
    setTemMais(novos.length >= 10);
    setCarregando(false);
    setCarregamentoInicial(false);
  }, [carregando, temMais, pagina]);

  return { itens, carregando, carregamentoInicial, temMais, carregarMais };
}
```

**8. Tela Home**

```tsx
// app/(tabs)/index.tsx
import React, { useEffect } from 'react';
import { View, FlatList, StyleSheet, ActivityIndicator } from 'react-native';
import { ContentCard } from '../../src/components/ContentCard';
import { CardSkeleton } from '../../src/components/CardSkeleton';
import { useInfiniteContent } from '../../src/hooks/useInfiniteContent';
import { useResponsive } from '../../src/hooks/useResponsive';

export default function HomeScreen() {
  const { itens, carregando, carregamentoInicial, carregarMais } =
    useInfiniteContent();
  const { gridColumns, width } = useResponsive();

  useEffect(() => {
    carregarMais();
  }, []);

  const gap = 12;
  const padding = 16;
  const itemWidth =
    (width - padding * 2 - gap * (gridColumns - 1)) / gridColumns;

  if (carregamentoInicial) {
    return (
      <View style={styles.skeletonGrid}>
        {Array.from({ length: gridColumns * 3 }, (_, i) => (
          <View key={i} style={{ width: itemWidth, marginBottom: gap }}>
            <CardSkeleton />
          </View>
        ))}
      </View>
    );
  }

  return (
    <FlatList
      data={itens}
      keyExtractor={(item) => item.id}
      numColumns={gridColumns}
      key={`grid-${gridColumns}`}
      contentContainerStyle={styles.lista}
      columnWrapperStyle={{ gap }}
      renderItem={({ item }) => (
        <View style={{ width: itemWidth }}>
          <ContentCard item={item} />
        </View>
      )}
      onEndReached={carregarMais}
      onEndReachedThreshold={0.3}
      ListFooterComponent={
        carregando ? (
          <ActivityIndicator size="small" style={styles.loader} />
        ) : null
      }
      ItemSeparatorComponent={() => <View style={{ height: gap }} />}
    />
  );
}

const styles = StyleSheet.create({
  skeletonGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 16,
    gap: 12,
  },
  lista: { padding: 16 },
  loader: { paddingVertical: 24 },
});
```

**9. Cabeçalho**

```tsx
// src/components/AppHeader.tsx
import React from 'react';
import { View, Text, TouchableOpacity, Image, StyleSheet } from 'react-native';

interface Props {
  logado: boolean;
  avatarUrl?: string;
  onMenuTap?: () => void;
  onEntrarTap?: () => void;
  onAvatarTap?: () => void;
}

export function AppHeader({
  logado,
  avatarUrl,
  onMenuTap,
  onEntrarTap,
  onAvatarTap,
}: Props) {
  return (
    <View style={styles.container}>
      <View style={styles.esquerda}>
        {onMenuTap && (
          <TouchableOpacity onPress={onMenuTap} style={styles.menuBtn}>
            <Text style={styles.menuIcon}>☰</Text>
          </TouchableOpacity>
        )}
        <Text style={styles.logo}>⬡ MeuApp</Text>
      </View>

      {logado ? (
        <TouchableOpacity onPress={onAvatarTap}>
          {avatarUrl ? (
            <Image source={{ uri: avatarUrl }} style={styles.avatar} />
          ) : (
            <View style={styles.avatarPlaceholder}>
              <Text style={styles.avatarTexto}>U</Text>
            </View>
          )}
        </TouchableOpacity>
      ) : (
        <TouchableOpacity onPress={onEntrarTap} style={styles.btnEntrar}>
          <Text style={styles.btnEntrarTexto}>Entrar</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingHorizontal: 16,
    paddingVertical: 12,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  esquerda: { flexDirection: 'row', alignItems: 'center', gap: 12 },
  menuBtn: { padding: 4 },
  menuIcon: { fontSize: 22 },
  logo: { fontSize: 18, fontWeight: '700', color: '#3949ab' },
  avatar: { width: 32, height: 32, borderRadius: 16 },
  avatarPlaceholder: {
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: '#c5cae9',
    justifyContent: 'center',
    alignItems: 'center',
  },
  avatarTexto: { color: '#283593', fontWeight: '600', fontSize: 14 },
  btnEntrar: {
    backgroundColor: '#3949ab',
    paddingHorizontal: 16,
    paddingVertical: 6,
    borderRadius: 8,
  },
  btnEntrarTexto: { color: '#fff', fontSize: 14, fontWeight: '600' },
});
```

**10. Sidebar (Drawer content)**

```tsx
// src/components/SidebarContent.tsx
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
} from 'react-native';

interface SidebarItemData {
  icone: string;
  titulo: string;
  tituloReduzido: string;
  rota: string;
}

const ITENS: SidebarItemData[] = [
  { icone: '🏠', titulo: 'Início', tituloReduzido: 'Início', rota: '/' },
  { icone: '🔍', titulo: 'Explorar', tituloReduzido: 'Explorar', rota: '/explorar' },
  { icone: '🔖', titulo: 'Salvos', tituloReduzido: 'Salvos', rota: '/salvos' },
  { icone: '⚙️', titulo: 'Configurações', tituloReduzido: 'Config', rota: '/config' },
];

interface Props {
  expandida?: boolean;
  rotaAtual?: string;
  onNavegar: (rota: string) => void;
}

export function SidebarContent({
  expandida = true,
  rotaAtual,
  onNavegar,
}: Props) {
  return (
    <ScrollView style={styles.container}>
      {ITENS.map((item) => {
        const selecionado = rotaAtual === item.rota;

        if (expandida) {
          return (
            <TouchableOpacity
              key={item.rota}
              style={[styles.itemExpandido, selecionado && styles.itemSelecionado]}
              onPress={() => onNavegar(item.rota)}
            >
              <Text style={styles.icone}>{item.icone}</Text>
              <Text
                style={[
                  styles.tituloExpandido,
                  selecionado && styles.textoSelecionado,
                ]}
              >
                {item.titulo}
              </Text>
            </TouchableOpacity>
          );
        }

        return (
          <TouchableOpacity
            key={item.rota}
            style={[styles.itemCompacto, selecionado && styles.itemSelecionado]}
            onPress={() => onNavegar(item.rota)}
          >
            <Text style={styles.icone}>{item.icone}</Text>
            <Text
              style={[
                styles.tituloCompacto,
                selecionado && styles.textoSelecionado,
              ]}
            >
              {item.tituloReduzido}
            </Text>
          </TouchableOpacity>
        );
      })}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 8 },
  itemExpandido: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 14,
    paddingHorizontal: 16,
    gap: 12,
  },
  itemCompacto: {
    alignItems: 'center',
    paddingVertical: 12,
    paddingHorizontal: 8,
  },
  itemSelecionado: {
    backgroundColor: 'rgba(57, 73, 171, 0.08)',
    borderLeftWidth: 3,
    borderLeftColor: '#3949ab',
  },
  icone: { fontSize: 20 },
  tituloExpandido: { fontSize: 15, color: '#333' },
  tituloCompacto: { fontSize: 10, color: '#666', marginTop: 4 },
  textoSelecionado: { color: '#3949ab', fontWeight: '600' },
});
```

**11. Layout com Drawer + Tabs**

```tsx
// app/_layout.tsx
import React from 'react';
import { useResponsive } from '../src/hooks/useResponsive';
import { MobileLayout } from '../src/layouts/MobileLayout';
import { TabletDesktopLayout } from '../src/layouts/TabletDesktopLayout';

export default function RootLayout() {
  const { screenType } = useResponsive();

  if (screenType === 'mobile') {
    return <MobileLayout />;
  }

  return <TabletDesktopLayout screenType={screenType} />;
}
```

```tsx
// src/layouts/MobileLayout.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { SidebarContent } from '../components/SidebarContent';
import { AppHeader } from '../components/AppHeader';
import HomeScreen from '../../app/(tabs)/index';

const Drawer = createDrawerNavigator();
const Tab = createBottomTabNavigator();

function HomeTabs() {
  return (
    <Tab.Navigator
      screenOptions={{
        header: () => (
          <AppHeader logado={true} onMenuTap={() => {}} />
        ),
      }}
    >
      <Tab.Screen
        name="Início"
        component={HomeScreen}
        options={{ tabBarIcon: () => <Text>🏠</Text> }}
      />
      <Tab.Screen
        name="Explorar"
        component={PlaceholderScreen('Explorar')}
        options={{ tabBarIcon: () => <Text>🔍</Text> }}
      />
      <Tab.Screen
        name="Salvos"
        component={PlaceholderScreen('Salvos')}
        options={{ tabBarIcon: () => <Text>🔖</Text> }}
      />
      <Tab.Screen
        name="Perfil"
        component={PlaceholderScreen('Perfil')}
        options={{ tabBarIcon: () => <Text>👤</Text> }}
      />
    </Tab.Navigator>
  );
}

function PlaceholderScreen(nome: string) {
  return function Screen() {
    return (
      <View style={styles.placeholder}>
        <Text>{nome} — em construção</Text>
      </View>
    );
  };
}

export function MobileLayout() {
  return (
    <Drawer.Navigator
      drawerContent={(props) => (
        <SidebarContent
          expandida
          onNavegar={(rota) => {
            props.navigation.closeDrawer();
          }}
        />
      )}
      screenOptions={{ headerShown: false }}
    >
      <Drawer.Screen name="Main" component={HomeTabs} />
    </Drawer.Navigator>
  );
}

const styles = StyleSheet.create({
  placeholder: { flex: 1, justifyContent: 'center', alignItems: 'center' },
});
```

```tsx
// src/layouts/TabletDesktopLayout.tsx
import React, { useState } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { AppHeader } from '../components/AppHeader';
import { SidebarContent } from '../components/SidebarContent';
import { ScreenType } from '../hooks/useResponsive';
import HomeScreen from '../../app/(tabs)/index';

interface Props {
  screenType: ScreenType;
}

export function TabletDesktopLayout({ screenType }: Props) {
  const [sidebarExpandida, setSidebarExpandida] = useState(
    screenType === 'desktop',
  );
  const larguraSidebar = sidebarExpandida ? 240 : 72;

  return (
    <View style={styles.container}>
      <AppHeader logado={true} onAvatarTap={() => {}} />

      <View style={styles.body}>
        <View style={[styles.sidebar, { width: larguraSidebar }]}>
          {screenType === 'desktop' && (
            <Text
              style={styles.toggleBtn}
              onPress={() => setSidebarExpandida(!sidebarExpandida)}
            >
              {sidebarExpandida ? '◀' : '▶'}
            </Text>
          )}
          <SidebarContent
            expandida={sidebarExpandida}
            rotaAtual="/"
            onNavegar={() => {}}
          />
        </View>

        <View style={styles.conteudo}>
          <HomeScreen />
          <View style={styles.footer}>
            <Text style={styles.footerTexto}>
              © 2025 MeuApp. Todos os direitos reservados.
            </Text>
          </View>
        </View>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fafafa' },
  body: { flex: 1, flexDirection: 'row' },
  sidebar: { backgroundColor: '#f5f5f5', borderRightWidth: 1, borderRightColor: '#e0e0e0' },
  toggleBtn: {
    textAlign: 'center',
    paddingVertical: 12,
    fontSize: 16,
    color: '#666',
  },
  conteudo: { flex: 1 },
  footer: {
    paddingVertical: 16,
    backgroundColor: '#f5f5f5',
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
  },
  footerTexto: { textAlign: 'center', fontSize: 12, color: '#999' },
});
```

---

### React Web — Bootstrap + Font Awesome

**1. Dependências**

```bash
npm create vite@latest meu-app -- --template react-ts
cd meu-app
npm install

# Bootstrap CSS (classes utilitárias, grid, componentes)
npm install bootstrap

# react-bootstrap apenas para componentes dinâmicos (Offcanvas, Collapse)
npm install react-bootstrap

# Font Awesome via classes CSS
npm install @fortawesome/fontawesome-free

# React Router para navegação
npm install react-router-dom
```

**2. Importação global dos estilos — `src/main.tsx`**

```tsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';

import 'bootstrap/dist/css/bootstrap.min.css';
import '@fortawesome/fontawesome-free/css/all.min.css';
import './index.css';

import { App } from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
);
```

**3. Estilos globais auxiliares — `src/index.css`**

```css
/* src/index.css */
:root {
  --sidebar-width-full: 240px;
  --sidebar-width-compact: 72px;
}

body {
  min-height: 100vh;
}

#root {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Skeleton animation */
.skeleton {
  background: linear-gradient(90deg, #e9ecef 25%, #f8f9fa 50%, #e9ecef 75%);
  background-size: 200% 100%;
  animation: skeleton-pulse 1.5s ease-in-out infinite;
  border-radius: 4px;
}

@keyframes skeleton-pulse {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Sidebar transitions */
.sidebar-compact .sidebar-label {
  font-size: 0.625rem;
  display: block;
  text-align: center;
  margin-top: 2px;
}

.sidebar-full .sidebar-item {
  flex-direction: row;
  gap: 0.75rem;
  padding: 0.625rem 1rem;
}

.sidebar-compact .sidebar-item {
  flex-direction: column;
  align-items: center;
  padding: 0.75rem 0.5rem;
}
```

**4. Tipos**

```typescript
// src/types/content.ts
export interface ContentItem {
  id: string;
  titulo: string;
  descricao: string;
  thumbnailUrl: string;
  badges: string[];
}
```

**5. Serviço de conteúdo simulado**

```typescript
// src/services/contentService.ts
import { ContentItem } from '../types/content';

export async function buscarConteudos(
  pagina: number,
  porPagina = 12,
): Promise<ContentItem[]> {
  await new Promise((r) => setTimeout(r, 800));

  return Array.from({ length: porPagina }, (_, i) => {
    const index = (pagina - 1) * porPagina + i + 1;
    return {
      id: `item-${index}`,
      titulo: `Conteúdo ${index}`,
      descricao: `Descrição breve do item ${index} para demonstrar o layout do card.`,
      thumbnailUrl: `https://picsum.photos/seed/item${index}/400/300`,
      badges: ['Tag A', 'Tag B'],
    };
  });
}
```

**6. Hook de scroll infinito**

```typescript
// src/hooks/useInfiniteContent.ts
import { useState, useCallback, useRef } from 'react';
import { ContentItem } from '../types/content';
import { buscarConteudos } from '../services/contentService';

export function useInfiniteContent() {
  const [itens, setItens] = useState<ContentItem[]>([]);
  const [carregando, setCarregando] = useState(false);
  const [temMais, setTemMais] = useState(true);
  const paginaRef = useRef(0);
  const carregandoRef = useRef(false);

  const carregarMais = useCallback(async () => {
    if (carregandoRef.current || !temMais) return;
    carregandoRef.current = true;
    setCarregando(true);

    const proxima = paginaRef.current + 1;
    const novos = await buscarConteudos(proxima);

    paginaRef.current = proxima;
    setItens((prev) => [...prev, ...novos]);
    setTemMais(novos.length >= 12);
    setCarregando(false);
    carregandoRef.current = false;
  }, [temMais]);

  return { itens, carregando, temMais, carregarMais };
}
```

**7. Componente skeleton do card**

```tsx
// src/components/CardSkeleton.tsx
export function CardSkeleton() {
  return (
    <div className="card h-100 border-0 shadow-sm">
      <div className="skeleton" style={{ height: 160 }} />
      <div className="card-body">
        <div className="skeleton mb-2" style={{ height: 16, width: '80%' }} />
        <div className="skeleton mb-3" style={{ height: 12, width: '60%' }} />
        <div className="d-flex gap-2">
          <span className="skeleton" style={{ height: 20, width: 48 }} />
          <span className="skeleton" style={{ height: 20, width: 48 }} />
        </div>
      </div>
    </div>
  );
}
```

**8. Componente do card de conteúdo**

```tsx
// src/components/ContentCard.tsx
import { ContentItem } from '../types/content';

interface Props {
  item: ContentItem;
}

export function ContentCard({ item }: Props) {
  return (
    <div className="card h-100 border-0 shadow-sm">
      <img
        src={item.thumbnailUrl}
        alt={item.titulo}
        className="card-img-top"
        style={{ height: 160, objectFit: 'cover' }}
        loading="lazy"
      />
      <div className="card-body d-flex flex-column">
        <h6 className="card-title text-truncate mb-1">{item.titulo}</h6>
        <p
          className="card-text text-muted small flex-grow-1"
          style={{
            display: '-webkit-box',
            WebkitLineClamp: 2,
            WebkitBoxOrient: 'vertical',
            overflow: 'hidden',
          }}
        >
          {item.descricao}
        </p>
        {item.badges.length > 0 && (
          <div className="d-flex flex-wrap gap-1 mt-2">
            {item.badges.map((b) => (
              <span key={b} className="badge bg-primary-subtle text-primary-emphasis">
                {b}
              </span>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
```

**9. Cabeçalho**

```tsx
// src/components/AppHeader.tsx
interface Props {
  logado: boolean;
  avatarUrl?: string;
  onMenuToggle?: () => void;
  onEntrar?: () => void;
  onAvatar?: () => void;
  mostrarBotaoMenu?: boolean;
}

export function AppHeader({
  logado,
  avatarUrl,
  onMenuToggle,
  onEntrar,
  onAvatar,
  mostrarBotaoMenu = false,
}: Props) {
  return (
    <header className="navbar navbar-light bg-white border-bottom px-3 py-2">
      <div className="d-flex align-items-center gap-3">
        {mostrarBotaoMenu && (
          <button
            className="btn btn-link text-dark p-0"
            onClick={onMenuToggle}
            aria-label="Menu"
          >
            <i className="fas fa-bars fa-lg" />
          </button>
        )}

        <a className="navbar-brand d-flex align-items-center gap-2 mb-0" href="/">
          <i className="fas fa-cubes text-primary" />
          <span className="fw-bold">MeuApp</span>
        </a>
      </div>

      <div>
        {logado ? (
          <button
            className="btn p-0 border-0"
            onClick={onAvatar}
            aria-label="Perfil do usuário"
          >
            {avatarUrl ? (
              <img
                src={avatarUrl}
                alt="Avatar"
                className="rounded-circle"
                width={32}
                height={32}
              />
            ) : (
              <span
                className="d-inline-flex align-items-center justify-content-center rounded-circle bg-primary-subtle text-primary fw-semibold"
                style={{ width: 32, height: 32 }}
              >
                U
              </span>
            )}
          </button>
        ) : (
          <button className="btn btn-primary btn-sm" onClick={onEntrar}>
            <i className="fas fa-sign-in-alt me-1" />
            Entrar
          </button>
        )}
      </div>
    </header>
  );
}
```

**10. Sidebar**

```tsx
// src/components/AppSidebar.tsx
import { NavLink } from 'react-router-dom';

interface SidebarItemData {
  icone: string;
  titulo: string;
  tituloReduzido: string;
  rota: string;
}

const ITENS: SidebarItemData[] = [
  { icone: 'fas fa-home', titulo: 'Início', tituloReduzido: 'Início', rota: '/' },
  { icone: 'fas fa-compass', titulo: 'Explorar', tituloReduzido: 'Explorar', rota: '/explorar' },
  { icone: 'fas fa-bookmark', titulo: 'Salvos', tituloReduzido: 'Salvos', rota: '/salvos' },
  { icone: 'fas fa-cog', titulo: 'Configurações', tituloReduzido: 'Config', rota: '/config' },
];

interface Props {
  expandida: boolean;
  onToggle?: () => void;
  mostrarToggle?: boolean;
}

export function AppSidebar({ expandida, onToggle, mostrarToggle = false }: Props) {
  const classe = expandida ? 'sidebar-full' : 'sidebar-compact';

  return (
    <nav
      className={`d-flex flex-column bg-body-tertiary border-end vh-100 ${classe}`}
      style={{
        width: expandida ? 'var(--sidebar-width-full)' : 'var(--sidebar-width-compact)',
        transition: 'width 0.2s ease',
        position: 'sticky',
        top: 0,
        flexShrink: 0,
      }}
    >
      {mostrarToggle && (
        <button
          className="btn btn-link text-muted text-center py-2 border-bottom"
          onClick={onToggle}
          aria-label={expandida ? 'Recolher menu' : 'Expandir menu'}
        >
          <i className={`fas fa-angles-${expandida ? 'left' : 'right'}`} />
        </button>
      )}

      <ul className="nav flex-column pt-2">
        {ITENS.map((item) => (
          <li key={item.rota} className="nav-item">
            <NavLink
              to={item.rota}
              end={item.rota === '/'}
              className={({ isActive }) =>
                `sidebar-item d-flex text-decoration-none ${
                  isActive
                    ? 'text-primary fw-semibold border-start border-3 border-primary bg-primary-subtle bg-opacity-10'
                    : 'text-body-secondary'
                }`
              }
            >
              <i className={`${item.icone}`} />
              {expandida ? (
                <span>{item.titulo}</span>
              ) : (
                <span className="sidebar-label">{item.tituloReduzido}</span>
              )}
            </NavLink>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

**11. Sidebar mobile (Offcanvas — usa react-bootstrap)**

```tsx
// src/components/AppSidebarMobile.tsx
import Offcanvas from 'react-bootstrap/Offcanvas';
import { NavLink } from 'react-router-dom';

const ITENS = [
  { icone: 'fas fa-home', titulo: 'Início', rota: '/' },
  { icone: 'fas fa-compass', titulo: 'Explorar', rota: '/explorar' },
  { icone: 'fas fa-bookmark', titulo: 'Salvos', rota: '/salvos' },
  { icone: 'fas fa-cog', titulo: 'Configurações', rota: '/config' },
];

interface Props {
  visivel: boolean;
  onFechar: () => void;
}

export function AppSidebarMobile({ visivel, onFechar }: Props) {
  return (
    <Offcanvas show={visivel} onHide={onFechar} placement="start">
      <Offcanvas.Header closeButton>
        <Offcanvas.Title>
          <i className="fas fa-cubes text-primary me-2" />
          MeuApp
        </Offcanvas.Title>
      </Offcanvas.Header>
      <Offcanvas.Body className="p-0">
        <ul className="nav flex-column">
          {ITENS.map((item) => (
            <li key={item.rota} className="nav-item">
              <NavLink
                to={item.rota}
                end={item.rota === '/'}
                onClick={onFechar}
                className={({ isActive }) =>
                  `nav-link d-flex align-items-center gap-3 px-4 py-3 ${
                    isActive
                      ? 'text-primary fw-semibold bg-primary-subtle bg-opacity-10'
                      : 'text-body-secondary'
                  }`
                }
              >
                <i className={item.icone} />
                <span>{item.titulo}</span>
              </NavLink>
            </li>
          ))}
        </ul>
      </Offcanvas.Body>
    </Offcanvas>
  );
}
```

**12. Rodapé mobile (barra inferior)**

```tsx
// src/components/BottomNavBar.tsx
import { NavLink } from 'react-router-dom';

const ITENS = [
  { icone: 'fas fa-home', titulo: 'Início', rota: '/' },
  { icone: 'fas fa-compass', titulo: 'Explorar', rota: '/explorar' },
  { icone: 'fas fa-bookmark', titulo: 'Salvos', rota: '/salvos' },
  { icone: 'fas fa-user', titulo: 'Perfil', rota: '/perfil' },
];

export function BottomNavBar() {
  return (
    <nav
      className="d-flex justify-content-around bg-white border-top py-2 d-md-none"
      style={{ position: 'sticky', bottom: 0, zIndex: 1020 }}
    >
      {ITENS.map((item) => (
        <NavLink
          key={item.rota}
          to={item.rota}
          end={item.rota === '/'}
          className={({ isActive }) =>
            `d-flex flex-column align-items-center text-decoration-none ${
              isActive ? 'text-primary' : 'text-body-tertiary'
            }`
          }
          style={{ fontSize: '0.7rem' }}
        >
          <i className={`${item.icone} mb-1`} style={{ fontSize: '1.1rem' }} />
          <span>{item.titulo}</span>
        </NavLink>
      ))}
    </nav>
  );
}
```

**13. Rodapé desktop/tablet**

```tsx
// src/components/AppFooter.tsx
export function AppFooter() {
  return (
    <footer className="bg-body-tertiary border-top py-3 text-center">
      <small className="text-body-secondary">
        © 2025 MeuApp. Todos os direitos reservados.
      </small>
    </footer>
  );
}
```

**14. Sentinela para IntersectionObserver (scroll infinito)**

```tsx
// src/components/ScrollSentinel.tsx
import { useEffect, useRef } from 'react';

interface Props {
  onIntersect: () => void;
  enabled: boolean;
}

export function ScrollSentinel({ onIntersect, enabled }: Props) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!enabled || !ref.current) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) onIntersect();
      },
      { rootMargin: '200px' },
    );

    observer.observe(ref.current);
    return () => observer.disconnect();
  }, [onIntersect, enabled]);

  return <div ref={ref} />;
}
```

**15. Página Home**

```tsx
// src/pages/HomePage.tsx
import { useEffect } from 'react';
import { ContentCard } from '../components/ContentCard';
import { CardSkeleton } from '../components/CardSkeleton';
import { ScrollSentinel } from '../components/ScrollSentinel';
import { useInfiniteContent } from '../hooks/useInfiniteContent';

export function HomePage() {
  const { itens, carregando, temMais, carregarMais } = useInfiniteContent();

  useEffect(() => {
    carregarMais();
  }, []);

  // Carregamento inicial: grid de skeletons
  if (itens.length === 0 && carregando) {
    return (
      <div className="container-fluid p-4">
        <div className="row g-3">
          {Array.from({ length: 8 }, (_, i) => (
            <div key={i} className="col-6 col-md-4 col-xl-3">
              <CardSkeleton />
            </div>
          ))}
        </div>
      </div>
    );
  }

  return (
    <div className="container-fluid p-4">
      <div className="row g-3">
        {itens.map((item) => (
          <div key={item.id} className="col-6 col-md-4 col-xl-3">
            <ContentCard item={item} />
          </div>
        ))}

        {/* Skeletons durante carregamento de novas páginas */}
        {carregando &&
          Array.from({ length: 4 }, (_, i) => (
            <div key={`skel-${i}`} className="col-6 col-md-4 col-xl-3">
              <CardSkeleton />
            </div>
          ))}
      </div>

      <ScrollSentinel onIntersect={carregarMais} enabled={temMais && !carregando} />
    </div>
  );
}
```

**16. Páginas placeholder**

```tsx
// src/pages/ExplorarPage.tsx
export function ExplorarPage() {
  return (
    <div className="d-flex align-items-center justify-content-center" style={{ minHeight: '50vh' }}>
      <p className="text-muted">Explorar — em construção</p>
    </div>
  );
}

// src/pages/SalvosPage.tsx
export function SalvosPage() {
  return (
    <div className="d-flex align-items-center justify-content-center" style={{ minHeight: '50vh' }}>
      <p className="text-muted">Salvos — em construção</p>
    </div>
  );
}

// src/pages/ConfigPage.tsx
export function ConfigPage() {
  return (
    <div className="d-flex align-items-center justify-content-center" style={{ minHeight: '50vh' }}>
      <p className="text-muted">Configurações — em construção</p>
    </div>
  );
}
```

**17. Layout principal — `src/App.tsx`**

```tsx
// src/App.tsx
import { useState } from 'react';
import { Routes, Route } from 'react-router-dom';
import { AppHeader } from './components/AppHeader';
import { AppSidebar } from './components/AppSidebar';
import { AppSidebarMobile } from './components/AppSidebarMobile';
import { BottomNavBar } from './components/BottomNavBar';
import { AppFooter } from './components/AppFooter';
import { HomePage } from './pages/HomePage';
import { ExplorarPage } from './pages/ExplorarPage';
import { SalvosPage } from './pages/SalvosPage';
import { ConfigPage } from './pages/ConfigPage';

export function App() {
  const [sidebarMobileAberta, setSidebarMobileAberta] = useState(false);
  const [sidebarExpandida, setSidebarExpandida] = useState(true);

  return (
    <>
      {/* Header */}
      <AppHeader
        logado={true}
        mostrarBotaoMenu
        onMenuToggle={() => setSidebarMobileAberta(true)}
        onAvatar={() => {}}
      />

      {/* Sidebar mobile (Offcanvas) — visível apenas em xs/sm */}
      <AppSidebarMobile
        visivel={sidebarMobileAberta}
        onFechar={() => setSidebarMobileAberta(false)}
      />

      <div className="d-flex flex-grow-1" style={{ minHeight: 0 }}>
        {/* Sidebar fixa — visível a partir de md */}
        <div className="d-none d-md-block">
          <AppSidebar
            expandida={sidebarExpandida}
            onToggle={() => setSidebarExpandida(!sidebarExpandida)}
            mostrarToggle
          />
        </div>

        {/* Conteúdo principal + rodapé desktop */}
        <div className="d-flex flex-column flex-grow-1" style={{ minWidth: 0 }}>
          <main className="flex-grow-1">
            <Routes>
              <Route path="/" element={<HomePage />} />
              <Route path="/explorar" element={<ExplorarPage />} />
              <Route path="/salvos" element={<SalvosPage />} />
              <Route path="/config" element={<ConfigPage />} />
            </Routes>
          </main>

          {/* Rodapé ilustrativo — visível a partir de md */}
          <div className="d-none d-md-block">
            <AppFooter />
          </div>
        </div>
      </div>

      {/* Barra inferior mobile — visível apenas em xs/sm */}
      <BottomNavBar />
    </>
  );
}
```

**Estrutura de arquivos:**

```
src/
├── main.tsx
├── index.css
├── App.tsx
├── types/
│   └── content.ts
├── services/
│   └── contentService.ts
├── hooks/
│   └── useInfiniteContent.ts
├── components/
│   ├── AppHeader.tsx
│   ├── AppSidebar.tsx
│   ├── AppSidebarMobile.tsx      ← usa react-bootstrap Offcanvas
│   ├── BottomNavBar.tsx
│   ├── AppFooter.tsx
│   ├── ContentCard.tsx
│   ├── CardSkeleton.tsx
│   └── ScrollSentinel.tsx
└── pages/
    ├── HomePage.tsx
    ├── ExplorarPage.tsx
    ├── SalvosPage.tsx
    └── ConfigPage.tsx
```

---

### Equivalência entre Plataformas — Layout Responsivo

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Grid responsivo | `GridView` + `MediaQuery` | `FlatList` + `numColumns` | Bootstrap `row` + `col-6 col-md-4 col-xl-3` |
| Skeleton loading | `shimmer` package | `Animated` opacity loop | CSS `@keyframes` gradient |
| Scroll infinito | `ScrollController` + listener | `FlatList.onEndReached` | `IntersectionObserver` |
| Sidebar mobile | `Drawer` widget | `@react-navigation/drawer` | `react-bootstrap` `Offcanvas` |
| Sidebar tablet | Widget fixo (`Row`) com `width: 72` | `View` fixa com `width: 72` | CSS `width: var(--sidebar-width-compact)` |
| Sidebar desktop toggle | `AnimatedContainer` + `StateProvider` | `useState` + `width` | CSS transition + `useState` |
| Bottom nav mobile | `NavigationBar` | `createBottomTabNavigator` | `NavLink` com `d-md-none` |
| Cabeçalho | `AppBar` customizado | Componente customizado | `navbar` Bootstrap |
| Rodapé desktop | Widget customizado | `View` com texto | `<footer>` com classes Bootstrap |
| Rotas (Flutter) | GoRouter `ShellRoute` / Modular `ChildRoute` | — | — |
| Ícones | Material Icons (built-in) | Emoji / `@expo/vector-icons` | Font Awesome classes (`fas fa-*`) |
| CSS framework | — | — | Bootstrap 5 (classes) + `react-bootstrap` (dinâmico) |
| Breakpoints | `MediaQuery.sizeOf(context).width` | `useWindowDimensions` | Bootstrap breakpoints (`sm/md/lg/xl`) |

---

## 5. Formulários Complexos com Validação e Máscaras

Formulário multi-etapas (stepper) com:

- **Etapa 1 — Dados pessoais:** nome, e-mail, CPF (com máscara `000.000.000-00`), telefone (com máscara `(00) 00000-0000`).
- **Etapa 2 — Endereço:** CEP (com máscara `00000-000`), logradouro, cidade, estado.
- **Etapa 3 — Foto e revisão:** upload de imagem de perfil, resumo dos dados, confirmação.
- Validação em tempo real com mensagens de erro por campo.

### Flutter — Form + mask_text_input_formatter

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  mask_text_input_formatter: ^2.9.0
  image_picker: ^1.1.0
```

```bash
flutter pub get
```

**2. Validadores**

```dart
// lib/core/validators.dart
class Validators {
  static String? obrigatorio(String? valor) {
    if (valor == null || valor.trim().isEmpty) return 'Campo obrigatório';
    return null;
  }

  static String? email(String? valor) {
    if (valor == null || valor.trim().isEmpty) return 'Campo obrigatório';
    final regex = RegExp(r'^[\w\-.]+@([\w\-]+\.)+[\w\-]{2,4}$');
    if (!regex.hasMatch(valor.trim())) return 'E-mail inválido';
    return null;
  }

  static String? cpf(String? valor) {
    if (valor == null || valor.trim().isEmpty) return 'Campo obrigatório';
    final digits = valor.replaceAll(RegExp(r'\D'), '');
    if (digits.length != 11) return 'CPF inválido';
    if (RegExp(r'^(\d)\1{10}$').hasMatch(digits)) return 'CPF inválido';

    int soma = 0;
    for (int i = 0; i < 9; i++) {
      soma += int.parse(digits[i]) * (10 - i);
    }
    int resto = (soma * 10) % 11;
    if (resto == 10) resto = 0;
    if (resto != int.parse(digits[9])) return 'CPF inválido';

    soma = 0;
    for (int i = 0; i < 10; i++) {
      soma += int.parse(digits[i]) * (11 - i);
    }
    resto = (soma * 10) % 11;
    if (resto == 10) resto = 0;
    if (resto != int.parse(digits[10])) return 'CPF inválido';

    return null;
  }

  static String? telefone(String? valor) {
    if (valor == null || valor.trim().isEmpty) return 'Campo obrigatório';
    final digits = valor.replaceAll(RegExp(r'\D'), '');
    if (digits.length < 10 || digits.length > 11) return 'Telefone inválido';
    return null;
  }

  static String? cep(String? valor) {
    if (valor == null || valor.trim().isEmpty) return 'Campo obrigatório';
    final digits = valor.replaceAll(RegExp(r'\D'), '');
    if (digits.length != 8) return 'CEP inválido';
    return null;
  }
}
```

**3. Formulário multi-etapas**

```dart
// lib/features/formulario/presentation/formulario_screen.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import 'package:mask_text_input_formatter/mask_text_input_formatter.dart';
import '../../../core/validators.dart';

class FormularioScreen extends StatefulWidget {
  const FormularioScreen({super.key});

  @override
  State<FormularioScreen> createState() => _FormularioScreenState();
}

class _FormularioScreenState extends State<FormularioScreen> {
  int _etapaAtual = 0;

  // Form keys por etapa
  final _formKeyEtapa1 = GlobalKey<FormState>();
  final _formKeyEtapa2 = GlobalKey<FormState>();

  // Controllers
  final _nomeCtrl = TextEditingController();
  final _emailCtrl = TextEditingController();
  final _cpfCtrl = TextEditingController();
  final _telefoneCtrl = TextEditingController();
  final _cepCtrl = TextEditingController();
  final _logradouroCtrl = TextEditingController();
  final _cidadeCtrl = TextEditingController();
  final _estadoCtrl = TextEditingController();

  // Máscaras
  final _cpfMask = MaskTextInputFormatter(
    mask: '###.###.###-##',
    filter: {'#': RegExp(r'[0-9]')},
  );
  final _telefoneMask = MaskTextInputFormatter(
    mask: '(##) #####-####',
    filter: {'#': RegExp(r'[0-9]')},
  );
  final _cepMask = MaskTextInputFormatter(
    mask: '#####-###',
    filter: {'#': RegExp(r'[0-9]')},
  );

  File? _imagemPerfil;
  bool _enviando = false;

  Future<void> _selecionarImagem() async {
    final picker = ImagePicker();
    final imagem = await picker.pickImage(
      source: ImageSource.gallery,
      maxWidth: 512,
      maxHeight: 512,
      imageQuality: 80,
    );
    if (imagem != null) {
      setState(() => _imagemPerfil = File(imagem.path));
    }
  }

  void _proximaEtapa() {
    if (_etapaAtual == 0 && !_formKeyEtapa1.currentState!.validate()) return;
    if (_etapaAtual == 1 && !_formKeyEtapa2.currentState!.validate()) return;
    setState(() => _etapaAtual++);
  }

  void _etapaAnterior() {
    if (_etapaAtual > 0) setState(() => _etapaAtual--);
  }

  Future<void> _enviar() async {
    setState(() => _enviando = true);
    await Future.delayed(const Duration(seconds: 2));
    if (mounted) {
      setState(() => _enviando = false);
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Cadastro enviado com sucesso!')),
      );
    }
  }

  @override
  void dispose() {
    _nomeCtrl.dispose();
    _emailCtrl.dispose();
    _cpfCtrl.dispose();
    _telefoneCtrl.dispose();
    _cepCtrl.dispose();
    _logradouroCtrl.dispose();
    _cidadeCtrl.dispose();
    _estadoCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cadastro')),
      body: Stepper(
        currentStep: _etapaAtual,
        onStepContinue: _etapaAtual < 2 ? _proximaEtapa : _enviar,
        onStepCancel: _etapaAnterior,
        controlsBuilder: (context, details) {
          return Padding(
            padding: const EdgeInsets.only(top: 16),
            child: Row(
              children: [
                FilledButton(
                  onPressed: _enviando ? null : details.onStepContinue,
                  child: Text(_etapaAtual == 2 ? 'Enviar' : 'Próximo'),
                ),
                if (_etapaAtual > 0) ...[
                  const SizedBox(width: 12),
                  OutlinedButton(
                    onPressed: details.onStepCancel,
                    child: const Text('Voltar'),
                  ),
                ],
              ],
            ),
          );
        },
        steps: [
          // --- Etapa 1: Dados pessoais ---
          Step(
            title: const Text('Dados pessoais'),
            isActive: _etapaAtual >= 0,
            state: _etapaAtual > 0 ? StepState.complete : StepState.indexed,
            content: Form(
              key: _formKeyEtapa1,
              child: Column(
                children: [
                  TextFormField(
                    controller: _nomeCtrl,
                    decoration: const InputDecoration(
                      labelText: 'Nome completo',
                      prefixIcon: Icon(Icons.person),
                    ),
                    validator: Validators.obrigatorio,
                    textInputAction: TextInputAction.next,
                  ),
                  const SizedBox(height: 16),
                  TextFormField(
                    controller: _emailCtrl,
                    decoration: const InputDecoration(
                      labelText: 'E-mail',
                      prefixIcon: Icon(Icons.email),
                    ),
                    validator: Validators.email,
                    keyboardType: TextInputType.emailAddress,
                    textInputAction: TextInputAction.next,
                  ),
                  const SizedBox(height: 16),
                  TextFormField(
                    controller: _cpfCtrl,
                    decoration: const InputDecoration(
                      labelText: 'CPF',
                      prefixIcon: Icon(Icons.badge),
                      hintText: '000.000.000-00',
                    ),
                    validator: Validators.cpf,
                    inputFormatters: [_cpfMask],
                    keyboardType: TextInputType.number,
                    textInputAction: TextInputAction.next,
                  ),
                  const SizedBox(height: 16),
                  TextFormField(
                    controller: _telefoneCtrl,
                    decoration: const InputDecoration(
                      labelText: 'Telefone',
                      prefixIcon: Icon(Icons.phone),
                      hintText: '(00) 00000-0000',
                    ),
                    validator: Validators.telefone,
                    inputFormatters: [_telefoneMask],
                    keyboardType: TextInputType.phone,
                  ),
                ],
              ),
            ),
          ),

          // --- Etapa 2: Endereço ---
          Step(
            title: const Text('Endereço'),
            isActive: _etapaAtual >= 1,
            state: _etapaAtual > 1 ? StepState.complete : StepState.indexed,
            content: Form(
              key: _formKeyEtapa2,
              child: Column(
                children: [
                  TextFormField(
                    controller: _cepCtrl,
                    decoration: const InputDecoration(
                      labelText: 'CEP',
                      prefixIcon: Icon(Icons.location_on),
                      hintText: '00000-000',
                    ),
                    validator: Validators.cep,
                    inputFormatters: [_cepMask],
                    keyboardType: TextInputType.number,
                    textInputAction: TextInputAction.next,
                  ),
                  const SizedBox(height: 16),
                  TextFormField(
                    controller: _logradouroCtrl,
                    decoration: const InputDecoration(
                      labelText: 'Logradouro',
                      prefixIcon: Icon(Icons.home),
                    ),
                    validator: Validators.obrigatorio,
                    textInputAction: TextInputAction.next,
                  ),
                  const SizedBox(height: 16),
                  Row(
                    children: [
                      Expanded(
                        flex: 3,
                        child: TextFormField(
                          controller: _cidadeCtrl,
                          decoration: const InputDecoration(labelText: 'Cidade'),
                          validator: Validators.obrigatorio,
                          textInputAction: TextInputAction.next,
                        ),
                      ),
                      const SizedBox(width: 16),
                      Expanded(
                        child: TextFormField(
                          controller: _estadoCtrl,
                          decoration: const InputDecoration(labelText: 'UF'),
                          validator: Validators.obrigatorio,
                          maxLength: 2,
                          textCapitalization: TextCapitalization.characters,
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),

          // --- Etapa 3: Foto e revisão ---
          Step(
            title: const Text('Foto e Revisão'),
            isActive: _etapaAtual >= 2,
            content: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Center(
                  child: GestureDetector(
                    onTap: _selecionarImagem,
                    child: CircleAvatar(
                      radius: 56,
                      backgroundImage: _imagemPerfil != null
                          ? FileImage(_imagemPerfil!)
                          : null,
                      child: _imagemPerfil == null
                          ? const Column(
                              mainAxisAlignment: MainAxisAlignment.center,
                              children: [
                                Icon(Icons.camera_alt, size: 32),
                                SizedBox(height: 4),
                                Text('Adicionar foto',
                                    style: TextStyle(fontSize: 10)),
                              ],
                            )
                          : null,
                    ),
                  ),
                ),
                const SizedBox(height: 24),
                const Text('Resumo',
                    style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
                const Divider(),
                _LinhaResumo(rotulo: 'Nome', valor: _nomeCtrl.text),
                _LinhaResumo(rotulo: 'E-mail', valor: _emailCtrl.text),
                _LinhaResumo(rotulo: 'CPF', valor: _cpfCtrl.text),
                _LinhaResumo(rotulo: 'Telefone', valor: _telefoneCtrl.text),
                _LinhaResumo(rotulo: 'CEP', valor: _cepCtrl.text),
                _LinhaResumo(
                  rotulo: 'Endereço',
                  valor:
                      '${_logradouroCtrl.text}, ${_cidadeCtrl.text} - ${_estadoCtrl.text}',
                ),
                if (_enviando) ...[
                  const SizedBox(height: 16),
                  const Center(child: CircularProgressIndicator()),
                ],
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class _LinhaResumo extends StatelessWidget {
  final String rotulo;
  final String valor;
  const _LinhaResumo({required this.rotulo, required this.valor});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            width: 80,
            child: Text('$rotulo:',
                style: const TextStyle(fontWeight: FontWeight.w500)),
          ),
          Expanded(child: Text(valor)),
        ],
      ),
    );
  }
}
```

---

### React Native — React Hook Form + react-native-mask-input

**1. Dependências**

```bash
npm install react-hook-form
npx expo install react-native-mask-input expo-image-picker
```

**2. Validadores**

```typescript
// src/core/validators.ts
export function validarCPF(valor: string): string | true {
  const digits = valor.replace(/\D/g, '');
  if (digits.length !== 11) return 'CPF inválido';
  if (/^(\d)\1{10}$/.test(digits)) return 'CPF inválido';

  let soma = 0;
  for (let i = 0; i < 9; i++) soma += Number(digits[i]) * (10 - i);
  let resto = (soma * 10) % 11;
  if (resto === 10) resto = 0;
  if (resto !== Number(digits[9])) return 'CPF inválido';

  soma = 0;
  for (let i = 0; i < 10; i++) soma += Number(digits[i]) * (11 - i);
  resto = (soma * 10) % 11;
  if (resto === 10) resto = 0;
  if (resto !== Number(digits[10])) return 'CPF inválido';

  return true;
}

export function validarEmail(valor: string): string | true {
  const regex = /^[\w\-.]+@([\w\-]+\.)+[\w\-]{2,4}$/;
  return regex.test(valor.trim()) ? true : 'E-mail inválido';
}

export function validarTelefone(valor: string): string | true {
  const digits = valor.replace(/\D/g, '');
  return digits.length >= 10 && digits.length <= 11 ? true : 'Telefone inválido';
}

export function validarCEP(valor: string): string | true {
  const digits = valor.replace(/\D/g, '');
  return digits.length === 8 ? true : 'CEP inválido';
}
```

**3. Formulário multi-etapas**

```tsx
// src/features/formulario/FormularioScreen.tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  Image,
  ScrollView,
  StyleSheet,
  ActivityIndicator,
  Alert,
} from 'react-native';
import { useForm, Controller } from 'react-hook-form';
import { MaskedTextInput } from 'react-native-mask-input';
import * as ImagePicker from 'expo-image-picker';
import { validarCPF, validarEmail, validarTelefone, validarCEP } from '../../core/validators';

interface FormData {
  nome: string;
  email: string;
  cpf: string;
  telefone: string;
  cep: string;
  logradouro: string;
  cidade: string;
  estado: string;
}

export function FormularioScreen() {
  const [etapa, setEtapa] = useState(0);
  const [imagemUri, setImagemUri] = useState<string | null>(null);
  const [enviando, setEnviando] = useState(false);

  const {
    control,
    handleSubmit,
    trigger,
    getValues,
    formState: { errors },
  } = useForm<FormData>({ mode: 'onBlur' });

  const selecionarImagem = async () => {
    const resultado = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ['images'],
      allowsEditing: true,
      aspect: [1, 1],
      quality: 0.8,
    });
    if (!resultado.canceled) {
      setImagemUri(resultado.assets[0].uri);
    }
  };

  const proximaEtapa = async () => {
    const camposEtapa: (keyof FormData)[][] = [
      ['nome', 'email', 'cpf', 'telefone'],
      ['cep', 'logradouro', 'cidade', 'estado'],
    ];
    const valido = await trigger(camposEtapa[etapa]);
    if (valido) setEtapa(etapa + 1);
  };

  const enviar = async () => {
    setEnviando(true);
    await new Promise((r) => setTimeout(r, 2000));
    setEnviando(false);
    Alert.alert('Sucesso', 'Cadastro enviado com sucesso!');
  };

  const dados = getValues();

  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.content}>
      {/* Indicador de etapas */}
      <View style={styles.indicador}>
        {['Dados', 'Endereço', 'Revisão'].map((label, i) => (
          <View key={label} style={styles.indicadorItem}>
            <View
              style={[
                styles.circulo,
                i <= etapa ? styles.circuloAtivo : styles.circuloInativo,
              ]}
            >
              <Text style={styles.circuloTexto}>{i + 1}</Text>
            </View>
            <Text style={styles.indicadorLabel}>{label}</Text>
          </View>
        ))}
      </View>

      {/* Etapa 1: Dados pessoais */}
      {etapa === 0 && (
        <View style={styles.etapa}>
          <Controller
            control={control}
            name="nome"
            rules={{ required: 'Campo obrigatório' }}
            render={({ field: { onChange, onBlur, value } }) => (
              <View style={styles.campo}>
                <Text style={styles.label}>Nome completo</Text>
                <TextInput
                  style={[styles.input, errors.nome && styles.inputErro]}
                  onBlur={onBlur}
                  onChangeText={onChange}
                  value={value}
                  placeholder="Seu nome"
                />
                {errors.nome && (
                  <Text style={styles.erro}>{errors.nome.message}</Text>
                )}
              </View>
            )}
          />
          <Controller
            control={control}
            name="email"
            rules={{ required: 'Campo obrigatório', validate: validarEmail }}
            render={({ field: { onChange, onBlur, value } }) => (
              <View style={styles.campo}>
                <Text style={styles.label}>E-mail</Text>
                <TextInput
                  style={[styles.input, errors.email && styles.inputErro]}
                  onBlur={onBlur}
                  onChangeText={onChange}
                  value={value}
                  placeholder="email@exemplo.com"
                  keyboardType="email-address"
                  autoCapitalize="none"
                />
                {errors.email && (
                  <Text style={styles.erro}>{errors.email.message}</Text>
                )}
              </View>
            )}
          />
          <Controller
            control={control}
            name="cpf"
            rules={{ required: 'Campo obrigatório', validate: validarCPF }}
            render={({ field: { onChange, onBlur, value } }) => (
              <View style={styles.campo}>
                <Text style={styles.label}>CPF</Text>
                <MaskedTextInput
                  mask="999.999.999-99"
                  style={[styles.input, errors.cpf && styles.inputErro]}
                  onBlur={onBlur}
                  onChangeText={onChange}
                  value={value}
                  placeholder="000.000.000-00"
                  keyboardType="numeric"
                />
                {errors.cpf && (
                  <Text style={styles.erro}>{errors.cpf.message}</Text>
                )}
              </View>
            )}
          />
          <Controller
            control={control}
            name="telefone"
            rules={{ required: 'Campo obrigatório', validate: validarTelefone }}
            render={({ field: { onChange, onBlur, value } }) => (
              <View style={styles.campo}>
                <Text style={styles.label}>Telefone</Text>
                <MaskedTextInput
                  mask="(99) 99999-9999"
                  style={[styles.input, errors.telefone && styles.inputErro]}
                  onBlur={onBlur}
                  onChangeText={onChange}
                  value={value}
                  placeholder="(00) 00000-0000"
                  keyboardType="phone-pad"
                />
                {errors.telefone && (
                  <Text style={styles.erro}>{errors.telefone.message}</Text>
                )}
              </View>
            )}
          />
        </View>
      )}

      {/* Etapa 2: Endereço */}
      {etapa === 1 && (
        <View style={styles.etapa}>
          <Controller
            control={control}
            name="cep"
            rules={{ required: 'Campo obrigatório', validate: validarCEP }}
            render={({ field: { onChange, onBlur, value } }) => (
              <View style={styles.campo}>
                <Text style={styles.label}>CEP</Text>
                <MaskedTextInput
                  mask="99999-999"
                  style={[styles.input, errors.cep && styles.inputErro]}
                  onBlur={onBlur}
                  onChangeText={onChange}
                  value={value}
                  placeholder="00000-000"
                  keyboardType="numeric"
                />
                {errors.cep && (
                  <Text style={styles.erro}>{errors.cep.message}</Text>
                )}
              </View>
            )}
          />
          <Controller
            control={control}
            name="logradouro"
            rules={{ required: 'Campo obrigatório' }}
            render={({ field: { onChange, onBlur, value } }) => (
              <View style={styles.campo}>
                <Text style={styles.label}>Logradouro</Text>
                <TextInput
                  style={[styles.input, errors.logradouro && styles.inputErro]}
                  onBlur={onBlur}
                  onChangeText={onChange}
                  value={value}
                  placeholder="Rua, Av, etc."
                />
                {errors.logradouro && (
                  <Text style={styles.erro}>{errors.logradouro.message}</Text>
                )}
              </View>
            )}
          />
          <View style={styles.linhaCampos}>
            <View style={{ flex: 3 }}>
              <Controller
                control={control}
                name="cidade"
                rules={{ required: 'Campo obrigatório' }}
                render={({ field: { onChange, onBlur, value } }) => (
                  <View style={styles.campo}>
                    <Text style={styles.label}>Cidade</Text>
                    <TextInput
                      style={[styles.input, errors.cidade && styles.inputErro]}
                      onBlur={onBlur}
                      onChangeText={onChange}
                      value={value}
                    />
                    {errors.cidade && (
                      <Text style={styles.erro}>{errors.cidade.message}</Text>
                    )}
                  </View>
                )}
              />
            </View>
            <View style={{ flex: 1, marginLeft: 12 }}>
              <Controller
                control={control}
                name="estado"
                rules={{ required: 'Obrigatório' }}
                render={({ field: { onChange, onBlur, value } }) => (
                  <View style={styles.campo}>
                    <Text style={styles.label}>UF</Text>
                    <TextInput
                      style={[styles.input, errors.estado && styles.inputErro]}
                      onBlur={onBlur}
                      onChangeText={(t) => onChange(t.toUpperCase())}
                      value={value}
                      maxLength={2}
                      autoCapitalize="characters"
                    />
                  </View>
                )}
              />
            </View>
          </View>
        </View>
      )}

      {/* Etapa 3: Foto e revisão */}
      {etapa === 2 && (
        <View style={styles.etapa}>
          <TouchableOpacity onPress={selecionarImagem} style={styles.avatarBtn}>
            {imagemUri ? (
              <Image source={{ uri: imagemUri }} style={styles.avatarImg} />
            ) : (
              <View style={styles.avatarPlaceholder}>
                <Text style={styles.avatarIcon}>📷</Text>
                <Text style={styles.avatarTexto}>Adicionar foto</Text>
              </View>
            )}
          </TouchableOpacity>

          <View style={styles.resumo}>
            <Text style={styles.resumoTitulo}>Resumo</Text>
            <LinhaResumo rotulo="Nome" valor={dados.nome} />
            <LinhaResumo rotulo="E-mail" valor={dados.email} />
            <LinhaResumo rotulo="CPF" valor={dados.cpf} />
            <LinhaResumo rotulo="Telefone" valor={dados.telefone} />
            <LinhaResumo rotulo="CEP" valor={dados.cep} />
            <LinhaResumo
              rotulo="Endereço"
              valor={`${dados.logradouro}, ${dados.cidade} - ${dados.estado}`}
            />
          </View>

          {enviando && <ActivityIndicator size="large" style={{ marginTop: 16 }} />}
        </View>
      )}

      {/* Botões */}
      <View style={styles.botoes}>
        {etapa > 0 && (
          <TouchableOpacity
            style={styles.btnVoltar}
            onPress={() => setEtapa(etapa - 1)}
          >
            <Text style={styles.btnVoltarTexto}>Voltar</Text>
          </TouchableOpacity>
        )}
        <TouchableOpacity
          style={styles.btnProximo}
          onPress={etapa < 2 ? proximaEtapa : handleSubmit(enviar)}
          disabled={enviando}
        >
          <Text style={styles.btnProximoTexto}>
            {etapa === 2 ? 'Enviar' : 'Próximo'}
          </Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
}

function LinhaResumo({ rotulo, valor }: { rotulo: string; valor: string }) {
  return (
    <View style={styles.linhaResumo}>
      <Text style={styles.linhaRotulo}>{rotulo}:</Text>
      <Text style={styles.linhaValor}>{valor}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  content: { padding: 20 },
  indicador: { flexDirection: 'row', justifyContent: 'center', gap: 24, marginBottom: 24 },
  indicadorItem: { alignItems: 'center' },
  circulo: { width: 32, height: 32, borderRadius: 16, justifyContent: 'center', alignItems: 'center' },
  circuloAtivo: { backgroundColor: '#3949ab' },
  circuloInativo: { backgroundColor: '#e0e0e0' },
  circuloTexto: { color: '#fff', fontWeight: '700' },
  indicadorLabel: { fontSize: 11, marginTop: 4, color: '#666' },
  etapa: { marginBottom: 16 },
  campo: { marginBottom: 16 },
  label: { fontSize: 14, fontWeight: '500', marginBottom: 6, color: '#333' },
  input: { borderWidth: 1, borderColor: '#ccc', borderRadius: 8, paddingHorizontal: 12, paddingVertical: 10, fontSize: 15 },
  inputErro: { borderColor: '#e53935' },
  erro: { color: '#e53935', fontSize: 12, marginTop: 4 },
  linhaCampos: { flexDirection: 'row' },
  avatarBtn: { alignSelf: 'center', marginBottom: 24 },
  avatarImg: { width: 112, height: 112, borderRadius: 56 },
  avatarPlaceholder: { width: 112, height: 112, borderRadius: 56, backgroundColor: '#e8eaf6', justifyContent: 'center', alignItems: 'center' },
  avatarIcon: { fontSize: 28 },
  avatarTexto: { fontSize: 11, color: '#666', marginTop: 4 },
  resumo: { backgroundColor: '#fafafa', borderRadius: 12, padding: 16 },
  resumoTitulo: { fontWeight: '700', fontSize: 16, marginBottom: 12 },
  linhaResumo: { flexDirection: 'row', marginBottom: 6 },
  linhaRotulo: { width: 80, fontWeight: '500', color: '#555' },
  linhaValor: { flex: 1, color: '#333' },
  botoes: { flexDirection: 'row', justifyContent: 'flex-end', gap: 12, marginTop: 8 },
  btnVoltar: { paddingHorizontal: 24, paddingVertical: 12, borderRadius: 8, borderWidth: 1, borderColor: '#ccc' },
  btnVoltarTexto: { color: '#666' },
  btnProximo: { paddingHorizontal: 24, paddingVertical: 12, borderRadius: 8, backgroundColor: '#3949ab' },
  btnProximoTexto: { color: '#fff', fontWeight: '600' },
});
```

---

### React Web — Bootstrap Forms + validação customizada

**1. Dependências**

```bash
npm install react-router-dom
# Bootstrap e Font Awesome já instalados (seção 4)
```

**2. Validadores** (mesmo arquivo, reutilizável)

```typescript
// src/core/validators.ts
export function validarCPF(valor: string): string | null {
  const digits = valor.replace(/\D/g, '');
  if (digits.length !== 11) return 'CPF inválido';
  if (/^(\d)\1{10}$/.test(digits)) return 'CPF inválido';

  let soma = 0;
  for (let i = 0; i < 9; i++) soma += Number(digits[i]) * (10 - i);
  let resto = (soma * 10) % 11;
  if (resto === 10) resto = 0;
  if (resto !== Number(digits[9])) return 'CPF inválido';

  soma = 0;
  for (let i = 0; i < 10; i++) soma += Number(digits[i]) * (11 - i);
  resto = (soma * 10) % 11;
  if (resto === 10) resto = 0;
  if (resto !== Number(digits[10])) return 'CPF inválido';

  return null;
}

export function validarEmail(valor: string): string | null {
  const regex = /^[\w\-.]+@([\w\-]+\.)+[\w\-]{2,4}$/;
  return regex.test(valor.trim()) ? null : 'E-mail inválido';
}

export function validarTelefone(valor: string): string | null {
  const digits = valor.replace(/\D/g, '');
  return digits.length >= 10 && digits.length <= 11 ? null : 'Telefone inválido';
}

export function validarCEP(valor: string): string | null {
  const digits = valor.replace(/\D/g, '');
  return digits.length === 8 ? null : 'CEP inválido';
}
```

**3. Hook de máscara (sem lib externa)**

```typescript
// src/hooks/useMask.ts
import { useCallback } from 'react';

type MaskPattern = 'cpf' | 'telefone' | 'cep';

const MASKS: Record<MaskPattern, { mask: string; maxLength: number }> = {
  cpf: { mask: '###.###.###-##', maxLength: 14 },
  telefone: { mask: '(##) #####-####', maxLength: 15 },
  cep: { mask: '#####-###', maxLength: 9 },
};

export function aplicarMascara(valor: string, pattern: MaskPattern): string {
  const { mask } = MASKS[pattern];
  const digits = valor.replace(/\D/g, '');
  let resultado = '';
  let digitIndex = 0;

  for (let i = 0; i < mask.length && digitIndex < digits.length; i++) {
    if (mask[i] === '#') {
      resultado += digits[digitIndex++];
    } else {
      resultado += mask[i];
    }
  }

  return resultado;
}

export function useMask(pattern: MaskPattern) {
  const onChange = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => {
      e.target.value = aplicarMascara(e.target.value, pattern);
    },
    [pattern],
  );

  return { onChange, maxLength: MASKS[pattern].maxLength };
}
```

**4. Formulário multi-etapas**

```tsx
// src/features/formulario/FormularioPage.tsx
import { useState, useRef } from 'react';
import { validarCPF, validarEmail, validarTelefone, validarCEP } from '../../core/validators';
import { aplicarMascara } from '../../hooks/useMask';

interface FormData {
  nome: string;
  email: string;
  cpf: string;
  telefone: string;
  cep: string;
  logradouro: string;
  cidade: string;
  estado: string;
}

interface FormErrors {
  [campo: string]: string | null;
}

export function FormularioPage() {
  const [etapa, setEtapa] = useState(0);
  const [dados, setDados] = useState<FormData>({
    nome: '', email: '', cpf: '', telefone: '',
    cep: '', logradouro: '', cidade: '', estado: '',
  });
  const [erros, setErros] = useState<FormErrors>({});
  const [imagemUrl, setImagemUrl] = useState<string | null>(null);
  const [enviando, setEnviando] = useState(false);
  const [enviado, setEnviado] = useState(false);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const setCampo = (campo: keyof FormData, valor: string) => {
    setDados((prev) => ({ ...prev, [campo]: valor }));
    setErros((prev) => ({ ...prev, [campo]: null }));
  };

  const setCampoMascarado = (
    campo: keyof FormData,
    valor: string,
    mascara: 'cpf' | 'telefone' | 'cep',
  ) => {
    setCampo(campo, aplicarMascara(valor, mascara));
  };

  const validarEtapa = (etapaNum: number): boolean => {
    const novosErros: FormErrors = {};

    if (etapaNum === 0) {
      if (!dados.nome.trim()) novosErros.nome = 'Campo obrigatório';
      novosErros.email = validarEmail(dados.email) ?? (!dados.email.trim() ? 'Campo obrigatório' : null);
      novosErros.cpf = !dados.cpf.trim() ? 'Campo obrigatório' : validarCPF(dados.cpf);
      novosErros.telefone = !dados.telefone.trim() ? 'Campo obrigatório' : validarTelefone(dados.telefone);
    }

    if (etapaNum === 1) {
      novosErros.cep = !dados.cep.trim() ? 'Campo obrigatório' : validarCEP(dados.cep);
      if (!dados.logradouro.trim()) novosErros.logradouro = 'Campo obrigatório';
      if (!dados.cidade.trim()) novosErros.cidade = 'Campo obrigatório';
      if (!dados.estado.trim()) novosErros.estado = 'Campo obrigatório';
    }

    // Remove entradas nulas
    const errosFiltrados: FormErrors = {};
    for (const [k, v] of Object.entries(novosErros)) {
      if (v) errosFiltrados[k] = v;
    }

    setErros(errosFiltrados);
    return Object.keys(errosFiltrados).length === 0;
  };

  const proximaEtapa = () => {
    if (validarEtapa(etapa)) setEtapa(etapa + 1);
  };

  const selecionarImagem = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) setImagemUrl(URL.createObjectURL(file));
  };

  const enviar = async () => {
    setEnviando(true);
    await new Promise((r) => setTimeout(r, 2000));
    setEnviando(false);
    setEnviado(true);
  };

  if (enviado) {
    return (
      <div className="container py-5 text-center">
        <i className="fas fa-check-circle text-success fa-3x mb-3" />
        <h4>Cadastro enviado com sucesso!</h4>
      </div>
    );
  }

  const etapas = ['Dados pessoais', 'Endereço', 'Foto e Revisão'];

  return (
    <div className="container py-4" style={{ maxWidth: 640 }}>
      {/* Indicador de etapas */}
      <div className="d-flex justify-content-center gap-4 mb-4">
        {etapas.map((label, i) => (
          <div key={label} className="text-center">
            <span
              className={`d-inline-flex align-items-center justify-content-center rounded-circle fw-bold text-white ${
                i <= etapa ? 'bg-primary' : 'bg-secondary'
              }`}
              style={{ width: 32, height: 32 }}
            >
              {i < etapa ? <i className="fas fa-check" /> : i + 1}
            </span>
            <div className="small text-muted mt-1">{label}</div>
          </div>
        ))}
      </div>

      {/* Etapa 1 */}
      {etapa === 0 && (
        <>
          <div className="mb-3">
            <label className="form-label">Nome completo</label>
            <div className="input-group">
              <span className="input-group-text"><i className="fas fa-user" /></span>
              <input
                type="text"
                className={`form-control ${erros.nome ? 'is-invalid' : ''}`}
                value={dados.nome}
                onChange={(e) => setCampo('nome', e.target.value)}
              />
              {erros.nome && <div className="invalid-feedback">{erros.nome}</div>}
            </div>
          </div>
          <div className="mb-3">
            <label className="form-label">E-mail</label>
            <div className="input-group">
              <span className="input-group-text"><i className="fas fa-envelope" /></span>
              <input
                type="email"
                className={`form-control ${erros.email ? 'is-invalid' : ''}`}
                value={dados.email}
                onChange={(e) => setCampo('email', e.target.value)}
                placeholder="email@exemplo.com"
              />
              {erros.email && <div className="invalid-feedback">{erros.email}</div>}
            </div>
          </div>
          <div className="mb-3">
            <label className="form-label">CPF</label>
            <div className="input-group">
              <span className="input-group-text"><i className="fas fa-id-card" /></span>
              <input
                type="text"
                className={`form-control ${erros.cpf ? 'is-invalid' : ''}`}
                value={dados.cpf}
                onChange={(e) => setCampoMascarado('cpf', e.target.value, 'cpf')}
                placeholder="000.000.000-00"
                maxLength={14}
              />
              {erros.cpf && <div className="invalid-feedback">{erros.cpf}</div>}
            </div>
          </div>
          <div className="mb-3">
            <label className="form-label">Telefone</label>
            <div className="input-group">
              <span className="input-group-text"><i className="fas fa-phone" /></span>
              <input
                type="text"
                className={`form-control ${erros.telefone ? 'is-invalid' : ''}`}
                value={dados.telefone}
                onChange={(e) => setCampoMascarado('telefone', e.target.value, 'telefone')}
                placeholder="(00) 00000-0000"
                maxLength={15}
              />
              {erros.telefone && <div className="invalid-feedback">{erros.telefone}</div>}
            </div>
          </div>
        </>
      )}

      {/* Etapa 2 */}
      {etapa === 1 && (
        <>
          <div className="mb-3">
            <label className="form-label">CEP</label>
            <div className="input-group">
              <span className="input-group-text"><i className="fas fa-map-marker-alt" /></span>
              <input
                type="text"
                className={`form-control ${erros.cep ? 'is-invalid' : ''}`}
                value={dados.cep}
                onChange={(e) => setCampoMascarado('cep', e.target.value, 'cep')}
                placeholder="00000-000"
                maxLength={9}
              />
              {erros.cep && <div className="invalid-feedback">{erros.cep}</div>}
            </div>
          </div>
          <div className="mb-3">
            <label className="form-label">Logradouro</label>
            <input
              type="text"
              className={`form-control ${erros.logradouro ? 'is-invalid' : ''}`}
              value={dados.logradouro}
              onChange={(e) => setCampo('logradouro', e.target.value)}
            />
            {erros.logradouro && <div className="invalid-feedback">{erros.logradouro}</div>}
          </div>
          <div className="row">
            <div className="col-8 mb-3">
              <label className="form-label">Cidade</label>
              <input
                type="text"
                className={`form-control ${erros.cidade ? 'is-invalid' : ''}`}
                value={dados.cidade}
                onChange={(e) => setCampo('cidade', e.target.value)}
              />
              {erros.cidade && <div className="invalid-feedback">{erros.cidade}</div>}
            </div>
            <div className="col-4 mb-3">
              <label className="form-label">UF</label>
              <input
                type="text"
                className={`form-control ${erros.estado ? 'is-invalid' : ''}`}
                value={dados.estado}
                onChange={(e) => setCampo('estado', e.target.value.toUpperCase())}
                maxLength={2}
              />
              {erros.estado && <div className="invalid-feedback">{erros.estado}</div>}
            </div>
          </div>
        </>
      )}

      {/* Etapa 3 */}
      {etapa === 2 && (
        <>
          <div className="text-center mb-4">
            <input
              type="file"
              accept="image/*"
              ref={fileInputRef}
              className="d-none"
              onChange={selecionarImagem}
            />
            <button
              type="button"
              className="btn p-0 border-0"
              onClick={() => fileInputRef.current?.click()}
            >
              {imagemUrl ? (
                <img
                  src={imagemUrl}
                  alt="Preview"
                  className="rounded-circle"
                  style={{ width: 112, height: 112, objectFit: 'cover' }}
                />
              ) : (
                <span
                  className="d-inline-flex flex-column align-items-center justify-content-center rounded-circle bg-primary-subtle"
                  style={{ width: 112, height: 112 }}
                >
                  <i className="fas fa-camera fa-lg text-primary" />
                  <small className="text-muted mt-1">Adicionar foto</small>
                </span>
              )}
            </button>
          </div>

          <div className="card">
            <div className="card-body">
              <h6 className="card-title mb-3">Resumo</h6>
              <table className="table table-sm table-borderless mb-0">
                <tbody>
                  {[
                    ['Nome', dados.nome],
                    ['E-mail', dados.email],
                    ['CPF', dados.cpf],
                    ['Telefone', dados.telefone],
                    ['CEP', dados.cep],
                    ['Endereço', `${dados.logradouro}, ${dados.cidade} - ${dados.estado}`],
                  ].map(([rotulo, valor]) => (
                    <tr key={rotulo}>
                      <th className="text-muted" style={{ width: 100 }}>{rotulo}</th>
                      <td>{valor}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </>
      )}

      {/* Botões */}
      <div className="d-flex justify-content-end gap-2 mt-4">
        {etapa > 0 && (
          <button className="btn btn-outline-secondary" onClick={() => setEtapa(etapa - 1)}>
            Voltar
          </button>
        )}
        <button
          className="btn btn-primary"
          onClick={etapa < 2 ? proximaEtapa : enviar}
          disabled={enviando}
        >
          {enviando && <span className="spinner-border spinner-border-sm me-2" />}
          {etapa === 2 ? 'Enviar' : 'Próximo'}
        </button>
      </div>
    </div>
  );
}
```

---

### Equivalência — Formulários

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Gerenciamento do form | `GlobalKey<FormState>` | `react-hook-form` | Estado local + validação manual |
| Máscaras | `mask_text_input_formatter` | `react-native-mask-input` | Função customizada (sem lib) |
| Validação CPF | Algoritmo manual | Algoritmo manual | Algoritmo manual |
| Multi-step | `Stepper` widget | Estado `etapa` + renderização condicional | Estado `etapa` + renderização condicional |
| Upload de imagem | `image_picker` | `expo-image-picker` | `<input type="file">` nativo |
| Indicador de progresso | `Stepper` built-in | Componente customizado | Badges numerados Bootstrap |
| Feedback de erro | `TextFormField.validator` | `errors` do `useForm` | Classes `is-invalid` + `invalid-feedback` |

---

## 6. Chat em Tempo Real (WebSocket)

Tela de chat com:

- Conexão WebSocket com reconexão automática.
- Lista de mensagens com scroll automático para a última.
- Indicador "digitando…" baseado em eventos do servidor.
- Campo de texto com envio ao pressionar Enter ou botão.

> **Nota:** os exemplos usam `wss://echo.websocket.org` como servidor de demonstração (que ecoa mensagens recebidas). Em produção, substituir pela URL do seu servidor WebSocket real.

### Flutter — web_socket_channel

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  web_socket_channel: ^3.0.0
```

```bash
flutter pub get
```

**2. Modelo de mensagem**

```dart
// lib/features/chat/domain/chat_message.dart
enum MessageAuthor { eu, outro }

class ChatMessage {
  final String id;
  final String texto;
  final MessageAuthor autor;
  final DateTime timestamp;

  ChatMessage({
    required this.texto,
    required this.autor,
    DateTime? timestamp,
  })  : id = '${DateTime.now().microsecondsSinceEpoch}',
        timestamp = timestamp ?? DateTime.now();
}
```

**3. Serviço WebSocket**

```dart
// lib/features/chat/data/chat_service.dart
import 'dart:async';
import 'package:web_socket_channel/web_socket_channel.dart';

class ChatService {
  WebSocketChannel? _channel;
  final _mensagensCtrl = StreamController<String>.broadcast();
  final _statusCtrl = StreamController<bool>.broadcast();
  Timer? _reconnectTimer;

  Stream<String> get mensagens => _mensagensCtrl.stream;
  Stream<bool> get statusConexao => _statusCtrl.stream;

  void conectar(String url) {
    try {
      _channel = WebSocketChannel.connect(Uri.parse(url));
      _statusCtrl.add(true);

      _channel!.stream.listen(
        (data) => _mensagensCtrl.add(data.toString()),
        onError: (_) => _reconectar(url),
        onDone: () => _reconectar(url),
      );
    } catch (_) {
      _reconectar(url);
    }
  }

  void enviar(String mensagem) {
    _channel?.sink.add(mensagem);
  }

  void _reconectar(String url) {
    _statusCtrl.add(false);
    _reconnectTimer?.cancel();
    _reconnectTimer = Timer(const Duration(seconds: 3), () => conectar(url));
  }

  void desconectar() {
    _reconnectTimer?.cancel();
    _channel?.sink.close();
    _mensagensCtrl.close();
    _statusCtrl.close();
  }
}
```

**4. Tela do chat**

```dart
// lib/features/chat/presentation/chat_screen.dart
import 'dart:async';
import 'package:flutter/material.dart';
import '../domain/chat_message.dart';
import '../data/chat_service.dart';

class ChatScreen extends StatefulWidget {
  const ChatScreen({super.key});

  @override
  State<ChatScreen> createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final _service = ChatService();
  final _inputCtrl = TextEditingController();
  final _scrollCtrl = ScrollController();
  final List<ChatMessage> _mensagens = [];
  bool _conectado = false;
  bool _outroDigitando = false;
  Timer? _digitandoTimer;
  StreamSubscription? _msgSub;
  StreamSubscription? _statusSub;

  @override
  void initState() {
    super.initState();
    _service.conectar('wss://echo.websocket.org');

    _statusSub = _service.statusConexao.listen((status) {
      if (mounted) setState(() => _conectado = status);
    });

    _msgSub = _service.mensagens.listen((texto) {
      if (mounted) {
        setState(() {
          _mensagens.add(ChatMessage(
            texto: texto,
            autor: MessageAuthor.outro,
          ));
          _outroDigitando = false;
        });
        _scrollParaFim();
      }
    });
  }

  void _enviarMensagem() {
    final texto = _inputCtrl.text.trim();
    if (texto.isEmpty) return;

    _service.enviar(texto);
    setState(() {
      _mensagens.add(ChatMessage(
        texto: texto,
        autor: MessageAuthor.eu,
      ));
    });
    _inputCtrl.clear();
    _scrollParaFim();

    // Simula indicador "digitando" do outro
    setState(() => _outroDigitando = true);
    _digitandoTimer?.cancel();
    _digitandoTimer = Timer(const Duration(seconds: 2), () {
      if (mounted) setState(() => _outroDigitando = false);
    });
  }

  void _scrollParaFim() {
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (_scrollCtrl.hasClients) {
        _scrollCtrl.animateTo(
          _scrollCtrl.position.maxScrollExtent,
          duration: const Duration(milliseconds: 300),
          curve: Curves.easeOut,
        );
      }
    });
  }

  @override
  void dispose() {
    _msgSub?.cancel();
    _statusSub?.cancel();
    _digitandoTimer?.cancel();
    _service.desconectar();
    _inputCtrl.dispose();
    _scrollCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Chat'),
        actions: [
          Padding(
            padding: const EdgeInsets.only(right: 16),
            child: Icon(
              Icons.circle,
              size: 12,
              color: _conectado ? Colors.green : Colors.red,
            ),
          ),
        ],
      ),
      body: Column(
        children: [
          // Lista de mensagens
          Expanded(
            child: ListView.builder(
              controller: _scrollCtrl,
              padding: const EdgeInsets.all(16),
              itemCount: _mensagens.length,
              itemBuilder: (_, i) {
                final msg = _mensagens[i];
                final euEnviei = msg.autor == MessageAuthor.eu;
                return Align(
                  alignment: euEnviei
                      ? Alignment.centerRight
                      : Alignment.centerLeft,
                  child: Container(
                    margin: const EdgeInsets.only(bottom: 8),
                    padding: const EdgeInsets.symmetric(
                        horizontal: 14, vertical: 10),
                    constraints: BoxConstraints(
                      maxWidth: MediaQuery.sizeOf(context).width * 0.7,
                    ),
                    decoration: BoxDecoration(
                      color: euEnviei
                          ? Theme.of(context).colorScheme.primary
                          : Colors.grey.shade200,
                      borderRadius: BorderRadius.circular(16),
                    ),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.end,
                      children: [
                        Text(
                          msg.texto,
                          style: TextStyle(
                            color: euEnviei ? Colors.white : Colors.black87,
                          ),
                        ),
                        const SizedBox(height: 4),
                        Text(
                          '${msg.timestamp.hour.toString().padLeft(2, '0')}:${msg.timestamp.minute.toString().padLeft(2, '0')}',
                          style: TextStyle(
                            fontSize: 10,
                            color: euEnviei
                                ? Colors.white70
                                : Colors.grey.shade600,
                          ),
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),
          ),

          // Indicador digitando
          if (_outroDigitando)
            Padding(
              padding: const EdgeInsets.only(left: 16, bottom: 4),
              child: Row(
                children: [
                  SizedBox(
                    width: 24,
                    height: 16,
                    child: _TypingDots(),
                  ),
                  const SizedBox(width: 8),
                  Text('digitando…',
                      style: TextStyle(
                          fontSize: 12, color: Colors.grey.shade600)),
                ],
              ),
            ),

          // Campo de texto
          Container(
            padding: const EdgeInsets.all(12),
            decoration: BoxDecoration(
              color: Theme.of(context).colorScheme.surface,
              border: Border(top: BorderSide(color: Colors.grey.shade300)),
            ),
            child: SafeArea(
              child: Row(
                children: [
                  Expanded(
                    child: TextField(
                      controller: _inputCtrl,
                      decoration: InputDecoration(
                        hintText: 'Digite uma mensagem…',
                        border: OutlineInputBorder(
                          borderRadius: BorderRadius.circular(24),
                        ),
                        contentPadding: const EdgeInsets.symmetric(
                            horizontal: 16, vertical: 10),
                      ),
                      textInputAction: TextInputAction.send,
                      onSubmitted: (_) => _enviarMensagem(),
                    ),
                  ),
                  const SizedBox(width: 8),
                  FloatingActionButton.small(
                    onPressed: _enviarMensagem,
                    child: const Icon(Icons.send),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class _TypingDots extends StatefulWidget {
  @override
  State<_TypingDots> createState() => _TypingDotsState();
}

class _TypingDotsState extends State<_TypingDots>
    with SingleTickerProviderStateMixin {
  late AnimationController _ctrl;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(
        vsync: this, duration: const Duration(milliseconds: 1200))
      ..repeat();
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _ctrl,
      builder: (_, __) {
        return Row(
          children: List.generate(3, (i) {
            final delay = i * 0.2;
            final t = ((_ctrl.value - delay) % 1.0).clamp(0.0, 1.0);
            final opacity = (1 - (t - 0.5).abs() * 2).clamp(0.3, 1.0);
            return Container(
              width: 6,
              height: 6,
              margin: const EdgeInsets.symmetric(horizontal: 1),
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: Colors.grey.withAlpha((opacity * 255).toInt()),
              ),
            );
          }),
        );
      },
    );
  }
}
```

---

### React Native — WebSocket API

**1. Tipos**

```typescript
// src/types/chat.ts
export interface ChatMessage {
  id: string;
  texto: string;
  autor: 'eu' | 'outro';
  timestamp: Date;
}
```

**2. Hook WebSocket**

```typescript
// src/hooks/useWebSocket.ts
import { useEffect, useRef, useState, useCallback } from 'react';

export function useWebSocket(url: string) {
  const wsRef = useRef<WebSocket | null>(null);
  const [conectado, setConectado] = useState(false);
  const [ultimaMensagem, setUltimaMensagem] = useState<string | null>(null);
  const reconnectRef = useRef<ReturnType<typeof setTimeout>>();

  const conectar = useCallback(() => {
    const ws = new WebSocket(url);
    wsRef.current = ws;

    ws.onopen = () => setConectado(true);
    ws.onmessage = (e) => setUltimaMensagem(e.data);
    ws.onclose = () => {
      setConectado(false);
      reconnectRef.current = setTimeout(conectar, 3000);
    };
    ws.onerror = () => ws.close();
  }, [url]);

  useEffect(() => {
    conectar();
    return () => {
      clearTimeout(reconnectRef.current);
      wsRef.current?.close();
    };
  }, [conectar]);

  const enviar = useCallback((mensagem: string) => {
    wsRef.current?.send(mensagem);
  }, []);

  return { conectado, ultimaMensagem, enviar };
}
```

**3. Tela do chat**

```tsx
// src/features/chat/ChatScreen.tsx
import React, { useEffect, useRef, useState } from 'react';
import {
  View,
  Text,
  TextInput,
  FlatList,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  StyleSheet,
} from 'react-native';
import { ChatMessage } from '../../types/chat';
import { useWebSocket } from '../../hooks/useWebSocket';

export function ChatScreen() {
  const { conectado, ultimaMensagem, enviar } = useWebSocket(
    'wss://echo.websocket.org',
  );
  const [mensagens, setMensagens] = useState<ChatMessage[]>([]);
  const [texto, setTexto] = useState('');
  const [outroDigitando, setOutroDigitando] = useState(false);
  const flatListRef = useRef<FlatList>(null);
  const digitandoTimerRef = useRef<ReturnType<typeof setTimeout>>();

  useEffect(() => {
    if (ultimaMensagem === null) return;
    setMensagens((prev) => [
      ...prev,
      {
        id: `msg-${Date.now()}-recv`,
        texto: ultimaMensagem,
        autor: 'outro',
        timestamp: new Date(),
      },
    ]);
    setOutroDigitando(false);
  }, [ultimaMensagem]);

  const enviarMensagem = () => {
    const msg = texto.trim();
    if (!msg) return;

    enviar(msg);
    setMensagens((prev) => [
      ...prev,
      { id: `msg-${Date.now()}-sent`, texto: msg, autor: 'eu', timestamp: new Date() },
    ]);
    setTexto('');

    setOutroDigitando(true);
    clearTimeout(digitandoTimerRef.current);
    digitandoTimerRef.current = setTimeout(() => setOutroDigitando(false), 2000);
  };

  const formatarHora = (d: Date) =>
    `${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}`;

  return (
    <KeyboardAvoidingView
      style={styles.container}
      behavior={Platform.OS === 'ios' ? 'padding' : undefined}
      keyboardVerticalOffset={90}
    >
      {/* Status */}
      <View style={styles.statusBar}>
        <View
          style={[styles.dot, { backgroundColor: conectado ? '#4caf50' : '#f44336' }]}
        />
        <Text style={styles.statusTexto}>
          {conectado ? 'Conectado' : 'Reconectando…'}
        </Text>
      </View>

      {/* Mensagens */}
      <FlatList
        ref={flatListRef}
        data={mensagens}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.lista}
        onContentSizeChange={() =>
          flatListRef.current?.scrollToEnd({ animated: true })
        }
        renderItem={({ item }) => {
          const euEnviei = item.autor === 'eu';
          return (
            <View
              style={[
                styles.balao,
                euEnviei ? styles.balaoEu : styles.balaoOutro,
              ]}
            >
              <Text style={euEnviei ? styles.textoEu : styles.textoOutro}>
                {item.texto}
              </Text>
              <Text style={euEnviei ? styles.horaEu : styles.horaOutro}>
                {formatarHora(item.timestamp)}
              </Text>
            </View>
          );
        }}
      />

      {/* Digitando */}
      {outroDigitando && (
        <Text style={styles.digitando}>digitando…</Text>
      )}

      {/* Input */}
      <View style={styles.inputBar}>
        <TextInput
          style={styles.input}
          value={texto}
          onChangeText={setTexto}
          placeholder="Digite uma mensagem…"
          onSubmitEditing={enviarMensagem}
          returnKeyType="send"
        />
        <TouchableOpacity style={styles.btnEnviar} onPress={enviarMensagem}>
          <Text style={styles.btnEnviarTexto}>➤</Text>
        </TouchableOpacity>
      </View>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f5f5f5' },
  statusBar: {
    flexDirection: 'row', alignItems: 'center', paddingHorizontal: 16,
    paddingVertical: 8, backgroundColor: '#fff', borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  dot: { width: 8, height: 8, borderRadius: 4, marginRight: 8 },
  statusTexto: { fontSize: 12, color: '#666' },
  lista: { padding: 16 },
  balao: {
    maxWidth: '75%', marginBottom: 8, paddingHorizontal: 14,
    paddingVertical: 10, borderRadius: 16,
  },
  balaoEu: { alignSelf: 'flex-end', backgroundColor: '#3949ab' },
  balaoOutro: { alignSelf: 'flex-start', backgroundColor: '#e0e0e0' },
  textoEu: { color: '#fff' },
  textoOutro: { color: '#212121' },
  horaEu: { fontSize: 10, color: 'rgba(255,255,255,0.7)', textAlign: 'right', marginTop: 4 },
  horaOutro: { fontSize: 10, color: '#999', textAlign: 'right', marginTop: 4 },
  digitando: { paddingLeft: 16, paddingBottom: 4, fontSize: 12, color: '#999' },
  inputBar: {
    flexDirection: 'row', padding: 12, backgroundColor: '#fff',
    borderTopWidth: 1, borderTopColor: '#e0e0e0',
  },
  input: {
    flex: 1, borderWidth: 1, borderColor: '#ddd', borderRadius: 24,
    paddingHorizontal: 16, paddingVertical: 8, fontSize: 15,
  },
  btnEnviar: {
    width: 44, height: 44, borderRadius: 22, backgroundColor: '#3949ab',
    justifyContent: 'center', alignItems: 'center', marginLeft: 8,
  },
  btnEnviarTexto: { color: '#fff', fontSize: 18 },
});
```

---

### React Web — WebSocket API + Bootstrap

**1. Hook WebSocket** (idêntico ao conceito do React Native)

```typescript
// src/hooks/useWebSocket.ts
import { useEffect, useRef, useState, useCallback } from 'react';

export function useWebSocket(url: string) {
  const wsRef = useRef<WebSocket | null>(null);
  const [conectado, setConectado] = useState(false);
  const [ultimaMensagem, setUltimaMensagem] = useState<string | null>(null);
  const reconnectRef = useRef<ReturnType<typeof setTimeout>>();

  const conectar = useCallback(() => {
    const ws = new WebSocket(url);
    wsRef.current = ws;

    ws.onopen = () => setConectado(true);
    ws.onmessage = (e) => setUltimaMensagem(e.data);
    ws.onclose = () => {
      setConectado(false);
      reconnectRef.current = setTimeout(conectar, 3000);
    };
    ws.onerror = () => ws.close();
  }, [url]);

  useEffect(() => {
    conectar();
    return () => {
      clearTimeout(reconnectRef.current);
      wsRef.current?.close();
    };
  }, [conectar]);

  const enviar = useCallback((msg: string) => {
    wsRef.current?.send(msg);
  }, []);

  return { conectado, ultimaMensagem, enviar };
}
```

**2. Tipos**

```typescript
// src/types/chat.ts
export interface ChatMessage {
  id: string;
  texto: string;
  autor: 'eu' | 'outro';
  timestamp: Date;
}
```

**3. Página do chat**

```tsx
// src/features/chat/ChatPage.tsx
import { useEffect, useRef, useState } from 'react';
import { ChatMessage } from '../../types/chat';
import { useWebSocket } from '../../hooks/useWebSocket';

export function ChatPage() {
  const { conectado, ultimaMensagem, enviar } = useWebSocket(
    'wss://echo.websocket.org',
  );
  const [mensagens, setMensagens] = useState<ChatMessage[]>([]);
  const [texto, setTexto] = useState('');
  const [outroDigitando, setOutroDigitando] = useState(false);
  const listaRef = useRef<HTMLDivElement>(null);
  const digitandoTimerRef = useRef<ReturnType<typeof setTimeout>>();

  useEffect(() => {
    if (ultimaMensagem === null) return;
    setMensagens((prev) => [
      ...prev,
      {
        id: `msg-${Date.now()}-recv`,
        texto: ultimaMensagem,
        autor: 'outro',
        timestamp: new Date(),
      },
    ]);
    setOutroDigitando(false);
  }, [ultimaMensagem]);

  useEffect(() => {
    listaRef.current?.scrollTo({ top: listaRef.current.scrollHeight, behavior: 'smooth' });
  }, [mensagens]);

  const enviarMensagem = () => {
    const msg = texto.trim();
    if (!msg) return;

    enviar(msg);
    setMensagens((prev) => [
      ...prev,
      { id: `msg-${Date.now()}-sent`, texto: msg, autor: 'eu', timestamp: new Date() },
    ]);
    setTexto('');

    setOutroDigitando(true);
    clearTimeout(digitandoTimerRef.current);
    digitandoTimerRef.current = setTimeout(() => setOutroDigitando(false), 2000);
  };

  const formatarHora = (d: Date) =>
    `${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}`;

  return (
    <div
      className="d-flex flex-column mx-auto"
      style={{ maxWidth: 720, height: 'calc(100vh - 56px)' }}
    >
      {/* Status */}
      <div className="d-flex align-items-center gap-2 px-3 py-2 bg-white border-bottom">
        <i
          className="fas fa-circle"
          style={{ fontSize: 8, color: conectado ? '#4caf50' : '#f44336' }}
        />
        <small className="text-muted">
          {conectado ? 'Conectado' : 'Reconectando…'}
        </small>
      </div>

      {/* Mensagens */}
      <div
        ref={listaRef}
        className="flex-grow-1 overflow-auto p-3"
        style={{ backgroundColor: '#f8f9fa' }}
      >
        {mensagens.map((msg) => {
          const euEnviei = msg.autor === 'eu';
          return (
            <div
              key={msg.id}
              className={`d-flex mb-2 ${euEnviei ? 'justify-content-end' : 'justify-content-start'}`}
            >
              <div
                className={`px-3 py-2 rounded-4 ${
                  euEnviei ? 'bg-primary text-white' : 'bg-white border'
                }`}
                style={{ maxWidth: '70%' }}
              >
                <div>{msg.texto}</div>
                <div
                  className={`text-end mt-1 ${euEnviei ? 'text-white-50' : 'text-muted'}`}
                  style={{ fontSize: '0.65rem' }}
                >
                  {formatarHora(msg.timestamp)}
                </div>
              </div>
            </div>
          );
        })}

        {outroDigitando && (
          <div className="text-muted small ps-1">
            <span className="spinner-grow spinner-grow-sm me-1" style={{ width: 6, height: 6 }} />
            <span className="spinner-grow spinner-grow-sm me-1" style={{ width: 6, height: 6, animationDelay: '0.2s' }} />
            <span className="spinner-grow spinner-grow-sm me-2" style={{ width: 6, height: 6, animationDelay: '0.4s' }} />
            digitando…
          </div>
        )}
      </div>

      {/* Input */}
      <div className="d-flex gap-2 p-3 bg-white border-top">
        <input
          type="text"
          className="form-control rounded-pill"
          value={texto}
          onChange={(e) => setTexto(e.target.value)}
          onKeyDown={(e) => e.key === 'Enter' && enviarMensagem()}
          placeholder="Digite uma mensagem…"
        />
        <button
          className="btn btn-primary rounded-circle d-flex align-items-center justify-content-center"
          style={{ width: 44, height: 44, flexShrink: 0 }}
          onClick={enviarMensagem}
        >
          <i className="fas fa-paper-plane" />
        </button>
      </div>
    </div>
  );
}
```

---

### Equivalência — Chat WebSocket

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| WebSocket | `web_socket_channel` | `WebSocket` API nativa | `WebSocket` API nativa |
| Reconexão | `Timer` + reconectar manual | `setTimeout` + reconectar | `setTimeout` + reconectar |
| Lista de mensagens | `ListView.builder` | `FlatList` | `div` com `overflow-auto` |
| Scroll automático | `ScrollController.animateTo` | `scrollToEnd` no `FlatList` | `scrollTo` via `useRef` |
| Indicador digitando | Widget animado customizado | Texto simples | `spinner-grow` Bootstrap |
| Envio com Enter | `TextInputAction.send` + `onSubmitted` | `returnKeyType="send"` + `onSubmitEditing` | `onKeyDown` Enter |
| Bolha de mensagem | `Container` com `BorderRadius` | `View` com `borderRadius` | `div` com `rounded-4` Bootstrap |

---

## 7. Notificações Push

Configuração e uso de notificações push (remotas via FCM) e locais, incluindo permissão do usuário, recebimento em foreground/background e badge de contagem.

### Flutter — firebase_messaging + flutter_local_notifications

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  firebase_core: ^3.6.0
  firebase_messaging: ^15.1.0
  flutter_local_notifications: ^18.0.0
```

```bash
flutter pub get
```

> **Pré-requisito:** projeto Firebase configurado com `flutterfire configure`.

**2. Configuração Android — `android/app/src/main/AndroidManifest.xml`**

```xml
<manifest>
  <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

  <application>
    <!-- Canal padrão para notificações FCM -->
    <meta-data
      android:name="com.google.firebase.messaging.default_notification_channel_id"
      android:value="high_importance_channel" />
  </application>
</manifest>
```

**3. Serviço de notificações**

```dart
// lib/core/notifications/notification_service.dart
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

class NotificationService {
  final _messaging = FirebaseMessaging.instance;
  final _localNotifications = FlutterLocalNotificationsPlugin();
  int _badgeCount = 0;

  int get badgeCount => _badgeCount;

  Future<void> inicializar({
    required void Function(RemoteMessage) onMensagemForeground,
    required void Function(RemoteMessage) onMensagemTap,
  }) async {
    // Solicitar permissão
    final settings = await _messaging.requestPermission(
      alert: true,
      badge: true,
      sound: true,
    );

    if (settings.authorizationStatus != AuthorizationStatus.authorized) return;

    // Canal Android
    const androidChannel = AndroidNotificationChannel(
      'high_importance_channel',
      'Notificações importantes',
      importance: Importance.high,
    );

    await _localNotifications
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(androidChannel);

    // Inicializar notificações locais
    await _localNotifications.initialize(
      const InitializationSettings(
        android: AndroidInitializationSettings('@mipmap/ic_launcher'),
        iOS: DarwinInitializationSettings(),
      ),
      onDidReceiveNotificationResponse: (_) {
        // Tap na notificação local — tratar navegação aqui
      },
    );

    // Token FCM
    final token = await _messaging.getToken();
    // Enviar token ao backend: await api.registrarToken(token);

    // Foreground
    FirebaseMessaging.onMessage.listen((message) {
      onMensagemForeground(message);
      _badgeCount++;

      if (message.notification != null) {
        _localNotifications.show(
          message.hashCode,
          message.notification!.title,
          message.notification!.body,
          NotificationDetails(
            android: AndroidNotificationDetails(
              androidChannel.id,
              androidChannel.name,
              icon: '@mipmap/ic_launcher',
            ),
          ),
        );
      }
    });

    // Tap na notificação (app em background)
    FirebaseMessaging.onMessageOpenedApp.listen(onMensagemTap);

    // App aberto via notificação (app terminado)
    final initialMessage = await _messaging.getInitialMessage();
    if (initialMessage != null) onMensagemTap(initialMessage);
  }

  void limparBadge() => _badgeCount = 0;

  Future<void> enviarNotificacaoLocal({
    required String titulo,
    required String corpo,
  }) async {
    await _localNotifications.show(
      DateTime.now().millisecondsSinceEpoch ~/ 1000,
      titulo,
      corpo,
      const NotificationDetails(
        android: AndroidNotificationDetails(
          'high_importance_channel',
          'Notificações importantes',
        ),
      ),
    );
  }
}
```

**4. Uso no app**

```dart
// lib/main.dart (trecho relevante)
import 'package:firebase_core/firebase_core.dart';
import 'core/notifications/notification_service.dart';

// Handler de background — precisa ser top-level
@pragma('vm:entry-point')
Future<void> _firebaseBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  // Processar dados em background se necessário
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  FirebaseMessaging.onBackgroundMessage(_firebaseBackgroundHandler);

  final notificationService = NotificationService();
  await notificationService.inicializar(
    onMensagemForeground: (msg) {
      // Atualizar UI se necessário
    },
    onMensagemTap: (msg) {
      // Navegar para tela específica com base em msg.data
    },
  );

  runApp(const ProviderScope(child: App()));
}
```

**5. Widget de badge**

```dart
// lib/core/notifications/notification_badge.dart
import 'package:flutter/material.dart';

class NotificationBadge extends StatelessWidget {
  final int count;
  final Widget child;

  const NotificationBadge({super.key, required this.count, required this.child});

  @override
  Widget build(BuildContext context) {
    return Badge(
      isLabelVisible: count > 0,
      label: Text(count > 99 ? '99+' : '$count'),
      child: child,
    );
  }
}
```

---

### React Native — expo-notifications

**1. Dependências**

```bash
npx expo install expo-notifications expo-device expo-constants
```

**2. Serviço de notificações**

```typescript
// src/core/notifications/notificationService.ts
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import Constants from 'expo-constants';
import { Platform } from 'react-native';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

export async function registrarPush(): Promise<string | null> {
  if (!Device.isDevice) {
    console.warn('Push não funciona em emuladores.');
    return null;
  }

  const { status: existente } = await Notifications.getPermissionsAsync();
  let statusFinal = existente;

  if (existente !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    statusFinal = status;
  }

  if (statusFinal !== 'granted') return null;

  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'Padrão',
      importance: Notifications.AndroidImportance.HIGH,
      vibrationPattern: [0, 250, 250, 250],
    });
  }

  const projectId = Constants.expoConfig?.extra?.eas?.projectId;
  const token = await Notifications.getExpoPushTokenAsync({ projectId });
  // Enviar token.data ao backend
  return token.data;
}

export async function enviarNotificacaoLocal(titulo: string, corpo: string) {
  await Notifications.scheduleNotificationAsync({
    content: { title: titulo, body: corpo, sound: true },
    trigger: null,
  });
}
```

**3. Hook de notificações**

```typescript
// src/hooks/useNotifications.ts
import { useEffect, useRef, useState } from 'react';
import * as Notifications from 'expo-notifications';
import { registrarPush } from '../core/notifications/notificationService';

export function useNotifications() {
  const [badgeCount, setBadgeCount] = useState(0);
  const notificationListener = useRef<Notifications.EventSubscription>();
  const responseListener = useRef<Notifications.EventSubscription>();

  useEffect(() => {
    registrarPush();

    notificationListener.current =
      Notifications.addNotificationReceivedListener(() => {
        setBadgeCount((c) => c + 1);
      });

    responseListener.current =
      Notifications.addNotificationResponseReceivedListener((response) => {
        const data = response.notification.request.content.data;
        // Navegar com base em data
      });

    return () => {
      notificationListener.current?.remove();
      responseListener.current?.remove();
    };
  }, []);

  const limparBadge = () => {
    setBadgeCount(0);
    Notifications.setBadgeCountAsync(0);
  };

  return { badgeCount, limparBadge };
}
```

---

### React Web — Notification API + Service Worker

**1. Serviço de notificações**

```typescript
// src/core/notifications/notificationService.ts
export async function solicitarPermissao(): Promise<boolean> {
  if (!('Notification' in window)) return false;
  if (Notification.permission === 'granted') return true;
  if (Notification.permission === 'denied') return false;

  const result = await Notification.requestPermission();
  return result === 'granted';
}

export function enviarNotificacaoLocal(titulo: string, corpo: string) {
  if (Notification.permission !== 'granted') return;

  new Notification(titulo, {
    body: corpo,
    icon: '/favicon.ico',
  });
}

export async function registrarServiceWorker(): Promise<ServiceWorkerRegistration | null> {
  if (!('serviceWorker' in navigator)) return null;

  try {
    return await navigator.serviceWorker.register('/sw.js');
  } catch {
    return null;
  }
}

export async function inscreverPush(
  registration: ServiceWorkerRegistration,
  vapidPublicKey: string,
): Promise<PushSubscription | null> {
  try {
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(vapidPublicKey),
    });
    // Enviar subscription ao backend via fetch POST
    return subscription;
  } catch {
    return null;
  }
}

function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - (base64String.length % 4)) % 4);
  const base64 = (base64String + padding).replace(/-/g, '+').replace(/_/g, '/');
  const rawData = window.atob(base64);
  return Uint8Array.from(rawData, (char) => char.charCodeAt(0));
}
```

**2. Service Worker — `public/sw.js`**

```javascript
// public/sw.js
self.addEventListener('push', (event) => {
  const data = event.data?.json() ?? {};
  const titulo = data.title || 'Nova notificação';
  const options = {
    body: data.body || '',
    icon: '/favicon.ico',
    badge: '/favicon.ico',
    data: data.url || '/',
  };

  event.waitUntil(self.registration.showNotification(titulo, options));
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  event.waitUntil(clients.openWindow(event.notification.data));
});
```

**3. Hook de notificações**

```typescript
// src/hooks/useNotifications.ts
import { useEffect, useState, useCallback } from 'react';
import { solicitarPermissao, registrarServiceWorker } from '../core/notifications/notificationService';

export function useNotifications() {
  const [permitido, setPermitido] = useState(false);
  const [badgeCount, setBadgeCount] = useState(0);

  useEffect(() => {
    (async () => {
      const ok = await solicitarPermissao();
      setPermitido(ok);
      if (ok) await registrarServiceWorker();
    })();
  }, []);

  const incrementarBadge = useCallback(() => {
    setBadgeCount((c) => c + 1);
    if ('setAppBadge' in navigator) {
      (navigator as Navigator & { setAppBadge: (n: number) => void })
        .setAppBadge(badgeCount + 1);
    }
  }, [badgeCount]);

  const limparBadge = useCallback(() => {
    setBadgeCount(0);
    if ('clearAppBadge' in navigator) {
      (navigator as Navigator & { clearAppBadge: () => void }).clearAppBadge();
    }
  }, []);

  return { permitido, badgeCount, incrementarBadge, limparBadge };
}
```

**4. Componente de badge**

```tsx
// src/components/NotificationBadge.tsx
interface Props {
  count: number;
  children: React.ReactNode;
}

export function NotificationBadge({ count, children }: Props) {
  return (
    <span className="position-relative d-inline-block">
      {children}
      {count > 0 && (
        <span
          className="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger"
          style={{ fontSize: '0.6rem' }}
        >
          {count > 99 ? '99+' : count}
        </span>
      )}
    </span>
  );
}
```

---

### Equivalência — Notificações Push

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Push remoto | Firebase Cloud Messaging | Expo Push Notifications | Web Push API + Service Worker |
| Notificação local | `flutter_local_notifications` | `expo-notifications` | `new Notification()` |
| Permissão | `FirebaseMessaging.requestPermission` | `Notifications.requestPermissionsAsync` | `Notification.requestPermission` |
| Token/registro | `getToken()` → backend | `getExpoPushTokenAsync` → backend | `pushManager.subscribe` → backend |
| Foreground | `FirebaseMessaging.onMessage` | `addNotificationReceivedListener` | Service Worker `push` event |
| Tap na notificação | `onMessageOpenedApp` | `addNotificationResponseReceivedListener` | `notificationclick` event |
| Badge | Widget customizado `Badge` | `setBadgeCountAsync` + estado | `navigator.setAppBadge` + Bootstrap badge |
| Canal Android | `AndroidNotificationChannel` | `setNotificationChannelAsync` | N/A (Web) |

---

## 8. CRUD Completo com Tabela de Dados

Tela de gerenciamento com:

- Tabela/listagem com busca por texto, filtro por status e ordenação por coluna.
- Paginação server-side.
- Formulário de criação/edição em modal/dialog.
- Confirmação de exclusão.

### Flutter — DataTable + AlertDialog

**1. Modelo e serviço**

```dart
// lib/features/crud/domain/produto.dart
class Produto {
  final String id;
  String nome;
  String categoria;
  double preco;
  bool ativo;

  Produto({
    required this.id,
    required this.nome,
    required this.categoria,
    required this.preco,
    this.ativo = true,
  });
}
```

```dart
// lib/features/crud/data/produto_service.dart
import '../domain/produto.dart';

class ProdutoService {
  final List<Produto> _dados = List.generate(
    55,
    (i) => Produto(
      id: '${i + 1}',
      nome: 'Produto ${i + 1}',
      categoria: i % 3 == 0 ? 'Eletrônicos' : i % 3 == 1 ? 'Roupas' : 'Alimentos',
      preco: (i + 1) * 9.90,
      ativo: i % 5 != 0,
    ),
  );

  Future<({List<Produto> itens, int total})> listar({
    required int pagina,
    int porPagina = 10,
    String? busca,
    String? filtroCategoria,
    String? ordenarPor,
    bool ascendente = true,
  }) async {
    await Future.delayed(const Duration(milliseconds: 300));

    var resultado = [..._dados];

    if (busca != null && busca.isNotEmpty) {
      final termo = busca.toLowerCase();
      resultado = resultado.where((p) => p.nome.toLowerCase().contains(termo)).toList();
    }

    if (filtroCategoria != null && filtroCategoria.isNotEmpty) {
      resultado = resultado.where((p) => p.categoria == filtroCategoria).toList();
    }

    if (ordenarPor != null) {
      resultado.sort((a, b) {
        int cmp;
        switch (ordenarPor) {
          case 'nome': cmp = a.nome.compareTo(b.nome);
          case 'preco': cmp = a.preco.compareTo(b.preco);
          case 'categoria': cmp = a.categoria.compareTo(b.categoria);
          default: cmp = 0;
        }
        return ascendente ? cmp : -cmp;
      });
    }

    final total = resultado.length;
    final inicio = (pagina - 1) * porPagina;
    final fim = (inicio + porPagina).clamp(0, total);

    return (itens: resultado.sublist(inicio, fim), total: total);
  }

  Future<void> salvar(Produto produto) async {
    await Future.delayed(const Duration(milliseconds: 200));
    final index = _dados.indexWhere((p) => p.id == produto.id);
    if (index >= 0) {
      _dados[index] = produto;
    } else {
      _dados.add(produto);
    }
  }

  Future<void> excluir(String id) async {
    await Future.delayed(const Duration(milliseconds: 200));
    _dados.removeWhere((p) => p.id == id);
  }
}
```

**2. Tela CRUD**

```dart
// lib/features/crud/presentation/crud_screen.dart
import 'package:flutter/material.dart';
import '../domain/produto.dart';
import '../data/produto_service.dart';

class CrudScreen extends StatefulWidget {
  const CrudScreen({super.key});

  @override
  State<CrudScreen> createState() => _CrudScreenState();
}

class _CrudScreenState extends State<CrudScreen> {
  final _service = ProdutoService();
  final _buscaCtrl = TextEditingController();

  List<Produto> _itens = [];
  int _total = 0;
  int _pagina = 1;
  final int _porPagina = 10;
  String? _filtroCategoria;
  String _ordenarPor = 'nome';
  bool _ascendente = true;
  bool _carregando = false;

  @override
  void initState() {
    super.initState();
    _carregar();
  }

  Future<void> _carregar() async {
    setState(() => _carregando = true);
    final resultado = await _service.listar(
      pagina: _pagina,
      porPagina: _porPagina,
      busca: _buscaCtrl.text,
      filtroCategoria: _filtroCategoria,
      ordenarPor: _ordenarPor,
      ascendente: _ascendente,
    );
    setState(() {
      _itens = resultado.itens;
      _total = resultado.total;
      _carregando = false;
    });
  }

  void _ordenar(String coluna) {
    setState(() {
      if (_ordenarPor == coluna) {
        _ascendente = !_ascendente;
      } else {
        _ordenarPor = coluna;
        _ascendente = true;
      }
      _pagina = 1;
    });
    _carregar();
  }

  void _abrirFormulario([Produto? produto]) {
    final nomeCtrl = TextEditingController(text: produto?.nome ?? '');
    final categoriaCtrl = TextEditingController(text: produto?.categoria ?? '');
    final precoCtrl = TextEditingController(
        text: produto != null ? produto.preco.toStringAsFixed(2) : '');
    bool ativo = produto?.ativo ?? true;

    showDialog(
      context: context,
      builder: (ctx) => StatefulBuilder(
        builder: (ctx, setDialogState) => AlertDialog(
          title: Text(produto == null ? 'Novo Produto' : 'Editar Produto'),
          content: SingleChildScrollView(
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                TextField(
                  controller: nomeCtrl,
                  decoration: const InputDecoration(labelText: 'Nome'),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller: categoriaCtrl,
                  decoration: const InputDecoration(labelText: 'Categoria'),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller: precoCtrl,
                  decoration: const InputDecoration(labelText: 'Preço'),
                  keyboardType: TextInputType.number,
                ),
                const SizedBox(height: 12),
                SwitchListTile(
                  title: const Text('Ativo'),
                  value: ativo,
                  onChanged: (v) => setDialogState(() => ativo = v),
                  contentPadding: EdgeInsets.zero,
                ),
              ],
            ),
          ),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(ctx),
              child: const Text('Cancelar'),
            ),
            FilledButton(
              onPressed: () async {
                final p = Produto(
                  id: produto?.id ?? DateTime.now().millisecondsSinceEpoch.toString(),
                  nome: nomeCtrl.text,
                  categoria: categoriaCtrl.text,
                  preco: double.tryParse(precoCtrl.text) ?? 0,
                  ativo: ativo,
                );
                await _service.salvar(p);
                if (ctx.mounted) Navigator.pop(ctx);
                _carregar();
              },
              child: const Text('Salvar'),
            ),
          ],
        ),
      ),
    );
  }

  void _confirmarExclusao(Produto produto) {
    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('Excluir produto'),
        content: Text('Deseja excluir "${produto.nome}"?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: const Text('Cancelar'),
          ),
          FilledButton(
            style: FilledButton.styleFrom(backgroundColor: Colors.red),
            onPressed: () async {
              await _service.excluir(produto.id);
              if (ctx.mounted) Navigator.pop(ctx);
              _carregar();
            },
            child: const Text('Excluir'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    final totalPaginas = (_total / _porPagina).ceil();

    return Scaffold(
      appBar: AppBar(title: const Text('Produtos')),
      body: Column(
        children: [
          // Barra de busca e filtro
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _buscaCtrl,
                    decoration: InputDecoration(
                      hintText: 'Buscar por nome…',
                      prefixIcon: const Icon(Icons.search),
                      suffixIcon: IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _buscaCtrl.clear();
                          _pagina = 1;
                          _carregar();
                        },
                      ),
                      border: const OutlineInputBorder(),
                      isDense: true,
                    ),
                    onSubmitted: (_) {
                      _pagina = 1;
                      _carregar();
                    },
                  ),
                ),
                const SizedBox(width: 12),
                DropdownButton<String>(
                  hint: const Text('Categoria'),
                  value: _filtroCategoria,
                  items: [
                    const DropdownMenuItem(value: '', child: Text('Todas')),
                    ...['Eletrônicos', 'Roupas', 'Alimentos'].map(
                      (c) => DropdownMenuItem(value: c, child: Text(c)),
                    ),
                  ],
                  onChanged: (v) {
                    _filtroCategoria = v == '' ? null : v;
                    _pagina = 1;
                    _carregar();
                  },
                ),
              ],
            ),
          ),

          // Tabela
          Expanded(
            child: _carregando
                ? const Center(child: CircularProgressIndicator())
                : SingleChildScrollView(
                    scrollDirection: Axis.horizontal,
                    child: DataTable(
                      sortColumnIndex: ['nome', 'categoria', 'preco']
                          .indexOf(_ordenarPor),
                      sortAscending: _ascendente,
                      columns: [
                        DataColumn(
                          label: const Text('Nome'),
                          onSort: (_, __) => _ordenar('nome'),
                        ),
                        DataColumn(
                          label: const Text('Categoria'),
                          onSort: (_, __) => _ordenar('categoria'),
                        ),
                        DataColumn(
                          label: const Text('Preço'),
                          numeric: true,
                          onSort: (_, __) => _ordenar('preco'),
                        ),
                        const DataColumn(label: Text('Status')),
                        const DataColumn(label: Text('Ações')),
                      ],
                      rows: _itens.map((p) {
                        return DataRow(cells: [
                          DataCell(Text(p.nome)),
                          DataCell(Text(p.categoria)),
                          DataCell(Text('R\$ ${p.preco.toStringAsFixed(2)}')),
                          DataCell(Chip(
                            label: Text(p.ativo ? 'Ativo' : 'Inativo',
                                style: const TextStyle(fontSize: 11)),
                            backgroundColor:
                                p.ativo ? Colors.green.shade50 : Colors.red.shade50,
                            visualDensity: VisualDensity.compact,
                          )),
                          DataCell(Row(
                            mainAxisSize: MainAxisSize.min,
                            children: [
                              IconButton(
                                icon: const Icon(Icons.edit, size: 20),
                                onPressed: () => _abrirFormulario(p),
                                tooltip: 'Editar',
                              ),
                              IconButton(
                                icon: const Icon(Icons.delete, size: 20,
                                    color: Colors.red),
                                onPressed: () => _confirmarExclusao(p),
                                tooltip: 'Excluir',
                              ),
                            ],
                          )),
                        ]);
                      }).toList(),
                    ),
                  ),
          ),

          // Paginação
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text('$_total registro(s)',
                    style: Theme.of(context).textTheme.bodySmall),
                Row(
                  children: [
                    IconButton(
                      icon: const Icon(Icons.chevron_left),
                      onPressed: _pagina > 1
                          ? () {
                              _pagina--;
                              _carregar();
                            }
                          : null,
                    ),
                    Text('$_pagina / $totalPaginas'),
                    IconButton(
                      icon: const Icon(Icons.chevron_right),
                      onPressed: _pagina < totalPaginas
                          ? () {
                              _pagina++;
                              _carregar();
                            }
                          : null,
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _abrirFormulario(),
        child: const Icon(Icons.add),
      ),
    );
  }

  @override
  void dispose() {
    _buscaCtrl.dispose();
    super.dispose();
  }
}
```

---

### React Native — FlatList + Modal

**1. Tipos e serviço**

```typescript
// src/types/produto.ts
export interface Produto {
  id: string;
  nome: string;
  categoria: string;
  preco: number;
  ativo: boolean;
}

export interface ListagemResult {
  itens: Produto[];
  total: number;
}
```

```typescript
// src/services/produtoService.ts
import { Produto, ListagemResult } from '../types/produto';

const DADOS: Produto[] = Array.from({ length: 55 }, (_, i) => ({
  id: String(i + 1),
  nome: `Produto ${i + 1}`,
  categoria: i % 3 === 0 ? 'Eletrônicos' : i % 3 === 1 ? 'Roupas' : 'Alimentos',
  preco: (i + 1) * 9.9,
  ativo: i % 5 !== 0,
}));

let dados = [...DADOS];

export async function listarProdutos(params: {
  pagina: number;
  porPagina?: number;
  busca?: string;
  categoria?: string;
  ordenarPor?: string;
  ascendente?: boolean;
}): Promise<ListagemResult> {
  await new Promise((r) => setTimeout(r, 300));

  let resultado = [...dados];

  if (params.busca) {
    const termo = params.busca.toLowerCase();
    resultado = resultado.filter((p) => p.nome.toLowerCase().includes(termo));
  }

  if (params.categoria) {
    resultado = resultado.filter((p) => p.categoria === params.categoria);
  }

  if (params.ordenarPor) {
    resultado.sort((a, b) => {
      const va = a[params.ordenarPor as keyof Produto];
      const vb = b[params.ordenarPor as keyof Produto];
      const cmp = va < vb ? -1 : va > vb ? 1 : 0;
      return params.ascendente !== false ? cmp : -cmp;
    });
  }

  const porPagina = params.porPagina ?? 10;
  const inicio = (params.pagina - 1) * porPagina;

  return {
    itens: resultado.slice(inicio, inicio + porPagina),
    total: resultado.length,
  };
}

export async function salvarProduto(produto: Produto) {
  await new Promise((r) => setTimeout(r, 200));
  const idx = dados.findIndex((p) => p.id === produto.id);
  if (idx >= 0) dados[idx] = produto;
  else dados.push(produto);
}

export async function excluirProduto(id: string) {
  await new Promise((r) => setTimeout(r, 200));
  dados = dados.filter((p) => p.id !== id);
}
```

**2. Tela CRUD**

```tsx
// src/features/crud/CrudScreen.tsx
import React, { useEffect, useState, useCallback } from 'react';
import {
  View, Text, TextInput, FlatList, TouchableOpacity,
  Modal, StyleSheet, Alert, ActivityIndicator,
} from 'react-native';
import { Produto } from '../../types/produto';
import { listarProdutos, salvarProduto, excluirProduto } from '../../services/produtoService';

export function CrudScreen() {
  const [itens, setItens] = useState<Produto[]>([]);
  const [total, setTotal] = useState(0);
  const [pagina, setPagina] = useState(1);
  const [busca, setBusca] = useState('');
  const [carregando, setCarregando] = useState(false);
  const [modalVisivel, setModalVisivel] = useState(false);
  const [produtoEdit, setProdutoEdit] = useState<Produto | null>(null);
  const [formNome, setFormNome] = useState('');
  const [formCategoria, setFormCategoria] = useState('');
  const [formPreco, setFormPreco] = useState('');

  const carregar = useCallback(async (pag = pagina) => {
    setCarregando(true);
    const res = await listarProdutos({ pagina: pag, busca });
    setItens(res.itens);
    setTotal(res.total);
    setCarregando(false);
  }, [pagina, busca]);

  useEffect(() => { carregar(); }, [carregar]);

  const abrirModal = (produto?: Produto) => {
    setProdutoEdit(produto ?? null);
    setFormNome(produto?.nome ?? '');
    setFormCategoria(produto?.categoria ?? '');
    setFormPreco(produto ? produto.preco.toFixed(2) : '');
    setModalVisivel(true);
  };

  const salvar = async () => {
    await salvarProduto({
      id: produtoEdit?.id ?? String(Date.now()),
      nome: formNome,
      categoria: formCategoria,
      preco: parseFloat(formPreco) || 0,
      ativo: produtoEdit?.ativo ?? true,
    });
    setModalVisivel(false);
    carregar();
  };

  const confirmarExclusao = (produto: Produto) => {
    Alert.alert('Excluir', `Deseja excluir "${produto.nome}"?`, [
      { text: 'Cancelar', style: 'cancel' },
      {
        text: 'Excluir',
        style: 'destructive',
        onPress: async () => {
          await excluirProduto(produto.id);
          carregar();
        },
      },
    ]);
  };

  const totalPaginas = Math.ceil(total / 10);

  return (
    <View style={styles.container}>
      {/* Busca */}
      <View style={styles.buscaRow}>
        <TextInput
          style={styles.buscaInput}
          placeholder="Buscar por nome…"
          value={busca}
          onChangeText={setBusca}
          onSubmitEditing={() => { setPagina(1); carregar(1); }}
          returnKeyType="search"
        />
        <TouchableOpacity style={styles.btnNovo} onPress={() => abrirModal()}>
          <Text style={styles.btnNovoTexto}>+ Novo</Text>
        </TouchableOpacity>
      </View>

      {/* Lista */}
      {carregando ? (
        <ActivityIndicator size="large" style={{ marginTop: 40 }} />
      ) : (
        <FlatList
          data={itens}
          keyExtractor={(item) => item.id}
          renderItem={({ item }) => (
            <View style={styles.card}>
              <View style={styles.cardInfo}>
                <Text style={styles.cardNome}>{item.nome}</Text>
                <Text style={styles.cardDetalhe}>
                  {item.categoria} • R$ {item.preco.toFixed(2)}
                </Text>
              </View>
              <View style={styles.cardAcoes}>
                <TouchableOpacity onPress={() => abrirModal(item)}>
                  <Text style={styles.btnEditar}>✏️</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={() => confirmarExclusao(item)}>
                  <Text style={styles.btnExcluir}>🗑️</Text>
                </TouchableOpacity>
              </View>
            </View>
          )}
        />
      )}

      {/* Paginação */}
      <View style={styles.paginacao}>
        <Text style={styles.totalTexto}>{total} registro(s)</Text>
        <View style={styles.paginacaoBtns}>
          <TouchableOpacity
            onPress={() => { setPagina(pagina - 1); carregar(pagina - 1); }}
            disabled={pagina <= 1}
          >
            <Text style={[styles.pagBtn, pagina <= 1 && styles.pagBtnDisabled]}>◀</Text>
          </TouchableOpacity>
          <Text style={styles.pagTexto}>{pagina} / {totalPaginas}</Text>
          <TouchableOpacity
            onPress={() => { setPagina(pagina + 1); carregar(pagina + 1); }}
            disabled={pagina >= totalPaginas}
          >
            <Text style={[styles.pagBtn, pagina >= totalPaginas && styles.pagBtnDisabled]}>▶</Text>
          </TouchableOpacity>
        </View>
      </View>

      {/* Modal */}
      <Modal visible={modalVisivel} animationType="slide" transparent>
        <View style={styles.modalOverlay}>
          <View style={styles.modalContent}>
            <Text style={styles.modalTitulo}>
              {produtoEdit ? 'Editar Produto' : 'Novo Produto'}
            </Text>
            <TextInput style={styles.modalInput} placeholder="Nome" value={formNome} onChangeText={setFormNome} />
            <TextInput style={styles.modalInput} placeholder="Categoria" value={formCategoria} onChangeText={setFormCategoria} />
            <TextInput style={styles.modalInput} placeholder="Preço" value={formPreco} onChangeText={setFormPreco} keyboardType="numeric" />
            <View style={styles.modalBtns}>
              <TouchableOpacity style={styles.btnCancelar} onPress={() => setModalVisivel(false)}>
                <Text>Cancelar</Text>
              </TouchableOpacity>
              <TouchableOpacity style={styles.btnSalvar} onPress={salvar}>
                <Text style={styles.btnSalvarTexto}>Salvar</Text>
              </TouchableOpacity>
            </View>
          </View>
        </View>
      </Modal>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f9f9f9' },
  buscaRow: { flexDirection: 'row', padding: 16, gap: 12 },
  buscaInput: { flex: 1, borderWidth: 1, borderColor: '#ddd', borderRadius: 8, paddingHorizontal: 12, paddingVertical: 8, backgroundColor: '#fff' },
  btnNovo: { backgroundColor: '#3949ab', paddingHorizontal: 16, paddingVertical: 10, borderRadius: 8, justifyContent: 'center' },
  btnNovoTexto: { color: '#fff', fontWeight: '600' },
  card: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', backgroundColor: '#fff', marginHorizontal: 16, marginBottom: 8, padding: 14, borderRadius: 10, elevation: 1 },
  cardInfo: { flex: 1 },
  cardNome: { fontSize: 15, fontWeight: '600' },
  cardDetalhe: { fontSize: 13, color: '#666', marginTop: 4 },
  cardAcoes: { flexDirection: 'row', gap: 12 },
  btnEditar: { fontSize: 18 },
  btnExcluir: { fontSize: 18 },
  paginacao: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', paddingHorizontal: 16, paddingVertical: 12 },
  totalTexto: { fontSize: 12, color: '#999' },
  paginacaoBtns: { flexDirection: 'row', alignItems: 'center', gap: 12 },
  pagBtn: { fontSize: 18, color: '#3949ab' },
  pagBtnDisabled: { color: '#ccc' },
  pagTexto: { fontSize: 14 },
  modalOverlay: { flex: 1, backgroundColor: 'rgba(0,0,0,0.4)', justifyContent: 'center', paddingHorizontal: 24 },
  modalContent: { backgroundColor: '#fff', borderRadius: 16, padding: 24 },
  modalTitulo: { fontSize: 18, fontWeight: '700', marginBottom: 16 },
  modalInput: { borderWidth: 1, borderColor: '#ddd', borderRadius: 8, paddingHorizontal: 12, paddingVertical: 10, marginBottom: 12 },
  modalBtns: { flexDirection: 'row', justifyContent: 'flex-end', gap: 12, marginTop: 8 },
  btnCancelar: { paddingHorizontal: 16, paddingVertical: 10, borderRadius: 8, borderWidth: 1, borderColor: '#ccc' },
  btnSalvar: { paddingHorizontal: 16, paddingVertical: 10, borderRadius: 8, backgroundColor: '#3949ab' },
  btnSalvarTexto: { color: '#fff', fontWeight: '600' },
});
```

---

### React Web — Bootstrap Table + react-bootstrap Modal

**1. Dependências adicionais**

```bash
# react-bootstrap já instalado (seção 4) — Modal é componente dinâmico
```

**2. Serviço** (mesmo `produtoService.ts` da versão React Native, com export ES module)

**3. Página CRUD**

```tsx
// src/features/crud/CrudPage.tsx
import { useEffect, useState, useCallback } from 'react';
import Modal from 'react-bootstrap/Modal';
import { Produto } from '../../types/produto';
import { listarProdutos, salvarProduto, excluirProduto } from '../../services/produtoService';

export function CrudPage() {
  const [itens, setItens] = useState<Produto[]>([]);
  const [total, setTotal] = useState(0);
  const [pagina, setPagina] = useState(1);
  const porPagina = 10;
  const [busca, setBusca] = useState('');
  const [categoria, setCategoria] = useState('');
  const [ordenarPor, setOrdenarPor] = useState('nome');
  const [ascendente, setAscendente] = useState(true);
  const [carregando, setCarregando] = useState(false);

  // Modal
  const [modalAberto, setModalAberto] = useState(false);
  const [editando, setEditando] = useState<Produto | null>(null);
  const [formNome, setFormNome] = useState('');
  const [formCategoria, setFormCategoria] = useState('');
  const [formPreco, setFormPreco] = useState('');

  // Confirmação
  const [excluindo, setExcluindo] = useState<Produto | null>(null);

  const carregar = useCallback(async () => {
    setCarregando(true);
    const res = await listarProdutos({
      pagina, busca, categoria, ordenarPor, ascendente,
    });
    setItens(res.itens);
    setTotal(res.total);
    setCarregando(false);
  }, [pagina, busca, categoria, ordenarPor, ascendente]);

  useEffect(() => { carregar(); }, [carregar]);

  const ordenar = (col: string) => {
    if (ordenarPor === col) setAscendente(!ascendente);
    else { setOrdenarPor(col); setAscendente(true); }
    setPagina(1);
  };

  const iconeOrdem = (col: string) => {
    if (ordenarPor !== col) return 'fas fa-sort text-muted';
    return ascendente ? 'fas fa-sort-up text-primary' : 'fas fa-sort-down text-primary';
  };

  const abrirModal = (produto?: Produto) => {
    setEditando(produto ?? null);
    setFormNome(produto?.nome ?? '');
    setFormCategoria(produto?.categoria ?? '');
    setFormPreco(produto ? produto.preco.toFixed(2) : '');
    setModalAberto(true);
  };

  const salvar = async () => {
    await salvarProduto({
      id: editando?.id ?? String(Date.now()),
      nome: formNome,
      categoria: formCategoria,
      preco: parseFloat(formPreco) || 0,
      ativo: editando?.ativo ?? true,
    });
    setModalAberto(false);
    carregar();
  };

  const confirmarExclusao = async () => {
    if (!excluindo) return;
    await excluirProduto(excluindo.id);
    setExcluindo(null);
    carregar();
  };

  const totalPaginas = Math.ceil(total / porPagina);

  return (
    <div className="container-fluid p-4">
      {/* Barra de busca/filtro */}
      <div className="row g-3 mb-4">
        <div className="col-md-5">
          <div className="input-group">
            <span className="input-group-text"><i className="fas fa-search" /></span>
            <input
              type="text"
              className="form-control"
              placeholder="Buscar por nome…"
              value={busca}
              onChange={(e) => { setBusca(e.target.value); setPagina(1); }}
            />
          </div>
        </div>
        <div className="col-md-3">
          <select
            className="form-select"
            value={categoria}
            onChange={(e) => { setCategoria(e.target.value); setPagina(1); }}
          >
            <option value="">Todas as categorias</option>
            <option>Eletrônicos</option>
            <option>Roupas</option>
            <option>Alimentos</option>
          </select>
        </div>
        <div className="col-md-4 text-end">
          <button className="btn btn-primary" onClick={() => abrirModal()}>
            <i className="fas fa-plus me-1" /> Novo Produto
          </button>
        </div>
      </div>

      {/* Tabela */}
      <div className="table-responsive">
        <table className="table table-hover align-middle">
          <thead className="table-light">
            <tr>
              {[
                { col: 'nome', label: 'Nome' },
                { col: 'categoria', label: 'Categoria' },
                { col: 'preco', label: 'Preço' },
              ].map(({ col, label }) => (
                <th
                  key={col}
                  role="button"
                  onClick={() => ordenar(col)}
                  className="user-select-none"
                >
                  {label} <i className={iconeOrdem(col)} />
                </th>
              ))}
              <th>Status</th>
              <th style={{ width: 120 }}>Ações</th>
            </tr>
          </thead>
          <tbody>
            {carregando ? (
              <tr>
                <td colSpan={5} className="text-center py-5">
                  <span className="spinner-border" />
                </td>
              </tr>
            ) : itens.length === 0 ? (
              <tr>
                <td colSpan={5} className="text-center text-muted py-4">
                  Nenhum produto encontrado.
                </td>
              </tr>
            ) : (
              itens.map((p) => (
                <tr key={p.id}>
                  <td>{p.nome}</td>
                  <td>{p.categoria}</td>
                  <td>R$ {p.preco.toFixed(2)}</td>
                  <td>
                    <span className={`badge ${p.ativo ? 'bg-success-subtle text-success-emphasis' : 'bg-danger-subtle text-danger-emphasis'}`}>
                      {p.ativo ? 'Ativo' : 'Inativo'}
                    </span>
                  </td>
                  <td>
                    <button
                      className="btn btn-sm btn-outline-primary me-1"
                      onClick={() => abrirModal(p)}
                      title="Editar"
                    >
                      <i className="fas fa-edit" />
                    </button>
                    <button
                      className="btn btn-sm btn-outline-danger"
                      onClick={() => setExcluindo(p)}
                      title="Excluir"
                    >
                      <i className="fas fa-trash" />
                    </button>
                  </td>
                </tr>
              ))
            )}
          </tbody>
        </table>
      </div>

      {/* Paginação */}
      <div className="d-flex justify-content-between align-items-center">
        <small className="text-muted">{total} registro(s)</small>
        <nav>
          <ul className="pagination pagination-sm mb-0">
            <li className={`page-item ${pagina <= 1 ? 'disabled' : ''}`}>
              <button className="page-link" onClick={() => setPagina(pagina - 1)}>
                <i className="fas fa-chevron-left" />
              </button>
            </li>
            <li className="page-item disabled">
              <span className="page-link">{pagina} / {totalPaginas || 1}</span>
            </li>
            <li className={`page-item ${pagina >= totalPaginas ? 'disabled' : ''}`}>
              <button className="page-link" onClick={() => setPagina(pagina + 1)}>
                <i className="fas fa-chevron-right" />
              </button>
            </li>
          </ul>
        </nav>
      </div>

      {/* Modal Edição (react-bootstrap — componente dinâmico) */}
      <Modal show={modalAberto} onHide={() => setModalAberto(false)} centered>
        <Modal.Header closeButton>
          <Modal.Title>{editando ? 'Editar Produto' : 'Novo Produto'}</Modal.Title>
        </Modal.Header>
        <Modal.Body>
          <div className="mb-3">
            <label className="form-label">Nome</label>
            <input className="form-control" value={formNome} onChange={(e) => setFormNome(e.target.value)} />
          </div>
          <div className="mb-3">
            <label className="form-label">Categoria</label>
            <input className="form-control" value={formCategoria} onChange={(e) => setFormCategoria(e.target.value)} />
          </div>
          <div className="mb-3">
            <label className="form-label">Preço</label>
            <input className="form-control" type="number" step="0.01" value={formPreco} onChange={(e) => setFormPreco(e.target.value)} />
          </div>
        </Modal.Body>
        <Modal.Footer>
          <button className="btn btn-secondary" onClick={() => setModalAberto(false)}>Cancelar</button>
          <button className="btn btn-primary" onClick={salvar}>Salvar</button>
        </Modal.Footer>
      </Modal>

      {/* Modal Confirmação de exclusão (react-bootstrap) */}
      <Modal show={!!excluindo} onHide={() => setExcluindo(null)} centered>
        <Modal.Header closeButton>
          <Modal.Title>Excluir produto</Modal.Title>
        </Modal.Header>
        <Modal.Body>
          Deseja excluir "<strong>{excluindo?.nome}</strong>"?
        </Modal.Body>
        <Modal.Footer>
          <button className="btn btn-secondary" onClick={() => setExcluindo(null)}>Cancelar</button>
          <button className="btn btn-danger" onClick={confirmarExclusao}>Excluir</button>
        </Modal.Footer>
      </Modal>
    </div>
  );
}
```

---

### Equivalência — CRUD

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Tabela | `DataTable` | `FlatList` com cards | `<table>` Bootstrap |
| Busca | `TextField` + filtro | `TextInput` + filtro | `<input>` Bootstrap |
| Ordenação | `DataColumn.onSort` | Implementação manual | Clique no `<th>` |
| Filtro por categoria | `DropdownButton` | Seleção manual | `<select>` Bootstrap |
| Paginação | Botões + estado | Botões + estado | `pagination` Bootstrap |
| Modal edição | `AlertDialog` | `Modal` nativo RN | `react-bootstrap Modal` |
| Confirmação exclusão | `AlertDialog` | `Alert.alert` | `react-bootstrap Modal` |
| Ícones de ação | `Icons.edit` / `Icons.delete` | Emoji | Font Awesome `fa-edit` / `fa-trash` |

---

## 9. Tema Claro/Escuro

Alternância entre tema claro e escuro com:

- Toggle manual (botão/switch).
- Respeito à preferência do sistema operacional como padrão inicial.
- Persistência da escolha do usuário entre sessões.

### Flutter — ThemeData + Riverpod

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: ^2.5.0
  shared_preferences: ^2.3.0
```

```bash
flutter pub get
```

**2. Provider de tema**

```dart
// lib/core/theme/theme_provider.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';

enum ThemeOption { sistema, claro, escuro }

class ThemeNotifier extends Notifier<ThemeOption> {
  static const _key = 'theme_option';

  @override
  ThemeOption build() {
    _carregar();
    return ThemeOption.sistema;
  }

  Future<void> _carregar() async {
    final prefs = await SharedPreferences.getInstance();
    final salvo = prefs.getString(_key);
    if (salvo != null) {
      state = ThemeOption.values.firstWhere(
        (e) => e.name == salvo,
        orElse: () => ThemeOption.sistema,
      );
    }
  }

  Future<void> setTheme(ThemeOption opcao) async {
    state = opcao;
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_key, opcao.name);
  }

  ThemeMode get themeMode => switch (state) {
    ThemeOption.claro => ThemeMode.light,
    ThemeOption.escuro => ThemeMode.dark,
    ThemeOption.sistema => ThemeMode.system,
  };
}

final themeProvider = NotifierProvider<ThemeNotifier, ThemeOption>(
  ThemeNotifier.new,
);
```

**3. Definição dos temas**

```dart
// lib/core/theme/app_themes.dart
import 'package:flutter/material.dart';

abstract final class AppThemes {
  static final light = ThemeData(
    brightness: Brightness.light,
    colorSchemeSeed: Colors.indigo,
    useMaterial3: true,
  );

  static final dark = ThemeData(
    brightness: Brightness.dark,
    colorSchemeSeed: Colors.indigo,
    useMaterial3: true,
  );
}
```

**4. main.dart com suporte a tema**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/theme/app_themes.dart';
import 'core/theme/theme_provider.dart';

void main() {
  runApp(const ProviderScope(child: App()));
}

class App extends ConsumerWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final themeNotifier = ref.watch(themeProvider.notifier);

    return MaterialApp(
      title: 'MeuApp',
      theme: AppThemes.light,
      darkTheme: AppThemes.dark,
      themeMode: themeNotifier.themeMode,
      home: const HomeScreen(),
    );
  }
}
```

**5. Widget de seleção de tema**

```dart
// lib/core/theme/theme_selector.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'theme_provider.dart';

class ThemeSelector extends ConsumerWidget {
  const ThemeSelector({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final opcaoAtual = ref.watch(themeProvider);

    return SegmentedButton<ThemeOption>(
      segments: const [
        ButtonSegment(
          value: ThemeOption.sistema,
          icon: Icon(Icons.brightness_auto),
          label: Text('Sistema'),
        ),
        ButtonSegment(
          value: ThemeOption.claro,
          icon: Icon(Icons.light_mode),
          label: Text('Claro'),
        ),
        ButtonSegment(
          value: ThemeOption.escuro,
          icon: Icon(Icons.dark_mode),
          label: Text('Escuro'),
        ),
      ],
      selected: {opcaoAtual},
      onSelectionChanged: (selected) {
        ref.read(themeProvider.notifier).setTheme(selected.first);
      },
    );
  }
}
```

---

### React Native — useColorScheme + Context

**1. Dependências**

```bash
npx expo install @react-native-async-storage/async-storage
```

**2. Context de tema**

```tsx
// src/core/theme/ThemeContext.tsx
import React, {
  createContext,
  useContext,
  useState,
  useEffect,
  useCallback,
  useMemo,
} from 'react';
import { useColorScheme } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

type ThemeOption = 'sistema' | 'claro' | 'escuro';
type ResolvedTheme = 'light' | 'dark';

interface ThemeContextValue {
  opcao: ThemeOption;
  tema: ResolvedTheme;
  setOpcao: (opcao: ThemeOption) => void;
  cores: typeof CORES_LIGHT;
}

const CORES_LIGHT = {
  background: '#ffffff',
  surface: '#f5f5f5',
  text: '#212121',
  textSecondary: '#666666',
  primary: '#3949ab',
  border: '#e0e0e0',
};

const CORES_DARK = {
  background: '#121212',
  surface: '#1e1e1e',
  text: '#e0e0e0',
  textSecondary: '#999999',
  primary: '#7986cb',
  border: '#333333',
};

const ThemeContext = createContext<ThemeContextValue | null>(null);
const STORAGE_KEY = 'theme_option';

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const systemScheme = useColorScheme();
  const [opcao, setOpcaoState] = useState<ThemeOption>('sistema');

  useEffect(() => {
    AsyncStorage.getItem(STORAGE_KEY).then((salvo) => {
      if (salvo) setOpcaoState(salvo as ThemeOption);
    });
  }, []);

  const setOpcao = useCallback((nova: ThemeOption) => {
    setOpcaoState(nova);
    AsyncStorage.setItem(STORAGE_KEY, nova);
  }, []);

  const tema: ResolvedTheme = useMemo(() => {
    if (opcao === 'sistema') return systemScheme === 'dark' ? 'dark' : 'light';
    return opcao === 'escuro' ? 'dark' : 'light';
  }, [opcao, systemScheme]);

  const cores = tema === 'dark' ? CORES_DARK : CORES_LIGHT;

  return (
    <ThemeContext.Provider value={{ opcao, tema, setOpcao, cores }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme deve ser usado dentro de ThemeProvider');
  return ctx;
}
```

**3. Componente seletor de tema**

```tsx
// src/components/ThemeSelector.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useTheme } from '../core/theme/ThemeContext';

const OPCOES = [
  { valor: 'sistema' as const, icone: '🔄', label: 'Sistema' },
  { valor: 'claro' as const, icone: '☀️', label: 'Claro' },
  { valor: 'escuro' as const, icone: '🌙', label: 'Escuro' },
];

export function ThemeSelector() {
  const { opcao, setOpcao, cores } = useTheme();

  return (
    <View style={styles.container}>
      {OPCOES.map((o) => (
        <TouchableOpacity
          key={o.valor}
          style={[
            styles.botao,
            { borderColor: cores.border },
            opcao === o.valor && { backgroundColor: cores.primary },
          ]}
          onPress={() => setOpcao(o.valor)}
        >
          <Text style={styles.icone}>{o.icone}</Text>
          <Text
            style={[
              styles.label,
              { color: opcao === o.valor ? '#fff' : cores.text },
            ]}
          >
            {o.label}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flexDirection: 'row', gap: 8 },
  botao: {
    flex: 1,
    alignItems: 'center',
    paddingVertical: 12,
    borderRadius: 10,
    borderWidth: 1,
  },
  icone: { fontSize: 20, marginBottom: 4 },
  label: { fontSize: 12, fontWeight: '500' },
});
```

---

### React Web — Bootstrap data-bs-theme + localStorage

Bootstrap 5.3+ suporta tema escuro nativamente via atributo `data-bs-theme` no `<html>`.

**1. Hook de tema**

```typescript
// src/hooks/useTheme.ts
import { useState, useEffect, useCallback, useMemo } from 'react';

type ThemeOption = 'sistema' | 'claro' | 'escuro';
type ResolvedTheme = 'light' | 'dark';

const STORAGE_KEY = 'theme_option';

function getSystemTheme(): ResolvedTheme {
  return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
}

export function useTheme() {
  const [opcao, setOpcaoState] = useState<ThemeOption>(() => {
    return (localStorage.getItem(STORAGE_KEY) as ThemeOption) || 'sistema';
  });

  const tema: ResolvedTheme = useMemo(() => {
    if (opcao === 'sistema') return getSystemTheme();
    return opcao === 'escuro' ? 'dark' : 'light';
  }, [opcao]);

  // Aplicar no <html> para que o Bootstrap leia
  useEffect(() => {
    document.documentElement.setAttribute('data-bs-theme', tema);
  }, [tema]);

  // Escutar mudança de preferência do sistema
  useEffect(() => {
    const mq = window.matchMedia('(prefers-color-scheme: dark)');
    const handler = () => {
      if (opcao === 'sistema') {
        document.documentElement.setAttribute('data-bs-theme', getSystemTheme());
      }
    };
    mq.addEventListener('change', handler);
    return () => mq.removeEventListener('change', handler);
  }, [opcao]);

  const setOpcao = useCallback((nova: ThemeOption) => {
    setOpcaoState(nova);
    localStorage.setItem(STORAGE_KEY, nova);
  }, []);

  return { opcao, tema, setOpcao };
}
```

**2. Componente seletor de tema**

```tsx
// src/components/ThemeSelector.tsx
import { useTheme } from '../hooks/useTheme';

const OPCOES = [
  { valor: 'sistema' as const, icone: 'fas fa-circle-half-stroke', label: 'Sistema' },
  { valor: 'claro' as const, icone: 'fas fa-sun', label: 'Claro' },
  { valor: 'escuro' as const, icone: 'fas fa-moon', label: 'Escuro' },
];

export function ThemeSelector() {
  const { opcao, setOpcao } = useTheme();

  return (
    <div className="btn-group" role="group" aria-label="Tema">
      {OPCOES.map((o) => (
        <button
          key={o.valor}
          type="button"
          className={`btn ${opcao === o.valor ? 'btn-primary' : 'btn-outline-secondary'}`}
          onClick={() => setOpcao(o.valor)}
        >
          <i className={`${o.icone} me-1`} />
          {o.label}
        </button>
      ))}
    </div>
  );
}
```

**3. Uso no App — basta chamar o hook uma vez no componente raiz**

```tsx
// src/App.tsx (trecho relevante)
import { useTheme } from './hooks/useTheme';
import { ThemeSelector } from './components/ThemeSelector';

export function App() {
  useTheme(); // Aplica data-bs-theme no <html>

  return (
    <>
      {/* Em qualquer lugar da UI: */}
      <ThemeSelector />
      {/* Resto do app — Bootstrap já respeita data-bs-theme automaticamente */}
    </>
  );
}
```

> **Como funciona:** o Bootstrap 5.3+ lê `data-bs-theme="dark"` no `<html>` e inverte automaticamente as cores de todos os componentes (`bg-body`, `text-body`, `table`, `card`, `form-control`, etc.). Não é necessário alterar classes CSS individuais — tudo é tratado via variáveis CSS do framework.

---

### Equivalência — Tema Claro/Escuro

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Tema do sistema | `ThemeMode.system` | `useColorScheme()` | `prefers-color-scheme` media query |
| Aplicação do tema | `MaterialApp.themeMode` | Cores via Context | `data-bs-theme` no `<html>` |
| Persistência | `shared_preferences` | `AsyncStorage` | `localStorage` |
| Toggle UI | `SegmentedButton` | Botões customizados | `btn-group` Bootstrap |
| Definição de cores | `ThemeData` light/dark | Objeto de cores | Variáveis CSS do Bootstrap |
| Reatividade ao sistema | Automático via `ThemeMode.system` | `useColorScheme` + `useMemo` | `matchMedia.addEventListener('change')` |
| Propagação | Material widgets herdam | Context provider | CSS cascade via `data-bs-theme` |

---

## 10. Câmera e Galeria

Funcionalidades de captura de foto pela câmera, seleção da galeria, recorte (crop) e preview antes do uso.

### Flutter — image_picker + image_cropper

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  image_picker: ^1.1.0
  image_cropper: ^8.0.0
```

```bash
flutter pub get
```

**2. Configuração Android — `android/app/src/main/AndroidManifest.xml`**

```xml
<manifest>
  <uses-permission android:name="android.permission.CAMERA" />
  <application>
    <activity
      android:name="com.yalantis.ucrop.UCropActivity"
      android:screenOrientation="portrait"
      android:theme="@style/Theme.AppCompat.Light.NoActionBar" />
  </application>
</manifest>
```

**3. Configuração iOS — `ios/Runner/Info.plist`**

```xml
<dict>
  <key>NSCameraUsageDescription</key>
  <string>Precisamos da câmera para tirar fotos.</string>
  <key>NSPhotoLibraryUsageDescription</key>
  <string>Precisamos acessar suas fotos.</string>
</dict>
```

**4. Serviço de imagem**

```dart
// lib/core/services/image_service.dart
import 'dart:io';
import 'package:image_picker/image_picker.dart';
import 'package:image_cropper/image_cropper.dart';
import 'package:flutter/material.dart';

class ImageService {
  final _picker = ImagePicker();

  Future<File?> capturarDaCamera() async {
    final xFile = await _picker.pickImage(
      source: ImageSource.camera,
      maxWidth: 1024,
      maxHeight: 1024,
      imageQuality: 85,
    );
    if (xFile == null) return null;
    return File(xFile.path);
  }

  Future<File?> selecionarDaGaleria() async {
    final xFile = await _picker.pickImage(
      source: ImageSource.gallery,
      maxWidth: 1024,
      maxHeight: 1024,
      imageQuality: 85,
    );
    if (xFile == null) return null;
    return File(xFile.path);
  }

  Future<List<File>> selecionarMultiplas() async {
    final xFiles = await _picker.pickMultiImage(
      maxWidth: 1024,
      maxHeight: 1024,
      imageQuality: 85,
    );
    return xFiles.map((xf) => File(xf.path)).toList();
  }

  Future<File?> recortar(File imagem) async {
    final resultado = await ImageCropper().cropImage(
      sourcePath: imagem.path,
      aspectRatio: const CropAspectRatio(ratioX: 1, ratioY: 1),
      uiSettings: [
        AndroidUiSettings(
          toolbarTitle: 'Recortar',
          toolbarColor: Colors.indigo,
          toolbarWidgetColor: Colors.white,
          lockAspectRatio: false,
        ),
        IOSUiSettings(title: 'Recortar'),
      ],
    );
    if (resultado == null) return null;
    return File(resultado.path);
  }
}
```

**5. Tela de câmera/galeria**

```dart
// lib/features/camera/presentation/camera_screen.dart
import 'dart:io';
import 'package:flutter/material.dart';
import '../../../core/services/image_service.dart';

class CameraScreen extends StatefulWidget {
  const CameraScreen({super.key});

  @override
  State<CameraScreen> createState() => _CameraScreenState();
}

class _CameraScreenState extends State<CameraScreen> {
  final _imageService = ImageService();
  File? _imagemAtual;
  List<File> _galeria = [];

  Future<void> _capturar() async {
    final imagem = await _imageService.capturarDaCamera();
    if (imagem != null) {
      final recortada = await _imageService.recortar(imagem);
      setState(() {
        _imagemAtual = recortada ?? imagem;
        _galeria = [..._galeria, _imagemAtual!];
      });
    }
  }

  Future<void> _selecionarGaleria() async {
    final imagem = await _imageService.selecionarDaGaleria();
    if (imagem != null) {
      final recortada = await _imageService.recortar(imagem);
      setState(() {
        _imagemAtual = recortada ?? imagem;
        _galeria = [..._galeria, _imagemAtual!];
      });
    }
  }

  Future<void> _selecionarMultiplas() async {
    final imagens = await _imageService.selecionarMultiplas();
    if (imagens.isNotEmpty) {
      setState(() {
        _galeria = [..._galeria, ...imagens];
        _imagemAtual = imagens.last;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Câmera e Galeria')),
      body: Column(
        children: [
          // Preview principal
          Container(
            height: 300,
            width: double.infinity,
            color: Colors.grey.shade200,
            child: _imagemAtual != null
                ? Image.file(_imagemAtual!, fit: BoxFit.contain)
                : const Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(Icons.image, size: 64, color: Colors.grey),
                        SizedBox(height: 8),
                        Text('Nenhuma imagem selecionada',
                            style: TextStyle(color: Colors.grey)),
                      ],
                    ),
                  ),
          ),

          // Botões de ação
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: FilledButton.icon(
                    onPressed: _capturar,
                    icon: const Icon(Icons.camera_alt),
                    label: const Text('Câmera'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: _selecionarGaleria,
                    icon: const Icon(Icons.photo_library),
                    label: const Text('Galeria'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: _selecionarMultiplas,
                    icon: const Icon(Icons.photo_library_outlined),
                    label: const Text('Múltiplas'),
                  ),
                ),
              ],
            ),
          ),

          // Grid de fotos capturadas
          if (_galeria.isNotEmpty) ...[
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text('Fotos (${_galeria.length})',
                      style: Theme.of(context).textTheme.titleSmall),
                  TextButton(
                    onPressed: () => setState(() {
                      _galeria = [];
                      _imagemAtual = null;
                    }),
                    child: const Text('Limpar'),
                  ),
                ],
              ),
            ),
            Expanded(
              child: GridView.builder(
                padding: const EdgeInsets.all(16),
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 3,
                  mainAxisSpacing: 8,
                  crossAxisSpacing: 8,
                ),
                itemCount: _galeria.length,
                itemBuilder: (_, i) {
                  return GestureDetector(
                    onTap: () => setState(() => _imagemAtual = _galeria[i]),
                    child: ClipRRect(
                      borderRadius: BorderRadius.circular(8),
                      child: Image.file(_galeria[i], fit: BoxFit.cover),
                    ),
                  );
                },
              ),
            ),
          ],
        ],
      ),
    );
  }
}
```

---

### React Native — expo-camera + expo-image-picker

**1. Dependências**

```bash
npx expo install expo-camera expo-image-picker expo-image-manipulator
```

**2. Hook de imagem**

```typescript
// src/hooks/useImageCapture.ts
import { useState, useCallback } from 'react';
import * as ImagePicker from 'expo-image-picker';
import * as ImageManipulator from 'expo-image-manipulator';

export function useImageCapture() {
  const [imagens, setImagens] = useState<string[]>([]);
  const [preview, setPreview] = useState<string | null>(null);

  const capturarDaCamera = useCallback(async () => {
    const permissao = await ImagePicker.requestCameraPermissionsAsync();
    if (!permissao.granted) return;

    const resultado = await ImagePicker.launchCameraAsync({
      allowsEditing: true,
      aspect: [1, 1],
      quality: 0.85,
    });

    if (!resultado.canceled) {
      const uri = resultado.assets[0].uri;
      setPreview(uri);
      setImagens((prev) => [...prev, uri]);
    }
  }, []);

  const selecionarDaGaleria = useCallback(async () => {
    const permissao = await ImagePicker.requestMediaLibraryPermissionsAsync();
    if (!permissao.granted) return;

    const resultado = await ImagePicker.launchImageLibraryAsync({
      allowsEditing: true,
      aspect: [1, 1],
      quality: 0.85,
    });

    if (!resultado.canceled) {
      const uri = resultado.assets[0].uri;
      setPreview(uri);
      setImagens((prev) => [...prev, uri]);
    }
  }, []);

  const selecionarMultiplas = useCallback(async () => {
    const resultado = await ImagePicker.launchImageLibraryAsync({
      allowsMultipleSelection: true,
      quality: 0.85,
    });

    if (!resultado.canceled) {
      const uris = resultado.assets.map((a) => a.uri);
      setImagens((prev) => [...prev, ...uris]);
      setPreview(uris[uris.length - 1]);
    }
  }, []);

  const redimensionar = useCallback(async (uri: string) => {
    const result = await ImageManipulator.manipulateAsync(
      uri,
      [{ resize: { width: 512 } }],
      { compress: 0.8, format: ImageManipulator.SaveFormat.JPEG },
    );
    return result.uri;
  }, []);

  const limpar = useCallback(() => {
    setImagens([]);
    setPreview(null);
  }, []);

  return {
    imagens, preview, setPreview,
    capturarDaCamera, selecionarDaGaleria, selecionarMultiplas,
    redimensionar, limpar,
  };
}
```

**3. Tela de câmera/galeria**

```tsx
// src/features/camera/CameraScreen.tsx
import React from 'react';
import {
  View, Text, Image, TouchableOpacity, FlatList, StyleSheet,
} from 'react-native';
import { useImageCapture } from '../../hooks/useImageCapture';

export function CameraScreen() {
  const {
    imagens, preview, setPreview,
    capturarDaCamera, selecionarDaGaleria, selecionarMultiplas, limpar,
  } = useImageCapture();

  return (
    <View style={styles.container}>
      {/* Preview principal */}
      <View style={styles.previewContainer}>
        {preview ? (
          <Image source={{ uri: preview }} style={styles.previewImg} />
        ) : (
          <View style={styles.previewPlaceholder}>
            <Text style={styles.placeholderIcon}>🖼️</Text>
            <Text style={styles.placeholderTexto}>Nenhuma imagem selecionada</Text>
          </View>
        )}
      </View>

      {/* Botões */}
      <View style={styles.botoes}>
        <TouchableOpacity style={styles.btn} onPress={capturarDaCamera}>
          <Text style={styles.btnTexto}>📷 Câmera</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.btnOutline} onPress={selecionarDaGaleria}>
          <Text style={styles.btnOutlineTexto}>🖼️ Galeria</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.btnOutline} onPress={selecionarMultiplas}>
          <Text style={styles.btnOutlineTexto}>📁 Múltiplas</Text>
        </TouchableOpacity>
      </View>

      {/* Grid de miniaturas */}
      {imagens.length > 0 && (
        <>
          <View style={styles.galeriaHeader}>
            <Text style={styles.galeriaTitulo}>Fotos ({imagens.length})</Text>
            <TouchableOpacity onPress={limpar}>
              <Text style={styles.limparTexto}>Limpar</Text>
            </TouchableOpacity>
          </View>
          <FlatList
            data={imagens}
            keyExtractor={(_, i) => String(i)}
            numColumns={3}
            contentContainerStyle={styles.grid}
            columnWrapperStyle={{ gap: 8 }}
            renderItem={({ item }) => (
              <TouchableOpacity
                style={styles.miniatura}
                onPress={() => setPreview(item)}
              >
                <Image source={{ uri: item }} style={styles.miniaturaImg} />
              </TouchableOpacity>
            )}
            ItemSeparatorComponent={() => <View style={{ height: 8 }} />}
          />
        </>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  previewContainer: { height: 280, backgroundColor: '#f0f0f0' },
  previewImg: { width: '100%', height: '100%', resizeMode: 'contain' },
  previewPlaceholder: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  placeholderIcon: { fontSize: 48 },
  placeholderTexto: { color: '#999', marginTop: 8 },
  botoes: { flexDirection: 'row', gap: 8, padding: 16 },
  btn: { flex: 1, backgroundColor: '#3949ab', paddingVertical: 12, borderRadius: 8, alignItems: 'center' },
  btnTexto: { color: '#fff', fontWeight: '600' },
  btnOutline: { flex: 1, borderWidth: 1, borderColor: '#3949ab', paddingVertical: 12, borderRadius: 8, alignItems: 'center' },
  btnOutlineTexto: { color: '#3949ab', fontWeight: '600' },
  galeriaHeader: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', paddingHorizontal: 16, marginBottom: 8 },
  galeriaTitulo: { fontWeight: '600' },
  limparTexto: { color: '#e53935' },
  grid: { paddingHorizontal: 16, paddingBottom: 16 },
  miniatura: { flex: 1 / 3 },
  miniaturaImg: { aspectRatio: 1, borderRadius: 8 },
});
```

---

### React Web — MediaDevices API + Canvas

**1. Hook de captura de imagem**

```typescript
// src/hooks/useImageCapture.ts
import { useRef, useState, useCallback } from 'react';

export function useImageCapture() {
  const [imagens, setImagens] = useState<string[]>([]);
  const [preview, setPreview] = useState<string | null>(null);
  const videoRef = useRef<HTMLVideoElement | null>(null);
  const streamRef = useRef<MediaStream | null>(null);
  const [cameraAberta, setCameraAberta] = useState(false);

  const abrirCamera = useCallback(async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: 'environment', width: 1024, height: 1024 },
      });
      streamRef.current = stream;
      setCameraAberta(true);
      requestAnimationFrame(() => {
        if (videoRef.current) {
          videoRef.current.srcObject = stream;
          videoRef.current.play();
        }
      });
    } catch {
      alert('Não foi possível acessar a câmera.');
    }
  }, []);

  const capturar = useCallback(() => {
    if (!videoRef.current) return;
    const canvas = document.createElement('canvas');
    canvas.width = videoRef.current.videoWidth;
    canvas.height = videoRef.current.videoHeight;
    canvas.getContext('2d')!.drawImage(videoRef.current, 0, 0);
    const dataUrl = canvas.toDataURL('image/jpeg', 0.85);

    setPreview(dataUrl);
    setImagens((prev) => [...prev, dataUrl]);
    fecharCamera();
  }, []);

  const fecharCamera = useCallback(() => {
    streamRef.current?.getTracks().forEach((t) => t.stop());
    streamRef.current = null;
    setCameraAberta(false);
  }, []);

  const selecionarArquivos = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => {
      const files = e.target.files;
      if (!files) return;
      Array.from(files).forEach((file) => {
        const url = URL.createObjectURL(file);
        setImagens((prev) => [...prev, url]);
        setPreview(url);
      });
      e.target.value = '';
    },
    [],
  );

  const limpar = useCallback(() => {
    setImagens([]);
    setPreview(null);
  }, []);

  return {
    imagens, preview, setPreview,
    videoRef, cameraAberta,
    abrirCamera, capturar, fecharCamera,
    selecionarArquivos, limpar,
  };
}
```

**2. Página de câmera/galeria**

```tsx
// src/features/camera/CameraPage.tsx
import { useRef } from 'react';
import { useImageCapture } from '../../hooks/useImageCapture';

export function CameraPage() {
  const {
    imagens, preview, setPreview,
    videoRef, cameraAberta,
    abrirCamera, capturar, fecharCamera,
    selecionarArquivos, limpar,
  } = useImageCapture();
  const fileInputRef = useRef<HTMLInputElement>(null);

  return (
    <div className="container py-4" style={{ maxWidth: 720 }}>
      {/* Preview / câmera */}
      <div
        className="bg-body-secondary rounded-3 d-flex align-items-center justify-content-center mb-3 overflow-hidden"
        style={{ height: 300 }}
      >
        {cameraAberta ? (
          <video ref={videoRef} className="w-100 h-100" style={{ objectFit: 'cover' }} />
        ) : preview ? (
          <img src={preview} alt="Preview" className="mw-100 mh-100" style={{ objectFit: 'contain' }} />
        ) : (
          <div className="text-center text-muted">
            <i className="fas fa-image fa-3x mb-2" />
            <p>Nenhuma imagem selecionada</p>
          </div>
        )}
      </div>

      {/* Botões */}
      <div className="d-flex gap-2 mb-4">
        {cameraAberta ? (
          <>
            <button className="btn btn-danger flex-fill" onClick={capturar}>
              <i className="fas fa-circle me-1" /> Capturar
            </button>
            <button className="btn btn-secondary" onClick={fecharCamera}>
              Cancelar
            </button>
          </>
        ) : (
          <>
            <button className="btn btn-primary flex-fill" onClick={abrirCamera}>
              <i className="fas fa-camera me-1" /> Câmera
            </button>
            <button
              className="btn btn-outline-primary flex-fill"
              onClick={() => fileInputRef.current?.click()}
            >
              <i className="fas fa-images me-1" /> Galeria
            </button>
            <input
              ref={fileInputRef}
              type="file"
              accept="image/*"
              multiple
              className="d-none"
              onChange={selecionarArquivos}
            />
          </>
        )}
      </div>

      {/* Grid de miniaturas */}
      {imagens.length > 0 && (
        <>
          <div className="d-flex justify-content-between align-items-center mb-2">
            <h6 className="mb-0">Fotos ({imagens.length})</h6>
            <button className="btn btn-sm btn-link text-danger" onClick={limpar}>
              Limpar
            </button>
          </div>
          <div className="row g-2">
            {imagens.map((src, i) => (
              <div key={i} className="col-4">
                <img
                  src={src}
                  alt={`Foto ${i + 1}`}
                  className="w-100 rounded-2"
                  role="button"
                  style={{ aspectRatio: '1', objectFit: 'cover' }}
                  onClick={() => setPreview(src)}
                />
              </div>
            ))}
          </div>
        </>
      )}
    </div>
  );
}
```

---

### Equivalência — Câmera e Galeria

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Câmera | `ImagePicker.pickImage(source: camera)` | `ImagePicker.launchCameraAsync` | `navigator.mediaDevices.getUserMedia` |
| Galeria | `ImagePicker.pickImage(source: gallery)` | `ImagePicker.launchImageLibraryAsync` | `<input type="file" accept="image/*">` |
| Múltiplas fotos | `ImagePicker.pickMultiImage` | `allowsMultipleSelection: true` | `<input multiple>` |
| Crop/recorte | `image_cropper` (UCrop) | `allowsEditing: true` no picker | Canvas API manual |
| Redimensionar | `maxWidth/maxHeight` no picker | `expo-image-manipulator` | Canvas `drawImage` |
| Permissões | `Info.plist` + `AndroidManifest` | `requestCameraPermissionsAsync` | Prompt automático do navegador |
| Preview | `Image.file()` | `Image source={{ uri }}` | `<img src={dataUrl \| objectUrl}>` |

---

## 11. Dashboard com Gráficos

Dashboard com gráficos de barras, linhas e pizza, alimentados por dados dinâmicos simulados.

### Flutter — fl_chart

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  fl_chart: ^0.69.0
```

```bash
flutter pub get
```

**2. Modelo de dados**

```dart
// lib/features/dashboard/domain/dashboard_data.dart
class VendasMensais {
  final String mes;
  final double valor;
  VendasMensais(this.mes, this.valor);
}

class CategoriaProporcao {
  final String nome;
  final double percentual;
  CategoriaProporcao(this.nome, this.percentual);
}

class DashboardData {
  final double receitaTotal;
  final int totalPedidos;
  final int novosClientes;
  final List<VendasMensais> vendasMensais;
  final List<CategoriaProporcao> categorias;

  DashboardData({
    required this.receitaTotal,
    required this.totalPedidos,
    required this.novosClientes,
    required this.vendasMensais,
    required this.categorias,
  });
}
```

**3. Serviço simulado**

```dart
// lib/features/dashboard/data/dashboard_service.dart
import '../domain/dashboard_data.dart';

class DashboardService {
  Future<DashboardData> buscarDados() async {
    await Future.delayed(const Duration(milliseconds: 500));

    return DashboardData(
      receitaTotal: 158430.00,
      totalPedidos: 1247,
      novosClientes: 89,
      vendasMensais: [
        VendasMensais('Jan', 12500),
        VendasMensais('Fev', 15200),
        VendasMensais('Mar', 18900),
        VendasMensais('Abr', 14300),
        VendasMensais('Mai', 21000),
        VendasMensais('Jun', 19800),
      ],
      categorias: [
        CategoriaProporcao('Eletrônicos', 35),
        CategoriaProporcao('Roupas', 25),
        CategoriaProporcao('Alimentos', 20),
        CategoriaProporcao('Outros', 20),
      ],
    );
  }
}
```

**4. Tela do dashboard**

```dart
// lib/features/dashboard/presentation/dashboard_screen.dart
import 'package:flutter/material.dart';
import 'package:fl_chart/fl_chart.dart';
import '../data/dashboard_service.dart';
import '../domain/dashboard_data.dart';

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  final _service = DashboardService();
  DashboardData? _dados;
  bool _carregando = true;

  @override
  void initState() {
    super.initState();
    _carregar();
  }

  Future<void> _carregar() async {
    final dados = await _service.buscarDados();
    setState(() { _dados = dados; _carregando = false; });
  }

  @override
  Widget build(BuildContext context) {
    if (_carregando) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }
    final d = _dados!;

    return Scaffold(
      appBar: AppBar(title: const Text('Dashboard')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Cards de resumo
            Row(
              children: [
                _CardResumo(titulo: 'Receita', valor: 'R\$ ${(d.receitaTotal / 1000).toStringAsFixed(1)}k', icone: Icons.attach_money, cor: Colors.green),
                const SizedBox(width: 12),
                _CardResumo(titulo: 'Pedidos', valor: '${d.totalPedidos}', icone: Icons.shopping_cart, cor: Colors.blue),
                const SizedBox(width: 12),
                _CardResumo(titulo: 'Novos Clientes', valor: '${d.novosClientes}', icone: Icons.person_add, cor: Colors.orange),
              ],
            ),
            const SizedBox(height: 24),

            // Gráfico de barras — Vendas mensais
            Text('Vendas Mensais', style: Theme.of(context).textTheme.titleMedium),
            const SizedBox(height: 12),
            SizedBox(
              height: 220,
              child: BarChart(
                BarChartData(
                  barGroups: d.vendasMensais.asMap().entries.map((e) {
                    return BarChartGroupData(x: e.key, barRods: [
                      BarChartRodData(
                        toY: e.value.valor / 1000,
                        color: Colors.indigo,
                        width: 20,
                        borderRadius: const BorderRadius.vertical(top: Radius.circular(4)),
                      ),
                    ]);
                  }).toList(),
                  titlesData: FlTitlesData(
                    bottomTitles: AxisTitles(sideTitles: SideTitles(
                      showTitles: true,
                      getTitlesWidget: (value, _) => Text(
                        d.vendasMensais[value.toInt()].mes,
                        style: const TextStyle(fontSize: 11),
                      ),
                    )),
                    leftTitles: AxisTitles(sideTitles: SideTitles(
                      showTitles: true,
                      reservedSize: 40,
                      getTitlesWidget: (value, _) => Text('${value.toInt()}k', style: const TextStyle(fontSize: 10)),
                    )),
                    topTitles: const AxisTitles(sideTitles: SideTitles(showTitles: false)),
                    rightTitles: const AxisTitles(sideTitles: SideTitles(showTitles: false)),
                  ),
                  borderData: FlBorderData(show: false),
                  gridData: const FlGridData(drawVerticalLine: false),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // Gráfico de linha — tendência
            Text('Tendência de Vendas', style: Theme.of(context).textTheme.titleMedium),
            const SizedBox(height: 12),
            SizedBox(
              height: 200,
              child: LineChart(
                LineChartData(
                  lineBarsData: [
                    LineChartBarData(
                      spots: d.vendasMensais.asMap().entries
                          .map((e) => FlSpot(e.key.toDouble(), e.value.valor / 1000))
                          .toList(),
                      isCurved: true,
                      color: Colors.indigo,
                      barWidth: 3,
                      dotData: const FlDotData(show: true),
                      belowBarData: BarAreaData(
                        show: true,
                        color: Colors.indigo.withAlpha(30),
                      ),
                    ),
                  ],
                  titlesData: FlTitlesData(
                    bottomTitles: AxisTitles(sideTitles: SideTitles(
                      showTitles: true,
                      getTitlesWidget: (value, _) {
                        final i = value.toInt();
                        if (i < 0 || i >= d.vendasMensais.length) return const SizedBox();
                        return Text(d.vendasMensais[i].mes, style: const TextStyle(fontSize: 11));
                      },
                    )),
                    leftTitles: AxisTitles(sideTitles: SideTitles(
                      showTitles: true, reservedSize: 40,
                      getTitlesWidget: (v, _) => Text('${v.toInt()}k', style: const TextStyle(fontSize: 10)),
                    )),
                    topTitles: const AxisTitles(sideTitles: SideTitles(showTitles: false)),
                    rightTitles: const AxisTitles(sideTitles: SideTitles(showTitles: false)),
                  ),
                  borderData: FlBorderData(show: false),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // Gráfico de pizza — categorias
            Text('Vendas por Categoria', style: Theme.of(context).textTheme.titleMedium),
            const SizedBox(height: 12),
            SizedBox(
              height: 200,
              child: PieChart(
                PieChartData(
                  sections: d.categorias.asMap().entries.map((e) {
                    final cores = [Colors.indigo, Colors.teal, Colors.orange, Colors.pink];
                    return PieChartSectionData(
                      value: e.value.percentual,
                      title: '${e.value.percentual.toInt()}%',
                      color: cores[e.key % cores.length],
                      radius: 80,
                      titleStyle: const TextStyle(fontSize: 12, fontWeight: FontWeight.bold, color: Colors.white),
                    );
                  }).toList(),
                  sectionsSpace: 2,
                  centerSpaceRadius: 0,
                ),
              ),
            ),
            const SizedBox(height: 8),
            Wrap(
              spacing: 16,
              children: d.categorias.asMap().entries.map((e) {
                final cores = [Colors.indigo, Colors.teal, Colors.orange, Colors.pink];
                return Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Container(width: 12, height: 12, color: cores[e.key % cores.length]),
                    const SizedBox(width: 6),
                    Text(e.value.nome, style: const TextStyle(fontSize: 12)),
                  ],
                );
              }).toList(),
            ),
          ],
        ),
      ),
    );
  }
}

class _CardResumo extends StatelessWidget {
  final String titulo;
  final String valor;
  final IconData icone;
  final Color cor;

  const _CardResumo({required this.titulo, required this.valor, required this.icone, required this.cor});

  @override
  Widget build(BuildContext context) {
    return Expanded(
      child: Card(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Icon(icone, color: cor, size: 28),
              const SizedBox(height: 8),
              Text(valor, style: Theme.of(context).textTheme.titleLarge?.copyWith(fontWeight: FontWeight.bold)),
              Text(titulo, style: Theme.of(context).textTheme.bodySmall),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

### React Native — react-native-chart-kit

**1. Dependências**

```bash
npm install react-native-chart-kit react-native-svg
```

**2. Tela do dashboard**

```tsx
// src/features/dashboard/DashboardScreen.tsx
import React, { useEffect, useState } from 'react';
import { View, Text, ScrollView, StyleSheet, ActivityIndicator } from 'react-native';
import { BarChart, LineChart, PieChart } from 'react-native-chart-kit';
import { useWindowDimensions } from 'react-native';

interface DashboardData {
  receita: number;
  pedidos: number;
  novosClientes: number;
  vendasMensais: { mes: string; valor: number }[];
  categorias: { nome: string; percentual: number; cor: string }[];
}

async function buscarDados(): Promise<DashboardData> {
  await new Promise((r) => setTimeout(r, 500));
  return {
    receita: 158430,
    pedidos: 1247,
    novosClientes: 89,
    vendasMensais: [
      { mes: 'Jan', valor: 12500 }, { mes: 'Fev', valor: 15200 },
      { mes: 'Mar', valor: 18900 }, { mes: 'Abr', valor: 14300 },
      { mes: 'Mai', valor: 21000 }, { mes: 'Jun', valor: 19800 },
    ],
    categorias: [
      { nome: 'Eletrônicos', percentual: 35, cor: '#3949ab' },
      { nome: 'Roupas', percentual: 25, cor: '#00897b' },
      { nome: 'Alimentos', percentual: 20, cor: '#f57c00' },
      { nome: 'Outros', percentual: 20, cor: '#e91e63' },
    ],
  };
}

export function DashboardScreen() {
  const [dados, setDados] = useState<DashboardData | null>(null);
  const { width } = useWindowDimensions();
  const chartWidth = width - 48;

  useEffect(() => { buscarDados().then(setDados); }, []);

  if (!dados) return <ActivityIndicator size="large" style={{ marginTop: 60 }} />;

  const chartConfig = {
    backgroundGradientFrom: '#fff',
    backgroundGradientTo: '#fff',
    color: (opacity = 1) => `rgba(57, 73, 171, ${opacity})`,
    labelColor: () => '#666',
    decimalCount: 0,
  };

  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.content}>
      {/* Cards de resumo */}
      <View style={styles.cardsRow}>
        <CardResumo titulo="Receita" valor={`R$ ${(dados.receita / 1000).toFixed(1)}k`} emoji="💰" />
        <CardResumo titulo="Pedidos" valor={String(dados.pedidos)} emoji="🛒" />
        <CardResumo titulo="Clientes" valor={String(dados.novosClientes)} emoji="👤" />
      </View>

      {/* Gráfico de barras */}
      <Text style={styles.titulo}>Vendas Mensais</Text>
      <BarChart
        data={{
          labels: dados.vendasMensais.map((v) => v.mes),
          datasets: [{ data: dados.vendasMensais.map((v) => v.valor / 1000) }],
        }}
        width={chartWidth}
        height={220}
        yAxisSuffix="k"
        chartConfig={chartConfig}
        style={styles.grafico}
      />

      {/* Gráfico de linha */}
      <Text style={styles.titulo}>Tendência de Vendas</Text>
      <LineChart
        data={{
          labels: dados.vendasMensais.map((v) => v.mes),
          datasets: [{ data: dados.vendasMensais.map((v) => v.valor / 1000) }],
        }}
        width={chartWidth}
        height={200}
        yAxisSuffix="k"
        chartConfig={chartConfig}
        bezier
        style={styles.grafico}
      />

      {/* Gráfico de pizza */}
      <Text style={styles.titulo}>Vendas por Categoria</Text>
      <PieChart
        data={dados.categorias.map((c) => ({
          name: c.nome,
          population: c.percentual,
          color: c.cor,
          legendFontColor: '#666',
          legendFontSize: 12,
        }))}
        width={chartWidth}
        height={200}
        chartConfig={chartConfig}
        accessor="population"
        backgroundColor="transparent"
        paddingLeft="16"
      />
    </ScrollView>
  );
}

function CardResumo({ titulo, valor, emoji }: { titulo: string; valor: string; emoji: string }) {
  return (
    <View style={styles.card}>
      <Text style={styles.cardEmoji}>{emoji}</Text>
      <Text style={styles.cardValor}>{valor}</Text>
      <Text style={styles.cardTitulo}>{titulo}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f9f9f9' },
  content: { padding: 16 },
  cardsRow: { flexDirection: 'row', gap: 10, marginBottom: 24 },
  card: { flex: 1, backgroundColor: '#fff', borderRadius: 12, padding: 16, elevation: 1 },
  cardEmoji: { fontSize: 24, marginBottom: 8 },
  cardValor: { fontSize: 20, fontWeight: '700' },
  cardTitulo: { fontSize: 12, color: '#999', marginTop: 4 },
  titulo: { fontSize: 16, fontWeight: '600', marginBottom: 12, marginTop: 8 },
  grafico: { borderRadius: 12, marginBottom: 16 },
});
```

---

### React Web — Chart.js + Bootstrap

**1. Dependências**

```bash
npm install chart.js react-chartjs-2
```

**2. Página do dashboard**

```tsx
// src/features/dashboard/DashboardPage.tsx
import { useEffect, useState } from 'react';
import {
  Chart as ChartJS, CategoryScale, LinearScale, BarElement, PointElement,
  LineElement, ArcElement, Tooltip, Legend, Filler,
} from 'chart.js';
import { Bar, Line, Pie } from 'react-chartjs-2';

ChartJS.register(
  CategoryScale, LinearScale, BarElement, PointElement,
  LineElement, ArcElement, Tooltip, Legend, Filler,
);

interface DashboardData {
  receita: number;
  pedidos: number;
  novosClientes: number;
  meses: string[];
  valores: number[];
  categorias: { nome: string; percentual: number; cor: string }[];
}

async function buscarDados(): Promise<DashboardData> {
  await new Promise((r) => setTimeout(r, 500));
  return {
    receita: 158430, pedidos: 1247, novosClientes: 89,
    meses: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'],
    valores: [12500, 15200, 18900, 14300, 21000, 19800],
    categorias: [
      { nome: 'Eletrônicos', percentual: 35, cor: '#3949ab' },
      { nome: 'Roupas', percentual: 25, cor: '#00897b' },
      { nome: 'Alimentos', percentual: 20, cor: '#f57c00' },
      { nome: 'Outros', percentual: 20, cor: '#e91e63' },
    ],
  };
}

export function DashboardPage() {
  const [dados, setDados] = useState<DashboardData | null>(null);

  useEffect(() => { buscarDados().then(setDados); }, []);

  if (!dados) {
    return (
      <div className="d-flex justify-content-center py-5">
        <span className="spinner-border" />
      </div>
    );
  }

  const resumos = [
    { titulo: 'Receita', valor: `R$ ${(dados.receita / 1000).toFixed(1)}k`, icone: 'fas fa-dollar-sign', cor: 'text-success' },
    { titulo: 'Pedidos', valor: String(dados.pedidos), icone: 'fas fa-shopping-cart', cor: 'text-primary' },
    { titulo: 'Novos Clientes', valor: String(dados.novosClientes), icone: 'fas fa-user-plus', cor: 'text-warning' },
  ];

  return (
    <div className="container-fluid p-4">
      {/* Cards de resumo */}
      <div className="row g-3 mb-4">
        {resumos.map((r) => (
          <div key={r.titulo} className="col-md-4">
            <div className="card border-0 shadow-sm h-100">
              <div className="card-body">
                <i className={`${r.icone} ${r.cor} fa-lg mb-2`} />
                <h4 className="fw-bold mb-0">{r.valor}</h4>
                <small className="text-muted">{r.titulo}</small>
              </div>
            </div>
          </div>
        ))}
      </div>

      <div className="row g-4">
        {/* Gráfico de barras */}
        <div className="col-lg-6">
          <div className="card border-0 shadow-sm">
            <div className="card-body">
              <h6 className="card-title">Vendas Mensais</h6>
              <Bar
                data={{
                  labels: dados.meses,
                  datasets: [{
                    label: 'Vendas (R$ mil)',
                    data: dados.valores.map((v) => v / 1000),
                    backgroundColor: 'rgba(57, 73, 171, 0.8)',
                    borderRadius: 4,
                  }],
                }}
                options={{ responsive: true, plugins: { legend: { display: false } } }}
              />
            </div>
          </div>
        </div>

        {/* Gráfico de linha */}
        <div className="col-lg-6">
          <div className="card border-0 shadow-sm">
            <div className="card-body">
              <h6 className="card-title">Tendência de Vendas</h6>
              <Line
                data={{
                  labels: dados.meses,
                  datasets: [{
                    label: 'Vendas (R$ mil)',
                    data: dados.valores.map((v) => v / 1000),
                    borderColor: '#3949ab',
                    backgroundColor: 'rgba(57, 73, 171, 0.1)',
                    fill: true,
                    tension: 0.4,
                  }],
                }}
                options={{ responsive: true, plugins: { legend: { display: false } } }}
              />
            </div>
          </div>
        </div>

        {/* Gráfico de pizza */}
        <div className="col-lg-6">
          <div className="card border-0 shadow-sm">
            <div className="card-body">
              <h6 className="card-title">Vendas por Categoria</h6>
              <div style={{ maxWidth: 300, margin: '0 auto' }}>
                <Pie
                  data={{
                    labels: dados.categorias.map((c) => c.nome),
                    datasets: [{
                      data: dados.categorias.map((c) => c.percentual),
                      backgroundColor: dados.categorias.map((c) => c.cor),
                    }],
                  }}
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

### Equivalência — Dashboard com Gráficos

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Biblioteca | `fl_chart` | `react-native-chart-kit` | `chart.js` + `react-chartjs-2` |
| Gráfico de barras | `BarChart` | `BarChart` | `<Bar>` |
| Gráfico de linha | `LineChart` | `LineChart` (bezier) | `<Line>` (tension) |
| Gráfico de pizza | `PieChart` | `PieChart` | `<Pie>` |
| Responsividade | Container com `SizedBox` | `useWindowDimensions` | `responsive: true` (Chart.js) |
| Cards de resumo | Widget `Card` customizado | `View` com estilos | Bootstrap `card` classes |

---

## 12. QR Code — Leitor e Gerador

Leitor de QR Code pela câmera e gerador de QR Code a partir de texto digitado.

### Flutter — mobile_scanner + qr_flutter

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  mobile_scanner: ^6.0.0
  qr_flutter: ^4.1.0
```

```bash
flutter pub get
```

**2. Tela com leitor e gerador**

```dart
// lib/features/qrcode/presentation/qrcode_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:mobile_scanner/mobile_scanner.dart';
import 'package:qr_flutter/qr_flutter.dart';

class QRCodeScreen extends StatefulWidget {
  const QRCodeScreen({super.key});

  @override
  State<QRCodeScreen> createState() => _QRCodeScreenState();
}

class _QRCodeScreenState extends State<QRCodeScreen>
    with SingleTickerProviderStateMixin {
  late TabController _tabCtrl;
  final _textoCtrl = TextEditingController();
  String? _resultadoLeitura;
  String _textoQR = 'https://exemplo.com';
  MobileScannerController? _scannerCtrl;

  @override
  void initState() {
    super.initState();
    _tabCtrl = TabController(length: 2, vsync: this);
    _textoCtrl.text = _textoQR;
  }

  @override
  void dispose() {
    _tabCtrl.dispose();
    _textoCtrl.dispose();
    _scannerCtrl?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('QR Code'),
        bottom: TabBar(
          controller: _tabCtrl,
          tabs: const [
            Tab(icon: Icon(Icons.qr_code_scanner), text: 'Leitor'),
            Tab(icon: Icon(Icons.qr_code), text: 'Gerador'),
          ],
        ),
      ),
      body: TabBarView(
        controller: _tabCtrl,
        children: [
          // --- Leitor ---
          Column(
            children: [
              Expanded(
                child: MobileScanner(
                  controller: _scannerCtrl ??= MobileScannerController(),
                  onDetect: (capture) {
                    final barcodes = capture.barcodes;
                    if (barcodes.isNotEmpty && barcodes.first.rawValue != null) {
                      setState(() => _resultadoLeitura = barcodes.first.rawValue);
                      _scannerCtrl?.stop();
                    }
                  },
                ),
              ),
              if (_resultadoLeitura != null)
                Container(
                  width: double.infinity,
                  padding: const EdgeInsets.all(16),
                  color: Colors.green.shade50,
                  child: Column(
                    children: [
                      Text('Resultado:', style: Theme.of(context).textTheme.labelLarge),
                      const SizedBox(height: 4),
                      SelectableText(_resultadoLeitura!,
                          style: const TextStyle(fontSize: 16, fontWeight: FontWeight.w500)),
                      const SizedBox(height: 8),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          FilledButton.icon(
                            onPressed: () {
                              Clipboard.setData(ClipboardData(text: _resultadoLeitura!));
                              ScaffoldMessenger.of(context).showSnackBar(
                                const SnackBar(content: Text('Copiado!')),
                              );
                            },
                            icon: const Icon(Icons.copy),
                            label: const Text('Copiar'),
                          ),
                          const SizedBox(width: 12),
                          OutlinedButton.icon(
                            onPressed: () {
                              setState(() => _resultadoLeitura = null);
                              _scannerCtrl?.start();
                            },
                            icon: const Icon(Icons.refresh),
                            label: const Text('Escanear novamente'),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
            ],
          ),

          // --- Gerador ---
          SingleChildScrollView(
            padding: const EdgeInsets.all(24),
            child: Column(
              children: [
                TextField(
                  controller: _textoCtrl,
                  decoration: const InputDecoration(
                    labelText: 'Texto ou URL',
                    border: OutlineInputBorder(),
                    prefixIcon: Icon(Icons.edit),
                  ),
                  maxLines: 3,
                  onChanged: (v) => setState(() => _textoQR = v),
                ),
                const SizedBox(height: 24),
                if (_textoQR.isNotEmpty)
                  Container(
                    padding: const EdgeInsets.all(16),
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(16),
                      border: Border.all(color: Colors.grey.shade300),
                    ),
                    child: QrImageView(
                      data: _textoQR,
                      version: QrVersions.auto,
                      size: 240,
                    ),
                  ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### React Native — expo-camera + react-native-qrcode-svg

**1. Dependências**

```bash
npx expo install expo-camera
npm install react-native-qrcode-svg react-native-svg
```

**2. Tela com leitor e gerador**

```tsx
// src/features/qrcode/QRCodeScreen.tsx
import React, { useState } from 'react';
import {
  View, Text, TextInput, TouchableOpacity, StyleSheet, ScrollView,
} from 'react-native';
import { CameraView, useCameraPermissions } from 'expo-camera';
import QRCode from 'react-native-qrcode-svg';
import * as Clipboard from 'expo-clipboard';

export function QRCodeScreen() {
  const [aba, setAba] = useState<'leitor' | 'gerador'>('leitor');
  const [resultado, setResultado] = useState<string | null>(null);
  const [textoQR, setTextoQR] = useState('https://exemplo.com');
  const [permission, requestPermission] = useCameraPermissions();
  const [scanning, setScanning] = useState(true);

  const copiar = async () => {
    if (resultado) {
      await Clipboard.setStringAsync(resultado);
    }
  };

  return (
    <View style={styles.container}>
      {/* Abas */}
      <View style={styles.tabs}>
        <TouchableOpacity
          style={[styles.tab, aba === 'leitor' && styles.tabAtiva]}
          onPress={() => setAba('leitor')}
        >
          <Text style={[styles.tabTexto, aba === 'leitor' && styles.tabTextoAtivo]}>
            📷 Leitor
          </Text>
        </TouchableOpacity>
        <TouchableOpacity
          style={[styles.tab, aba === 'gerador' && styles.tabAtiva]}
          onPress={() => setAba('gerador')}
        >
          <Text style={[styles.tabTexto, aba === 'gerador' && styles.tabTextoAtivo]}>
            📱 Gerador
          </Text>
        </TouchableOpacity>
      </View>

      {aba === 'leitor' ? (
        <View style={styles.content}>
          {!permission?.granted ? (
            <View style={styles.permissao}>
              <Text style={styles.permissaoTexto}>Acesso à câmera necessário</Text>
              <TouchableOpacity style={styles.btnPermissao} onPress={requestPermission}>
                <Text style={styles.btnPermissaoTexto}>Permitir acesso</Text>
              </TouchableOpacity>
            </View>
          ) : scanning ? (
            <CameraView
              style={styles.camera}
              barcodeScannerSettings={{ barcodeTypes: ['qr'] }}
              onBarcodeScanned={(scan) => {
                setResultado(scan.data);
                setScanning(false);
              }}
            />
          ) : null}

          {resultado && (
            <View style={styles.resultado}>
              <Text style={styles.resultadoLabel}>Resultado:</Text>
              <Text style={styles.resultadoTexto} selectable>{resultado}</Text>
              <View style={styles.resultadoBtns}>
                <TouchableOpacity style={styles.btnCopiar} onPress={copiar}>
                  <Text style={styles.btnCopiarTexto}>Copiar</Text>
                </TouchableOpacity>
                <TouchableOpacity
                  style={styles.btnNovo}
                  onPress={() => { setResultado(null); setScanning(true); }}
                >
                  <Text style={styles.btnNovoTexto}>Escanear novamente</Text>
                </TouchableOpacity>
              </View>
            </View>
          )}
        </View>
      ) : (
        <ScrollView contentContainerStyle={styles.geradorContent}>
          <TextInput
            style={styles.input}
            placeholder="Texto ou URL"
            value={textoQR}
            onChangeText={setTextoQR}
            multiline
          />
          {textoQR.length > 0 && (
            <View style={styles.qrContainer}>
              <QRCode value={textoQR} size={240} />
            </View>
          )}
        </ScrollView>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  tabs: { flexDirection: 'row', borderBottomWidth: 1, borderBottomColor: '#e0e0e0' },
  tab: { flex: 1, paddingVertical: 14, alignItems: 'center' },
  tabAtiva: { borderBottomWidth: 2, borderBottomColor: '#3949ab' },
  tabTexto: { fontSize: 15, color: '#999' },
  tabTextoAtivo: { color: '#3949ab', fontWeight: '600' },
  content: { flex: 1 },
  camera: { flex: 1 },
  permissao: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  permissaoTexto: { fontSize: 16, marginBottom: 16, color: '#666' },
  btnPermissao: { backgroundColor: '#3949ab', paddingHorizontal: 24, paddingVertical: 12, borderRadius: 8 },
  btnPermissaoTexto: { color: '#fff', fontWeight: '600' },
  resultado: { padding: 16, backgroundColor: '#e8f5e9' },
  resultadoLabel: { fontWeight: '600', marginBottom: 4 },
  resultadoTexto: { fontSize: 16, marginBottom: 12 },
  resultadoBtns: { flexDirection: 'row', gap: 12 },
  btnCopiar: { backgroundColor: '#3949ab', paddingHorizontal: 16, paddingVertical: 10, borderRadius: 8 },
  btnCopiarTexto: { color: '#fff', fontWeight: '600' },
  btnNovo: { borderWidth: 1, borderColor: '#3949ab', paddingHorizontal: 16, paddingVertical: 10, borderRadius: 8 },
  btnNovoTexto: { color: '#3949ab' },
  geradorContent: { padding: 24, alignItems: 'center' },
  input: { borderWidth: 1, borderColor: '#ddd', borderRadius: 8, padding: 12, fontSize: 15, width: '100%', marginBottom: 24 },
  qrContainer: { padding: 24, backgroundColor: '#fff', borderRadius: 16, borderWidth: 1, borderColor: '#e0e0e0' },
});
```

---

### React Web — html5-qrcode + qrcode

**1. Dependências**

```bash
npm install html5-qrcode qrcode
npm install -D @types/qrcode
```

**2. Página QR Code**

```tsx
// src/features/qrcode/QRCodePage.tsx
import { useEffect, useRef, useState } from 'react';
import { Html5Qrcode } from 'html5-qrcode';
import QRCode from 'qrcode';

export function QRCodePage() {
  const [aba, setAba] = useState<'leitor' | 'gerador'>('leitor');
  const [resultado, setResultado] = useState<string | null>(null);
  const [textoQR, setTextoQR] = useState('https://exemplo.com');
  const [qrImgUrl, setQrImgUrl] = useState('');
  const [scanning, setScanning] = useState(false);
  const scannerRef = useRef<Html5Qrcode | null>(null);
  const readerId = 'qr-reader';

  // Gerar QR
  useEffect(() => {
    if (textoQR) {
      QRCode.toDataURL(textoQR, { width: 256, margin: 2 }).then(setQrImgUrl);
    }
  }, [textoQR]);

  const iniciarScanner = async () => {
    setResultado(null);
    setScanning(true);
    const scanner = new Html5Qrcode(readerId);
    scannerRef.current = scanner;

    try {
      await scanner.start(
        { facingMode: 'environment' },
        { fps: 10, qrbox: 250 },
        (decodedText) => {
          setResultado(decodedText);
          scanner.stop().then(() => setScanning(false));
        },
        () => {},
      );
    } catch {
      setScanning(false);
      alert('Não foi possível acessar a câmera.');
    }
  };

  const pararScanner = () => {
    scannerRef.current?.stop().then(() => setScanning(false));
  };

  const copiar = () => {
    if (resultado) navigator.clipboard.writeText(resultado);
  };

  return (
    <div className="container py-4" style={{ maxWidth: 600 }}>
      {/* Abas */}
      <ul className="nav nav-tabs mb-4">
        <li className="nav-item">
          <button
            className={`nav-link ${aba === 'leitor' ? 'active' : ''}`}
            onClick={() => { setAba('leitor'); pararScanner(); }}
          >
            <i className="fas fa-qrcode me-1" /> Leitor
          </button>
        </li>
        <li className="nav-item">
          <button
            className={`nav-link ${aba === 'gerador' ? 'active' : ''}`}
            onClick={() => { setAba('gerador'); pararScanner(); }}
          >
            <i className="fas fa-pen me-1" /> Gerador
          </button>
        </li>
      </ul>

      {aba === 'leitor' ? (
        <>
          <div id={readerId} className="mb-3 rounded overflow-hidden" />

          {!scanning && !resultado && (
            <button className="btn btn-primary w-100" onClick={iniciarScanner}>
              <i className="fas fa-camera me-2" />Iniciar scanner
            </button>
          )}

          {scanning && (
            <button className="btn btn-danger w-100" onClick={pararScanner}>
              <i className="fas fa-stop me-2" />Parar
            </button>
          )}

          {resultado && (
            <div className="alert alert-success">
              <strong>Resultado:</strong>
              <p className="mb-2 user-select-all">{resultado}</p>
              <div className="d-flex gap-2">
                <button className="btn btn-sm btn-primary" onClick={copiar}>
                  <i className="fas fa-copy me-1" />Copiar
                </button>
                <button className="btn btn-sm btn-outline-primary" onClick={iniciarScanner}>
                  <i className="fas fa-redo me-1" />Escanear novamente
                </button>
              </div>
            </div>
          )}
        </>
      ) : (
        <>
          <div className="mb-3">
            <label className="form-label">Texto ou URL</label>
            <textarea
              className="form-control"
              rows={3}
              value={textoQR}
              onChange={(e) => setTextoQR(e.target.value)}
            />
          </div>
          {qrImgUrl && (
            <div className="text-center">
              <div className="d-inline-block p-3 border rounded-3">
                <img src={qrImgUrl} alt="QR Code" />
              </div>
              <div className="mt-3">
                <a href={qrImgUrl} download="qrcode.png" className="btn btn-outline-primary btn-sm">
                  <i className="fas fa-download me-1" />Baixar PNG
                </a>
              </div>
            </div>
          )}
        </>
      )}
    </div>
  );
}
```

---

### Equivalência — QR Code

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Leitor | `mobile_scanner` | `expo-camera` (barcode) | `html5-qrcode` |
| Gerador | `qr_flutter` (`QrImageView`) | `react-native-qrcode-svg` | `qrcode` (canvas → DataURL) |
| Permissão de câmera | `AndroidManifest` + `Info.plist` | `useCameraPermissions` | Prompt automático |
| Copiar resultado | `Clipboard.setData` | `expo-clipboard` | `navigator.clipboard.writeText` |
| Download do QR | — (salvar via share) | — (salvar via share) | `<a download>` |

---

## 13. Armazenamento Offline e Sincronização

Armazenamento local de dados com fila de sincronização quando a conexão é restaurada.

### Flutter — sqflite + connectivity_plus

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  sqflite: ^2.4.0
  path: ^1.9.0
  connectivity_plus: ^6.1.0
```

```bash
flutter pub get
```

**2. Banco de dados local**

```dart
// lib/core/database/local_database.dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class LocalDatabase {
  static Database? _db;

  Future<Database> get database async {
    _db ??= await _initDb();
    return _db!;
  }

  Future<Database> _initDb() async {
    final path = join(await getDatabasesPath(), 'app.db');
    return openDatabase(
      path,
      version: 1,
      onCreate: (db, version) async {
        await db.execute('''
          CREATE TABLE notas (
            id TEXT PRIMARY KEY,
            titulo TEXT NOT NULL,
            conteudo TEXT NOT NULL,
            atualizado_em INTEGER NOT NULL,
            sincronizado INTEGER NOT NULL DEFAULT 0
          )
        ''');
        await db.execute('''
          CREATE TABLE fila_sync (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            tabela TEXT NOT NULL,
            registro_id TEXT NOT NULL,
            operacao TEXT NOT NULL,
            dados TEXT,
            criado_em INTEGER NOT NULL
          )
        ''');
      },
    );
  }
}
```

**3. Repositório com fila de sync**

```dart
// lib/features/notas/data/nota_repository.dart
import 'dart:convert';
import '../../../core/database/local_database.dart';

class Nota {
  final String id;
  String titulo;
  String conteudo;
  DateTime atualizadoEm;
  bool sincronizado;

  Nota({
    required this.id,
    required this.titulo,
    required this.conteudo,
    required this.atualizadoEm,
    this.sincronizado = false,
  });

  Map<String, dynamic> toMap() => {
    'id': id, 'titulo': titulo, 'conteudo': conteudo,
    'atualizado_em': atualizadoEm.millisecondsSinceEpoch,
    'sincronizado': sincronizado ? 1 : 0,
  };

  factory Nota.fromMap(Map<String, dynamic> map) => Nota(
    id: map['id'], titulo: map['titulo'], conteudo: map['conteudo'],
    atualizadoEm: DateTime.fromMillisecondsSinceEpoch(map['atualizado_em']),
    sincronizado: map['sincronizado'] == 1,
  );
}

class NotaRepository {
  final _localDb = LocalDatabase();

  Future<List<Nota>> listar() async {
    final db = await _localDb.database;
    final maps = await db.query('notas', orderBy: 'atualizado_em DESC');
    return maps.map(Nota.fromMap).toList();
  }

  Future<void> salvar(Nota nota) async {
    final db = await _localDb.database;
    await db.insert('notas', nota.toMap(),
        conflictAlgorithm: ConflictAlgorithm.replace);

    await db.insert('fila_sync', {
      'tabela': 'notas',
      'registro_id': nota.id,
      'operacao': 'upsert',
      'dados': jsonEncode(nota.toMap()),
      'criado_em': DateTime.now().millisecondsSinceEpoch,
    });
  }

  Future<void> excluir(String id) async {
    final db = await _localDb.database;
    await db.delete('notas', where: 'id = ?', whereArgs: [id]);

    await db.insert('fila_sync', {
      'tabela': 'notas',
      'registro_id': id,
      'operacao': 'delete',
      'dados': null,
      'criado_em': DateTime.now().millisecondsSinceEpoch,
    });
  }

  Future<int> pendentesSync() async {
    final db = await _localDb.database;
    final result = await db.rawQuery('SELECT COUNT(*) as total FROM fila_sync');
    return Sqflite.firstIntValue(result) ?? 0;
  }
}
```

**4. Serviço de sincronização**

```dart
// lib/core/sync/sync_service.dart
import 'dart:async';
import 'package:connectivity_plus/connectivity_plus.dart';
import '../database/local_database.dart';

class SyncService {
  final _localDb = LocalDatabase();
  final _connectivity = Connectivity();
  StreamSubscription? _connectivitySub;
  final _statusCtrl = StreamController<SyncStatus>.broadcast();

  Stream<SyncStatus> get status => _statusCtrl.stream;

  void iniciar() {
    _connectivitySub = _connectivity.onConnectivityChanged.listen(
      (results) {
        final conectado = results.any((r) => r != ConnectivityResult.none);
        if (conectado) sincronizar();
      },
    );
  }

  Future<void> sincronizar() async {
    final db = await _localDb.database;
    final pendentes = await db.query('fila_sync', orderBy: 'criado_em ASC');

    if (pendentes.isEmpty) {
      _statusCtrl.add(SyncStatus(pendentes: 0, sincronizando: false));
      return;
    }

    _statusCtrl.add(SyncStatus(pendentes: pendentes.length, sincronizando: true));

    for (final item in pendentes) {
      try {
        // Simula envio ao servidor
        await Future.delayed(const Duration(milliseconds: 200));

        // Marcar registro como sincronizado
        if (item['operacao'] == 'upsert') {
          await db.update(
            'notas',
            {'sincronizado': 1},
            where: 'id = ?',
            whereArgs: [item['registro_id']],
          );
        }

        // Remover da fila
        await db.delete('fila_sync', where: 'id = ?', whereArgs: [item['id']]);
      } catch (_) {
        break;
      }
    }

    final restantes = await db.rawQuery('SELECT COUNT(*) as total FROM fila_sync');
    _statusCtrl.add(SyncStatus(
      pendentes: Sqflite.firstIntValue(restantes) ?? 0,
      sincronizando: false,
    ));
  }

  void parar() {
    _connectivitySub?.cancel();
    _statusCtrl.close();
  }
}

class SyncStatus {
  final int pendentes;
  final bool sincronizando;
  SyncStatus({required this.pendentes, required this.sincronizando});
}
```

---

### React Native — expo-sqlite + NetInfo

**1. Dependências**

```bash
npx expo install expo-sqlite @react-native-community/netinfo
```

**2. Banco de dados e repositório**

```typescript
// src/core/database/localDb.ts
import * as SQLite from 'expo-sqlite';

let db: SQLite.SQLiteDatabase;

export async function getDb(): Promise<SQLite.SQLiteDatabase> {
  if (!db) {
    db = await SQLite.openDatabaseAsync('app.db');
    await db.execAsync(`
      CREATE TABLE IF NOT EXISTS notas (
        id TEXT PRIMARY KEY, titulo TEXT NOT NULL, conteudo TEXT NOT NULL,
        atualizado_em INTEGER NOT NULL, sincronizado INTEGER NOT NULL DEFAULT 0
      );
      CREATE TABLE IF NOT EXISTS fila_sync (
        id INTEGER PRIMARY KEY AUTOINCREMENT, tabela TEXT NOT NULL,
        registro_id TEXT NOT NULL, operacao TEXT NOT NULL,
        dados TEXT, criado_em INTEGER NOT NULL
      );
    `);
  }
  return db;
}
```

```typescript
// src/features/notas/notaRepository.ts
import { getDb } from '../../core/database/localDb';

export interface Nota {
  id: string;
  titulo: string;
  conteudo: string;
  atualizadoEm: number;
  sincronizado: boolean;
}

export async function listarNotas(): Promise<Nota[]> {
  const db = await getDb();
  const rows = await db.getAllAsync<{
    id: string; titulo: string; conteudo: string;
    atualizado_em: number; sincronizado: number;
  }>('SELECT * FROM notas ORDER BY atualizado_em DESC');

  return rows.map((r) => ({
    id: r.id, titulo: r.titulo, conteudo: r.conteudo,
    atualizadoEm: r.atualizado_em, sincronizado: r.sincronizado === 1,
  }));
}

export async function salvarNota(nota: Nota) {
  const db = await getDb();
  await db.runAsync(
    'INSERT OR REPLACE INTO notas (id, titulo, conteudo, atualizado_em, sincronizado) VALUES (?, ?, ?, ?, 0)',
    [nota.id, nota.titulo, nota.conteudo, nota.atualizadoEm],
  );
  await db.runAsync(
    'INSERT INTO fila_sync (tabela, registro_id, operacao, dados, criado_em) VALUES (?, ?, ?, ?, ?)',
    ['notas', nota.id, 'upsert', JSON.stringify(nota), Date.now()],
  );
}

export async function excluirNota(id: string) {
  const db = await getDb();
  await db.runAsync('DELETE FROM notas WHERE id = ?', [id]);
  await db.runAsync(
    'INSERT INTO fila_sync (tabela, registro_id, operacao, dados, criado_em) VALUES (?, ?, ?, ?, ?)',
    ['notas', id, 'delete', null, Date.now()],
  );
}

export async function contarPendentes(): Promise<number> {
  const db = await getDb();
  const result = await db.getFirstAsync<{ total: number }>('SELECT COUNT(*) as total FROM fila_sync');
  return result?.total ?? 0;
}
```

**3. Hook de sincronização**

```typescript
// src/hooks/useSync.ts
import { useEffect, useState, useCallback } from 'react';
import NetInfo from '@react-native-community/netinfo';
import { getDb } from '../core/database/localDb';

export function useSync() {
  const [pendentes, setPendentes] = useState(0);
  const [sincronizando, setSincronizando] = useState(false);

  const sincronizar = useCallback(async () => {
    const db = await getDb();
    const fila = await db.getAllAsync<{ id: number; registro_id: string; operacao: string }>(
      'SELECT * FROM fila_sync ORDER BY criado_em ASC',
    );

    if (fila.length === 0) { setPendentes(0); return; }
    setSincronizando(true);

    for (const item of fila) {
      try {
        await new Promise((r) => setTimeout(r, 200)); // simula envio
        if (item.operacao === 'upsert') {
          await db.runAsync('UPDATE notas SET sincronizado = 1 WHERE id = ?', [item.registro_id]);
        }
        await db.runAsync('DELETE FROM fila_sync WHERE id = ?', [item.id]);
      } catch { break; }
    }

    const result = await db.getFirstAsync<{ total: number }>('SELECT COUNT(*) as total FROM fila_sync');
    setPendentes(result?.total ?? 0);
    setSincronizando(false);
  }, []);

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      if (state.isConnected) sincronizar();
    });
    return unsubscribe;
  }, [sincronizar]);

  return { pendentes, sincronizando, sincronizar };
}
```

---

### React Web — IndexedDB + navigator.onLine

**1. Banco IndexedDB**

```typescript
// src/core/database/localDb.ts
const DB_NAME = 'app_db';
const DB_VERSION = 1;

function abrirDb(): Promise<IDBDatabase> {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open(DB_NAME, DB_VERSION);
    request.onerror = () => reject(request.error);
    request.onsuccess = () => resolve(request.result);
    request.onupgradeneeded = () => {
      const db = request.result;
      if (!db.objectStoreNames.contains('notas')) {
        db.createObjectStore('notas', { keyPath: 'id' });
      }
      if (!db.objectStoreNames.contains('fila_sync')) {
        db.createObjectStore('fila_sync', { keyPath: 'id', autoIncrement: true });
      }
    };
  });
}

export async function listarNotas(): Promise<Nota[]> {
  const db = await abrirDb();
  return new Promise((resolve) => {
    const tx = db.transaction('notas', 'readonly');
    const store = tx.objectStore('notas');
    const request = store.getAll();
    request.onsuccess = () => {
      const notas = (request.result as Nota[]).sort((a, b) => b.atualizadoEm - a.atualizadoEm);
      resolve(notas);
    };
  });
}

export interface Nota {
  id: string;
  titulo: string;
  conteudo: string;
  atualizadoEm: number;
  sincronizado: boolean;
}

export async function salvarNota(nota: Nota) {
  const db = await abrirDb();
  const tx = db.transaction(['notas', 'fila_sync'], 'readwrite');
  tx.objectStore('notas').put({ ...nota, sincronizado: false });
  tx.objectStore('fila_sync').add({
    tabela: 'notas', registroId: nota.id, operacao: 'upsert',
    dados: JSON.stringify(nota), criadoEm: Date.now(),
  });
}

export async function excluirNota(id: string) {
  const db = await abrirDb();
  const tx = db.transaction(['notas', 'fila_sync'], 'readwrite');
  tx.objectStore('notas').delete(id);
  tx.objectStore('fila_sync').add({
    tabela: 'notas', registroId: id, operacao: 'delete',
    dados: null, criadoEm: Date.now(),
  });
}

export async function contarPendentes(): Promise<number> {
  const db = await abrirDb();
  return new Promise((resolve) => {
    const tx = db.transaction('fila_sync', 'readonly');
    const request = tx.objectStore('fila_sync').count();
    request.onsuccess = () => resolve(request.result);
  });
}

export async function processarFila(): Promise<number> {
  const db = await abrirDb();
  const tx = db.transaction('fila_sync', 'readonly');
  const todos: { id: number; registroId: string; operacao: string }[] =
    await new Promise((resolve) => {
      const req = tx.objectStore('fila_sync').getAll();
      req.onsuccess = () => resolve(req.result);
    });

  for (const item of todos) {
    try {
      await new Promise((r) => setTimeout(r, 200)); // simula envio
      const tx2 = db.transaction(['notas', 'fila_sync'], 'readwrite');
      if (item.operacao === 'upsert') {
        const nota = await new Promise<Nota | undefined>((resolve) => {
          const req = tx2.objectStore('notas').get(item.registroId);
          req.onsuccess = () => resolve(req.result);
        });
        if (nota) tx2.objectStore('notas').put({ ...nota, sincronizado: true });
      }
      tx2.objectStore('fila_sync').delete(item.id);
    } catch { break; }
  }

  return contarPendentes();
}
```

**2. Hook de sincronização**

```typescript
// src/hooks/useSync.ts
import { useEffect, useState, useCallback } from 'react';
import { processarFila, contarPendentes } from '../core/database/localDb';

export function useSync() {
  const [pendentes, setPendentes] = useState(0);
  const [sincronizando, setSincronizando] = useState(false);

  const sincronizar = useCallback(async () => {
    setSincronizando(true);
    const restantes = await processarFila();
    setPendentes(restantes);
    setSincronizando(false);
  }, []);

  useEffect(() => {
    contarPendentes().then(setPendentes);

    const handler = () => {
      if (navigator.onLine) sincronizar();
    };
    window.addEventListener('online', handler);
    return () => window.removeEventListener('online', handler);
  }, [sincronizar]);

  return { pendentes, sincronizando, sincronizar };
}
```

**3. Indicador de status (componente)**

```tsx
// src/components/SyncIndicator.tsx
import { useSync } from '../hooks/useSync';

export function SyncIndicator() {
  const { pendentes, sincronizando, sincronizar } = useSync();

  if (pendentes === 0 && !sincronizando) return null;

  return (
    <div className="d-flex align-items-center gap-2 px-3 py-2 bg-warning-subtle border-bottom">
      {sincronizando ? (
        <>
          <span className="spinner-border spinner-border-sm" />
          <small>Sincronizando…</small>
        </>
      ) : (
        <>
          <i className="fas fa-exclamation-triangle text-warning" />
          <small>{pendentes} alteração(ões) pendente(s)</small>
          <button className="btn btn-sm btn-link p-0 ms-2" onClick={sincronizar}>
            Sincronizar agora
          </button>
        </>
      )}
    </div>
  );
}
```

---

### Equivalência — Armazenamento Offline

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Banco local | `sqflite` (SQLite) | `expo-sqlite` | IndexedDB |
| Fila de sync | Tabela `fila_sync` | Tabela `fila_sync` | Object store `fila_sync` |
| Detecção de conectividade | `connectivity_plus` | `@react-native-community/netinfo` | `navigator.onLine` + evento `online` |
| Sync automática | `onConnectivityChanged` | `NetInfo.addEventListener` | `window.addEventListener('online')` |
| Conflito | Timestamp mais recente | Timestamp mais recente | Timestamp mais recente |
| Indicador de pendências | Widget customizado | Componente customizado | Bootstrap `alert` |

---

## 14. Internacionalização (i18n)

Suporte a múltiplos idiomas com:

- Troca de idioma em tempo de execução.
- Formatação de datas, números e moedas conforme o locale.
- Detecção do idioma preferido do sistema como padrão.

### Flutter — flutter_localizations + intl

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_localizations:
    sdk: flutter
  intl: ^0.19.0
  shared_preferences: ^2.3.0

flutter:
  generate: true
```

```bash
flutter pub get
```

**2. Configuração — `l10n.yaml`**

```yaml
# l10n.yaml (na raiz do projeto)
arb-dir: lib/l10n
template-arb-file: app_pt.arb
output-localization-file: app_localizations.dart
```

**3. Arquivos de tradução**

```json
// lib/l10n/app_pt.arb
{
  "@@locale": "pt",
  "appTitle": "MeuApp",
  "home": "Início",
  "settings": "Configurações",
  "language": "Idioma",
  "welcome": "Bem-vindo, {nome}!",
  "@welcome": {
    "placeholders": {
      "nome": { "type": "String" }
    }
  },
  "itemCount": "{count, plural, =0{Nenhum item} =1{1 item} other{{count} itens}}",
  "@itemCount": {
    "placeholders": {
      "count": { "type": "int" }
    }
  },
  "price": "Preço: {valor}",
  "@price": {
    "placeholders": {
      "valor": { "type": "double", "format": "currency", "optionalParameters": { "symbol": "R$" } }
    }
  },
  "lastUpdate": "Última atualização: {data}",
  "@lastUpdate": {
    "placeholders": {
      "data": { "type": "DateTime", "format": "yMd" }
    }
  }
}
```

```json
// lib/l10n/app_en.arb
{
  "@@locale": "en",
  "appTitle": "MyApp",
  "home": "Home",
  "settings": "Settings",
  "language": "Language",
  "welcome": "Welcome, {nome}!",
  "itemCount": "{count, plural, =0{No items} =1{1 item} other{{count} items}}",
  "price": "Price: {valor}",
  "lastUpdate": "Last update: {data}"
}
```

```json
// lib/l10n/app_es.arb
{
  "@@locale": "es",
  "appTitle": "MiApp",
  "home": "Inicio",
  "settings": "Configuración",
  "language": "Idioma",
  "welcome": "¡Bienvenido, {nome}!",
  "itemCount": "{count, plural, =0{Ningún elemento} =1{1 elemento} other{{count} elementos}}",
  "price": "Precio: {valor}",
  "lastUpdate": "Última actualización: {data}"
}
```

**4. Provider de locale**

```dart
// lib/core/i18n/locale_provider.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';

class LocaleNotifier extends Notifier<Locale?> {
  static const _key = 'app_locale';

  @override
  Locale? build() {
    _carregar();
    return null; // null = usar locale do sistema
  }

  Future<void> _carregar() async {
    final prefs = await SharedPreferences.getInstance();
    final salvo = prefs.getString(_key);
    if (salvo != null) state = Locale(salvo);
  }

  Future<void> setLocale(Locale? locale) async {
    state = locale;
    final prefs = await SharedPreferences.getInstance();
    if (locale != null) {
      await prefs.setString(_key, locale.languageCode);
    } else {
      await prefs.remove(_key);
    }
  }
}

final localeProvider = NotifierProvider<LocaleNotifier, Locale?>(
  LocaleNotifier.new,
);
```

**5. main.dart com i18n**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import 'core/i18n/locale_provider.dart';

void main() {
  runApp(const ProviderScope(child: App()));
}

class App extends ConsumerWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final locale = ref.watch(localeProvider);

    return MaterialApp(
      title: 'MeuApp',
      locale: locale,
      localizationsDelegates: const [
        AppLocalizations.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      supportedLocales: AppLocalizations.supportedLocales,
      home: const HomeScreen(),
    );
  }
}
```

**6. Seletor de idioma**

```dart
// lib/core/i18n/language_selector.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import 'locale_provider.dart';

class LanguageSelector extends ConsumerWidget {
  const LanguageSelector({super.key});

  static const _idiomas = [
    (locale: null, nome: 'Sistema', bandeira: '🌐'),
    (locale: Locale('pt'), nome: 'Português', bandeira: '🇧🇷'),
    (locale: Locale('en'), nome: 'English', bandeira: '🇺🇸'),
    (locale: Locale('es'), nome: 'Español', bandeira: '🇪🇸'),
  ];

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final localeAtual = ref.watch(localeProvider);
    final l10n = AppLocalizations.of(context)!;

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(l10n.language, style: Theme.of(context).textTheme.titleMedium),
        const SizedBox(height: 12),
        ..._idiomas.map((item) {
          final selecionado = localeAtual?.languageCode == item.locale?.languageCode
              && (item.locale != null || localeAtual == null);
          return ListTile(
            leading: Text(item.bandeira, style: const TextStyle(fontSize: 24)),
            title: Text(item.nome),
            trailing: selecionado ? const Icon(Icons.check, color: Colors.green) : null,
            selected: selecionado,
            onTap: () => ref.read(localeProvider.notifier).setLocale(item.locale),
          );
        }),
      ],
    );
  }
}
```

**7. Uso nas telas**

```dart
// Qualquer widget:
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

final l10n = AppLocalizations.of(context)!;
Text(l10n.welcome('João'));
Text(l10n.itemCount(42));
Text(l10n.price(29.90));
Text(l10n.lastUpdate(DateTime.now()));
```

---

### React Native — expo-localization + i18next

**1. Dependências**

```bash
npx expo install expo-localization
npm install i18next react-i18next
```

**2. Arquivos de tradução**

```typescript
// src/i18n/locales/pt.ts
export default {
  appTitle: 'MeuApp',
  home: 'Início',
  settings: 'Configurações',
  language: 'Idioma',
  welcome: 'Bem-vindo, {{nome}}!',
  itemCount_zero: 'Nenhum item',
  itemCount_one: '1 item',
  itemCount_other: '{{count}} itens',
  price: 'Preço: {{valor}}',
  lastUpdate: 'Última atualização: {{data}}',
};

// src/i18n/locales/en.ts
export default {
  appTitle: 'MyApp',
  home: 'Home',
  settings: 'Settings',
  language: 'Language',
  welcome: 'Welcome, {{nome}}!',
  itemCount_zero: 'No items',
  itemCount_one: '1 item',
  itemCount_other: '{{count}} items',
  price: 'Price: {{valor}}',
  lastUpdate: 'Last update: {{data}}',
};

// src/i18n/locales/es.ts
export default {
  appTitle: 'MiApp',
  home: 'Inicio',
  settings: 'Configuración',
  language: 'Idioma',
  welcome: '¡Bienvenido, {{nome}}!',
  itemCount_zero: 'Ningún elemento',
  itemCount_one: '1 elemento',
  itemCount_other: '{{count}} elementos',
  price: 'Precio: {{valor}}',
  lastUpdate: 'Última actualización: {{data}}',
};
```

**3. Configuração do i18next**

```typescript
// src/i18n/i18n.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import { getLocales } from 'expo-localization';
import AsyncStorage from '@react-native-async-storage/async-storage';
import pt from './locales/pt';
import en from './locales/en';
import es from './locales/es';

const STORAGE_KEY = 'app_language';

async function getStoredLanguage(): Promise<string | null> {
  return AsyncStorage.getItem(STORAGE_KEY);
}

export async function initI18n() {
  const stored = await getStoredLanguage();
  const systemLocale = getLocales()[0]?.languageCode ?? 'pt';

  await i18n.use(initReactI18next).init({
    resources: {
      pt: { translation: pt },
      en: { translation: en },
      es: { translation: es },
    },
    lng: stored ?? systemLocale,
    fallbackLng: 'pt',
    interpolation: { escapeValue: false },
  });
}

export async function changeLanguage(lng: string) {
  await i18n.changeLanguage(lng);
  await AsyncStorage.setItem(STORAGE_KEY, lng);
}

export default i18n;
```

**4. Formatação de datas e moedas por locale**

```typescript
// src/i18n/formatters.ts
export function formatarMoeda(valor: number, locale: string): string {
  const config: Record<string, { locale: string; currency: string }> = {
    pt: { locale: 'pt-BR', currency: 'BRL' },
    en: { locale: 'en-US', currency: 'USD' },
    es: { locale: 'es-ES', currency: 'EUR' },
  };
  const c = config[locale] ?? config.pt;
  return new Intl.NumberFormat(c.locale, { style: 'currency', currency: c.currency }).format(valor);
}

export function formatarData(data: Date, locale: string): string {
  const localeMap: Record<string, string> = { pt: 'pt-BR', en: 'en-US', es: 'es-ES' };
  return new Intl.DateTimeFormat(localeMap[locale] ?? 'pt-BR', { dateStyle: 'medium' }).format(data);
}
```

**5. Seletor de idioma**

```tsx
// src/components/LanguageSelector.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useTranslation } from 'react-i18next';
import { changeLanguage } from '../i18n/i18n';

const IDIOMAS = [
  { codigo: 'pt', nome: 'Português', bandeira: '🇧🇷' },
  { codigo: 'en', nome: 'English', bandeira: '🇺🇸' },
  { codigo: 'es', nome: 'Español', bandeira: '🇪🇸' },
];

export function LanguageSelector() {
  const { t, i18n } = useTranslation();

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>{t('language')}</Text>
      {IDIOMAS.map((idioma) => {
        const selecionado = i18n.language === idioma.codigo;
        return (
          <TouchableOpacity
            key={idioma.codigo}
            style={[styles.item, selecionado && styles.itemSelecionado]}
            onPress={() => changeLanguage(idioma.codigo)}
          >
            <Text style={styles.bandeira}>{idioma.bandeira}</Text>
            <Text style={styles.nome}>{idioma.nome}</Text>
            {selecionado && <Text style={styles.check}>✓</Text>}
          </TouchableOpacity>
        );
      })}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 16 },
  titulo: { fontSize: 16, fontWeight: '600', marginBottom: 12 },
  item: { flexDirection: 'row', alignItems: 'center', paddingVertical: 14, paddingHorizontal: 12, borderRadius: 8, marginBottom: 4 },
  itemSelecionado: { backgroundColor: '#e8eaf6' },
  bandeira: { fontSize: 24, marginRight: 12 },
  nome: { flex: 1, fontSize: 15 },
  check: { color: '#4caf50', fontSize: 18, fontWeight: '700' },
});
```

---

### React Web — i18next + react-i18next + Bootstrap

**1. Dependências**

```bash
npm install i18next react-i18next i18next-browser-languagedetector
```

**2. Arquivos de tradução** (mesma estrutura do React Native)

```typescript
// src/i18n/locales/pt.ts
export default {
  appTitle: 'MeuApp',
  home: 'Início',
  settings: 'Configurações',
  language: 'Idioma',
  welcome: 'Bem-vindo, {{nome}}!',
  itemCount_zero: 'Nenhum item',
  itemCount_one: '1 item',
  itemCount_other: '{{count}} itens',
  price: 'Preço: {{valor}}',
  lastUpdate: 'Última atualização: {{data}}',
};

// src/i18n/locales/en.ts
export default {
  appTitle: 'MyApp',
  home: 'Home',
  settings: 'Settings',
  language: 'Language',
  welcome: 'Welcome, {{nome}}!',
  itemCount_zero: 'No items',
  itemCount_one: '1 item',
  itemCount_other: '{{count}} items',
  price: 'Price: {{valor}}',
  lastUpdate: 'Last update: {{data}}',
};

// src/i18n/locales/es.ts
export default {
  appTitle: 'MiApp',
  home: 'Inicio',
  settings: 'Configuración',
  language: 'Idioma',
  welcome: '¡Bienvenido, {{nome}}!',
  itemCount_zero: 'Ningún elemento',
  itemCount_one: '1 elemento',
  itemCount_other: '{{count}} elementos',
  price: 'Precio: {{valor}}',
  lastUpdate: 'Última actualización: {{data}}',
};
```

**3. Configuração do i18next**

```typescript
// src/i18n/i18n.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import pt from './locales/pt';
import en from './locales/en';
import es from './locales/es';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      pt: { translation: pt },
      en: { translation: en },
      es: { translation: es },
    },
    fallbackLng: 'pt',
    interpolation: { escapeValue: false },
    detection: {
      order: ['localStorage', 'navigator'],
      caches: ['localStorage'],
    },
  });

export default i18n;
```

**4. Formatação por locale**

```typescript
// src/i18n/formatters.ts
export function formatarMoeda(valor: number, locale: string): string {
  const config: Record<string, { locale: string; currency: string }> = {
    pt: { locale: 'pt-BR', currency: 'BRL' },
    en: { locale: 'en-US', currency: 'USD' },
    es: { locale: 'es-ES', currency: 'EUR' },
  };
  const c = config[locale] ?? config.pt;
  return new Intl.NumberFormat(c.locale, { style: 'currency', currency: c.currency }).format(valor);
}

export function formatarData(data: Date, locale: string): string {
  const localeMap: Record<string, string> = { pt: 'pt-BR', en: 'en-US', es: 'es-ES' };
  return new Intl.DateTimeFormat(localeMap[locale] ?? 'pt-BR', { dateStyle: 'medium' }).format(data);
}
```

**5. Importar no main.tsx**

```tsx
// src/main.tsx (adicionar antes dos outros imports)
import './i18n/i18n';
```

**6. Seletor de idioma**

```tsx
// src/components/LanguageSelector.tsx
import { useTranslation } from 'react-i18next';

const IDIOMAS = [
  { codigo: 'pt', nome: 'Português', bandeira: '🇧🇷' },
  { codigo: 'en', nome: 'English', bandeira: '🇺🇸' },
  { codigo: 'es', nome: 'Español', bandeira: '🇪🇸' },
];

export function LanguageSelector() {
  const { t, i18n } = useTranslation();

  return (
    <div>
      <h6 className="mb-3">{t('language')}</h6>
      <div className="list-group">
        {IDIOMAS.map((idioma) => {
          const selecionado = i18n.language.startsWith(idioma.codigo);
          return (
            <button
              key={idioma.codigo}
              type="button"
              className={`list-group-item list-group-item-action d-flex align-items-center gap-3 ${
                selecionado ? 'active' : ''
              }`}
              onClick={() => i18n.changeLanguage(idioma.codigo)}
            >
              <span style={{ fontSize: '1.5rem' }}>{idioma.bandeira}</span>
              <span className="flex-grow-1">{idioma.nome}</span>
              {selecionado && <i className="fas fa-check" />}
            </button>
          );
        })}
      </div>
    </div>
  );
}
```

**7. Uso nos componentes**

```tsx
import { useTranslation } from 'react-i18next';
import { formatarMoeda, formatarData } from '../i18n/formatters';

function MeuComponente() {
  const { t, i18n } = useTranslation();
  const lang = i18n.language;

  return (
    <>
      <h1>{t('welcome', { nome: 'João' })}</h1>
      <p>{t('itemCount', { count: 42 })}</p>
      <p>{formatarMoeda(29.9, lang)}</p>
      <p>{formatarData(new Date(), lang)}</p>
    </>
  );
}
```

---

### Equivalência — Internacionalização

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Framework i18n | `flutter_localizations` + `intl` | `i18next` + `react-i18next` | `i18next` + `react-i18next` |
| Arquivos de tradução | `.arb` (JSON com metadados) | `.ts` (objetos exportados) | `.ts` (objetos exportados) |
| Geração de código | `flutter gen-l10n` (automática) | Manual (importação) | Manual (importação) |
| Detecção do sistema | `Locale` do `MaterialApp` | `expo-localization` | `i18next-browser-languagedetector` |
| Troca em runtime | `MaterialApp.locale` via provider | `i18n.changeLanguage` | `i18n.changeLanguage` |
| Persistência | `shared_preferences` | `AsyncStorage` | `localStorage` (via detector) |
| Pluralização | ICU syntax no `.arb` | Sufixos `_zero`, `_one`, `_other` | Sufixos `_zero`, `_one`, `_other` |
| Formatação moeda | `NumberFormat.currency` (intl) | `Intl.NumberFormat` | `Intl.NumberFormat` |
| Formatação data | `DateFormat` (intl) | `Intl.DateTimeFormat` | `Intl.DateTimeFormat` |
| Acesso às strings | `AppLocalizations.of(context)` | `useTranslation().t()` | `useTranslation().t()` |

---

## 15. Autenticação Biométrica

Integração com biometria do dispositivo (impressão digital / Face ID) para:

- Verificar disponibilidade de biometria no dispositivo.
- Solicitar autenticação biométrica com fallback para PIN/senha do sistema.
- Proteger acesso a telas ou ações sensíveis (ex: confirmar pagamento, acessar dados pessoais).
- Tela de bloqueio biométrico ao abrir o app ou ao retornar do background.

> **Importante:** a biometria autentica o **usuário no dispositivo** — ela confirma que quem está usando é o dono do aparelho, mas não substitui o login no servidor. O fluxo típico é: login via credenciais/OAuth na primeira vez → armazenar token seguro → nas próximas aberturas, usar biometria para desbloquear o token salvo.

### Flutter — local_auth

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  local_auth: ^2.3.0
  flutter_secure_storage: ^9.2.0
```

```bash
flutter pub get
```

**2. Configuração Android**

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest>
  <uses-permission android:name="android.permission.USE_BIOMETRIC" />
  <uses-permission android:name="android.permission.USE_FINGERPRINT" />
</manifest>
```

```kotlin
// android/app/src/main/kotlin/.../MainActivity.kt
import io.flutter.embedding.android.FlutterFragmentActivity

// Trocar FlutterActivity por FlutterFragmentActivity (obrigatório para local_auth)
class MainActivity: FlutterFragmentActivity()
```

**3. Configuração iOS — `ios/Runner/Info.plist`**

```xml
<dict>
  <key>NSFaceIDUsageDescription</key>
  <string>Usamos Face ID para proteger o acesso ao aplicativo.</string>
</dict>
```

**4. Serviço de biometria**

```dart
// lib/core/auth/biometric_service.dart
import 'package:local_auth/local_auth.dart';
import 'package:flutter/services.dart';

enum BiometricStatus {
  disponivel,
  indisponivel,
  naoConfigurado,
}

class BiometricService {
  final _auth = LocalAuthentication();

  Future<BiometricStatus> verificarDisponibilidade() async {
    try {
      final podeVerificar = await _auth.canCheckBiometrics;
      final suportado = await _auth.isDeviceSupported();

      if (!suportado) return BiometricStatus.indisponivel;

      if (!podeVerificar) return BiometricStatus.naoConfigurado;

      final biometrias = await _auth.getAvailableBiometrics();
      if (biometrias.isEmpty) return BiometricStatus.naoConfigurado;

      return BiometricStatus.disponivel;
    } on PlatformException {
      return BiometricStatus.indisponivel;
    }
  }

  Future<List<BiometricType>> listarTipos() async {
    try {
      return await _auth.getAvailableBiometrics();
    } on PlatformException {
      return [];
    }
  }

  Future<bool> autenticar({
    String motivo = 'Confirme sua identidade para continuar',
  }) async {
    try {
      return await _auth.authenticate(
        localizedReason: motivo,
        options: const AuthenticationOptions(
          stickyAuth: true,
          biometricOnly: false, // permite fallback para PIN/senha do sistema
          useErrorDialogs: true,
        ),
      );
    } on PlatformException {
      return false;
    }
  }

  Future<bool> autenticarApenasBiometria({
    String motivo = 'Use sua biometria para continuar',
  }) async {
    try {
      return await _auth.authenticate(
        localizedReason: motivo,
        options: const AuthenticationOptions(
          stickyAuth: true,
          biometricOnly: true, // somente biometria, sem fallback para PIN
        ),
      );
    } on PlatformException {
      return false;
    }
  }
}
```

**5. Serviço de sessão com biometria**

```dart
// lib/core/auth/secure_session_service.dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'biometric_service.dart';

class SecureSessionService {
  final _storage = const FlutterSecureStorage();
  final _biometric = BiometricService();

  static const _keyToken = 'auth_token';
  static const _keyBiometricEnabled = 'biometric_enabled';

  // Salvar token após login bem-sucedido
  Future<void> salvarToken(String token) async {
    await _storage.write(key: _keyToken, value: token);
  }

  // Habilitar/desabilitar biometria
  Future<void> setBiometriaHabilitada(bool habilitada) async {
    await _storage.write(
      key: _keyBiometricEnabled,
      value: habilitada.toString(),
    );
  }

  Future<bool> isBiometriaHabilitada() async {
    final valor = await _storage.read(key: _keyBiometricEnabled);
    return valor == 'true';
  }

  // Recuperar token protegido por biometria
  Future<String?> recuperarTokenComBiometria() async {
    final habilitada = await isBiometriaHabilitada();
    if (!habilitada) return null;

    final autenticado = await _biometric.autenticar(
      motivo: 'Desbloqueie para acessar sua conta',
    );
    if (!autenticado) return null;

    return await _storage.read(key: _keyToken);
  }

  Future<void> limpar() async {
    await _storage.deleteAll();
  }
}
```

**6. Tela de bloqueio biométrico**

```dart
// lib/features/auth/presentation/biometric_lock_screen.dart
import 'package:flutter/material.dart';
import '../../../core/auth/biometric_service.dart';

class BiometricLockScreen extends StatefulWidget {
  final VoidCallback onDesbloqueado;
  final VoidCallback? onUsarSenha;

  const BiometricLockScreen({
    super.key,
    required this.onDesbloqueado,
    this.onUsarSenha,
  });

  @override
  State<BiometricLockScreen> createState() => _BiometricLockScreenState();
}

class _BiometricLockScreenState extends State<BiometricLockScreen> {
  final _biometric = BiometricService();
  bool _verificando = false;
  String? _erro;
  List<BiometricType> _tipos = [];

  @override
  void initState() {
    super.initState();
    _inicializar();
  }

  Future<void> _inicializar() async {
    _tipos = await _biometric.listarTipos();
    _autenticar();
  }

  Future<void> _autenticar() async {
    setState(() {
      _verificando = true;
      _erro = null;
    });

    final sucesso = await _biometric.autenticar(
      motivo: 'Desbloqueie para acessar o aplicativo',
    );

    if (mounted) {
      if (sucesso) {
        widget.onDesbloqueado();
      } else {
        setState(() {
          _verificando = false;
          _erro = 'Autenticação não reconhecida. Tente novamente.';
        });
      }
    }
  }

  IconData get _icone {
    if (_tipos.contains(BiometricType.face)) return Icons.face;
    if (_tipos.contains(BiometricType.fingerprint)) return Icons.fingerprint;
    return Icons.lock;
  }

  String get _rotulo {
    if (_tipos.contains(BiometricType.face)) return 'Face ID';
    if (_tipos.contains(BiometricType.fingerprint)) return 'Impressão digital';
    return 'Biometria';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Padding(
            padding: const EdgeInsets.all(32),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(_icone, size: 80, color: Theme.of(context).colorScheme.primary),
                const SizedBox(height: 24),
                Text('App Bloqueado',
                    style: Theme.of(context).textTheme.headlineMedium),
                const SizedBox(height: 8),
                Text('Use $_rotulo para desbloquear',
                    style: Theme.of(context).textTheme.bodyLarge?.copyWith(
                          color: Colors.grey,
                        )),
                const SizedBox(height: 48),

                if (_verificando)
                  const CircularProgressIndicator()
                else ...[
                  FilledButton.icon(
                    onPressed: _autenticar,
                    icon: Icon(_icone),
                    label: Text('Usar $_rotulo'),
                    style: FilledButton.styleFrom(
                      minimumSize: const Size(double.infinity, 52),
                    ),
                  ),
                  if (widget.onUsarSenha != null) ...[
                    const SizedBox(height: 16),
                    OutlinedButton(
                      onPressed: widget.onUsarSenha,
                      style: OutlinedButton.styleFrom(
                        minimumSize: const Size(double.infinity, 52),
                      ),
                      child: const Text('Usar senha'),
                    ),
                  ],
                ],

                if (_erro != null) ...[
                  const SizedBox(height: 24),
                  Text(_erro!,
                      style: const TextStyle(color: Colors.red),
                      textAlign: TextAlign.center),
                ],
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**7. Tela de configurações de biometria**

```dart
// lib/features/auth/presentation/biometric_settings_screen.dart
import 'package:flutter/material.dart';
import '../../../core/auth/biometric_service.dart';
import '../../../core/auth/secure_session_service.dart';

class BiometricSettingsScreen extends StatefulWidget {
  const BiometricSettingsScreen({super.key});

  @override
  State<BiometricSettingsScreen> createState() =>
      _BiometricSettingsScreenState();
}

class _BiometricSettingsScreenState extends State<BiometricSettingsScreen> {
  final _biometric = BiometricService();
  final _session = SecureSessionService();

  BiometricStatus _status = BiometricStatus.indisponivel;
  List<BiometricType> _tipos = [];
  bool _habilitada = false;
  bool _carregando = true;

  @override
  void initState() {
    super.initState();
    _carregar();
  }

  Future<void> _carregar() async {
    final status = await _biometric.verificarDisponibilidade();
    final tipos = await _biometric.listarTipos();
    final habilitada = await _session.isBiometriaHabilitada();

    setState(() {
      _status = status;
      _tipos = tipos;
      _habilitada = habilitada;
      _carregando = false;
    });
  }

  Future<void> _toggleBiometria(bool valor) async {
    if (valor) {
      // Confirmar identidade antes de habilitar
      final autenticado = await _biometric.autenticar(
        motivo: 'Confirme sua identidade para ativar a biometria',
      );
      if (!autenticado) return;
    }

    await _session.setBiometriaHabilitada(valor);
    setState(() => _habilitada = valor);

    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text(valor
              ? 'Biometria ativada com sucesso!'
              : 'Biometria desativada.'),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_carregando) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }

    return Scaffold(
      appBar: AppBar(title: const Text('Segurança')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // Status do dispositivo
          Card(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('Status do Dispositivo',
                      style: Theme.of(context).textTheme.titleMedium),
                  const SizedBox(height: 12),
                  _InfoRow(
                    rotulo: 'Suporte a biometria',
                    valor: _status == BiometricStatus.disponivel
                        ? 'Disponível'
                        : _status == BiometricStatus.naoConfigurado
                            ? 'Não configurado'
                            : 'Indisponível',
                    icone: _status == BiometricStatus.disponivel
                        ? Icons.check_circle
                        : Icons.cancel,
                    corIcone: _status == BiometricStatus.disponivel
                        ? Colors.green
                        : Colors.red,
                  ),
                  const SizedBox(height: 8),
                  _InfoRow(
                    rotulo: 'Tipos disponíveis',
                    valor: _tipos.isEmpty
                        ? 'Nenhum'
                        : _tipos.map((t) {
                            return switch (t) {
                              BiometricType.fingerprint => 'Impressão digital',
                              BiometricType.face => 'Reconhecimento facial',
                              BiometricType.iris => 'Íris',
                              _ => t.name,
                            };
                          }).join(', '),
                    icone: Icons.info_outline,
                    corIcone: Colors.blue,
                  ),
                ],
              ),
            ),
          ),
          const SizedBox(height: 16),

          // Toggle de biometria
          Card(
            child: SwitchListTile(
              title: const Text('Desbloquear com biometria'),
              subtitle: Text(_habilitada
                  ? 'Use sua biometria para acessar o app'
                  : 'Ativar desbloqueio biométrico'),
              secondary: Icon(
                _tipos.contains(BiometricType.face)
                    ? Icons.face
                    : Icons.fingerprint,
                size: 32,
                color: _habilitada
                    ? Theme.of(context).colorScheme.primary
                    : Colors.grey,
              ),
              value: _habilitada,
              onChanged: _status == BiometricStatus.disponivel
                  ? _toggleBiometria
                  : null,
            ),
          ),
          const SizedBox(height: 16),

          // Botão de teste
          if (_status == BiometricStatus.disponivel)
            OutlinedButton.icon(
              onPressed: () async {
                final ok = await _biometric.autenticar(
                  motivo: 'Teste de biometria',
                );
                if (mounted) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(
                      content: Text(ok
                          ? 'Autenticação bem-sucedida!'
                          : 'Falha na autenticação.'),
                      backgroundColor: ok ? Colors.green : Colors.red,
                    ),
                  );
                }
              },
              icon: const Icon(Icons.science),
              label: const Text('Testar biometria agora'),
            ),

          if (_status == BiometricStatus.naoConfigurado)
            Padding(
              padding: const EdgeInsets.only(top: 16),
              child: Text(
                'Nenhuma biometria configurada no dispositivo. '
                'Acesse as configurações do sistema para cadastrar '
                'sua impressão digital ou rosto.',
                style: Theme.of(context).textTheme.bodyMedium?.copyWith(
                      color: Colors.orange.shade800,
                    ),
              ),
            ),
        ],
      ),
    );
  }
}

class _InfoRow extends StatelessWidget {
  final String rotulo;
  final String valor;
  final IconData icone;
  final Color corIcone;

  const _InfoRow({
    required this.rotulo,
    required this.valor,
    required this.icone,
    required this.corIcone,
  });

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Icon(icone, size: 20, color: corIcone),
        const SizedBox(width: 8),
        Expanded(
          child: RichText(
            text: TextSpan(
              style: Theme.of(context).textTheme.bodyMedium,
              children: [
                TextSpan(
                  text: '$rotulo: ',
                  style: const TextStyle(fontWeight: FontWeight.w500),
                ),
                TextSpan(text: valor),
              ],
            ),
          ),
        ),
      ],
    );
  }
}
```

**8. Uso: bloqueio ao abrir o app (integração com GoRouter)**

```dart
// lib/routes/app_router.dart (trecho)
import '../features/auth/presentation/biometric_lock_screen.dart';

// Dentro do GoRouter, adicionar antes da rota raiz:
GoRoute(
  path: '/lock',
  builder: (_, __) => BiometricLockScreen(
    onDesbloqueado: () => context.go('/'),
    onUsarSenha: () => context.go('/login'),
  ),
),

// No redirect global, verificar se biometria está habilitada:
redirect: (context, state) async {
  final destino = state.matchedLocation;
  if (destino == '/lock') return null;

  final session = SecureSessionService();
  final bioHabilitada = await session.isBiometriaHabilitada();
  final token = await session.recuperarTokenSemBiometria();

  // Se tem token salvo e biometria habilitada, exigir desbloqueio
  if (token != null && bioHabilitada && !appDesbloqueado) {
    return '/lock';
  }

  return null;
},
```

---

### React Native — expo-local-authentication

**1. Dependências**

```bash
npx expo install expo-local-authentication expo-secure-store
```

**2. Serviço de biometria**

```typescript
// src/core/auth/biometricService.ts
import * as LocalAuthentication from 'expo-local-authentication';

export type BiometricStatus = 'disponivel' | 'indisponivel' | 'nao_configurado';

export async function verificarDisponibilidade(): Promise<BiometricStatus> {
  const compativel = await LocalAuthentication.hasHardwareAsync();
  if (!compativel) return 'indisponivel';

  const cadastrado = await LocalAuthentication.isEnrolledAsync();
  if (!cadastrado) return 'nao_configurado';

  return 'disponivel';
}

export async function listarTipos(): Promise<LocalAuthentication.AuthenticationType[]> {
  return LocalAuthentication.supportedAuthenticationTypesAsync();
}

export function nomeTipo(tipo: LocalAuthentication.AuthenticationType): string {
  switch (tipo) {
    case LocalAuthentication.AuthenticationType.FINGERPRINT:
      return 'Impressão digital';
    case LocalAuthentication.AuthenticationType.FACIAL_RECOGNITION:
      return 'Reconhecimento facial';
    case LocalAuthentication.AuthenticationType.IRIS:
      return 'Íris';
    default:
      return 'Biometria';
  }
}

export async function autenticar(motivo: string): Promise<boolean> {
  const resultado = await LocalAuthentication.authenticateAsync({
    promptMessage: motivo,
    cancelLabel: 'Cancelar',
    disableDeviceFallback: false, // permite fallback para PIN/senha
    fallbackLabel: 'Usar senha',
  });
  return resultado.success;
}

export async function autenticarApenasBiometria(motivo: string): Promise<boolean> {
  const resultado = await LocalAuthentication.authenticateAsync({
    promptMessage: motivo,
    cancelLabel: 'Cancelar',
    disableDeviceFallback: true, // somente biometria
  });
  return resultado.success;
}
```

**3. Serviço de sessão segura**

```typescript
// src/core/auth/secureSessionService.ts
import * as SecureStore from 'expo-secure-store';
import { autenticar } from './biometricService';

const KEY_TOKEN = 'auth_token';
const KEY_BIO_ENABLED = 'biometric_enabled';

export async function salvarToken(token: string) {
  await SecureStore.setItemAsync(KEY_TOKEN, token);
}

export async function setBiometriaHabilitada(habilitada: boolean) {
  await SecureStore.setItemAsync(KEY_BIO_ENABLED, String(habilitada));
}

export async function isBiometriaHabilitada(): Promise<boolean> {
  const valor = await SecureStore.getItemAsync(KEY_BIO_ENABLED);
  return valor === 'true';
}

export async function recuperarTokenComBiometria(): Promise<string | null> {
  const habilitada = await isBiometriaHabilitada();
  if (!habilitada) return null;

  const ok = await autenticar('Desbloqueie para acessar sua conta');
  if (!ok) return null;

  return SecureStore.getItemAsync(KEY_TOKEN);
}

export async function limparSessao() {
  await SecureStore.deleteItemAsync(KEY_TOKEN);
  await SecureStore.deleteItemAsync(KEY_BIO_ENABLED);
}
```

**4. Hook de biometria**

```typescript
// src/hooks/useBiometric.ts
import { useEffect, useState, useCallback } from 'react';
import * as LocalAuthentication from 'expo-local-authentication';
import {
  verificarDisponibilidade,
  listarTipos,
  autenticar,
  nomeTipo,
  BiometricStatus,
} from '../core/auth/biometricService';
import {
  isBiometriaHabilitada,
  setBiometriaHabilitada,
} from '../core/auth/secureSessionService';

export function useBiometric() {
  const [status, setStatus] = useState<BiometricStatus>('indisponivel');
  const [tipos, setTipos] = useState<LocalAuthentication.AuthenticationType[]>([]);
  const [habilitada, setHabilitada] = useState(false);
  const [carregando, setCarregando] = useState(true);

  useEffect(() => {
    (async () => {
      const [s, t, h] = await Promise.all([
        verificarDisponibilidade(),
        listarTipos(),
        isBiometriaHabilitada(),
      ]);
      setStatus(s);
      setTipos(t);
      setHabilitada(h);
      setCarregando(false);
    })();
  }, []);

  const toggle = useCallback(async (valor: boolean) => {
    if (valor) {
      const ok = await autenticar('Confirme sua identidade para ativar a biometria');
      if (!ok) return;
    }
    await setBiometriaHabilitada(valor);
    setHabilitada(valor);
  }, []);

  const tiposTexto = tipos.map(nomeTipo);

  return { status, tipos, tiposTexto, habilitada, carregando, toggle };
}
```

**5. Tela de bloqueio biométrico**

```tsx
// src/features/auth/BiometricLockScreen.tsx
import React, { useEffect, useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { autenticar, listarTipos, nomeTipo } from '../../core/auth/biometricService';
import * as LocalAuthentication from 'expo-local-authentication';

interface Props {
  onDesbloqueado: () => void;
  onUsarSenha?: () => void;
}

export function BiometricLockScreen({ onDesbloqueado, onUsarSenha }: Props) {
  const [verificando, setVerificando] = useState(false);
  const [erro, setErro] = useState<string | null>(null);
  const [tipoLabel, setTipoLabel] = useState('Biometria');

  useEffect(() => {
    listarTipos().then((tipos) => {
      if (tipos.length > 0) setTipoLabel(nomeTipo(tipos[0]));
    });
    tentarAutenticar();
  }, []);

  const tentarAutenticar = async () => {
    setVerificando(true);
    setErro(null);
    const ok = await autenticar('Desbloqueie para acessar o aplicativo');
    setVerificando(false);
    if (ok) {
      onDesbloqueado();
    } else {
      setErro('Autenticação não reconhecida. Tente novamente.');
    }
  };

  const icone = tipoLabel.includes('facial') ? '🧑' : '🔒';

  return (
    <View style={styles.container}>
      <Text style={styles.icone}>{icone}</Text>
      <Text style={styles.titulo}>App Bloqueado</Text>
      <Text style={styles.subtitulo}>Use {tipoLabel} para desbloquear</Text>

      {verificando ? (
        <Text style={styles.verificando}>Verificando…</Text>
      ) : (
        <View style={styles.botoes}>
          <TouchableOpacity style={styles.btnPrimario} onPress={tentarAutenticar}>
            <Text style={styles.btnPrimarioTexto}>Usar {tipoLabel}</Text>
          </TouchableOpacity>
          {onUsarSenha && (
            <TouchableOpacity style={styles.btnSecundario} onPress={onUsarSenha}>
              <Text style={styles.btnSecundarioTexto}>Usar senha</Text>
            </TouchableOpacity>
          )}
        </View>
      )}

      {erro && <Text style={styles.erro}>{erro}</Text>}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 32, backgroundColor: '#fff' },
  icone: { fontSize: 72, marginBottom: 24 },
  titulo: { fontSize: 24, fontWeight: '700', marginBottom: 8 },
  subtitulo: { fontSize: 16, color: '#666', marginBottom: 48 },
  verificando: { fontSize: 14, color: '#999' },
  botoes: { width: '100%', gap: 12 },
  btnPrimario: { backgroundColor: '#3949ab', paddingVertical: 16, borderRadius: 12, alignItems: 'center' },
  btnPrimarioTexto: { color: '#fff', fontSize: 16, fontWeight: '600' },
  btnSecundario: { borderWidth: 1, borderColor: '#ccc', paddingVertical: 16, borderRadius: 12, alignItems: 'center' },
  btnSecundarioTexto: { color: '#666', fontSize: 16 },
  erro: { color: '#e53935', marginTop: 24, textAlign: 'center' },
});
```

**6. Tela de configurações de biometria**

```tsx
// src/features/auth/BiometricSettingsScreen.tsx
import React from 'react';
import {
  View, Text, Switch, TouchableOpacity, StyleSheet, Alert,
} from 'react-native';
import { useBiometric } from '../../hooks/useBiometric';
import { autenticar } from '../../core/auth/biometricService';

export function BiometricSettingsScreen() {
  const { status, tiposTexto, habilitada, carregando, toggle } = useBiometric();

  if (carregando) {
    return (
      <View style={styles.center}>
        <Text>Carregando…</Text>
      </View>
    );
  }

  const testar = async () => {
    const ok = await autenticar('Teste de biometria');
    Alert.alert(
      ok ? 'Sucesso' : 'Falha',
      ok ? 'Autenticação bem-sucedida!' : 'Falha na autenticação.',
    );
  };

  return (
    <View style={styles.container}>
      {/* Status */}
      <View style={styles.card}>
        <Text style={styles.cardTitulo}>Status do Dispositivo</Text>
        <View style={styles.info}>
          <Text style={styles.infoLabel}>Suporte a biometria:</Text>
          <Text style={[
            styles.infoValor,
            { color: status === 'disponivel' ? '#4caf50' : '#f44336' },
          ]}>
            {status === 'disponivel' ? 'Disponível' : status === 'nao_configurado' ? 'Não configurado' : 'Indisponível'}
          </Text>
        </View>
        <View style={styles.info}>
          <Text style={styles.infoLabel}>Tipos:</Text>
          <Text style={styles.infoValor}>
            {tiposTexto.length > 0 ? tiposTexto.join(', ') : 'Nenhum'}
          </Text>
        </View>
      </View>

      {/* Toggle */}
      <View style={styles.card}>
        <View style={styles.toggleRow}>
          <View style={{ flex: 1 }}>
            <Text style={styles.toggleTitulo}>Desbloquear com biometria</Text>
            <Text style={styles.toggleSub}>
              {habilitada ? 'Use sua biometria para acessar o app' : 'Ativar desbloqueio biométrico'}
            </Text>
          </View>
          <Switch
            value={habilitada}
            onValueChange={toggle}
            disabled={status !== 'disponivel'}
          />
        </View>
      </View>

      {/* Botão de teste */}
      {status === 'disponivel' && (
        <TouchableOpacity style={styles.btnTeste} onPress={testar}>
          <Text style={styles.btnTesteTexto}>🧪 Testar biometria agora</Text>
        </TouchableOpacity>
      )}

      {status === 'nao_configurado' && (
        <Text style={styles.aviso}>
          Nenhuma biometria configurada no dispositivo. Acesse as configurações do sistema para cadastrar sua impressão digital ou rosto.
        </Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#f9f9f9' },
  center: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  card: { backgroundColor: '#fff', borderRadius: 12, padding: 16, marginBottom: 16, elevation: 1 },
  cardTitulo: { fontSize: 16, fontWeight: '600', marginBottom: 12 },
  info: { flexDirection: 'row', marginBottom: 6 },
  infoLabel: { fontWeight: '500', color: '#555', marginRight: 6 },
  infoValor: { color: '#333' },
  toggleRow: { flexDirection: 'row', alignItems: 'center' },
  toggleTitulo: { fontSize: 15, fontWeight: '600' },
  toggleSub: { fontSize: 13, color: '#999', marginTop: 4 },
  btnTeste: { borderWidth: 1, borderColor: '#3949ab', paddingVertical: 14, borderRadius: 12, alignItems: 'center' },
  btnTesteTexto: { color: '#3949ab', fontWeight: '600' },
  aviso: { color: '#e65100', marginTop: 8, lineHeight: 20 },
});
```

---

### React Web — Web Authentication API (WebAuthn)

A Web Authentication API permite usar biometria (impressão digital, Face ID) e chaves de segurança no navegador. O fluxo envolve **registro** (criar credencial) e **autenticação** (verificar credencial).

> **Pré-requisito:** HTTPS obrigatório (exceto `localhost` para desenvolvimento). O servidor precisa fornecer um `challenge` aleatório para cada operação.

**1. Serviço WebAuthn**

```typescript
// src/core/auth/webauthnService.ts

function bufferToBase64Url(buffer: ArrayBuffer): string {
  return btoa(String.fromCharCode(...new Uint8Array(buffer)))
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=+$/, '');
}

function base64UrlToBuffer(base64url: string): ArrayBuffer {
  const base64 = base64url.replace(/-/g, '+').replace(/_/g, '/');
  const raw = atob(base64);
  return Uint8Array.from(raw, (c) => c.charCodeAt(0)).buffer;
}

export function isWebAuthnDisponivel(): boolean {
  return (
    window.isSecureContext &&
    'credentials' in navigator &&
    typeof PublicKeyCredential !== 'undefined'
  );
}

export async function verificarSuporteBiometria(): Promise<boolean> {
  if (!isWebAuthnDisponivel()) return false;
  try {
    return await PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable();
  } catch {
    return false;
  }
}

// Registrar credencial biométrica (chamado uma vez, após login com senha)
export async function registrarCredencial(
  userId: string,
  userName: string,
  challenge: string, // vem do servidor, base64url
): Promise<{ credentialId: string; publicKey: string } | null> {
  try {
    const credential = (await navigator.credentials.create({
      publicKey: {
        challenge: base64UrlToBuffer(challenge),
        rp: { name: 'MeuApp', id: window.location.hostname },
        user: {
          id: new TextEncoder().encode(userId),
          name: userName,
          displayName: userName,
        },
        pubKeyCredParams: [
          { alg: -7, type: 'public-key' },   // ES256
          { alg: -257, type: 'public-key' },  // RS256
        ],
        authenticatorSelection: {
          authenticatorAttachment: 'platform', // biometria do dispositivo
          userVerification: 'required',
          residentKey: 'preferred',
        },
        timeout: 60000,
      },
    })) as PublicKeyCredential | null;

    if (!credential) return null;

    const response = credential.response as AuthenticatorAttestationResponse;
    return {
      credentialId: bufferToBase64Url(credential.rawId),
      publicKey: bufferToBase64Url(response.getPublicKey()!),
    };
  } catch {
    return null;
  }
}

// Autenticar com biometria (chamado a cada acesso)
export async function autenticarComBiometria(
  challenge: string, // vem do servidor, base64url
  credentialId?: string,
): Promise<{ credentialId: string; signature: string } | null> {
  try {
    const options: PublicKeyCredentialRequestOptions = {
      challenge: base64UrlToBuffer(challenge),
      rpId: window.location.hostname,
      userVerification: 'required',
      timeout: 60000,
    };

    if (credentialId) {
      options.allowCredentials = [
        { id: base64UrlToBuffer(credentialId), type: 'public-key' },
      ];
    }

    const assertion = (await navigator.credentials.get({
      publicKey: options,
    })) as PublicKeyCredential | null;

    if (!assertion) return null;

    const response = assertion.response as AuthenticatorAssertionResponse;
    return {
      credentialId: bufferToBase64Url(assertion.rawId),
      signature: bufferToBase64Url(response.signature),
    };
  } catch {
    return null;
  }
}
```

**2. Hook de biometria Web**

```typescript
// src/hooks/useBiometric.ts
import { useEffect, useState, useCallback } from 'react';
import {
  verificarSuporteBiometria,
  registrarCredencial,
  autenticarComBiometria,
  isWebAuthnDisponivel,
} from '../core/auth/webauthnService';

const CREDENTIAL_KEY = 'webauthn_credential_id';
const BIO_ENABLED_KEY = 'biometric_enabled';

export function useBiometric() {
  const [disponivel, setDisponivel] = useState(false);
  const [habilitada, setHabilitada] = useState(false);
  const [carregando, setCarregando] = useState(true);

  useEffect(() => {
    (async () => {
      const ok = await verificarSuporteBiometria();
      setDisponivel(ok);
      setHabilitada(localStorage.getItem(BIO_ENABLED_KEY) === 'true');
      setCarregando(false);
    })();
  }, []);

  const registrar = useCallback(async (userId: string, userName: string) => {
    // Em produção, o challenge vem do servidor
    const challenge = bufferToBase64Url(crypto.getRandomValues(new Uint8Array(32)));

    const resultado = await registrarCredencial(userId, userName, challenge);
    if (resultado) {
      localStorage.setItem(CREDENTIAL_KEY, resultado.credentialId);
      localStorage.setItem(BIO_ENABLED_KEY, 'true');
      setHabilitada(true);
      return true;
    }
    return false;
  }, []);

  const autenticar = useCallback(async () => {
    const credentialId = localStorage.getItem(CREDENTIAL_KEY) ?? undefined;
    // Em produção, o challenge vem do servidor
    const challenge = bufferToBase64Url(crypto.getRandomValues(new Uint8Array(32)));

    const resultado = await autenticarComBiometria(challenge, credentialId);
    return resultado !== null;
  }, []);

  const desabilitar = useCallback(() => {
    localStorage.removeItem(CREDENTIAL_KEY);
    localStorage.removeItem(BIO_ENABLED_KEY);
    setHabilitada(false);
  }, []);

  return { disponivel, habilitada, carregando, registrar, autenticar, desabilitar };
}

function bufferToBase64Url(buffer: ArrayBuffer | Uint8Array): string {
  const bytes = buffer instanceof Uint8Array ? buffer : new Uint8Array(buffer);
  return btoa(String.fromCharCode(...bytes))
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=+$/, '');
}
```

**3. Tela de configurações de biometria**

```tsx
// src/features/auth/BiometricSettingsPage.tsx
import { useState } from 'react';
import { useBiometric } from '../../hooks/useBiometric';

export function BiometricSettingsPage() {
  const { disponivel, habilitada, carregando, registrar, autenticar, desabilitar } =
    useBiometric();
  const [mensagem, setMensagem] = useState<{ texto: string; tipo: 'success' | 'danger' } | null>(null);

  if (carregando) {
    return (
      <div className="d-flex justify-content-center py-5">
        <span className="spinner-border" />
      </div>
    );
  }

  const handleToggle = async () => {
    if (habilitada) {
      desabilitar();
      setMensagem({ texto: 'Biometria desativada.', tipo: 'success' });
    } else {
      const ok = await registrar('user-1', 'usuario@exemplo.com');
      setMensagem(
        ok
          ? { texto: 'Biometria ativada com sucesso!', tipo: 'success' }
          : { texto: 'Não foi possível ativar a biometria.', tipo: 'danger' },
      );
    }
  };

  const handleTestar = async () => {
    const ok = await autenticar();
    setMensagem(
      ok
        ? { texto: 'Autenticação bem-sucedida!', tipo: 'success' }
        : { texto: 'Falha na autenticação.', tipo: 'danger' },
    );
  };

  return (
    <div className="container py-4" style={{ maxWidth: 600 }}>
      <h4 className="mb-4">
        <i className="fas fa-fingerprint me-2" />
        Segurança
      </h4>

      {mensagem && (
        <div className={`alert alert-${mensagem.tipo} alert-dismissible`}>
          {mensagem.texto}
          <button className="btn-close" onClick={() => setMensagem(null)} />
        </div>
      )}

      {/* Status */}
      <div className="card mb-3">
        <div className="card-body">
          <h6 className="card-title">Status do Dispositivo</h6>
          <div className="d-flex align-items-center gap-2 mb-2">
            <i className={`fas fa-${disponivel ? 'check-circle text-success' : 'times-circle text-danger'}`} />
            <span>WebAuthn: {disponivel ? 'Disponível' : 'Indisponível'}</span>
          </div>
          <div className="d-flex align-items-center gap-2">
            <i className="fas fa-info-circle text-primary" />
            <span>
              Suporta: {disponivel
                ? 'Impressão digital / Face ID / Chave de segurança'
                : 'Navegador ou contexto sem suporte'}
            </span>
          </div>
        </div>
      </div>

      {/* Toggle */}
      <div className="card mb-3">
        <div className="card-body d-flex align-items-center">
          <i className={`fas fa-fingerprint fa-2x me-3 ${habilitada ? 'text-primary' : 'text-muted'}`} />
          <div className="flex-grow-1">
            <strong>Desbloquear com biometria</strong>
            <div className="text-muted small">
              {habilitada
                ? 'Use sua biometria para acessar o app'
                : 'Ativar desbloqueio biométrico'}
            </div>
          </div>
          <div className="form-check form-switch">
            <input
              className="form-check-input"
              type="checkbox"
              role="switch"
              checked={habilitada}
              onChange={handleToggle}
              disabled={!disponivel}
            />
          </div>
        </div>
      </div>

      {/* Botão de teste */}
      {disponivel && habilitada && (
        <button className="btn btn-outline-primary w-100" onClick={handleTestar}>
          <i className="fas fa-vial me-2" />
          Testar biometria agora
        </button>
      )}

      {!disponivel && (
        <div className="alert alert-warning mt-3">
          <i className="fas fa-exclamation-triangle me-2" />
          WebAuthn não está disponível neste navegador ou contexto.
          Verifique se está usando HTTPS e um navegador compatível.
        </div>
      )}
    </div>
  );
}
```

**4. Tela de bloqueio biométrico**

```tsx
// src/features/auth/BiometricLockPage.tsx
import { useEffect, useState } from 'react';
import { useBiometric } from '../../hooks/useBiometric';

interface Props {
  onDesbloqueado: () => void;
  onUsarSenha?: () => void;
}

export function BiometricLockPage({ onDesbloqueado, onUsarSenha }: Props) {
  const { autenticar } = useBiometric();
  const [verificando, setVerificando] = useState(false);
  const [erro, setErro] = useState<string | null>(null);

  const tentarAutenticar = async () => {
    setVerificando(true);
    setErro(null);
    const ok = await autenticar();
    setVerificando(false);
    if (ok) {
      onDesbloqueado();
    } else {
      setErro('Autenticação não reconhecida. Tente novamente.');
    }
  };

  useEffect(() => {
    tentarAutenticar();
  }, []);

  return (
    <div
      className="d-flex flex-column align-items-center justify-content-center text-center"
      style={{ minHeight: '100vh' }}
    >
      <i className="fas fa-lock fa-4x text-primary mb-4" />
      <h3>App Bloqueado</h3>
      <p className="text-muted mb-5">Use biometria para desbloquear</p>

      {verificando ? (
        <span className="spinner-border text-primary" />
      ) : (
        <div className="d-grid gap-3" style={{ width: 280 }}>
          <button className="btn btn-primary btn-lg" onClick={tentarAutenticar}>
            <i className="fas fa-fingerprint me-2" />
            Desbloquear
          </button>
          {onUsarSenha && (
            <button className="btn btn-outline-secondary" onClick={onUsarSenha}>
              Usar senha
            </button>
          )}
        </div>
      )}

      {erro && (
        <div className="alert alert-danger mt-4" style={{ maxWidth: 360 }}>
          {erro}
        </div>
      )}
    </div>
  );
}
```

---

### Equivalência — Autenticação Biométrica

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Biblioteca | `local_auth` | `expo-local-authentication` | Web Authentication API (WebAuthn) |
| Verificar disponibilidade | `canCheckBiometrics` + `isDeviceSupported` | `hasHardwareAsync` + `isEnrolledAsync` | `isUserVerifyingPlatformAuthenticatorAvailable` |
| Listar tipos | `getAvailableBiometrics` | `supportedAuthenticationTypesAsync` | N/A (o navegador escolhe) |
| Autenticar | `authenticate()` | `authenticateAsync()` | `navigator.credentials.get()` |
| Fallback para PIN | `biometricOnly: false` | `disableDeviceFallback: false` | Gerenciado pelo sistema |
| Registrar credencial | N/A (gerenciado pelo SO) | N/A (gerenciado pelo SO) | `navigator.credentials.create()` |
| Armazenamento seguro | `flutter_secure_storage` | `expo-secure-store` | `localStorage` (credentialId) |
| Tela de bloqueio | Widget customizado | Componente customizado | Página customizada |
| Configuração Android | `USE_BIOMETRIC` no Manifest + `FlutterFragmentActivity` | Automático via Expo | N/A |
| Configuração iOS | `NSFaceIDUsageDescription` no Info.plist | Automático via Expo | N/A |
| Requisito HTTPS | N/A | N/A | Obrigatório (exceto localhost) |
| Ícones | `Icons.fingerprint` / `Icons.face` | Emoji ou `@expo/vector-icons` | Font Awesome `fa-fingerprint` / `fa-lock` |

| Tipo biométrico | Flutter | React Native | React Web |
|---|---|---|---|
| Impressão digital | `BiometricType.fingerprint` | `AuthenticationType.FINGERPRINT` | Gerenciado pelo navegador |
| Face ID / Reconhecimento facial | `BiometricType.face` | `AuthenticationType.FACIAL_RECOGNITION` | Gerenciado pelo navegador |
| Íris | `BiometricType.iris` | `AuthenticationType.IRIS` | N/A |

> **Diferença fundamental no Web:** enquanto Flutter e React Native usam APIs locais que verificam a biometria diretamente no dispositivo, a versão Web usa WebAuthn — um padrão W3C que combina biometria local com criptografia de chave pública. Isso significa que no Web o servidor participa do processo (fornecendo challenges e verificando assinaturas), tornando-o mais seguro contra replay attacks, mas também mais complexo de implementar no backend.

---

## 16. Bluetooth — Integração com Beacons

Detecção e interação com **beacons BLE** (Bluetooth Low Energy) para cenários como:

- Proximidade em lojas, museus, eventos (ex: "Você está perto do setor X").
- Check-in automático em espaços físicos.
- Navegação indoor.
- Coleta de dados de presença.

---

### Pré-requisitos e Infraestrutura

Antes do código, é necessário ter a infraestrutura de beacons funcionando. Esta seção explica o que é preciso **fora da aplicação**.

#### 1. Hardware — Beacons físicos

Beacons são pequenos dispositivos BLE que transmitem um sinal periodicamente. Precisam ser comprados e instalados nos locais desejados.

| Fabricante | Modelos populares | Faixa de preço (unidade) |
|---|---|---|
| Estimote | Proximity Beacons, Stickers | US$ 20–40 |
| Kontakt.io | Smart Beacon, Tough Beacon | US$ 15–30 |
| RadiusNetworks | RadBeacon | US$ 20–35 |
| Minew | E7, E8, C6 | US$ 5–15 |
| Genéricos (AliExpress) | nRF52-based | US$ 3–10 |

> **Alternativa para desenvolvimento/testes:** qualquer smartphone com BLE pode simular um beacon usando apps como **Beacon Simulator** (Android) ou **Locate Beacon** (iOS).

#### 2. Protocolos de beacon

| Protocolo | Criado por | Identificação | Dados transmitidos |
|---|---|---|---|
| **iBeacon** | Apple | `UUID` + `major` + `minor` | Apenas identificação; dados de contexto ficam no servidor |
| **Eddystone-UID** | Google | `namespace` (10 bytes) + `instance` (6 bytes) | Identificação (similar ao iBeacon) |
| **Eddystone-URL** | Google | URL compactada | URL acessível diretamente (Physical Web) |
| **AltBeacon** | Radius Networks | `beacon ID` (20 bytes) | Aberto, sem royalties |

> **Recomendação:** usar **iBeacon** para projetos iOS-first ou **Eddystone-UID** para multiplataforma. A maioria dos beacons suporta ambos simultaneamente.

#### 3. Configuração do beacon

Cada beacon precisa ser configurado (via app do fabricante ou NFC) com:

```
iBeacon:
  UUID:  f7826da6-4fa2-4e98-8024-bc5b71e0893e  ← identifica o "projeto" / empresa
  Major: 1                                       ← identifica a "região" (ex: filial SP)
  Minor: 42                                      ← identifica o ponto específico (ex: entrada)
  Tx Power: -59 dBm                              ← calibração de distância

Eddystone-UID:
  Namespace: 0x2f234454f4911ba9ffa6   ← identifica o projeto
  Instance:  0x00000000012a           ← identifica o ponto
```

#### 4. Servidor / Backend

O beacon transmite apenas seu **identificador**. O app precisa consultar um backend para saber **o que aquele beacon significa**:

```
GET /api/beacons/{uuid}/{major}/{minor}

Resposta:
{
  "local": "Loja Centro - Entrada principal",
  "mensagem": "Bem-vindo! Confira nossas promoções.",
  "acao": "abrir_promocoes",
  "imagemUrl": "https://cdn.exemplo.com/banner-promo.jpg"
}
```

#### 5. Permissões do sistema operacional

| Plataforma | Permissões necessárias | Observações |
|---|---|---|
| **Android** | `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`, `ACCESS_FINE_LOCATION` | Location obrigatória para BLE scanning (política do Android) |
| **iOS** | `NSBluetoothAlwaysUsageDescription`, `NSLocationWhenInUseUsageDescription` | Também `NSLocationAlwaysUsageDescription` para monitoramento em background |
| **Web** | Prompt automático do navegador | Bluetooth requer HTTPS; suporte limitado (apenas Chrome/Edge/Opera) |

#### 6. Conceitos: Ranging vs Monitoring

| Conceito | Descrição | Uso típico |
|---|---|---|
| **Ranging** | Detecta beacons **continuamente** e calcula distância aproximada (immediate/near/far). Funciona apenas com o app em foreground. | Navegação indoor, exibir conteúdo contextual |
| **Monitoring** | Detecta **entrada e saída** de uma região de beacon. Funciona em background. | Check-in automático, notificações de proximidade |

#### 7. Estimativa de distância

A distância é **estimada** a partir do RSSI (intensidade do sinal) e do Tx Power configurado no beacon. É inerentemente imprecisa:

| Zona | Distância estimada | Precisão |
|---|---|---|
| Immediate | < 0.5 m | Boa |
| Near | 0.5 – 3 m | Razoável |
| Far | > 3 m | Baixa (interferência de paredes, pessoas, etc.) |

> **Não use BLE para distância exata.** Use como proximidade relativa (perto/longe) ou presença (entrou/saiu).

---

### Flutter — flutter_beacon

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_beacon: ^0.5.1
  permission_handler: ^11.3.0
```

```bash
flutter pub get
```

**2. Configuração Android — `android/app/src/main/AndroidManifest.xml`**

```xml
<manifest>
  <uses-permission android:name="android.permission.BLUETOOTH" />
  <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
  <uses-permission android:name="android.permission.BLUETOOTH_SCAN"
      android:usesPermissionFlags="neverForLocation" />
  <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

  <!-- Para scanning em background (opcional) -->
  <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />

  <uses-feature android:name="android.hardware.bluetooth_le" android:required="true" />
</manifest>
```

**3. Configuração iOS — `ios/Runner/Info.plist`**

```xml
<dict>
  <key>NSBluetoothAlwaysUsageDescription</key>
  <string>Usamos Bluetooth para detectar beacons próximos.</string>
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>Precisamos da localização para detectar beacons.</string>
  <key>NSLocationAlwaysUsageDescription</key>
  <string>Detectamos beacons mesmo com o app em segundo plano.</string>
  <key>UIBackgroundModes</key>
  <array>
    <string>bluetooth-central</string>
    <string>location</string>
  </array>
</dict>
```

**4. Modelo de dados**

```dart
// lib/features/beacon/domain/beacon_info.dart
class BeaconInfo {
  final String uuid;
  final int major;
  final int minor;
  final double distancia;
  final String proximidade; // immediate, near, far, unknown
  final int rssi;
  final DateTime detectadoEm;

  BeaconInfo({
    required this.uuid,
    required this.major,
    required this.minor,
    required this.distancia,
    required this.proximidade,
    required this.rssi,
    DateTime? detectadoEm,
  }) : detectadoEm = detectadoEm ?? DateTime.now();
}

class BeaconContext {
  final String local;
  final String mensagem;
  final String? acao;
  final String? imagemUrl;

  BeaconContext({
    required this.local,
    required this.mensagem,
    this.acao,
    this.imagemUrl,
  });
}
```

**5. Serviço de beacon**

```dart
// lib/features/beacon/data/beacon_service.dart
import 'dart:async';
import 'package:flutter_beacon/flutter_beacon.dart';
import 'package:permission_handler/permission_handler.dart';
import '../domain/beacon_info.dart';

class BeaconService {
  StreamSubscription<RangingResult>? _rangingSub;
  StreamSubscription<MonitoringResult>? _monitoringSub;
  final _beaconsCtrl = StreamController<List<BeaconInfo>>.broadcast();
  final _eventosCtrl = StreamController<String>.broadcast();

  Stream<List<BeaconInfo>> get beacons => _beaconsCtrl.stream;
  Stream<String> get eventos => _eventosCtrl.stream;

  Future<bool> inicializar() async {
    // Solicitar permissões
    final btStatus = await Permission.bluetoothScan.request();
    final locStatus = await Permission.locationWhenInUse.request();

    if (!btStatus.isGranted || !locStatus.isGranted) return false;

    try {
      await flutterBeacon.initializeScanning;
      return true;
    } catch (_) {
      return false;
    }
  }

  // Ranging — detecção contínua com distância
  void iniciarRanging({required String uuid}) {
    final regiao = Region(
      identifier: 'meu-app',
      proximityUUID: uuid,
    );

    _rangingSub = flutterBeacon.ranging([regiao]).listen((result) {
      final lista = result.beacons.map((b) {
        return BeaconInfo(
          uuid: b.proximityUUID,
          major: b.major,
          minor: b.minor,
          distancia: b.accuracy,
          proximidade: _proximidadeTexto(b.proximity),
          rssi: b.rssi,
        );
      }).toList();

      // Ordenar por distância (mais próximo primeiro)
      lista.sort((a, b) => a.distancia.compareTo(b.distancia));
      _beaconsCtrl.add(lista);
    });
  }

  // Monitoring — detectar entrada/saída de região
  void iniciarMonitoring({required String uuid}) {
    final regiao = Region(
      identifier: 'meu-app',
      proximityUUID: uuid,
    );

    _monitoringSub = flutterBeacon.monitoring([regiao]).listen((result) {
      final evento = result.monitoringEventType == MonitoringEventType.didEnterRegion
          ? 'ENTROU na região (Major: ${result.region?.major}, Minor: ${result.region?.minor})'
          : 'SAIU da região (Major: ${result.region?.major}, Minor: ${result.region?.minor})';
      _eventosCtrl.add(evento);
    });
  }

  String _proximidadeTexto(Proximity p) {
    return switch (p) {
      Proximity.immediate => 'immediate',
      Proximity.near => 'near',
      Proximity.far => 'far',
      _ => 'unknown',
    };
  }

  void pararRanging() {
    _rangingSub?.cancel();
    _rangingSub = null;
  }

  void pararMonitoring() {
    _monitoringSub?.cancel();
    _monitoringSub = null;
  }

  void dispose() {
    pararRanging();
    pararMonitoring();
    _beaconsCtrl.close();
    _eventosCtrl.close();
  }
}
```

**6. Serviço de contexto (consultar backend)**

```dart
// lib/features/beacon/data/beacon_context_service.dart
import '../domain/beacon_info.dart';

class BeaconContextService {
  // Em produção, trocar por chamada HTTP ao backend
  Future<BeaconContext?> buscarContexto(BeaconInfo beacon) async {
    await Future.delayed(const Duration(milliseconds: 100));

    // Simulação de dados do servidor
    final contextos = {
      '1-1': BeaconContext(
        local: 'Entrada Principal',
        mensagem: 'Bem-vindo! Confira nossas promoções.',
        acao: 'abrir_promocoes',
      ),
      '1-2': BeaconContext(
        local: 'Setor de Eletrônicos',
        mensagem: 'Smartphones com até 30% de desconto.',
        acao: 'abrir_categoria',
        imagemUrl: 'https://picsum.photos/seed/eletronicos/400/200',
      ),
      '1-3': BeaconContext(
        local: 'Caixa',
        mensagem: 'Pague com o app e ganhe cashback!',
        acao: 'abrir_pagamento',
      ),
    };

    final chave = '${beacon.major}-${beacon.minor}';
    return contextos[chave];
  }
}
```

**7. Tela de beacons**

```dart
// lib/features/beacon/presentation/beacon_screen.dart
import 'dart:async';
import 'package:flutter/material.dart';
import '../data/beacon_service.dart';
import '../data/beacon_context_service.dart';
import '../domain/beacon_info.dart';

class BeaconScreen extends StatefulWidget {
  const BeaconScreen({super.key});

  @override
  State<BeaconScreen> createState() => _BeaconScreenState();
}

class _BeaconScreenState extends State<BeaconScreen> {
  final _beaconService = BeaconService();
  final _contextService = BeaconContextService();

  // UUID do projeto — todos os beacons da empresa usam o mesmo UUID
  static const _uuid = 'f7826da6-4fa2-4e98-8024-bc5b71e0893e';

  List<BeaconInfo> _beacons = [];
  List<String> _eventos = [];
  bool _escaneando = false;
  bool _inicializado = false;
  String? _erro;

  // Cache de contextos já consultados
  final Map<String, BeaconContext> _contextosCache = {};

  StreamSubscription? _beaconsSub;
  StreamSubscription? _eventosSub;

  @override
  void initState() {
    super.initState();
    _inicializar();
  }

  Future<void> _inicializar() async {
    final ok = await _beaconService.inicializar();
    setState(() {
      _inicializado = ok;
      if (!ok) _erro = 'Permissões negadas ou Bluetooth desativado.';
    });

    if (ok) {
      _beaconsSub = _beaconService.beacons.listen((lista) async {
        for (final b in lista) {
          final chave = '${b.major}-${b.minor}';
          if (!_contextosCache.containsKey(chave)) {
            final ctx = await _contextService.buscarContexto(b);
            if (ctx != null) _contextosCache[chave] = ctx;
          }
        }
        if (mounted) setState(() => _beacons = lista);
      });

      _eventosSub = _beaconService.eventos.listen((evento) {
        if (mounted) {
          setState(() => _eventos = [
            '${TimeOfDay.now().format(context)} — $evento',
            ..._eventos,
          ]);
        }
      });
    }
  }

  void _toggleScanning() {
    if (_escaneando) {
      _beaconService.pararRanging();
      _beaconService.pararMonitoring();
      setState(() {
        _escaneando = false;
        _beacons = [];
      });
    } else {
      _beaconService.iniciarRanging(uuid: _uuid);
      _beaconService.iniciarMonitoring(uuid: _uuid);
      setState(() => _escaneando = true);
    }
  }

  Color _corProximidade(String proximidade) {
    return switch (proximidade) {
      'immediate' => Colors.green,
      'near' => Colors.orange,
      'far' => Colors.red,
      _ => Colors.grey,
    };
  }

  String _labelProximidade(String proximidade) {
    return switch (proximidade) {
      'immediate' => 'Muito perto (< 0.5 m)',
      'near' => 'Perto (0.5 – 3 m)',
      'far' => 'Longe (> 3 m)',
      _ => 'Desconhecida',
    };
  }

  @override
  void dispose() {
    _beaconsSub?.cancel();
    _eventosSub?.cancel();
    _beaconService.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Beacons')),
      body: !_inicializado
          ? Center(
              child: _erro != null
                  ? Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        const Icon(Icons.bluetooth_disabled,
                            size: 64, color: Colors.red),
                        const SizedBox(height: 16),
                        Text(_erro!, textAlign: TextAlign.center),
                      ],
                    )
                  : const CircularProgressIndicator(),
            )
          : Column(
              children: [
                // Controle de scanning
                Container(
                  padding: const EdgeInsets.all(16),
                  color: _escaneando
                      ? Colors.blue.shade50
                      : Theme.of(context).colorScheme.surface,
                  child: Row(
                    children: [
                      Icon(
                        _escaneando ? Icons.bluetooth_searching : Icons.bluetooth,
                        color: _escaneando ? Colors.blue : Colors.grey,
                      ),
                      const SizedBox(width: 12),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              _escaneando ? 'Escaneando…' : 'Scanner parado',
                              style: const TextStyle(fontWeight: FontWeight.w600),
                            ),
                            Text(
                              _escaneando
                                  ? '${_beacons.length} beacon(s) detectado(s)'
                                  : 'Toque para iniciar',
                              style: Theme.of(context).textTheme.bodySmall,
                            ),
                          ],
                        ),
                      ),
                      FilledButton(
                        onPressed: _toggleScanning,
                        style: FilledButton.styleFrom(
                          backgroundColor: _escaneando ? Colors.red : null,
                        ),
                        child: Text(_escaneando ? 'Parar' : 'Iniciar'),
                      ),
                    ],
                  ),
                ),

                // Lista de beacons detectados
                if (_beacons.isNotEmpty)
                  Expanded(
                    flex: 3,
                    child: ListView.builder(
                      padding: const EdgeInsets.all(16),
                      itemCount: _beacons.length,
                      itemBuilder: (_, i) {
                        final b = _beacons[i];
                        final chave = '${b.major}-${b.minor}';
                        final ctx = _contextosCache[chave];

                        return Card(
                          margin: const EdgeInsets.only(bottom: 12),
                          child: Padding(
                            padding: const EdgeInsets.all(16),
                            child: Column(
                              crossAxisAlignment: CrossAxisAlignment.start,
                              children: [
                                // Identificação + proximidade
                                Row(
                                  children: [
                                    Container(
                                      width: 12, height: 12,
                                      decoration: BoxDecoration(
                                        shape: BoxShape.circle,
                                        color: _corProximidade(b.proximidade),
                                      ),
                                    ),
                                    const SizedBox(width: 8),
                                    Expanded(
                                      child: Text(
                                        ctx?.local ?? 'Beacon $chave',
                                        style: const TextStyle(
                                          fontWeight: FontWeight.w600, fontSize: 15),
                                      ),
                                    ),
                                    Chip(
                                      label: Text(
                                        '${b.distancia.toStringAsFixed(1)} m',
                                        style: const TextStyle(fontSize: 11),
                                      ),
                                      visualDensity: VisualDensity.compact,
                                    ),
                                  ],
                                ),
                                const SizedBox(height: 8),

                                // Proximidade
                                Text(
                                  _labelProximidade(b.proximidade),
                                  style: TextStyle(
                                    fontSize: 12,
                                    color: _corProximidade(b.proximidade),
                                    fontWeight: FontWeight.w500,
                                  ),
                                ),

                                // Contexto do servidor
                                if (ctx != null) ...[
                                  const SizedBox(height: 8),
                                  Text(ctx.mensagem,
                                      style: Theme.of(context).textTheme.bodyMedium),
                                ],

                                // Dados técnicos
                                const SizedBox(height: 8),
                                Text(
                                  'Major: ${b.major} | Minor: ${b.minor} | RSSI: ${b.rssi} dBm',
                                  style: TextStyle(
                                    fontSize: 11, color: Colors.grey.shade600),
                                ),
                              ],
                            ),
                          ),
                        );
                      },
                    ),
                  ),

                // Log de eventos (monitoring)
                if (_eventos.isNotEmpty) ...[
                  const Divider(height: 1),
                  Padding(
                    padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
                    child: Text('Eventos de região',
                        style: Theme.of(context).textTheme.titleSmall),
                  ),
                  Expanded(
                    flex: 1,
                    child: ListView.builder(
                      padding: const EdgeInsets.symmetric(horizontal: 16),
                      itemCount: _eventos.length,
                      itemBuilder: (_, i) => Text(
                        _eventos[i],
                        style: TextStyle(fontSize: 12, color: Colors.grey.shade700),
                      ),
                    ),
                  ),
                ],

                // Estado vazio
                if (_escaneando && _beacons.isEmpty)
                  const Expanded(
                    child: Center(
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          CircularProgressIndicator(),
                          SizedBox(height: 16),
                          Text('Procurando beacons…',
                              style: TextStyle(color: Colors.grey)),
                        ],
                      ),
                    ),
                  ),
              ],
            ),
    );
  }
}
```

---

### React Native — react-native-ble-plx + beacons

**1. Dependências**

```bash
# Para projetos Expo (requer development build, não funciona no Expo Go)
npx expo install react-native-ble-plx
npx expo install expo-location
```

> **Nota:** `react-native-ble-plx` requer um Expo **development build** (`npx expo prebuild` + `npx expo run:android`). Não funciona no Expo Go porque precisa de código nativo.

**2. Configuração — `app.json`**

```json
{
  "expo": {
    "plugins": [
      [
        "react-native-ble-plx",
        {
          "isBackgroundEnabled": true,
          "modes": ["peripheral", "central"],
          "bluetoothAlwaysPermission": "Usamos Bluetooth para detectar beacons próximos."
        }
      ]
    ],
    "ios": {
      "infoPlist": {
        "NSBluetoothAlwaysUsageDescription": "Usamos Bluetooth para detectar beacons próximos.",
        "NSLocationWhenInUseUsageDescription": "Precisamos da localização para detectar beacons.",
        "UIBackgroundModes": ["bluetooth-central", "location"]
      }
    },
    "android": {
      "permissions": [
        "BLUETOOTH", "BLUETOOTH_ADMIN", "BLUETOOTH_SCAN",
        "BLUETOOTH_CONNECT", "ACCESS_FINE_LOCATION"
      ]
    }
  }
}
```

**3. Parser de beacon (iBeacon)**

```typescript
// src/core/bluetooth/ibeaconParser.ts

export interface IBeaconData {
  uuid: string;
  major: number;
  minor: number;
  txPower: number;
  rssi: number;
  distancia: number;
  proximidade: 'immediate' | 'near' | 'far' | 'unknown';
}

// iBeacon usa manufacturer data com company ID 0x004C (Apple)
export function parseIBeacon(
  manufacturerData: string | null,
  rssi: number,
): IBeaconData | null {
  if (!manufacturerData) return null;

  const bytes = base64ToBytes(manufacturerData);
  // iBeacon: 2 bytes company (4C 00) + 2 bytes type (02 15) + 16 bytes UUID + 2 major + 2 minor + 1 txPower
  if (bytes.length < 23) return null;

  // Verificar header iBeacon
  if (bytes[0] !== 0x02 || bytes[1] !== 0x15) return null;

  const uuid = [
    bytesToHex(bytes, 2, 6),
    bytesToHex(bytes, 6, 8),
    bytesToHex(bytes, 8, 10),
    bytesToHex(bytes, 10, 12),
    bytesToHex(bytes, 12, 18),
  ].join('-');

  const major = (bytes[18] << 8) | bytes[19];
  const minor = (bytes[20] << 8) | bytes[21];
  const txPower = bytes[22] > 127 ? bytes[22] - 256 : bytes[22];

  const distancia = calcularDistancia(rssi, txPower);

  return {
    uuid: uuid.toLowerCase(),
    major,
    minor,
    txPower,
    rssi,
    distancia,
    proximidade:
      distancia < 0.5 ? 'immediate' :
      distancia < 3 ? 'near' :
      distancia < 30 ? 'far' : 'unknown',
  };
}

function calcularDistancia(rssi: number, txPower: number): number {
  if (rssi === 0) return -1;
  const ratio = (txPower - rssi) / (10 * 2.0);
  return Math.pow(10, ratio);
}

function base64ToBytes(base64: string): Uint8Array {
  const binary = atob(base64);
  return Uint8Array.from(binary, (c) => c.charCodeAt(0));
}

function bytesToHex(bytes: Uint8Array, start: number, end: number): string {
  return Array.from(bytes.slice(start, end))
    .map((b) => b.toString(16).padStart(2, '0'))
    .join('');
}
```

**4. Hook de scanning de beacons**

```typescript
// src/hooks/useBeaconScanner.ts
import { useEffect, useRef, useState, useCallback } from 'react';
import { BleManager, Device, State } from 'react-native-ble-plx';
import { Platform, PermissionsAndroid } from 'react-native';
import { IBeaconData, parseIBeacon } from '../core/bluetooth/ibeaconParser';

const TARGET_UUID = 'f7826da6-4fa2-4e98-8024-bc5b71e0893e';

export function useBeaconScanner() {
  const managerRef = useRef(new BleManager());
  const [beacons, setBeacons] = useState<Map<string, IBeaconData>>(new Map());
  const [escaneando, setEscaneando] = useState(false);
  const [erro, setErro] = useState<string | null>(null);
  const [inicializado, setInicializado] = useState(false);

  useEffect(() => {
    const manager = managerRef.current;

    const subscription = manager.onStateChange((state) => {
      if (state === State.PoweredOn) {
        setInicializado(true);
        subscription.remove();
      }
    }, true);

    return () => {
      subscription.remove();
      manager.stopDeviceScan();
      manager.destroy();
    };
  }, []);

  const solicitarPermissoes = useCallback(async (): Promise<boolean> => {
    if (Platform.OS === 'android') {
      const result = await PermissionsAndroid.requestMultiple([
        PermissionsAndroid.PERMISSIONS.BLUETOOTH_SCAN,
        PermissionsAndroid.PERMISSIONS.BLUETOOTH_CONNECT,
        PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
      ]);
      return Object.values(result).every(
        (r) => r === PermissionsAndroid.RESULTS.GRANTED,
      );
    }
    return true; // iOS solicita automaticamente
  }, []);

  const iniciar = useCallback(async () => {
    const permitido = await solicitarPermissoes();
    if (!permitido) {
      setErro('Permissões negadas.');
      return;
    }

    setEscaneando(true);
    setErro(null);

    managerRef.current.startDeviceScan(
      null,
      { allowDuplicates: true },
      (error, device) => {
        if (error) {
          setErro(error.message);
          setEscaneando(false);
          return;
        }

        if (!device) return;

        const beacon = parseIBeacon(device.manufacturerData, device.rssi ?? -100);
        if (!beacon) return;
        if (beacon.uuid !== TARGET_UUID) return;

        const chave = `${beacon.major}-${beacon.minor}`;
        setBeacons((prev) => {
          const novo = new Map(prev);
          novo.set(chave, beacon);
          return novo;
        });
      },
    );
  }, [solicitarPermissoes]);

  const parar = useCallback(() => {
    managerRef.current.stopDeviceScan();
    setEscaneando(false);
    setBeacons(new Map());
  }, []);

  return {
    beacons: Array.from(beacons.values()).sort((a, b) => a.distancia - b.distancia),
    escaneando,
    erro,
    inicializado,
    iniciar,
    parar,
  };
}
```

**5. Tela de beacons**

```tsx
// src/features/beacon/BeaconScreen.tsx
import React from 'react';
import {
  View, Text, FlatList, TouchableOpacity, StyleSheet, ActivityIndicator,
} from 'react-native';
import { useBeaconScanner } from '../../hooks/useBeaconScanner';
import { IBeaconData } from '../../core/bluetooth/ibeaconParser';

const CONTEXTOS: Record<string, { local: string; mensagem: string }> = {
  '1-1': { local: 'Entrada Principal', mensagem: 'Bem-vindo! Confira nossas promoções.' },
  '1-2': { local: 'Setor de Eletrônicos', mensagem: 'Smartphones com até 30% de desconto.' },
  '1-3': { local: 'Caixa', mensagem: 'Pague com o app e ganhe cashback!' },
};

export function BeaconScreen() {
  const { beacons, escaneando, erro, inicializado, iniciar, parar } =
    useBeaconScanner();

  const corProximidade = (p: string) =>
    p === 'immediate' ? '#4caf50' : p === 'near' ? '#ff9800' : p === 'far' ? '#f44336' : '#999';

  const labelProximidade = (p: string) =>
    p === 'immediate' ? 'Muito perto' : p === 'near' ? 'Perto' : p === 'far' ? 'Longe' : '?';

  if (!inicializado) {
    return (
      <View style={styles.center}>
        {erro ? (
          <>
            <Text style={styles.erroIcon}>📵</Text>
            <Text style={styles.erro}>{erro}</Text>
          </>
        ) : (
          <ActivityIndicator size="large" />
        )}
      </View>
    );
  }

  return (
    <View style={styles.container}>
      {/* Controle */}
      <View style={[styles.header, escaneando && styles.headerAtivo]}>
        <View style={styles.headerInfo}>
          <Text style={styles.headerTitulo}>
            {escaneando ? `Escaneando… (${beacons.length})` : 'Scanner parado'}
          </Text>
          <Text style={styles.headerSub}>
            {escaneando ? 'Detectando beacons BLE' : 'Toque para iniciar'}
          </Text>
        </View>
        <TouchableOpacity
          style={[styles.btnToggle, escaneando && styles.btnParar]}
          onPress={escaneando ? parar : iniciar}
        >
          <Text style={styles.btnToggleTexto}>{escaneando ? 'Parar' : 'Iniciar'}</Text>
        </TouchableOpacity>
      </View>

      {erro && <Text style={styles.erroBar}>{erro}</Text>}

      {/* Lista */}
      {escaneando && beacons.length === 0 ? (
        <View style={styles.center}>
          <ActivityIndicator size="large" color="#3949ab" />
          <Text style={styles.procurando}>Procurando beacons…</Text>
        </View>
      ) : (
        <FlatList
          data={beacons}
          keyExtractor={(item) => `${item.major}-${item.minor}`}
          contentContainerStyle={styles.lista}
          renderItem={({ item }) => {
            const chave = `${item.major}-${item.minor}`;
            const ctx = CONTEXTOS[chave];
            return (
              <View style={styles.card}>
                <View style={styles.cardHeader}>
                  <View style={[styles.dot, { backgroundColor: corProximidade(item.proximidade) }]} />
                  <Text style={styles.cardTitulo}>{ctx?.local ?? `Beacon ${chave}`}</Text>
                  <View style={styles.distBadge}>
                    <Text style={styles.distTexto}>{item.distancia.toFixed(1)} m</Text>
                  </View>
                </View>
                <Text style={[styles.proxLabel, { color: corProximidade(item.proximidade) }]}>
                  {labelProximidade(item.proximidade)}
                </Text>
                {ctx && <Text style={styles.mensagem}>{ctx.mensagem}</Text>}
                <Text style={styles.tecnico}>
                  Major: {item.major} | Minor: {item.minor} | RSSI: {item.rssi} dBm
                </Text>
              </View>
            );
          }}
        />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f9f9f9' },
  center: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  header: { flexDirection: 'row', alignItems: 'center', padding: 16, backgroundColor: '#fff', borderBottomWidth: 1, borderBottomColor: '#e0e0e0' },
  headerAtivo: { backgroundColor: '#e3f2fd' },
  headerInfo: { flex: 1 },
  headerTitulo: { fontWeight: '600', fontSize: 15 },
  headerSub: { fontSize: 12, color: '#999', marginTop: 2 },
  btnToggle: { backgroundColor: '#3949ab', paddingHorizontal: 20, paddingVertical: 10, borderRadius: 8 },
  btnParar: { backgroundColor: '#e53935' },
  btnToggleTexto: { color: '#fff', fontWeight: '600' },
  erroIcon: { fontSize: 48, marginBottom: 12 },
  erro: { color: '#e53935', textAlign: 'center', paddingHorizontal: 32 },
  erroBar: { backgroundColor: '#ffebee', padding: 12, color: '#c62828', fontSize: 13, textAlign: 'center' },
  procurando: { color: '#999', marginTop: 16 },
  lista: { padding: 16 },
  card: { backgroundColor: '#fff', borderRadius: 12, padding: 16, marginBottom: 12, elevation: 1 },
  cardHeader: { flexDirection: 'row', alignItems: 'center', marginBottom: 6 },
  dot: { width: 10, height: 10, borderRadius: 5, marginRight: 8 },
  cardTitulo: { flex: 1, fontWeight: '600', fontSize: 15 },
  distBadge: { backgroundColor: '#f0f0f0', paddingHorizontal: 8, paddingVertical: 2, borderRadius: 12 },
  distTexto: { fontSize: 12, fontWeight: '500' },
  proxLabel: { fontSize: 12, fontWeight: '500', marginBottom: 4 },
  mensagem: { fontSize: 14, color: '#333', marginTop: 4, marginBottom: 6 },
  tecnico: { fontSize: 11, color: '#aaa' },
});
```

---

### React Web — Web Bluetooth API

> **Limitações importantes:**
> - Suporte apenas em **Chrome, Edge e Opera** (desktop e Android). **Não funciona no Safari, Firefox ou iOS**.
> - Requer **HTTPS** (exceto `localhost`).
> - Não consegue fazer scan passivo de beacons iBeacon/Eddystone (precisa de interação do usuário para cada conexão). Para uso real de beacons no Web, a alternativa é usar a **Physical Web** (Eddystone-URL) ou um **gateway BLE** que encaminha dados de beacons para um servidor, e o frontend consulta via API.
> - O exemplo abaixo demonstra scan BLE genérico com filtro por serviço e leitura de dados, aplicável a dispositivos BLE interativos (sensores, wearables).

**1. Serviço BLE**

```typescript
// src/core/bluetooth/bleService.ts

export interface BLEDevice {
  id: string;
  nome: string;
  rssi: number | null;
  conectado: boolean;
  servicos: string[];
}

export function isBLEDisponivel(): boolean {
  return 'bluetooth' in navigator;
}

// Solicitar dispositivo BLE (requer gesto do usuário — click)
export async function solicitarDispositivo(
  filtroServicos?: string[],
): Promise<BluetoothDevice | null> {
  if (!isBLEDisponivel()) return null;

  try {
    const device = await navigator.bluetooth.requestDevice({
      acceptAllDevices: filtroServicos ? undefined : true,
      filters: filtroServicos
        ? filtroServicos.map((s) => ({ services: [s] }))
        : undefined,
      optionalServices: filtroServicos ?? [],
    });
    return device;
  } catch {
    return null;
  }
}

// Conectar e listar serviços
export async function conectar(
  device: BluetoothDevice,
): Promise<BluetoothRemoteGATTServer | null> {
  try {
    const server = await device.gatt!.connect();
    return server;
  } catch {
    return null;
  }
}

// Ler característica de um serviço
export async function lerCaracteristica(
  server: BluetoothRemoteGATTServer,
  serviceUUID: string,
  characteristicUUID: string,
): Promise<DataView | null> {
  try {
    const service = await server.getPrimaryService(serviceUUID);
    const characteristic = await service.getCharacteristic(characteristicUUID);
    return await characteristic.readValue();
  } catch {
    return null;
  }
}

// Inscrever para notificações de uma característica
export async function inscreverNotificacoes(
  server: BluetoothRemoteGATTServer,
  serviceUUID: string,
  characteristicUUID: string,
  onValor: (value: DataView) => void,
): Promise<() => void> {
  const service = await server.getPrimaryService(serviceUUID);
  const characteristic = await service.getCharacteristic(characteristicUUID);
  await characteristic.startNotifications();

  const handler = (event: Event) => {
    const target = event.target as BluetoothRemoteGATTCharacteristic;
    if (target.value) onValor(target.value);
  };
  characteristic.addEventListener('characteristicvaluechanged', handler);

  return () => {
    characteristic.stopNotifications();
    characteristic.removeEventListener('characteristicvaluechanged', handler);
  };
}
```

**2. Hook de BLE**

```typescript
// src/hooks/useBLE.ts
import { useState, useCallback, useRef } from 'react';
import {
  isBLEDisponivel,
  solicitarDispositivo,
  conectar,
  lerCaracteristica,
  BLEDevice,
} from '../core/bluetooth/bleService';

export function useBLE() {
  const [disponivel] = useState(isBLEDisponivel());
  const [dispositivo, setDispositivo] = useState<BLEDevice | null>(null);
  const [conectando, setConectando] = useState(false);
  const [dados, setDados] = useState<Record<string, string>>({});
  const [erro, setErro] = useState<string | null>(null);
  const deviceRef = useRef<BluetoothDevice | null>(null);
  const serverRef = useRef<BluetoothRemoteGATTServer | null>(null);

  const procurar = useCallback(async () => {
    setErro(null);
    const device = await solicitarDispositivo();
    if (!device) return;

    deviceRef.current = device;
    setDispositivo({
      id: device.id,
      nome: device.name ?? 'Desconhecido',
      rssi: null,
      conectado: false,
      servicos: [],
    });

    device.addEventListener('gattserverdisconnected', () => {
      setDispositivo((prev) => prev ? { ...prev, conectado: false } : null);
      serverRef.current = null;
    });
  }, []);

  const conectarDispositivo = useCallback(async () => {
    if (!deviceRef.current) return;
    setConectando(true);
    setErro(null);

    const server = await conectar(deviceRef.current);
    setConectando(false);

    if (!server) {
      setErro('Falha ao conectar.');
      return;
    }

    serverRef.current = server;

    try {
      const services = await server.getPrimaryServices();
      const uuids = services.map((s) => s.uuid);
      setDispositivo((prev) => prev ? { ...prev, conectado: true, servicos: uuids } : null);
    } catch {
      setDispositivo((prev) => prev ? { ...prev, conectado: true, servicos: [] } : null);
    }
  }, []);

  const lerServico = useCallback(async (serviceUUID: string, charUUID: string) => {
    if (!serverRef.current) return;
    const value = await lerCaracteristica(serverRef.current, serviceUUID, charUUID);
    if (value) {
      const decoder = new TextDecoder();
      setDados((prev) => ({ ...prev, [charUUID]: decoder.decode(value) }));
    }
  }, []);

  const desconectar = useCallback(() => {
    deviceRef.current?.gatt?.disconnect();
    setDispositivo(null);
    setDados({});
    serverRef.current = null;
    deviceRef.current = null;
  }, []);

  return {
    disponivel, dispositivo, conectando, dados, erro,
    procurar, conectarDispositivo, lerServico, desconectar,
  };
}
```

**3. Página BLE**

```tsx
// src/features/beacon/BLEPage.tsx
import { useBLE } from '../../hooks/useBLE';

export function BLEPage() {
  const {
    disponivel, dispositivo, conectando, dados, erro,
    procurar, conectarDispositivo, lerServico, desconectar,
  } = useBLE();

  if (!disponivel) {
    return (
      <div className="container py-5 text-center">
        <i className="fas fa-exclamation-triangle fa-3x text-warning mb-3" />
        <h5>Web Bluetooth não disponível</h5>
        <p className="text-muted">
          Use Chrome, Edge ou Opera em HTTPS. Não suportado em Safari, Firefox ou iOS.
        </p>

        <div className="alert alert-info text-start mt-4" style={{ maxWidth: 600, margin: '0 auto' }}>
          <h6><i className="fas fa-lightbulb me-2" />Alternativa para Beacons no Web</h6>
          <p className="mb-0">
            Para detectar beacons iBeacon/Eddystone no Web, use um <strong>gateway BLE</strong>
            (dispositivo que escaneia beacons e envia dados para um servidor via WiFi/Ethernet).
            O frontend consulta o servidor via API REST ou WebSocket para receber atualizações
            de proximidade em tempo real.
          </p>
        </div>
      </div>
    );
  }

  return (
    <div className="container py-4" style={{ maxWidth: 700 }}>
      <h4 className="mb-4">
        <i className="fab fa-bluetooth-b text-primary me-2" />
        Bluetooth Low Energy
      </h4>

      {/* Aviso sobre beacons */}
      <div className="alert alert-warning small">
        <i className="fas fa-info-circle me-1" />
        A Web Bluetooth API não faz scan passivo de beacons. O exemplo abaixo demonstra
        conexão com dispositivos BLE interativos (sensores, wearables). Para beacons,
        considere um gateway BLE + API REST.
      </div>

      {erro && (
        <div className="alert alert-danger">
          <i className="fas fa-times-circle me-1" />{erro}
        </div>
      )}

      {/* Sem dispositivo */}
      {!dispositivo && (
        <div className="text-center py-5">
          <i className="fab fa-bluetooth fa-4x text-muted mb-3" />
          <p className="text-muted">Nenhum dispositivo conectado</p>
          <button className="btn btn-primary btn-lg" onClick={procurar}>
            <i className="fas fa-search me-2" />
            Procurar dispositivo BLE
          </button>
        </div>
      )}

      {/* Dispositivo encontrado */}
      {dispositivo && (
        <div className="card">
          <div className="card-body">
            <div className="d-flex align-items-center mb-3">
              <i className={`fab fa-bluetooth-b fa-2x me-3 ${dispositivo.conectado ? 'text-primary' : 'text-muted'}`} />
              <div className="flex-grow-1">
                <h6 className="mb-0">{dispositivo.nome}</h6>
                <small className="text-muted">ID: {dispositivo.id}</small>
              </div>
              <span className={`badge ${dispositivo.conectado ? 'bg-success' : 'bg-secondary'}`}>
                {dispositivo.conectado ? 'Conectado' : 'Desconectado'}
              </span>
            </div>

            <div className="d-flex gap-2 mb-3">
              {!dispositivo.conectado ? (
                <button
                  className="btn btn-primary"
                  onClick={conectarDispositivo}
                  disabled={conectando}
                >
                  {conectando && <span className="spinner-border spinner-border-sm me-2" />}
                  Conectar
                </button>
              ) : (
                <button className="btn btn-outline-danger" onClick={desconectar}>
                  Desconectar
                </button>
              )}
              <button className="btn btn-outline-secondary" onClick={procurar}>
                Trocar dispositivo
              </button>
            </div>

            {/* Serviços */}
            {dispositivo.conectado && dispositivo.servicos.length > 0 && (
              <>
                <h6 className="mt-3">Serviços GATT</h6>
                <ul className="list-group list-group-flush">
                  {dispositivo.servicos.map((uuid) => (
                    <li key={uuid} className="list-group-item d-flex align-items-center">
                      <code className="flex-grow-1 small">{uuid}</code>
                    </li>
                  ))}
                </ul>
              </>
            )}

            {/* Dados lidos */}
            {Object.keys(dados).length > 0 && (
              <>
                <h6 className="mt-3">Dados lidos</h6>
                <table className="table table-sm">
                  <tbody>
                    {Object.entries(dados).map(([uuid, valor]) => (
                      <tr key={uuid}>
                        <td><code className="small">{uuid}</code></td>
                        <td>{valor}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </>
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

---

### Equivalência — Bluetooth / Beacons

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Biblioteca | `flutter_beacon` | `react-native-ble-plx` | Web Bluetooth API |
| Scan passivo (beacons) | `flutterBeacon.ranging()` | `startDeviceScan` + parser iBeacon | **Não suportado** (scan passivo proibido) |
| Ranging (distância) | Built-in (`accuracy` + `proximity`) | Manual (RSSI → fórmula) | N/A |
| Monitoring (entra/sai) | `flutterBeacon.monitoring()` | Manual (comparar scans) | N/A |
| Conexão GATT | Não (foco em beacons) | `device.connect()` | `device.gatt.connect()` |
| Ler característica | — | `readCharacteristic()` | `characteristic.readValue()` |
| Background scanning | iOS: sim, Android: limitado | iOS: sim (com config), Android: limitado | Não |
| Permissão Bluetooth | `BLUETOOTH_SCAN` + manifest | `BLUETOOTH_SCAN` + `app.json` | Prompt automático |
| Permissão Location | Obrigatória (Android) | Obrigatória (Android) | Não necessária |
| Suporte a iBeacon | Nativo (parser built-in) | Manual (parser customizado) | Não suportado |
| Suporte a Eddystone | Nativo | Manual (parser customizado) | Não suportado |
| Funciona no Expo Go | Não (precisa build nativo) | Não (precisa development build) | — |

### Alternativas para Beacons no Web

| Abordagem | Como funciona | Quando usar |
|---|---|---|
| **Gateway BLE → API REST** | Um dispositivo (Raspberry Pi, gateway comercial) escaneia beacons localmente e publica dados em servidor. O frontend consulta via REST/WebSocket. | Instalações fixas (lojas, museus) |
| **Eddystone-URL (Physical Web)** | Beacon transmite uma URL curta. Chrome Android detecta automaticamente (via notificação). | Cenários simples sem app customizado |
| **PWA + Gateway** | Combina gateway BLE com uma PWA que recebe atualizações via Server-Sent Events ou WebSocket. | Quando não é possível instalar app nativo |

### Fluxo típico de produção

```
┌──────────┐     BLE      ┌──────────────┐     WiFi/4G     ┌──────────────┐
│  Beacon  │  ─────────►  │  Smartphone  │  ─────────────► │   Backend    │
│ (iBeacon │   broadcast  │  (app nativo)│  POST /checkin  │   (REST API) │
│  / Eddy) │              │              │                 │              │
└──────────┘              └──────────────┘                 └──────┬───────┘
                                                                  │
                           ┌──────────────┐                       │
                           │  Dashboard   │ ◄─────────────────────┘
                           │  (React Web) │   GET /api/presencas
                           └──────────────┘
```

1. **Beacon** transmite UUID + major + minor continuamente.
2. **App nativo** (Flutter ou React Native) detecta o beacon via BLE.
3. App envia dados de presença/contexto ao **backend** (POST com beacon ID + timestamp + user).
4. **Dashboard web** consulta o backend para exibir dados de presença, heatmaps, etc.

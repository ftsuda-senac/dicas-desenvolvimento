# Frontend — Exemplos Avançados (Parte 2)

Continuação do documento [Frontend-Exemplos.md](Frontend-Exemplos.md), com mais implementações equivalentes entre Flutter, React Native e React Web.

**Convenções mantidas do documento anterior:**

- **React Web:** TypeScript, Bootstrap CSS (classes diretas), Font Awesome (classes `fas fa-*`), `react-bootstrap` apenas para componentes dinâmicos.
- **Flutter:** Material 3, Riverpod para estado quando necessário.
- **React Native:** Expo (com development build quando necessário).

---

## Sumário

1. [Drag & Drop — Reordenação de Listas](#1-drag--drop--reordenação-de-listas)
   - [Flutter — ReorderableListView](#flutter--reorderablelistview)
   - [React Native — react-native-draggable-flatlist](#react-native--react-native-draggable-flatlist)
   - [React Web — HTML5 Drag and Drop + Bootstrap](#react-web--html5-drag-and-drop--bootstrap)
2. [Animações e Transições](#2-animações-e-transições)
   - [Flutter — AnimationController + Hero](#flutter--animationcontroller--hero)
   - [React Native — Animated + react-native-reanimated](#react-native--animated--react-native-reanimated)
   - [React Web — CSS Transitions + Keyframes](#react-web--css-transitions--keyframes)
3. [Busca com Autocomplete e Debounce](#3-busca-com-autocomplete-e-debounce)
   - [Flutter — SearchDelegate + Timer](#flutter--searchdelegate--timer)
   - [React Native — TextInput + debounce](#react-native--textinput--debounce)
   - [React Web — Bootstrap + debounce](#react-web--bootstrap--debounce)
4. [Compartilhamento e Deep Links](#4-compartilhamento-e-deep-links)
   - [Flutter — share_plus + GoRouter deep links](#flutter--share_plus--gorouter-deep-links)
   - [React Native — expo-sharing + expo-linking](#react-native--expo-sharing--expo-linking)
   - [React Web — Web Share API + React Router](#react-web--web-share-api--react-router)
5. [Reprodução de Áudio / Podcast Player](#5-reprodução-de-áudio--podcast-player)
   - [Flutter — just_audio](#flutter--just_audio)
   - [React Native — expo-av](#react-native--expo-av)
   - [React Web — HTML5 Audio API + Bootstrap](#react-web--html5-audio-api--bootstrap)
6. [Acesso a Contatos e Calendário](#6-acesso-a-contatos-e-calendário)
   - [Flutter — flutter_contacts + add_2_calendar](#flutter--flutter_contacts--add_2_calendar)
   - [React Native — expo-contacts + expo-calendar](#react-native--expo-contacts--expo-calendar)
   - [React Web — Contact Picker API + download .ics](#react-web--contact-picker-api--download-ics)
7. [Upload de Arquivos com Progresso](#7-upload-de-arquivos-com-progresso)
   - [Flutter — Dio + onSendProgress](#flutter--dio--onsendprogress)
   - [React Native — XMLHttpRequest + progresso](#react-native--xmlhttprequest--progresso)
   - [React Web — XMLHttpRequest + Bootstrap Progress](#react-web--xmlhttprequest--bootstrap-progress)
8. [Cache de Imagens e Dados com Estratégia](#8-cache-de-imagens-e-dados-com-estratégia)
   - [Flutter — cached_network_image + Hive](#flutter--cached_network_image--hive)
   - [React Native — expo-image + MMKV](#react-native--expo-image--mmkv)
   - [React Web — Service Worker Cache API](#react-web--service-worker-cache-api)
9. [Paginação com Cursor](#9-paginação-com-cursor)
   - [Flutter — cursor + ListView](#flutter--cursor--listview)
   - [React Native — cursor + FlatList](#react-native--cursor--flatlist)
   - [React Web — cursor + IntersectionObserver](#react-web--cursor--intersectionobserver)
10. [Pagamentos In-App](#10-pagamentos-in-app)
    - [Flutter — flutter_stripe](#flutter--flutter_stripe)
    - [React Native — @stripe/stripe-react-native](#react-native--stripestripe-react-native)
    - [React Web — Stripe.js + Bootstrap](#react-web--stripejs--bootstrap)
11. [Criptografia de Dados Locais](#11-criptografia-de-dados-locais)
    - [Flutter — encrypt + flutter_secure_storage](#flutter--encrypt--flutter_secure_storage)
    - [React Native — expo-crypto + expo-secure-store](#react-native--expo-crypto--expo-secure-store)
    - [React Web — Web Crypto API](#react-web--web-crypto-api)
12. [Server-Sent Events (SSE)](#12-server-sent-events-sse)
    - [Flutter — http stream](#flutter--http-stream)
    - [React Native — EventSource polyfill](#react-native--eventsource-polyfill)
    - [React Web — EventSource API + Bootstrap](#react-web--eventsource-api--bootstrap)

---

## 1. Drag & Drop — Reordenação de Listas

Lista reordenável por arraste e quadro Kanban com movimentação entre colunas.

### Flutter — ReorderableListView

**1. Lista reordenável**

```dart
// lib/features/dragdrop/presentation/reorderable_list_screen.dart
import 'package:flutter/material.dart';

class ReorderableListScreen extends StatefulWidget {
  const ReorderableListScreen({super.key});

  @override
  State<ReorderableListScreen> createState() => _ReorderableListScreenState();
}

class _ReorderableListScreenState extends State<ReorderableListScreen> {
  final _itens = List.generate(
    10,
    (i) => _TarefaItem(id: 'tarefa-$i', titulo: 'Tarefa ${i + 1}'),
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Reordenar Tarefas')),
      body: ReorderableListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: _itens.length,
        onReorder: (oldIndex, newIndex) {
          setState(() {
            if (newIndex > oldIndex) newIndex--;
            final item = _itens.removeAt(oldIndex);
            _itens.insert(newIndex, item);
          });
        },
        itemBuilder: (context, i) {
          final item = _itens[i];
          return Card(
            key: ValueKey(item.id),
            margin: const EdgeInsets.only(bottom: 8),
            child: ListTile(
              leading: ReorderableDragStartListener(
                index: i,
                child: const Icon(Icons.drag_handle),
              ),
              title: Text(item.titulo),
              subtitle: Text('Posição: ${i + 1}'),
              trailing: IconButton(
                icon: const Icon(Icons.delete_outline, color: Colors.red),
                onPressed: () => setState(() => _itens.removeAt(i)),
              ),
            ),
          );
        },
      ),
    );
  }
}

class _TarefaItem {
  final String id;
  final String titulo;
  _TarefaItem({required this.id, required this.titulo});
}
```

**2. Quadro Kanban simplificado**

```dart
// lib/features/dragdrop/presentation/kanban_screen.dart
import 'package:flutter/material.dart';

class KanbanItem {
  final String id;
  final String titulo;
  KanbanItem({required this.id, required this.titulo});
}

class KanbanScreen extends StatefulWidget {
  const KanbanScreen({super.key});

  @override
  State<KanbanScreen> createState() => _KanbanScreenState();
}

class _KanbanScreenState extends State<KanbanScreen> {
  final Map<String, List<KanbanItem>> _colunas = {
    'A Fazer': [
      KanbanItem(id: '1', titulo: 'Definir escopo'),
      KanbanItem(id: '2', titulo: 'Criar wireframes'),
    ],
    'Fazendo': [
      KanbanItem(id: '3', titulo: 'Implementar login'),
    ],
    'Feito': [
      KanbanItem(id: '4', titulo: 'Configurar projeto'),
    ],
  };

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Kanban')),
      body: Row(
        children: _colunas.entries.map((entry) {
          return Expanded(
            child: DragTarget<MapEntry<String, KanbanItem>>(
              onAcceptWithDetails: (details) {
                setState(() {
                  final origem = details.data.key;
                  final item = details.data.value;
                  _colunas[origem]!.remove(item);
                  _colunas[entry.key]!.add(item);
                });
              },
              builder: (context, candidatos, rejeitados) {
                return Container(
                  margin: const EdgeInsets.all(8),
                  decoration: BoxDecoration(
                    color: candidatos.isNotEmpty
                        ? Colors.blue.shade50
                        : Colors.grey.shade100,
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Column(
                    children: [
                      Padding(
                        padding: const EdgeInsets.all(12),
                        child: Text(
                          '${entry.key} (${entry.value.length})',
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                      ),
                      const Divider(height: 1),
                      Expanded(
                        child: ListView(
                          padding: const EdgeInsets.all(8),
                          children: entry.value.map((item) {
                            return Draggable<MapEntry<String, KanbanItem>>(
                              data: MapEntry(entry.key, item),
                              feedback: Material(
                                elevation: 8,
                                borderRadius: BorderRadius.circular(8),
                                child: Container(
                                  width: 160,
                                  padding: const EdgeInsets.all(12),
                                  decoration: BoxDecoration(
                                    color: Colors.white,
                                    borderRadius: BorderRadius.circular(8),
                                  ),
                                  child: Text(item.titulo),
                                ),
                              ),
                              childWhenDragging: Opacity(
                                opacity: 0.3,
                                child: _KanbanCard(titulo: item.titulo),
                              ),
                              child: _KanbanCard(titulo: item.titulo),
                            );
                          }).toList(),
                        ),
                      ),
                    ],
                  ),
                );
              },
            ),
          );
        }).toList(),
      ),
    );
  }
}

class _KanbanCard extends StatelessWidget {
  final String titulo;
  const _KanbanCard({required this.titulo});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 8),
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Text(titulo),
      ),
    );
  }
}
```

---

### React Native — react-native-draggable-flatlist

**1. Dependências**

```bash
npm install react-native-draggable-flatlist
npx expo install react-native-gesture-handler react-native-reanimated
```

**2. Lista reordenável**

```tsx
// src/features/dragdrop/ReorderableListScreen.tsx
import React, { useState, useCallback } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import DraggableFlatList, {
  RenderItemParams,
  ScaleDecorator,
} from 'react-native-draggable-flatlist';
import { GestureHandlerRootView } from 'react-native-gesture-handler';

interface TarefaItem {
  id: string;
  titulo: string;
}

export function ReorderableListScreen() {
  const [itens, setItens] = useState<TarefaItem[]>(
    Array.from({ length: 10 }, (_, i) => ({
      id: `tarefa-${i}`,
      titulo: `Tarefa ${i + 1}`,
    })),
  );

  const renderItem = useCallback(
    ({ item, drag, isActive, getIndex }: RenderItemParams<TarefaItem>) => {
      const index = getIndex() ?? 0;
      return (
        <ScaleDecorator>
          <TouchableOpacity
            onLongPress={drag}
            disabled={isActive}
            style={[styles.card, isActive && styles.cardAtivo]}
          >
            <Text style={styles.dragHandle}>☰</Text>
            <View style={styles.cardInfo}>
              <Text style={styles.cardTitulo}>{item.titulo}</Text>
              <Text style={styles.cardSub}>Posição: {index + 1}</Text>
            </View>
          </TouchableOpacity>
        </ScaleDecorator>
      );
    },
    [],
  );

  return (
    <GestureHandlerRootView style={styles.container}>
      <DraggableFlatList
        data={itens}
        keyExtractor={(item) => item.id}
        renderItem={renderItem}
        onDragEnd={({ data }) => setItens(data)}
        contentContainerStyle={styles.lista}
      />
    </GestureHandlerRootView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f9f9f9' },
  lista: { padding: 16 },
  card: {
    flexDirection: 'row', alignItems: 'center', backgroundColor: '#fff',
    borderRadius: 10, padding: 14, marginBottom: 8, elevation: 1,
  },
  cardAtivo: { backgroundColor: '#e3f2fd', elevation: 4 },
  dragHandle: { fontSize: 20, color: '#999', marginRight: 12 },
  cardInfo: { flex: 1 },
  cardTitulo: { fontSize: 15, fontWeight: '600' },
  cardSub: { fontSize: 12, color: '#999', marginTop: 2 },
});
```

---

### React Web — HTML5 Drag and Drop + Bootstrap

**1. Lista reordenável**

```tsx
// src/features/dragdrop/ReorderableListPage.tsx
import { useState, useRef } from 'react';

interface TarefaItem {
  id: string;
  titulo: string;
}

export function ReorderableListPage() {
  const [itens, setItens] = useState<TarefaItem[]>(
    Array.from({ length: 10 }, (_, i) => ({
      id: `tarefa-${i}`,
      titulo: `Tarefa ${i + 1}`,
    })),
  );

  const dragIndexRef = useRef<number | null>(null);
  const [dragOverIndex, setDragOverIndex] = useState<number | null>(null);

  const onDragStart = (index: number) => {
    dragIndexRef.current = index;
  };

  const onDragOver = (e: React.DragEvent, index: number) => {
    e.preventDefault();
    setDragOverIndex(index);
  };

  const onDrop = (index: number) => {
    const from = dragIndexRef.current;
    if (from === null || from === index) {
      setDragOverIndex(null);
      return;
    }

    setItens((prev) => {
      const novo = [...prev];
      const [item] = novo.splice(from, 1);
      novo.splice(index, 0, item);
      return novo;
    });
    dragIndexRef.current = null;
    setDragOverIndex(null);
  };

  return (
    <div className="container py-4" style={{ maxWidth: 600 }}>
      <h4 className="mb-4">
        <i className="fas fa-grip-vertical me-2" />
        Reordenar Tarefas
      </h4>

      <ul className="list-group">
        {itens.map((item, i) => (
          <li
            key={item.id}
            className={`list-group-item d-flex align-items-center gap-3 ${
              dragOverIndex === i ? 'border-primary bg-primary-subtle' : ''
            }`}
            draggable
            onDragStart={() => onDragStart(i)}
            onDragOver={(e) => onDragOver(e, i)}
            onDrop={() => onDrop(i)}
            onDragEnd={() => setDragOverIndex(null)}
            style={{ cursor: 'grab' }}
          >
            <i className="fas fa-grip-vertical text-muted" />
            <div className="flex-grow-1">
              <strong>{item.titulo}</strong>
              <small className="text-muted d-block">Posição: {i + 1}</small>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**2. Quadro Kanban**

```tsx
// src/features/dragdrop/KanbanPage.tsx
import { useState, useRef } from 'react';

interface KanbanItem {
  id: string;
  titulo: string;
}

type Colunas = Record<string, KanbanItem[]>;

export function KanbanPage() {
  const [colunas, setColunas] = useState<Colunas>({
    'A Fazer': [
      { id: '1', titulo: 'Definir escopo' },
      { id: '2', titulo: 'Criar wireframes' },
    ],
    'Fazendo': [{ id: '3', titulo: 'Implementar login' }],
    'Feito': [{ id: '4', titulo: 'Configurar projeto' }],
  });

  const dragRef = useRef<{ coluna: string; item: KanbanItem } | null>(null);
  const [dropTarget, setDropTarget] = useState<string | null>(null);

  const onDragStart = (coluna: string, item: KanbanItem) => {
    dragRef.current = { coluna, item };
  };

  const onDrop = (colunaDestino: string) => {
    if (!dragRef.current) return;
    const { coluna: origem, item } = dragRef.current;
    if (origem === colunaDestino) { setDropTarget(null); return; }

    setColunas((prev) => ({
      ...prev,
      [origem]: prev[origem].filter((i) => i.id !== item.id),
      [colunaDestino]: [...prev[colunaDestino], item],
    }));
    dragRef.current = null;
    setDropTarget(null);
  };

  return (
    <div className="container-fluid py-4">
      <h4 className="mb-4"><i className="fas fa-columns me-2" />Kanban</h4>
      <div className="row g-3">
        {Object.entries(colunas).map(([nome, itens]) => (
          <div key={nome} className="col-md-4">
            <div
              className={`card h-100 ${dropTarget === nome ? 'border-primary' : ''}`}
              onDragOver={(e) => { e.preventDefault(); setDropTarget(nome); }}
              onDragLeave={() => setDropTarget(null)}
              onDrop={() => onDrop(nome)}
            >
              <div className="card-header fw-semibold">
                {nome}
                <span className="badge bg-secondary ms-2">{itens.length}</span>
              </div>
              <div className="card-body" style={{ minHeight: 200 }}>
                {itens.map((item) => (
                  <div
                    key={item.id}
                    className="card mb-2"
                    draggable
                    onDragStart={() => onDragStart(nome, item)}
                    style={{ cursor: 'grab' }}
                  >
                    <div className="card-body py-2 px-3">
                      <small>{item.titulo}</small>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

### Equivalência — Drag & Drop

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Lista reordenável | `ReorderableListView` (built-in) | `react-native-draggable-flatlist` | HTML5 Drag and Drop API |
| Gesto de arraste | `ReorderableDragStartListener` | Long press | `draggable` attribute |
| Feedback visual | `Material` com elevation | `ScaleDecorator` | Classes CSS condicionais |
| Kanban (entre colunas) | `Draggable` + `DragTarget` | Implementação manual | `onDragOver` + `onDrop` |
| Animação ao soltar | Automática | `react-native-reanimated` | CSS `transition` |

---

## 2. Animações e Transições

Animações de entrada/saída, transições entre telas, micro-interações e pull-to-refresh customizado.

### Flutter — AnimationController + Hero

**1. Animação de entrada (fade + slide)**

```dart
// lib/features/animacoes/presentation/animated_list_screen.dart
import 'package:flutter/material.dart';

class AnimatedListScreen extends StatefulWidget {
  const AnimatedListScreen({super.key});

  @override
  State<AnimatedListScreen> createState() => _AnimatedListScreenState();
}

class _AnimatedListScreenState extends State<AnimatedListScreen> {
  final _itens = List.generate(8, (i) => 'Item ${i + 1}');
  final _listKey = GlobalKey<AnimatedListState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Lista Animada')),
      body: AnimatedList(
        key: _listKey,
        initialItemCount: _itens.length,
        padding: const EdgeInsets.all(16),
        itemBuilder: (context, index, animation) {
          return SlideTransition(
            position: animation.drive(
              Tween(begin: const Offset(1, 0), end: Offset.zero)
                  .chain(CurveTween(curve: Curves.easeOutCubic)),
            ),
            child: FadeTransition(
              opacity: animation,
              child: Card(
                margin: const EdgeInsets.only(bottom: 8),
                child: ListTile(
                  title: Text(_itens[index]),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete),
                    onPressed: () => _removerItem(index),
                  ),
                ),
              ),
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _adicionarItem,
        child: const Icon(Icons.add),
      ),
    );
  }

  void _adicionarItem() {
    final index = _itens.length;
    _itens.add('Item ${index + 1}');
    _listKey.currentState?.insertItem(index,
        duration: const Duration(milliseconds: 400));
  }

  void _removerItem(int index) {
    final item = _itens.removeAt(index);
    _listKey.currentState?.removeItem(
      index,
      (context, animation) => FadeTransition(
        opacity: animation,
        child: SizeTransition(
          sizeFactor: animation,
          child: Card(
            color: Colors.red.shade50,
            margin: const EdgeInsets.only(bottom: 8),
            child: ListTile(title: Text(item)),
          ),
        ),
      ),
      duration: const Duration(milliseconds: 300),
    );
  }
}
```

**2. Hero transition (detalhe de imagem)**

```dart
// lib/features/animacoes/presentation/hero_grid_screen.dart
import 'package:flutter/material.dart';

class HeroGridScreen extends StatelessWidget {
  const HeroGridScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Hero Transition')),
      body: GridView.builder(
        padding: const EdgeInsets.all(16),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 3, mainAxisSpacing: 8, crossAxisSpacing: 8,
        ),
        itemCount: 9,
        itemBuilder: (_, i) {
          final tag = 'img-$i';
          final url = 'https://picsum.photos/seed/hero$i/400/400';
          return GestureDetector(
            onTap: () => Navigator.push(
              context,
              MaterialPageRoute(
                builder: (_) => _DetalheScreen(tag: tag, url: url),
              ),
            ),
            child: Hero(
              tag: tag,
              child: ClipRRect(
                borderRadius: BorderRadius.circular(8),
                child: Image.network(url, fit: BoxFit.cover),
              ),
            ),
          );
        },
      ),
    );
  }
}

class _DetalheScreen extends StatelessWidget {
  final String tag;
  final String url;
  const _DetalheScreen({required this.tag, required this.url});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(backgroundColor: Colors.transparent, foregroundColor: Colors.white),
      body: Center(
        child: Hero(
          tag: tag,
          child: Image.network(url),
        ),
      ),
    );
  }
}
```

**3. Botão com micro-interação (scale + ripple)**

```dart
// lib/features/animacoes/presentation/widgets/animated_button.dart
import 'package:flutter/material.dart';

class AnimatedButton extends StatefulWidget {
  final String label;
  final VoidCallback onPressed;
  const AnimatedButton({super.key, required this.label, required this.onPressed});

  @override
  State<AnimatedButton> createState() => _AnimatedButtonState();
}

class _AnimatedButtonState extends State<AnimatedButton>
    with SingleTickerProviderStateMixin {
  late AnimationController _ctrl;
  late Animation<double> _scale;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 100),
    );
    _scale = Tween(begin: 1.0, end: 0.95).animate(
      CurvedAnimation(parent: _ctrl, curve: Curves.easeInOut),
    );
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) => _ctrl.forward(),
      onTapUp: (_) {
        _ctrl.reverse();
        widget.onPressed();
      },
      onTapCancel: () => _ctrl.reverse(),
      child: ScaleTransition(
        scale: _scale,
        child: FilledButton(
          onPressed: null,
          child: Text(widget.label),
        ),
      ),
    );
  }
}
```

---

### React Native — Animated + react-native-reanimated

**1. Dependências**

```bash
npx expo install react-native-reanimated
```

**2. Lista com animação de entrada**

```tsx
// src/features/animacoes/AnimatedListScreen.tsx
import React, { useEffect, useRef, useState } from 'react';
import {
  View, Text, FlatList, TouchableOpacity, Animated, StyleSheet,
} from 'react-native';

interface Item {
  id: string;
  titulo: string;
}

function AnimatedItem({ item, index, onRemove }: {
  item: Item; index: number; onRemove: () => void;
}) {
  const translateX = useRef(new Animated.Value(300)).current;
  const opacity = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.parallel([
      Animated.timing(translateX, {
        toValue: 0, duration: 400, delay: index * 80,
        useNativeDriver: true,
      }),
      Animated.timing(opacity, {
        toValue: 1, duration: 400, delay: index * 80,
        useNativeDriver: true,
      }),
    ]).start();
  }, []);

  return (
    <Animated.View style={[styles.card, { transform: [{ translateX }], opacity }]}>
      <Text style={styles.cardTitulo}>{item.titulo}</Text>
      <TouchableOpacity onPress={onRemove}>
        <Text style={styles.btnRemover}>✕</Text>
      </TouchableOpacity>
    </Animated.View>
  );
}

export function AnimatedListScreen() {
  const [itens, setItens] = useState<Item[]>(
    Array.from({ length: 8 }, (_, i) => ({
      id: `item-${i}`,
      titulo: `Item ${i + 1}`,
    })),
  );

  return (
    <View style={styles.container}>
      <FlatList
        data={itens}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.lista}
        renderItem={({ item, index }) => (
          <AnimatedItem
            item={item}
            index={index}
            onRemove={() => setItens((prev) => prev.filter((i) => i.id !== item.id))}
          />
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f9f9f9' },
  lista: { padding: 16 },
  card: {
    flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center',
    backgroundColor: '#fff', borderRadius: 10, padding: 16, marginBottom: 8, elevation: 1,
  },
  cardTitulo: { fontSize: 15, fontWeight: '500' },
  btnRemover: { fontSize: 18, color: '#e53935', padding: 4 },
});
```

---

### React Web — CSS Transitions + Keyframes

**1. Estilos de animação — `src/styles/animations.css`**

```css
/* src/styles/animations.css */
@keyframes slideInRight {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; max-height: 80px; }
  to { opacity: 0; max-height: 0; padding: 0; margin: 0; overflow: hidden; }
}

.animate-slide-in {
  animation: slideInRight 0.4s ease-out both;
}

.animate-fade-out {
  animation: fadeOut 0.3s ease-in forwards;
}

.scale-hover {
  transition: transform 0.15s ease;
}
.scale-hover:active {
  transform: scale(0.96);
}
```

**2. Lista animada**

```tsx
// src/features/animacoes/AnimatedListPage.tsx
import { useState } from 'react';
import '../../styles/animations.css';

interface Item {
  id: string;
  titulo: string;
  removendo?: boolean;
}

export function AnimatedListPage() {
  const [itens, setItens] = useState<Item[]>(
    Array.from({ length: 8 }, (_, i) => ({
      id: `item-${i}`,
      titulo: `Item ${i + 1}`,
    })),
  );

  const remover = (id: string) => {
    setItens((prev) =>
      prev.map((item) => (item.id === id ? { ...item, removendo: true } : item)),
    );
    setTimeout(() => {
      setItens((prev) => prev.filter((item) => item.id !== id));
    }, 300);
  };

  const adicionar = () => {
    const id = `item-${Date.now()}`;
    setItens((prev) => [...prev, { id, titulo: `Item ${prev.length + 1}` }]);
  };

  return (
    <div className="container py-4" style={{ maxWidth: 600 }}>
      <div className="d-flex justify-content-between align-items-center mb-4">
        <h4 className="mb-0"><i className="fas fa-magic me-2" />Lista Animada</h4>
        <button className="btn btn-primary scale-hover" onClick={adicionar}>
          <i className="fas fa-plus me-1" /> Adicionar
        </button>
      </div>

      <ul className="list-group">
        {itens.map((item, i) => (
          <li
            key={item.id}
            className={`list-group-item d-flex justify-content-between align-items-center ${
              item.removendo ? 'animate-fade-out' : 'animate-slide-in'
            }`}
            style={{ animationDelay: item.removendo ? '0s' : `${i * 0.08}s` }}
          >
            <span>{item.titulo}</span>
            <button
              className="btn btn-sm btn-outline-danger"
              onClick={() => remover(item.id)}
            >
              <i className="fas fa-times" />
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

### Equivalência — Animações

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Animação de entrada | `AnimatedList` + `SlideTransition` | `Animated.timing` com delay | CSS `@keyframes` + `animation-delay` |
| Animação de saída | `removeItem` com builder | `Animated` reverse | CSS `animation` + `setTimeout` |
| Transição entre telas | `Hero` widget | `SharedElement` (React Navigation) | CSS `View Transition API` |
| Micro-interação (tap) | `AnimationController` + `ScaleTransition` | `Animated.spring` | CSS `transform: scale()` + `:active` |
| Pull-to-refresh | `RefreshIndicator` (built-in) | `RefreshControl` (built-in) | Implementação manual |
| Biblioteca avançada | `flutter_animate` | `react-native-reanimated` | CSS nativo / Framer Motion |

---

## 3. Busca com Autocomplete e Debounce

Campo de busca com sugestões em tempo real, debounce para evitar requisições excessivas, highlight do termo e histórico de buscas recentes.

### Flutter — SearchDelegate + Timer

**1. Serviço de busca simulado**

```dart
// lib/features/busca/data/search_service.dart
class SearchService {
  static const _dados = [
    'Arroz integral', 'Arroz branco', 'Feijão preto', 'Feijão carioca',
    'Macarrão espaguete', 'Macarrão penne', 'Azeite extra virgem',
    'Farinha de trigo', 'Açúcar cristal', 'Sal refinado',
    'Leite integral', 'Leite desnatado', 'Manteiga', 'Queijo mussarela',
    'Café torrado', 'Chá verde', 'Chocolate ao leite', 'Biscoito cream cracker',
  ];

  Future<List<String>> buscar(String termo) async {
    await Future.delayed(const Duration(milliseconds: 300));
    if (termo.isEmpty) return [];
    final lower = termo.toLowerCase();
    return _dados.where((d) => d.toLowerCase().contains(lower)).toList();
  }
}
```

**2. Tela de busca com autocomplete e debounce**

```dart
// lib/features/busca/presentation/search_screen.dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';
import '../data/search_service.dart';

class SearchScreen extends StatefulWidget {
  const SearchScreen({super.key});

  @override
  State<SearchScreen> createState() => _SearchScreenState();
}

class _SearchScreenState extends State<SearchScreen> {
  final _service = SearchService();
  final _ctrl = TextEditingController();

  List<String> _resultados = [];
  List<String> _historico = [];
  bool _carregando = false;
  Timer? _debounce;

  @override
  void initState() {
    super.initState();
    _carregarHistorico();
  }

  Future<void> _carregarHistorico() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      _historico = prefs.getStringList('search_history') ?? [];
    });
  }

  Future<void> _salvarHistorico(String termo) async {
    _historico.remove(termo);
    _historico.insert(0, termo);
    if (_historico.length > 10) _historico = _historico.sublist(0, 10);
    final prefs = await SharedPreferences.getInstance();
    await prefs.setStringList('search_history', _historico);
  }

  void _onChanged(String valor) {
    _debounce?.cancel();
    _debounce = Timer(const Duration(milliseconds: 400), () {
      _buscar(valor);
    });
  }

  Future<void> _buscar(String termo) async {
    if (termo.trim().isEmpty) {
      setState(() => _resultados = []);
      return;
    }

    setState(() => _carregando = true);
    final resultados = await _service.buscar(termo);
    setState(() {
      _resultados = resultados;
      _carregando = false;
    });
  }

  void _selecionarItem(String item) {
    _ctrl.text = item;
    _salvarHistorico(item);
    setState(() => _resultados = []);
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Selecionado: $item')),
    );
  }

  Widget _highlightTexto(String texto, String termo) {
    if (termo.isEmpty) return Text(texto);
    final lower = texto.toLowerCase();
    final termoLower = termo.toLowerCase();
    final inicio = lower.indexOf(termoLower);
    if (inicio < 0) return Text(texto);

    return RichText(
      text: TextSpan(
        style: DefaultTextStyle.of(context).style,
        children: [
          TextSpan(text: texto.substring(0, inicio)),
          TextSpan(
            text: texto.substring(inicio, inicio + termo.length),
            style: const TextStyle(fontWeight: FontWeight.bold, color: Colors.indigo),
          ),
          TextSpan(text: texto.substring(inicio + termo.length)),
        ],
      ),
    );
  }

  @override
  void dispose() {
    _debounce?.cancel();
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final mostrarHistorico = _ctrl.text.isEmpty && _historico.isNotEmpty;

    return Scaffold(
      appBar: AppBar(title: const Text('Busca')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              controller: _ctrl,
              onChanged: _onChanged,
              decoration: InputDecoration(
                hintText: 'Buscar produto…',
                prefixIcon: const Icon(Icons.search),
                suffixIcon: _ctrl.text.isNotEmpty
                    ? IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _ctrl.clear();
                          setState(() => _resultados = []);
                        },
                      )
                    : null,
                border: const OutlineInputBorder(),
              ),
            ),
          ),

          if (_carregando)
            const LinearProgressIndicator(),

          if (mostrarHistorico) ...[
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text('Recentes', style: Theme.of(context).textTheme.titleSmall),
                  TextButton(
                    onPressed: () async {
                      final prefs = await SharedPreferences.getInstance();
                      await prefs.remove('search_history');
                      setState(() => _historico = []);
                    },
                    child: const Text('Limpar'),
                  ),
                ],
              ),
            ),
            Expanded(
              child: ListView(
                children: _historico.map((h) => ListTile(
                  leading: const Icon(Icons.history, color: Colors.grey),
                  title: Text(h),
                  onTap: () {
                    _ctrl.text = h;
                    _buscar(h);
                  },
                )).toList(),
              ),
            ),
          ],

          if (!mostrarHistorico)
            Expanded(
              child: _resultados.isEmpty && !_carregando && _ctrl.text.isNotEmpty
                  ? const Center(child: Text('Nenhum resultado encontrado.'))
                  : ListView.builder(
                      itemCount: _resultados.length,
                      itemBuilder: (_, i) => ListTile(
                        title: _highlightTexto(_resultados[i], _ctrl.text),
                        onTap: () => _selecionarItem(_resultados[i]),
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

### React Native — TextInput + debounce

```tsx
// src/features/busca/SearchScreen.tsx
import React, { useState, useCallback, useRef, useEffect } from 'react';
import {
  View, Text, TextInput, FlatList, TouchableOpacity, StyleSheet,
  ActivityIndicator,
} from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const DADOS = [
  'Arroz integral', 'Arroz branco', 'Feijão preto', 'Feijão carioca',
  'Macarrão espaguete', 'Macarrão penne', 'Azeite extra virgem',
  'Farinha de trigo', 'Açúcar cristal', 'Sal refinado',
  'Leite integral', 'Leite desnatado', 'Manteiga', 'Queijo mussarela',
  'Café torrado', 'Chá verde', 'Chocolate ao leite', 'Biscoito cream cracker',
];

async function buscar(termo: string): Promise<string[]> {
  await new Promise((r) => setTimeout(r, 300));
  if (!termo) return [];
  const lower = termo.toLowerCase();
  return DADOS.filter((d) => d.toLowerCase().includes(lower));
}

export function SearchScreen() {
  const [texto, setTexto] = useState('');
  const [resultados, setResultados] = useState<string[]>([]);
  const [historico, setHistorico] = useState<string[]>([]);
  const [carregando, setCarregando] = useState(false);
  const timerRef = useRef<ReturnType<typeof setTimeout>>();

  useEffect(() => {
    AsyncStorage.getItem('search_history').then((data) => {
      if (data) setHistorico(JSON.parse(data));
    });
  }, []);

  const salvarHistorico = async (termo: string) => {
    const novo = [termo, ...historico.filter((h) => h !== termo)].slice(0, 10);
    setHistorico(novo);
    await AsyncStorage.setItem('search_history', JSON.stringify(novo));
  };

  const onChange = useCallback((valor: string) => {
    setTexto(valor);
    clearTimeout(timerRef.current);
    timerRef.current = setTimeout(async () => {
      setCarregando(true);
      const res = await buscar(valor);
      setResultados(res);
      setCarregando(false);
    }, 400);
  }, []);

  const selecionar = (item: string) => {
    setTexto(item);
    setResultados([]);
    salvarHistorico(item);
  };

  const mostrarHistorico = texto === '' && historico.length > 0;

  return (
    <View style={styles.container}>
      <View style={styles.inputRow}>
        <TextInput
          style={styles.input}
          placeholder="Buscar produto…"
          value={texto}
          onChangeText={onChange}
        />
        {texto !== '' && (
          <TouchableOpacity onPress={() => { setTexto(''); setResultados([]); }}>
            <Text style={styles.limpar}>✕</Text>
          </TouchableOpacity>
        )}
      </View>

      {carregando && <ActivityIndicator style={{ marginTop: 8 }} />}

      {mostrarHistorico ? (
        <View style={styles.secao}>
          <Text style={styles.secaoTitulo}>Recentes</Text>
          {historico.map((h) => (
            <TouchableOpacity key={h} style={styles.item} onPress={() => { setTexto(h); onChange(h); }}>
              <Text style={styles.historicoIcon}>🕐</Text>
              <Text>{h}</Text>
            </TouchableOpacity>
          ))}
        </View>
      ) : (
        <FlatList
          data={resultados}
          keyExtractor={(item) => item}
          renderItem={({ item }) => (
            <TouchableOpacity style={styles.item} onPress={() => selecionar(item)}>
              <Text>{item}</Text>
            </TouchableOpacity>
          )}
        />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff', padding: 16 },
  inputRow: { flexDirection: 'row', alignItems: 'center', borderWidth: 1, borderColor: '#ddd', borderRadius: 8, paddingHorizontal: 12 },
  input: { flex: 1, paddingVertical: 10, fontSize: 15 },
  limpar: { fontSize: 18, color: '#999', padding: 4 },
  secao: { marginTop: 16 },
  secaoTitulo: { fontWeight: '600', marginBottom: 8 },
  item: { flexDirection: 'row', alignItems: 'center', gap: 8, paddingVertical: 12, borderBottomWidth: 1, borderBottomColor: '#f0f0f0' },
  historicoIcon: { fontSize: 14 },
});
```

---

### React Web — Bootstrap + debounce

```tsx
// src/features/busca/SearchPage.tsx
import { useState, useRef, useEffect, useCallback } from 'react';

const DADOS = [
  'Arroz integral', 'Arroz branco', 'Feijão preto', 'Feijão carioca',
  'Macarrão espaguete', 'Macarrão penne', 'Azeite extra virgem',
  'Farinha de trigo', 'Açúcar cristal', 'Sal refinado',
  'Leite integral', 'Leite desnatado', 'Manteiga', 'Queijo mussarela',
  'Café torrado', 'Chá verde', 'Chocolate ao leite', 'Biscoito cream cracker',
];

async function buscar(termo: string): Promise<string[]> {
  await new Promise((r) => setTimeout(r, 300));
  if (!termo) return [];
  const lower = termo.toLowerCase();
  return DADOS.filter((d) => d.toLowerCase().includes(lower));
}

function highlight(texto: string, termo: string) {
  if (!termo) return texto;
  const idx = texto.toLowerCase().indexOf(termo.toLowerCase());
  if (idx < 0) return texto;
  return (
    <>
      {texto.slice(0, idx)}
      <mark className="bg-primary-subtle px-0">{texto.slice(idx, idx + termo.length)}</mark>
      {texto.slice(idx + termo.length)}
    </>
  );
}

const HIST_KEY = 'search_history';

export function SearchPage() {
  const [texto, setTexto] = useState('');
  const [resultados, setResultados] = useState<string[]>([]);
  const [historico, setHistorico] = useState<string[]>(() => {
    const salvo = localStorage.getItem(HIST_KEY);
    return salvo ? JSON.parse(salvo) : [];
  });
  const [carregando, setCarregando] = useState(false);
  const timerRef = useRef<ReturnType<typeof setTimeout>>();

  const salvar = (termo: string) => {
    const novo = [termo, ...historico.filter((h) => h !== termo)].slice(0, 10);
    setHistorico(novo);
    localStorage.setItem(HIST_KEY, JSON.stringify(novo));
  };

  const onChange = useCallback((valor: string) => {
    setTexto(valor);
    clearTimeout(timerRef.current);
    timerRef.current = setTimeout(async () => {
      setCarregando(true);
      const res = await buscar(valor);
      setResultados(res);
      setCarregando(false);
    }, 400);
  }, []);

  const mostrarHistorico = texto === '' && historico.length > 0;

  return (
    <div className="container py-4" style={{ maxWidth: 600 }}>
      <div className="input-group mb-3">
        <span className="input-group-text"><i className="fas fa-search" /></span>
        <input
          type="text"
          className="form-control"
          placeholder="Buscar produto…"
          value={texto}
          onChange={(e) => onChange(e.target.value)}
        />
        {texto && (
          <button className="btn btn-outline-secondary" onClick={() => { setTexto(''); setResultados([]); }}>
            <i className="fas fa-times" />
          </button>
        )}
      </div>

      {carregando && (
        <div className="progress mb-3" style={{ height: 3 }}>
          <div className="progress-bar progress-bar-striped progress-bar-animated w-100" />
        </div>
      )}

      {mostrarHistorico && (
        <>
          <div className="d-flex justify-content-between mb-2">
            <small className="fw-semibold">Recentes</small>
            <button className="btn btn-sm btn-link p-0" onClick={() => {
              setHistorico([]);
              localStorage.removeItem(HIST_KEY);
            }}>Limpar</button>
          </div>
          <ul className="list-group">
            {historico.map((h) => (
              <li
                key={h}
                className="list-group-item list-group-item-action d-flex align-items-center gap-2"
                role="button"
                onClick={() => onChange(h) || setTexto(h)}
              >
                <i className="fas fa-clock text-muted" />
                {h}
              </li>
            ))}
          </ul>
        </>
      )}

      {!mostrarHistorico && resultados.length > 0 && (
        <ul className="list-group">
          {resultados.map((item) => (
            <li
              key={item}
              className="list-group-item list-group-item-action"
              role="button"
              onClick={() => { setTexto(item); setResultados([]); salvar(item); }}
            >
              {highlight(item, texto)}
            </li>
          ))}
        </ul>
      )}

      {!mostrarHistorico && texto && !carregando && resultados.length === 0 && (
        <p className="text-muted text-center mt-4">Nenhum resultado encontrado.</p>
      )}
    </div>
  );
}
```

---

### Equivalência — Busca com Autocomplete

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Debounce | `Timer` (dart:async) | `setTimeout` | `setTimeout` |
| Highlight do termo | `RichText` + `TextSpan` | `Text` com trechos | `<mark>` HTML |
| Histórico | `shared_preferences` | `AsyncStorage` | `localStorage` |
| Loading | `LinearProgressIndicator` | `ActivityIndicator` | Bootstrap `progress-bar` |
| Limpar campo | `IconButton` no suffixIcon | Botão `✕` | Botão no `input-group` |

---

## 4. Compartilhamento e Deep Links

Compartilhar conteúdo via share sheet nativo e receber deep links que abrem telas específicas do app.

### Flutter — share_plus + GoRouter deep links

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  share_plus: ^10.1.0
  go_router: ^14.0.0
```

**2. Compartilhamento**

```dart
// lib/core/sharing/share_service.dart
import 'package:share_plus/share_plus.dart';

class ShareService {
  Future<void> compartilharTexto({
    required String texto,
    String? assunto,
  }) async {
    await Share.share(texto, subject: assunto);
  }

  Future<void> compartilharLink({
    required String url,
    required String titulo,
  }) async {
    await Share.share(
      '$titulo\n$url',
      subject: titulo,
    );
  }

  Future<void> compartilharArquivo({
    required String caminhoArquivo,
    String? texto,
  }) async {
    await Share.shareXFiles(
      [XFile(caminhoArquivo)],
      text: texto,
    );
  }
}
```

**3. Deep links — configuração GoRouter**

```dart
// lib/routes/app_router.dart
import 'package:go_router/go_router.dart';
import 'package:flutter/material.dart';

final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (_, __) => const HomeScreen(),
    ),
    GoRoute(
      path: '/produto/:id',
      builder: (_, state) {
        final id = state.pathParameters['id']!;
        return ProdutoScreen(produtoId: id);
      },
    ),
    GoRoute(
      path: '/promo',
      builder: (_, state) {
        final codigo = state.uri.queryParameters['codigo'];
        return PromoScreen(codigo: codigo);
      },
    ),
  ],
);
```

**4. Configuração Android — `android/app/src/main/AndroidManifest.xml`**

```xml
<activity>
  <!-- Deep links: meuapp://produto/123 -->
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="meuapp" />
  </intent-filter>

  <!-- Universal links: https://meuapp.com/produto/123 -->
  <intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="meuapp.com" />
  </intent-filter>
</activity>
```

**5. Configuração iOS — `ios/Runner/Info.plist`**

```xml
<dict>
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
      <key>CFBundleURLSchemes</key>
      <array>
        <string>meuapp</string>
      </array>
    </dict>
  </array>
  <!-- Universal Links requerem Associated Domains no Xcode -->
</dict>
```

**6. Botão de compartilhamento com deep link**

```dart
// Em qualquer tela:
IconButton(
  icon: const Icon(Icons.share),
  onPressed: () {
    ShareService().compartilharLink(
      titulo: 'Confira este produto!',
      url: 'https://meuapp.com/produto/$produtoId',
    );
  },
)
```

---

### React Native — expo-sharing + expo-linking

**1. Dependências**

```bash
npx expo install expo-sharing expo-linking
```

**2. Compartilhamento e deep links**

```typescript
// src/core/sharing/shareService.ts
import * as Sharing from 'expo-sharing';
import { Share, Platform } from 'react-native';

export async function compartilharTexto(texto: string, titulo?: string) {
  await Share.share(
    Platform.OS === 'ios'
      ? { message: texto }
      : { message: texto, title: titulo },
  );
}

export async function compartilharLink(url: string, titulo: string) {
  await Share.share({ message: `${titulo}\n${url}` });
}
```

```typescript
// src/core/linking/deepLinkService.ts
import * as Linking from 'expo-linking';

export function criarDeepLink(path: string, params?: Record<string, string>): string {
  return Linking.createURL(path, { queryParams: params });
}

// No app.json: { "expo": { "scheme": "meuapp" } }
// Deep link resultante: meuapp://produto/123
```

**3. Hook para receber deep links**

```typescript
// src/hooks/useDeepLink.ts
import { useEffect } from 'react';
import * as Linking from 'expo-linking';

export function useDeepLink(onLink: (url: string) => void) {
  useEffect(() => {
    // Link que abriu o app
    Linking.getInitialURL().then((url) => {
      if (url) onLink(url);
    });

    // Link recebido com app aberto
    const sub = Linking.addEventListener('url', (event) => {
      onLink(event.url);
    });

    return () => sub.remove();
  }, [onLink]);
}
```

---

### React Web — Web Share API + React Router

```tsx
// src/core/sharing/shareService.ts
export async function compartilharLink(url: string, titulo: string, texto?: string) {
  if (navigator.share) {
    await navigator.share({ title: titulo, text: texto, url });
  } else {
    await navigator.clipboard.writeText(url);
    alert('Link copiado para a área de transferência!');
  }
}
```

```tsx
// Botão de compartilhamento em qualquer componente:
<button
  className="btn btn-outline-primary btn-sm"
  onClick={() => compartilharLink(
    `${window.location.origin}/produto/${id}`,
    'Confira este produto!',
  )}
>
  <i className="fas fa-share-alt me-1" />Compartilhar
</button>

// Deep links: React Router já trata naturalmente — qualquer URL como /produto/123
// é resolvida pelo Routes. Basta configurar o servidor para redirecionar 404 → index.html.
```

---

### Equivalência — Compartilhamento e Deep Links

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Share sheet | `share_plus` (`Share.share`) | `Share` (RN) + `expo-sharing` | `navigator.share()` |
| Compartilhar arquivo | `Share.shareXFiles` | `Sharing.shareAsync` | N/A (download link) |
| Deep link scheme | `meuapp://` (AndroidManifest + Info.plist) | `meuapp://` (`app.json` scheme) | URL path (React Router) |
| Universal links | `https://meuapp.com/...` (autoVerify) | `expo-linking` + Associated Domains | Nativo (URL = deep link) |
| Receber link (app aberto) | GoRouter trata automaticamente | `Linking.addEventListener('url')` | React Router trata automaticamente |
| Receber link (app fechado) | GoRouter `initialLocation` | `Linking.getInitialURL()` | URL path no carregamento |
| Fallback (sem share API) | — | — | `navigator.clipboard.writeText` |

---

## 5. Reprodução de Áudio / Podcast Player

Player de áudio com controles, reprodução em background, velocidade, mini-player persistente e fila de reprodução.

### Flutter — just_audio

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  just_audio: ^0.9.40
  audio_session: ^0.1.21
```

**2. Modelo e serviço**

```dart
// lib/features/audio/domain/audio_track.dart
class AudioTrack {
  final String id;
  final String titulo;
  final String artista;
  final String url;
  final Duration? duracao;
  final String? capaUrl;

  const AudioTrack({
    required this.id,
    required this.titulo,
    required this.artista,
    required this.url,
    this.duracao,
    this.capaUrl,
  });
}
```

**3. Player screen**

```dart
// lib/features/audio/presentation/audio_player_screen.dart
import 'package:flutter/material.dart';
import 'package:just_audio/just_audio.dart';
import '../domain/audio_track.dart';

class AudioPlayerScreen extends StatefulWidget {
  final AudioTrack track;
  const AudioPlayerScreen({super.key, required this.track});

  @override
  State<AudioPlayerScreen> createState() => _AudioPlayerScreenState();
}

class _AudioPlayerScreenState extends State<AudioPlayerScreen> {
  late AudioPlayer _player;
  double _velocidade = 1.0;

  @override
  void initState() {
    super.initState();
    _player = AudioPlayer();
    _player.setUrl(widget.track.url);
  }

  @override
  void dispose() {
    _player.dispose();
    super.dispose();
  }

  String _formatDuration(Duration d) {
    final m = d.inMinutes.remainder(60).toString().padLeft(2, '0');
    final s = d.inSeconds.remainder(60).toString().padLeft(2, '0');
    return '${d.inHours > 0 ? "${d.inHours}:" : ""}$m:$s';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(widget.track.titulo)),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          children: [
            // Capa
            Container(
              width: 240, height: 240,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(16),
                color: Colors.grey.shade200,
                image: widget.track.capaUrl != null
                    ? DecorationImage(
                        image: NetworkImage(widget.track.capaUrl!),
                        fit: BoxFit.cover,
                      )
                    : null,
              ),
              child: widget.track.capaUrl == null
                  ? const Icon(Icons.music_note, size: 64, color: Colors.grey)
                  : null,
            ),
            const SizedBox(height: 24),
            Text(widget.track.titulo,
                style: Theme.of(context).textTheme.titleLarge),
            Text(widget.track.artista,
                style: Theme.of(context).textTheme.bodyMedium?.copyWith(color: Colors.grey)),
            const SizedBox(height: 32),

            // Barra de progresso
            StreamBuilder<Duration?>(
              stream: _player.positionStream,
              builder: (_, snapshot) {
                final posicao = snapshot.data ?? Duration.zero;
                final total = _player.duration ?? Duration.zero;
                return Column(
                  children: [
                    Slider(
                      value: posicao.inMilliseconds.toDouble(),
                      max: total.inMilliseconds.toDouble().clamp(1, double.infinity),
                      onChanged: (v) => _player.seek(Duration(milliseconds: v.toInt())),
                    ),
                    Padding(
                      padding: const EdgeInsets.symmetric(horizontal: 16),
                      child: Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(_formatDuration(posicao), style: const TextStyle(fontSize: 12)),
                          Text(_formatDuration(total), style: const TextStyle(fontSize: 12)),
                        ],
                      ),
                    ),
                  ],
                );
              },
            ),
            const SizedBox(height: 16),

            // Controles
            StreamBuilder<PlayerState>(
              stream: _player.playerStateStream,
              builder: (_, snapshot) {
                final state = snapshot.data;
                final tocando = state?.playing ?? false;

                return Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    IconButton(
                      icon: const Icon(Icons.replay_10),
                      iconSize: 36,
                      onPressed: () => _player.seek(
                        (_player.position - const Duration(seconds: 10)),
                      ),
                    ),
                    const SizedBox(width: 16),
                    FloatingActionButton(
                      onPressed: () => tocando ? _player.pause() : _player.play(),
                      child: Icon(tocando ? Icons.pause : Icons.play_arrow, size: 32),
                    ),
                    const SizedBox(width: 16),
                    IconButton(
                      icon: const Icon(Icons.forward_30),
                      iconSize: 36,
                      onPressed: () => _player.seek(
                        (_player.position + const Duration(seconds: 30)),
                      ),
                    ),
                  ],
                );
              },
            ),
            const SizedBox(height: 24),

            // Velocidade
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [0.5, 1.0, 1.25, 1.5, 2.0].map((v) {
                return Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 4),
                  child: ChoiceChip(
                    label: Text('${v}x'),
                    selected: _velocidade == v,
                    onSelected: (_) {
                      _player.setSpeed(v);
                      setState(() => _velocidade = v);
                    },
                  ),
                );
              }).toList(),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### React Native — expo-av

**1. Dependências**

```bash
npx expo install expo-av
```

**2. Player**

```tsx
// src/features/audio/AudioPlayerScreen.tsx
import React, { useEffect, useState, useRef } from 'react';
import { View, Text, TouchableOpacity, Image, StyleSheet } from 'react-native';
import { Audio, AVPlaybackStatus } from 'expo-av';
import Slider from '@react-native-community/slider';

interface Track {
  titulo: string;
  artista: string;
  url: string;
  capaUrl?: string;
}

export function AudioPlayerScreen({ track }: { track: Track }) {
  const soundRef = useRef<Audio.Sound | null>(null);
  const [tocando, setTocando] = useState(false);
  const [posicao, setPosicao] = useState(0);
  const [duracao, setDuracao] = useState(0);
  const [velocidade, setVelocidade] = useState(1);

  useEffect(() => {
    Audio.setAudioModeAsync({
      playsInSilentModeIOS: true,
      staysActiveInBackground: true,
    });

    const carregar = async () => {
      const { sound } = await Audio.Sound.createAsync(
        { uri: track.url },
        { shouldPlay: false },
        (status: AVPlaybackStatus) => {
          if (!status.isLoaded) return;
          setPosicao(status.positionMillis);
          setDuracao(status.durationMillis ?? 0);
          setTocando(status.isPlaying);
        },
      );
      soundRef.current = sound;
    };
    carregar();

    return () => { soundRef.current?.unloadAsync(); };
  }, [track.url]);

  const togglePlay = () => {
    tocando ? soundRef.current?.pauseAsync() : soundRef.current?.playAsync();
  };

  const seek = (ms: number) => soundRef.current?.setPositionAsync(ms);
  const setSpeed = (rate: number) => {
    soundRef.current?.setRateAsync(rate, true);
    setVelocidade(rate);
  };

  const fmt = (ms: number) => {
    const s = Math.floor(ms / 1000);
    const m = String(Math.floor(s / 60)).padStart(2, '0');
    const sec = String(s % 60).padStart(2, '0');
    return `${m}:${sec}`;
  };

  return (
    <View style={styles.container}>
      {track.capaUrl ? (
        <Image source={{ uri: track.capaUrl }} style={styles.capa} />
      ) : (
        <View style={[styles.capa, styles.capaPlaceholder]}>
          <Text style={{ fontSize: 48 }}>🎵</Text>
        </View>
      )}
      <Text style={styles.titulo}>{track.titulo}</Text>
      <Text style={styles.artista}>{track.artista}</Text>

      <Slider
        style={styles.slider}
        value={posicao}
        minimumValue={0}
        maximumValue={duracao || 1}
        onSlidingComplete={(v) => seek(v)}
        minimumTrackTintColor="#3949ab"
      />
      <View style={styles.tempoRow}>
        <Text style={styles.tempo}>{fmt(posicao)}</Text>
        <Text style={styles.tempo}>{fmt(duracao)}</Text>
      </View>

      <View style={styles.controles}>
        <TouchableOpacity onPress={() => seek(Math.max(0, posicao - 10000))}>
          <Text style={styles.ctrlIcon}>⏪ 10s</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.playBtn} onPress={togglePlay}>
          <Text style={styles.playIcon}>{tocando ? '⏸' : '▶️'}</Text>
        </TouchableOpacity>
        <TouchableOpacity onPress={() => seek(posicao + 30000)}>
          <Text style={styles.ctrlIcon}>30s ⏩</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.speedRow}>
        {[0.5, 1, 1.25, 1.5, 2].map((v) => (
          <TouchableOpacity
            key={v}
            style={[styles.speedBtn, velocidade === v && styles.speedBtnAtivo]}
            onPress={() => setSpeed(v)}
          >
            <Text style={[styles.speedTxt, velocidade === v && styles.speedTxtAtivo]}>
              {v}x
            </Text>
          </TouchableOpacity>
        ))}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', padding: 24, backgroundColor: '#fff' },
  capa: { width: 200, height: 200, borderRadius: 16, marginBottom: 24 },
  capaPlaceholder: { backgroundColor: '#e0e0e0', justifyContent: 'center', alignItems: 'center' },
  titulo: { fontSize: 20, fontWeight: '700' },
  artista: { fontSize: 14, color: '#999', marginBottom: 24 },
  slider: { width: '100%', marginTop: 16 },
  tempoRow: { flexDirection: 'row', justifyContent: 'space-between', width: '100%', paddingHorizontal: 8 },
  tempo: { fontSize: 12, color: '#999' },
  controles: { flexDirection: 'row', alignItems: 'center', gap: 24, marginTop: 24 },
  ctrlIcon: { fontSize: 14, color: '#666' },
  playBtn: { width: 64, height: 64, borderRadius: 32, backgroundColor: '#3949ab', justifyContent: 'center', alignItems: 'center' },
  playIcon: { fontSize: 28 },
  speedRow: { flexDirection: 'row', gap: 8, marginTop: 32 },
  speedBtn: { paddingHorizontal: 12, paddingVertical: 6, borderRadius: 16, borderWidth: 1, borderColor: '#ccc' },
  speedBtnAtivo: { backgroundColor: '#3949ab', borderColor: '#3949ab' },
  speedTxt: { fontSize: 13, color: '#666' },
  speedTxtAtivo: { color: '#fff' },
});
```

---

### React Web — HTML5 Audio API + Bootstrap

```tsx
// src/features/audio/AudioPlayerPage.tsx
import { useRef, useState, useEffect } from 'react';

interface Track {
  titulo: string;
  artista: string;
  url: string;
  capaUrl?: string;
}

export function AudioPlayerPage({ track }: { track: Track }) {
  const audioRef = useRef<HTMLAudioElement>(null);
  const [tocando, setTocando] = useState(false);
  const [posicao, setPosicao] = useState(0);
  const [duracao, setDuracao] = useState(0);
  const [velocidade, setVelocidade] = useState(1);

  useEffect(() => {
    const audio = audioRef.current;
    if (!audio) return;

    const onTime = () => setPosicao(audio.currentTime);
    const onMeta = () => setDuracao(audio.duration);
    const onPlay = () => setTocando(true);
    const onPause = () => setTocando(false);

    audio.addEventListener('timeupdate', onTime);
    audio.addEventListener('loadedmetadata', onMeta);
    audio.addEventListener('play', onPlay);
    audio.addEventListener('pause', onPause);

    return () => {
      audio.removeEventListener('timeupdate', onTime);
      audio.removeEventListener('loadedmetadata', onMeta);
      audio.removeEventListener('play', onPlay);
      audio.removeEventListener('pause', onPause);
    };
  }, []);

  const toggle = () => tocando ? audioRef.current?.pause() : audioRef.current?.play();
  const seek = (s: number) => { if (audioRef.current) audioRef.current.currentTime = s; };
  const setSpeed = (r: number) => {
    if (audioRef.current) audioRef.current.playbackRate = r;
    setVelocidade(r);
  };

  const fmt = (s: number) => {
    const m = String(Math.floor(s / 60)).padStart(2, '0');
    const sec = String(Math.floor(s % 60)).padStart(2, '0');
    return `${m}:${sec}`;
  };

  return (
    <div className="container py-4 text-center" style={{ maxWidth: 480 }}>
      <audio ref={audioRef} src={track.url} preload="metadata" />

      {track.capaUrl ? (
        <img src={track.capaUrl} alt="Capa" className="rounded-4 mb-4" style={{ width: 200, height: 200, objectFit: 'cover' }} />
      ) : (
        <div className="d-inline-flex align-items-center justify-content-center rounded-4 bg-body-secondary mb-4" style={{ width: 200, height: 200 }}>
          <i className="fas fa-music fa-3x text-muted" />
        </div>
      )}

      <h5>{track.titulo}</h5>
      <p className="text-muted">{track.artista}</p>

      <input
        type="range" className="form-range mb-1"
        min={0} max={duracao || 1} step={0.1} value={posicao}
        onChange={(e) => seek(Number(e.target.value))}
      />
      <div className="d-flex justify-content-between px-1 mb-4">
        <small className="text-muted">{fmt(posicao)}</small>
        <small className="text-muted">{fmt(duracao)}</small>
      </div>

      <div className="d-flex justify-content-center align-items-center gap-4 mb-4">
        <button className="btn btn-link text-dark" onClick={() => seek(posicao - 10)}>
          <i className="fas fa-undo" /> 10s
        </button>
        <button className="btn btn-primary rounded-circle d-flex align-items-center justify-content-center" style={{ width: 56, height: 56 }} onClick={toggle}>
          <i className={`fas fa-${tocando ? 'pause' : 'play'} fa-lg`} />
        </button>
        <button className="btn btn-link text-dark" onClick={() => seek(posicao + 30)}>
          30s <i className="fas fa-redo" />
        </button>
      </div>

      <div className="btn-group" role="group">
        {[0.5, 1, 1.25, 1.5, 2].map((v) => (
          <button key={v} className={`btn btn-sm ${velocidade === v ? 'btn-primary' : 'btn-outline-secondary'}`} onClick={() => setSpeed(v)}>
            {v}x
          </button>
        ))}
      </div>
    </div>
  );
}
```

---

### Equivalência — Áudio / Podcast

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Biblioteca | `just_audio` | `expo-av` | HTML5 `<audio>` API |
| Background play | `audio_session` config | `staysActiveInBackground: true` | Automático (aba ativa) |
| Velocidade | `setSpeed()` | `setRateAsync()` | `playbackRate` |
| Seek | `seek(Duration)` | `setPositionAsync(ms)` | `currentTime = s` |
| Stream de posição | `positionStream` | Callback no `createAsync` | `timeupdate` event |
| Slider | `Slider` (Material) | `@react-native-community/slider` | `<input type="range">` Bootstrap |

---

## 6. Acesso a Contatos e Calendário

Ler contatos do dispositivo e criar eventos no calendário nativo.

### Flutter — flutter_contacts + add_2_calendar

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_contacts: ^1.1.9
  add_2_calendar: ^3.0.1
```

**2. Configuração**

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.WRITE_CALENDAR" />
<uses-permission android:name="android.permission.READ_CALENDAR" />
```

```xml
<!-- ios/Runner/Info.plist -->
<key>NSContactsUsageDescription</key>
<string>Precisamos acessar seus contatos.</string>
<key>NSCalendarsUsageDescription</key>
<string>Precisamos acessar seu calendário para criar eventos.</string>
```

**3. Tela de contatos e calendário**

```dart
// lib/features/contatos/presentation/contacts_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_contacts/flutter_contacts.dart';
import 'package:add_2_calendar/add_2_calendar.dart';

class ContactsScreen extends StatefulWidget {
  const ContactsScreen({super.key});

  @override
  State<ContactsScreen> createState() => _ContactsScreenState();
}

class _ContactsScreenState extends State<ContactsScreen> {
  List<Contact> _contatos = [];
  bool _carregando = false;

  Future<void> _carregarContatos() async {
    if (!await FlutterContacts.requestPermission()) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Permissão negada.')),
        );
      }
      return;
    }

    setState(() => _carregando = true);
    final contatos = await FlutterContacts.getContacts(
      withProperties: true,
      withPhoto: true,
    );
    setState(() {
      _contatos = contatos;
      _carregando = false;
    });
  }

  void _criarEvento() {
    Add2Calendar.addEvent2Cal(Event(
      title: 'Reunião de Projeto',
      description: 'Discussão sobre o novo módulo do app.',
      location: 'Sala 3 — Escritório',
      startDate: DateTime.now().add(const Duration(days: 1, hours: 2)),
      endDate: DateTime.now().add(const Duration(days: 1, hours: 3)),
    ));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Contatos & Calendário')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: FilledButton.icon(
                    onPressed: _carregarContatos,
                    icon: const Icon(Icons.contacts),
                    label: const Text('Carregar Contatos'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: _criarEvento,
                    icon: const Icon(Icons.event),
                    label: const Text('Criar Evento'),
                  ),
                ),
              ],
            ),
          ),
          if (_carregando) const LinearProgressIndicator(),
          Expanded(
            child: ListView.builder(
              itemCount: _contatos.length,
              itemBuilder: (_, i) {
                final c = _contatos[i];
                return ListTile(
                  leading: c.photo != null
                      ? CircleAvatar(backgroundImage: MemoryImage(c.photo!))
                      : CircleAvatar(child: Text(c.displayName.isNotEmpty ? c.displayName[0] : '?')),
                  title: Text(c.displayName),
                  subtitle: Text(
                    c.phones.isNotEmpty ? c.phones.first.number : 'Sem telefone',
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### React Native — expo-contacts + expo-calendar

**1. Dependências**

```bash
npx expo install expo-contacts expo-calendar
```

**2. Tela**

```tsx
// src/features/contatos/ContactsScreen.tsx
import React, { useState } from 'react';
import { View, Text, FlatList, TouchableOpacity, Image, StyleSheet, Alert } from 'react-native';
import * as Contacts from 'expo-contacts';
import * as Calendar from 'expo-calendar';

export function ContactsScreen() {
  const [contatos, setContatos] = useState<Contacts.Contact[]>([]);
  const [carregando, setCarregando] = useState(false);

  const carregarContatos = async () => {
    const { status } = await Contacts.requestPermissionsAsync();
    if (status !== 'granted') { Alert.alert('Permissão negada'); return; }

    setCarregando(true);
    const { data } = await Contacts.getContactsAsync({
      fields: [Contacts.Fields.PhoneNumbers, Contacts.Fields.Image],
    });
    setContatos(data);
    setCarregando(false);
  };

  const criarEvento = async () => {
    const { status } = await Calendar.requestCalendarPermissionsAsync();
    if (status !== 'granted') { Alert.alert('Permissão negada'); return; }

    const calendars = await Calendar.getCalendarsAsync(Calendar.EntityTypes.EVENT);
    const defaultCal = calendars.find((c) => c.isPrimary) ?? calendars[0];
    if (!defaultCal) { Alert.alert('Nenhum calendário encontrado'); return; }

    const inicio = new Date();
    inicio.setDate(inicio.getDate() + 1);
    inicio.setHours(14, 0, 0, 0);
    const fim = new Date(inicio);
    fim.setHours(15);

    await Calendar.createEventAsync(defaultCal.id, {
      title: 'Reunião de Projeto',
      notes: 'Discussão sobre o novo módulo do app.',
      location: 'Sala 3 — Escritório',
      startDate: inicio,
      endDate: fim,
    });
    Alert.alert('Sucesso', 'Evento criado no calendário!');
  };

  return (
    <View style={styles.container}>
      <View style={styles.botoes}>
        <TouchableOpacity style={styles.btn} onPress={carregarContatos}>
          <Text style={styles.btnTxt}>📇 Contatos</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.btnOutline} onPress={criarEvento}>
          <Text style={styles.btnOutlineTxt}>📅 Criar Evento</Text>
        </TouchableOpacity>
      </View>
      <FlatList
        data={contatos}
        keyExtractor={(c) => c.id!}
        renderItem={({ item }) => (
          <View style={styles.item}>
            {item.image?.uri ? (
              <Image source={{ uri: item.image.uri }} style={styles.avatar} />
            ) : (
              <View style={[styles.avatar, styles.avatarPh]}>
                <Text style={styles.avatarTxt}>{item.name?.[0] ?? '?'}</Text>
              </View>
            )}
            <View>
              <Text style={styles.nome}>{item.name}</Text>
              <Text style={styles.tel}>
                {item.phoneNumbers?.[0]?.number ?? 'Sem telefone'}
              </Text>
            </View>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  botoes: { flexDirection: 'row', gap: 12, padding: 16 },
  btn: { flex: 1, backgroundColor: '#3949ab', paddingVertical: 12, borderRadius: 8, alignItems: 'center' },
  btnTxt: { color: '#fff', fontWeight: '600' },
  btnOutline: { flex: 1, borderWidth: 1, borderColor: '#3949ab', paddingVertical: 12, borderRadius: 8, alignItems: 'center' },
  btnOutlineTxt: { color: '#3949ab', fontWeight: '600' },
  item: { flexDirection: 'row', alignItems: 'center', gap: 12, paddingHorizontal: 16, paddingVertical: 10, borderBottomWidth: 1, borderBottomColor: '#f0f0f0' },
  avatar: { width: 40, height: 40, borderRadius: 20 },
  avatarPh: { backgroundColor: '#e0e0e0', justifyContent: 'center', alignItems: 'center' },
  avatarTxt: { fontWeight: '600', color: '#666' },
  nome: { fontWeight: '600' },
  tel: { fontSize: 13, color: '#999' },
});
```

---

### React Web — Contact Picker API + download .ics

> **Contact Picker API:** suportada apenas em Chrome Android. Para outros navegadores, não há acesso programático aos contatos.

```tsx
// src/features/contatos/ContactsPage.tsx
export function ContactsPage() {
  const carregarContatos = async () => {
    if (!('contacts' in navigator)) {
      alert('Contact Picker API não suportada neste navegador.');
      return;
    }
    try {
      const contatos = await (navigator as any).contacts.select(
        ['name', 'tel'],
        { multiple: true },
      );
      console.log('Contatos selecionados:', contatos);
    } catch { /* cancelado */ }
  };

  const criarEvento = () => {
    const inicio = new Date();
    inicio.setDate(inicio.getDate() + 1);
    inicio.setHours(14, 0, 0, 0);
    const fim = new Date(inicio);
    fim.setHours(15);

    const fmt = (d: Date) => d.toISOString().replace(/[-:]/g, '').replace(/\.\d{3}/, '');

    const ics = [
      'BEGIN:VCALENDAR', 'VERSION:2.0',
      'BEGIN:VEVENT',
      `DTSTART:${fmt(inicio)}`,
      `DTEND:${fmt(fim)}`,
      'SUMMARY:Reunião de Projeto',
      'DESCRIPTION:Discussão sobre o novo módulo do app.',
      'LOCATION:Sala 3 — Escritório',
      'END:VEVENT',
      'END:VCALENDAR',
    ].join('\r\n');

    const blob = new Blob([ics], { type: 'text/calendar' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'evento.ics';
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <div className="container py-4" style={{ maxWidth: 600 }}>
      <h4 className="mb-4"><i className="fas fa-address-book me-2" />Contatos & Calendário</h4>
      <div className="d-flex gap-3 mb-4">
        <button className="btn btn-primary flex-fill" onClick={carregarContatos}>
          <i className="fas fa-users me-2" />Selecionar Contatos
        </button>
        <button className="btn btn-outline-primary flex-fill" onClick={criarEvento}>
          <i className="fas fa-calendar-plus me-2" />Criar Evento (.ics)
        </button>
      </div>
      <div className="alert alert-info small">
        <i className="fas fa-info-circle me-1" />
        A Contact Picker API funciona apenas no Chrome Android. O evento é exportado como arquivo .ics compatível com qualquer app de calendário.
      </div>
    </div>
  );
}
```

---

### Equivalência — Contatos e Calendário

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Ler contatos | `flutter_contacts` | `expo-contacts` | Contact Picker API (Chrome Android) |
| Criar evento | `add_2_calendar` (abre calendário) | `expo-calendar` (cria direto) | Download de arquivo `.ics` |
| Permissões | Manual via manifest/plist | `requestPermissionsAsync` | Prompt automático |
| Foto do contato | `Contact.photo` (bytes) | `Contact.image.uri` | N/A |

---

## 7. Upload de Arquivos com Progresso

Upload de arquivos com barra de progresso, cancelamento e upload múltiplo.

### Flutter — Dio + onSendProgress

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  dio: ^5.7.0
  file_picker: ^8.1.0
```

**2. Serviço de upload**

```dart
// lib/core/upload/upload_service.dart
import 'package:dio/dio.dart';

class UploadService {
  final _dio = Dio();

  Future<void> upload({
    required String url,
    required String caminhoArquivo,
    required String nomeArquivo,
    required void Function(int enviado, int total) onProgresso,
    CancelToken? cancelToken,
  }) async {
    final formData = FormData.fromMap({
      'file': await MultipartFile.fromFile(
        caminhoArquivo,
        filename: nomeArquivo,
      ),
    });

    await _dio.post(
      url,
      data: formData,
      cancelToken: cancelToken,
      onSendProgress: onProgresso,
    );
  }
}
```

**3. Tela de upload**

```dart
// lib/features/upload/presentation/upload_screen.dart
import 'package:flutter/material.dart';
import 'package:file_picker/file_picker.dart';
import 'package:dio/dio.dart';
import '../../../core/upload/upload_service.dart';

class UploadScreen extends StatefulWidget {
  const UploadScreen({super.key});

  @override
  State<UploadScreen> createState() => _UploadScreenState();
}

class _UploadScreenState extends State<UploadScreen> {
  final _uploadService = UploadService();
  final List<_UploadItem> _uploads = [];

  Future<void> _selecionarArquivos() async {
    final result = await FilePicker.platform.pickFiles(allowMultiple: true);
    if (result == null) return;

    for (final file in result.files) {
      if (file.path == null) continue;
      final item = _UploadItem(
        nome: file.name,
        tamanho: file.size,
        caminho: file.path!,
      );
      setState(() => _uploads.add(item));
      _iniciarUpload(item);
    }
  }

  Future<void> _iniciarUpload(_UploadItem item) async {
    try {
      await _uploadService.upload(
        url: 'https://httpbin.org/post',
        caminhoArquivo: item.caminho,
        nomeArquivo: item.nome,
        cancelToken: item.cancelToken,
        onProgresso: (enviado, total) {
          setState(() {
            item.progresso = enviado / total;
            item.status = 'uploading';
          });
        },
      );
      setState(() => item.status = 'done');
    } on DioException catch (e) {
      setState(() {
        item.status = e.type == DioExceptionType.cancel ? 'cancelled' : 'error';
      });
    }
  }

  String _formatarTamanho(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1048576) return '${(bytes / 1024).toStringAsFixed(1)} KB';
    return '${(bytes / 1048576).toStringAsFixed(1)} MB';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Upload de Arquivos')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: FilledButton.icon(
              onPressed: _selecionarArquivos,
              icon: const Icon(Icons.upload_file),
              label: const Text('Selecionar Arquivos'),
              style: FilledButton.styleFrom(
                minimumSize: const Size(double.infinity, 48),
              ),
            ),
          ),
          Expanded(
            child: ListView.builder(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              itemCount: _uploads.length,
              itemBuilder: (_, i) {
                final u = _uploads[i];
                return Card(
                  margin: const EdgeInsets.only(bottom: 8),
                  child: Padding(
                    padding: const EdgeInsets.all(12),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Row(
                          children: [
                            Icon(
                              u.status == 'done'
                                  ? Icons.check_circle
                                  : u.status == 'error'
                                      ? Icons.error
                                      : Icons.insert_drive_file,
                              color: u.status == 'done'
                                  ? Colors.green
                                  : u.status == 'error'
                                      ? Colors.red
                                      : Colors.grey,
                            ),
                            const SizedBox(width: 8),
                            Expanded(
                              child: Text(u.nome, overflow: TextOverflow.ellipsis),
                            ),
                            Text(_formatarTamanho(u.tamanho),
                                style: const TextStyle(fontSize: 12, color: Colors.grey)),
                            if (u.status == 'uploading')
                              IconButton(
                                icon: const Icon(Icons.close, size: 18),
                                onPressed: () => u.cancelToken.cancel(),
                              ),
                          ],
                        ),
                        if (u.status == 'uploading') ...[
                          const SizedBox(height: 8),
                          LinearProgressIndicator(value: u.progresso),
                          const SizedBox(height: 4),
                          Text('${(u.progresso * 100).toInt()}%',
                              style: const TextStyle(fontSize: 11, color: Colors.grey)),
                        ],
                      ],
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class _UploadItem {
  final String nome;
  final int tamanho;
  final String caminho;
  final CancelToken cancelToken = CancelToken();
  double progresso = 0;
  String status = 'pending'; // pending, uploading, done, error, cancelled

  _UploadItem({required this.nome, required this.tamanho, required this.caminho});
}
```

---

### React Native — XMLHttpRequest + progresso

```tsx
// src/features/upload/UploadScreen.tsx
import React, { useState, useRef } from 'react';
import { View, Text, TouchableOpacity, FlatList, StyleSheet } from 'react-native';
import * as DocumentPicker from 'expo-document-picker';

interface UploadItem {
  id: string;
  nome: string;
  tamanho: number;
  uri: string;
  progresso: number;
  status: 'pending' | 'uploading' | 'done' | 'error' | 'cancelled';
  xhr?: XMLHttpRequest;
}

export function UploadScreen() {
  const [uploads, setUploads] = useState<UploadItem[]>([]);

  const atualizar = (id: string, patch: Partial<UploadItem>) => {
    setUploads((prev) => prev.map((u) => (u.id === id ? { ...u, ...patch } : u)));
  };

  const selecionarArquivos = async () => {
    const result = await DocumentPicker.getDocumentAsync({ multiple: true });
    if (result.canceled) return;

    for (const asset of result.assets) {
      const item: UploadItem = {
        id: `upload-${Date.now()}-${Math.random()}`,
        nome: asset.name,
        tamanho: asset.size ?? 0,
        uri: asset.uri,
        progresso: 0,
        status: 'pending',
      };
      setUploads((prev) => [...prev, item]);
      enviar(item);
    }
  };

  const enviar = (item: UploadItem) => {
    const xhr = new XMLHttpRequest();
    item.xhr = xhr;

    const formData = new FormData();
    formData.append('file', { uri: item.uri, name: item.nome, type: 'application/octet-stream' } as any);

    xhr.upload.onprogress = (e) => {
      if (e.lengthComputable) {
        atualizar(item.id, { progresso: e.loaded / e.total, status: 'uploading' });
      }
    };
    xhr.onload = () => atualizar(item.id, { status: 'done', progresso: 1 });
    xhr.onerror = () => atualizar(item.id, { status: 'error' });
    xhr.onabort = () => atualizar(item.id, { status: 'cancelled' });

    xhr.open('POST', 'https://httpbin.org/post');
    xhr.send(formData);
  };

  const fmt = (bytes: number) =>
    bytes < 1024 ? `${bytes} B` :
    bytes < 1048576 ? `${(bytes / 1024).toFixed(1)} KB` :
    `${(bytes / 1048576).toFixed(1)} MB`;

  return (
    <View style={styles.container}>
      <TouchableOpacity style={styles.btn} onPress={selecionarArquivos}>
        <Text style={styles.btnTxt}>📁 Selecionar Arquivos</Text>
      </TouchableOpacity>
      <FlatList
        data={uploads}
        keyExtractor={(u) => u.id}
        contentContainerStyle={styles.lista}
        renderItem={({ item }) => (
          <View style={styles.card}>
            <View style={styles.cardRow}>
              <Text style={styles.nome} numberOfLines={1}>{item.nome}</Text>
              <Text style={styles.tamanho}>{fmt(item.tamanho)}</Text>
              {item.status === 'uploading' && (
                <TouchableOpacity onPress={() => item.xhr?.abort()}>
                  <Text style={styles.cancelar}>✕</Text>
                </TouchableOpacity>
              )}
            </View>
            {item.status === 'uploading' && (
              <View style={styles.barraFundo}>
                <View style={[styles.barraPreenchida, { width: `${item.progresso * 100}%` }]} />
              </View>
            )}
            {item.status === 'done' && <Text style={styles.statusOk}>✓ Concluído</Text>}
            {item.status === 'error' && <Text style={styles.statusErro}>✕ Erro</Text>}
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#f9f9f9' },
  btn: { backgroundColor: '#3949ab', paddingVertical: 14, borderRadius: 8, alignItems: 'center', marginBottom: 16 },
  btnTxt: { color: '#fff', fontWeight: '600', fontSize: 15 },
  lista: { gap: 8 },
  card: { backgroundColor: '#fff', borderRadius: 10, padding: 12, elevation: 1 },
  cardRow: { flexDirection: 'row', alignItems: 'center', gap: 8 },
  nome: { flex: 1, fontWeight: '500' },
  tamanho: { fontSize: 12, color: '#999' },
  cancelar: { fontSize: 16, color: '#e53935', paddingHorizontal: 4 },
  barraFundo: { height: 4, backgroundColor: '#e0e0e0', borderRadius: 2, marginTop: 8 },
  barraPreenchida: { height: 4, backgroundColor: '#3949ab', borderRadius: 2 },
  statusOk: { fontSize: 12, color: '#4caf50', marginTop: 4 },
  statusErro: { fontSize: 12, color: '#e53935', marginTop: 4 },
});
```

---

### React Web — XMLHttpRequest + Bootstrap Progress

```tsx
// src/features/upload/UploadPage.tsx
import { useState, useRef } from 'react';

interface UploadItem {
  id: string;
  nome: string;
  tamanho: number;
  file: File;
  progresso: number;
  status: 'pending' | 'uploading' | 'done' | 'error' | 'cancelled';
  xhr?: XMLHttpRequest;
}

export function UploadPage() {
  const [uploads, setUploads] = useState<UploadItem[]>([]);
  const fileRef = useRef<HTMLInputElement>(null);

  const atualizar = (id: string, patch: Partial<UploadItem>) => {
    setUploads((prev) => prev.map((u) => (u.id === id ? { ...u, ...patch } : u)));
  };

  const selecionar = (e: React.ChangeEvent<HTMLInputElement>) => {
    const files = e.target.files;
    if (!files) return;

    Array.from(files).forEach((file) => {
      const item: UploadItem = {
        id: `upload-${Date.now()}-${Math.random()}`,
        nome: file.name,
        tamanho: file.size,
        file,
        progresso: 0,
        status: 'pending',
      };
      setUploads((prev) => [...prev, item]);
      enviar(item);
    });
    e.target.value = '';
  };

  const enviar = (item: UploadItem) => {
    const xhr = new XMLHttpRequest();
    item.xhr = xhr;
    const formData = new FormData();
    formData.append('file', item.file);

    xhr.upload.onprogress = (e) => {
      if (e.lengthComputable) {
        atualizar(item.id, { progresso: e.loaded / e.total, status: 'uploading' });
      }
    };
    xhr.onload = () => atualizar(item.id, { status: 'done', progresso: 1 });
    xhr.onerror = () => atualizar(item.id, { status: 'error' });
    xhr.onabort = () => atualizar(item.id, { status: 'cancelled' });

    xhr.open('POST', 'https://httpbin.org/post');
    xhr.send(formData);
  };

  const fmt = (bytes: number) =>
    bytes < 1024 ? `${bytes} B` :
    bytes < 1048576 ? `${(bytes / 1024).toFixed(1)} KB` :
    `${(bytes / 1048576).toFixed(1)} MB`;

  return (
    <div className="container py-4" style={{ maxWidth: 640 }}>
      <h4 className="mb-4"><i className="fas fa-cloud-upload-alt me-2" />Upload de Arquivos</h4>

      <input ref={fileRef} type="file" multiple className="d-none" onChange={selecionar} />
      <button className="btn btn-primary w-100 mb-4" onClick={() => fileRef.current?.click()}>
        <i className="fas fa-folder-open me-2" />Selecionar Arquivos
      </button>

      {uploads.map((u) => (
        <div key={u.id} className="card mb-2">
          <div className="card-body py-2 px-3">
            <div className="d-flex align-items-center gap-2">
              <i className={`fas fa-${
                u.status === 'done' ? 'check-circle text-success' :
                u.status === 'error' ? 'times-circle text-danger' :
                'file text-muted'
              }`} />
              <span className="flex-grow-1 text-truncate">{u.nome}</span>
              <small className="text-muted">{fmt(u.tamanho)}</small>
              {u.status === 'uploading' && (
                <button className="btn btn-sm btn-link text-danger p-0" onClick={() => u.xhr?.abort()}>
                  <i className="fas fa-times" />
                </button>
              )}
            </div>
            {u.status === 'uploading' && (
              <div className="progress mt-2" style={{ height: 6 }}>
                <div
                  className="progress-bar"
                  style={{ width: `${u.progresso * 100}%` }}
                />
              </div>
            )}
          </div>
        </div>
      ))}
    </div>
  );
}
```

---

### Equivalência — Upload com Progresso

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Seletor de arquivos | `file_picker` | `expo-document-picker` | `<input type="file" multiple>` |
| Upload com progresso | `Dio.onSendProgress` | `XMLHttpRequest.upload.onprogress` | `XMLHttpRequest.upload.onprogress` |
| Cancelar upload | `CancelToken.cancel()` | `xhr.abort()` | `xhr.abort()` |
| Barra de progresso | `LinearProgressIndicator` | `View` com width % | Bootstrap `progress-bar` |

---

## 8. Cache de Imagens e Dados com Estratégia

Cache de imagens e dados de API com estratégias cache-first e network-first, invalidação por tempo e tamanho máximo.

### Flutter — cached_network_image + Hive

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  cached_network_image: ^3.4.0
  hive_flutter: ^1.1.0
```

**2. Serviço de cache de dados**

```dart
// lib/core/cache/data_cache_service.dart
import 'package:hive_flutter/hive_flutter.dart';

class DataCacheService {
  static const _boxName = 'api_cache';
  late Box<Map> _box;

  Future<void> inicializar() async {
    await Hive.initFlutter();
    _box = await Hive.openBox<Map>(_boxName);
  }

  // Cache-first: retorna cache se válido, senão busca na rede
  Future<Map<String, dynamic>?> cacheFirst({
    required String chave,
    required Future<Map<String, dynamic>> Function() buscar,
    Duration ttl = const Duration(minutes: 10),
  }) async {
    final cached = _box.get(chave);
    if (cached != null) {
      final timestamp = cached['_cached_at'] as int? ?? 0;
      final idade = DateTime.now().millisecondsSinceEpoch - timestamp;
      if (idade < ttl.inMilliseconds) {
        return Map<String, dynamic>.from(cached)..remove('_cached_at');
      }
    }

    final dados = await buscar();
    await _box.put(chave, {
      ...dados,
      '_cached_at': DateTime.now().millisecondsSinceEpoch,
    });
    return dados;
  }

  // Network-first: tenta rede, fallback para cache
  Future<Map<String, dynamic>?> networkFirst({
    required String chave,
    required Future<Map<String, dynamic>> Function() buscar,
  }) async {
    try {
      final dados = await buscar();
      await _box.put(chave, {
        ...dados,
        '_cached_at': DateTime.now().millisecondsSinceEpoch,
      });
      return dados;
    } catch (_) {
      final cached = _box.get(chave);
      if (cached != null) {
        return Map<String, dynamic>.from(cached)..remove('_cached_at');
      }
      return null;
    }
  }

  Future<void> invalidar(String chave) => _box.delete(chave);

  Future<void> limparTudo() => _box.clear();
}
```

**3. Uso com imagens (cached_network_image já tem cache built-in)**

```dart
// Em qualquer widget:
CachedNetworkImage(
  imageUrl: 'https://picsum.photos/400/300',
  placeholder: (_, __) => const Center(child: CircularProgressIndicator(strokeWidth: 2)),
  errorWidget: (_, __, ___) => const Icon(Icons.broken_image),
  fadeInDuration: const Duration(milliseconds: 300),
  memCacheWidth: 400,  // limita tamanho em memória
)
```

---

### React Native — expo-image + MMKV

**1. Dependências**

```bash
npx expo install expo-image
npm install react-native-mmkv
```

**2. Serviço de cache**

```typescript
// src/core/cache/dataCacheService.ts
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV({ id: 'api-cache' });

interface CachedEntry<T> {
  data: T;
  cachedAt: number;
}

export function cacheFirst<T>(
  chave: string,
  buscar: () => Promise<T>,
  ttlMs = 10 * 60 * 1000,
): Promise<T> {
  const raw = storage.getString(chave);
  if (raw) {
    const entry: CachedEntry<T> = JSON.parse(raw);
    if (Date.now() - entry.cachedAt < ttlMs) return Promise.resolve(entry.data);
  }
  return buscar().then((data) => {
    storage.set(chave, JSON.stringify({ data, cachedAt: Date.now() }));
    return data;
  });
}

export async function networkFirst<T>(
  chave: string,
  buscar: () => Promise<T>,
): Promise<T | null> {
  try {
    const data = await buscar();
    storage.set(chave, JSON.stringify({ data, cachedAt: Date.now() }));
    return data;
  } catch {
    const raw = storage.getString(chave);
    if (raw) return (JSON.parse(raw) as CachedEntry<T>).data;
    return null;
  }
}

export function invalidar(chave: string) { storage.delete(chave); }
export function limparTudo() { storage.clearAll(); }
```

**3. Cache de imagens (expo-image já tem cache built-in)**

```tsx
import { Image } from 'expo-image';

<Image
  source={{ uri: 'https://picsum.photos/400/300' }}
  style={{ width: '100%', height: 200 }}
  contentFit="cover"
  cachePolicy="memory-disk"  // cache automático em memória e disco
  transition={300}
  placeholder={require('../../assets/placeholder.png')}
/>
```

---

### React Web — Service Worker Cache API

**1. Serviço de cache de dados**

```typescript
// src/core/cache/dataCacheService.ts
interface CachedEntry<T> {
  data: T;
  cachedAt: number;
}

export async function cacheFirst<T>(
  chave: string,
  buscar: () => Promise<T>,
  ttlMs = 10 * 60 * 1000,
): Promise<T> {
  const raw = localStorage.getItem(`cache:${chave}`);
  if (raw) {
    const entry: CachedEntry<T> = JSON.parse(raw);
    if (Date.now() - entry.cachedAt < ttlMs) return entry.data;
  }

  const data = await buscar();
  localStorage.setItem(`cache:${chave}`, JSON.stringify({ data, cachedAt: Date.now() }));
  return data;
}

export async function networkFirst<T>(
  chave: string,
  buscar: () => Promise<T>,
): Promise<T | null> {
  try {
    const data = await buscar();
    localStorage.setItem(`cache:${chave}`, JSON.stringify({ data, cachedAt: Date.now() }));
    return data;
  } catch {
    const raw = localStorage.getItem(`cache:${chave}`);
    if (raw) return (JSON.parse(raw) as CachedEntry<T>).data;
    return null;
  }
}

export function invalidar(chave: string) { localStorage.removeItem(`cache:${chave}`); }
```

**2. Service Worker para cache de imagens — `public/sw-cache.js`**

```javascript
// public/sw-cache.js
const CACHE_NAME = 'img-cache-v1';
const MAX_ENTRIES = 100;

self.addEventListener('fetch', (event) => {
  const { request } = event;
  if (!request.url.match(/\.(jpg|jpeg|png|gif|webp|svg)(\?.*)?$/i)) return;

  event.respondWith(
    caches.open(CACHE_NAME).then(async (cache) => {
      const cached = await cache.match(request);
      if (cached) return cached;

      const response = await fetch(request);
      if (response.ok) {
        cache.put(request, response.clone());
        // Limitar tamanho do cache
        const keys = await cache.keys();
        if (keys.length > MAX_ENTRIES) {
          await cache.delete(keys[0]);
        }
      }
      return response;
    }),
  );
});
```

**3. Registro do service worker**

```typescript
// src/main.tsx (adicionar)
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw-cache.js');
}
```

---

### Equivalência — Cache

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Cache de imagens | `cached_network_image` (disco + memória) | `expo-image` (`cachePolicy`) | Service Worker Cache API |
| Cache de dados | `hive_flutter` (NoSQL local) | `react-native-mmkv` (key-value rápido) | `localStorage` |
| Cache-first | TTL + timestamp | TTL + timestamp | TTL + timestamp |
| Network-first | try/catch + fallback | try/catch + fallback | try/catch + fallback |
| Tamanho máximo | Manual | Manual | `MAX_ENTRIES` no SW |
| Invalidação | `box.delete(chave)` | `storage.delete(chave)` | `localStorage.removeItem` |

---

## 9. Paginação com Cursor

Alternativa à paginação por offset. O servidor retorna um `cursor` (ponteiro opaco) junto com os dados, e o cliente envia esse cursor para obter a próxima página. Padrão usado por APIs do Twitter, Slack, GitHub GraphQL, entre outros.

**Vantagens sobre offset:** não pula/duplica registros quando itens são inseridos ou removidos durante a navegação.

### Formato da API

```
GET /api/itens?limit=10&cursor=eyJpZCI6MTAwfQ

Resposta:
{
  "data": [ ... ],
  "nextCursor": "eyJpZCI6MTEwfQ",   // null se não há mais páginas
  "hasMore": true
}
```

### Flutter — cursor + ListView

```dart
// lib/features/cursor/data/cursor_service.dart
import 'dart:convert';

class CursorPage<T> {
  final List<T> data;
  final String? nextCursor;
  final bool hasMore;

  CursorPage({required this.data, this.nextCursor, required this.hasMore});
}

class CursorService {
  final _todos = List.generate(85, (i) => 'Item ${i + 1}');

  Future<CursorPage<String>> buscar({String? cursor, int limit = 15}) async {
    await Future.delayed(const Duration(milliseconds: 400));

    int startIndex = 0;
    if (cursor != null) {
      final decoded = json.decode(utf8.decode(base64Decode(cursor)));
      startIndex = decoded['offset'] as int;
    }

    final endIndex = (startIndex + limit).clamp(0, _todos.length);
    final data = _todos.sublist(startIndex, endIndex);
    final hasMore = endIndex < _todos.length;

    return CursorPage(
      data: data,
      nextCursor: hasMore
          ? base64Encode(utf8.encode(json.encode({'offset': endIndex})))
          : null,
      hasMore: hasMore,
    );
  }
}
```

```dart
// lib/features/cursor/presentation/cursor_list_screen.dart
import 'package:flutter/material.dart';
import '../data/cursor_service.dart';

class CursorListScreen extends StatefulWidget {
  const CursorListScreen({super.key});

  @override
  State<CursorListScreen> createState() => _CursorListScreenState();
}

class _CursorListScreenState extends State<CursorListScreen> {
  final _service = CursorService();
  final _scrollCtrl = ScrollController();
  final List<String> _itens = [];
  String? _nextCursor;
  bool _carregando = false;
  bool _hasMore = true;

  @override
  void initState() {
    super.initState();
    _carregar();
    _scrollCtrl.addListener(() {
      if (_scrollCtrl.position.pixels >= _scrollCtrl.position.maxScrollExtent - 200) {
        _carregar();
      }
    });
  }

  Future<void> _carregar() async {
    if (_carregando || !_hasMore) return;
    setState(() => _carregando = true);

    final page = await _service.buscar(cursor: _nextCursor);
    setState(() {
      _itens.addAll(page.data);
      _nextCursor = page.nextCursor;
      _hasMore = page.hasMore;
      _carregando = false;
    });
  }

  @override
  void dispose() {
    _scrollCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Cursor Pagination (${_itens.length} itens)')),
      body: ListView.builder(
        controller: _scrollCtrl,
        itemCount: _itens.length + (_carregando ? 1 : 0),
        itemBuilder: (_, i) {
          if (i >= _itens.length) {
            return const Center(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: CircularProgressIndicator(),
              ),
            );
          }
          return ListTile(title: Text(_itens[i]));
        },
      ),
    );
  }
}
```

---

### React Native — cursor + FlatList

```tsx
// src/features/cursor/CursorListScreen.tsx
import React, { useEffect, useState, useCallback } from 'react';
import { View, Text, FlatList, ActivityIndicator, StyleSheet } from 'react-native';

interface CursorPage { data: string[]; nextCursor: string | null; hasMore: boolean; }

async function buscar(cursor?: string | null, limit = 15): Promise<CursorPage> {
  await new Promise((r) => setTimeout(r, 400));
  const todos = Array.from({ length: 85 }, (_, i) => `Item ${i + 1}`);
  const start = cursor ? JSON.parse(atob(cursor)).offset : 0;
  const end = Math.min(start + limit, todos.length);
  return {
    data: todos.slice(start, end),
    nextCursor: end < todos.length ? btoa(JSON.stringify({ offset: end })) : null,
    hasMore: end < todos.length,
  };
}

export function CursorListScreen() {
  const [itens, setItens] = useState<string[]>([]);
  const [nextCursor, setNextCursor] = useState<string | null>(null);
  const [carregando, setCarregando] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const carregar = useCallback(async () => {
    if (carregando || !hasMore) return;
    setCarregando(true);
    const page = await buscar(nextCursor);
    setItens((prev) => [...prev, ...page.data]);
    setNextCursor(page.nextCursor);
    setHasMore(page.hasMore);
    setCarregando(false);
  }, [carregando, hasMore, nextCursor]);

  useEffect(() => { carregar(); }, []);

  return (
    <FlatList
      data={itens}
      keyExtractor={(_, i) => String(i)}
      onEndReached={carregar}
      onEndReachedThreshold={0.3}
      renderItem={({ item }) => (
        <View style={styles.item}><Text>{item}</Text></View>
      )}
      ListFooterComponent={carregando ? <ActivityIndicator style={styles.loader} /> : null}
    />
  );
}

const styles = StyleSheet.create({
  item: { padding: 16, borderBottomWidth: 1, borderBottomColor: '#f0f0f0' },
  loader: { padding: 20 },
});
```

---

### React Web — cursor + IntersectionObserver

```tsx
// src/features/cursor/CursorListPage.tsx
import { useState, useCallback, useEffect, useRef } from 'react';

interface CursorPage { data: string[]; nextCursor: string | null; hasMore: boolean; }

async function buscar(cursor?: string | null): Promise<CursorPage> {
  await new Promise((r) => setTimeout(r, 400));
  const todos = Array.from({ length: 85 }, (_, i) => `Item ${i + 1}`);
  const start = cursor ? JSON.parse(atob(cursor)).offset : 0;
  const end = Math.min(start + 15, todos.length);
  return {
    data: todos.slice(start, end),
    nextCursor: end < todos.length ? btoa(JSON.stringify({ offset: end })) : null,
    hasMore: end < todos.length,
  };
}

export function CursorListPage() {
  const [itens, setItens] = useState<string[]>([]);
  const [nextCursor, setNextCursor] = useState<string | null>(null);
  const [carregando, setCarregando] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const sentinelRef = useRef<HTMLDivElement>(null);

  const carregar = useCallback(async () => {
    if (carregando || !hasMore) return;
    setCarregando(true);
    const page = await buscar(nextCursor);
    setItens((prev) => [...prev, ...page.data]);
    setNextCursor(page.nextCursor);
    setHasMore(page.hasMore);
    setCarregando(false);
  }, [carregando, hasMore, nextCursor]);

  useEffect(() => { carregar(); }, []);

  useEffect(() => {
    if (!sentinelRef.current || !hasMore) return;
    const observer = new IntersectionObserver(
      ([entry]) => { if (entry.isIntersecting) carregar(); },
      { rootMargin: '200px' },
    );
    observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [carregar, hasMore]);

  return (
    <div className="container py-4" style={{ maxWidth: 600 }}>
      <h4 className="mb-3">Cursor Pagination ({itens.length} itens)</h4>
      <ul className="list-group">
        {itens.map((item, i) => (
          <li key={i} className="list-group-item">{item}</li>
        ))}
      </ul>
      {carregando && (
        <div className="text-center py-3"><span className="spinner-border spinner-border-sm" /></div>
      )}
      {hasMore && !carregando && <div ref={sentinelRef} />}
      {!hasMore && <p className="text-muted text-center mt-3">Fim da lista.</p>}
    </div>
  );
}
```

---

### Equivalência — Paginação com Cursor

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Cursor (opaco) | base64-encoded JSON | base64-encoded JSON | base64-encoded JSON |
| Scroll infinito | `ScrollController` listener | `FlatList.onEndReached` | `IntersectionObserver` |
| Indicador de loading | `CircularProgressIndicator` | `ActivityIndicator` | Bootstrap `spinner-border` |
| Detecção de fim | `hasMore` flag | `hasMore` flag | `hasMore` flag |
| Cursor nulo = primeira página | Sim | Sim | Sim |

---

## 10. Pagamentos In-App

Integração com Stripe para processar pagamentos com cartão de crédito. O fluxo seguro é: o **backend** cria um `PaymentIntent` (com valor e moeda) e retorna o `clientSecret` ao app. O app coleta os dados do cartão usando o SDK do Stripe (que nunca envia dados sensíveis ao seu servidor) e confirma o pagamento.

> **Pré-requisito:** conta Stripe com chaves de API (publishable key + secret key). O backend precisa de um endpoint `POST /create-payment-intent` que usa a secret key para criar o PaymentIntent.

### Flutter — flutter_stripe

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  flutter_stripe: ^11.2.0
  dio: ^5.7.0
```

**2. Configuração**

```dart
// lib/main.dart (no início do main)
import 'package:flutter_stripe/flutter_stripe.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  Stripe.publishableKey = 'pk_test_SEU_PUBLISHABLE_KEY';
  runApp(const App());
}
```

**3. Serviço de pagamento**

```dart
// lib/core/payment/payment_service.dart
import 'package:dio/dio.dart';
import 'package:flutter_stripe/flutter_stripe.dart';

class PaymentService {
  final _dio = Dio(BaseOptions(baseUrl: 'https://seu-backend.com'));

  Future<String> criarPaymentIntent({
    required int valorCentavos,
    String moeda = 'brl',
  }) async {
    final response = await _dio.post('/create-payment-intent', data: {
      'amount': valorCentavos,
      'currency': moeda,
    });
    return response.data['clientSecret'] as String;
  }

  Future<bool> processarPagamento({
    required int valorCentavos,
    String moeda = 'brl',
  }) async {
    // 1. Backend cria PaymentIntent
    final clientSecret = await criarPaymentIntent(
      valorCentavos: valorCentavos,
      moeda: moeda,
    );

    // 2. Inicializar payment sheet
    await Stripe.instance.initPaymentSheet(
      paymentSheetParameters: SetupPaymentSheetParameters(
        paymentIntentClientSecret: clientSecret,
        merchantDisplayName: 'MeuApp',
        style: ThemeMode.system,
      ),
    );

    // 3. Apresentar payment sheet
    await Stripe.instance.presentPaymentSheet();

    return true; // se chegar aqui, pagamento confirmado
  }
}
```

**4. Tela de checkout**

```dart
// lib/features/pagamento/presentation/checkout_screen.dart
import 'package:flutter/material.dart';
import '../../../core/payment/payment_service.dart';

class CheckoutScreen extends StatefulWidget {
  const CheckoutScreen({super.key});

  @override
  State<CheckoutScreen> createState() => _CheckoutScreenState();
}

class _CheckoutScreenState extends State<CheckoutScreen> {
  final _paymentService = PaymentService();
  bool _processando = false;
  String? _resultado;

  Future<void> _pagar() async {
    setState(() { _processando = true; _resultado = null; });

    try {
      await _paymentService.processarPagamento(valorCentavos: 4990); // R$ 49,90
      setState(() => _resultado = 'sucesso');
    } catch (e) {
      setState(() => _resultado = 'erro');
    } finally {
      setState(() => _processando = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Checkout')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('Resumo do pedido',
                        style: Theme.of(context).textTheme.titleMedium),
                    const SizedBox(height: 12),
                    const Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [Text('Plano Premium'), Text('R\$ 49,90')],
                    ),
                    const Divider(height: 24),
                    const Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text('Total', style: TextStyle(fontWeight: FontWeight.bold)),
                        Text('R\$ 49,90', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 18)),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            const Spacer(),
            SizedBox(
              width: double.infinity,
              child: FilledButton.icon(
                onPressed: _processando ? null : _pagar,
                icon: _processando
                    ? const SizedBox(width: 20, height: 20,
                        child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white))
                    : const Icon(Icons.payment),
                label: Text(_processando ? 'Processando…' : 'Pagar R\$ 49,90'),
                style: FilledButton.styleFrom(
                  minimumSize: const Size(0, 52),
                ),
              ),
            ),
            if (_resultado == 'sucesso')
              const Padding(
                padding: EdgeInsets.only(top: 16),
                child: Row(
                  children: [
                    Icon(Icons.check_circle, color: Colors.green),
                    SizedBox(width: 8),
                    Text('Pagamento aprovado!', style: TextStyle(color: Colors.green)),
                  ],
                ),
              ),
            if (_resultado == 'erro')
              const Padding(
                padding: EdgeInsets.only(top: 16),
                child: Row(
                  children: [
                    Icon(Icons.error, color: Colors.red),
                    SizedBox(width: 8),
                    Text('Pagamento falhou ou foi cancelado.', style: TextStyle(color: Colors.red)),
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

---

### React Native — @stripe/stripe-react-native

**1. Dependências**

```bash
npx expo install @stripe/stripe-react-native
```

**2. Provider (no layout raiz)**

```tsx
// app/_layout.tsx
import { StripeProvider } from '@stripe/stripe-react-native';

export default function Layout() {
  return (
    <StripeProvider publishableKey="pk_test_SEU_PUBLISHABLE_KEY">
      {/* ... */}
    </StripeProvider>
  );
}
```

**3. Tela de checkout**

```tsx
// src/features/pagamento/CheckoutScreen.tsx
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useStripe } from '@stripe/stripe-react-native';

export function CheckoutScreen() {
  const { initPaymentSheet, presentPaymentSheet } = useStripe();
  const [processando, setProcessando] = useState(false);

  const pagar = async () => {
    setProcessando(true);

    // 1. Buscar clientSecret do backend
    const res = await fetch('https://seu-backend.com/create-payment-intent', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ amount: 4990, currency: 'brl' }),
    });
    const { clientSecret } = await res.json();

    // 2. Inicializar payment sheet
    const { error: initError } = await initPaymentSheet({
      paymentIntentClientSecret: clientSecret,
      merchantDisplayName: 'MeuApp',
    });
    if (initError) { setProcessando(false); return; }

    // 3. Apresentar
    const { error } = await presentPaymentSheet();
    setProcessando(false);

    if (error) {
      Alert.alert('Cancelado', error.message);
    } else {
      Alert.alert('Sucesso', 'Pagamento aprovado!');
    }
  };

  return (
    <View style={styles.container}>
      <View style={styles.resumo}>
        <Text style={styles.tituloResumo}>Resumo do pedido</Text>
        <View style={styles.linha}>
          <Text>Plano Premium</Text>
          <Text>R$ 49,90</Text>
        </View>
        <View style={[styles.linha, styles.total]}>
          <Text style={styles.totalTxt}>Total</Text>
          <Text style={styles.totalValor}>R$ 49,90</Text>
        </View>
      </View>

      <TouchableOpacity
        style={[styles.btnPagar, processando && styles.btnDisabled]}
        onPress={pagar}
        disabled={processando}
      >
        <Text style={styles.btnPagarTxt}>
          {processando ? 'Processando…' : '💳 Pagar R$ 49,90'}
        </Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24, justifyContent: 'space-between', backgroundColor: '#fff' },
  resumo: { backgroundColor: '#f9f9f9', borderRadius: 12, padding: 16 },
  tituloResumo: { fontWeight: '600', fontSize: 16, marginBottom: 12 },
  linha: { flexDirection: 'row', justifyContent: 'space-between', marginBottom: 8 },
  total: { borderTopWidth: 1, borderTopColor: '#e0e0e0', paddingTop: 12, marginTop: 4 },
  totalTxt: { fontWeight: '700' },
  totalValor: { fontWeight: '700', fontSize: 18 },
  btnPagar: { backgroundColor: '#3949ab', paddingVertical: 16, borderRadius: 12, alignItems: 'center' },
  btnDisabled: { opacity: 0.6 },
  btnPagarTxt: { color: '#fff', fontSize: 16, fontWeight: '600' },
});
```

---

### React Web — Stripe.js + Bootstrap

**1. Dependências**

```bash
npm install @stripe/stripe-js @stripe/react-stripe-js
```

**2. Provider**

```tsx
// src/main.tsx (wrapping)
import { Elements } from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe('pk_test_SEU_PUBLISHABLE_KEY');

// Envolver o app:
<Elements stripe={stripePromise}>
  <App />
</Elements>
```

**3. Página de checkout**

```tsx
// src/features/pagamento/CheckoutPage.tsx
import { useState } from 'react';
import { useStripe, useElements, CardElement } from '@stripe/react-stripe-js';

export function CheckoutPage() {
  const stripe = useStripe();
  const elements = useElements();
  const [processando, setProcessando] = useState(false);
  const [resultado, setResultado] = useState<'sucesso' | 'erro' | null>(null);

  const pagar = async () => {
    if (!stripe || !elements) return;
    setProcessando(true);
    setResultado(null);

    // 1. Criar PaymentIntent no backend
    const res = await fetch('https://seu-backend.com/create-payment-intent', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ amount: 4990, currency: 'brl' }),
    });
    const { clientSecret } = await res.json();

    // 2. Confirmar pagamento
    const { error } = await stripe.confirmCardPayment(clientSecret, {
      payment_method: { card: elements.getElement(CardElement)! },
    });

    setProcessando(false);
    setResultado(error ? 'erro' : 'sucesso');
  };

  return (
    <div className="container py-4" style={{ maxWidth: 480 }}>
      <h4 className="mb-4"><i className="fas fa-shopping-bag me-2" />Checkout</h4>

      <div className="card mb-4">
        <div className="card-body">
          <h6>Resumo do pedido</h6>
          <div className="d-flex justify-content-between mb-2">
            <span>Plano Premium</span><span>R$ 49,90</span>
          </div>
          <hr />
          <div className="d-flex justify-content-between">
            <strong>Total</strong><strong className="fs-5">R$ 49,90</strong>
          </div>
        </div>
      </div>

      <div className="card mb-4">
        <div className="card-body">
          <label className="form-label">Dados do cartão</label>
          <div className="border rounded p-3">
            <CardElement options={{
              style: {
                base: { fontSize: '16px', color: '#333' },
                invalid: { color: '#e53935' },
              },
            }} />
          </div>
        </div>
      </div>

      <button
        className="btn btn-primary w-100 py-3"
        onClick={pagar}
        disabled={processando || !stripe}
      >
        {processando && <span className="spinner-border spinner-border-sm me-2" />}
        {processando ? 'Processando…' : 'Pagar R$ 49,90'}
      </button>

      {resultado === 'sucesso' && (
        <div className="alert alert-success mt-3">
          <i className="fas fa-check-circle me-2" />Pagamento aprovado!
        </div>
      )}
      {resultado === 'erro' && (
        <div className="alert alert-danger mt-3">
          <i className="fas fa-times-circle me-2" />Pagamento falhou ou foi cancelado.
        </div>
      )}
    </div>
  );
}
```

---

### Equivalência — Pagamentos

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| SDK | `flutter_stripe` | `@stripe/stripe-react-native` | `@stripe/stripe-js` + `@stripe/react-stripe-js` |
| Coleta de cartão | Payment Sheet (nativo) | Payment Sheet (nativo) | `<CardElement>` |
| Fluxo | `initPaymentSheet` → `presentPaymentSheet` | `initPaymentSheet` → `presentPaymentSheet` | `confirmCardPayment` |
| PaymentIntent | Backend cria, retorna `clientSecret` | Backend cria, retorna `clientSecret` | Backend cria, retorna `clientSecret` |
| Apple Pay / Google Pay | Suportado via Payment Sheet | Suportado via Payment Sheet | `PaymentRequestButtonElement` |
| PCI compliance | SDK nunca envia dados do cartão ao seu server | Idem | Idem |

---

## 11. Criptografia de Dados Locais

Cifrar dados sensíveis antes de persistir localmente — útil para tokens, dados pessoais e documentos offline.

### Flutter — encrypt + flutter_secure_storage

**1. Dependências**

```yaml
# pubspec.yaml
dependencies:
  encrypt: ^5.0.3
  flutter_secure_storage: ^9.2.0
```

**2. Serviço de criptografia**

```dart
// lib/core/crypto/crypto_service.dart
import 'dart:convert';
import 'dart:math';
import 'package:encrypt/encrypt.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class CryptoService {
  final _storage = const FlutterSecureStorage();
  static const _keyAlias = 'app_encryption_key';

  Future<Key> _getOrCreateKey() async {
    var keyBase64 = await _storage.read(key: _keyAlias);
    if (keyBase64 == null) {
      final random = Random.secure();
      final keyBytes = List<int>.generate(32, (_) => random.nextInt(256));
      keyBase64 = base64Encode(keyBytes);
      await _storage.write(key: _keyAlias, value: keyBase64);
    }
    return Key.fromBase64(keyBase64);
  }

  Future<String> criptografar(String texto) async {
    final key = await _getOrCreateKey();
    final iv = IV.fromSecureRandom(16);
    final encrypter = Encrypter(AES(key, mode: AESMode.cbc));
    final encrypted = encrypter.encrypt(texto, iv: iv);
    // Armazena IV + dados cifrados juntos
    return '${iv.base64}:${encrypted.base64}';
  }

  Future<String> descriptografar(String cifrado) async {
    final key = await _getOrCreateKey();
    final partes = cifrado.split(':');
    final iv = IV.fromBase64(partes[0]);
    final encrypted = Encrypted.fromBase64(partes[1]);
    final encrypter = Encrypter(AES(key, mode: AESMode.cbc));
    return encrypter.decrypt(encrypted, iv: iv);
  }
}
```

---

### React Native — expo-crypto + expo-secure-store

**1. Dependências**

```bash
npx expo install expo-crypto expo-secure-store
```

**2. Serviço de criptografia**

```typescript
// src/core/crypto/cryptoService.ts
import * as Crypto from 'expo-crypto';
import * as SecureStore from 'expo-secure-store';

const KEY_ALIAS = 'app_encryption_key';

async function getOrCreateKey(): Promise<string> {
  let key = await SecureStore.getItemAsync(KEY_ALIAS);
  if (!key) {
    const bytes = await Crypto.getRandomBytesAsync(32);
    key = bytesToBase64(bytes);
    await SecureStore.setItemAsync(KEY_ALIAS, key);
  }
  return key;
}

// expo-crypto não tem AES direto — usando digest como derivação + XOR cipher
// Para produção, usar react-native-quick-crypto para AES-256-CBC completo

export async function criptografar(texto: string): Promise<string> {
  const key = await getOrCreateKey();
  const iv = await Crypto.getRandomBytesAsync(16);
  const textoBytes = new TextEncoder().encode(texto);

  // Derivar keystream via HMAC-SHA256 em blocos
  const cifrado = new Uint8Array(textoBytes.length);
  for (let i = 0; i < textoBytes.length; i += 32) {
    const counter = new Uint8Array([...iv, ...intToBytes(i / 32)]);
    const hash = await Crypto.digest(
      Crypto.CryptoDigestAlgorithm.SHA256,
      counter,
    );
    const hashBytes = new Uint8Array(hexToBytes(hash));
    for (let j = 0; j < 32 && i + j < textoBytes.length; j++) {
      cifrado[i + j] = textoBytes[i + j] ^ hashBytes[j];
    }
  }

  return `${bytesToBase64(iv)}:${bytesToBase64(cifrado)}`;
}

export async function descriptografar(cifrado: string): Promise<string> {
  // Processo inverso — XOR é simétrico
  const [ivB64, dadosB64] = cifrado.split(':');
  const iv = base64ToBytes(ivB64);
  const dadosCifrados = base64ToBytes(dadosB64);

  const textoBytes = new Uint8Array(dadosCifrados.length);
  for (let i = 0; i < dadosCifrados.length; i += 32) {
    const counter = new Uint8Array([...iv, ...intToBytes(i / 32)]);
    const hash = await Crypto.digest(
      Crypto.CryptoDigestAlgorithm.SHA256,
      counter,
    );
    const hashBytes = new Uint8Array(hexToBytes(hash));
    for (let j = 0; j < 32 && i + j < dadosCifrados.length; j++) {
      textoBytes[i + j] = dadosCifrados[i + j] ^ hashBytes[j];
    }
  }

  return new TextDecoder().decode(textoBytes);
}

function bytesToBase64(bytes: Uint8Array): string {
  return btoa(String.fromCharCode(...bytes));
}
function base64ToBytes(b64: string): Uint8Array {
  return Uint8Array.from(atob(b64), (c) => c.charCodeAt(0));
}
function intToBytes(n: number): Uint8Array {
  return new Uint8Array([n >> 24, n >> 16, n >> 8, n].map((b) => b & 0xff));
}
function hexToBytes(hex: string): number[] {
  const bytes: number[] = [];
  for (let i = 0; i < hex.length; i += 2) {
    bytes.push(parseInt(hex.substr(i, 2), 16));
  }
  return bytes;
}
```

> **Nota:** para AES-256-CBC real em React Native, usar `react-native-quick-crypto` que expõe a API `crypto` completa do Node.js.

---

### React Web — Web Crypto API

```typescript
// src/core/crypto/cryptoService.ts
const KEY_STORAGE = 'crypto_key_jwk';

async function getOrCreateKey(): Promise<CryptoKey> {
  const stored = localStorage.getItem(KEY_STORAGE);
  if (stored) {
    const jwk = JSON.parse(stored);
    return crypto.subtle.importKey('jwk', jwk, { name: 'AES-GCM' }, true, ['encrypt', 'decrypt']);
  }

  const key = await crypto.subtle.generateKey(
    { name: 'AES-GCM', length: 256 },
    true,
    ['encrypt', 'decrypt'],
  );
  const jwk = await crypto.subtle.exportKey('jwk', key);
  localStorage.setItem(KEY_STORAGE, JSON.stringify(jwk));
  return key;
}

export async function criptografar(texto: string): Promise<string> {
  const key = await getOrCreateKey();
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const encoded = new TextEncoder().encode(texto);

  const cifrado = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    key,
    encoded,
  );

  const ivB64 = btoa(String.fromCharCode(...iv));
  const dadosB64 = btoa(String.fromCharCode(...new Uint8Array(cifrado)));
  return `${ivB64}:${dadosB64}`;
}

export async function descriptografar(cifrado: string): Promise<string> {
  const key = await getOrCreateKey();
  const [ivB64, dadosB64] = cifrado.split(':');
  const iv = Uint8Array.from(atob(ivB64), (c) => c.charCodeAt(0));
  const dados = Uint8Array.from(atob(dadosB64), (c) => c.charCodeAt(0));

  const decifrado = await crypto.subtle.decrypt(
    { name: 'AES-GCM', iv },
    key,
    dados,
  );

  return new TextDecoder().decode(decifrado);
}
```

---

### Equivalência — Criptografia

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| Algoritmo | AES-256-CBC (`encrypt`) | SHA-256 derivado / `quick-crypto` | AES-256-GCM (Web Crypto) |
| Geração de chave | `Random.secure` (32 bytes) | `Crypto.getRandomBytesAsync` | `crypto.subtle.generateKey` |
| Armazenamento da chave | `flutter_secure_storage` (Keychain/Keystore) | `expo-secure-store` | `localStorage` (JWK) |
| IV/Nonce | 16 bytes aleatórios | 16 bytes aleatórios | 12 bytes aleatórios |
| Formato cifrado | `iv_base64:dados_base64` | `iv_base64:dados_base64` | `iv_base64:dados_base64` |

> **Segurança da chave no Web:** `localStorage` não é tão seguro quanto Keychain/Keystore dos dispositivos nativos. Para dados altamente sensíveis no Web, considerar derivar a chave a partir de uma senha do usuário (`PBKDF2`) em vez de armazená-la.

---

## 12. Server-Sent Events (SSE)

Alternativa leve ao WebSocket para comunicação **unidirecional** (servidor → cliente). Ideal para feeds em tempo real, dashboards ao vivo e atualizações de status.

**Diferença vs WebSocket:** SSE é unidirecional (servidor envia), usa HTTP padrão, reconecta automaticamente, e é mais simples de implementar no backend. WebSocket é bidirecional.

### Formato do endpoint SSE (backend)

```
GET /api/events
Accept: text/event-stream

Resposta (stream contínuo):
data: {"tipo": "novo_pedido", "dados": {"id": 42, "valor": 89.90}}

data: {"tipo": "estoque_baixo", "dados": {"produto": "Widget X", "qtd": 3}}

event: heartbeat
data: ping
```

### Flutter — http stream

```dart
// lib/core/sse/sse_service.dart
import 'dart:async';
import 'dart:convert';
import 'package:http/http.dart' as http;

class SSEEvent {
  final String tipo;
  final Map<String, dynamic> dados;
  final DateTime timestamp;

  SSEEvent({required this.tipo, required this.dados})
      : timestamp = DateTime.now();
}

class SSEService {
  StreamSubscription? _sub;
  final _eventCtrl = StreamController<SSEEvent>.broadcast();

  Stream<SSEEvent> get eventos => _eventCtrl.stream;

  void conectar(String url, {Map<String, String>? headers}) {
    final client = http.Client();
    final request = http.Request('GET', Uri.parse(url));
    request.headers['Accept'] = 'text/event-stream';
    request.headers['Cache-Control'] = 'no-cache';
    if (headers != null) request.headers.addAll(headers);

    client.send(request).then((response) {
      _sub = response.stream
          .transform(utf8.decoder)
          .transform(const LineSplitter())
          .listen((linha) {
        if (linha.startsWith('data: ')) {
          final json = linha.substring(6);
          try {
            final map = jsonDecode(json) as Map<String, dynamic>;
            _eventCtrl.add(SSEEvent(
              tipo: map['tipo'] as String? ?? 'unknown',
              dados: map['dados'] as Map<String, dynamic>? ?? {},
            ));
          } catch (_) {}
        }
      });
    });
  }

  void desconectar() {
    _sub?.cancel();
    _eventCtrl.close();
  }
}
```

```dart
// lib/features/sse/presentation/sse_feed_screen.dart
import 'package:flutter/material.dart';
import '../../../core/sse/sse_service.dart';
import 'dart:async';

class SSEFeedScreen extends StatefulWidget {
  const SSEFeedScreen({super.key});

  @override
  State<SSEFeedScreen> createState() => _SSEFeedScreenState();
}

class _SSEFeedScreenState extends State<SSEFeedScreen> {
  final _sse = SSEService();
  final List<SSEEvent> _eventos = [];
  StreamSubscription? _sub;

  @override
  void initState() {
    super.initState();
    _sse.conectar('https://seu-backend.com/api/events');
    _sub = _sse.eventos.listen((evento) {
      if (mounted) setState(() => _eventos.insert(0, evento));
    });
  }

  @override
  void dispose() {
    _sub?.cancel();
    _sse.desconectar();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Feed em Tempo Real')),
      body: _eventos.isEmpty
          ? const Center(child: Text('Aguardando eventos…'))
          : ListView.builder(
              itemCount: _eventos.length,
              itemBuilder: (_, i) {
                final e = _eventos[i];
                return ListTile(
                  leading: Icon(
                    e.tipo == 'novo_pedido' ? Icons.shopping_cart :
                    e.tipo == 'estoque_baixo' ? Icons.warning : Icons.info,
                    color: e.tipo == 'estoque_baixo' ? Colors.orange : Colors.blue,
                  ),
                  title: Text(e.tipo),
                  subtitle: Text(e.dados.toString()),
                  trailing: Text(
                    '${e.timestamp.hour.toString().padLeft(2, '0')}:${e.timestamp.minute.toString().padLeft(2, '0')}:${e.timestamp.second.toString().padLeft(2, '0')}',
                    style: const TextStyle(fontSize: 11, color: Colors.grey),
                  ),
                );
              },
            ),
    );
  }
}
```

---

### React Native — EventSource polyfill

**1. Dependências**

```bash
npm install event-source-polyfill
```

**2. Hook SSE**

```typescript
// src/hooks/useSSE.ts
import { useEffect, useState, useRef } from 'react';
import { EventSourcePolyfill } from 'event-source-polyfill';

interface SSEEvent {
  tipo: string;
  dados: Record<string, unknown>;
  timestamp: Date;
}

export function useSSE(url: string) {
  const [eventos, setEventos] = useState<SSEEvent[]>([]);
  const [conectado, setConectado] = useState(false);
  const esRef = useRef<EventSourcePolyfill | null>(null);

  useEffect(() => {
    const es = new EventSourcePolyfill(url);
    esRef.current = es;

    es.onopen = () => setConectado(true);
    es.onerror = () => setConectado(false);

    es.onmessage = (e: MessageEvent) => {
      try {
        const parsed = JSON.parse(e.data);
        setEventos((prev) => [
          {
            tipo: parsed.tipo ?? 'unknown',
            dados: parsed.dados ?? {},
            timestamp: new Date(),
          },
          ...prev,
        ]);
      } catch {}
    };

    return () => es.close();
  }, [url]);

  return { eventos, conectado };
}
```

**3. Tela**

```tsx
// src/features/sse/SSEFeedScreen.tsx
import React from 'react';
import { View, Text, FlatList, StyleSheet } from 'react-native';
import { useSSE } from '../../hooks/useSSE';

export function SSEFeedScreen() {
  const { eventos, conectado } = useSSE('https://seu-backend.com/api/events');

  const fmt = (d: Date) =>
    `${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}:${String(d.getSeconds()).padStart(2, '0')}`;

  return (
    <View style={styles.container}>
      <View style={styles.status}>
        <View style={[styles.dot, { backgroundColor: conectado ? '#4caf50' : '#f44336' }]} />
        <Text style={styles.statusTxt}>{conectado ? 'Conectado' : 'Reconectando…'}</Text>
      </View>
      <FlatList
        data={eventos}
        keyExtractor={(_, i) => String(i)}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <Text style={styles.tipo}>{item.tipo}</Text>
            <Text style={styles.dados}>{JSON.stringify(item.dados)}</Text>
            <Text style={styles.hora}>{fmt(item.timestamp)}</Text>
          </View>
        )}
        ListEmptyComponent={<Text style={styles.vazio}>Aguardando eventos…</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f9f9f9' },
  status: { flexDirection: 'row', alignItems: 'center', gap: 8, padding: 12, backgroundColor: '#fff', borderBottomWidth: 1, borderBottomColor: '#e0e0e0' },
  dot: { width: 8, height: 8, borderRadius: 4 },
  statusTxt: { fontSize: 12, color: '#666' },
  item: { padding: 12, borderBottomWidth: 1, borderBottomColor: '#f0f0f0' },
  tipo: { fontWeight: '600' },
  dados: { fontSize: 13, color: '#666', marginTop: 2 },
  hora: { fontSize: 11, color: '#999', marginTop: 4 },
  vazio: { textAlign: 'center', color: '#999', marginTop: 40 },
});
```

---

### React Web — EventSource API + Bootstrap

```tsx
// src/features/sse/SSEFeedPage.tsx
import { useEffect, useState, useRef } from 'react';

interface SSEEvent {
  tipo: string;
  dados: Record<string, unknown>;
  timestamp: Date;
}

export function SSEFeedPage() {
  const [eventos, setEventos] = useState<SSEEvent[]>([]);
  const [conectado, setConectado] = useState(false);

  useEffect(() => {
    const es = new EventSource('https://seu-backend.com/api/events');

    es.onopen = () => setConectado(true);
    es.onerror = () => setConectado(false);

    es.onmessage = (e) => {
      try {
        const parsed = JSON.parse(e.data);
        setEventos((prev) => [
          { tipo: parsed.tipo ?? 'unknown', dados: parsed.dados ?? {}, timestamp: new Date() },
          ...prev,
        ]);
      } catch {}
    };

    return () => es.close();
  }, []);

  const fmt = (d: Date) =>
    `${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}:${String(d.getSeconds()).padStart(2, '0')}`;

  const icone = (tipo: string) =>
    tipo === 'novo_pedido' ? 'fa-shopping-cart text-primary' :
    tipo === 'estoque_baixo' ? 'fa-exclamation-triangle text-warning' :
    'fa-info-circle text-secondary';

  return (
    <div className="container py-4" style={{ maxWidth: 700 }}>
      <div className="d-flex justify-content-between align-items-center mb-4">
        <h4 className="mb-0"><i className="fas fa-rss me-2" />Feed em Tempo Real</h4>
        <span className="d-flex align-items-center gap-2">
          <i className="fas fa-circle" style={{ fontSize: 8, color: conectado ? '#4caf50' : '#f44336' }} />
          <small className="text-muted">{conectado ? 'Conectado' : 'Reconectando…'}</small>
        </span>
      </div>

      {eventos.length === 0 ? (
        <p className="text-muted text-center mt-5">Aguardando eventos…</p>
      ) : (
        <div className="list-group">
          {eventos.map((e, i) => (
            <div key={i} className="list-group-item">
              <div className="d-flex justify-content-between align-items-start">
                <div className="d-flex align-items-center gap-2">
                  <i className={`fas ${icone(e.tipo)}`} />
                  <strong>{e.tipo}</strong>
                </div>
                <small className="text-muted">{fmt(e.timestamp)}</small>
              </div>
              <small className="text-muted d-block mt-1">{JSON.stringify(e.dados)}</small>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

---

### Equivalência — Server-Sent Events

| Funcionalidade | Flutter | React Native | React Web |
|---|---|---|---|
| API | `http` stream + parser manual | `event-source-polyfill` | `EventSource` (nativo) |
| Reconexão automática | Manual (implementar) | Built-in (polyfill) | Built-in (EventSource) |
| Protocolo | HTTP/1.1 `text/event-stream` | HTTP/1.1 `text/event-stream` | HTTP/1.1 `text/event-stream` |
| Headers customizados | Sim (`http.Request.headers`) | Sim (`EventSourcePolyfill`) | Não (EventSource nativo) |
| Eventos nomeados | Parser manual | `es.addEventListener(name)` | `es.addEventListener(name)` |
| Comparação com WebSocket | Unidirecional, mais simples | Unidirecional, mais simples | Unidirecional, mais simples |

### Quando usar SSE vs WebSocket

| Critério | SSE | WebSocket |
|---|---|---|
| Direção | Servidor → Cliente | Bidirecional |
| Protocolo | HTTP padrão | Protocolo próprio (ws://) |
| Reconexão | Automática | Manual |
| Proxies/firewalls | Funciona com HTTP | Pode ser bloqueado |
| Formato | Texto (event-stream) | Texto ou binário |
| Caso de uso | Feeds, dashboards, notificações | Chat, jogos, colaboração em tempo real |

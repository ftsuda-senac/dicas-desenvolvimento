# WebSocket com Spring Boot

> **Baseline:** Spring Boot 4.x · Spring Framework 7.x · Java 25+ · Jakarta EE 11
>
> **Pré-requisitos:** familiaridade com Spring MVC, Spring Security e JWT. Consulte [Dicas-Spring-MVC-REST.md](Dicas-Spring-MVC-REST.md) e [Dicas-Spring-Security.md](Dicas-Spring-Security.md).

---

Este documento reúne informações e exemplos práticos sobre comunicação em tempo real com WebSocket em aplicações Spring Boot:

- conceitos e diferenças entre WebSocket puro e STOMP;
- configuração com e sem STOMP, incluindo interceptors e handlers;
- autenticação e autorização via JWT com reconhecimento de papéis (`COMUM`, `MODERADOR`, `ADMIN`);
- múltiplos chats simultâneos com salas dinâmicas;
- envio e compartilhamento de arquivos via WebSocket;
- interface web com HTML e JavaScript puro (sem frameworks frontend);
- heartbeat e detecção de conexões mortas;
- tratamento de erros com `@MessageExceptionHandler` e `StompSubProtocolErrorHandler`;
- rate limiting por usuário para evitar flood de mensagens;
- histórico de mensagens com carregamento ao entrar na sala;
- indicadores de digitação e presença online;
- testes de integração com `WebSocketStompClient`.

---

## Sumário

1. [Conceitos fundamentais](#1-conceitos-fundamentais)
2. [WebSocket sem STOMP — API nativa](#2-websocket-sem-stomp--api-nativa)
3. [WebSocket com STOMP](#3-websocket-com-stomp)
4. [Autenticação e autorização com JWT](#4-autenticação-e-autorização-com-jwt)
5. [Múltiplos chats simultâneos](#5-múltiplos-chats-simultâneos)
6. [Envio e compartilhamento de arquivos](#6-envio-e-compartilhamento-de-arquivos)
7. [Interface web com HTML e JavaScript](#7-interface-web-com-html-e-javascript)
8. [Heartbeat e detecção de conexões mortas](#8-heartbeat-e-detecção-de-conexões-mortas)
9. [Tratamento de erros](#9-tratamento-de-erros)
10. [Rate limiting por usuário](#10-rate-limiting-por-usuário)
11. [Histórico de mensagens](#11-histórico-de-mensagens)
12. [Indicadores de digitação e presença](#12-indicadores-de-digitação-e-presença)
13. [Testes de integração](#13-testes-de-integração)
14. [Boas práticas](#14-boas-práticas)
15. [Anexo — Evolução para web conferência com WebRTC](#15-anexo--evolução-para-web-conferência-com-webrtc)
16. [Referências](#16-referências)

---

## 1. Conceitos fundamentais

### 1.1. O que é WebSocket

WebSocket é um protocolo de comunicação bidirecional e full-duplex sobre uma única conexão TCP. Diferente do HTTP tradicional, onde o cliente sempre inicia a comunicação, o WebSocket permite que o servidor envie mensagens ao cliente a qualquer momento após o handshake inicial.

Fluxo simplificado:

```text
Cliente                           Servidor
   |                                  |
   |--- HTTP GET /ws (Upgrade) ------>|
   |<-- 101 Switching Protocols ------|
   |                                  |
   |<========= WebSocket ==========> |
   |   (bidirecional, full-duplex)    |
   |                                  |
```

Casos de uso comuns:

- chat em tempo real;
- notificações push;
- dashboards com dados ao vivo;
- jogos multiplayer;
- edição colaborativa;
- streaming de dados financeiros ou IoT.

### 1.2. WebSocket puro vs STOMP

O Spring Boot oferece duas abordagens para trabalhar com WebSocket:

| Aspecto | WebSocket puro | STOMP sobre WebSocket |
| --- | --- | --- |
| Protocolo | WebSocket nativo (RFC 6455) | Sub-protocolo de mensageria sobre WebSocket |
| Modelo | handler de baixo nível, mensagens livres | pub-sub com destinations (`/topic`, `/queue`) |
| Roteamento | manual, dentro do handler | automático via `@MessageMapping` |
| Broadcast | manual, iterando sessões | automático via `@SendTo` ou `SimpMessagingTemplate` |
| Escalabilidade | implementação manual | suporte a broker externo (RabbitMQ, ActiveMQ) |
| Complexidade | menor para casos simples | maior inicialmente, mais poderoso para chat e pub-sub |
| Fallback (SockJS) | não disponível nativamente | suporte embutido |
| Quando usar | notificações simples, binário puro, controle total | chat, múltiplas salas, cenários pub-sub |

### 1.3. Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

Para autenticação JWT com Nimbus JOSE+JWT:

```xml
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
    <version>10.3</version>
</dependency>
```

Para o frontend com SockJS e STOMP (quando não se usa CDN):

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>sockjs-client</artifactId>
    <version>1.5.1</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>stomp-websocket</artifactId>
    <version>2.3.4</version>
</dependency>
```

### 1.4. Papéis de usuário usados nos exemplos

Todos os exemplos deste documento usam os seguintes papéis para controle de acesso:

```java
package br.com.exemplo.websocket.auth;

public enum UserRole {
    COMUM,
    MODERADOR,
    ADMIN
}
```

O papel é declarado no JWT como uma claim `role` e extraído durante a autenticação WebSocket.

---

## 2. WebSocket sem STOMP — API nativa

### 2.1. Handler básico

A abordagem sem STOMP usa `TextWebSocketHandler` (ou `BinaryWebSocketHandler`) para tratar mensagens diretamente.

```java
package br.com.exemplo.websocket.raw;

import java.io.IOException;
import java.util.Set;
import java.util.concurrent.CopyOnWriteArraySet;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

public class ChatRawHandler extends TextWebSocketHandler {

    private final Set<WebSocketSession> sessions = new CopyOnWriteArraySet<>();

    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        sessions.add(session);
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws IOException {
        String payload = session.getId() + ": " + message.getPayload();
        for (WebSocketSession s : sessions) {
            if (s.isOpen()) {
                s.sendMessage(new TextMessage(payload));
            }
        }
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        sessions.remove(session);
    }
}
```

### 2.2. Configuração

```java
package br.com.exemplo.websocket.raw;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class RawWebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(chatRawHandler(), "/ws/chat-raw")
                .setAllowedOrigins("*");
    }

    @Bean
    public ChatRawHandler chatRawHandler() {
        return new ChatRawHandler();
    }
}
```

### 2.3. Interceptor para autenticação JWT no handshake

Na abordagem sem STOMP, a autenticação acontece durante o handshake HTTP, antes da conexão WebSocket ser estabelecida:

```java
package br.com.exemplo.websocket.raw;

import br.com.exemplo.websocket.auth.JwtTokenProvider;
import br.com.exemplo.websocket.auth.UserRole;
import java.util.Map;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.HandshakeInterceptor;

public class JwtHandshakeInterceptor implements HandshakeInterceptor {

    private final JwtTokenProvider jwtTokenProvider;

    public JwtHandshakeInterceptor(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    public boolean beforeHandshake(
            ServerHttpRequest request,
            ServerHttpResponse response,
            WebSocketHandler wsHandler,
            Map<String, Object> attributes
    ) {
        if (request instanceof ServletServerHttpRequest servletRequest) {
            String token = servletRequest.getServletRequest().getParameter("token");
            if (token != null && jwtTokenProvider.isValid(token)) {
                attributes.put("username", jwtTokenProvider.getUsername(token));
                attributes.put("role", jwtTokenProvider.getRole(token));
                return true;
            }
        }
        return false;
    }

    @Override
    public void afterHandshake(
            ServerHttpRequest request,
            ServerHttpResponse response,
            WebSocketHandler wsHandler,
            Exception exception
    ) {
    }
}
```

Registro do interceptor:

```java
@Override
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    registry.addHandler(chatRawHandler(), "/ws/chat-raw")
            .addInterceptors(new JwtHandshakeInterceptor(jwtTokenProvider))
            .setAllowedOrigins("*");
}
```

### 2.4. Acessando dados do usuário no handler

Após o handshake, os atributos ficam disponíveis na sessão:

```java
package br.com.exemplo.websocket.raw;

import br.com.exemplo.websocket.auth.UserRole;
import java.io.IOException;
import java.util.Set;
import java.util.concurrent.CopyOnWriteArraySet;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

public class ChatRawAuthHandler extends TextWebSocketHandler {

    private final Set<WebSocketSession> sessions = new CopyOnWriteArraySet<>();

    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        sessions.add(session);
        String username = (String) session.getAttributes().get("username");
        broadcast("Sistema: " + username + " entrou no chat");
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws IOException {
        String username = (String) session.getAttributes().get("username");
        UserRole role = (UserRole) session.getAttributes().get("role");

        String payload = message.getPayload();

        if (payload.startsWith("/kick") && (role == UserRole.MODERADOR || role == UserRole.ADMIN)) {
            String targetUser = payload.substring(6).trim();
            kickUser(targetUser);
            return;
        }

        String prefix = "[" + role + "] " + username;
        broadcast(prefix + ": " + payload);
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        sessions.remove(session);
        String username = (String) session.getAttributes().get("username");
        broadcast("Sistema: " + username + " saiu do chat");
    }

    private void broadcast(String text) {
        TextMessage msg = new TextMessage(text);
        for (WebSocketSession s : sessions) {
            if (s.isOpen()) {
                try {
                    s.sendMessage(msg);
                } catch (IOException ignored) {
                }
            }
        }
    }

    private void kickUser(String targetUser) {
        for (WebSocketSession s : sessions) {
            if (targetUser.equals(s.getAttributes().get("username"))) {
                try {
                    s.sendMessage(new TextMessage("Sistema: você foi removido pelo moderador"));
                    s.close(CloseStatus.POLICY_VIOLATION);
                } catch (IOException ignored) {
                }
            }
        }
    }
}
```

### 2.5. Cliente JavaScript para WebSocket puro

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Chat — WebSocket Puro</title>
</head>
<body>
    <div id="chat-log" style="height:300px;overflow-y:auto;border:1px solid #ccc;padding:8px;margin-bottom:8px;"></div>
    <input type="text" id="msg-input" placeholder="Digite sua mensagem..." style="width:80%;">
    <button id="btn-send">Enviar</button>

    <script>
        const token = prompt("Informe seu JWT:");
        const ws = new WebSocket("ws://localhost:8080/ws/chat-raw?token=" + encodeURIComponent(token));
        const chatLog = document.getElementById("chat-log");

        ws.onopen = function () {
            appendLog("Conectado ao chat.");
        };

        ws.onmessage = function (event) {
            appendLog(event.data);
        };

        ws.onclose = function () {
            appendLog("Conexão encerrada.");
        };

        document.getElementById("btn-send").addEventListener("click", function () {
            const input = document.getElementById("msg-input");
            if (input.value.trim() !== "") {
                ws.send(input.value);
                input.value = "";
            }
        });

        document.getElementById("msg-input").addEventListener("keydown", function (e) {
            if (e.key === "Enter") {
                document.getElementById("btn-send").click();
            }
        });

        function appendLog(text) {
            const div = document.createElement("div");
            div.textContent = text;
            chatLog.appendChild(div);
            chatLog.scrollTop = chatLog.scrollHeight;
        }
    </script>
</body>
</html>
```

---

## 3. WebSocket com STOMP

### 3.1. Configuração do message broker

STOMP (Simple Text Oriented Messaging Protocol) adiciona uma camada de mensageria sobre o WebSocket, com conceitos de destinations, subscriptions e message routing.

```java
package br.com.exemplo.websocket.stomp;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class StompWebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/queue");
        registry.setApplicationDestinationPrefixes("/app");
        registry.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws/stomp")
                .setAllowedOrigins("*")
                .withSockJS();
    }
}
```

Conceitos-chave:

- `/topic` — destinations para broadcast (todos os inscritos recebem);
- `/queue` — destinations para mensagens ponto a ponto;
- `/app` — prefixo para mensagens enviadas ao servidor via `@MessageMapping`;
- `/user` — prefixo para mensagens direcionadas a um usuário específico;
- `withSockJS()` — habilita fallback para navegadores sem suporte a WebSocket.

### 3.2. DTOs de mensagem

```java
package br.com.exemplo.websocket.stomp;

public record ChatMessageRequest(
        String content,
        String roomId
) {
}
```

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;

public record ChatMessageResponse(
        String sender,
        String role,
        String content,
        String roomId,
        Instant timestamp,
        MessageType type
) {
}
```

```java
package br.com.exemplo.websocket.stomp;

public enum MessageType {
    CHAT,
    JOIN,
    LEAVE,
    SYSTEM,
    FILE
}
```

### 3.3. Controller com `@MessageMapping`

```java
package br.com.exemplo.websocket.stomp;

import java.security.Principal;
import java.time.Instant;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.stereotype.Controller;

@Controller
public class ChatController {

    @MessageMapping("/chat/{roomId}")
    @SendTo("/topic/chat/{roomId}")
    public ChatMessageResponse sendMessage(
            @DestinationVariable String roomId,
            ChatMessageRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        String role = (String) headerAccessor.getSessionAttributes().get("role");

        return new ChatMessageResponse(
                username,
                role,
                request.content(),
                roomId,
                Instant.now(),
                MessageType.CHAT
        );
    }
}
```

### 3.4. Enviando mensagens programaticamente com `SimpMessagingTemplate`

Para enviar mensagens de qualquer ponto da aplicação, como services, listeners ou jobs:

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

@Service
public class ChatNotificationService {

    private final SimpMessagingTemplate messagingTemplate;

    public ChatNotificationService(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    public void enviarParaSala(String roomId, String mensagem) {
        ChatMessageResponse response = new ChatMessageResponse(
                "Sistema", "SYSTEM", mensagem, roomId, Instant.now(), MessageType.SYSTEM
        );
        messagingTemplate.convertAndSend("/topic/chat/" + roomId, response);
    }

    public void enviarParaUsuario(String username, String mensagem) {
        ChatMessageResponse response = new ChatMessageResponse(
                "Sistema", "SYSTEM", mensagem, null, Instant.now(), MessageType.SYSTEM
        );
        messagingTemplate.convertAndSendToUser(username, "/queue/private", response);
    }
}
```

### 3.5. Detectando conexão e desconexão

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;
import org.springframework.context.event.EventListener;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.messaging.SessionConnectedEvent;
import org.springframework.web.socket.messaging.SessionDisconnectEvent;

@Component
public class WebSocketEventListener {

    private final SimpMessagingTemplate messagingTemplate;

    public WebSocketEventListener(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    @EventListener
    public void onConnect(SessionConnectedEvent event) {
        SimpMessageHeaderAccessor headers = SimpMessageHeaderAccessor.wrap(event.getMessage());
        String username = headers.getUser() != null ? headers.getUser().getName() : "anônimo";
        System.out.println("Usuário conectado: " + username);
    }

    @EventListener
    public void onDisconnect(SessionDisconnectEvent event) {
        SimpMessageHeaderAccessor headers = SimpMessageHeaderAccessor.wrap(event.getMessage());
        String username = headers.getUser() != null ? headers.getUser().getName() : "anônimo";
        String roomId = (String) headers.getSessionAttributes().get("roomId");

        if (roomId != null) {
            ChatMessageResponse response = new ChatMessageResponse(
                    username, "SYSTEM", username + " saiu do chat",
                    roomId, Instant.now(), MessageType.LEAVE
            );
            messagingTemplate.convertAndSend("/topic/chat/" + roomId, response);
        }
    }
}
```

---

## 4. Autenticação e autorização com JWT

### 4.1. Estrutura do JWT

Os exemplos deste documento assumem um JWT com a seguinte estrutura de payload:

```json
{
  "sub": "joao.silva",
  "role": "MODERADOR",
  "iat": 1750000000,
  "exp": 1750003600
}
```

O campo `role` pode ser `COMUM`, `MODERADOR` ou `ADMIN`.

### 4.2. Provider para validação do token

```java
package br.com.exemplo.websocket.auth;

import com.nimbusds.jose.JWSAlgorithm;
import com.nimbusds.jose.JWSVerifier;
import com.nimbusds.jose.crypto.MACVerifier;
import com.nimbusds.jwt.JWTClaimsSet;
import com.nimbusds.jwt.SignedJWT;
import java.util.Date;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class JwtTokenProvider {

    private final JWSVerifier verifier;

    public JwtTokenProvider(@Value("${jwt.secret}") String secret) {
        try {
            this.verifier = new MACVerifier(secret.getBytes());
        } catch (Exception ex) {
            throw new IllegalStateException("Falha ao criar verificador JWT", ex);
        }
    }

    public boolean isValid(String token) {
        try {
            SignedJWT signedJWT = SignedJWT.parse(token);
            if (!signedJWT.verify(verifier)) {
                return false;
            }
            if (!JWSAlgorithm.HS256.equals(signedJWT.getHeader().getAlgorithm())) {
                return false;
            }
            Date expiration = signedJWT.getJWTClaimsSet().getExpirationTime();
            return expiration != null && expiration.after(new Date());
        } catch (Exception ex) {
            return false;
        }
    }

    public String getUsername(String token) {
        return getClaims(token).getSubject();
    }

    public UserRole getRole(String token) {
        String role = (String) getClaims(token).getClaim("role");
        return UserRole.valueOf(role);
    }

    private JWTClaimsSet getClaims(String token) {
        try {
            return SignedJWT.parse(token).getJWTClaimsSet();
        } catch (Exception ex) {
            throw new IllegalArgumentException("Token JWT inválido", ex);
        }
    }
}
```

Notas sobre a implementação com Nimbus JOSE+JWT:

- `MACVerifier` valida assinaturas HMAC (HS256, HS384, HS512); para RSA ou EC, use `RSASSAVerifier` ou `ECDSAVerifier`;
- `SignedJWT.parse()` faz o parsing do token compacto;
- `verify(verifier)` valida a assinatura criptográfica;
- `getJWTClaimsSet()` retorna as claims (subject, expiration, claims customizadas);
- a verificação do algoritmo no header evita ataques de troca de algoritmo.

### 4.3. Autenticação STOMP via `ChannelInterceptor`

Na abordagem com STOMP, a autenticação acontece na mensagem `CONNECT`, não no handshake HTTP. Isso é feito com um `ChannelInterceptor` no canal de entrada:

```java
package br.com.exemplo.websocket.auth;

import java.util.List;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.ChannelInterceptor;
import org.springframework.messaging.support.MessageHeaderAccessor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.authority.SimpleGrantedAuthority;

public class JwtStompInterceptor implements ChannelInterceptor {

    private final JwtTokenProvider jwtTokenProvider;

    public JwtStompInterceptor(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);

        if (accessor != null && StompCommand.CONNECT.equals(accessor.getCommand())) {
            String token = accessor.getFirstNativeHeader("Authorization");
            if (token != null && token.startsWith("Bearer ")) {
                token = token.substring(7);
            }

            if (token == null || !jwtTokenProvider.isValid(token)) {
                throw new IllegalArgumentException("Token JWT inválido ou ausente");
            }

            String username = jwtTokenProvider.getUsername(token);
            UserRole role = jwtTokenProvider.getRole(token);

            accessor.getSessionAttributes().put("role", role.name());

            UsernamePasswordAuthenticationToken auth = new UsernamePasswordAuthenticationToken(
                    username,
                    null,
                    List.of(new SimpleGrantedAuthority("ROLE_" + role.name()))
            );
            accessor.setUser(auth);
        }

        return message;
    }
}
```

### 4.4. Registro do interceptor na configuração STOMP

```java
package br.com.exemplo.websocket.stomp;

import br.com.exemplo.websocket.auth.JwtStompInterceptor;
import br.com.exemplo.websocket.auth.JwtTokenProvider;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.messaging.simp.config.ChannelRegistration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
@Order(Ordered.HIGHEST_PRECEDENCE + 99)
public class StompWebSocketSecurityConfig implements WebSocketMessageBrokerConfigurer {

    private final JwtTokenProvider jwtTokenProvider;

    public StompWebSocketSecurityConfig(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/queue");
        registry.setApplicationDestinationPrefixes("/app");
        registry.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws/stomp")
                .setAllowedOrigins("*")
                .withSockJS();
    }

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new JwtStompInterceptor(jwtTokenProvider));
    }
}
```

### 4.5. Controle de acesso por papel nas mensagens

Com o interceptor configurado, o papel do usuário fica disponível em qualquer `@MessageMapping`:

```java
package br.com.exemplo.websocket.stomp;

import br.com.exemplo.websocket.auth.UserRole;
import java.time.Instant;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class ModerationController {

    private final SimpMessagingTemplate messagingTemplate;
    private final ActiveSessionRegistry sessionRegistry;

    public ModerationController(
            SimpMessagingTemplate messagingTemplate,
            ActiveSessionRegistry sessionRegistry
    ) {
        this.messagingTemplate = messagingTemplate;
        this.sessionRegistry = sessionRegistry;
    }

    @MessageMapping("/moderate/{roomId}/mute")
    public void muteUser(
            @DestinationVariable String roomId,
            MuteRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        UserRole role = UserRole.valueOf(
                (String) headerAccessor.getSessionAttributes().get("role")
        );

        if (role != UserRole.MODERADOR && role != UserRole.ADMIN) {
            messagingTemplate.convertAndSendToUser(
                    headerAccessor.getUser().getName(),
                    "/queue/errors",
                    "Permissão negada: apenas moderadores podem silenciar usuários"
            );
            return;
        }

        ChatMessageResponse response = new ChatMessageResponse(
                "Sistema", "SYSTEM",
                request.targetUser() + " foi silenciado por " + headerAccessor.getUser().getName(),
                roomId, Instant.now(), MessageType.SYSTEM
        );
        messagingTemplate.convertAndSend("/topic/chat/" + roomId, response);
    }

    @MessageMapping("/admin/broadcast")
    public void adminBroadcast(
            AdminBroadcastRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        UserRole role = UserRole.valueOf(
                (String) headerAccessor.getSessionAttributes().get("role")
        );

        if (role != UserRole.ADMIN) {
            return;
        }

        ChatMessageResponse response = new ChatMessageResponse(
                "Admin", "ADMIN", request.message(),
                null, Instant.now(), MessageType.SYSTEM
        );

        for (String roomId : sessionRegistry.getActiveRoomIds()) {
            messagingTemplate.convertAndSend("/topic/chat/" + roomId, response);
        }
    }
}
```

```java
package br.com.exemplo.websocket.stomp;

public record MuteRequest(String targetUser) {
}
```

```java
package br.com.exemplo.websocket.stomp;

public record AdminBroadcastRequest(String message) {
}
```

### 4.6. Configuração do Spring Security para WebSocket

O Spring Security pode proteger o endpoint de handshake HTTP e as destinations STOMP:

```java
package br.com.exemplo.websocket.auth;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class WebSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/ws/**").permitAll()
                        .requestMatchers("/index.html", "/chat.html", "/js/**", "/css/**").permitAll()
                        .anyRequest().authenticated()
                );
        return http.build();
    }
}
```

Para proteger destinations STOMP com autorização baseada em authorities Spring Security:

```java
package br.com.exemplo.websocket.auth;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.Message;
import org.springframework.security.authorization.AuthorizationManager;
import org.springframework.security.config.annotation.web.socket.EnableWebSocketSecurity;
import org.springframework.security.messaging.access.intercept.MessageMatcherDelegatingAuthorizationManager;

@Configuration
@EnableWebSocketSecurity
public class WebSocketSecurityConfig {

    @Bean
    public AuthorizationManager<Message<?>> messageAuthorizationManager(
            MessageMatcherDelegatingAuthorizationManager.Builder messages
    ) {
        messages
                .simpDestMatchers("/app/admin/**").hasRole("ADMIN")
                .simpDestMatchers("/app/moderate/**").hasAnyRole("MODERADOR", "ADMIN")
                .simpDestMatchers("/app/chat/**").authenticated()
                .simpSubscribeDestMatchers("/topic/**", "/queue/**").authenticated()
                .anyMessage().authenticated();
        return messages.build();
    }
}
```

---

## 5. Múltiplos chats simultâneos

### 5.1. Registro de salas e sessões ativas

Para gerenciar múltiplas salas de chat simultaneamente, é necessário manter o mapeamento de quais usuários estão em quais salas:

```java
package br.com.exemplo.websocket.stomp;

import java.util.Collections;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import org.springframework.stereotype.Component;

@Component
public class ActiveSessionRegistry {

    private final Map<String, Set<String>> roomUsers = new ConcurrentHashMap<>();
    private final Map<String, Set<String>> userRooms = new ConcurrentHashMap<>();

    public void joinRoom(String roomId, String username) {
        roomUsers.computeIfAbsent(roomId, k -> ConcurrentHashMap.newKeySet()).add(username);
        userRooms.computeIfAbsent(username, k -> ConcurrentHashMap.newKeySet()).add(roomId);
    }

    public void leaveRoom(String roomId, String username) {
        Set<String> users = roomUsers.get(roomId);
        if (users != null) {
            users.remove(username);
            if (users.isEmpty()) {
                roomUsers.remove(roomId);
            }
        }
        Set<String> rooms = userRooms.get(roomId);
        if (rooms != null) {
            rooms.remove(roomId);
        }
    }

    public void disconnectUser(String username) {
        Set<String> rooms = userRooms.remove(username);
        if (rooms != null) {
            for (String roomId : rooms) {
                Set<String> users = roomUsers.get(roomId);
                if (users != null) {
                    users.remove(username);
                    if (users.isEmpty()) {
                        roomUsers.remove(roomId);
                    }
                }
            }
        }
    }

    public Set<String> getUsersInRoom(String roomId) {
        return roomUsers.getOrDefault(roomId, Collections.emptySet());
    }

    public Set<String> getRoomsForUser(String username) {
        return userRooms.getOrDefault(username, Collections.emptySet());
    }

    public Set<String> getActiveRoomIds() {
        return roomUsers.keySet();
    }
}
```

### 5.2. Controller com suporte a múltiplas salas

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;
import java.util.Set;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class MultiRoomChatController {

    private final ActiveSessionRegistry sessionRegistry;
    private final SimpMessagingTemplate messagingTemplate;

    public MultiRoomChatController(
            ActiveSessionRegistry sessionRegistry,
            SimpMessagingTemplate messagingTemplate
    ) {
        this.sessionRegistry = sessionRegistry;
        this.messagingTemplate = messagingTemplate;
    }

    @MessageMapping("/chat/{roomId}/join")
    public void joinRoom(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        String role = (String) headerAccessor.getSessionAttributes().get("role");

        sessionRegistry.joinRoom(roomId, username);
        headerAccessor.getSessionAttributes().put("roomId", roomId);

        ChatMessageResponse joinMsg = new ChatMessageResponse(
                username, role, username + " entrou na sala",
                roomId, Instant.now(), MessageType.JOIN
        );
        messagingTemplate.convertAndSend("/topic/chat/" + roomId, joinMsg);

        Set<String> onlineUsers = sessionRegistry.getUsersInRoom(roomId);
        messagingTemplate.convertAndSendToUser(
                username, "/queue/room-users",
                new RoomUsersResponse(roomId, onlineUsers)
        );
    }

    @MessageMapping("/chat/{roomId}/leave")
    public void leaveRoom(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        sessionRegistry.leaveRoom(roomId, username);

        ChatMessageResponse leaveMsg = new ChatMessageResponse(
                username, "SYSTEM", username + " saiu da sala",
                roomId, Instant.now(), MessageType.LEAVE
        );
        messagingTemplate.convertAndSend("/topic/chat/" + roomId, leaveMsg);
    }

    @MessageMapping("/chat/{roomId}")
    @SendTo("/topic/chat/{roomId}")
    public ChatMessageResponse sendMessage(
            @DestinationVariable String roomId,
            ChatMessageRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        String role = (String) headerAccessor.getSessionAttributes().get("role");

        return new ChatMessageResponse(
                username, role, request.content(),
                roomId, Instant.now(), MessageType.CHAT
        );
    }
}
```

```java
package br.com.exemplo.websocket.stomp;

import java.util.Set;

public record RoomUsersResponse(String roomId, Set<String> users) {
}
```

### 5.3. Mensagens privadas entre usuários

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class PrivateMessageController {

    private final SimpMessagingTemplate messagingTemplate;

    public PrivateMessageController(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    @MessageMapping("/private")
    public void sendPrivateMessage(
            PrivateMessageRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String sender = headerAccessor.getUser().getName();
        String role = (String) headerAccessor.getSessionAttributes().get("role");

        ChatMessageResponse response = new ChatMessageResponse(
                sender, role, request.content(),
                null, Instant.now(), MessageType.CHAT
        );

        messagingTemplate.convertAndSendToUser(request.targetUser(), "/queue/private", response);
        messagingTemplate.convertAndSendToUser(sender, "/queue/private", response);
    }
}
```

```java
package br.com.exemplo.websocket.stomp;

public record PrivateMessageRequest(String targetUser, String content) {
}
```

O `convertAndSendToUser` resolve o destination para `/user/{username}/queue/private`, que é roteado pela infraestrutura do Spring para a sessão correta do usuário.

### 5.4. Limpeza de sessões na desconexão

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;
import java.util.Set;
import org.springframework.context.event.EventListener;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.messaging.SessionDisconnectEvent;

@Component
public class DisconnectCleanupListener {

    private final ActiveSessionRegistry sessionRegistry;
    private final SimpMessagingTemplate messagingTemplate;

    public DisconnectCleanupListener(
            ActiveSessionRegistry sessionRegistry,
            SimpMessagingTemplate messagingTemplate
    ) {
        this.sessionRegistry = sessionRegistry;
        this.messagingTemplate = messagingTemplate;
    }

    @EventListener
    public void onDisconnect(SessionDisconnectEvent event) {
        SimpMessageHeaderAccessor headers = SimpMessageHeaderAccessor.wrap(event.getMessage());
        if (headers.getUser() == null) {
            return;
        }

        String username = headers.getUser().getName();
        Set<String> rooms = sessionRegistry.getRoomsForUser(username);

        for (String roomId : rooms) {
            ChatMessageResponse response = new ChatMessageResponse(
                    username, "SYSTEM", username + " desconectou",
                    roomId, Instant.now(), MessageType.LEAVE
            );
            messagingTemplate.convertAndSend("/topic/chat/" + roomId, response);
        }

        sessionRegistry.disconnectUser(username);
    }
}
```

---

## 6. Envio e compartilhamento de arquivos

### 6.1. Estratégias para envio de arquivos

Existem duas abordagens principais para compartilhar arquivos em aplicações WebSocket:

| Estratégia | Como funciona | Quando usar |
| --- | --- | --- |
| Upload via HTTP + notificação via WebSocket | arquivo sobe por `POST /api/files`, WebSocket notifica a sala com o link | arquivos grandes, controle de progresso, validações de segurança |
| Envio direto via WebSocket (Base64) | arquivo é codificado em Base64 e enviado como mensagem | arquivos pequenos (< 1 MB), simplicidade |

A abordagem recomendada para produção é **upload via HTTP + notificação via WebSocket**, pois oferece:

- validação de tipo e tamanho no servidor;
- progresso de upload no cliente;
- melhor uso de memória;
- possibilidade de armazenamento em disco ou object storage.

### 6.2. Upload HTTP com notificação via WebSocket

Controller REST para upload:

```java
package br.com.exemplo.websocket.file;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.UUID;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/files")
public class FileUploadController {

    private static final long MAX_FILE_SIZE = 10 * 1024 * 1024;
    private static final java.util.Set<String> ALLOWED_TYPES = java.util.Set.of(
            "image/png", "image/jpeg", "image/gif",
            "application/pdf",
            "text/plain"
    );

    private final Path uploadDir;
    private final FileShareNotifier fileShareNotifier;

    public FileUploadController(
            @Value("${app.upload.dir:uploads}") String uploadDirPath,
            FileShareNotifier fileShareNotifier
    ) throws IOException {
        this.uploadDir = Path.of(uploadDirPath);
        this.fileShareNotifier = fileShareNotifier;
        Files.createDirectories(uploadDir);
    }

    @PostMapping
    public ResponseEntity<FileUploadResponse> upload(
            @RequestParam MultipartFile file,
            @RequestParam String roomId,
            @RequestParam String username
    ) throws IOException {
        if (file.isEmpty()) {
            return ResponseEntity.badRequest().build();
        }

        if (file.getSize() > MAX_FILE_SIZE) {
            return ResponseEntity.badRequest().build();
        }

        String contentType = file.getContentType();
        if (contentType == null || !ALLOWED_TYPES.contains(contentType)) {
            return ResponseEntity.badRequest().build();
        }

        String storedName = UUID.randomUUID() + "_" + file.getOriginalFilename();
        Path target = uploadDir.resolve(storedName);
        file.transferTo(target);

        String downloadUrl = "/api/files/" + storedName;
        FileUploadResponse response = new FileUploadResponse(
                file.getOriginalFilename(), downloadUrl, file.getSize(), contentType
        );

        fileShareNotifier.notifyRoom(roomId, username, response);

        return ResponseEntity.ok(response);
    }
}
```

```java
package br.com.exemplo.websocket.file;

public record FileUploadResponse(
        String fileName,
        String downloadUrl,
        long size,
        String contentType
) {
}
```

Download de arquivos:

```java
package br.com.exemplo.websocket.file;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/files")
public class FileDownloadController {

    private final Path uploadDir;

    public FileDownloadController(@Value("${app.upload.dir:uploads}") String uploadDirPath) {
        this.uploadDir = Path.of(uploadDirPath);
    }

    @GetMapping("/{fileName}")
    public ResponseEntity<Resource> download(@PathVariable String fileName) throws IOException {
        Path file = uploadDir.resolve(fileName).normalize();
        if (!file.startsWith(uploadDir) || !Files.exists(file)) {
            return ResponseEntity.notFound().build();
        }

        Resource resource = new UrlResource(file.toUri());
        String contentType = Files.probeContentType(file);

        return ResponseEntity.ok()
                .contentType(MediaType.parseMediaType(contentType != null ? contentType : "application/octet-stream"))
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
                .body(resource);
    }
}
```

Serviço que notifica a sala via WebSocket após o upload:

```java
package br.com.exemplo.websocket.file;

import br.com.exemplo.websocket.stomp.ChatMessageResponse;
import br.com.exemplo.websocket.stomp.MessageType;
import java.time.Instant;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

@Service
public class FileShareNotifier {

    private final SimpMessagingTemplate messagingTemplate;

    public FileShareNotifier(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    public void notifyRoom(String roomId, String username, FileUploadResponse fileInfo) {
        String content = String.format(
                "📎 %s compartilhou: %s (%s)",
                username, fileInfo.fileName(), formatSize(fileInfo.size())
        );

        ChatMessageResponse response = new ChatMessageResponse(
                username, "SYSTEM", content,
                roomId, Instant.now(), MessageType.FILE
        );
        messagingTemplate.convertAndSend("/topic/chat/" + roomId, response);
        messagingTemplate.convertAndSend("/topic/chat/" + roomId + "/files", fileInfo);
    }

    private String formatSize(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return (bytes / 1024) + " KB";
        return String.format("%.1f MB", bytes / (1024.0 * 1024.0));
    }
}
```

### 6.3. Envio direto via WebSocket (arquivos pequenos em Base64)

Para arquivos pequenos, é possível enviar diretamente via mensagem STOMP:

```java
package br.com.exemplo.websocket.file;

public record FileMessageRequest(
        String roomId,
        String fileName,
        String contentType,
        String base64Content
) {
}
```

```java
package br.com.exemplo.websocket.file;

import br.com.exemplo.websocket.stomp.MessageType;
import java.time.Instant;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class FileMessageController {

    private static final int MAX_BASE64_SIZE = 1_400_000;

    private final SimpMessagingTemplate messagingTemplate;

    public FileMessageController(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    @MessageMapping("/chat/file")
    public void handleFileMessage(
            FileMessageRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();

        if (request.base64Content().length() > MAX_BASE64_SIZE) {
            messagingTemplate.convertAndSendToUser(
                    username, "/queue/errors",
                    "Arquivo muito grande para envio direto. Use o upload HTTP."
            );
            return;
        }

        FileMessageResponse response = new FileMessageResponse(
                username,
                request.fileName(),
                request.contentType(),
                request.base64Content(),
                request.roomId(),
                Instant.now()
        );

        messagingTemplate.convertAndSend("/topic/chat/" + request.roomId() + "/files", response);
    }
}
```

```java
package br.com.exemplo.websocket.file;

import java.time.Instant;

public record FileMessageResponse(
        String sender,
        String fileName,
        String contentType,
        String base64Content,
        String roomId,
        Instant timestamp
) {
}
```

### 6.4. Configuração de tamanho de mensagem WebSocket

Para suportar mensagens maiores (necessário para Base64), ajuste os limites:

```java
package br.com.exemplo.websocket.stomp;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketTransportRegistration;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketTransportConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        registration.setMessageSizeLimit(2 * 1024 * 1024);
        registration.setSendBufferSizeLimit(2 * 1024 * 1024);
        registration.setSendTimeLimit(20_000);
    }
}
```

---

## 7. Interface web com HTML e JavaScript

### 7.1. Página de chat completa com STOMP

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chat — WebSocket STOMP</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: sans-serif; display: flex; height: 100vh; }

        #sidebar {
            width: 240px; background: #2c3e50; color: #ecf0f1;
            display: flex; flex-direction: column; padding: 12px;
        }
        #sidebar h3 { margin-bottom: 12px; }
        #room-list { list-style: none; flex: 1; overflow-y: auto; }
        #room-list li {
            padding: 8px; cursor: pointer; border-radius: 4px; margin-bottom: 4px;
        }
        #room-list li:hover, #room-list li.active { background: #34495e; }
        #new-room-input {
            padding: 8px; border: none; border-radius: 4px; margin-top: 8px;
        }

        #main { flex: 1; display: flex; flex-direction: column; }
        #header {
            background: #3498db; color: white;
            padding: 12px 16px; display: flex; justify-content: space-between; align-items: center;
        }
        #chat-area { flex: 1; overflow-y: auto; padding: 16px; background: #f5f5f5; }
        .message {
            margin-bottom: 12px; padding: 8px 12px;
            background: white; border-radius: 8px; max-width: 70%;
        }
        .message.system { background: #ffeaa7; font-style: italic; max-width: 100%; }
        .message .meta { font-size: 0.8em; color: #7f8c8d; margin-bottom: 4px; }
        .message .role-tag {
            display: inline-block; font-size: 0.7em; padding: 1px 6px;
            border-radius: 3px; color: white; margin-left: 4px;
        }
        .role-ADMIN { background: #e74c3c; }
        .role-MODERADOR { background: #e67e22; }
        .role-COMUM { background: #95a5a6; }

        #input-area {
            display: flex; padding: 12px; background: white; border-top: 1px solid #ddd;
        }
        #msg-input { flex: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
        #input-area button {
            margin-left: 8px; padding: 10px 16px; border: none;
            border-radius: 4px; cursor: pointer; background: #3498db; color: white;
        }
        #input-area button:hover { background: #2980b9; }

        #users-panel {
            width: 180px; background: #ecf0f1; padding: 12px;
            border-left: 1px solid #ddd; overflow-y: auto;
        }
        #users-panel h4 { margin-bottom: 8px; }
        #user-list { list-style: none; }
        #user-list li { padding: 4px 0; }
    </style>
</head>
<body>

<div id="sidebar">
    <h3>Salas</h3>
    <ul id="room-list"></ul>
    <input type="text" id="new-room-input" placeholder="Nova sala...">
</div>

<div id="main">
    <div id="header">
        <span id="room-title">Selecione uma sala</span>
        <span id="user-info"></span>
    </div>
    <div id="chat-area"></div>
    <div id="input-area">
        <input type="text" id="msg-input" placeholder="Digite sua mensagem..." disabled>
        <input type="file" id="file-input" style="display:none;">
        <button id="btn-file" disabled title="Enviar arquivo">📎</button>
        <button id="btn-send" disabled>Enviar</button>
    </div>
</div>

<div id="users-panel">
    <h4>Online</h4>
    <ul id="user-list"></ul>
</div>

<script src="https://cdn.jsdelivr.net/npm/sockjs-client@1/dist/sockjs.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/stompjs@2.3.3/lib/stomp.min.js"></script>
<script>
    const token = prompt("Informe seu JWT:");
    if (!token) { alert("Token obrigatório"); throw new Error("Token ausente"); }

    let stompClient = null;
    let currentRoom = null;
    const roomSubscriptions = {};

    const chatArea = document.getElementById("chat-area");
    const msgInput = document.getElementById("msg-input");
    const btnSend = document.getElementById("btn-send");
    const btnFile = document.getElementById("btn-file");
    const fileInput = document.getElementById("file-input");
    const roomList = document.getElementById("room-list");
    const userList = document.getElementById("user-list");
    const roomTitle = document.getElementById("room-title");
    const userInfo = document.getElementById("user-info");
    const newRoomInput = document.getElementById("new-room-input");

    function connect() {
        const socket = new SockJS("/ws/stomp");
        stompClient = Stomp.over(socket);
        stompClient.debug = null;

        stompClient.connect(
            { "Authorization": "Bearer " + token },
            function (frame) {
                const username = frame.headers["user-name"] || "usuário";
                userInfo.textContent = username;

                stompClient.subscribe("/user/queue/private", function (msg) {
                    const data = JSON.parse(msg.body);
                    appendMessage(data, true);
                });

                stompClient.subscribe("/user/queue/room-users", function (msg) {
                    const data = JSON.parse(msg.body);
                    if (data.roomId === currentRoom) {
                        updateUserList(data.users);
                    }
                });

                stompClient.subscribe("/user/queue/errors", function (msg) {
                    alert("Erro: " + msg.body);
                });

                msgInput.disabled = false;
                btnSend.disabled = false;
                btnFile.disabled = false;
            },
            function (error) {
                console.error("Erro de conexão:", error);
                setTimeout(connect, 5000);
            }
        );
    }

    function joinRoom(roomId) {
        if (roomSubscriptions[roomId]) {
            switchToRoom(roomId);
            return;
        }

        const chatSub = stompClient.subscribe("/topic/chat/" + roomId, function (msg) {
            const data = JSON.parse(msg.body);
            if (data.roomId === currentRoom) {
                appendMessage(data, false);
            }
        });

        const fileSub = stompClient.subscribe("/topic/chat/" + roomId + "/files", function (msg) {
            const data = JSON.parse(msg.body);
            if (currentRoom === roomId) {
                appendFileMessage(data);
            }
        });

        roomSubscriptions[roomId] = { chatSub: chatSub, fileSub: fileSub };

        stompClient.send("/app/chat/" + roomId + "/join", {}, "{}");

        addRoomToList(roomId);
        switchToRoom(roomId);
    }

    function switchToRoom(roomId) {
        currentRoom = roomId;
        roomTitle.textContent = "Sala: " + roomId;
        chatArea.innerHTML = "";

        document.querySelectorAll("#room-list li").forEach(function (li) {
            li.classList.toggle("active", li.dataset.room === roomId);
        });
    }

    function addRoomToList(roomId) {
        if (document.querySelector('[data-room="' + roomId + '"]')) return;
        const li = document.createElement("li");
        li.textContent = roomId;
        li.dataset.room = roomId;
        li.addEventListener("click", function () { switchToRoom(roomId); });
        roomList.appendChild(li);
    }

    function sendMessage() {
        const content = msgInput.value.trim();
        if (!content || !currentRoom) return;
        stompClient.send("/app/chat/" + currentRoom, {},
            JSON.stringify({ content: content, roomId: currentRoom })
        );
        msgInput.value = "";
    }

    function appendMessage(data, isPrivate) {
        const div = document.createElement("div");
        div.className = "message" + (data.type === "SYSTEM" || data.type === "JOIN" || data.type === "LEAVE" ? " system" : "");

        let roleTag = "";
        if (data.role && data.role !== "SYSTEM") {
            roleTag = '<span class="role-tag role-' + data.role + '">' + data.role + '</span>';
        }

        const prefix = isPrivate ? "[privado] " : "";
        const time = data.timestamp ? new Date(data.timestamp).toLocaleTimeString() : "";

        div.innerHTML =
            '<div class="meta">' + prefix + data.sender + roleTag + ' <small>' + time + '</small></div>' +
            '<div>' + escapeHtml(data.content) + '</div>';
        chatArea.appendChild(div);
        chatArea.scrollTop = chatArea.scrollHeight;
    }

    function appendFileMessage(data) {
        const div = document.createElement("div");
        div.className = "message";

        if (data.downloadUrl) {
            div.innerHTML =
                '<div class="meta">' + (data.sender || "arquivo") + '</div>' +
                '<a href="' + data.downloadUrl + '" target="_blank">📎 ' + escapeHtml(data.fileName) + '</a>';
        } else if (data.base64Content) {
            const link = "data:" + data.contentType + ";base64," + data.base64Content;
            div.innerHTML =
                '<div class="meta">' + (data.sender || "arquivo") + '</div>' +
                '<a href="' + link + '" download="' + escapeHtml(data.fileName) + '">📎 ' + escapeHtml(data.fileName) + '</a>';
        }
        chatArea.appendChild(div);
        chatArea.scrollTop = chatArea.scrollHeight;
    }

    function updateUserList(users) {
        userList.innerHTML = "";
        users.forEach(function (u) {
            const li = document.createElement("li");
            li.textContent = u;
            userList.appendChild(li);
        });
    }

    function escapeHtml(text) {
        const div = document.createElement("div");
        div.textContent = text;
        return div.innerHTML;
    }

    btnSend.addEventListener("click", sendMessage);
    msgInput.addEventListener("keydown", function (e) {
        if (e.key === "Enter") sendMessage();
    });

    btnFile.addEventListener("click", function () { fileInput.click(); });
    fileInput.addEventListener("change", function () {
        const file = fileInput.files[0];
        if (!file || !currentRoom) return;

        const formData = new FormData();
        formData.append("file", file);
        formData.append("roomId", currentRoom);
        formData.append("username", userInfo.textContent);

        fetch("/api/files", { method: "POST", body: formData })
            .then(function (resp) {
                if (!resp.ok) throw new Error("Falha no upload");
                return resp.json();
            })
            .then(function () {
                fileInput.value = "";
            })
            .catch(function (err) {
                alert("Erro no upload: " + err.message);
            });
    });

    newRoomInput.addEventListener("keydown", function (e) {
        if (e.key === "Enter") {
            const roomId = newRoomInput.value.trim();
            if (roomId) {
                joinRoom(roomId);
                newRoomInput.value = "";
            }
        }
    });

    connect();
</script>
</body>
</html>
```

Essa página oferece:

- barra lateral com lista de salas e entrada para criar novas;
- área central de chat com mensagens coloridas por papel do usuário;
- painel lateral com lista de usuários online na sala;
- suporte a múltiplas salas simultâneas com assinatura independente;
- envio de arquivos via botão com upload HTTP;
- indicação visual de role (`ADMIN`, `MODERADOR`, `COMUM`);
- reconexão automática em caso de perda de conexão.

### 7.2. Página de chat com WebSocket puro

Para a abordagem sem STOMP, um exemplo mais simples com múltiplas salas gerenciadas manualmente:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Chat — WebSocket Puro</title>
    <style>
        body { font-family: sans-serif; margin: 20px; }
        #chat-log {
            height: 400px; overflow-y: auto; border: 1px solid #ccc;
            padding: 12px; margin-bottom: 12px; background: #f9f9f9;
        }
        .msg { margin-bottom: 6px; }
        .msg-system { color: #888; font-style: italic; }
        .msg-role { font-size: 0.75em; padding: 1px 4px; border-radius: 3px; color: white; }
        .msg-role-ADMIN { background: #e74c3c; }
        .msg-role-MODERADOR { background: #e67e22; }
        .msg-role-COMUM { background: #95a5a6; }
        #controls { display: flex; gap: 8px; }
        #msg-input { flex: 1; padding: 8px; }
        button { padding: 8px 16px; cursor: pointer; }
    </style>
</head>
<body>
    <h2>Chat — WebSocket Puro</h2>
    <div id="chat-log"></div>
    <div id="controls">
        <input type="text" id="msg-input" placeholder="Mensagem...">
        <button id="btn-send">Enviar</button>
    </div>

    <script>
        const token = prompt("Informe seu JWT:");
        const ws = new WebSocket("ws://localhost:8080/ws/chat-raw?token=" + encodeURIComponent(token));
        const chatLog = document.getElementById("chat-log");

        ws.onmessage = function (event) {
            try {
                const data = JSON.parse(event.data);
                const roleSpan = data.role
                    ? '<span class="msg-role msg-role-' + data.role + '">' + data.role + '</span> '
                    : '';
                const div = document.createElement("div");
                div.className = "msg" + (data.type === "SYSTEM" ? " msg-system" : "");
                div.innerHTML = roleSpan + '<strong>' + escapeHtml(data.sender) + ':</strong> ' + escapeHtml(data.content);
                chatLog.appendChild(div);
            } catch (e) {
                const div = document.createElement("div");
                div.className = "msg";
                div.textContent = event.data;
                chatLog.appendChild(div);
            }
            chatLog.scrollTop = chatLog.scrollHeight;
        };

        ws.onclose = function () {
            const div = document.createElement("div");
            div.className = "msg msg-system";
            div.textContent = "Conexão encerrada.";
            chatLog.appendChild(div);
        };

        document.getElementById("btn-send").addEventListener("click", send);
        document.getElementById("msg-input").addEventListener("keydown", function (e) {
            if (e.key === "Enter") send();
        });

        function send() {
            const input = document.getElementById("msg-input");
            if (input.value.trim() && ws.readyState === WebSocket.OPEN) {
                ws.send(input.value);
                input.value = "";
            }
        }

        function escapeHtml(text) {
            const d = document.createElement("div");
            d.textContent = text;
            return d.innerHTML;
        }
    </script>
</body>
</html>
```

---

## 8. Heartbeat e detecção de conexões mortas

Conexões WebSocket podem se tornar inativas silenciosamente — por exemplo, quando o cliente perde a rede sem enviar um close frame. Sem detecção ativa, essas sessões fantasma continuam consumindo recursos e aparecem como usuários online.

### 8.1. Heartbeat no STOMP

O protocolo STOMP suporta heartbeat nativo. O servidor e o cliente negociam intervalos durante o `CONNECT`:

```java
package br.com.exemplo.websocket.stomp;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class StompHeartbeatConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/queue")
                .setHeartbeatValue(new long[]{10000, 10000})
                .setTaskScheduler(heartbeatScheduler());
        registry.setApplicationDestinationPrefixes("/app");
        registry.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws/stomp")
                .setAllowedOrigins("*")
                .withSockJS();
    }

    @org.springframework.context.annotation.Bean
    public org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler heartbeatScheduler() {
        org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler scheduler =
                new org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler();
        scheduler.setPoolSize(1);
        scheduler.setThreadNamePrefix("ws-heartbeat-");
        scheduler.initialize();
        return scheduler;
    }
}
```

O valor `{10000, 10000}` significa:

- o servidor envia heartbeat a cada 10 segundos;
- o servidor espera heartbeat do cliente a cada 10 segundos;
- se o cliente não responder dentro do prazo, a sessão é encerrada automaticamente.

No lado do cliente JavaScript, o STOMP.js envia heartbeats automaticamente quando configurado:

```javascript
stompClient.heartbeat.outgoing = 10000;
stompClient.heartbeat.incoming = 10000;
```

### 8.2. Ping/pong no WebSocket puro

Na abordagem sem STOMP, o protocolo WebSocket (RFC 6455) define frames de controle `ping` e `pong`. O Spring permite enviar pings periodicamente para detectar conexões mortas:

```java
package br.com.exemplo.websocket.raw;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.util.Set;
import java.util.concurrent.CopyOnWriteArraySet;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.PingMessage;
import org.springframework.web.socket.WebSocketSession;

@Component
public class WebSocketPingService {

    private final Set<WebSocketSession> sessions;

    public WebSocketPingService(ChatRawAuthHandler handler) {
        this.sessions = handler.getSessions();
    }

    @Scheduled(fixedRate = 30_000)
    public void sendPing() {
        PingMessage ping = new PingMessage(ByteBuffer.wrap("ping".getBytes()));
        for (WebSocketSession session : sessions) {
            if (session.isOpen()) {
                try {
                    session.sendMessage(ping);
                } catch (IOException ex) {
                    try {
                        session.close();
                    } catch (IOException ignored) {
                    }
                }
            }
        }
    }
}
```

O navegador responde automaticamente com um `pong`. Se o `sendMessage` falhar, a sessão está morta e deve ser removida.

Para que o handler exponha as sessões:

```java
public Set<WebSocketSession> getSessions() {
    return Collections.unmodifiableSet(sessions);
}
```

### 8.3. Configuração de timeout da sessão WebSocket

O Spring permite configurar limites de tempo na camada de transporte:

```java
package br.com.exemplo.websocket.raw;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import org.springframework.web.socket.server.standard.ServletServerContainerFactoryBean;

@Configuration
@EnableWebSocket
public class WebSocketTimeoutConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // handlers registrados aqui
    }

    @org.springframework.context.annotation.Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxSessionIdleTimeout(60_000L);
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        return container;
    }
}
```

O `maxSessionIdleTimeout` encerra automaticamente sessões que não trocam mensagens dentro do prazo definido.

---

## 9. Tratamento de erros

### 9.1. `@MessageExceptionHandler` em controllers STOMP

Análogo ao `@ExceptionHandler` do Spring MVC, o `@MessageExceptionHandler` captura exceções em métodos `@MessageMapping` e permite enviar uma resposta de erro ao usuário:

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;
import org.springframework.messaging.handler.annotation.MessageExceptionHandler;
import org.springframework.messaging.simp.annotation.SendToUser;
import org.springframework.web.bind.annotation.ControllerAdvice;

@ControllerAdvice
public class WebSocketExceptionHandler {

    @MessageExceptionHandler(IllegalArgumentException.class)
    @SendToUser("/queue/errors")
    public ErrorResponse handleIllegalArgument(IllegalArgumentException ex) {
        return new ErrorResponse("VALIDATION_ERROR", ex.getMessage(), Instant.now());
    }

    @MessageExceptionHandler(SecurityException.class)
    @SendToUser("/queue/errors")
    public ErrorResponse handleSecurity(SecurityException ex) {
        return new ErrorResponse("ACCESS_DENIED", ex.getMessage(), Instant.now());
    }

    @MessageExceptionHandler(Exception.class)
    @SendToUser("/queue/errors")
    public ErrorResponse handleGeneric(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "Erro interno no processamento da mensagem", Instant.now());
    }
}
```

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;

public record ErrorResponse(String code, String message, Instant timestamp) {
}
```

O `@SendToUser("/queue/errors")` direciona a resposta apenas ao remetente da mensagem que causou o erro. O cliente deve assinar `/user/queue/errors` para receber esses erros.

Também é possível colocar `@MessageExceptionHandler` diretamente em um `@Controller` específico, limitando o escopo ao controller:

```java
@Controller
public class ChatController {

    @MessageMapping("/chat/{roomId}")
    @SendTo("/topic/chat/{roomId}")
    public ChatMessageResponse sendMessage(/* ... */) {
        // ...
    }

    @MessageExceptionHandler
    @SendToUser("/queue/errors")
    public ErrorResponse handleError(Exception ex) {
        return new ErrorResponse("CHAT_ERROR", ex.getMessage(), Instant.now());
    }
}
```

### 9.2. `StompSubProtocolErrorHandler` para erros de protocolo

Para tratar erros que ocorrem antes do roteamento para `@MessageMapping` — como falhas de autenticação no `CONNECT`, mensagens mal-formadas ou frames inválidos:

```java
package br.com.exemplo.websocket.stomp;

import org.springframework.messaging.Message;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.web.socket.messaging.StompSubProtocolErrorHandler;

public class CustomStompErrorHandler extends StompSubProtocolErrorHandler {

    @Override
    public Message<byte[]> handleClientMessageProcessingError(
            Message<byte[]> clientMessage,
            Throwable ex
    ) {
        Throwable cause = ex.getCause() != null ? ex.getCause() : ex;

        String errorMessage;
        if (cause instanceof IllegalArgumentException) {
            errorMessage = "Erro de validação: " + cause.getMessage();
        } else if (cause instanceof SecurityException) {
            errorMessage = "Acesso negado";
        } else {
            errorMessage = "Erro no processamento da mensagem";
        }

        StompHeaderAccessor accessor = StompHeaderAccessor.create(
                org.springframework.messaging.simp.stomp.StompCommand.ERROR
        );
        accessor.setMessage(errorMessage);
        accessor.setLeaveMutable(true);

        return MessageBuilder.createMessage(
                errorMessage.getBytes(java.nio.charset.StandardCharsets.UTF_8),
                accessor.getMessageHeaders()
        );
    }
}
```

Registro na configuração:

```java
package br.com.exemplo.websocket.stomp;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class StompErrorConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws/stomp")
                .setAllowedOrigins("*")
                .withSockJS();
        registry.setErrorHandler(new CustomStompErrorHandler());
    }
}
```

### 9.3. Tratamento de erros no WebSocket puro

Na abordagem sem STOMP, erros são capturados no handler:

```java
package br.com.exemplo.websocket.raw;

import java.io.IOException;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

public class ChatRawErrorHandler extends TextWebSocketHandler {

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws IOException {
        try {
            String payload = message.getPayload();
            if (payload.isBlank()) {
                session.sendMessage(new TextMessage("{\"error\":\"Mensagem vazia\"}"));
                return;
            }
            // processamento normal
        } catch (Exception ex) {
            session.sendMessage(new TextMessage("{\"error\":\"Erro interno\"}"));
        }
    }

    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        System.err.println("Erro de transporte na sessão " + session.getId() + ": " + exception.getMessage());
        if (session.isOpen()) {
            session.close(CloseStatus.SERVER_ERROR);
        }
    }
}
```

O `handleTransportError` é chamado quando há falha na camada de transporte (rede, encoding, etc.) — é o equivalente do `StompSubProtocolErrorHandler` para WebSocket puro.

### 9.4. Recebendo erros no cliente JavaScript

Para STOMP, o cliente assina a fila de erros do usuário:

```javascript
stompClient.subscribe("/user/queue/errors", function (msg) {
    const error = JSON.parse(msg.body);
    console.error("Erro do servidor:", error.code, error.message);
    showNotification("Erro: " + error.message);
});
```

O STOMP também pode receber error frames de protocolo no callback de erro da conexão:

```javascript
stompClient.connect(
    headers,
    function () { /* sucesso */ },
    function (frame) {
        if (frame.headers && frame.headers.message) {
            console.error("Erro STOMP:", frame.headers.message);
        }
    }
);
```

Para WebSocket puro:

```javascript
ws.onmessage = function (event) {
    const data = JSON.parse(event.data);
    if (data.error) {
        showNotification("Erro: " + data.error);
        return;
    }
    // processamento normal
};

ws.onerror = function (event) {
    console.error("Erro WebSocket:", event);
};
```

---

## 10. Rate limiting por usuário

Em aplicações de chat, é comum que usuários enviem muitas mensagens em sequência — por excesso de entusiasmo ou por abuso intencional (flood). Limitar a taxa de mensagens por usuário protege o servidor e melhora a experiência dos demais participantes.

### 10.1. Rate limiter para mensagens STOMP

```java
package br.com.exemplo.websocket.ratelimit;

import io.github.bucket4j.Bandwidth;
import io.github.bucket4j.Bucket;
import java.time.Duration;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import org.springframework.stereotype.Component;

@Component
public class MessageRateLimiter {

    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

    public boolean tryConsume(String username) {
        Bucket bucket = buckets.computeIfAbsent(username, key ->
                Bucket.builder()
                        .addLimit(Bandwidth.builder()
                                .capacity(10)
                                .refillGreedy(10, Duration.ofSeconds(5))
                                .build())
                        .build()
        );
        return bucket.tryConsume(1);
    }

    public void removeBucket(String username) {
        buckets.remove(username);
    }
}
```

O limite de 10 mensagens a cada 5 segundos é um ponto de partida razoável para chat. O valor deve ser ajustado conforme o tipo de aplicação.

### 10.2. Interceptor para aplicar rate limiting

```java
package br.com.exemplo.websocket.ratelimit;

import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.ChannelInterceptor;
import org.springframework.messaging.support.MessageHeaderAccessor;

public class RateLimitInterceptor implements ChannelInterceptor {

    private final MessageRateLimiter rateLimiter;

    public RateLimitInterceptor(MessageRateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);

        if (accessor == null || accessor.getCommand() != StompCommand.SEND) {
            return message;
        }

        if (accessor.getUser() == null) {
            return message;
        }

        String username = accessor.getUser().getName();
        if (!rateLimiter.tryConsume(username)) {
            throw new MessageRateLimitExceededException(
                    "Limite de mensagens excedido. Aguarde alguns segundos."
            );
        }

        return message;
    }
}
```

```java
package br.com.exemplo.websocket.ratelimit;

public class MessageRateLimitExceededException extends RuntimeException {

    public MessageRateLimitExceededException(String message) {
        super(message);
    }
}
```

### 10.3. Registro do interceptor

O interceptor de rate limiting deve ser registrado no canal de entrada, junto com o interceptor de autenticação:

```java
@Override
public void configureClientInboundChannel(ChannelRegistration registration) {
    registration.interceptors(
            new JwtStompInterceptor(jwtTokenProvider),
            new RateLimitInterceptor(rateLimiter)
    );
}
```

A ordem importa: a autenticação deve ocorrer antes do rate limiting, para que o username já esteja disponível.

### 10.4. Capturando o erro de rate limit no `@MessageExceptionHandler`

```java
@MessageExceptionHandler(MessageRateLimitExceededException.class)
@SendToUser("/queue/errors")
public ErrorResponse handleRateLimit(MessageRateLimitExceededException ex) {
    return new ErrorResponse("RATE_LIMITED", ex.getMessage(), Instant.now());
}
```

### 10.5. Rate limiting no WebSocket puro

Na abordagem sem STOMP, o controle fica dentro do handler:

```java
package br.com.exemplo.websocket.raw;

import br.com.exemplo.websocket.ratelimit.MessageRateLimiter;
import java.io.IOException;
import java.util.Set;
import java.util.concurrent.CopyOnWriteArraySet;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

public class ChatRawRateLimitedHandler extends TextWebSocketHandler {

    private final Set<WebSocketSession> sessions = new CopyOnWriteArraySet<>();
    private final MessageRateLimiter rateLimiter;

    public ChatRawRateLimitedHandler(MessageRateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws IOException {
        String username = (String) session.getAttributes().get("username");

        if (!rateLimiter.tryConsume(username)) {
            session.sendMessage(new TextMessage(
                    "{\"type\":\"SYSTEM\",\"content\":\"Você está enviando mensagens rápido demais. Aguarde.\"}"
            ));
            return;
        }

        // processamento normal — broadcast para as sessões
    }
}
```

### 10.6. Feedback visual no cliente

```javascript
stompClient.subscribe("/user/queue/errors", function (msg) {
    const error = JSON.parse(msg.body);
    if (error.code === "RATE_LIMITED") {
        msgInput.disabled = true;
        btnSend.disabled = true;
        showNotification(error.message);
        setTimeout(function () {
            msgInput.disabled = false;
            btnSend.disabled = false;
        }, 3000);
    }
});
```

---

## 11. Histórico de mensagens

Em aplicações de chat reais, é importante que o usuário veja as mensagens anteriores ao entrar numa sala. Isso normalmente envolve persistência e um endpoint para carregamento.

### 11.1. Entidade de mensagem

```java
package br.com.exemplo.websocket.history;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.EnumType;
import jakarta.persistence.Enumerated;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Index;
import jakarta.persistence.Table;
import java.time.Instant;
import br.com.exemplo.websocket.stomp.MessageType;

@Entity
@Table(name = "chat_message", indexes = {
        @Index(name = "idx_chat_message_room_time", columnList = "roomId, timestamp")
})
public class ChatMessageEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String roomId;

    @Column(nullable = false)
    private String sender;

    @Column(nullable = false)
    private String role;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;

    @Column(nullable = false)
    private Instant timestamp;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private MessageType type;

    public ChatMessageEntity() {
    }

    public ChatMessageEntity(String roomId, String sender, String role, String content,
                             Instant timestamp, MessageType type) {
        this.roomId = roomId;
        this.sender = sender;
        this.role = role;
        this.content = content;
        this.timestamp = timestamp;
        this.type = type;
    }

    // Getters omitidos por brevidade.
}
```

### 11.2. Repositório

```java
package br.com.exemplo.websocket.history;

import java.time.Instant;
import java.util.List;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ChatMessageRepository extends JpaRepository<ChatMessageEntity, Long> {

    List<ChatMessageEntity> findByRoomIdOrderByTimestampDesc(String roomId, Pageable pageable);

    List<ChatMessageEntity> findByRoomIdAndTimestampBeforeOrderByTimestampDesc(
            String roomId, Instant before, Pageable pageable
    );
}
```

### 11.3. Serviço de persistência

```java
package br.com.exemplo.websocket.history;

import br.com.exemplo.websocket.stomp.ChatMessageResponse;
import br.com.exemplo.websocket.stomp.MessageType;
import java.time.Instant;
import java.util.Collections;
import java.util.List;
import org.springframework.data.domain.PageRequest;
import org.springframework.stereotype.Service;

@Service
public class ChatHistoryService {

    private static final int DEFAULT_PAGE_SIZE = 50;

    private final ChatMessageRepository repository;

    public ChatHistoryService(ChatMessageRepository repository) {
        this.repository = repository;
    }

    public void save(ChatMessageResponse message) {
        if (message.type() == MessageType.JOIN || message.type() == MessageType.LEAVE) {
            return;
        }

        ChatMessageEntity entity = new ChatMessageEntity(
                message.roomId(),
                message.sender(),
                message.role(),
                message.content(),
                message.timestamp(),
                message.type()
        );
        repository.save(entity);
    }

    public List<ChatMessageResponse> getHistory(String roomId, int size) {
        List<ChatMessageEntity> entities = repository.findByRoomIdOrderByTimestampDesc(
                roomId, PageRequest.of(0, size)
        );
        List<ChatMessageResponse> result = entities.stream()
                .map(this::toResponse)
                .toList();
        return result.reversed();
    }

    public List<ChatMessageResponse> getHistoryBefore(String roomId, Instant before, int size) {
        List<ChatMessageEntity> entities = repository.findByRoomIdAndTimestampBeforeOrderByTimestampDesc(
                roomId, before, PageRequest.of(0, size)
        );
        List<ChatMessageResponse> result = entities.stream()
                .map(this::toResponse)
                .toList();
        return result.reversed();
    }

    private ChatMessageResponse toResponse(ChatMessageEntity entity) {
        return new ChatMessageResponse(
                entity.getSender(),
                entity.getRole(),
                entity.getContent(),
                entity.getRoomId(),
                entity.getTimestamp(),
                entity.getType()
        );
    }
}
```

### 11.4. Endpoint REST para carregar histórico

```java
package br.com.exemplo.websocket.history;

import br.com.exemplo.websocket.stomp.ChatMessageResponse;
import java.time.Instant;
import java.util.List;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/chat/history")
public class ChatHistoryController {

    private final ChatHistoryService historyService;

    public ChatHistoryController(ChatHistoryService historyService) {
        this.historyService = historyService;
    }

    @GetMapping("/{roomId}")
    public List<ChatMessageResponse> getHistory(
            @PathVariable String roomId,
            @RequestParam(defaultValue = "50") int size
    ) {
        return historyService.getHistory(roomId, Math.min(size, 200));
    }

    @GetMapping("/{roomId}/before")
    public List<ChatMessageResponse> getHistoryBefore(
            @PathVariable String roomId,
            @RequestParam Instant before,
            @RequestParam(defaultValue = "50") int size
    ) {
        return historyService.getHistoryBefore(roomId, before, Math.min(size, 200));
    }
}
```

### 11.5. Salvando mensagens no fluxo do chat

O `ChatController` persiste as mensagens após o envio:

```java
package br.com.exemplo.websocket.stomp;

import br.com.exemplo.websocket.history.ChatHistoryService;
import java.time.Instant;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.stereotype.Controller;

@Controller
public class ChatControllerWithHistory {

    private final ChatHistoryService historyService;

    public ChatControllerWithHistory(ChatHistoryService historyService) {
        this.historyService = historyService;
    }

    @MessageMapping("/chat/{roomId}")
    @SendTo("/topic/chat/{roomId}")
    public ChatMessageResponse sendMessage(
            @DestinationVariable String roomId,
            ChatMessageRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        String role = (String) headerAccessor.getSessionAttributes().get("role");

        ChatMessageResponse response = new ChatMessageResponse(
                username, role, request.content(),
                roomId, Instant.now(), MessageType.CHAT
        );

        historyService.save(response);

        return response;
    }
}
```

### 11.6. Carregando histórico no cliente ao entrar na sala

```javascript
function joinRoom(roomId) {
    // carrega histórico via REST antes de assinar o tópico
    fetch("/api/chat/history/" + encodeURIComponent(roomId) + "?size=50")
        .then(function (resp) { return resp.json(); })
        .then(function (messages) {
            chatArea.innerHTML = "";
            messages.forEach(function (msg) {
                appendMessage(msg, false);
            });

            // após carregar o histórico, assina o tópico para novas mensagens
            subscribeToRoom(roomId);
        });
}

// scroll infinito para carregar mensagens mais antigas
chatArea.addEventListener("scroll", function () {
    if (chatArea.scrollTop === 0 && currentRoom) {
        const firstMsg = chatArea.querySelector(".message");
        if (firstMsg && firstMsg.dataset.timestamp) {
            fetch("/api/chat/history/" + encodeURIComponent(currentRoom)
                + "/before?before=" + firstMsg.dataset.timestamp + "&size=50")
                .then(function (resp) { return resp.json(); })
                .then(function (olderMessages) {
                    const prevHeight = chatArea.scrollHeight;
                    olderMessages.forEach(function (msg) {
                        const div = createMessageDiv(msg);
                        chatArea.insertBefore(div, chatArea.firstChild);
                    });
                    chatArea.scrollTop = chatArea.scrollHeight - prevHeight;
                });
        }
    }
});
```

---

## 12. Indicadores de digitação e presença

### 12.1. Indicador de digitação ("fulano está digitando...")

O indicador de digitação é uma mensagem efêmera enviada ao tópico da sala. O cliente envia um evento quando o usuário começa a digitar e para de enviar quando ele para. Não é necessário persistir essas mensagens.

DTO:

```java
package br.com.exemplo.websocket.stomp;

public record TypingEvent(String username, String roomId, boolean typing) {
}
```

Controller:

```java
package br.com.exemplo.websocket.stomp;

import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.stereotype.Controller;

@Controller
public class TypingController {

    @MessageMapping("/chat/{roomId}/typing")
    @SendTo("/topic/chat/{roomId}/typing")
    public TypingEvent handleTyping(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        return new TypingEvent(username, roomId, true);
    }

    @MessageMapping("/chat/{roomId}/stop-typing")
    @SendTo("/topic/chat/{roomId}/typing")
    public TypingEvent handleStopTyping(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        return new TypingEvent(username, roomId, false);
    }
}
```

Cliente JavaScript com debounce para evitar excesso de mensagens:

```javascript
let typingTimer = null;
let isTyping = false;

msgInput.addEventListener("input", function () {
    if (!currentRoom) return;

    if (!isTyping) {
        isTyping = true;
        stompClient.send("/app/chat/" + currentRoom + "/typing", {}, "{}");
    }

    clearTimeout(typingTimer);
    typingTimer = setTimeout(function () {
        isTyping = false;
        stompClient.send("/app/chat/" + currentRoom + "/stop-typing", {}, "{}");
    }, 2000);
});

// assinatura para receber eventos de digitação
stompClient.subscribe("/topic/chat/" + roomId + "/typing", function (msg) {
    const event = JSON.parse(msg.body);
    updateTypingIndicator(event);
});

const typingUsers = new Set();

function updateTypingIndicator(event) {
    if (event.typing) {
        typingUsers.add(event.username);
    } else {
        typingUsers.delete(event.username);
    }

    const indicator = document.getElementById("typing-indicator");
    if (typingUsers.size === 0) {
        indicator.textContent = "";
    } else if (typingUsers.size === 1) {
        indicator.textContent = [...typingUsers][0] + " está digitando...";
    } else {
        indicator.textContent = typingUsers.size + " pessoas estão digitando...";
    }
}
```

### 12.2. Presença online (status do usuário)

Para mostrar quem está online, ausente ou offline, o servidor mantém o estado de presença e notifica as salas:

```java
package br.com.exemplo.websocket.stomp;

public enum PresenceStatus {
    ONLINE,
    AWAY,
    OFFLINE
}
```

```java
package br.com.exemplo.websocket.stomp;

public record PresenceEvent(String username, PresenceStatus status) {
}
```

Serviço de presença:

```java
package br.com.exemplo.websocket.stomp;

import java.time.Instant;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.stream.Collectors;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

@Service
public class PresenceService {

    private static final long AWAY_THRESHOLD_MS = 5 * 60 * 1000;

    private final Map<String, Instant> lastActivity = new ConcurrentHashMap<>();
    private final Map<String, PresenceStatus> currentStatus = new ConcurrentHashMap<>();
    private final SimpMessagingTemplate messagingTemplate;
    private final ActiveSessionRegistry sessionRegistry;

    public PresenceService(SimpMessagingTemplate messagingTemplate, ActiveSessionRegistry sessionRegistry) {
        this.messagingTemplate = messagingTemplate;
        this.sessionRegistry = sessionRegistry;
    }

    public void markActive(String username) {
        lastActivity.put(username, Instant.now());
        updateStatus(username, PresenceStatus.ONLINE);
    }

    public void markOffline(String username) {
        lastActivity.remove(username);
        updateStatus(username, PresenceStatus.OFFLINE);
        currentStatus.remove(username);
    }

    public Map<String, PresenceStatus> getPresenceForRoom(String roomId) {
        Set<String> users = sessionRegistry.getUsersInRoom(roomId);
        return users.stream()
                .collect(Collectors.toMap(
                        u -> u,
                        u -> currentStatus.getOrDefault(u, PresenceStatus.OFFLINE)
                ));
    }

    @Scheduled(fixedRate = 30_000)
    public void checkAway() {
        Instant threshold = Instant.now().minusMillis(AWAY_THRESHOLD_MS);
        for (Map.Entry<String, Instant> entry : lastActivity.entrySet()) {
            if (entry.getValue().isBefore(threshold)) {
                updateStatus(entry.getKey(), PresenceStatus.AWAY);
            }
        }
    }

    private void updateStatus(String username, PresenceStatus newStatus) {
        PresenceStatus previous = currentStatus.put(username, newStatus);
        if (previous == newStatus) {
            return;
        }

        PresenceEvent event = new PresenceEvent(username, newStatus);
        Set<String> rooms = sessionRegistry.getRoomsForUser(username);
        for (String roomId : rooms) {
            messagingTemplate.convertAndSend("/topic/chat/" + roomId + "/presence", event);
        }
    }
}
```

Controller para solicitar a lista de presença de uma sala:

```java
package br.com.exemplo.websocket.stomp;

import java.util.Map;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class PresenceController {

    private final PresenceService presenceService;
    private final SimpMessagingTemplate messagingTemplate;

    public PresenceController(PresenceService presenceService, SimpMessagingTemplate messagingTemplate) {
        this.presenceService = presenceService;
        this.messagingTemplate = messagingTemplate;
    }

    @MessageMapping("/chat/{roomId}/presence")
    public void requestPresence(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        presenceService.markActive(username);

        Map<String, PresenceStatus> presence = presenceService.getPresenceForRoom(roomId);
        messagingTemplate.convertAndSendToUser(username, "/queue/presence", presence);
    }

    @MessageMapping("/activity")
    public void heartbeatActivity(SimpMessageHeaderAccessor headerAccessor) {
        String username = headerAccessor.getUser().getName();
        presenceService.markActive(username);
    }
}
```

Cliente JavaScript com heartbeat de atividade e exibição de presença:

```javascript
// heartbeat de atividade a cada 60 segundos
setInterval(function () {
    if (stompClient && stompClient.connected) {
        stompClient.send("/app/activity", {}, "{}");
    }
}, 60_000);

// assinar presença da sala
stompClient.subscribe("/topic/chat/" + roomId + "/presence", function (msg) {
    const event = JSON.parse(msg.body);
    updateUserPresence(event.username, event.status);
});

stompClient.subscribe("/user/queue/presence", function (msg) {
    const presenceMap = JSON.parse(msg.body);
    for (const [user, status] of Object.entries(presenceMap)) {
        updateUserPresence(user, status);
    }
});

// solicitar estado atual de presença ao entrar na sala
stompClient.send("/app/chat/" + roomId + "/presence", {}, "{}");

function updateUserPresence(username, status) {
    const items = document.querySelectorAll("#user-list li");
    items.forEach(function (li) {
        if (li.textContent.startsWith(username)) {
            const indicator = status === "ONLINE" ? "🟢"
                            : status === "AWAY" ? "🟡"
                            : "⚫";
            li.textContent = username + " " + indicator;
        }
    });
}
```

---

## 13. Testes de integração

### 13.1. Dependências de teste

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
    <scope>test</scope>
</dependency>
```

### 13.2. Teste de WebSocket puro

```java
package br.com.exemplo.websocket;

import java.net.URI;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketHttpHeaders;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.client.standard.StandardWebSocketClient;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RawWebSocketIntegrationTest {

    @LocalServerPort
    private int port;

    @Test
    void deveEnviarEReceberMensagem() throws Exception {
        BlockingQueue<String> messages = new LinkedBlockingQueue<>();

        StandardWebSocketClient client = new StandardWebSocketClient();
        URI uri = URI.create("ws://localhost:" + port + "/ws/chat-raw?token=TOKEN_VALIDO");

        WebSocketSession session = client.execute(new TextWebSocketHandler() {
            @Override
            protected void handleTextMessage(WebSocketSession s, TextMessage message) {
                messages.add(message.getPayload());
            }
        }, new WebSocketHttpHeaders(), uri).get(5, TimeUnit.SECONDS);

        session.sendMessage(new TextMessage("Olá!"));

        String response = messages.poll(5, TimeUnit.SECONDS);
        assertThat(response).contains("Olá!");

        session.close();
    }
}
```

### 13.3. Teste de STOMP com `WebSocketStompClient`

```java
package br.com.exemplo.websocket;

import br.com.exemplo.websocket.stomp.ChatMessageRequest;
import br.com.exemplo.websocket.stomp.ChatMessageResponse;
import java.lang.reflect.Type;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.messaging.converter.MappingJackson2MessageConverter;
import org.springframework.messaging.simp.stomp.StompFrameHandler;
import org.springframework.messaging.simp.stomp.StompHeaders;
import org.springframework.messaging.simp.stomp.StompSession;
import org.springframework.messaging.simp.stomp.StompSessionHandlerAdapter;
import org.springframework.web.socket.client.standard.StandardWebSocketClient;
import org.springframework.web.socket.messaging.WebSocketStompClient;
import org.springframework.web.socket.sockjs.client.SockJsClient;
import org.springframework.web.socket.sockjs.client.WebSocketTransport;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class StompWebSocketIntegrationTest {

    @LocalServerPort
    private int port;

    private WebSocketStompClient stompClient;

    @BeforeEach
    void setUp() {
        SockJsClient sockJsClient = new SockJsClient(
                List.of(new WebSocketTransport(new StandardWebSocketClient()))
        );
        stompClient = new WebSocketStompClient(sockJsClient);
        stompClient.setMessageConverter(new MappingJackson2MessageConverter());
    }

    @AfterEach
    void tearDown() {
        if (stompClient.isRunning()) {
            stompClient.stop();
        }
    }

    @Test
    void deveEnviarMensagemParaSala() throws Exception {
        BlockingQueue<ChatMessageResponse> messages = new LinkedBlockingQueue<>();

        String url = "ws://localhost:" + port + "/ws/stomp";

        StompHeaders connectHeaders = new StompHeaders();
        connectHeaders.add("Authorization", "Bearer TOKEN_VALIDO");

        StompSession session = stompClient.connectAsync(
                url,
                new StompSessionHandlerAdapter() {}
        ).get(5, TimeUnit.SECONDS);

        session.subscribe("/topic/chat/sala-teste", new StompFrameHandler() {
            @Override
            public Type getPayloadType(StompHeaders headers) {
                return ChatMessageResponse.class;
            }

            @Override
            public void handleFrame(StompHeaders headers, Object payload) {
                messages.add((ChatMessageResponse) payload);
            }
        });

        Thread.sleep(500);

        session.send("/app/chat/sala-teste",
                new ChatMessageRequest("Mensagem de teste", "sala-teste"));

        ChatMessageResponse response = messages.poll(5, TimeUnit.SECONDS);
        assertThat(response).isNotNull();
        assertThat(response.content()).isEqualTo("Mensagem de teste");
        assertThat(response.roomId()).isEqualTo("sala-teste");

        session.disconnect();
    }
}
```

### 13.4. Teste de múltiplos clientes em uma sala

```java
package br.com.exemplo.websocket;

import br.com.exemplo.websocket.stomp.ChatMessageRequest;
import br.com.exemplo.websocket.stomp.ChatMessageResponse;
import java.lang.reflect.Type;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.messaging.converter.MappingJackson2MessageConverter;
import org.springframework.messaging.simp.stomp.StompFrameHandler;
import org.springframework.messaging.simp.stomp.StompHeaders;
import org.springframework.messaging.simp.stomp.StompSession;
import org.springframework.messaging.simp.stomp.StompSessionHandlerAdapter;
import org.springframework.web.socket.client.standard.StandardWebSocketClient;
import org.springframework.web.socket.messaging.WebSocketStompClient;
import org.springframework.web.socket.sockjs.client.SockJsClient;
import org.springframework.web.socket.sockjs.client.WebSocketTransport;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class MultiClientStompTest {

    @LocalServerPort
    private int port;

    @Test
    void ambosClientesRecebemMensagemDaSala() throws Exception {
        WebSocketStompClient client1 = createStompClient();
        WebSocketStompClient client2 = createStompClient();

        String url = "ws://localhost:" + port + "/ws/stomp";

        StompSession session1 = client1.connectAsync(url, new StompSessionHandlerAdapter() {})
                .get(5, TimeUnit.SECONDS);
        StompSession session2 = client2.connectAsync(url, new StompSessionHandlerAdapter() {})
                .get(5, TimeUnit.SECONDS);

        BlockingQueue<ChatMessageResponse> messages1 = new LinkedBlockingQueue<>();
        BlockingQueue<ChatMessageResponse> messages2 = new LinkedBlockingQueue<>();

        session1.subscribe("/topic/chat/sala-multi", frameHandler(messages1));
        session2.subscribe("/topic/chat/sala-multi", frameHandler(messages2));

        Thread.sleep(500);

        session1.send("/app/chat/sala-multi",
                new ChatMessageRequest("Olá de client1", "sala-multi"));

        ChatMessageResponse resp1 = messages1.poll(5, TimeUnit.SECONDS);
        ChatMessageResponse resp2 = messages2.poll(5, TimeUnit.SECONDS);

        assertThat(resp1).isNotNull();
        assertThat(resp2).isNotNull();
        assertThat(resp1.content()).isEqualTo("Olá de client1");
        assertThat(resp2.content()).isEqualTo("Olá de client1");

        session1.disconnect();
        session2.disconnect();
        client1.stop();
        client2.stop();
    }

    private WebSocketStompClient createStompClient() {
        SockJsClient sockJsClient = new SockJsClient(
                List.of(new WebSocketTransport(new StandardWebSocketClient()))
        );
        WebSocketStompClient client = new WebSocketStompClient(sockJsClient);
        client.setMessageConverter(new MappingJackson2MessageConverter());
        return client;
    }

    private StompFrameHandler frameHandler(BlockingQueue<ChatMessageResponse> queue) {
        return new StompFrameHandler() {
            @Override
            public Type getPayloadType(StompHeaders headers) {
                return ChatMessageResponse.class;
            }

            @Override
            public void handleFrame(StompHeaders headers, Object payload) {
                queue.add((ChatMessageResponse) payload);
            }
        };
    }
}
```

### 13.5. Teste de mensagem privada

```java
package br.com.exemplo.websocket;

import br.com.exemplo.websocket.stomp.ChatMessageResponse;
import br.com.exemplo.websocket.stomp.PrivateMessageRequest;
import java.lang.reflect.Type;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.messaging.converter.MappingJackson2MessageConverter;
import org.springframework.messaging.simp.stomp.StompFrameHandler;
import org.springframework.messaging.simp.stomp.StompHeaders;
import org.springframework.messaging.simp.stomp.StompSession;
import org.springframework.messaging.simp.stomp.StompSessionHandlerAdapter;
import org.springframework.web.socket.client.standard.StandardWebSocketClient;
import org.springframework.web.socket.messaging.WebSocketStompClient;
import org.springframework.web.socket.sockjs.client.SockJsClient;
import org.springframework.web.socket.sockjs.client.WebSocketTransport;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class PrivateMessageStompTest {

    @LocalServerPort
    private int port;

    @Test
    void deveEnviarMensagemPrivada() throws Exception {
        SockJsClient sockJsClient = new SockJsClient(
                List.of(new WebSocketTransport(new StandardWebSocketClient()))
        );
        WebSocketStompClient stompClient = new WebSocketStompClient(sockJsClient);
        stompClient.setMessageConverter(new MappingJackson2MessageConverter());

        String url = "ws://localhost:" + port + "/ws/stomp";

        StompSession senderSession = stompClient.connectAsync(url, new StompSessionHandlerAdapter() {})
                .get(5, TimeUnit.SECONDS);

        StompSession receiverSession = stompClient.connectAsync(url, new StompSessionHandlerAdapter() {})
                .get(5, TimeUnit.SECONDS);

        BlockingQueue<ChatMessageResponse> receiverMessages = new LinkedBlockingQueue<>();

        receiverSession.subscribe("/user/queue/private", new StompFrameHandler() {
            @Override
            public Type getPayloadType(StompHeaders headers) {
                return ChatMessageResponse.class;
            }

            @Override
            public void handleFrame(StompHeaders headers, Object payload) {
                receiverMessages.add((ChatMessageResponse) payload);
            }
        });

        Thread.sleep(500);

        senderSession.send("/app/private",
                new PrivateMessageRequest("receiver-user", "Mensagem secreta"));

        ChatMessageResponse received = receiverMessages.poll(5, TimeUnit.SECONDS);
        assertThat(received).isNotNull();
        assertThat(received.content()).isEqualTo("Mensagem secreta");

        senderSession.disconnect();
        receiverSession.disconnect();
        stompClient.stop();
    }
}
```

### 13.6. Boas práticas em testes WebSocket

- use `@SpringBootTest(webEnvironment = RANDOM_PORT)` para evitar conflito de portas;
- use `BlockingQueue` com `poll(timeout)` para aguardar mensagens assíncronas;
- adicione um `Thread.sleep` curto entre subscribe e send para garantir que a assinatura esteja ativa;
- feche sessões e pare clientes no `@AfterEach` para evitar vazamento de recursos;
- para testes que envolvem autenticação JWT, crie um token de teste válido no `@BeforeEach` ou use um `TestJwtTokenProvider`;
- separe testes de integração (que sobem o servidor) de testes unitários (que testam handlers isoladamente).

---

## 14. Boas práticas

### 14.1. Segurança

- nunca confie em dados vindos do cliente WebSocket sem validação;
- valide o JWT no handshake (WebSocket puro) ou no CONNECT (STOMP);
- proteja destinations sensíveis com autorização por role;
- evite expor credenciais na URL de conexão em produção — prefira headers STOMP;
- higienize conteúdo de mensagens antes de renderizar no frontend (XSS);
- valide tipo e tamanho de arquivos no servidor antes de aceitar uploads;
- verifique path traversal em nomes de arquivos (`file.normalize()`, `file.startsWith(uploadDir)`);
- limite o tamanho de mensagens WebSocket para evitar abuso.

### 14.2. Escalabilidade

- para múltiplas instâncias, substitua o simple broker por um broker externo (RabbitMQ, ActiveMQ);
- sessões em memória (`CopyOnWriteArraySet`, `ConcurrentHashMap`) funcionam para uma instância — para cluster, use Redis ou banco compartilhado;
- considere heartbeat para detectar conexões mortas;
- monitore métricas de conexões ativas, mensagens por segundo e erros.

Exemplo de configuração com broker externo RabbitMQ:

```java
@Override
public void configureMessageBroker(MessageBrokerRegistry registry) {
    registry.enableStompBrokerRelay("/topic", "/queue")
            .setRelayHost("localhost")
            .setRelayPort(61613)
            .setClientLogin("guest")
            .setClientPasscode("guest");
    registry.setApplicationDestinationPrefixes("/app");
    registry.setUserDestinationPrefix("/user");
}
```

### 14.3. Resiliência no cliente

- implemente reconexão automática com backoff exponencial;
- armazene mensagens pendentes localmente durante a desconexão;
- trate erros de conexão de forma silenciosa para o usuário.

Exemplo de reconexão com backoff:

```javascript
let reconnectDelay = 1000;
const MAX_DELAY = 30000;

function connect() {
    const socket = new SockJS("/ws/stomp");
    stompClient = Stomp.over(socket);

    stompClient.connect(
        { "Authorization": "Bearer " + token },
        function () {
            reconnectDelay = 1000;
            // subscriptions...
        },
        function () {
            console.log("Reconectando em " + reconnectDelay + "ms...");
            setTimeout(connect, reconnectDelay);
            reconnectDelay = Math.min(reconnectDelay * 2, MAX_DELAY);
        }
    );
}
```

### 14.4. Organização do projeto

```text
src/main/java/br/com/exemplo/websocket/
  auth/
    JwtTokenProvider.java
    JwtHandshakeInterceptor.java
    JwtStompInterceptor.java
    UserRole.java
    WebSecurityConfig.java
    WebSocketSecurityConfig.java
  raw/
    ChatRawHandler.java
    ChatRawAuthHandler.java
    ChatRawErrorHandler.java
    ChatRawRateLimitedHandler.java
    RawWebSocketConfig.java
    WebSocketPingService.java
  stomp/
    StompWebSocketConfig.java
    StompHeartbeatConfig.java
    StompErrorConfig.java
    CustomStompErrorHandler.java
    ChatController.java
    ChatControllerWithHistory.java
    MultiRoomChatController.java
    PrivateMessageController.java
    ModerationController.java
    TypingController.java
    PresenceController.java
    ChatNotificationService.java
    PresenceService.java
    ActiveSessionRegistry.java
    WebSocketEventListener.java
    DisconnectCleanupListener.java
    WebSocketExceptionHandler.java
    ChatMessageRequest.java
    ChatMessageResponse.java
    ErrorResponse.java
    TypingEvent.java
    PresenceEvent.java
    PresenceStatus.java
    MessageType.java
  file/
    FileUploadController.java
    FileDownloadController.java
    FileShareNotifier.java
    FileMessageController.java
    FileUploadResponse.java
    FileMessageRequest.java
    FileMessageResponse.java
  history/
    ChatMessageEntity.java
    ChatMessageRepository.java
    ChatHistoryService.java
    ChatHistoryController.java
  ratelimit/
    MessageRateLimiter.java
    RateLimitInterceptor.java
    MessageRateLimitExceededException.java
src/main/resources/
  static/
    chat-stomp.html
    chat-raw.html
src/test/java/br/com/exemplo/websocket/
  RawWebSocketIntegrationTest.java
  StompWebSocketIntegrationTest.java
  MultiClientStompTest.java
  PrivateMessageStompTest.java
```

---

## 15. Anexo — Evolução para web conferência com WebRTC

Esta seção mostra como evoluir o chat deste documento para uma sala de videoconferência com áudio, vídeo e compartilhamento de tela, usando WebRTC para mídia e o WebSocket/STOMP já existente como canal de sinalização.

### 15.1. Visão geral da arquitetura

WebRTC é uma tecnologia de navegador que permite comunicação de áudio, vídeo e dados em tempo real, diretamente entre peers (navegadores), sem que a mídia precise passar pelo servidor da aplicação.

O servidor Spring Boot participa apenas da **sinalização**: troca de ofertas SDP, respostas SDP e candidatos ICE. A mídia flui peer-to-peer.

```text
┌─────────────┐                                          ┌─────────────┐
│ Participante│     SDP offer / answer / ICE candidates   │ Participante│
│      A      │◄──────── STOMP (Spring Boot) ────────────►│      B      │
│             │                                           │             │
│  getUserMedia()                                getUserMedia()         │
│  RTCPeerConnection                        RTCPeerConnection          │
│             │◄═══════ WebRTC (áudio/vídeo P2P) ════════►│             │
└─────────────┘                                          └─────────────┘
```

Conceitos-chave do WebRTC:

- **SDP (Session Description Protocol)**: descreve as capacidades de mídia de cada peer (codecs, resolução, etc.);
- **ICE (Interactive Connectivity Establishment)**: descobre o melhor caminho de rede entre os peers;
- **STUN server**: ajuda peers atrás de NAT a descobrirem seu IP público;
- **TURN server**: relay de mídia quando a conexão direta falha (NATs restritivos, firewalls corporativos);
- **Offer/Answer**: um peer cria uma oferta (offer), o outro responde (answer), ambas trocadas via sinalização.

### 15.2. Topologias: mesh, SFU e MCU

| Topologia | Participantes | Servidor de mídia | Upload do cliente | Download do cliente | Quando usar |
| --- | --- | --- | --- | --- | --- |
| Mesh | 2–5 | nenhum | N-1 streams | N-1 streams | chamadas pequenas, MVP, sem infraestrutura extra |
| SFU | 5–50+ | sim (forwarding) | 1 stream | N-1 streams | reuniões médias/grandes, qualidade adaptativa |
| MCU | 50+ | sim (transcodificação) | 1 stream | 1 stream (mixado) | webinars, muitos participantes com banda limitada |

```text
Mesh (3 participantes):          SFU:                          MCU:
  A ◄══► B                      A ──► SFU ──► A               A ──► MCU ──► A (mix)
  A ◄══► C                      B ──► SFU ──► B               B ──► MCU ──► B (mix)
  B ◄══► C                      C ──► SFU ──► C               C ──► MCU ──► C (mix)
  (6 streams no total)           (cada um envia 1,              (cada um envia 1,
                                  recebe N-1)                    recebe 1 mixado)
```

Esta seção implementa a topologia **mesh** completa com sinalização STOMP e apresenta direcionamentos para SFU e MCU ao final.

### 15.3. DTOs de sinalização

```java
package br.com.exemplo.websocket.webrtc;

public record SignalMessage(
        String type,
        String sender,
        String target,
        String roomId,
        Object payload
) {
}
```

O campo `type` pode ser:

- `offer` — SDP offer de um peer;
- `answer` — SDP answer de um peer;
- `ice-candidate` — candidato ICE descoberto;
- `join-call` — notificação de que um participante entrou na chamada;
- `leave-call` — notificação de saída.

O campo `target` pode ser `null` para broadcast ou conter o username de um peer específico.

### 15.4. Controller de sinalização

```java
package br.com.exemplo.websocket.webrtc;

import java.util.Set;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class WebRtcSignalingController {

    private final SimpMessagingTemplate messagingTemplate;
    private final CallSessionRegistry callRegistry;

    public WebRtcSignalingController(
            SimpMessagingTemplate messagingTemplate,
            CallSessionRegistry callRegistry
    ) {
        this.messagingTemplate = messagingTemplate;
        this.callRegistry = callRegistry;
    }

    @MessageMapping("/signal/{roomId}")
    public void handleSignal(
            @DestinationVariable String roomId,
            SignalMessage message,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String sender = headerAccessor.getUser().getName();

        SignalMessage outgoing = new SignalMessage(
                message.type(), sender, message.target(), roomId, message.payload()
        );

        if (message.target() != null) {
            messagingTemplate.convertAndSendToUser(
                    message.target(), "/queue/signal", outgoing
            );
        } else {
            messagingTemplate.convertAndSend("/topic/signal/" + roomId, outgoing);
        }
    }

    @MessageMapping("/signal/{roomId}/join")
    public void joinCall(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();

        Set<String> existingParticipants = callRegistry.getParticipants(roomId);

        callRegistry.addParticipant(roomId, username);

        messagingTemplate.convertAndSendToUser(
                username, "/queue/signal",
                new SignalMessage("existing-participants", "server", username, roomId, existingParticipants)
        );

        messagingTemplate.convertAndSend("/topic/signal/" + roomId,
                new SignalMessage("join-call", username, null, roomId, null)
        );
    }

    @MessageMapping("/signal/{roomId}/leave")
    public void leaveCall(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        String username = headerAccessor.getUser().getName();
        callRegistry.removeParticipant(roomId, username);

        messagingTemplate.convertAndSend("/topic/signal/" + roomId,
                new SignalMessage("leave-call", username, null, roomId, null)
        );
    }
}
```

### 15.5. Registro de participantes na chamada

```java
package br.com.exemplo.websocket.webrtc;

import java.util.Collections;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import org.springframework.stereotype.Component;

@Component
public class CallSessionRegistry {

    private final Map<String, Set<String>> callParticipants = new ConcurrentHashMap<>();

    public void addParticipant(String roomId, String username) {
        callParticipants.computeIfAbsent(roomId, k -> ConcurrentHashMap.newKeySet()).add(username);
    }

    public void removeParticipant(String roomId, String username) {
        Set<String> participants = callParticipants.get(roomId);
        if (participants != null) {
            participants.remove(username);
            if (participants.isEmpty()) {
                callParticipants.remove(roomId);
            }
        }
    }

    public Set<String> getParticipants(String roomId) {
        return callParticipants.getOrDefault(roomId, Collections.emptySet());
    }

    public void disconnectUser(String username) {
        for (Map.Entry<String, Set<String>> entry : callParticipants.entrySet()) {
            entry.getValue().remove(username);
            if (entry.getValue().isEmpty()) {
                callParticipants.remove(entry.getKey());
            }
        }
    }
}
```

### 15.6. Controle de permissões por papel na chamada

```java
package br.com.exemplo.websocket.webrtc;

import br.com.exemplo.websocket.auth.UserRole;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class CallModerationController {

    private final SimpMessagingTemplate messagingTemplate;

    public CallModerationController(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    @MessageMapping("/signal/{roomId}/mute-participant")
    public void muteParticipant(
            @DestinationVariable String roomId,
            MuteParticipantRequest request,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        UserRole role = UserRole.valueOf(
                (String) headerAccessor.getSessionAttributes().get("role")
        );

        if (role != UserRole.MODERADOR && role != UserRole.ADMIN) {
            messagingTemplate.convertAndSendToUser(
                    headerAccessor.getUser().getName(), "/queue/errors",
                    "Apenas moderadores podem silenciar participantes"
            );
            return;
        }

        messagingTemplate.convertAndSendToUser(
                request.targetUser(), "/queue/signal",
                new SignalMessage("force-mute", headerAccessor.getUser().getName(),
                        request.targetUser(), roomId,
                        new MutePayload(request.muteAudio(), request.muteVideo()))
        );
    }

    @MessageMapping("/signal/{roomId}/end-call")
    public void endCall(
            @DestinationVariable String roomId,
            SimpMessageHeaderAccessor headerAccessor
    ) {
        UserRole role = UserRole.valueOf(
                (String) headerAccessor.getSessionAttributes().get("role")
        );

        if (role != UserRole.ADMIN) {
            return;
        }

        messagingTemplate.convertAndSend("/topic/signal/" + roomId,
                new SignalMessage("end-call", headerAccessor.getUser().getName(), null, roomId, null)
        );
    }
}
```

```java
package br.com.exemplo.websocket.webrtc;

public record MuteParticipantRequest(String targetUser, boolean muteAudio, boolean muteVideo) {
}
```

```java
package br.com.exemplo.websocket.webrtc;

public record MutePayload(boolean muteAudio, boolean muteVideo) {
}
```

### 15.7. Interface web — videoconferência com WebRTC mesh

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Videoconferência — WebRTC + STOMP</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: sans-serif; background: #1a1a2e; color: #eee; }

        #controls {
            display: flex; gap: 8px; justify-content: center;
            padding: 12px; background: #16213e;
        }
        #controls button {
            padding: 10px 20px; border: none; border-radius: 6px;
            cursor: pointer; font-size: 14px; color: white;
        }
        .btn-on { background: #27ae60; }
        .btn-off { background: #c0392b; }
        .btn-action { background: #2980b9; }
        .btn-danger { background: #e74c3c; }

        #video-grid {
            display: grid; gap: 8px; padding: 16px;
            grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
        }
        .video-container {
            position: relative; background: #0f3460;
            border-radius: 8px; overflow: hidden; aspect-ratio: 16/9;
        }
        .video-container video {
            width: 100%; height: 100%; object-fit: cover;
        }
        .video-label {
            position: absolute; bottom: 8px; left: 8px;
            background: rgba(0,0,0,0.6); padding: 4px 10px;
            border-radius: 4px; font-size: 13px;
        }
        .role-tag {
            font-size: 0.7em; padding: 1px 5px; border-radius: 3px;
            color: white; margin-left: 4px;
        }
        .role-ADMIN { background: #e74c3c; }
        .role-MODERADOR { background: #e67e22; }
        .role-COMUM { background: #95a5a6; }
    </style>
</head>
<body>

<div id="controls">
    <button id="btn-mic" class="btn-on" onclick="toggleAudio()">🎤 Mic</button>
    <button id="btn-cam" class="btn-on" onclick="toggleVideo()">📹 Cam</button>
    <button id="btn-screen" class="btn-action" onclick="shareScreen()">🖥️ Tela</button>
    <button id="btn-join" class="btn-action" onclick="joinCall()">📞 Entrar</button>
    <button id="btn-leave" class="btn-danger" onclick="leaveCall()">❌ Sair</button>
</div>

<div id="video-grid"></div>

<script src="https://cdn.jsdelivr.net/npm/sockjs-client@1/dist/sockjs.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/stompjs@2.3.3/lib/stomp.min.js"></script>
<script>
    const token = prompt("Informe seu JWT:");
    const roomId = prompt("ID da sala:", "sala-1");

    const ICE_SERVERS = [
        { urls: "stun:stun.l.google.com:19302" },
        { urls: "stun:stun1.l.google.com:19302" }
    ];

    let stompClient = null;
    let localStream = null;
    let screenStream = null;
    let audioEnabled = true;
    let videoEnabled = true;

    const peerConnections = {};
    const videoGrid = document.getElementById("video-grid");

    // ── Conexão STOMP ──

    function connect() {
        const socket = new SockJS("/ws/stomp");
        stompClient = Stomp.over(socket);
        stompClient.debug = null;

        stompClient.connect(
            { "Authorization": "Bearer " + token },
            function () {
                stompClient.subscribe("/topic/signal/" + roomId, function (msg) {
                    handleSignal(JSON.parse(msg.body));
                });

                stompClient.subscribe("/user/queue/signal", function (msg) {
                    handleSignal(JSON.parse(msg.body));
                });

                initLocalMedia();
            }
        );
    }

    // ── Mídia local ──

    function initLocalMedia() {
        navigator.mediaDevices.getUserMedia({ audio: true, video: true })
            .then(function (stream) {
                localStream = stream;
                addVideoElement("local", stream, "Você", true);
            })
            .catch(function (err) {
                console.error("Erro ao acessar mídia:", err);
                navigator.mediaDevices.getUserMedia({ audio: true, video: false })
                    .then(function (stream) {
                        localStream = stream;
                        addVideoElement("local", stream, "Você (sem câmera)", true);
                    });
            });
    }

    // ── Entrar/sair da chamada ──

    function joinCall() {
        if (!localStream) { alert("Mídia local não inicializada"); return; }
        stompClient.send("/app/signal/" + roomId + "/join", {}, "{}");
    }

    function leaveCall() {
        stompClient.send("/app/signal/" + roomId + "/leave", {}, "{}");
        for (const peerId in peerConnections) {
            peerConnections[peerId].close();
            removeVideoElement(peerId);
            delete peerConnections[peerId];
        }
    }

    // ── Sinalização ──

    function handleSignal(msg) {
        switch (msg.type) {
            case "existing-participants":
                var participants = msg.payload;
                if (Array.isArray(participants)) {
                    participants.forEach(function (p) { createPeerAndOffer(p); });
                }
                break;

            case "join-call":
                if (msg.sender !== getMyUsername()) {
                    createPeerAndOffer(msg.sender);
                }
                break;

            case "offer":
                handleOffer(msg.sender, msg.payload);
                break;

            case "answer":
                handleAnswer(msg.sender, msg.payload);
                break;

            case "ice-candidate":
                handleIceCandidate(msg.sender, msg.payload);
                break;

            case "leave-call":
                if (peerConnections[msg.sender]) {
                    peerConnections[msg.sender].close();
                    removeVideoElement(msg.sender);
                    delete peerConnections[msg.sender];
                }
                break;

            case "force-mute":
                if (msg.payload.muteAudio) { muteLocalAudio(true); }
                if (msg.payload.muteVideo) { muteLocalVideo(true); }
                break;

            case "end-call":
                leaveCall();
                alert("A chamada foi encerrada pelo administrador.");
                break;
        }
    }

    // ── WebRTC — gerenciamento de peers ──

    function createPeerConnection(peerId) {
        var pc = new RTCPeerConnection({ iceServers: ICE_SERVERS });

        localStream.getTracks().forEach(function (track) {
            pc.addTrack(track, localStream);
        });

        pc.onicecandidate = function (event) {
            if (event.candidate) {
                stompClient.send("/app/signal/" + roomId, {},
                    JSON.stringify({
                        type: "ice-candidate",
                        target: peerId,
                        payload: event.candidate
                    })
                );
            }
        };

        pc.ontrack = function (event) {
            var existingVideo = document.getElementById("video-" + peerId);
            if (!existingVideo) {
                addVideoElement(peerId, event.streams[0], peerId, false);
            }
        };

        pc.onconnectionstatechange = function () {
            if (pc.connectionState === "disconnected" || pc.connectionState === "failed") {
                pc.close();
                removeVideoElement(peerId);
                delete peerConnections[peerId];
            }
        };

        peerConnections[peerId] = pc;
        return pc;
    }

    function createPeerAndOffer(peerId) {
        var pc = createPeerConnection(peerId);
        pc.createOffer()
            .then(function (offer) { return pc.setLocalDescription(offer); })
            .then(function () {
                stompClient.send("/app/signal/" + roomId, {},
                    JSON.stringify({
                        type: "offer",
                        target: peerId,
                        payload: peerConnections[peerId].localDescription
                    })
                );
            });
    }

    function handleOffer(senderId, offer) {
        var pc = createPeerConnection(senderId);
        pc.setRemoteDescription(new RTCSessionDescription(offer))
            .then(function () { return pc.createAnswer(); })
            .then(function (answer) { return pc.setLocalDescription(answer); })
            .then(function () {
                stompClient.send("/app/signal/" + roomId, {},
                    JSON.stringify({
                        type: "answer",
                        target: senderId,
                        payload: pc.localDescription
                    })
                );
            });
    }

    function handleAnswer(senderId, answer) {
        var pc = peerConnections[senderId];
        if (pc) {
            pc.setRemoteDescription(new RTCSessionDescription(answer));
        }
    }

    function handleIceCandidate(senderId, candidate) {
        var pc = peerConnections[senderId];
        if (pc) {
            pc.addIceCandidate(new RTCIceCandidate(candidate));
        }
    }

    // ── Controles de áudio/vídeo ──

    function toggleAudio() {
        audioEnabled = !audioEnabled;
        muteLocalAudio(!audioEnabled);
        var btn = document.getElementById("btn-mic");
        btn.className = audioEnabled ? "btn-on" : "btn-off";
        btn.textContent = audioEnabled ? "🎤 Mic" : "🔇 Mic";
    }

    function toggleVideo() {
        videoEnabled = !videoEnabled;
        muteLocalVideo(!videoEnabled);
        var btn = document.getElementById("btn-cam");
        btn.className = videoEnabled ? "btn-on" : "btn-off";
        btn.textContent = videoEnabled ? "📹 Cam" : "📷 Cam";
    }

    function muteLocalAudio(mute) {
        if (localStream) {
            localStream.getAudioTracks().forEach(function (t) { t.enabled = !mute; });
        }
        audioEnabled = !mute;
    }

    function muteLocalVideo(mute) {
        if (localStream) {
            localStream.getVideoTracks().forEach(function (t) { t.enabled = !mute; });
        }
        videoEnabled = !mute;
    }

    // ── Compartilhamento de tela ──

    function shareScreen() {
        navigator.mediaDevices.getDisplayMedia({ video: true })
            .then(function (stream) {
                screenStream = stream;
                var screenTrack = stream.getVideoTracks()[0];

                for (var peerId in peerConnections) {
                    var pc = peerConnections[peerId];
                    var videoSender = pc.getSenders().find(function (s) {
                        return s.track && s.track.kind === "video";
                    });
                    if (videoSender) {
                        videoSender.replaceTrack(screenTrack);
                    }
                }

                screenTrack.onended = function () { stopScreenShare(); };
            });
    }

    function stopScreenShare() {
        if (!localStream) return;
        var cameraTrack = localStream.getVideoTracks()[0];
        for (var peerId in peerConnections) {
            var pc = peerConnections[peerId];
            var videoSender = pc.getSenders().find(function (s) {
                return s.track && s.track.kind === "video";
            });
            if (videoSender && cameraTrack) {
                videoSender.replaceTrack(cameraTrack);
            }
        }
        if (screenStream) {
            screenStream.getTracks().forEach(function (t) { t.stop(); });
            screenStream = null;
        }
    }

    // ── UI helpers ──

    function addVideoElement(id, stream, label, muted) {
        var container = document.createElement("div");
        container.className = "video-container";
        container.id = "video-" + id;

        var video = document.createElement("video");
        video.srcObject = stream;
        video.autoplay = true;
        video.playsInline = true;
        if (muted) video.muted = true;

        var labelDiv = document.createElement("div");
        labelDiv.className = "video-label";
        labelDiv.textContent = label;

        container.appendChild(video);
        container.appendChild(labelDiv);
        videoGrid.appendChild(container);
    }

    function removeVideoElement(id) {
        var el = document.getElementById("video-" + id);
        if (el) el.remove();
    }

    function getMyUsername() {
        try {
            var parts = token.split(".");
            var payload = JSON.parse(atob(parts[1]));
            return payload.sub;
        } catch (e) {
            return "unknown";
        }
    }

    connect();
</script>
</body>
</html>
```

Essa página oferece:

- grid responsivo de vídeos que cresce conforme entram participantes;
- botões para mutar microfone, desligar câmera, compartilhar tela, entrar e sair;
- sinalização via STOMP reutilizando a infraestrutura de WebSocket já existente;
- troca automática entre câmera e compartilhamento de tela via `replaceTrack`;
- tratamento de `force-mute` e `end-call` enviados por moderadores/admins;
- detecção de desconexão de peers via `onconnectionstatechange`.

### 15.8. Configuração de STUN e TURN

Para funcionar fora de redes locais, é necessário configurar servidores STUN e TURN:

```javascript
const ICE_SERVERS = [
    { urls: "stun:stun.l.google.com:19302" },
    {
        urls: "turn:turn.exemplo.com.br:3478",
        username: "usuario",
        credential: "senha"
    }
];
```

- **STUN** é gratuito e leve — o Google disponibiliza servidores públicos para descoberta de IP;
- **TURN** é obrigatório quando peers estão atrás de NATs restritivos ou firewalls corporativos — ele faz relay da mídia, portanto consome banda;
- em produção, soluções comuns para TURN são **coturn** (open source, self-hosted) ou serviços gerenciados como Twilio Network Traversal ou Xirsys.

O servidor TURN pode ser provisionado com credenciais temporárias geradas pelo backend Spring Boot:

```java
package br.com.exemplo.websocket.webrtc;

import java.time.Instant;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/webrtc")
public class IceServerController {

    @GetMapping("/ice-servers")
    public IceServersResponse getIceServers() {
        long ttl = 86400;
        long expiry = Instant.now().getEpochSecond() + ttl;
        String username = expiry + ":webrtc-user";
        String credential = generateHmac(username);

        return new IceServersResponse(
                "stun:stun.l.google.com:19302",
                "turn:turn.exemplo.com.br:3478",
                username,
                credential,
                ttl
        );
    }

    private String generateHmac(String username) {
        // HMAC-SHA1 com shared secret do coturn
        return "credencial-calculada";
    }
}
```

```java
package br.com.exemplo.websocket.webrtc;

public record IceServersResponse(
        String stunUrl,
        String turnUrl,
        String username,
        String credential,
        long ttl
) {
}
```

### 15.9. Limitações do mesh e quando migrar

A topologia mesh funciona bem para **2 a 5 participantes**. Acima disso, a degradação é significativa:

| Participantes | Streams de upload | Streams de download | Total por cliente |
| --- | --- | --- | --- |
| 2 | 1 | 1 | 2 |
| 3 | 2 | 2 | 4 |
| 5 | 4 | 4 | 8 |
| 10 | 9 | 9 | 18 |

Cada stream adicional consome CPU (encoding), memória e banda. Em dispositivos móveis ou conexões limitadas, o limite prático é ainda menor.

Sinais de que é hora de migrar para SFU ou MCU:

- qualidade de vídeo cai com mais de 4 participantes;
- dispositivos móveis apresentam superaquecimento ou travamento;
- usuários reportam atraso ou congelamento de vídeo;
- a aplicação precisa suportar reuniões com 10+ pessoas regularmente.

### 15.10. Direcionamento para SFU (Selective Forwarding Unit)

Na topologia SFU, cada participante envia **um único stream** para o servidor, e o servidor o redistribui para os demais sem transcodificar. Isso reduz drasticamente o upload de cada cliente.

#### Soluções SFU recomendadas

| Solução | Linguagem | Destaques | Integração com Spring Boot |
| --- | --- | --- | --- |
| **LiveKit** | Go | open source, SDKs para Java e JS, self-hosted ou cloud, suporte a simulcast e gravação | SDK Java para gerar tokens de acesso às salas; Spring Boot gerencia autenticação e salas, LiveKit gerencia mídia |
| **mediasoup** | Node.js (C++ core) | muito popular, alta performance, flexível | serviço separado; Spring Boot como API principal, mediasoup como serviço de mídia acessado via API |
| **Janus** | C | leve, extensível por plugins, bem documentado | plugin de videoroom; Spring Boot se comunica via API HTTP/WebSocket do Janus |
| **OpenVidu** | Java | baseado em Kurento, interface de admin, orientado a empresas | integração nativa com Spring Boot via REST API e SDK Java |

#### Exemplo conceitual com LiveKit

O backend Spring Boot gera tokens de acesso para a sala:

```xml
<dependency>
    <groupId>io.livekit</groupId>
    <artifactId>livekit-server</artifactId>
    <version>0.7.1</version>
</dependency>
```

```java
package br.com.exemplo.websocket.webrtc;

import io.livekit.server.AccessToken;
import io.livekit.server.RoomJoin;
import io.livekit.server.RoomName;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/webrtc/livekit")
public class LiveKitTokenController {

    @Value("${livekit.api-key}")
    private String apiKey;

    @Value("${livekit.api-secret}")
    private String apiSecret;

    @GetMapping("/token")
    public LiveKitTokenResponse generateToken(
            @RequestParam String roomId,
            @RequestParam String username,
            @RequestParam String role
    ) {
        boolean canPublish = true;
        boolean canSubscribe = true;
        boolean isAdmin = "ADMIN".equals(role) || "MODERADOR".equals(role);

        AccessToken token = new AccessToken(apiKey, apiSecret);
        token.setName(username);
        token.setIdentity(username);
        token.addGrants(new RoomJoin(true), new RoomName(roomId));
        token.setMetadata("{\"role\":\"" + role + "\"}");

        return new LiveKitTokenResponse(token.toJwt(), roomId);
    }
}
```

```java
package br.com.exemplo.websocket.webrtc;

public record LiveKitTokenResponse(String token, String roomId) {
}
```

No frontend, o LiveKit JS SDK substitui o WebRTC manual:

```javascript
// import { Room, RoomEvent } from 'livekit-client';

async function joinLiveKitRoom(token, livekitUrl) {
    const room = new Room();

    room.on(RoomEvent.TrackSubscribed, function (track, publication, participant) {
        const element = track.attach();
        document.getElementById("video-grid").appendChild(element);
    });

    room.on(RoomEvent.TrackUnsubscribed, function (track) {
        track.detach().forEach(function (el) { el.remove(); });
    });

    await room.connect(livekitUrl, token);
    await room.localParticipant.enableCameraAndMicrophone();
}
```

Nesse modelo:

- o Spring Boot cuida de autenticação JWT, autorização por papel e geração de tokens LiveKit;
- o LiveKit cuida de toda a mídia: recebe, redistribui, adapta qualidade (simulcast);
- o chat textual via STOMP continua funcionando em paralelo;
- a topologia mesh é completamente substituída pelo SFU.

### 15.11. Direcionamento para MCU (Multipoint Control Unit)

Na topologia MCU, o servidor recebe todos os streams, **transcodifica e mixa** em um único stream composto, e envia apenas esse stream para cada participante.

```text
Participantes:                MCU:                        Saída:
  A (vídeo+áudio) ──►┐                               ┌──► A recebe mix de B+C+D
  B (vídeo+áudio) ──►├──► MCU processa e mixa ────────├──► B recebe mix de A+C+D
  C (vídeo+áudio) ──►│    (transcodificação pesada)   ├──► C recebe mix de A+B+D
  D (vídeo+áudio) ──►┘                               └──► D recebe mix de A+B+C
```

#### Vantagens e desvantagens

| Aspecto | Vantagem | Desvantagem |
| --- | --- | --- |
| Banda do cliente | mínima — cada um envia 1 e recebe 1 stream | — |
| CPU do cliente | mínima — decodifica apenas 1 stream | — |
| CPU do servidor | — | muito alta — transcodifica e compõe N streams |
| Custo de infra | — | alto — requer hardware dedicado ou GPUs |
| Latência | — | maior que SFU — mixagem adiciona delay |
| Flexibilidade de layout | servidor decide layout do grid | cliente tem pouco controle sobre apresentação |

#### Quando preferir MCU

- participantes com banda ou dispositivos muito limitados (ex: celulares em redes 3G);
- webinars com muitos espectadores que precisam de um único stream;
- gravação server-side onde o resultado já precisa estar mixado;
- cenários onde o cliente não pode decodificar múltiplos streams simultaneamente.

#### Soluções MCU

| Solução | Tipo | Destaques |
| --- | --- | --- |
| **Kurento** | MCU + SFU | Java, open source, suporta pipelines de mídia, filtros, gravação e composição |
| **OpenVidu** | Plataforma sobre Kurento | API REST e SDK Java para Spring Boot, dashboard de admin, orientado a empresas |
| **Jitsi Meet** | SFU com composição opcional | open source, self-hosted, composição via Jibri para gravação |
| **FreeSWITCH** | MCU | C, maduro, muito usado em telefonia e conferência, suporte a SIP |

#### Exemplo conceitual de integração com Kurento/OpenVidu

O OpenVidu expõe uma REST API que o Spring Boot pode consumir:

```java
package br.com.exemplo.websocket.webrtc;

import java.util.Map;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class OpenViduService {

    private final RestClient restClient;

    public OpenViduService(
            @Value("${openvidu.url}") String openviduUrl,
            @Value("${openvidu.secret}") String secret
    ) {
        this.restClient = RestClient.builder()
                .baseUrl(openviduUrl)
                .defaultHeader(HttpHeaders.AUTHORIZATION,
                        "Basic " + java.util.Base64.getEncoder()
                                .encodeToString(("OPENVIDUAPP:" + secret).getBytes()))
                .build();
    }

    public Map<String, Object> createSession(String roomId) {
        return restClient.post()
                .uri("/openvidu/api/sessions")
                .contentType(MediaType.APPLICATION_JSON)
                .body(Map.of("customSessionId", roomId))
                .retrieve()
                .body(Map.class);
    }

    public Map<String, Object> generateToken(String sessionId, String role) {
        String openViduRole = switch (role) {
            case "ADMIN" -> "MODERATOR";
            case "MODERADOR" -> "MODERATOR";
            default -> "PUBLISHER";
        };

        return restClient.post()
                .uri("/openvidu/api/sessions/{sessionId}/connection", sessionId)
                .contentType(MediaType.APPLICATION_JSON)
                .body(Map.of("role", openViduRole))
                .retrieve()
                .body(Map.class);
    }
}
```

```java
package br.com.exemplo.websocket.webrtc;

import java.util.Map;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/webrtc/openvidu")
public class OpenViduController {

    private final OpenViduService openViduService;

    public OpenViduController(OpenViduService openViduService) {
        this.openViduService = openViduService;
    }

    @PostMapping("/session")
    public Map<String, Object> createSession(@RequestParam String roomId) {
        return openViduService.createSession(roomId);
    }

    @PostMapping("/token")
    public Map<String, Object> getToken(
            @RequestParam String sessionId,
            @RequestParam String role
    ) {
        return openViduService.generateToken(sessionId, role);
    }
}
```

#### Resumo comparativo final

| Critério | Mesh | SFU | MCU |
| --- | --- | --- | --- |
| Participantes | 2–5 | 5–50+ | 10–100+ |
| Infraestrutura extra | nenhuma | servidor SFU | servidor MCU com alta CPU |
| Upload do cliente | N-1 streams | 1 stream | 1 stream |
| Download do cliente | N-1 streams | N-1 streams | 1 stream (mixado) |
| Latência | baixa | baixa | média |
| Custo | zero | moderado | alto |
| Complexidade | baixa | média | alta |
| Integração com Spring Boot | sinalização via STOMP | API REST para tokens | API REST para sessões e tokens |

Para a maioria dos projetos, a progressão recomendada é:

1. começar com **mesh** para validar o produto;
2. migrar para **SFU** (LiveKit ou mediasoup) quando o número de participantes crescer;
3. considerar **MCU** apenas quando houver requisito específico de banda mínima no cliente ou gravação mixada.

Em todos os cenários, o Spring Boot continua como servidor de aplicação principal — gerenciando autenticação, autorização por papéis, salas, chat textual e integração com o serviço de mídia.

---

## 16. Referências

### Spring Framework e Spring Boot

- Spring WebSocket Reference: https://docs.spring.io/spring-framework/reference/web/websocket.html
- Spring WebSocket STOMP: https://docs.spring.io/spring-framework/reference/web/websocket/stomp.html
- Spring WebSocket Messaging: https://docs.spring.io/spring-framework/reference/web/websocket/stomp/handle-annotations.html
- Spring Security WebSocket: https://docs.spring.io/spring-security/reference/servlet/integrations/websocket.html
- Spring Boot WebSocket: https://docs.spring.io/spring-boot/reference/messaging/websockets.html
- Spring WebSocket Testing: https://docs.spring.io/spring-framework/reference/web/websocket/stomp/testing.html

### Rate limiting

- Bucket4j: https://bucket4j.com/

### Protocolos e especificações

- RFC 6455 — The WebSocket Protocol: https://datatracker.ietf.org/doc/html/rfc6455
- STOMP Protocol Specification: https://stomp.github.io/stomp-specification-1.2.html
- WebRTC — MDN Web Docs: https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API
- WebRTC — Getting Started: https://webrtc.org/getting-started/overview
- RTCPeerConnection — MDN: https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection
- getUserMedia — MDN: https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia
- getDisplayMedia — MDN: https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia

### STUN e TURN

- coturn (servidor TURN open source): https://github.com/coturn/coturn

### SFU e MCU

- LiveKit: https://livekit.io/
- LiveKit Server SDK (Java): https://github.com/livekit/server-sdk-java
- mediasoup: https://mediasoup.org/
- Janus WebRTC Server: https://janus.conf.meetecho.com/
- OpenVidu: https://openvidu.io/
- Kurento Media Server: https://kurento.openvidu.io/
- Jitsi Meet: https://jitsi.org/jitsi-meet/
- FreeSWITCH: https://freeswitch.com/

### Bibliotecas cliente

- SockJS Client: https://github.com/sockjs/sockjs-client
- STOMP.js: https://github.com/stomp-js/stompjs

### JWT

- Nimbus JOSE+JWT: https://connect2id.com/products/nimbus-jose-jwt
- Nimbus JOSE+JWT — Javadoc: https://www.javadoc.io/doc/com.nimbusds/nimbus-jose-jwt/latest/index.html
- Nimbus JOSE+JWT — Maven Central: https://mvnrepository.com/artifact/com.nimbusds/nimbus-jose-jwt

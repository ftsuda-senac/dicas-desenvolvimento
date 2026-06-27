# Biometria com Java/Spring Boot e React — Captura, Armazenamento Seguro e Autenticação (LGPD)

> **Versões de referência:** Spring Boot 3.5.x · Spring Security 6.x · Java 21+ · React 19 · TypeScript 5.x · PostgreSQL 16+

> **Objetivo:** Apresentar uma arquitetura completa para captura de dados biométricos (reconhecimento facial e impressão digital), armazenamento seguro e autenticação biométrica, em conformidade com a Lei Geral de Proteção de Dados (LGPD — Lei 13.709/2018).

---

## Sumário

1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Conformidade com a LGPD](#2-conformidade-com-a-lgpd)
   - [2.1 Bases Legais e Princípios](#21-bases-legais-e-princípios)
   - [2.2 Consentimento Explícito](#22-consentimento-explícito)
   - [2.3 Relatório de Impacto à Proteção de Dados (RIPD)](#23-relatório-de-impacto-à-proteção-de-dados-ripd)
   - [2.4 Direitos do Titular](#24-direitos-do-titular)
3. [Modelagem de Dados](#3-modelagem-de-dados)
   - [3.1 Esquema de Banco de Dados](#31-esquema-de-banco-de-dados)
   - [3.2 Entidades JPA](#32-entidades-jpa)
4. [Criptografia e Armazenamento Seguro dos Templates Biométricos](#4-criptografia-e-armazenamento-seguro-dos-templates-biométricos)
   - [4.1 Estratégia de Criptografia](#41-estratégia-de-criptografia)
   - [4.2 Serviço de Criptografia](#42-serviço-de-criptografia)
5. [Captura de Face — Frontend React](#5-captura-de-face--frontend-react)
   - [5.1 Componente de Captura Facial](#51-componente-de-captura-facial)
   - [5.1.1 Alternativa: Extração do Template Facial no Backend (Java + OpenCV DNN)](#511-alternativa-extração-do-template-facial-no-backend-java--opencv-dnn)
   - [5.2 Componente de Captura de Impressão Digital (WebAuthn)](#52-componente-de-captura-de-impressão-digital-webauthn)
6. [Backend — API REST para Cadastro Biométrico](#6-backend--api-rest-para-cadastro-biométrico)
   - [6.1 Configuração e Dependências](#61-configuração-e-dependências)
   - [6.2 DTOs](#62-dtos)
   - [6.3 Controller de Cadastro Biométrico](#63-controller-de-cadastro-biométrico)
   - [6.4 Serviço de Cadastro Facial](#64-serviço-de-cadastro-facial)
   - [6.5 Serviço de Cadastro de Impressão Digital (FIDO2/WebAuthn)](#65-serviço-de-cadastro-de-impressão-digital-fido2webauthn)
7. [Autenticação Biométrica](#7-autenticação-biométrica)
   - [7.1 Fluxo de Autenticação Facial](#71-fluxo-de-autenticação-facial)
   - [7.2 Fluxo de Autenticação por Impressão Digital (WebAuthn)](#72-fluxo-de-autenticação-por-impressão-digital-webauthn)
   - [7.3 Integração com Spring Security](#73-integração-com-spring-security)
8. [Liveness Detection — Prova de Vida](#8-liveness-detection--prova-de-vida)
9. [Auditoria e Logs de Acesso](#9-auditoria-e-logs-de-acesso)
10. [Boas Práticas e Checklist de Segurança](#10-boas-práticas-e-checklist-de-segurança)

---

## 1. Visão Geral da Arquitetura

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (React)                      │
│  ┌──────────────┐  ┌────────────────┐  ┌─────────────┐  │
│  │ Câmera/WebRTC│  │ WebAuthn API   │  │ Tela de     │  │
│  │ (Captura     │  │ (Impressão     │  │ Consentimento│  │
│  │  Facial)     │  │  Digital)      │  │ LGPD        │  │
│  └──────┬───────┘  └───────┬────────┘  └──────┬──────┘  │
│         │ HTTPS            │ HTTPS             │         │
└─────────┼──────────────────┼──────────────────┼─────────┘
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────┐
│               Backend (Spring Boot)                      │
│  ┌──────────────┐  ┌────────────────┐  ┌─────────────┐  │
│  │ Face Service  │  │ FIDO2/WebAuthn │  │ Consent     │  │
│  │ (Extração de │  │ Service        │  │ Service     │  │
│  │  Template)   │  │                │  │ (LGPD)      │  │
│  └──────┬───────┘  └───────┬────────┘  └──────┬──────┘  │
│         │                  │                   │         │
│         ▼                  ▼                   ▼         │
│  ┌─────────────────────────────────────────────────────┐ │
│  │          Camada de Criptografia (AES-256-GCM)       │ │
│  └──────────────────────┬──────────────────────────────┘ │
│                         │                                │
│  ┌──────────────┐  ┌────┴────────┐  ┌─────────────────┐ │
│  │ Audit Log    │  │ PostgreSQL  │  │ HSM / Vault     │ │
│  │ Service      │  │ (Templates  │  │ (Chaves         │ │
│  │              │  │  cifrados)  │  │  criptográficas)│ │
│  └──────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

**Princípios da arquitetura:**

| Princípio | Descrição |
|---|---|
| **Dados biométricos nunca são armazenados como imagem** | Apenas **templates matemáticos** (vetores de características) são armazenados, não fotos ou impressões digitais brutas |
| **Criptografia em repouso** | Templates são cifrados com AES-256-GCM antes de persistir no banco |
| **Criptografia em trânsito** | Toda comunicação usa HTTPS/TLS 1.3 |
| **Minimização de dados (LGPD Art. 6°, III)** | Coleta-se apenas o necessário para a finalidade declarada |
| **Separação de responsabilidades** | Chaves criptográficas ficam em HSM/Vault, nunca no banco de dados |

---

## 2. Conformidade com a LGPD

### 2.1 Bases Legais e Princípios

Dados biométricos são classificados como **dados pessoais sensíveis** (Art. 5°, II da LGPD). Seu tratamento exige:

| Requisito LGPD | Artigo | Implementação no Sistema |
|---|---|---|
| **Consentimento explícito e específico** | Art. 11, I | Tela dedicada com opt-in granular antes da captura |
| **Finalidade determinada** | Art. 6°, I | Declarar: "autenticação biométrica no sistema X" |
| **Necessidade / Minimização** | Art. 6°, III | Armazenar apenas templates, descartar imagens brutas |
| **Segurança** | Art. 6°, VII e Art. 46 | Criptografia AES-256-GCM, controle de acesso, auditoria |
| **Não discriminação** | Art. 6°, IX | Testes de viés em algoritmos de reconhecimento facial |
| **Responsabilização** | Art. 6°, X | Logs de auditoria, RIPD documentado |
| **Relatório de Impacto (RIPD)** | Art. 38 | Documento obrigatório para tratamento de dados sensíveis |
| **Direito de eliminação** | Art. 18, VI | Endpoint para exclusão completa dos dados biométricos |

### 2.2 Consentimento Explícito

```java
@Entity
@Table(name = "biometric_consent")
public class BiometricConsent {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Column(nullable = false)
    @Enumerated(EnumType.STRING)
    private BiometricType type; // FACE, FINGERPRINT

    @Column(nullable = false)
    private String purpose; // ex: "Autenticação biométrica no sistema AcmeCorp"

    @Column(name = "consent_text", nullable = false, length = 2000)
    private String consentText; // texto completo apresentado ao titular

    @Column(name = "consented_at", nullable = false)
    private Instant consentedAt;

    @Column(name = "revoked_at")
    private Instant revokedAt;

    @Column(name = "ip_address", nullable = false, length = 45)
    private String ipAddress;

    @Column(name = "user_agent", nullable = false, length = 500)
    private String userAgent;

    @Column(name = "consent_version", nullable = false)
    private Integer consentVersion;
}
```

```java
public enum BiometricType {
    FACE,
    FINGERPRINT
}
```

**Componente React de consentimento:**

```tsx
interface ConsentFormProps {
  biometricType: "FACE" | "FINGERPRINT";
  onConsent: (consentData: ConsentPayload) => void;
  onDecline: () => void;
}

const CONSENT_TEXT_FACE = `Autorizo a coleta e o tratamento dos meus dados biométricos 
faciais exclusivamente para fins de autenticação neste sistema. Estou ciente de que:
• Apenas um template matemático será armazenado, não minha foto
• Posso revogar este consentimento a qualquer momento
• Meus dados serão eliminados mediante solicitação
• O tratamento segue a Lei 13.709/2018 (LGPD)`;

const CONSENT_TEXT_FINGERPRINT = `Autorizo a coleta e o tratamento dos meus dados biométricos 
de impressão digital exclusivamente para fins de autenticação neste sistema. Estou ciente de que:
• Será utilizado o padrão FIDO2/WebAuthn para registro seguro
• A chave privada permanece no meu dispositivo
• Posso revogar este consentimento a qualquer momento
• O tratamento segue a Lei 13.709/2018 (LGPD)`;

function BiometricConsentForm({ biometricType, onConsent, onDecline }: ConsentFormProps) {
  const [accepted, setAccepted] = useState(false);
  const consentText = biometricType === "FACE" ? CONSENT_TEXT_FACE : CONSENT_TEXT_FINGERPRINT;

  const handleConsent = () => {
    onConsent({
      type: biometricType,
      consentText,
      consentedAt: new Date().toISOString(),
      consentVersion: 1,
    });
  };

  return (
    <div className="consent-form">
      <h2>Consentimento para Coleta de Dados Biométricos</h2>
      <div className="consent-text-box">
        <p style={{ whiteSpace: "pre-line" }}>{consentText}</p>
      </div>
      <label className="consent-checkbox">
        <input
          type="checkbox"
          checked={accepted}
          onChange={(e) => setAccepted(e.target.checked)}
        />
        Li e concordo com os termos acima
      </label>
      <div className="consent-actions">
        <button onClick={onDecline} className="btn-secondary">Não autorizo</button>
        <button onClick={handleConsent} disabled={!accepted} className="btn-primary">
          Autorizo a coleta
        </button>
      </div>
    </div>
  );
}
```

### 2.3 Relatório de Impacto à Proteção de Dados (RIPD)

O RIPD é obrigatório para tratamento de dados sensíveis. Estrutura mínima:

| Seção | Conteúdo |
|---|---|
| **Descrição do tratamento** | Captura e armazenamento de templates biométricos para autenticação |
| **Necessidade e proporcionalidade** | Justificar por que autenticação biométrica é necessária vs. alternativas menos invasivas |
| **Riscos identificados** | Vazamento de templates, falso positivo/negativo, viés algorítmico, uso indevido |
| **Medidas mitigatórias** | Criptografia AES-256-GCM, HSM, auditoria, liveness detection, testes de viés |
| **Encarregado (DPO)** | Dados de contato do DPO da organização |

### 2.4 Direitos do Titular

```java
@RestController
@RequestMapping("/api/v1/biometric/titular")
public class TitularRightsController {

    private final BiometricDataService biometricDataService;
    private final ConsentService consentService;
    private final AuditService auditService;

    // Art. 18, II — Acesso: titular pode consultar quais dados biométricos estão cadastrados
    @GetMapping("/dados")
    public ResponseEntity<TitularDataResponse> consultarDados(
            @AuthenticationPrincipal UserDetails userDetails) {
        return ResponseEntity.ok(biometricDataService.getDadosTitular(userDetails.getUsername()));
    }

    // Art. 18, VI — Eliminação: titular pode solicitar exclusão completa
    @DeleteMapping("/dados")
    public ResponseEntity<Void> solicitarEliminacao(
            @AuthenticationPrincipal UserDetails userDetails) {
        biometricDataService.eliminarDadosBiometricos(userDetails.getUsername());
        consentService.revokeAll(userDetails.getUsername());
        auditService.log(userDetails.getUsername(), "BIOMETRIC_DATA_DELETED",
                "Eliminação solicitada pelo titular conforme Art. 18, VI da LGPD");
        return ResponseEntity.noContent().build();
    }

    // Art. 18, VIII — Revogação de consentimento
    @PostMapping("/revogar-consentimento")
    public ResponseEntity<Void> revogarConsentimento(
            @AuthenticationPrincipal UserDetails userDetails,
            @RequestParam BiometricType type) {
        consentService.revoke(userDetails.getUsername(), type);
        biometricDataService.eliminarPorTipo(userDetails.getUsername(), type);
        auditService.log(userDetails.getUsername(), "CONSENT_REVOKED",
                "Consentimento revogado para " + type + " conforme Art. 18, VIII da LGPD");
        return ResponseEntity.noContent().build();
    }

    // Art. 19 — Portabilidade: exportar dados em formato interoperável
    @GetMapping("/exportar")
    public ResponseEntity<byte[]> exportarDados(
            @AuthenticationPrincipal UserDetails userDetails) {
        byte[] jsonExport = biometricDataService.exportarDadosTitular(userDetails.getUsername());
        return ResponseEntity.ok()
                .header("Content-Disposition",
                        "attachment; filename=meus-dados-biometricos.json")
                .contentType(MediaType.APPLICATION_JSON)
                .body(jsonExport);
    }
}
```

---

## 3. Modelagem de Dados

### 3.1 Esquema de Banco de Dados

```sql
-- Template biométrico facial (vetor de características cifrado)
CREATE TABLE face_template (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT NOT NULL REFERENCES app_user(id) ON DELETE CASCADE,
    template_data   BYTEA NOT NULL,           -- template cifrado com AES-256-GCM
    iv              BYTEA NOT NULL,           -- vetor de inicialização (12 bytes)
    auth_tag        BYTEA NOT NULL,           -- tag de autenticação GCM (16 bytes)
    key_id          VARCHAR(64) NOT NULL,     -- referência à chave no HSM/Vault
    algorithm       VARCHAR(20) NOT NULL DEFAULT 'AES-256-GCM',
    quality_score   DECIMAL(5,2),             -- pontuação de qualidade da captura
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT fk_face_user UNIQUE (user_id)  -- um template facial por usuário
);

-- Credencial FIDO2/WebAuthn para impressão digital
CREATE TABLE webauthn_credential (
    id                  BIGSERIAL PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES app_user(id) ON DELETE CASCADE,
    credential_id       BYTEA NOT NULL UNIQUE,   -- ID da credencial (público)
    public_key_cose     BYTEA NOT NULL,           -- chave pública COSE
    attestation_format  VARCHAR(20) NOT NULL,     -- 'packed', 'tpm', 'android-key', etc.
    sign_count          BIGINT NOT NULL DEFAULT 0,
    transports          VARCHAR(100),             -- 'internal', 'usb', 'ble', 'nfc'
    aaguid              VARCHAR(36),              -- identificador do autenticador
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_used_at        TIMESTAMPTZ
);

-- Consentimento LGPD
CREATE TABLE biometric_consent (
    id                BIGSERIAL PRIMARY KEY,
    user_id           BIGINT NOT NULL REFERENCES app_user(id) ON DELETE CASCADE,
    type              VARCHAR(20) NOT NULL,        -- 'FACE', 'FINGERPRINT'
    purpose           VARCHAR(500) NOT NULL,
    consent_text      TEXT NOT NULL,
    consented_at      TIMESTAMPTZ NOT NULL,
    revoked_at        TIMESTAMPTZ,
    ip_address        VARCHAR(45) NOT NULL,
    user_agent        VARCHAR(500) NOT NULL,
    consent_version   INTEGER NOT NULL,
    CONSTRAINT uq_consent_user_type UNIQUE (user_id, type)
);

-- Log de auditoria
CREATE TABLE biometric_audit_log (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT REFERENCES app_user(id),
    action          VARCHAR(50) NOT NULL,
    detail          TEXT,
    ip_address      VARCHAR(45),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_audit_user ON biometric_audit_log(user_id, created_at DESC);
```

### 3.2 Entidades JPA

```java
@Entity
@Table(name = "face_template")
public class FaceTemplate {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false, unique = true)
    private User user;

    @Column(name = "template_data", nullable = false, columnDefinition = "BYTEA")
    private byte[] templateData; // template cifrado

    @Column(nullable = false, columnDefinition = "BYTEA")
    private byte[] iv;

    @Column(name = "auth_tag", nullable = false, columnDefinition = "BYTEA")
    private byte[] authTag;

    @Column(name = "key_id", nullable = false, length = 64)
    private String keyId;

    @Column(nullable = false, length = 20)
    private String algorithm;

    @Column(name = "quality_score")
    private BigDecimal qualityScore;

    @Column(name = "created_at", nullable = false, updatable = false)
    private Instant createdAt;

    @Column(name = "updated_at", nullable = false)
    private Instant updatedAt;

    @PrePersist
    void prePersist() {
        createdAt = updatedAt = Instant.now();
    }

    @PreUpdate
    void preUpdate() {
        updatedAt = Instant.now();
    }

    // getters e setters omitidos
}
```

```java
@Entity
@Table(name = "webauthn_credential")
public class WebAuthnCredential {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Column(name = "credential_id", nullable = false, unique = true, columnDefinition = "BYTEA")
    private byte[] credentialId;

    @Column(name = "public_key_cose", nullable = false, columnDefinition = "BYTEA")
    private byte[] publicKeyCose;

    @Column(name = "attestation_format", nullable = false, length = 20)
    private String attestationFormat;

    @Column(name = "sign_count", nullable = false)
    private long signCount;

    @Column(length = 100)
    private String transports;

    @Column(length = 36)
    private String aaguid;

    @Column(name = "created_at", nullable = false, updatable = false)
    private Instant createdAt;

    @Column(name = "last_used_at")
    private Instant lastUsedAt;

    @PrePersist
    void prePersist() {
        createdAt = Instant.now();
    }

    // getters e setters omitidos
}
```

---

## 4. Criptografia e Armazenamento Seguro dos Templates Biométricos

### 4.1 Estratégia de Criptografia

```
┌──────────────────────────────────────────────────────────────────┐
│                  Fluxo de Criptografia                           │
│                                                                  │
│  Template (vetor float[])                                        │
│       │                                                          │
│       ▼                                                          │
│  Serializar para byte[]                                          │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────────────────────────┐                                 │
│  │ AES-256-GCM                 │◀── Chave AES do HSM/Vault      │
│  │ • IV: 12 bytes (aleatório)  │    (referenciada por key_id)    │
│  │ • Tag: 16 bytes (integridade│                                 │
│  │   + autenticidade)          │                                 │
│  └──────────────┬──────────────┘                                 │
│                 │                                                 │
│                 ▼                                                 │
│  Armazenar no PostgreSQL:                                        │
│  { template_data: cifrado, iv: IV, auth_tag: TAG, key_id: ref } │
└──────────────────────────────────────────────────────────────────┘
```

**Por que AES-256-GCM:**

| Propriedade | Benefício |
|---|---|
| **Criptografia autenticada (AEAD)** | Garante confidencialidade E integridade — detecta adulteração |
| **IV único por operação** | Previne ataques de repetição e análise de padrões |
| **Performance** | Aceleração por hardware (AES-NI) em CPUs modernas |
| **Padrão NIST** | Conformidade com padrões internacionais de segurança |

### 4.2 Serviço de Criptografia

```java
@Service
public class BiometricCryptoService {

    private static final String ALGORITHM = "AES/GCM/NoPadding";
    private static final int IV_LENGTH = 12;
    private static final int TAG_LENGTH = 128; // bits

    private final VaultKeyService vaultKeyService; // abstrai acesso ao HSM/Vault

    public BiometricCryptoService(VaultKeyService vaultKeyService) {
        this.vaultKeyService = vaultKeyService;
    }

    public EncryptedTemplate encrypt(byte[] plainTemplate, String keyId) {
        try {
            SecretKey key = vaultKeyService.getKey(keyId);
            byte[] iv = generateIv();

            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, key, new GCMParameterSpec(TAG_LENGTH, iv));

            byte[] cipherTextWithTag = cipher.doFinal(plainTemplate);

            // GCM anexa o auth tag ao final do ciphertext
            int cipherTextLength = cipherTextWithTag.length - (TAG_LENGTH / 8);
            byte[] cipherText = Arrays.copyOfRange(cipherTextWithTag, 0, cipherTextLength);
            byte[] authTag = Arrays.copyOfRange(cipherTextWithTag, cipherTextLength,
                    cipherTextWithTag.length);

            return new EncryptedTemplate(cipherText, iv, authTag, keyId, ALGORITHM);
        } catch (GeneralSecurityException e) {
            throw new BiometricCryptoException("Falha ao cifrar template biométrico", e);
        }
    }

    public byte[] decrypt(EncryptedTemplate encrypted) {
        try {
            SecretKey key = vaultKeyService.getKey(encrypted.keyId());
            byte[] cipherTextWithTag = concatenate(encrypted.cipherText(), encrypted.authTag());

            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, key,
                    new GCMParameterSpec(TAG_LENGTH, encrypted.iv()));

            return cipher.doFinal(cipherTextWithTag);
        } catch (GeneralSecurityException e) {
            throw new BiometricCryptoException("Falha ao decifrar template biométrico", e);
        }
    }

    private byte[] generateIv() {
        byte[] iv = new byte[IV_LENGTH];
        SecureRandom.getInstanceStrong().nextBytes(iv);
        return iv;
    }

    private byte[] concatenate(byte[] a, byte[] b) {
        byte[] result = new byte[a.length + b.length];
        System.arraycopy(a, 0, result, 0, a.length);
        System.arraycopy(b, 0, result, a.length, b.length);
        return result;
    }
}
```

```java
public record EncryptedTemplate(
    byte[] cipherText,
    byte[] iv,
    byte[] authTag,
    String keyId,
    String algorithm
) {}
```

**Integração com HashiCorp Vault (application.yml):**

```yaml
spring:
  cloud:
    vault:
      uri: https://vault.internal:8200
      authentication: KUBERNETES  # ou TOKEN, APPROLE
      kv:
        backend: secret
        default-context: biometric
      token: ${VAULT_TOKEN}

biometric:
  crypto:
    active-key-id: "biometric-aes-key-v1"
    key-rotation-days: 90
```

---

## 5. Captura de Face — Frontend React

### 5.1 Componente de Captura Facial

Utiliza a **MediaDevices API** (WebRTC) para capturar a imagem da câmera e a biblioteca **face-api.js** (baseada em TensorFlow.js) para extrair o template facial no próprio navegador, minimizando o tráfego de dados sensíveis.

```bash
npm install face-api.js @types/dom-mediacapture-record
```

```tsx
import * as faceapi from "face-api.js";
import { useRef, useState, useEffect, useCallback } from "react";

interface FaceCaptureProps {
  onTemplateExtracted: (template: Float32Array) => void;
  onError: (error: string) => void;
}

function FaceCapture({ onTemplateExtracted, onError }: FaceCaptureProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [modelsLoaded, setModelsLoaded] = useState(false);
  const [capturing, setCapturing] = useState(false);
  const [feedback, setFeedback] = useState("Carregando modelos...");

  useEffect(() => {
    const loadModels = async () => {
      const MODEL_URL = "/models"; // servir modelos estáticos via public/models
      await Promise.all([
        faceapi.nets.ssdMobilenetv1.loadFromUri(MODEL_URL),
        faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL),
        faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL),
      ]);
      setModelsLoaded(true);
      setFeedback("Modelos carregados. Clique para iniciar a câmera.");
    };
    loadModels().catch(() => onError("Falha ao carregar modelos de reconhecimento facial"));
  }, [onError]);

  const startCamera = useCallback(async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: "user", width: 640, height: 480 },
      });
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
        setCapturing(true);
        setFeedback("Posicione seu rosto no centro da tela");
      }
    } catch {
      onError("Não foi possível acessar a câmera. Verifique as permissões.");
    }
  }, [onError]);

  const captureTemplate = useCallback(async () => {
    if (!videoRef.current || !canvasRef.current) return;

    setFeedback("Processando...");
    const detection = await faceapi
      .detectSingleFace(videoRef.current)
      .withFaceLandmarks()
      .withFaceDescriptor();

    if (!detection) {
      setFeedback("Nenhum rosto detectado. Tente novamente.");
      return;
    }

    if (detection.detection.score < 0.9) {
      setFeedback("Qualidade baixa. Melhore a iluminação e tente novamente.");
      return;
    }

    // O descriptor é um Float32Array de 128 dimensões (template facial)
    // Nunca armazenar a imagem — apenas o template
    onTemplateExtracted(detection.descriptor);

    // Liberar a câmera imediatamente após captura
    const stream = videoRef.current.srcObject as MediaStream;
    stream?.getTracks().forEach((track) => track.stop());
    setCapturing(false);
    setFeedback("Template capturado com sucesso!");
  }, [onTemplateExtracted]);

  return (
    <div className="face-capture">
      <p className="capture-feedback">{feedback}</p>
      <video ref={videoRef} autoPlay muted playsInline width={640} height={480} />
      <canvas ref={canvasRef} style={{ display: "none" }} />
      <div className="capture-actions">
        {!capturing && modelsLoaded && (
          <button onClick={startCamera} className="btn-primary">
            Iniciar Câmera
          </button>
        )}
        {capturing && (
          <button onClick={captureTemplate} className="btn-primary">
            Capturar Rosto
          </button>
        )}
      </div>
    </div>
  );
}
```

**Envio do template ao backend:**

```tsx
async function enviarTemplateFacial(template: Float32Array, token: string): Promise<void> {
  // Converter Float32Array para Base64 para transporte JSON
  const bytes = new Uint8Array(template.buffer);
  const base64Template = btoa(String.fromCharCode(...bytes));

  const response = await fetch("/api/v1/biometric/face/register", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify({ template: base64Template, qualityScore: 0.95 }),
  });

  if (!response.ok) {
    throw new Error("Falha ao registrar template facial");
  }
}
```

### 5.1.1 Alternativa: Extração do Template Facial no Backend (Java + OpenCV DNN)

Na abordagem anterior, o template é extraído no navegador com face-api.js. Em cenários onde se deseja **maior controle sobre o modelo de IA**, **consistência entre plataformas** (web, mobile, desktop) ou **processamento de imagens enviadas por upload**, a extração pode ser feita no backend Java usando **OpenCV** com módulo DNN (Deep Neural Network).

```
┌──────────┐                   ┌──────────────────────────────────────────┐
│  React   │  POST /face       │         Spring Boot Backend              │
│ (browser)│  { imageBase64 }  │                                          │
│          │──────────────────▶│  1. Decodificar Base64 → byte[]          │
│  Captura │                   │  2. OpenCV: imread → Mat                 │
│  imagem  │                   │  3. DNN face detector → recortar rosto   │
│  da      │                   │  4. DNN face recognizer → float[128]     │
│  câmera  │                   │  5. Descartar imagem da memória          │
│          │                   │  6. Cifrar template (AES-256-GCM)        │
│          │◀──────────────────│  7. Persistir template cifrado           │
│  { ok }  │                   │                                          │
└──────────┘                   └──────────────────────────────────────────┘
```

**Dependências Maven — OpenCV com JavaCPP:**

```xml
<dependencies>
    <!-- OpenCV via JavaCPP (inclui binários nativos automaticamente) -->
    <dependency>
        <groupId>org.bytedeco</groupId>
        <artifactId>opencv-platform</artifactId>
        <version>4.10.0-1.5.11</version>
    </dependency>
    <dependency>
        <groupId>org.bytedeco</groupId>
        <artifactId>javacpp</artifactId>
        <version>1.5.11</version>
    </dependency>

    <!-- Alternativa: ONNX Runtime para modelos no formato ONNX -->
    <dependency>
        <groupId>com.microsoft.onnxruntime</groupId>
        <artifactId>onnxruntime</artifactId>
        <version>1.19.0</version>
    </dependency>
</dependencies>
```

**Modelos pré-treinados necessários (colocar em `src/main/resources/models/`):**

| Modelo | Arquivo | Finalidade |
|---|---|---|
| Face Detector (SSD) | `face_detection_yunet_2023mar.onnx` | Detectar e localizar rostos na imagem |
| Face Recognizer (SFace) | `face_recognition_sface_2021dec.onnx` | Extrair embedding de 128 dimensões |

Ambos os modelos são do [OpenCV Model Zoo](https://github.com/opencv/opencv_zoo) e podem ser usados livremente.

**Serviço de extração de template facial:**

```java
@Service
public class FaceTemplateExtractorService {

    private static final int TEMPLATE_DIMENSIONS = 128;

    private final FaceDetectorYN faceDetector;
    private final FaceRecognizerSF faceRecognizer;

    public FaceTemplateExtractorService(
            @Value("${biometric.face.detector-model-path}") String detectorPath,
            @Value("${biometric.face.recognizer-model-path}") String recognizerPath) {

        // Inicializar detector facial YuNet
        this.faceDetector = FaceDetectorYN.create(
                detectorPath,
                "",          // config (vazio para ONNX)
                new Size(640, 480), // tamanho de entrada
                0.9f,        // score threshold
                0.3f,        // NMS threshold
                5000         // top_k
        );

        // Inicializar reconhecedor facial SFace
        this.faceRecognizer = FaceRecognizerSF.create(recognizerPath, "");
    }

    public FaceExtractionResult extractTemplate(byte[] imageBytes) {
        // 1. Decodificar imagem
        Mat image = imdecode(new Mat(new BytePointer(imageBytes)), IMREAD_COLOR);
        try {
            if (image.empty()) {
                throw new InvalidBiometricDataException("Imagem inválida ou corrompida");
            }

            // 2. Detectar rosto(s)
            faceDetector.setInputSize(new Size(image.cols(), image.rows()));
            Mat detections = new Mat();
            faceDetector.detect(image, detections);

            if (detections.rows() == 0) {
                throw new InvalidBiometricDataException("Nenhum rosto detectado na imagem");
            }

            if (detections.rows() > 1) {
                throw new InvalidBiometricDataException(
                        "Múltiplos rostos detectados. Envie uma imagem com apenas um rosto");
            }

            // 3. Extrair a região do rosto detectado
            Mat faceRegion = new Mat();
            faceRecognizer.alignCrop(image, detections.row(0), faceRegion);

            // 4. Gerar embedding (template de 128 dimensões)
            Mat embedding = new Mat();
            faceRecognizer.feature(faceRegion, embedding);

            // 5. Converter Mat para float[]
            float[] templateArray = new float[TEMPLATE_DIMENSIONS];
            embedding.createIndexer()
                    .get(new long[]{0, 0}, templateArray, 0, TEMPLATE_DIMENSIONS);

            // 6. Calcular score de qualidade baseado na confiança da detecção
            float detectionScore = detections.createIndexer()
                    .getFloat(0, detections.cols() - 1);

            return new FaceExtractionResult(templateArray, detectionScore);
        } finally {
            // CRÍTICO: liberar memória nativa e descartar imagem imediatamente
            image.close();
        }
    }

    public double compareFaces(float[] template1, float[] template2) {
        // Converter para Mat para usar o comparador do OpenCV
        Mat feature1 = new Mat(1, TEMPLATE_DIMENSIONS, CV_32F, new FloatPointer(template1));
        Mat feature2 = new Mat(1, TEMPLATE_DIMENSIONS, CV_32F, new FloatPointer(template2));
        try {
            // Distância cosseno — quanto menor, mais similares (0 = idênticos)
            return FaceRecognizerSF.match(feature1, feature2,
                    FaceRecognizerSF.FR_COSINE);
        } finally {
            feature1.close();
            feature2.close();
        }
    }
}
```

```java
public record FaceExtractionResult(
    float[] template,       // vetor de 128 dimensões
    float qualityScore      // confiança da detecção (0.0 a 1.0)
) {
    public byte[] templateAsBytes() {
        ByteBuffer buffer = ByteBuffer.allocate(template.length * Float.BYTES)
                .order(ByteOrder.LITTLE_ENDIAN);
        for (float v : template) {
            buffer.putFloat(v);
        }
        return buffer.array();
    }
}
```

**Componente React simplificado — envia imagem (não template) ao backend:**

```tsx
function FaceCaptureServerSide({ onSuccess, onError }: FaceCaptureServerSideProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [capturing, setCapturing] = useState(false);

  const startCamera = async () => {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: "user", width: 640, height: 480 },
    });
    if (videoRef.current) {
      videoRef.current.srcObject = stream;
      setCapturing(true);
    }
  };

  const captureAndSend = async () => {
    if (!videoRef.current || !canvasRef.current) return;

    const canvas = canvasRef.current;
    canvas.width = 640;
    canvas.height = 480;
    const ctx = canvas.getContext("2d")!;
    ctx.drawImage(videoRef.current, 0, 0);

    // Converter canvas para Blob JPEG (comprime a imagem)
    const blob = await new Promise<Blob>((resolve) =>
      canvas.toBlob((b) => resolve(b!), "image/jpeg", 0.9)
    );

    // Enviar imagem via multipart/form-data
    const formData = new FormData();
    formData.append("image", blob, "face-capture.jpg");

    const response = await fetch("/api/v1/biometric/face/register-with-image", {
      method: "POST",
      headers: { Authorization: `Bearer ${getToken()}` },
      body: formData,
    });

    // Liberar câmera
    const stream = videoRef.current.srcObject as MediaStream;
    stream?.getTracks().forEach((track) => track.stop());
    setCapturing(false);

    if (response.ok) {
      onSuccess();
    } else {
      const error = await response.json();
      onError(error.message);
    }
  };

  return (
    <div className="face-capture">
      <video ref={videoRef} autoPlay muted playsInline width={640} height={480} />
      <canvas ref={canvasRef} style={{ display: "none" }} />
      {!capturing && <button onClick={startCamera}>Iniciar Câmera</button>}
      {capturing && <button onClick={captureAndSend}>Capturar e Registrar</button>}
    </div>
  );
}
```

**Controller para receber imagem e extrair template no backend:**

```java
@RestController
@RequestMapping("/api/v1/biometric/face")
public class FaceRegistrationController {

    private final FaceTemplateExtractorService extractorService;
    private final BiometricCryptoService cryptoService;
    private final FaceTemplateRepository templateRepository;
    private final ConsentService consentService;
    private final AuditService auditService;
    private final UserRepository userRepository;

    @Value("${biometric.crypto.active-key-id}")
    private String activeKeyId;

    @Value("${biometric.face.min-quality-score}")
    private float minQualityScore;

    @PostMapping("/register-with-image")
    public ResponseEntity<Void> registerWithImage(
            @RequestParam("image") MultipartFile imageFile,
            @AuthenticationPrincipal UserDetails userDetails) throws IOException {

        consentService.requireActiveConsent(userDetails.getUsername(), BiometricType.FACE);

        // 1. Validar arquivo recebido
        validateImageFile(imageFile);

        // 2. Extrair template facial via OpenCV DNN
        FaceExtractionResult extraction = extractorService.extractTemplate(
                imageFile.getBytes());
        // A imagem NÃO é persistida — apenas o template permanece em memória

        // 3. Verificar qualidade mínima
        if (extraction.qualityScore() < minQualityScore) {
            throw new InvalidBiometricDataException(
                    "Qualidade da captura insuficiente (%.2f). Mínimo: %.2f"
                            .formatted(extraction.qualityScore(), minQualityScore));
        }

        // 4. Cifrar template
        EncryptedTemplate encrypted = cryptoService.encrypt(
                extraction.templateAsBytes(), activeKeyId);

        // 5. Persistir template cifrado
        User user = userRepository.findByUsername(userDetails.getUsername()).orElseThrow();
        templateRepository.deleteByUser(user);

        FaceTemplate faceTemplate = new FaceTemplate();
        faceTemplate.setUser(user);
        faceTemplate.setTemplateData(encrypted.cipherText());
        faceTemplate.setIv(encrypted.iv());
        faceTemplate.setAuthTag(encrypted.authTag());
        faceTemplate.setKeyId(encrypted.keyId());
        faceTemplate.setAlgorithm(encrypted.algorithm());
        faceTemplate.setQualityScore(BigDecimal.valueOf(extraction.qualityScore()));
        templateRepository.save(faceTemplate);

        auditService.log(userDetails.getUsername(), "FACE_REGISTERED",
                "Template facial extraído no servidor. Quality: " + extraction.qualityScore());

        return ResponseEntity.status(HttpStatus.CREATED).build();
    }

    private void validateImageFile(MultipartFile file) {
        if (file.isEmpty()) {
            throw new InvalidBiometricDataException("Arquivo de imagem vazio");
        }
        String contentType = file.getContentType();
        if (contentType == null ||
                !(contentType.equals("image/jpeg") || contentType.equals("image/png"))) {
            throw new InvalidBiometricDataException(
                    "Formato de imagem não suportado. Use JPEG ou PNG");
        }
        if (file.getSize() > 5 * 1024 * 1024) { // 5 MB
            throw new InvalidBiometricDataException("Imagem excede o tamanho máximo de 5 MB");
        }
    }
}
```

**Configuração (application.yml):**

```yaml
biometric:
  face:
    detector-model-path: classpath:models/face_detection_yunet_2023mar.onnx
    recognizer-model-path: classpath:models/face_recognition_sface_2021dec.onnx
    similarity-threshold: 0.363   # threshold recomendado pelo OpenCV Zoo para FR_COSINE
    min-quality-score: 0.85
```

**Alternativa com ONNX Runtime (sem OpenCV):**

Para ambientes onde instalar binários nativos do OpenCV é inviável, o ONNX Runtime permite carregar os mesmos modelos `.onnx` diretamente:

```java
@Service
public class FaceTemplateExtractorOnnxService {

    private final OrtEnvironment env;
    private final OrtSession detectorSession;
    private final OrtSession recognizerSession;

    public FaceTemplateExtractorOnnxService(
            @Value("${biometric.face.detector-model-path}") Resource detectorModel,
            @Value("${biometric.face.recognizer-model-path}") Resource recognizerModel)
            throws OrtException, IOException {

        this.env = OrtEnvironment.getEnvironment();

        OrtSession.SessionOptions opts = new OrtSession.SessionOptions();
        opts.setOptimizationLevel(OrtSession.SessionOptions.OptLevel.ALL_OPT);

        this.detectorSession = env.createSession(
                detectorModel.getContentAsByteArray(), opts);
        this.recognizerSession = env.createSession(
                recognizerModel.getContentAsByteArray(), opts);
    }

    public float[] extractTemplate(byte[] imageBytes) throws OrtException {
        // 1. Pré-processar imagem: decodificar JPEG → RGB float tensor [1, 3, 112, 112]
        float[][][][] inputTensor = preprocessImage(imageBytes);

        // 2. Detectar rosto
        OnnxTensor detInput = OnnxTensor.createTensor(env, inputTensor);
        try (OrtSession.Result detResult = detectorSession.run(
                Map.of("input", detInput))) {

            float[][] detections = (float[][]) detResult.get(0).getValue();
            if (detections.length == 0) {
                throw new InvalidBiometricDataException("Nenhum rosto detectado");
            }

            // 3. Recortar e alinhar rosto detectado
            float[][][][] alignedFace = cropAndAlign(imageBytes, detections[0]);

            // 4. Extrair embedding
            OnnxTensor recInput = OnnxTensor.createTensor(env, alignedFace);
            try (OrtSession.Result recResult = recognizerSession.run(
                    Map.of("input", recInput))) {

                float[][] embeddings = (float[][]) recResult.get(0).getValue();
                return normalize(embeddings[0]); // L2-normalize
            }
        }
    }

    private float[] normalize(float[] vector) {
        double norm = 0.0;
        for (float v : vector) norm += v * v;
        norm = Math.sqrt(norm);
        float[] normalized = new float[vector.length];
        for (int i = 0; i < vector.length; i++) {
            normalized[i] = (float) (vector[i] / norm);
        }
        return normalized;
    }

    private float[][][][] preprocessImage(byte[] imageBytes) {
        // Implementação: usar javax.imageio.ImageIO para decodificar,
        // redimensionar para 112x112 (ou 640x480 para detector),
        // normalizar para [0,1] e transpor para NCHW
        // (detalhes omitidos por depender do modelo específico)
        throw new UnsupportedOperationException("Implementar pré-processamento");
    }

    private float[][][][] cropAndAlign(byte[] imageBytes, float[] detection) {
        // Recortar região do rosto usando bounding box da detecção
        // e alinhar usando landmarks (olhos, nariz)
        throw new UnsupportedOperationException("Implementar crop e alinhamento");
    }
}
```

**Comparação das abordagens — Frontend vs. Backend:**

| Aspecto | Extração no Frontend (face-api.js) | Extração no Backend (OpenCV/ONNX) |
|---|---|---|
| **Dados trafegados** | Apenas template (512 bytes) | Imagem JPEG (50-200 KB) |
| **Privacidade em trânsito** | Melhor — imagem nunca sai do dispositivo | Imagem trafega via HTTPS |
| **Controle do modelo** | Modelo fixo no navegador | Modelo atualizado no servidor sem deploy frontend |
| **Consistência** | Variação entre navegadores/dispositivos | Resultado idêntico para todas as plataformas |
| **Carga no servidor** | Menor (recebe template pronto) | Maior (inferência DNN por requisição) |
| **Segurança contra adulteração** | Template pode ser manipulado no client | Servidor controla toda a pipeline |
| **Liveness detection** | Mais fácil (acesso contínuo à câmera) | Requer múltiplos frames ou SDK dedicado |
| **Recomendação** | Apps web onde privacidade é prioridade | Sistemas corporativos com controle centralizado |

### 5.2 Componente de Captura de Impressão Digital (WebAuthn)

Para impressão digital em navegadores, utiliza-se a **Web Authentication API (WebAuthn/FIDO2)**, que delega a captura ao autenticador do dispositivo (leitor biométrico, Windows Hello, Touch ID). A chave privada **nunca sai do dispositivo**.

```tsx
interface FingerprintRegistrationProps {
  userId: string;
  onSuccess: (credentialId: string) => void;
  onError: (error: string) => void;
}

function FingerprintRegistration({ userId, onSuccess, onError }: FingerprintRegistrationProps) {
  const [status, setStatus] = useState<"idle" | "registering" | "done">("idle");

  const startRegistration = async () => {
    try {
      setStatus("registering");

      // 1. Obter challenge do servidor
      const optionsResponse = await fetch("/api/v1/biometric/webauthn/register/options", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ userId }),
      });
      const options: PublicKeyCredentialCreationOptions = await optionsResponse.json();

      // Decodificar campos Base64URL do servidor
      options.challenge = base64UrlToBuffer(options.challenge as unknown as string);
      options.user.id = base64UrlToBuffer(options.user.id as unknown as string);
      if (options.excludeCredentials) {
        options.excludeCredentials = options.excludeCredentials.map((cred) => ({
          ...cred,
          id: base64UrlToBuffer(cred.id as unknown as string),
        }));
      }

      // 2. Solicitar registro biométrico ao navegador/dispositivo
      const credential = (await navigator.credentials.create({
        publicKey: options,
      })) as PublicKeyCredential;

      if (!credential) {
        throw new Error("Registro cancelado pelo usuário");
      }

      // 3. Enviar resposta ao servidor para validação
      const attestationResponse = credential.response as AuthenticatorAttestationResponse;

      const registrationResult = await fetch("/api/v1/biometric/webauthn/register/verify", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          credentialId: bufferToBase64Url(credential.rawId),
          attestationObject: bufferToBase64Url(attestationResponse.attestationObject),
          clientDataJSON: bufferToBase64Url(attestationResponse.clientDataJSON),
          transports: attestationResponse.getTransports?.() ?? [],
        }),
      });

      if (!registrationResult.ok) {
        throw new Error("Falha na verificação do registro");
      }

      setStatus("done");
      onSuccess(bufferToBase64Url(credential.rawId));
    } catch (err) {
      setStatus("idle");
      onError(err instanceof Error ? err.message : "Erro desconhecido no registro");
    }
  };

  return (
    <div className="fingerprint-registration">
      <h3>Registro de Impressão Digital</h3>
      <p>Utilize o leitor biométrico do seu dispositivo para registrar sua impressão digital.</p>
      {status === "idle" && (
        <button onClick={startRegistration} className="btn-primary">
          Registrar Impressão Digital
        </button>
      )}
      {status === "registering" && (
        <p>Aguardando leitura biométrica no dispositivo...</p>
      )}
      {status === "done" && (
        <p className="success-msg">Impressão digital registrada com sucesso!</p>
      )}
    </div>
  );
}

// Utilitários Base64URL <-> ArrayBuffer
function base64UrlToBuffer(base64url: string): ArrayBuffer {
  const base64 = base64url.replace(/-/g, "+").replace(/_/g, "/");
  const padding = "=".repeat((4 - (base64.length % 4)) % 4);
  const binary = atob(base64 + padding);
  const bytes = new Uint8Array(binary.length);
  for (let i = 0; i < binary.length; i++) {
    bytes[i] = binary.charCodeAt(i);
  }
  return bytes.buffer;
}

function bufferToBase64Url(buffer: ArrayBuffer): string {
  const bytes = new Uint8Array(buffer);
  let binary = "";
  bytes.forEach((b) => (binary += String.fromCharCode(b)));
  return btoa(binary).replace(/\+/g, "-").replace(/\//g, "_").replace(/=+$/, "");
}
```

---

## 6. Backend — API REST para Cadastro Biométrico

### 6.1 Configuração e Dependências

```xml
<dependencies>
    <!-- Spring Boot Starters -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- HashiCorp Vault -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-vault-config</artifactId>
    </dependency>

    <!-- WebAuthn Server (Yubico) -->
    <dependency>
        <groupId>com.yubico</groupId>
        <artifactId>webauthn-server-core</artifactId>
        <version>2.6.0</version>
    </dependency>

    <!-- PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

### 6.2 DTOs

```java
public record FaceRegistrationRequest(
    @NotBlank String template,      // Base64 do Float32Array (128 dimensões)
    @DecimalMin("0.0") @DecimalMax("1.0") BigDecimal qualityScore
) {}

public record FaceAuthRequest(
    @NotBlank String template       // Base64 do template facial capturado
) {}

public record WebAuthnRegistrationOptionsRequest(
    @NotBlank String userId
) {}

public record WebAuthnRegistrationVerifyRequest(
    @NotBlank String credentialId,
    @NotBlank String attestationObject,
    @NotBlank String clientDataJSON,
    List<String> transports
) {}

public record WebAuthnAuthenticationRequest(
    @NotBlank String credentialId,
    @NotBlank String authenticatorData,
    @NotBlank String clientDataJSON,
    @NotBlank String signature
) {}

public record BiometricAuthResponse(
    String accessToken,
    String tokenType,
    long expiresIn
) {}
```

### 6.3 Controller de Cadastro Biométrico

```java
@RestController
@RequestMapping("/api/v1/biometric")
public class BiometricController {

    private final FaceRegistrationService faceRegistrationService;
    private final WebAuthnService webAuthnService;
    private final ConsentService consentService;

    @PostMapping("/face/register")
    public ResponseEntity<Void> registerFace(
            @Valid @RequestBody FaceRegistrationRequest request,
            @AuthenticationPrincipal UserDetails userDetails,
            HttpServletRequest httpRequest) {

        // Verificar consentimento ativo
        consentService.requireActiveConsent(userDetails.getUsername(), BiometricType.FACE);

        faceRegistrationService.register(userDetails.getUsername(), request);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }

    @PostMapping("/webauthn/register/options")
    public ResponseEntity<PublicKeyCredentialCreationOptions> webauthnRegisterOptions(
            @Valid @RequestBody WebAuthnRegistrationOptionsRequest request,
            @AuthenticationPrincipal UserDetails userDetails) {

        consentService.requireActiveConsent(userDetails.getUsername(), BiometricType.FINGERPRINT);

        var options = webAuthnService.generateRegistrationOptions(userDetails.getUsername());
        return ResponseEntity.ok(options);
    }

    @PostMapping("/webauthn/register/verify")
    public ResponseEntity<Void> webauthnRegisterVerify(
            @Valid @RequestBody WebAuthnRegistrationVerifyRequest request,
            @AuthenticationPrincipal UserDetails userDetails) {

        webAuthnService.verifyRegistration(userDetails.getUsername(), request);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

### 6.4 Serviço de Cadastro Facial

```java
@Service
@Transactional
public class FaceRegistrationService {

    private static final int EXPECTED_DESCRIPTOR_LENGTH = 128; // face-api.js gera 128 floats

    private final FaceTemplateRepository faceTemplateRepository;
    private final UserRepository userRepository;
    private final BiometricCryptoService cryptoService;
    private final AuditService auditService;

    @Value("${biometric.crypto.active-key-id}")
    private String activeKeyId;

    public void register(String username, FaceRegistrationRequest request) {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("Usuário não encontrado"));

        // Decodificar e validar template
        byte[] templateBytes = Base64.getDecoder().decode(request.template());
        validateTemplateSize(templateBytes);

        // Cifrar template antes de persistir
        EncryptedTemplate encrypted = cryptoService.encrypt(templateBytes, activeKeyId);

        // Remover template anterior se existir (um por usuário)
        faceTemplateRepository.deleteByUser(user);

        FaceTemplate faceTemplate = new FaceTemplate();
        faceTemplate.setUser(user);
        faceTemplate.setTemplateData(encrypted.cipherText());
        faceTemplate.setIv(encrypted.iv());
        faceTemplate.setAuthTag(encrypted.authTag());
        faceTemplate.setKeyId(encrypted.keyId());
        faceTemplate.setAlgorithm(encrypted.algorithm());
        faceTemplate.setQualityScore(request.qualityScore());

        faceTemplateRepository.save(faceTemplate);

        auditService.log(username, "FACE_REGISTERED",
                "Template facial registrado. Quality score: " + request.qualityScore());
    }

    private void validateTemplateSize(byte[] templateBytes) {
        // Float32Array com 128 elementos = 512 bytes
        if (templateBytes.length != EXPECTED_DESCRIPTOR_LENGTH * Float.BYTES) {
            throw new InvalidBiometricDataException(
                    "Template facial inválido: tamanho esperado %d bytes, recebido %d bytes"
                            .formatted(EXPECTED_DESCRIPTOR_LENGTH * Float.BYTES,
                                    templateBytes.length));
        }
    }
}
```

### 6.5 Serviço de Cadastro de Impressão Digital (FIDO2/WebAuthn)

```java
@Service
@Transactional
public class WebAuthnService {

    private final RelyingParty relyingParty;
    private final WebAuthnCredentialRepository credentialRepository;
    private final UserRepository userRepository;
    private final AuditService auditService;

    // Cache temporário de challenges pendentes (usar Redis em produção)
    private final Map<String, PublicKeyCredentialCreationOptions> pendingRegistrations =
            new ConcurrentHashMap<>();

    public WebAuthnService(
            WebAuthnCredentialRepository credentialRepository,
            UserRepository userRepository,
            AuditService auditService,
            @Value("${biometric.webauthn.rp-id}") String rpId,
            @Value("${biometric.webauthn.rp-name}") String rpName,
            @Value("${biometric.webauthn.origin}") String origin) {

        this.credentialRepository = credentialRepository;
        this.userRepository = userRepository;
        this.auditService = auditService;

        this.relyingParty = RelyingParty.builder()
                .identity(RelyingPartyIdentity.builder()
                        .id(rpId)
                        .name(rpName)
                        .build())
                .origins(Set.of(origin))
                .credentialRepository(new YubicoCredentialRepositoryAdapter(credentialRepository))
                .build();
    }

    public PublicKeyCredentialCreationOptions generateRegistrationOptions(String username) {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("Usuário não encontrado"));

        UserIdentity userIdentity = UserIdentity.builder()
                .name(username)
                .displayName(user.getDisplayName())
                .id(new ByteArray(username.getBytes(StandardCharsets.UTF_8)))
                .build();

        StartRegistrationOptions options = StartRegistrationOptions.builder()
                .user(userIdentity)
                .authenticatorSelection(AuthenticatorSelectionCriteria.builder()
                        .authenticatorAttachment(AuthenticatorAttachment.PLATFORM)
                        .userVerification(UserVerificationRequirement.REQUIRED)
                        .residentKey(ResidentKeyRequirement.PREFERRED)
                        .build())
                .build();

        PublicKeyCredentialCreationOptions creationOptions =
                relyingParty.startRegistration(options);

        pendingRegistrations.put(username, creationOptions);
        return creationOptions;
    }

    public void verifyRegistration(String username,
                                   WebAuthnRegistrationVerifyRequest request) {
        PublicKeyCredentialCreationOptions creationOptions =
                pendingRegistrations.remove(username);

        if (creationOptions == null) {
            throw new BiometricAuthenticationException("Nenhum registro pendente para o usuário");
        }

        try {
            FinishRegistrationOptions finishOptions = FinishRegistrationOptions.builder()
                    .request(creationOptions)
                    .response(parseRegistrationResponse(request))
                    .build();

            RegistrationResult result = relyingParty.finishRegistration(finishOptions);

            // Persistir credencial
            WebAuthnCredential credential = new WebAuthnCredential();
            credential.setUser(userRepository.findByUsername(username).orElseThrow());
            credential.setCredentialId(result.getKeyId().getId().getBytes());
            credential.setPublicKeyCose(result.getPublicKeyCose().getBytes());
            credential.setAttestationFormat(result.getAttestationMetadata()
                    .map(m -> m.getAttestationFormat().name())
                    .orElse("none"));
            credential.setSignCount(result.getSignatureCount());
            credential.setTransports(String.join(",", request.transports()));

            credentialRepository.save(credential);

            auditService.log(username, "FINGERPRINT_REGISTERED",
                    "Credencial WebAuthn registrada");
        } catch (RegistrationFailedException e) {
            throw new BiometricAuthenticationException("Falha na verificação do registro", e);
        }
    }

    private AuthenticatorAttestationResponse parseRegistrationResponse(
            WebAuthnRegistrationVerifyRequest request) {
        // Converter campos Base64URL para objetos Yubico
        return AuthenticatorAttestationResponse.builder()
                .attestationObject(ByteArray.fromBase64Url(request.attestationObject()))
                .clientDataJSON(ByteArray.fromBase64Url(request.clientDataJSON()))
                .build();
    }
}
```

**application.yml:**

```yaml
biometric:
  webauthn:
    rp-id: "app.example.com"
    rp-name: "AcmeCorp Biometric Auth"
    origin: "https://app.example.com"
  crypto:
    active-key-id: "biometric-aes-key-v1"
    key-rotation-days: 90
  face:
    similarity-threshold: 0.6    # distância euclidiana máxima para match
    min-quality-score: 0.85
```

---

## 7. Autenticação Biométrica

### 7.1 Fluxo de Autenticação Facial

```
┌──────────┐         ┌──────────────┐         ┌──────────────┐
│  React   │         │ Spring Boot  │         │  PostgreSQL   │
│ (browser)│         │   Backend    │         │  + Vault      │
└────┬─────┘         └──────┬───────┘         └──────┬───────┘
     │  1. Captura face          │                    │
     │  (câmera + face-api.js)   │                    │
     │                           │                    │
     │  2. POST /auth/face       │                    │
     │  { template, username }   │                    │
     │──────────────────────────▶│                    │
     │                           │  3. Buscar template│
     │                           │     cifrado do user│
     │                           │───────────────────▶│
     │                           │◀───────────────────│
     │                           │                    │
     │                           │  4. Decifrar com   │
     │                           │     chave do Vault │
     │                           │                    │
     │                           │  5. Calcular       │
     │                           │     distância      │
     │                           │     euclidiana     │
     │                           │                    │
     │                           │  6. distância <    │
     │                           │     threshold?     │
     │                           │                    │
     │  7. { JWT access_token }  │                    │
     │◀──────────────────────────│                    │
     │                           │                    │
```

```java
@Service
public class FaceAuthenticationService {

    private final FaceTemplateRepository faceTemplateRepository;
    private final BiometricCryptoService cryptoService;
    private final JwtTokenService jwtTokenService;
    private final AuditService auditService;

    @Value("${biometric.face.similarity-threshold}")
    private double similarityThreshold;

    public BiometricAuthResponse authenticate(String username, FaceAuthRequest request) {
        FaceTemplate stored = faceTemplateRepository.findByUserUsername(username)
                .orElseThrow(() -> new BiometricAuthenticationException(
                        "Nenhum template facial cadastrado para o usuário"));

        // Decifrar template armazenado
        EncryptedTemplate encrypted = new EncryptedTemplate(
                stored.getTemplateData(), stored.getIv(), stored.getAuthTag(),
                stored.getKeyId(), stored.getAlgorithm());
        byte[] storedDescriptorBytes = cryptoService.decrypt(encrypted);

        // Decodificar template recebido na requisição
        byte[] receivedDescriptorBytes = Base64.getDecoder().decode(request.template());

        // Converter para float arrays
        float[] storedDescriptor = bytesToFloatArray(storedDescriptorBytes);
        float[] receivedDescriptor = bytesToFloatArray(receivedDescriptorBytes);

        // Calcular distância euclidiana
        double distance = euclideanDistance(storedDescriptor, receivedDescriptor);

        auditService.log(username, "FACE_AUTH_ATTEMPT",
                "Distância: %.4f, Threshold: %.4f, Resultado: %s"
                        .formatted(distance, similarityThreshold,
                                distance <= similarityThreshold ? "SUCESSO" : "FALHA"));

        if (distance > similarityThreshold) {
            throw new BiometricAuthenticationException(
                    "Autenticação facial falhou: rosto não corresponde");
        }

        String token = jwtTokenService.generateToken(username, Map.of(
                "auth_method", "FACE_BIOMETRIC",
                "biometric_confidence", String.valueOf(1.0 - distance)
        ));

        return new BiometricAuthResponse(token, "Bearer", 3600);
    }

    private double euclideanDistance(float[] a, float[] b) {
        if (a.length != b.length) {
            throw new InvalidBiometricDataException("Templates com dimensões incompatíveis");
        }
        double sum = 0.0;
        for (int i = 0; i < a.length; i++) {
            double diff = a[i] - b[i];
            sum += diff * diff;
        }
        return Math.sqrt(sum);
    }

    private float[] bytesToFloatArray(byte[] bytes) {
        FloatBuffer buffer = ByteBuffer.wrap(bytes)
                .order(ByteOrder.LITTLE_ENDIAN)
                .asFloatBuffer();
        float[] result = new float[buffer.remaining()];
        buffer.get(result);
        return result;
    }
}
```

### 7.2 Fluxo de Autenticação por Impressão Digital (WebAuthn)

```
┌──────────┐         ┌──────────────┐         ┌──────────────┐
│  React   │         │ Spring Boot  │         │  PostgreSQL   │
│ (browser)│         │   Backend    │         │               │
└────┬─────┘         └──────┬───────┘         └──────┬───────┘
     │  1. POST /auth/          │                    │
     │     webauthn/options     │                    │
     │  { username }            │                    │
     │─────────────────────────▶│  2. Buscar         │
     │                          │     credenciais    │
     │                          │────────────────────▶│
     │                          │◀────────────────────│
     │  3. { challenge,         │                    │
     │     allowCredentials }   │                    │
     │◀─────────────────────────│                    │
     │                          │                    │
     │  4. navigator.credentials│                    │
     │     .get() → biometria   │                    │
     │     do dispositivo       │                    │
     │                          │                    │
     │  5. POST /auth/          │                    │
     │     webauthn/verify      │                    │
     │  { assertion }           │                    │
     │─────────────────────────▶│                    │
     │                          │  6. Verificar      │
     │                          │     assinatura     │
     │                          │     com pub key    │
     │                          │                    │
     │  7. { JWT access_token } │                    │
     │◀─────────────────────────│                    │
```

**Componente React de autenticação:**

```tsx
async function authenticateWithFingerprint(username: string): Promise<string> {
  // 1. Obter challenge e credenciais permitidas do servidor
  const optionsResponse = await fetch("/api/v1/auth/webauthn/options", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ username }),
  });
  const options = await optionsResponse.json();

  // Decodificar campos Base64URL
  options.challenge = base64UrlToBuffer(options.challenge);
  options.allowCredentials = options.allowCredentials.map(
    (cred: { id: string; type: string }) => ({
      ...cred,
      id: base64UrlToBuffer(cred.id),
    })
  );

  // 2. Solicitar autenticação biométrica ao dispositivo
  const assertion = (await navigator.credentials.get({
    publicKey: options,
  })) as PublicKeyCredential;

  if (!assertion) throw new Error("Autenticação cancelada");

  const assertionResponse = assertion.response as AuthenticatorAssertionResponse;

  // 3. Enviar assinatura ao servidor
  const verifyResponse = await fetch("/api/v1/auth/webauthn/verify", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      credentialId: bufferToBase64Url(assertion.rawId),
      authenticatorData: bufferToBase64Url(assertionResponse.authenticatorData),
      clientDataJSON: bufferToBase64Url(assertionResponse.clientDataJSON),
      signature: bufferToBase64Url(assertionResponse.signature),
      userHandle: assertionResponse.userHandle
        ? bufferToBase64Url(assertionResponse.userHandle)
        : null,
    }),
  });

  if (!verifyResponse.ok) throw new Error("Autenticação biométrica falhou");

  const { accessToken } = await verifyResponse.json();
  return accessToken;
}
```

**Serviço de autenticação WebAuthn no backend:**

```java
@Service
public class WebAuthnAuthenticationService {

    private final RelyingParty relyingParty;
    private final WebAuthnCredentialRepository credentialRepository;
    private final JwtTokenService jwtTokenService;
    private final AuditService auditService;

    private final Map<String, AssertionRequest> pendingAuthentications =
            new ConcurrentHashMap<>();

    public AssertionRequest generateAuthenticationOptions(String username) {
        StartAssertionOptions options = StartAssertionOptions.builder()
                .username(username)
                .userVerification(UserVerificationRequirement.REQUIRED)
                .build();

        AssertionRequest assertionRequest = relyingParty.startAssertion(options);
        pendingAuthentications.put(username, assertionRequest);
        return assertionRequest;
    }

    public BiometricAuthResponse verifyAuthentication(String username,
                                                       WebAuthnAuthenticationRequest request) {
        AssertionRequest assertionRequest = pendingAuthentications.remove(username);
        if (assertionRequest == null) {
            throw new BiometricAuthenticationException(
                    "Nenhuma autenticação pendente para o usuário");
        }

        try {
            FinishAssertionOptions finishOptions = FinishAssertionOptions.builder()
                    .request(assertionRequest)
                    .response(parseAssertionResponse(request))
                    .build();

            AssertionResult result = relyingParty.finishAssertion(finishOptions);

            if (!result.isSuccess()) {
                auditService.log(username, "WEBAUTHN_AUTH_FAILED",
                        "Verificação de assinatura falhou");
                throw new BiometricAuthenticationException("Assinatura inválida");
            }

            // Atualizar sign count para detectar clonagem
            credentialRepository.updateSignCount(
                    result.getCredentialId().getBytes(),
                    result.getSignatureCount());

            auditService.log(username, "WEBAUTHN_AUTH_SUCCESS",
                    "Autenticação por impressão digital bem-sucedida");

            String token = jwtTokenService.generateToken(username, Map.of(
                    "auth_method", "FINGERPRINT_WEBAUTHN"
            ));

            return new BiometricAuthResponse(token, "Bearer", 3600);
        } catch (AssertionFailedException e) {
            throw new BiometricAuthenticationException("Falha na autenticação WebAuthn", e);
        }
    }

    private AuthenticatorAssertionResponse parseAssertionResponse(
            WebAuthnAuthenticationRequest request) {
        return AuthenticatorAssertionResponse.builder()
                .authenticatorData(ByteArray.fromBase64Url(request.authenticatorData()))
                .clientDataJSON(ByteArray.fromBase64Url(request.clientDataJSON()))
                .signature(ByteArray.fromBase64Url(request.signature()))
                .build();
    }
}
```

### 7.3 Integração com Spring Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final JwtTokenService jwtTokenService;

    @Bean
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        return http
                .securityMatcher("/api/**")
                .csrf(csrf -> csrf.disable())
                .sessionManagement(session ->
                        session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authorizeHttpRequests(auth -> auth
                        // Endpoints de autenticação biométrica são públicos
                        .requestMatchers("/api/v1/auth/face/**").permitAll()
                        .requestMatchers("/api/v1/auth/webauthn/**").permitAll()
                        // Cadastro de biometria exige autenticação prévia (senha/JWT)
                        .requestMatchers("/api/v1/biometric/**").authenticated()
                        // Direitos do titular exigem autenticação
                        .requestMatchers("/api/v1/biometric/titular/**").authenticated()
                        .anyRequest().authenticated()
                )
                .oauth2ResourceServer(oauth2 -> oauth2
                        .jwt(jwt -> jwt.decoder(jwtDecoder()))
                )
                .build();
    }

    @Bean
    public JwtDecoder jwtDecoder() {
        return jwtTokenService.jwtDecoder();
    }
}
```

**Controller de autenticação biométrica:**

```java
@RestController
@RequestMapping("/api/v1/auth")
public class BiometricAuthController {

    private final FaceAuthenticationService faceAuthService;
    private final WebAuthnAuthenticationService webAuthnAuthService;
    private final AuditService auditService;

    @PostMapping("/face")
    public ResponseEntity<BiometricAuthResponse> authenticateByFace(
            @Valid @RequestBody FaceAuthRequest request,
            @RequestParam String username,
            HttpServletRequest httpRequest) {

        auditService.log(username, "FACE_AUTH_INITIATED", 
                "IP: " + httpRequest.getRemoteAddr());

        BiometricAuthResponse response = faceAuthService.authenticate(username, request);
        return ResponseEntity.ok(response);
    }

    @PostMapping("/webauthn/options")
    public ResponseEntity<AssertionRequest> webauthnAuthOptions(
            @RequestParam String username) {

        AssertionRequest options = webAuthnAuthService.generateAuthenticationOptions(username);
        return ResponseEntity.ok(options);
    }

    @PostMapping("/webauthn/verify")
    public ResponseEntity<BiometricAuthResponse> webauthnAuthVerify(
            @Valid @RequestBody WebAuthnAuthenticationRequest request,
            @RequestParam String username,
            HttpServletRequest httpRequest) {

        auditService.log(username, "WEBAUTHN_AUTH_INITIATED",
                "IP: " + httpRequest.getRemoteAddr());

        BiometricAuthResponse response =
                webAuthnAuthService.verifyAuthentication(username, request);
        return ResponseEntity.ok(response);
    }
}
```

---

## 8. Liveness Detection — Prova de Vida

Liveness detection impede ataques de **spoofing** (uso de fotos, vídeos ou máscaras para enganar o sistema).

### Estratégias de implementação

| Nível | Técnica | Descrição |
|---|---|---|
| **Básico** | Challenge-response | Solicitar ação aleatória: piscar, virar a cabeça, sorrir |
| **Intermediário** | Análise de textura | Detectar artefatos de tela/impressão via análise de frequência |
| **Avançado** | 3D depth map | Usar câmera de profundidade (infravermelha) para verificar volume real do rosto |
| **Cloud** | APIs especializadas | Amazon Rekognition, Azure Face API, Google Cloud Vision |

**Exemplo de challenge-response no frontend:**

```tsx
const LIVENESS_CHALLENGES = [
  { action: "BLINK", instruction: "Pisque os dois olhos", detectFn: detectBlink },
  { action: "TURN_LEFT", instruction: "Vire o rosto para a esquerda", detectFn: detectTurnLeft },
  { action: "SMILE", instruction: "Sorria", detectFn: detectSmile },
] as const;

function LivenessCheck({ onVerified, onFailed }: LivenessProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [challenge, setChallenge] = useState<(typeof LIVENESS_CHALLENGES)[number] | null>(null);
  const [timeLeft, setTimeLeft] = useState(10);

  useEffect(() => {
    // Selecionar desafio aleatório
    const randomIndex = Math.floor(Math.random() * LIVENESS_CHALLENGES.length);
    setChallenge(LIVENESS_CHALLENGES[randomIndex]);
  }, []);

  useEffect(() => {
    if (!challenge || !videoRef.current) return;

    const interval = setInterval(async () => {
      if (!videoRef.current) return;

      const detections = await faceapi
        .detectSingleFace(videoRef.current)
        .withFaceLandmarks()
        .withFaceExpressions();

      if (detections && challenge.detectFn(detections)) {
        clearInterval(interval);
        onVerified();
      }
    }, 500);

    const timeout = setTimeout(() => {
      clearInterval(interval);
      onFailed("Tempo esgotado para prova de vida");
    }, 10000);

    return () => {
      clearInterval(interval);
      clearTimeout(timeout);
    };
  }, [challenge, onVerified, onFailed]);

  return (
    <div className="liveness-check">
      <h3>Prova de Vida</h3>
      <p>{challenge?.instruction}</p>
      <p>Tempo restante: {timeLeft}s</p>
      <video ref={videoRef} autoPlay muted playsInline />
    </div>
  );
}

function detectBlink(detection: faceapi.WithFaceExpressions<any>): boolean {
  // face-api.js não tem detecção direta de piscar,
  // mas landmarks dos olhos podem ser analisados
  // Em produção, usar SDK especializado (ex: FaceTec, iProov)
  return false; // placeholder — implementação depende do SDK
}
```

---

## 9. Auditoria e Logs de Acesso

A auditoria é obrigatória pela LGPD (Art. 6°, X — responsabilização) e essencial para investigação de incidentes.

```java
@Service
public class AuditService {

    private static final Logger auditLogger = LoggerFactory.getLogger("BIOMETRIC_AUDIT");

    private final BiometricAuditLogRepository auditLogRepository;

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(String username, String action, String detail) {
        // Persistir no banco para consultas estruturadas
        BiometricAuditLog entry = new BiometricAuditLog();
        entry.setUserId(resolveUserId(username));
        entry.setAction(action);
        entry.setDetail(detail);
        entry.setIpAddress(getCurrentIpAddress());
        entry.setCreatedAt(Instant.now());
        auditLogRepository.save(entry);

        // Log estruturado para SIEM/ELK
        auditLogger.info("action={} user={} detail={} ip={}",
                action, username, detail, getCurrentIpAddress());
    }

    private String getCurrentIpAddress() {
        ServletRequestAttributes attrs =
                (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        if (attrs == null) return "unknown";
        HttpServletRequest request = attrs.getRequest();
        String forwarded = request.getHeader("X-Forwarded-For");
        return forwarded != null ? forwarded.split(",")[0].trim() : request.getRemoteAddr();
    }

    private Long resolveUserId(String username) {
        // implementação simplificada
        return null;
    }
}
```

**Eventos auditados:**

| Evento | Quando |
|---|---|
| `CONSENT_GRANTED` | Titular autoriza coleta biométrica |
| `CONSENT_REVOKED` | Titular revoga consentimento |
| `FACE_REGISTERED` | Template facial cadastrado |
| `FACE_AUTH_INITIATED` | Tentativa de autenticação facial iniciada |
| `FACE_AUTH_SUCCESS` | Autenticação facial bem-sucedida |
| `FACE_AUTH_FAILED` | Autenticação facial falhou |
| `FINGERPRINT_REGISTERED` | Credencial WebAuthn cadastrada |
| `WEBAUTHN_AUTH_SUCCESS` | Autenticação por impressão digital bem-sucedida |
| `WEBAUTHN_AUTH_FAILED` | Autenticação por impressão digital falhou |
| `BIOMETRIC_DATA_DELETED` | Dados biométricos eliminados (Art. 18, VI) |
| `BIOMETRIC_DATA_EXPORTED` | Dados exportados pelo titular (Art. 19) |
| `LIVENESS_CHECK_PASSED` | Prova de vida aprovada |
| `LIVENESS_CHECK_FAILED` | Prova de vida reprovada |

---

## 10. Boas Práticas e Checklist de Segurança

### Armazenamento e Criptografia

- [ ] Templates biométricos cifrados com AES-256-GCM (ou superior)
- [ ] Chaves criptográficas em HSM ou serviço de gerenciamento de chaves (Vault, AWS KMS, Azure Key Vault)
- [ ] IV único e aleatório para cada operação de criptografia
- [ ] Rotação periódica de chaves (re-cifrar templates com nova chave)
- [ ] Imagens brutas descartadas imediatamente após extração do template
- [ ] Backups de templates também cifrados

### Comunicação

- [ ] TLS 1.3 obrigatório em todas as conexões
- [ ] Certificate pinning no aplicativo mobile (se aplicável)
- [ ] Rate limiting nos endpoints de autenticação biométrica
- [ ] Proteção contra replay attacks (nonce/challenge com TTL curto)

### LGPD

- [ ] Consentimento explícito e específico coletado antes da captura
- [ ] Finalidade declarada de forma clara e restrita
- [ ] RIPD (Relatório de Impacto à Proteção de Dados) documentado
- [ ] Endpoints de direitos do titular implementados (acesso, eliminação, portabilidade)
- [ ] Logs de auditoria completos e protegidos contra adulteração
- [ ] DPO (Encarregado de Dados) designado e acessível
- [ ] Política de retenção definida (por quanto tempo os templates são mantidos)
- [ ] Procedimento de notificação de incidentes à ANPD (Art. 48)

### Anti-Spoofing

- [ ] Liveness detection implementado (prova de vida)
- [ ] Testes com fotos, vídeos e máscaras para validar eficácia
- [ ] Limite de tentativas de autenticação (lockout temporário)
- [ ] Monitoramento de padrões anômalos (muitas tentativas de IPs diferentes)

### Testes e Qualidade

- [ ] Testes de viés algorítmico (diferentes etnias, gêneros, idades)
- [ ] Taxa de falso positivo (FPR) e falso negativo (FNR) documentadas
- [ ] Threshold de similaridade calibrado com dataset representativo
- [ ] Testes de penetração específicos para fluxo biométrico
- [ ] Fallback para autenticação não-biométrica (senha, OTP) sempre disponível

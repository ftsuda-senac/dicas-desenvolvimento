# Go (Golang) — Guia Prático para Desenvolvimento Web e Banco de Dados

Go (oficialmente denominada Go, popularmente conhecida como Golang) é uma linguagem compilada, estaticamente tipada, criada pelo Google em 2007 e tornada open-source em 2009. Seu design prioriza **simplicidade, desempenho e concorrência nativa**, tornando-a uma escolha popular para APIs de alta performance, microsserviços, CLIs e infraestrutura de nuvem.

---

## Sumário

1. [Por que Go?](#por-que-go)
2. [Casos de Uso e Ecossistema](#casos-de-uso-e-ecossistema)
3. [Ambiente de Desenvolvimento](#ambiente-de-desenvolvimento)
4. [IDEs e Ferramentas](#ides-e-ferramentas)
5. [Estrutura de um Projeto Go](#estrutura-de-um-projeto-go)
6. [Fundamentos da Linguagem](#fundamentos-da-linguagem)
   - [Visibilidade: Exportado vs. Não-exportado](#visibilidade-exportado-vs-não-exportado)
7. [Coleções de Dados](#coleções-de-dados)
8. [Funções e Métodos](#funções-e-métodos)
9. [Structs e Interfaces](#structs-e-interfaces)
10. [Tratamento de Erros](#tratamento-de-erros)
11. [Concorrência com Goroutines e Channels](#concorrência-com-goroutines-e-channels)
12. [Desenvolvimento Web](#desenvolvimento-web)
13. [Integração com Banco de Dados](#integração-com-banco-de-dados)
14. [Testes](#testes)
15. [Referências](#referências)

---

## Por que Go?

| Característica | Detalhe |
|---|---|
| **Compilação rápida** | Binários únicos e auto-contidos, sem dependência de runtime externo |
| **Concorrência nativa** | Goroutines são leves (poucos KB de memória) vs. threads do SO |
| **Tipagem estática** | Erros de tipo são detectados em tempo de compilação |
| **Coleta de lixo** | GC de baixa latência sem intervenção manual |
| **Tooling integrado** | Formatador (`gofmt`), testes, benchmarks e documentação na stdlib |
| **Ecossistema web** | Gin, Echo, Fiber, Chi — frameworks maduros para APIs REST |
| **Deploy simplificado** | Um binário estático que roda em containers mínimos (distroless, scratch) |

---

## Casos de Uso e Ecossistema

Go é uma linguagem de propósito geral com pontos fortes bem definidos. Além do desenvolvimento web, é amplamente adotada nas seguintes áreas:

### Ferramentas de Linha de Comando (CLIs)

O segundo uso mais popular de Go. O binário estático único sem dependência de runtime torna a distribuição trivial: basta copiar o executável. Ferramentas amplamente conhecidas escritas em Go:

| Ferramenta | Categoria |
|---|---|
| `kubectl` | Orquestração de containers (Kubernetes) |
| `terraform` | Infraestrutura como código |
| `gh` | GitHub CLI oficial |
| `hugo` | Gerador de sites estáticos |
| `golangci-lint` | Análise estática de código Go |
| `helm` | Gerenciador de pacotes Kubernetes |

**Bibliotecas recomendadas para CLIs:**

- **[Cobra](https://github.com/spf13/cobra)** — subcomandos, flags, autocompletion (base do `kubectl` e `gh`)
- **[Viper](https://github.com/spf13/viper)** — leitura de configuração via env vars, arquivos YAML/TOML e flags
- **[Bubble Tea](https://github.com/charmbracelet/bubbletea)** — TUIs interativas com modelo Elm

```bash
go get github.com/spf13/cobra@latest
go get github.com/spf13/viper
```

```go
// Exemplo mínimo com Cobra
package main

import (
    "fmt"
    "os"

    "github.com/spf13/cobra"
)

func main() {
    var porta int

    rootCmd := &cobra.Command{
        Use:   "minha-cli",
        Short: "Ferramenta de exemplo",
    }

    servirCmd := &cobra.Command{
        Use:   "servir",
        Short: "Iniciar servidor",
        Run: func(cmd *cobra.Command, args []string) {
            fmt.Printf("Servidor iniciado na porta %d\n", porta)
        },
    }

    servirCmd.Flags().IntVarP(&porta, "porta", "p", 8080, "porta do servidor")
    rootCmd.AddCommand(servirCmd)

    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}
```

```bash
./minha-cli servir --porta 9090
# Servidor iniciado na porta 9090

./minha-cli servir -p 3000
# Servidor iniciado na porta 3000
```

---

### Infraestrutura, DevOps e Cloud

Go é a linguagem dominante na infraestrutura moderna. A combinação de concorrência nativa, binário pequeno e baixo consumo de memória é ideal para daemons, agentes e sistemas que gerenciam recursos.

| Projeto | Descrição |
|---|---|
| **Docker** | Plataforma de containers |
| **Kubernetes** | Orquestração de containers |
| **Prometheus** | Monitoramento e alertas |
| **Grafana** | Visualização de métricas |
| **etcd** | Armazenamento distribuído de chave-valor |
| **Consul** | Service discovery e service mesh |
| **Vault** | Gerenciamento de segredos |

---

### Microsserviços e gRPC

Go tem suporte nativo excelente a **gRPC** — o protocolo RPC do Google baseado em Protocol Buffers (Protobuf). É amplamente usado para comunicação interna entre serviços onde performance e contrato de interface são mais importantes do que uma API REST pública.

```bash
go get google.golang.org/grpc
go get google.golang.org/protobuf
```

**Definição do serviço (`produto.proto`):**

```protobuf
syntax = "proto3";
package produto;
option go_package = "./proto";

service ProdutoService {
    rpc BuscarPorID (BuscarRequest) returns (ProdutoResponse);
    rpc Listar      (ListarRequest) returns (stream ProdutoResponse);
}

message BuscarRequest { int32 id = 1; }
message ListarRequest  {}

message ProdutoResponse {
    int32  id    = 1;
    string nome  = 2;
    double preco = 3;
}
```

```bash
# Gerar código Go a partir do .proto
protoc --go_out=. --go-grpc_out=. produto.proto
```

**Servidor gRPC:**

```go
package main

import (
    "context"
    "log"
    "net"

    "google.golang.org/grpc"
    pb "github.com/usuario/minha-api/proto"
)

type server struct {
    pb.UnimplementedProdutoServiceServer
}

func (s *server) BuscarPorID(_ context.Context, req *pb.BuscarRequest) (*pb.ProdutoResponse, error) {
    return &pb.ProdutoResponse{
        Id:    req.Id,
        Nome:  "Notebook",
        Preco: 3500.00,
    }, nil
}

func main() {
    lis, _ := net.Listen("tcp", ":50051")
    s := grpc.NewServer()
    pb.RegisterProdutoServiceServer(s, &server{})
    log.Println("gRPC server em :50051")
    log.Fatal(s.Serve(lis))
}
```

---

### Proxies Reversos e Gateways de API

Ferramentas como **Traefik** e **Caddy** são escritas inteiramente em Go. O modelo de concorrência por goroutines permite lidar com dezenas de milhares de conexões simultâneas com overhead mínimo de memória — sem thread pool fixo como em servidores Java ou Node.

| Projeto | Descrição |
|---|---|
| **Traefik** | Proxy reverso cloud-native com suporte automático a Let's Encrypt |
| **Caddy** | Servidor web com HTTPS automático |
| **Kong** (parcial) | Gateway de API empresarial |

---

### Processamento de Dados e Workers

Pipelines de ETL, consumidores de filas de mensagens e jobs de processamento em background são casos onde o modelo de goroutines + channels se encaixa naturalmente no padrão produtor/consumidor.

```go
package main

import (
    "fmt"
    "sync"
)

type Mensagem struct {
    ID   int
    Body string
}

// Worker pool: N goroutines processam uma fila de mensagens em paralelo
func workerPool(mensagens <-chan Mensagem, numWorkers int) {
    var wg sync.WaitGroup

    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()
            for msg := range mensagens {
                fmt.Printf("[worker %d] processando mensagem %d: %s\n",
                    workerID, msg.ID, msg.Body)
                // lógica de processamento...
            }
        }(i + 1)
    }

    wg.Wait()
}

func main() {
    fila := make(chan Mensagem, 100)

    // Produtor
    go func() {
        for i := 1; i <= 20; i++ {
            fila <- Mensagem{ID: i, Body: fmt.Sprintf("evento-%d", i)}
        }
        close(fila)
    }()

    workerPool(fila, 4) // 4 workers em paralelo
}
```

**Bibliotecas populares para mensageria:**

| Biblioteca | Broker suportado |
|---|---|
| `github.com/segmentio/kafka-go` | Apache Kafka |
| `github.com/rabbitmq/amqp091-go` | RabbitMQ |
| `github.com/nats-io/nats.go` | NATS |
| `cloud.google.com/go/pubsub` | Google Cloud Pub/Sub |

---

### Ferramentas de Segurança

A facilidade de manipular bytes, sockets e processos em baixo nível atrai desenvolvedores de segurança. Go é usado em scanners de rede, ferramentas de pentest, proxies de interceptação e fuzzing.

| Ferramenta | Descrição |
|---|---|
| **Nuclei** | Scanner de vulnerabilidades baseado em templates |
| **Subfinder** | Descoberta de subdomínios |
| **httpx** | Probe HTTP de alta performance |
| **ffuf** | Fuzzer web rápido |

---

### Onde Go *não* é a melhor escolha

| Caso de uso | Alternativa mais adequada | Motivo |
|---|---|---|
| Machine Learning / Data Science | Python | NumPy, PyTorch, scikit-learn não têm equivalentes em Go |
| GUIs desktop | Kotlin, Swift, Flutter | Ecossistema de UI nativa em Go é imaturo |
| Scripts de automação pontuais | Python, Bash | Necessidade de compilar antes de executar é improdutiva |
| Sistemas de muito baixo nível (drivers, OS) | Rust, C | GC impede controle total de memória e latência determinística |
| WebAssembly (frontend) | Rust, TypeScript | Binário WASM gerado por Go ainda é grande |

---

## Ambiente de Desenvolvimento

### 1. Instalação do Go

**Linux / macOS**

```bash
# Baixar e instalar (substituir pela versão mais recente em go.dev/dl)
wget https://go.dev/dl/go1.24.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.24.0.linux-amd64.tar.gz

# Adicionar ao PATH (~/.bashrc ou ~/.zshrc)
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

source ~/.zshrc   # ou source ~/.bashrc
go version        # confirmar instalação
```

**Windows**

1. Baixar o instalador `.msi` em [go.dev/dl](https://go.dev/dl)
2. Executar o instalador — ele configura o `PATH` automaticamente
3. Abrir um novo terminal e verificar:

```powershell
go version
# go version go1.24.0 windows/amd64
```

**macOS com Homebrew**

```bash
brew install go
go version
```

### 2. Variáveis de ambiente relevantes

| Variável | Descrição | Padrão |
|---|---|---|
| `GOROOT` | Onde o Go está instalado | Definido pelo instalador |
| `GOPATH` | Raiz dos módulos baixados e binários | `~/go` |
| `GOBIN` | Onde `go install` deposita binários | `$GOPATH/bin` |
| `GOPROXY` | Proxy de módulos (relevante em redes corporativas) | `https://proxy.golang.org` |
| `GONOSUMCHECK` | Ignora verificação de checksum (ambientes privados) | — |

### 3. Ferramentas essenciais do toolchain

```bash
# Formatar código (obrigatório antes de commit)
gofmt -w .

# Análise estática incluída no SDK
go vet ./...

# Gerenciar dependências
go mod tidy      # limpar e sincronizar go.sum
go mod download  # baixar dependências offline

# Build e run
go build -o minha-api ./cmd/api
go run ./cmd/api

# Instalar um binário externo (ex.: linter)
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

---

## IDEs e Ferramentas

### Visual Studio Code

A opção mais popular para iniciantes e equipes mistas.

**Extensões recomendadas:**

| Extensão | Publisher | Função |
|---|---|---|
| **Go** | Go Team at Google | IntelliSense, debug, formatação automática, snippets |
| **Error Lens** | Alexander | Exibe erros inline no editor |
| **REST Client** | Huachao Mao | Testar APIs `.http` diretamente no editor |
| **Docker** | Microsoft | Integração com Dockerfile e Compose |

**Configuração sugerida (`settings.json`):**

```json
{
  "go.useLanguageServer": true,
  "go.lintTool": "golangci-lint",
  "go.formatTool": "goimports",
  "[go]": {
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  }
}
```

### GoLand (JetBrains)

IDE comercial específica para Go — recomendada para projetos maiores. Inclui refatoração avançada, depurador integrado, suporte a Docker/Kubernetes e integração com bancos de dados. Versão Community disponível para estudantes.

### Neovim / Vim

Para usuários avançados, o plugin **nvim-lspconfig** com o language server **gopls** oferece experiência equivalente ao VS Code via linha de comando.

---

## Estrutura de um Projeto Go

### Inicializar um módulo

```bash
mkdir minha-api && cd minha-api
go mod init github.com/usuario/minha-api
```

O arquivo `go.mod` gerado funciona como o `pom.xml` (Maven) ou `package.json` (Node) — declara o módulo e as dependências.

### Layout recomendado (projeto web)

```
minha-api/
├── cmd/
│   └── api/
│       └── main.go          # ponto de entrada
├── internal/
│   ├── handler/             # controllers HTTP
│   ├── service/             # lógica de negócio
│   ├── repository/          # acesso a dados
│   └── model/               # structs de domínio
├── config/
│   └── config.go            # leitura de variáveis de ambiente
├── migrations/              # scripts SQL de migração
├── go.mod
├── go.sum
└── Dockerfile
```

> **`internal/`** é especial no Go: pacotes dentro dele só podem ser importados por código no mesmo módulo — garante encapsulamento sem precisar de modificadores de acesso.

---

## Fundamentos da Linguagem

### Variáveis e Tipos

```go
package main

import "fmt"

func main() {
    // Declaração explícita
    var nome string = "Maria"
    var idade int = 30
    var ativo bool = true

    // Declaração curta (inferência de tipo — mais comum)
    cidade := "São Paulo"
    salario := 5500.75

    // Constantes
    const Pi = 3.14159
    const MaxTentativas = 3

    fmt.Println(nome, idade, ativo, cidade, salario, Pi, MaxTentativas)

    // Tipos numéricos comuns
    var i int     = 42        // int nativo (64-bit em plataformas 64-bit)
    var i32 int32 = 100
    var f64 float64 = 3.14
    var u uint = 255          // sem sinal

    // Conversão explícita (Go não faz coerção implícita)
    total := float64(i) + f64
    fmt.Println(i32, total, u)
}
```

### Tipos compostos básicos

```go
// string é imutável, UTF-8 nativo
s := "Olá, mundo!"
fmt.Println(len(s))         // comprimento em bytes
fmt.Println([]rune(s)[0])   // acesso por rune (Unicode code point)

// Conversão string ↔ []byte
b := []byte(s)
s2 := string(b)
fmt.Println(s2)

// Múltiplos retornos (idioma central do Go)
q, r := divide(10, 3)
fmt.Println(q, r) // 3 1

func divide(a, b int) (int, int) {
    return a / b, a % b
}
```

### Visibilidade: Exportado vs. Não-exportado

Em Go não existem palavras-chave `public`, `private` ou `protected`. A visibilidade de um identificador é determinada exclusivamente pela **primeira letra do seu nome**:

| Primeira letra | Visibilidade | Equivalente em outras linguagens |
|---|---|---|
| **Maiúscula** (`Produto`, `Save`, `MaxSize`) | Exportado — visível fora do pacote | `public` |
| **Minúscula** (`produto`, `save`, `maxSize`) | Não-exportado — visível apenas dentro do pacote | `private` |

Essa regra se aplica a **todos os identificadores** da linguagem:

```go
package estoque

import "fmt"

// Exportado: pode ser usado por outros pacotes
const QuantidadeMaxima = 100

// Não-exportado: uso restrito a este pacote
const quantidadeMinima = 1

// Exportado
type Produto struct {
    ID    int     // exportado — acessível fora do pacote
    Nome  string  // exportado
    preco float64 // não-exportado — só acessível dentro do pacote estoque
}

// Exportado — construtora pública
func NovoProduto(id int, nome string, preco float64) *Produto {
    return &Produto{ID: id, Nome: nome, preco: preco}
}

// Exportado
func (p *Produto) Preco() float64 {
    return p.preco // acessa o campo não-exportado dentro do próprio pacote
}

// Exportado
func (p *Produto) AplicarDesconto(percentual float64) {
    p.preco = aplicarCalculo(p.preco, percentual) // chama função não-exportada
}

// Não-exportada — detalhe de implementação interna
func aplicarCalculo(valor, percentual float64) float64 {
    return valor * (1 - percentual/100)
}

// Não-exportada — uso interno ao pacote
type estoqueInterno struct {
    itens map[int]*Produto
}
```

**Tentativa de acesso a membro não-exportado fora do pacote:**

```go
package main

import "github.com/usuario/minha-api/estoque"

func main() {
    p := estoque.NovoProduto(1, "Notebook", 3500.00)

    fmt.Println(p.ID)       // OK — exportado
    fmt.Println(p.Nome)     // OK — exportado
    fmt.Println(p.Preco())  // OK — método exportado

    // fmt.Println(p.preco) // ERRO de compilação:
                            // p.preco undefined (cannot refer to unexported field)
}
```

**Visibilidade em interfaces:**

```go
// Interface exportada com método exportado — pode ser implementada por qualquer pacote
type Repositorio interface {
    Salvar(id int) error   // exportado
    Buscar(id int) error   // exportado
}

// Interface não-exportada — uso interno ao pacote
type cache interface {
    gravar(chave string, valor []byte)
    ler(chave string) ([]byte, bool)
}
```

**Visibilidade de campos em structs com JSON/encoding:**

```go
type Config struct {
    Host     string `json:"host"`      // exportado → aparece no JSON
    Porta    int    `json:"porta"`     // exportado → aparece no JSON
    senha    string                    // não-exportado → ignorado por json.Marshal/Unmarshal
    apiToken string                    // não-exportado → nunca serializado acidentalmente
}
```

> **Boas práticas:**
> - Exponha apenas o que precisa ser público — o mínimo necessário
> - Campos de struct sensíveis (senhas, tokens) devem ser não-exportados, com acesso via métodos *getters*
> - Use `internal/` para encapsulamento em nível de módulo: pacotes dentro de `internal/` só podem ser importados por código do mesmo módulo, independentemente de terem identificadores exportados

---

### Condicionais

```go
package main

import "fmt"

func classificarIdade(idade int) string {
    // if/else — sem parênteses na condição
    if idade < 0 {
        return "inválido"
    } else if idade < 12 {
        return "criança"
    } else if idade < 18 {
        return "adolescente"
    } else if idade < 60 {
        return "adulto"
    } else {
        return "idoso"
    }
}

func diaDaSemana(dia int) string {
    // switch sem break explícito (não há fall-through por padrão)
    switch dia {
    case 1:
        return "domingo"
    case 2:
        return "segunda-feira"
    case 3:
        return "terça-feira"
    case 4:
        return "quarta-feira"
    case 5:
        return "quinta-feira"
    case 6:
        return "sexta-feira"
    case 7:
        return "sábado"
    default:
        return "desconhecido"
    }
}

func categorizarNota(nota float64) string {
    // switch sem expressão funciona como if/else encadeado
    switch {
    case nota >= 9.0:
        return "A"
    case nota >= 7.0:
        return "B"
    case nota >= 5.0:
        return "C"
    default:
        return "F"
    }
}

func main() {
    fmt.Println(classificarIdade(25))   // adulto
    fmt.Println(diaDaSemana(6))         // sexta-feira
    fmt.Println(categorizarNota(8.5))   // B
}
```

### Loops

Go tem apenas `for` — mas ele é flexível o suficiente para substituir `while` e `do/while`.

```go
package main

import "fmt"

func main() {
    // Loop clássico (C-style)
    for i := 0; i < 5; i++ {
        fmt.Println("i =", i)
    }

    // Loop como while
    n := 1
    for n < 100 {
        n *= 2
    }
    fmt.Println("n =", n) // 128

    // Loop infinito (interrompido por break)
    contador := 0
    for {
        contador++
        if contador == 5 {
            break
        }
    }
    fmt.Println("contador =", contador)

    // range — itera sobre slice, array, map, string, channel
    frutas := []string{"maçã", "banana", "laranja"}
    for i, fruta := range frutas {
        fmt.Printf("[%d] %s\n", i, fruta)
    }

    // Ignorar índice com _
    for _, fruta := range frutas {
        fmt.Println(fruta)
    }

    // range em map (ordem não garantida)
    capitais := map[string]string{
        "BR": "Brasília",
        "PT": "Lisboa",
        "AR": "Buenos Aires",
    }
    for pais, capital := range capitais {
        fmt.Printf("%s → %s\n", pais, capital)
    }

    // continue — pular iteração
    for i := 0; i < 10; i++ {
        if i%2 == 0 {
            continue // pular pares
        }
        fmt.Println(i) // imprime 1, 3, 5, 7, 9
    }
}
```

---

## Coleções de Dados

### Arrays

Arrays têm tamanho fixo em tempo de compilação. São raramente usados diretamente — prefere-se slices.

```go
// Array de tamanho 5
var notas [5]float64 = [5]float64{9.5, 8.0, 7.5, 6.0, 5.5}
fmt.Println(notas[0]) // 9.5
fmt.Println(len(notas)) // 5
```

### Slices

Slices são a estrutura de sequência padrão do Go — fatias dinâmicas sobre um array subjacente.

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    // Criação com literal
    numeros := []int{10, 20, 30, 40, 50}

    // Criação com make(tipo, len, cap)
    nomes := make([]string, 0, 10) // len=0, cap=10

    // append — adiciona elementos (pode realocal o array interno)
    nomes = append(nomes, "Ana", "Bruno", "Carlos")
    fmt.Println(nomes) // [Ana Bruno Carlos]

    // Fatiamento [inicio:fim] — fim exclusivo
    sub := numeros[1:4] // [20 30 40]
    fmt.Println(sub)

    // copy — copia elementos entre slices
    destino := make([]int, 3)
    copiados := copy(destino, numeros)
    fmt.Println(destino, "copiados:", copiados)

    // Ordenação
    dados := []int{5, 2, 8, 1, 9, 3}
    sort.Ints(dados)
    fmt.Println(dados) // [1 2 3 5 8 9]

    nomesBagunçados := []string{"Zé", "Ana", "Maria", "João"}
    sort.Strings(nomesBagunçados)
    fmt.Println(nomesBagunçados) // [Ana João Maria Zé]

    // Filtrar com range (idioma funcional)
    pares := filtrar(numeros, func(n int) bool { return n%20 == 0 })
    fmt.Println(pares) // [20 40]
}

func filtrar(s []int, pred func(int) bool) []int {
    result := make([]int, 0)
    for _, v := range s {
        if pred(v) {
            result = append(result, v)
        }
    }
    return result
}
```

### Maps

Dicionários hash com chaves e valores de qualquer tipo comparável.

```go
package main

import "fmt"

func main() {
    // Criação com literal
    populacao := map[string]int{
        "São Paulo":      12325000,
        "Rio de Janeiro": 6775000,
        "Brasília":       3094000,
    }

    // Acesso
    fmt.Println(populacao["Brasília"]) // 3094000

    // Verificar existência (two-value assignment)
    pop, ok := populacao["Manaus"]
    if ok {
        fmt.Println("Manaus:", pop)
    } else {
        fmt.Println("Manaus não encontrada")
    }

    // Adicionar / atualizar
    populacao["Manaus"] = 2255000
    populacao["São Paulo"] = 12400000

    // Deletar
    delete(populacao, "Brasília")

    // Iterar
    for cidade, pop := range populacao {
        fmt.Printf("%s: %d habitantes\n", cidade, pop)
    }

    // Map como conjunto (set)
    visitados := map[string]struct{}{}
    visitados["São Paulo"] = struct{}{}
    visitados["Manaus"] = struct{}{}

    _, jaVisitou := visitados["Manaus"]
    fmt.Println("Já visitou Manaus?", jaVisitou)
}
```

---

## Funções e Métodos

### Funções de primeira classe

```go
package main

import (
    "fmt"
    "strings"
)

// Função variádica
func soma(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// Retornos nomeados
func minMax(nums []int) (min, max int) {
    min, max = nums[0], nums[0]
    for _, n := range nums[1:] {
        if n < min {
            min = n
        }
        if n > max {
            max = n
        }
    }
    return // retorno implícito dos valores nomeados
}

// Função como parâmetro (higher-order function)
func aplicar(s []string, fn func(string) string) []string {
    result := make([]string, len(s))
    for i, v := range s {
        result[i] = fn(v)
    }
    return result
}

// Closure
func contador() func() int {
    n := 0
    return func() int {
        n++
        return n
    }
}

func main() {
    fmt.Println(soma(1, 2, 3, 4, 5)) // 15

    nums := []int{3, 1, 4, 1, 5, 9, 2, 6}
    min, max := minMax(nums)
    fmt.Printf("min=%d max=%d\n", min, max)

    palavras := []string{"go", "é", "incrível"}
    maiusculas := aplicar(palavras, strings.ToUpper)
    fmt.Println(maiusculas) // [GO É INCRÍVEL]

    contar := contador()
    fmt.Println(contar(), contar(), contar()) // 1 2 3
}
```

### Defer, Panic e Recover

```go
package main

import (
    "fmt"
    "os"
)

// defer: executa ao sair da função (LIFO)
func lerArquivo(caminho string) error {
    f, err := os.Open(caminho)
    if err != nil {
        return err
    }
    defer f.Close() // garantia de fechamento, independente do fluxo

    // ... processar arquivo
    return nil
}

// recover: captura panics (semelhante a try/catch, mas raramente necessário)
func safeDiv(a, b int) (resultado int, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic capturado: %v", r)
        }
    }()
    resultado = a / b // panic se b == 0
    return
}

func main() {
    res, err := safeDiv(10, 0)
    fmt.Println(res, err) // 0 panic capturado: runtime error: integer divide by zero
}
```

---

## Structs e Interfaces

### Structs

Go não tem classes — structs + métodos cumprem o mesmo papel.

```go
package main

import (
    "fmt"
    "math"
)

// Struct com tags JSON (usadas por encoding/json e ORMs)
type Produto struct {
    ID       int     `json:"id"`
    Nome     string  `json:"nome"`
    Preco    float64 `json:"preco"`
    Estoque  int     `json:"estoque"`
    Ativo    bool    `json:"ativo"`
}

// Método no receiver por valor (não modifica o original)
func (p Produto) String() string {
    return fmt.Sprintf("[%d] %s — R$ %.2f", p.ID, p.Nome, p.Preco)
}

// Método no receiver por ponteiro (modifica o original)
func (p *Produto) AplicarDesconto(percentual float64) {
    p.Preco *= (1 - percentual/100)
}

// Construtor convencional
func NovoProduto(id int, nome string, preco float64) *Produto {
    return &Produto{
        ID:      id,
        Nome:    nome,
        Preco:   preco,
        Estoque: 0,
        Ativo:   true,
    }
}

// Struct aninhada (composição)
type Pedido struct {
    ID     int
    Itens  []ItemPedido
}

type ItemPedido struct {
    Produto  Produto
    Qtd      int
}

func (p Pedido) Total() float64 {
    var total float64
    for _, item := range p.Itens {
        total += item.Produto.Preco * float64(item.Qtd)
    }
    return total
}

// Embedding (herança por composição)
type Animal struct {
    Nome string
}

func (a Animal) Descrever() string {
    return "Sou " + a.Nome
}

type Cachorro struct {
    Animal           // campos e métodos de Animal são promovidos
    Raca  string
}

func main() {
    p := NovoProduto(1, "Notebook", 3500.00)
    fmt.Println(p)          // [1] Notebook — R$ 3500.00
    p.AplicarDesconto(10)
    fmt.Println(p.Preco)    // 3150

    pedido := Pedido{
        ID: 42,
        Itens: []ItemPedido{
            {Produto: *p, Qtd: 2},
        },
    }
    fmt.Printf("Total: R$ %.2f\n", pedido.Total())

    dog := Cachorro{
        Animal: Animal{Nome: "Rex"},
        Raca:   "Labrador",
    }
    fmt.Println(dog.Descrever()) // Sou Rex (método promovido)

    // Struct anônima (util para dados temporários / testes)
    config := struct {
        Host string
        Port int
    }{"localhost", 8080}
    fmt.Println(config.Host, config.Port)
}

// Exemplo com interface Stringer da stdlib
// (qualquer tipo que implemente String() string satisfaz fmt.Stringer)
var _ fmt.Stringer = (*Produto)(nil) // verificação em tempo de compilação

type Circulo struct{ Raio float64 }
type Retangulo struct{ Largura, Altura float64 }

type Forma interface {
    Area() float64
    Perimetro() float64
}

func (c Circulo) Area() float64      { return math.Pi * c.Raio * c.Raio }
func (c Circulo) Perimetro() float64 { return 2 * math.Pi * c.Raio }

func (r Retangulo) Area() float64      { return r.Largura * r.Altura }
func (r Retangulo) Perimetro() float64 { return 2 * (r.Largura + r.Altura) }

func imprimirForma(f Forma) {
    fmt.Printf("Área: %.2f | Perímetro: %.2f\n", f.Area(), f.Perimetro())
}
```

### Interfaces — polimorfismo implícito

Go usa **duck typing estrutural**: um tipo satisfaz uma interface automaticamente ao implementar seus métodos, sem declaração explícita (`implements`).

```go
package main

import (
    "fmt"
    "strings"
)

// Interface de repositório (facilita testes com mocks)
type UsuarioRepository interface {
    BuscarPorID(id int) (*Usuario, error)
    Salvar(u *Usuario) error
    Listar() ([]*Usuario, error)
}

type Usuario struct {
    ID    int
    Nome  string
    Email string
}

// Implementação real (banco de dados)
type PostgresUsuarioRepository struct {
    // db *sql.DB
}

func (r *PostgresUsuarioRepository) BuscarPorID(id int) (*Usuario, error) {
    // query real ao banco
    return &Usuario{ID: id, Nome: "Demo"}, nil
}

func (r *PostgresUsuarioRepository) Salvar(u *Usuario) error   { return nil }
func (r *PostgresUsuarioRepository) Listar() ([]*Usuario, error) { return nil, nil }

// Implementação em memória (para testes)
type InMemoryUsuarioRepository struct {
    dados map[int]*Usuario
}

func (r *InMemoryUsuarioRepository) BuscarPorID(id int) (*Usuario, error) {
    u, ok := r.dados[id]
    if !ok {
        return nil, fmt.Errorf("usuário %d não encontrado", id)
    }
    return u, nil
}

func (r *InMemoryUsuarioRepository) Salvar(u *Usuario) error {
    r.dados[u.ID] = u
    return nil
}

func (r *InMemoryUsuarioRepository) Listar() ([]*Usuario, error) {
    result := make([]*Usuario, 0, len(r.dados))
    for _, u := range r.dados {
        result = append(result, u)
    }
    return result, nil
}

// Serviço depende da interface, não da implementação concreta
type UsuarioService struct {
    repo UsuarioRepository
}

func (s *UsuarioService) BuscarPorID(id int) (*Usuario, error) {
    if id <= 0 {
        return nil, fmt.Errorf("id inválido: %d", id)
    }
    return s.repo.BuscarPorID(id)
}

// Type assertion e type switch
func inspecionar(v interface{}) {
    switch t := v.(type) {
    case int:
        fmt.Printf("int: %d\n", t)
    case string:
        fmt.Printf("string: %q (upper: %s)\n", t, strings.ToUpper(t))
    case []int:
        fmt.Printf("[]int com %d elementos\n", len(t))
    default:
        fmt.Printf("tipo desconhecido: %T\n", t)
    }
}

func main() {
    inspecionar(42)
    inspecionar("golang")
    inspecionar([]int{1, 2, 3})
    inspecionar(3.14)
}
```

---

## Tratamento de Erros

Go trata erros como valores — sem exceções. O padrão idiomático é retornar `(resultado, error)`.

```go
package main

import (
    "errors"
    "fmt"
)

// Sentinel errors — erros nomeados para comparação
var (
    ErrNaoEncontrado  = errors.New("registro não encontrado")
    ErrPermissaoNegada = errors.New("permissão negada")
)

// Erro customizado com contexto
type ValidationError struct {
    Campo   string
    Mensagem string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validação falhou: campo '%s' — %s", e.Campo, e.Mensagem)
}

func buscarUsuario(id int) (string, error) {
    if id <= 0 {
        return "", &ValidationError{Campo: "id", Mensagem: "deve ser positivo"}
    }
    if id > 100 {
        return "", fmt.Errorf("buscarUsuario: %w", ErrNaoEncontrado) // wrap
    }
    return fmt.Sprintf("Usuário#%d", id), nil
}

func main() {
    // Padrão idiomático: checar erro imediatamente
    nome, err := buscarUsuario(50)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    fmt.Println("Encontrado:", nome)

    // errors.Is — verifica a cadeia de wrapping
    _, err = buscarUsuario(200)
    if errors.Is(err, ErrNaoEncontrado) {
        fmt.Println("Não encontrado!")
    }

    // errors.As — extrai tipo específico do erro
    _, err = buscarUsuario(-1)
    var valErr *ValidationError
    if errors.As(err, &valErr) {
        fmt.Printf("Erro de validação no campo: %s\n", valErr.Campo)
    }

    // Encadeamento de erros com contexto adicional
    _, err = buscarUsuario(999)
    err2 := fmt.Errorf("camada de serviço: %w", err)
    fmt.Println(err2) // camada de serviço: buscarUsuario: registro não encontrado
}
```

---

## Concorrência com Goroutines e Channels

### Goroutines

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func tarefa(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Printf("Tarefa %d iniciada\n", id)
    time.Sleep(100 * time.Millisecond) // simular trabalho
    fmt.Printf("Tarefa %d concluída\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go tarefa(i, &wg) // inicia goroutine (leve, ~2KB de stack inicial)
    }

    wg.Wait() // aguardar todas terminarem
    fmt.Println("Todas as tarefas concluídas")
}
```

### Channels

```go
package main

import (
    "fmt"
)

func gerador(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n // envia
        }
        close(out) // sinaliza que não há mais dados
    }()
    return out
}

func quadrado(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in { // range em channel lê até close()
            out <- n * n
        }
        close(out)
    }()
    return out
}

func main() {
    // Pipeline: gerador → quadrado → impressão
    nums := gerador(2, 3, 4, 5)
    quadrados := quadrado(nums)

    for n := range quadrados {
        fmt.Println(n) // 4, 9, 16, 25
    }

    // select — multiplexar channels
    c1 := make(chan string)
    c2 := make(chan string)

    go func() { c1 <- "canal 1" }()
    go func() { c2 <- "canal 2" }()

    for i := 0; i < 2; i++ {
        select {
        case msg := <-c1:
            fmt.Println("Recebido de", msg)
        case msg := <-c2:
            fmt.Println("Recebido de", msg)
        }
    }
}
```

### sync.Mutex e atomic (acesso concorrente a dados)

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

type Contador struct {
    mu    sync.Mutex
    valor int
}

func (c *Contador) Incrementar() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.valor++
}

func (c *Contador) Valor() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.valor
}

func main() {
    c := &Contador{}
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            c.Incrementar()
        }()
    }

    wg.Wait()
    fmt.Println("Contador:", c.Valor()) // sempre 1000

    // Para contadores simples, atomic é mais eficiente
    var n int64
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            atomic.AddInt64(&n, 1)
        }()
    }
    wg.Wait()
    fmt.Println("Atomic:", n) // sempre 1000
}
```

---

## Desenvolvimento Web

### net/http (stdlib)

A biblioteca padrão já é suficiente para APIs simples.

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "strconv"
)

type Produto struct {
    ID    int     `json:"id"`
    Nome  string  `json:"nome"`
    Preco float64 `json:"preco"`
}

var produtos = []Produto{
    {1, "Notebook", 3500.00},
    {2, "Mouse", 89.90},
    {3, "Teclado", 199.00},
}

func responderJSON(w http.ResponseWriter, status int, dado interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(dado)
}

func listarProdutos(w http.ResponseWriter, r *http.Request) {
    responderJSON(w, http.StatusOK, produtos)
}

func buscarProduto(w http.ResponseWriter, r *http.Request) {
    idStr := r.PathValue("id") // Go 1.22+
    id, err := strconv.Atoi(idStr)
    if err != nil {
        http.Error(w, "id inválido", http.StatusBadRequest)
        return
    }
    for _, p := range produtos {
        if p.ID == id {
            responderJSON(w, http.StatusOK, p)
            return
        }
    }
    http.Error(w, "não encontrado", http.StatusNotFound)
}

func criarProduto(w http.ResponseWriter, r *http.Request) {
    var p Produto
    if err := json.NewDecoder(r.Body).Decode(&p); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    p.ID = len(produtos) + 1
    produtos = append(produtos, p)
    responderJSON(w, http.StatusCreated, p)
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("GET /produtos", listarProdutos)
    mux.HandleFunc("GET /produtos/{id}", buscarProduto)
    mux.HandleFunc("POST /produtos", criarProduto)

    log.Println("Servidor rodando em :8080")
    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

### Gin — Framework web de alta performance

Gin é o framework web mais popular do ecossistema Go — roteamento rápido, middlewares, binding automático de JSON/form.

```bash
go get github.com/gin-gonic/gin
```

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

type CriarProdutoDTO struct {
    Nome  string  `json:"nome"  binding:"required,min=2,max=100"`
    Preco float64 `json:"preco" binding:"required,gt=0"`
}

type Produto struct {
    ID    int     `json:"id"`
    Nome  string  `json:"nome"`
    Preco float64 `json:"preco"`
}

var db = []Produto{
    {1, "Notebook", 3500.00},
    {2, "Mouse", 89.90},
}

func main() {
    r := gin.Default() // inclui Logger e Recovery middleware

    // Grupo de rotas com prefixo /api/v1
    v1 := r.Group("/api/v1")
    {
        v1.GET("/produtos", listarProdutos)
        v1.GET("/produtos/:id", buscarProduto)
        v1.POST("/produtos", criarProduto)
        v1.PUT("/produtos/:id", atualizarProduto)
        v1.DELETE("/produtos/:id", deletarProduto)
    }

    r.Run(":8080") // escuta em 0.0.0.0:8080
}

func listarProdutos(c *gin.Context) {
    c.JSON(http.StatusOK, db)
}

func buscarProduto(c *gin.Context) {
    id := c.Param("id")
    for _, p := range db {
        if strconv.Itoa(p.ID) == id {
            c.JSON(http.StatusOK, p)
            return
        }
    }
    c.JSON(http.StatusNotFound, gin.H{"erro": "produto não encontrado"})
}

func criarProduto(c *gin.Context) {
    var dto CriarProdutoDTO
    if err := c.ShouldBindJSON(&dto); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"erro": err.Error()})
        return
    }
    p := Produto{
        ID:    len(db) + 1,
        Nome:  dto.Nome,
        Preco: dto.Preco,
    }
    db = append(db, p)
    c.JSON(http.StatusCreated, p)
}

func atualizarProduto(c *gin.Context) {
    id := c.Param("id")
    var dto CriarProdutoDTO
    if err := c.ShouldBindJSON(&dto); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"erro": err.Error()})
        return
    }
    for i, p := range db {
        if strconv.Itoa(p.ID) == id {
            db[i].Nome = dto.Nome
            db[i].Preco = dto.Preco
            c.JSON(http.StatusOK, db[i])
            return
        }
    }
    c.JSON(http.StatusNotFound, gin.H{"erro": "produto não encontrado"})
}

func deletarProduto(c *gin.Context) {
    id := c.Param("id")
    for i, p := range db {
        if strconv.Itoa(p.ID) == id {
            db = append(db[:i], db[i+1:]...)
            c.Status(http.StatusNoContent)
            return
        }
    }
    c.JSON(http.StatusNotFound, gin.H{"erro": "produto não encontrado"})
}
```

### Middlewares com Gin

```go
package main

import (
    "log"
    "net/http"
    "time"

    "github.com/gin-gonic/gin"
)

// Middleware de logging customizado
func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        c.Next() // executar próximos handlers
        latencia := time.Since(start)
        log.Printf("[%d] %s %s | %v", c.Writer.Status(), c.Request.Method, c.Request.URL.Path, latencia)
    }
}

// Middleware de autenticação simples (Bearer token)
func Auth(tokenValido string) gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token != "Bearer "+tokenValido {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"erro": "não autorizado"})
            return
        }
        c.Next()
    }
}

// Middleware CORS
func CORS() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Access-Control-Allow-Origin", "*")
        c.Header("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,OPTIONS")
        c.Header("Access-Control-Allow-Headers", "Content-Type,Authorization")

        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(http.StatusNoContent)
            return
        }
        c.Next()
    }
}

func main() {
    r := gin.New()
    r.Use(gin.Recovery())
    r.Use(Logger())
    r.Use(CORS())

    publico := r.Group("/api")
    publico.GET("/health", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"status": "ok"})
    })

    privado := r.Group("/api")
    privado.Use(Auth("meu-token-secreto"))
    {
        privado.GET("/perfil", func(c *gin.Context) {
            c.JSON(http.StatusOK, gin.H{"usuario": "admin"})
        })
    }

    r.Run(":8080")
}
```

### Estrutura de projeto completa com Gin (Clean Architecture)

```
cmd/
└── api/
    └── main.go

internal/
├── handler/
│   └── produto_handler.go
├── service/
│   └── produto_service.go
├── repository/
│   └── produto_repository.go
└── model/
    └── produto.go
```

**model/produto.go**

```go
package model

type Produto struct {
    ID    int     `json:"id"    db:"id"`
    Nome  string  `json:"nome"  db:"nome"`
    Preco float64 `json:"preco" db:"preco"`
}

type CriarProdutoDTO struct {
    Nome  string  `json:"nome"  binding:"required,min=2"`
    Preco float64 `json:"preco" binding:"required,gt=0"`
}
```

**repository/produto_repository.go**

```go
package repository

import (
    "database/sql"
    "github.com/usuario/minha-api/internal/model"
)

type ProdutoRepository interface {
    FindAll() ([]model.Produto, error)
    FindByID(id int) (*model.Produto, error)
    Create(p *model.Produto) error
    Update(p *model.Produto) error
    Delete(id int) error
}

type produtoRepository struct {
    db *sql.DB
}

func NewProdutoRepository(db *sql.DB) ProdutoRepository {
    return &produtoRepository{db}
}

func (r *produtoRepository) FindAll() ([]model.Produto, error) {
    rows, err := r.db.Query(`SELECT id, nome, preco FROM produtos ORDER BY id`)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var lista []model.Produto
    for rows.Next() {
        var p model.Produto
        if err := rows.Scan(&p.ID, &p.Nome, &p.Preco); err != nil {
            return nil, err
        }
        lista = append(lista, p)
    }
    return lista, rows.Err()
}

func (r *produtoRepository) FindByID(id int) (*model.Produto, error) {
    var p model.Produto
    err := r.db.QueryRow(`SELECT id, nome, preco FROM produtos WHERE id = $1`, id).
        Scan(&p.ID, &p.Nome, &p.Preco)
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return &p, err
}

func (r *produtoRepository) Create(p *model.Produto) error {
    return r.db.QueryRow(
        `INSERT INTO produtos (nome, preco) VALUES ($1, $2) RETURNING id`,
        p.Nome, p.Preco,
    ).Scan(&p.ID)
}

func (r *produtoRepository) Update(p *model.Produto) error {
    _, err := r.db.Exec(
        `UPDATE produtos SET nome=$1, preco=$2 WHERE id=$3`,
        p.Nome, p.Preco, p.ID,
    )
    return err
}

func (r *produtoRepository) Delete(id int) error {
    _, err := r.db.Exec(`DELETE FROM produtos WHERE id=$1`, id)
    return err
}
```

**service/produto_service.go**

```go
package service

import (
    "fmt"
    "github.com/usuario/minha-api/internal/model"
    "github.com/usuario/minha-api/internal/repository"
)

type ProdutoService struct {
    repo repository.ProdutoRepository
}

func NewProdutoService(repo repository.ProdutoRepository) *ProdutoService {
    return &ProdutoService{repo}
}

func (s *ProdutoService) Listar() ([]model.Produto, error) {
    return s.repo.FindAll()
}

func (s *ProdutoService) BuscarPorID(id int) (*model.Produto, error) {
    p, err := s.repo.FindByID(id)
    if err != nil {
        return nil, err
    }
    if p == nil {
        return nil, fmt.Errorf("produto %d não encontrado", id)
    }
    return p, nil
}

func (s *ProdutoService) Criar(dto model.CriarProdutoDTO) (*model.Produto, error) {
    p := &model.Produto{Nome: dto.Nome, Preco: dto.Preco}
    if err := s.repo.Create(p); err != nil {
        return nil, err
    }
    return p, nil
}
```

**handler/produto_handler.go**

```go
package handler

import (
    "net/http"
    "strconv"

    "github.com/gin-gonic/gin"
    "github.com/usuario/minha-api/internal/model"
    "github.com/usuario/minha-api/internal/service"
)

type ProdutoHandler struct {
    svc *service.ProdutoService
}

func NewProdutoHandler(svc *service.ProdutoService) *ProdutoHandler {
    return &ProdutoHandler{svc}
}

func (h *ProdutoHandler) RegisterRoutes(r *gin.RouterGroup) {
    r.GET("", h.listar)
    r.GET("/:id", h.buscar)
    r.POST("", h.criar)
}

func (h *ProdutoHandler) listar(c *gin.Context) {
    lista, err := h.svc.Listar()
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"erro": err.Error()})
        return
    }
    c.JSON(http.StatusOK, lista)
}

func (h *ProdutoHandler) buscar(c *gin.Context) {
    id, err := strconv.Atoi(c.Param("id"))
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"erro": "id inválido"})
        return
    }
    p, err := h.svc.BuscarPorID(id)
    if err != nil {
        c.JSON(http.StatusNotFound, gin.H{"erro": err.Error()})
        return
    }
    c.JSON(http.StatusOK, p)
}

func (h *ProdutoHandler) criar(c *gin.Context) {
    var dto model.CriarProdutoDTO
    if err := c.ShouldBindJSON(&dto); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"erro": err.Error()})
        return
    }
    p, err := h.svc.Criar(dto)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"erro": err.Error()})
        return
    }
    c.JSON(http.StatusCreated, p)
}
```

**cmd/api/main.go**

```go
package main

import (
    "database/sql"
    "log"
    "os"

    "github.com/gin-gonic/gin"
    _ "github.com/lib/pq" // driver PostgreSQL
    "github.com/usuario/minha-api/internal/handler"
    "github.com/usuario/minha-api/internal/repository"
    "github.com/usuario/minha-api/internal/service"
)

func main() {
    dsn := os.Getenv("DATABASE_URL")
    if dsn == "" {
        dsn = "postgres://postgres:postgres@localhost:5432/meubanco?sslmode=disable"
    }

    db, err := sql.Open("postgres", dsn)
    if err != nil {
        log.Fatal("Erro ao abrir banco:", err)
    }
    defer db.Close()

    if err := db.Ping(); err != nil {
        log.Fatal("Erro ao conectar ao banco:", err)
    }

    // Injeção de dependências manual
    repo := repository.NewProdutoRepository(db)
    svc := service.NewProdutoService(repo)
    hdl := handler.NewProdutoHandler(svc)

    r := gin.Default()
    v1 := r.Group("/api/v1/produtos")
    hdl.RegisterRoutes(v1)

    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }
    log.Printf("API rodando em :%s\n", port)
    log.Fatal(r.Run(":" + port))
}
```

---

## Integração com Banco de Dados

### database/sql (stdlib) com PostgreSQL

```bash
# Driver PostgreSQL
go get github.com/lib/pq

# Driver SQLite (ótimo para desenvolvimento local)
go get modernc.org/sqlite  # implementação pura Go, sem CGO
```

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/lib/pq" // registra o driver, não usa diretamente
)

func main() {
    db, err := sql.Open("postgres",
        "postgres://user:pass@localhost:5432/meubanco?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Pool de conexões
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(10)

    // DDL
    _, err = db.Exec(`
        CREATE TABLE IF NOT EXISTS usuarios (
            id    SERIAL PRIMARY KEY,
            nome  TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL,
            idade INT
        )
    `)
    if err != nil {
        log.Fatal(err)
    }

    // INSERT com parâmetro (previne SQL injection)
    var id int
    err = db.QueryRow(
        `INSERT INTO usuarios (nome, email, idade) VALUES ($1, $2, $3) RETURNING id`,
        "Maria Silva", "maria@email.com", 30,
    ).Scan(&id)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Usuário criado com ID:", id)

    // SELECT múltiplas linhas
    rows, err := db.Query(`SELECT id, nome, email FROM usuarios WHERE idade >= $1`, 18)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()

    for rows.Next() {
        var (
            userID int
            nome   string
            email  string
        )
        if err := rows.Scan(&userID, &nome, &email); err != nil {
            log.Fatal(err)
        }
        fmt.Printf("[%d] %s (%s)\n", userID, nome, email)
    }
    if err := rows.Err(); err != nil {
        log.Fatal(err)
    }

    // UPDATE
    result, err := db.Exec(
        `UPDATE usuarios SET nome=$1 WHERE id=$2`,
        "Maria Souza", id,
    )
    if err != nil {
        log.Fatal(err)
    }
    linhasAfetadas, _ := result.RowsAffected()
    fmt.Println("Linhas atualizadas:", linhasAfetadas)

    // DELETE
    db.Exec(`DELETE FROM usuarios WHERE id=$1`, id)
}
```

### Transações

```go
func transferirSaldo(db *sql.DB, deID, paraID int, valor float64) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback() // no-op se já fez commit

    _, err = tx.Exec(
        `UPDATE contas SET saldo = saldo - $1 WHERE id = $2 AND saldo >= $1`,
        valor, deID,
    )
    if err != nil {
        return fmt.Errorf("débito falhou: %w", err)
    }

    _, err = tx.Exec(
        `UPDATE contas SET saldo = saldo + $1 WHERE id = $2`,
        valor, paraID,
    )
    if err != nil {
        return fmt.Errorf("crédito falhou: %w", err)
    }

    return tx.Commit()
}
```

### GORM — ORM para Go

GORM é o ORM mais popular do ecossistema Go. Suporta PostgreSQL, MySQL, SQLite e SQL Server.

```bash
go get gorm.io/gorm
go get gorm.io/driver/postgres
go get gorm.io/driver/sqlite
```

**Definição de modelos**

```go
package model

import (
    "time"
    "gorm.io/gorm"
)

type Usuario struct {
    gorm.Model               // ID, CreatedAt, UpdatedAt, DeletedAt (soft delete)
    Nome      string         `gorm:"not null;size:100"`
    Email     string         `gorm:"uniqueIndex;not null"`
    Senha     string         `gorm:"not null"`
    Pedidos   []Pedido       `gorm:"foreignKey:UsuarioID"`
}

type Produto struct {
    gorm.Model
    Nome       string    `gorm:"not null;size:200"`
    Preco      float64   `gorm:"not null;check:preco > 0"`
    Estoque    int       `gorm:"default:0"`
    CategoriaID uint
    Categoria  Categoria `gorm:"constraint:OnDelete:RESTRICT"`
}

type Categoria struct {
    ID       uint   `gorm:"primaryKey"`
    Nome     string `gorm:"uniqueIndex;not null"`
    Produtos []Produto
}

type Pedido struct {
    gorm.Model
    UsuarioID uint
    Usuario   Usuario
    Status    string       `gorm:"default:'pendente'"`
    Itens     []ItemPedido `gorm:"foreignKey:PedidoID"`
}

type ItemPedido struct {
    ID        uint    `gorm:"primaryKey;autoIncrement"`
    PedidoID  uint
    ProdutoID uint
    Produto   Produto
    Qtd       int     `gorm:"not null;check:qtd > 0"`
    Preco     float64 `gorm:"not null"` // preço no momento da compra
}
```

**Conexão e migração automática**

```go
package config

import (
    "log"
    "os"

    "gorm.io/driver/postgres"
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
    "github.com/usuario/minha-api/internal/model"
)

func NovoDatabase() *gorm.DB {
    var dialector gorm.Dialector

    env := os.Getenv("ENV")
    if env == "development" || env == "" {
        // SQLite para desenvolvimento local (sem Docker)
        dialector = sqlite.Open("local.db")
    } else {
        dsn := os.Getenv("DATABASE_URL")
        dialector = postgres.Open(dsn)
    }

    db, err := gorm.Open(dialector, &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    if err != nil {
        log.Fatal("Falha ao conectar ao banco:", err)
    }

    // Cria/atualiza tabelas automaticamente
    err = db.AutoMigrate(
        &model.Categoria{},
        &model.Produto{},
        &model.Usuario{},
        &model.Pedido{},
        &model.ItemPedido{},
    )
    if err != nil {
        log.Fatal("Falha na migração:", err)
    }

    return db
}
```

**CRUD com GORM**

```go
package repository

import (
    "github.com/usuario/minha-api/internal/model"
    "gorm.io/gorm"
)

type GormProdutoRepository struct {
    db *gorm.DB
}

func NewGormProdutoRepository(db *gorm.DB) *GormProdutoRepository {
    return &GormProdutoRepository{db}
}

// CREATE
func (r *GormProdutoRepository) Create(p *model.Produto) error {
    return r.db.Create(p).Error
}

// READ — todos
func (r *GormProdutoRepository) FindAll() ([]model.Produto, error) {
    var lista []model.Produto
    err := r.db.Preload("Categoria").Find(&lista).Error
    return lista, err
}

// READ — por ID
func (r *GormProdutoRepository) FindByID(id uint) (*model.Produto, error) {
    var p model.Produto
    err := r.db.Preload("Categoria").First(&p, id).Error
    if err == gorm.ErrRecordNotFound {
        return nil, nil
    }
    return &p, err
}

// READ — com filtros
func (r *GormProdutoRepository) BuscarPorCategoria(catID uint, minPreco float64) ([]model.Produto, error) {
    var lista []model.Produto
    err := r.db.
        Where("categoria_id = ? AND preco >= ?", catID, minPreco).
        Order("preco ASC").
        Limit(50).
        Find(&lista).Error
    return lista, err
}

// UPDATE — campos específicos
func (r *GormProdutoRepository) UpdatePreco(id uint, novoPreco float64) error {
    return r.db.Model(&model.Produto{}).
        Where("id = ?", id).
        Update("preco", novoPreco).Error
}

// UPDATE — struct completo
func (r *GormProdutoRepository) Save(p *model.Produto) error {
    return r.db.Save(p).Error
}

// DELETE — soft delete (marca deleted_at, não remove)
func (r *GormProdutoRepository) Delete(id uint) error {
    return r.db.Delete(&model.Produto{}, id).Error
}

// Paginação
func (r *GormProdutoRepository) Paginar(pagina, tamanhoPagina int) ([]model.Produto, int64, error) {
    var lista []model.Produto
    var total int64

    r.db.Model(&model.Produto{}).Count(&total)
    err := r.db.
        Offset((pagina - 1) * tamanhoPagina).
        Limit(tamanhoPagina).
        Find(&lista).Error

    return lista, total, err
}
```

**Transações com GORM**

```go
func (r *GormProdutoRepository) CriarPedidoComItens(pedido *model.Pedido) error {
    return r.db.Transaction(func(tx *gorm.DB) error {
        if err := tx.Create(pedido).Error; err != nil {
            return err
        }
        for i := range pedido.Itens {
            pedido.Itens[i].PedidoID = pedido.ID
            if err := tx.Create(&pedido.Itens[i]).Error; err != nil {
                return err
            }
            // Decrementar estoque
            if err := tx.Model(&model.Produto{}).
                Where("id = ? AND estoque >= ?", pedido.Itens[i].ProdutoID, pedido.Itens[i].Qtd).
                UpdateColumn("estoque", gorm.Expr("estoque - ?", pedido.Itens[i].Qtd)).Error; err != nil {
                return fmt.Errorf("estoque insuficiente para produto %d", pedido.Itens[i].ProdutoID)
            }
        }
        return nil
    })
}
```

### sqlx — Alternativa entre stdlib e ORM

`sqlx` adiciona conveniências ao `database/sql` sem ser um ORM completo — mapeia linhas diretamente para structs via tags `db`.

```bash
go get github.com/jmoiron/sqlx
```

```go
package main

import (
    "fmt"
    "log"

    "github.com/jmoiron/sqlx"
    _ "github.com/lib/pq"
)

type Usuario struct {
    ID    int    `db:"id"`
    Nome  string `db:"nome"`
    Email string `db:"email"`
}

func main() {
    db, _ := sqlx.Connect("postgres",
        "postgres://user:pass@localhost/meubanco?sslmode=disable")
    defer db.Close()

    // Get — única linha diretamente para struct
    var u Usuario
    err := db.Get(&u, "SELECT * FROM usuarios WHERE id=$1", 1)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(u.Nome)

    // Select — múltiplas linhas para slice de struct
    var lista []Usuario
    err = db.Select(&lista, "SELECT * FROM usuarios ORDER BY nome")
    if err != nil {
        log.Fatal(err)
    }
    for _, usr := range lista {
        fmt.Printf("[%d] %s\n", usr.ID, usr.Nome)
    }

    // Named query — usa :campo em vez de $1
    usuario := Usuario{Nome: "João", Email: "joao@email.com"}
    _, err = db.NamedExec(
        `INSERT INTO usuarios (nome, email) VALUES (:nome, :email)`,
        usuario,
    )
    if err != nil {
        log.Fatal(err)
    }
}
```

### Migrações com golang-migrate

Para ambientes de produção, use migrações versionadas em vez de `AutoMigrate`.

```bash
go get -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```

```bash
# Criar nova migração
migrate create -ext sql -dir migrations -seq criar_tabela_produtos

# Aplicar migrações
migrate -path migrations -database "postgres://user:pass@localhost/db?sslmode=disable" up

# Reverter última migração
migrate -path migrations -database "$DATABASE_URL" down 1
```

**migrations/000001_criar_tabela_produtos.up.sql**

```sql
CREATE TABLE produtos (
    id     SERIAL PRIMARY KEY,
    nome   VARCHAR(200) NOT NULL,
    preco  DECIMAL(10,2) NOT NULL CHECK (preco > 0),
    estoque INT NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_produtos_nome ON produtos (nome);
```

**migrations/000001_criar_tabela_produtos.down.sql**

```sql
DROP TABLE IF EXISTS produtos;
```

**Executar migrações na inicialização da aplicação**

```go
import (
    "github.com/golang-migrate/migrate/v4"
    _ "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
)

func RunMigrations(databaseURL string) error {
    m, err := migrate.New("file://migrations", databaseURL)
    if err != nil {
        return err
    }
    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        return err
    }
    return nil
}
```

---

## Testes

Go inclui um framework de testes na stdlib — sem dependências externas para casos básicos.

```go
// internal/service/produto_service_test.go
package service_test

import (
    "testing"

    "github.com/usuario/minha-api/internal/model"
    "github.com/usuario/minha-api/internal/service"
)

// Mock do repositório
type mockRepo struct {
    produtos []model.Produto
}

func (m *mockRepo) FindAll() ([]model.Produto, error) { return m.produtos, nil }
func (m *mockRepo) FindByID(id int) (*model.Produto, error) {
    for _, p := range m.produtos {
        if p.ID == id {
            return &p, nil
        }
    }
    return nil, nil
}
func (m *mockRepo) Create(p *model.Produto) error {
    p.ID = len(m.produtos) + 1
    m.produtos = append(m.produtos, *p)
    return nil
}
func (m *mockRepo) Update(p *model.Produto) error { return nil }
func (m *mockRepo) Delete(id int) error           { return nil }

func TestBuscarPorID_Encontrado(t *testing.T) {
    repo := &mockRepo{
        produtos: []model.Produto{
            {ID: 1, Nome: "Notebook", Preco: 3500},
        },
    }
    svc := service.NewProdutoService(repo)

    p, err := svc.BuscarPorID(1)

    if err != nil {
        t.Fatalf("esperava nil, obteve erro: %v", err)
    }
    if p == nil {
        t.Fatal("esperava produto, obteve nil")
    }
    if p.Nome != "Notebook" {
        t.Errorf("nome esperado 'Notebook', obteve '%s'", p.Nome)
    }
}

func TestBuscarPorID_NaoEncontrado(t *testing.T) {
    repo := &mockRepo{}
    svc := service.NewProdutoService(repo)

    _, err := svc.BuscarPorID(99)
    if err == nil {
        t.Fatal("esperava erro para ID inexistente")
    }
}

func TestBuscarPorID_IDInvalido(t *testing.T) {
    repo := &mockRepo{}
    svc := service.NewProdutoService(repo)

    _, err := svc.BuscarPorID(-1)
    if err == nil {
        t.Fatal("esperava erro para ID negativo")
    }
}

// Table-driven tests — idioma recomendado para múltiplos cenários
func TestCategorizarNota(t *testing.T) {
    casos := []struct {
        nome     string
        nota     float64
        esperado string
    }{
        {"aprovado com excelência", 9.5, "A"},
        {"aprovado", 7.0, "B"},
        {"recuperação", 5.0, "C"},
        {"reprovado", 4.9, "F"},
        {"zero", 0, "F"},
    }

    for _, tc := range casos {
        t.Run(tc.nome, func(t *testing.T) {
            resultado := categorizarNota(tc.nota)
            if resultado != tc.esperado {
                t.Errorf("nota %.1f: esperado '%s', obteve '%s'", tc.nota, tc.esperado, resultado)
            }
        })
    }
}
```

**Executar testes**

```bash
# Todos os testes do módulo
go test ./...

# Com verbose
go test -v ./internal/service/...

# Com cobertura
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out   # abre relatório no browser

# Benchmark
go test -bench=. -benchmem ./...
```

---

## Docker e Deploy

**Dockerfile multi-stage (imagem final ~20MB)**

```dockerfile
# Stage 1: build
FROM golang:1.24-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o api ./cmd/api

# Stage 2: imagem final mínima
FROM scratch
COPY --from=builder /app/api /api
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
EXPOSE 8080
ENTRYPOINT ["/api"]
```

**docker-compose.yml (desenvolvimento local)**

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/meubanco?sslmode=disable
      PORT: 8080
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: meubanco
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

---

## Referências

| Recurso | URL | Descrição |
|---|---|---|
| **Tour oficial** | tour.golang.org | Introdução interativa à linguagem |
| **Documentação** | pkg.go.dev | Referência da stdlib e pacotes externos |
| **Effective Go** | go.dev/doc/effective_go | Guia de boas práticas oficiais |
| **Go by Example** | gobyexample.com | Exemplos práticos por tópico |
| **Gin** | github.com/gin-gonic/gin | Framework web rápido |
| **Echo** | echo.labstack.com | Alternativa ao Gin, foco em extensibilidade |
| **Fiber** | gofiber.io | API inspirada no Express.js |
| **Chi** | github.com/go-chi/chi | Roteador leve, compatível com net/http |
| **GORM** | gorm.io | ORM completo para Go |
| **sqlx** | github.com/jmoiron/sqlx | Extensão ergonômica da database/sql |
| **golang-migrate** | github.com/golang-migrate/migrate | Migrações versionadas |
| **golangci-lint** | golangci-lint.run | Linter com dezenas de análises |
| **Testify** | github.com/stretchr/testify | Assertions e mocks para testes |
| **Viper** | github.com/spf13/viper | Configuração via env, arquivo, flags |
| **Zap** | go.uber.org/zap | Logger estruturado de alta performance |

# Python 3 — Guia Prático Abrangente

Python é uma linguagem de propósito geral, interpretada e dinamicamente tipada, criada por Guido van Rossum em 1991. Sua filosofia de design enfatiza **legibilidade, expressividade e produtividade**, com uma sintaxe limpa que torna o código fácil de escrever e manter. Python é hoje a linguagem mais popular do mundo, dominando áreas como ciência de dados, machine learning, automação, desenvolvimento web e scripts de infraestrutura.

---

## Sumário

1. [Por que Python?](#por-que-python)
2. [Casos de Uso e Ecossistema](#casos-de-uso-e-ecossistema)
3. [Ambiente de Desenvolvimento](#ambiente-de-desenvolvimento)
4. [IDEs e Ferramentas](#ides-e-ferramentas)
5. [Fundamentos da Linguagem](#fundamentos-da-linguagem)
6. [Coleções de Dados](#coleções-de-dados)
7. [Programação Orientada a Objetos](#programação-orientada-a-objetos)
8. [Tratamento de Erros](#tratamento-de-erros)
9. [Módulos, Pacotes e Ambiente Virtual](#módulos-pacotes-e-ambiente-virtual)
   - [Expressões Regulares (re)](#expressões-regulares-re)
10. [Desenvolvimento Web com FastAPI](#desenvolvimento-web-com-fastapi)
11. [Desenvolvimento Web com Django](#desenvolvimento-web-com-django)
12. [Integração com Banco de Dados](#integração-com-banco-de-dados)
13. [Machine Learning](#machine-learning)
14. [Visão Computacional](#visão-computacional)
15. [Automação e Web Scraping](#automação-e-web-scraping)
16. [Processamento e Análise de Dados](#processamento-e-análise-de-dados)
17. [Scripting, DevOps e Infraestrutura](#scripting-devops-e-infraestrutura)
18. [Testes](#testes)
19. [Referências](#referências)

---

## Por que Python?

| Característica | Detalhe |
|---|---|
| **Sintaxe legível** | Indentação obrigatória e palavras-chave em inglês aproximam o código de pseudocódigo |
| **Ecossistema imenso** | PyPI tem mais de 500 mil pacotes; de HTTP a redes neurais em `pip install` |
| **Multiplataforma** | Windows, Linux e macOS sem mudança de código |
| **Interatividade** | REPL e Jupyter Notebooks permitem experimentação incremental |
| **Comunidade** | Maior comunidade de ciência de dados e IA do mundo |
| **Versatilidade** | A mesma linguagem que automatiza tarefas, treina modelos e serve APIs REST |
| **Produtividade** | Tipagem dinâmica e stdlib rica permitem protótipos em horas, não dias |

---

## Casos de Uso e Ecossistema

### Visão Geral das Principais Áreas

| Área | Bibliotecas / Frameworks Principais |
|---|---|
| **Web (APIs REST)** | FastAPI, Flask, Starlette |
| **Web (full-stack)** | Django, Django REST Framework |
| **Machine Learning** | scikit-learn, XGBoost, LightGBM |
| **Deep Learning** | PyTorch, TensorFlow/Keras |
| **Visão Computacional** | OpenCV, Pillow, torchvision |
| **Análise de Dados** | pandas, NumPy, Polars |
| **Visualização** | matplotlib, seaborn, Plotly, Altair |
| **Automação / Scraping** | Selenium, Playwright, requests, BeautifulSoup, Scrapy |
| **DevOps / Scripts** | Fabric, Paramiko, boto3, click, Typer |
| **NLP / IA Generativa** | Hugging Face Transformers, LangChain, spaCy |
| **Dados em tempo real** | Apache Kafka (confluent-kafka), PySpark, Faust |

---

## Ambiente de Desenvolvimento

### Instalação do Python 3

**Windows:**

```powershell
# Opção 1: instalador oficial
# Acesse https://www.python.org/downloads/ e baixe Python 3.12+
# Marque "Add Python to PATH" durante a instalação

# Verificar instalação
python --version
pip --version

# Opção 2: via winget (recomendado)
winget install Python.Python.3.12

# Opção 3: via Chocolatey
choco install python
```

**macOS:**

```bash
# Via Homebrew (recomendado)
brew install python@3.12

python3 --version
pip3 --version
```

**Linux (Debian/Ubuntu):**

```bash
sudo apt update
sudo apt install python3.12 python3.12-venv python3.12-dev python3-pip -y

python3 --version
```

---

### pyenv — Gerenciador de Versões

`pyenv` permite instalar e alternar entre múltiplas versões do Python sem conflitos de sistema.

```bash
# Instalar pyenv (Linux/macOS)
curl https://pyenv.run | bash

# Adicionar ao shell (~/.bashrc ou ~/.zshrc)
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# Instalar uma versão específica
pyenv install 3.12.3
pyenv global 3.12.3      # define como padrão global
pyenv local 3.11.9       # define por diretório (cria .python-version)

python --version
```

---

### Ambientes Virtuais

Ambientes virtuais isolam dependências por projeto, evitando conflitos de versão de pacotes.

```bash
# Criar ambiente virtual (venv — stdlib do Python 3)
python -m venv .venv

# Ativar
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows (cmd/PowerShell)

# Verificar que está usando o Python do venv
which python                     # deve apontar para .venv/

# Instalar pacotes
pip install fastapi uvicorn sqlalchemy

# Salvar dependências
pip freeze > requirements.txt

# Restaurar em outro ambiente
pip install -r requirements.txt

# Desativar
deactivate
```

**uv — Gerenciador moderno e ultrarrápido (recomendado):**

`uv` é um gerenciador de pacotes e projetos escrito em Rust, muito mais rápido que `pip`.

```bash
# Instalar uv
pip install uv
# ou: curl -Ls https://astral.sh/uv/install.sh | sh

# Criar projeto com ambiente virtual integrado
uv init meu-projeto
cd meu-projeto

# Adicionar dependências (atualiza pyproject.toml automaticamente)
uv add fastapi uvicorn sqlalchemy

# Executar
uv run python main.py

# Sincronizar ambiente a partir do pyproject.toml
uv sync
```

---

### Estrutura Recomendada de Projeto

```
meu-projeto/
├── .venv/                  # ambiente virtual (não versionar)
├── src/
│   └── meu_projeto/
│       ├── __init__.py
│       ├── main.py
│       ├── models/
│       ├── services/
│       └── routers/
├── tests/
│   ├── __init__.py
│   └── test_main.py
├── pyproject.toml          # metadados e dependências (moderno)
├── requirements.txt        # ou usar pyproject.toml
├── .env                    # variáveis de ambiente (não versionar)
├── .gitignore
└── README.md
```

**pyproject.toml (formato moderno):**

```toml
[project]
name = "meu-projeto"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.115",
    "uvicorn[standard]>=0.32",
    "sqlalchemy>=2.0",
    "pydantic-settings>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "httpx>=0.27",
    "ruff>=0.7",
    "mypy>=1.11",
]

[tool.ruff]
line-length = 120
target-version = "py312"

[tool.mypy]
python_version = "3.12"
strict = true
```

---

## IDEs e Ferramentas

### IDEs Recomendadas

| IDE / Editor | Pontos Fortes | Ideal Para |
|---|---|---|
| **VS Code** | Extensões Python/Pylance, Jupyter integrado, leve | Uso geral, FastAPI, scripts |
| **PyCharm Professional** | Refatoração avançada, depurador visual, suporte Django e banco de dados | Django, projetos grandes |
| **PyCharm Community** | Versão gratuita com recursos Python completos | Estudo, projetos médios |
| **Jupyter Lab** | Notebooks interativos, visualização inline | Ciência de dados, análise |
| **Spyder** | Interface científica integrada com numpy/pandas | Análise científica |
| **Cursor** | Fork do VS Code com IA (Claude/GPT) integrada | Desenvolvimento assistido por IA |

### Extensões VS Code Essenciais

```
Python (ms-python.python)       — suporte principal: linting, debug, testes
Pylance (ms-python.vscode-pylance) — análise de tipos estática
Jupyter (ms-toolsai.jupyter)    — notebooks no editor
Ruff (charliermarsh.ruff)       — linter e formatador ultrarrápido
Black Formatter (ms-python.black-formatter) — formatação automática
GitLens                         — histórico git inline
Thunder Client / REST Client    — teste de APIs
```

### Ferramentas de Qualidade de Código

```bash
# Ruff — linter + formatador moderno (substitui flake8, isort, black)
pip install ruff
ruff check .          # verificar
ruff format .         # formatar

# mypy — verificação de tipos estáticos
pip install mypy
mypy src/

# Black — formatador de código
pip install black
black .

# isort — organizar imports
pip install isort
isort .

# pre-commit — hooks automáticos de qualidade
pip install pre-commit
# .pre-commit-config.yaml define quais hooks rodam antes de cada commit
```

---

## Fundamentos da Linguagem

### Variáveis e Tipos Básicos

```python
# Python é dinamicamente tipado — o tipo é inferido em tempo de execução
nome = "Alice"
idade = 30
altura = 1.75
ativo = True
nada = None

# Type hints (Python 3.5+) — opcional, melhora legibilidade e ferramentas de análise
nome: str = "Alice"
idade: int = 30
altura: float = 1.75
ativo: bool = True
nada: None = None

# Verificar tipo
print(type(nome))    # <class 'str'>
print(type(idade))   # <class 'int'>

# Conversão de tipos
numero_str = "42"
numero_int = int(numero_str)    # 42
numero_float = float("3.14")    # 3.14
texto = str(100)                # "100"

# F-strings (Python 3.6+) — interpolação de strings
mensagem = f"Olá, {nome}! Você tem {idade} anos."
print(mensagem)
# Olá, Alice! Você tem 30 anos.

# Formatação com expressões
print(f"Altura: {altura:.1f}m")        # Altura: 1.8m
print(f"Dobro da idade: {idade * 2}")  # Dobro da idade: 60
```

---

### Operadores

```python
# Aritméticos
print(10 + 3)   # 13
print(10 - 3)   # 7
print(10 * 3)   # 30
print(10 / 3)   # 3.3333... (sempre float em Python 3)
print(10 // 3)  # 3 (divisão inteira)
print(10 % 3)   # 1 (módulo / resto)
print(2 ** 10)  # 1024 (exponenciação)

# Comparação
print(5 == 5)   # True
print(5 != 3)   # True
print(5 > 3)    # True
print(5 >= 5)   # True

# Lógicos
print(True and False)  # False
print(True or False)   # True
print(not True)        # False

# Identidade e pertinência
lista = [1, 2, 3]
print(2 in lista)      # True
print(5 not in lista)  # True
x = None
print(x is None)       # True (use is para None, não ==)
```

---

### Condicionais

```python
temperatura = 28

# if / elif / else
if temperatura >= 35:
    print("Calor extremo")
elif temperatura >= 25:
    print("Quente")
elif temperatura >= 15:
    print("Agradável")
else:
    print("Frio")
# Saída: Quente

# Expressão condicional (ternário)
status = "maior de idade" if idade >= 18 else "menor de idade"
print(status)  # maior de idade

# match / case (Python 3.10+) — substitui switch/case
comando = "sair"

match comando:
    case "iniciar":
        print("Iniciando...")
    case "pausar":
        print("Pausando...")
    case "sair":
        print("Encerrando...")
    case _:
        print(f"Comando desconhecido: {comando}")
# Saída: Encerrando...

# match com padrões estruturais
ponto = (0, 1)

match ponto:
    case (0, 0):
        print("Origem")
    case (x, 0):
        print(f"Eixo X em {x}")
    case (0, y):
        print(f"Eixo Y em {y}")
    case (x, y):
        print(f"Ponto em ({x}, {y})")
# Saída: Eixo Y em 1
```

---

### Loops

```python
# for — itera sobre qualquer iterável
frutas = ["maçã", "banana", "laranja"]

for fruta in frutas:
    print(fruta)

# range — sequência de números
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 11):      # 1 a 10
    print(i)

for i in range(0, 20, 2):   # 0, 2, 4, 6, ..., 18 (step 2)
    print(i)

# enumerate — índice + valor
for indice, fruta in enumerate(frutas):
    print(f"{indice}: {fruta}")
# 0: maçã
# 1: banana
# 2: laranja

# zip — iteração paralela
nomes = ["Alice", "Bob", "Carlos"]
notas = [9.5, 7.0, 8.2]

for nome, nota in zip(nomes, notas):
    print(f"{nome}: {nota}")

# while
contador = 0
while contador < 5:
    print(contador)
    contador += 1

# break e continue
for numero in range(10):
    if numero == 3:
        continue   # pula o 3
    if numero == 7:
        break      # para no 7
    print(numero)
# 0, 1, 2, 4, 5, 6

# else em loops — executa se o loop terminou sem break
for i in range(5):
    if i == 10:
        break
else:
    print("Loop concluído sem break")
```

---

### Funções

```python
# Definição básica
def saudacao(nome: str) -> str:
    return f"Olá, {nome}!"

print(saudacao("Maria"))  # Olá, Maria!

# Parâmetros padrão
def potencia(base: float, expoente: int = 2) -> float:
    return base ** expoente

print(potencia(3))      # 9.0
print(potencia(2, 10))  # 1024.0

# Argumentos posicionais e nomeados
def criar_usuario(nome: str, email: str, ativo: bool = True) -> dict:
    return {"nome": nome, "email": email, "ativo": ativo}

usuario = criar_usuario("Ana", email="ana@exemplo.com", ativo=False)

# *args — múltiplos argumentos posicionais
def somar(*numeros: float) -> float:
    return sum(numeros)

print(somar(1, 2, 3, 4, 5))  # 15.0

# **kwargs — argumentos nomeados variáveis
def exibir_info(**dados):
    for chave, valor in dados.items():
        print(f"  {chave}: {valor}")

exibir_info(nome="Carlos", cidade="São Paulo", profissao="Dev")

# Funções como primeira classe e lambda
numeros = [3, 1, 4, 1, 5, 9, 2, 6]
ordenados = sorted(numeros, key=lambda x: -x)  # decrescente
print(ordenados)  # [9, 6, 5, 4, 3, 2, 1, 1]

dobrar = lambda x: x * 2
print(dobrar(7))  # 14

# Generators — produzem valores sob demanda (eficiente para grandes volumes)
def fibonacci(limite: int):
    a, b = 0, 1
    while a <= limite:
        yield a
        a, b = b, a + b

for n in fibonacci(100):
    print(n, end=" ")
# 0 1 1 2 3 5 8 13 21 34 55 89

# Decoradores — modificam o comportamento de funções
import functools
import time

def medir_tempo(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        inicio = time.perf_counter()
        resultado = func(*args, **kwargs)
        fim = time.perf_counter()
        print(f"{func.__name__} executou em {fim - inicio:.4f}s")
        return resultado
    return wrapper

@medir_tempo
def operacao_lenta():
    time.sleep(0.1)
    return "pronto"

operacao_lenta()
# operacao_lenta executou em 0.1002s
```

---

### Compreensões (Comprehensions)

Sintaxe concisa e Pythonica para criar coleções a partir de iteráveis.

```python
# List comprehension
quadrados = [x ** 2 for x in range(10)]
print(quadrados)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Com filtro
pares = [x for x in range(20) if x % 2 == 0]
print(pares)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# Dict comprehension
quadrados_dict = {x: x ** 2 for x in range(6)}
print(quadrados_dict)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Set comprehension
letras_unicas = {letra.lower() for letra in "Hello World" if letra != " "}
print(letras_unicas)  # {'h', 'e', 'l', 'o', 'w', 'r', 'd'}

# Generator expression (lazy — não cria lista em memória)
total = sum(x ** 2 for x in range(1_000_000))
```

---

## Coleções de Dados

### Listas

```python
# Criação
frutas: list[str] = ["maçã", "banana", "laranja"]
numeros = list(range(1, 6))  # [1, 2, 3, 4, 5]
mista = [1, "texto", True, 3.14, None]

# Acesso e fatiamento (slicing)
print(frutas[0])    # maçã (primeiro)
print(frutas[-1])   # laranja (último)
print(frutas[1:3])  # ['banana', 'laranja']
print(frutas[::-1]) # lista invertida

# Modificação
frutas.append("uva")          # adiciona no final
frutas.insert(1, "morango")   # insere na posição 1
frutas.extend(["kiwi", "pêra"]) # adiciona múltiplos
frutas.remove("banana")       # remove por valor
removido = frutas.pop()       # remove e retorna o último
removido = frutas.pop(0)      # remove e retorna o índice 0

# Consulta
print(len(frutas))            # quantidade de elementos
print("uva" in frutas)        # True
print(frutas.index("laranja"))# índice do elemento
print(frutas.count("maçã"))   # ocorrências

# Ordenação
frutas.sort()                       # in-place
frutas_ordenadas = sorted(frutas)   # nova lista
frutas.sort(key=len, reverse=True)  # por comprimento, decrescente

# Cópia
copia_rasa = frutas.copy()     # ou frutas[:]
import copy
copia_profunda = copy.deepcopy(frutas)  # necessário para listas aninhadas

# Desempacotamento (unpacking)
primeiro, *resto = [1, 2, 3, 4, 5]
print(primeiro)  # 1
print(resto)     # [2, 3, 4, 5]

a, b, *_, ultimo = range(10)
print(a, b, ultimo)  # 0 1 9
```

---

### Tuplas

Sequências **imutáveis** — ótimas para dados que não devem mudar.

```python
# Criação
coordenada = (10.5, -23.4)
ponto = 1, 2, 3           # parênteses opcionais
singleton = (42,)         # vírgula obrigatória para tupla de um elemento

# Acesso igual às listas
print(coordenada[0])   # 10.5

# Desempacotamento
lat, lon = coordenada
x, y, z = ponto

# Imutabilidade — gera TypeError se tentar modificar
# coordenada[0] = 0.0  # TypeError!

# Named tuples — acesso por atributo
from collections import namedtuple
Ponto = namedtuple("Ponto", ["x", "y"])
p = Ponto(3, 4)
print(p.x, p.y)        # 3 4
print(p)               # Ponto(x=3, y=4)

# typing.NamedTuple — com type hints
from typing import NamedTuple

class Produto(NamedTuple):
    nome: str
    preco: float
    estoque: int = 0

prod = Produto("Notebook", 3500.00, 10)
print(prod.nome)   # Notebook
```

---

### Dicionários

Estrutura de chave-valor (hash map) com complexidade O(1) para acesso.

```python
# Criação
usuario = {
    "nome": "Alice",
    "email": "alice@exemplo.com",
    "idade": 30,
    "ativo": True
}

# A partir do Python 3.9+, dicts mantêm ordem de inserção
produto = dict(nome="Notebook", preco=3500.0, estoque=5)

# Acesso
print(usuario["nome"])                      # Alice
print(usuario.get("telefone"))              # None (sem KeyError)
print(usuario.get("telefone", "não informado"))  # não informado

# Modificação
usuario["telefone"] = "11-99999-9999"       # adicionar/atualizar
usuario.update({"email": "novo@email.com", "cidade": "SP"})
del usuario["ativo"]                         # remover chave

# Verificação
print("nome" in usuario)        # True
print("senha" not in usuario)   # True

# Iteração
for chave in usuario:
    print(chave)

for chave, valor in usuario.items():
    print(f"{chave}: {valor}")

for valor in usuario.values():
    print(valor)

# Desempacotamento (Python 3.9+)
config_padrao = {"debug": False, "porta": 8080}
config_custom = {"porta": 3000, "host": "0.0.0.0"}
config_final = {**config_padrao, **config_custom}
# {"debug": False, "porta": 3000, "host": "0.0.0.0"}

# defaultdict — valor padrão para chaves ausentes
from collections import defaultdict

contagem = defaultdict(int)
palavras = "the quick brown fox jumps over the lazy dog".split()
for palavra in palavras:
    contagem[palavra] += 1

print(dict(sorted(contagem.items(), key=lambda x: -x[1])))
# {'the': 2, 'quick': 1, 'brown': 1, ...}
```

---

### Sets (Conjuntos)

Coleções **não ordenadas** de elementos **únicos** — operações de conjunto eficientes.

```python
# Criação
vogais = {"a", "e", "i", "o", "u"}
pares = set(range(0, 10, 2))          # {0, 2, 4, 6, 8}
letras = set("abracadabra")           # {'a', 'b', 'r', 'c', 'd'}

# Modificação
vogais.add("y")
vogais.discard("y")   # remove sem erro se não existir
vogais.remove("a")    # remove, KeyError se não existir

# Operações de conjunto
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

print(a | b)   # união:        {1, 2, 3, 4, 5, 6, 7, 8}
print(a & b)   # interseção:   {4, 5}
print(a - b)   # diferença:    {1, 2, 3}
print(a ^ b)   # dif. simétrica: {1, 2, 3, 6, 7, 8}

# Verificações
print(a.issubset({1, 2, 3, 4, 5, 6}))    # True
print(a.issuperset({1, 2}))               # True
print({1, 2}.isdisjoint({3, 4}))          # True (sem elementos comuns)

# frozenset — set imutável (pode ser chave de dicionário)
imutavel = frozenset([1, 2, 3])
```

---

## Programação Orientada a Objetos

### Classes e Instâncias

```python
from typing import ClassVar

class ContaBancaria:
    # Atributo de classe (compartilhado por todas as instâncias)
    taxa_juros: ClassVar[float] = 0.02

    def __init__(self, titular: str, saldo_inicial: float = 0.0) -> None:
        self.titular = titular          # atributo público
        self._saldo = saldo_inicial     # atributo protegido (convenção)
        self.__numero = id(self)        # atributo privado (name mangling)

    # Propriedade com getter/setter
    @property
    def saldo(self) -> float:
        return self._saldo

    @saldo.setter
    def saldo(self, valor: float) -> None:
        if valor < 0:
            raise ValueError("Saldo não pode ser negativo")
        self._saldo = valor

    def depositar(self, valor: float) -> None:
        if valor <= 0:
            raise ValueError("Valor de depósito deve ser positivo")
        self._saldo += valor

    def sacar(self, valor: float) -> bool:
        if valor > self._saldo:
            return False
        self._saldo -= valor
        return True

    def aplicar_juros(self) -> None:
        self._saldo *= 1 + self.taxa_juros

    # Representação
    def __repr__(self) -> str:
        return f"ContaBancaria(titular={self.titular!r}, saldo={self._saldo:.2f})"

    def __str__(self) -> str:
        return f"Conta de {self.titular}: R$ {self._saldo:,.2f}"

    # Métodos de classe e estáticos
    @classmethod
    def criar_conta_salario(cls, titular: str, salario: float) -> "ContaBancaria":
        return cls(titular, salario * 0.1)  # 10% do salário como saldo inicial

    @staticmethod
    def validar_cpf(cpf: str) -> bool:
        return len(cpf.replace(".", "").replace("-", "")) == 11


# Uso
conta = ContaBancaria("Alice", 1000.00)
conta.depositar(500.00)
conta.sacar(200.00)
conta.aplicar_juros()
print(conta)
# Conta de Alice: R$ 1.326,00

conta2 = ContaBancaria.criar_conta_salario("Bob", 5000.00)
```

---

### Herança e Polimorfismo

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    def __init__(self, nome: str, peso_kg: float) -> None:
        self.nome = nome
        self.peso_kg = peso_kg

    @abstractmethod
    def emitir_som(self) -> str:
        ...

    def apresentar(self) -> str:
        return f"Sou {self.nome} ({self.peso_kg}kg): {self.emitir_som()}"


class Cachorro(Animal):
    def __init__(self, nome: str, peso_kg: float, raca: str) -> None:
        super().__init__(nome, peso_kg)
        self.raca = raca

    def emitir_som(self) -> str:
        return "Au au!"

    def buscar_objeto(self) -> str:
        return f"{self.nome} foi buscar o objeto!"


class Gato(Animal):
    def emitir_som(self) -> str:
        return "Miau!"


# Polimorfismo
animais: list[Animal] = [
    Cachorro("Rex", 25.0, "Labrador"),
    Gato("Whiskers", 4.5),
    Cachorro("Buddy", 30.0, "Golden"),
]

for animal in animais:
    print(animal.apresentar())
# Sou Rex (25.0kg): Au au!
# Sou Whiskers (4.5kg): Miau!
# Sou Buddy (30.0kg): Au au!
```

---

### Dataclasses

`dataclasses` elimina código repetitivo (`__init__`, `__repr__`, `__eq__`) automaticamente.

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class Produto:
    nome: str
    preco: float
    estoque: int = 0
    tags: list[str] = field(default_factory=list)
    criado_em: datetime = field(default_factory=datetime.now)

    def aplicar_desconto(self, percentual: float) -> "Produto":
        novo_preco = self.preco * (1 - percentual / 100)
        return Produto(self.nome, novo_preco, self.estoque, self.tags)

    @property
    def valor_total_estoque(self) -> float:
        return self.preco * self.estoque


# Uso
notebook = Produto("Notebook", 3500.00, 10, ["eletrônico", "informática"])
print(notebook)
# Produto(nome='Notebook', preco=3500.0, estoque=10, ...)

com_desconto = notebook.aplicar_desconto(10)
print(f"Preço com desconto: R$ {com_desconto.preco:.2f}")
# Preço com desconto: R$ 3150.00

# @dataclass(frozen=True) cria objetos imutáveis (equivalente a value objects)
@dataclass(frozen=True)
class Coordenada:
    latitude: float
    longitude: float
```

---

## Tratamento de Erros

```python
# Hierarquia: BaseException > Exception > (ValueError, TypeError, ...)

# try / except / else / finally
def dividir(a: float, b: float) -> float:
    try:
        resultado = a / b
    except ZeroDivisionError:
        print("Erro: divisão por zero")
        return 0.0
    except TypeError as e:
        print(f"Tipo inválido: {e}")
        raise
    else:
        print("Divisão realizada com sucesso")
        return resultado
    finally:
        print("Bloco finally sempre executa")


# Capturar múltiplas exceções
try:
    valor = int(input("Digite um número: "))
    print(100 / valor)
except (ValueError, ZeroDivisionError) as e:
    print(f"Erro: {e}")

# Exceções customizadas
class SaldoInsuficienteError(Exception):
    def __init__(self, saldo_atual: float, valor_solicitado: float) -> None:
        self.saldo_atual = saldo_atual
        self.valor_solicitado = valor_solicitado
        super().__init__(
            f"Saldo insuficiente: R$ {saldo_atual:.2f} disponível, "
            f"R$ {valor_solicitado:.2f} solicitado"
        )

def sacar(saldo: float, valor: float) -> float:
    if valor > saldo:
        raise SaldoInsuficienteError(saldo, valor)
    return saldo - valor

try:
    novo_saldo = sacar(100.0, 200.0)
except SaldoInsuficienteError as e:
    print(e)
    print(f"Faltam R$ {e.valor_solicitado - e.saldo_atual:.2f}")

# Context managers com with — garante limpeza de recursos
with open("arquivo.txt", "w", encoding="utf-8") as f:
    f.write("Olá, mundo!\n")
# arquivo é fechado automaticamente ao sair do bloco

# contextlib para criar context managers próprios
from contextlib import contextmanager

@contextmanager
def cronometro(nome: str):
    import time
    inicio = time.perf_counter()
    try:
        yield
    finally:
        fim = time.perf_counter()
        print(f"{nome}: {fim - inicio:.4f}s")

with cronometro("minha operação"):
    sum(range(1_000_000))
# minha operação: 0.0234s
```

---

## Módulos, Pacotes e Ambiente Virtual

### Imports

```python
# Import simples
import math
print(math.sqrt(16))   # 4.0

# Import com alias (comum em data science)
import numpy as np
import pandas as pd

# Import seletivo
from datetime import datetime, timedelta
from pathlib import Path
from typing import Optional, Union

# Import relativo (dentro de um pacote)
from .models import Usuario       # mesmo diretório
from ..services import auth       # diretório pai

# Verificar se é módulo principal
if __name__ == "__main__":
    print("Executando diretamente")
```

### Módulos da Biblioteca Padrão Úteis

```python
# pathlib — manipulação de caminhos (moderno, substitui os.path)
from pathlib import Path

base = Path(".")
arquivo = base / "dados" / "resultado.csv"
arquivo.parent.mkdir(parents=True, exist_ok=True)
arquivo.write_text("col1,col2\n1,2\n")
conteudo = arquivo.read_text()
print(list(base.glob("**/*.py")))   # todos os .py recursivamente

# datetime
from datetime import datetime, timedelta, date, timezone

agora = datetime.now(timezone.utc)
ontem = agora - timedelta(days=1)
formatado = agora.strftime("%d/%m/%Y %H:%M")
parseado = datetime.strptime("25/12/2024", "%d/%m/%Y")

# json
import json

dados = {"nome": "Alice", "notas": [9.0, 8.5, 10.0]}
json_str = json.dumps(dados, ensure_ascii=False, indent=2)
de_volta = json.loads(json_str)

# os / subprocess — sistema operacional
import os
import subprocess

home = Path(os.environ.get("HOME", "~"))
resultado = subprocess.run(["git", "log", "--oneline", "-5"], capture_output=True, text=True)
print(resultado.stdout)

# logging — logs estruturados
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s — %(message)s"
)
logger = logging.getLogger(__name__)
logger.info("Aplicação iniciada")
logger.warning("Configuração padrão em uso")
logger.error("Falha ao conectar: %s", "timeout")

# re — expressões regulares (ver seção dedicada abaixo)
import re
```

---

### Expressões Regulares (re)

O módulo `re` da stdlib implementa o motor de expressões regulares do Python. Regex são usadas para busca, validação e transformação de texto de forma concisa e poderosa.

#### Funções principais

```python
import re

texto = "Pedido #1042 feito em 25/12/2024 por alice@email.com (R$ 350,00)"

# re.search — encontra a primeira ocorrência em qualquer posição
m = re.search(r"\d{2}/\d{2}/\d{4}", texto)
if m:
    print(m.group())   # 25/12/2024
    print(m.start())   # posição inicial no texto
    print(m.end())     # posição final
    print(m.span())    # (início, fim)

# re.match — ancorando no INÍCIO da string
m = re.match(r"Pedido #(\d+)", texto)
if m:
    print(m.group(0))  # Pedido #1042  (captura completa)
    print(m.group(1))  # 1042          (grupo 1)

# re.fullmatch — a string inteira deve corresponder ao padrão
valido = re.fullmatch(r"\d{5}-\d{3}", "01310-100")
print(bool(valido))   # True

# re.findall — retorna lista de todas as ocorrências
emails = re.findall(r"[\w.+-]+@[\w-]+\.[a-z]{2,}", texto)
print(emails)         # ['alice@email.com']

numeros = re.findall(r"\d+", texto)
print(numeros)        # ['1042', '25', '12', '2024', '350', '00']

# re.finditer — iterador de Match objects (mais eficiente que findall para textos grandes)
for m in re.finditer(r"\d+", texto):
    print(f"'{m.group()}' na posição {m.start()}")

# re.sub — substituição
anonimizado = re.sub(r"[\w.+-]+@[\w-]+\.[a-z]{2,}", "[EMAIL]", texto)
print(anonimizado)
# Pedido #1042 feito em 25/12/2024 por [EMAIL] (R$ 350,00)

# sub com função de substituição
def formatar_data(m: re.Match) -> str:
    dia, mes, ano = m.group().split("/")
    return f"{ano}-{mes}-{dia}"

formatado = re.sub(r"\d{2}/\d{2}/\d{4}", formatar_data, texto)
print(formatado)
# Pedido #1042 feito em 2024-12-25 por alice@email.com (R$ 350,00)

# re.split — dividir por padrão
partes = re.split(r"[,;]\s*", "maçã, banana; laranja,uva")
print(partes)  # ['maçã', 'banana', 'laranja', 'uva']
```

---

#### Grupos de Captura

```python
import re

log = "2024-12-25 14:32:01 ERROR auth.service — Token expirado para usuário 42"

# Grupos numerados
m = re.search(r"(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) (\w+) ([\w.]+)", log)
if m:
    print(m.group(1))  # 2024-12-25
    print(m.group(2))  # 14:32:01
    print(m.group(3))  # ERROR
    print(m.group(4))  # auth.service

# Grupos nomeados — muito mais legível
padrao = re.compile(
    r"(?P<data>\d{4}-\d{2}-\d{2})\s"
    r"(?P<hora>\d{2}:\d{2}:\d{2})\s"
    r"(?P<nivel>\w+)\s"
    r"(?P<modulo>[\w.]+)"
)
m = padrao.search(log)
if m:
    print(m.group("data"))    # 2024-12-25
    print(m.group("nivel"))   # ERROR
    print(m.groupdict())
    # {'data': '2024-12-25', 'hora': '14:32:01', 'nivel': 'ERROR', 'modulo': 'auth.service'}

# Grupos não-capturantes (?:...) — agrupam sem criar um grupo numerado
versao = "Python 3.12.3"
m = re.search(r"Python (?:\d+\.)+(\d+)", versao)
print(m.group(1))  # 3 (último número — grupos anteriores descartados)

# Lookahead (?=...) e lookbehind (?<=...) — verificam contexto sem capturar
precos = "produto: R$ 150,00 e frete: R$ 25,50"
valores = re.findall(r"(?<=R\$ )[\d,]+", precos)
print(valores)  # ['150,00', '25,50']
```

---

#### Flags e Compilação

```python
import re

# re.compile — pré-compila o padrão para reutilização eficiente
PADRAO_EMAIL = re.compile(r"[\w.+-]+@[\w-]+\.[a-z]{2,}", re.IGNORECASE)
PADRAO_CPF   = re.compile(r"\d{3}\.?\d{3}\.?\d{3}-?\d{2}")

emails = PADRAO_EMAIL.findall("Alice@Email.COM e bob@TESTE.org")
print(emails)  # ['Alice@Email.COM', 'bob@TESTE.org']

# Flags principais
texto_multilinea = """
Linha 1: erro crítico
Linha 2: aviso
Linha 3: erro leve
"""

# re.MULTILINE — ^ e $ passam a corresponder ao início/fim de cada linha
erros = re.findall(r"^Linha \d+: erro.*$", texto_multilinea, re.MULTILINE)
print(erros)
# ['Linha 1: erro crítico', 'Linha 3: erro leve']

# re.IGNORECASE — ignora maiúsculas/minúsculas
print(re.findall(r"python", "Python é PYTHON!", re.IGNORECASE))
# ['Python', 'PYTHON']

# re.DOTALL — . passa a corresponder também a \n
html = "<div>\n  conteúdo\n</div>"
m = re.search(r"<div>(.*?)</div>", html, re.DOTALL)
print(m.group(1).strip())  # conteúdo

# re.VERBOSE — permite escrever regex com comentários e espaços (legibilidade)
PADRAO_DATA = re.compile(r"""
    (?P<dia>  \d{1,2}) /   # dia (1 ou 2 dígitos)
    (?P<mes>  \d{1,2}) /   # mês
    (?P<ano>  \d{4})       # ano com 4 dígitos
""", re.VERBOSE)

m = PADRAO_DATA.match("25/12/2024")
print(m.groupdict())  # {'dia': '25', 'mes': '12', 'ano': '2024'}
```

---

#### Casos de Uso Práticos

```python
import re

# Validação de CPF (formato com ou sem pontuação)
def validar_cpf_formato(cpf: str) -> bool:
    return bool(re.fullmatch(r"\d{3}\.?\d{3}\.?\d{3}-?\d{2}", cpf.strip()))

print(validar_cpf_formato("123.456.789-09"))  # True
print(validar_cpf_formato("12345678909"))     # True
print(validar_cpf_formato("1234-567"))        # False

# Validação de e-mail (simplificada)
REGEX_EMAIL = re.compile(r"^[\w.+-]+@[\w-]+\.[a-z]{2,}$", re.IGNORECASE)

def validar_email(email: str) -> bool:
    return bool(REGEX_EMAIL.match(email.strip()))

# Validação de URL
REGEX_URL = re.compile(
    r"https?://"                  # esquema
    r"(?:[\w-]+\.)+[a-z]{2,}"    # domínio
    r"(?:/[^\s]*)?"               # caminho opcional
, re.IGNORECASE)

# Extração de dados de log Apache/Nginx
LOG_LINE = '192.168.1.1 - - [25/Dec/2024:14:32:01 +0000] "GET /api/v1/produtos HTTP/1.1" 200 1024'

PADRAO_LOG = re.compile(
    r'(?P<ip>[\d.]+) \S+ \S+ \[(?P<data>[^\]]+)\] '
    r'"(?P<metodo>\w+) (?P<rota>\S+)[^"]*" '
    r'(?P<status>\d{3}) (?P<bytes>\d+)'
)

m = PADRAO_LOG.match(LOG_LINE)
if m:
    print(m.group("ip"))      # 192.168.1.1
    print(m.group("metodo"))  # GET
    print(m.group("rota"))    # /api/v1/produtos
    print(m.group("status"))  # 200

# Extração de variáveis de template
template = "Olá {{ nome }}, seu pedido {{ numero }} foi confirmado."
variaveis = re.findall(r"\{\{\s*(\w+)\s*\}\}", template)
print(variaveis)  # ['nome', 'numero']

# Remover tags HTML
html = "<p>Texto com <strong>negrito</strong> e <em>itálico</em>.</p>"
sem_tags = re.sub(r"<[^>]+>", "", html)
print(sem_tags)  # Texto com negrito e itálico.

# Normalizar espaços em branco
bagunçado = "texto   com    espaços\textra\n\ne quebras"
limpo = re.sub(r"\s+", " ", bagunçado).strip()
print(limpo)  # texto com espaços extra e quebras

# Substituição com grupos de referência (\1, \2 ou \g<nome>)
# Trocar formato de data DD/MM/AAAA → AAAA-MM-DD
datas = "Entrega: 25/12/2024, Retorno: 05/01/2025"
iso = re.sub(r"(\d{2})/(\d{2})/(\d{4})", r"\3-\2-\1", datas)
print(iso)  # Entrega: 2024-12-25, Retorno: 2025-01-05
```

---

#### Referência Rápida de Padrões

| Padrão | Significado | Exemplo |
|---|---|---|
| `.` | Qualquer caractere exceto `\n` | `a.c` → "abc", "axc" |
| `\d` | Dígito `[0-9]` | `\d{4}` → "2024" |
| `\D` | Não-dígito | `\D+` → "abc" |
| `\w` | Palavra `[a-zA-Z0-9_]` | `\w+` → "hello_42" |
| `\W` | Não-palavra | ` `, `.`, `,` |
| `\s` | Espaço em branco | espaço, tab, `\n` |
| `\S` | Não-espaço | qualquer outro |
| `^` | Início da string (ou linha com `re.M`) | `^HTTP` |
| `$` | Fim da string (ou linha com `re.M`) | `\d$` |
| `\b` | Fronteira de palavra | `\bcat\b` → "cat" mas não "catch" |
| `*` | 0 ou mais repetições (guloso) | `a*` → "", "a", "aaa" |
| `+` | 1 ou mais repetições (guloso) | `\d+` → "1", "42" |
| `?` | 0 ou 1 repetição | `colou?r` → "color", "colour" |
| `{n}` | Exatamente n repetições | `\d{4}` → "2024" |
| `{n,m}` | Entre n e m repetições | `\d{2,4}` → "12", "123" |
| `*?` `+?` | Versão não-gulosa (lazy) | `<.*?>` captura cada tag |
| `[abc]` | Conjunto — a, b ou c | `[aeiou]` → vogais |
| `[^abc]` | Negação — nenhum de a, b, c | `[^\d]` → não-dígito |
| `a\|b` | Alternância — a ou b | `jpg\|png\|gif` |
| `(...)` | Grupo de captura numerado | `(\d+)-(\d+)` |
| `(?:...)` | Grupo não-capturante | `(?:www\.)?` |
| `(?P<n>...)` | Grupo de captura nomeado | `(?P<ano>\d{4})` |
| `(?=...)` | Lookahead positivo | `\d+(?= reais)` |
| `(?!...)` | Lookahead negativo | `\d+(?! anos)` |
| `(?<=...)` | Lookbehind positivo | `(?<=R\$ )\d+` |

---

## Desenvolvimento Web com FastAPI

FastAPI é um framework moderno para construção de APIs REST com Python 3.8+, baseado em **Pydantic** para validação e **Starlette** para a camada ASGI. É um dos frameworks Python mais rápidos e tem documentação OpenAPI/Swagger gerada automaticamente.

### Instalação

```bash
pip install fastapi uvicorn[standard] pydantic-settings python-jose[cryptography] passlib[bcrypt]
```

### Estrutura do Projeto

```
api/
├── main.py
├── config.py
├── database.py
├── models/
│   ├── __init__.py
│   └── usuario.py
├── schemas/
│   ├── __init__.py
│   └── usuario.py
├── routers/
│   ├── __init__.py
│   ├── usuarios.py
│   └── produtos.py
├── services/
│   ├── __init__.py
│   └── auth.py
└── dependencies.py
```

---

### API Básica

```python
# main.py
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime
from typing import Optional

app = FastAPI(
    title="API de Exemplo",
    description="API RESTful com FastAPI",
    version="1.0.0",
)

# Schema (contrato da API)
class ProdutoCreate(BaseModel):
    nome: str = Field(..., min_length=2, max_length=100)
    preco: float = Field(..., gt=0, description="Preço em reais")
    estoque: int = Field(default=0, ge=0)
    categoria: Optional[str] = None

class ProdutoResponse(ProdutoCreate):
    id: int
    criado_em: datetime

    class Config:
        from_attributes = True  # compatibilidade com ORM

# "Banco de dados" em memória para exemplo
produtos_db: dict[int, dict] = {}
proximo_id = 1

@app.get("/produtos", response_model=list[ProdutoResponse])
def listar_produtos():
    return list(produtos_db.values())

@app.get("/produtos/{produto_id}", response_model=ProdutoResponse)
def buscar_produto(produto_id: int):
    if produto_id not in produtos_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Produto {produto_id} não encontrado"
        )
    return produtos_db[produto_id]

@app.post("/produtos", response_model=ProdutoResponse, status_code=status.HTTP_201_CREATED)
def criar_produto(produto: ProdutoCreate):
    global proximo_id
    novo = {**produto.model_dump(), "id": proximo_id, "criado_em": datetime.now()}
    produtos_db[proximo_id] = novo
    proximo_id += 1
    return novo

@app.put("/produtos/{produto_id}", response_model=ProdutoResponse)
def atualizar_produto(produto_id: int, produto: ProdutoCreate):
    if produto_id not in produtos_db:
        raise HTTPException(status_code=404, detail="Produto não encontrado")
    produtos_db[produto_id].update(produto.model_dump())
    return produtos_db[produto_id]

@app.delete("/produtos/{produto_id}", status_code=status.HTTP_204_NO_CONTENT)
def remover_produto(produto_id: int):
    if produto_id not in produtos_db:
        raise HTTPException(status_code=404, detail="Produto não encontrado")
    del produtos_db[produto_id]
```

```bash
# Executar
uvicorn main:app --reload --port 8000
# Swagger UI: http://localhost:8000/docs
# ReDoc:      http://localhost:8000/redoc
```

---

### FastAPI com Banco de Dados (SQLAlchemy Async)

```python
# database.py
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase

DATABASE_URL = "postgresql+asyncpg://user:senha@localhost/meubanco"

engine = create_async_engine(DATABASE_URL, echo=False)
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        yield session
```

```python
# models/produto.py
from sqlalchemy import String, Float, Integer, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from database import Base

class Produto(Base):
    __tablename__ = "produtos"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    nome: Mapped[str] = mapped_column(String(100), nullable=False)
    preco: Mapped[float] = mapped_column(Float, nullable=False)
    estoque: Mapped[int] = mapped_column(Integer, default=0)
    criado_em: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())
```

```python
# routers/produtos.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from database import get_db
from models.produto import Produto
from schemas.produto import ProdutoCreate, ProdutoResponse

router = APIRouter(prefix="/produtos", tags=["Produtos"])

@router.get("/", response_model=list[ProdutoResponse])
async def listar(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Produto).order_by(Produto.id))
    return result.scalars().all()

@router.post("/", response_model=ProdutoResponse, status_code=201)
async def criar(dados: ProdutoCreate, db: AsyncSession = Depends(get_db)):
    produto = Produto(**dados.model_dump())
    db.add(produto)
    await db.commit()
    await db.refresh(produto)
    return produto
```

---

### Autenticação JWT com FastAPI

```python
# services/auth.py
from datetime import datetime, timedelta, timezone
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

SECRET_KEY = "sua-chave-secreta-256-bits"  # use variável de ambiente
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

def verificar_senha(senha_plana: str, senha_hash: str) -> bool:
    return pwd_context.verify(senha_plana, senha_hash)

def gerar_hash(senha: str) -> str:
    return pwd_context.hash(senha)

def criar_token(dados: dict, expiracao: Optional[timedelta] = None) -> str:
    payload = dados.copy()
    expire = datetime.now(timezone.utc) + (expiracao or timedelta(minutes=15))
    payload.update({"exp": expire})
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def verificar_token(token: str = Depends(oauth2_scheme)) -> str:
    credencial_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Token inválido",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credencial_exception
        return username
    except JWTError:
        raise credencial_exception
```

```python
# routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from services.auth import verificar_senha, criar_token, ACCESS_TOKEN_EXPIRE_MINUTES
from datetime import timedelta

router = APIRouter(prefix="/auth", tags=["Autenticação"])

@router.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    # buscar usuário no banco e verificar senha (simplificado)
    usuario = {"username": "admin", "senha_hash": gerar_hash("segredo")}
    if form_data.username != usuario["username"] or \
       not verificar_senha(form_data.password, usuario["senha_hash"]):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)
    
    token = criar_token(
        {"sub": form_data.username},
        timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    return {"access_token": token, "token_type": "bearer"}

# Rota protegida
@router.get("/me")
async def meu_perfil(usuario_atual: str = Depends(verificar_token)):
    return {"usuario": usuario_atual}
```

---

### Middlewares e Configurações Avançadas

```python
# main.py completo com CORS, eventos e middleware
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager
from database import engine, Base

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: criar tabelas
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    # Shutdown: fechar engine
    await engine.dispose()

app = FastAPI(lifespan=lifespan)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://meusite.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Incluir routers
from routers import produtos, auth
app.include_router(produtos.router)
app.include_router(auth.router)

# Handler global de erros
from fastapi.responses import JSONResponse
from fastapi.requests import Request

@app.exception_handler(Exception)
async def handler_generico(request: Request, exc: Exception):
    return JSONResponse(status_code=500, content={"detail": "Erro interno"})
```

---

## Desenvolvimento Web com Django

Django é um framework web "batteries included" focado em desenvolvimento rápido e manutenção a longo prazo. Segue o padrão MTV (Model-Template-View) e inclui ORM, admin, autenticação, migração de banco de dados e muito mais.

### Instalação e Criação do Projeto

```bash
pip install django djangorestframework django-cors-headers Pillow

# Criar projeto e aplicação
django-admin startproject config .
python manage.py startapp core

# Estrutura gerada
# manage.py
# config/
#   settings.py
#   urls.py
#   wsgi.py
#   asgi.py
# core/
#   models.py
#   views.py
#   urls.py
#   admin.py
#   apps.py

# Migrações e servidor
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
# Admin: http://localhost:8000/admin
```

---

### Models (ORM Django)

```python
# core/models.py
from django.db import models
from django.contrib.auth.models import User
from django.core.validators import MinValueValidator

class Categoria(models.Model):
    nome = models.CharField(max_length=100, unique=True)
    descricao = models.TextField(blank=True)

    class Meta:
        ordering = ["nome"]
        verbose_name_plural = "categorias"

    def __str__(self) -> str:
        return self.nome


class Produto(models.Model):
    nome = models.CharField(max_length=200)
    descricao = models.TextField(blank=True)
    preco = models.DecimalField(max_digits=10, decimal_places=2,
                                validators=[MinValueValidator(0)])
    estoque = models.PositiveIntegerField(default=0)
    categoria = models.ForeignKey(Categoria, on_delete=models.PROTECT,
                                  related_name="produtos")
    imagem = models.ImageField(upload_to="produtos/", null=True, blank=True)
    ativo = models.BooleanField(default=True)
    criado_em = models.DateTimeField(auto_now_add=True)
    atualizado_em = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["-criado_em"]
        indexes = [
            models.Index(fields=["nome"]),
            models.Index(fields=["categoria", "ativo"]),
        ]

    def __str__(self) -> str:
        return f"{self.nome} (R$ {self.preco})"


class Pedido(models.Model):
    class Status(models.TextChoices):
        PENDENTE = "PE", "Pendente"
        PAGO = "PA", "Pago"
        ENVIADO = "EN", "Enviado"
        ENTREGUE = "ET", "Entregue"
        CANCELADO = "CA", "Cancelado"

    usuario = models.ForeignKey(User, on_delete=models.PROTECT)
    status = models.CharField(max_length=2, choices=Status.choices,
                               default=Status.PENDENTE)
    total = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    criado_em = models.DateTimeField(auto_now_add=True)

    def calcular_total(self) -> None:
        self.total = sum(item.subtotal for item in self.itens.all())
        self.save(update_fields=["total"])


class ItemPedido(models.Model):
    pedido = models.ForeignKey(Pedido, on_delete=models.CASCADE,
                                related_name="itens")
    produto = models.ForeignKey(Produto, on_delete=models.PROTECT)
    quantidade = models.PositiveIntegerField(default=1)
    preco_unitario = models.DecimalField(max_digits=10, decimal_places=2)

    @property
    def subtotal(self):
        return self.quantidade * self.preco_unitario
```

```bash
# Criar e aplicar migrações
python manage.py makemigrations core
python manage.py migrate
```

---

### Django Admin

```python
# core/admin.py
from django.contrib import admin
from .models import Categoria, Produto, Pedido, ItemPedido

@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    list_display = ["nome", "descricao"]
    search_fields = ["nome"]

class ItemPedidoInline(admin.TabularInline):
    model = ItemPedido
    extra = 0

@admin.register(Produto)
class ProdutoAdmin(admin.ModelAdmin):
    list_display = ["nome", "preco", "estoque", "categoria", "ativo"]
    list_filter = ["categoria", "ativo"]
    search_fields = ["nome", "descricao"]
    list_editable = ["preco", "estoque", "ativo"]
    readonly_fields = ["criado_em", "atualizado_em"]

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    list_display = ["id", "usuario", "status", "total", "criado_em"]
    list_filter = ["status"]
    inlines = [ItemPedidoInline]
    readonly_fields = ["total", "criado_em"]
```

---

### Django REST Framework (DRF)

```python
# config/settings.py — adicionar
INSTALLED_APPS = [
    ...
    "rest_framework",
    "corsheaders",
    "core",
]

MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",
    ...
]

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticatedOrReadOnly",
    ],
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 20,
    "DEFAULT_FILTER_BACKENDS": [
        "django_filters.rest_framework.DjangoFilterBackend",
        "rest_framework.filters.SearchFilter",
        "rest_framework.filters.OrderingFilter",
    ],
}
```

```python
# core/serializers.py
from rest_framework import serializers
from .models import Produto, Categoria, Pedido, ItemPedido

class CategoriaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Categoria
        fields = ["id", "nome", "descricao"]

class ProdutoSerializer(serializers.ModelSerializer):
    categoria_nome = serializers.CharField(source="categoria.nome", read_only=True)

    class Meta:
        model = Produto
        fields = ["id", "nome", "descricao", "preco", "estoque",
                  "categoria", "categoria_nome", "ativo", "criado_em"]
        read_only_fields = ["criado_em"]

    def validate_preco(self, value):
        if value <= 0:
            raise serializers.ValidationError("Preço deve ser positivo")
        return value

class ItemPedidoSerializer(serializers.ModelSerializer):
    subtotal = serializers.DecimalField(max_digits=12, decimal_places=2, read_only=True)

    class Meta:
        model = ItemPedido
        fields = ["id", "produto", "quantidade", "preco_unitario", "subtotal"]

class PedidoSerializer(serializers.ModelSerializer):
    itens = ItemPedidoSerializer(many=True, read_only=True)
    status_display = serializers.CharField(source="get_status_display", read_only=True)

    class Meta:
        model = Pedido
        fields = ["id", "usuario", "status", "status_display", "total",
                  "criado_em", "itens"]
        read_only_fields = ["total", "criado_em"]
```

```python
# core/views.py
from rest_framework import viewsets, permissions, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from django_filters.rest_framework import DjangoFilterBackend
from .models import Produto, Categoria, Pedido
from .serializers import ProdutoSerializer, CategoriaSerializer, PedidoSerializer

class ProdutoViewSet(viewsets.ModelViewSet):
    queryset = Produto.objects.select_related("categoria").filter(ativo=True)
    serializer_class = ProdutoSerializer
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ["categoria", "ativo"]
    search_fields = ["nome", "descricao"]
    ordering_fields = ["preco", "criado_em", "nome"]
    ordering = ["-criado_em"]

    def get_permissions(self):
        if self.action in ["list", "retrieve"]:
            return [permissions.AllowAny()]
        return [permissions.IsAdminUser()]

    @action(detail=True, methods=["post"])
    def aplicar_desconto(self, request, pk=None):
        produto = self.get_object()
        percentual = request.data.get("percentual", 0)
        produto.preco *= (1 - float(percentual) / 100)
        produto.save(update_fields=["preco"])
        return Response({"novo_preco": produto.preco})

class PedidoViewSet(viewsets.ModelViewSet):
    serializer_class = PedidoSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        return Pedido.objects.filter(usuario=self.request.user).prefetch_related("itens")

    def perform_create(self, serializer):
        serializer.save(usuario=self.request.user)
```

```python
# config/urls.py
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from core.views import ProdutoViewSet, PedidoViewSet
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

router = DefaultRouter()
router.register("produtos", ProdutoViewSet)
router.register("pedidos", PedidoViewSet, basename="pedido")

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/v1/", include(router.urls)),
    path("api/token/", TokenObtainPairView.as_view()),
    path("api/token/refresh/", TokenRefreshView.as_view()),
]
```

---

### Django com Templates (SSR)

```python
# core/views.py — view baseada em classe
from django.views.generic import ListView, DetailView, CreateView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from .models import Produto
from .forms import ProdutoForm

class ProdutoListView(ListView):
    model = Produto
    template_name = "core/produto_list.html"
    context_object_name = "produtos"
    paginate_by = 12

    def get_queryset(self):
        qs = super().get_queryset().filter(ativo=True)
        q = self.request.GET.get("q")
        if q:
            qs = qs.filter(nome__icontains=q)
        return qs

class ProdutoDetailView(DetailView):
    model = Produto
    template_name = "core/produto_detail.html"

class ProdutoCreateView(LoginRequiredMixin, CreateView):
    model = Produto
    form_class = ProdutoForm
    template_name = "core/produto_form.html"
    success_url = reverse_lazy("produto-list")
```

```html
<!-- templates/core/produto_list.html -->
{% extends "base.html" %}
{% block content %}
<h1>Produtos</h1>
<form method="get">
  <input name="q" value="{{ request.GET.q }}" placeholder="Buscar...">
  <button type="submit">Buscar</button>
</form>

<div class="grid">
  {% for produto in produtos %}
  <div class="card">
    <h3>{{ produto.nome }}</h3>
    <p>R$ {{ produto.preco|floatformat:2 }}</p>
    <a href="{% url 'produto-detail' produto.pk %}">Ver detalhes</a>
  </div>
  {% empty %}
  <p>Nenhum produto encontrado.</p>
  {% endfor %}
</div>

{% include "pagination.html" %}
{% endblock %}
```

---

## Integração com Banco de Dados

### SQLAlchemy 2.x — ORM Completo

```bash
pip install sqlalchemy alembic psycopg2-binary asyncpg
```

```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, Session, sessionmaker

DATABASE_URL = "postgresql://user:senha@localhost/meubanco"

engine = create_engine(DATABASE_URL, pool_size=10, max_overflow=20)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

class Base(DeclarativeBase):
    pass

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

```python
# models.py
from sqlalchemy import String, Float, Integer, ForeignKey, Text, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship
from datetime import datetime
from database import Base

class Usuario(Base):
    __tablename__ = "usuarios"

    id: Mapped[int] = mapped_column(primary_key=True)
    nome: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(String(200), unique=True, index=True)
    senha_hash: Mapped[str] = mapped_column(String(200))
    criado_em: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())

    pedidos: Mapped[list["Pedido"]] = relationship(back_populates="usuario")

class Pedido(Base):
    __tablename__ = "pedidos"

    id: Mapped[int] = mapped_column(primary_key=True)
    usuario_id: Mapped[int] = mapped_column(ForeignKey("usuarios.id"))
    total: Mapped[float] = mapped_column(Float, default=0.0)

    usuario: Mapped["Usuario"] = relationship(back_populates="pedidos")
    itens: Mapped[list["ItemPedido"]] = relationship(back_populates="pedido",
                                                     cascade="all, delete-orphan")
```

```python
# repository.py — padrão repository
from sqlalchemy.orm import Session
from sqlalchemy import select, func
from models import Usuario, Pedido

class UsuarioRepository:
    def __init__(self, db: Session) -> None:
        self.db = db

    def buscar_por_email(self, email: str) -> Usuario | None:
        return self.db.execute(
            select(Usuario).where(Usuario.email == email)
        ).scalar_one_or_none()

    def criar(self, nome: str, email: str, senha_hash: str) -> Usuario:
        usuario = Usuario(nome=nome, email=email, senha_hash=senha_hash)
        self.db.add(usuario)
        self.db.commit()
        self.db.refresh(usuario)
        return usuario

    def listar_com_pedidos(self) -> list[Usuario]:
        return list(
            self.db.execute(
                select(Usuario).options(
                    selectinload(Usuario.pedidos)
                ).order_by(Usuario.nome)
            ).scalars().all()
        )
```

---

### Alembic — Migrações de Banco de Dados

```bash
# Inicializar
alembic init alembic

# alembic/env.py — configurar
from database import Base
from models import Usuario, Pedido  # importar todos os models

target_metadata = Base.metadata

# Gerar migração automaticamente
alembic revision --autogenerate -m "criar tabelas iniciais"

# Aplicar
alembic upgrade head

# Rollback
alembic downgrade -1

# Histórico
alembic history
```

---

### MongoDB com Motor (Async)

```bash
pip install motor pymongo
```

```python
# database_mongo.py
from motor.motor_asyncio import AsyncIOMotorClient
from bson import ObjectId

MONGO_URL = "mongodb://localhost:27017"
client = AsyncIOMotorClient(MONGO_URL)
db = client["meu_banco"]

# Repositório de documentos
class ProdutoMongoRepository:
    def __init__(self):
        self.collection = db["produtos"]

    async def criar(self, dados: dict) -> str:
        resultado = await self.collection.insert_one(dados)
        return str(resultado.inserted_id)

    async def buscar_por_id(self, produto_id: str) -> dict | None:
        doc = await self.collection.find_one({"_id": ObjectId(produto_id)})
        if doc:
            doc["id"] = str(doc.pop("_id"))
        return doc

    async def listar(self, filtro: dict = None, limite: int = 20) -> list[dict]:
        cursor = self.collection.find(filtro or {}).limit(limite)
        docs = await cursor.to_list(length=limite)
        for doc in docs:
            doc["id"] = str(doc.pop("_id"))
        return docs

    async def buscar_texto(self, termo: str) -> list[dict]:
        return await self.listar({"$text": {"$search": termo}})
```

---

### Redis com aioredis

```bash
pip install redis[asyncio]
```

```python
# cache.py
import redis.asyncio as redis
import json
from functools import wraps

redis_client = redis.from_url("redis://localhost:6379", decode_responses=True)

def cache_redis(ttl: int = 300):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            chave = f"{func.__name__}:{args}:{kwargs}"
            cached = await redis_client.get(chave)
            if cached:
                return json.loads(cached)
            resultado = await func(*args, **kwargs)
            await redis_client.setex(chave, ttl, json.dumps(resultado))
            return resultado
        return wrapper
    return decorator

@cache_redis(ttl=600)
async def buscar_produto_caro(produto_id: int) -> dict:
    # simulação de consulta demorada
    return {"id": produto_id, "nome": "Notebook", "preco": 3500}
```

---

## Machine Learning

### scikit-learn — ML Clássico

```bash
pip install scikit-learn pandas numpy matplotlib seaborn joblib
```

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, confusion_matrix, roc_auc_score
import joblib

# Carregar e preparar dados
df = pd.read_csv("dados.csv")

# Separar features e target
X = df.drop("target", axis=1)
y = df["target"]

# Divisão treino/teste
X_treino, X_teste, y_treino, y_teste = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Pipeline — pré-processamento + modelo em etapas encadeadas
pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("modelo", RandomForestClassifier(n_estimators=100, random_state=42))
])

# Treinar
pipeline.fit(X_treino, y_treino)

# Avaliar
y_pred = pipeline.predict(X_teste)
y_proba = pipeline.predict_proba(X_teste)[:, 1]

print(classification_report(y_teste, y_pred))
print(f"AUC-ROC: {roc_auc_score(y_teste, y_proba):.4f}")

# Validação cruzada
scores = cross_val_score(pipeline, X, y, cv=5, scoring="roc_auc")
print(f"CV AUC: {scores.mean():.4f} ± {scores.std():.4f}")

# Ajuste de hiperparâmetros
param_grid = {
    "modelo__n_estimators": [50, 100, 200],
    "modelo__max_depth": [None, 10, 20],
}
busca = GridSearchCV(pipeline, param_grid, cv=5, scoring="roc_auc", n_jobs=-1)
busca.fit(X_treino, y_treino)
print(f"Melhores parâmetros: {busca.best_params_}")

# Salvar e carregar modelo
joblib.dump(busca.best_estimator_, "modelo.pkl")
modelo_carregado = joblib.load("modelo.pkl")
```

---

### PyTorch — Deep Learning

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
# CPU apenas:
pip install torch torchvision torchaudio
```

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
from torchvision import datasets, transforms

# Verificar GPU
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print(f"Usando: {device}")

# Definir rede neural
class RedeNeural(nn.Module):
    def __init__(self, entrada: int, oculta: int, saida: int) -> None:
        super().__init__()
        self.camadas = nn.Sequential(
            nn.Linear(entrada, oculta),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(oculta, oculta // 2),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(oculta // 2, saida),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.camadas(x)

# Classificação de imagens com CNN
class CNN(nn.Module):
    def __init__(self, num_classes: int = 10) -> None:
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 32, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(32, 64, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(64 * 7 * 7, 128),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(128, num_classes),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.classifier(self.features(x))

# Loop de treinamento
def treinar(modelo, loader, otimizador, criterio, epochs=10):
    modelo.train()
    for epoch in range(epochs):
        total_loss = 0.0
        corretos = 0

        for X_batch, y_batch in loader:
            X_batch, y_batch = X_batch.to(device), y_batch.to(device)

            otimizador.zero_grad()
            saida = modelo(X_batch)
            loss = criterio(saida, y_batch)
            loss.backward()
            otimizador.step()

            total_loss += loss.item()
            corretos += (saida.argmax(1) == y_batch).sum().item()

        acuracia = corretos / len(loader.dataset)
        print(f"Epoch {epoch+1}/{epochs} — Loss: {total_loss/len(loader):.4f} — Acc: {acuracia:.4f}")

# Exemplo com MNIST
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

train_data = datasets.MNIST(".", train=True, download=True, transform=transform)
train_loader = DataLoader(train_data, batch_size=64, shuffle=True)

modelo = CNN().to(device)
otimizador = optim.Adam(modelo.parameters(), lr=1e-3)
criterio = nn.CrossEntropyLoss()

treinar(modelo, train_loader, otimizador, criterio, epochs=5)
torch.save(modelo.state_dict(), "cnn_mnist.pth")
```

---

### Hugging Face Transformers — NLP e IA Generativa

```bash
pip install transformers datasets accelerate sentencepiece
```

```python
from transformers import pipeline, AutoTokenizer, AutoModelForSequenceClassification
import torch

# Pipeline de alto nível — uso imediato
classificador = pipeline("sentiment-analysis",
                          model="nlptown/bert-base-multilingual-uncased-sentiment")
resultado = classificador("Esse produto é incrível!")
print(resultado)  # [{'label': '5 stars', 'score': 0.82}]

# Sumarização
sumarizador = pipeline("summarization", model="facebook/bart-large-cnn")
texto = """
Python is a high-level, general-purpose programming language. Its design
philosophy emphasizes code readability, with the use of significant indentation.
Python is dynamically typed and garbage-collected. It supports multiple programming
paradigms, including structured, object-oriented and functional programming.
"""
resumo = sumarizador(texto, max_length=60, min_length=20, do_sample=False)
print(resumo[0]["summary_text"])

# Tradução
tradutor = pipeline("translation_pt_to_en", model="Helsinki-NLP/opus-mt-pt-en")
print(tradutor("Olá, como você está?"))

# Fine-tuning com Trainer
from transformers import Trainer, TrainingArguments
from datasets import load_dataset

dataset = load_dataset("imdb")
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

def tokenizar(batch):
    return tokenizer(batch["text"], truncation=True, padding=True, max_length=512)

dataset = dataset.map(tokenizar, batched=True)

args = TrainingArguments(
    output_dir="./resultados",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
)

modelo = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2)
trainer = Trainer(model=modelo, args=args, train_dataset=dataset["train"],
                  eval_dataset=dataset["test"])
trainer.train()
```

---

## Visão Computacional

### OpenCV — Processamento de Imagens e Vídeo

```bash
pip install opencv-python-headless Pillow
```

```python
import cv2
import numpy as np
from pathlib import Path

# Leitura e exibição
img = cv2.imread("foto.jpg")
print(f"Shape: {img.shape}")  # (altura, largura, canais)

# Conversão de espaço de cor
cinza = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)  # para matplotlib

# Redimensionar
pequena = cv2.resize(img, (320, 240))
proporcional = cv2.resize(img, None, fx=0.5, fy=0.5)

# Filtros
borrado = cv2.GaussianBlur(img, (5, 5), 0)
nitido = cv2.filter2D(img, -1, np.array([[-1,-1,-1],[-1,9,-1],[-1,-1,-1]]))

# Detecção de bordas
bordas = cv2.Canny(cinza, threshold1=100, threshold2=200)

# Detecção de faces com Haar Cascade
cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")
faces = cascade.detectMultiScale(cinza, scaleFactor=1.1, minNeighbors=5)

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 2)

cv2.imwrite("faces_detectadas.jpg", img)

# Captura de vídeo
cap = cv2.VideoCapture(0)  # 0 = câmera padrão, ou path para arquivo

while True:
    ret, frame = cap.read()
    if not ret:
        break

    cinza = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    cv2.imshow("Câmera", cinza)

    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()

# Segmentação por cor (detectar objetos vermelhos)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
vermelho_low = np.array([0, 120, 70])
vermelho_high = np.array([10, 255, 255])
mascara = cv2.inRange(hsv, vermelho_low, vermelho_high)
objetos = cv2.bitwise_and(img, img, mask=mascara)
```

---

### Detecção de Objetos com YOLO (Ultralytics)

```bash
pip install ultralytics
```

```python
from ultralytics import YOLO
import cv2

# Carregar modelo pré-treinado
modelo = YOLO("yolo11n.pt")  # nano — rápido; ou yolo11x.pt para máxima precisão

# Inferência em imagem
resultados = modelo("foto.jpg")
resultados[0].show()  # exibir
resultados[0].save("resultado.jpg")  # salvar

# Detecção em vídeo com rastreamento
for resultado in modelo.track("video.mp4", stream=True, persist=True):
    frame = resultado.plot()
    cv2.imshow("YOLO", frame)
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

# Treinar com dataset customizado
modelo = YOLO("yolo11n.pt")
modelo.train(
    data="meu_dataset.yaml",
    epochs=100,
    imgsz=640,
    batch=16,
    device="cuda",
)

# Exportar para ONNX/TensorRT
modelo.export(format="onnx", half=True)
```

---

### Pillow — Manipulação de Imagens

```bash
pip install Pillow
```

```python
from PIL import Image, ImageFilter, ImageEnhance, ImageDraw, ImageFont
from pathlib import Path

# Abrir e inspecionar
img = Image.open("foto.jpg")
print(img.size, img.mode)  # (1920, 1080) RGB

# Operações básicas
miniatura = img.copy()
miniatura.thumbnail((300, 300))  # redimensionar preservando proporção
miniatura.save("thumb.jpg", quality=85, optimize=True)

# Recortar região
recortada = img.crop((100, 100, 400, 400))

# Filtros
borrada = img.filter(ImageFilter.GaussianBlur(radius=3))
contornos = img.filter(ImageFilter.CONTOUR)

# Ajustes
brilho = ImageEnhance.Brightness(img).enhance(1.5)   # +50%
contraste = ImageEnhance.Contrast(img).enhance(1.2)

# Converter formato e modo
img.convert("L").save("cinza.png")       # escala de cinza
img.convert("RGBA").save("transparente.png")

# Adicionar texto sobre imagem
draw = ImageDraw.Draw(img)
fonte = ImageFont.truetype("arial.ttf", size=40)  # ou ImageFont.load_default()
draw.text((50, 50), "Python + Pillow", fill=(255, 255, 255), font=fonte)
img.save("com_texto.jpg")

# Processar pasta de imagens
pasta_entrada = Path("imagens/")
pasta_saida = Path("thumbnails/")
pasta_saida.mkdir(exist_ok=True)

for caminho in pasta_entrada.glob("*.jpg"):
    with Image.open(caminho) as im:
        im.thumbnail((200, 200))
        im.save(pasta_saida / caminho.name)
```

---

## Automação e Web Scraping

### requests + BeautifulSoup — HTTP e Parsing HTML

```bash
pip install requests beautifulsoup4 lxml
```

```python
import requests
from bs4 import BeautifulSoup
import json
import time

# Requisição simples
resposta = requests.get("https://httpbin.org/get", timeout=10)
resposta.raise_for_status()  # levanta HTTPError para status 4xx/5xx
dados = resposta.json()

# Com headers, auth e parâmetros
headers = {"User-Agent": "MeuBot/1.0", "Accept": "application/json"}
params = {"pagina": 1, "limite": 50}
auth = ("usuario", "senha")

resposta = requests.get(
    "https://api.exemplo.com/produtos",
    headers=headers,
    params=params,
    auth=auth,
    timeout=30,
)

# Session — reutiliza conexão TCP, persiste cookies
with requests.Session() as sessao:
    sessao.headers.update({"Authorization": "Bearer TOKEN"})
    produtos = sessao.get("/api/produtos").json()
    pedidos = sessao.get("/api/pedidos").json()

# Parsing HTML
html = requests.get("https://exemplo.com/noticias").text
soup = BeautifulSoup(html, "lxml")

# Seletores CSS
titulos = soup.select("h2.titulo-noticia")
for titulo in titulos:
    print(titulo.get_text(strip=True))

# Navegação na árvore
tabela = soup.find("table", id="resultados")
linhas = tabela.find_all("tr")[1:]  # pular cabeçalho

dados = []
for linha in linhas:
    celulas = linha.find_all("td")
    dados.append({
        "nome": celulas[0].text.strip(),
        "preco": celulas[1].text.strip(),
        "estoque": celulas[2].text.strip(),
    })

# Salvar
with open("dados.json", "w", encoding="utf-8") as f:
    json.dump(dados, f, ensure_ascii=False, indent=2)
```

---

### Playwright — Automação de Browser

```bash
pip install playwright
playwright install chromium
```

```python
from playwright.async_api import async_playwright
import asyncio

async def scraping_dinamico():
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        context = await browser.new_context(
            user_agent="Mozilla/5.0 ..."
        )
        page = await context.new_page()

        # Navegar
        await page.goto("https://exemplo.com/login")

        # Preencher formulário
        await page.fill("#email", "usuario@email.com")
        await page.fill("#senha", "minha_senha")
        await page.click("button[type='submit']")

        # Aguardar navegação
        await page.wait_for_url("**/dashboard**")

        # Coletar dados de tabela com paginação
        todos_dados = []
        while True:
            await page.wait_for_selector("table.dados")
            linhas = await page.query_selector_all("table.dados tbody tr")

            for linha in linhas:
                celulas = await linha.query_selector_all("td")
                textos = [await c.inner_text() for c in celulas]
                todos_dados.append(textos)

            proximo = await page.query_selector("a.proxima-pagina")
            if not proximo:
                break
            await proximo.click()
            await page.wait_for_load_state("networkidle")

        await browser.close()
        return todos_dados

dados = asyncio.run(scraping_dinamico())
```

---

### Automação com pyautogui — Controle de Desktop

```bash
pip install pyautogui keyboard mouse
```

```python
import pyautogui
import time
import keyboard

pyautogui.FAILSAFE = True   # mover mouse para canto superior esquerdo para parar

# Controle do mouse
pyautogui.moveTo(500, 300, duration=0.5)  # mover suavemente
pyautogui.click()
pyautogui.doubleClick()
pyautogui.rightClick()
pyautogui.scroll(3)           # scroll para cima

# Teclado
pyautogui.typewrite("Olá, mundo!", interval=0.05)
pyautogui.hotkey("ctrl", "c")   # copiar
pyautogui.hotkey("ctrl", "v")   # colar
pyautogui.press("enter")

# Screenshot e localização de imagens
screenshot = pyautogui.screenshot()
screenshot.save("tela.png")

posicao = pyautogui.locateCenterOnScreen("botao_ok.png", confidence=0.9)
if posicao:
    pyautogui.click(posicao)

# Exemplo: abrir planilha e inserir dados
def preencher_planilha(dados: list[dict]) -> None:
    pyautogui.hotkey("ctrl", "End")   # ir para última célula
    for linha in dados:
        pyautogui.typewrite(str(linha["nome"]))
        pyautogui.press("tab")
        pyautogui.typewrite(str(linha["valor"]))
        pyautogui.press("enter")
```

---

### Scrapy — Web Scraping em Escala

```bash
pip install scrapy
scrapy startproject meu_scraper
```

```python
# spiders/produtos_spider.py
import scrapy

class ProdutosSpider(scrapy.Spider):
    name = "produtos"
    start_urls = ["https://livros.exemplo.com/"]

    def parse(self, response):
        for produto in response.css("article.product_pod"):
            yield {
                "titulo": produto.css("h3 a::attr(title)").get(),
                "preco": produto.css(".price_color::text").get(),
                "disponivel": produto.css(".instock::text").get(default="").strip(),
                "url": response.urljoin(produto.css("h3 a::attr(href)").get()),
            }

        proxima = response.css("li.next a::attr(href)").get()
        if proxima:
            yield response.follow(proxima, callback=self.parse)
```

```bash
scrapy crawl produtos -o resultado.json
scrapy crawl produtos -o resultado.csv
```

---

## Processamento e Análise de Dados

### pandas — Análise e Transformação de Dados

```bash
pip install pandas numpy openpyxl xlrd
```

```python
import pandas as pd
import numpy as np

# Criação de DataFrames
df = pd.DataFrame({
    "nome": ["Alice", "Bob", "Carlos", "Diana", "Eduardo"],
    "departamento": ["TI", "RH", "TI", "Vendas", "TI"],
    "salario": [8500.0, 6000.0, 9200.0, 7800.0, 8100.0],
    "anos_empresa": [3, 7, 1, 5, 4],
})

# Leitura de arquivos
df_csv = pd.read_csv("dados.csv", sep=";", encoding="utf-8")
df_excel = pd.read_excel("planilha.xlsx", sheet_name="Dados")
df_json = pd.read_json("dados.json")

# Inspeção
print(df.head())          # primeiras 5 linhas
print(df.info())          # tipos e nulos
print(df.describe())      # estatísticas
print(df.shape)           # (linhas, colunas)
print(df.dtypes)          # tipos por coluna

# Seleção
print(df["salario"])                        # coluna
print(df[["nome", "salario"]])              # múltiplas colunas
print(df.loc[0])                            # linha por índice
print(df.iloc[1:3, 0:2])                    # slicing posicional
print(df[df["salario"] > 8000])             # filtro booleano
print(df.query("departamento == 'TI' and salario > 8000"))

# Transformações
df["bonus"] = df["salario"] * 0.10
df["nivel"] = df["anos_empresa"].apply(lambda x: "Sênior" if x >= 5 else "Júnior")
df["nome_upper"] = df["nome"].str.upper()

# Agrupamentos
por_depto = df.groupby("departamento").agg(
    media_salario=("salario", "mean"),
    total_funcionarios=("nome", "count"),
    max_salario=("salario", "max"),
).reset_index()

print(por_depto)

# Pivot table
pivot = df.pivot_table(
    values="salario",
    index="departamento",
    columns="nivel",
    aggfunc="mean",
    fill_value=0
)

# Merge (JOIN)
df_cargos = pd.DataFrame({
    "departamento": ["TI", "RH", "Vendas"],
    "cargo_chefe": ["CTO", "CHRO", "CSO"]
})
df_completo = df.merge(df_cargos, on="departamento", how="left")

# Tratamento de valores nulos
df_limpo = df.copy()
df_limpo["salario"] = df_limpo["salario"].fillna(df_limpo["salario"].median())
df_limpo = df_limpo.dropna(subset=["nome", "email"])
df_limpo["salario"] = df_limpo["salario"].clip(lower=0)   # remover negativos

# Datas
df["data_admissao"] = pd.to_datetime(df["data_admissao"], format="%d/%m/%Y")
df["ano_admissao"] = df["data_admissao"].dt.year
df["mes_admissao"] = df["data_admissao"].dt.month

# Exportar
df.to_csv("resultado.csv", index=False, encoding="utf-8")
df.to_excel("resultado.xlsx", sheet_name="Dados", index=False)
df.to_json("resultado.json", orient="records", force_ascii=False)
```

---

### NumPy — Computação Numérica

```python
import numpy as np

# Arrays
arr = np.array([1, 2, 3, 4, 5])
matriz = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# Criação especial
zeros = np.zeros((3, 4))
uns = np.ones((2, 5), dtype=np.float32)
identidade = np.eye(4)
sequencia = np.arange(0, 10, 0.5)
espacado = np.linspace(0, 1, num=50)
aleatorio = np.random.rand(3, 4)         # uniforme [0,1)
normal = np.random.randn(1000)           # normal padrão

# Operações vetorizadas (muito mais rápido que loops)
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

print(a + b)        # [11 22 33 44]
print(a * b)        # [10 40 90 160]
print(a ** 2)       # [1 4 9 16]
print(np.sqrt(a))   # [1. 1.414 1.732 2.]

# Álgebra linear
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

produto = A @ B              # multiplicação de matrizes
transposta = A.T
inversa = np.linalg.inv(A)
det = np.linalg.det(A)
autovalores, autovetores = np.linalg.eig(A)

# Estatísticas
dados = np.random.normal(loc=50, scale=10, size=1000)
print(f"Média: {dados.mean():.2f}")
print(f"Desvio: {dados.std():.2f}")
print(f"Mediana: {np.median(dados):.2f}")
print(f"Percentil 95: {np.percentile(dados, 95):.2f}")

# Broadcasting
matriz = np.ones((3, 4))
vetor = np.array([1, 2, 3, 4])
resultado = matriz * vetor   # multiplica cada linha por vetor
```

---

### Visualização com matplotlib e seaborn

```bash
pip install matplotlib seaborn plotly
```

```python
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import seaborn as sns
import pandas as pd
import numpy as np

# Estilo global
sns.set_theme(style="whitegrid", palette="husl", font_scale=1.1)

# Gráfico simples
x = np.linspace(0, 2 * np.pi, 100)
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].plot(x, np.sin(x), label="sin(x)", color="blue")
axes[0].plot(x, np.cos(x), label="cos(x)", color="red", linestyle="--")
axes[0].set_title("Funções Trigonométricas")
axes[0].legend()
axes[0].grid(True)

# Histograma
dados = np.random.normal(100, 15, 500)
axes[1].hist(dados, bins=30, color="steelblue", edgecolor="white", alpha=0.8)
axes[1].axvline(dados.mean(), color="red", linestyle="--", label="Média")
axes[1].set_title("Distribuição Normal")
axes[1].legend()

plt.tight_layout()
plt.savefig("graficos.png", dpi=150, bbox_inches="tight")
plt.show()

# seaborn — gráficos estatísticos avançados
tips = sns.load_dataset("tips")

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

sns.barplot(data=tips, x="day", y="total_bill", ax=axes[0,0])
sns.boxplot(data=tips, x="day", y="tip", hue="sex", ax=axes[0,1])
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="smoker",
                size="size", ax=axes[1,0])
sns.heatmap(tips.corr(numeric_only=True), annot=True, fmt=".2f",
            cmap="coolwarm", ax=axes[1,1])

plt.tight_layout()
plt.savefig("analise_tips.png", dpi=150)

# Plotly — gráficos interativos
import plotly.express as px

fig = px.scatter(tips, x="total_bill", y="tip", color="sex", size="size",
                 hover_data=["day", "time"], title="Gorjetas por Conta")
fig.write_html("interativo.html")
fig.show()
```

---

### Polars — DataFrame Moderno e Ultrarrápido

```bash
pip install polars
```

```python
import polars as pl

# Leitura lazy (não carrega em memória até executar)
df = pl.scan_csv("dados_grandes.csv")

resultado = (
    df
    .filter(pl.col("salario") > 5000)
    .with_columns([
        (pl.col("salario") * 1.1).alias("salario_reajuste"),
        pl.col("nome").str.to_uppercase().alias("nome_upper"),
    ])
    .group_by("departamento")
    .agg([
        pl.col("salario").mean().alias("media_salario"),
        pl.len().alias("total"),
    ])
    .sort("media_salario", descending=True)
    .collect()  # executa o plano lazy
)

print(resultado)

# Sintaxe de expressões encadeadas
df_eager = pl.DataFrame({
    "a": [1, 2, 3, 4, 5],
    "b": ["x", "y", "x", "y", "x"],
})

print(
    df_eager
    .filter(pl.col("a") > 2)
    .group_by("b")
    .agg(pl.col("a").sum())
)
```

---

## Scripting, DevOps e Infraestrutura

### Scripts de Sistema com click / Typer

```bash
pip install click typer rich
```

```python
# cli_typer.py
import typer
from pathlib import Path
from rich.console import Console
from rich.table import Table
from rich import print as rprint

app = typer.Typer(help="CLI de administração de sistema")
console = Console()

@app.command()
def listar_arquivos(
    diretorio: Path = typer.Argument(Path("."), help="Diretório a listar"),
    extensao: str = typer.Option("", "--ext", "-e", help="Filtrar por extensão"),
    recursivo: bool = typer.Option(False, "--recursivo", "-r"),
):
    """Lista arquivos em um diretório."""
    padrao = f"**/*.{extensao}" if extensao else "**/*" if recursivo else f"*.{extensao or '*'}"
    arquivos = list(diretorio.glob(padrao))

    tabela = Table("Nome", "Tamanho", "Modificado")
    for arq in sorted(arquivos):
        if arq.is_file():
            stat = arq.stat()
            tabela.add_row(arq.name, f"{stat.st_size / 1024:.1f} KB", 
                          str(arq.stat().st_mtime))

    console.print(tabela)
    rprint(f"[green]Total: {len(arquivos)} arquivo(s)[/green]")

@app.command()
def renomear_em_lote(
    diretorio: Path = typer.Argument(...),
    prefixo: str = typer.Option("", help="Prefixo a adicionar"),
    confirmar: bool = typer.Option(False, "--sim", "-y"),
):
    """Renomeia todos os arquivos com um prefixo."""
    arquivos = list(diretorio.iterdir())
    for arq in arquivos:
        novo_nome = diretorio / f"{prefixo}{arq.name}"
        rprint(f"  {arq.name} → {novo_nome.name}")

    if confirmar or typer.confirm("Confirmar renomeação?"):
        for arq in arquivos:
            arq.rename(diretorio / f"{prefixo}{arq.name}")
        rprint("[bold green]Concluído![/bold green]")

if __name__ == "__main__":
    app()
```

```bash
python cli_typer.py listar-arquivos ./src --ext py
python cli_typer.py renomear-em-lote ./fotos --prefixo "2024_" --sim
```

---

### AWS com boto3

```bash
pip install boto3 python-dotenv
```

```python
import boto3
from pathlib import Path
from botocore.exceptions import ClientError

# Configurar via variáveis de ambiente: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION

s3 = boto3.client("s3")

def upload_arquivo(caminho: Path, bucket: str, chave: str | None = None) -> str:
    chave = chave or caminho.name
    s3.upload_file(str(caminho), bucket, chave)
    return f"s3://{bucket}/{chave}"

def download_arquivo(bucket: str, chave: str, destino: Path) -> None:
    s3.download_file(bucket, chave, str(destino))

def listar_objetos(bucket: str, prefixo: str = "") -> list[dict]:
    paginator = s3.get_paginator("list_objects_v2")
    objetos = []
    for pagina in paginator.paginate(Bucket=bucket, Prefix=prefixo):
        objetos.extend(pagina.get("Contents", []))
    return objetos

# Lambda
lambda_client = boto3.client("lambda")

resposta = lambda_client.invoke(
    FunctionName="MinhaFuncao",
    InvocationType="RequestResponse",
    Payload='{"chave": "valor"}'
)
resultado = resposta["Payload"].read()

# SQS
sqs = boto3.resource("sqs")
fila = sqs.get_queue_by_name(QueueName="minha-fila")

fila.send_message(MessageBody='{"evento": "novo_pedido", "id": 123}')

for mensagem in fila.receive_messages(MaxNumberOfMessages=10, WaitTimeSeconds=5):
    print(mensagem.body)
    mensagem.delete()
```

---

### Fabric — Automação de Servidores via SSH

```bash
pip install fabric
```

```python
# fabfile.py
from fabric import Connection, task

SERVIDORES = ["app1.exemplo.com", "app2.exemplo.com"]

@task
def deploy(c, branch="main"):
    """Faz deploy da aplicação."""
    with Connection("deploy@app.exemplo.com") as conn:
        with conn.cd("/var/www/minha-app"):
            conn.run(f"git pull origin {branch}")
            conn.run("pip install -r requirements.txt")
            conn.run("python manage.py migrate --noinput")
            conn.run("python manage.py collectstatic --noinput")
            conn.run("sudo systemctl restart gunicorn")
            print("Deploy concluído!")

@task
def status(c):
    """Verifica status dos serviços."""
    for servidor in SERVIDORES:
        with Connection(servidor) as conn:
            resultado = conn.run("systemctl is-active gunicorn", warn=True)
            estado = "✓ ativo" if resultado.ok else "✗ inativo"
            print(f"{servidor}: {estado}")
```

```bash
fab deploy --branch main
fab status
```

---

## Testes

### pytest — Framework de Testes

```bash
pip install pytest pytest-cov pytest-asyncio httpx
```

```python
# tests/test_servicos.py
import pytest
from unittest.mock import MagicMock, patch, AsyncMock
from services.produto_service import ProdutoService
from models import Produto

# Fixture — setup reutilizável
@pytest.fixture
def db_mock():
    return MagicMock()

@pytest.fixture
def servico(db_mock):
    return ProdutoService(db_mock)

@pytest.fixture
def produto_exemplo():
    return Produto(id=1, nome="Notebook", preco=3500.00, estoque=10)

# Testes de unidade
def test_criar_produto_valido(servico, db_mock, produto_exemplo):
    db_mock.add = MagicMock()
    db_mock.commit = MagicMock()
    db_mock.refresh = MagicMock(side_effect=lambda p: setattr(p, "id", 1))

    resultado = servico.criar("Notebook", 3500.00, 10)

    db_mock.add.assert_called_once()
    db_mock.commit.assert_called_once()
    assert resultado.nome == "Notebook"

def test_criar_produto_preco_invalido(servico):
    with pytest.raises(ValueError, match="Preço deve ser positivo"):
        servico.criar("Produto", -10.0, 5)

@pytest.mark.parametrize("preco,estoque,esperado", [
    (100.0, 5, 500.0),
    (200.0, 0, 0.0),
    (50.0, 10, 500.0),
])
def test_calcular_valor_total(produto_exemplo, preco, estoque, esperado):
    produto_exemplo.preco = preco
    produto_exemplo.estoque = estoque
    assert produto_exemplo.valor_total_estoque == esperado

# Teste de API FastAPI
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_listar_produtos():
    resposta = client.get("/produtos")
    assert resposta.status_code == 200
    assert isinstance(resposta.json(), list)

def test_criar_produto():
    resposta = client.post("/produtos", json={
        "nome": "Mouse", "preco": 150.0, "estoque": 30
    })
    assert resposta.status_code == 201
    assert resposta.json()["nome"] == "Mouse"

def test_buscar_produto_inexistente():
    resposta = client.get("/produtos/99999")
    assert resposta.status_code == 404

# Testes assíncronos
@pytest.mark.asyncio
async def test_criar_produto_async(servico_async, db_async_mock):
    resultado = await servico_async.criar("Teclado", 250.0)
    assert resultado.nome == "Teclado"

# Teste com mocking de requests
@patch("services.email_service.requests.post")
def test_enviar_email(mock_post):
    mock_post.return_value.status_code = 200
    from services.email_service import EnviarEmail
    resultado = EnviarEmail("test@email.com", "Assunto", "Corpo")
    assert resultado is True
    mock_post.assert_called_once()
```

```bash
# Executar testes
pytest                               # todos os testes
pytest -v                            # verboso
pytest tests/test_servicos.py        # arquivo específico
pytest -k "test_criar"               # filtrar por nome
pytest --cov=src --cov-report=html   # relatório de cobertura
pytest -x                            # parar no primeiro erro
pytest --tb=short                    # traceback resumido
```

---

### Testes com Django

```python
# core/tests/test_models.py
from django.test import TestCase
from django.contrib.auth.models import User
from decimal import Decimal
from core.models import Produto, Categoria, Pedido

class ProdutoTests(TestCase):
    def setUp(self):
        self.categoria = Categoria.objects.create(nome="Eletrônicos")
        self.produto = Produto.objects.create(
            nome="Notebook",
            preco=Decimal("3500.00"),
            estoque=10,
            categoria=self.categoria,
        )

    def test_str_retorna_nome_e_preco(self):
        self.assertIn("Notebook", str(self.produto))

    def test_estoque_nao_negativo(self):
        from django.core.exceptions import ValidationError
        self.produto.estoque = -1
        with self.assertRaises(ValidationError):
            self.produto.full_clean()

# core/tests/test_api.py
from rest_framework.test import APITestCase
from rest_framework import status

class ProdutoAPITests(APITestCase):
    def setUp(self):
        self.user = User.objects.create_superuser("admin", "admin@test.com", "senha")
        self.client.force_authenticate(user=self.user)
        self.categoria = Categoria.objects.create(nome="TI")

    def test_listar_produtos(self):
        resposta = self.client.get("/api/v1/produtos/")
        self.assertEqual(resposta.status_code, status.HTTP_200_OK)

    def test_criar_produto(self):
        resposta = self.client.post("/api/v1/produtos/", {
            "nome": "Mouse", "preco": "150.00", "estoque": 20,
            "categoria": self.categoria.id
        })
        self.assertEqual(resposta.status_code, status.HTTP_201_CREATED)
        self.assertEqual(resposta.data["nome"], "Mouse")
```

---

## Referências

### Documentação Oficial

- **Python 3** — [docs.python.org](https://docs.python.org/3/)
- **FastAPI** — [fastapi.tiangolo.com](https://fastapi.tiangolo.com/)
- **Django** — [docs.djangoproject.com](https://docs.djangoproject.com/)
- **SQLAlchemy 2.x** — [docs.sqlalchemy.org](https://docs.sqlalchemy.org/)
- **scikit-learn** — [scikit-learn.org](https://scikit-learn.org/)
- **PyTorch** — [pytorch.org/docs](https://pytorch.org/docs/stable/index.html)
- **Hugging Face** — [huggingface.co/docs](https://huggingface.co/docs)
- **pandas** — [pandas.pydata.org](https://pandas.pydata.org/docs/)
- **OpenCV** — [docs.opencv.org](https://docs.opencv.org/)
- **Playwright** — [playwright.dev/python](https://playwright.dev/python/)

### Livros e Cursos

| Recurso | Área | Nível |
|---|---|---|
| *Fluent Python* (Luciano Ramalho) | Python avançado | Intermediário/Avançado |
| *Python Crash Course* (Eric Matthes) | Fundamentos | Iniciante |
| *Hands-On Machine Learning* (Aurélien Géron) | ML/DL | Intermediário |
| *Two Scoops of Django* | Django best practices | Intermediário |
| *FastAPI — From Zero to Production* | FastAPI | Iniciante/Intermediário |
| CS50P (Harvard, gratuito) | Python | Iniciante |
| fast.ai — Practical Deep Learning | Deep Learning | Intermediário |

### Recursos da Comunidade

- **PyPI** (pypi.org) — repositório de pacotes Python
- **Real Python** (realpython.com) — tutoriais aprofundados
- **PEP 8** — guia de estilo oficial de código Python
- **PEP 20** — o Zen do Python (`import this`)
- **Awesome Python** (github.com/vinta/awesome-python) — lista curada de bibliotecas
- **Python Discord** — comunidade ativa para dúvidas

# Aula 1 — Objetos e classes em Python: do uso cotidiano à criação de classes simples

**Pré-requisitos:** variáveis, tipos, condicionais, loops, funções, listas, dicionários, tuplas.

**Objetivo da aula:** entender o que é Programação Orientada a Objetos (OOP), reconhecer objetos que já usamos em Python e criar classes simples para agrupar dados e comportamentos.

> **Nota de vocabulário:** **OOP** é a sigla para *Object-Oriented Programming* (Programação Orientada a Objetos). É um estilo de organizar código que agrupa dados e funções relacionadas em unidades chamadas **objetos**.

---

## 0. Introdução conceitual: o que é OOP

### 0.1 Como escrevemos código até agora

Até este momento, a maior parte do nosso código em Python segue um estilo **procedural** (organizado em sequência de passos e funções separadas dos dados):

- criamos variáveis para guardar dados;
- escrevemos funções que recebem essas variáveis e fazem algo com elas;
- os dados e as funções ficam separados.

Exemplo baseado no gerenciador de tarefas que já fizemos:

```python
tarefas = []

def adicionar_tarefa(lista, descricao):
    lista.append(descricao)

def concluir_tarefa(lista, posicao):
    lista[posicao] = "✅ " + lista[posicao]
```

Aqui, `tarefas` é uma lista de strings. A função `adicionar_tarefa` recebe a lista e altera a lista. A função `concluir_tarefa` também recebe a lista e altera a lista.

Isso funciona bem para problemas simples.

---

### 0.2 O problema que aparece quando o código cresce

Suponha que uma tarefa não seja apenas uma string. Ela pode ter:

```python
descricao = "Estudar Python"
concluida = False
prazo = "2026-05-20"
prioridade = "alta"
```

Se tivermos várias tarefas, precisamos de várias listas paralelas ou de uma lista de dicionários:

```python
tarefas = [
    {"descricao": "Estudar Python", "concluida": False},
    {"descricao": "Fazer exercícios", "concluida": True},
]
```

Agora, para concluir uma tarefa, precisamos escrever uma função separada:

```python
def concluir_tarefa(tarefa):
    tarefa["concluida"] = True
```

E para validar:

```python
def validar_tarefa(tarefa):
    if not tarefa["descricao"]:
        return False
    return True
```

E para formatar:

```python
def resumo_tarefa(tarefa):
    status = "✅" if tarefa["concluida"] else "⬜"
    return f"{status} {tarefa['descricao']}"
```

O problema é que os dados (`descricao`, `concluida`) e as operações (`concluir`, `validar`, `resumo`) ficam espalhados. Nada garante que todas as partes do código usem as mesmas regras.

---

### 0.3 A ideia central da OOP

Programação Orientada a Objetos resolve isso juntando duas coisas que pertencem uma à outra:

1. **dados** que descrevem algo;
2. **operações** que podem ser feitas com esses dados.

Essa junção é chamada de **objeto**.

> **Definição rápida:** um **objeto** é uma unidade de código que combina dados (valores guardados) e funções que operam sobre esses dados.

Um objeto guarda informações e também possui funções associadas a essas informações.

Em vez de:

```python
tarefa = {"descricao": "Estudar Python", "concluida": False}
concluir_tarefa(tarefa)
```

Podemos escrever:

```python
tarefa = Tarefa("Estudar Python")
tarefa.concluir()
```

A diferença principal não é apenas sintática. A diferença é que a tarefa passa a ser uma unidade que sabe:

- quais dados ela possui;
- quais operações podem ser feitas com ela.

---

### 0.4 Classe e instância

Para criar objetos, usamos uma **classe**.

> **Definição rápida:** uma **classe** é um molde que define quais dados e quais funções um tipo de objeto terá. Uma **instância** é um objeto concreto criado a partir desse molde.

Uma classe define:

- quais dados um objeto terá;
- quais métodos um objeto poderá executar.

Exemplo:

```python
tarefa1 = Tarefa("Estudar Python")
tarefa2 = Tarefa("Fazer exercícios")
```

Aqui, `Tarefa` é a classe. `tarefa1` e `tarefa2` são instâncias, ou seja, objetos concretos criados a partir da classe `Tarefa`.

Cada instância guarda seus próprios dados. `tarefa1` tem uma descrição, `tarefa2` tem outra.

---

### 0.5 Estado e comportamento

Dois termos aparecem com frequência em OOP:

| Termo | Significado | Exemplo em uma tarefa |
|---|---|---|
| Estado | Dados guardados pelo objeto | `descricao`, `concluida` |
| Comportamento | Ações que o objeto sabe executar | `concluir()`, `resumo()` |

> **Definição rápida:** **estado** é o conjunto de valores que um objeto guarda em um dado momento. **Comportamento** são as ações que o objeto pode executar.

Uma forma simples de pensar:

- estado é o que o objeto **sabe**;
- comportamento é o que o objeto **faz**.

---

### 0.6 OOP não é sempre obrigatório

OOP é útil quando:

- temos dados que sempre andam juntos;
- existem regras e operações que pertencem a esses dados;
- o mesmo tipo de dado aparece várias vezes no código;
- queremos organizar melhor um projeto que está crescendo.

OOP não é necessário quando:

- o código é pequeno e direto;
- uma função simples resolve;
- os dados são apenas transporte, como um dicionário vindo de JSON;
- a lógica é puramente cálculo sem estado próprio.

Exemplo: a função `calcular_imc` vista em Flask pode continuar sendo uma função simples:

```python
def calcular_imc(peso, altura):
    return peso / (altura ** 2)
```

Não há motivo para transformar isso em classe se ela apenas recebe valores e retorna um resultado.

O objetivo desta aula não é transformar tudo em classe. É aprender a criar classes quando isso ajuda a organizar o código.

---

## 1. Você já usa objetos sem perceber

### 1.1 Listas são objetos

Quando escrevemos:

```python
frutas = ["maçã", "banana"]
frutas.append("laranja")
```

Estamos chamando o método `append()` da lista `frutas`.

> **Definição rápida:** um **método** é uma função que pertence a um objeto e é chamada usando a sintaxe `objeto.metodo()`.

A lista:

- guarda dados: `["maçã", "banana", "laranja"]`;
- possui comportamentos: `.append()`, `.pop()`, `.sort()`, `.reverse()`.

---

### 1.2 Dicionários são objetos

Quando escrevemos:

```python
usuario = {"nome": "Ana", "idade": 28}
nome = usuario.get("nome")
```

Estamos chamando o método `.get()` do dicionário `usuario`.

O dicionário:

- guarda pares chave-valor;
- possui métodos como `.get()`, `.keys()`, `.values()`, `.items()`, `.pop()`.

---

### 1.3 Strings são objetos

Quando escrevemos:

```python
entrada = "  42  "
entrada_limpa = entrada.strip()
```

Estamos chamando o método `.strip()` da string `entrada`.

Strings possuem métodos como:

```python
texto.strip()
texto.upper()
texto.isdigit()
texto.replace("a", "b")
```

---

### 1.4 O padrão objeto.metodo()

A sintaxe:

```python
objeto.metodo(argumentos)
```

significa:

> chame o método `metodo` pertencente ao objeto `objeto`, passando `argumentos`.

Isso já é orientação a objetos na prática.

A diferença agora é que vamos aprender a criar nossos próprios objetos, em vez de usar apenas os objetos que já vêm prontos no Python.

---

## 2. Criando uma classe simples

### 2.1 Exemplo inicial: Tarefa

Vamos criar uma classe para representar uma tarefa.

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False
```

Explicando cada parte:

```python
class Tarefa:
```

Cria uma classe chamada `Tarefa`.

Por convenção, nomes de classes em Python começam com letra maiúscula.

```python
def __init__(self, descricao):
```

Define o método `__init__`.

> **Definição rápida:** `__init__` é o **construtor** da classe, ou seja, o método executado automaticamente quando um novo objeto é criado. Ele serve para inicializar o objeto.

```python
self.descricao = descricao
```

Cria um **atributo** chamado `descricao` no objeto e guarda nele o valor recebido.

> **Definição rápida:** um **atributo** é um dado armazenado dentro de um objeto. Ele é acessado com a sintaxe `objeto.nome_do_atributo`.

```python
self.concluida = False
```

Cria um atributo chamado `concluida` e define seu valor inicial como `False`.

---

### 2.2 Criando objetos da classe

Depois de definir a classe, podemos criar objetos:

```python
tarefa1 = Tarefa("Estudar Python")
tarefa2 = Tarefa("Fazer exercícios")
```

A linha:

```python
tarefa1 = Tarefa("Estudar Python")
```

faz o seguinte:

1. cria um novo objeto do tipo `Tarefa`;
2. chama automaticamente o método `__init__`;
3. passa `"Estudar Python"` como valor de `descricao`;
4. devolve o objeto criado.

> **Definição rápida:** o ato de criar um objeto a partir de uma classe é chamado de **instanciação**. Dizemos que `tarefa1` é uma instância de `Tarefa`.

Depois disso:

```python
print(tarefa1.descricao)    # Estudar Python
print(tarefa1.concluida)    # False
```

---

### 2.3 Cada objeto tem seu próprio estado

```python
tarefa1 = Tarefa("Estudar Python")
tarefa2 = Tarefa("Fazer exercícios")

tarefa1.concluida = True

print(tarefa1.concluida)    # True
print(tarefa2.concluida)    # False
```

Alterar `tarefa1` não altera `tarefa2`. Cada instância guarda seus próprios atributos.

---

## 3. O parâmetro self

### 3.1 O que é self

O `self` representa o próprio objeto que está usando o método.

> **Definição rápida:** `self` é o parâmetro que recebe automaticamente a referência ao objeto atual dentro de um método.

Quando escrevemos:

```python
tarefa1.concluir()
```

o Python interpreta isso aproximadamente como:

```python
Tarefa.concluir(tarefa1)
```

Ou seja, o objeto `tarefa1` é passado automaticamente como primeiro argumento para o método. Esse primeiro argumento é chamado `self`.

---

### 3.2 Por que self precisa aparecer

Dentro da classe, precisamos usar `self` para acessar os atributos do objeto atual.

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False
```

Se escrevermos apenas:

```python
descricao = descricao
```

estaremos criando uma variável local chamada `descricao`, e não um atributo do objeto.

O correto é:

```python
self.descricao = descricao
```

---

### 3.3 Exemplo do erro comum

```python
class Tarefa:
    def __init__(self, descricao):
        descricao = descricao  # errado: não cria atributo no objeto
```

Isso não causa erro imediato, mas também não guarda a descrição no objeto.

Depois:

```python
t = Tarefa("Estudar")
print(t.descricao)  # AttributeError: 'Tarefa' object has no attribute 'descricao'
```

O erro acontece porque o atributo nunca foi criado.

---

## 4. Métodos: comportamento do objeto

### 4.1 Método simples

Vamos adicionar um método para concluir a tarefa:

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

    def concluir(self):
        self.concluida = True
```

Uso:

```python
tarefa = Tarefa("Estudar Python")

print(tarefa.concluida)   # False

tarefa.concluir()

print(tarefa.concluida)   # True
```

O método `concluir()` altera o estado do objeto. Ele modifica o atributo `concluida`.

---

### 4.2 Método que retorna informação

Um método nem sempre altera o objeto. Ele também pode apenas calcular ou devolver alguma informação.

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

    def concluir(self):
        self.concluida = True

    def resumo(self):
        if self.concluida:
            return f"✅ {self.descricao}"
        else:
            return f"⬜ {self.descricao}"
```

Uso:

```python
tarefa = Tarefa("Estudar Python")

print(tarefa.resumo())  # ⬜ Estudar Python

tarefa.concluir()

print(tarefa.resumo())  # ✅ Estudar Python
```

---

### 4.3 Métodos que alteram estado geralmente retornam None

Assim como acontece com métodos de listas:

```python
frutas = ["maçã"]
resultado = frutas.append("banana")
print(resultado)  # None
```

Métodos que alteram o objeto normalmente não precisam retornar nada.

```python
def concluir(self):
    self.concluida = True
```

Se um método não tem `return`, ele retorna `None`.

Isso não é um erro. É o comportamento esperado quando o objetivo do método é alterar o próprio objeto.

---

## 5. Exemplo mais completo: Produto

Vamos criar uma classe para representar um produto.

```python
class Produto:
    def __init__(self, nome, preco, estoque):
        self.nome = nome
        self.preco = preco
        self.estoque = estoque

    def validar(self):
        if not self.nome:
            return False

        if self.preco <= 0:
            return False

        if self.estoque < 0:
            return False

        return True

    def resumo(self):
        return f"{self.nome}: R$ {self.preco:.2f} ({self.estoque} em estoque)"
```

Uso:

```python
produto = Produto("Teclado", 150.0, 10)

print(produto.resumo())      # Teclado: R$ 150.00 (10 em estoque)
print(produto.validar())     # True
```

Testando um produto inválido:

```python
produto_invalido = Produto("", -50, 3)

print(produto_invalido.validar())  # False
```

---

## 6. Objetos e dicionários

### 6.1 Método to_dict()

Em aplicações Web, frequentemente precisamos transformar dados em JSON.

No Flask, já vimos que retornar um dicionário produz JSON automaticamente:

```python
@app.route("/api/status")
def status():
    return {"servidor": "ok"}
```

Para facilitar essa ponte, podemos criar um método `to_dict()` nas nossas classes:

> **Definição rápida:** `to_dict()` não é um método especial do Python. É uma **convenção** (um nome comum adotado pela comunidade) para um método que converte os dados do objeto em um dicionário.

```python
class Produto:
    def __init__(self, nome, preco, estoque):
        self.nome = nome
        self.preco = preco
        self.estoque = estoque

    def to_dict(self):
        return {
            "nome": self.nome,
            "preco": self.preco,
            "estoque": self.estoque,
        }
```

Uso:

```python
produto = Produto("Teclado", 150.0, 10)

dados = produto.to_dict()

print(dados)
# {'nome': 'Teclado', 'preco': 150.0, 'estoque': 10}
```

Esse dicionário pode ser retornado por uma rota Flask:

```python
@app.route("/api/produto")
def api_produto():
    produto = Produto("Teclado", 150.0, 10)
    return produto.to_dict()
```

---

### 6.2 Classe não substitui dicionário em todos os casos

Dicionários continuam sendo muito úteis.

Use dicionário quando:

- os dados são simples;
- você precisa de acesso por chave;
- os dados vieram de JSON;
- não há comportamento associado.

Exemplo:

```python
usuario = {
    "nome": "Ana",
    "idade": 28,
}
```

Use classe quando:

- existem regras associadas aos dados;
- existem métodos que pertencem àquele dado;
- o mesmo tipo de entidade aparece várias vezes;
- você quer organizar melhor o código.

Exemplo:

```python
class Usuario:
    def __init__(self, nome, idade):
        self.nome = nome
        self.idade = idade

    def maior_de_idade(self):
        return self.idade >= 18
```

A escolha entre dicionário e classe depende do problema. Não existe regra absoluta.

---

## 7. Validação dentro do objeto

### 7.1 Validação simples

A validação pode ficar dentro do objeto quando a regra pertence ao objeto.

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

    def validar(self):
        if not self.descricao:
            return False

        if len(self.descricao) > 100:
            return False

        return True
```

Uso:

```python
tarefa = Tarefa("")

if tarefa.validar():
    print("Tarefa válida")
else:
    print("Tarefa inválida")
```

---

### 7.2 Validação com exceção

Em alguns casos, em vez de retornar `True` ou `False`, podemos levantar uma exceção.

> **Definição rápida:** uma **exceção** é um objeto que representa um erro. Levantar uma exceção (com `raise`) interrompe a execução normal e sinaliza que algo deu errado. Já vimos isso em Flask com `ValueError`.

Já vimos isso em Flask:

```python
def calcular_imc(peso, altura):
    if altura <= 0 or peso <= 0:
        raise ValueError("peso e altura devem ser positivos")

    imc = peso / (altura ** 2)
    return {"imc": round(imc, 2)}
```

Podemos fazer algo semelhante dentro de um método:

```python
class Produto:
    def __init__(self, nome, preco):
        self.nome = nome
        self.preco = preco

    def validar_ou_erro(self):
        if not self.nome:
            raise ValueError("nome não pode ser vazio")

        if self.preco <= 0:
            raise ValueError("preço deve ser positivo")
```

Nesta aula, usaremos principalmente validações que retornam `True` ou `False`, por serem mais simples. Mais adiante, poderemos usar exceções para integrar melhor com Flask.

---

## 8. Objeto e referência

### 8.1 Variáveis guardam referências

Em Python, variáveis não guardam o objeto diretamente. Elas guardam uma **referência** para o objeto.

> **Definição rápida:** uma **referência** é um valor interno que aponta para onde o objeto está armazenado na memória. Duas variáveis podem apontar para o mesmo objeto.

Isso tem consequências importantes.

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao

tarefa1 = Tarefa("Estudar Python")
tarefa2 = tarefa1
```

Aqui, `tarefa2` não cria uma nova tarefa. Ela aponta para o mesmo objeto que `tarefa1`.

```python
tarefa2.descricao = "Estudar Flask"

print(tarefa1.descricao)  # Estudar Flask
```

O mesmo já acontecia com listas:

```python
lista_a = [1, 2, 3]
lista_b = lista_a

lista_b.append(4)

print(lista_a)  # [1, 2, 3, 4]
```

---

### 8.2 Criando objetos independentes

Se quisermos uma cópia independente, precisamos criar outro objeto:

```python
tarefa1 = Tarefa("Estudar Python")
tarefa2 = Tarefa("Estudar Python")
```

Agora:

```python
tarefa2.descricao = "Estudar Flask"

print(tarefa1.descricao)  # Estudar Python
print(tarefa2.descricao)  # Estudar Flask
```

---

## 9. Erros comuns

### 9.1 Esquecer self na definição do método

Errado:

```python
class Tarefa:
    def concluir():
        self.concluida = True
```

Correto:

```python
class Tarefa:
    def concluir(self):
        self.concluida = True
```

---

### 9.2 Esquecer self ao acessar atributos

Errado:

```python
def resumo(self):
    return descricao
```

Correto:

```python
def resumo(self):
    return self.descricao
```

---

### 9.3 Criar classe quando uma função simples bastaria

Nem tudo precisa ser classe.

Se você só precisa calcular algo:

```python
def calcular_total(preco, quantidade):
    return preco * quantidade
```

não há necessidade de:

```python
class CalculadoraTotal:
    def calcular(self, preco, quantidade):
        return preco * quantidade
```

Use classe quando houver estado e comportamento que pertencem ao mesmo conceito.

---

### 9.4 Usar classe apenas para agrupar dados sem comportamento

Se o objetivo é apenas agrupar dados simples, um dicionário pode ser suficiente:

```python
produto = {
    "nome": "Teclado",
    "preco": 150.0,
    "estoque": 10,
}
```

Uma classe começa a fazer mais sentido quando existem métodos como `validar()`, `to_dict()`, `resumo()`, `baixar_estoque()`, etc.

---

## 10. Exercícios

### Exercício 1 — Reconhecer objetos

Observe os códigos abaixo e identifique:

- qual é o objeto;
- qual é o método chamado;
- qual é o estado provável desse objeto.

```python
frutas = ["maçã", "banana"]
frutas.append("laranja")
```

```python
usuario = {"nome": "Ana"}
nome = usuario.get("nome")
```

```python
texto = "  42  "
numero = texto.strip()
```

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
# 1) Objeto: lista frutas | Método: .append() | Estado: elementos armazenados ["maçã", "banana", "laranja"]
# 2) Objeto: dicionário usuario | Método: .get() | Estado: pares chave-valor {"nome": "Ana"}
# 3) Objeto: string texto | Método: .strip() | Estado: sequência de caracteres "  42  "
```

</details>

---

### Exercício 2 — Classe Livro

Crie uma classe chamada `Livro`.

Ela deve ter os atributos:

- `titulo`;
- `autor`;
- `ano_publicacao`.

Depois crie dois objetos e imprima seus atributos.

Exemplo de uso esperado:

```python
livro1 = Livro("O Senhor dos Anéis", "J.R.R. Tolkien", 1954)
livro2 = Livro("1984", "George Orwell", 1949)

print(livro1.titulo)
print(livro2.autor)
```

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class Livro:
    def __init__(self, titulo, autor, ano_publicacao):
        self.titulo = titulo
        self.autor = autor
        self.ano_publicacao = ano_publicacao

livro1 = Livro("O Senhor dos Anéis", "J.R.R. Tolkien", 1954)
livro2 = Livro("1984", "George Orwell", 1949)

print(livro1.titulo)   # O Senhor dos Anéis
print(livro2.autor)    # George Orwell
```

</details>

---

### Exercício 3 — Método to_dict()

Adicione à classe `Livro` um método chamado `to_dict()`.

Ele deve retornar um dicionário com os dados do livro.

Exemplo esperado:

```python
livro = Livro("1984", "George Orwell", 1949)

print(livro.to_dict())
```

Saída esperada:

```python
{
    "titulo": "1984",
    "autor": "George Orwell",
    "ano_publicacao": 1949
}
```

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class Livro:
    def __init__(self, titulo, autor, ano_publicacao):
        self.titulo = titulo
        self.autor = autor
        self.ano_publicacao = ano_publicacao

    def to_dict(self):
        return {
            "titulo": self.titulo,
            "autor": self.autor,
            "ano_publicacao": self.ano_publicacao,
        }

livro = Livro("1984", "George Orwell", 1949)
print(livro.to_dict())
# {'titulo': '1984', 'autor': 'George Orwell', 'ano_publicacao': 1949}
```

</details>

---

### Exercício 4 — Classe Produto com validação

Crie uma classe chamada `Produto`.

Atributos:

- `nome`;
- `preco`;
- `estoque`.

Métodos:

- `validar()`: retorna `False` se `nome` for vazio, `preco` for menor ou igual a zero, ou `estoque` for menor que zero. Caso contrário, retorna `True`;
- `resumo()`: retorna uma string formatada com os dados do produto.

Teste com um produto válido e um produto inválido.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class Produto:
    def __init__(self, nome, preco, estoque):
        self.nome = nome
        self.preco = preco
        self.estoque = estoque

    def validar(self):
        return bool(self.nome) and self.preco > 0 and self.estoque >= 0

    def resumo(self):
        return f"{self.nome}: R$ {self.preco:.2f} ({self.estoque} em estoque)"

p1 = Produto("Teclado", 150.0, 10)
p2 = Produto("", -50, 3)

print(p1.validar())   # True
print(p1.resumo())    # Teclado: R$ 150.00 (10 em estoque)
print(p2.validar())   # False
```

</details>

---

### Exercício 5 — Classe Tarefa com estado

Crie uma classe chamada `Tarefa`.

Atributos:

- `descricao`;
- `concluida`, iniciando como `False`.

Métodos:

- `concluir()`: marca a tarefa como concluída;
- `resumo()`: retorna uma string com um marcador visual e a descrição.

Exemplo:

```python
tarefa = Tarefa("Estudar Python")

print(tarefa.resumo())   # ⬜ Estudar Python

tarefa.concluir()

print(tarefa.resumo())   # ✅ Estudar Python
```

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

    def concluir(self):
        self.concluida = True

    def resumo(self):
        return f"{'✅' if self.concluida else '⬜'} {self.descricao}"

tarefa = Tarefa("Estudar Python")
print(tarefa.resumo())   # ⬜ Estudar Python
tarefa.concluir()
print(tarefa.resumo())   # ✅ Estudar Python
```

</details>

---

### Exercício 6 — Lista de objetos

Crie uma lista com três objetos `Tarefa`.

Depois use um loop `for` para imprimir o resumo de cada tarefa.

Exemplo esperado:

```python
tarefas = [
    Tarefa("Estudar Python"),
    Tarefa("Fazer exercícios"),
    Tarefa("Revisar HTML"),
]

for tarefa in tarefas:
    print(tarefa.resumo())
```

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
tarefas = [
    Tarefa("Estudar Python"),
    Tarefa("Fazer exercícios"),
    Tarefa("Revisar HTML"),
]

for tarefa in tarefas:
    print(tarefa.resumo())

# ⬜ Estudar Python
# ⬜ Fazer exercícios
# ⬜ Revisar HTML
```

</details>

---

### Exercício 7 — Desafio simples

Adicione à classe `Tarefa` um método chamado `to_dict()`.

Ele deve retornar:

```python
{
    "descricao": "Estudar Python",
    "concluida": False
}
```

Depois crie uma lista de tarefas, conclua uma delas e imprima a lista de dicionários resultante.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

    def concluir(self):
        self.concluida = True

    def to_dict(self):
        return {"descricao": self.descricao, "concluida": self.concluida}

tarefas = [
    Tarefa("Estudar Python"),
    Tarefa("Fazer exercícios"),
    Tarefa("Revisar HTML"),
]

tarefas[0].concluir()

for tarefa in tarefas:
    print(tarefa.to_dict())

# {'descricao': 'Estudar Python', 'concluida': True}
# {'descricao': 'Fazer exercícios', 'concluida': False}
# {'descricao': 'Revisar HTML', 'concluida': False}
```

</details>

---

## 11. Checklist da aula

Ao final desta aula, você deve conseguir:

- explicar o que é um objeto;
- explicar a diferença entre classe e instância;
- explicar o que é estado e comportamento;
- reconhecer objetos já existentes em Python, como listas, dicionários e strings;
- criar uma classe simples;
- usar `__init__` para inicializar atributos;
- entender o papel do `self`;
- criar métodos simples;
- instanciar objetos;
- acessar atributos;
- chamar métodos;
- criar um método `to_dict()`;
- decidir quando uma classe é útil e quando uma função ou dicionário é suficiente.

---

## 12. Recapitulação

| Conceito | Significado |
|---|---|
| Objeto | Unidade que junta dados e comportamentos relacionados |
| Classe | Molde usado para criar objetos |
| Instância | Objeto concreto criado a partir de uma classe |
| Atributo | Dado guardado pelo objeto |
| Método | Função que pertence ao objeto |
| Estado | Conjunto de atributos e valores atuais do objeto |
| Comportamento | Ações que o objeto pode executar |
| `__init__` | Método executado na criação do objeto |
| `self` | Referência ao próprio objeto dentro da classe |
| `to_dict()` | Método útil para converter objeto em dicionário |

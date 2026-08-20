# Aula 1 — Código-fonte completo e executável

Abaixo estão todos os arquivos necessários para demonstrar os conceitos da Aula 1. Cada arquivo é **auto-contido**: pode ser executado diretamente com `python nome_do_arquivo.py`, sem depender de nenhum outro arquivo.

## Estrutura de pastas sugerida

```
aula1_oop/
├── exemplos/
│   ├── 01_objetos_embutidos.py
│   ├── 02_classe_tarefa.py
│   ├── 03_metodos_estado.py
│   ├── 04_produto_validacao.py
│   ├── 05_referencias.py
│   ├── 06_to_dict_json.py
│   └── 07_lista_objetos.py
└── exercicios_resolvidos/
    ├── ex02_livro.py
    ├── ex04_produto.py
    └── ex05_tarefa.py
```

---

## Exemplos

### `exemplos/01_objetos_embutidos.py`

Demonstra que listas, dicionários e strings já são objetos com métodos.

```python
# 01_objetos_embutidos.py
# Objetivo: mostrar que objetos já existem no código que escrevemos.
# Execute: python 01_objetos_embutidos.py

# --- Listas são objetos ---
frutas = ["maçã", "banana"]
frutas.append("laranja")       # método .append() pertence à lista
frutas.sort()                  # método .sort() pertence à lista
print("Lista após append e sort:", frutas)

# --- Dicionários são objetos ---
usuario = {"nome": "Ana", "idade": 28}
nome = usuario.get("nome")     # método .get() pertence ao dicionário
chaves = list(usuario.keys())  # método .keys() pertence ao dicionário
print("Nome:", nome)
print("Chaves:", chaves)

# --- Strings são objetos ---
texto = "  42  "
limpo = texto.strip()          # método .strip() pertence à string
print("Texto limpo:", limpo)
print("É dígito?", limpo.isdigit())
```

**Saída esperada:**

```
Lista após append e sort: ['banana', 'laranja', 'maçã']
Nome: Ana
Chaves: ['nome', 'idade']
Texto limpo: 42
É dígito? True
```

---

### `exemplos/02_classe_tarefa.py`

Primeira classe: `__init__`, `self`, atributos de instância.

```python
# 02_classe_tarefa.py
# Objetivo: criar a primeira classe e entender __init__, self e atributos.
# Execute: python 02_classe_tarefa.py

class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

# Criando instâncias (objetos concretos)
tarefa1 = Tarefa("Estudar Python")
tarefa2 = Tarefa("Fazer exercícios")

# Acessando atributos
print("tarefa1.descricao:", tarefa1.descricao)
print("tarefa1.concluida:", tarefa1.concluida)
print("tarefa2.descricao:", tarefa2.descricao)

# Cada objeto tem seu próprio estado
tarefa1.concluida = True
print("Após alterar tarefa1:")
print("tarefa1.concluida:", tarefa1.concluida)
print("tarefa2.concluida:", tarefa2.concluida)  # não foi alterada
```

**Saída esperada:**

```
tarefa1.descricao: Estudar Python
tarefa1.concluida: False
tarefa2.descricao: Fazer exercícios
Após alterar tarefa1:
tarefa1.concluida: True
tarefa2.concluida: False
```

---

### `exemplos/03_metodos_estado.py`

Métodos que alteram estado e métodos que retornam informação.

```python
# 03_metodos_estado.py
# Objetivo: métodos como comportamento do objeto.
# Execute: python 03_metodos_estado.py

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

tarefa = Tarefa("Estudar Python")

print("Antes de concluir:", tarefa.resumo())
tarefa.concluir()
print("Depois de concluir:", tarefa.resumo())
```

**Saída esperada:**

```
Antes de concluir: ⬜ Estudar Python
Depois de concluir: ✅ Estudar Python
```

---

### `exemplos/04_produto_validacao.py`

Classe com validação e método `resumo()`.

```python
# 04_produto_validacao.py
# Objetivo: validação dentro do objeto.
# Execute: python 04_produto_validacao.py

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

# Produto válido
produto1 = Produto("Teclado", 150.0, 10)
print(produto1.resumo())
print("produto1 válido?", produto1.validar())

print()

# Produto inválido (nome vazio e preço negativo)
produto2 = Produto("", -50, 3)
print("produto2 válido?", produto2.validar())
```

**Saída esperada:**

```
Teclado: R$ 150.00 (10 em estoque)
produto1 válido? True

produto2 válido? False
```

---

### `exemplos/05_referencias.py`

Variáveis guardam referências, não cópias.

```python
# 05_referencias.py
# Objetivo: entender que variáveis apontam para o mesmo objeto.
# Execute: python 05_referencias.py

class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao

# --- Duas variáveis apontando para o MESMO objeto ---
tarefa1 = Tarefa("Estudar Python")
tarefa2 = tarefa1  # tarefa2 aponta para o mesmo objeto

tarefa2.descricao = "Estudar Flask"

print("Após alterar tarefa2.descricao:")
print("tarefa1.descricao:", tarefa1.descricao)  # foi alterado também
print("tarefa2.descricao:", tarefa2.descricao)

print()

# --- Criando objetos independentes ---
tarefa3 = Tarefa("Estudar Python")
tarefa4 = Tarefa("Estudar Python")

tarefa4.descricao = "Estudar Flask"

print("Objetos independentes:")
print("tarefa3.descricao:", tarefa3.descricao)  # não foi alterado
print("tarefa4.descricao:", tarefa4.descricao)
```

**Saída esperada:**

```
Após alterar tarefa2.descricao:
tarefa1.descricao: Estudar Flask
tarefa2.descricao: Estudar Flask

Objetos independentes:
tarefa3.descricao: Estudar Python
tarefa4.descricao: Estudar Flask
```

---

### `exemplos/06_to_dict_json.py`

Método `to_dict()` e ponte com JSON.

```python
# 06_to_dict_json.py
# Objetivo: converter objeto em dicionário e depois em JSON.
# Execute: python 06_to_dict_json.py

import json

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

produto = Produto("Teclado", 150.0, 10)

# Objeto -> dicionário
dados = produto.to_dict()
print("Dicionário:", dados)

# Dicionário -> string JSON (equivalente ao que Flask faz com jsonify)
json_str = json.dumps(dados, ensure_ascii=False, indent=2)
print("JSON:")
print(json_str)

# String JSON -> dicionário (equivalente ao que o navegador faz)
dados_de_volta = json.loads(json_str)
print("Nome vindo do JSON:", dados_de_volta["nome"])
```

**Saída esperada:**

```
Dicionário: {'nome': 'Teclado', 'preco': 150.0, 'estoque': 10}
JSON:
{
  "nome": "Teclado",
  "preco": 150.0,
  "estoque": 10
}
Nome vindo do JSON: Teclado
```

---

### `exemplos/07_lista_objetos.py`

Lista de objetos e iteração com `for`.

```python
# 07_lista_objetos.py
# Objetivo: lista de objetos e iteração.
# Execute: python 07_lista_objetos.py

class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

    def concluir(self):
        self.concluida = True

    def resumo(self):
        status = "✅" if self.concluida else "⬜"
        return f"{status} {self.descricao}"

# Lista de objetos
tarefas = [
    Tarefa("Estudar Python"),
    Tarefa("Fazer exercícios"),
    Tarefa("Revisar HTML"),
]

# Concluir a primeira tarefa
tarefas[0].concluir()

# Iterar sobre objetos
for tarefa in tarefas:
    print(tarefa.resumo())
```

**Saída esperada:**

```
✅ Estudar Python
⬜ Fazer exercícios
⬜ Revisar HTML
```

---

## Exercícios resolvidos

### `exercicios_resolvidos/ex02_livro.py`

Exercícios 2 e 3: classe `Livro` com `to_dict()`.

```python
# ex02_livro.py
# Exercícios 2 e 3: classe Livro com atributos e to_dict().
# Execute: python ex02_livro.py

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

# Exercício 2: criar dois livros e imprimir atributos
livro1 = Livro("O Senhor dos Anéis", "J.R.R. Tolkien", 1954)
livro2 = Livro("1984", "George Orwell", 1949)

print(livro1.titulo)   # O Senhor dos Anéis
print(livro2.autor)    # George Orwell

print()

# Exercício 3: usar to_dict()
print(livro2.to_dict())
```

**Saída esperada:**

```
O Senhor dos Anéis
George Orwell

{'titulo': '1984', 'autor': 'George Orwell', 'ano_publicacao': 1949}
```

---

### `exercicios_resolvidos/ex04_produto.py`

Exercício 4: classe `Produto` com validação e resumo.

```python
# ex04_produto.py
# Exercício 4: classe Produto com validar() e resumo().
# Execute: python ex04_produto.py

class Produto:
    def __init__(self, nome, preco, estoque):
        self.nome = nome
        self.preco = preco
        self.estoque = estoque

    def validar(self):
        return bool(self.nome) and self.preco > 0 and self.estoque >= 0

    def resumo(self):
        return f"{self.nome}: R$ {self.preco:.2f} ({self.estoque} em estoque)"

# Produto válido
p1 = Produto("Teclado", 150.0, 10)
print("p1 válido?", p1.validar())
print(p1.resumo())

print()

# Produto inválido (nome vazio e preço negativo)
p2 = Produto("", -50, 3)
print("p2 válido?", p2.validar())
```

**Saída esperada:**

```
p1 válido? True
Teclado: R$ 150.00 (10 em estoque)

p2 válido? False
```

---

### `exercicios_resolvidos/ex05_tarefa.py`

Exercícios 5, 6 e 7: classe `Tarefa` com estado, lista de objetos e `to_dict()`.

```python
# ex05_tarefa.py
# Exercícios 5, 6 e 7: classe Tarefa com estado, lista e to_dict().
# Execute: python ex05_tarefa.py

class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False

    def concluir(self):
        self.concluida = True

    def resumo(self):
        status = "✅" if self.concluida else "⬜"
        return f"{status} {self.descricao}"

    def to_dict(self):
        return {
            "descricao": self.descricao,
            "concluida": self.concluida,
        }

# Exercício 5: criar tarefa, imprimir resumo, concluir, imprimir de novo
print("--- Exercício 5 ---")
tarefa = Tarefa("Estudar Python")
print(tarefa.resumo())   # ⬜ Estudar Python
tarefa.concluir()
print(tarefa.resumo())   # ✅ Estudar Python

print()

# Exercício 6: lista de objetos e iteração
print("--- Exercício 6 ---")
tarefas = [
    Tarefa("Estudar Python"),
    Tarefa("Fazer exercícios"),
    Tarefa("Revisar HTML"),
]

for t in tarefas:
    print(t.resumo())

print()

# Exercício 7: to_dict() e lista de dicionários
print("--- Exercício 7 ---")
tarefas[0].concluir()

for t in tarefas:
    print(t.to_dict())
```

**Saída esperada:**

```
--- Exercício 5 ---
⬜ Estudar Python
✅ Estudar Python

--- Exercício 6 ---
⬜ Estudar Python
⬜ Fazer exercícios
⬜ Revisar HTML

--- Exercício 7 ---
{'descricao': 'Estudar Python', 'concluida': True}
{'descricao': 'Fazer exercícios', 'concluida': False}
{'descricao': 'Revisar HTML', 'concluida': False}
```

---

## Como executar

Todos os arquivos podem ser executados diretamente no terminal:

```bash
cd aula1_oop/exemplos
python 01_objetos_embutidos.py
python 02_classe_tarefa.py
python 03_metodos_estado.py
python 04_produto_validacao.py
python 05_referencias.py
python 06_to_dict_json.py
python 07_lista_objetos.py

cd ../exercicios_resolvidos
python ex02_livro.py
python ex04_produto.py
python ex05_tarefa.py
```

Nenhum arquivo depende de outro. Cada um define as classes de que precisa, cria instâncias e imprime os resultados.

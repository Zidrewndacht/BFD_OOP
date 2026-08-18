# Aula 2 — OOP aplicada ao backend Flask + SQLite

**Pré-requisitos:** Aula 1, funções, listas, dicionários, Flask básico, rotas, `request`, retorno JSON, SQLite básico, `sqlite3.Row`, queries parametrizadas.

**Objetivo da aula:** usar Programação Orientada a Objetos para organizar melhor um backend Python com Flask e SQLite, separando dados, regras de negócio e acesso ao banco de dados.

> **Nota de vocabulário:** nesta aula, usaremos OOP principalmente para resolver três problemas práticos:
>
> 1. evitar que dados e regras fiquem espalhados;
> 2. evitar que SQL fique repetido em várias rotas;
> 3. facilitar a conversão de objetos para JSON em APIs.

---

## 0. Revisão e motivação

### 0.1 O que já sabemos fazer sem OOP

Na aula de SQLite, já construímos um gerenciador de tarefas parecido com este:

```python
import sqlite3

DB_NAME = "tarefas.db"

def get_conexao():
    conexao = sqlite3.connect(DB_NAME)
    conexao.row_factory = sqlite3.Row
    return conexao

def inicializar_banco():
    conexao = get_conexao()
    conexao.execute("""
        CREATE TABLE IF NOT EXISTS tarefas (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            descricao TEXT NOT NULL,
            concluida INTEGER DEFAULT 0
        )
    """)
    conexao.commit()
    conexao.close()

def adicionar_tarefa(descricao):
    conexao = get_conexao()
    with conexao:
        conexao.execute(
            "INSERT INTO tarefas (descricao) VALUES (?)",
            (descricao,)
        )
    conexao.close()

def listar_tarefas():
    conexao = get_conexao()
    tarefas = conexao.execute("SELECT * FROM tarefas").fetchall()
    conexao.close()
    return tarefas

def concluir_tarefa(id_tarefa):
    conexao = get_conexao()
    tarefa = conexao.execute(
        "SELECT * FROM tarefas WHERE id = ?",
        (id_tarefa,)
    ).fetchone()

    if tarefa is None:
        print("Tarefa não encontrada.")
    else:
        with conexao:
            conexao.execute(
                "UPDATE tarefas SET concluida = 1 WHERE id = ?",
                (id_tarefa,)
            )
    conexao.close()
```

Esse código funciona.

Mas perceba alguns problemas:

- a tarefa não é uma entidade clara no código;
- os dados vêm como linhas do banco, não como objetos;
- a regra de conclusão está diretamente ligada ao SQL;
- se quisermos retornar isso em uma API Flask, precisamos transformar manualmente cada linha em dicionário;
- validações e regras de negócio ficam espalhadas.

---

### 0.2 O que queremos melhorar

Queremos chegar a algo mais próximo disso:

```python
tarefa = Tarefa("Estudar Flask")

if tarefa.validar():
    repository.inserir(tarefa)
```

E em uma API Flask:

```python
@app.route("/api/tarefas", methods=["GET"])
def listar():
    tarefas = repository.listar()
    return [tarefa.to_dict() for tarefa in tarefas]
```

Ou seja:

- o banco devolve objetos `Tarefa`;
- o objeto sabe se descrever com `resumo()`;
- o objeto sabe se validar com `validar()`;
- o objeto sabe se converter em dicionário com `to_dict()`;
- o SQL fica concentrado em um lugar só.

A ideia é usar OOP para organizar o código, não para burocratizá-lo.

---

## 1. Classe de domínio: Tarefa

### 1.1 Recuperando a classe da Aula 1 de OOP

Na Aula 1, criamos uma classe simples chamada `Tarefa`:

```python
class Tarefa:
    def __init__(self, descricao):
        self.descricao = descricao
        self.concluida = False
```

Agora, como vamos usar banco de dados, precisamos adicionar um novo dado: o `id`.

Quando uma tarefa ainda não foi salva no banco, ela não possui `id`. Por isso, o `id` pode começar como `None`.

```python
class Tarefa:
    def __init__(self, descricao, id=None, concluida=False):
        self.id = id
        self.descricao = descricao
        self.concluida = concluida
```

> **Definição rápida:** `None` representa ausência de valor. Um `id=None` significa que a tarefa ainda não foi salva no banco.

---

### 1.2 Métodos da Tarefa

Podemos recuperar os métodos da Aula 1:

```python
class Tarefa:
    def __init__(self, descricao, id=None, concluida=False):
        self.id = id
        self.descricao = descricao
        self.concluida = concluida

    def validar(self):
        if not isinstance(self.descricao, str):
            return False

        if not self.descricao.strip():
            return False

        if len(self.descricao) > 100:
            return False

        return True

    def concluir(self):
        self.concluida = True

    def resumo(self):
        if self.concluida:
            return f"✅ {self.descricao}"
        return f"⬜ {self.descricao}"

    def to_dict(self):
        return {
            "id": self.id,
            "descricao": self.descricao,
            "concluida": self.concluida,
        }
```

Uso:

```python
tarefa = Tarefa("Estudar Flask")

print(tarefa.validar())  # True
print(tarefa.resumo())   # ⬜ Estudar Flask

tarefa.concluir()

print(tarefa.resumo())   # ✅ Estudar Flask
print(tarefa.to_dict())
```

Saída esperada:

```text
True
⬜ Estudar Flask
✅ Estudar Flask
{'id': None, 'descricao': 'Estudar Flask', 'concluida': True}
```

---

### 1.3 Por que `to_dict()` é importante aqui

No Flask moderno, dicionários e listas viram JSON automaticamente:

```python
@app.route("/api/status")
def status():
    return {"servidor": "ok"}
```

Mas o Flask não sabe transformar automaticamente um objeto `Tarefa` em JSON.

Ou seja, isto não funciona diretamente:

```python
return tarefa  # errado para API JSON
```

Precisamos devolver um dicionário:

```python
return tarefa.to_dict()
```

Esse método `to_dict()` será muito útil nesta aula e também na Aula 3, quando o JavaScript consumir nossa API.

---

## 2. Do banco de dados para objeto

### 2.1 Relembrando `sqlite3.Row`

No SQLite, configuramos:

```python
conexao.row_factory = sqlite3.Row
```

Isso faz com que cada linha retornada pelo banco permita acesso por nome de coluna:

```python
row["id"]
row["descricao"]
row["concluida"]
```

Isso é mais legível do que acesso por posição:

```python
row[0]
row[1]
row[2]
```

---

### 2.2 Função auxiliar para converter linha em objeto

Vamos criar uma função auxiliar para transformar uma linha do banco em um objeto Tarefa.

```python
def tarefa_from_row(row):
    return Tarefa(
        id=row["id"],
        descricao=row["descricao"],
        concluida=bool(row["concluida"])
    )
```

> **Definição rápida:** `bool()` converte um valor para verdadeiro ou falso. No SQLite, `concluida` foi definida como `INTEGER`, normalmente `0` ou `1`. `bool(0)` vira `False`, e `bool(1)` vira `True`.

Exemplo de uso:

```python
conexao = sqlite3.connect("tarefas.db")
conexao.row_factory = sqlite3.Row

row = conexao.execute(
    "SELECT * FROM tarefas WHERE id = ?",
    (1,)
).fetchone()

tarefa = tarefa_from_row(row)

print(tarefa.descricao)
print(tarefa.concluida)
print(tarefa.to_dict())
```

---

## 3. Repositório: concentrando o SQL

### 3.1 O problema do SQL espalhado

Se cada rota Flask ou cada função do sistema tiver seu próprio SQL, qualquer mudança se torna mais difícil.

Exemplo: se a tabela `tarefas` ganhar uma nova coluna, talvez seja necessário alterar vários trechos de código.

Uma solução simples é criar uma classe responsável apenas por acessar o banco.

Essa classe costuma ser chamada de **repositório** ou **DAO**.

> **Definição rápida:** um repositório é uma classe que concentra as operações de acesso a uma fonte de dados, como um banco de dados.

Neste curso, não precisamos tratar isso como arquitetura obrigatória. Vamos usar porque é prático e melhora a organização.

---

### 3.2 Classe `TarefaRepository`

```python
import sqlite3

class TarefaRepository:
    def __init__(self, conexao):
        self.conexao = conexao

    def inicializar(self):
        with self.conexao:
            self.conexao.execute("""
                CREATE TABLE IF NOT EXISTS tarefas (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    descricao TEXT NOT NULL,
                    concluida INTEGER DEFAULT 0
                )
            """)

    def inserir(self, tarefa):
        sql = """
            INSERT INTO tarefas (descricao, concluida)
            VALUES (?, ?)
        """
        with self.conexao:
            cursor = self.conexao.execute(
                sql,
                (tarefa.descricao, int(tarefa.concluida))
            )

        tarefa.id = cursor.lastrowid
        return tarefa

    def listar(self):
        sql = "SELECT * FROM tarefas ORDER BY id"
        rows = self.conexao.execute(sql).fetchall()
        return [tarefa_from_row(row) for row in rows]

    def buscar_por_id(self, id_tarefa):
        sql = "SELECT * FROM tarefas WHERE id = ?"
        row = self.conexao.execute(sql, (id_tarefa,)).fetchone()

        if row is None:
            return None

        return tarefa_from_row(row)

    def atualizar(self, tarefa):
        sql = """
            UPDATE tarefas
            SET descricao = ?, concluida = ?
            WHERE id = ?
        """
        with self.conexao:
            self.conexao.execute(
                sql,
                (tarefa.descricao, int(tarefa.concluida), tarefa.id)
            )

    def deletar(self, id_tarefa):
        sql = "DELETE FROM tarefas WHERE id = ?"
        with self.conexao:
            self.conexao.execute(sql, (id_tarefa,))
```

---

### 3.3 Entendendo o repositório

A classe `TarefaRepository` recebe uma conexão no construtor:

```python
def __init__(self, conexao):
    self.conexao = conexao
```

Isso é uma composição simples:

- o repositório usa uma conexão;
- ele não precisa criar a conexão necessariamente;
- ele apenas sabe executar operações sobre tarefas.

> **Definição rápida:** composição é quando um objeto usa outro objeto para realizar seu trabalho. Aqui, `TarefaRepository` usa uma conexão SQLite.

---

### 3.4 Por que usar `?` no SQL

O repositório usa queries parametrizadas:

```python
self.conexao.execute(sql, (tarefa.descricao, int(tarefa.concluida)))
```

Nunca devemos montar SQL com f-string:

```python
# ERRADO! Risco de SQL Injection
sql = f"INSERT INTO tarefas (descricao) VALUES ('{tarefa.descricao}')"
```

O correto é:

```python
sql = "INSERT INTO tarefas (descricao) VALUES (?)"
conexao.execute(sql, (tarefa.descricao,))
```

Isso já foi visto na aula de SQLite e continua essencial aqui.

---

## 4. Usando o repositório em um script simples

### 4.1 Criando a conexão

```python
import sqlite3

DB_NAME = "tarefas.db"

def get_conexao():
    conexao = sqlite3.connect(DB_NAME)
    conexao.row_factory = sqlite3.Row
    return conexao
```

---

### 4.2 Exemplo de uso

```python
if __name__ == "__main__":
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    repository.inicializar()

    nova_tarefa = Tarefa("Estudar Flask")

    if nova_tarefa.validar():
        repository.inserir(nova_tarefa)
        print("Tarefa inserida:", nova_tarefa.to_dict())
    else:
        print("Tarefa inválida.")

    for tarefa in repository.listar():
        print(tarefa.resumo())

    conexao.close()
```

Saída possível:

```text
Tarefa inserida: {'id': 1, 'descricao': 'Estudar Flask', 'concluida': False}
⬜ Estudar Flask
```

---

### 4.3 Concluindo uma tarefa

```python
tarefa = repository.buscar_por_id(1)

if tarefa is None:
    print("Tarefa não encontrada.")
else:
    tarefa.concluir()
    repository.atualizar(tarefa)
    print(tarefa.resumo())
```

Saída esperada:

```text
✅ Estudar Flask
```

---

### 4.4 O que ganhamos com isso

Antes:

```python
conexao.execute(
    "UPDATE tarefas SET concluida = 1 WHERE id = ?",
    (id_tarefa,)
)
```

Depois:

```python
tarefa = repository.buscar_por_id(1)
tarefa.concluir()
repository.atualizar(tarefa)
```

A diferença é que agora:

- a tarefa é um objeto;
- a ação `concluir()` pertence à tarefa;
- o SQL de atualização está no repositório;
- o código principal fica mais legível.

---

## 5. Integrando com Flask

Agora vamos transformar isso em uma API.

A ideia é que as rotas Flask fiquem finas:

- recebem a requisição;
- chamam o repositório ou serviço;
- devolvem JSON.

---

### 5.1 Estrutura simples

Para uma aula inicial, podemos manter tudo em um arquivo só:

```text
projeto/
├── app.py
└── tarefas.db
```

Mas, se o projeto crescer, podemos separar:

```text
projeto/
├── app.py
├── tarefa.py
├── tarefa_repository.py
└── tarefas.db
```

Por simplicidade, os exemplos abaixo podem ficar em um único arquivo.

---

### 5.2 Imports iniciais

```python
import sqlite3
from flask import Flask, request
```

---

### 5.3 Classe `Tarefa`

```python
class Tarefa:
    def __init__(self, descricao, id=None, concluida=False):
        self.id = id
        self.descricao = descricao
        self.concluida = concluida

    def validar(self):
        if not isinstance(self.descricao, str):
            return False

        if not self.descricao.strip():
            return False

        if len(self.descricao) > 100:
            return False

        return True

    def concluir(self):
        self.concluida = True

    def resumo(self):
        if self.concluida:
            return f"✅ {self.descricao}"
        return f"⬜ {self.descricao}"

    def to_dict(self):
        return {
            "id": self.id,
            "descricao": self.descricao,
            "concluida": self.concluida,
        }
```

---

### 5.4 Função auxiliar

```python
def tarefa_from_row(row):
    return Tarefa(
        id=row["id"],
        descricao=row["descricao"],
        concluida=bool(row["concluida"])
    )
```

---

### 5.5 Repositório

```python
class TarefaRepository:
    def __init__(self, conexao):
        self.conexao = conexao

    def inicializar(self):
        with self.conexao:
            self.conexao.execute("""
                CREATE TABLE IF NOT EXISTS tarefas (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    descricao TEXT NOT NULL,
                    concluida INTEGER DEFAULT 0
                )
            """)

    def inserir(self, tarefa):
        sql = """
            INSERT INTO tarefas (descricao, concluida)
            VALUES (?, ?)
        """
        with self.conexao:
            cursor = self.conexao.execute(
                sql,
                (tarefa.descricao, int(tarefa.concluida))
            )

        tarefa.id = cursor.lastrowid
        return tarefa

    def listar(self):
        sql = "SELECT * FROM tarefas ORDER BY id"
        rows = self.conexao.execute(sql).fetchall()
        return [tarefa_from_row(row) for row in rows]

    def buscar_por_id(self, id_tarefa):
        sql = "SELECT * FROM tarefas WHERE id = ?"
        row = self.conexao.execute(sql, (id_tarefa,)).fetchone()

        if row is None:
            return None

        return tarefa_from_row(row)

    def atualizar(self, tarefa):
        sql = """
            UPDATE tarefas
            SET descricao = ?, concluida = ?
            WHERE id = ?
        """
        with self.conexao:
            self.conexao.execute(
                sql,
                (tarefa.descricao, int(tarefa.concluida), tarefa.id)
            )

    def deletar(self, id_tarefa):
        sql = "DELETE FROM tarefas WHERE id = ?"
        with self.conexao:
            self.conexao.execute(sql, (id_tarefa,))
```

---

### 5.6 Conexão com o BD

```python
DB_NAME = "tarefas.db"

def get_conexao():
    conexao = sqlite3.connect(DB_NAME)
    conexao.row_factory = sqlite3.Row
    return conexao
```

---

### 5.7 Criando o Flask app e inicializando o banco

```python
app = Flask(__name__)

def inicializar_banco():
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        repository.inicializar()
    finally:
        conexao.close()

inicializar_banco()

if __name__ == "__main__":
    app.run(debug=True)
```

Aqui não existe um repositório global. A função `inicializar_banco()` cria um repositório temporário apenas para criar a tabela se ela não existir. Cada rota criará seu próprio repositório junto com sua própria conexão.

> **Nota:** aqui não usamos `app.app_context()` porque o código não depende de `current_app`, `g`, `url_for` nem de extensões que exijam contexto de aplicação.
>
> Também não usamos `g` neste primeiro momento porque queremos deixar o ciclo de vida da conexão visível e explícito em cada rota. Em projetos Flask maiores, é comum centralizar a conexão por request usando `g` e `teardown_appcontext`.

---

## 6. Rotas da API

### 6.1 Listar tarefas

```python
@app.route("/api/tarefas", methods=["GET"])
def api_listar_tarefas():
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        tarefas = repository.listar()
        return [tarefa.to_dict() for tarefa in tarefas]
    finally:
        conexao.close()
```

Teste no navegador:

```text
http://127.0.0.1:5000/api/tarefas
```

Resposta esperada:

```json
[]
```

Ou, se houver tarefas:

```json
[
  {
    "id": 1,
    "descricao": "Estudar Flask",
    "concluida": false
  }
]
```

---

### 6.2 Criar tarefa

```python
@app.route("/api/tarefas", methods=["POST"])
def api_criar_tarefa():
    dados = request.get_json(silent=True)

    if not isinstance(dados, dict):
        return {"erro": "Envie um JSON com o campo descricao."}, 400

    tarefa = Tarefa(descricao=dados.get("descricao", ""))

    if not tarefa.validar():
        return {"erro": "Descricao inválida."}, 400

    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        repository.inserir(tarefa)
        return tarefa.to_dict(), 201
    finally:
        conexao.close()
```

Exemplo de corpo enviado:

```json
{
  "descricao": "Estudar Flask"
}
```

Resposta esperada:

```json
{
  "id": 1,
  "descricao": "Estudar Flask",
  "concluida": false
}
```

Status HTTP:

```text
201 Created
```

---

### 6.3 Buscar tarefa por ID

```python
@app.route("/api/tarefas/<int:id_tarefa>", methods=["GET"])
def api_buscar_tarefa(id_tarefa):
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        tarefa = repository.buscar_por_id(id_tarefa)

        if tarefa is None:
            return {"erro": "Tarefa não encontrada."}, 404

        return tarefa.to_dict()
    finally:
        conexao.close()
```

Exemplo:

```text
GET /api/tarefas/1
```

Resposta:

```json
{
  "id": 1,
  "descricao": "Estudar Flask",
  "concluida": false
}
```

Se não existir:

```json
{
  "erro": "Tarefa não encontrada."
}
```

Status:

```text
404 Not Found
```

---

### 6.4 Concluir tarefa

```python
@app.route("/api/tarefas/<int:id_tarefa>/concluir", methods=["POST"])
def api_concluir_tarefa(id_tarefa):
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        tarefa = repository.buscar_por_id(id_tarefa)

        if tarefa is None:
            return {"erro": "Tarefa não encontrada."}, 404

        tarefa.concluir()
        repository.atualizar(tarefa)

        return tarefa.to_dict()
    finally:
        conexao.close()
```

Exemplo:

```text
POST /api/tarefas/1/concluir
```

Resposta:

```json
{
  "id": 1,
  "descricao": "Estudar Flask",
  "concluida": true
}
```

> **Nota sobre REST:** Em APIs REST mais rigorosas, a atualização parcial de um recurso (como apenas mudar o status para concluído) costuma usar o método HTTP `PATCH`. O uso de `POST` em um endpoint específico (`/concluir`) é uma abordagem comum e pragmática (chamada de *RPC-style*) que funciona perfeitamente bem e é mais simples de entender neste momento do curso.

---

### 6.5 Deletar tarefa

```python
@app.route("/api/tarefas/<int:id_tarefa>", methods=["DELETE"])
def api_deletar_tarefa(id_tarefa):
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        tarefa = repository.buscar_por_id(id_tarefa)

        if tarefa is None:
            return {"erro": "Tarefa não encontrada."}, 404

        repository.deletar(id_tarefa)

        return {"mensagem": "Tarefa removida."}
    finally:
        conexao.close()
```

Exemplo:

```text
DELETE /api/tarefas/1
```

---

## 7. Camada de serviço: aprofundamento opcional

Se você quiser organizar ainda mais o código, pode dar mais um passo: criar uma classe de serviço.

> **Definição rápida:** uma classe de serviço concentra regras de negócio. Ela usa o repositório para acessar o banco, mas não contém SQL diretamente.

---

### 7.1 Exemplo simples de serviço

```python
class TarefaService:
    def __init__(self, repository):
        self.repository = repository

    def criar_tarefa(self, descricao):
        tarefa = Tarefa(descricao=descricao)

        if not tarefa.validar():
            raise ValueError("Descrição inválida.")

        return self.repository.inserir(tarefa)

    def concluir_tarefa(self, id_tarefa):
        tarefa = self.repository.buscar_por_id(id_tarefa)

        if tarefa is None:
            raise LookupError("Tarefa não encontrada.")

        tarefa.concluir()
        self.repository.atualizar(tarefa)

        return tarefa
```

---

### 7.2 Usando o serviço em uma rota

```python
@app.route("/api/tarefas", methods=["POST"])
def api_criar_tarefa_com_service():
    dados = request.get_json(silent=True)

    if not isinstance(dados, dict):
        return {"erro": "Envie um JSON com o campo descricao."}, 400

    conexao = get_conexao()
    repository = TarefaRepository(conexao)
    service = TarefaService(repository)

    try:
        tarefa = service.criar_tarefa(dados.get("descricao", ""))
        return tarefa.to_dict(), 201
    except ValueError as e:
        return {"erro": str(e)}, 400
    finally:
        conexao.close()
```

---

### 7.3 Conclusão com serviço

```python
@app.route("/api/tarefas/<int:id_tarefa>/concluir", methods=["POST"])
def api_concluir_tarefa_com_service(id_tarefa):
    conexao = get_conexao()
    repository = TarefaRepository(conexao)
    service = TarefaService(repository)

    try:
        tarefa = service.concluir_tarefa(id_tarefa)
        return tarefa.to_dict()
    except LookupError:
        return {"erro": "Tarefa não encontrada."}, 404
    finally:
        conexao.close()
```

---

### 7.4 Quando usar serviço

Use serviço quando:

- houver regras de negócio mais claras;
- a mesma regra puder ser usada por várias rotas;
- você quiser separar ainda mais o código;
- a lógica estiver começando a crescer dentro das rotas.

Não use serviço se:

- o código for pequeno;
- a rota apenas chama o repositório sem regra adicional;
- a abstração deixar tudo mais difícil sem benefício claro.

---

## 8. Exceções personalizadas

Exceções também são objetos, e podemos criar tipos próprios de erro herdando de `Exception`:

```python
class TarefaNaoEncontrada(Exception):
    pass
```

Uso:

```python
raise TarefaNaoEncontrada("Tarefa não encontrada.")
```

Neste curso, usaremos principalmente `ValueError` e `LookupError` para manter as coisas simples. A ideia de exceções personalizadas fica como referência para quando você encontrar esse padrão em projetos maiores.

O importante aqui é perceber que herança aparece de forma natural e útil: `TarefaNaoEncontrada` **é uma** `Exception`, então pode ser capturada com `except Exception` ou de forma mais específica com `except TarefaNaoEncontrada`.

---

## 9. Erros comuns nesta etapa

### 9.1 Retornar objeto diretamente no Flask

Errado:

```python
return tarefa
```

Correto:

```python
return tarefa.to_dict()
```

---

### 9.2 Esquecer `row_factory`

Sem:

```python
conexao.row_factory = sqlite3.Row
```

não conseguimos fazer:

```python
row["descricao"]
```

---

### 9.3 Montar SQL com f-string

Errado:

```python
sql = f"SELECT * FROM tarefas WHERE id = {id_tarefa}"
```

Correto:

```python
sql = "SELECT * FROM tarefas WHERE id = ?"
row = conexao.execute(sql, (id_tarefa,)).fetchone()
```

---

### 9.4 Esquecer de converter `concluida`

No banco, usamos:

```sql
concluida INTEGER DEFAULT 0
```

No Python, é mais agradável usar:

```python
True
False
```

Por isso:

- ao ler do banco: `bool(row["concluida"])`;
- ao escrever no banco: `int(tarefa.concluida)`.

---

### 9.5 Usar classe onde função simples bastaria

Exemplo desnecessário:

```python
class CalculadoraTotal:
    def calcular(self, preco, quantidade):
        return preco * quantidade
```

Se não há estado nem regras associadas, uma função simples basta:

```python
def calcular_total(preco, quantidade):
    return preco * quantidade
```

---

## 10. Exercícios

### Exercício 1 — Classe Tarefa com ID

Crie uma classe chamada `Tarefa`.

Ela deve ter:

- `id`, opcional, iniciando como `None`;
- `descricao`;
- `concluida`, iniciando como `False`.

Métodos:

- `validar()`: retorna `False` se `descricao` não for string, estiver vazia ou tiver mais de 100 caracteres;
- `concluir()`: marca como concluída;
- `resumo()`: retorna `⬜ descricao` ou `✅ descricao`;
- `to_dict()`: retorna um dicionário com `id`, `descricao` e `concluida`.

Teste criando uma tarefa sem `id`.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class Tarefa:
    def __init__(self, descricao, id=None, concluida=False):
        self.id = id
        self.descricao = descricao
        self.concluida = concluida

    def validar(self):
        if not isinstance(self.descricao, str):
            return False

        if not self.descricao.strip():
            return False

        if len(self.descricao) > 100:
            return False

        return True

    def concluir(self):
        self.concluida = True

    def resumo(self):
        if self.concluida:
            return f"✅ {self.descricao}"
        return f"⬜ {self.descricao}"

    def to_dict(self):
        return {
            "id": self.id,
            "descricao": self.descricao,
            "concluida": self.concluida,
        }

tarefa = Tarefa("Estudar Flask")

print(tarefa.validar())
print(tarefa.resumo())
print(tarefa.to_dict())
```

</details>

---

### Exercício 2 — Converter linha em objeto

Crie uma função chamada `tarefa_from_row`.

Ela deve receber uma linha do SQLite e retornar um objeto `Tarefa`.

A linha terá os campos:

- `id`;
- `descricao`;
- `concluida`.

Lembre-se de converter `concluida` para booleano.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
def tarefa_from_row(row):
    return Tarefa(
        id=row["id"],
        descricao=row["descricao"],
        concluida=bool(row["concluida"])
    )
```

</details>

---

### Exercício 3 — Repositório básico

Crie uma classe chamada `TarefaRepository`.

Ela deve receber uma conexão no construtor.

Implemente os métodos:

- `inicializar()`: cria a tabela `tarefas` se ela não existir;
- `inserir(tarefa)`: insere uma tarefa no banco e atualiza o `id` dela;
- `listar()`: retorna uma lista de objetos `Tarefa`;
- `buscar_por_id(id_tarefa)`: retorna um objeto `Tarefa` ou `None`;
- `atualizar(tarefa)`: atualiza a descrição e o status da tarefa no banco.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class TarefaRepository:
    def __init__(self, conexao):
        self.conexao = conexao

    def inicializar(self):
        with self.conexao:
            self.conexao.execute("""
                CREATE TABLE IF NOT EXISTS tarefas (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    descricao TEXT NOT NULL,
                    concluida INTEGER DEFAULT 0
                )
            """)

    def inserir(self, tarefa):
        sql = """
            INSERT INTO tarefas (descricao, concluida)
            VALUES (?, ?)
        """
        with self.conexao:
            cursor = self.conexao.execute(
                sql,
                (tarefa.descricao, int(tarefa.concluida))
            )

        tarefa.id = cursor.lastrowid
        return tarefa

    def listar(self):
        sql = "SELECT * FROM tarefas ORDER BY id"
        rows = self.conexao.execute(sql).fetchall()
        return [tarefa_from_row(row) for row in rows]

    def buscar_por_id(self, id_tarefa):
        sql = "SELECT * FROM tarefas WHERE id = ?"
        row = self.conexao.execute(sql, (id_tarefa,)).fetchone()

        if row is None:
            return None

        return tarefa_from_row(row)

    def atualizar(self, tarefa):
        sql = """
            UPDATE tarefas
            SET descricao = ?, concluida = ?
            WHERE id = ?
        """
        with self.conexao:
            self.conexao.execute(
                sql,
                (tarefa.descricao, int(tarefa.concluida), tarefa.id)
            )
```

</details>

---

### Exercício 4 — Script usando repositório

Crie um script que:

1. abre conexão com `tarefas.db`;
2. cria um `TarefaRepository`;
3. inicializa o banco;
4. cadastra duas tarefas válidas;
5. lista todas as tarefas usando `resumo()`;
6. conclui a primeira tarefa;
7. lista novamente;
8. fecha a conexão.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
import sqlite3

DB_NAME = "tarefas.db"

def get_conexao():
    conexao = sqlite3.connect(DB_NAME)
    conexao.row_factory = sqlite3.Row
    return conexao

conexao = get_conexao()
repository = TarefaRepository(conexao)
repository.inicializar()

t1 = Tarefa("Estudar Flask")
t2 = Tarefa("Fazer exercícios")

if t1.validar():
    repository.inserir(t1)

if t2.validar():
    repository.inserir(t2)

print("--- Antes de concluir ---")
for tarefa in repository.listar():
    print(tarefa.resumo())

tarefas = repository.listar()

if tarefas:
    primeira = tarefas[0]
    primeira.concluir()
    repository.atualizar(primeira)

    # Em um caso real, se já conhecemos o id da tarefa,
    # repository.buscar_por_id(id) seria mais direto do que
    # listar todas as tarefas apenas para pegar a primeira.

print("--- Depois de concluir ---")
for tarefa in repository.listar():
    print(tarefa.resumo())

conexao.close()
```

</details>

---

### Exercício 5 — API GET de tarefas

Crie uma rota Flask:

```text
GET /api/tarefas
```

Ela deve:

1. abrir conexão com o banco;
2. criar um `TarefaRepository`;
3. listar as tarefas;
4. retornar JSON com a lista de tarefas;
5. fechar a conexão.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
@app.route("/api/tarefas", methods=["GET"])
def api_listar_tarefas():
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        tarefas = repository.listar()
        return [tarefa.to_dict() for tarefa in tarefas]
    finally:
        conexao.close()
```

</details>

---

### Exercício 6 — API POST de tarefas

Crie uma rota Flask:

```text
POST /api/tarefas
```

Ela deve:

1. receber JSON;
2. validar se o corpo é um dicionário;
3. criar uma `Tarefa` com o campo `descricao`;
4. chamar `validar()`;
5. retornar erro `400` se inválida;
6. inserir no banco;
7. retornar a tarefa criada com status `201`.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
@app.route("/api/tarefas", methods=["POST"])
def api_criar_tarefa():
    dados = request.get_json(silent=True)

    if not isinstance(dados, dict):
        return {"erro": "Envie um JSON com o campo descricao."}, 400

    tarefa = Tarefa(descricao=dados.get("descricao", ""))

    if not tarefa.validar():
        return {"erro": "Descricao inválida."}, 400

    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        repository.inserir(tarefa)
        return tarefa.to_dict(), 201
    finally:
        conexao.close()
```

</details>

---

### Exercício 7 — API para concluir tarefa

Crie uma rota Flask:

```text
POST /api/tarefas/<int:id_tarefa>/concluir
```

Ela deve:

1. buscar a tarefa pelo ID;
2. retornar `404` se não existir;
3. chamar `tarefa.concluir()`;
4. atualizar no banco;
5. retornar a tarefa atualizada em JSON.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
@app.route("/api/tarefas/<int:id_tarefa>/concluir", methods=["POST"])
def api_concluir_tarefa(id_tarefa):
    conexao = get_conexao()
    repository = TarefaRepository(conexao)

    try:
        tarefa = repository.buscar_por_id(id_tarefa)

        if tarefa is None:
            return {"erro": "Tarefa não encontrada."}, 404

        tarefa.concluir()
        repository.atualizar(tarefa)

        return tarefa.to_dict()
    finally:
        conexao.close()
```

</details>

---

### Exercício 8 — Desafio: LivroRepository

Usando como base a tabela de livros vista na aula de SQLite:

```sql
CREATE TABLE IF NOT EXISTS livros (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    titulo TEXT NOT NULL,
    autor TEXT NOT NULL,
    ano_publicacao INTEGER
);
```

Crie:

1. uma classe `Livro`, com:
   - `id`;
   - `titulo`;
   - `autor`;
   - `ano_publicacao`;
   - `validar()`;
   - `to_dict()`;

2. uma função `livro_from_row(row)`;

3. uma classe `LivroRepository` com:
   - `inicializar()`;
   - `inserir(livro)`;
   - `listar()`;
   - `buscar_por_id(id)`;
   - `deletar(id)`.

Não é necessário criar Flask para este desafio, apenas o script Python.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```python
class Livro:
    def __init__(self, titulo, autor, ano_publicacao, id=None):
        self.id = id
        self.titulo = titulo
        self.autor = autor
        self.ano_publicacao = ano_publicacao

    def validar(self):
        if not isinstance(self.titulo, str) or not self.titulo.strip():
            return False

        if not isinstance(self.autor, str) or not self.autor.strip():
            return False

        if not isinstance(self.ano_publicacao, int):
            return False

        if self.ano_publicacao < 0:
            return False

        return True

    def to_dict(self):
        return {
            "id": self.id,
            "titulo": self.titulo,
            "autor": self.autor,
            "ano_publicacao": self.ano_publicacao,
        }


def livro_from_row(row):
    return Livro(
        titulo=row["titulo"],
        autor=row["autor"],
        ano_publicacao=row["ano_publicacao"],
        id=row["id"]
    )


class LivroRepository:
    def __init__(self, conexao):
        self.conexao = conexao

    def inicializar(self):
        with self.conexao:
            self.conexao.execute("""
                CREATE TABLE IF NOT EXISTS livros (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    titulo TEXT NOT NULL,
                    autor TEXT NOT NULL,
                    ano_publicacao INTEGER
                )
            """)

    def inserir(self, livro):
        sql = """
            INSERT INTO livros (titulo, autor, ano_publicacao)
            VALUES (?, ?, ?)
        """
        with self.conexao:
            cursor = self.conexao.execute(
                sql,
                (livro.titulo, livro.autor, livro.ano_publicacao)
            )

        livro.id = cursor.lastrowid
        return livro

    def listar(self):
        sql = "SELECT * FROM livros ORDER BY id"
        rows = self.conexao.execute(sql).fetchall()
        return [livro_from_row(row) for row in rows]

    def buscar_por_id(self, id_livro):
        sql = "SELECT * FROM livros WHERE id = ?"
        row = self.conexao.execute(sql, (id_livro,)).fetchone()

        if row is None:
            return None

        return livro_from_row(row)

    def deletar(self, id_livro):
        sql = "DELETE FROM livros WHERE id = ?"
        with self.conexao:
            self.conexao.execute(sql, (id_livro,))
```

</details>

---

## 11. Checklist da aula

Ao final desta aula, você deve conseguir:

- explicar por que OOP pode ajudar a organizar backend;
- criar uma classe de domínio simples, como `Tarefa`;
- converter uma linha do SQLite em objeto;
- converter um objeto em dicionário com `to_dict()`;
- concentrar SQL em uma classe repositório;
- usar queries parametrizadas com `?`;
- usar repositório dentro de rotas Flask;
- retornar objetos como JSON usando `to_dict()`;
- entender quando usar repositório, serviço ou apenas função;
- evitar transformar tudo em classe sem necessidade.

---

## 12. Recapitulação

| Conceito | Significado nesta aula |
|---|---|
| Classe de domínio | Representa uma entidade do sistema, como `Tarefa` |
| Objeto | Instância concreta da classe, com dados próprios |
| Repositório | Classe que concentra o acesso ao banco de dados |
| Serviço | Classe opcional que concentra regras de negócio |
| `to_dict()` | Converte objeto em dicionário para JSON |
| `tarefa_from_row()` | Converte linha do SQLite em objeto |
| `sqlite3.Row` | Permite acessar colunas pelo nome |
| Query parametrizada | Uso de `?` para evitar SQL Injection |
| Rota fina | Rota Flask que delega lógica para classes/funções auxiliares |
| Composição | Objeto que usa outro objeto, como repositório usando conexão |

---

## 13. O que não vimos no curso

Os tópicos abaixo são relevantes em Python e aparecem com frequência em projetos maiores, mas ficaram fora do escopo deste curso curto. Eles não são pré-requisitos para as próximas aulas.

| Tópico | Síntese | Documentação oficial |
|---|---|---|
| `@classmethod` | Método que recebe a própria classe como primeiro parâmetro e é comum em construtores alternativos, como criar um objeto a partir de uma linha do banco. | [Python Docs — `classmethod`](https://docs.python.org/3/library/functions.html#classmethod) |
| `@staticmethod` | Método ligado à classe apenas por organização, sem receber automaticamente a instância nem a classe. | [Python Docs — `staticmethod`](https://docs.python.org/3/library/functions.html#staticmethod) |
| `@property` | Permite expor um método como se fosse um atributo, útil para validações ou valores calculados. | [Python Docs — `property`](https://docs.python.org/3/library/functions.html#property) |
| `dataclasses` | Cria classes focadas em dados com menos repetição de código, gerando automaticamente métodos como `__init__` e `__repr__`. | [Python Docs — `dataclasses`](https://docs.python.org/3/library/dataclasses.html) |
| Herança simples | Permite que uma classe reaproveite ou estenda comportamento de outra, devendo ser usada com cuidado em projetos pequenos. | [Python Docs — Inheritance](https://docs.python.org/3/tutorial/classes.html#inheritance) |
| Herança múltipla | Permite que uma classe herde de várias classes ao mesmo tempo, mas costuma aumentar bastante a complexidade. | [Python Docs — Multiple Inheritance](https://docs.python.org/3/tutorial/classes.html#multiple-inheritance) |
| `super()` | Chama métodos da classe mãe para reaproveitar ou estender comportamento herdado. | [Python Docs — `super`](https://docs.python.org/3/library/functions.html#super) |
| Exceções personalizadas | Permite criar tipos próprios de erro, normalmente herdando de `Exception`, para representar falhas específicas do domínio. | [Python Docs — User-defined Exceptions](https://docs.python.org/3/tutorial/errors.html#user-defined-exceptions) |
| Métodos especiais / dunder | Métodos como `__init__`, `__repr__` e `__eq__` integram objetos a recursos internos do Python. | [Python Docs — Special Method Names](https://docs.python.org/3/reference/datamodel.html#special-method-names) |
| Classes abstratas — ABC | Definem contratos formais que obrigam subclasses a implementar certos métodos, sendo mais úteis em bibliotecas e projetos grandes. | [Python Docs — `abc`](https://docs.python.org/3/library/abc.html) |
| Atributos privados por convenção | Nomes iniciados com `_` indicam uso interno do objeto, embora Python não bloqueie o acesso externo. | [Python Docs — Private Variables](https://docs.python.org/3/tutorial/classes.html#private-variables) |
| `__slots__` | Restringe os atributos que as instâncias podem ter, economizando memória em troca de menos flexibilidade. | [Python Docs — `__slots__`](https://docs.python.org/3/reference/datamodel.html#slots) |
| Duck typing | Python geralmente se importa mais com os métodos que o objeto possui do que com o nome formal da sua classe. | [Python Docs — Duck Typing](https://docs.python.org/3/glossary.html#term-duck-typing) |
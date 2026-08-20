# Aula 3 — OOP aplicada ao JavaScript client-side e integração com APIs

**Pré-requisitos:** Aula 1 e 2 de OOP, JavaScript básico (DOM, eventos, `fetch`), Flask básico (Aula 2).

**Objetivo da aula:** entender como o JavaScript lida com objetos no navegador, aprender a sintaxe de classes em JS, resolver o problema do `this` em eventos e construir um frontend estruturado que consome a API Flask criada na aula anterior.

> **Nota de vocabulário:** nesta aula, usaremos OOP no JavaScript principalmente para resolver dois problemas práticos:
>
> 1. evitar que o código JS vire uma "sopa" de variáveis globais e funções soltas;
> 2. encapsular componentes de interface (um pedaço do HTML + sua lógica) e clientes de API.

---

## 0. Revisão e motivação

### 0.1 O problema do JavaScript "espaguete"

Quando começamos a programar em JavaScript para o navegador, é muito comum escrevermos código assim:

```javascript
let contador = 0;
const btn = document.getElementById('btn');
const display = document.getElementById('display');

function incrementar() {
    contador++;
    display.textContent = contador;
}

btn.addEventListener('click', incrementar);
```

Isso funciona perfeitamente para um botão. Mas imagine uma página com um formulário de login, uma lista de tarefas, um carrinho de compras e um gráfico. Você terá dezenas de variáveis globais (`let contador`, `let tarefas`, `let usuario`) e funções soltas (`function buscar()`, `function renderizar()`, `function validar()`).

O código se torna difícil de manter, pois não fica claro quais variáveis e funções pertencem a qual parte da tela.

### 0.2 A solução: Encapsulamento

Assim como no Python, podemos usar OOP para agrupar o **estado** (os elementos do DOM, os dados) e o **comportamento** (os eventos, as chamadas de API) em unidades coesas.

Em vez de funções soltas, teremos um objeto `TarefaApp` que sabe como buscar tarefas, renderizá-las e ouvir o clique do usuário.

---

## 1. Você já usa objetos em JS sem perceber

Assim como no Python, o JavaScript é profundamente orientado a objetos no seu uso diário.

### 1.1 O DOM é feito de objetos

Quando você faz:

```javascript
const btn = document.getElementById('meu-btn');
btn.addEventListener('click', handler);
btn.classList.add('ativo');
```

- `btn` é um objeto (uma instância de `HTMLElement`).
- `classList` é um objeto dentro de `btn`, com seus próprios métodos (`add`, `remove`, `toggle`).
- `addEventListener` é um método do objeto `btn`.

📚 **Referência:** [MDN - Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)

### 1.2 Fetch e Promises

Quando consumimos uma API:

```javascript
fetch('/api/tarefas')
    .then(resp => resp.json())
    .then(dados => console.log(dados));
```

- `fetch` retorna um objeto do tipo `Promise`.
- `resp` é um objeto do tipo `Response`.
- `resp.json()` retorna um objeto JavaScript (o JSON parseado).

📚 **Referência:** [MDN - Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

### 1.3 JSON vs Objeto Literal

No JavaScript, a sintaxe para criar um objeto solto (sem classe) é quase idêntica ao JSON:

```javascript
const tarefa = {
    id: 1,
    descricao: "Estudar JS",
    concluida: false
};
```

Isso é um **objeto literal**. Ele é excelente para transportar dados (como o JSON que vem do Flask), mas não possui métodos próprios (comportamento).

📚 **Referência:** [MDN - Working with Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects)

---

## 2. Sintaxe de Classe em JavaScript

O JavaScript moderno (ES6+) introduziu a palavra-chave `class`, que torna a sintaxe muito parecida com a do Python, mas com algumas diferenças importantes.

### 2.1 Estrutura básica

Vamos recriar a classe `Tarefa` que fizemos em Python, agora em JavaScript.

```javascript
class Tarefa {
    constructor(id, descricao, concluida = false) {
        this.id = id;
        this.descricao = descricao;
        this.concluida = concluida;
    }

    resumo() {
        return this.concluida ? `✅ ${this.descricao}` : `⬜ ${this.descricao}`;
    }

    toDict() {
        return {
            id: this.id,
            descricao: this.descricao,
            concluida: this.concluida
        };
    }
}
```

📚 **Referência:** [MDN - Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

### 2.2 Comparação Rápida: Python vs JavaScript

| Conceito | Python | JavaScript |
|---|---|---|
| Definição | `class Tarefa:` | `class Tarefa { ... }` |
| Construtor | `def __init__(self, ...):` | `constructor(...) { ... }` |
| Referência ao objeto | `self` | `this` |
| Declaração de métodos | `def metodo(self):` | `metodo() { ... }` (sem `def` e sem `this` nos parâmetros) |
| Instanciação | `t = Tarefa(...)` | `const t = new Tarefa(...)` (exige `new`) |

### 2.3 Instanciando e usando

```javascript
const t1 = new Tarefa(1, "Estudar JS");

console.log(t1.resumo()); // ⬜ Estudar JS

t1.concluida = true;

console.log(t1.resumo()); // ✅ Estudar JS
```

---

## 3. O maior "gotcha" do JS: O problema do `this`

Em Python, o `self` é sempre a instância do objeto, não importa como o método seja chamado.
Em JavaScript, o valor de `this` é **dinâmico**: ele depende de *como* a função foi chamada, não de *onde* foi definida.

Isso causa o erro mais comum para iniciantes em OOP com JS: perder a referência do `this` ao lidar com eventos do DOM.

### 3.1 O cenário do erro

```javascript
class Contador {
    constructor() {
        this.valor = 0;
        this.btn = document.getElementById('btn-contar');
        
        // ⚠️ CUIDADO: Isso vai quebrar!
        this.btn.addEventListener('click', this.incrementar);
    }

    incrementar() {
        // O 'this' aqui não será o objeto Contador, mas sim o botão HTML!
        this.valor++; 
        console.log(this.valor);
    }
}
```

Quando o usuário clica no botão, o navegador chama a função `incrementar`. Mas ele a chama como se fosse um método do botão, fazendo o `this` apontar para o elemento HTML, e não para a classe `Contador`. O resultado é `NaN` ou um erro.

### 3.2 A solução pragmática: Arrow Functions

As **Arrow Functions** (`() => {}`) têm uma característica especial: elas não possuem seu próprio `this`. Elas "herdam" o `this` do contexto onde foram criadas.

A regra de ouro para eventos em classes JS é: **use arrow functions no `addEventListener`**.

```javascript
class Contador {
    constructor() {
        this.valor = 0;
        this.btn = document.getElementById('btn-contar');
        
        // ✅ CORRETO: A arrow function preserva o 'this' da classe
        this.btn.addEventListener('click', () => this.incrementar());
    }

    incrementar() {
        this.valor++;
        console.log(this.valor);
    }
}
```

📚 **Referência:** [MDN - Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

---

## 4. Encapsulando um Componente de Interface

Vamos criar uma classe que gerencia um pedaço inteiro da nossa página. Isso evita poluir o escopo global com variáveis de DOM.

```javascript
class ContadorVisual {
    constructor(containerId) {
        // Estado
        this.valor = 0;
        
        // Referências ao DOM (encapsuladas)
        this.container = document.getElementById(containerId);
        this.display = this.container.querySelector('.display');
        this.btnAdd = this.container.querySelector('.btn-add');
        this.btnReset = this.container.querySelector('.btn-reset');
        
        // Eventos
        this.btnAdd.addEventListener('click', () => this.incrementar());
        this.btnReset.addEventListener('click', () => this.resetar());
        
        this.renderizar();
    }

    incrementar() {
        this.valor++;
        this.renderizar();
    }

    resetar() {
        this.valor = 0;
        this.renderizar();
    }

    renderizar() {
        this.display.textContent = this.valor;
    }
}

// Inicialização limpa
const meuContador = new ContadorVisual('painel-contador');
```

---

## 5. Criando um Cliente de API

Na Aula 2, criamos uma API Flask para gerenciar tarefas. Vamos criar uma classe em JS cuja única responsabilidade é saber como conversar com essa API.

*Nota: Usaremos `async/await` em vez de `.then()`, pois deixa o código assíncrono com uma leitura muito próxima do código síncrono do Python.*

```javascript
class TarefaAPI {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
    }

    async listar() {
        const resp = await fetch(`${this.baseUrl}/api/tarefas`);
        if (!resp.ok) throw new Error("Erro ao buscar tarefas");
        return await resp.json(); // Retorna um array de objetos literais
    }

    async criar(descricao) {
        const resp = await fetch(`${this.baseUrl}/api/tarefas`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ descricao })
        });
        if (!resp.ok) throw new Error("Erro ao criar tarefa");
        return await resp.json();
    }

    async concluir(id) {
        const resp = await fetch(`${this.baseUrl}/api/tarefas/${id}/concluir`, {
            method: 'POST'
        });
        if (!resp.ok) throw new Error("Erro ao concluir tarefa");
        return await resp.json();
    }
}
```

📚 **Referência:** [MDN - async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

---

## 6. Juntando tudo: O Frontend Full-Stack

Agora vamos criar a classe principal da nossa aplicação, que usa o `TarefaAPI` para buscar dados e manipula o DOM para exibi-los.

### 6.1 O HTML base

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Gerenciador de Tarefas</title>
</head>
<body>
    <h1>Minhas Tarefas</h1>
    
    <form id="form-tarefa">
        <input type="text" id="input-descricao" placeholder="Nova tarefa..." required>
        <button type="submit">Adicionar</button>
    </form>

    <ul id="lista-tarefas"></ul>

    <script src="app.js"></script>
</body>
</html>
```

### 6.2 O JavaScript (`app.js`)

```javascript
class TarefaApp {
    constructor() {
        // Instancia o cliente de API (apontando para o Flask rodando em localhost)
        this.api = new TarefaAPI('http://127.0.0.1:5000');
        
        // Referências ao DOM
        this.form = document.getElementById('form-tarefa');
        this.input = document.getElementById('input-descricao');
        this.lista = document.getElementById('lista-tarefas');
        
        // Eventos
        this.form.addEventListener('submit', (e) => this.handleSubmit(e));
        
        // Carga inicial
        this.carregarTarefas();
    }

    async carregarTarefas() {
        try {
            const tarefas = await this.api.listar();
            this.renderizarLista(tarefas);
        } catch (erro) {
            console.error("Falha ao carregar tarefas:", erro);
            this.lista.innerHTML = "<li>Erro ao conectar com o servidor.</li>";
        }
    }

    async handleSubmit(e) {
        e.preventDefault(); // Impede o recarregamento da página
        const descricao = this.input.value.trim();
        
        if (!descricao) return;

        try {
            await this.api.criar(descricao);
            this.input.value = ''; // Limpa o input
            this.carregarTarefas(); // Recarrega a lista
        } catch (erro) {
            alert("Erro ao criar tarefa.");
        }
    }

    renderizarLista(tarefas) {
        this.lista.innerHTML = ''; // Limpa a lista atual
        
        if (tarefas.length === 0) {
            this.lista.innerHTML = "<li>Nenhuma tarefa cadastrada.</li>";
            return;
        }

        for (const t of tarefas) {
            const li = document.createElement('li');
            
            // Exibe o texto com base no estado
            const texto = t.concluida ? `✅ ${t.descricao}` : `⬜ ${t.descricao}`;
            li.textContent = texto;
            
            // Se não estiver concluída, adiciona botão de concluir
            if (!t.concluida) {
                const btn = document.createElement('button');
                btn.textContent = 'Concluir';
                btn.style.marginLeft = '10px';
                
                // Arrow function para manter o 'this' e capturar o 't.id'
                btn.addEventListener('click', async () => {
                    await this.api.concluir(t.id);
                    this.carregarTarefas();
                });
                
                li.appendChild(btn);
            }
            
            this.lista.appendChild(li);
        }
    }
}

// Ponto de entrada: Inicia o app quando o DOM estiver pronto
document.addEventListener('DOMContentLoaded', () => {
    new TarefaApp();
});
```

---

## 7. JSON vs Instância de Classe no Frontend

Note um detalhe crucial no método `renderizarLista(tarefas)`:

O método `this.api.listar()` retorna o JSON parseado. Isso significa que `tarefas` é um array de **objetos literais**, e não instâncias da classe `Tarefa` que criamos na Aula 1 ou na Seção 2.

```javascript
// O que vem do fetch:
const t = { id: 1, descricao: "Estudar", concluida: false };

// Isso NÃO funciona, pois 't' é um objeto literal, não tem métodos:
// t.resumo() -> TypeError: t.resumo is not a function
```

Para um frontend simples, acessar as propriedades diretamente (`t.descricao`, `t.concluida`) é perfeitamente aceitável e prático.

Se você fizesse questão de ter os métodos da classe disponíveis no frontend, precisaria mapear os dados:

```javascript
const tarefasObjetos = tarefas.map(t => new Tarefa(t.id, t.descricao, t.concluida));
```

Mas, no desenvolvimento Web moderno, é muito comum tratar o JSON da API como "dados puros" (objetos literais) e deixar as classes do frontend focadas apenas em **Componentes de UI** (como `TarefaApp` e `TarefaAPI`).

---

## 8. Erros comuns nesta etapa

### 8.1 Esquecer a palavra `new`

Errado:
```javascript
const app = TarefaApp(); // TypeError: Cannot call a class as a function
```

Correto:
```javascript
const app = new TarefaApp();
```

### 8.2 Perder o `this` em callbacks assíncronos

Se você usar funções tradicionais dentro de métodos de classe, o `this` pode se perder.

Errado:
```javascript
async carregar() {
    fetch('/api').then(function(resp) {
        this.renderizar(); // 'this' é undefined ou window
    });
}
```

Correto (usando arrow function):
```javascript
async carregar() {
    fetch('/api').then((resp) => {
        this.renderizar(); // 'this' é a instância da classe
    });
}
```

### 8.3 Tentar modificar o JSON diretamente como se fosse o Banco

No frontend, alterar uma propriedade do objeto literal não altera o banco de dados. Você precisa chamar a API.

Errado:
```javascript
tarefa.concluida = true; // Altera apenas a variável na memória do navegador
```

Correto:
```javascript
await this.api.concluir(tarefa.id); // Envia POST para o Flask, que altera o SQLite
```

---

## 9. Exercícios

### Exercício 1 — Classe Cronômetro

Crie uma classe `Cronometro` que encapsule um cronômetro simples.

Ela deve ter:
- Um método `iniciar()` que use `setInterval` para incrementar os segundos a cada 1000ms.
- Um método `parar()` que limpe o intervalo.
- Um método `resetar()` que zere o tempo e pare o intervalo.
- Um método `renderizar()` que atualize um elemento HTML com o tempo formatado (`MM:SS`).

O construtor deve receber o ID do container e já configurar os botões de Iniciar, Parar e Reset usando arrow functions nos eventos.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```javascript
class Cronometro {
    constructor(containerId) {
        this.segundos = 0;
        this.intervalId = null;
        
        this.container = document.getElementById(containerId);
        this.display = this.container.querySelector('.display');
        this.btnStart = this.container.querySelector('.btn-start');
        this.btnStop = this.container.querySelector('.btn-stop');
        this.btnReset = this.container.querySelector('.btn-reset');
        
        this.btnStart.addEventListener('click', () => this.iniciar());
        this.btnStop.addEventListener('click', () => this.parar());
        this.btnReset.addEventListener('click', () => this.resetar());
        
        this.renderizar();
    }
    
    iniciar() {
        if (this.intervalId) return; // Evita múltiplos intervalos
        this.intervalId = setInterval(() => {
            this.segundos++;
            this.renderizar();
        }, 1000);
    }
    
    parar() {
        clearInterval(this.intervalId);
        this.intervalId = null;
    }
    
    resetar() {
        this.parar();
        this.segundos = 0;
        this.renderizar();
    }
    
    renderizar() {
        const min = Math.floor(this.segundos / 60).toString().padStart(2, '0');
        const sec = (this.segundos % 60).toString().padStart(2, '0');
        this.display.textContent = `${min}:${sec}`;
    }
}
```

</details>

---

### Exercício 2 — Adicionando Deletar na API

Na Aula 2, criamos a rota `DELETE /api/tarefas/<id>`. 

Adicione o método `deletar(id)` na classe `TarefaAPI` e, em seguida, adicione um botão "Deletar" ao lado de cada tarefa no método `renderizarLista` da classe `TarefaApp`.

<details>
<summary><strong>Ver solução resumida</strong></summary>

```javascript
// Na classe TarefaAPI:
async deletar(id) {
    const resp = await fetch(`${this.baseUrl}/api/tarefas/${id}`, {
        method: 'DELETE'
    });
    if (!resp.ok) throw new Error("Erro ao deletar tarefa");
    return await resp.json();
}

// Na classe TarefaApp, dentro do loop de renderizarLista:
const btnDel = document.createElement('button');
btnDel.textContent = 'Deletar';
btnDel.style.marginLeft = '5px';
btnDel.style.color = 'red';

btnDel.addEventListener('click', async () => {
    await this.api.deletar(t.id);
    this.carregarTarefas();
});

li.appendChild(btnDel);
```

</details>

---

## 10. Checklist da aula

Ao final desta aula, você deve conseguir:

- reconhecer objetos no DOM e em respostas de API;
- diferenciar um objeto literal (JSON) de uma instância de classe;
- criar uma classe simples em JavaScript usando `constructor` e `this`;
- instanciar objetos usando a palavra-chave `new`;
- entender por que o `this` se perde em eventos e como resolver isso com Arrow Functions;
- encapsular um componente de interface (DOM + Estado + Eventos) em uma classe;
- criar uma classe "Cliente de API" para isolar as chamadas `fetch`;
- integrar um frontend estruturado com um backend Flask que retorna JSON.

---

## 11. Recapitulação

| Conceito | Significado no JS Client-Side |
|---|---|
| Objeto Literal | Estrutura de dados chave-valor, equivalente ao JSON e ao `dict` do Python. |
| Classe (`class`) | Molde para criar objetos com estado e comportamento encapsulados. |
| `constructor` | Método especial executado automaticamente ao usar `new`. |
| `this` | Referência ao objeto atual (cuidado: é dinâmico em funções tradicionais). |
| Arrow Function | Função anônima (`() => {}`) que preserva o `this` do contexto onde foi criada. Essencial para eventos. |
| Componente | Classe que agrupa referências do DOM, estado da interface e ouvintes de eventos. |
| Cliente de API | Classe dedicada a fazer chamadas `fetch` e tratar as respostas do servidor. |
| `async/await` | Sintaxe moderna para lidar com Promises, fazendo código assíncrono parecer síncrono. |

---

## 12. O que não vimos no curso

Assim como nas aulas de Python, o JavaScript possui mecanismos de OOP muito profundos e complexos que ficam fora do escopo deste curso curto, pois não são necessários para construir aplicações Web full-stack funcionais e bem organizadas no dia a dia.

| Tópico | Síntese | Documentação oficial |
|---|---|---|
| **Prototype Chain** | O JS não tem classes no nível do motor (como Java/C++). A sintaxe `class` é apenas um "açúcar sintático" por cima de um sistema de herança baseado em protótipos. Entender a cadeia de protótipos é fascinante, mas não é necessário para usar `class` no dia a dia. | [MDN - Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain) |
| **Herança (`extends`)** | Criar classes que herdam de outras classes (ex: `class Admin extends User`). No frontend moderno, prefere-se **Composição** (um objeto que usa outro objeto) em vez de herança profunda, pois gera código mais flexível e menos acoplado. | [MDN - Extending classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends) |
| **`Object.create` e `Object.assign`** | Métodos de baixo nível para manipulação e clonagem de objetos e protótipos. | [MDN - Object.create](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create), [MDN - Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) |
| **Módulos ES6 (`import`/`export`)** | Como dividir o código JS em múltiplos arquivos e importá-los. Em projetos reais, isso é feito com bundlers (como Vite ou Webpack), que fogem do escopo de um curso introdutório de backend/full-stack. | [MDN - JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) |
| **Private Fields (`#`)** | Sintaxe moderna para criar atributos verdadeiramente privados em JS (ex: `#saldo`). No JS, a convenção de usar `_` (como no Python) ou simplesmente confiar no encapsulamento do escopo ainda é muito forte. | [MDN - Private class features](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields) |
| **Symbols e Iterators** | Mecanismos avançados para criar propriedades únicas e tornar objetos iteráveis (usáveis em `for...of`). | [MDN - Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol), [MDN - Iterators and generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_generators) |

---

Links essenciais para consulta durante e após o curso:

### JavaScript Core
- [MDN - JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference) - A bíblia do JavaScript
- [MDN - Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) - Sintaxe completa de classes
- [MDN - Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) - Entendendo o `this` léxico
- [MDN - async/await](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Promises) - Programação assíncrona moderna

### DOM e Web APIs
- [MDN - Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) - Manipulação do DOM
- [MDN - EventTarget.addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) - Ouvintes de eventos
- [MDN - Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) - Requisições HTTP modernas
- [MDN - Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) - Guia prático de fetch

### JSON e Dados
- [MDN - Working with Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects) - Objetos literais e manipulação
- [MDN - JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON) - Serialização e parsing
- [JSON specification](https://www.json.org/json-en.html) - Especificação oficial do formato JSON
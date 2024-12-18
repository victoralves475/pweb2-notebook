# Guia Completo sobre o Thymeleaf com o Spring Framework

## Introdução

O **Thymeleaf** é um motor de templates (template engine) para Java que facilita a criação de páginas HTML dinâmicas. Ele se integra perfeitamente ao **Spring Framework**, especialmente em aplicações construídas com **Spring MVC**. Isso significa que você pode usar Thymeleaf para gerar a camada de apresentação de sua aplicação web de forma simples, flexível e bem organizada.

Comparado a outras tecnologias de visualização, como JSP, o Thymeleaf oferece:

- **HTML válido**: Seus templates são arquivos HTML comuns, que podem ser visualizados no navegador mesmo sem o servidor estar em execução.
- **Integração nativa com Spring MVC**: Acesso direto aos dados enviados pelo controlador do Spring.
- **Sintaxe Simples e Intuitiva**: Menos verbosidade e mais clareza no código.
- **Recursos avançados**: Layouts, fragmentos reutilizáveis e integração com formulários.

Este guia mostrará, do básico ao avançado, como usar o Thymeleaf junto ao Spring para criar páginas web dinâmicas.

---

## O que é o Thymeleaf?

O Thymeleaf é um mecanismo de template que processa arquivos HTML, inserindo dados dinâmicos fornecidos pela aplicação. Por exemplo:

- Você tem um template HTML com marcadores especiais.
- O Spring MVC executa o controlador, obtém dados do banco de dados ou serviços, e repassa esses dados para o Thymeleaf.
- O Thymeleaf substitui os marcadores no HTML pelos dados reais, gerando a página final enviada ao navegador.

Isso resulta em páginas bonitas, dinâmicas e fáceis de manter, sem precisar escrever códigos de script complicados.

---

## Integração com o Spring MVC

O fluxo típico em uma aplicação Spring MVC com Thymeleaf é:

1. **Usuário acessa uma URL** no navegador (ex: `/clientes`).
2. **Controller (Spring MVC)** recebe a requisição, processa a lógica de negócio e obtém dados (por exemplo, lista de clientes).
3. O Controller adiciona esses dados ao **modelo** e retorna o nome de um **template Thymeleaf**.
4. O **Thymeleaf** processa o template, inserindo os dados do modelo nos locais adequados no HTML.
5. O resultado é uma página HTML completa, enviada ao **navegador** do usuário.

---

## Configuração Básica

### Dependências

Se você estiver usando Spring Boot, basta adicionar no `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

O Spring Boot automaticamente detecta o Thymeleaf e o configura para procurar templates em `src/main/resources/templates`.

### Estrutura de Arquivos

Padrão do Spring Boot:

- `src/main/resources/templates`: Aqui ficam os templates HTML do Thymeleaf.
- `src/main/resources/static`: Aqui ficam arquivos estáticos (CSS, JS, imagens).

Por exemplo:

```
src/
 ├─ main/
 │   ├─ java/...
 │   └─ resources/
 │       ├─ templates/
 │       │   └─ index.html
 │       └─ static/
 │           ├─ css/
 │           └─ js/
```

---

## Sintaxe do Thymeleaf

O Thymeleaf utiliza atributos HTML especiais (prefixados com `th:`) e expressões para inserir dados, controlar a exibição de elementos e muito mais.

### Expressões de Variáveis

- `${variavel}`: Acessa uma variável do modelo fornecida pelo controlador.  
  Exemplo: Se o model possui `model.addAttribute("nome", "João")`, no template podemos usar `[[${nome}]]` para exibir "João".

### Atributos Thymeleaf Comuns

- **`th:text`**: Substitui o conteúdo de um elemento pelo valor de uma expressão.
  ```html
  <p th:text="${nome}">Nome do usuário</p>
  ```
  Se `${nome}` é "João", o parágrafo exibirá "João".

- **`th:if` e `th:unless`**: Exibição condicional.
  ```html
  <p th:if="${usuarioLogado}">Bem-vindo!</p>
  <p th:unless="${usuarioLogado}">Por favor, faça login.</p>
  ```

- **`th:each`**: Iteração sobre listas.
  ```html
  <ul>
    <li th:each="cliente : ${clientes}" th:text="${cliente.nome}"></li>
  </ul>
  ```
  Cada cliente da lista será exibido em um `<li>`.

- **`th:href`, `th:src`**: Definem atributos de link e imagem dinamicamente.
  ```html
  <a th:href="@{/clientes}">Lista de clientes</a>
  ```

### Links e Paths

`@{...}` é usado para criar URLs baseadas nas rotas mapeadas:

```html
<a th:href="@{/clientes/{id}(id=${cliente.id})}">Detalhes do Cliente</a>
```

Se `cliente.id` for 10, a URL final será `/clientes/10`.

### Formulários

Com o Thymeleaf, você pode facilmente vincular objetos do modelo a campos de formulário:

```html
<form th:object="${cliente}" method="post" action="/clientes/salvar">
    <input type="text" th:field="*{nome}" placeholder="Nome" />
    <button type="submit">Salvar</button>
</form>
```

No controlador:

```java
@GetMapping("/clientes/novo")
public String novoCliente(Model model) {
    model.addAttribute("cliente", new Cliente());
    return "clienteForm";
}

@PostMapping("/clientes/salvar")
public String salvarCliente(@ModelAttribute Cliente cliente) {
    // Lógica de salvamento
    return "redirect:/clientes";
}
```

Os campos `th:field="*{nome}"` ligam o input ao atributo `nome` do objeto `cliente`.

---

## Fragmentos (Reuso de Código)

Os **fragmentos** permitem criar partes do template reutilizáveis (como cabeçalhos, rodapés, menus).

### Definir um Fragmento

```html
<!-- fragments.html -->
<header th:fragment="cabecalho">
    <h1>Minha Aplicação</h1>
</header>
```

### Incluir o Fragmento

```html
<!-- index.html -->
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <div th:replace="fragments :: cabecalho"></div>
    <p>Conteúdo da página principal</p>
</body>
</html>
```

`th:replace` substitui o elemento atual pelo fragmento. Assim, o cabeçalho definido em `fragments.html` é incluído em `index.html`.

---

## Layouts com Thymeleaf Layout Dialect

Para gerenciar layouts complexos (como um template base e páginas filhas), use o **Thymeleaf Layout Dialect**.

### Dependência

```xml
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

### Exemplo de Layout Base

```html
<!-- layout.html -->
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
    <title>Aplicação</title>
</head>
<body>
    <header>Header padrão</header>
    <div layout:fragment="conteudo">Conteúdo padrão</div>
    <footer>Rodapé padrão</footer>
</body>
</html>
```

### Página que Herda do Layout

```html
<!-- pagina.html -->
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="layout.html">
<head>
    <title layout:fragment="title">Minha Página</title>
</head>
<body>
    <div layout:fragment="conteudo">Conteúdo específico da página</div>
</body>
</html>
```

Assim, `pagina.html` usa o layout definido em `layout.html`, herdando cabeçalho, rodapé e apenas substituindo o conteúdo central.

---

## Comentários e Blocos

- **Comentários Thymeleaf**:  
  ```html
  <!--/* Este comentário será removido no processamento do Thymeleaf */-->
  ```

- **`th:block`**:  
  Agrupa lógica sem gerar tags extras. Útil para criar estruturas lógicas sem alterar a árvore HTML.
  ```html
  <th:block th:if="${mostrarMensagem}">
    <p>Mensagem visível apenas se mostrarMensagem=true</p>
  </th:block>
  ```

---

## Acesso a Recursos Estáticos

Arquivos CSS, JS e imagens devem ficar em `src/main/resources/static`. Para referenciá-los, use `@{...}`:

```html
<link rel="stylesheet" th:href="@{/css/style.css}" />
<script th:src="@{/js/script.js}"></script>
<img th:src="@{/images/logo.png}" />
```

---

## Exemplo Completo

### Controlador

```java
@Controller
public class ClienteController {

    @GetMapping("/clientes")
    public String listarClientes(Model model) {
        List<Cliente> clientes = buscarDoBanco(); // busca clientes
        model.addAttribute("clientes", clientes);
        return "clientes/lista"; // templates/clientes/lista.html
    }
}
```

### Template (clientes/lista.html)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Lista de Clientes</title>
</head>
<body>
    <h1>Clientes</h1>
    <ul>
        <li th:each="cliente : ${clientes}" th:text="${cliente.nome}"></li>
    </ul>
</body>
</html>
```

Quando `/clientes` for acessado, o controlador envia uma lista de clientes para o template, que então exibe cada nome em um `<li>`.

---

## Benefícios do Thymeleaf

- **Produtividade**: A sintaxe simples e clara torna o desenvolvimento mais rápido.
- **Pré-visualização**: Você pode abrir os arquivos HTML no navegador antes mesmo do servidor rodar, enxergando o layout básico.
- **Integração Perfeita com Spring**: Acesso direto a dados do modelo, suporte a formulários, URLs dinâmicas.
- **Flexibilidade**: Fragmentos, layouts, condicionais e iterações facilitam criar páginas complexas sem bagunça.

---

## Conclusão

O Thymeleaf é uma ferramenta poderosa que torna a criação de interfaces web dinâmicas mais simples e organizada. Ao integrá-lo ao Spring MVC, você pode desenvolver aplicações completas, com camadas bem definidas, separando a lógica de negócio da apresentação. Com o Thymeleaf, você ganha:

- **Facilidade**: Menos código boilerplate, mais foco no que importa.
- **Manutenibilidade**: Layouts e fragmentos reduzem duplicação de código.
- **Eficiência**: Renderização de páginas rápidas, HTML válido, experiência fluída.

Aprender e dominar o Thymeleaf, junto com Spring MVC, é um passo importante para construir aplicações web robustas, escaláveis e fáceis de manter.

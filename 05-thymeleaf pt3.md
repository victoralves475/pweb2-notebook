# Guia Completo sobre o Thymeleaf com o Spring Framework

## Introdução

O **Thymeleaf** é um motor de templates para Java que facilita a geração de páginas HTML dinâmicas. Ele se integra perfeitamente ao **Spring Framework**, especialmente ao Spring MVC, tornando-se uma solução ideal para construir camadas de visualização de aplicações web. Neste guia, apresentaremos em detalhes como o Thymeleaf funciona, como configurá-lo e como utilizá-lo para criar aplicações ricas e fáceis de manter.

## O que é o Thymeleaf?

O Thymeleaf é um template engine capaz de processar arquivos HTML e gerar páginas dinâmicas. Suas principais características incluem:

- **HTML válido**: Você pode abrir o template diretamente no navegador e visualizar sua estrutura antes mesmo de executar a aplicação.
- **Integração com Spring MVC**: O Thymeleaf obtém dados do modelo disponibilizado pelo controlador e os insere no HTML.
- **Substituição do JSP**: Comparado ao JSP, o Thymeleaf oferece melhor desempenho, maior flexibilidade e uma curva de aprendizado mais suave.

### Como o Thymeleaf Processa Templates

1. O navegador faz uma requisição a um endpoint do controlador.
2. O controlador do Spring MVC processa a lógica de negócios, obtém dados e adiciona-os ao modelo (Model).
3. Ao retornar o nome de um template para o Spring, o Thymeleaf é acionado.
4. O Thymeleaf mescla os dados do modelo com o template HTML, gerando uma página completamente renderizada em HTML.
5. Essa página HTML final é enviada ao navegador do cliente.

## Configuração Básica

### Dependência Maven

Para habilitar o Thymeleaf em uma aplicação Spring Boot, basta adicionar a dependência:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

O Spring Boot detecta essa dependência e configura automaticamente o Thymeleaf.

### Estrutura de Pastas

- **Templates**: Por padrão, os templates Thymeleaf devem ficar em `src/main/resources/templates`.
- **Arquivos Estáticos**: CSS, JavaScript e imagens podem ser colocados em `src/main/resources/static`.

Estrutura típica:

```
src/
 ├─ main/
 │   ├─ java/
 │   │   └─ com/meuprojeto/...
 │   └─ resources/
 │       ├─ templates/
 │       │   └─ index.html
 │       └─ static/
 │           ├─ css/
 │           └─ js/
```

## Sintaxe e Conceitos do Thymeleaf

O Thymeleaf utiliza o padrão de expressão OGNL (Object-Graph Navigation Language) para acessar dados do modelo e outras funcionalidades.

### Expressões Comuns

- **Variáveis do Modelo**: `${variavel}`  
  Exemplo: `th:text="${nome}"` insere o valor da variável `nome` no elemento HTML.

- **Links Dinâmicos**: `@{url}`  
  Exemplo: `th:href="@{/clientes}"` cria um link para a rota `/clientes`.

- **Fragmentos**: `~{template :: fragment}`  
  Permite incluir partes de outros templates.

- **Propriedades de Objetos**: Se `cliente` for um objeto, `${cliente.nome}` acessa o atributo `nome`.

### Operações Suportadas

- **Textuais**: `'Olá ' + ${nome}`
- **Aritméticas**: `+`, `-`, `*`, `/`, `%`
- **Booleanas**: `and`, `or`, `not`
- **Comparações**: `<`, `>`, `==`, `!=`
- **Condicionais (ternário)**: `condicao ? 'Verdadeiro' : 'Falso'`

## Fragmentos e Reutilização de Código

Os fragmentos permitem dividir o layout em partes menores e reutilizáveis, facilitando a manutenção e padronização da interface.

### Definição de Fragmento

```html
<!-- fragments.html -->
<header th:fragment="header">
    <h1>Meu Site</h1>
</header>
```

### Inclusão de Fragmento

```html
<!-- index.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <div th:replace="fragments :: header"></div>
    <p>Conteúdo da página principal</p>
</body>
</html>
```

- **`th:insert`**: Insere o fragmento dentro da tag.
- **`th:replace`**: Substitui a tag hospedeira pelo fragmento.

### Fragmentos Parametrizados

Podemos passar parâmetros para fragmentos:

```html
<!-- fragments.html -->
<footer th:fragment="footer (ano)">&copy; [[${ano}]]</footer>
```

Inclusão do fragmento com parâmetro:

```html
<div th:replace="fragments :: footer(${#calendars.year(#calendars.createNow())})"></div>
```

## Layouts com o Thymeleaf Layout Dialect

O **Thymeleaf Layout Dialect** é uma extensão que ajuda a criar layouts usando um padrão de herança (semelhante a um decorator pattern).

### Dependência Maven

```xml
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

### Exemplo de Layout

```html
<!-- layout.html -->
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Título Base</title>
</head>
<body>
    <header th:fragment="header">Cabeçalho Padrão</header>
    <div th:fragment="body">Conteúdo Padrão</div>
</body>
</html>
```

### Página que Extende o Layout

```html
<!-- pagina.html -->
<html xmlns:th="http://www.thymeleaf.org" 
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="layout.html">
<head>
    <title layout:fragment="title">Minha Página</title>
</head>
<body>
    <div layout:fragment="body">Conteúdo Específico da Minha Página</div>
</body>
</html>
```

Aqui, `pagina.html` aproveita o layout definido em `layout.html`, substituindo apenas partes específicas.

## Manipulação de Formulários

O Thymeleaf facilita o binding entre objetos Java e campos de formulários HTML.

### Exemplo de Formulário

```html
<form th:object="${usuario}" method="post" action="/usuarios">
    <input type="text" th:field="*{nome}" placeholder="Nome" />
    <input type="email" th:field="*{email}" placeholder="Email" />
    <button type="submit">Enviar</button>
</form>
```

No controlador:

```java
@GetMapping("/usuarios/novo")
public String novoUsuario(Model model) {
    model.addAttribute("usuario", new Usuario());
    return "formulario";
}

@PostMapping("/usuarios")
public String salvarUsuario(@ModelAttribute Usuario usuario) {
    // Salvar o usuário
    return "redirect:/usuarios";
}
```

## Iterações e Condicionais

### Iteração (`th:each`)

Para iterar sobre coleções:

```html
<ul>
    <li th:each="item : ${itens}" th:text="${item}"></li>
</ul>
```

### Condicionais (`th:if`, `th:unless`, `th:switch`)

```html
<p th:if="${usuarioLogado}">Bem-vindo, usuário!</p>
<p th:unless="${usuarioLogado}">Faça login para continuar.</p>

<div th:switch="${tipoUsuario}">
    <span th:case="'ADMIN'">Admin</span>
    <span th:case="'USER'">Usuário Comum</span>
    <span th:case="*">Desconhecido</span>
</div>
```

## Comentários e Blocos

### Comentários Processados pelo Thymeleaf

```html
<!--/* Este comentário será ignorado pelo navegador, mas processado pelo Thymeleaf /*/-->
```

### `th:block`

O `th:block` é um elemento virtual do Thymeleaf que não gera nenhuma tag HTML, mas pode ser usado para agrupar lógica:

```html
<th:block th:each="item : ${itens}">
    <p th:text="${item.nome}"></p>
    <p th:text="${item.idade}"></p>
</th:block>
```

## Conclusão

O Thymeleaf integra-se de forma nativa com o Spring, fornecendo uma solução eficiente e elegante para criar interfaces web dinâmicas. Suas funcionalidades, como fragmentos, layout dialect, iterações e condicionais, permitem desenvolver páginas modulares, escaláveis e de fácil manutenção. Ao dominar o Thymeleaf, você estará apto a construir interfaces ricas, mantendo seu código de apresentação limpo e organizado, aproveitando ao máximo a produtividade do Spring Framework.

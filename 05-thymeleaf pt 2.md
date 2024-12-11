# Guia Completo de Spring Boot e Thymeleaf

## Introdução

Ao combinar o **Spring Boot** com o **Thymeleaf**, os desenvolvedores obtêm uma plataforma poderosa e simplificada para criar aplicações web dinâmicas. O Spring Boot automatiza grande parte da configuração do projeto, enquanto o Thymeleaf oferece um modo elegante de gerar páginas HTML dinâmicas, integrando-se perfeitamente ao ecossistema Spring.

## O que é o Thymeleaf?

O **Thymeleaf** é um motor de templates (template engine) para Java, projetado para criar páginas dinâmicas a partir de documentos HTML válidos. Ao contrário de tecnologias anteriores, como JSP, o Thymeleaf permite que o template possa ser visualizado diretamente no navegador sem a necessidade de um servidor em execução, tornando o desenvolvimento e o design mais ágeis. Além disso:

- **HTML real**: Os templates Thymeleaf são arquivos HTML válidos.
- **Integração com Spring**: O Thymeleaf se integra perfeitamente com o Spring MVC, permitindo acesso fácil às variáveis do modelo.
- **Flexibilidade**: Oferece recursos avançados, como fragmentos (para reaproveitar partes comuns de layout), condicionais, iterações e formatação de dados.

## Integração com o Spring Boot

Um dos pontos fortes do Spring Boot é a sua configuração automática. Ao adicionar a dependência do Thymeleaf no projeto, o Spring Boot detecta e configura tudo automaticamente.

### Dependência no Maven

Basta incluir no `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Com isso, o Spring Boot:

- Cria automaticamente um `ThymeleafViewResolver`.
- Configura a pasta padrão dos templates em `src/main/resources/templates`.
- Permite acessar arquivos estáticos (CSS, JS, imagens) em `src/main/resources/static`.

Nenhuma configuração extra é necessária na maioria dos casos.

## Exemplo Prático

### Controller

Um controller do Spring Boot retorna o nome do template Thymeleaf a ser exibido e coloca dados no modelo:

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    public String sayHello(Model model) {
        model.addAttribute("name", "Usuário");
        return "hello"; // aponta para o arquivo hello.html em templates/
    }
}
```

### Template Thymeleaf (hello.html)

Crie um arquivo `hello.html` em `src/main/resources/templates`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Bem-vindo</title>
</head>
<body>
    <h1>Olá, <span th:text="${name}">nome padrão</span>!</h1>
</body>
</html>
```

Neste template:

- `th:text="${name}"` é substituído pelo valor da variável `name` do modelo (no caso, "Usuário").

Ao acessar `http://localhost:8080/hello`, o navegador exibirá: "Olá, Usuário!".

## Recursos do Thymeleaf

### Expressões e Sintaxe

- **Variáveis**: `${variavel}` para acessar dados do modelo.
- **Atributos Thymeleaf**: `th:text`, `th:href`, `th:src`, etc., para definir dinamicamente o valor desses atributos.
- **Links**: `@{...}` para gerar URLs relativas ao contexto da aplicação.

Exemplo de link dinâmico:

```html
<a th:href="@{/clientes}">Ver Clientes</a>
```

### Iterações

Itere sobre coleções usando `th:each`:

```html
<ul>
    <li th:each="item : ${itens}" th:text="${item}"></li>
</ul>
```

Isso criará um `<li>` para cada elemento da coleção `itens`.

### Condicionais

Exiba ou oculte elementos condicionalmente com `th:if` e `th:unless`:

```html
<p th:if="${usuarioLogado}">Bem-vindo, usuário!</p>
<p th:unless="${usuarioLogado}">Faça login para continuar.</p>
```

Use `th:switch` e `th:case` para múltiplos cenários, parecido com o `switch/case` do Java:

```html
<div th:switch="${tipoUsuario}">
    <p th:case="'ADMIN'">Acesso completo</p>
    <p th:case="'USER'">Acesso limitado</p>
    <p th:case="*">Acesso desconhecido</p>
</div>
```

### Objetos Utilitários

O Thymeleaf fornece objetos auxiliares, como:

- **#strings**: Manipulação de strings (como `#strings.contains(...)`)
- **#dates**: Formatação e manipulação de datas.
- **#numbers**: Formatação numérica.
  
Exemplo de formatação de data:

```html
<p>Data atual: [[${#dates.formatIso(#calendars.createNow())}]]</p>
```

## Fragmentos e Reutilização de Código

Os **fragmentos** permitem definir partes do template (cabeçalho, rodapé, menu) em um arquivo separado e incluí-los em outros templates.

### Definição de Fragmentos

```html
<!-- fragments.html -->
<div th:fragment="header">
    <header>
        <h1>Meu Site</h1>
    </header>
</div>
```

### Inclusão de Fragmentos

```html
<!-- main.html -->
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Página Principal</title>
</head>
<body>
    <div th:replace="fragments :: header"></div> <!-- Insere o fragmento "header" -->
    <div>Conteúdo da página principal</div>
</body>
</html>
```

Fragmentos também podem receber parâmetros:

```html
<!-- fragments.html -->
<div th:fragment="footer (ano)">Rodapé - [[${ano}]]</div>
```

E para incluí-lo:

```html
<div th:replace="fragments :: footer(${#calendars.year(#calendars.createNow())})"></div>
```

## Layouts com Thymeleaf

Para layouts mais sofisticados, pode-se usar o **Thymeleaf Layout Dialect**, que permite definir um arquivo de layout base e várias páginas que injetam o conteúdo nesse layout.

### Dependência

```xml
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
    <version>3.0.0</version>
</dependency>
```

### Definição do Layout

```html
<!-- layout.html -->
<html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
    <title layout:fragment="title">Título Padrão</title>
</head>
<body>
    <header layout:fragment="header">
        <h1>Cabecalho Padrão</h1>
    </header>
    <div layout:fragment="body">
        Conteúdo padrão
    </div>
</body>
</html>
```

### Página de Conteúdo

```html
<!-- pagina.html -->
<html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="layout.html">
<head>
    <title layout:fragment="title">Minha Página</title>
</head>
<body>
    <div layout:fragment="body">
        <p>Conteúdo específico da minha página</p>
    </div>
</body>
</html>
```

Nesse exemplo, `pagina.html` herda o layout de `layout.html`, substituindo apenas partes específicas.

## Conclusão

O **Spring Boot** e o **Thymeleaf** formam uma dupla ideal para criação de aplicações web Java, graças à configuração automática do Spring Boot e à flexibilidade do Thymeleaf. Juntos, eles permitem:

- **Desenvolvimento Ágil**: Menos configurações e mais foco na lógica de negócio.
- **HTML Válido**: Visualização direta dos templates no navegador.
- **Reutilização de Código**: Fragmentos, layouts e objetos utilitários reduzem duplicação e promovem manutenção fácil.
- **Integração Perfeita**: Acesso direto às variáveis do modelo, formatação de dados e controle fino da renderização.

Ao dominar esses conceitos, o desenvolvedor estará preparado para criar interfaces ricas, responsivas e de fácil manutenção, aproveitando todo o potencial oferecido pela plataforma Spring Boot e pelo Thymeleaf.

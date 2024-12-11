# Guia Completo do Thymeleaf

## O que é o Thymeleaf?

O **Thymeleaf** é um **motor de templates (template engine)** para aplicações Java, amplamente utilizado na camada de apresentação de projetos web. Ele permite a criação de páginas HTML dinâmicas, integrando-se perfeitamente com o Spring MVC, embora possa ser usado também em contextos não-web. Considerado uma evolução em relação ao JSP, o Thymeleaf oferece um desenvolvimento mais ágil e flexível.

**Principais Características**:
- **HTML real**: Seus templates são arquivos HTML válidos, permitindo que a página seja visualizada diretamente no navegador mesmo sem a aplicação estar em execução (modo estático).
- **Integração com Spring MVC**: A integração é simples e o modelo de dados retornado pelo controlador pode ser facilmente exibido na página.
- **Fragmentos Reutilizáveis**: É fácil criar cabeçalhos, rodapés e outros componentes reutilizáveis, mantendo consistência e reduzindo código duplicado.

## Como o Thymeleaf Funciona?

Em uma aplicação web típica utilizando Spring MVC e Thymeleaf, o fluxo de requisição funciona assim:

1. **Navegador → Controller**: O usuário acessa uma URL, o controlador do Spring recebe a requisição e executa a lógica de negócios.
2. **Controller → Model + View**: O controller retorna o nome de um template Thymeleaf e um conjunto de dados (Model) que serão utilizados na renderização.
3. **Thymeleaf → HTML Dinâmico**: O Thymeleaf processa o template, insere os dados do Model nas expressões definidas no HTML e gera uma página completa.
4. **Resposta ao Navegador**: A página HTML totalmente montada é enviada ao cliente, pronta para ser exibida.

## Configuração Básica

### Dependência no Maven

Certifique-se de adicionar a dependência do Thymeleaf no `pom.xml` do projeto:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Se estiver usando o Spring Boot, isso geralmente basta para habilitar o Thymeleaf.

### Pastas e Templates

Por padrão, o Spring Boot procura templates Thymeleaf dentro de `src/main/resources/templates`. Os arquivos de template geralmente têm extensão `.html`.

Exemplo de estrutura:

```
src/
└── main/
    ├── java/
    │   └── com/
    │       └── exemplo/
    │           ├── controller/
    │           └── Application.java
    └── resources/
        ├── templates/
        │   └── index.html
        └── static/
            ├── css/
            └── js/
```

## Sintaxe Básica

O Thymeleaf utiliza expressões e atributos específicos para inserir dados dinâmicos e controlar a renderização do HTML.

### Expressões Simples

- **Exibir valores do modelo**: Utilize `${}` para acessar propriedades do objeto passado pelo controller.

```html
<p>Olá, [[${nome}]]!</p>
```

No controller, se retornarmos um `Model` com `model.addAttribute("nome", "Maria")`, a página exibirá "Olá, Maria!".

### Acessando Variáveis e Propriedades

- **Variáveis**: `${variavel}` obtém o valor de uma variável colocada no Model.
- **Propriedades de Objetos**: `${objeto.propriedade}` para acessar atributos de um objeto. Ex: `${cliente.nome}`.
- **Links**: `@{...}` para criar URLs relativas ao contexto da aplicação. Ex: `th:href="@{/clientes}"`.

### Fragmentos

É possível definir partes reutilizáveis de código (cabeçalhos, rodapés) como fragmentos:

**Definição de Fragmento (`th:fragment`)**:

```html
<!-- fragments.html -->
<div th:fragment="header">
    <header>Meu Site</header>
</div>
```

**Inclusão de Fragmento (`th:insert` ou `th:replace`)**:

```html
<!-- index.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments :: header"></head>
<body>
    <h1>Página Inicial</h1>
</body>
</html>
```

- `th:insert` insere o conteúdo do fragmento dentro da tag.
- `th:replace` substitui a tag atual pelo fragmento.

### Laços e Condicionais

#### Iterações com `th:each`

Para exibir uma lista de itens:

```html
<ul>
    <li th:each="produto : ${produtos}">
        [[${produto.nome}]] - [[${produto.preco}]]
    </li>
</ul>
```

#### Condicionais com `th:if` e `th:unless`

Para exibir elementos apenas se uma condição for verdadeira:

```html
<p th:if="${usuarioLogado}">Bem-vindo!</p>
<p th:unless="${usuarioLogado}">Faça login para continuar.</p>
```

#### Condicionais com `th:switch` e `th:case`

Funciona de forma semelhante ao `switch/case` do Java:

```html
<div th:switch="${tipoUsuario}">
    <p th:case="'ADMIN'">Acesso total</p>
    <p th:case="'USER'">Acesso limitado</p>
    <p th:case="*">Tipo de usuário desconhecido</p>
</div>
```

### Atributos e Metadados

- **`th:text`**: Substitui o conteúdo de um elemento pelo valor da expressão.
- **`th:utext`**: Semelhante ao `th:text`, mas não escapa caracteres HTML.
- **`th:attr`**: Define atributos dinamicamente.
- **`th:value`, `th:href`, `th:src`**: Específicos para definir valor de inputs, links, imagens etc.

Exemplo definindo um link dinâmico:

```html
<a th:href="@{/clientes}">Lista de Clientes</a>
```

### Manipulação de Strings, Datas e Funções

O Thymeleaf oferece suporte a diversas operações no template:

- **Concatenação de Strings**: `'Olá ' + ${nome}`
- **Formatação de Datas**: Utilizar objetos ou funções auxiliares para formatar datas.
- **Funções Utilitárias**: É possível registrar funções personalizadas ou utilizar funções do Spring Expression Language (SpEL).

## Recursos Estáticos (CSS, JS, Imagens)

Arquivos estáticos (CSS, JS, imagens) ficam na pasta `src/main/resources/static`. Para referenciá-los, use `@{}`:

```html
<link rel="stylesheet" th:href="@{/css/style.css}" />
<script th:src="@{/js/script.js}"></script>
<img th:src="@{/images/logo.png}" />
```

## Parâmetros em URLs

É possível passar parâmetros em URLs:

```html
<a th:href="@{/clientes/{id}(id=${cliente.id})}">Detalhes</a>
```

Neste exemplo, se `cliente.id` for `10`, o link resultará em `/clientes/10`.

## Exemplos Práticos

### Exemplo de Controller

```java
@Controller
public class ClienteController {

    @GetMapping("/clientes")
    public String listarClientes(Model model) {
        List<Cliente> clientes = ...; // Buscar do banco
        model.addAttribute("clientes", clientes);
        return "clientes/lista"; // Retorna o template "lista.html" em templates/clientes
    }
}
```

### Exemplo de Template (lista.html)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Lista de Clientes</title>
</head>
<body>
    <h1>Clientes</h1>
    <ul>
        <li th:each="cliente : ${clientes}">
            [[${cliente.nome}]] - [[${cliente.email}]]
        </li>
    </ul>
</body>
</html>
```

## Benefícios do Thymeleaf

- **Produtividade**: Syntax simples e intuitiva.
- **Pré-visualização Estática**: O template HTML pode ser visualizado no navegador sem o servidor rodando, auxiliando na criação do layout.
- **Integração com Spring**: Suporte nativo às funcionalidades do Spring MVC, tornando o desenvolvimento fluido.
- **Modularidade e Reutilização**: Fragmentos e layouts facilitam a manutenção, evitando repetição de código.

## Conclusão

O Thymeleaf é um motor de templates robusto e flexível, ideal para criar aplicações web modernas em Java. Ele oferece uma abordagem não-intrusiva e produtiva para gerar páginas dinâmicas, simplificando a integração com o Spring MVC e ajudando a manter o código mais limpo e organizado. Com recursos como fragmentos, condicionais, iterações e fácil acesso a dados do servidor, o Thymeleaf tornou-se a escolha preferencial de muitos desenvolvedores que buscam manter sua camada de apresentação clara, escalável e bem estruturada.

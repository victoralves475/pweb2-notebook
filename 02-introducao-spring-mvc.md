# Guia Completo sobre Spring MVC

## Introdução ao Spring MVC

O **Spring MVC** (Model-View-Controller) é um módulo do Spring Framework que fornece uma infraestrutura poderosa e flexível para construir aplicações web baseadas no padrão arquitetural MVC. Ele integra os princípios de **Inversão de Controle (IoC)** e **Injeção de Dependência (DI)** do Spring, permitindo que os desenvolvedores criem aplicações modulares, testáveis e de fácil manutenção.

O padrão MVC separa a aplicação em três componentes principais:

1. **Model (Modelo)**:
   - **Responsabilidade**: Representa os dados e a lógica de negócios.
   - **Função**: Lida com o gerenciamento de dados, regras de negócio e interação com a camada de persistência (como bancos de dados).
   - **Exemplo**: Classes de entidade, serviços e repositórios.

2. **View (Visão)**:
   - **Responsabilidade**: Apresentar os dados ao usuário final.
   - **Função**: Renderiza a interface do usuário, exibindo informações e coletando entradas do usuário.
   - **Exemplo**: Páginas HTML, templates Thymeleaf, JSPs.

3. **Controller (Controlador)**:
   - **Responsabilidade**: Intermediar as interações entre a View e o Model.
   - **Função**: Processa as requisições HTTP, invoca a lógica de negócios apropriada e determina a View a ser renderizada.
   - **Exemplo**: Classes anotadas com `@Controller` ou `@RestController`.

## Fluxo de Processamento no Spring MVC

O Spring MVC segue um fluxo de processamento bem definido para lidar com requisições web:

1. **Recebimento da Requisição**:
   - O cliente (navegador ou aplicativo) envia uma requisição HTTP ao servidor.

2. **DispatcherServlet (Front Controller)**:
   - Todas as requisições passam pelo `DispatcherServlet`, que atua como um **Front Controller**.
   - Ele é responsável por interceptar as requisições e delegar o processamento ao componente adequado.

3. **Handler Mapping**:
   - O `DispatcherServlet` consulta os `Handler Mappings` para determinar qual Controller deve lidar com a requisição com base na URL e no método HTTP.

4. **Controller**:
   - O Controller correspondente é invocado.
   - Ele processa a lógica de negócios, interage com o Model (serviços e repositórios) e prepara os dados para a View.

5. **View Resolver**:
   - O Controller retorna o nome da View a ser renderizada.
   - O `View Resolver` localiza o template da View correspondente.

6. **Renderização da View**:
   - A View é renderizada com os dados fornecidos pelo Controller.
   - A resposta HTTP é enviada de volta ao cliente.

**Fluxo Simplificado**:

```plaintext
Cliente -> DispatcherServlet -> Controller -> Model -> View Resolver -> View -> Cliente
```

Essa arquitetura modulariza a aplicação, separando claramente as responsabilidades e facilitando a manutenção e escalabilidade.

## Benefícios do Spring MVC

- **Integração com o Spring Framework**:
  - Aproveita os recursos de IoC e DI, promovendo um código mais limpo e desacoplado.

- **Configuração Flexível**:
  - Suporta configurações via XML ou anotações, permitindo que os desenvolvedores escolham a abordagem que melhor se adapte ao projeto.

- **Suporte a Várias Tecnologias de View**:
  - Compatível com **Thymeleaf**, **JSP**, **FreeMarker**, **Velocity**, entre outros.

- **Processamento de Formulários**:
  - Facilita a criação e validação de formulários web, incluindo data binding e validação com Bean Validation.

- **Internacionalização (i18n)**:
  - Oferece suporte integrado para internacionalização, permitindo que aplicações sejam adaptadas para diferentes idiomas e culturas.

- **Tratamento de Exceções**:
  - Fornece mecanismos para manipulação de erros e exceções de forma centralizada.

## Estrutura de uma Aplicação Spring MVC

Uma aplicação típica com Spring MVC é estruturada em camadas e pastas organizadas, facilitando a navegação e manutenção do código.

### Estrutura de Pastas

```plaintext
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── exemplo/
│   │           ├── controller/
│   │           ├── model/
│   │           ├── service/
│   │           ├── repository/
│   │           └── Application.java
│   └── resources/
│       ├── templates/      // Views (Thymeleaf templates)
│       ├── static/         // Recursos estáticos (CSS, JS, imagens)
│       └── application.properties
```

### Componentes Principais

- **Controllers e Services**:
  - **Controllers**: Classes anotadas com `@Controller` que lidam com as requisições HTTP.
  - **Services**: Contêm a lógica de negócios, anotadas com `@Service`.

- **Models**:
  - Classes que representam as entidades do domínio da aplicação.

- **Repositories**:
  - Interfaces que estendem `JpaRepository` ou outras, responsáveis pela interação com o banco de dados.

- **Views (Templates)**:
  - Arquivos HTML ou JSP que definem a interface do usuário.

- **Configuração**:
  - Pode ser feita via anotações em classes Java (por exemplo, `@Configuration`, `@EnableWebMvc`) ou via arquivos XML.

## Criando Controllers e Views

### 1. Criar a Classe do Controller

Anote a classe com `@Controller` para indicar que ela será um componente responsável por lidar com requisições web.

```java
@Controller
@RequestMapping("/clientes")
public class ClienteController {

    @Autowired
    private ClienteService clienteService;

    // Métodos handler serão definidos aqui
}
```

### 2. Definir Métodos Handler

Os métodos dentro do Controller são chamados de handlers e são responsáveis por processar requisições específicas.

```java
@GetMapping
public String listarClientes(Model model) {
    List<Cliente> clientes = clienteService.listarTodos();
    model.addAttribute("clientes", clientes);
    return "clientes/lista";
}
```

- **Anotações de Mapeamento**:
  - `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`: Mapeiam métodos HTTP às URLs.

- **Parâmetros e Retorno**:
  - Utiliza o objeto `Model` para passar dados para a View.
  - Retorna uma `String` com o nome da View a ser renderizada.

### 3. Implementar a View

No caso de usar Thymeleaf, a View é um arquivo HTML localizado na pasta `src/main/resources/templates`.

**Exemplo de View (`clientes/lista.html`)**:

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
            <span th:text="${cliente.nome}">Nome do Cliente</span>
        </li>
    </ul>
</body>
</html>
```

### 4. Configurar o Template Engine

Certifique-se de que o Thymeleaf (ou outro mecanismo de template) esteja configurado no `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## Recebendo Dados de Formulários

### Exibir o Formulário

No Controller, crie um método para exibir o formulário de cadastro:

```java
@GetMapping("/novo")
public String exibirFormularioCadastro(Model model) {
    model.addAttribute("cliente", new Cliente());
    return "clientes/formulario";
}
```

### Processar o Formulário

Crie um método para processar os dados enviados pelo formulário:

```java
@PostMapping
public String salvarCliente(@ModelAttribute Cliente cliente) {
    clienteService.salvar(cliente);
    return "redirect:/clientes";
}
```

- **Anotação `@ModelAttribute`**:
  - Liga automaticamente os campos do formulário aos atributos do objeto `Cliente`.

### Exemplo de Formulário (`clientes/formulario.html`)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Cadastro de Cliente</title>
</head>
<body>
    <h1>Cadastrar Cliente</h1>
    <form th:action="@{/clientes}" th:object="${cliente}" method="post">
        <label for="nome">Nome:</label>
        <input type="text" id="nome" th:field="*{nome}" />
        <button type="submit">Salvar</button>
    </form>
</body>
</html>
```

## Model e Data Binding

### Model

O objeto `Model` é utilizado para passar dados do Controller para a View.

```java
model.addAttribute("atributo", valor);
```

### Data Binding

O Spring MVC facilita o data binding entre os campos do formulário e os objetos Java.

- **Requisitos**:
  - Os nomes dos campos `name` no HTML devem corresponder aos nomes dos atributos do objeto Java.
  - A classe deve possuir os métodos `getters` e `setters` para os atributos.

**Exemplo da Classe `Cliente`**:

```java
public class Cliente {
    private String nome;
    private String email;

    // Getters e Setters
}
```

## Validação de Dados

O Spring MVC integra-se com o Bean Validation para validar os dados de entrada.

### Anotações de Validação

- Utilize anotações como `@NotNull`, `@Size`, `@Email` nos atributos da classe.

```java
public class Cliente {

    @NotBlank(message = "O nome é obrigatório")
    private String nome;

    @Email(message = "Email inválido")
    private String email;

    // Getters e Setters
}
```

### Validando no Controller

Use `@Valid` e `BindingResult` para validar os dados e capturar erros.

```java
@PostMapping
public String salvarCliente(@Valid @ModelAttribute Cliente cliente, BindingResult result, Model model) {
    if (result.hasErrors()) {
        return "clientes/formulario";
    }
    clienteService.salvar(cliente);
    return "redirect:/clientes";
}
```

### Exibindo Mensagens de Erro na View

```html
<form th:action="@{/clientes}" th:object="${cliente}" method="post">
    <label for="nome">Nome:</label>
    <input type="text" id="nome" th:field="*{nome}" />
    <div th:if="${#fields.hasErrors('nome')}" th:errors="*{nome}">Nome Error</div>

    <label for="email">Email:</label>
    <input type="email" id="email" th:field="*{email}" />
    <div th:if="${#fields.hasErrors('email')}" th:errors="*{email}">Email Error</div>

    <button type="submit">Salvar</button>
</form>
```

## Parâmetros de Path e Request Mapping

### Uso de @PathVariable

Permite extrair valores de segmentos da URL e vinculá-los aos parâmetros do método.

**Exemplo**:

```java
@GetMapping("/{id}")
public String detalhesCliente(@PathVariable Long id, Model model) {
    Cliente cliente = clienteService.buscarPorId(id);
    model.addAttribute("cliente", cliente);
    return "clientes/detalhes";
}
```

- A URL `/clientes/1` chamará este método com `id = 1`.

### Uso de @RequestParam

Captura parâmetros de consulta (query parameters) da URL.

**Exemplo**:

```java
@GetMapping("/buscar")
public String buscarClientes(@RequestParam String nome, Model model) {
    List<Cliente> clientes = clienteService.buscarPorNome(nome);
    model.addAttribute("clientes", clientes);
    return "clientes/lista";
}
```

- A URL `/clientes/buscar?nome=João` passará o parâmetro `nome` com o valor `João`.

### Definindo Request Mapping a Nível de Classe

Anotando a classe com `@RequestMapping`, todas as rotas definidas nos métodos serão relativas a este caminho base.

```java
@Controller
@RequestMapping("/clientes")
public class ClienteController {
    // Métodos com caminhos relativos a /clientes
}
```

- O método com `@GetMapping("/novo")` responderá à URL `/clientes/novo`.

## Tratamento de Exceções

O Spring MVC permite o tratamento centralizado de exceções.

### Uso de @ExceptionHandler

Dentro do Controller, capture exceções específicas.

```java
@ExceptionHandler(ClienteNotFoundException.class)
public String handleClienteNotFound(Model model) {
    model.addAttribute("erro", "Cliente não encontrado");
    return "erro";
}
```

### Controller Advice Global

Use `@ControllerAdvice` para aplicar a múltiplos controllers.

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public String handleException(Exception ex, Model model) {
        model.addAttribute("erro", ex.getMessage());
        return "erro";
    }
}
```

## Internacionalização (i18n)

### Configuração

Defina o `LocaleResolver` e o interceptor de mudança de locale.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public LocaleResolver localeResolver() {
        SessionLocaleResolver slr = new SessionLocaleResolver();
        slr.setDefaultLocale(Locale.forLanguageTag("pt-BR"));
        return slr;
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor lci = new LocaleChangeInterceptor();
        lci.setParamName("lang");
        return lci;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }
}
```

### Arquivos de Mensagens

Crie arquivos `messages.properties`, `messages_en.properties`, `messages_pt_BR.properties` em `src/main/resources`.

**Exemplo (`messages_pt_BR.properties`)**:

```properties
titulo=Bem-vindo ao Sistema de Clientes
```

### Uso na View

```html
<h1 th:text="#{titulo}">Título</h1>
```

### Alterando o Idioma

- Ao adicionar `?lang=en` na URL, o idioma mudará para inglês.

## Configuração do Projeto

### Dependências no pom.xml

Inclua as dependências necessárias no `pom.xml`:

```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Thymeleaf Template Engine -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Banco de Dados H2 (para testes) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>

    <!-- Bean Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
</dependencies>
```

### Configurações no application.properties

Defina as configurações da aplicação em `src/main/resources/application.properties`:

```properties
# Configurações do Thymeleaf
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html

# Configurações do H2
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Configurações do JPA
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
```

## Boas Práticas e Considerações Finais

- **Organização do Código**:
  - Separe claramente as camadas (Controller, Service, Repository).
  - Nomeie pacotes e classes de forma consistente.

- **Injeção de Dependências**:
  - Utilize `@Autowired` para injeção automática.
  - Prefira a injeção via construtor para facilitar testes unitários.

- **Tratamento de Erros**:
  - Implemente tratamento adequado de exceções para melhorar a experiência do usuário.

- **Segurança**:
  - Considere implementar segurança com Spring Security para proteger as rotas e dados.

- **Testes**:
  - Escreva testes unitários e de integração para garantir a qualidade do código.

- **Documentação**:
  - Mantenha o código bem documentado e com comentários claros.

## Conclusão

O Spring MVC é uma ferramenta robusta e flexível para o desenvolvimento de aplicações web em Java. Ao seguir o padrão MVC e integrar-se perfeitamente com o ecossistema Spring, ele oferece aos desenvolvedores uma plataforma poderosa para construir aplicações escaláveis, modulares e de fácil manutenção.

Com recursos avançados para processamento de formulários, validação, internacionalização e tratamento de exceções, o Spring MVC simplifica muitas das tarefas comuns no desenvolvimento web. Ao adotar as práticas e conceitos apresentados neste guia, os desenvolvedores podem criar aplicações web eficientes e prontas para atender às necessidades dos usuários.

Lembre-se de que a chave para o sucesso no uso do Spring MVC é compreender profundamente seus componentes e como eles interagem. Continue explorando a documentação oficial, experimentando com código e mantendo-se atualizado com as melhores práticas da comunidade.
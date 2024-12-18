# Guia Completo sobre Spring MVC

## Introdução ao Spring MVC

O **Spring MVC** é um módulo do **Spring Framework** projetado para facilitar o desenvolvimento de aplicações web usando o padrão **MVC (Model-View-Controller)**. Esse padrão ajuda a separar claramente as responsabilidades da sua aplicação em três camadas:

- **Model (Modelo)**: Cuida dos dados e da lógica de negócio.
- **View (Visão)**: Responsável por exibir as informações para o usuário.
- **Controller (Controlador)**: Recebe requisições do usuário, decide o que fazer e retorna o resultado para a View.

Ao integrar-se com o Spring Framework, o Spring MVC herda os benefícios da **Inversão de Controle (IoC)** e da **Injeção de Dependência (DI)**, tornando seu código mais flexível, modular e fácil de manter.

---

## Como o Padrão MVC Funciona

Antes de entendermos o Spring MVC em si, vale relembrar o padrão MVC:

1. **Model (Modelo)**:
   - Representa os dados e as regras de negócio.
   - É aqui que você define suas classes de domínio, serviços e repositórios.
   - Exemplo: Uma classe `Cliente` que representa um cliente e um `ClienteService` que lida com a lógica para buscar e salvar clientes.

2. **View (Visão)**:
   - Mostra dados ao usuário e coleta suas interações (como preenchimento de formulários).
   - Normalmente são páginas HTML, templates Thymeleaf ou JSPs.
   - Exemplo: Uma página `lista.html` que exibe todos os clientes cadastrados.

3. **Controller (Controlador)**:
   - Recebe requisições do navegador (ex.: `/clientes`, `/clientes/novo`).
   - Invoca o Model para obter ou manipular dados.
   - Decide qual View deve ser retornada, passando os dados necessários.
   - Exemplo: `ClienteController` que, ao receber a requisição `GET /clientes`, consulta o `ClienteService` e retorna a lista de clientes para a View.

Essa separação torna a aplicação mais organizada, facilitando a manutenção e evolução do sistema.

---

## Fluxo de Processamento no Spring MVC

Quando uma requisição chega à sua aplicação web:

1. **Cliente faz a requisição HTTP** (ex.: `GET /clientes`).
2. **DispatcherServlet**: O Spring MVC tem um servlet especial, chamado `DispatcherServlet`, que age como um ponto central (Front Controller). Todas as requisições passam por ele primeiro.
3. **Handler Mapping**: O DispatcherServlet verifica qual Controller é responsável por aquela URL.
4. **Controller**: O método apropriado do Controller é chamado. Ele utiliza o Model (serviços, repositórios) para obter ou alterar dados.
5. **View Resolver**: Após processar a lógica, o Controller retorna o nome de uma View. O Spring usa o View Resolver para encontrar o template correto.
6. **View**: O template (por exemplo, `lista.html`) é renderizado com os dados obtidos, gerando a página final.
7. **Resposta ao Cliente**: A página HTML gerada é enviada de volta para o navegador do usuário.

Fluxo resumido:

```plaintext
Cliente -> DispatcherServlet -> Controller -> Model -> View Resolver -> View -> Cliente
```

---

## Benefícios do Spring MVC

- **Integração com o Spring**: Aproveita IoC e DI, facilitando a injeção de dependências e gerenciamento de beans.
- **Configuração Flexível**: Pode usar anotações ou XML, conforme sua preferência.
- **Suporte a Diversas Tecnologias de View**: Pode usar Thymeleaf, JSP, FreeMarker, etc.
- **Data Binding e Validação**: Facilita o processamento de formulários, vinculando dados diretamente a objetos Java.
- **Internacionalização (i18n)**: Suporte integrado para lidar com múltiplos idiomas.
- **Tratamento de Exceções**: Centralizado e fácil de gerenciar.

---

## Estrutura de uma Aplicação Spring MVC

Uma aplicação típica segue uma organização de pastas:

```plaintext
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── exemplo/
│   │           ├── controller/   // Controladores
│   │           ├── model/        // Entidades de domínio
│   │           ├── service/      // Lógica de negócio
│   │           ├── repository/   // Acesso a dados
│   │           └── Application.java // Classe principal
│   └── resources/
│       ├── templates/            // Templates HTML (Views)
│       ├── static/               // Recursos estáticos (CSS, JS, imagens)
│       └── application.properties
```

**Principais Componentes**:

- **Controllers** (`@Controller`): Lida com requisições e retorna a View.
- **Services** (`@Service`): Contêm a lógica de negócio.
- **Repositories** (`@Repository`): Interagem com o banco de dados.
- **Models**: Classes que representam dados (entidades, DTOs).
- **Views (Templates)**: Páginas HTML (Thymeleaf) ou JSP para renderizar informações.

---

## Criando um Controller e Uma View

### Exemplo de Controller

```java
@Controller
@RequestMapping("/clientes")
public class ClienteController {

    @Autowired
    private ClienteService clienteService;

    @GetMapping
    public String listarClientes(Model model) {
        List<Cliente> clientes = clienteService.listarTodos();
        model.addAttribute("clientes", clientes);
        return "clientes/lista"; // Nome do template HTML
    }
}
```

O método `listarClientes` é chamado quando o usuário acessa `/clientes`. Ele obtém todos os clientes, adiciona ao `Model` e retorna o nome da View (`clientes/lista`).

### Exemplo de View (Thymeleaf)

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

Aqui, `${clientes}` é a lista passada pelo Controller. O `th:each` itera sobre ela para exibir cada `cliente`.

---

## Processamento de Formulários

Quando você precisa inserir ou atualizar dados, utilizar formulários é essencial.

1. **Exibindo o Formulário**:

```java
@GetMapping("/novo")
public String exibirFormularioCadastro(Model model) {
    model.addAttribute("cliente", new Cliente());
    return "clientes/formulario";
}
```

Este método mostra um formulário vazio para criar um novo cliente.

2. **Processando o Formulário**:

```java
@PostMapping
public String salvarCliente(@ModelAttribute Cliente cliente) {
    clienteService.salvar(cliente);
    return "redirect:/clientes"; // Redireciona para a lista após salvar
}
```

Ao enviar o formulário, o `@ModelAttribute` vincula os campos do HTML diretamente ao objeto `Cliente`.

### Exemplo de Formulário (Thymeleaf)

```html
<form th:action="@{/clientes}" th:object="${cliente}" method="post">
    <label for="nome">Nome:</label>
    <input type="text" id="nome" th:field="*{nome}" />
    <button type="submit">Salvar</button>
</form>
```

O `th:field="*{nome}"` liga o campo de entrada ao atributo `nome` da classe `Cliente`.

---

## Data Binding e Validação

O Spring MVC facilita a validação dos dados do formulário usando o Bean Validation.

### Exemplo de Validação

```java
public class Cliente {

    @NotBlank(message = "O nome é obrigatório")
    private String nome;

    @Email(message = "Digite um email válido")
    private String email;

    // getters e setters
}
```

No Controller:

```java
@PostMapping
public String salvarCliente(@Valid @ModelAttribute Cliente cliente, BindingResult result) {
    if (result.hasErrors()) {
        return "clientes/formulario";
    }
    clienteService.salvar(cliente);
    return "redirect:/clientes";
}
```

`@Valid` aciona a validação e `BindingResult` verifica se houve erros. Se houver erros, a View é reapresentada, mostrando as mensagens adequadas.

### Exibindo Erros na View

```html
<div th:if="${#fields.hasErrors('nome')}" th:errors="*{nome}">Erro no Nome</div>
```

Essa linha exibe a mensagem de erro associada ao campo `nome` caso exista.

---

## Trabalhando com Parâmetros na URL

### @PathVariable

Para capturar parte da URL como parâmetro:

```java
@GetMapping("/{id}")
public String detalhesCliente(@PathVariable Long id, Model model) {
    Cliente cliente = clienteService.buscarPorId(id);
    model.addAttribute("cliente", cliente);
    return "clientes/detalhes";
}
```

Se o usuário acessar `/clientes/1`, `id = 1`.

### @RequestParam

Para parâmetros de consulta (ex.: `/clientes/buscar?nome=João`):

```java
@GetMapping("/buscar")
public String buscar(@RequestParam String nome, Model model) {
    List<Cliente> clientes = clienteService.buscarPorNome(nome);
    model.addAttribute("clientes", clientes);
    return "clientes/lista";
}
```

---

## Tratamento de Exceções

Quando um erro acontece, você pode lidar com isso de forma centralizada.

### @ExceptionHandler

No próprio Controller:

```java
@ExceptionHandler(ClienteNotFoundException.class)
public String handleNotFound(Model model) {
    model.addAttribute("erro", "Cliente não encontrado");
    return "erro";
}
```

### Controller Advice Global

Para tratar exceções em todos os Controllers, use `@ControllerAdvice`:

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

---

## Internacionalização (i18n)

Você pode tornar sua aplicação multi-idioma:

1. **LocaleResolver** e Interceptor**: Configura o Spring para mudar o idioma com base em um parâmetro, por exemplo `lang=en`.

2. **Arquivos de Mensagem**: Crie `messages.properties`, `messages_en.properties`, etc., e use `th:text="#{chave}` para exibir textos traduzidos.

---

## Configuração do Projeto

### Dependências no pom.xml

Você precisará de:

- `spring-boot-starter-web` (Spring MVC)
- `spring-boot-starter-thymeleaf` (Para View com Thymeleaf)
- `spring-boot-starter-data-jpa` e `h2` (Para persistência)
- `spring-boot-starter-validation` (Para validação de dados)

### application.properties

Defina configurações no arquivo `application.properties`:

```properties
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.h2.console.enabled=true
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
```

---

## Boas Práticas

- **Organize seu código**: Mantenha Controllers, Services, Repositories e Models separados.
- **Injeção via Construtor**: Prefira injeção de dependências pelo construtor para facilitar testes.
- **Tratamento de Erros Adequado**: Crie páginas de erro amigáveis.
- **Testes**: Escreva testes para garantir a qualidade do código.
- **Segurança**: Utilize Spring Security para proteger rotas e dados sensíveis.

---

## Conclusão

O Spring MVC é uma solução completa para desenvolver aplicações web em Java, permitindo que você:

- Trabalhe com o padrão MVC de forma simples.
- Aproveite todo o ecossistema Spring (IoC, DI, Spring Data, etc.).
- Use templates modernos (como Thymeleaf) para gerar interfaces agradáveis.
- Simplifique o desenvolvimento com tratamento de formulários, validações, internacionalização e muito mais.

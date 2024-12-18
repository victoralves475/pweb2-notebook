# Guia Completo sobre Spring Data JPA

## Introdução ao Spring Data JPA

O **Spring Data JPA** é um módulo do ecossistema Spring voltado para simplificar o acesso a dados em aplicações Java. Ele se integra de forma transparente com a **JPA (Java Persistence API)** e implementações como o **Hibernate**, abstraindo grande parte da complexidade da persistência de dados.

Em vez de escrever códigos longos para operações de criação, leitura, atualização e exclusão (CRUD), o Spring Data JPA fornece repositórios prontos, reduzindo o esforço de desenvolvimento. Além disso, você pode criar consultas personalizadas de forma simples, aproveitando o poder da JPA com uma sintaxe mais declarativa e intuitiva.

---

## Arquitetura da Aplicação com Spring Data JPA

Em um projeto típico de Spring, é comum dividir o código em camadas, separando as responsabilidades:

1. **Controller (Controlador)**  
   - Lida com as requisições HTTP (por exemplo: GET, POST, PUT, DELETE).
   - É a camada mais próxima do usuário (navegador, aplicativo móvel, etc.).
   - Chama a camada de serviço para executar a lógica de negócio.

   ```java
   @RestController
   @RequestMapping("/clientes")
   public class ClienteController {

       @Autowired
       private ClienteService clienteService;

       @GetMapping
       public List<Cliente> listarTodos() {
           return clienteService.listarTodos();
       }
   }
   ```

2. **Service (Serviço)**  
   - Contém a lógica de negócio.
   - Faz a ligação entre o Controller e o Repositório.
   - Aplica validações, regras de negócio e transforma dados, se necessário.

   ```java
   @Service
   public class ClienteService {

       @Autowired
       private ClienteRepository clienteRepository;

       public List<Cliente> listarTodos() {
           return clienteRepository.findAll();
       }

       public Cliente salvar(Cliente cliente) {
           // Aplicar regras de negócio, se necessário
           return clienteRepository.save(cliente);
       }
   }
   ```

3. **Repository (Repositório)**  
   - Responsável pela interação com o banco de dados.
   - Trabalha diretamente com as entidades e faz o CRUD.
   - Com o Spring Data JPA, você só precisa declarar interfaces, sem implementá-las manualmente.

   ```java
   public interface ClienteRepository extends JpaRepository<Cliente, Long> {
       // Pode adicionar métodos personalizados de consulta aqui
   }
   ```

Essa organização torna a aplicação mais fácil de manter, testar e evoluir.

---

## Preparando o Ambiente

### Dependências

No `pom.xml` (para projetos Maven), inclua:

```xml
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Driver do Banco de Dados (Exemplo: MySQL) -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

### Configuração do Banco de Dados

No arquivo `application.properties` ou `application.yml`, configure a conexão:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/minha_base
spring.datasource.username=usuario
spring.datasource.password=senha
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

- `spring.datasource.url`, `spring.datasource.username`, `spring.datasource.password`: Dados de conexão com o banco.
- `spring.jpa.hibernate.ddl-auto=update`: Atualiza o schema do banco conforme as entidades. Outras opções: `create`, `create-drop`, `validate`.
- `spring.jpa.show-sql=true`: Mostra as queries SQL geradas.
- `spring.jpa.properties.hibernate.dialect`: Especifica o dialeto do banco.

---

## Entidades

As entidades representam as tabelas do banco de dados em forma de classes Java. Use a anotação `@Entity` e `@Id` para definir a chave primária:

```java
@Entity
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private Integer idade;
    private String cidade;

    // getters e setters
}
```

- `@Entity`: Indica que a classe é uma entidade gerenciada pelo JPA.
- `@Id`: Indica a chave primária.
- `@GeneratedValue`: Define como o ID é gerado (auto-incremento, por exemplo).

---

## Repositórios

Com o Spring Data JPA, criar um repositório é simples:

```java
public interface ClienteRepository extends JpaRepository<Cliente, Long> {
    // Pode adicionar métodos de consulta personalizados
}
```

- `JpaRepository<T, ID>`: T é a entidade, ID é o tipo da chave primária.
- O JpaRepository oferece métodos prontos como `save()`, `findAll()`, `findById()`, `delete()`, etc.

```java
// Exemplo de uso no serviço
@Service
public class ClienteService {

    @Autowired
    private ClienteRepository clienteRepository;

    public List<Cliente> listarTodos() {
        return clienteRepository.findAll();
    }

    public Cliente salvar(Cliente cliente) {
        return clienteRepository.save(cliente);
    }
}
```

A implementação desses métodos é gerada automaticamente em tempo de execução. Você não precisa escrever código SQL manualmente para operações básicas.

---

## Criando Consultas Personalizadas

Às vezes, você precisa de consultas além do básico. O Spring Data JPA oferece duas abordagens:

### 1. Query Methods por Convenção do Nome

Você pode criar métodos no repositório que seguem uma convenção de nomes. O Spring entende o nome do método e cria a consulta:

```java
List<Cliente> findByNome(String nome);
List<Cliente> findByCidadeAndIdade(String cidade, Integer idade);
List<Cliente> findByNomeContaining(String parteDoNome);
```

- `findByNome`: Retorna clientes cujo nome coincida com o parâmetro.
- `findByCidadeAndIdade`: Retorna clientes pela cidade **e** idade.
- `findByNomeContaining`: Retorna clientes cujo nome contenha a sequência informada.

A documentação do Spring Data lista diversas palavras-chave que você pode usar (`And`, `Or`, `Between`, `LessThan`, `GreaterThan`, `Like`, `OrderBy`, etc.).

### 2. Uso da Anotação @Query

Para consultas mais complexas, use `@Query`:

```java
@Query("SELECT c FROM Cliente c WHERE c.idade > :idade")
List<Cliente> buscarClientesComIdadeMaiorQue(@Param("idade") int idade);
```

Isso permite que você escreva **JPQL** (uma linguagem de consulta orientada a objetos) diretamente. Também é possível usar SQL nativo:

```java
@Query(value = "SELECT * FROM clientes WHERE idade > :idade", nativeQuery = true)
List<Cliente> buscarClientesComIdadeMaiorQueNativo(@Param("idade") int idade);
```

---

## Paginação e Ordenação

Se sua tabela tiver muitos registros, exibir tudo de uma vez não é eficiente. O Spring Data JPA oferece suporte a paginação e ordenação:

```java
Page<Cliente> findByCidade(String cidade, Pageable pageable);
```

No Controller, por exemplo:

```java
@GetMapping
public Page<Cliente> listarClientes(@RequestParam int page, @RequestParam int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("nome").ascending());
    return clienteService.listarPaginado("São Paulo", pageable);
}
```

Assim, você pode controlar quantos registros são exibidos por página e a ordem deles.

---

## Transações

O Spring Data JPA trabalha com transações de forma automática. Se você precisar, pode usar a anotação `@Transactional`:

```java
@Service
public class ClienteService {

    @Autowired
    private ClienteRepository clienteRepository;

    @Transactional
    public void atualizarCliente(Cliente cliente) {
        // Operações de gravação no banco ocorrerão numa transação
        clienteRepository.save(cliente);
    }
}
```

Isso garante que, se ocorrer algum erro, as alterações sejam revertidas, mantendo a consistência dos dados.

---

## Validação de Dados

Você pode validar os dados antes de salvá-los, usando as anotações do Bean Validation:

```java
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "O nome é obrigatório")
    private String nome;

    @Min(value = 18, message = "O cliente deve ter pelo menos 18 anos")
    private Integer idade;

    // getters e setters
}
```

Ao salvar um cliente, caso os dados não cumpram as regras de validação, uma exceção será lançada, garantindo integridade e qualidade dos dados.

---

## Dicas e Boas Práticas

- **Simplicidade primeiro**: Use os métodos padrão do JpaRepository antes de criar consultas complexas.
- **Clareza nos nomes**: Ao usar query methods, crie nomes expressivos que deixem claro o critério de busca.
- **Paginação e Ordenação**: Sempre que possível, pagine resultados grandes para melhorar a performance e a experiência do usuário.
- **Transações**: Use `@Transactional` quando for necessário garantir atomicidade.
- **Monitoramento**: Ative `spring.jpa.show-sql=true` no `application.properties` para ver as queries no console e otimizar quando necessário.
- **Teste o código**: Escreva testes unitários e de integração, garantindo que as consultas funcionam como esperado e que as regras de negócio estejam corretas.

---

## Conclusão

O Spring Data JPA é uma ferramenta poderosa que simplifica muito o desenvolvimento de aplicações que acessam bancos de dados. Com ele, você obtém:

- **Produtividade**: Menos código para escrever e manter.
- **Flexibilidade**: Criação de consultas personalizadas sem complexidade.
- **Manutenibilidade**: Código mais limpo, modular e testável.

# Resumo PWeb II

1. **POJO (Plain Old Java Object)**  
   - **Conceito:** Um POJO é um objeto Java simples, sem regras ou restrições específicas além das do próprio Java. Ele não necessita de herdar de uma superclasse específica nem implementar interfaces complexas, tampouco exige anotações do framework.
   - **Resumo:** POJOs são classes simples, sem dependências a bibliotecas externas, servindo como modelos de dados ou entidades.

2. **Anotação @Controller no Spring**  
   - **Conceito:** Em aplicações Spring MVC, usamos `@Controller` para marcar classes que atuam como controladoras de requisições. A anotação mencionada no enunciado, `@SpringController`, não existe como padrão. A correta é `@Controller`.
   
3. **Uso do @ModelAttribute**  
   - **Conceito:** `@ModelAttribute` serve para colocar objetos no modelo, tornando-os disponíveis para as views, sem precisar configurar manualmente em cada método. Quando colocada sobre um método, tudo que ele retorna é adicionado ao modelo de forma global dentro da controladora.
   - **Resumo:** Métodos anotados com `@ModelAttribute` (sem especificar o nome do atributo ou especificando-o) disponibilizam dados para todas as views atendidas por aquele controller.

4. **Injeção de dependência com @Inject**  
   - **Conceito:** O Spring suporta diversas anotações para injeção de dependências. Além de `@Autowired` (do Spring) e `@Resource`, também pode ser utilizada a anotação `@Inject` (padrão JSR-330). Assim, anotar um atributo com `@Inject` é uma forma válida de injetar dependências no Spring.
   
5. **Construtores seguros para injeção**  
   - **Conceito:** O uso de injeção de dependências por construtor é não só seguro como recomendável, pois garante que o objeto seja criado já com todas as suas dependências satisfeitas. Portanto, é seguro e até boa prática inicializar variáveis injetadas no construtor.
   
6. **Extensão do template Thymeleaf**  
   - **Conceito:** Ao retornar uma view no Spring MVC usando Thymeleaf, não é obrigatório colocar a extensão `.html` (ou outra) no nome do template. Por exemplo, se o template é `index.html`, você pode retornar apenas `"index"` no método e o resolver irá localizá-lo. Assim, podemos ou não usar a extensão.
   
7. **Expressão JSP-EL `${a.b}` e tipos de dados**  
   - **Conceito:** Com a expressão JSP-EL `${a.b}`, `a` pode ser um mapa, lista, array ou qualquer objeto que ofereça acesso a propriedades. Por exemplo, se `a` for um mapa, `b` seria uma chave. Para listas e arrays, `b` seria um índice. É incorreto dizer que não pode ser um mapa.  
   - **Resumo:** JSP-EL é flexível, permitindo acessar propriedades de objetos, índices de arrays, listas, e chaves de mapas.
   
8. **Uso de `{..}` no Thymeleaf**  
   - **Conceito:** O `th:object` é utilizado para definir o objeto principal de um formulário, permitindo o uso de `{...}` para acessar suas propriedades. A expressão `{...}` só faz sentido no contexto de um objeto configurado pelo `th:object` na tag pai (por exemplo, o formulário).
   
9. **Iterações no Thymeleaf**  
   - **Conceito:** Para iterar coleções no Thymeleaf, usa-se `th:each` e não `th:for`. O `th:each` permite iterar sobre listas ou arrays diretamente no template.
   
10. **Mapeamento de URLs com @RequestMapping**  
    - **Conceito:** A anotação `@RequestMapping` (e suas variantes, como `@GetMapping`, `@PostMapping`, etc.) é usada para mapear rotas a métodos do controlador, recebendo um path (por exemplo, `@RequestMapping("/home")`).

---

## Redirect

- **Problema que o redirect resolve:**  
  Quando se envia um formulário e o método do controller retorna uma view diretamente, caso o usuário atualize a página (F5), a requisição POST anterior pode ser reenviada, ocasionando a repetição da operação (por exemplo, cadastro duplo). Esse é o famoso problema do "F5" em postagens de formulários, conhecido como o problema do "POST-Redirect-GET" (PRG).

- **Como o redirect resolve o problema:**  
  Ao retornar um "redirect:" em vez de uma view, o navegador faz primeiro o POST inicial, o servidor processa a ação (salva no banco, por exemplo) e então retorna uma resposta de redirecionamento (HTTP 302) para uma outra URL, normalmente um GET. Ao recarregar a página, o navegador estará solicitando essa nova URL (GET), não reenviando o POST original. Dessa forma, evita-se múltiplas submissões da mesma ação.

- **Exemplo de uso no Spring:**  
  ```java
  @Controller
  public class MeuController {
      
      @PostMapping("/salvar")
      public String salvar(DadosForm formulario) {
          // Lógica para salvar dados
          // ...
          return "redirect:/lista"; // Redireciona para uma página de listagem, por exemplo
      }
      
      @GetMapping("/lista")
      public String listar(Model model) {
          // Carrega dados a serem exibidos na view
          // ...
          return "lista";
      }
  }
  ```
  
---

## Injeção de Dependência no Spring

- **O que é injeção de dependência (DI):**  
  É um padrão em que o próprio framework (Spring, neste caso) gerencia a criação e fornecimento de objetos (dependências) a outras classes, sem que essas classes precisem criá-las diretamente. Isso promove um baixo acoplamento e facilita testes e manutenção.

- **O que torna um objeto apto a ser injetado:**  
  Objetos gerenciados pelo Spring são conhecidos como beans. Para que o Spring saiba que uma classe é um componente injetável, é preciso anotá-la com uma anotação indicativa, como:
  - `@Component`
  - `@Service`
  - `@Repository`
  - `@Controller`
  
  Qualquer classe anotada assim e varrida pelo processo de scan do Spring é registrada no contexto e pode ser injetada em outras classes.

- **Formas de indicar ao Spring que um objeto é injetável:**  
  Além das anotações acima, podemos configurar beans através de:
  - XML de configuração (menos comum hoje em dia).
  - Classe de configuração Java com métodos anotados com `@Bean`.
  - Uso do `@ComponentScan` para dizer ao Spring onde buscar classes anotadas.
  
  Uma vez o bean registrado, podemos injetá-lo em outro bean usando anotações como `@Autowired`, `@Inject` (JSR-330) ou `@Resource`.

---

## Uso do JPARepository do Spring

- **O que é o JPARepository:**  
  O `JpaRepository` é uma interface do Spring Data que provê métodos prontos para acesso ao banco de dados (CRUD básico) e permite criar queries personalizadas através de convenções de nomenclatura de métodos ou anotações de consulta.

- **Como utilizar um JPARepository:**  
  1. Criar uma interface que estenda `JpaRepository`:
     ```java
     public interface PessoaRepository extends JpaRepository<Pessoa, Long> {
     }
     ```
     
     Aqui, `Pessoa` é a entidade e `Long` é o tipo da chave primária.
  
  2. Uma vez configurado o Spring Data JPA e mapeadas as entidades, você pode injetar `PessoaRepository` em um controlador ou serviço:
     ```java
     @Service
     public class PessoaService {
         @Autowired
         private PessoaRepository pessoaRepo;
         
         public Pessoa salvar(Pessoa p) {
             return pessoaRepo.save(p);
         }
         
         public List<Pessoa> listarTodas() {
             return pessoaRepo.findAll();
         }
     }
     ```
  
- **Tipos de métodos no JPARepository:**  
  - **Métodos herdados do `JpaRepository`:** `save()`, `findAll()`, `delete()`, `findById()`, entre outros.
  - **Métodos de query por convenção de nomenclatura:**  
    Por exemplo, se na entidade `Pessoa` existe um campo `nome`, você pode criar:
    ```java
    List<Pessoa> findByNome(String nome);
    ```
    O Spring cria a query automaticamente com base no nome do método.
  
  - **Consultas personalizadas com @Query:**  
    ```java
    @Query("SELECT p FROM Pessoa p WHERE p.idade > :idade")
    List<Pessoa> listarPorIdadeMaiorQue(@Param("idade") int idade);
    ```
  
  Assim, o JPARepository abstrai o acesso ao banco, tornando a manipulação de dados mais simples e produtiva.

---

### Conclusões

- **POJOs:** Objetos simples, sem dependências a frameworks.
- **Anotações de controller e injeção:** `@Controller`, `@ModelAttribute`, `@Autowired`, `@Inject`.
- **Redirecionamentos:** Solução do problema de reenvio de POST com PRG (Post-Redirect-Get).
- **Injeção de dependências:** Objeto gerenciado pelo Spring através de anotações como `@Component`, `@Service`, etc.
- **Thymeleaf:** Uso de `th:object`, `th:each`, e expressões para manipular dados na view.
- **JPARepository:** Interface do Spring Data JPA para simplificar operações CRUD e consultas personalizadas.

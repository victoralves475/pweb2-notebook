# Guia Completo sobre Tratamento de Exceções no Spring MVC

## Introdução

Erros acontecem. Seja um recurso não encontrado, uma falha ao acessar o banco de dados ou problemas de validação, é importante lidar com exceções de forma adequada em aplicações web. No **Spring MVC**, o tratamento de exceções é flexível e pode ser feito de várias maneiras, desde o uso de configurações padrão até mecanismos avançados e centralizados.

Ao tratar exceções corretamente, você:

- Evita expor informações sensíveis para o usuário final.
- Oferece páginas de erro mais amigáveis, em vez de telas genéricas e confusas.
- Retorna códigos de status HTTP adequados, melhorando a interação com APIs REST.
- Mantém o código organizado, facilitando manutenção e futuras evoluções.

---

## Formas de Tratar Exceções

O Spring MVC oferece diferentes níveis de tratamento de exceções:

1. **Padrão**: Se nada for configurado, o Spring Boot exibe a **Whitelabel Error Page**, uma página de erro genérica.
2. **Local (por Controlador)**: Você pode tratar exceções específicas dentro de um controlador, usando `@ExceptionHandler`.
3. **Global (toda a aplicação)**: Com `@ControllerAdvice`, é possível definir um ponto único para tratar exceções de todos os controladores.

---

## Página de Erro Padrão (Whitelabel Error Page)

Por padrão, ao ocorrer um erro não tratado, o Spring Boot retorna a **Whitelabel Error Page**, uma página simples e genérica. Embora útil para desenvolvimento, não é apropriada para produção, pois não é customizável e não oferece uma boa experiência ao usuário.

### Customizando a Página de Erro

O Spring mapeia erros para a rota `/error`. Se você criar um arquivo `error.html` em `src/main/resources/templates`, ele substituirá a página padrão.

Exemplo (`error.html`):

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Ocorreu um Erro</title>
</head>
<body>
    <h1>Ops, algo deu errado!</h1>
    <p th:text="${error}">Mensagem de erro não disponível</p>
</body>
</html>
```

Assim, quando um erro ocorrer e não houver tratamento específico, o usuário verá essa página personalizada, em vez da Whitelabel Error Page.

---

## Definindo Códigos de Status HTTP com @ResponseStatus

Por padrão, uma exceção não tratada retorna um **status 500 (Internal Server Error)**. Mas muitas vezes você quer retornar outros códigos de status, como 404 (Not Found) para recursos inexistentes.

Use `@ResponseStatus` na classe da exceção:

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class RecursoNaoEncontradoException extends RuntimeException {
    public RecursoNaoEncontradoException(String mensagem) {
        super(mensagem);
    }
}
```

Agora, se essa exceção for lançada, o Spring retornará automaticamente 404. Isso ajuda a diferenciar tipos específicos de erro e torna a API mais intuitiva para clientes REST.

---

## Tratamento Local de Exceções com @ExceptionHandler

Se um determinado controlador (controller) é o único que lança um tipo de exceção, pode ser mais lógico tratar essa exceção dentro dele.

### Exemplo:

```java
@Controller
public class ContaController {

    @GetMapping("/conta/{id}")
    public String exibirConta(@PathVariable Long id, Model model) {
        Conta conta = contaService.buscarConta(id);
        if (conta == null) {
            throw new RecursoNaoEncontradoException("Conta não encontrada!");
        }
        model.addAttribute("conta", conta);
        return "contaView";
    }

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    public String tratarRecursoNaoEncontrado(RecursoNaoEncontradoException ex, Model model) {
        model.addAttribute("mensagemErro", ex.getMessage());
        return "erroPersonalizado";
    }
}
```

- Se `RecursoNaoEncontradoException` ocorrer no `ContaController`, o método anotado com `@ExceptionHandler` será chamado.
- Ele prepara o modelo com a mensagem de erro e retorna a view `erroPersonalizado`, mostrando ao usuário algo mais amigável que um 500 genérico.

**Vantagem**: Mantém o tratamento de erros próximo da lógica que gera essas exceções.

**Limitação**: Esse tratamento vale apenas para esse controlador. Se outra parte da aplicação lançar a mesma exceção, não será tratada aqui.

---

## Tratamento Global de Exceções com @ControllerAdvice

Se você quiser um ponto central para tratar exceções de toda a aplicação, use `@ControllerAdvice`. Assim, você não precisa replicar métodos `@ExceptionHandler` em cada controlador.

### Exemplo:

```java
@ControllerAdvice
public class TratadorGlobalDeExcecoes {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    public ModelAndView tratarRecursoNaoEncontrado(RecursoNaoEncontradoException ex) {
        ModelAndView mv = new ModelAndView("erroGlobal");
        mv.addObject("mensagemErro", ex.getMessage());
        return mv;
    }

    @ExceptionHandler(Exception.class)
    public ModelAndView tratarExcecoesGenericas(Exception ex) {
        ModelAndView mv = new ModelAndView("erroGenerico");
        mv.addObject("mensagemErro", "Ocorreu um erro inesperado. Tente novamente mais tarde.");
        return mv;
    }
}
```

- `@ExceptionHandler(RecursoNaoEncontradoException.class)`: Se qualquer controlador lançar essa exceção, esse método será chamado.
- `@ExceptionHandler(Exception.class)`: Captura qualquer outra exceção não tratada, garantindo que o usuário nunca fique sem resposta amigável.

**Vantagem**: Centraliza o tratamento de erros. Uma única classe controla o comportamento de exceções em todo o projeto.

**Ordem de execução**: Métodos `@ExceptionHandler` no próprio controlador têm prioridade. Se o controlador não tiver tratamento específico, o `@ControllerAdvice` assume.

---

## Quando Usar Cada Abordagem?

- **Página de Erro Padrão**: Útil como fallback. Se você não tratar a exceção explicitamente, ao menos o usuário verá uma página customizada, não a genérica Whitelabel.

- **@ResponseStatus**: Quando você cria exceções customizadas e quer retornar códigos de status adequados (404, 400, etc.) automaticamente.

- **Tratamento Local (@ExceptionHandler no Controller)**:  
  - Use quando a exceção se aplica especificamente a um controlador.
  - Mantém o tratamento de erros perto da lógica que causa a exceção.

- **Tratamento Global (@ControllerAdvice)**:  
  - Ideal quando você quer uma abordagem consistente para toda a aplicação.
  - Menos repetição, mais organização.
  - Útil para capturar exceções genéricas e fornecer uma resposta padrão.

---

## Boas Práticas

- **Seja Específico**: Crie exceções personalizadas para cenários comuns (como `RecursoNaoEncontradoException`) e associe-as a códigos de status coerentes.
- **Evite Informações Sensíveis**: Nunca exiba stack traces ou detalhes internos ao usuário final.
- **Forneça Mensagens Claras**: Diga ao usuário o que aconteceu e, se possível, o que ele pode fazer.
- **Mantenha a Consistência**: Se algumas exceções são tratadas localmente e outras globalmente, certifique-se de manter um padrão de apresentação.

---

## Conclusão

O tratamento de exceções no Spring MVC é um aspecto crucial da robustez e da experiência do usuário. Ao dominar recursos como `@ExceptionHandler`, `@ControllerAdvice` e `@ResponseStatus`, você estará preparado para criar aplicações mais estáveis, seguras e amigáveis.

Com essas práticas, mesmo em situações de erro, seus usuários terão respostas adequadas, páginas de erro significativas e a sensação de que a aplicação foi bem projetada.

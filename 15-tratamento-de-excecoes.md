# Guia Completo sobre Tratamento de Exceções no Spring MVC

## Introdução

O tratamento de exceções no **Spring MVC** é essencial para melhorar a experiência do usuário, garantir respostas adequadas a erros e evitar exposições desnecessárias de informações internas. Ao lidar com exceções de forma sistemática, é possível manter a aplicação mais robusta e segura, além de oferecer mensagens de erro claras e úteis.

Existem três níveis principais de tratamento de exceções no Spring MVC:

1. **Tratamento Automático (Padrão)**: Se nada for configurado, o Spring retorna a Whitelabel Error Page.
2. **Tratamento Local**: Configurado dentro de um controlador específico, usando `@ExceptionHandler`.
3. **Tratamento Global**: Centralizado para toda a aplicação, usando `@ControllerAdvice`.

## Página de Erro Padrão

Quando uma exceção não é tratada explicitamente, o Spring Boot exibe uma Whitelabel Error Page, uma página genérica com informações sobre o erro. Essa página não é adequada para o ambiente de produção.

### Página de Erro Customizada

Por convenção, qualquer requisição que resulte em erro é mapeada para a URL `/error`. Se você criar um arquivo `error.html` em `src/main/resources/templates`, o Spring Boot usará esse template como página de erro, substituindo a Whitelabel Error Page.

**Exemplo**: Crie `error.html` em `templates`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Erro</title>
</head>
<body>
<h1>Ocorreu um erro</h1>
<p th:text="${error}">Mensagem de erro padrão</p>
</body>
</html>
```

Assim, sem necessidade de um controlador adicional, a página `error.html` será exibida quando ocorrer um erro não tratado.

## Exceções e Status HTTP

Quando uma exceção não é tratada, por padrão o Spring retorna um status HTTP **500 (Internal Server Error)**. No entanto, podemos associar exceções a códigos de status HTTP específicos.

### Uso de @ResponseStatus

Anote sua classe de exceção com `@ResponseStatus` para informar ao Spring qual código HTTP utilizar quando essa exceção for lançada:

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class RecursoNaoEncontradoException extends RuntimeException {
    public RecursoNaoEncontradoException(String mensagem) {
        super(mensagem);
    }
}
```

Agora, se `RecursoNaoEncontradoException` for lançada, o Spring retornará automaticamente o status 404 (Not Found).

## Tratamento Local de Exceções com @ExceptionHandler

É possível tratar exceções diretamente dentro de um controlador, usando `@ExceptionHandler`. Esse método é útil para exceções específicas daquele contexto.

**Exemplo**:

```java
@Controller
public class ContaController {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    public String tratarRecursoNaoEncontrado(RecursoNaoEncontradoException ex, Model model) {
        model.addAttribute("mensagemErro", ex.getMessage());
        return "erroPersonalizado";
    }

    @GetMapping("/conta/{id}")
    public String exibirConta(@PathVariable Long id, Model model) {
        Conta conta = contaService.buscarConta(id);
        if (conta == null) {
            throw new RecursoNaoEncontradoException("Conta não encontrada!");
        }
        model.addAttribute("conta", conta);
        return "contaView";
    }
}
```

Nesse exemplo, se `RecursoNaoEncontradoException` for lançada dentro de `ContaController`, o método anotado com `@ExceptionHandler` irá capturá-la, montar a resposta e retornar a view `erroPersonalizado`.

**Vantagens do tratamento local**:  
- Encapsula o tratamento de exceções que fazem sentido apenas no contexto daquele controlador.
- Mantém o código mais organizado e fácil de entender.

**Limitações**:  
- Não cobre exceções lançadas em outros controladores ou fora do escopo do controlador atual.

## Tratamento Global de Exceções com @ControllerAdvice

Se você precisa tratar exceções de forma centralizada, use `@ControllerAdvice`. Essa anotação cria um componente que observa todas as exceções lançadas nos controladores da aplicação.

**Exemplo**:

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
        mv.addObject("mensagemErro", "Ocorreu um erro inesperado");
        return mv;
    }
}
```

Agora, qualquer `RecursoNaoEncontradoException` lançada em qualquer controlador será tratada pelo método `tratarRecursoNaoEncontrado`. Para outras exceções genéricas, `tratarExcecoesGenericas` será acionado.

**Vantagens do tratamento global**:  
- Centraliza o tratamento de exceções em um único lugar.
- Evita duplicação de código.
- Facilita a manutenção e evolução da lógica de tratamento de erros.

### Prioridade entre Tratamentos

- Métodos `@ExceptionHandler` locais (no próprio controlador) têm prioridade sobre métodos globais `@ControllerAdvice`.
- Se o controlador não tiver um tratamento específico, o `@ControllerAdvice` entra em ação.

## Quando Usar Cada Abordagem?

- **@ResponseStatus**:  
  Quando você tem exceções específicas criadas para indicar um determinado tipo de erro (por exemplo, recurso não encontrado), associe essas exceções a um status HTTP adequado.  
  Uso típico: `@ResponseStatus(HttpStatus.NOT_FOUND)` em `RecursoNaoEncontradoException`.

- **Tratamento Local com @ExceptionHandler**:  
  Quando uma exceção é relevante apenas para um determinado controlador.  
  Uso típico: tratar `RecursoNaoEncontradoException` dentro de `ContaController`.

- **Tratamento Global com @ControllerAdvice**:  
  Quando você quer um ponto único de tratamento de exceções para toda a aplicação, mantendo a lógica de erro centralizada.  
  Uso típico: tratar exceções genéricas e comuns a vários controladores, fornecendo páginas de erro padronizadas.

## Resumo

1. **Página de Erro Padrão**:  
   Crie um `error.html` para substituir a Whitelabel Error Page, fornecendo uma experiência melhor em caso de erros não tratados.

2. **@ResponseStatus**:  
   Associe exceções personalizadas a códigos de status HTTP específicos, dando feedback imediato ao cliente sobre o tipo de erro (404, 400, 500, etc.).

3. **Tratamento Local (@ExceptionHandler)**:  
   Encapsule o tratamento de exceções específicas de um controlador, mantendo a lógica de erros relacionada apenas às funcionalidades daquele controlador.

4. **Tratamento Global (@ControllerAdvice)**:  
   Centralize o tratamento de exceções para toda a aplicação, garantindo uniformidade nas respostas a erros e reduzindo a duplicação de código.

Ao adotar essas práticas, você garante que a aplicação Spring MVC seja mais robusta, mantenha uma experiência de usuário coerente mesmo em situações de erro e siga as melhores práticas de desenvolvimento web.

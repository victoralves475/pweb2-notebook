# Guia Completo sobre Redirect e Flash Attributes no Spring MVC

## Introdução

Em aplicações web desenvolvidas com **Spring MVC**, é comum lidar com fluxos que envolvem o envio de dados (como formulários) e o retorno de mensagens de confirmação ou erro para o usuário. Dois recursos importantes nesse contexto são:

- **Redirect**: Redireciona o navegador do usuário para outra página após a conclusão de uma ação.
- **Flash Attributes**: Permite enviar mensagens temporárias (como “Operação realizada com sucesso”) entre requisições, sem armazená-las permanentemente.

Esses recursos resolvem problemas comuns, como o reenvio acidental do mesmo formulário ao pressionar a tecla F5, e a necessidade de exibir mensagens informativas ao usuário após uma operação bem-sucedida ou falha.

---

## Entendendo o ModelAndView

Antes de falarmos de redirect e flash attributes, é útil entender o `ModelAndView`:

- **ModelAndView**: É um objeto que pode ser retornado pelos métodos do seu controller. Ele combina:
  - O nome da view (a página que será exibida).
  - Os dados (modelo) que serão mostrados nessa página.

```java
@RequestMapping("/exemplo")
public ModelAndView exemplo() {
    ModelAndView mv = new ModelAndView("nomeDaView");
    mv.addObject("mensagem", "Olá, Mundo!");
    return mv;
}
```

Nesse caso, a view `nomeDaView` receberá o atributo `mensagem` com o valor "Olá, Mundo!". Assim, a página poderá exibir essa mensagem ao usuário.

Embora `ModelAndView` seja útil, muitas vezes no Spring MVC retornamos apenas o nome da view (como uma `String`) e deixamos o Spring cuidar do resto, especialmente quando usamos anotações como `@Controller` e `@GetMapping`. Ainda assim, entender o `ModelAndView` é bom para ter mais flexibilidade em casos complexos.

---

## Problema: Reenvio de Formulário ao Atualizar a Página (F5)

Imagine um cenário comum:

1. O usuário preenche um formulário e clica em “Salvar” (requisição **POST**).
2. O servidor processa o formulário e salva os dados no banco, retornando uma página de listagem ou de detalhes.

Porém, se o usuário pressiona F5 (atualizar a página) logo depois, o navegador tenta reenviar a **mesma requisição POST**. Isso pode causar duplicações de registros ou executar a mesma ação duas vezes, o que não é desejável.

Esse problema é conhecido como **problema do reenvio de formulário** ou **Double Submit Problem**.

---

## A Solução: POST-Redirect-GET (PRG)

Para evitar o reenvio do formulário, usa-se o padrão **POST-Redirect-GET (PRG)**:

1. **POST**: O usuário envia o formulário.
2. **Redirect**: Após processar o formulário, o servidor não retorna diretamente a página final. Em vez disso, retorna uma resposta de redirecionamento (redirect) para outra URL.
3. **GET**: O navegador, ao receber o redirect, faz uma nova requisição GET para a URL fornecida. Agora, se o usuário pressionar F5, só enviará um GET (normalmente inofensivo), não repetindo o POST.

### Sem Redirect (Exemplo Problemático)

```java
@PostMapping("/salvar")
public String salvar(@ModelAttribute Livro livro) {
    // Lógica de salvamento no banco de dados
    return "listarLivros"; // Retorna a view de listagem diretamente
}
```

Se o usuário pressionar F5 nessa página, o navegador tentará repetir o POST, causando duplicação de dados.

### Com Redirect (Exemplo Correto)

```java
@PostMapping("/salvar")
public String salvar(@ModelAttribute Livro livro) {
    // Salva o livro
    return "redirect:/livros"; // Redireciona para "/livros" (GET)
}
```

Agora, após salvar, o navegador recebe um redirect e faz um GET em `/livros`. Pressionar F5 no `/livros` não reenviará o POST anterior.

---

## Flash Attributes: Mensagens Temporárias entre Requisições

Ao usar o PRG, surge outro problema: como mostrar para o usuário uma mensagem de sucesso ou erro após o redirecionamento?

- Se tentarmos passar dados no `Model`, eles não ficarão disponíveis após o redirect, pois o redirect é uma nova requisição.
- Não queremos guardar essas mensagens de forma permanente (como em sessão de longo prazo).

A solução é usar **Flash Attributes**:

- **Flash Attributes**: Atributos que persistem apenas até a próxima requisição. Eles sobrevivem ao redirect, mas depois são removidos.
- Perfeitos para mensagens do tipo “Operação realizada com sucesso!”.

### Como Adicionar Flash Attributes

Use `RedirectAttributes`:

```java
@PostMapping("/salvar")
public String salvar(@ModelAttribute Livro livro, RedirectAttributes redirectAttributes) {
    // Salva o livro
    redirectAttributes.addFlashAttribute("mensagemSucesso", "Livro salvo com sucesso!");
    return "redirect:/livros";
}
```

O atributo `mensagemSucesso` estará disponível para a próxima requisição (a do `/livros`), podendo ser exibido na página.

### Exibindo a Mensagem na View

No template da página `/livros` (por exemplo, usando Thymeleaf):

```html
<p th:if="${mensagemSucesso}" th:text="${mensagemSucesso}"></p>
```

Se `mensagemSucesso` estiver presente, ela será mostrada. Caso contrário, nada será exibido.

---

## Como o Redirect Funciona por Baixo dos Panos

O redirect acontece assim:

1. O servidor responde à requisição POST com um status de redirecionamento (por exemplo, 302) e um cabeçalho `Location` com a nova URL.
2. O navegador recebe isso e automaticamente faz um GET para a nova URL.
3. O usuário vê o resultado final sem perceber tecnicamente o que aconteceu nos bastidores, apenas a mudança de página.

---

## Resumo dos Conceitos

- **ModelAndView**:  
  Permite retornar view + dados em um só objeto, mas é opcional. Você pode retornar `String` para a view e usar `Model` para atributos.

- **Redirect**:  
  Resolve o problema de reenvio de formulário. Com o padrão POST-Redirect-GET, a requisição POST é seguida por um redirect, assegurando que a atualização da página não repita a operação POST.

- **Flash Attributes**:  
  Transportam dados temporários (como mensagens de sucesso ou erro) da requisição POST para a próxima requisição GET após o redirect. São excluídos automaticamente depois, evitando armazenamento desnecessário.

---

## Conclusão

Ao combinar Redirect e Flash Attributes, você:

- Evita que o usuário reenvie formulários acidentalmente ao atualizar a página.
- Fornece feedback visual imediato após uma operação (por exemplo, “Livro salvo com sucesso!”) sem manter dados em sessão indefinidamente.

Essas práticas melhoram a **experiência do usuário** e garantem um fluxo de navegação mais seguro e fluido em suas aplicações Spring MVC.

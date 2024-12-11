# Guia Completo sobre Redirect e Flash Attributes no Spring MVC

## Introdução

No desenvolvimento de aplicações web usando o **Spring MVC**, o controle do fluxo de requisições é fundamental. Dois conceitos essenciais para um fluxo saudável de navegação e experiência do usuário são o **Redirect** e o uso de **Flash Attributes**. Eles resolvem problemas comuns, como o reenvio indesejado de formulários (ao pressionar F5) e a necessidade de exibir mensagens de sucesso ou erro após redirecionamentos.

## ModelAndView

Antes de aprofundarmos em Redirect e Flash Attributes, é importante entender o papel da classe `ModelAndView`.

- **ModelAndView**: Combina o nome da View a ser renderizada com os dados do modelo a serem exibidos nessa View.
- Alternativa ao retorno direto de strings (nomes de views). Permite maior flexibilidade, agrupando a lógica de qual view será renderizada e quais dados serão repassados para a página.

**Exemplo de Uso**:

```java
@RequestMapping("/exemplo")
public ModelAndView exemplo() {
    ModelAndView mv = new ModelAndView("nomeDaView");
    mv.addObject("mensagem", "Olá, Mundo!");
    return mv;
}
```

Nesta abordagem, a view `nomeDaView` receberá o atributo `mensagem` com o valor "Olá, Mundo!".

## Problema com POST sem Redirect

Considere um cenário típico: uma página de cadastro onde o usuário envia um formulário (requisição POST), e o servidor salva os dados. Caso, após o salvamento, a resposta do servidor seja simplesmente a renderização da mesma página ou de uma página de listagem, um problema pode surgir:

- Se o usuário pressionar **F5** (atualizar a página), o navegador reenviará a última requisição POST.
- Isso pode causar duplicação de dados no servidor, pois a operação de salvamento seria executada novamente.

Esse problema é conhecido como **Double Submit Problem** (ou reenvio de formulários ao atualizar a página).

## Solução: Redirect (Redirecionamento)

Para resolver o problema de reenviar a requisição POST ao atualizar a página, utiliza-se o padrão **POST-Redirect-GET (PRG)**:

1. **POST**: O usuário envia um formulário (requisição POST).
2. **Redirect**: O servidor salva os dados e em vez de retornar diretamente a view, retorna uma resposta de redirecionamento para outra URL (geralmente um GET).
3. **GET**: O navegador, ao receber o redirect, faz uma nova requisição GET para a URL fornecida.

Assim, ao pressionar F5, o navegador só reenviará a última requisição GET (visualização de uma página), e não o POST de salvamento. Isso previne a duplicação de registros.

### Exemplo sem Redirect

```java
@PostMapping("/salvar")
public String salvar(@ModelAttribute Livro livro) {
    // Lógica para salvar o livro no banco de dados
    return "listarLivros"; // Retorna diretamente a view "listarLivros.html"
}
```

Neste caso, se o usuário apertar F5 na página de listagem, o navegador tentará reenviar o POST anterior, causando duplicatas.

### Exemplo com Redirect

```java
@PostMapping("/salvar")
public String salvar(@ModelAttribute Livro livro) {
    // Lógica para salvar o livro no banco de dados
    return "redirect:/livros"; // Redireciona para a rota "/livros"
}
```

Agora, após o salvamento, o servidor envia um redirect. O navegador faz uma nova requisição GET para `/livros`. Pressionar F5 agora só reenviará o GET, não o POST, evitando problemas de duplicidade.

## Flash Attributes

O **Redirect** resolve o problema do reenvio de formulário, porém cria uma nova questão: como passar mensagens ou dados temporários entre a requisição POST e a página exibida após o redirecionamento?

Se quisermos exibir uma mensagem de confirmação ("Livro salvo com sucesso!") após o redirecionamento, precisamos de uma forma de transportar essa informação para a próxima requisição sem armazená-la permanentemente no servidor.

É aí que entram os **Flash Attributes**.

### O que são Flash Attributes?

- São atributos temporários armazenados no escopo de flash.
- Duram apenas até a próxima requisição.
- São úteis para mensagens de sucesso, erro, ou qualquer informação transitória entre um POST e o GET subsequente após um redirect.

### Como usar Flash Attributes

A injeção de `RedirectAttributes` no método do controller permite adicionar flash attributes facilmente:

```java
@PostMapping("/salvar")
public String salvar(@ModelAttribute Livro livro, RedirectAttributes redirectAttributes) {
    // Salvar o livro
    redirectAttributes.addFlashAttribute("mensagem", "Livro salvo com sucesso!");
    return "redirect:/livros";
}
```

No exemplo acima:

1. Ao salvar o livro, adicionamos o atributo `"mensagem"` ao escopo de flash.
2. Após o redirect, a próxima requisição GET a `/livros` poderá acessar `mensagem`.
3. Esse atributo é então removido automaticamente após ser acessado.

### Exibindo Mensagens na View

Na página `/livros` (template Thymeleaf, por exemplo):

```html
<p th:if="${mensagem}" th:text="${mensagem}"></p>
```

Se houver uma `mensagem` no modelo, ela será exibida. Como `mensagem` veio via flash attribute, só estará disponível nessa primeira requisição após o redirect.

## Dinâmica das Conexões HTTP com Redirect

Um redirect é baseado nos protocolos HTTP. Quando o servidor quer redirecionar:

1. O servidor envia uma resposta com código de status 3xx (por exemplo, **302 Found**), juntamente com o cabeçalho `Location: /nova-url`.
2. O navegador interpreta esse cabeçalho `Location` e faz automaticamente um novo **GET** para a URL especificada.
3. Esse processo é transparente para o usuário.

O protocolo HTTP e os navegadores dão suporte nativo a esse fluxo, permitindo implementá-lo facilmente no Spring MVC.

## Resumo dos Conceitos

1. **ModelAndView**:  
   Permite retornar tanto o nome da view quanto dados do modelo em um único objeto. Uma alternativa ao retorno simples de strings, dando mais flexibilidade e clareza.

2. **Redirect**:  
   - Resolve o problema do reenvio de formulários (POST-Redirect-GET).
   - Garante que ao pressionar F5 o usuário não repetirá a ação de envio de dados.
   - Mantém a URL atualizada corretamente no navegador.

3. **Flash Attributes**:  
   - Permite passar dados temporários de uma requisição para outra, especialmente útil ao usar `redirect`.
   - Ideal para exibir mensagens de sucesso, erro ou avisos após redirecionamentos.
   - Atributos expiram após a próxima requisição, evitando necessidade de armazenar dados no servidor ou em sessão de forma duradoura.

## Conclusão

A compreensão e aplicação correta de **Redirect** e **Flash Attributes** é essencial para uma experiência do usuário sólida e livre de problemas como duplicação de dados ou ausência de feedback após operações sensíveis. Ao dominar esses recursos, você garantirá que sua aplicação Spring MVC siga as melhores práticas, tornando a navegação fluída, confiável e com uma UX consistente.

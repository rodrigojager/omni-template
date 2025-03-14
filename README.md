# OmniTemplate

Have you ever lost a lot of time creating a user interface/component from scratch?

Do you work with an uncommon stack for which no one has yet developed something like Shadcn, Flutterflow, or Bootstrap Studio? Or it is just not completely free, open source and extensible by you or the community?

Or maybe you made a lot of web components in your favorite UI framework to speed up the creating process of your web pages, and now you or your company have decided that you need to replicate the same user interface (or components) in a different stack and/or for a different device, such as web, desktop, mobile, watch, TV, car, refrigerator, or even a capybara (they are probably very hard to code for)?

This is where **OmniTemplate** joins the scene to help you.

OmniTemplate provides a menu with **dozens of components** that you can symply  **drag and drop, easily customize, and nest it** to make your desired interface for **ANY** **codeable stack** and not just the usual React/Flutter. Can't find the component you need? Ask the community or create it yourself. Found the component, but it's not yet available for your stack? Again, reach out to the community or develop it yourself. Do you want to create it, but want to keep it for yourself/your company and not sharing? That made us sad, but go ahead, just make it and place it on your plugins folder. 

How does it work? You choose a component from the menu, you drag and drop it on the canvas or inside another component, edit its properties and build your complete layout. When you are done, you choose the desired output (stack) and language (localization) from the menu with the current available options and press the "Generate" button. OmniTemplate will process the template and generate the corresponding code for you. Want to integrate OmniTemplate in Visual Studio or Visual Studio Code? There is an extension available for that.

# Project Architecture Overview
This project implements a modular code generation pipeline based on a structured JSON AST. The goal is to separate concerns and allow contributors to extend support for new languages and localizations easily. The pipeline is divided into four main layers:

## 1. Skeleton (AST in JSON):
A structured JSON file that defines the generic component architecture. This file serves as the Abstract Syntax Tree (AST) directly and preserves the hierarchy and order of elements.

- ** Structure:**
The skeleton is organized as a tree with nodes that have a type, children, and optional attributes. For example:

```json
    {
      "type": "component",
      "children": [
        {
          "type": "form",
          "children": [
            { "type": "title", "key": "formTitle" },
            {
              "type": "field",
              "children": [
                { "type": "label", "key": "field1Label" },
                { "type": "input", "subtype": "text", "key": "field1Input" }
              ]
            }
          ]
        },
        {
          "type": "style",
          "attributes": { "bgColor": "bgColor", "fontColor": "fontColor" },
          "children": []
        },
        {
          "type": "script",
          "attributes": { "onClick": "onClickAction" },
          "children": []
        }
      ]
    }
```

The skeleton represents the generic structure of your component, including markup, style, and script sections. It is used as the input for the conversion engine.

## 2.** Data (Localized JSON):**
A separate JSON file contains the actual data values for the placeholders defined in the skeleton. The properties will be edited in a proper window/form.

- **Structure Example (data-en.json):**

```json
{
  "formTitle": "Contact Form",
  "field1Label": "Name",
  "field1Input": "Enter your name",
  "bgColor": "#ffffff",
  "fontColor": "#000000",
  "onClickAction": "handleClick"
}
```

This layer allows for easy localization and customization of component content without altering the structure.



## 3. **Conversion Rules:**
A single JSON file contains the conversion rules that transform the generic skeleton into the final code for the target language (e.g., HTML, XAML/MAUI, Flutter etc.).

- **Structure Example:**

```json
{
  "rules": [
    {
      "name": "convertTitle",
      "nodeType": "title",
      "type": "static",
      "replacement": "<h1>{{formTitle}}</h1>"
    },
    {
      "name": "convertLabel",
      "nodeType": "label",
      "type": "static",
      "replacement": "<label>{{field1Label}}</label>"
    },
    {
      "name": "convertInput",
      "nodeType": "input",
      "type": "static",
      "replacement": "<input type=\"{{subtype}}\" placeholder=\"{{field1Input}}\" />"
    },
    {
      "name": "convertStyle",
      "nodeType": "style",
      "type": "static",
      "replacement": "<style>body { background-color: {{bgColor}}; color: {{fontColor}}; }</style>"
    },
    {
      "name": "convertOnClick",
      "nodeType": "script",
      "attribute": "onClick",
      "type": "js",
      "functionBody": "return 'onclick=\"' + group1.trim() + '()\"';"
    }
  ]
}
```

This file defines how each element in the skeleton should be converted. Each rule includes a regex pattern for matching parts of the AST and a replacement that can be either a static string or a JavaScript function (wrapped in a safe environment) to produce dynamic output. I am considering making some sort of conversion engine with plugin architecture to only allow a few pre-approved subset of functions to make the conversions safe/secure, but I don't want to make it a more closed approach than it has to be.

##### Note on Security:
For JavaScript-based rules, the community and I will work together to prevent arbitrary code execution. So to make your component future-proof, use it as much as **needed only** and if you have suggestions on how to avoid it becoming a huge mess, discuss it on the board or with issues/pull requests.

## **Localization:**
Files that provide localized strings in different languages (e.g., data-en.json for English, data-ptBR.json for Portuguese).

This layer ensures that the generated code can support multiple languages by swapping the data file without affecting the component structure or conversion rules.
#### How It Works
**1. Skeleton Input:**
The conversion engine takes the JSON (skeleton) as input, which already contains the hierarchical structure of the component.

**2. Conversion Process:**
The engine traverses the skeleton, applying the conversion rules defined in the rules file.

- For each node, the engine checks the type and applies the corresponding static or dynamic (JS-based) rule.
- If a rule is of type "static", it simply replaces the matched content using the provided replacement string.
- If a rule is of type "js", the engine converts the functionBody string into a function and calls it with the appropriate match groups (e.g., group1).

**3. Localization and Final Output:**
After conversion, the output template (which may still contain placeholders) is processed with a template engine (such as Handlebars) using the localized data JSON, generating the final code output for the target language.

## How to Contribute:

Fork the repository and create a new branch. When you are done developing and testing, make a pull request. Always ensure your changes are well documented and tested. Below are some more specific guidelines for each kind of contribution:

- ** Developing thes system itself:**
This is going to be developed in .NET MAUI for the sole reason that I want it to be desktop compatible with any operating system and that I rather work with C# .NET than with other programming languages. Windows Forms are just too ugly and won't work with Linux/Mac properly, so...
- **Adding New Conversion Rules:**
 - Add or modify rules in the conversion rules JSON file for your target language/system. 
 
 If your contribution is extremely specific to your company/business/application, you can just output the files to the plugins folder and if you are following the guidelines properly it will just work out of the box, but it won't make to this repository.

- **Extending the Components Creating new Skeletons:**
 - Add new component types, following the original schema.
 
- ** Localization Contributions:**

- Add new localization files (e.g., data-es.json for Spanish) following the same field structure as the default data-en.json and also the same folder structure for each component.
- Ensure all keys are translated consistently.
### General Guidelines:

- Keep changes modular. Each layer (Skeleton, Data, Conversion Rules, Localization) should be independent.
- Follow the coding standards outlined in our CONTRIBUTING.md.
- Test your changes by generating sample output and verifying that the final code behaves as expected in the target environment.
- By following this architecture, our project remains flexible, secure, and easily extensible. Contributors can work on specific layers without interfering with others, enabling rapid expansion of support for new target languages and localized content.

Feel free to reach out with questions or suggestions through our GitHub Issues or Discussion boards.

Happy coding!

***
<br>


<br>

# OmniTemplate

Você já perdeu muito tempo criando uma interface de usuário/componente do zero?

Você trabalha com uma stack incomum para a qual ninguém desenvolveu algo como Shadcn, Flutterflow ou Bootstrap Studio? Ou simplesmente não é completamente gratuito, de código aberto e extensível por você ou pela comunidade?

Ou talvez você tenha criado muitos componentes web no seu framework de UI favorito para acelerar o processo de criação das suas páginas web, e agora você ou sua empresa decidiram que precisam replicar a mesma interface de usuário (ou componentes) em uma stack diferente e/ou para um dispositivo diferente, como web, desktop, mobile, relógio, TV, carro, geladeira ou até mesmo uma capivara (que provavelmente são muito difíceis de programar)?

É aqui que o **OmniTemplate** entra em cena para ajudar você.

O OmniTemplate fornece um menu com **dezenas de componentes** que você pode simplesmente **arrastar e soltar, personalizar facilmente e aninhar** para criar a interface desejada para **QUALQUER stack programável** e não apenas o React/Flutter habitual. Não encontrou o componente que precisa? Pergunte à comunidade ou crie você mesmo. Encontrou o componente, mas ele ainda não está disponível para sua stack? Novamente, entre em contato com a comunidade ou desenvolva você mesmo. Quer criá-lo, mas deseja mantê-lo para você/sua empresa e não compartilhar? Isso nos deixou tristes, mas vá em frente, basta criá-lo e colocá-lo na sua pasta de plugins.

Como funciona? Você escolhe um componente do menu, arrasta e solta no canvas ou dentro de outro componente, edita suas propriedades e constrói seu layout completo. Quando terminar, escolha a saída desejada (stack) e o idioma (localização) no menu com as opções disponíveis atualmente e pressione o botão "Gerar". O OmniTemplate processará o template e gerará o código correspondente para você. Quer integrar o OmniTemplate no Visual Studio ou Visual Studio Code? Há uma extensão disponível para isso.

# Visão Geral da Arquitetura do Projeto
Este projeto implementa um pipeline modular de geração de código baseado em uma AST JSON estruturada. O objetivo é separar as preocupações e permitir que colaboradores adicionem facilmente suporte para novos idiomas e localizações. O pipeline é dividido em quatro camadas principais:

## 1. Esqueleto (AST em JSON):
Um arquivo JSON estruturado que define a arquitetura genérica do componente. Este arquivo serve diretamente como a Árvore de Sintaxe Abstrata (AST) e preserva a hierarquia e ordem dos elementos.

- **Estrutura:**
O esqueleto é organizado como uma árvore com nós que possuem um tipo, filhos e atributos opcionais. Por exemplo:

```json
    {
      "type": "component",
      "children": [
        {
          "type": "form",
          "children": [
            { "type": "title", "key": "formTitle" },
            {
              "type": "field",
              "children": [
                { "type": "label", "key": "field1Label" },
                { "type": "input", "subtype": "text", "key": "field1Input" }
              ]
            }
          ]
        },
        {
          "type": "style",
          "attributes": { "bgColor": "bgColor", "fontColor": "fontColor" },
          "children": []
        },
        {
          "type": "script",
          "attributes": { "onClick": "onClickAction" },
          "children": []
        }
      ]
    }
```

O esqueleto representa a estrutura genérica do seu componente, incluindo seções de marcação, estilo e script. Ele é usado como entrada para o motor de conversão.

## 2. **Dados (JSON Localizado):**
Um arquivo JSON separado contém os valores de dados reais para os marcadores definidos no esqueleto. As propriedades serão editadas em uma janela/formulário adequado.

- **Exemplo de Estrutura (data-pt-BR.json):**

```json
{
  "formTitle": "Formulário de Contato",
  "field1Label": "Nome",
  "field1Input": "Digite seu nome",
  "bgColor": "#ffffff",
  "fontColor": "#000000",
  "onClickAction": "handleClick"
}
```

Esta camada permite a fácil localização e personalização do conteúdo do componente sem alterar a estrutura.

## 3. **Regras de Conversão:**
Um único arquivo JSON contém as regras de conversão que transformam o esqueleto genérico no código final para o idioma de destino (por exemplo, HTML, XAML/MAUI, Flutter etc.).

- **Exemplo de Estrutura:**

```json
{
  "rules": [
    {
      "name": "convertTitle",
      "nodeType": "title",
      "type": "static",
      "replacement": "<h1>{{formTitle}}</h1>"
    },
    {
      "name": "convertLabel",
      "nodeType": "label",
      "type": "static",
      "replacement": "<label>{{field1Label}}</label>"
    },
    {
      "name": "convertInput",
      "nodeType": "input",
      "type": "static",
      "replacement": "<input type=\"{{subtype}}\" placeholder=\"{{field1Input}}\" />"
    },
    {
      "name": "convertStyle",
      "nodeType": "style",
      "type": "static",
      "replacement": "<style>body { background-color: {{bgColor}}; color: {{fontColor}}; }</style>"
    },
    {
      "name": "convertOnClick",
      "nodeType": "script",
      "attribute": "onClick",
      "type": "js",
      "functionBody": "return 'onclick=\"' + group1.trim() + '()\"';"
    }
  ]
}
```

Este arquivo define como cada elemento no esqueleto deve ser convertido. Cada regra inclui um padrão regex para corresponder partes da AST e uma substituição que pode ser uma string estática ou uma função JavaScript (envolvida em um ambiente seguro) para produzir saída dinâmica. Estou considerando fazer algum tipo de motor de conversão com arquitetura de plugins para permitir apenas um subconjunto pré-aprovado de funções para tornar as conversões seguras, mas não quero torná-lo uma abordagem mais fechada do que o necessário.

##### Nota sobre Segurança:
Para regras baseadas em JavaScript, a comunidade e eu trabalharemos juntos para evitar a execução arbitrária de código. Portanto, para tornar seu componente à prova de futuro, use-o **apenas quando necessário** e se tiver sugestões sobre como evitar que se torne uma grande bagunça, discuta no fórum ou com issues/pull requests.

## **Localização:**
Arquivos que fornecem strings localizadas em diferentes idiomas (por exemplo, data-en.json para inglês, data-ptBR.json para português).

Esta camada garante que o código gerado possa suportar vários idiomas, trocando o arquivo de dados sem afetar a estrutura do componente ou as regras de conversão.

#### Como Funciona
**1. Entrada do Esqueleto:**
O motor de conversão recebe o JSON (esqueleto) como entrada, que já contém a estrutura hierárquica do componente.

**2. Processo de Conversão:**
O motor percorre o esqueleto, aplicando as regras de conversão definidas no arquivo de regras.

- Para cada nó, o motor verifica o tipo e aplica a regra estática ou dinâmica (baseada em JS) correspondente.
- Se uma regra for do tipo "static", ela simplesmente substitui o conteúdo correspondente usando a string de substituição fornecida.
- Se uma regra for do tipo "js", o motor converte a string functionBody em uma função e a chama com os grupos de correspondência apropriados (por exemplo, group1).

**3. Localização e Saída Final:**
Após a conversão, o template de saída (que ainda pode conter marcadores) é processado com um motor de template (como Handlebars) usando o JSON de dados localizado, gerando o código final de saída para o idioma de destino.

## Como Contribuir:

Faça um fork do repositório e crie um novo branch. Quando terminar de desenvolver e testar, faça um pull request. Sempre garanta que suas alterações estejam bem documentadas e testadas. Abaixo estão algumas diretrizes mais específicas para cada tipo de contribuição:

- **Desenvolvendo o próprio sistema:**
Isso será desenvolvido em .NET MAUI pelo único motivo de que eu quero que seja compatível com desktop em qualquer sistema operacional e prefiro trabalhar com C# .NET do que com outras linguagens de programação. Os Windows Forms são muito feios e não funcionarão corretamente com Linux/Mac, então...

- **Adicionando Novas Regras de Conversão:**
 - Adicione ou modifique regras no arquivo JSON de regras de conversão para seu idioma/sistema alvo.
 
 Se sua contribuição for extremamente específica para sua empresa/negócio/aplicação, você pode simplesmente gerar os arquivos para a pasta de plugins e, se estiver seguindo as diretrizes corretamente, funcionará imediatamente, mas não será incluído neste repositório.

- **Estendendo os Componentes Criando novos Esqueletos:**
 - Adicione novos tipos de componentes, seguindo o esquema original.
 
- **Contribuições de Localização:**
 - Adicione novos arquivos de localização (por exemplo, data-es.json para espanhol) seguindo a mesma estrutura de campos do data-en.json padrão e também a mesma estrutura de pastas para cada componente.
 - Garanta que todas as chaves sejam traduzidas de forma consistente.

### Diretrizes Gerais:

- Mantenha as alterações modulares. Cada camada (Esqueleto, Dados, Regras de Conversão, Localização) deve ser independente.
- Siga os padrões de codificação descritos em nosso CONTRIBUTING.md.
- Teste suas alterações gerando saídas de exemplo e verificando se o código final se comporta conforme esperado no ambiente de destino.

Ao seguir esta arquitetura, nosso projeto permanece flexível, seguro e facilmente extensível. Os colaboradores podem trabalhar em camadas específicas sem interferir em outras, permitindo a rápida expansão do suporte para novos idiomas-alvo e conteúdo localizado.

Sinta-se à vontade para entrar em contato com perguntas ou sugestões através de nossos Issues ou fóruns de discussão do GitHub.

Bom código!

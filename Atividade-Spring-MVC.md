# Atividade Spring MVC

## Objetivos

* Montar um projeto Spring Boot básico através do Spring Initializr (https://start.spring.io/)
* Abrir projeto no Netbeans ou outra IDE de preferência
* Criar um arquivo HTML estático
* Criar um Controller + template dinâmico

## Requisitos

* Java JDK 17 instalado
* Netbeans 14

## Montar projeto no Spring Initializr

* Usar as seguintes configurações:
    * Project: __Maven Project__
    * Language: __Java__
    * Spring Boot: __2.7.3__ (ou 2.7 mais recente)
    * Project Metadata
        * Group: __br.senac.tads.dsw__
        * Artifact: __exemplo__
        * Description: __Primeiro exemplo Spring Boot__
        * Packaging: __Jar__
        * Java: __17__ (OU A VERSÃO DO JDK INSTALADA NA MÁQUINA)
    * Dependencies - Adicionar as seguintes depenências clicando no botão "Add Dependencies"
        * Spring Boot Devtools
        * Lombok
        * Spring Configuration Processor
        * Spring Web
        * Thymeleaf

* Link para acessar o Initializr com as opções acima selecionadas: https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.7.3&packaging=jar&jvmVersion=17&groupId=br.senac.tads.dsw&artifactId=exemplo&name=exemplo&description=Primeiro%20exemplo%20Spring%20Boot&packageName=br.senac.tads.dsw.exemplo&dependencies=devtools,lombok,configuration-processor,web,thymeleaf
* Após montar o projeto, clicar em "Generate" para fazer o download do .zip com o projeto

## Abrir projeto no Netbeans

* Descompactar o .zip em algum diretório adequado
* Abrir o Netbeans e abrir o projeto, apontando para o diretório onde o .zip foi descompactado
* Após abrir, clicar com botão direito do mouse em cima do nome do projeto e no menu selecionar a opção "Clean and Build"

## Atividade 1 - Criar HTML + CSS estático

1. Em `Other Sources > src/main/resources > static, clicar com botão direito do mouse na pasta "static" e selecionar opções "New > Other"
1. Na janela, selecinar "Other" do lado Categories e "HTML File" do lado File Types e clicar em "Next"
1. Na próxima tela, colocar em File name o nome "index.html" e clicar em Finish
1. No arquivo HTML criado, substituir todo o texto pelo código abaixo:

    ```html
    <!DOCTYPE html>
    <html>
        <head>
            <title>Exemplo Spring Boot - Página estática</title>
            <meta charset="UTF-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1.0" />
            <link rel="stylesheet" href="estilos.css" />
        </head>
        <body>
            <h1>Exemplo página estática</h1>
            <p class="data-hora">Data e hora: <span>01/01/2022 00:00</span></p>
        </body>
    </html>
    ```

1. Em `Other Sources > src/main/resources > static, clicar com botão direito do mouse na pasta "static" e selecionar opções "New > Other"
1. Na janela, selecinar "Other" do lado Categories e "Cacading Style Sheet" do lado File Types e clicar em "Next"
1. Na próxima tela, colocar em File name o nome "index.html" e clicar em Finish
1. No arquivo CSS criado, substituir todo o texto pelo código abaixo

    ```css
    @charset "utf-8";

    body {
        font-size: 16px;
        color: #2c3e50;
        background: rgb(156,150,255);
        background: linear-gradient(90deg, rgba(156,150,255,1) 0%, rgba(255,226,165,1) 100%);
    }

    h1 {
        font-size: 2rem;
        color: #8e44ad;
    }

    p.data-hora {
        font-size: 1.5rem;
    }
    ```

1. Para rodar o projeto e ver as mudanças, expandir a pasta "Source Package" > br.senac.tads.dsw.exemplo" e ver o arquivo "ExemploApplication.java"
1. Clicar com botão direito sobre este arquivo e selecionar opção "Run file"
1. Acompanhar na aba "Output" os logs de inicialização e ver se a aplicação iniciou corretamente. Mo final, devem aparecer mensagens parecidas com as ilustradas abaixo indicando que a aplicação iniciou corretamente:

    ```
    2022-01-21 00:00:00.000  INFO 12345 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
    2022-01-21 00:00:00.000  INFO 12345 --- [  restartedMain] b.s.tads.dsw.exemplo.ExemploApplication  : Started ExemploApplication in 2.99 seconds (JVM running for 2.415)
    ```
    
1. Abrir o navegador, acessar a URL: `http://localhost:8080` e ver o resultado
1. Depois que ver os resultados, parar a aplicação clicando no botão "Stop" na aba do Output

### Questões

1. Explique o significado de uma página estática, pensando em uma forma de alterar a data e hora SEM o uso de Javascript.

## Atividade 2 - Criar Controller + Template dinâmico

1. Em "Source Packages > br.senac.tads.dsw.exemplo", clicar com o botão direito do mouse sobre o package e selecionar a opção "New > Java class"
1. Na janela, preencher o Class name com o valor "ExemploController" e clicar em "Finish"
1. Substituir todo o conteúdo da classe pelo código abaixo:
    ```java
    package br.senac.tads.dsw.exemplo;

    import java.time.LocalDateTime;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.servlet.ModelAndView;

    @Controller
    @RequestMapping("/exemplo-dinamico")
    public class ExemploController {

        @GetMapping
        public ModelAndView gerarInfos() {
            ModelAndView mv = new ModelAndView("index-dinamico");
            mv.addObject("titulo", "Exemplo página dinâmica - Texto criado no Controller");
            mv.addObject("dataHora", LocalDateTime.now());
            return mv;
        }

    }
    ```

1. Em `Other Sources > src/main/resources > templates, clicar com botão direito do mouse na pasta "static" e selecionar opções "New > Other"
1. Na janela, selecinar "Other" do lado Categories e "HTML File" do lado File Types e clicar em "Next"
1. Na próxima tela, colocar em File name o nome "index-dinamico.html" e clicar em Finish
1. No arquivo HTML criado, substituir todo o texto pelo código abaixo (alguns erros de sintaxe apontados pelo Netbeans podem ser ignorados):

    ```html
    <!DOCTYPE html>
    <html>
        <head>
            <title>Exemplo Spring Boot - Página dinâmica</title>
            <meta charset="UTF-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1.0" />
            <link rel="stylesheet" href="estilos.css" />
        </head>
        <body>
            <h1 th:text="${titulo}">Exemplo página dinâmica</h1>
            <p class="data-hora">Data e hora: <span th:text="${dataHora}">01/01/2022 00:00</span></p>
        </body>
    </html>
    ```
      
1. Rodar a aplicação seguindo os passos da atividade anterior
1. Abrir o navegador, acessar a URL: `http://localhost:8080/exemplo-dinamico` e ver o resultado
1. Atualizar a tela algumas vezes para verificar a data e hora.
1. Depois que ver os resultados, parar a aplicação clicando no botão "Stop" na aba do Output

### Questões e testes adicionais

1. Abrir o arquivo `index-dinamico.html` diretamente no navegador. O que acontece?
1. No arquivo `index-dinamico.html` criado, remover os textos das tags `<h1>` e `<span>`, para deixá-las vazias. O que acontece ao rodar o projeto e acessar pelo navegador
1. Na classe `ExemploController.java`, como a anotação `@RequestMapping("/exemplo-dinamico")` afeta o funcionamento da aplicação?
1. Qual a relação que existe entre a palavra "titulo" que aparece no Controler em `mv.addObject("titulo", "Exemplo página dinâmica - Texto criado no Controller");` e no template em `<h1 th:text="${titulo}">Exemplo página dinâmica</h1>`

## Problemas comuns no Netbeans

Algumas mensagens apresentadas aqui podem ser verificas na aba "Output" quando a aplicação é executada

1. Problema de porta 8080 sendo usada (mensagem `Web server failed to start. Port 8080 was already in use.`)
    * Verificar se a aplicação já não está rodando. Se estiver, pode parar a aplicação primeiro
    * No arquivo `application.properties`, incluir a configuração `server.port=8084` (ou qualquer outra porta válida)
    
1. Aba "Output" não aparece ao rodar a aplicação
    * Ir no menu "Window > Output" para forçar a exibição
    

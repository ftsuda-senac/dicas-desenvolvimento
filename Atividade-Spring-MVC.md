# Atividade Spring MVC

## Objetivos

* Montar um projeto Spring Boot básico através do Spring Initializr (https://start.spring.io/)
* Abrir projeto no Netbeans ou outra IDE de preferência
* Criar um arquivo HTML estático
* Criar um Controller + template dinâmico

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


![Uploading image.png…]()

## Abrir projeto no Netbeans


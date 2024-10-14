# Documentação da Biblioteca Testcontainers

## Introdução

Testcontainers é uma biblioteca Java que permite a criação e gerenciamento de containers Docker diretamente nos testes de integração. Ela facilita o teste de aplicações que dependem de outros serviços, como bancos de dados, filas de mensagens (RabbitMQ), servidores de e-mail (MailHog), ou outros componentes de infraestrutura, garantindo um ambiente consistente e isolado.

A biblioteca Testcontainers é particularmente útil para testar integrações de forma rápida e segura, sem a necessidade de configurar manualmente ambientes complexos. Isso garante que os testes possam ser realizados de forma reprodutível, independentemente do ambiente em que estejam sendo executados.

## Características Principais

- **Ambiente Isolado**: Criação de containers dedicados para cada teste, evitando interferências.
- **Reprodutibilidade**: Permite que todos os testes usem um ambiente configurado da mesma forma.
- **Integração Simples**: Pode ser utilizado facilmente em projetos que usam frameworks como Spring Boot, JUnit, etc.
- **Amplo Suporte a Tecnologias**: Suporte a bancos de dados (PostgreSQL, MySQL, etc.), sistemas de mensageria (RabbitMQ, Kafka), e outros serviços (Redis, Elasticsearch, etc.).

## Instalação

Para usar o Testcontainers, você precisa adicionar a dependência ao seu projeto. Caso esteja usando Maven, adicione o seguinte trecho ao seu `pom.xml`:

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.20.1</version>
</dependency>
```

Além disso, é necessário ter o Docker instalado e em execução na máquina onde os testes serão realizados.

## Documentação Oficial do Testcontainers

https\://testcontainers.com/

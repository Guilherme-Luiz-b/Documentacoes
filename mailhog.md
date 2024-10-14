# Documentação do MailHog com Testcontainers

## Introdução

O MailHog é um servidor SMTP de desenvolvimento que captura e-mails enviados por uma aplicação, permitindo que os desenvolvedores visualizem e analisem essas mensagens através de uma interface web ou API. Integrar o MailHog em testes automatizados garante que os fluxos de envio de e-mail da sua aplicação estejam funcionando corretamente sem a necessidade de enviar e-mails reais.

O Testcontainers é uma biblioteca Java que facilita o uso de contêineres Docker em testes de unidade e integração. Com ele, é possível iniciar um contêiner do MailHog durante os testes, proporcionando um ambiente isolado e consistente para verificar o envio de e-mails.

## Pré-requisitos

- **Java 8** ou superior
- **JUnit 5** para testes
- **Docker** instalado e em execução no ambiente onde os testes serão executados.
- **Testcontainers** na versão mais recente
- **MailHog** para capturar e visualizar e-mails

## Dependências

Certifique-se de adicionar as dependências ao seu arquivo `pom.xml` (se estiver usando Maven):

```xml
<!-- Dependência do Testcontainers -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.20.1</version>
    <scope>test</scope>
</dependency>

<!-- Dependência para o JavaMail API -->
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.6.0</version>
    <scope>test</scope>
</dependency>
```

## Integração com o Projeto

### Iniciando o MailHog com Testcontainers

Como o Testcontainers não possui um módulo específico para o MailHog, usaremos o GenericContainer para iniciar um contêiner com a imagem Docker oficial do MailHog.

Abaixo está um exemplo de integração com o MailHog usando Testcontainers em Java no orbeagro:

```java
package br.chronos.service.requisicaocompraservice;

import br.chronos.automacao.service.IntegracaoService;
import br.chronos.security.repository.UserRepository;
import br.chronos.service.RequisicaoCompraService;
import br.chronos.utils.MockIntegrationUtils;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.transaction.annotation.Transactional;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.containers.wait.strategy.Wait;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;

@Testcontainers
@ActiveProfiles("test")
@SpringBootTest
@Transactional
class SendEmailCentroCustoMailhogTest {
    private static final Integer mailhogPort1025 = 1025;
    private static final Integer mailhogPort8025 = 8025;

    @Container
    public static GenericContainer<?> mailhog = new GenericContainer<>("mailhog/mailhog")
            .withExposedPorts(mailhogPort1025, mailhogPort8025)
            .waitingFor(Wait.defaultWaitStrategy());

    @Autowired
    private RequisicaoCompraService requisicaoCompraservice;

    @MockBean
    private UserRepository userRepository;

    @MockBean
    private IntegracaoService integracaoService;

    @MockBean
    private ConnectionFactory connectionFactory;

    @BeforeAll
    static void setUpAll() {
        mailhog.start();
    }

    @AfterAll
    static void tearDown() {
        if (mailhog != null) mailhog.stop();
    }

    @Test
    void testSendEmailCentroCusto() {
        Long requisicaoId = 1L;
        Set<Long> centros = new HashSet<>(Set.of(1L));
        when(userRepository.findEmailsByFuncionalidadeAndCentrosCusto(any(), any())).thenReturn(new ArrayList<>(List.of("test@mailhog.com")));
        when(integracaoService.isProducao()).thenReturn(false);

        configureMockIntegrations();

        requisicaoCompraservice.sendEmailCentroCusto(requisicaoId, centros);
    }

    private void configureMockIntegrations() {
        String mailhogPort = mailhog.getMappedPort(mailhogPort1025).toString();
        String mailhogHost = mailhog.getHost();
        String mailhogUsernameEmail = "test@mailhog.com";
        String mailhogPasswordEmail = "123456";
        MockIntegrationUtils.configureMockIntegrations(integracaoService, mailhogPort, mailhogHost, mailhogUsernameEmail, mailhogPasswordEmail);
    }
}
```



```java
package br.chronos.utils;

import br.chronos.automacao.entity.Integracao;
import br.chronos.automacao.enums.TipoParametro;
import br.chronos.automacao.service.IntegracaoService;
import org.springframework.context.annotation.Profile;

import static org.mockito.Mockito.*;

@Profile("test")
public class MockIntegrationUtils {

    public static void configureMockIntegrations(IntegracaoService integracao, String mailhogPort, String mailhogHost, String mailhogUsernameEmail, String mailhogPasswordEmail) {
        Integracao mockIntegracaoPortEmail = mock(Integracao.class);
        when(mockIntegracaoPortEmail.getInformacao()).thenReturn(mailhogPort);
        when(integracao.findByTipo(TipoParametro.PORT_EMAIL)).thenReturn(mockIntegracaoPortEmail);

        Integracao mockIntegracaoHostEmail = mock(Integracao.class);
        when(mockIntegracaoHostEmail.getInformacao()).thenReturn(mailhogHost);
        when(integracao.findByTipo(TipoParametro.HOST_EMAIL)).thenReturn(mockIntegracaoHostEmail);

        Integracao mockIntegracaoUsernameEmail = mock(Integracao.class);
        when(mockIntegracaoUsernameEmail.getInformacao()).thenReturn(mailhogUsernameEmail);
        when(integracao.findByTipo(TipoParametro.USERNAME_EMAIL)).thenReturn(mockIntegracaoUsernameEmail);

        Integracao mockIntegracaoPasswordEmail = mock(Integracao.class);
        when(mockIntegracaoPasswordEmail.getInformacao()).thenReturn(mailhogPasswordEmail);
        when(integracao.findByTipo(TipoParametro.PASSWORD_EMAIL)).thenReturn(mockIntegracaoPasswordEmail);
    }
}
```

## Explicação Detalhada

### Visão Geral

O código apresentado é um teste de integração utilizando o framework Spring Boot Test, com o auxílio da biblioteca Testcontainers para gerenciar um contêiner do MailHog. O objetivo do teste é verificar o envio de e-mails para um centro de custo específico na aplicação, garantindo que a funcionalidade de envio esteja operando corretamente em um ambiente controlado.

### Configuração do Contêiner MailHog

```java
@Container
public static GenericContainer<?> mailhog = new GenericContainer<>("mailhog/mailhog")
        .withExposedPorts(mailhogPort1025, mailhogPort8025)
        .waitingFor(Wait.defaultWaitStrategy());
```

* `@Container`: Anotação do Testcontainers que indica que este contêiner deve ser gerenciado automaticamente.
* `GenericContainer`: Classe genérica do Testcontainers para contêineres Docker não suportados diretamente.
* `"mailhog/mailhog"`: Imagem Docker oficial do MailHog.
* `withExposedPorts(...)`: Expõe as portas necessárias para comunicação com o MailHog.
* `waitingFor(Wait.defaultWaitStrategy())`: Define uma estratégia padrão de espera até que o contêiner esteja pronto para uso.

### Configuração das Integrações:

* Chama o método `configureMockIntegrations()` para ajustar as configurações do `integracaoService` com os detalhes do contêiner do MailHog.

### Método Auxiliar: `configureMockIntegrations`

```java
private void configureMockIntegrations() {
    String mailhogPort = mailhog.getMappedPort(mailhogPort1025).toString();
    String mailhogHost = mailhog.getHost();
    String mailhogUsernameEmail = "test@mailhog.com";
    String mailhogPasswordEmail = "123456";
    MockIntegrationUtils.configureMockIntegrations(integracaoService, mailhogPort, mailhogHost, mailhogUsernameEmail, mailhogPasswordEmail);
}
```

#### Obtenção das Informações do Contêiner MailHog:

* `mailhog.getMappedPort(mailhogPort1025)`: Obtém a porta mapeada no host para a porta SMTP (1025) do contêiner.
* `mailhog.getHost()`: Obtém o endereço do host onde o contêiner está sendo executado (normalmente localhost).

#### Definição das Credenciais de E-mail:

* `mailhogUsernameEmail`: Define um e-mail de usuário fictício para autenticação (se necessário).
* `mailhogPasswordEmail`: Define uma senha fictícia.

#### Configuração do Serviço de Integração:

* `MockIntegrationUtils.configureMockIntegrations(...)`

## Detalhes da classe `MockIntegrationUtils`:

#### O método configureMockIntegrations recebe como parâmetros:

* `IntegracaoService integracao`: O serviço de integração que será mockado.
* Strings contendo os valores para:
    * `mailhogPort`: Porta SMTP do MailHog.
    * `mailhogHost`: Endereço do host do MailHog.
    * `mailhogUsernameEmail`: Nome de usuário para autenticação no servidor SMTP (se aplicável).
    * `mailhogPasswordEmail`: Senha para autenticação no servidor SMTP (se aplicável).

#### Mock de `Integracao` para a Porta de E-mail:

* Cria um mock da classe `Integracao`.
* Configura o método `getInformacao()` para retornar o valor de `mailhogPort`.
* Configura o `integracaoService` para retornar este mock quando `findByTipo(TipoParametro.PORT_EMAIL)` for chamado.

#### Mock de `Integracao` para o Host de E-mail:

* Segue o mesmo padrão, mas para o `mailhogHost` e `TipoParametro.HOST_EMAIL`.

#### Mock de `Integracao` para o Nome de Usuário de E-mail:

* Cria e configura o mock para retornar `mailhogUsernameEmail` quando solicitado com `TipoParametro.USERNAME_EMAIL`.

#### Mock de `Integracao` para a Senha de E-mail:

* Cria e configura o mock para retornar `mailhogPasswordEmail` quando solicitado com `TipoParametro.PASSWORD_EMAIL`.

#### Objetivo

Ao configurar o `integracaoService` para retornar objetos `Integracao` mockados com os valores fornecidos, qualquer componente que dependa deste serviço receberá os parâmetros especificados, permitindo que o teste controle completamente o ambiente de execução.

## Uso Prático

Após a execução do teste acima um container docker é criado:

![containers](https://github.com/Guilherme-Luiz-b/imagens/blob/main/containers_criados.png)

Ao acessar localhost:{porta} a caixa de entrada do mailhog estará vazia:

![caixa de entrada vazia](https://github.com/Guilherme-Luiz-b/imagens/blob/main/mailhog_caixadeentrada_vazia.png)

Durante a execução do teste um email será enviado e cairá na caixa de entrada do mailhog:

![caixa de entrada com email](https://github.com/Guilherme-Luiz-b/imagens/blob/main/caixadeentrada_comemail.png)

Agora é possível ver o email que foi enviado durante os testes:

![email recebido](https://github.com/Guilherme-Luiz-b/imagens/blob/main/email_recebido.png)

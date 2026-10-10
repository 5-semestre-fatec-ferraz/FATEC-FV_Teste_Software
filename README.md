* **FATEC-FV Faculdade de Tecnologia de Ferraz de Vasconcelos**
* **ADS - Análise e Desenvolvimento de Sistemas**
* **Teste de Software**
* **Professor:** Alexander Bastos
* **Grupos:**
---

# Preparando o Ambiente Java para Testes BDD com Gherkin

Este guia orienta detalhadamente como preparar o ambiente **Java** para executar testes BDD automatizados a partir do arquivo de especificação [login.feature](file:///var/www/html/teste-software/Grupo/features/login.feature).

---

## 📌 Visão Geral do Cenário de Teste

O arquivo [login.feature](file:///var/www/html/teste-software/Grupo/features/login.feature) abrange a funcionalidade **"Autenticação de Usuários e Gestão de Sessão de Login"**, contemplando os seguintes cenários:
1. **Login com sucesso** (`@sucesso`, `@login_valido`)
2. **Tentativa com senha incorreta** (`@falha`, `@seguranca`, `@anti_enumeracao`)
3. **Tentativa com e-mail não cadastrado** (`@falha`, `@seguranca`, `@anti_enumeracao`)
4. **Tentativa com conta desativada** (`@falha`, `@conta_inativa`)
5. **Tentativa de envio com campos obrigatórios vazios** (`@falha`, `@validacao_campos`)
6. **Encerramento de sessão (Logout) com sucesso** (`@logout`, `@sessao`)

---

## 🛠️ 1. Pré-requisitos de Ambiente

| Componente | Versão Recomendada | Verificação no Terminal |
| :--- | :--- | :--- |
| **Java JDK** | **21 LTS** (OpenJDK / Eclipse Temurin) | `java -version` |
| **Build Tool** | **Apache Maven 3.9+** | `mvn -version` |
| **Navegador** | Google Chrome / Chromium | `google-chrome --version` ou `chromium --version` |

---

## 📦 2. Estrutura Padrão de Diretórios (Maven Standard Directory Layout)

```text
├── pom.xml                                   # Configuração de dependências e plugins Maven
├── features/
│   └── login.feature                         # Especificação BDD Gherkin (pt)
└── src/
    └── test/
        ├── java/
        │   └── com/fatec/login/
        │       ├── runner/
        │       │   └── CucumberTestRunner.java  # Suite de execução JUnit Platform
        │       └── steps/
        │           └── LoginSteps.java          # Step Definitions em Java com Playwright
        └── resources/
            └── cucumber.properties              # Propriedades de relatório e plugins
```

---

## ⚙️ 3. Configuração do `pom.xml` (Dependências Maven)

Crie ou atualize o arquivo `pom.xml` na raiz do projeto com as dependências do **Cucumber 7+**, **JUnit 5 Jupiter Platform** e **Playwright Java**:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>br.gov.sp.fatec</groupId>
    <artifactId>teste-login-bdd</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <cucumber.version>7.18.1</cucumber.version>
        <junit.jupiter.version>5.10.3</junit.jupiter.version>
        <playwright.version>1.46.0</playwright.version>
        <assertj.version>3.26.3</assertj.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.cucumber</groupId>
                <artifactId>cucumber-bom</artifactId>
                <version>${cucumber.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.junit</groupId>
                <artifactId>junit-bom</artifactId>
                <version>${junit.jupiter.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- Cucumber Java & JUnit Platform Engine -->
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit-platform-engine</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- JUnit 5 Suite Runner -->
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-suite</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Playwright Java para automação Web moderna -->
        <dependency>
            <groupId>com.microsoft.playwright</groupId>
            <artifactId>playwright</artifactId>
            <version>${playwright.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Fluent Assertions -->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>${assertj.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.3.1</version>
                <configuration>
                    <properties>
                        <configurationParameters>
                            cucumber.features=features
                            cucumber.glue=com.fatec.login.steps
                            cucumber.plugin=pretty, html:target/cucumber-reports/cucumber.html, json:target/cucumber-reports/cucumber.json
                        </configurationParameters>
                    </properties>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## ☕ 4. Implementação em Java

### 4.1. Runner JUnit 5 (`src/test/java/com/fatec/login/runner/CucumberTestRunner.java`)

```java
package com.fatec.login.runner;

import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.IncludeEngines;
import org.junit.platform.suite.api.SelectClasspathResource;
import org.junit.platform.suite.api.Suite;

import static io.cucumber.junit.platform.engine.Constants.FEATURES_PROPERTY_NAME;
import static io.cucumber.junit.platform.engine.Constants.GLUE_PROPERTY_NAME;
import static io.cucumber.junit.platform.engine.Constants.PLUGIN_PROPERTY_NAME;

@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(key = FEATURES_PROPERTY_NAME, value = "features")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.fatec.login.steps")
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, value = "pretty, html:target/cucumber-reports/cucumber.html")
public class CucumberTestRunner {
    // Orquestrador de execução de cenários BDD
}
```

### 4.2. Step Definitions (`src/test/java/com/fatec/login/steps/LoginSteps.java`)

Implementação que vincula as anotações Gherkin em português (`@Dado`, `@Quando`, `@E`, `@Então`) aos seletores e ações no navegador:

```java
package com.fatec.login.steps;

import com.microsoft.playwright.*;
import io.cucumber.java.After;
import io.cucumber.java.Before;
import io.cucumber.java.pt.Dado;
import io.cucumber.java.pt.E;
import io.cucumber.java.pt.Então;
import io.cucumber.java.pt.Quando;

import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;

public class LoginSteps {

    private Playwright playwright;
    private Browser browser;
    private Page page;

    @Before
    public void setup() {
        playwright = Playwright.create();
        boolean headless = !"false".equalsIgnoreCase(System.getenv("TESTE_HEADLESS"));
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(headless));
        page = browser.newPage();
    }

    @After
    public void teardown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
    }

    @Dado("que o usuário está na página de login")
    public void queOUsuarioEstaNaPaginaDeLogin() {
        String baseUrl = System.getenv("TEST_BASE_URL");
        if (baseUrl == null || baseUrl.isBlank()) {
            baseUrl = "http://localhost:8080/login.html";
        }
        page.navigate(baseUrl);
        assertThat(page.getByTestId("input-email")).isVisible();
    }

    @Quando("o usuário preenche o e-mail com {string}")
    public void oUsuarioPreencheOEmailCom(String email) {
        page.getByTestId("input-email").fill(email);
    }

    @E("o usuário preenche a senha com {string}")
    public void oUsuarioPreencheASenhaCom(String senha) {
        page.getByTestId("input-password").fill(senha);
    }

    @Quando("o usuário deixa os campos de e-mail e senha vazios")
    public void oUsuarioDeixaOsCamposDeEmailESenhaVazios() {
        page.getByTestId("input-email").fill("");
        page.getByTestId("input-password").fill("");
    }

    @E("clica no botão de entrar")
    public void clicaNoBotaoDeEntrar() {
        page.getByTestId("button-submit").click();
    }

    @Quando("o usuário clica no botão de sair")
    public void oUsuarioClicaNoBotaoDeSair() {
        page.getByTestId("button-logout").click();
    }

    @Então("o sistema deve exibir a mensagem {string}")
    @Então("o sistema deve exibir a mensagem de erro {string}")
    @Então("o sistema deve exibir o alerta {string}")
    public void oSistemaDeveExibirAMensagem(String mensagem) {
        Locator feedback = page.getByTestId("feedback-message");
        assertThat(feedback).isVisible();
        assertThat(feedback).hasText(mensagem);
    }

    @E("deve apresentar o painel principal simulado")
    public void deveApresentarOPainelPrincipalSimulado() {
        assertThat(page.getByTestId("dashboard-panel")).isVisible();
        assertThat(page.locator("#login-form-wrapper")).isHidden();
    }

    @Então("o formulário de login deve ser reexibido")
    public void oFormularioDeLoginDeveSerReexibido() {
        assertThat(page.locator("#login-form-wrapper")).isVisible();
        assertThat(page.getByTestId("dashboard-panel")).isHidden();
    }
}
```

---

## 🚀 5. Comandos Rápidos de Linha de Comando (CLI)

Execute os comandos a partir da raiz do projeto:

### Instalar dependências e baixar navegadores Playwright:
```bash
mvn clean compile test-compile
mvn exec:java -e -Dexec.mainClass=com.microsoft.playwright.CLI -Dexec.args="install --with-deps chromium"
```

### Executar todos os testes BDD:
```bash
mvn test
```

### Executar filtrando por Tags do Cucumber:
```bash
# Executar apenas cenários felizes de sucesso:
mvn test -Dcucumber.filter.tags="@sucesso"

# Executar cenários de segurança e anti-enumeração:
mvn test -Dcucumber.filter.tags="@seguranca"

# Executar todos os cenários de falha e validação:
mvn test -Dcucumber.filter.tags="@falha"

# Executar todos exceto logout:
mvn test -Dcucumber.filter.tags="not @logout"
```

### Executar em modo visual (Navegador com interface gráfica aberta):
```bash
TESTE_HEADLESS=false mvn test
```

### Acessar Relatório de Execução BDD:
```bash
xdg-open target/cucumber-reports/cucumber.html
```

---

# 📋 Testes Manuais de Sistema

Roteiro complementar para validação exploratória antes da execução automatizada:

1. **Responsividade & Layout:** Verificar se o formulário de login mantém a proporção e legibilidade em dispositivos móveis (375px), tablets (768px) e monitores widescreen (1920px).
2. **Mascaramento de Credenciais:** Confirmar que o campo de senha oculta os caracteres digitados (`type="password"`) e permite alternância de visibilidade se houver botão de olho.
3. **Prevenção de Cliques Múltiplos:** Validar que disparos rápidos consecutivos no botão "Entrar" desabilitam o botão ou evitam requisições duplicadas.
4. **Acessibilidade Web (a11y):** Testar a navegação sequencial utilizando apenas o teclado (`Tab`, `Shift+Tab`, `Enter`) e verificar o contraste de cores das mensagens de erro e sucesso.

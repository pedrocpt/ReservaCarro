1. Estrutura do Projeto

reserva_carro/
├── pom.xml
├── src/
│   ├── main/
│   │   └── java/
│   │       └── reserva/
│   │           └── ReservaService.java
│   └── test/
│       ├── java/
│       │   └── runner/
│       │       └── ReservaServiceTest.java   
│       │       └── ReservaRun.java   
│       │   └── steps/
│       │       └── ReservaSteps.java          
│       └── resources/
│           └── features/
│               └── reserva.feature                   

2. Implementação TDD
pom.xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.reserva</groupId>
    <artifactId>reserva-carro</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <cucumber.version>7.14.0</cucumber.version>
        <junit.version>5.9.2</junit.version>
    </properties>

    <dependencies>
        <!-- Cucumber -->
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <version>${cucumber.version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit</artifactId>
            <version>${cucumber.version}</version>
            <scope>test</scope>
        </dependency>
        
        <!-- JUnit -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M7</version>
            </plugin>
        </plugins>
    </build>
</project>

Implemantando src/main/java/reserva/AreaCobertura.java

package reserva;
import java.util.Arrays;
import java.util.List;

public class AreaCobertura {
    private static final List<String> CIDADES_COBERTAS = Arrays.asList(
        "Belo Horizonte", 
        "Contagem"
    );
    
    public static boolean cidadeEstaNaAreaDeCobertura(String cidade) {
        return CIDADES_COBERTAS.contains(cidade);
    }
}


Implementando src/main/java/reserva/ReservaService.java

package reserva;

public class ReservaService {
    public String reservar(String cidade, String endereco) {
        if (AreaCobertura.cidadeEstaNaAreaDeCobertura(cidade)) {
            return "Motorista a caminho";
        }
        return "Área fora de cobertura";
    }
}

Implementando src/test/java/ReservaServiceTest.java

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import reserva.ReservaService;

class ReservaServiceTest {
    private final ReservaService reservaService = new ReservaService();

    @Test
    void reservar_deveRetornarMotoristaACaminho_paraBeloHorizonte() {
        String resultado = reservaService.reservar("Belo Horizonte", "Av. Afonso Pena, 1000");
        assertEquals("Motorista a caminho", resultado);
    }

    @Test
    void reservar_deveRetornarMotoristaACaminho_paraContagem() {
        String resultado = reservaService.reservar("Contagem", "Rua São Paulo, 500");
        assertEquals("Motorista a caminho", resultado);
    }

    @Test
    void reservar_deveRetornarAreaForaDeCobertura_paraCidadeNaoCoberta() {
        String resultado = reservaService.reservar("Betim", "Rua das Flores, 200");
        assertEquals("Área fora de cobertura", resultado);
    }
}



3. Implementação BDD com Cucumber
src/test/resources/features/reserva.feature


# language: pt
Funcionalidade: Reserva de carro por aplicativo
  Como passageiro de um aplicativo de transporte
  Quero reservar um carro informando meu local
  Para que eu possa me deslocar com comodidade e segurança

  Cenário: Reserva dentro da área de cobertura - Belo Horizonte
    Dado que estou na cidade "Belo Horizonte"
    E meu endereço é "Av. Afonso Pena, 1000"
    Quando solicito uma reserva de carro
    Então deve retornar a mensagem "Motorista a caminho"

  Cenário: Reserva dentro da área de cobertura - Contagem
    Dado que estou na cidade "Contagem"
    E meu endereço é "Rua São Paulo, 500"
    Quando solicito uma reserva de carro
    Então deve retornar a mensagem "Motorista a caminho"

  Cenário: Reserva fora da área de cobertura
    Dado que estou na cidade "Betim"
    E meu endereço é "Rua das Flores, 200"
    Quando solicito uma reserva de carro
    Então deve retornar a mensagem "Área fora de cobertura"

Implementando src/test/java/steps/ReservaSteps.java

package steps;

import io.cucumber.java.pt.Dado;
import io.cucumber.java.pt.Quando;
import io.cucumber.java.pt.Então;
import static org.junit.jupiter.api.Assertions.*;
import reserva.ReservaService;

public class ReservaSteps {
    private String cidade;
    private String endereco;
    private String mensagemRetorno;
    private final ReservaService reservaService = new ReservaService();

    @Dado("que estou na cidade {string}")
    public void que_estou_na_cidade(String cidade) {
        this.cidade = cidade;
    }

    @Dado("meu endereço é {string}")
    public void meu_endereço_é(String endereco) {
        this.endereco = endereco;
    }

    @Quando("solicito uma reserva de carro")
    public void solicito_uma_reserva_de_carro() {
        this.mensagemRetorno = reservaService.reservar(cidade, endereco);
    }
    
    @Quando("solicito uma reserva de carro")
    public void solicito_uma_reserva_de_carro_sem_typo() {
        this.mensagemRetorno = reservaService.reservar(cidade, endereco);
    }

    @Então("deve retornar a mensagem {string}")
    public void deve_retornar_a_mensagem(String mensagemEsperada) {
        assertEquals(mensagemEsperada, mensagemRetorno);
    }
}

Implementando src/test/java/runner/ReservaRun.java

package runner;

import io.cucumber.junit.Cucumber;
import io.cucumber.junit.CucumberOptions;
import org.junit.runner.RunWith;

@RunWith(Cucumber.class)
@CucumberOptions(
    features = "src/test/resources/features",
    glue = "steps",
    plugin = {"pretty", "html:target/cucumber-reports.html"},
    monochrome = true
)
public class ReservaRun {
}

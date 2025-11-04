🚀 Biblioteca de Jogos

Guia Completo: Desenvolvendo e Implantando uma Biblioteca de Jogos com Spring Boot e Thymeleaf
Nível: Intermediário

Este guia completo foi elaborado para desenvolvedores Java com alguma experiência em Spring Boot que desejam aprofundar seus conhecimentos na criação de aplicações web robustas. Ao final deste tutorial, você terá construído uma aplicação completa de CRUD (Create, Read, Update, Delete) para gerenciar uma biblioteca de jogos, utilizando tecnologias modernas e as melhores práticas do mercado.

1. Visão Geral da Arquitetura e Tecnologias
   Nossa aplicação seguirá uma arquitetura em camadas, padrão em projetos Spring Boot, para garantir a separação de responsabilidades e a manutenibilidade do código.

Diagrama de Arquitetura do Projeto
Este diagrama ilustra a estrutura geral do nosso sistema, desde a interação do usuário até a infraestrutura em contêineres que configuraremos ao final.

Snippet de código

graph TD
subgraph "Ambiente Docker"
direction LR
subgraph "Container: App Spring Boot"
direction TB
A[Controller] --> B[Service]
B --> C[Repository / JPA]
end

        subgraph "Container: Banco de Dados"
            direction TB
            DB[(PostgreSQL)]
        end

        subgraph "Container: Gerenciador de DB"
            direction TB
            PGADMIN([pgAdmin])
        end

        C --> DB
        PGADMIN -.-> DB
    end

    USER[Usuário] -->|Requisição HTTP| A
    A -->|HTML com Thymeleaf| USER

    style USER fill:#f9f,stroke:#333,stroke-width:2px
    style DB fill:#add,stroke:#333,stroke-width:2px
    style PGADMIN fill:#f8d5a2,stroke:#333,stroke-width:2px

Camadas da Aplicação
Camada de Apresentação (View): Utilizaremos o Thymeleaf e Bootstrap para criar páginas HTML dinâmicas, responsivas e com uma sintaxe elegante.

Camada de Controle (Controller): Os Controllers receberão as requisições HTTP, interagindo com a camada de serviço e retornando as respostas adequadas.

Camada de Serviço (Service): A lógica de negócio residirá aqui, orquestrando as operações de CRUD e garantindo a integridade dos dados.

Camada de Acesso a Dados (Repository): O Spring Data JPA simplificará a interação com o banco de dados.

Banco de Dados: Usaremos o H2 (em memória) para desenvolvimento e PostgreSQL para produção.

Linguagem e Framework: Java 21 e Spring Boot 3.

2. Configurando o Ambiente e o Projeto
   Ferramentas: Certifique-se de ter o JDK 21, Maven (ou Gradle) e sua IDE de preferência.

Spring Initializr: Inicie seu projeto no Spring Initializr com as seguintes configurações:

Project: Maven

Language: Java

Spring Boot: 3.3.1 (ou a versão estável mais recente)

Group: br.com.bibliotecajogos

Artifact: bibliotecajogos

Packaging: Jar

Java: 21

Dependencies:

Spring Web

Thymeleaf

Spring Data JPA

H2 Database

PostgreSQL Driver

Spring Boot DevTools (Opcional, mas recomendado)

Após gerar e baixar o projeto, abra-o na sua IDE.

pom.xml
O Spring Initializr gerará um pom.xml similar a este. Note a versão do spring-boot-starter-parent corrigida para uma versão válida.

XML

<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>3.3.1</version>
<relativePath/> </parent>
<groupId>br.com.bibliotecajogos</groupId>
<artifactId>bibliotecajogos</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>bibliotecajogos</name>
<description>Demo project for Spring Boot</description>
<properties>
<java.version>21</java.version>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
<dependencies>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>

    	<dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-devtools</artifactId>
    		<scope>runtime</scope>
    		<optional>true</optional>
    	</dependency>
    	<dependency>
    		<groupId>com.h2database</groupId>
    		<artifactId>h2</artifactId>
    		<scope>runtime</scope>
    	</dependency>
    	<dependency>
    		<groupId>org.postgresql</groupId>
    		<artifactId>postgresql</artifactId>
    		<scope>runtime</scope>
    	</dependency>
    	<dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-test</artifactId>
    		<scope>test</scope>
    	</dependency>
    </dependencies>

    <build>
    	<plugins>
    		<plugin>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-maven-plugin</artifactId>
    		</plugin>
    	</plugins>
    </build>

</project>
3. Estrutura do Projeto
Esta é a estrutura de arquivos final que teremos:

src
└── main
├── java
│ └── br
│ └── com
│ └── bibliotecajogos
│ ├── BibliotecajogosApplication.java
│ ├── config
│ │ └── DataInitializer.java <-- Para popular dados
│ ├── controller
│ │ └── JogoController.java
│ ├── entity
│ │ ├── Categoria.java
│ │ └── Jogo.java
│ ├── repository
│ │ ├── CategoriaRepository.java
│ │ └── JogoRepository.java
│ └── service
│ ├── CategoriaService.java <-- Novo
│ └── JogoService.java
└── resources
├── static/
├── templates
│ ├── formulario-jogo.html
│ ├── jogos.html
│ └── layout.html <-- Template base com Bootstrap
├── application.properties
├── application-dev.properties
└── application-prod.properties 4. Modelagem e Fluxo (Diagramas)
Antes de codificar, vamos entender o modelo de dados e o fluxo de navegação.

Diagrama de Entidade-Relacionamento (ERD)
Snippet de código

erDiagram
CATEGORIA ||--o{ JOGO : "possui"

    CATEGORIA {
        BIGINT id PK "Chave Primária"
        VARCHAR nome "Nome da categoria (ex: RPG, Ação)"
    }

    JOGO {
        BIGINT id PK "Chave Primária"
        VARCHAR titulo "Título do jogo"
        VARCHAR autor "Desenvolvedora ou Autor"
        INT anoPublicacao "Ano de lançamento"
        VARCHAR genero "Gênero específico (ex: RPG de Ação)"
        BOOLEAN finalizado "Indica se o jogo foi concluído"
        VARCHAR urlCapa "URL para a imagem da capa"
        BIGINT categoria_id FK "Chave Estrangeira para Categoria"
    }

Explicação do Modelo:

CATEGORIA: Armazena as categorias gerais (RPG, Ação, etc.).

JOGO: Armazena as informações de cada jogo.

Relacionamento: Um-para-Muitos. Uma CATEGORIA pode ter vários JOGOs, mas um JOGO pertence a apenas uma CATEGORIA. A coluna categoria_id na tabela JOGO cria essa ligação.

Diagrama de Fluxo Acadêmico (Navegador ↔ Servidor)
Este diagrama detalha as requisições HTTP que conectam o navegador do usuário ao nosso servidor Spring Boot.

Snippet de código

graph TD
subgraph "Lado do Cliente (Navegador)"
A["Página: Lista de Jogos<br>(<b>GET</b> /jogos)"]
B["Página: Formulário<br>(<b>GET</b> /jogos/novo ou /jogos/editar/{id})"]
end

    subgraph "Lado do Servidor (Spring Boot)"
        C["Controller processa<br><b>POST</b> /jogos"]
        D["Controller processa<br><b>GET</b> /jogos/excluir/{id}"]
        E["Redirecionamento<br>(HTTP 302 Found)"]
    end

    %% Fluxos de Navegação
    A --"1. Clica em 'Adicionar' ou 'Editar'"--> B

    B --"2. Clica em 'Salvar'"--> C
    C --"3. Salva no Banco de Dados"--> E

    A --"4. Clica em 'Excluir'"--> D
    D --"5. Exclui do Banco de Dados"--> E

    E --"6. Navegador é redirecionado para a lista"--> A

    B --"7. Clica em 'Cancelar' (Link simples)"--> A

    %% Estilos para clareza
    style A fill:#cde4ff,stroke:#333
    style B fill:#e8dff5,stroke:#333
    style C fill:#fff5cc,stroke:#333
    style D fill:#fff5cc,stroke:#333
    style E fill:#d4edda,stroke:#333

5. Backend: Entidades e Repositórios
   Vamos criar as classes que representam nossas tabelas e as interfaces para acessá-las.

Categoria.java

Java

package br.com.bibliotecajogos.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.OneToMany;
import java.util.List;

@Entity
public class Categoria {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToMany(mappedBy = "categoria")
    private List<Jogo> jogos;

    // Getters e Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    public List<Jogo> getJogos() { return jogos; }
    public void setJogos(List<Jogo> jogos) { this.jogos = jogos; }

}
Jogo.java

Java

package br.com.bibliotecajogos.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.ManyToOne;

@Entity
public class Jogo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titulo;
    private String autor;
    private Integer anoPublicacao;
    private String genero;
    private String urlCapa; // Campo para a imagem da capa
    private boolean finalizado = false;

    @ManyToOne
    @JoinColumn(name = "categoria_id") // Esta é a FK
    private Categoria categoria;

    // Getters e Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    public String getAutor() { return autor; }
    public void setAutor(String autor) { this.autor = autor; }
    public Integer getAnoPublicacao() { return anoPublicacao; }
    public void setAnoPublicacao(Integer anoPublicacao) { this.anoPublicacao = anoPublicacao; }
    public String getGenero() { return genero; }
    public void setGenero(String genero) { this.genero = genero; }
    public String getUrlCapa() { return urlCapa; }
    public void setUrlCapa(String urlCapa) { this.urlCapa = urlCapa; }
    public boolean isFinalizado() { return finalizado; }
    public void setFinalizado(boolean finalizado) { this.finalizado = finalizado; }
    public Categoria getCategoria() { return categoria; }
    public void setCategoria(Categoria categoria) { this.categoria = categoria; }

}
CategoriaRepository.java

Java

package br.com.bibliotecajogos.repository;

import br.com.bibliotecajogos.entity.Categoria;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CategoriaRepository extends JpaRepository<Categoria, Long> {
}
JogoRepository.java

Java

package br.com.bibliotecajogos.repository;

import br.com.bibliotecajogos.entity.Jogo;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface JogoRepository extends JpaRepository<Jogo, Long> {

    // Métodos de busca customizados (Query Methods)
    List<Jogo> findByTituloContainingIgnoreCase(String titulo);
    List<Jogo> findByAutorContainingIgnoreCase(String autor);
    List<Jogo> findByGeneroContainingIgnoreCase(String genero);

} 6. Backend: Lógica de Negócio (Services)
A camada de serviço orquestra as operações.

CategoriaService.java (NOVO E NECESSÁRIO)

Nota: Este serviço é crucial para carregar as categorias no formulário.

Java

package br.com.bibliotecajogos.service;

import br.com.bibliotecajogos.entity.Categoria;
import br.com.bibliotecajogos.repository.CategoriaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class CategoriaService {

    @Autowired
    private CategoriaRepository categoriaRepository;

    public List<Categoria> listarTodas() {
        // Retorna todas as categorias, ordenadas por nome
        return categoriaRepository.findAll();
    }

}
JogoService.java

Java

package br.com.bibliotecajogos.service;

import br.com.bibliotecajogos.entity.Jogo;
import br.com.bibliotecajogos.repository.JogoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class JogoService {

    @Autowired
    private JogoRepository jogoRepository;

    public List<Jogo> listarTodos(String sortBy) {
        return jogoRepository.findAll(Sort.by(sortBy));
    }

    public Jogo salvar(Jogo jogo) {
        return jogoRepository.save(jogo);
    }

    public Jogo buscarPorId(Long id) {
        return jogoRepository.findById(id).orElse(null);
    }

    public void excluir(Long id) {
        jogoRepository.deleteById(id);
    }

    public List<Jogo> pesquisar(String termo, String tipoPesquisa) {
        return switch (tipoPesquisa) {
            case "titulo" -> jogoRepository.findByTituloContainingIgnoreCase(termo);
            case "autor" -> jogoRepository.findByAutorContainingIgnoreCase(termo);
            case "genero" -> jogoRepository.findByGeneroContainingIgnoreCase(termo);
            default -> List.of();
        };
    }

} 7. Backend: Controladores (Controllers)
O Controller recebe as requisições web. Preste atenção nas modificações para injetar o CategoriaService e carregar as categorias para o formulário.

JogoController.java (Refatorado)

Java

package br.com.bibliotecajogos.controller;

import br.com.bibliotecajogos.entity.Jogo;
import br.com.bibliotecajogos.service.CategoriaService; // IMPORTADO
import br.com.bibliotecajogos.service.JogoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class JogoController {

    @Autowired
    private JogoService jogoService;

    // INJEÇÃO DO NOVO SERVIÇO
    @Autowired
    private CategoriaService categoriaService;

    @GetMapping("/jogos")
    public String listarJogos(Model model, @RequestParam(defaultValue = "titulo") String sortBy) {
        model.addAttribute("jogos", jogoService.listarTodos(sortBy));
        return "jogos";
    }

    @GetMapping("/jogos/novo")
    public String novoJogoForm(Model model) {
        model.addAttribute("jogo", new Jogo());
        // ADICIONADO: Carrega categorias para o dropdown
        model.addAttribute("categorias", categoriaService.listarTodas());
        return "formulario-jogo";
    }

    @PostMapping("/jogos")
    public String salvarJogo(@ModelAttribute("jogo") Jogo jogo) {
        jogoService.salvar(jogo);
        return "redirect:/jogos";
    }

    @GetMapping("/jogos/editar/{id}")
    public String editarJogoForm(@PathVariable Long id, Model model) {
        model.addAttribute("jogo", jogoService.buscarPorId(id));
        // ADICIONADO: Carrega categorias para o dropdown
        model.addAttribute("categorias", categoriaService.listarTodas());
        return "formulario-jogo";
    }

    @GetMapping("/jogos/excluir/{id}")
    public String excluirJogo(@PathVariable Long id) {
        jogoService.excluir(id);
        return "redirect:/jogos";
    }

    @GetMapping("/jogos/pesquisar")
    public String pesquisarJogos(@RequestParam String termo, @RequestParam String tipo, Model model) {
        model.addAttribute("jogos", jogoService.pesquisar(termo, tipo));
        return "jogos";
    }

} 8. Backend: Ponto de Entrada e Seeding de Dados
Para testar a aplicação, vamos popular o banco com dados iniciais assim que ela subir (apenas no perfil dev).

BibliotecajogosApplication.java

Java

package br.com.bibliotecajogos;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
// Anotações @EntityScan e @EnableJpaRepositories são geralmente desnecessárias
// se as classes estiverem em subpacotes da classe principal.
public class BibliotecajogosApplication {

    public static void main(String[] args) {
    	SpringApplication.run(BibliotecajogosApplication.class, args);
    }

}
DataInitializer.java (Populando o Banco)

Java

package br.com.bibliotecajogos.config;

import br.com.bibliotecajogos.entity.Categoria;
import br.com.bibliotecajogos.entity.Jogo;
import br.com.bibliotecajogos.repository.CategoriaRepository;
import br.com.bibliotecajogos.repository.JogoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Component;

import java.util.Arrays;

@Component
@Profile("dev") // Só executa este bean se o perfil 'dev' estiver ativo
public class DataInitializer implements CommandLineRunner {

    @Autowired
    private JogoRepository jogoRepository;

    @Autowired
    private CategoriaRepository categoriaRepository;

    @Override
    public void run(String... args) throws Exception {
        if (categoriaRepository.count() == 0) {
            System.out.println(">>> Populando banco de dados de desenvolvimento...");

            // Criar Categorias
            Categoria rpg = new Categoria(); rpg.setNome("RPG");
            Categoria acao = new Categoria(); acao.setNome("Ação");
            Categoria estrategia = new Categoria(); estrategia.setNome("Estratégia");
            Categoria aventura = new Categoria(); aventura.setNome("Aventura");
            categoriaRepository.saveAll(Arrays.asList(rpg, acao, estrategia, aventura));

            // Criar Jogos
            Jogo j1 = new Jogo();
            j1.setTitulo("The Witcher 3: Wild Hunt");
            j1.setAutor("CD Projekt Red"); j1.setAnoPublicacao(2015);
            j1.setGenero("RPG de Ação"); j1.setCategoria(rpg); j1.setFinalizado(true);
            j1.setUrlCapa("https://upload.wikimedia.org/wikipedia/pt/thumb/0/06/TW3_Wild_Hunt.png/270px-TW3_Wild_Hunt.png");

            Jogo j2 = new Jogo();
            j2.setTitulo("Red Dead Redemption 2");
            j2.setAutor("Rockstar Games"); j2.setAnoPublicacao(2018);
            j2.setGenero("Ação-Aventura"); j2.setCategoria(acao); j2.setFinalizado(true);
            j2.setUrlCapa("https://upload.wikimedia.org/wikipedia/pt/e/e7/Red_Dead_Redemption_2.png");

            Jogo j3 = new Jogo();
            j3.setTitulo("Baldur's Gate 3");
            j3.setAutor("Larian Studios"); j3.setAnoPublicacao(2023);
            j3.setGenero("RPG"); j3.setCategoria(rpg); j3.setFinalizado(false);
            j3.setUrlCapa("https://upload.wikimedia.org/wikipedia/pt/1/18/Baldur%27s_Gate_III_Larian_Studios_key_art.png");

            Jogo j4 = new Jogo();
            j4.setTitulo("The Legend of Zelda: Tears of the Kingdom");
            j4.setAutor("Nintendo"); j4.setAnoPublicacao(2023);
            j4.setGenero("Ação-Aventura"); j4.setCategoria(aventura); j4.setFinalizado(false);
            j4.setUrlCapa("https://upload.wikimedia.org/wikipedia/pt/2/25/Zelda_TotK_capa.jpg");

            jogoRepository.saveAll(Arrays.asList(j1, j2, j3, j4));
            System.out.println(">>> Banco de dados populado com sucesso!");
        }
    }

} 9. Configuração do Banco de Dados (Properties)
Configuramos perfis do Spring para alternar facilmente entre H2 (desenvolvimento) e PostgreSQL (produção).

src/main/resources/application.properties (Principal)

Properties

spring.application.name=bibliotecajogos

# Define o perfil ativo. Mude para 'prod' para usar o PostgreSQL

spring.profiles.active=dev

# Configurações do H2 (ativas apenas se o perfil 'dev' estiver ligado)

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Recria o banco a cada inicialização no H2

spring.jpa.hibernate.ddl-auto=create-drop
src/main/resources/application-dev.properties (Desenvolvimento)

Properties

# Configurações do H2 em memória

spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop # Garante que o DataInitializer rode
src/main/resources/application-prod.properties (Produção)

Properties

# Configurações do PostgreSQL

spring.datasource.url=jdbc:postgresql://localhost:5432/bibliotecajogos
spring.datasource.username=seu_usuario
spring.datasource.password=sua_senha
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Em produção, 'update' é mais seguro para não perder dados.

# Use 'validate' para apenas checar, ou 'none' se usar migrations (ex: Flyway).

spring.jpa.hibernate.ddl-auto=update 10. Frontend: Interface com Bootstrap e Thymeleaf
Criamos um layout base com Bootstrap e duas páginas que o utilizam. Removemos as versões HTML antigas e sem estilo para evitar confusão.

src/main/resources/templates/layout.html (Template Base)

HTML

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="pt-br" th:fragment="layout(title, content)">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title th:replace="${title}">Biblioteca de Jogos</title>
    <link
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
      crossorigin="anonymous"
    />
  </head>
  <body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
      <div class="container-fluid">
        <a class="navbar-brand" th:href="@{/jogos}">🚀 Biblioteca de Jogos</a>
      </div>
    </nav>

    <main class="container">
      <div th:replace="${content}"></div>
    </main>

    <footer class="container text-center mt-5">
      <p>&copy;  Biblioteca de Jogos 2025</p>
    </footer>

    <script
      src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"
      integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz"
      crossorigin="anonymous"
    ></script>

  </body>
</html>
src/main/resources/templates/jogos.html (Página de Listagem)

HTML

<!DOCTYPE html>
<html
  xmlns:th="http://www.thymeleaf.org"
  th:replace="~{layout :: layout (title=~{::title}, content=~{::section})}"
>
  <head>
    <title>Biblioteca de Jogos</title>
  </head>
  <body>
    <section>
      <div class="d-flex justify-content-between align-items-center mb-4">
        <h1 class="mb-0">Listagem de Jogos</h1>
        <a th:href="@{/jogos/novo}" class="btn btn-primary">Adicionar Novo Jogo</a>
      </div>

      <form th:action="@{/jogos/pesquisar}" method="get" class="mb-4">
        <div class="input-group">
          <input type="text" name="termo" class="form-control" placeholder="Pesquisar..."/>
          <select name="tipo" class="form-select">
            <option value="titulo">Título</option>
            <option value="autor">Autor</option>
            <option value="genero">Gênero</option>
          </select>
          <button type="submit" class="btn btn-outline-secondary">Pesquisar</button>
        </div>
      </form>

      <table class="table table-striped table-hover align-middle">
        <thead class="table-dark">
          <tr>
            <th>Capa</th>
            <th><a class="text-white" th:href="@{/jogos(sortBy='titulo')}">Título</a></th>
            <th><a class="text-white" th:href="@{/jogos(sortBy='autor')}">Autor</a></th>
            <th><a class="text-white" th:href="@{/jogos(sortBy='anoPublicacao')}">Ano</a></th>
            <th>Gênero</th>
            <th>Categoria</th>
            <th>Finalizado</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          <tr th:each="jogo : ${jogos}">
            <td>
              <img
                th:if="${jogo.urlCapa != null and !jogo.urlCapa.isEmpty()}"
                th:src="${jogo.urlCapa}"
                alt="Capa" width="50" class="img-thumbnail"
                onerror="this.onerror=null;this.src='https://via.placeholder.com/50x75.png?text=Capa';"
              />
            </td>
            <td th:text="${jogo.titulo}"></td>
            <td th:text="${jogo.autor}"></td>
            <td th:text="${jogo.anoPublicacao}"></td>
            <td th:text="${jogo.genero}"></td>
            <td th:text="${jogo.categoria?.nome}"></td> <td>
              <span th:if="${jogo.finalizado}" class="badge bg-success">Sim</span>
              <span th:unless="${jogo.finalizado}" class="badge bg-warning text-dark">Não</span>
            </td>
            <td>
              <a th:href="@{/jogos/editar/{id}(id=${jogo.id})}" class="btn btn-sm btn-info">Editar</a>
              <a th:href="@{/jogos/excluir/{id}(id=${jogo.id})}" class="btn btn-sm btn-danger">Excluir</a>
            </td>
          </tr>
        </tbody>
      </table>
    </section>

  </body>
</html>
src/main/resources/templates/formulario-jogo.html (Formulário - CORRIGIDO)

Nota: Esta é a correção funcional. Adicionamos um <select> para listar as categorias carregadas pelo Controller.

HTML

<!DOCTYPE html>
<html
  xmlns:th="http://www.thymeleaf.org"
  th:replace="~{layout :: layout (title=~{::title}, content=~{::section})}"
>
  <head>
    <title th:text="${jogo.id == null ? 'Novo Jogo' : 'Editar Jogo'}">Formulário</title>
  </head>
  <body>
    <section>
      <h1 th:text="${jogo.id == null ? 'Adicionar Novo Jogo' : 'Editar Jogo'}">Formulário</h1>

      <div class="card">
        <div class="card-body">
          <form th:action="@{/jogos}" th:object="${jogo}" method="post">
            <input type="hidden" th:field="*{id}" />

            <div class="mb-3">
              <label for="titulo" class="form-label">Título:</label>
              <input type="text" id="titulo" class="form-control" th:field="*{titulo}"/>
            </div>

            <div class="mb-3">
              <label for="categoria" class="form-label">Categoria:</label>
              <select id="categoria" class="form-select" th:field="*{categoria}">
                <option value="">Selecione uma Categoria</option>
                <option th:each="cat : ${categorias}"
                        th:value="${cat.id}"
                        th:text="${cat.nome}">Categoria
                </option>
              </select>
            </div>

            <div class="row">
              <div class="col-md-8 mb-3">
                <label for="autor" class="form-label">Autor:</label>
                <input type="text" id="autor" class="form-control" th:field="*{autor}"/>
              </div>
              <div class="col-md-4 mb-3">
                <label for="anoPublicacao" class="form-label">Ano de Publicação:</label>
                <input type="number" id="anoPublicacao" class="form-control" th:field="*{anoPublicacao}"/>
              </div>
            </div>

            <div class="mb-3">
              <label for="genero" class="form-label">Gênero:</label>
              <input type="text" id="genero" class="form-control" th:field="*{genero}"/>
            </div>

            <div class="mb-3">
              <label for="urlCapa" class="form-label">URL da Capa:</label>
              <input type="text" id="urlCapa" class="form-control" th:field="*{urlCapa}"/>
            </div>

            <div class="form-check mb-3">
              <input type="checkbox" id="finalizado" class="form-check-input" th:field="*{finalizado}"/>
              <label for="finalizado" class="form-check-label">Finalizado</label>
            </div>

            <div>
              <button type="submit" class="btn btn-success">Salvar</button>
              <a th:href="@{/jogos}" class="btn btn-secondary">Cancelar</a>
            </div>
          </form>
        </div>
      </div>
    </section>

  </body>
</html>
11. Executando e Testando a Aplicação
Execute a aplicação: Rode a classe BibliotecaJogosApplication.java a partir da sua IDE.

Acesse o console H2: Abra http://localhost:8080/h2-console.

JDBC URL: jdbc:h2:mem:testdb

User Name: sa

Password: (deixe em branco)

Clique em "Connect" para ver as tabelas criadas.

Acesse a aplicação: Abra http://localhost:8080/jogos.

Teste o CRUD:

Você verá os jogos pré-carregados pelo DataInitializer.

Adicione um novo jogo (agora você pode selecionar a categoria!).

Edite e exclua jogos.

Teste a pesquisa e a ordenação.

12. Containerização com Docker e Docker Compose
    Vamos empacotar nossa aplicação e seu banco de dados (PostgreSQL) para rodar em qualquer ambiente de forma consistente usando Docker. Também incluiremos o pgAdmin, uma interface gráfica para gerenciar o banco.

Passo 1: Criar o Dockerfile
Na raiz do seu projeto, crie um arquivo chamado Dockerfile (sem extensão).

Dockerfile

# Estágio 1: Build da aplicação com Maven

FROM maven:3.9.6-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY .mvn .mvn
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Estágio 2: Criação da imagem final

FROM eclipse-temurin:21-jre-jammy
WORKDIR /app
COPY --from=build /app/target/\*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
Passo 2: Criar o arquivo .dockerignore
Para acelerar o build, crie um arquivo .dockerignore na raiz do projeto:

.git
.idea
target/
Passo 3: Orquestrando com Docker Compose (App + DB + pgAdmin)
Crie um arquivo docker-compose.yml na raiz do projeto. Este arquivo define todos os serviços (aplicação, banco de dados e pgAdmin) e como eles se conectam.

YAML

version: "3.8"

services:

# Serviço do banco de dados PostgreSQL

db:
image: postgres:15
container_name: postgres-db
environment:
POSTGRES_USER: seu_usuario # Mesmo usuário do application-prod.properties
POSTGRES_PASSWORD: sua_senha # Mesma senha do application-prod.properties
POSTGRES_DB: bibliotecajogos # Mesmo nome do banco do application-prod.properties
ports: - "5432:5432"
volumes: - postgres_data:/var/lib/postgresql/data

# Serviço da nossa aplicação

app:
build: . # Constrói a imagem a partir do Dockerfile local
container_name: bibliotecajogos-app
depends_on: - db # Garante que o banco inicie antes da aplicação
ports: - "8080:8080"
environment: # Sobrescreve as configurações para apontar para o container 'db' - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/bibliotecajogos - SPRING_DATASOURCE_USERNAME=seu_usuario - SPRING_DATASOURCE_PASSWORD=sua_senha - SPRING_PROFILES_ACTIVE=prod # Ativa o perfil de produção

# Serviço do pgAdmin (Interface Gráfica para o DB)

pgadmin:
image: dpage/pgadmin4
container_name: pgadmin4-web
environment:
PGADMIN_DEFAULT_EMAIL: "admin@admin.com" # Email para login no pgAdmin
PGADMIN_DEFAULT_PASSWORD: "admin" # Senha para login no pgAdmin
ports: - "5050:80" # Acessível em http://localhost:5050
depends_on: - db
volumes: - pgadmin_data:/var/lib/pgadmin

# Volumes para persistir os dados

volumes:
postgres_data:
pgadmin_data:
Passo 4: Construir e Rodar o Ambiente Completo
Abra um terminal na raiz do projeto e execute:

Bash

docker-compose up --build
docker-compose up: Inicia todos os serviços.

--build: Força a reconstrução da imagem da sua aplicação (app).

Passo 5: Acessando os Serviços
Aguarde os logs estabilizarem. Agora você pode acessar:

Sua Aplicação: http://localhost:8080/jogos

A aplicação agora está conectada ao banco PostgreSQL dentro do Docker e usando o perfil prod (sem os dados de seeding).

pgAdmin (Gerenciador do DB): http://localhost:5050

Login: admin@admin.com

Senha: admin

Para conectar o pgAdmin ao banco db:

Na tela inicial, clique em "Add New Server".

Na aba General, dê um nome (ex: Biblioteca Docker).

Na aba Connection:

Host name/address: db (Este é o nome do serviço no Docker Compose)

Port: 5432

Maintenance database: bibliotecajogos

Username: seu_usuario

Password: sua_senha

Clique em "Save".

Agora você pode visualizar as tabelas que o Spring Boot criou no banco PostgreSQL!

13. Próximos Passos
    Com a aplicação funcional e containerizada, você pode evoluir o projeto:

Validação de Dados: Use jakarta.validation (ex: @NotBlank, @Size) nas suas entidades e trate os erros no formulário.

Paginação: Implemente PagingAndSortingRepository para lidar com grandes volumes de jogos.

Autenticação: Adicione o Spring Security para criar áreas de login e proteger as rotas de CRUD.

Testes: Escreva testes unitários para os serviços e testes de integração para os controllers.

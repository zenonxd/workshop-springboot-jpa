# Project: Web Services com Spring Boot e JPA / Hibernate 


## Sumário
- [Domain Model](#Domain-Model)
- [Domain Instance](#Domain-Instance)
- [User entity and resource](#User-entity-and-resource)
- [H2 database, test profile, JPA](#H2-database-test-profile-JPA)
- [JPA repository, dependency injection, database seeding](#JPA-repository,-dependency-injection,-database-seeding)
- [Service layer, component registration](#Service-layer,-component-registration)


## Conteúdo
- Criar projeto Spring Boot Java
- Implementar modelo de domínio
- Estruturar camadas lógicas: resource, service, repository
- Configurar banco de dados de teste (H2)
- Povoar o banco de dados
- CRUD - Create, Retrieve, Update, Delete
- Tratamento de exceções

## Domain Model
![img.png](img.png)

## Domain Instance
![img_1.png](img_1.png)


## User entity and resource

### Spring Annotations

- @RestController - Indica que o dado que será retornada por cada método, está sendo escrita diretamente no body ao invés de renderizar um template.
  O RestController é inserido em classes que possuem ResourceLayer. Se temos uma classe <b>Usuario</b> por exemplo, teremos uma UserResource, conforme abaixo.

```java
@RestController
@RequestMapping(value = "/users")
public class UserResource {
    @GetMapping
    public ResponseEntity<User> findAll() {
        User u = new User(1L, "Maria", "maria@gmail.com", "99999", "12345");
        return ResponseEntity.ok().body(u);
    }
}
```
- @RequestMapping - Utilizamos ela nas classes Controllers, porque assim todos os métodos herdam o endereço do controller. Se usássemos @GetMapping, teríamos que repetir a anotação em todo método da classe. <br>
  Por exemplo, acima passamos o Mapping com o valor "/users". Portanto, todo método por padrão, irá herder esse endereço.


- @GetMapping - Feito para requisições GET, conforme utilizado no método acima.


## H2 database, test profile, JPA

### Checklist:
- JPA & H2 dependencies
- application.properties
- application-test.properties
- Entity: JPA mapping 

<details>
  <summary style="font-weight: bold; font-size: 18px">Dependencies:</summary>

```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>

  <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
  </dependency>
```
</details>


<details>
  <summary style="font-weight: bold; font-size: 18px" >application.properties:</summary>

```
spring.profiles.active=test
spring.jpa.open-in-view=true
```

</details>


<details> 
  <summary style="font-weight: bold; font-size: 18px" >application-test.properties:</summary>

```
# DATASOURCE
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=
# H2 CLIENT
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
# JPA, SQL
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.defer-datasource-initialization=true
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```
</details> 

<details>
  <summary style="font-weight: bold; font-size: 18px">Entity: JPA Mapping:</summary>

  Colocar na classe User algumas anotações do JPA. Instruindo ao JPA como converter os objetos para o modelo relacional.
  1. Colocar @Entity em cima da classe.
  2. Colocar uma @Table(name = "tb_user"). Isso porque a palavra User é uma palavra reservada do banco de dados H2 então precisamos renomear para essa tabela não ter nenhum tipo de conflito.
  3. Settar primaryKey, nesse caso ID.
     - Sabemos que ID é uma coluna autoincrementada, então colocamos @GeneratedValue(strategy = GenerationType.IDENTITY).
    
```java
@Entity
@Table(name = "tb_user")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
```
</details>

## JPA repository, dependency injection, database seeding

### Checklist:
- UserRepository extends JPARepository<User, Long>
- Configuration class for "test" profile
- @Autowired UserRepository
- Instantiate objects in memory
- Persist objects

<details>
    <summary style="font-weight: bold; font-size: 18px">UserRepository</summary>

UserRepository será responsável por fazer operações com a entidade User. Para criamos, faremos o UserRepository extender o JPARepository, passando o tipo da Entidade que vamos acessar e o tipo da chave.
1. Criamos um package chamado repositories dentro de course e criamos o UserRepository (que será uma interface);
2. Dentro da interface criada, extendemos passando o JpaRepository e parâmetros;
```java
public interface UserRepository extends JpaRepository <User, Long > {
}
```
</details>

<details>
    <summary style="font-weight: bold; font-size: 18px">Configuration class for "test" profile</summary>

Essa classe de configuração ela não é nem controller, service ou repository. Ela é uma classe auxiliar que vai fazer umas configurações na aplicação.
1. Criamos um package chamado config e criamos uma entidade chamada TestConfig.
2. Para o Spring identificar que é uma classe de configuração, passaremos uma anottation @Configuration.
    Além disso, para caracterizarmos essa classe como perfil de teste, passaremos uma anottation @Profile.
```java
@Configuration
@Profile("test")
public class TestConfig {
}
```
</details>

<details>
    <summary style="font-weight: bold; font-size: 18px">@Autowired UserRepository</summary>

Essa classe TestConfig vai servir para database seeding. Ou seja, popular o banco de dados. Sabemos que para acessar/salvar coisas no banco de dados utilizamos o Repository. <br>

Portanto, teremos o nosso primeiro caso de injeção de dependência. <br>

Para fazermos o Spring entender que o objeto dependerá de outro, importar o UserRepository:
```java
    @Autowired
    private UserRepository userRepository;
```
A anotação @AutoWired irá associar uma instância de UserRepository dentro de TestConfig através do Spring.
</details>

<details>
    <summary style="font-weight: bold; font-size: 18px">Instantiate objects in memory</summary>

O TestConfig irá implementar a interface CommandLineRunner. 
```java
public class TestConfig implements CommandLineRunner {,
    @Override
    public void run(String... args) throws Exception {
        User u1 = new User(null, "Maria Brown", "maria@gmail.com", "988888888", "123456");
        User u2 = new User(null, "Alex Green", "alex@gmail.com", "977777777", "123456");
    }
}
```
</details>

<details>
    <summary style="font-weight: bold; font-size: 18px">Persist Objects (Saving)</summary>

Tudo dentro do método run será executado quando a operação for iniciada (no programa principal -CourseApplication-).
1. Instanciamos os objetos desejados;
2. E depois chamamos userRepository.saveAll(e aqui dentro, passamos um Arrays inserindo os objetos acima).

```java
    @Override
    public void run(String... args) throws Exception {
        User u1 = new User(null, "Maria Brown", "maria@gmail.com", "988888888", "123456");
        User u2 = new User(null, "Alex Green", "alex@gmail.com", "977777777", "123456");

        userRepository.saveAll(Arrays.asList(u1, u2));
    }
}
```
</details>

## Service layer, component registration
### Order, Instant, ISO 8601

### Checklist:
- Basic new entity checklist:
  - Entity
    - "To many" association, lazy loading, JsonIgnore
- Repository
- Seed
- Service
- Resource
# IMPORTANTE
A camada de Resources é um <b>controlador</b>. Deve somente intermediar as operações de aplicação e regras de negócio.

Portanto, o Resources (controlador/UserResource) irá <b>depender do Service (service/UserService).</b> 

E o Service depende do Repository/UserRepository - <b>a interface</b>.

### Resumo: @RestControllers (UserResource) tem dependencia de @Services (UserService) e por sua vez, Services dependem de @Repository.

## MAIS SOBRE

Sabemos que o UserService importado para dentro do UserResource com @AutoWired foi injetada automaticamente pelo Spring.

Mas para a variável service (advinda do UserService) funcionar dentro da classe (UserResource), a classe UserService deve ser registrada como um componente do Spring.

Para fazer isso usamos uma Annotation:
```java
@Component
public class UserService {}

    E assim, essa classe poderá ser injetada automaticamente com o @AutoWired na outra classe:

public class UserResource {
    @Autowired
    private UserService service;
```

### Além do @Component, temos outras Annotations com uma melhor semântica, a saber:

@Repository - Registra um repository.

@Service - Registra um serviço na camada de serviço. 

Como a nossa classe é uma classe de serviço, daremos preferência para essa annotation.
```java
@Service
public class UserService {
```

### Implementando findById

#### Dentro do Service: 

Faz a implementação do método.

Esse método vai retornar um User.

Para recuperarmos esse User por Id, usaremos novamente o repository.findById(id) e alocaremos esse Id em um Optional.

```java
    public User findById(Long id) {
        Optional<User> obj = repository.findById(id);
        return obj.get();
    }
```
o obj.get retorna o objeto do tipo que especificamos, neste caso, User.

#### Dentro do UserResource:

Criamos a função, mas no Mapping passamos um value:
```java
    @GetMapping(value = "/{id}")
    public ResponseEntity<User> findById() {}
```
Isso indica que a nossa requisição vai aceitar um id dentro da URL.

O nosso parâmetro dentro da função vai ser exatamente o que está dentro do value.

Para o Spring aceitar esse id e considerar ele como parâmetro, a gente coloca uma Anottation: @PathVariable
```java
    @GetMapping(value = "/{id}")
    public ResponseEntity<User> findById(@PathVariable Long id) {}
```

O @PathVariable ele serve para usarmos como parâmetro o que está dentro das chaves {}.

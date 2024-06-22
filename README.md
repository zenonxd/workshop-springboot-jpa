# Project: Web Services com Spring Boot e JPA / Hibernate 


## Sumário
- [Domain Model](##Domain-Model)
- [Domain Instance](##Domain-Instance)
- [User entity and resource](##User-entity-and-resource)
- [H2 database, test profile, JPA](##H2-database-test-profile-JPA)


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


## H2 database test profile JPA

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

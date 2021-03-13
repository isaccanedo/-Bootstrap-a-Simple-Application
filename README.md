# Bootstrap-a-Simple-Application
:star2: # Bootstrap a Simple Application - Uma maneira de começar de maneira simples, com um aplicativo web básico

# 1. Visão Geral

Spring Boot é uma adição opinativa e focada em convenção sobre configuração para a plataforma Spring - 
altamente útil para começar com o mínimo de esforço e criar aplicativos autônomos de nível de produção.

Este tutorial é um ponto de partida para o Boot - uma maneira de começar de maneira simples, com um aplicativo web básico.

Veremos algumas configurações básicas, um front-end, manipulação rápida de dados e tratamento de exceções.

# 2. Configuração
Primeiro, vamos usar o Spring Initializr para gerar a base para nosso projeto.

O projeto gerado depende do pai de inicialização:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath />
</parent>
```
As dependências iniciais serão bastante simples:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

# 3. Configuração do aplicativo
A seguir, configuraremos uma classe principal simples para nosso aplicativo:

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Observe como estamos usando @SpringBootApplication como nossa classe de configuração de aplicativo primária; 
nos bastidores, isso é equivalente a 
- @Configuration;
- @EnableAutoConfiguration;
- @ComponentScan juntos.

Por fim, definiremos um arquivo application.properties simples - que por enquanto tem apenas uma propriedade:
```
server.port=8081
```

server.port altera a porta do servidor do padrão 8080 para 8081.

# 4. Visualização MVC Simples
Vamos agora adicionar um front end simples usando Thymeleaf.

Primeiro, precisamos adicionar a dependência spring-boot-starter-thymeleaf ao nosso pom.xml:
```
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-thymeleaf</artifactId> 
</dependency>
```
Isso ativa o Thymeleaf por padrão - nenhuma configuração extra é necessária.

Agora podemos configurá-lo em nosso application.properties:

```
spring.thymeleaf.cache=false
spring.thymeleaf.enabled=true 
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html

spring.application.name=Bootstrap Spring Boot
```

A seguir, definiremos um controlador simples e uma página inicial básica - com uma mensagem de boas-vindas:

```
@Controller
public class SimpleController {
    @Value("${spring.application.name}")
    String appName;

    @GetMapping("/")
    public String homePage(Model model) {
        model.addAttribute("appName", appName);
        return "home";
    }
}
```
Finalmente, aqui está nosso home.html:

```
<html>
<head><title>Home Page</title></head>
<body>
<h1>Hello !</h1>
<p>Welcome to <span th:text="${appName}">Our App</span></p>
</body>
</html>
```

Observe como usamos uma propriedade que definimos em nossas propriedades - 
e então injetamos para que possamos mostrá-la em nossa página inicial.

# 5. Segurança
A seguir, vamos adicionar segurança ao nosso aplicativo - primeiro incluindo o iniciador de segurança:

```
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-security</artifactId> 
</dependency>
```

Agora, você deve estar notando um padrão - a maioria das bibliotecas do Spring são facilmente 
importadas para o nosso projeto com o uso de iniciadores de inicialização simples.

Uma vez que a dependência do spring-boot-starter-security do classpath do aplicativo - 
todos os endpoints são protegidos por padrão, usando httpBasic ou formLogin com base na 
estratégia de negociação de conteúdo do Spring Security.

É por isso que, se tivermos o iniciador no caminho de classe, devemos geralmente definir 
nossa própria configuração de segurança personalizada estendendo a classe WebSecurityConfigurerAdapter:

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .anyRequest()
            .permitAll()
            .and().csrf().disable();
    }
}
```

Em nosso exemplo, estamos permitindo acesso irrestrito a todos os terminais. 

Claro, Spring Security é um tópico extenso e não é facilmente abordado em algumas 
linhas de configuração - então, eu definitivamente encorajo você a se aprofundar no tópico.

# 6. Persistência Simples
Vamos começar definindo nosso modelo de dados - uma entidade Book simples:

```
@Entity
public class Book {
 
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @Column(nullable = false, unique = true)
    private String title;

    @Column(nullable = false)
    private String author;
}
```
E seu repositório, fazendo bom uso do Spring Data aqui:

```
public interface BookRepository extends CrudRepository<Book, Long> {
    List<Book> findByTitle(String title);
}
```

Finalmente, precisamos configurar nossa nova camada de persistência:

```
@EnableJpaRepositories("com.baeldung.persistence.repo") 
@EntityScan("com.baeldung.persistence.model")
@SpringBootApplication 
public class Application {
   ...
}
```
Observe que estamos usando:

- @EnableJpaRepositories para verificar o pacote especificado em busca de repositórios
- @EntityScan para selecionar nossas entidades JPA
Para manter as coisas simples, estamos usando um banco de dados H2 em memória aqui - 
para que não tenhamos nenhuma dependência externa ao executar o projeto.

Assim que incluímos a dependência H2, Spring Boot a detecta automaticamente e 
configura nossa persistência sem a necessidade de configuração extra, a não ser as 
propriedades da fonte de dados:

```
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:bootapp;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=
```

É claro que, assim como a segurança, a persistência é um tópico mais amplo do 
que este conjunto básico aqui e que você certamente deve explorar mais.

# 7. Web e o controlador

A seguir, vamos dar uma olhada em uma camada da web - e vamos começar configurando 
um controlador simples - o BookController.

Implementaremos operações CRUD básicas expondo os recursos do Livro com algumas 
validações simples:

```
@RestController
@RequestMapping("/api/books")
public class BookController {

    @Autowired
    private BookRepository bookRepository;

    @GetMapping
    public Iterable findAll() {
        return bookRepository.findAll();
    }

    @GetMapping("/title/{bookTitle}")
    public List findByTitle(@PathVariable String bookTitle) {
        return bookRepository.findByTitle(bookTitle);
    }

    @GetMapping("/{id}")
    public Book findOne(@PathVariable Long id) {
        return bookRepository.findById(id)
          .orElseThrow(BookNotFoundException::new);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Book create(@RequestBody Book book) {
        return bookRepository.save(book);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        bookRepository.findById(id)
          .orElseThrow(BookNotFoundException::new);
        bookRepository.deleteById(id);
    }

    @PutMapping("/{id}")
    public Book updateBook(@RequestBody Book book, @PathVariable Long id) {
        if (book.getId() != id) {
          throw new BookIdMismatchException();
        }
        bookRepository.findById(id)
          .orElseThrow(BookNotFoundException::new);
        return bookRepository.save(book);
    }
}
```
Dado que este aspecto do aplicativo é uma API, usamos a anotação @RestController aqui - 
que equivale a um @Controller junto com @ResponseBody - de forma que cada método empacote 
o recurso retornado direto para a resposta HTTP.

Apenas uma observação que vale a pena destacar - estamos expondo nossa entidade Livro 
como nosso recurso externo aqui. Isso é bom para nosso aplicativo simples aqui, mas em 
um aplicativo do mundo real, você provavelmente desejará separar esses dois conceitos.

8. Tratamento de erros
Agora que o aplicativo principal está pronto, vamos nos concentrar em um mecanismo 
simples de tratamento de erros centralizado usando @ControllerAdvice:

```
@ControllerAdvice
public class RestExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler({ BookNotFoundException.class })
    protected ResponseEntity<Object> handleNotFound(
      Exception ex, WebRequest request) {
        return handleExceptionInternal(ex, "Book not found", 
          new HttpHeaders(), HttpStatus.NOT_FOUND, request);
    }

    @ExceptionHandler({ BookIdMismatchException.class, 
      ConstraintViolationException.class, 
      DataIntegrityViolationException.class })
    public ResponseEntity<Object> handleBadRequest(
      Exception ex, WebRequest request) {
        return handleExceptionInternal(ex, ex.getLocalizedMessage(), 
          new HttpHeaders(), HttpStatus.BAD_REQUEST, request);
    }
}
```
Além das exceções padrão que estamos tratando aqui, também estamos usando uma exceção personalizada:

BookNotFoundException:

```
public class BookNotFoundException extends RuntimeException {

    public BookNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
    // ...
}
```

Isso deve dar uma ideia do que é possível com esse mecanismo de tratamento de exceção global. 
Se você gostaria de ver uma implementação completa, dê uma olhada no tutorial detalhado.

Observe que Spring Boot também fornece um mapeamento / erro por padrão. Podemos personalizar
sua visualização criando um error.html simples:

```
<html lang="en">
<head><title>Error Occurred</title></head>
<body>
    <h1>Error Occurred!</h1>    
    <b>[<span th:text="${status}">status</span>]
        <span th:text="${error}">error</span>
    </b>
    <p th:text="${message}">message</p>
</body>
</html>
```

Como a maioria dos outros aspectos do Boot, podemos controlar isso com uma propriedade simples:

```
server.error.path=/error2
```

# 9. Teste
Por fim, vamos testar nossa nova API de livros.

Podemos usar @SpringBootTest para carregar o contexto do aplicativo e verificar se não há erros ao executar o aplicativo:

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringContextTest {

    @Test
    public void contextLoads() {
    }
}
```

A seguir, vamos adicionar um teste JUnit que verifica as chamadas para a API 
que escrevemos, usando RestAssured:

```
public class SpringBootBootstrapLiveTest {

    private static final String API_ROOT
      = "http://localhost:8081/api/books";

    private Book createRandomBook() {
        Book book = new Book();
        book.setTitle(randomAlphabetic(10));
        book.setAuthor(randomAlphabetic(15));
        return book;
    }

    private String createBookAsUri(Book book) {
        Response response = RestAssured.given()
          .contentType(MediaType.APPLICATION_JSON_VALUE)
          .body(book)
          .post(API_ROOT);
        return API_ROOT + "/" + response.jsonPath().get("id");
    }
}
```

Primeiro, podemos tentar encontrar livros usando métodos variantes:

```
@Test
public void whenGetAllBooks_thenOK() {
    Response response = RestAssured.get(API_ROOT);
 
    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
}

@Test
public void whenGetBooksByTitle_thenOK() {
    Book book = createRandomBook();
    createBookAsUri(book);
    Response response = RestAssured.get(
      API_ROOT + "/title/" + book.getTitle());
    
    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
    assertTrue(response.as(List.class)
      .size() > 0);
}
@Test
public void whenGetCreatedBookById_thenOK() {
    Book book = createRandomBook();
    String location = createBookAsUri(book);
    Response response = RestAssured.get(location);
    
    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
    assertEquals(book.getTitle(), response.jsonPath()
      .get("title"));
}

@Test
public void whenGetNotExistBookById_thenNotFound() {
    Response response = RestAssured.get(API_ROOT + "/" + randomNumeric(4));
    
    assertEquals(HttpStatus.NOT_FOUND.value(), response.getStatusCode());
}
```

A seguir, testaremos a criação de um novo livro:

```
@Test
public void whenCreateNewBook_thenCreated() {
    Book book = createRandomBook();
    Response response = RestAssured.given()
      .contentType(MediaType.APPLICATION_JSON_VALUE)
      .body(book)
      .post(API_ROOT);
    
    assertEquals(HttpStatus.CREATED.value(), response.getStatusCode());
}

@Test
public void whenInvalidBook_thenError() {
    Book book = createRandomBook();
    book.setAuthor(null);
    Response response = RestAssured.given()
      .contentType(MediaType.APPLICATION_JSON_VALUE)
      .body(book)
      .post(API_ROOT);
    
    assertEquals(HttpStatus.BAD_REQUEST.value(), response.getStatusCode());
}
```
Atualizar um livro existente:

```
@Test
public void whenUpdateCreatedBook_thenUpdated() {
    Book book = createRandomBook();
    String location = createBookAsUri(book);
    book.setId(Long.parseLong(location.split("api/books/")[1]));
    book.setAuthor("newAuthor");
    Response response = RestAssured.given()
      .contentType(MediaType.APPLICATION_JSON_VALUE)
      .body(book)
      .put(location);
    
    assertEquals(HttpStatus.OK.value(), response.getStatusCode());

    response = RestAssured.get(location);
    
    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
    assertEquals("newAuthor", response.jsonPath()
      .get("author"));
}
```

E exclua um livro:

```
@Test
public void whenDeleteCreatedBook_thenOk() {
    Book book = createRandomBook();
    String location = createBookAsUri(book);
    Response response = RestAssured.delete(location);
    
    assertEquals(HttpStatus.OK.value(), response.getStatusCode());

    response = RestAssured.get(location);
    assertEquals(HttpStatus.NOT_FOUND.value(), response.getStatusCode());
}
```

# 10. Conclusão
Esta foi uma introdução rápida, mas abrangente, do Spring Boot.

É claro que mal tocamos a superfície aqui - há muito mais nesta estrutura que podemos 
cobrir em um único artigo de introdução.

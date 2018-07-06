# Swagger

Modelar e documentar o codigo de API's do estilo REST

## Short description

Linguagem com formato JSON com campos padronizados para descrever:
- Recursos
- Modelo de dados
- URI's
- Metodos HTTP
- Content Types
- Responses

## Pré requisitos

- Maven
- Java 10


## Configuração

- Adicionar a dependência no ficheiro *pom.xml*

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.0</version>
</dependency>
```
Verificar a última versão: [Maven dependency](https://mvnrepository.com/artifact/io.springfox/springfox-swagger2/) 

- Configurar o Swagger 2 na aplicação

```
...
@Configuration
@EnableSwagger2
public class SwaggerConfig {                                    
    @Bean
    public Docket api() { 
        return new Docket(DocumentationType.SWAGGER_2)  
          .select()                                  
          .apis(RequestHandlerSelectors.any())              
          .paths(PathSelectors.any())                          
          .build();                                           
    }
}   
```
Swagger 2 é utilizado atráves da anotação *@EnableSwagger2* .

Após o Docker bean ser definido, seu método select () retorna uma instância do ApiSelectorBuilder, que fornece uma maneira de controlar os terminais expostos pelo Swagger.

Opções para seleção de RequestHandlers podem ser configurados com a ajuda de RequestHandlerSelectors e PathSelectors. O uso de any () para ambos faz com que a documentação de toda a API esteja disponível através do Swagger.

![](https://i1.wp.com/springframework.guru/wp-content/uploads/2017/02/Swagger_JSON_Output.png?ssl=1)

### Swagger UI
O Swagger UI é uma solução grafica que facilita muito a interação do utilizador com a documentação da API gerada pelo Swagger.

- Adicionar a dependência no ficheiro *pom.xml*

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.0</version>
</dependency>
```
 Verificar a última versão: [Maven dependency](https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui/) 

É possivel visualizar o Swagger com uma interface gráfica atráves do URL no browser: *http://localhost:8080/your-app-root/swagger-ui.html*

![](https://i1.wp.com/springframework.guru/wp-content/uploads/2017/02/Swagger_2_generated_documentation.png)


- Filtrar a API for Swagger’s Response. 
É possivel personalizar as informações que aparecem bem como que informações queremos que sejam visualizadas

**PathSelectors** fornece personalização adicional com funções que verificam os caminhos de pedidos do aplicativo. Pode-se usar any(), none (), regex() ou ant().

No exemplo abaixo, informamos o Swagger a incluir apenas controladores de um componente específico, com caminhos específicos, usando o predicado ant ().
```
      .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
      .paths(PathSelectors.ant("/foos/*"))      
```
**ApiInfo** define as informações personalizadas a apresentar sobre a API.

```
@Bean
public Docket api() {                
    return new Docket(DocumentationType.SWAGGER_2)          
      .select()
      .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
      .paths(PathSelectors.ant("/foos/*"))
      .build()
      .apiInfo(apiInfo());
}
 
private ApiInfo apiInfo() {
     return new ApiInfo(
       "My REST API", 
       "Some custom description of API.", 
       "API TOS", 
       "Terms of service", 
       new Contact("John Doe", "www.example.com", "myeaddress@company.com"), 
       "License of API", "API license URL", Collections.emptyList());
}
```

- Custom Methods Response Messages .

O Swagger permite substituir mensagens de resposta globais de métodos HTTP através do método **globalResponseMessage ()** do Docker.

Primeiro, temos de dizer ao Swagger a não usar mensagens de resposta padrão, depois devemos definir o que pretendemos adicionando a informação ao Docker.

Por exemplo: alterar as mensagens 403 para todos os métodos GET. 

```
                ...
                .apiInfo(apiInfo())
                .useDefaultResponseMessages(false)
                .globalResponseMessage(RequestMethod.GET,
                        newArrayList(
                                new ResponseMessageBuilder()
                                        .code(403)
                                        .message("Forbidden! Don't have permissions ")
                                        .build()));

    }
    ```

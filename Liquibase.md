# Liquibase

Ferramenta de versionamento de base de dados relacional.

## Short description

- Permite guardar/realizar todas as alterações a base de dados
- Permite especificar as alterações nos formatos XML, YAML, JSON e SQL
- Permite escolher as alterações a serem aplicadas em diferentes ambientes (dev, staging, QA)

## Principais conceitos

- `databaseChangelog`: Ficheiro onde todas as alterações são especificadas. 
- `changeSets`: São mudanças atômicas que podem ser aplicadas ao banco de dados. Cada changeset é identificado exclusivamente 
por id, autor e pacote do arquivo onde o changeset é definido.Cada conjunto de alterações é executado como uma única transação. 
- Tabela `DATABASECHANGELOG`: Mantém registros do que todos os changesets que foram aplicados no banco de dados até o momento.
  + Uma das colunas criadas é `MD5SUM`. O Liquibase cria o Checksum MD5 de cada changeset que foi aplicado. Ao aplicar novas alterações,
  ele compara a checksum dos conjuntos de alterações existentes ao que já foi aplicado ao banco de dados. Esta etapa garante que os 
  conjuntos de alterações existentes não tenham sido modificados. Se a checksum de verificação for diferente, o Liquibase falhará. 
  Esta é uma etapa que garante que um conjunto de alterações, uma vez aplicado, seja imutável e que quaisquer alterações em um esquema 
  de base de dados devem ser aplicadas como um novo conjunto de alterações.
  
- Tabela `DATABASECHANGELOGLOCK`: O Liquibase tenta adquirir um bloqueio desta tabela. Isso garante que nenhuma outra migração esteja 
acontecendo. Se não conseguir adquirir o bloqueio, a etapa de migração falhará.

## Pré requisitos

- Maven
- Java 10

## Configuração

- Adicionar a dependência no ficheiro *pom.xml*

```
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
    <version>3.6.2</version>
</dependency>

```
*Verificar ultima versão* - [Maven dependency](https://mvnrepository.com/artifact/org.liquibase/liquibase-core)

- Organizar os changelogs numa localização

![ ](https://cdn-images-1.medium.com/max/800/1*ky1iq5tE1qFnccaaKAHcjw.png)


Para organizare simplificar os changesets, dividimos os changesets em arquivos separados e utiliza-se a tag *include* no 
databaseChangeLog para incluí-los.

```
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

  <include file="changelog/01-create-books-and-author-schema.xml" relativeToChangelogFile="true"/>
  <include file="changelog/02-insert-data-authors.xml" relativeToChangelogFile="true"/>
  <include file="changelog/02-insert-data-books.xml" relativeToChangelogFile="true"/>
</databaseChangeLog>
```
- No `src/main/resource/application.properties` adiciona-se: 

```
...
liquibase.change-log=classpath:db/liquibase-changelog.xml
spring.jpa.hibernate.ddl-auto=none
...
```
A 1ª linha informa que quando o aplicativo Spring-boot é iniciado, ele sabe onde é o ponto de partida para o seu databaseChangeLog

A 2ª linha diz ao Spring-boot para não criar o banco de dados atráves do hibernate. Isso porque queremos que a Liquibase faça esse 
trabalho para nós


**Ao executar a aplicação o liquibase adiciona automaticamente à nossa base de dados as aleraçoes presentes nos `changesets`**

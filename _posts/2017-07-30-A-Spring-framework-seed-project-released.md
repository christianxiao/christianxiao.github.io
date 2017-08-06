---
layout: post
title: Spring Seed -- A J2EE Spring framework seed project
category: programming
---

A seed project based on Spring Framework family, including Spring Data JPA, Spring Data Redis and so on.

## Why this project
During Java web development, we usually have to work on several projects at the same time and we try to make those applications follow the same structure,
including directory structure, annotation based configuration over xml based configuration, naming convensions and so on.  
This is really useful when we need to create a new project. For example, the new project should share the same directory structure, third party dependencies and version (Spring, Hibernate, Redis..),
build scripts & tools and so on.  
So I try to create a common seed project including all those things to avoid repeating.

## The quick structure overview
- Just like other Spring frameworks, annotation based configuration is heavily used, like @Enable-* annotations.
- The build tool is gradle. Of course you can use Maven but then you need to write your own build scripts instead of those provided.
- Third party dependencies including Spring Framework 4, Hibernate, Spring Data JPA, Spring Data Redis, Spring Batch, Apache commons, Guava.
- Common J2EE applications include batch job module, domain module, web module. Spring seed have special support for those modules.
- Configure Swagger2 to check and test all Rest Apis.
- Thanks to docker, we don't need to install Mysql, Redis and config them locally, just install docker and run those shell scripts. Then your Mysql and redis is ready.

## Jcenter hosted
This project is hosted on Jcenter:
- Gradle
```
compile 'com.profullstack:spring-seed:1.0.0'
```

- Maven
  
```xml
<dependency>
  <groupId>com.profullstack</groupId>
  <artifactId>spring-seed</artifactId> 
  <version>1.0.0</version>
  <type>pom</type>
</dependency>
```

## Run this project is damn easily 
We provide docker shell script to help you run this project, but it is MacOS based, but I guess it will be easy to adapt to Windows.
- Install Docker and git.
- Git clone this project and go to sample folder.
```
git clone https://github.com/profullstack/spring-seed
cd spring-seed/sample
```
- Run Mysql docker, the database and database users will be automatically created. Note: you need to stop your local mysql first in case of port 3306 conflict.
```
cd deploy/mysql
./mysql-docker-start.sh
#Will print the unique docker instance ID
#Now an empty Mysql datase and two users dml and ddl is created.
```
- Run Redis cluster docker. Note: you need to stop all your local redis instances if you have any.
```
cd ../redis-cluster/
./redis-cluster-start.sh
#Will print docker instances process
```
- Migrate datase structure and insert test data with Flyway.
```
cd ../..
./gradlew :domain:flywayMigrate -Dflyway.extLocation=src/migration/data
#Will print success at the end.
#You can use any Mysql client to check the database. The root password is root.
```
- Finally, run our tomcat server.
```
./gradlew web:tomcatRun
```
- Go to http://localhost:8080/api/swagger-ui.html to check and test all the Rest Apis.

## Implementation details
### Project modules
Just like the sample project, we recommended you seperate your J2EE project into three different modules:
- **Domain module**. All the core business logic code should put here, such as database operations, core entity definations, core business services. The name ```domain``` came from the Domain Driven Design.
- **Web module**. All controllers should put here, and it depends on Domain module.
- **Batch module**. All batch job codes should put here, and it also depends on Domain module.
### Project profile and property
The project should also support **profile** and **property file** settings, such as local, develop and production profile. Under different profile, different property files will be read. And every module should have its own property file, for example:
- [Domain property files](https://github.com/profullstack/spring-seed/tree/master/sample/domain/src/main/resources/properties). This folder includes different profile properties, such as **domain-default.xml**, **domain-develop.xml**, **domain-production.xml**.
- [Batch property files](https://github.com/profullstack/spring-seed/tree/master/sample/batch/src/main/resources/properties).
- [Web property files](https://github.com/profullstack/spring-seed/tree/master/sample/web/src/main/resources/properties).  

The profle feature is based on Spring profile and use the same system property **spring.profiles.active** to set profile. Predefined profiles are **default**, **local**, **develop**, **integration** and **production**. The **default** profile is the default profile if none is set and the default property file is always read.  
Upon Spring context initialized, those property files will be read to Spring ```Environment``` object automatically and you can also use ```@Value("${}")``` to retrive those values. The core implemention code is under [profile package](https://github.com/profullstack/spring-seed/tree/master/src/main/java/com/profullstack/springseed/core/profile).  

### Domain module features
The [```DomainContextConfig.java```](https://github.com/profullstack/spring-seed/blob/master/sample/domain/src/main/java/com/profullstack/springseed/sample/domain/config/DomainContextConfig.java) is the root config class for domain module.
#### JPA support
```@EnableSpringSeedJpa(propertyPrefix="jdbc.basicDataSource", baseEntityClasses = {Group.class})```. This annotation will read all the properties start with ```propertyPrefix``` to new a DBCP2 ```BasicDataSource``` object as datasource. This feature use Java reflection to call the set method given the property name. For example, in the [default profile property file](https://github.com/profullstack/spring-seed/blob/master/sample/domain/src/main/resources/properties/domain-default.xml), we have
```
    <entry key="jdbc.basicDataSource.driverClassName">com.mysql.jdbc.Driver</entry>
```
Then we will automatically call ```setDriverClassName``` on ```basicDataSource``` object. This reduce a lot of code and we can set any value in the ```basicDataSource``` object.  
This annotation also import an ```EntityManagerFactory``` bean and a ```JpaTransactionManager``` bean.

Multiple datasources is also supported, you can set the ```beanNamePrefix``` to defined the bean name. The implementation is more complex than I thought. Because ```@Bean``` annotation only accept static name value, but we need to generate bean name at runtime. After reading some source code from the Spring Data JPA's ```@Enable-*``` annotation, I find the ```BeanDefinition``` object defines a Spring bean at runtime and we can use ```BeanDefinitionRegistry``` to register it. Check the code [here](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/jpa/SpringSeedJpa.java).

#### Redis cluster support
```
@EnableSpringSeedRedis(
	propertyPrefix = "redis.jedisConnectionFactory",
	cacheExpirations = {
		@cacheExpiration(value = "userCache", expiration = 60)
	}
)
```
This annotation will import ```jedisConnectionFactory```, ```redisTemplate``` and ```ExpirableRedisCacheManager```. 
- The most important feature is to add expiration support on Spring Cache abstraction. This is the known unsupported feature in Spring ```@Cacheable```. So we provide another implementation, ```@ExpirableCacheable```:
```
public @interface ExpirableCacheable {

	/**
	 * the cache name.
	 */
	String cacheName();

	/**
	 * the cache key. Will override default keyName by generator.
	 */
	String key() default "";

	/**
	 * -1: use default, in seconds.
	 */
	long expiration() default -1;
}
```
- A new ```keyGenerator```, the set the default key name to ```className + methodName + args``` to avoid a globle unique key set.
- A distributed lock implemantation ```SimpleRedisDlmLock``` based on Redis. Check [here](https://redis.io/topics/distlock) for more details.

### Batch module
```@EnableBatchProcessing``` is the native Spring Batch configuration, but you need to set up the batch database to use it for Spring batch need somewhere to store its metadata. Sometimes we don't want to set up a batch db, so [```@EnableSpringSeedBatchProcessing```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/batch/EnableSpringSeedBatchProcessing.java) is the replacement. It uses a in-memory Map to store the metadata, and a ResourcelessTransaction to handle transactions.  
```
@Configuration
//@EnableBatchProcessing
@EnableSpringSeedBatchProcessing
@Import(value={DomainContextConfig.class})
@ComponentScan(basePackageClasses = {Batchs.class})
public class BatchContextConfig {
}
```
Writing a simple batch job could also be complex, expecially when you just need a simple one step job. You need to define a job, a step and a tasklet. To simplify this process, you only need to extend the [```AbstractTaskletJob```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/batch/AbstractTaskletJob.java).
```
@Component
@Slf4j
public class SampleJob extends AbstractTaskletJob {

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        JobParameters jobParameters = chunkContext.getStepContext().getStepExecution().getJobParameters();

        for(int i = 0;i<10;i++) {
            log.info(jobParameters.getParameters().toString());
        }
        return RepeatStatus.FINISHED;
    }
}
```
Finally, the native job runner ```CommandLineJobRunner``` dose not support ```ApplicationContextInitializer```, so we have to write another job runner [```SpringSeedBatchJobRunner```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/batch/SpringSeedBatchJobRunner.java). 
```
Usage: java -cp "batch/*" com.profullstack.springseed.core.batch.SpringSeedBatchJobRunner com.profullstack.springseed.sample.batch.configuration.BatchContextConfig sampleJob param1=value1
```
### Web module
Just like the native Spring MVC , you can replace ```@EnableWebMvc``` with ```@EnableSpringSeedRestApiMvc``` to get those features provided by Spring Seed module.
```java
@Configuration
@ComponentScan(basePackageClasses = {Apis.class})
@EnableSpringSeedRestApiMvc(enableJwtConfig = @EnableJwtConfig(value=true, secretPropertyName="jwt.secret", expiration=10*60),
	enableSwagger2 = true) //replace @EnableWebMvc
public class ApiServletConfig extends WebMvcConfigurerAdapter {
}
```
#### RESTful API error handler
Provide a basic [```RestApiException```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/web/restapi/RestApiException.java) for you to throw Api exceptions, and all exceptions will be catched by [```RestApiExceptionAdvice```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/web/restapi/RestApiExceptionAdvice.java), then send a [```RestApiErrorResponse```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/web/restapi/RestApiErrorResponse.java) to client.
#### HTTP Authorization Header Parser
Provide a [```HttpAuthorizationHeaderParser```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/web/restapi/HttpAuthorizationHeaderParser.java) to parse the Authorization header, the header format follows the ref: http://www.ietf.org/rfc/rfc2617.txt
```
 * Implementation ref: http://www.ietf.org/rfc/rfc2617.txt
 * schema: Authorization: Digest username="Mufasa",realm="testrealm@host.com" ....
``` 
#### HTTP RESTful API Token Implementation
Provide a [```JwtTokenFactory```](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/web/restapi/JwtTokenFactory.java) to implement the JSON Web Token standard, a non database based solution, check ref: https://en.wikipedia.org/wiki/JSON_Web_Token.  
Another much simple token implementation ```SimpleAccessToken```, it's a database based solution.

### Gradle script support
All build scripts are inside [here](https://github.com/profullstack/spring-seed/tree/master/src/main/resources/gradle-script). Use it like in the [sample project](https://github.com/profullstack/spring-seed/blob/master/sample/build.gradle):
```
apply from: "file:build/springseed/gradle-script/plugins.gradle"
//This will import several plugins
```
Then you can use imported plugins:
```
    apply from: java8Plugin
    apply from: springPlugin
```
[Those plugins](https://github.com/profullstack/spring-seed/tree/master/src/main/resources/gradle-script/plugins) are:
- java8Plugin. Java 8 support.
- springPlugin. All dependencies defined here.
- jpaPlugin. Generate entity criteria classes.
- flywayPlugin. Database migration flyway plugin.
- batchPlugin. Package all batch dependecies to a simple zip file to run batch job.
- tomcatPlugin. Embbed tomcat plugin.
- jcenterPlugin. Upload to Jcenter.



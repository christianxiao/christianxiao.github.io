---
layout: post
title: "How to implement Spring @Enable-* annotation and register beans at runtime"
category: programming
---
We already know that Spring has a great plugin or module like system based on the ```@Enable-*``` annotation.
For example, Spring Cache provides ```@EnableCaching``` to support cache abstraction, Spring Data JPA provides ```@EnableJpaRepositories``` for
JPA repositories support, Spring Batch provides ```@EnableBatchProcessing``` for batch processing. So how do those annotations work and what can we benefit from them?

## The internal of the @EnableCaching
Fisrt, let's have a look on @EnableCaching. Check the source code we can find that:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(CachingConfigurationSelector.class)
public @interface EnableCaching {
```
Dive deep to **CachingConfigurationSelector**, we can find that:
```java
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {

	/**
	 * {@inheritDoc}
	 * @return {@link ProxyCachingConfiguration} or {@code AspectJCacheConfiguration} for
	 * {@code PROXY} and {@code ASPECTJ} values of {@link EnableCaching#mode()}, respectively
	 */
	@Override
	public String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return getProxyImports();
			case ASPECTJ:
				return getAspectJImports();
			default:
				return null;
		}
	}
```
Just like the name ```AdviceModeImportSelector```, this class is used for select different proxy mode beans based on the Proxy mode or AspectJ mode.
Well, then let's continue:
```java
public abstract class AdviceModeImportSelector<A extends Annotation> implements ImportSelector {
	/**
	 * This implementation resolves the type of annotation from generic metadata and
	 * validates that (a) the annotation is in fact present on the importing
	 * {@code @Configuration} class and (b) that the given annotation has an
	 * {@linkplain #getAdviceModeAttributeName() advice mode attribute} of type
	 * {@link AdviceMode}.
	 * <p>The {@link #selectImports(AdviceMode)} method is then invoked, allowing the
	 * concrete implementation to choose imports in a safe and convenient fashion.
	 * @throws IllegalArgumentException if expected annotation {@code A} is not present
	 * on the importing {@code @Configuration} class or if {@link #selectImports(AdviceMode)}
	 * returns {@code null}
	 */
	@Override
	public final String[] selectImports(AnnotationMetadata importingClassMetadata) {
```
The core method is the ```selectImports```, the returned String[] is the configuration classes that will import the actual beans.
### Then how are the configuration classes imported?
Just check the code in the fisrt step:
```
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {

	private static final String PROXY_JCACHE_CONFIGURATION_CLASS =
			"org.springframework.cache.jcache.config.ProxyJCacheConfiguration";
```
If the proxy mode is PROXY, then the ```org.springframework.cache.jcache.config.ProxyJCacheConfiguration``` class is imported.  
Check this class, we can see:
```
@Configuration
public class ProxyJCacheConfiguration extends AbstractJCacheConfiguration {

	@Bean(name = CacheManagementConfigUtils.JCACHE_ADVISOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public BeanFactoryJCacheOperationSourceAdvisor cacheAdvisor() {
		BeanFactoryJCacheOperationSourceAdvisor advisor =
				new BeanFactoryJCacheOperationSourceAdvisor();
		advisor.setCacheOperationSource(cacheOperationSource());
		advisor.setAdvice(cacheInterceptor());
		advisor.setOrder(this.enableCaching.<Integer>getNumber("order"));
		return advisor;
	}
```
No magic here, this class is just a common Spring ```@Configuration``` class, of course you can put any ```@Bean```s here, and those beans will be
imported automatically. And if we follow the extend chain, we can finally find all those magic Spring cache classes:
```
@Configuration
public abstract class AbstractCachingConfiguration implements ImportAware {

	protected AnnotationAttributes enableCaching;

	protected CacheManager cacheManager;

	protected CacheResolver cacheResolver;

	protected KeyGenerator keyGenerator;

	protected CacheErrorHandler errorHandler;
```
So, we get our first conclusion, the magic of **@Enable-*** is just a notification which will import a ```@Configuration``` class, and this class
will import those actual beans, most of the time you can just use the @Bean methods.
### But what if we want to register beans dynamically?
For example, if you write your own @EnableDataSource annotation, and this annotation will import a Configuration class which has @Bean method to return a ```BasicDataSource```.
Your DataSource bean will have the default name "dataSource", that's the common practice. But what if we want to support multiple dataSources?
If your client code uses your annotation twice, then you will create two BasicDataSource beans with the same name, that's the problem. What's worse is that the
@Bean annotation only accept static name as bean name, and we want to create the bean name provided by our client configuration. So we can't use the @Bean method.
### What we need is to register beans dynamically
If we can register beans dynamically, then we can set the bean name as we want.
Let's start from our original purpose, we need to write a **enable-* annotation to support multiple DataSource objects and configure their names dynamically.  
First, we write our annotation like this:
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(SpringSeedJpa.class)
public @interface EnableSpringSeedJpa {
	/**
	 * Useful when you need mutiple dataSources.
	 * @return Named used for transaction beans prefix.
     */
	String beanNamePrefix() default "";
}
```
Then our configuration class:
```java
public class SpringSeedJpa implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware, BeanFactoryAware {

	private BeanFactory beanFactory;
	private ResourceLoader resourceLoader;
	private Environment environment;

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		AnnotationAttributes attributes = AnnotationAttributes.fromMap(importingClassMetadata.getAnnotationAttributes(EnableSpringSeedJpa.class.getName()));
		String beanNamePrefix =  attributes.getString("beanNamePrefix");
    
    //register bean
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(Dbcp2DataSourceFactoryBean.class)
			.addPropertyValue("prefix", propertyPrefix)
			.addPropertyValue("environment", this.environment)
			.setDestroyMethodName("close");
		String dataSourceName = beanNamePrefix.isEmpty()? "dataSource": beanNamePrefix + "DataSource";
		registry.registerBeanDefinition(dataSourceName, builder.getBeanDefinition());
```
The most important interface is the ```ImportBeanDefinitionRegistrar```, with this class we can register beans dynamically. Also note that how
we get the annotation values from ```AnnotationAttributes```.  
First, we use the **BeanDefinitionBuilder** to build a DataSource factory bean, then we set the parameters needed to construct this factory bean.
Just like ordinary factory beans, this factory bean will build a real DataSource object.  
Finally, we call
```
registry.registerBeanDefinition(dataSourceName, builder.getBeanDefinition());
```
to register the name and the bean.  
In fact, there is another simple way to register bean without ```BeanDefinition```:
```java
		DataSource dataSource = JpaBuilderUtil.newDataSource(environment, prefix);
		beanFactory.registerSingleton(prefix + DataSource.class.getSimpleName(), dataSource);
```
The problem is that the registerred bean will have no Spring bean lifecycle, for example, ```close()``` is not supported. So this way is not recommended for complex beans.  
You can check the [final working codes](https://github.com/profullstack/spring-seed/blob/master/src/main/java/com/profullstack/springseed/core/jpa/SpringSeedJpa.java) here.






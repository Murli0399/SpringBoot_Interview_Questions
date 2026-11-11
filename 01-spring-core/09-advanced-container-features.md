<details><summary>

## [Medium] What do @Lazy and @DependsOn do?
</summary>

**`@Lazy`** – delays creating a singleton bean until it's first needed (instead of at startup). On an injection point, `@Lazy` injects a **proxy** that resolves the real bean on first use, which is also a way to break circular dependencies.

```java
@Service
@Lazy
class HeavyReportEngine { }           // created on first use

@Service
class ReportService {
	ReportService(@Lazy HeavyReportEngine engine) { }   // proxy injected now, real bean later
}
```

Spring Boot can make every bean lazy with `spring.main.lazy-initialization=true` (faster startup in dev, but errors appear later at first request).

**`@DependsOn("beanName")`** – forces other beans to be **initialised first** (and destroyed after), even when there's no direct injection between them. Example: a cache-warming bean that needs the Flyway migration bean to run first.

```java
@Component
@DependsOn("flyway")
class ReferenceDataLoader { }
```

Prefer real dependencies (constructor injection) when possible; `@DependsOn` is for hidden ordering requirements.
</details>

<details><summary>

## [Medium] What are @Conditional annotations?
</summary>

`@Conditional` registers a bean or configuration **only if a condition is true**. Spring Boot's auto-configuration is built on these:

| Annotation | Creates the bean when… |
|---|---|
| `@ConditionalOnClass` / `@ConditionalOnMissingClass` | A class is / isn't on the classpath |
| `@ConditionalOnBean` / `@ConditionalOnMissingBean` | A bean of a type exists / doesn't exist yet |
| `@ConditionalOnProperty` | A property has a given value (`havingValue`, `matchIfMissing`) |
| `@ConditionalOnWebApplication` / `@ConditionalOnNotWebApplication` | The app is / isn't a web app |
| `@ConditionalOnResource` | A resource file exists |
| `@ConditionalOnExpression` | A SpEL expression is true |
| `@Profile("prod")` | A profile is active (built on `@Conditional`) |

```java
@Configuration
class CacheConfig {
	@Bean
	@ConditionalOnProperty(name = "app.cache.type", havingValue = "redis")
	CacheManager redisCacheManager(RedisConnectionFactory cf) { return RedisCacheManager.create(cf); }

	@Bean
	@ConditionalOnMissingBean(CacheManager.class)
	CacheManager inMemoryCacheManager() { return new ConcurrentMapCacheManager(); }
}
```

You can write your own by implementing `Condition.matches(...)` and using `@Conditional(MyCondition.class)`.
</details>

<details><summary>

## [Medium] What is the purpose of @Import?
</summary>

`@Import` pulls other configuration into the current one:
- **`@Configuration` classes** that aren't picked up by component scanning (e.g. in a shared library).
- **`ImportSelector`** implementations that decide at runtime which configurations to load.
- **`ImportBeanDefinitionRegistrar`** implementations that register bean definitions programmatically.

```java
@Configuration
@Import({ SecurityConfig.class, KafkaConfig.class })
class AppConfig { }
```

Many `@Enable...` annotations are just `@Import` underneath: `@EnableScheduling`, `@EnableCaching`, `@EnableAsync`, and `@EnableAutoConfiguration` (which uses an `ImportSelector` to load all auto-configurations). `@ImportResource("classpath:legacy-beans.xml")` imports XML configuration.
</details>

<details><summary>

## [Medium] How do you inject a prototype bean into a singleton bean correctly?
</summary>

Normal injection happens **once**, when the singleton is created, so the singleton keeps **the same prototype instance forever** – the prototype scope is effectively lost.

Ways to get a new instance each time:

```java
@Component
@Scope("prototype")
class ReportBuilder { }

@Service
class ReportService {
	private final ObjectProvider<ReportBuilder> builders;          // 1. ObjectProvider (recommended)
	ReportService(ObjectProvider<ReportBuilder> builders) { this.builders = builders; }

	Report build() {
		ReportBuilder fresh = builders.getObject();                 // new instance every call
		return fresh.build();
	}
}
```

2. **`@Lookup` method injection** – an abstract (or stub) method that Spring overrides to return a new bean.
3. **`jakarta.inject.Provider<ReportBuilder>`** – standard alternative to `ObjectProvider`.
4. **Scoped proxy** – `@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)`; careful, each **method call** on the proxy then hits a new instance.
5. `applicationContext.getBean(ReportBuilder.class)` – works but couples your code to Spring.

The same problem (and solution) applies to request/session-scoped beans used inside singletons; the `@RequestScope`/`@SessionScope` annotations add a scoped proxy automatically.
</details>

<details><summary>

## [Medium] What are Aware interfaces?
</summary>

`*Aware` interfaces let a bean receive container infrastructure objects through a callback during initialization:

| Interface | Gives you |
|---|---|
| `BeanNameAware` | The bean's own name |
| `BeanFactoryAware` | The `BeanFactory` |
| `ApplicationContextAware` | The `ApplicationContext` |
| `EnvironmentAware` | The `Environment` (properties, profiles) |
| `ResourceLoaderAware` | A `ResourceLoader` |
| `ApplicationEventPublisherAware` | The event publisher |

In modern code you rarely need them: you can simply **inject** `ApplicationContext`, `Environment`, `ApplicationEventPublisher` or `ResourceLoader` through the constructor. Aware interfaces are mostly used inside frameworks and libraries.
</details>

<details><summary>

## [Medium] How does Spring resolve @Value and property placeholders?
</summary>

`@Value("${app.page-size:20}")` reads `app.page-size` from the **`Environment`**, which merges property sources in precedence order (command-line args, environment variables, `application-{profile}.yml`, `application.yml`, …). The part after `:` is the **default** used if the property is missing. Spring converts the text to the target type (int, boolean, `Duration`, `List`, …).

- `${...}` – property placeholder.
- `#{...}` – **SpEL** expression evaluated at runtime, e.g. `#{${app.timeout-seconds} * 1000}` or `#{@pricingService.defaultRate}`.

```java
@Component
class PagingConfig {
	@Value("${app.page-size:20}") int pageSize;
	@Value("${app.allowed-origins}") List<String> origins;            // comma-separated -> list
	@Value("#{T(java.time.Duration).ofSeconds(${app.timeout-seconds:5})}") Duration timeout;
}
```

For groups of related settings, prefer **`@ConfigurationProperties`** (type-safe, validated, IDE completion). A missing property without a default fails at startup with "Could not resolve placeholder".
</details>

<details><summary>

## [Hard] What is the difference between BeanPostProcessor and BeanFactoryPostProcessor?
</summary>

| | `BeanFactoryPostProcessor` | `BeanPostProcessor` |
|---|---|---|
| Works on | **Bean definitions** (metadata) | **Bean instances** (objects) |
| When | After definitions are loaded, **before any bean is created** | Around each bean's initialization (`postProcessBeforeInitialization` / `postProcessAfterInitialization`) |
| Typical use | Change definitions: resolve `${...}` placeholders (`PropertySourcesPlaceholderConfigurer`), register extra definitions (`ConfigurationClassPostProcessor` processes `@Configuration`/`@Bean`) | Wrap or modify beans: AOP proxies for `@Transactional`/`@Async`, `@Autowired` injection (`AutowiredAnnotationBeanPostProcessor`), `@PostConstruct` handling |

```java
@Component
class TimingProxyPostProcessor implements BeanPostProcessor {
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) {
		if (bean instanceof PaymentGateway gateway) {        // PaymentGateway: a single-method interface
			return (PaymentGateway) (customerId, amount) -> {        // return a wrapper instead of the bean
				long start = System.nanoTime();
				try { return gateway.charge(customerId, amount); }
				finally { System.out.println("charge took " + (System.nanoTime() - start) / 1_000_000 + " ms"); }
			};
		}
		return bean;
	}
}
```

Declare `BeanFactoryPostProcessor` `@Bean` methods as **`static`**, so Spring can create them before the configuration class itself.
</details>

<details><summary>

## [Hard] What is a FactoryBean?
</summary>

A `FactoryBean<T>` is a bean whose job is to **create another object**. When you inject or look up the bean by name, Spring returns the object produced by `getObject()`, not the factory itself. Prefix the name with `&` to get the factory.

It's used for objects with complex creation logic, especially in libraries and older integrations: `LocalContainerEntityManagerFactoryBean`, `SqlSessionFactoryBean` (MyBatis), Spring Data repository factory beans, `ProxyFactoryBean`.

```java
class ClientFactoryBean implements FactoryBean<RestClient> {
	@Override public RestClient getObject() { return RestClient.builder().baseUrl("https://api.example.com").build(); }
	@Override public Class<?> getObjectType() { return RestClient.class; }
}
```

In application code, a plain `@Bean` method is usually simpler.
</details>

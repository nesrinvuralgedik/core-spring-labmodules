# Core Spring and Spring Boot Lab Projects by Spring Academy-VMware

Labs for the Core Spring and Spring Boot courses

To import these labs into our IDE, import the parent pom `lab/pom.xml` as Maven projects or `lab/build.gradle` as Gradle projects.

# RewardNetwork Domain and API
Reward Dining functions as a restaurant loyalty program. 
The application rewards an account for dining at a restaurant participating in the reward network. 
A reward takes the form of a monetary contribution to an account that is distributed among the account's beneficiaries.

# MODULE 1 - Spring Essentials Overview

## Spring Overview Lab
The Rewards Network application using Plain Old Java Objects (POJOs).

### Prerequisites
* **core-spring-labmodules/lab** contains the lab projects.
* Run **./mvnw clean install** (if you use Maven as your build tool).

### Instructions
1) Review project dependencies. There are several dependencies on Spring Framework jars, on Hibernate, DOM4J, etc.

  **<core-spring-labmodules/lab> ./mvnw -pl 10-spring-intro dependency:tree**

2) All projects also have a dependency on a common project called **00-rewards-common**. 
This project contains a number of useful types such as `MonetaryAmount`, `SimpleDate` and `Percentage`.


3) Implementation of the **RewardNetworkImpl** Application Logic

`RewardNetwork` interface is responsible for carrying out `rewardAccountFor(Dining)` operations which is implemented by `RewardNetworkImpl` class.
In this part, the application logic necessary to complete the `rewardAccountFor(Dining)` operation was implemented by delegating to the dependencies (`accountRepository`, `restaurantRepository`, `rewardRepository`).

4) Unit Test the `RewardNetworkImpl` Application Logic

Stubs are used to test the application logic in isolation, without involving external dependencies such as a database or Spring by using JUnit.

# MODULE 2 - Java Configuration

## Java Configuration Lab

1) Created Spring Application Configuration class (`RewardsConfig`) by placing a `@Configuration` annotation. It tells Spring to treat this class as a set of configuration instructions to be used when the application is starting up.
2) Created factory methods annotated with the `@Bean` annotation within the `RewardsConfig`. Each method instantiates and returns one of the beans we will use in our application.
3) Injected `DataSource` through constructor injection to be set by each repository implementation. Since the `DataSource` is defined elsewhere (`TestInfrastructureConfig`), we need to define a constructor for this class that accepts a `DataSource` parameter. 
As it is the only constructor, `@Autowired` is optional. 
4) **Constructor Injection:**
     * We want to abstract the data source to make it easy to test.
     * Safer than field injection, particularly when forcing immutability of injected members through final.
     * Ability to decouple domain POJOs from Spring.
5) Implemented factory methods (each `@Bean` method) to contain the code needed to instantiate its object from implementation classes and set its dependencies. 
     * Spring will assign each bean an ID based on the `@Bean` method name.
     * For best practices, a bean’s name should describe the service it provides.
     * It should not describe implementation details.
     * For this reason, a bean’s name often corresponds to its interface.
     * The return type of each bean method is an interface not an implementation (a concrete class).
6) Unit tested the configuration class (`RewardsConfigTests`) by defining a mock dataSource.
7) Test infrastructure configuration (`TestInfrastructureConfig`)
     * To test our application, each JDBC-based repository needs a `DataSource` to work.
     * It contains a `@Bean` method that returns `DataSourc`e.
     * Created an in-memory HSQL database tables using two SQL scripts (`schema.sql`, `data.sql`) for testing in a separate configuration file in our test tree.
     * The `EmbeddedDatabaseBuilder` references external files that contain SQL statements. 
     * These SQL scripts will be executed when the application starts. Both scripts are on the classpath, so we can use Spring’s resource loading mechanism and prefix both of the paths with `classpath:`.
8) Imported our application configuration file (`RewardsConfig`) to `TestInfrastructureConfig` class by using `@Import` annotation.
     * To import `RewardsConfig`, Spring will create it as a Spring bean and automatically call its constructor passing in the `DataSource` created by `TestInfrastructureConfig`.
     * The test code should have access to all the beans defined in the `RewardsConfig` configuration class. 
     * `TestInfrastructureConfig` class is the master configuration class for our upcoming test.
9) Created the system test class (`RewardNetworkTests`) with `setUp()` method annotated with `@BeforeEach`.
10) Implemented the test setup logic
     * Created a Spring ApplicationContext using `TestInfrastructureConfig` in the `setUp()` method that bootstraps our application.
     * Then get the bean (`rewardNetwork`) from the application context to invoke the application, which represents the entry-point into the Rewards application.
11) Tested the setup by running an empty test.
     * Added a `testRewardForDining` method and annotate it with `@Test`, no code in the method body.
     * Ran the test to see if our `setup()` is working.
12) Ran a real test
     * Copy the unit test (the `@Test` method) from `RewardNetworkImplTests#testRewardForDining()` under `rewards.internal` test package.
     * Tested the same code, but using a different setup.
     * Ran the test to see if it passes.

# MODULE 3 - More on Java Configuration
No Lab for this module.

# MODULE 4 - Component Scanning 

## Annotations and Component Scanning Lab

1) Ran `RewardNetworkTests` and verified the integration test runs successfully.
    * In the application configuration class (`RewardsConfig`) all the dependencies were wired up with `@Bean` and constructor arguments were used.
2) Refactoring the current code that uses Spring configuration with `@Bean` methods so that it uses annotation and component-scanning instead.
    * In the `RewardsConfig` class removed the `@Bean` methods for all beans.
    * Removed the `@Autowired` DataSource.
    * The resulting class contains no methods and no variables.
    * Re-run the test to see it fails. It fails because Spring has no idea how to inject the dependencies anymore.
3) Annotated `RewardNetworkImpl` and wired its dependencies to let it be found in component-scanning and create a Spring bean from this class.
    * Annotated it with the `@Service` stereotype annotation. Because it is reflective of domain logic that could be exposed as a service. 
    * We could have annotated with `@Component` as well. All stereotype annotations derive from `@Component`. 
    * The `@Component` annotation is what Spring component scanning looks for to create Spring beans during application startup.
    * Annotated the constructor with `@Autowired` to inject all 3 dependencies (constructor injection) which is highly preferred over field injection. 
      In constructor injection, if there is a single constructor, the usage of `@Autowired` is optional.
    * Or we can annotate the individual private fields for repositories with `@Autowired` (field injection).
4) Annotated `JdbcRewardRepository` and wired its dependencies to let it be detected in component-scanning and load this bean.    
    * Annotated it with the `@Repository` stereotype annotation.
    * Injected dataSource by annotating `setDataSource()` method with `@Autowired` (Setter/method injection). 
    * This will tell Spring to inject the setter with an instance of a bean matching the DataSource type.
5) Annotated `JdbcAccountRepository` and wired its dependencies to let it be detected in component-scanning and load this bean.
    * Annotated it with the `@Repository` stereotype annotation.
    * Injected dataSource by annotating `setDataSource()` method with `@Autowired` (Setter/method injection).
6) Annotated `JdbcRestaurantRepository` and wired its dependencies to let it be detected in component-scanning and load this bean.
    * Annotated it with the `@Repository` stereotype annotation.
    * Injected dataSource by annotating constructor with `@Autowired` instead of a setter. (constructor injection).
    * Note that there are already two constructors, one is no-arg constructor.
7) Set up component scanning to tell Spring to search through our Java classes to find the annotated classes and carry out the configuration.
   * Added the `@ComponentScan("rewards.internal")` annotation to `RewardsConfig` class.
   * This annotation turns on a feature called component scanning which looks for all classes annotated with annotations such as `@Component`, `@Repository` or `@Servic`e and creates Spring beans from those classes.
   * It also enables detection of the dependency injection annotations. 
   * The `rewards.internal` argument is the base package that we want Spring to look from. 
   * This will keep Spring from unnecessarily scanning all packages on the classpath.
   * Re-run the test to see it passes.
8) Used Setter injection for DataSource instead of Constructor injection in `JdbcRestaurantRepository` class.
   * Note that `JdbcRestaurantRepository` has been implemented using a simple cache. Because restaurant data is read often but rarely changes.
   * So restaurant objects are cached to improve performance (see methods `populateRestaurantCache` and `clearRestaurantCache` for more details).
   * The cache works as follows:
     * When `JdbcRestaurantRepository` is initialized, it eagerly populates its cache by loading all restaurants from its DataSource.
     * Each time a finder method is called, it simply queries `Restaurant` objects from its cache.
     * When the repository is destroyed, the cache should be cleared to release memory.
   * Changed the dependency injection style from constructor injection to setter injection by moving the `@Autowired` from the constructor to the `setDataSource` method.
   * Executed `RewardNetworkTests` to verify the test.
     * It failed with a `NullPointerException`.
     * Investigated the stack-trace to see the root cause of the exception.
     * Spring now used the default constructor by default instead of the alternate constructor in `JdbcRestaurantRepository`.
     * This means the `populateRestaurantCache()` is never called.
     * Moving this method to the default constructor will not address the issue as it requires the datasource to be set first.
     * `populateRestaurantCache()` must be executed after all initialization is complete.
9) Handled `populateRestaurantCache` on `@PostConstruct` to fix the error.
    * Added `@PostConstruct` annotation to `populateRestaurantCache` method to cause Spring to call this method during the initialization phase of the life cycle.
    * Spring will invoke it after constructor & setter initialization has occurred and bean gets created.
    * Optionally the `populateRestaurantCache()` call from the constructor may be removed if you like.
    * Re-run the `RewardNetworkTests` to see it passes.
    * Using a post-construct rather than the constructor is a better practice. 
    * Because it goes beyond constructing the object to running application code so is not really a valid construction activity. 
10) Handled clearRestaurantCache on `@PreDestroy`
    * Added a simple print statement in `clearRestaurantCache` to see when it is being invoked.
    * Re-run `RewardNetworkTests` and checked the console output and see there is no message.
    * This means `clearRestaurantCache` is not called and the cache is never cleared.
    * `@PreDestroy` annotation added to `clearRestaurantCache` method to mark this method to be called on shutdown.
    * It is registered for a destruction lifecycle callback and invoked before a bean gets destroyed.
    * Re-run `RewardNetworkTests` one more time and see the output message to the console, meaning it was called.

# MODULE 5 - Inside the Spring Container
No Lab for this module.

# MODULE 6 - Introducing Aspect Oriented Programming

## Aspect Oriented Programming Lab

In this lab we will fulfill two non-functional requirements in the Rewards application by using Spring AOP.

* **REQUIREMENT 1:** Create a simple logging aspect for repository find methods.
* **REQUIREMENT 2:** Implement an `@Around` Advice which logs the time spent in each of our repository update methods.


1) Annotated the `rewards.internal.aspects.LoggingAspect` class with the `@Aspect` annotation. 
   * The `@Aspect` annotation will indicate this class is an aspect that contains cross-cutting behavior called "advice" that should be woven into our application.
2) Made `LoggingAspect` class as a component (Spring managed bean) with `@Component` annotation.
   * The `@Aspect` annotation marks the class as an aspect, but it is still not a Spring bean. 
   * Component scanning can be very effective for aspects, so we marked this class with the `@Component` annotation.
3) Optionally placed an `@Autowired` annotation on the constructor where `MonitorFactory` dependency is being injected. 
   (It is optional since there is only a single constructor in the class.)
4) Defined logging aspect pointcut expression and its advice
   * Added `@Before` advice annotation on the `implLogging()` method in `LoggingAspect` class.
   * It takes a `JoinPoint` object as a parameter, and logs information about the target objects invoked during the application execution.
   * We will not log every method of our application, only a subset.
   * Defined a pointcut expression that matches all the `find*` methods, such as `findByCreditCard()`, in the `AccountRepository`, `RestaurantRepository`, or `RewardRepository` interfaces.
     `@Before("execution(* rewards.internal.*.*Repository.find*(..))")`
5) Configured Spring to weave the Logging Aspect into our application.
   * Inside the `config/AspectsConfig` configuration class, added class-level `@ComponentScan("rewards.internal.aspects")` annotation to scan for components ONLY in the specified package.
   * This annotation will cause our `LoggingAspect` to be detected and deployed as a Spring bean.
   * Added `@EnableAspectJAutoProxy` annotation to `AspectsConfig` class to instruct Spring to process beans that have the `@Aspect` annotation by weaving them into the application using the proxy pattern.
   * Note that this annotation is redundant for Spring Boot application since it will be automatically added through Auto Configuration.
6) Plugged in logging aspect to application system test config
   * Modified the `@Import` to include `AspectsConfig.class` in the `SystemTestConfig` configuration class.
7) Tested the Aspect Implementation
   * Ran the `LoggingAspectTests` to see it passes and writes one line of `LoggingAspect` output in the console.
   * Saw the logging output, that means our aspect is being applied.
8) Implemented an `@Around` Performance Monitor Aspect
   * Added `@Around` advice annotation on the `monitor(ProceedingJoinPoint)` method in `LoggingAspect` class.
   * Defined a pointcut expression that matches all the `update*` methods (such as `JdbcAccountRepository.updateBeneficiaries(...)` on the `AccountRepository`, `RestaurantRepository`, or `RewardRepository` interfaces.
9) Completed the implementation of `monitor(ProceedingJoinPoint)` advice method in `LoggingAspect` class.
   * Added the logic to proceed (`repositoryMethod.proceed()`) with the target method invocation by returning the target method's return value to the caller.
10) Re-Ran the `RewardNetworkTests` class to see it passes.
   * Modified the `expectedMatches` value in `RewardNetworkTests` class because there should be 4 lines of logging output not 2.
   * The test passed and saw 4 logging information in the console from the `LoggingAspect`, that means our monitoring behavior has been implemented correctly.
11) Created Exception Handling Aspect to log an exception
   * Annotated the method `implExceptionHandling(RewardDataAccessException e)` with @AfterThrowing Advice in the `DBExceptionHandlingAspect` class.
   * The method implExceptionHandling(RewardDataAccessException e) is used in the event of an exception to enable logging.
   * Defined a pointcut expression that matches all the methods in any of the Repository class.
   * `@AfterThrowing(value="execution(* rewards.internal.*.*Repository.*(..))", throwing = "e")`
12) Annotated `DBExceptionHandlingAspect` class as a Spring-managed bean with `@Component` annotation.
   * Since we enabled component scanning in an earlier step, we just wired DBExceptionHandlingAspect class for component scan to be picked up.
13) Ran `DBExceptionHandlingAspectTests` to validate our AOP is working.
   * The test passed with a warning message in the console, this means our exception handling behavior has been implemented correctly.

# MODULE 7 - Testin Spring Applications

## Testin Spring Applications Lab

In this lab we will refactor the `RewardNetworkTests` using Spring’s system test support library (`TestContext framework`) to simplify and improve the performance of our testing system. 
We will then use Spring profiles to define multiple tests using different implementations of the `AccountRepository`, `RestaurantRepository` and `RewardRepository` for different environments.

Specific subjects we will gain experience with:
* JUnit 5
* Spring’s TestContext framework
* Spring Profiles

1) Refactored `rewards.RewardNetworkTests` to use Spring’s `TestContext` framework
   * In `rewards.RewardNetworkTests`, we set up our test-environment using Spring in the `@BeforeEach` setUp method. 
   * So the ApplicationContext gets recreated each time test method gets invoked. This results in inefficient and slow testing performance.
   * Instead we are going to use Spring’s test-extension which is `TestContext` framework.
   * Removed `setUp()` and `tearDown()` methods in the `rewards/RewardNetworkTests` class.
   * To use Spring TestContext Framework, annotated the class with `@SpringJUnitConfig(classes=TestInfrastructureConfig.class)`, which is a composite annotation of `@ExtendWith(SpringExtension.class)` and `@ContextConfiguration(classes=TestInfrastructureConfig.class)`
   * Removed the attribute `context` because it is not needed anymore.
   * Ran the test to see it fails. Because `rewardNetwork` is not injected.
   * Used `@Autowired` to populate the `rewardNetwork` bean from the ApplicationContext.
   * Re-ran the test to verify the test passes.
   * We've successfully reconfigured the rewards integration test, and at the same time simplified our system test by leveraging Spring’s test support.
   * In addition, the performance of our system test has potentially improved as the ApplicationContext is now created once per test case run (and cached). 
2) Configured Repository Implementations using Profiles
   * Modified the test to use different repository implementations - either Stubs or using JDBC.
   * Annotated all `Stub*Repository` classes in `/src/test/java/rewards/internal` with `@Repository` to make them Spring beans.
   * Ran `RewardNetworkTests` to see it fails because we have multiple beans of the same type in the application context - the original JDBC implementations and now the stubs and Spring doesn't know which one to inject (Ambiguity situation).
   * To fix this problem defined two profiles:
     * Assigned the `"jdbc"` profile to all actual `Jdbc*Repository` classes in `src/main/java` by using `@Profile` annotation.
     * In the same way, assigned the `"stub"` profile to all test `Stub*Repository` classes in `src/main/test`.
     * Annotated the `RewardNetworkTests` class with `@ActiveProfiles` to make `"stub"` the active profile.
     * Reran the test to see it passes now.
     * Checked the console to see that the stub repository implementations are being used.
     * Noticed that the embedded database is also being created even though we don’t use it. We will fix this soon.
   * Switched the active-profile to `"jdbc"` in `RewardNetworkTests` class.
     * Reran the test to see it still passes.
     * Checked the console again to see that the JDBC repository implementations are being used.
3) Switching between Development and Production Profiles
   * Profiles allow different configurations for different environments such as development, testing, QA (Quality Assurance), UAT (User Acceptance Testing), production and so forth.
   * We will define two new profiles: `"local"` and `"jndi"`.
   * In both cases, we will be using the JDBC implementations of our repositories so two profiles will need to be active at once.
   * The difference between `local` and `jndi` is different infrastructure configuration.
   * We are going to swap between an in-memory test database (local profile) and the "real" database defined as a JNDI resource (jndi profile).
     * Assigned beans to the `"local"` profile in `TestInfrastructureLocalConfig` class.
     * Ran `RewardNetworkTests` class to see it fails because now that the bean `dataSource` is specific to the `local` profile.
     * Added "local" as active profile to the @ActiveProfiles annotation in `RewardNetworkTests` class so the current test uses 2 profiles ('jdbc' and 'local').
     * Reran the test to see it passes now.
     * In `TestInfrastructureJndiConfig` class, a "real" data source using a JNDI lookup has already been set up by using a standalone JNDI implementation.
     * Checked `TestInfrastructureJndiConfig` class to see that the different datasource will be used if the profile = `"jndi"`.
     * Updated `RewardNetworkTests` class update, so it uses profiles 'jdbc' and 'jndi' as active profiles. 
     * Reran the test to see it passes.
     * Checked the console to see what has changed; logging from an extra bean called `SimpleJndiHelper`.
     * Switched the profile back from "jndi" to "local" and reran. Check the console and note that the `SimpleJndiHelper` is no longer used.
4) Further Refactoring to create an inner static class from `TestInfrastructureConfig` (Optional, not applied to the code)
   * When no class is specified, Spring’s test framework will look for an inner static class marked with `@Configuration`.
   * Since the `TestInfrastructureConfig` class is so small anyway, copy the entire class definition, including annotations, to an inner static class within the test class (make the class static).
   * Remove the configuration class reference from the `@SpringJUnitConfig` annotation (no property in the brackets).
   * This is an example of convention over configuration.
   * Run the test to see it should still pass.

# MODULE 8 - JDBC Simplification with JdbcTemplate

## JDBC Simplification with JdbcTemplate Lab

Specific subjects we will gain experience with:
* JdbcTemplate class
* RowMapper interface
* ResultSetExtractor interface

The existing JDBC based repository codebase uses low-level DataSource object directly for performing various database operations, which results in complex and duplicating boilerplate code.
It also requires for mapping the data read from database into the domain objects. This could be a tedious programming task.
In this lab, we are going to refactor this codebase to use Spring-provided `JdbcTemplate` class, which will result in simpler and easy to read code.

1) Refactored `JdbcRewardRepositoryTests` class
   * Used `JdbcTemplate` to query for the number of rows in the `getRewardCount()` method.
   * Used `JdbcTemplate` to query for a map of all column values of a row based on the `confirmationNumber` in the `verifyRewardInserted(RewardConfirmation, Dining)` method.
   * Ran the test class to see it passes.
2) Refactored `JdbcRewardRepository` class to use JdbcTemplate
   * Added a private field of type `JdbcTemplate`.
   * In the constructor, instantiated the `JdbcTemplate` object from the given `DataSource` object and assigned it to the private field we just created.
   * Refactored the `confirmReward(...)` method to use `JdbcTemplate`
     * Used update(String, Object...) method.
   * Refactored the `nextConfirmationNumber()` method to use `JdbcTemplate`
     * Used `queryForObject(String, Class<T>, Object...)` method. 
     * The `Object...` represents a variable argument list allowing us to append an arbitrary number of arguments to a method invocation, including no arguments at all.
   * Ran `JdbcRewardRepositoryTests` to verity it passes.
3) Refactored `JdbcRestaurantRepository` class to use JdbcTemplate and RowMapper for creating Domain Object
   * Added a private field of type `JdbcTemplate`.
   * In the constructor, instantiated the `JdbcTemplate` object from the given `DataSource` object and assigned it to the private field we just created.
   * Refactored the `findByMerchantNumber(..)` method to use `JdbcTemplate` and `RowMapper` for creating a `Restaurant` object.
     * Created a `RowMapper` object, which we will pass as an argument to the `jdbcTemplate.queryForObject(String, RowMapper<T>, Object...)` method.
       * Used a Lambda expression to create a `RowMapper` and then refactored `mapRestaurant` method to use it as a method reference which is the shorthand notation of a lambda expression to call a method.
       * We can also create a RowMapper object two other ways:
         * Create it from an anonymous inner class
         * Write a private inner class and create an object from it.
   * Ran `JdbcRestaurantRepositoryTests` to verity it passes
4) Refactored `JdbcAccountRepository` class to use JdbcTemplate
   * Added a private field of type `JdbcTemplate`.
   * In the constructor, instantiated the `JdbcTemplate` object from the given `DataSource` object and assigned it to the private field we just created.
   * Refactored the `updateBeneficiaries(Account)` method to use `JdbcTemplate`
   * Ran `JdbcAccountRepositoryTests` to verity it passes.
5) Used a `ResultSetExtractor` to Traverse a ResultSet for Creating `Account` Objects
   * Refactored `findByCreditCard(..)` method to use the `query(String, ResultSetExtractor<T>, Object...)` method of the `JdbcTemplate`
   * We can create a `ResultSetExtractor` object in three different ways:
     * Create it as a Lambda expression
     * Create it from an anonymous inner class
     * Write a private inner class and create an object from it
   * Created the `ResultSetExtractor` as a method reference which is the shorthand notation of a lambda expression to call a method.
     * The implementation of the `extractData(ResultSet)` method of the `ResultSetExtractor` object should delegate to the provided `mapAccount(ResultSet)` method for actually creating an `Account` object.
   * Ran `JdbcAccountRepositoryTests` to verity it passes.
6) Refactored the constructor of `JdbcRewardRepository` to get the `JdbcTemplate` injected directly (instead of `DataSource` getting injected)
    * Refactored `RewardsConfig` accordingly by defining `@Bean` for `JdbcTemplate` and using it in the `rewardRepository` to create `JdbcRewardRepository`.
    * Refactored `JdbcRewardRepositoryTests` accordingly by changing `setUp()` method, first creating `JdbcTemplate` and then `Repository`.
    * Ran `JdbcRewardRepositoryTests` to verity it passes.
7) Refactored the constructor of `JdbcRestaurantRepository` to get the `JdbcTemplate` injected directly (instead of `DataSource` getting injected)
    * Refactored `RewardsConfig` accordingly by using predefined `jdbcTemplate` in the `restaurantRepository` to create `JdbcRestaurantRepository`.
    * Refactored `JdbcRestaurantRepositoryTests` accordingly by changing `setUp()` method.
    * Ran `JdbcRestaurantRepositoryTests` to verity it passes.
8) Refactored the constructor of `JdbcAccountRepository` to get the `JdbcTemplate` injected directly (instead of `DataSource` getting injected)
    * Refactored `RewardsConfig` accordingly by using predefined `jdbcTemplate` in the `accountRepository` to create `JdbcAccountRepository`.
    * Refactored `JdbcAccountRepositoryTests` accordingly by changing `setUp()` method.
    * Ran `JdbcAccountRepositoryTests` to verity it passes.

# MODULE 9 - Transaction Management with Spring

## Transaction Management with Spring Lab

In this lab, we will learn how to identify where to apply transactionality and how to apply transactionality to a method.

1) Marked Transactional Boundaries by using annotations to identify where transactionality should be applied and what configuration to use.
   * Added `@Transactional` annotation to `rewardAccountFor(Dining)` method in the `rewards.internal.RewardNetworkImpl` class
   * Made sure to use the annotation from Spring not from Java EE.
   * Adding the annotation will identify this method as a place to apply transactional semantics at runtime.
   * Added a `DataSourceTransactionManager` bean to the `SystemTestConfig` configuration class.
     * Defined a bean named `transactionManager` that configures a `DataSourceTransactionManager` by setting the dataSource property on this bean.
   * Added `@EnableTransactionManagement` annotation to the RewardsConfig file to enable Spring transaction
     * For backwards compatibility with older applications, Spring annotations are not enabled automatically, so we have to turn them on.
2) Verified that our transaction declarations are working correctly by running the `RewardNetworkTests` class from the `src/test/java` source folder.
   * Noticed that DEBUG logging is enabled in `setup()`, so checked the logging output.
   * The test passed and verified that only a single connection was acquired and a single transaction is created.
3) Modified Propagation Behavior
   * Reviewed `RewardNetworkPropagationTests` from the rewards package in the `src/test/java` source folder.
     * It's a simple verification of data in the database and also performs manual transaction management.
     * The test opens a transaction at the beginning, (using the `transactionManager.getTransaction(..)` call).
     * Next, it executes `rewardAccountFor(Dining)`
     * Then rolls back the transaction (manual rollback)
     * And finally tested to see if data has been correctly inserted into the database.
   * Ran the test class to see that it failed because the rollback removed all data from the database, including the data that was created by the `rewardAccountFor(Dining)` method.
   * Changed the propagation level of the transactional attributes of the `rewardAccountFor()` method in `RewardNetworkImpl` to require a NEW transaction whenever invoked.
     * The `@Transactional` annotation will use the default propagation level of `Propagation.REQUIRED` which means that it will participate in any transaction that already exists.
     * Overrode the default propagation behavior with `Propagation.REQUIRES_NEW`.
   * Reran the `RewardNetworkPropagationTests` to verify it passes now.
     * We have verified that the test’s transaction was suspended and the `rewardAccountFor(Dining)` method executed in its own transaction.
4) For Developing Transactional Tests Use `@Transactional` to isolate test cases
   * When dealing with persistent data in a test scenario, it can be very expensive to ensure that preconditions are met before executing a test case.
   * It can also be error-prone with later tests inadvertently depending on the effects of earlier tests.
   * Spring provides some support classes for helping with these issues.
     * Restored Default Propagation(REQUIRED) by removing the propagation attribute of `@Transactional` in `RewardNetworkImpl`.
     * Examine the `@Test` logic in `RewardNetworkSideEffectTests` from the rewards package in the `src/test/java` source folder.
     * Ran this test to see the second test fails. Because the committed results from the first test will invalidate the assertions in the second test.
   * Added `@Transactional` annotation to `RewardNetworkSideEffectTests` at the class level and reran the test to see it passes now.
     * Spring has a facility to help avoid the corruption of test data in a DataSource.
     * Simply annotate each test method with `@Transactional` or put `@Transactional` at the class level to apply to all tests in the class.
     * Spring provides Automatic Rollback in Transactional Tests by wrapping each test case in its own transaction and rolls back that transaction when the test case is finished.
     * The data is never committed to the tables and therefore, the database is in its original state for the start of the next test case.

# MODULE 10 - Spring Boot Feature Introduction

## Spring Boot Feature Introduction Lab

In this lab we will create a Spring Boot project from the beginning, taking advantage of the Spring starters to easily configure our project

### Use Case

We will create a very simple application to access and display information about the accounts in our Reward database.

In a traditional Spring project, we enjoy the developer productivity provided by the framework to simplify application development. 
However, configuring a project with the right dependencies to ensure we have all the right Spring libraries can be challenging and time-consuming.

Consider the necessary setup just to configure a JDBC project.

1) We need to obtain the right JDBC drivers for the database
2) We need to obtain the core Spring and Spring JDBC libraries
3) We need to configure our JDBC connections and potentially initialize
4) We may wish to add JDBC connection pooling for efficiency
5) We may need to set up database initialization with schema definitions and data population

The most basic Spring Boot application really only requires the following three files to get started.
1) `pom.xml`: Setup Spring Boot (and any other) dependencies (Could be a `build.gradle` file for Gradle)
2) `application.properties`: General configuration such as Database setup
3) `Application class`: Application launcher

* We will use the `Spring Initializr` to create a basic `JDBC Spring Boot Project` using an `HSQLDB (Hyper SQL)` database. 
* We will make a few minor modifications to add SQL scripts to create tables and populate the database with data for the `Reward Network` application. 
* The initial application will use a very simple `JDBC` call to query the `T_ACCOUNT` table and display the total number of accounts. 
* Using Spring Boot, we will do this with a minimal amount of setup and configuration.

### Instructions

1) Created a Spring Boot Project by using `Spring Initializr` website directly ([https://start.spring.io/](https://start.spring.io/)) with the following project values:
   * **Project:** Maven Project
   * **Language:** Java
   * **Spring Boot Version:** 3.3.2 (or latest)
   * **Group:** io.spring.training.boot
   * **Artifact:** 30-jdbc-boot
   * **Name:** 30-jdbc-boot
   * **Description:** First Boot Project
   * **Package name:** rewards
   * **Packaging:** JAR
   * **Java:** 17 (or later version)
   * **Starter Dependencies:** 
     * **JDBC API** - Adds general JDBC support, including the Spring JDBC libraries.
     * **HyperSQL Database** - Adds specific HSQL libraries and drivers needed for the HSQL database.
2) Downloaded the generated zip and imported into the IDE.
   * Open `pom.xml`(or `build.gradle`) and noted the dependencies and the parent reference.
   
     * 3 dependencies are defined:
```xml
      <dependencies>
       <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-jdbc</artifactId>
       </dependency>
       <dependency>
           <groupId>org.hsqldb</groupId>
           <artifactId>hsqldb</artifactId>
           <scope>runtime</scope>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
```
   * Parent reference:
```xml
     <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.2</version>             
     </parent>
```
3) Checked `JdbcBootApplication` class which is annotated with `@SpringBootApplication`. This annotation makes it behave as a Spring Boot application.
4) Set the logging level to DEBUG by adding `logging.level.root=DEBUG` to the `src/main/resources/application.properties` file.
5) Ran the `JdbcBootApplication` successfully and saw `CONDITIONS EVALUATION REPORT` which shows that a set of AutoConfiguration classes including `DataSourceAutoConfiguration` are executed.
6) Normally, a JDBC DataSource is initialized using a JDBC driver class, the URL to the database server and more. 
Since these were not provided explicitly by us, Spring Boot configured a data source of an embedded database just because we have these dependencies in the class path.
This is how autoconfiguration works in a Spring Boot Application.
7) Used SQL Scripts to Initialize Database with Tables and Load the Tables with Data. 
   * Copied the `schema.sql` and `data.sql` files in `src/test/resources/rewards/testdb` of JDBC Spring project to `src/main/resources` folder of our new project. 
   * The two files should reside directly under the `resources` folder.
   * Reran the `JdbcBootApplication` and noticed the console output now shows a bit more taking place.
     * Noted that a `HikariPool` JDBC connection pool was created and configured for the DataSource.
     * Noticed also that the two scripts we copied to the `resources` folder now automatically get executed.
     * Resulting in the database being initialized with the tables defined in the `schema.sql` file and loaded with the data in the `data.sql` file. 
     * We can see the output by searching for _"Executed SQL script"_ string in the console.
     * It turns out that there was something special about the location where we copied the SQL scripts (`src/main/resources`).
     * Because in a Maven or Gradle project, this directory is always on the classpath. 
     * The SQL files were detected and invoked because Spring Boot automatically looks for and runs two files `schema.sql` and `data.sql` if it finds them on the classpath.
     * Alternatively, we could have placed those files anywhere in our project and named them anything we wanted. 
     * In that case, we would need to add the following entries to our `application.properties` file to tell our Spring Boot application where to find these files.
```
       spring.sql.init.schema-locations=<path to your schema SQL file>
       spring.sql.init.data-locations=<path to your data SQL file>
``` 
8) Used `CommandLineRunner` Bean
   * The `run` method of `CommandLineRunner` bean gets executed when a Spring Boot application gets started.
   * Added a `CommandLineRunner` bean in the `JdbcBootApplication` class and used `JdbcTemplate` bean to retrieve number of accounts.
   * Defined Query to select number of accounts from Account table.
   * Used lambda expression to display the result.
   * Reran the `JdbcBootApplication` to see the output line in the console.
   * Here are some questions to consider:
     * **Where did the JdbcTemplate bean come from?**
       The `JdbcTemplate` bean is provided by the Spring Boot framework. When we include the `spring-boot-starter-jdbc` dependency in our `pom.xml`, Spring Boot automatically configures the necessary beans, including the `JdbcTemplate`.
     * **How did the DataSource get created and configured and dependency injected into the JdbcTemplate?**
       1) **Spring Boot Auto-Configuration:** Spring Boot uses autoconfiguration to set up the `DataSource` bean automatically. 
       Spring Boot simplifies development by reducing the need for explicit bean definitions and configurations.
       When the `spring-boot-starter-jdbc` dependency is included, Spring Boot scans the classpath for JDBC drivers and automatically configures a `DataSource` bean based on properties defined in the `application.properties` file.
       2) **Default Configuration:** If we do not provide a specific `DataSource` configuration in the `application.properties`, Spring Boot will use its default settings to create a `DataSource`. 
       This includes defaults for in-memory databases like `H2, HSQLDB, or Derby`.
       3) **Property Configuration:** In a typical Spring Boot application, we would specify the database connection details in the `application.properties` file. 
       4) **Dependency Injection:** Once the `DataSource` bean is created and configured, Spring Boot injects it into the `JdbcTemplate`. 
       This is done automatically by Spring Boot's dependency injection mechanism. 
       The `JdbcTemplate` bean is created by Spring Boot and is injected wherever it is required (like in our `CommandLineRunner` bean in the `JdbcBootApplication` class).
9) Set the logging level to ERROR by adding `logging.level.root=ERROR`
10) Created Customized Banner
    * Created `banner.txt` under `src/main/resources` folder. 
    * Generated a banner text and copied it into the `banner.txt` file.
    * Reran the application to see the previous output with the banner text.
11) Wrote Integration Testing Code
    * Wrote testing code using `@SpringBootTest` annotation in the `JdbcBootApplicationTests`.
    * `JdbcTemplate` is injected to the test by defining a field with `@Autowired` annotation
    * Ran the test to verify it succeeds.

# MODULE 11 - Spring Boot A Closer Look

## Spring Boot A Closer Look Lab

In this lab we will learn about Spring Boot:
* Dependency Management
* Auto-configuration
* Packaging & Runtime
* Spring Boot Testing

Specific subjects we will gain experience with:
* Spring Boot Starter Bill-of-Materials (BOMs)
* Datasource Auto-configuration
* Application deployment artifact packaging
* Simple application runtime with CommandLineRunner
* Simple SpringBootTest

### Use Case
Take a Spring application using `JDBC` and convert it to a Spring Boot application.
The application builds the data access layer using a `Datasource` and a `JdbcTemplate`. 

We will alter that application in three ways:

* Bootify the application, meaning we will wrap the application with Spring Boot, and demonstrate how Spring Boot simplifies our development experience through starters and autoconfiguration.
* Bootify the application's integration test.
* Add a `CommandLineRunner` to demonstrate how Spring Boot can package and run an application with no additional runtime dependencies.
* Demonstrate how to exclude an autoconfiguration.
* Demonstrate external configuration.

We will also need a Terminal or Command window to run Maven or Gradle manually.

### Instructions

1) **Reviewed of a Spring App – Dependencies**
   * Ran the full suite of tests to verify the build is successful and tests pass.
   * Examined the `pom.xml` or `build.gradle` file of the project. 
   * For the project dependencies, there are a set of libraries covering `Spring Framework`, `Spring JDBC` and `HSQLDB` database, and `Spring Test`.
   * There is also the `spring-boot-starter` as this is required for `SpringApplication.run()` whether we use the rest of Spring Boot or not.
2) **Bootified our Spring Application**

   * Refactored to Starters to wrap our app with Spring Boot by adding `Spring Boot Plugin` for Maven to the project's `pom.xml` file (or `build.gradle`)
     * Added the following xml:
        
        ```xml
        <build>
             <plugins>
                 <plugin>
                     <groupId>org.springframework.boot</groupId>
                     <artifactId>spring-boot-maven-plugin</artifactId>
                 </plugin>
             </plugins>
        </build> 

     * The Spring Boot plugin simplifies the process of building and packaging Spring Boot applications.
     * As we build the project, `Spring Boot Plugin` will generate the runtime deployment artifact (_a JAR or WAR file that we can deploy to run our Spring Boot application_) for us through the `repackage` goal for Maven (the `bootJar` task for Gradle).
     * The `repackage` goal is responsible for creating an executable JAR or WAR file, which includes all necessary dependencies, classes, and resources (uber/fat jar).
     * The `repackage` goal is run as part of the Maven `package` goal (the `bootJar` task is run as part of Gradle `assemble` task).
     * Removed the `Spring JDBC` and `Spring Test` dependencies from the project's `pom.xml`(or `build.gradle`) file.
     * Added the Spring Boot Starters for JDBC and Testing (only for Maven).

   * Turned the `RewardsApplication` class into a Spring Boot application by adding `@SpringBootApplication` annotation.
     * `@SpringBootApplication` annotation is a composition of multiple annotations such as `@SpringBootConfiguration`, `@EnableAutoConfiguration`, `@ComponentScan`.
     * Moved the SQL scripts (`schema.sql` and `data.sql`) from `src/test/resources/rewards/testdb` directory to o `src/main/resources/` directory.
        * We are refactoring the directory structure to the default that Spring Boot expects for its life-cycle initialization - specifically for automatic database initialization.
        * In the non-Spring Boot version, the `EmbeddedDatabaseBuilder` in `SystemTestConfig` can specify where the SQL initialization files are found.
        * In Spring Boot applications, the default files are `schema.sql` and `data.sql`, and they must be in the classpath root.
        * We may choose to specify these files using properties as well.
        * The starting point of the project does not have a runner, so the files originally were provided to set up a test data fixture.
        * We are moving the files from `test/resources` to `main/resources`, so we can use the same files in the application runtime to demonstrate the `CommandLineRunner`.
    * Implemented a `CommandLineRunner` in `RewardsApplication` class configured as a Spring bean by annotating `@Bean`.
      * It will query count from `T_ACCOUNT` table and log the count to the console.
      * Added code to use a `JdbcTemplate` to query the number of accounts using SQL query.
      * Spring Boot autoconfigured a `JdbcTemplate` bean automatically.
      * Used the provided logger to log the number of accounts at info level.
      * Ran the application and verified the log message.
      * The Spring Boot `CommandLineRunner` and `ApplicationRunner` abstractions are guaranteed to run at most once before `SpringApplication.run()` method returns. 
      * Multiple runners may be configured, and can be ordered with the `@Order` annotation.
    * Captured properties into a class using `@ConfigurationProperties`
      * `application.properties` file already contains some properties which has prefix like `rewards.recipient`
      * Annotated `RewardsRecipientProperties` class by using `@ConfigurationProperties` with prefix attribute set to `rewards.recipient`
      * Created fields (along with needed getters/setters) that reflect the properties above in the `RewardsRecipientProperties` class.
      * Added `@EnableConfigurationProperties(RewardsRecipientProperties.class)` to `RewardsApplication` class to enable `@ConfigurationProperties` and make it Spring managed bean.
      * There are also two other options to do that:
        * You can add `@ConfigurationPropertiesScan` to `RewardsApplication class` (This is supported from Spring Boot 2.2.1)
        * You can annotate the `RewardsRecipientProperties` class with `@Component`
      * Implemented a new command line runner that logs the name of the rewards recipient by using `RewardsRecipientProperties`.
   * Enabled full debugging in order to observe how Spring Boot performs its autoconfiguration logic.
     * Added `debug=true` property to our `application.properties`. This causes Spring Boot to log everything it does and what autoconfiguration choices it does and does not make.
     * Ran the application and see the Auto-Configuration Report, that is prefixed as `"CONDITIONS EVALUATION REPORT"` with `Positive matches:` in the console output.
        * Checked the logs for `"JdbcTemplateAutoConfiguration matched:"` and `"DataSourceAutoConfiguration matched:"`.
        * Noted that each `@Conditional*` represents a single conditional statement in the `JdbcTemplateAutoConfiguration` and `DataSourceAutoConfiguration` classes.
3) **Bootified our Integration Test**
   * Removed explicit `DataSource` creation in `SystemTestConfig` class used by `RewardNetworkTests`.
   * Refactored `RewardNetworkTests` into a Spring Boot Integration test.
     * Ran this test without making any change, it failed. 
     * Because we removed the explicit `DataSource` creation and the Spring Boot autoconfiguration is not enabled yet.
     * Removed `@ExtendWith` and `@ContextConfiguration` annotations in RewardNetworkTests.
     * Added `@SpringBootTest` annotation to run as a Spring Boot Test.
     * There is no need to specify any configuration classes, because `@SpringBootTest` will automatically component scan for any `@Component` (or `@Configuration`) classes in the current package or below.
     * Since the `SystemTestConfig` class is in the same package, it will be discovered and processed. This includes processing the `@Import` annotation that references the `RewardsConfig` class containing all the other bean definitions.
     * This is considered an End-To-End integration test, including the wired components.
     * Note that in a real production application we are most likely to configure an external database. Spring Boot offers properties to do this.
     * Ran the `RewardNetworkTests`, it passed now.
     
4) Override Auto-Configuration
   * We have seen to this point that Spring Boot will detect and automatically configure a `DataSource` on our behalf. 
   * But if we need to configure multiple databases from our application, autoconfiguration cannot really help us.
   * We have two options to handle this situation:
     1) Use the default `DataSource` autoconfiguration with supporting default configuration, and also explicitly set additional `DataSource` beans with different names, as specified by the `@Qualifier` annotation. 
       * We can also set the order of precedence using the `@Order` annotation.
     2) Disable autoconfiguration for `DataSource`, and explicitly declare multiple `DataSource` beans using Java Configuration.

5) Disabled `DataSource` Auto-Configuration Programmatically.
   * Switched back to explicit `DataSource` configuration.
   * Added `DataSource` bean explicitly in the `RewardsConfig` class by uncommenting `@Bean` method.
   * Removed the all code above that performs `DataSource` injection.
   * Fixed the compile errors in the `RewardsConfig` class.
   * Noticed this reverts to the standard Spring way of building a DataSource.
   * Disabled `DataSource` autoconfiguration to use our own `DataSource` bean by annotating the `@SpringBootApplication` with `exclude` attribute for `DataSourceAutoConfiguration.class`.
   * Ran this application to observe it failed to start.
   * Imported the `RewardsConfig` configuration to fix the error. 
   * This import is required since the `RewardsConfig` configuration now provides `DataSource` bean and will not be auto-detected through component scanning because it is in a different package than the application. It must be in the same package or sub-packages with the application class.  
   * Ran the application again and observed a successful execution.
   * Technically we don't have to disable `DataSource` autoconfiguration given that Spring Boot will use application defined `DataSource` bean over autoconfigured one.
   * Changed the logging level of `config` package to `DEBUG` in `application.properties`.
   * Reran the `RewardNetworkTests` test to see that DataSourceAutoConfiguration being excluded in the debug logging message.

6) Built and Ran Using Command Line Tools
   * To see what the Spring Boot Maven/Gradle plugin is doing:
     * From our `Terminal/Command` window, executed the `Maven package goal` (or `Gradle assemble task`) from the project root lab directory.
       ```
        mvnw clean package -pl *common -pl *jdbc-autoconfig -Dmaven.test.skip=true
       or
        gradlew :32-jdbc-autoconfig:clean :32-jdbc-autoconfig:assemble
       
    * In `32-jdbc-autoconfig`, a `target` directory (for Maven), or a `build/libs` directory (for Gradle) created. 
      * There are two generated JAR files:
        * `32-jdbc-autoconfig-5.3.23.jar` and
        * `32-jdbc-autoconfig-5.3.23.jar.original` (for Maven) / `32-jdbc-autoconfig-5.3.23-original.jar` (for Gradle)
      * The "original" is much smaller. The other JAR is executable and contains all the necessary dependencies (hence it is called a "fat"/uber JAR!)
    * Extract the jar file to a temporary directory, and view the contents.
    * We should see the jar is generated to be run as a standalone application on our behalf.
        * We will see the classpath resources, manifest file and supporting compile scope package classes are included.
        * Contains all the necessary runtime dependencies - `BOOT-INF` holds all our compiled classes and all the dependency jars. 
        * In the `META-INF` directory `MANIFEST.MF` declares a main entry point (the Main-Class: property)
    * There are many ways to run this application, either directly using the JAR, using `spring-boot:run` goal from Maven or in our IDE as we did earlier.
        * Ran `java -jar 32-jdbc-autoconfig-5.3.23.jar`, got the same output as before.

# MODULE 12 - Spring Boot - Spring Data JPA

## Spring Boot - Spring Data JPA Lab

In this lab we will learn:
* How to implement a `Spring JPA` application using Spring Boot.
* How to create `Spring Data` repositories using `JPA`.

Specific subjects we will gain experience with:
* `Spring Data JPA`
* `Hibernate ORM`

### Use Case

In this lab we will replace `Account` and `Restaurant` JDBC repositories with use of `Spring Data JPA`.
The scope of the lab is not to refactor from JDBC. 
This lab will include pre-annotated data entity classes, and empty repository references that we will implement JPA backing entities, 
and accompanying repositories to demonstrate the ease of implementation.


### Instructions

1) Checked JPA dependencies for the project.
   * Reviewed the dependency for the `Spring Boot Data JPA Starter` in the `pom.xml` (or `build.gradle`).
   * Some of the major functionality of `Spring Boot Data JPA` starter:
     * Repository interface
       * `Spring Data Commons` gives functionality to declaratively define an interface for a repository without requiring an explicit implementation class.
       *  Spring Data will implement the behavior via a Spring proxy.
     * `JPA`
     * `Hibernate`
     * `JDBC`
     * `Transactions`
     * `AOP` 
       * It uses aspects behind the scenes to wire transaction managers and other cross-cutting behavior to our Spring proxies.
2) Given JPA includes JDBC by default.
   * We may remove the JDBC Starter from our `pom.xml`(or `build.gradle`). 
   * Or, we may leave it to explicitly declare the dependency given us will also use `JdbcTemplate` in our code.
3) Implemented JPA for Account
   * Reviewed the JPA annotations on the `Account` class to make sure what each does.
     * `@Entity` - Marks this class as a JPA persistent class.
     * `@Table` - Specifies the exact table name to use on the DB (would be "Account" if unspecified).
     * `@Id` - Indicates the field to use as the primary key on the database.
     * `@Column` - Identifies column-level customization, such as the exact name of the column on the table.
     * No need for `@Column` for `NUMBER` and `NAME` columns. These fields will automatically be mapped using the same name.
     * `@OneToMany` - Specifies a one-to-many relationship between two entities. This means that one `Account` can have many `Beneficiary` entities associated with it. 
                      Identifies the field on the 'one' side of a one-to-many relationship, and it is placed the `Set<Beneficiary>` field.
     * `@JoinColumn` - Identifies the column on the 'many' table containing the column to be used when joining.  Usually a foreign key.
                       `ACCOUNT_ID` is the foreign key in `Beneficiary` table.
   * **What might be a concern when annotating our domain classes with JPA behavior?**
     * We are tying JPA and ORM to our domain entities.
     * We are requiring JPA and ORM coupled with our runtime.
     * In reasonably complex projects we may want to keep JPA coupled code independent of POJO domain objects, 
       and use either `Proxies or Adapters` design patterns to decouple them.
   * Implemented the `AccountRepository` to be a `Spring Data JPA Repository` Interface.
     * Altered this interface to extend the `Repository<Account, Long>` interface.
     * Refactored the finder method on this class to obey Spring Data conventions. 
   * Reviewed the JPA annotations on the `Beneficiary` class to make sure what each does.
     * `@AttributeOverride` - Tells JPA to use the `ALLOCATION_PERCENTAGE` column on `T_ACCOUNT_BENEFICIARY` to populate `Percentage.value` 
        and `SAVINGS` column on `T_ACCOUNT_BENEFICIARY` to populate `MonetaryAmount.value`.
4) Implemented JPA for `Restaurant`
   * Mapped the `Restaurant` class using JPA annotations.
     * Annotated the `Restaurant` class with `@Entity` to mark this class as a JPA persistent class.
     * Annotated with `@Table` to specify the exact table name on the DB.
     * Annotated with `@Id` to identify the primary key on the database.
     * Annotated with `@Column` to identify the exact name of the columns on the table.
     * Annotated with `@AttributeOverride` to use the `BENEFIT_PERCENTAGE` column on `T_RESTAURANT` to populate `Percentage.value`
   * Implemented the `RestaurantRepository` to be a `Spring Data JPA Repository` Interface.
     * Altered this interface to extend the `Repository<Restaurant, Long>` interface.
     * Refactored the finder method on this class to obey Spring Data finder naming conventions so Spring Data will implement it automatically for us.
     

5) Configured for JPA

If we are building a basic Spring JPA project, we would also need to configure our project to enable the use of JPA and Spring Data Repositories by performing the following tasks.
    * Enable wiring of previously defined JPA annotations to an `ORM` backend through an `EntityManager`.
    * Enable use of `Hibernate` as the ORM backend. Hibernate would be wired to the datasource that is autoconfigured.
    * Enabled scanning for JPA repositories using the `@EnableJpaRepositories` annotation.
    * Set up a `TransactionManager` and enable transactions with `@EnableTransactionManagement`.

However, since this is a Spring Boot enabled test and Autoconfiguration is enabled, we will find that it is not necessary to do any of these. 
This allows us to focus on configuring just the custom values needed to override default configurations.

The previous steps cover a set of reasonable defaults for wiring together an application to use JPA and ORM.

**But what if we want to override or extend behavior?**

* Configured JPA in `src/test/resources/application.properties` as follows:
  * Defined properties to make Spring Boot to run SQL scripts (`test-schema.sql` and `test-data.sql`) located under `rewards.testdb` directory. 
```
    spring.sql.init.schema-locations=/rewards/testdb/test-schema.sql
    spring.sql.init.data-locations=/rewards/testdb/test-data.sql
```
  * Defined Spring Boot properties to make JPA show the SQL it is running nicely formatted.
```
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

  * Defined a Spring Boot property to prevent hibernate from auto-creating and auto-populating database tables, our scripts did it already. (Bypassed DDL execution on startup)
```
spring.jpa.hibernate.ddl-auto=none
```

6) Ran `RewardNetworkTests` to verify it succeeds. Reviewed the console carefully for hibernate execution of SQL statements formatted.

# MODULE 13 - Web Application with Spring Boot

## Web Application with Spring Boot Lab

In this lab we will implement a `Spring MVC REST Controller` to fetch account information and return results to the user.

What we will learn:
* How to set up required `Spring MVC` infrastructure with `Spring Boot`
* How to expose `Controllers` as endpoints mapped to web application URLs by creating `Spring MVC REST Controller`
* Implementing handlers that handle `HTTP GET` request
* Using proper annotation for extracting value from the URL
* Exercising Spring Dev Tools
* Changing a port number of a Web application using `server.port` property
* Writing assertions in the test code

Specific subjects we will gain experience with:
* `Spring Boot for Web`
* `@RestController`

### Use Case

For this and subsequent labs we have added account management functionality to the Rewards application.
An `AccountManager` service-layer class has been added for fetching account details. Later we will extend it to support updating accounts.

### Instructions

1) Checked the Project is working.
   * Opened `pom.xml` (or `build.gradle`) to check the dependencies we are using. There are Spring Boot starters for:
        * `Web` - This will allow Spring Boot set up the full Spring MVC environment for our new controller to use.
        * `Devtools` - Enables automatic restart of the application whenever we change a Java file or resource.
        * `JPA` - For convenience the `AccountManager` is implemented using `JPA` to access the account table in the database.
        * `Mustache` - The home page is a minimal web-page just to show the application is working, implemented using server-side rendering and `Mustache templates`. 
    * Ran the application as a Spring Boot or Java application.
       * Once it is running accessed the home page: http://localhost:8080 in our browser.
       * Clicked "List accounts as JSON" link in the homepage and note that it returns 404 - Not Found. 
       * Because it's not implemented yet.
2) Implemented a REST Controller
   * Added `@RestController` annotation to make `AccountController` class a REST controller. So, Spring MVC knows it is a controller for handling REST requests.
     * `@RestController` annotation is internally annotated with `@Controller` and `@ResponseBody`. 
     * `@ResponseBody` is used to indicate that the return value of a controller method should be written directly to the HTTP response body.  
     * Note that the `AccountManager` is already available - it will be injected by Spring when the controller is instantiated.
   * Added `@GetMapping("/accounts")` annotation to make the `accountList` method RESTful and handle `"/accounts"` endpoint.
   * Implemented the `accountsList` method to find and return all accounts.
     * Used "accountManger" object to get all accounts and return them from the method.
     * Recompile this class if necessary, and wait for the application to restart (Spring Boot Devtools causes the restart automatically)
     * Returned to the home-page in my browser and clicked on the "List accounts in JSON" link and saw the accounts in JSON format all is well.
     * We can also use `curl` or `Postman` to make a `GET` request to http://localhost:8080/accounts.
     * You might find it useful to add JSON pretty-print capability to your browser.
   * Unit tested the `AccountController` in `AccountControllerTests` class.
     * The test class uses a `StubAccountManager` managing a single test Account.
     * We're not going to make any HTTP request. We're just testing the controller in isolation.
     * In the setUp method, we're going to create a new AccountController and pass a Stub implementation of Account Manager.
     * StubAccountManager is basically adding a single account into an internal map.
     * Removed `@Disabled` annotation and ran the test to see it passes.
     * We should have tested the Controller before running the application, but as the application was already running we exercised it first. This is not normal development practice - always run tests first!
3) Implemented the `"/accounts/{entityId}"` request handling method to fetch an individual Account.
   * Added a new Controller method `accountDetails` to fetch just a single account by its entity id in `AccountController` class.
   * Annotated with `@GetMapping` to define URL mapping for `"/accounts/{entityId}"`
   * Defined a method parameter to obtain the URI template parameter needed to retrieve an account by using `@PathVariable("entityId")` annotation.
   * Used `accountManager.getAccount(entityId)` to obtain an account.
   * Added a new test code which calls the `accountDetails()` method on the controller.
     * Added code to `testHandleDetailsRequest()` to invoke the new method on the controller.
     * Verified the results by defining some assertions such as null check, matching checks:
     * Removed the `@Disabled` annotation, ran the test, it passed.
   * Reran the application as a Java or Spring Boot application.
     * Using your Browser, Postman or curl try the following URLs:
        * http://localhost:8080/accounts/0
        * http://localhost:8080/accounts/1
4) Made this server listen on port 8088.
   * Set the Spring Boot property to make the server listen on port 8088 in `application.properties`.
     ```
     server.port=8088
     ```
   * Once the server restarted, accessed to http://localhost:8088.

# MODULE 14 - RESTful Application with Spring Boot

## RESTful Application with Spring Boot Lab

What we will learn:
* Working with `RESTful URLs` that expose resources
* Mapping request and response bodies using `HTTP message converters`
* Use `Spring MVC` to implement server-side `REST`
* Writing a programmatic HTTP client to consume `RESTful web services`

Specific subjects we will gain experience with:
* Processing URI Templates using `@PathVariable`
* Using `@RequestBody` and `@ResponseBody`
* Using the `RestTemplate`

### Use Case

* With the emergence of the Single Page Application (SPA) architecture and with the need to support diverse set of client types including mobile devices, more and more back-end applications are developed and deployed as RESTful web services. 
Spring makes building RESTful web services very easy with a rich set of annotations.

* In the prior MVC lab, we implemented a few RESTful endpoints for obtaining a list of accounts and details for a specific account. 
* In this lab, we will complete the implementation of the `AccountController` 
by adding RESTful methods to create a new account, add a beneficiary to an account and delete an account.

### Instructions

1) Exposing Accounts and Beneficiaries As RESTful Resources using Spring’s URI template support, HTTP Message Converters and RestTemplate

   * Inspected the Current Application in `RestWsApplication` class in the accounts package.
     * Checked to see how the application is bootstrapped: 
       * It imports the `AppConfig` configuration class, which contains an `accountManager` bean that provides transactional data access operations to manage Account instances.
       * As `RestWsApplication` uses the `@SpringBootApplication` annotation, it will use component scanning and will define a bean for the `AccountController` class.
     * `AccountClientTests` JUnit test case under the `src/test/java` source folder is to interact with the running RESTful web services on the server.
     * Ran the Spring Boot application in `RestWsApplication` class and verified that the application deployed successfully by accessing the home page from a browser.

   * Reviewed implementation for getting a list of accounts (`accountSummary` method in `AccountController` class)
     * Noted the URI and the HTTP verb used to perform this action, also what is being returned.
     * Received a response using a `JSON` representation (`JavaScript Object Notation`).
     * The reason is that the project includes the `Jackson` library on its classpath.
     * So, an `HTTP Message Converter` that uses Jackson will be active by default.
    
   * Retrieved the List of Accounts Using a RestTemplate to test the `accountSummary()` REST endpoint.
     * A client can process this JSON response anyway it sees fit.
     * In our case, we’ll rely on the same HTTP Message Converter to deserialize the JSON contents back into Account objects.
     * In `AccountClientTests` class:
       * This class uses a plain `RestTemplate` to connect to the server.
       * Removed the `@Disabled` on `listAccounts()` test method.
       * Used the `restTemplate.getForObject` to retrieve an array containing all Account instances from the server by using `BASE_URL` variable.
       * We cannot assign to a `List<Account>` here, since `Jackson` won’t be able to determine the generic type to deserialize, therefore we use an `Account[]` instead.
       * Ran the test to see it passes.
        
   * Reviewed the REST endpoint for the `accountDetails()` method in `AccountController` class
     * Noted the URI and the HTTP verb used to perform this action, also what is being returned.
     * Tested the code by accessing account 0 from our browser to verify the result.
     * Received a single Account in JSON format.

   * Retrieved the Account with id 0 using a URI Template to test the `accountDetails()` REST endpoint.
     * In `AccountClientTests` class:
         * Removed the `@Disabled` on `getAccount()` test method.
         * The RestTemplate also supports URI templates with variables, so use urlVariables varargs parameter.
         * Used the `restTemplate.getForObject` to retrieve an Account from the server by using `BASE_URL` variable.
         * Ran the test to see it passes.

2) Created a new Account Using POST
   * Made a REST endpoint from `createAccount` method in the `AccountController` class.
   * Updated the `createAccount` method by adding a `@PostMapping` to `"/accounts"`. 
   * The body of the POST will contain a JSON representation of an Account.
   * Annotated the account parameter with `@RequestBody` to let the request’s body be un-marshaled into an Account object.
   * When the method completes successfully, the client will receive a `201 Created` instead of `200 OK` 

3) Added Location response header on a Response to URI of the newly created resource and returned it.
   * RESTful clients that receive a `201 Created` response will expect a Location response header in the response containing the URL of the newly created resource.
   * Used `ServletUriComponentsBuilder` to generate the full URL without hard-coding the server name and servlet mapping in `entityWithLocation()` method.
   * Used `ResponseEntity.created(..)` to generate the response. 

4) Tested the `createAccount` REST endpoint
   * Removed the `@Disabled` on this test method.
   * Created a new Account by POSTing to the "/accounts" URL and stored its location in a variable.
   * Used `restTemplate.postForLocation` method that returns the location of the newly created resource
   * Retrieved the new account on the given location. The returned Account will be equal to the one we POSTed, but will also have been given an `entityId` when it was saved to the database.
   * Ran this test to see it passes.

5) Made a REST endpoint from `addBeneficiary` method by adding a Beneficiary with the given Account id, setting its URL as the Location header on the response.
   * Completed the `addBeneficiary` method in the `AccountController`.
   * Added `@PostMapping` annotation to `/accounts/{accountId}/beneficiaries` to respond to a POST.
   * Used `@PathVariable`, a URI template to parse the `accountId`.
   * Extracted a beneficiary name from the incoming request by annotating `@RequestBody`. 
   * The request’s body will only contain the name of the beneficiary. An HTTP Message Converter will convert this to a String.
   * Used `accountManager`'s `addBeneficiary` method to add a beneficiary to an account.
   * Create a `ResponseEntity` containing the location of the newly created beneficiary by using `entityWithLocation()` method again.
   * Returned a `201 Created` status.

6) Made a REST endpoint from `removeBeneficiary` method to remove the Beneficiary with the given name from the Account.
   * Added `@DeleteMapping` annotation to respond to a DELETE to `/accounts/{accountId}/beneficiaries/{beneficiaryName}`
   * Used two `@PathVariable` to parse the `accountId` and `beneficiaryName`.
   * Completed the `removeBeneficiary` method and this time, returned a `204 No Content status`.
   
7) Tested the REST endpoints for `addBeneficiary` in `AccountClientTests` class.
   * Removed the `@Disabled` on `addAndDeleteBeneficiary` test method.  
   * Created a new `Beneficiary` called "David" for the account with id 1. 
   * Posted the String "David" to the "/accounts/{accountId}/beneficiaries" URL.
   * Stored the returned location URI in a variable.
   * Deleted the newly created `Beneficiary` by using `restTemplate.delete(uri)`.
   * Tried to retrieve the newly created Beneficiary again.
   * Ran the test, it passed because we got a 404 Not Found exception.

8) Added an exception handler for `409 Conflict`
   * The current test ensures that we always create a new account using a unique number. 
   * Edited the `createAccount` method in the `AccountClientTests` test class to use a fixed account number, like "123123123".
   * Ran the test: the first time it succeeded.
   * Ran the test again: this time it failed.
   * The server returned a `500 Internal Server Error`.
   * What caused this: a `DataIntegrityViolationException`, ultimately caused by a `ConstraintViolationException` indicating that the account number is violating a unique constraint.
   * This is not really a server error: this is caused by the client providing conflicting data when attempting to create a new account.
   * To properly indicate that to the client, we should return a `409 Conflict` rather than the `500 Internal Server Error` that's returned by default for uncaught exceptions.
   * To enable this, added a new exception handling method that returns the correct code in case of a `DataIntegrityViolationException`. 
   * Added `handleAlreadyExists` method in the `AccountConroller` class.
     * Annotated with `@ExceptionHandler` and `@ResponseStatus` to map `DataIntegrityViolationException` to `409 Conflict`.
   * Ran the test twice again, and we now received the correct status code.

# MODULE 15 - Spring Boot Testing

## Spring Boot Testing Lab

We will learn how to:
* Enable Spring Boot Testing
* Perform integration testing without having to manually run our application
* Perform Web slice testing using mocking

### Instructions

1) **Added Spring Boot Testing Starter**

`pom.xml` (or `build.gradle`) of the parent project already has `spring-boot-starter-test`. 
The starter adds several libraries including `Mockito framework`.

2) **Performed Integration Testing**

   a) **Ran Integration Testing Without Using Spring Boot Testing**
      - Without the use of Spring Boot Testing, we will have to start our application manually before running integration tests. 
      - Stopped any application that is running on port 8080.
      - Ran existing tests in the `AccountClientTests` and observed that they fail. 
      - Started the application in the `BootTestApplication` class. 
      - Reran the tests and observed that they now pass.
      - Stopped the application since we will not need a manually started application.

   b) **Ran Integration Testing Using Spring Boot Testing**
      - Refactored `AccountClientTests` to use Spring Boot Testing.
      - Made `AccountClientTests` class a Spring Boot test class by annotating `@SpringBootTest` with `WebEnvironment.RANDOM_PORT` attribute. 
      - We're choosing random port to start the embedded servlet container which is Tomcat by default.
      - Autowired `TestRestTemplate` bean to a private field as `restTemplate`. It's a component that will be configured and injected by Spring Boot.

   c) **Updated code to use TestRestTemplate**
      - Remove `RestTemplate` from `AccountClientTests` class since it is no longer needed.
      - Changed the value of `BASE_URL` to "" since `TestRestTemplate` will use relative path.
      - Ran the tests and observed that they pass except `addAndDeleteBeneficiary` test.

   d) Modified the `addAndDeleteBeneficiary()` method so that it handles 404 HTTP response status from the server (instead of handling it as an exception as in the case of `RestTemplate`)
      - TestRestTemplate is, by design, fault-tolerant. This means that it does not throw exceptions when an error response (400 or greater) is received.
      - This makes it easier to test error scenarios as, rather than having to catch an exception, you can simply assert that the response's status code is as expected for the scenario in question.
      - Removed the `assertThrows` statement (since we are not going to check if an exception is thrown)
      - Used `getForEntity` method (instead of `getForObject` method) of `TestRestTemplate`
      - Verified that the HTTP response status is 404
      - Ran all tests and they all passed.
      - Observed, in the console log, that Spring Boot Testing framework started an embedded Tomcat server on a random port.

3) **Performed Web Slice Testing**

    a) Got ourselves familiarized with various testing utility classes
   
    ```java
    import static org.mockito.BDDMockito.*;
    import static org.mockito.Mockito.verify;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
    ```
     b) To test Spring MVC Controllers, we can use `@WebMvcTest` which autoconfigures the Spring MVC infrastructure (and nothing else) for the web slice tests.
        - Note that the Web slice testing runs faster than an integration testing since it does not need to start a server.
        - In order to do this part of the lab, we need to know how to use the following API's:
          
          * `BDDMockito`: given(..), willReturn(..), willThrow(..)
          * `MockMvc`: perform(..)
          * `ResultActions`: andExpect(..)
          * `MockMvcRequestBuilders`: get(..), post(..), put(..), delete(..)
          * `MockMvcResultMatchers`: status(), content(), jsonPath(..), header()

    c) Wrote slice test code for the controller with `@WebMvcTest`
        - Replaced `@ExtendWith(SpringExtension.class)` with `@WebMvcTest(AccountController.class)` in `AccountControllerBootTests` testing class under `accounts.web` package in `src/test/java`.
        - `@WebMvcTest` includes `@ExtendWith(SpringExtension.class)` and autoconfigures `MockMvc` bean.
        - Autowired `MockMvc` bean in `AccountControllerBootTests` class. 
        - Now that we have just the web layer, we will use `MockMvc` component to simulate HTTP requests to our controller endpoints as if they were handling real HTTP requests from a client.
        - Created `AccountManager` mock bean using `@MockBean` annotation.

4) **Wrote Test for GET Request for an Account**
    
    a) Wrote positive test for GET request for an account - test for getting account detail for a valid account with `arrange/act and assert/verify` structure as shown below.
    
   ```java
    @Test
    public void testAccountDetails() throws Exception {
    
        // arrange
        given(..).willReturn(..);
    
        // act and assert
        mockMvc.perform(get("/URL-TO-FETCH"))
               .andExpect(..);
    
        // verify
        verify(..).getAccount(0L);
    
    } 
   ```
     
     - The testing should verify the following: 
        * The returned Http status is 200
        * The returned content type is JSON (MediaType.APPLICATION_JSON)
        * The returned data contains correct name and number of an account
        * Make sure you are using the correct imports for @Test and jsonPath

   b) Wrote negative test for GET request for a non-existent account -
      - The testing should verify the following:
        * The returned Http status is 404

   c) Ran the tests to verify they pass.

5) Wrote Test for POST Request for Account - 

   a) Test for creating a new account. The testing should verify the following:
       * The returned Http status is 201.
       * The Location header should contain the URI of the newly created Account.
       * For converting an Account to JSON string, add and use the method asJsonString.

   b) Ran the test to verify it passes.

6) Experiment With `@MockBean` vs `@Mock`
  
  a) Changed `@MockBean` to `@Mock` for the `AccountManager` dependency.
  b) Ran the test to see the test fails.
  c) Because even if we have a mock for accountManager, it's not going to be in the application context.
     Whenever we create the account controller, there won't be any account manager to be injected.
     It's just going to be available inside our test.

7) Tested Get All Accounts

   a) Added another MockMvc test (`getAllAccounts()`) for getting all accounts. The test verified the following:
      * The returned Http status is 200.
      * The returned content type is JSON (MediaType.APPLICATION_JSON).
      * The returned data contains the correct name, number.
      * And contains the correct number of accounts (at least 21 - maybe more due to the POST test creating new accounts)

# MODULE 16 - Securing REST Application with Spring Security

## Securing REST Application with Spring Security Lab

We will learn how to:
* How to implement authentication
* How to restrict access to URLs and/or HTTP methods
* How to test secured application

Specific subjects we will gain experience with:
* Spring Security and the `SecurityFilterChain`
* Built-in and custom `UserDetailsService`
* Security test annotations such as `@WithMockUser` and `@WithUserDetails`

### Instructions

The `RestWsApplication` we built in the previous lab is not secured. We are going to secure it through authentication and authorization.

1) **Explored Default Behaviour of Spring Boot Security** 

    a) Verified the presence of Spring security dependencies in the `pom.xml` for Maven (or `build.gradle` for Gradle).
       When Spring Boot sees `spring-boot-starter-security` on the classpath, it will set up default authentication and authorization - 
       it will set up a single user and all endpoints are secured.
       The `spring-security-test` dependency provides testing support.

    b) Observed the default security behaviour of the Spring Boot application using a browser.

    - Spring Boot application relies on Spring Security’s content-negotiation strategy to determine whether to use Basic authentication or Form-based authentication.
    - When we are using a browser, Spring Boot application will use **Form-based authentication**. 
    - Run this application.
    - Using Chrome Incognito browser, access http://localhost:8080/accounts and observe that a login page gets displayed.
    - Enter `user` in the Username field and Spring Boot generated password into the Password field and verify that the accounts get displayed.
    - Access http://localhost:8080/logout and click Log out button.
   
    c) Observed the default security behaviour of the Spring Boot application using `curl` command (or `Postman`).

    - When we are sending a REST request using `curl` command, Spring Boot application will use **Basic authentication**.
    - When we are sending a REST request without valid username/password pair, the application will respond with "401 Unauthorized".
    - Open a terminal window (if you are using "curl")

        Run `"curl -i localhost:8080/accounts"` and observe 401 Unauthorized response

        Run `"curl -i -u user: localhost:8080/accounts"` and observe a successful response

2) **Enabled Web Security**

    a) Spring Boot's default setup is deliberately minimal. It is not intended for production. Instead, we should configure our own security rules.

    b) Imported security configuration class.

    - The `RestSecurityConfig` class, which is located under `config` package, needs to be explicitly imported since it will not be found through component-scanning from the accounts package.
    - So, we should use `@Import(RestSecurityConfig.class)` annotation in `RestWsApplication` class.


3) **Configured Authentication and Authorization**

Since Web security is enabled, we are ready to configure authentication and authorization.

a) Configured authorization using `requestMatchers` method.

- The most common form of authorization (access control) is through the usage of roles.
  - We are going define 3 roles - `USER`, `ADMIN`, and `SUPERADMIN` with the following access control rules:
    - Allow DELETE on the /accounts resource (or any sub-resource) for "SUPERADMIN" role only
    - Allow POST or PUT on the /accounts resource (or any sub-resource) for "ADMIN" or "SUPERADMIN" role only
    - Allow GET on the /accounts resource (or any sub-resource) for all roles - "USER", "ADMIN", "SUPERADMIN"
    - Allow GET on the /authorities resource for all roles - "USER", "ADMIN", "SUPERADMIN"
    - Deny any request that doesn't match any authorization rule
  - Added the following code to the `filterChain` method in `RestSecurityConfig` class.
    ```java
    http.authorizeHttpRequests((authz) -> authz
    .requestMatchers(HttpMethod.DELETE, "/accounts/**").hasRole("SUPERADMIN")
    .requestMatchers(HttpMethod.POST, "/accounts/**").hasAnyRole("ADMIN", "SUPERADMIN")
    .requestMatchers(HttpMethod.PUT, "/accounts/**").hasAnyRole("ADMIN", "SUPERADMIN")
    .requestMatchers(HttpMethod.GET, "/accounts/**").hasAnyRole("USER", "ADMIN", "SUPERADMIN")
    .requestMatchers(HttpMethod.GET, "/authorities").hasAnyRole("USER", "ADMIN", "SUPERADMIN")
    .anyRequest().denyAll())
     ```

      b) Configured authentication using in-memory storage.
    
      - We are going to set up in-memory `UserDetailsService` and define three users with a corresponding set of roles assigned.
    
          - "user"/"user" with "USER" role
          - "admin"/"admin" with "USER" and "ADMIN" roles
          - "superadmin"/"superadmin" with "USER", "ADMIN", and "SUPERADMIN" roles
    
        - Made sure to store the password in encoded form.
        - Passed all users in the `InMemoryUserDetailsManager` constructor.
        - Added the following code to the `userDetailsService` method in `RestSecurityConfig` class.

        ```java
        UserDetails user = User.withUsername("user").password(passwordEncoder.encode("user")).roles("USER").build();
        UserDetails admin = User.withUsername("admin").password(passwordEncoder.encode("admin")).roles("USER", "ADMIN").build();
        UserDetails superadmin = User.withUsername("superadmin").password(passwordEncoder.encode("superadmin")).roles("USER", "ADMIN", "SUPERADMIN").build();
    
        return new InMemoryUserDetailsManager(user, admin, superadmin);
        ```

        c) Performed security testing against MVC layer.

        - We are going to test if the security configuration works against MVC layer using `@WebMvcTest` and `@WithMockUser` annotations.
        - In the `AccountControllerTests` test class, the tests cover the following scenarios:

            - Using an `invalid` user credential to perform any operation should result in `401 Unauthorized` response.
            - Using `USER` role, we can only perform read operation.
            - Using `ADMI`N role, we can perform create/update operation.
            - Using `SUPERADMIN` role, we can perform delete operation.

        d) Wrote a test that verifies that a user with `USER` role is not permitted to perform a `POST` operation. (`createAccount_with_USER_role_should_return_403`)

        e) Performed security testing against a running server.

        - We are going to test if the security configuration works using end-to-end testing against the application running over the embedded server.
        - The tests cover the similar set of scenarios mentioned above in the `AccountClientTests` test class.
        - Wrote a test that verifies that "user"/"user" is not permitted to create a new Account by using ResponseEntity.

4) **Retrieved Authorities (Roles) for the Logged-In User**

    a) Once authentication is successful, security context is being maintained in the `ThreadLocal` during the processing of a request.

    b) This allows any method in the call stack in the same thread of execution can access the security context even if the security context is not explicitly passed around as an argument to those methods.

    c) Using a `ThreadLocal` in this way is quite safe if care is taken to clear the thread after the present principal’s request is processed. 
       Of course, Spring Security takes care of this for you automatically so there is no need to worry about it.

    d) We are going to access the security context via `SecurityContextHolder` class.

    e) In the `AccountService` class, used the `SecurityContextHolder` to get Security context, which in turn can be used to `getAuthentication` object, which is then used to `getAuthorities`.

    ```java
    Collection<? extends GrantedAuthority> grantedAuthorities 
    = SecurityContextHolder.getContext()
    .getAuthentication()
    .getAuthorities();
    ```

     f) Restart the application (or let Spring Boot Devtools to restart the app).
        
     g) Using Chrome Incognito browser or "curl", access http://localhost:8080/authorities?username=<username> to verify that roles of the logged-in user get displayed.

5) **Implemented Method Security**

    a) Added method security annotation to a method so that the method will be invoked only when both of the following conditions are met:
        - The logged-in user belongs to `ADMIN` role.
        - The value of the username request parameter of the request URL matches the value of the principal's username or authentication's name. 
          This condition can be specified using SpEL (Spring Expression language).

```java
 @PreAuthorize("hasRole('ADMIN') && #username == principal.username")
```

or
        
```java
 @PreAuthorize("hasRole('ADMIN') && #username == authentication.name")
```

b) Enabled method security by adding `@EnableMethodSecurity` annotation in `RestSecurityConfig` class.
   The `prePostEnabled` attribute is set to true by default, this allows the usage of the `@PreAuthorize` annotation.

c) Tested the method security using a browser or curl.

  - Re-ran this application.
  - Using Chrome Incognito browser, 
        - Access http://localhost:8080/authorities?username=user, Enter "user"/"user" and verify that 403 failure occurs
        - If you want to use "curl", use
           curl -i -u user:user http://localhost:8080/authorities?username=user
  - Close the Chrome Incognito browser and start a new one,
      - Access http://localhost:8080/authorities?username=admin, Enter "admin"/"admin" and verify that the roles are displayed successfully
      - If you want to use "curl", use
        curl -i -u admin:admin http://localhost:8080/authorities?username=admin

  - Close the Chrome Incognito browser and start a new one,
      - Access http://localhost:8080/authorities?username=superadmin
      - Enter "superadmin"/"superadmin" and verify that the roles are displayed successfully
      - If you want to use "curl", use
        curl -i -u superadmin:superadmin http://localhost:8080/authorities?username=superadmin

d) Performed method security testing with a running server in `AccountServiceMethodSecurityTest` class.

e) Wrote a test that verifies that getting authorities using http://localhost:8080/authorities?username=superadmin with superadmin/superadmin credential should return three roles ROLE_SUPERADMIN, ROLE_ADMIN, and ROLE_USER in `AccountServiceMethodSecurityTest` class.

6) **Created Custom UserDetailsService**

    a) In the `CustomUserDetailsServic`e class, it needs to implement `loadUserByUsername` method of the `UserDetailsService` interface.
       We would use some kind of persistence storage for maintaining user data but in this lab, for simplicity, we are going to just return hard-coded UserDetails object given a username.
       The custom UserDetailsService maintains UserDetails of two users:
         
      - mary/mary with USER role and
      - joe/joe with USER and ADMIN role

    b) Added authentication based upon the custom `UserDetailsService`. Annotated the `CustomUserDetailsService` class with `@Component` to make it a Spring managed bean.
       Spring Boot will automatically discover it, and it will be used as the `UserDetailsService`.

    c) Removed the `InMemoryUserDetailsManager` definition in `RestSecurityConfig` class by commenting the `@Bean` annotation.

    d) Verified that the newly added custom `UserDetailsService` works.

    - Re-ran this application.
    - Since the custom `UserDetailsService` maintains `UserDetails` on two users, mary/mary and joe/joe, we can now verify that we can access a secured resource using the identity of one of these two users.
        - Using Chrome Incognito browser, 
          - Access http://localhost:8080/accounts/0
          - Enter "mary"/"mary" and verify accounts data gets displayed
        - If you want to use "curl", use
            curl -i -u mary:mary http://localhost:8080/accounts/0
        - Close the Chrome Incognito browser and start a new one.
        - Using Chrome Incognito browser, 
          - Access http://localhost:8080/accounts/0
          - Enter "joe"/"joe" and verify accounts data gets displayed
        - If you want to use "curl", use
            curl -i -u joe:joe http://localhost:8080/accounts/0

    e) Performed security testing for the two users added through custom `UserDetailsService` in `AccountControllerCustomUserDetailsServiceTests` class.

7) **Created custom AuthenticationProvider**

    a) Created custom AuthenticationProvider by implementing `AuthenticationProvider` interface in `CustomAuthenticationProvider` class.
       Typically, we will use a custom authentication system for the verification of the passed identity.
       In this lab, we are just faking it so that it handles a user with the following identity.
         - spring/spring with ROLE_ADMIN role

    b) Added authentication based upon the custom AuthenticationProvider by annotating the `CustomAuthenticationProvider` class with `@Component` to make it a Spring managed bean.
    
    c) Removed the CustomUserDetailsService definition in `CustomUserDetailsService` class by commenting the `@Component` annotation added in a previous task.

    d) Verified that the newly added custom AuthenticationProvider works. Since the custom `AuthenticationProvider` can validate the identity of spring/spring, you can now verify that you can access a secured resource using it.
        - Re-ran this application.
        - Using Chrome Incognito browser, 
          - Access http://localhost:8080/accounts/0
          - Enter "spring"/"spring" and verify accounts data
        - If you want to use "curl", use
            curl -i -u spring:spring http://localhost:8080/accounts/0

    e) Performed security testing for the user added through custom AuthenticationProvider in `AccountControllerCustomAuthenticationProviderTests` class.

# MODULE 17 - Spring Actuator

## Spring Actuator Lab

We will learn how to:
* Configure Spring Boot Actuator
* Expose some or all Actuator endpoints
* Define custom metrics
* Extend the /actuator/health endpoint to add custom health checks

All production applications should have health monitoring, and will often need metric gathering. 
Actuator gives us both of these. We will be enabling actuator in our project and implementing mechanisms to provide an additional custom metric 
to indicate the number of times Account details have been requested. Additionally, you will be creating a custom HealthIndicator and adding that 
to the overall health status that is obtained.

### Instructions

1) In the `pom.xml`(or `build.gradle`) for the actuator project, added the Spring Boot Actuator starter.

```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
          </dependency>
```

2) Reviewed application
    
    a) Ran the application and using a browser open the following links:

      - http://localhost:8080/actuator - worked
      - http://localhost:8080/actuator/metrics - failed (404)
        Even though there are many valid endpoints, only the health endpoint is automatically exposed in Actuator.

    b) If you prefer you can use `curl` or `Postman` to examine the links.

3) **Exposed HTTP actuator endpoints.**

    a) In the `application.properties` file, exposed the `metrics` and `beans` endpoints by setting the appropriate Spring Boot property.

    ```properties
    management.endpoints.web.exposure.include=beans, metrics
    ```   

    b) Restarted the application and accessed beans and metrics endpoints tracked by Actuator.

     - http://localhost:8080/actuator/metrics
     - http://localhost:8080/actuator/beans

    c) Tried fetching the data for one of the metrics by constructing a url based on the name of the metric.
    
    - http://localhost:8080/actuator/metrics/jvm.memory.max

    d) Exposed all HTTP actuator endpoints by modifying the above property.

    ```properties
    management.endpoints.web.exposure.include=*
    ```   

    e) Restarted the application and accessed some metrics such as following.

    - http://localhost:8080/actuator/beans
    - http://localhost:8080/actuator/health
    - http://localhost:8080/actuator/info
    - http://localhost:8080/actuator/mappings
    - http://localhost:8080/actuator/loggers
    - http://localhost:8080/actuator/metrics/jvm.memory.max
    - http://localhost:8080/actuator/metrics/http.server.requests
    - http://localhost:8080/actuator/metrics/http.server.requests?tag=method:GET
    - http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/actuator/beans
    - http://localhost:8080/notexistent (404 is expected)
    - http://localhost:8080/actuator/metrics/http.server.requests?tag=status:404


4) **Changed log level via `./actuator/loggers` endpoint**

    a) Verified the current logging level of the `accounts.web` package is `DEBUG`.

     - http://localhost:8080/actuator/loggers/accounts.web
   
    ```json
    {
      "configuredLevel": null,
      "effectiveLevel": "DEBUG"
    }
    ```

    b) Added a log statement as shown below to the `accountSummary()` method of the `AccountController` class. This message will be used to verify if we can change the logging level without restarting the application.

    ```java
    logger.debug("Logging message within accountSummary()");
    ```
    c) Restarted the application and access "/accounts" URL and verify the log message gets displayed.

    d) Changed logging level of "accounts.web" package to INFO using curl command below (or use Postman)

    ```shell
    curl -i -XPOST -H"Content-Type: application/json" localhost:8080/actuator/loggers/accounts.web -d'{"configuredLevel": "INFO"}'
    ```
    
    e) Accessed "/accounts" URL (WITHOUT restarting the application) and verified the logging message no longer gets displayed.
    
    f) Verified that the effectiveLevel of the `accounts.web` package is now changed to `INFO`.

5) **Publish Build Information**

Spring Boot Actuator’s info endpoint publishes information about our application specified in the `META-INF/build-info.properties` file.
The `META-INF/build-info.properties` can be created by the Spring Boot Maven or Gradle Plugin during build.

   a) Added Maven goal (or Gradle task)

   - If you are using Maven, add `build-info` goal:

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <executions>
    <execution>
      <goals>
        <goal>build-info</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

   - if you are using Gradle, add BuildInfo task:

```groovy
springBoot  {
    buildInfo()
}
```

   - Rebuild the application preferably at the command line:
 
```shell
./mvnw -pl 00-rewards-common -pl 01-rewards-db -pl 44-actuator clean install (for Maven)
./gradlew 44-actuator:clean 44-actuator:build (for Gradle)
```

   - Verify the presence of the `build-info.properties` file as following:

```shell
./target/classes/META-INF/build-info.properties (for Maven)
./build/resources/main/META-INF/build-info.properties (for Gradle)
```

   - Restarted the application and access "info" endpoint and verify the build info gets displayed.

  b) Added additional properties to the info endpoint to expose the Java runtime information.
  The Java info contributor is disabled by default, enabled it by adding the following property in the `application.properties`.
  Also, Added some custom properties to the `application.properties`.

```properties
management.info.java.enabled=true
management.info.env.enabled=true
info.restaurant.location=New York
info.restaurant.discountPercentage=10
```

  c) Restarted the application and accessed "info" endpoint and verified additional properties are displayed.

6) **Defined Custom Metrics - Counter**

By default, Actuator exposes a number of useful but rather generic metrics. 
Often tracking metrics that are specific to our application's domain provides insights into operational, business, or other concerns.
We'll add a custom metric specific to our application that counts the number of times the account finder method is used.

  a)  Added a Micrometer Counter with a tag. 

   - In the constructor of `AccountController`, injected a `MeterRegistry` through constructor injection by adding an instance of `MeterRegistry` as a second parameter.
   - Using the `MeterRegistry`, created a `Counter` called "account.fetch" with a tag of "type"/"fromCode" key/value pair by calling `meterRegistry.counter("account.fetch", "type", "fromCode");`.
   - Stored the counter as a new data-member of `AccountController`.

  b) Incremented the Counter each time `"accountDetails"` method is called. In the `accountDetails()` method, called `counter.increment()`.

  c) Fixed compiler error in `AccountControllerTests` class since we changed the signature of the AccountController's constructor. Passed MeterRegistry object as a second parameter.

  d) Ran the tests in `AccountControllerTests` class to verift they pass. Particularly the `testHandleDetailsRequest()`, which checks that the counter is working correctly.

  e) Verified the result of "account.fetch" metric.
     - Accessed the new "http://localhost:8080/actuator/metrics/account.fetch" endpoint, and verified the presence of "account.fetch" metric.
       This displays 0 since no accounts have been fetched yet.
     - Accessed some accounts (i.e. http://localhost:8080/accounts/1).
     - Viewed the counter value again at http://localhost:8080/actuator/metrics/account.fetch, now the counter increases with each account fetch.
     - If we restart the application, the value of the counter resets to 0 since it's kept in memory.

7) **Defined Custom Metrics - Timer**

    a) Added timer using `@Timed` annotation to the accountSummary(..) method.

    ```java
    @Timed(value="account.timer", extraTags = {"source", "accountSummary"})
    ```

    b) Added timer using `@Timed` annotation to the accountDetails(..) method.

    ```java
    @Timed(value="account.timer", extraTags = {"source", "accountDetails"})
    ```
    c) Restarted the application and verified the timer result.
       - Visited the http://localhost:8080/accounts/1 and http://localhost:8080/accounts for a few times.
       - Visited the (http://localhost:8080/actuator/metrics/account.timer and verified the timer metric.

8) **Got Detailed Health Checks**  
 
    a) Visited the http://localhost:8080/actuator/health endpoint. By default, there is very little info displayed at this endpoint.

    b) Enabled more detailed health info by setting the following property to always displayed in `application.properties` file.

    ```properties
    management.endpoint.health.show-details=always
    ```

    c) Restarted the application and refreshed http://localhost:8080/actuator/health to see more health details.

9) **Created Custom Health Checks**

    a) We can extend the default health checks so that our application reports whether it is down or up based on custom criteria or domain logic.
    
    b) In this case, we will determine the health of the application based on whether there are any restaurants in the database.
       If there are no restaurants, then the health status of the application will be considered DOWN.

    c) Set up tests in `RestaurantHealthCheckTest` class.
       Modified the code to use the `RestaurantHealthCheck.health()` method in each test.

    d) Created custom health indicator in `RestaurantHealthCheck` class by implementing `HealthIndicator` interface.
        - Made this class implements HealthIndicator interface by implementing health() method.
            - If there are one or more restaurants in the database, return Health.up().build(), otherwise return Health.down().build().
        - Made this class a component by annotating `@Component`.
        - Injected `RestaurantRepository` through constructor injection.

    e) Tested custom health indicator in `RestaurantHealthCheckTest` class.
        - Created an instance of `RestaurantHealthCheck` class in the `setup` method.
        - Ran the tests and verified they pass.

    e) Verified the behavior of custom health indicator.
        - Restarted the application.
        - Accessed the health endpoint - our application is DOWN because there are no restaurants in the database.

    f) Verified the behavior of custom health indicator with restaurant data.
        - To populate our database with a Restaurant, changed the `application.properties` to set `spring.sql.init.data-locations` property with `classpath:/data-with-restaurants.sql`.
        - Restarted the application.
        - Accessed the health indicator - it is UP this time.

    g) Verified that all tests in `AccountControllerTests` class pass.

10) **Organized Health Indicators Into Groups**

    a) Created 3 groups: "system", "web", and "application" in `the application.properties` as following:

       - The "system" group includes "diskSpace" and "db" health indicators
       - The "web" group includes "ping" health indicator
       - The "application" group includes "restaurantHealthCheck" health indicator

    b) For "system" and "application" groups, configured "show-details" with "always"

```properties
management.endpoint.health.group.system.include=diskSpace, db
management.endpoint.health.group.system.show-details=always
management.endpoint.health.group.web.include=ping
management.endpoint.health.group.application.include=restaurantHealthCheck
management.endpoint.health.group.application.show-details=always
```

   c) Removed "management.endpoint.health.show-details=always" we added earlier since it is no longer needed.

   d) Restarted the application and accessed health endpoint of each group.

   - http://localhost:8080/actuator/health/system
   - http://localhost:8080/actuator/health/web
   - http://localhost:8080/actuator/health/application

11) **Secured Actuator Endpoints**

    a) Added Spring Boot Security starter to the `pom.xml`(or `build.gradle`) file.

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

   b) Added security configuration to `ActuatorSecurityConfiguration` class.

   c) Configured access control to actuator endpoints as following:
     
- Anybody can access "health" and "info" endpoints
- ADMIN role can access "conditions" endpoint
- ACTUATOR role can access all the other endpoints

```java
    http.authorizeHttpRequests((authz) -> authz
    .requestMatchers(EndpointRequest.to(HealthEndpoint.class, InfoEndpoint.class)).permitAll()
    .requestMatchers(EndpointRequest.to(ConditionsReportEndpoint.class)).hasRole("ADMIN")
    .requestMatchers(EndpointRequest.toAnyEndpoint()).hasRole("ACTUATOR")
    .anyRequest().authenticated())
    .httpBasic(withDefaults())
    .csrf(CsrfConfigurer::disable);
```

   d) Ran the tests checking security configuration for the actuator endpoints in the `AccountClientSecurityTests`.

12) ***Experiment with HealthIndicator**

    a) Changed "spring.sql.init.data-locations" property in the "application.properties" file back to use "classpath:data-no-restaurants.sql".

    b) Instead of returning DOWN when there are no restaurants, defined and used a custom status called NO_RESTAURANTS.

    c) Set "management.endpoint.health.group.application.status.order" property in the "application.properties" file so that NO_RESTAURANTS gets displayed as top-level status for the "application" health group.

```properties
management.endpoint.health.group.application.status.order=NO_RESTAURANTS, DOWN, UP
```
   d) Restarted the application and verified the result.

13) **Used AOP for counting logic**

In general, mixing up different concerns (controller logic and counter logic in this example code) is not considered a good practice:
it violates Single Responsibility Principle. Instead, usage of AOP provides cleaner code.

   a) Added `spring-boot-starter-aop` starter to the `pom.xml` (or the `build.gradle`).

   b) Created an aspect by annotating `AccountAspect` class with `@Component` and `@Aspect`, 
   through which `account.fetch` counter, which has a tag of `type`/`fromAspect` key/value pair, 
   gets incremented every time `accountSummary` method of the `AccountController class` class is invoked.

```java
@Component
@Aspect
public class AccountAspect {

    private Counter counter;

    public AccountAspect(MeterRegistry meterRegistry) {
        counter = meterRegistry.counter("account.fetch", "type", "fromAspect");
    }

    @Before("execution(* accounts.web.AccountController.accountSummary(..))")
    private void incrementCounter() {
        counter.increment();
    }

}
```

   c) Accessed `/accounts` several times and verified the metrics of /actuator/metrics/account.fetch?tag=type:fromAspect.

14) **Accessed Actuator endpoints using JMX**

    a) Since JMX is disabled by default, added "spring.jmx.enabled=true" to the "application.properties".
    
    b) Restarted the application.

    c) In a terminal window, ran "jconsole" (from <JDK-directory>/bin).

    d) Selected "accounts.ActuatorApplication" under "Local Process" then clicked "Connect" and accepted the prompted insecure connection.

    e) Selected the MBeans tab, found the org.springframework.boot folder, then opened the Endpoint sub-folder.

    f) Noted that all actuator endpoints ARE exposed for JMX.

    g) Expanded Health->Operations->health, Clicked "health" button on the top right panel and observed the health data gets displayed.
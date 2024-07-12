# Core Spring and Spring Boot Lab Projects by Spring Academy-VMware

Labs for the Core Spring and Spring Boot courses

To import these labs into your IDE, import the parent pom `lab/pom.xml` as Maven projects or `lab/build.gradle` as Gradle projects.

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

2) All projects also have a dependency on a common project called 00-rewards-common. 
This project contains a number of useful types such as MonetaryAmount, SimpleDate and Percentage.


3) Implementation of the **RewardNetworkImpl** Application Logic

RewardNetwork interface is responsible for carrying out rewardAccountFor(Dining) operations which is implemented by RewardNetworkImpl class.
In this part, the application logic necessary to complete the rewardAccountFor(Dining) operation was implemented by delegating to the dependencies (accountRepository, restaurantRepository, rewardRepository).

4) Unit Test the RewardNetworkImpl Application Logic

Stubs are used to test the application logic in isolation, without involving external dependencies such as a database or Spring by using JUnit.

# MODULE 2 - Java Configuration

## Java Configuration Lab

1) Created Spring Application Configuration class (RewardsConfig) by placing a @Configuration annotation. It tells Spring to treat this class as a set of configuration instructions to be used when the application is starting up.
2) Created factory methods annotated with the @Bean annotation within the RewardsConfig. Each method instantiates and returns one of the beans we will use in our application.
3) Injected DataSource through constructor injection to be set by each repository implementation. Since the DataSource is defined elsewhere (TestInfrastructureConfig), we need to define a constructor for this class that accepts a DataSource parameter. As it is the only constructor, @Autowired is optional. 
4) **Constructor Injection:**
     * We want to abstract the data source to make it easy to test.
     * Safer than field injection, particularly when forcing immutability of injected members through final.
     * Ability to decouple domain POJOs from Spring.
5) Implemented factory methods (each @Bean method) to contain the code needed to instantiate its object from implementation classes and set its dependencies. 
     * Spring will assign each bean an ID based on the @Bean method name.
     * For best practices, a bean’s name should describe the service it provides.
     * It should not describe implementation details.
     * For this reason, a bean’s name often corresponds to its interface.
     * The return type of each bean method is an interface not an implementation (a concrete class).
6) Unit tested the configuration class (RewardsConfigTests) by defining a mock dataSource.
7) Test infrastructure configuration (TestInfrastructureConfig)
     * To test our application, each JDBC-based repository needs a DataSource to work.
     * It contains a @Bean method that returns DataSource.
     * Created an in-memory HSQL database tables using two SQL scripts (schema.sql, data.sql) for testing in a separate configuration file in our test tree.
     * The EmbeddedDatabaseBuilder references external files that contain SQL statements. 
     * These SQL scripts will be executed when the application starts. Both scripts are on the classpath, so we can use Spring’s resource loading mechanism and prefix both of the paths with classpath:.
8) Imported our application configuration file (RewardsConfig) to TestInfrastructureConfig class by using @Import annotation.
     * To import RewardsConfig, Spring will create it as a Spring bean and automatically call its constructor passing in the DataSource created by TestInfrastructureConfig.
     * The test code should have access to all the beans defined in the RewardsConfig configuration class. 
     * TestInfrastructureConfig class is the master configuration class for our upcoming test.
9) Created the system test class (RewardNetworkTests) with setUp() method annotated with @BeforeEach.
10) Implemented the test setup logic
     * Created a Spring ApplicationContext using TestInfrastructureConfig in the setUp() method that bootstraps our application.
     * Then get the bean (rewardNetwork) from the application context to invoke the application, which represents the entry-point into the rewards application.
11) Tested the setup by running an empty test.
     * Added a testRewardForDining method and annotate it with @Test, no code in the method body.
     * Ran the test to see if our setup() is working.
12) Ran a real test
     * Copy the unit test (the @Test method) from RewardNetworkImplTests#testRewardForDining() under rewards.internal test package.
     * Tested the same code, but using a different setup.
     * Ran the test to see if it passes.

# MODULE 3 - More on Java Configuration
No Lab for this module.

# MODULE 4 - Component Scanning 

## Annotations and Component Scanning Lab

1) Ran RewardNetworkTests and verified the integration test runs successfully.
    * In the application configuration class (RewardsConfig) all the dependencies were wired up with @Bean and constructor arguments were used.
2) Refactoring the current code that uses Spring configuration with @Bean methods so that it uses annotation and component-scanning instead.
    * In the RewardsConfig class removed the @Bean methods for all beans.
    * Removed the @Autowired DataSource.
    * The resulting class contains no methods and no variables.
    * Re-run the test to see it fails. It fails because Spring has no idea how to inject the dependencies anymore.
3) Annotated RewardNetworkImpl and wired its dependencies to let it be found in component-scanning and create a Spring bean from this class.
    * Annotated it with the @Service stereotype annotation. Because it is reflective of domain logic that could be exposed as a service. 
    * We could have annotated with @Component as well. All stereotype annotations derive from @Component. 
    * The @Component annotation is what Spring component scanning looks for to create Spring beans during application startup.
    * Annotated the constructor with @Autowired to inject all 3 dependencies (constructor injection) which is highly preferred over field injection. In constructor injection, if there is a single constructor, the usage of @Autowired is optional.
    * Or we can annotate the individual private fields for repositories with @Autowired (field injection).
4) Annotated JdbcRewardRepository and wired its dependencies to let it be detected in component-scanning and load this bean.    
    * Annotated it with the @Repository stereotype annotation.
    * Injected dataSource by annotating setDataSource() method with @Autowired (Setter/method injection). 
    * This will tell Spring to inject the setter with an instance of a bean matching the DataSource type.
5) Annotated JdbcAccountRepository and wired its dependencies to let it be detected in component-scanning and load this bean.
    * Annotated it with the @Repository stereotype annotation.
    * Injected dataSource by annotating setDataSource() method with @Autowired (Setter/method injection).
6) Annotated JdbcRestaurantRepository and wired its dependencies to let it be detected in component-scanning and load this bean.
    * Annotated it with the @Repository stereotype annotation.
    * Injected dataSource by annotating constructor with @Autowired instead of a setter. (constructor injection).
    * Note that there are already two constructors, one is no-arg constructor.
7) Set up component scanning to tell Spring to search through our Java classes to find the annotated classes and carry out the configuration.
   * Added the @ComponentScan("rewards.internal") annotation to RewardsConfig class.
   * This annotation turns on a feature called component scanning which looks for all classes annotated with annotations such as @Component, @Repository or @Service and creates Spring beans from those classes.
   * It also enables detection of the dependency injection annotations. 
   * The "rewards.internal" argument is the base package that we want Spring to look from. 
   * This will keep Spring from unnecessarily scanning all packages on the classpath.
   * Re-run the test to see it passes.
8) Used Setter injection for DataSource instead of Constructor injection in JdbcRestaurantRepository class.
   * Note that JdbcRestaurantRepository has been implemented using a simple cache. Because restaurant data is read often but rarely changes.
   * So restaurant objects are cached to improve performance (see methods populateRestaurantCache and clearRestaurantCache for more details).
   * The cache works as follows:
     * When JdbcRestaurantRepository is initialized, it eagerly populates its cache by loading all restaurants from its DataSource.
     * Each time a finder method is called, it simply queries Restaurant objects from its cache.
     * When the repository is destroyed, the cache should be cleared to release memory.
   * Changed the dependency injection style from constructor injection to setter injection by moving the @Autowired from the constructor to the setDataSource method.
   * Executed RewardNetworkTests to verify the test.
     * It failed with a NullPointerException.
     * Investigated the stack-trace to see the root cause of the exception.
     * Spring now used the default constructor by default instead of the alternate constructor in JdbcRestaurantRepository.
     * This means the populateRestaurantCache() is never called.
     * Moving this method to the default constructor will not address the issue as it requires the datasource to be set first.
     * populateRestaurantCache() must be executed after all initialization is complete.
9) Handled populateRestaurantCache on @PostConstruct to fix the error.
    * Added @PostConstruct annotation to populateRestaurantCache method to cause Spring to call this method during the initialization phase of the lifecyle.
    * Spring will invoke it after constructor & setter initialization has occurred and bean gets created.
    * Optionally the populateRestaurantCache() call from the constructor may be removed if you like.
    * Re-run the RewardNetworkTests to see it passes.
    * Using a post-construct rather than the constructor is a better practice. 
    * Because it goes beyond constructing the object to running application code so is not really a valid construction activityit. 
10) Handled clearRestaurantCache on @PreDestroy
    * Added a simple print statement in clearRestaurantCache to see when it is being invoked.
    * Re-run RewardNetworkTests and checked the console output and see there is no message.
    * This means clearRestaurantCache is not called and the cache is never cleared.
    * @PreDestroy annotation added to clearRestaurantCache method to mark this method to be called on shutdown.
    * It is registered for a destruction lifecycle callback and invoked before a bean gets destroyed.
    * Re-run RewardNetworkTests one more time and see the output message to the console, meaning it was called.
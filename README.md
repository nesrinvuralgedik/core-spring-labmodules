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

5) **Java Configuration Lab**
   * Created Spring Application Configuration class (RewardsConfig) by placing a @Configuration annotation. It tells Spring to treat this class as a set of configuration instructions to be used when the application is starting up.
   * Created factory methods annotated with the @Bean annotation within the RewardsConfig. Each method instantiates and returns one of the beans we will use in our application.
   * Injected DataSource through constructor injection to be set by each repository implementation. Since the DataSource is defined elsewhere (TestInfrastructureConfig), we need to define a constructor for this class that accepts a DataSource parameter. As it is the only constructor, @Autowired is optional. 
   * **Constructor Injection:**
     * We want to abstract the data source to make it easy to test.
     * Safer than field injection, particularly when forcing immutability of injected members through final.
     * Ability to decouple domain POJOs from Spring.
   * Implemented factory methods (each @Bean method) to contain the code needed to instantiate its object from implementation classes and set its dependencies. 
     * Spring will assign each bean an ID based on the @Bean method name.
     * For best practices, a bean’s name should describe the service it provides.
     * It should not describe implementation details.
     * For this reason, a bean’s name often corresponds to its interface.
     * The return type of each bean method is an interface not an implementation (a concrete class).
   * Unit tested the configuration class (RewardsConfigTests) by defining a mock dataSource.
   * Test infrastructure configuration (TestInfrastructureConfig)
     * To test our application, each JDBC-based repository needs a DataSource to work.
     * It contains a @Bean method that returns DataSource.
     * Created an in-memory HSQL database tables using two SQL scripts (schema.sql, data.sql) for testing in a separate configuration file in our test tree.
     * The EmbeddedDatabaseBuilder references external files that contain SQL statements. 
     * These SQL scripts will be executed when the application starts. Both scripts are on the classpath, so we can use Spring’s resource loading mechanism and prefix both of the paths with classpath:.
   * Imported our application configuration file (RewardsConfig) to TestInfrastructureConfig class by using @Import annotation.
     * To import RewardsConfig, Spring will create it as a Spring bean and automatically call its constructor passing in the DataSource created by TestInfrastructureConfig.
     * The test code should have access to all the beans defined in the RewardsConfig configuration class. 
     * TestInfrastructureConfig class is the master configuration class for our upcoming test.
   * Created the system test class (RewardNetworkTests) with setUp() method annotated with @BeforeEach.
   * Implemented the test setup logic
     * Created a Spring ApplicationContext using TestInfrastructureConfig in the setUp() method that bootstraps our application.
     * Then get the bean (rewardNetwork) from the application context to invoke the application, which represents the entry-point into the rewards application.
   * Tested the setup by running an empty test.
     * Added a testRewardForDining method and annotate it with @Test, no code in the method body.
     * Ran the test to see if our setup() is working.
   * Ran a real test
     * Copy the unit test (the @Test method) from RewardNetworkImplTests#testRewardForDining() under rewards.internal test package.
     * Tested the same code, but using a different setup.
     * Ran the test to see if it passes.
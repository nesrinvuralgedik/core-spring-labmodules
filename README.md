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
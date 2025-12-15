# spring-transactions-note
Spring Transactions and @Transactional Annotation
========================================
https://www.youtube.com/watch?v=lJjeSQDEwYY

Critical Operation or Code section
---------------------------------------------------
Code Segment where shared resources are being accessed and modified

{
	Read Flight Seat with ID 1 {
		if SeatStatus is Available
			update to booked
	}
}

When multiple requests try to access this critical section, data inconsietency can happen .

What is the Solution??

Transactional
-------------------------
Helps to acheive ACID properties

A(Atomicity)
	Ensure all the operationss in  transcation are completly successfully, if any operation fails then entire transaction is rolled back.
C(Consitency)
	Ensure the DB start before and after transaction is consistency.
I(Isolation)
	Ensure that, if multiple transactions are running parallel, they donot interfere with each other.
D(Durability)
	Ensure that commited transactions will never be lost despite of system failures.
	
BEGIN_TRANSACTION
	- Debit from A
	- Credit to B
	If All Success
		COMMIT;
	Else:
		ROLLBACK;
END_TRANSACTION

spring-boot-starter-data-jpa
In SpringBootApplication

@SBA
// @EnableTrasactionManagement   --> Springboot will do automatically when we use @Transactional annotaion
p class SpringBootApplication {
}

@Transaction can be class level (apply for all public method only) or method level 
if Private method annotate with @Transactional then it will not apply

Transaction Management in Spring uses AOP
1. Uses pointcut expression to search the method having @Transactional annotaion
	@within("org.sf.transaction.annotaion.Transactional")
2. once the pointcut expression mathches, run an Around type Advice

TransactionInterceptor.java
TransactionAsceptSupport.java
    - invokeWithInTransaction
		- CreatTransactionIfNeccessary
		- completTransactionAfterThrowing
	    - CommitTransactionAfterReturning
		
Transactional Context
-------------------------------
Transaction Managers
--------------------------------
1.Programatically
2. Declarative

Transaction Propagations
- REQUIRED
- SUPPORTS
- MANDATORY
-REQUIRES_NEW
-NOT_SUPPORTED
-NEVER
-NESTED

Isolation Levels
-DEFAULT
-READ_UNCOMMITTED
-READ_COMMITED
-REAPEATABLE_READ
-SEARIALIZABLE

Transactios Timeout
-Read Only

Spring Transactions Part 2: Understanding Transaction Managers in Spring
-----------------------------------------------------------------------------------------------------------------
https://www.youtube.com/watch?v=wDMMD2pYNmg


Transaction-1.png
Transaction-2.png
Transaction-3.png

DataSourceTransactionManager
JPA Transaction Manager --> used this one 
JTA Transaction Manager (Used for two phase commit transactions i.e distributed transaction)

We can tell which Transaction Manger need to use but default JPA Transaction MAnager

@Bean
public PlatformTranasctionManager transactionManager(DataSource dataSource) {
	return new DataSourceTransactionManager(datasource);
}

Declarative
@Transactional(transactionManager= "transactionManager")

Programatically
-----------------
-Handle by Us
Transaction-4.png
Approach-1
-----------------------
Transaction-5.png

Approach-2
-----------------------
@Bean
public PlatformTranasctionManager transactionManager(DataSource dataSource) {
	return new DataSourceTransactionManager(datasource);
}

@Bean
public TransactionTemplate transactionTemplate(PlatformTranasctionManager transactionManager){
	return new TransactionTemplate(transactionManager);
}
TransactionTemplate
Transaction-6.png

Spring Transaction Propagation Explained | Spring Boot Transactions Part 3
------------------------------------------------------------------------------------------------------------
https://www.youtube.com/watch?v=LCltftLZ_W0

## What is The readOnly = true
The readOnly = true attribute is a configuration option used within Spring's @Transactional annotation to optimize database operations that only read data and do not modify it.
It acts as a performance hint to both the Spring framework and the underlying JPA provider (like Hibernate).

#Key Functions of readOnly = true
**Performance Optimization:** When set to true, the persistence provider knows it doesn't need to track changes to the entities you load. It skips internal "dirty checking," which significantly reduces overhead and speeds up the transaction execution.
**Prevents Data Modification:** It ensures that the transaction session is used purely for reading. If your code attempts an INSERT, UPDATE, or DELETE operation within a readOnly = true block, the provider will usually throw an error or simply ignore the write operation.
**Database Connection Hints:** In some database systems, this flag allows Spring to use database connection configurations optimized for reading, potentially using different isolation levels or fewer locks.

**When to Use It**
You should apply @Transactional(readOnly = true) to any service method that only fetches data and does not modify the database state.

* Data 1
  
import org.springframework.transaction.annotation.Transactional;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    // This method only retrieves a product; it does not change anything in the DB.
    @Transactional(readOnly = true) 
    public Product getProductDetails(Long productId) {
        return productRepository.findById(productId).orElse(null);
    }

    // This method needs to write data, so we use the default (readOnly = false).
    @Transactional // same as @Transactional(readOnly = false)
    public void updateProductPrice(Long productId, double newPrice) {
        Product product = productRepository.findById(productId).get();
        product.setPrice(newPrice);
        // Hibernate tracks this change and executes an UPDATE query
        productRepository.save(product); 
    }
}


Spring-WebFlux
-----------------------

Learn to write Reactive programming and build Reactive MicroServices using Spring WebFlux and project Reactor
Instructor
Pragmatic Code School
Technology Enthusiast, Online Instruct  (Dillip)
https://happiestminds.udemy.com/user/dilipsundarraj2/


The AWS SDK for Java 1.x is not fully deprecated but is in maintenance mode and will reach end-of-support on December 31, 2025. This means it will only receive critical bug and security fixes and will no longer get new features or region updates. The recommendation is to migrate to the AWS SDK for Java 2.x. 


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

A(Atomicity) <br>
	Ensure all the operationss in  transcation are completly successfully, if any operation fails then entire transaction is rolled back. <br>
C(Consitency) <br>
	Ensure the DB start before and after transaction is consistency.<br>
I(Isolation)<br>
	Ensure that, if multiple transactions are running parallel, they donot interfere with each other.<br>
D(Durability)<br>
	Ensure that commited transactions will never be lost despite of system failures.<br>
	
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

**The key options (attributes) for the @Transactional annotation are:**
Core Options
- **propagation**: Defines how a method should behave in relation to an existing transaction. <br>
	--** REQUIRED (Default):** Use the current transaction; create a new one if none exists.<br>
	-- **REQUIRES_NEW**: Always start a new, independent transaction; suspend the current one if it exists.<br>
	-- **SUPPORTS:** Use the current transaction if one exists; otherwise, run non-transactionally.<br>
	-- **NOT_SUPPORTED**: Run non-transactionally; suspend the current transaction if one exists.<br>
	-- **MANDATORY:** Requires an existing transaction; throws an exception if none is present.<br>
	-- **NEVER**: Must not run within a transaction; throws an exception if a transaction is active.<br>
	-- **NESTED:** Execute within a nested transaction (if the database supports savepoints); otherwise, behave like REQUIRED. <br>
- **isolation:** This defines the level of isolation from other concurrent transactions to prevent issues like dirty reads, non-repeatable reads, and phantom reads. <br>
  	-- **DEFAULT (default):** Uses the underlying database's default isolation level (often READ_COMMITTED). <br>
	-- **READ_UNCOMMITTED:** The lowest level; one transaction can read uncommitted changes from another (dirty reads are possible). <br>
	-- **READ_COMMITTED:** Prevents dirty reads (only committed data can be read) but allows non-repeatable reads. <br>
	-- **REPEATABLE_READ:** Prevents dirty and non-repeatable reads (re-reading data yields the same result) but might still allow phantom reads. <br>
	-- **SERIALIZABLE:** The highest, strictest level; fully isolates transactions, preventing all concurrency issues, but can impact performance. <br>
- **readOnly:** A performance optimization flag (default false). Setting it to true hints to the persistence provider (like Hibernate/JPA) and the database that no data modifications will occur, allowing for potential performance gains by bypassing dirty checking or using read-only database replicas. <br>
-- **timeout:** Specifies the maximum duration (in seconds) the transaction can run before the underlying transaction infrastructure automatically rolls it back (default is none or the system default, typically -1). <br>
-- **rollbackFor:** An array of exception types that, when thrown, will explicitly mark the transaction for rollback. By default, only unchecked exceptions (RuntimeException and Error) trigger a rollback. <br>
-- **noRollbackFor:** An array of exception types that, when thrown, will explicitly not cause the transaction to roll back, even if they are unchecked exceptions.
value or transactionManager: A string qualifier used to specify which PlatformTransactionManager bean to use if multiple are configured in the application context. <br>

Spring-WebFlux
-----------------------

Learn to write Reactive programming and build Reactive MicroServices using Spring WebFlux and project Reactor
Instructor
Pragmatic Code School
Technology Enthusiast, Online Instruct  (Dillip)
https://happiestminds.udemy.com/user/dilipsundarraj2/


The AWS SDK for Java 1.x is not fully deprecated but is in maintenance mode and will reach end-of-support on December 31, 2025. This means it will only receive critical bug and security fixes and will no longer get new features or region updates. The recommendation is to migrate to the AWS SDK for Java 2.x. 


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
	
BEGIN_TRANSACTION <br>
	- Debit from A <br>
	- Credit to B <br>
	If All Success <br>
		COMMIT;<br>
	Else:
		ROLLBACK;<br>
END_TRANSACTION<br>

spring-boot-starter-data-jpa<br>
In SpringBootApplication<br>

@SBA<br>
// @EnableTrasactionManagement   --> Springboot will do automatically when we use @Transactional annotaion<br>
p class SpringBootApplication {<br>
}<br>

@Transaction can be class level (apply for all public method only) or method level <br>
if Private method annotate with @Transactional then it will not apply<br>

Transaction Management in Spring uses AOP<br>
1. Uses pointcut expression to search the method having @Transactional annotaion<br>
	@within("org.sf.transaction.annotaion.Transactional")<br>
2. once the pointcut expression mathches, run an Around type Advice<br>

TransactionInterceptor.java<br>
TransactionAsceptSupport.java<br>
    - invokeWithInTransaction<br>
		- CreatTransactionIfNeccessary<br>
		- completTransactionAfterThrowing<br>
	    - CommitTransactionAfterReturning<br>
		
Transactional Context<br>
-------------------------------
Transaction Managers<br>
--------------------------------
1.Programatically<br>
2. Declarative<br>

Transaction Propagations<br>
- REQUIRED<br>
- SUPPORTS<br>
- MANDATORY<br>
-REQUIRES_NEW<br>
-NOT_SUPPORTED<br>
-NEVER<br>
-NESTED<br>

Isolation Levels<br>
-DEFAULT<br>
-READ_UNCOMMITTED<br>
-READ_COMMITED<br>
-REAPEATABLE_READ<br>
-SEARIALIZABLE<br>

Transactios Timeout<br>
-Read Only<br>

Spring Transactions Part 2: Understanding Transaction Managers in Spring<br>
-----------------------------------------------------------------------------------------------------------------
https://www.youtube.com/watch?v=wDMMD2pYNmg<br>


Transaction-1.png<br>
Transaction-2.png<br>
Transaction-3.png<br>

DataSourceTransactionManager<br>
JPA Transaction Manager --> used this one <br>
JTA Transaction Manager (Used for two phase commit transactions i.e distributed transaction)<br>

We can tell which Transaction Manger need to use but default JPA Transaction MAnager<br>

@Bean<br>
public PlatformTranasctionManager transactionManager(DataSource dataSource) {<br>
	return new DataSourceTransactionManager(datasource);<br>
}<br>

Declarative<br>
@Transactional(transactionManager= "transactionManager")<br>

Programatically
-----------------
-Handle by Us<br>
Transaction-4.png<br>
Approach-1<br>
-----------------------
Transaction-5.png<br>

Approach-2
-----------------------
@Bean
public PlatformTranasctionManager transactionManager(DataSource dataSource) {<br>
	return new DataSourceTransactionManager(datasource);<br>
}

@Bean
public TransactionTemplate transactionTemplate(PlatformTranasctionManager transactionManager){<br>
	return new TransactionTemplate(transactionManager);<br>
}<br>
TransactionTemplate<br>
Transaction-6.png<br>

Spring Transaction Propagation Explained | Spring Boot Transactions Part 3<br>
------------------------------------------------------------------------------------------------------------
https://www.youtube.com/watch?v=LCltftLZ_W0<br>

## What is The readOnly = true<br>
The readOnly = true attribute is a configuration option used within Spring's @Transactional annotation to optimize database operations that only read data and do not modify it.<br>
It acts as a performance hint to both the Spring framework and the underlying JPA provider (like Hibernate).<br>

#Key Functions of readOnly = true<br>
**Performance Optimization:** When set to true, the persistence provider knows it doesn't need to track changes to the entities you load. It skips internal "dirty checking," which significantly reduces overhead and speeds up the transaction execution.<br>
**Prevents Data Modification:** It ensures that the transaction session is used purely for reading. If your code attempts an INSERT, UPDATE, or DELETE operation within a readOnly = true block, the provider will usually throw an error or simply ignore the write operation.<br>
**Database Connection Hints:** In some database systems, this flag allows Spring to use database connection configurations optimized for reading, potentially using different isolation levels or fewer locks.<br>

**When to Use It**
You should apply @Transactional(readOnly = true) to any service method that only fetches data and does not modify the database state.<br><br>

* Data 1<br>
  
import org.springframework.transaction.annotation.Transactional;<br>
import org.springframework.stereotype.Service;<br>

@Service<br>
public class ProductService {<br><br>

    // This method only retrieves a product; it does not change anything in the DB.<br>
    @Transactional(readOnly = true) <br>
    public Product getProductDetails(Long productId) {<br>
        return productRepository.findById(productId).orElse(null);<br>
    }<br>

    // This method needs to write data, so we use the default (readOnly = false).<br>
    @Transactional // same as @Transactional(readOnly = false)<br>
    public void updateProductPrice(Long productId, double newPrice) {<br>
        Product product = productRepository.findById(productId).get();<br>
        product.setPrice(newPrice);<br>
        // Hibernate tracks this change and executes an UPDATE query<br>
        productRepository.save(product); <br>
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


-- By default, Spring only rolls back for Unchecked Exceptions (those extending RuntimeException or Error). It does not roll back for Checked Exceptions (those extending Exception but not RuntimeException, such as IOException or SQLException). <br>
-- You use this attribute when you want the transaction to roll back even if a Checked Exception occurs. <br>
// Example: Rollback for a checked exception (IOException) <br>
@Transactional(rollbackFor = {IOException.class, MyCustomCheckedException.class}) <br>
public void updateData() throws IOException { <br>
    repository.updateSomeFields(); <br>
    if (someCondition) { <br>
        throw new IOException("Critical failure, rolling back."); <br>
    }
} <br>

Rollback for All Exceptions: A very common pattern in modern Spring applications is to ensure that any exception triggers a rollback. <br>
@Transactional(rollbackFor = Exception.class) <br>


Spring-WebFlux
-----------------------

Learn to write Reactive programming and build Reactive MicroServices using Spring WebFlux and project Reactor<br>
Instructor<br>
Pragmatic Code School<br>
Technology Enthusiast, Online Instruct  (Dillip)<br>
https://happiestminds.udemy.com/user/dilipsundarraj2/<br>


The AWS SDK for Java 1.x is not fully deprecated but is in maintenance mode and will reach end-of-support on December 31, 2025. This means it will only receive critical bug and security fixes and will no longer get new features or region updates. The recommendation is to migrate to the AWS SDK for Java 2.x. <br>


Spring AOP (Aspect-Oriented Programming) is a programming technique used to separate cross-cutting concerns from business logic.

It helps us avoid repeating common functionality across multiple classes.

For example:

- Logging
- Security and authorization
- Transaction management
- Performance monitoring
- Exception handling
- Auditing

### **Benefits of AOP:**

Code for Aspect is defined in a single class
• Much better than being scattered everywhere
• Promotes code reuse and easier to change

Business code in your application is cleaner
• Only applies to business functionality: addAccount
• Reduces code complexity

Configurable
• Based on configuration, apply Aspects selectively to different parts of app
• No need to make changes to main application code 

### Important terminology in Spring AOP

| Term | Meaning |
| --- | --- |
| Aspect | Class containing cross-cutting logic |
| Advice | Action performed by an aspect |
| Join Point | A point during program execution where advice can run |
| Pointcut | Expression that selects join points (A predicate expression for where advice should be applied) |
| Target Object | Actual object whose method is intercepted |
| AOP Proxy | Object that intercepts method calls |
| Weaving | Process of applying aspects to target objects |

### Example to understand

Suppose you want to log every method execution inside your service package.

- Aspect: LoggingAspect
- Advice: Log before method execution
- Pointcut: All methods inside service package
- Join Point: Execution of a particular service method
- Target: OrderService
- Proxy: Spring-generated proxy around OrderService

### Types of Advice

Spring AOP provides five main types of advice.

| Advice | When it executes |
| --- | --- |
| `@Before` | Before method execution |
| `@After` | After method execution, whether successful or exceptional |
| `@AfterReturning` | After successful method execution |
| `@AfterThrowing` | When method throws an exception |
| `@Around` | Before and after method execution; controls whether the method proceeds |

#### @Before advice

Adding before advice to addAccount() method

```java
@Aspect
@Component
public class MyDemoLoggingAspect {

@Before("execution(public void addAccount())")
public void beforeAddAccountAdvice() {
System.out.println("Executing @Before advice on addAccount()");

}
}
```

### @AfterReturning Advice

This advice will run after the method call (success execution). It can be used to access the return value of method called.

```java
@AfterReturning(
pointcut="execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
returning="**result**")
public void afterReturningFindAccountsAdvice(
JoinPoint theJoinPoint, List<Account> **result**) {
// print out the results of the method call
System.out.println("\n=====>>> result is: " + result);
}
```

### @AfterThrowing Advice

 This advice will run after an exception is thrown

```java
@AfterThrowing(
pointcut="execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
throwing="theExc")
public void afterThrowingFindAccountsAdvice(
JoinPoint theJoinPoint, Throwable theExc) {
// log the exception
System.out.println("\n=====>>> The exception is: " + theExc);
}
```

However, the exception is still propagated to calling program.

If you want to stop the exception propagation, then use the @Around advice

### @After Advice

Runs after a method is completed. Regardless of outcome / exceptions. 

The @After advice does not have access to the exception.  If you need exception, then use @AfterThrowing advice.

```java
@After("execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))")
public void afterFinallyFindAccountsAdvice() {
System.out.println("Executing @After (finally) advice");
}
```

### @Around Advice

`@Around` gives you complete control over method execution.

When using **@Around** advice, You will get a reference to a “proceeding join point” which is a handle to the target method. Your code can use the proceeding join point to execute target method

```java
@Around("execution(* com.example.service.*.*(..))")
public Object logExecutionTime(ProceedingJoinPoint joinPoint)
        throws Throwable {

    long start = System.currentTimeMillis();

    Object result = joinPoint.proceed();

    long end = System.currentTimeMillis();

    System.out.println("Execution time: " + (end - start));

    return result;
}
```

### Reading Method Arguments with JoinPoints

!image.png

!image.png

Important methods:

| Method | Purpose |
| --- | --- |
| `proceed()` | Executes the actual target method |
| `getSignature()` | Gets method signature |
| `getArgs()` | Gets method arguments |
| `getTarget()` | Gets target object |
| `getThis()` | Gets proxy object |

### Weaving

• Connecting aspects to target objects to create an advised object
• Different types of weaving - Compile-time, load-time or run-time

### Pointcut expressions

Pointcuts determine which methods should be intercepted. A predicate expression for where advice should be applied.

!image.png

Common expressions:

| Expression | Meaning |
| --- | --- |
| `execution(* *(..))` | All methods |
| `execution(* com.example.service.*.*(..))` | All methods in service package |
| `execution(public * *(..))` | All public methods |
| `execution(* *Service.*(..))` | All methods in classes ending with Service |
| `within(com.example.service.*)` | Types in a package |
| `@annotation(org.springframework.transaction.annotation.Transactional)` | Methods annotated with Transactional |
| `bean(orderService)` | Spring bean named orderService |

!image.png

### Reusable pointcut

Instead of repeating expressions: When we have to use multiple advices to a single package / method we can create a pointcut and reuse it.

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}

@Before("serviceLayer()")
public void logBefore() {
    System.out.println("Before service execution to log");
}

@Before("serviceLayer()")
public void performApiAnalytics()  {
    System.out.println("Before service execution to perform api analytics");
}
```

### Combining Pointcut Expressions:

Combine pointcut expressions using logic operators
• AND (&&)
• OR (||)
• NOT (!)

Example :  All methods in a package EXCEPT getter/setter methods

```java
//methods in package
@Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
private void forDaoPackage() {}

// create pointcut for getter methods
@Pointcut("execution(* com.luv2code.aopdemo.dao.*.get*(..))")
private void getter() {}

// create pointcut for setter methods
@Pointcut("execution(* com.luv2code.aopdemo.dao.*.set*(..))")
private void setter() {}

// combine pointcut: include package ... exclude getter/setter
@Pointcut("forDaoPackage() && !(getter() || setter())")
private void forDaoPackageNoGetterSetter() {

//then use in advice
@Before("forDaoPackageNoGetterSetter()")
public void beforeAddAccountAdvice() {
…
}
```

## How Spring AOP works internally

Spring AOP uses proxies to intercept method calls.

There are two main proxy mechanisms:

1. JDK Dynamic Proxy

- Uses Java's built-in proxy mechanism.
- Works through interfaces.
- The proxy implements the target's interfaces.

2. CGLIB Proxy

- Creates a subclass of the target class.
- Does not require an interface.
- Cannot advise final classes or final methods.

| Feature | JDK Dynamic Proxy | CGLIB |
| --- | --- | --- |
| Interface required | Yes | No |
| Mechanism | Interface implementation | Class subclassing |
| Final class | Can proxy through interface | Cannot subclass |
| Final method | Can intercept interface invocation through proxy | Cannot override |
| Private method | Cannot directly intercept | Cannot override |
| Performance | Generally efficient | Generally efficient |
| Java implementation | Built-in Proxy API | Bytecode generation |

!image.png

Typical execution flow:

```
Client
   |
   v
Spring AOP Proxy
   |
   v
Advice / Interceptor
   |
   v
Target Method
   |
   v
Return Result
```

**Spring AOP performs runtime proxy-based weaving.**

## Spring AOP vs AspectJ

| Spring AOP | AspectJ |
| --- | --- |
| Proxy-based | Bytecode weaving |
| Runtime proxy creation | Compile-time, post-compile or load-time weaving |
| Method execution join points | Supports a wider range of join points (method-level, constructor, field) |
| Simpler Spring integration | More powerful and flexible |
| Self-invocation bypasses advice | Can intercept self-invocation |

### Can Spring AOP intercept private methods?

No. Spring AOP cannot intercept private methods.

Reason:

Spring AOP uses proxies to intercept method calls.

- JDK Dynamic Proxy works through public interface methods.
- CGLIB works by creating a subclass and overriding methods.
- Private methods cannot be overridden by subclasses.
- Therefore, private methods cannot be intercepted by Spring AOP.

### **Can Spring AOP intercept constructors?**

No. Spring AOP cannot intercept constructors.

Why?

Spring AOP is proxy-based and primarily intercepts method executions.

A constructor is responsible for creating an object. It is executed before the object is fully constructed and before Spring can wrap the initialized bean in an AOP proxy.

### Difference between Spring AOP and AspectJ?

Both are used to implement Aspect-Oriented Programming, but their implementation mechanisms are different.

| Feature | Spring AOP | AspectJ |
| --- | --- | --- |
| Implementation | Proxy-based | Bytecode weaving |
| Weaving | Runtime proxy-based | Compile-time, post-compile or load-time |
| Method execution | Yes | Yes |
| Method calls | No | Yes |
| Constructor interception | No | Yes |
| Field access | No | Yes |
| Private methods | No | Yes |
| Self-invocation | Not intercepted | Can be intercepted |
| Spring integration | Native | Requires AspectJ integration/configuration for weaving |
| Complexity | Easier | More complex |

### Can we apply multiple aspects to the same method?

Yes. Multiple aspects can intercept the same method execution.

For example, suppose an application has three aspects:

1. LoggingAspect
2. SecurityAspect
3. PerformanceAspect

All three can apply to the same service method. We can define the order using  @Order() annotation. It also supports negative values. 

`@Order(1)`
`public class LoggingAspect {}`

`@Order(2)`
`public class SecurityAspect{}`

`@Order(3)`
`public class PerformanceAspect {}`
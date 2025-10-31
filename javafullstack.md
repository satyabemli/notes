### Organized Interview Questions: From Basic to Advanced

#### 1. Introduction and Personal/Project Overview (Basic: Icebreakers)
1. Introduce yourself.
2. Explain about your project in both business and technical perspectives.
3. What kind of work you have done in your previous project/experience?
4. What tools and technologies have you worked on in your previous experience?
5. Do you have production support experience?
6. What kind of queries do you get on daily basis in production support?
7. How do you figure out data missing in production?
8. Do you have production database access?
9. When an error occurs in production environment, do you check log file or database tables?
10. Which application server do you use (e.g., JBoss)?
11. Do you know Unix?
12. How to search a keyword in a log file in Unix?
13. What is ‘grep’ in Unix?
14. How to check the processes which are running in Unix?

#### 2. Core Java Basics (Basic: Fundamentals and OOP)
15. What is the super class of all objects in Java?
16. Methods defined in Object class.
17. What is static? What is the use of static (e.g., static class, method, variable)?
18. What is the meaning of final (e.g., final class, method, variable)? Final keyword usage.
19. What is polymorphism? What is static polymorphism? What is dynamic polymorphism?
20. Can multiple inheritance be supported in Java? If so, how? Is multiple class inheritance supported in Java?
21. What is a marker interface? Purpose of it.
22. What is a thread?
23. What is Heap memory vs Stack memory? Local variables stored in which area?
24. What is the difference between == and .equals()?
25. What is generics?
26. Difference between overloading and overriding (w.r.t. exception thrown, return type, parameters).
27. What is covariant type?
28. Overloading rules for access and non-access modifiers?
29. What are different access and non-access modifiers in Java?
30. Abstract class vs final class difference?
31. Can we have a constructor in an abstract class? Can we instantiate the abstract class?
32. Singleton class vs Static Class? What is a singleton class? How to write our own singleton class? How do you create a singleton class using Spring framework? In general, how do you create a singleton class in Java?
33. What will happen if we use clone() on a singleton object?
34. What is cloning? Different ways to clone the object. What is de-cloning?
35. Have you heard of Object Cloning in Java?
36. Deep copy vs shallow copy.
37. Static vs singleton memory allocation.
38. Class loaders in Java.
39. What is RMI?
40. What Java version do you use (e.g., 1.8)? What are Java features used in 1.8? What is the use of Stream API? What are default methods in Java 8? What is a lambda expression? What is a functional interface?

#### 3. Exceptions and Error Handling (Basic-Intermediate)
41. Exception hierarchy (Throwable -> Exceptions & Error).
42. Differentiate runtime exception and checked exception in Java.
43. What is exception bundling? How do you handle exceptions in a chain (e.g., a->b->c->d, exception in d method)?
44. Do you know Garbage Collection? How does Garbage Collection work in Java? How to enforce Garbage Collection in Java? Do you know memory leaks in Java and when they will popup? How can you fix Garbage Collection and memory leak issues?

#### 4. Collections Framework (Intermediate: Data Structures)
45. Why do we have a collection framework in Java? What are different interfaces in it? Collection hierarchy and types of collection interfaces & differences.
46. What is the difference between Set and List? ArrayList vs Set? ArrayList vs LinkedList vs Vector?
47. What are the collections you have used/implemented frequently in your previous project? Operation of LinkedList?
48. Difference between enumeration vs iterator vs listiterator vs spliterator.
49. Difference between comparable and comparator.
50. What are the methods to sort collections for primitive types/classes (both natural and custom sort)? Collections.sort, comparable, and comparator.
51. How to sort a list on the basis of employeeID? How to sort a list on the basis of employeeID without using Comparator?
52. Employee class is present and its objects are added in a list. Employee class is present in a JAR file and its objects are added in a list.
53. What is HashMap? Have you used HashMap in Java? Can the key be integer, string, or a class? Can I use an object as a key in HashMap? Is it possible to add duplicate keys in HashMap? What will happen if we add duplicate keys? Will HashMap allow null keys?
54. Difference between HashMap vs TreeMap vs LinkedHashMap vs Hashtable? Difference between HashMap and Hashtable?
55. How HashMap works? Internal implementation of put/get method? Logic of storing and retrieving objects from HashMap. How does HashMap internally work? Can you give an example? How to iterate HashMap elements in a HashMap object?
56. Use of equals and hashcode methods. What is required to use a user-defined method/class as a key in MAP? HashMap internal implementation. HashCode and equals method implementation in code if we use custom class as a key in HashMap.
57. What all things to keep in mind if we want to implement Map on our own?
58. How TreeSet works? Have you used TreeSet? Adding employee details in TreeSet in the type of class Employee, how is sorting done in TreeSet? TreeSet implementation and on what basis does it sort if we have custom objects in it?

#### 5. Multithreading and Concurrency (Intermediate-Advanced)
59. How to create a thread? How many ways to create a thread?
60. Thread life cycle.
61. Have you used Synchronization in Java? Which keyword is used and can it be applied on a class?
62. What is synchronized used for?
63. What is deadlock?
64. Lock API vs Synchronization.
65. Executor framework in Java 1.5.
66. How can I implement multithreading in Java with 30 threads?
67. Observable, Observer, CyclicBarrier, CountdownLatch.
68. Have you worked on threads? We have some tasks to handle large amount of data—how are you going to handle with threading?

#### 6. I/O and Serialization (Intermediate)
69. What is serialization? What is the use of serialization? Serializable interface and why do we need it?
70. I have a class with 3 variables (var1, var2, var3), want to serialize that class but don’t want to serialize var3? How will you implement this? What is Serialization? What if we declare a variable as transient and still we want to serialize it?

#### 7. Servlets and Web Basics (Intermediate)
71. Servlet config vs Servlet context.
72. HTTP methods in Servlet? GET vs POST in Servlet?
73. What is a single-page architecture?

#### 8. Logging (Intermediate)
74. What is logging? Which log4j version did you implement? Which log level do you use frequently? What is the order of log levels?
75. Logging framework—how do we declare and implement different log levels in our code?

#### 9. Spring Framework Basics (Intermediate-Advanced)
76. What is the use of Spring? Why do we need Spring framework? Benefits of Spring? What are Spring features and benefits?
77. What is Dependency Injection? What is Dependency Injection/Inversion of Control in Spring?
78. How you develop a Spring application and what configurations need to be done?
79. List of scopes used in Spring? How we declare scope in Spring? What are bean scopes available? Define singleton scope.
80. What is Session Scope? What is Global-Session Scope? What is HttpSession? One client sends open session today and after 3 days if the same client comes back and tries to access, will he get the first day's session or not? In a Singleton bean, I injected a Session-scoped bean? What will happen?
81. Different scopes in Spring framework? Spring MVC flow and how do we use application context in Spring?
82. How container will know about spring-config? Where will you mention spring-config? How Spring knows about classes like controllers, services, etc.?
83. How will you inject dependent beans? (By using XML and by using annotations).
84. What is auto-wire and its types? How will you use auto-wiring? What is @Qualifier?
85. Explain annotations: @Controller, @Service, @Repository, @Autowired? @Bean, @Configuration, @Auto-wire, @Qualifier, @Repository, @Service, @Component.
86. Autowiring types. Dependency check (simple, object, complex, none).
87. Can we have more than 1 application context? FileSystemXmlApplicationContext. ApplicationContext context = new ClassPathXmlApplicationContext(new String[] {}); ApplicationContext context = new ClassPathXmlApplicationContext(Spring-All-Module.xml);
88. Life cycle of a Spring bean.
89. How you achieve security using Spring in your application?

#### 10. Spring MVC (Advanced)
90. Tell me about Spring MVC? Spring MVC Flow. Request life cycle of Spring MVC.
91. Difference: SimpleURLHandlerMapping, ControllerClassNameHandlerMapping, BeanNameUrl Handler mapping.
92. Different controller types and purpose of them. Life cycle methods of SimpleFormController, AbstractController.
93. View Resolver purpose and what are the different view resolvers you have used.
94. Validations in Spring MVC—how to handle on server side.
95. Exceptions in Spring MVC—ExceptionHandler and ExceptionResolver.
96. How to map the controller URL in Spring?
97. How you will map the JSON data coming from UI in Spring?

#### 11. Spring Boot (Advanced)
98. What is Spring Boot? Spring vs Spring Boot?
99. How can you define custom/own servlets in Spring Boot?
100. What is filter register bean?
101. What is SpringJdbcTemplate?
102. How can you create a datasource in a Spring Boot application?
103. How to load properties in a Spring Boot application?

#### 12. Web Services and REST (Advanced)
104. Do you have experience in RESTful web service framework?
105. Web service flow and configurations: How service is triggered from front end and at back end.
106. Sample REST web services with example?
107. Explain the flow of RESTful Web Services?
108. How do you create a Rest Client in Web Services?
109. How do we write a RESTful Web Service method?
110. Have you published any web service? How can we consume the web service? We have a WSDL file—how can we consume the external web service operations?
111. I have a REST controller, want to do some process before processing the request?
112. What is a Filter?
113. How do you test the REST service? How do you authenticate the microservice—any idea?
114. How does a microservice interact with other external services?
115. You have millions of data in database—how are you going to design the REST service with search 'ABC' key?
116. If the system goes with OOM error, how are you going to handle this scenario?

#### 13. Hibernate (Advanced)
117. What is the use of Hibernate? Why is Hibernate required? Advantages of Hibernate.
118. How to connect database through Hibernate framework?
119. How to integrate Hibernate with Spring? (Dependencies, model classes, DAO, SessionFactory, @Transactional).
120. What kind of work you have done in Hibernate?
121. Session.Load vs Session.Get.
122. What are different states of an entity bean? (Transient, Persistent, Detached—explain each).
123. Different annotations purpose (e.g., @Entity, @Table, @Id, @Column, @GeneratedValue, @OneToOne, @Cascade, @PrimaryKeyJoinColumn).
124. We have department and employee (one-to-many) relationships—how are you going to design the relationships in Hibernate? How can we get only department details? Any idea on cascading and lazy loading in Hibernate?
125. How to handle a situation like when read and write operation requests at a time in Hibernate?
126. How to read native SQL queries in Hibernate?
127. What is the difference between first-level cache and second-level cache?

#### 14. Database and SQL (Advanced)
128. What is GROUP BY clause? WHERE clause vs HAVING clause.
129. Explain DDL/DML/DCL statements.
130. DML statements: SELECT, UPDATE, DELETE, INSERT.
131. Difference between Truncate and Delete.
132. Procedure vs Function.
133. Table vs View.
134. What are different joins?
135. Indexes in database. Clustered vs non-clustered index.
136. Have you worked on stored procedures? How can we call stored procedure from Java? Benefits of stored procedures?
137. Do you have any experience in SQL? Are you good in basic joins in SQL?
138. If you have a large amount of data in a CSV file, how are you going to handle that data?

#### 15. Design Patterns and Principles (Advanced)
139. Design patterns: Creation (Prototype, Singleton, Factory, Abstract Factory). Structural (Facade, Adaptor, Composite). Behavioral (Chain of Responsibility, Observer, Template Method).
140. Core J2EE patterns: 2 presentation tier, 2 business tier, 2 integration tier (why used?).
141. SOLID Principles (optional for code reviews).
142. How are you going to design the cache for repeated objects?

#### 16. Angular/Frontend (Advanced: If Relevant)
143. What is the difference between Angular & Angular 7/8?
144. What is a module? How do we create a module?
145. What is a component? How do we create a component?
146. What is a directive? What are the different directive types? How do we create each directive type?
147. What is a pipe? What are default pipes available in Angular 8? How do we create a custom pipe?
148. What is routing? How do we implement routing in Angular 8—explain the steps.
149. What is an Angular Service? How to create an Angular service? How to use an Angular service?
150. How to handle form submission in Angular? (Reactive vs Template-driven forms vs Material Design forms).
151. How to pass data from parent component to child component, child to parent?
152. Have you worked on UI framework? What's the security mechanism you have used in your project?

#### 17. Program Logic and Scenarios (Advanced: Coding/Problem-Solving)
153. What's the importance of hashcode and equals methods in Java?
154. How will you sort the customized Employee class? How will you implement the Employee class?
155. We have employee (id, name, address) with 1000 employee objects—how to convert employee list to map and how are you going to form the key-value pair? How to iterate the data from the list?
156. Program logic: What is wrong/not wrong about below program? Abstract Class A {} Class B extends A { public static void main() { B obj = new B(); } } Can we compile or run this program? (Class A does not have a default constructor—error on instantiation).
157. If System goes with OOM error, how are you going to handle this scenario?

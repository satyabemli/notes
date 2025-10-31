# Quick Reference: Java Interview Questions

## Core Java: Basics

1. What is polymorphism? Real time use of Abstraction and interface.
2. Two objects of same class with fields having same values in both objects, what is the outcome if we use ‘==’? How to check equality?
3. Implement equals and hashcode in your class, why we need to do it, if only either of the one implemented what happens.
4. How Queue is implemented in java? Implement Queue data structure.
5. Difference between Stack and Queue.
6. How Tree set works internally?
7. Difference between Hashset, TreeSet and Linked Hashset.
8. String s1=”abcd”; String s2=”cdab”. Write a program to find how many times s2 is rotated.
9. What is transient, volatile?
10. How to do serialization, what is it?
11. Difference between Comparable, comparator. Which one is to be used and in which scenario?
12. Can we make class as Static?
13. SOLID principles with real time example
14. Why strings are immutable in java?
15. Why Composition is preferred over inheritance?
16. What is string pool?
17. What will happen when parent class is serialized and child class is not serialized?
18. What will happen when child class is serialized and parent class is not serialized?
19. In how many ways can an object be created?
20. What is a deadlock and how can you avoid it?
21. How can you analyze heap dump?
22. What is a functional interface in java 8?
23. How is the hashmap rehashed to add new elements?
24. What is the complexity of a HashMap?
25. Explain OOPS concepts in Java.
26. Why is Java NOT 100% object oriented?
27. What are the methods of an object class?
28. How is a class loaded into JVM and how many types of class loaders are present in Java.
29. What will happen if the low level class loader loads the class again?
30. Class constructor is defined as private. Can it be instantiated?
31. Why do we use final keyword in Java? When do we declare a class as final?
32. What is the requirement of this keyword?
33. What is wrapper class in Java?
34. How do you find a maximum number in a list in Java1.8 in a single statement?
35. How to convert String to integer(int) without using parseInt()
36. What is best way to concatenate two strings in java?
37. Difference b/w String, StringBuilder and StringBuffer?
38. In Java 8, how will loop through Collections?
39. Difference b/w map and flatMap in java 8.
40. What are optional classes?
41. How will you use filter() in java 8?
42. How will you throw exceptions when using streams on collection?
43. What will happen if we put a return 1 in try block?
44. What will happen if we return 1 in hashcode method?
45. What is the difference between  
   a. ArrayList and Vector  
   b. HashMap and ConcurrentHashMap.  
   c. SynchronizedMap and ConcurrentHashMap  
   d. HashTable and SynchronizedMap.  
   e. Synchronized collections & Concurrent Collections in java  
   f. HashMap and Hashtable.  
   g. TreeSet and HashSet.
46. What will happen if you add a duplicate key in TreeMap?
47. What is clonable interface? What is deep cloning?
48. What is a marker interface?
49. What is an exception? How can we create a custom exception?
50. What will happen if checked exception is not handled (or) checked exception is not at all there?
51. Difference between == and .equals()?
52. What's the use of shutdownhook?
53. What's the difference between DOM and SAX parser?
54. Can we allocate memory to threads?
55. Can we execute the java application of size 3 MB with 2GB heap space?
56. What's the difference between JRE, JVM and JDK?
57. How to avoid the sql injection attack?
58. Can we use runnable with executors? In that case, what's the difference between callable and runnable with executors?
59. How do you create your own thread pool?
60. Will the class loader loads the .xml files?

## Core Java: Programming

1. Write a program identify loop in LinkedList and remove the same.
2. Write a program to remove duplicate character from the String.
3. Write a program to get the required output.  
   Input: String - abbccbbbdd  
   Output: a1b2c2b3d2
4. Write a code for creating deadlock.
5. Write a code for creating memory leak.
6. Write code for all the ways you can traverse a collection before Java8? For loop, for in loop, for Each loop syntax
7. Write a Program to print a star pyramid
8. Write a program to find two strings are anagrams.
9. There are 8 marbles and one weighing machine. In how many iterations can you find the heaviest marble.
10. There is list of values. How will you find the two numbers give the sum of a given number.
11. Write a thread program of two threads and print odd and even numbers alternatively.
12. Write a program to print the binary tree
13. Write a program to hold and release the lock
14. Write a program for insertion/Bubble/Merge/Quick sort.

## Data Structures

1. What is the difference between O(1), O(N), O(n^2), data structures with examples?
2. Which data structure used for collections(LinkedList, HashMap) behind and explain?
3. What is o(log n) data structure?
4. Difference between array and ArrayList and LinkedList?
5. Write a customized arraylist.
6. Sort a third party object in an arraylist.
7. Why do we need to use linked list if we can use array list?
8. Design a customized map.
9. Design a customized Linked list.
10. Linked list traversing logic.
11. How to clone a linked list with data?
12. How do you find out similar elements from two linked list.
13. How to find a merge node in a linked list?
14. Write a java program for circular LinkedList implementation?
15. What are the different data structures. Enumeration; BitSet; Vector; Stack; Dictionary; Hashtable; Properties
16. How is Binary Search implemented?
17. How do you find a loop in linked list and remove the same.
18. How to reverse a linked list?
19. Cloning of doubly linkedlist, where the next_pointer holds a random value.
20. Write binary tree and explain binary search tree and its complexity.
21. Write a program to implement a stack.

## Real-Time Questions

1. Explain about your project architecture.
2. How to parse a huge file of 10GB.
3. Last project details, Architecture, Complexity & challenges faced
4. Producer Consumer problems.
5. Design application Mobile SMS backup and recovery.
6. Threads producer and consumer problem. Sleep, yield, join, notify, notify all
7. What all Security measures used for handling payment through payment gateways.
8. A user is getting 504 errors whenever he is searching something on a website? His net speed is fine and server net speed is also good. How will you debug this?
9. 1000 users are accessing a web site and out of 1000, 20 users are getting 500 error How will you debug it and what is the reason for this?
10. Now only 1 user out of 1000 getting 500 or 504 error. How will you debug it?
11. When someone report you that application is slow what do you check and why?
12. How to confirm data coming from UI is perfect and there is only problem in Java code?
13. How do you track defect in a big application.
14. What happens when you submit a form; complete process including DB and ORM in REST till we get the response.
15. I dont have UI, just I have end point through that we are getting data from Client (Like Student class have basic Property). Write Restfull Webservices code to save and also update.
16. If banking application starts form scartch , which one (SOAP/REST) will be suggested and why?

## Spring Interview Questions

1. What is Spring DI pattern?
2. What are the differences between constructor and setter injection? Which scenario we prefer setter injection?
3. Life cycle of Spring
4. What is the use of @Postconstruct and @Predestroy used for?
5. What is @component, @controller, @repository and @service annotations used for?
6. If we are going to inject the prototype bean in singleton bean, how will it behave?
7. What is the default scope of spring bean?
8. Why can't we use @component annotation instead of @controller?
9. How is transaction is managed in Spring?
10. @Transaction can be applied to class level as well?
11. What are the features that you have used in Spring?
12. What is Autowire in Spring?
13. Challenges faced while using spring annotations?
14. Write a program to display employee details in UI using Spring framework?
15. What is the use of dispatcher-servlet in spring?

## Spring Boot Interview Questions

1. Why we go for spring boot instead of spring?
2. What is the equivalent for @EnableWebConfiguration?
3. What are the steps in spring boot while loading application.properties/application.yaml file?
4. If we want to deploy the springboot application in weblogic server, what are the steps we need to follow?
5. Explain Actuator in Spring boot application.
6. How to configure a spring boot application?
7. How will you make sure that your java class in spring boot application doesn’t have any performance issue?
8. What's spring boot actuator?

## Microservices Interview Questions

1. How will we verify the authentication when each service is invoked?
2. How will the microservices communicate with each other?

## AWS Interview Questions

1. What is the difference between SQS and SNS?
2. What is role?

## Database Interview Questions

1. What is deadlock and how do you avoid it?

## Hibernate Interview Questions

1. Write parent and child Class. Define the basic property and think which mapping in Hibernate is suitable
2. We have one Employee table in database with fields EmpId, EmpName, age. Fetch the records from database through JDBC/hibernate and sort records on the basis of age. After using above scenario- need to display records in UI page. Using layers from Persistence->DAO-> Service->Controller->presentation layer.
3. What are different levels of Cache in Hibernate? What is the difference between first level and second level cache in Hibernate?
4. What are the uses of Hibernate? Why we use hibernate instead JDBC API?
5. Hibernate Object states, different Transactions, different level of caching and Mapping.
6. Advantages of hibernate.
7. Difference between save() and persist().
8. Write parent and child Class define its basic property and think which mapping in Hibernate is suitable.
9. Difference between load and get
10. The query is executed to get the records and execute the same query again. Meanwhile some other query write the results to DB. From where will the results be fetched? Cache/DB. Why?

# 🚀 Ultimate Spring Boot Interview Questions (470+ Questions) 2025 Edition  

## 1️⃣ Core Spring & Spring Boot (Beginner to Advanced)  
1. What is Spring Framework?  
2. What is Spring Boot and how is it different from Spring?  
3. What are the advantages of using Spring Boot?  
4. What is dependency injection in Spring?  
5. What is the difference between @Component, @Service, @Repository, and @Controller?  
6. What is ApplicationContext?  
7. What is the use of @SpringBootApplication?  
8. What is @EnableAutoConfiguration?  
9. How does Spring Boot decide what to auto-configure?  
10. What is the role of application.properties or application.yml?  
11. What is a Bean in Spring?  
12. How do you define a bean?  
13. What is the lifecycle of a Spring Bean?  
14. What is @Qualifier used for?  
15. What is the difference between Singleton and Prototype scope in Spring?  
16. What is the use of @Value annotation?  
17. What is Spring AOP?  
18. What are the key modules in Spring?  
19. What is circular dependency and how does Spring handle it?  
20. How do you run a Spring Boot application?  
21. What is the difference between BeanFactory and ApplicationContext?  
22. What is @Primary annotation?  
23. What is @Lazy annotation?  
24. What is the difference between @Inject, @Autowired, and @Resource?  
25. What is Spring Expression Language (SpEL)?  
26. What is the difference between @Controller and @RestController?  
27. What are Spring Boot Starters?  
28. How does Spring Boot handle embedded servers?  
29. What is the difference between @PostConstruct and @PreDestroy?  
30. What is the Spring IoC container?  

## 2️⃣ Spring Boot Annotations & Configurations  
31. What is @Configuration?  
32. What is @Bean?  
33. What is @ComponentScan?  
34. What is @Component?  
35. What is the difference between @Component and @Bean?  
36. What is @RestController?  
37. What is @RequestMapping?  
38. What is the difference between @GetMapping and @RequestMapping(method=GET)?  
39. What is @Autowired?  
40. What is constructor-based vs field-based injection?  
41. What is @ConditionalOnProperty?  
42. What is @Profile?  
43. How do you set active profiles in Spring Boot?  
44. What is @PropertySource?  
45. What is @ConfigurationProperties?  
46. What is @EnableConfigurationProperties?  
47. What is @EnableScheduling?  
48. What is @Scheduled?  
49. What is @EnableAsync?  
50. What is @Async?  
51. What is @PathVariable?  
52. What is @RequestParam?  
53. What is @RequestBody?  
54. What is @ResponseBody?  
55. What is @RequestHeader?  
56. What is @CookieValue?  
57. What is @ModelAttribute?  
58. What is @ExceptionHandler?  
59. What is @ControllerAdvice?  
60. What is @ConditionalOnClass?  
61. What is @ConditionalOnMissingBean?  
62. What is @Order?  
63. What is @DependsOn?  
64. What is @Scope?  

## 3️⃣ Spring Data JPA & Transactions  
65. What is Spring Data JPA?  
66. What is JpaRepository?  
67. What is the difference between CrudRepository and JpaRepository?  
68. How do you define a custom query in Spring Data JPA?  
69. What is the use of @Entity?  
70. What is @Id?  
71. What is @GeneratedValue?  
72. What is the N+1 select problem in JPA?  
73. How to solve N+1 problem?  
74. What is the default fetch type for @OneToMany and @ManyToOne?  
75. What is @Transactional?  
76. Where should @Transactional be placed?  
77. What is the difference between @Transactional(readOnly = true) and without it?  
78. Can @Transactional be used on private methods?  
79. How do you handle transactions in nested service calls?  
80. What is propagation in Spring transactions?  
81. What is isolation in transactions?  
82. How do you handle optimistic locking in JPA?  
83. How do you implement pagination in Spring Data JPA?  
84. How do you audit entities in JPA?  
85. What is @OneToMany and @ManyToOne?  
86. What is @ManyToMany?  
87. What is @JoinColumn?  
88. What is @JoinTable?  
89. What is the difference between JPQL and native queries?  
90. How do you handle database migrations in Spring Boot?  
91. What is the difference between save() and saveAndFlush()?  
92. What is @Modifying annotation?  
93. What is @Query annotation?  
94. What is @NamedQuery and @NamedNativeQuery?  
95. What is the difference between EntityManager and EntityManagerFactory?  
96. What is the first-level cache in Hibernate?  
97. What is the second-level cache in Hibernate?  
98. How do you configure connection pooling in Spring Boot?  
99. What is @TransactionalEventListener?  
100. What is the difference between merge() and persist()?  

## 4️⃣ Spring Security (Basic to OAuth2/JWT)  
101. What is Spring Security?  
102. What are the key components of Spring Security?  
103. How do you configure HTTP Basic Authentication in Spring Boot?  
104. What is the difference between authentication and authorization?  
105. What is the default security behavior of Spring Boot?  
106. How do you create a custom login form in Spring Security?  
107. What is CSRF and how does Spring handle it?  
108. How do you disable CSRF protection in Spring Security?  
109. How do you secure REST APIs in Spring Boot?  
110. What is JWT?  
111. What are the parts of a JWT token?  
112. How do you implement JWT-based authentication in Spring Boot?  
113. What is the role of OncePerRequestFilter in JWT authentication?  
114. How do you store user details for authentication?  
115. What is PasswordEncoder in Spring Security?  
116. What is @EnableWebSecurity?  
117. How to implement method-level security in Spring?  
118. What is OAuth2 and how is it different from JWT?  
119. How do you use Spring Security OAuth2 for login?  
120. What is the difference between stateful and stateless security?  
121. What is the SecurityContext?  
122. What is @PreAuthorize?  
123. What is @PostAuthorize?  
124. What is @Secured?  
125. What is the difference between hasRole() and hasAuthority()?  
126. What is CORS and how do you configure it in Spring Security?  
127. What is XSS and how does Spring Security prevent it?  
128. What is the difference between authentication and principal?  
129. How do you implement remember-me functionality?  
130. What is session management in Spring Security?  
131. What is the difference between stateless and stateful session management?  
132. How do you implement logout in Spring Security?  
133. What is the role of SecurityFilterChain?  
134. What is @AuthenticationPrincipal?  
135. How do you implement custom authentication provider?  

## 5️⃣ Spring Boot Testing (Unit, Integration, Testcontainers)  
136. What is @SpringBootTest used for?  
137. What is the difference between @WebMvcTest and @SpringBootTest?  
138. How do you test a controller in isolation?  
139. What is MockMvc?  
140. What is @DataJpaTest?  
141. How do you mock beans in Spring tests?  
142. What is @TestConfiguration?  
143. How do you test service layer methods?  
144. What is the role of TestEntityManager in @DataJpaTest?  
145. How do you test REST endpoints end-to-end?  
146. What is @Transactional used for in tests?  
147. How do you test Kafka or external systems without Docker?  
148. What are Testcontainers?  
149. How do you mock external API calls in Spring Boot tests?  
150. How do you verify caching behavior in tests?  
151. How do you test retry logic or circuit breakers?  
152. What is @EnabledIf and how is it useful in tests?  
153. How to write parameterized tests in Spring Boot?  
154. How to assert exceptions in Spring tests?  
155. How to measure test coverage in Spring Boot projects?  
156. What is @MockitoBean vs @MockBean?  
157. What is @SpyBean?  
158. How do you test reactive code in Spring Boot?  
159. What is @JsonTest?  
160. What is @RestClientTest?  
161. How do you test scheduled tasks?  
162. What is @DynamicPropertySource?  
163. How do you test security configurations?  
164. What is the difference between @DirtiesContext and @Transactional in tests?  
165. How do you test async methods?  

## 6️⃣ Microservices with Spring Boot  
166. What are microservices?  
167. What are the benefits of microservices architecture?  
168. How do microservices communicate with each other?  
169. What is service discovery and why is it needed?  
170. What tools can you use for service discovery?  
171. What is Spring Cloud?  
172. What is Spring Cloud Config?  
173. How do you externalize configuration in Spring Boot microservices?  
174. How do you implement interservice REST communication in Spring Boot?  
175. What is Spring Cloud OpenFeign?  
176. What is circuit breaker pattern?  
177. How do you implement a circuit breaker in Spring Boot?  
178. What is retry pattern in microservices?  
179. How to apply retries in Spring Boot?  
180. What is rate limiting and how is it done?  
181. What is service registry and discovery using Eureka?  
182. How do you implement centralized logging in microservices?  
183. What is the role of Spring Cloud Gateway?  
184. What is the difference between Spring Cloud Gateway and Zuul?  
185. How do you version REST APIs in microservices?  
186. How do you trace requests across microservices?  
187. How do you secure microservices communication?  
188. What is API Gateway and its role in microservices?  
189. How do you handle partial failures in microservices?  
190. What are sagas in microservices?  
191. How do you implement sagas in Spring Boot?  
192. What is an orchestrator vs. choreographer in saga pattern?  
193. How do you handle data consistency in microservices?  
194. How do you implement async communication between microservices?  
195. What is idempotency and why is it important in microservices?  
196. What is the difference between synchronous and asynchronous communication?  
197. What is API composition pattern?  
198. What is the database per service pattern?  
199. What is the strangler fig pattern?  
200. How do you handle service-to-service authentication?  

## 7️⃣ Kafka & Messaging Patterns in Spring Boot  
206. What is Apache Kafka?  
207. What are topics and partitions in Kafka?  
208. How do producers and consumers work in Kafka?  
209. What is Kafka broker?  
210. What is a Kafka consumer group?  
211. How do you ensure message ordering in Kafka?  
212. How do you integrate Kafka with Spring Boot?  
213. What is @KafkaListener?  
214. How do you send messages to Kafka in Spring Boot?  
215. How do you ensure reliable delivery in Kafka consumers?  
216. What is offset in Kafka?  
217. How do you handle offset commits in Kafka?  
218. What is dead letter topic (DLT)?  
219. How do you configure DLT in Spring Kafka?  
220. What is idempotent producer in Kafka?  
221. How do you handle retries in Spring Kafka consumer?  
222. What is a Kafka schema registry?  
223. How do you serialize/deserialize messages in Kafka?  
224. How do you ensure message durability in Kafka?  
225. What is the difference between at-most-once, at-least-once, and exactly-once delivery?  
226. How do you scale Kafka consumers in Spring Boot?  
227. What is the use of Kafka headers?  
228. How do you simulate Kafka in tests without Docker?  
229. How do you log or trace Kafka messages?  
230. How do you implement request-reply over Kafka?  
231. What is Kafka Streams?  
232. What is the difference between Kafka Streams and Spring Cloud Stream?  

## 8️⃣ Spring Boot Advanced Concepts  
241. How do you make a method asynchronous in Spring Boot?  
242. What is the return type of an @Async method?  
243. How do you schedule tasks in Spring Boot?  
244. What are different @Scheduled options?  
245. What is the difference between fixedRate and fixedDelay?  
246. How do you enable caching in Spring Boot?  
247. What is @Cacheable?  
248. What is @CachePut?  
249. What is @CacheEvict?  
250. How do you configure cache manager in Spring Boot?  
251. What is a custom key generator in caching?  
252. How to disable cache during tests?  
253. How do you validate user input in Spring Boot?  
254. How to apply validation to nested objects?  
255. What is the difference between @Valid and @Validated?  
256. How do you handle validation errors globally?  
257. How to return custom error messages from validation failures?  
258. What is Spring Boot Actuator?  
259. How do you expose actuator endpoints securely?  
260. How to override application properties per environment?  
261. What is the order of property resolution in Spring Boot?  
262. How do you use profile-specific beans?  
263. How do you package a Spring Boot application?  
264. How do you deploy Spring Boot apps?  
265. How do you monitor Spring Boot in production?  
266. What is the use of Micrometer in Spring Boot?  
267. What is Spring Boot DevTools?  

## 9️⃣ Spring Cloud Ecosystem  
286. What is Spring Cloud Config?  
287. How do you set up a Spring Cloud Config Server?  
288. How do client services connect to Config Server?  
289. What's the difference between bootstrap.yml and application.yml?  
290. How do you refresh config properties at runtime?  
291. What happens if the config server is down?  
292. How do you secure Spring Cloud Config Server?  
293. Can Spring Cloud Config auto-refresh values without endpoint hit?  
294. What is Eureka Server?  
295. How do you create a Eureka server?  
296. How do you make a Spring Boot service discoverable?  
297. What is the difference between Eureka Client and Service Discovery?  
298. What is self-preservation mode in Eureka?  
299. How does Eureka handle service health checks?  
300. What is instance metadata in Eureka?  
301. What is Spring Cloud Gateway?  
302. How does routing work in Gateway?  
303. What are filters in Spring Cloud Gateway?  
304. What is a global filter vs route filter?  
305. How do you apply rate limiting in Gateway?  
306. How do you secure APIs in Gateway?  
307. Can you load balance routes in Gateway?  
308. What is path rewriting in Gateway?  
309. What is Resilience4j?  
310. How to use Resilience4j in Spring Boot?  
311. What happens when a circuit breaker is open?  
312. What is the fallback method in Resilience4j?  
313. How do you configure Resilience4j?  

## 🔟 Performance, Observability & Production Readiness  
326. How do you analyze performance bottlenecks in Spring Boot apps?  
327. What is lazy initialization in Spring Boot?  
328. How to reduce startup time of Spring Boot apps?  
329. What is the role of @Indexed in performance?  
330. How do you optimize Hibernate performance?  
332. How do you handle memory leaks in Spring Boot apps?  
333. What is garbage collection tuning in Spring Boot?  
334. What is observability in microservices?  
335. What is the difference between monitoring and observability?  
336. What are the 3 pillars of observability?  
337. How do you enable logging in Spring Boot?  
338. How do you structure logs for production?  
339. How do you trace requests across services?  
340. What does Sleuth add to logs?  
341. What is the difference between trace ID and span ID?  
342. How to collect metrics in Spring Boot?  
343. What are common out-of-box metrics from Actuator?  
344. How to create custom metrics?  
345. What is the role of tags/labels in metrics?  
346. What is Spring Boot Actuator and why is it important?  
347. How do you secure Actuator endpoints?  
348. What are critical Actuator endpoints to expose in production?  
349. How do you dynamically change logging level in production?  
350. What should go in the /info endpoint?  

## 1️⃣1️⃣ Spring AI — Latest 2025 Features  
449. What is Spring AI and why was it created?  
450. How do you set up Spring AI in a Spring Boot project?  
451. What AI models are supported by Spring AI?  
452. How do you configure OpenAI integration in Spring Boot?  
453. How do you configure Azure OpenAI in Spring Boot?  
454. What is the ChatClient interface and how do you use it?  
455. How do you create and manage prompts in Spring AI?  
456. What is prompt engineering and how do you implement it?  
457. How do you implement streaming responses from AI models?  
458. What are embeddings and how do you use them in Spring AI?  
459. What are vector stores and why are they important?  
460. How do you implement RAG (Retrieval Augmented Generation) with Spring AI?  
461. What is function calling in AI models and how do you use it?  
462. How do you build AI agents with Spring AI?  
463. How do you implement document processing with Spring AI?  
464. How do you handle AI model errors and retries?  
465. What is the difference between synchronous and asynchronous AI calls?  
466. How do you implement multi-modal AI (text, images) with Spring AI?  
467. How do you test AI integrations in Spring Boot?  
468. What are the best practices for production AI applications?  
469. How do you implement cost optimization for AI API calls?  
470. How do you implement caching for AI responses?  
471. What is prompt templating and how do you use it?  
472. How do you implement conversation memory and context management?  
473. How do you implement AI-powered search with vector similarity?  
474. What vector stores are supported by Spring AI?  
475. How do you configure vector stores in Spring Boot?  
476. What is the difference between RAG and fine-tuning?  
477. What is Model Context Protocol (MCP) and how do you use it in Spring AI?  
478. How do you implement observability and monitoring with Micrometer in Spring AI?  
479. How do you implement advanced chat memory with compaction and retention policies?  
480. How do you implement hybrid search in vector stores?  
481. What are enterprise guardrails and security features in Spring AI?  
482. How do you implement structured output validation with JSON Schema?  

🔥 Good luck crushing your Spring Boot interviews!  

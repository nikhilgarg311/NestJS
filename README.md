# NestJS Interview Questions & Detailed Answers

## **Round 1 – Basics (Foundation Check)**

### **1. What is NestJS and why would you use it instead of Express?**
NestJS is a progressive Node.js framework for building efficient, reliable, and scalable server-side applications.  
It is built on top of **Express.js** (or optionally **Fastify**) and uses **TypeScript** by default.  
Key advantages over Express:
- **TypeScript Support**: Built-in, providing static typing and better maintainability.
- **Modular Architecture**: Code is organized into modules for better scalability.
- **Dependency Injection**: Built-in DI container for cleaner, testable code.
- **Testability**: Out-of-the-box testing utilities.
- **Opinionated Structure**: Encourages best practices for large-scale applications.

---

### **2. Explain the architecture of a NestJS application.**
NestJS architecture is heavily inspired by Angular and follows a modular structure:
- **Modules**: Group related capabilities (controllers, services, providers).
- **Controllers**: Handle incoming HTTP requests and return responses.
- **Providers**: Services, repositories, and helpers managed by Nest’s DI container.
- **Dependency Injection Container**: Provides instances of classes automatically.
- **Pipes, Guards, Interceptors, Filters**: For validation, security, transformation, and error handling.

---

### **3. What are Modules in NestJS? Why is AppModule important?**
- **Modules**: Logical boundaries of an application, grouping related controllers and providers.
- **AppModule**: The root module, bootstrapped by NestJS, where all other modules are imported.

```ts
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

---

### **4. Difference between @Injectable() and @Controller()?**
- **@Injectable()**: Marks a class as a provider/service that can be injected into other components.
- **@Controller()**: Marks a class as a controller that handles HTTP requests.

---

### **5. How do you handle configuration in NestJS for different environments?**
- Use **@nestjs/config** module.
- Create `.env` files for dev, stage, prod.
- Inject configuration via `ConfigService`.

```ts
@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true })],
})
export class AppModule {}
```

---

### **6. Difference between synchronous and asynchronous providers?**
- **Synchronous**: Value is available immediately.
- **Asynchronous**: Created after async operations (e.g., DB connection).

```ts
{
  provide: 'ASYNC_PROVIDER',
  useFactory: async () => await someAsyncFunction(),
}
```

---

### **7. How does routing work in NestJS?**
- Decorators like `@Get()`, `@Post()` define routes.
- Route params: `@Param('id')`.
- Query params: `@Query('name')`.

```ts
@Get(':id')
findOne(@Param('id') id: string) {}
```

---

## **Round 2 – Intermediate (Job-Ready)**

### **8. Explain Dependency Injection in NestJS.**
DI means classes do not manually create dependencies; they request them via constructor injection.  
NestJS automatically resolves dependencies from its DI container.

---

### **9. Difference between @Inject() and constructor injection?**
- **Constructor Injection**: Automatic resolution by type.
- **@Inject()**: Explicit token-based resolution.

---

### **10. How to implement middleware in NestJS?**
- Implement `NestMiddleware` interface.
- Apply via `configure()` in a module.

---

### **11. Difference between middleware, guards, and interceptors:**
- **Middleware**: Runs before route handling.
- **Guards**: Decide if a request is allowed.
- **Interceptors**: Modify request/response.

---

### **12. What are Guards in NestJS?**
Guards implement `CanActivate` and decide access, e.g., role-based access.

---

### **13. What are Interceptors?**
Used for:
- Logging
- Transforming responses
- Caching
- Error mapping

---

### **14. Global error handling?**
- Use `@Catch()` and custom `ExceptionFilter`.
- Apply globally via `app.useGlobalFilters()`.

---

### **15. Validate incoming data?**
- Install `class-validator` & `class-transformer`.
- Use `ValidationPipe` globally.

---

### **16. Pipes vs Interceptors**
- **Pipes**: Transform/validate incoming data.
- **Interceptors**: Transform outgoing response.

---

### **17. Structuring large-scale microservices NestJS app:**
- Use domain-driven modules.
- Shared modules for common utilities.
- Separate gateway, API, and worker services.

---

### **18. Scaling WebSockets:**
- Use `@WebSocketGateway()`.
- Scale via Redis pub/sub with `socket.io-redis`.

---

## **Round 3 – Advanced (Product Company Grill)**

### **19. Microservices in NestJS:**
- Supported protocols: TCP, Redis, NATS, Kafka, gRPC.

---

### **20. @MessagePattern() vs REST routes:**
- `@MessagePattern()`: Listens for messages on microservice transport.
- REST: Handles HTTP requests.

---

### **21. Long-running tasks:**
- Use queues like **BullMQ** with Redis.

---

### **22. Distributed caching:**
- Use `CacheModule` with Redis store.

---

### **23. Request lifecycle:**
Middleware → Guards → Interceptors (before) → Pipes → Controller → Service → Interceptors (after) → Exception Filters.

---

### **24. @Module() internals:**
Registers imports, controllers, and providers into DI container.

---

### **25. Custom decorator:**
```ts
export const User = createParamDecorator(
  (data, ctx: ExecutionContext) => ctx.switchToHttp().getRequest().user,
);
```

---

### **26. Backward-compatible API changes:**
- Use versioning in URLs or headers.
- Keep old endpoints until clients migrate.

---

### **27. Securing NestJS API:**
- Helmet for headers
- CSRF protection middleware
- SQL injection prevention via ORM

---

### **28. Dynamic Modules:**
Modules whose configuration is determined at runtime.

---

### **29. GraphQL integration:**
- Use `@nestjs/graphql`.
- Define schemas & resolvers.

---

### **30. Unit testing a service with injected repository:**
Mock repository using `jest.fn()`.

---

### **31. WebSocket scaling problem debug:**
- Enable Redis adapter.
- Check sticky sessions on load balancer.
- Debug logs for connection drops.

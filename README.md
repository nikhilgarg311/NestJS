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

How It Works in NestJS
- You mark a class with @Injectable() so Nest knows it can be provided as a dependency.
- You request it in a constructor parameter of another class (like a controller or another service).
- Nest’s IoC (Inversion of Control) container automatically finds and injects the right instance.

---

### **9. Difference between @Inject() and constructor injection?**
- **Constructor Injection**: Automatic resolution by type.
- **@Inject()**: Explicit token-based resolution.

#### **1. Constructor Injection (Implicit Injection)**
This is the **most common** way to inject dependencies in NestJS.  
You simply declare a **parameter type** in your constructor, and Nest’s **Dependency Injection container** automatically matches it by **type**.

##### **Example**
```ts
@Injectable()
export class UserService {
  getUser() {
    return 'John Doe';
  }
}

@Controller()
export class AppController {
  constructor(private readonly userService: UserService) {} // No @Inject()
  
  @Get()
  getUser() {
    return this.userService.getUser();
  }
}
```

How it works:
- Nest sees private readonly userService: UserService in the constructor.
- It finds a provider registered with the token UserService (the class itself is the token).
- It injects the instance automatically.


#### 2. @Inject() (Explicit Injection)

`@Inject()` is used when:

- The dependency does not have a class type (e.g., a constant, string, or symbol).
- You registered the provider using a custom token.
- You want to be explicit about which token to resolve.

---

#### Example with a Custom Token

```typescript
export const DATABASE_CONNECTION = 'DATABASE_CONNECTION';

@Module({
  providers: [
    {
      provide: DATABASE_CONNECTION,
      useValue: { connect: () => 'Connected to DB' },
    },
  ],
})
export class AppModule {}

@Injectable()
export class UserService {
  constructor(
    @Inject(DATABASE_CONNECTION) private readonly dbConnection: any
  ) {}

  getStatus() {
    return this.dbConnection.connect();
  }
}
```

How it works:
- We manually register a provider with provide: DATABASE_CONNECTION.
- DATABASE_CONNECTION is not a class, so constructor type-based injection won’t work.
- We use @Inject(DATABASE_CONNECTION) to tell Nest explicitly which token to resolve.

---

### **10. How to implement middleware in NestJS?**
- Implement `NestMiddleware` interface.
- Apply via `configure()` in a module.

In **NestJS**, middleware is a function that is called **before** the route handler is invoked.  
It is commonly used for:

- Logging requests
- Authentication and token validation
- Modifying request/response objects
- Performing tasks before the request reaches a controller

---

#### **1. Middleware Basics in NestJS**

- **Runs before guards, interceptors, and pipes** in the request lifecycle.
- Can access both `req` and `res` objects (like in Express).
- Can either:
  - End the request-response cycle (e.g., sending a response directly).
  - Or call `next()` to pass control to the next middleware/handler.

---

#### **2. Creating a Middleware**

We can create middleware in two ways:
- **Functional Middleware** → A simple function.
- **Class-based Middleware** → Implements `NestMiddleware` interface.

**Example: Class-based Middleware**

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
    next(); // Pass to next handler
  }
}
```
---

### **11. Difference between middleware, guards, and interceptors:**
- **Middleware**: Runs before route handling.
- **Guards**: Decide if a request is allowed (Refer next question for more details).
- **Interceptors**: Modify request/response.

In NestJS, **middleware**, **guards**, and **interceptors** are all parts of the request lifecycle —  
but they serve **different purposes** and run at **different stages**.

**Purpose:**  
- Runs **before** the request reaches any route handler (controller method).  
- Used for **pre-processing** requests — similar to Express middleware.  

**Common Use Cases:**
- Logging HTTP requests
- Parsing request bodies
- Authentication token validation (basic)
- Adding properties to `req` object

**Key Points:**
- Works with raw `req` and `res` objects from Express or Fastify.
- Runs **before** guards, interceptors, and pipes.
- Can end the request-response cycle without calling the controller (by not calling `next()`).
- Not tied to NestJS-specific dependency injection for the route.

**Example: Class-based Middleware**
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`[${req.method}] ${req.url}`);
    next();
  }
}
```
---

### **12. What are Guards in NestJS?**
Guards implement `CanActivate` and decide access, e.g., role-based access.

### **Purpose**
- Decide if a given request is allowed to proceed to the route handler.
- Primarily used for **authorization** and **authentication**.

---

### **Common Use Cases**
- Checking if a user is logged in.
- Role-based access control.
- Permission checks.

---

### **Key Points**
- Implement the `CanActivate` interface.
- Run **after middleware**, but **before interceptors and pipes**.
- Return `true` to allow the request, `false` or throw an exception to block it.
- Fully integrated with **NestJS Dependency Injection** (can inject services).

---

### **Example**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const req = context.switchToHttp().getRequest();
    return !!req.user; // Allow if user is attached to the request
  }
}
```

---

### **13. What are Interceptors in NestJS?**

Interceptors in NestJS are special classes that can **intercept** incoming requests **before they reach the route handler**, and/or **outgoing responses** before they are sent to the client.  
They follow the **Aspect-Oriented Programming (AOP)** pattern, which allows you to **wrap additional behavior** around method execution.

---

##### **Purpose**
Interceptors are used for:
- **Logging** → Measure request duration, track incoming and outgoing data.
- **Transforming responses** → Modify the data before sending it back to the client (e.g., wrap in a consistent API format).
- **Caching** → Store results of expensive operations and return cached responses.
- **Error mapping** → Catch errors and transform them into a standardized format.
- **Additional cross-cutting concerns** → e.g., rate limiting, request/response modification.

---

##### **Key Points**
- Implement the `NestInterceptor` interface.
- Must define an `intercept()` method that has access to:
  - `ExecutionContext` → Provides details about the current request lifecycle.
  - `CallHandler` → Gives access to the stream of data returned from the route handler.
- Interceptors can:
  - Run **before** the route handler (pre-processing).
  - Run **after** the route handler (post-processing).
- Support **RxJS Observables** for asynchronous operations and transformation using operators like `map()` and `catchError()`.

---

##### **Basic Example: Logging Interceptor**
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before handler execution...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After handler execution... ${Date.now() - now}ms`)),
      );
  }
}
```

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

Although **Pipes** and **Interceptors** may seem similar because both can transform data, they serve **different purposes** and work at **different stages** of the request-response lifecycle.

---

#### **Pipes**
**Purpose:**  
- Handle **validation** and **transformation** of **incoming data** before it reaches the controller/route handler.

**Key Points:**
- Run **before** the controller method is called.
- Often used for:
  - Validating request payloads (e.g., DTO validation with `class-validator`).
  - Transforming input types (e.g., converting strings to numbers).
- Applied at parameter, method, or global level.

**Example:**
```typescript
@UsePipes(new ValidationPipe())
@Post()
createUser(@Body() createUserDto: CreateUserDto) {
  return this.userService.create(createUserDto);
}
```
#### **Interceptors**
**Purpose**
- Can manipulate the **request** before it reaches the route handler.
- Can transform the **outgoing response** before it’s sent back to the client.

---

**Key Points**
- Run **before and after** the route handler.
- Commonly used for:
  - **Response transformation** (e.g., wrapping all responses in a standard format).
  - **Logging and monitoring**.
  - **Caching**.
  - **Error mapping**.

---

**Example**
```typescript
@UseInterceptors(new TransformResponseInterceptor())
@Get()
getAllUsers() {
  return this.userService.findAll();
}
```

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

# Deep Dive: From HTTP Request to Database — Annotated Stack Trace & Architecture

> Generated from a real production stack trace in the kyc-sg-markets-api project.
> Goal: master every layer a request traverses, understand what is a building block vs a customization,
> and learn to read stack traces fluently.

---

## Table of Contents

1. [How to Read a Stack Trace](#1-how-to-read-a-stack-trace)
2. [The Full Annotated Stack Trace](#2-the-full-annotated-stack-trace)
3. [Layer 1 — Tomcat Internals](#3-layer-1--tomcat-internals)
4. [Layer 2 — Servlet Filters](#4-layer-2--servlet-filters)
5. [Layer 3 — Spring Security Filter Chain](#5-layer-3--spring-security-filter-chain)
6. [Layer 4 — Spring MVC Dispatch](#6-layer-4--spring-mvc-dispatch)
7. [Layer 5 — Spring AOP on the Controller](#7-layer-5--spring-aop-on-the-controller)
8. [Layer 6 — Your Application Code](#8-layer-6--your-application-code)
9. [Layer 7 — Spring Data JPA Repository Proxy](#9-layer-7--spring-data-jpa-repository-proxy)
10. [Layer 8 — JPA EntityManager Proxy](#10-layer-8--jpa-entitymanager-proxy)
11. [Layer 9 — Hibernate Session & Merge](#11-layer-9--hibernate-session--merge)
12. [Layer 10 — The Crash Site](#12-layer-10--the-crash-site)
13. [Architecture Deep Dive: Tomcat](#13-architecture-deep-dive-tomcat)
14. [Architecture Deep Dive: Spring MVC & How It Sits on Tomcat](#14-architecture-deep-dive-spring-mvc--how-it-sits-on-tomcat)
15. [Architecture Deep Dive: Hibernate & Spring Data JPA](#15-architecture-deep-dive-hibernate--spring-data-jpa)
16. [Embedded vs Standalone Tomcat](#16-embedded-vs-standalone-tomcat)
17. [Tomcat vs Jetty vs Undertow](#17-tomcat-vs-jetty-vs-undertow)

---

## 1. How to Read a Stack Trace

**Rule #1:** Read from BOTTOM to TOP. The bottom is where execution started; the top is where it crashed.

**Rule #2:** Group consecutive frames from the same package — they represent one logical step.

**Rule #3:** When you see `$Proxy`, `$$SpringCGLIB$$`, `Generated*Accessor` — those are dynamic proxies / AOP. They are the "glue" between layers. They are not your code and not bugs — they are infrastructure that enables interception (transactions, security, etc.).

**Rule #4:** The first frame with YOUR package name (e.g. `com.gbis.kyc`) is where you should focus to understand what triggered the chain.

---

## 2. The Full Annotated Stack Trace

Below is EVERY frame from your stack trace, in execution order (bottom to top = chronological order). Each frame is annotated with its role.

### ═══════════════════════════════════════════════════════════════
### LAYER 1: TOMCAT — Thread Pool & Connection Handling
### Pattern: Valve Pipeline (Chain of Responsibility)
### Role: Accept TCP connection, parse HTTP, hand to servlet container
### These are BUILDING BLOCKS of Tomcat — always present
### ═══════════════════════════════════════════════════════════════

```
java.lang.Thread.run()
```
> **Role:** JVM thread entry point. Every request runs on a thread from Tomcat's thread pool.
> **What it does:** Starts the thread's `Runnable.run()` method.
> **Category:** JVM building block.

```
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run()
```
> **Role:** Tomcat's thread wrapper. Wraps the actual task to set the correct classloader.
> **What it does:** Sets the thread context classloader to the webapp's classloader, then delegates.
> **Category:** Tomcat building block. Noise for debugging — skip.

```
org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run()
```
> **Role:** Java's standard thread pool worker.
> **What it does:** Pulls tasks from the work queue and runs them.
> **Category:** JDK building block. Noise.

```
org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker()
```
> **Role:** The actual executor loop.
> **What it does:** Calls `task.run()` for each queued task.
> **Category:** JDK building block. Noise.

```
org.apache.tomcat.util.net.SocketProcessorBase.run()
```
> **Role:** Entry point for processing a socket event.
> **What it does:** Wraps the socket processing in error handling, delegates to the concrete processor.
> **Category:** Tomcat building block.

```
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun()
```
> **Role:** NIO-specific socket processor.
> **What it does:** Reads data from the NIO channel (non-blocking I/O), dispatches to the protocol handler.
> **Why NIO?** Tomcat uses Java NIO (New I/O) for non-blocking socket handling. This allows one thread to manage many connections. When data is ready, it dispatches to a worker thread (this one).
> **Category:** Tomcat building block.

```
org.apache.coyote.AbstractProtocol$ConnectionHandler.process()
```
> **Role:** Protocol-level connection handler.
> **What it does:** Manages connection state machine (keep-alive, upgrades, timeouts). Creates/reuses the HTTP processor for this connection.
> **Category:** Tomcat building block (Coyote connector layer).

```
org.apache.coyote.AbstractProcessorLight.process()
```
> **Role:** Lightweight processor dispatching.
> **What it does:** Determines the type of socket event (read, write, error) and routes to the right handler method.
> **Category:** Tomcat building block. Noise.

```
org.apache.coyote.http11.Http11Processor.service()
```
> **Role:** HTTP/1.1 protocol processor. THIS IS WHERE HTTP PARSING HAPPENS.
> **What it does:**
> - Parses the raw HTTP request bytes (method, URL, headers, body)
> - Creates the internal `Request` and `Response` objects
> - Calls the adapter to hand off to the servlet container
> **Category:** Tomcat building block. Important — this is the boundary between "network bytes" and "HTTP request object."

```
org.apache.catalina.connector.CoyoteAdapter.service()
```
> **Role:** Bridge between Coyote (connector) and Catalina (servlet container).
> **What it does:**
> - Wraps Coyote's internal Request/Response into servlet-standard `HttpServletRequest` / `HttpServletResponse`
> - Maps the URL to the correct virtual host, context (webapp), and servlet
> - Starts the Valve pipeline
> **Category:** Tomcat building block. CRITICAL — this is where "Tomcat the network server" hands off to "Tomcat the servlet container."

### ─── Tomcat Valve Pipeline starts here ───
### Valves are Tomcat's interceptor pattern. They form a chain.
### Each valve does something, then calls the next valve.
### Think of it like a chain of middleware in Express.js.

```
org.apache.catalina.valves.RemoteIpValve.invoke()
```
> **Role:** Handles reverse proxy headers (X-Forwarded-For, X-Forwarded-Proto).
> **What it does:** If your app is behind a load balancer/proxy, this valve reads the forwarding headers and updates the request's remote IP and scheme (http→https) so your app sees the real client IP.
> **Category:** CUSTOMIZATION — added via server.xml or Spring Boot config. Not always present.

```
org.apache.catalina.valves.AbstractAccessLogValve.invoke()
```
> **Role:** Access logging.
> **What it does:** Logs the request (method, URL, status, time) to an access log file. Like Apache's `access.log`.
> **Category:** CUSTOMIZATION — configurable, can be removed.

```
org.apache.catalina.core.StandardEngineValve.invoke()
```
> **Role:** Engine-level valve — routes to the correct Host.
> **What it does:** In Tomcat's hierarchy: Engine → Host → Context → Wrapper. This valve selects the right Host (usually "localhost").
> **Category:** Tomcat building block.

```
org.apache.catalina.valves.ErrorReportValve.invoke()
```
> **Role:** Error page rendering.
> **What it does:** If a downstream valve/servlet throws an exception, this valve catches it and renders an error page (the default Tomcat error page with stack trace).
> **Category:** Tomcat building block (but configurable).

```
org.apache.catalina.core.StandardHostValve.invoke()
```
> **Role:** Host-level valve — routes to the correct Context (webapp).
> **What it does:** Selects the right webapp based on the URL path prefix.
> **Category:** Tomcat building block.

```
ch.qos.logback.access.tomcat.LogbackValve.invoke()
```
> **Role:** Logback access logging integration.
> **What it does:** Alternative to Tomcat's built-in access log — logs requests using Logback's formatting and appenders.
> **Category:** CUSTOMIZATION — added by your project. This is why you see `logback-access` in your dependencies.

```
org.apache.catalina.authenticator.AuthenticatorBase.invoke()
```
> **Role:** Container-managed authentication.
> **What it does:** Checks if the resource requires authentication at the container level. For Spring Security apps, this usually just passes through (Spring Security handles auth itself).
> **Category:** Tomcat building block (usually a no-op when Spring Security is present).

```
org.apache.catalina.core.StandardContextValve.invoke()
```
> **Role:** Context-level valve — routes to the correct Wrapper (servlet).
> **What it does:** Selects the servlet that matches the URL mapping.
> **Category:** Tomcat building block.

```
org.apache.catalina.core.StandardWrapperValve.invoke()
```
> **Role:** Wrapper-level valve — THE FINAL VALVE. Invokes the servlet's filter chain.
> **What it does:**
> - Allocates a servlet instance (or reuses one)
> - Builds the FilterChain
> - Calls `filterChain.doFilter()` which starts the servlet filter pipeline
> **Category:** Tomcat building block. CRITICAL — this is where Tomcat's valve pipeline ends and the Servlet Filter chain begins.



### ═══════════════════════════════════════════════════════════════
### LAYER 2: SERVLET FILTERS (before Spring Security)
### Pattern: Chain of Responsibility (FilterChain)
### Role: Pre/post-process every request before it reaches the servlet
### These run INSIDE the servlet container but BEFORE your controller
### ═══════════════════════════════════════════════════════════════

```
org.apache.catalina.core.ApplicationFilterChain.internalDoFilter()
org.apache.catalina.core.ApplicationFilterChain.doFilter()
```
> **Role:** Tomcat's implementation of the `FilterChain` interface.
> **What it does:** Maintains an array of filters. Calls each filter in order. Each filter calls `chain.doFilter()` to pass to the next one. After the last filter, calls the servlet's `service()` method.
> **Category:** Tomcat building block.
> **NOTE:** You'll see these two frames repeated MANY times — once per filter. They are NOISE. Train your eye to skip them and focus on the filter class between each pair.

```
org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Sets character encoding (UTF-8) on request and response.
> **What it does:** Calls `request.setCharacterEncoding("UTF-8")`. Simple but important — without it, special characters in form data break.
> **Category:** Spring Boot auto-configured CUSTOMIZATION.
> **Note:** `OncePerRequestFilter` is a Spring base class that ensures the filter runs only once per request (even if the request is forwarded internally). Most Spring filters extend this — the `doFilter()` call is noise, `doFilterInternal()` is the real logic.

```
org.apache.catalina.core.ApplicationFilterChain.internalDoFilter()
org.apache.catalina.core.ApplicationFilterChain.doFilter()
```
> Noise — FilterChain mechanics, calling the next filter.

```
org.springframework.web.filter.ServerHttpObservationFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Micrometer observation/metrics collection.
> **What it does:** Starts an "observation" (timer + tags) for this HTTP request. Records duration, status code, URI pattern for metrics (Prometheus, etc.).
> **Category:** CUSTOMIZATION — added by Spring Boot Actuator / Micrometer.

```
org.apache.catalina.core.ApplicationFilterChain.internalDoFilter()
org.apache.catalina.core.ApplicationFilterChain.doFilter()
```
> Noise.

```
org.springframework.web.servlet.v6_0.OpenTelemetryHandlerMappingFilter
```
> **Role:** OpenTelemetry distributed tracing integration.
> **What it does:** Creates or propagates a trace span for this HTTP request. Adds the HTTP route pattern to the span so traces show meaningful endpoint names instead of raw URLs.
> **Category:** CUSTOMIZATION — added by the OpenTelemetry Java agent (`io.opentelemetry.javaagent`). This is an auto-instrumentation library — it injects this filter at startup without you writing any code.

```
org.apache.catalina.core.ApplicationFilterChain.internalDoFilter()
org.apache.catalina.core.ApplicationFilterChain.doFilter()
```
> Noise.

```
org.springframework.web.filter.FormContentFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Parses form data for PUT/PATCH/DELETE requests.
> **What it does:** By default, servlet containers only parse form data (`application/x-www-form-urlencoded`) for POST requests. This filter adds the same parsing for PUT, PATCH, DELETE.
> **Category:** Spring Boot auto-configured CUSTOMIZATION.

```
org.apache.catalina.core.ApplicationFilterChain.internalDoFilter()
org.apache.catalina.core.ApplicationFilterChain.doFilter()
```
> Noise.

```
org.springframework.web.filter.RequestContextFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Exposes request attributes to the current thread via `RequestContextHolder`.
> **What it does:** Stores the current `HttpServletRequest` in a `ThreadLocal` so that any code in the call chain can access the request via `RequestContextHolder.getRequestAttributes()` without passing it as a parameter.
> **Category:** Spring Boot auto-configured building block.

```
org.apache.catalina.core.ApplicationFilterChain.internalDoFilter()
org.apache.catalina.core.ApplicationFilterChain.doFilter()
```
> Noise.

### ─── DelegatingFilterProxy: the bridge from Servlet world to Spring world ───

```
org.springframework.web.filter.DelegatingFilterProxy.doFilter()
org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate()
```
> **Role:** THE BRIDGE between Tomcat's filter chain and Spring's filter chain.
> **What it does:** Tomcat knows nothing about Spring beans. `DelegatingFilterProxy` is a standard servlet filter registered with Tomcat that looks up a Spring bean by name and delegates to it. The bean it delegates to is `springSecurityFilterChain`.
> **Category:** Spring Security building block. CRITICAL — this is where Tomcat's world ends and Spring Security's world begins in the filter layer.
> **Key insight:** Tomcat manages the outer `FilterChain`. Spring Security manages its own inner `VirtualFilterChain`. `DelegatingFilterProxy` is the adapter between the two.

```
org.springframework.security.config.annotation.web.configuration.WebMvcSecurityConfiguration$CompositeFilterChainProxy
org.springframework.web.filter.CompositeFilter.doFilter()
org.springframework.web.filter.CompositeFilter$VirtualFilterChain.doFilter()
```
> **Role:** Spring Security's composite wrapper.
> **What it does:** Wraps multiple filter chains into one. In Spring Security 6+, the security configuration can define multiple `SecurityFilterChain` beans — this composite selects the right one based on the request URL pattern.
> **Category:** Spring Security building block.

```
org.springframework.web.servlet.handler.HandlerMappingIntrospector.doFilter()
org.springframework.web.filter.CompositeFilter$VirtualFilterChain.doFilter()
```
> **Role:** Caches handler mapping info for Spring Security to use.
> **What it does:** Spring Security needs to know which controller method will handle the request (to check `@PreAuthorize`, for example). This filter pre-computes that mapping so it's available to the security filters.
> **Category:** Spring Security building block.

```
org.springframework.web.filter.ServletRequestPathFilter
org.springframework.web.filter.CompositeFilter$VirtualFilterChain.doFilter()
```
> **Role:** Parses and caches the request path.
> **What it does:** Parses the URL path once and stores it as a `RequestPath` object on the request attributes, so all downstream components can use the parsed path without re-parsing.
> **Category:** Spring MVC building block. Noise.

### ═══════════════════════════════════════════════════════════════
### LAYER 3: SPRING SECURITY FILTER CHAIN
### Pattern: Virtual Filter Chain (Chain of Responsibility inside Spring)
### Role: Authentication, authorization, CSRF, CORS, session management
### These are CUSTOMIZATIONS — you choose which ones to enable
### ═══════════════════════════════════════════════════════════════

```
org.springframework.security.web.FilterChainProxy.doFilter()
org.springframework.security.web.FilterChainProxy.doFilterInternal()
```
> **Role:** Spring Security's main filter chain proxy. THE ENTRY POINT of Spring Security.
> **What it does:** Selects the correct `SecurityFilterChain` for this request (based on URL patterns configured in your `SecurityConfiguration`) and iterates through its filters.
> **Category:** Spring Security building block.

```
org.springframework.security.web.ObservationFilterChainDecorator$VirtualFilterChain.doFilter()
```
> **Role:** Spring Security's own filter chain iterator, wrapped with observability.
> **What it does:** Same as `ApplicationFilterChain` but for Spring Security's internal filters. The `ObservationFilterChainDecorator` adds timing/metrics to each security filter.
> **Category:** Spring Security building block. Noise — you'll see this between every security filter.

```
org.springframework.security.web.ObservationFilterChainDecorator$ObservationFilter.doFilter() (×2)
```
> **Role:** Observation wrapper around each individual security filter.
> **What it does:** Records timing metrics for the wrapped security filter. The "×2" is because it wraps both `doFilterBefore` and `doFilterAfter` around the actual filter.
> **Category:** Noise — metric wrapper.

#### Security Filter 1: DisableEncodeUrlFilter
```
org.springframework.security.web.session.DisableEncodeUrlFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Prevents session ID from being appended to URLs.
> **What it does:** Wraps the response to override `encodeURL()` / `encodeRedirectURL()` to be no-ops. This prevents `;jsessionid=xxx` from appearing in URLs (a security risk — session fixation).
> **Category:** Spring Security CUSTOMIZATION (enabled by default).

#### Security Filter 2: ChannelProcessingFilter
```
org.springframework.security.web.access.channel.ChannelProcessingFilter.doFilter()
```
> **Role:** Enforces HTTPS.
> **What it does:** If a request comes over HTTP but the endpoint requires HTTPS (or vice versa), redirects to the correct scheme.
> **Category:** Spring Security CUSTOMIZATION.

#### Security Filter 3: WebAsyncManagerIntegrationFilter
```
org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Propagates SecurityContext to async threads.
> **What it does:** When a controller uses `@Async` or `DeferredResult`, the security context needs to be available on the async thread. This filter ensures that.
> **Category:** Spring Security building block.

#### Security Filter 4: SecurityContextHolderFilter
```
org.springframework.security.web.context.SecurityContextHolderFilter.doFilter()
org.springframework.security.web.context.SecurityContextHolderFilter.doFilter()
```
> **Role:** Sets up the SecurityContext for the current request.
> **What it does:** Loads the `SecurityContext` (which holds the `Authentication` object = "who is the user?") from the `SecurityContextRepository` and puts it in `SecurityContextHolder` (a ThreadLocal). After the request, clears it.
> **Two frames:** The double frame is because the outer `doFilter` does setup, calls the chain, then does cleanup — the second frame is the same method but at the cleanup phase.
> **Category:** Spring Security building block. CRITICAL — without this, no code downstream would know who the user is.

#### Security Filter 5: HeaderWriterFilter
```
org.springframework.security.web.header.HeaderWriterFilter.doFilter()
org.springframework.security.web.header.HeaderWriterFilter.doFilter()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Adds security-related HTTP response headers.
> **What it does:** Adds headers like:
> - `X-Content-Type-Options: nosniff`
> - `X-Frame-Options: DENY`
> - `Strict-Transport-Security: max-age=...`
> - `Cache-Control: no-cache, no-store`
> **Category:** Spring Security CUSTOMIZATION (enabled by default).

#### Security Filter 6: CorsFilter
```
org.springframework.web.filter.CorsFilter.doFilter()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Handles Cross-Origin Resource Sharing (CORS).
> **What it does:** For preflight requests (OPTIONS), responds with CORS headers. For actual requests, adds `Access-Control-Allow-Origin` etc.
> **Category:** Spring Security CUSTOMIZATION — configured in your `SecurityConfiguration`.

#### Security Filter 7: LogoutFilter
```
org.springframework.security.web.authentication.logout.LogoutFilter.doFilter()
org.springframework.security.web.authentication.logout.LogoutFilter.doFilter()
```
> **Role:** Handles logout requests.
> **What it does:** If the request matches the logout URL (default `/logout`), invalidates the session and clears the SecurityContext. Otherwise, passes through (no-op for non-logout requests).
> **Category:** Spring Security CUSTOMIZATION.

#### Security Filter 8: AuthenticationServiceUnavailableHandlerFilter
```
com.socgen.apibank.autoconfigure.sgconnect.servlet.AuthenticationServiceUnavailableHandlerFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** SG Connect custom authentication resilience filter.
> **What it does:** Catches authentication service unavailability and returns a proper error response instead of a 500. This is a SocGen-specific CUSTOMIZATION.
> **Category:** YOUR COMPANY's customization.

#### Security Filter 9: BearerTokenAuthenticationFilter
```
org.springframework.security.oauth2.server.resource.web.authentication.BearerTokenAuthenticationFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** OAuth2 Bearer token extraction and validation. THIS IS WHERE YOUR JWT IS VALIDATED.
> **What it does:**
> 1. Extracts the Bearer token from the `Authorization` header
> 2. Creates a `BearerTokenAuthenticationToken`
> 3. Passes it to the `AuthenticationManager` which validates the JWT (signature, expiry, claims)
> 4. If valid, creates an `Authentication` object and stores it in `SecurityContextHolder`
> **Category:** Spring Security CUSTOMIZATION — enabled by `oauth2ResourceServer()` in your SecurityConfiguration.
> **IMPORTANT FOR YOUR FLOW:** After this filter, the system knows WHO the user is.

#### Security Filter 10: AuthenticationFilter
```
org.springframework.security.web.authentication.AuthenticationFilter.doFilter()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** Generic authentication filter (SG Connect integration).
> **What it does:** Additional authentication logic beyond Bearer token — may handle SG-specific authentication schemes.
> **Category:** CUSTOMIZATION.

#### Security Filter 11: RequestCacheAwareFilter
```
org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter()
```
> **Role:** Restores saved requests after authentication redirect.
> **What it does:** If a user was redirected to login, the original request (URL, parameters) was saved. After login, this filter restores that saved request. For API calls, usually a no-op.
> **Category:** Spring Security building block. Noise for REST APIs.

#### Security Filter 12: SecurityContextHolderAwareRequestFilter
```
org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter()
```
> **Role:** Makes servlet API security methods work with Spring Security.
> **What it does:** Wraps the `HttpServletRequest` so that `request.isUserInRole()`, `request.getUserPrincipal()`, `request.getRemoteUser()` all delegate to Spring Security's `SecurityContext`.
> **Category:** Spring Security building block.

#### Security Filter 13: AnonymousAuthenticationFilter
```
org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter()
```
> **Role:** Ensures there's always an Authentication object.
> **What it does:** If no previous filter set an Authentication (user is not logged in), this creates an `AnonymousAuthenticationToken`. This way, downstream code can always call `SecurityContextHolder.getContext().getAuthentication()` without null checks.
> **Category:** Spring Security building block.

#### Security Filter 14: SessionManagementFilter
```
org.springframework.security.web.session.SessionManagementFilter.doFilter()
org.springframework.security.web.session.SessionManagementFilter.doFilter()
```
> **Role:** Session fixation protection and concurrent session control.
> **What it does:** Detects if a user just authenticated and migrates their session (creates new session ID, copies attributes). Also enforces max concurrent sessions if configured.
> **Category:** Spring Security CUSTOMIZATION.

#### Security Filter 15: ExceptionTranslationFilter
```
org.springframework.security.web.access.ExceptionTranslationFilter.doFilter()
org.springframework.security.web.access.ExceptionTranslationFilter.handleAccessDeniedException()
```
> **Role:** Converts security exceptions to HTTP responses. THE CATCH BLOCK FOR SECURITY.
> **What it does:** Wraps everything downstream in a try/catch. If an `AccessDeniedException` or `AuthenticationException` is thrown anywhere below, this filter converts it to a 403 or 401 response (or redirects to login).
> **Category:** Spring Security building block. CRITICAL.

#### Security Filter 16: AuthorizationFilter
```
org.springframework.security.web.access.intercept.AuthorizationFilter.doFilter()
```
> **Role:** URL-level authorization. THE FIRST AUTHORIZATION CHECK.
> **What it does:** Checks if the current user is authorized to access this URL based on the rules defined in `SecurityConfiguration` (e.g., `.requestMatchers("/api/**").authenticated()`).
> **Category:** Spring Security building block. CRITICAL.

```
org.springframework.security.web.FilterChainProxy.doFilter()
```
> **Role:** End of the security filter chain — control returns to the outer filter chain.
> **Category:** Spring Security building block.

### ─── Back to Servlet Filters (after Security) ───

```
org.springframework.web.filter.CompositeFilter$VirtualFilterChain.doFilter()
org.springframework.web.filter.CompositeFilter.doFilter()
org.springframework.security.config.annotation.web.configuration.WebSecurityConfiguration$CompositeFilterChainProxy.doFilter()
```
> Noise — unwinding the composite filter chain wrapper.

```
org.springframework.web.filter.ResourceUrlEncodingFilter.doFilter()
```
> **Role:** URL rewriting for versioned static resources.
> **What it does:** Rewrites URLs for static resources to include content hashes (e.g., `/css/style-abc123.css`) for cache-busting. Mostly irrelevant for REST APIs.
> **Category:** Spring MVC CUSTOMIZATION. Noise for your API.

```
com.socgen.apibank.sgmonitoring.server.servlet.SgObservationBodySizeWebFilter.doFilterInternal()
org.springframework.web.filter.OncePerRequestFilter.doFilter()
```
> **Role:** SG Monitoring — tracks request/response body sizes.
> **What it does:** Measures the body size of requests and responses for monitoring/alerting.
> **Category:** YOUR COMPANY's customization.

```
com.socgen.apibank.http.ratelimiter.RateLimiterFilter.doFilter()
```
> **Role:** Rate limiting.
> **What it does:** Checks if the current client has exceeded their allowed request rate. If so, returns 429 Too Many Requests.
> **Category:** YOUR COMPANY's customization.

```
org.apache.tomcat.websocket.server.WsFilter.doFilter()
```
> **Role:** WebSocket upgrade detection.
> **What it does:** Checks if the request is a WebSocket upgrade request. If not (which is your case — it's a normal REST call), passes through.
> **Category:** Tomcat building block. Noise for REST APIs.

### ═══════════════════════════════════════════════════════════════
### LAYER 4: SPRING MVC — DispatcherServlet
### Pattern: Front Controller + Handler Adapter + Strategy
### Role: Map request to controller method, invoke it, render response
### This is where "Tomcat's Servlet" becomes "Spring's Controller"
### ═══════════════════════════════════════════════════════════════

```
jakarta.servlet.http.HttpServlet.service()
```
> **Role:** Standard Servlet API entry point.
> **What it does:** Routes to `doGet()`, `doPost()`, etc. based on HTTP method. Spring's `FrameworkServlet` overrides these.
> **Category:** Servlet API building block.

```
org.springframework.web.servlet.FrameworkServlet.service()
```
> **Role:** Spring's servlet base class — overrides `HttpServlet.service()`.
> **What it does:** Delegates to `processRequest()` after setting up locale, theme, and request attributes.
> **Category:** Spring MVC building block.

```
jakarta.servlet.http.HttpServlet.service() (2nd occurrence)
```
> Noise — internal dispatch within the servlet API.

```
org.springframework.web.servlet.FrameworkServlet.doPost() / doGet() / etc.
```
> **Role:** HTTP-method-specific handler.
> **What it does:** Calls `processRequest()` which calls `doService()`.
> **Category:** Spring MVC building block. Noise.

```
org.springframework.web.servlet.FrameworkServlet.processRequest()
```
> **Role:** Request lifecycle manager.
> **What it does:** Sets up `LocaleContext`, `RequestAttributes`, publishes `ServletRequestHandledEvent` after completion.
> **Category:** Spring MVC building block.

```
org.springframework.web.servlet.DispatcherServlet.doService()
```
> **Role:** THE HEART OF SPRING MVC.
> **What it does:** Sets framework-specific request attributes, then calls `doDispatch()`.
> **Category:** Spring MVC building block. CRITICAL.

```
org.springframework.web.servlet.DispatcherServlet.doDispatch()
```
> **Role:** The main dispatch algorithm. THIS IS WHERE THE MAGIC HAPPENS.
> **What it does (step by step):**
> 1. Calls `getHandler()` — iterates through `HandlerMapping` beans to find which controller method matches the URL + HTTP method. Returns a `HandlerExecutionChain` (handler + interceptors).
> 2. Calls `getHandlerAdapter()` — finds the right `HandlerAdapter` that can invoke the handler (almost always `RequestMappingHandlerAdapter` for `@RequestMapping` methods).
> 3. Calls pre-handle interceptors (`HandlerInterceptor.preHandle()`).
> 4. Calls `handlerAdapter.handle()` — invokes the controller method.
> 5. Calls post-handle interceptors (`HandlerInterceptor.postHandle()`).
> 6. Resolves the view and renders the response (for REST APIs, Jackson serialization happens here).
> **Category:** Spring MVC building block. CRITICAL — this is the algorithm you need to understand.

```
org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle()
```
> **Role:** Adapter pattern — adapts `HandlerMethod` to the `HandlerAdapter` interface.
> **What it does:** Calls `handleInternal()` on the concrete adapter.
> **Category:** Spring MVC building block. Noise.

```
org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal()
```
> **Role:** The adapter for `@RequestMapping` / `@GetMapping` / `@PostMapping` etc.
> **What it does:** Checks if session synchronization is needed, then calls `invokeHandlerMethod()`.
> **Category:** Spring MVC building block.

```
org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod()
```
> **Role:** Prepares the invocation environment for the controller method.
> **What it does:**
> - Creates `ServletInvocableHandlerMethod` wrapper
> - Sets up argument resolvers (how to convert request params/body to Java objects)
> - Sets up return value handlers (how to convert Java objects to response body)
> - Calls `invokeAndHandle()` on the `ServletInvocableHandlerMethod`
> **Category:** Spring MVC building block. IMPORTANT — this is where `@RequestBody`, `@PathVariable`, `@RequestParam` resolvers are configured.

```
org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle()
```
> **Role:** Invokes the method AND handles the return value.
> **What it does:** Calls `invokeForRequest()` then passes the return value to `HandlerMethodReturnValueHandler` (e.g., `RequestResponseBodyMethodProcessor` for `@ResponseBody`).
> **Category:** Spring MVC building block.

```
org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest()
```
> **Role:** Resolves method arguments and invokes the method via reflection.
> **What it does:**
> 1. For each parameter of the controller method, finds the right `HandlerMethodArgumentResolver` and resolves the value.
> 2. Calls `doInvoke()` with the resolved arguments.
> **Category:** Spring MVC building block.

```
org.springframework.web.method.support.InvocableHandlerMethod.doInvoke()
```
> **Role:** Actual reflective invocation.
> **What it does:** Calls `method.invoke(bean, args)` via Java reflection. But because the controller bean is proxied (CGLIB), the call goes to the proxy first.
> **Category:** Spring MVC building block.

```
java.lang.reflect.Method.invoke()
jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke()
jdk.internal.reflect.GeneratedMethodAccessor5019.invoke()
```
> **Role:** JDK reflection machinery.
> **What it does:** `Method.invoke()` delegates to a `MethodAccessor`. The JVM generates optimized accessor classes (`GeneratedMethodAccessor5019`) at runtime after enough invocations (replaces slower `NativeMethodAccessorImpl`).
> **Category:** JVM building block. Noise.

### ═══════════════════════════════════════════════════════════════
### LAYER 5: SPRING AOP ON THE CONTROLLER (CGLIB Proxy)
### Pattern: Proxy / Decorator (CGLIB bytecode generation)
### Role: Intercept controller method calls for security, transactions
### ═══════════════════════════════════════════════════════════════

```
com.gbis.kyc.sg.market.api.controller.ECollectApiController$$SpringCGLIB$$0
```
> **Role:** CGLIB-generated subclass of your controller.
> **What it does:** Spring cannot intercept method calls on a plain object. So it creates a subclass at runtime using CGLIB bytecode generation. This subclass overrides every method and routes calls through the AOP interceptor chain before calling the real method.
> **Category:** Spring AOP building block. Noise — but important to understand WHY it exists.
> **Key insight:** `$$SpringCGLIB$$0` means "first proxy generated by Spring CGLIB for this class."

```
org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept()
```
> **Role:** The CGLIB interceptor that drives the AOP chain.
> **What it does:** Creates a `ReflectiveMethodInvocation` and calls `proceed()` on it — which walks through all the AOP interceptors (advisors) registered for this method.
> **Category:** Spring AOP building block.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
```
> **Role:** AOP chain walker.
> **What it does:** Iterates through the list of `MethodInterceptor`s. Calls each one's `invoke()`, which may call `proceed()` again to continue the chain, or short-circuit.
> **Category:** Spring AOP building block. Noise — you'll see this between every interceptor.

#### AOP Interceptor 1: Spring Security Method-Level Authorization
```
org.springframework.security.config.annotation.method.configuration.DeferringMethodInterceptor.invoke()
```
> **Role:** Lazy-loading wrapper for method security interceptors.
> **What it does:** Defers the creation of the actual security interceptor until first use.
> **Category:** Spring Security building block. Noise.

```
org.springframework.security.authorization.method.AuthorizationManagerBeforeMethodInterceptor.invoke()
org.springframework.security.authorization.method.AuthorizationManagerBeforeMethodInterceptor.attemptAuthorization()
org.springframework.security.authorization.method.AuthorizationManagerBeforeMethodInterceptor.authorize()
```
> **Role:** METHOD-LEVEL AUTHORIZATION. THIS IS WHERE `@PreAuthorize` IS CHECKED.
> **What it does:** 
> 1. Evaluates the SpEL expression in `@PreAuthorize` (e.g., `@PreAuthorize("hasRole('ADMIN')")`)
> 2. If the expression returns `false`, throws `AccessDeniedException` (caught by `ExceptionTranslationFilter` above → 403)
> 3. If `true`, calls `proceed()` to continue to the actual controller method.
> **Category:** Spring Security CUSTOMIZATION. CRITICAL — this is the second authorization layer (first was URL-level in `AuthorizationFilter`).
> **Difference from AuthorizationFilter:**
> - `AuthorizationFilter` = URL-based rules: "all `/api/**` must be authenticated"
> - `AuthorizationManagerBeforeMethodInterceptor` = method-based rules: "this specific method requires this specific role/permission"

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
```
> Noise — continuing the AOP chain after security check passed.

```
java.lang.reflect.Method.invoke()
jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke()
jdk.internal.reflect.GeneratedMethodAccessor5019.invoke()
```
> Noise — JDK reflection invoking the actual controller method.

### ═══════════════════════════════════════════════════════════════
### LAYER 6: YOUR APPLICATION CODE
### Pattern: Controller → Service → Service → Repository
### Role: Your business logic
### ═══════════════════════════════════════════════════════════════

```
com.gbis.kyc.sg.market.api.controller.ECollectApiController (endpoint method)
```
> **Role:** YOUR controller method — the HTTP endpoint.
> **What it does:** Receives the request (already deserialized from JSON by Spring MVC), calls `ApiResponseHandler`.
> **Category:** YOUR CODE. IMPORTANT.

```
com.gbis.kyc.sg.market.shared.jpa.controller.ApiResponseHandler (×2)
```
> **Role:** Custom response wrapper from your shared library.
> **What it does:** Wraps the controller logic to standardize API responses (success/error envelope). The two frames are likely `handleApiResponse()` calling an internal method.
> **Category:** YOUR CODE — shared library utility.

```
com.gbis.kyc.sg.market.shared.jpa.controller.TransactionHandler$$SpringCGLIB$$0
```
> **Role:** CGLIB proxy for TransactionHandler.
> **What it does:** `TransactionHandler` is a Spring bean with `@Transactional`. CGLIB proxy intercepts the call to manage transaction boundaries.
> **Category:** Spring AOP. Noise.

```
org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept()
org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection()
java.lang.reflect.Method.invoke()
jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke()
jdk.internal.reflect.GeneratedMethodAccessor779.invoke()
```
> Noise — CGLIB proxy mechanics + reflection for TransactionHandler.

```
com.gbis.kyc.sg.market.shared.jpa.controller.TransactionHandler (first real method)
```
> **Role:** Custom transaction management wrapper from your shared library.
> **What it does:** Wraps your business logic in a transaction.
> **Category:** YOUR CODE — shared library.

#### ─── @Transactional boundary starts here (first pass) ───
```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.transaction.interceptor.TransactionInterceptor.invoke()
org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction()
```
> **Role:** SPRING TRANSACTION MANAGEMENT. THE @Transactional INTERCEPTOR.
> **What it does:**
> 1. Checks if a transaction already exists (propagation handling)
> 2. If `REQUIRED` (default): joins existing or creates new DB transaction
> 3. Calls `PlatformTransactionManager.getTransaction()` → opens DB connection, calls `BEGIN`
> 4. Invokes the actual method
> 5. If method returns normally: `commit()`
> 6. If method throws: `rollback()`
> **Category:** Spring building block. CRITICAL — this is where the DB transaction starts.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept()
com.gbis.kyc.sg.market.shared.jpa.controller.TransactionHandler$$SpringCGLIB$$0
```
> Noise — second CGLIB proxy call on TransactionHandler (nested transactional method).

```
com.gbis.kyc.sg.market.shared.jpa.controller.TransactionHandler (second method)
```
> **Role:** Another `TransactionHandler` method (nested call inside the same transactional boundary).
> **Category:** YOUR CODE.

#### ─── @Transactional boundary (second pass — nested) ───
```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.transaction.interceptor.TransactionInterceptor.invoke()
org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction()
```
> **Role:** Second `@Transactional` interceptor.
> **What it does:** Since a transaction already exists (from the first TransactionHandler call), with default `REQUIRED` propagation, this just JOINS the existing transaction — no new BEGIN.
> **Category:** Spring building block.

```
org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection()
java.lang.reflect.Method.invoke()
jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke()
jdk.internal.reflect.GeneratedMethodAccessor780.invoke()
```
> Noise — reflection for invoking actual TransactionHandler method.

```
com.gbis.kyc.sg.market.shared.jpa.controller.TransactionHandler
```
> **Role:** Actual TransactionHandler logic.
> **Category:** YOUR CODE.

```
com.gbis.kyc.sg.market.api.controller.ECollectApiController
```
> **Role:** Back in your controller code (called by TransactionHandler via lambda/callback).
> **Category:** YOUR CODE.

#### ─── Your Service Layer ───
```
com.gbis.kyc.sg.market.api.service.ECollectCaseService
com.gbis.kyc.sg.market.api.service.ECollectCaseService
com.gbis.kyc.sg.market.api.service.ECollectCaseService
```
> **Role:** Three frames from ECollectCaseService — three chained method calls within the same service.
> **What it does:** Based on the stack and code: likely `refreshThirdPartySnapshot()` → `save()` flow or `updateCase()` → `updateCaseStatusesAndCascade()` → nested call.
> **Category:** YOUR CODE. IMPORTANT — this is your business logic.

```
com.gbis.kyc.sg.market.api.service.RefreshCaseService
```
> **Role:** The RefreshCaseService that contains the code we've been fixing.
> **What it does:** `refreshThirdPartySnapshotsAndRequirements(eCollectCase, performFullRefresh)` → eventually calls `eCollectCaseRepository.save(eCollectCase)`.
> **Category:** YOUR CODE. THIS IS WHERE THE BUG ORIGINATES (setting immutable lists on entities).

### ═══════════════════════════════════════════════════════════════
### LAYER 7: SPRING DATA JPA REPOSITORY PROXY
### Pattern: JDK Dynamic Proxy + AOP Interceptor Chain
### Role: Turn your interface method call into a JPA operation
### ═══════════════════════════════════════════════════════════════

```
jdk.proxy3.$Proxy306                          ← your repository interface proxy
```
> **Role:** JDK dynamic proxy implementing your `ECollectCaseRepository` interface.
> **What it does:** Unlike CGLIB (subclass), JDK dynamic proxies implement interfaces. Spring Data creates one for each `@Repository` interface. Every method call is intercepted.
> **Category:** Spring Data building block. Noise.

```
org.springframework.aop.framework.JdkDynamicAopProxy.invoke()
```
> **Role:** JDK proxy's InvocationHandler — drives the AOP chain for repository methods.
> **Category:** Spring AOP building block. Noise.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
```
> Noise — AOP chain walker (same as controller layer).

```
io.opentelemetry.javaagent.instrumentation.spring.data.v1_8.SpringDataInstrumentationModule$RepositoryInterceptor
```
> **Role:** OpenTelemetry tracing for Spring Data calls.
> **What it does:** Creates a span for the repository method call (e.g., `ECollectCaseRepository.save`) so it appears in distributed traces with its timing.
> **Category:** CUSTOMIZATION — auto-injected by OpenTelemetry Java agent.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.data.jpa.repository.support.CrudMethodMetadataPostProcessor$CrudMethodMetadataPopulatingMethodInterceptor
```
> **Role:** Captures CRUD method metadata (lock mode, query hints, entity graph).
> **What it does:** If you annotated your repository method with `@Lock`, `@QueryHints`, or `@EntityGraph`, this interceptor reads those annotations and stores the info for later use by the query executor.
> **Category:** Spring Data building block. Noise for simple `save()`.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.dao.support.PersistenceExceptionTranslationInterceptor
```
> **Role:** Translates persistence exceptions to Spring's DataAccessException hierarchy.
> **What it does:** Catches JPA/Hibernate-specific exceptions (e.g., `ConstraintViolationException`) and wraps them in Spring's portable `DataAccessException` subtypes.
> **Category:** Spring Data building block.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.transaction.interceptor.TransactionInterceptor.invoke()
```
> **Role:** Transaction management for the repository method.
> **What it does:** `SimpleJpaRepository` methods are `@Transactional`. This interceptor handles that — but since a transaction ALREADY EXISTS from the `TransactionHandler` above, it just joins it (REQUIRED propagation).
> **Category:** Spring building block.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor
```
> **Role:** Handles Java 8+ default methods on repository interfaces.
> **What it does:** If your repository has `default` methods, this interceptor routes calls to them. Otherwise, passes through.
> **Category:** Spring Data building block. Noise.

```
org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
org.springframework.data.repository.core.support.QueryExecutorMethodInterceptor (×2)
```
> **Role:** Decides HOW to execute the repository method.
> **What it does:** Checks if the method is a derived query (`findByEntityId`), a `@Query`, or a CRUD built-in (`save`, `findById`). Routes to the correct executor.
> **Category:** Spring Data building block.

```
org.springframework.data.repository.core.support.RepositoryMethodInvoker (×2)
org.springframework.data.repository.core.support.RepositoryMethodInvoker$RepositoryFragmentMethodInvoker
org.springframework.data.repository.core.support.RepositoryComposition$RepositoryFragments
org.springframework.data.repository.core.support.RepositoryComposition
org.springframework.data.repository.core.support.RepositoryFactorySupport$ImplementationMethodExecutionInterceptor
```
> **Role:** Repository composition machinery.
> **What it does:** Spring Data composes your repository from multiple "fragments" (e.g., `JpaRepository` + custom implementation). This finds the right fragment and invokes the method on it.
> **Category:** Spring Data building block. Noise.

```
org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection()
java.lang.reflect.Method.invoke()
jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke()
jdk.internal.reflect.GeneratedMethodAccessor1028.invoke()
```
> Noise — reflection to call the actual implementation method.

```
org.springframework.data.jpa.repository.support.SimpleJpaRepository.save()
```
> **Role:** THE ACTUAL SAVE IMPLEMENTATION. This is where Spring Data calls JPA.
> **What it does:**
> ```java
> if (entityInformation.isNew(entity)) {
>     em.persist(entity);  // INSERT
> } else {
>     return em.merge(entity);  // UPDATE — THIS IS YOUR CASE
> }
> ```
> Since `eCollectCase` already has an ID (loaded from DB), `isNew()` returns `false` → calls `em.merge()`.
> **Category:** Spring Data building block. CRITICAL.

### ═══════════════════════════════════════════════════════════════
### LAYER 8: JPA ENTITYMANAGER PROXY
### Pattern: JDK Dynamic Proxy (double-wrapped)
### Role: Manage EntityManager lifecycle (shared vs extended)
### ═══════════════════════════════════════════════════════════════

```
jdk.proxy3.$Proxy295                          ← SharedEntityManager proxy
org.springframework.orm.jpa.SharedEntityManagerCreator$SharedEntityManagerInvocationHandler.invoke()
java.lang.reflect.Method.invoke()
jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke()
jdk.internal.reflect.GeneratedMethodAccessor829.invoke()
```
> **Role:** Spring's shared EntityManager proxy.
> **What it does:** In Spring, you don't get a raw `EntityManager` — you get a proxy that delegates to the current transaction's EntityManager. This ensures the same EM is used across all DAOs within one transaction.
> **Category:** Spring JPA building block. Noise.

```
jdk.proxy3.$Proxy295                          ← ExtendedEntityManager proxy (second layer)
org.springframework.orm.jpa.ExtendedEntityManagerCreator$ExtendedEntityManagerInvocationHandler.invoke()
java.lang.reflect.Method.invoke()
jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke()
jdk.internal.reflect.GeneratedMethodAccessor829.invoke()
```
> **Role:** Extended EntityManager proxy — supports extended persistence context.
> **What it does:** Another proxy layer for managing the EntityManager lifecycle. In transaction-scoped contexts, this mostly passes through.
> **Category:** Spring JPA building block. Noise.

### ═══════════════════════════════════════════════════════════════
### LAYER 9: HIBERNATE SESSION — The Merge Operation
### Pattern: Event/Listener + Cascade (Visitor pattern on entity graph)
### Role: Synchronize in-memory entity state with the database
### ═══════════════════════════════════════════════════════════════

```
org.hibernate.internal.SessionImpl.merge()
org.hibernate.internal.SessionImpl.merge()
```
> **Role:** Hibernate's `Session.merge()` — entry point for the merge operation.
> **What it does:** Creates a `MergeEvent` and fires it to the event listener system.
> **Category:** Hibernate building block. CRITICAL.

```
org.hibernate.event.service.internal.EventListenerGroupImpl.fireEventOnEachListener()
```
> **Role:** Hibernate's event system — dispatches to all registered listeners for this event type.
> **What it does:** Iterates registered `MergeEventListener`s and calls `onMerge()` on each.
> **Category:** Hibernate building block (event/listener architecture).

```
org.hibernate.event.internal.DefaultMergeEventListener.onMerge()
org.hibernate.event.internal.DefaultMergeEventListener.onMerge() (overload)
org.hibernate.event.internal.DefaultMergeEventListener.entityIsDetached() / mergeTransientEntity()
org.hibernate.event.internal.DefaultMergeEventListener.cascadeOnMerge()
org.hibernate.event.internal.DefaultMergeEventListener.cascadeBeforeSave()
```
> **Role:** THE MERGE ALGORITHM — processes the root entity.
> **What it does:**
> 1. Determines entity state: DETACHED (has ID, not in session) vs TRANSIENT (no ID)
> 2. For DETACHED: loads the persisted version from DB/cache, copies state from your object onto it
> 3. For TRANSIENT: creates a new managed copy
> 4. **Then cascades**: for every `@OneToMany(cascade=MERGE/ALL)` collection, recursively merges children
> **Category:** Hibernate building block. CRITICAL.

#### ─── First cascade level: ECollectCase → ThirdPartySnapshot → KycAgreementSnapshots ───
```
org.hibernate.engine.internal.Cascade.cascade()
org.hibernate.engine.internal.Cascade.cascadeCollectionElements()
org.hibernate.engine.internal.Cascade.cascadeToOne()
org.hibernate.engine.internal.Cascade.cascade() (multiple frames)
```
> **Role:** Cascade walker — traverses entity relationships.
> **What it does:** For each `@OneToMany`/`@ManyToOne` with cascade, recursively calls `session.merge()` on each child entity.
> **Category:** Hibernate building block.

```
org.hibernate.engine.spi.CascadingActions$6.cascade()
org.hibernate.engine.spi.CascadingActions$6.performOnLazyInitializer()
```
> **Role:** The specific cascade action for MERGE.
> **What it does:** `CascadingActions$6` = the MERGE action enum constant. Calls `session.merge(childEntity)` for each cascaded child.
> **Category:** Hibernate building block.

```
org.hibernate.internal.SessionImpl.merge() (recursive)
org.hibernate.internal.SessionImpl.merge()
org.hibernate.event.service.internal.EventListenerGroupImpl.fireEventOnEachListener()
org.hibernate.event.internal.DefaultMergeEventListener.onMerge() (×5 frames)
```
> **Role:** Second-level merge — processing KycAgreementSnapshot children.
> **What it does:** Same algorithm as above, but now for each `KycAgreementSnapshot` in the collection. Then cascades again to THEIR children (KycRestrictionTypeSnapshot).
> **Category:** Hibernate building block.

#### ─── Second cascade level: KycAgreementSnapshot → KycRestrictionTypeSnapshot ───
```
org.hibernate.engine.internal.Cascade.cascade() (multiple frames)
org.hibernate.engine.spi.CascadingActions$6.cascade()
org.hibernate.internal.SessionImpl.merge() (recursive again)
org.hibernate.event.internal.DefaultMergeEventListener.onMerge() (×5 frames)
```
> **Role:** Third-level merge — processing KycRestrictionTypeSnapshot.
> **Category:** Hibernate building block.

### ═══════════════════════════════════════════════════════════════
### LAYER 10: THE CRASH — CollectionType.replaceElements
### Role: Hibernate copying collection state during merge
### ═══════════════════════════════════════════════════════════════

```
org.hibernate.type.TypeHelper.replace()
```
> **Role:** Replaces property values from the "source" entity (yours) to the "target" entity (managed copy).
> **What it does:** For each property of the entity, calls the appropriate `Type.replace()`. For collections, delegates to `CollectionType.replaceElements()`.
> **Category:** Hibernate building block.

```
org.hibernate.type.CollectionType.replaceElements()
org.hibernate.type.CollectionType.replaceElements()
```
> **Role:** REPLACES COLLECTION CONTENTS during merge.
> **What it does:** Takes the collection from your detached/transient entity and copies its elements into the managed entity's PersistentBag. Internally may call `.clear()` or `.add()` on the SOURCE collection depending on the Hibernate version.
> **Category:** Hibernate building block. CRITICAL — THIS IS WHERE THE CRASH HAPPENS.

```
org.hibernate.collection.spi.PersistentBag.addAll()
```
> **Role:** Hibernate's collection wrapper trying to populate itself.
> **What it does:** Calls `addAll()` to copy elements. Internally iterates and calls `.add()`.
> **Category:** Hibernate building block.

```
java.util.ImmutableCollections$AbstractImmutableCollection.add()
java.util.ImmutableCollections.uoe()
```
> **Role:** 💥 **THE CRASH** — `UnsupportedOperationException`.
> **What it does:** Your `Stream.toList()` produced an immutable `List12` or `ListN` (from `java.util.ImmutableCollections`). When Hibernate calls `.add()` on it — BOOM.
> **Category:** JDK. THE BUG.

---

## 13. Architecture Deep Dive: Tomcat

### Tomcat's Two Halves

```
┌─────────────────────────────────────────────┐
│                  COYOTE                       │  ← Network Connector
│  (NioEndpoint, Http11Processor, Adapter)     │
│  Handles: TCP sockets, HTTP parsing          │
├─────────────────────────────────────────────┤
│                 CATALINA                      │  ← Servlet Container
│  (Engine→Host→Context→Wrapper→FilterChain)   │
│  Handles: Servlet lifecycle, context, valves │
└─────────────────────────────────────────────┘
```

### Tomcat Hierarchy (each level has its own Valve pipeline)

```
Server (1 per JVM)
└── Service (groups connector + container)
    ├── Connector (Coyote — HTTP/1.1 NIO on port 8080)
    └── Engine (top-level container)
        └── Host (virtual host: "localhost")
            └── Context (webapp: your Spring Boot app = "/")
                └── Wrapper (servlet: DispatcherServlet)
                    └── FilterChain (servlet filters)
```

### The Valve Pattern
Each container level (Engine, Host, Context, Wrapper) has a **Pipeline** of **Valves**.
A Valve is like a filter but at the Tomcat container level (before servlets).
- **Standard valves** (always present): `StandardEngineValve`, `StandardHostValve`, `StandardContextValve`, `StandardWrapperValve`
- **Custom valves** (pluggable): `AccessLogValve`, `RemoteIpValve`, `ErrorReportValve`, `LogbackValve`

---

## 14. Architecture Deep Dive: Spring MVC & How It Sits on Tomcat

### Where Tomcat Stops and Spring Starts

```
Tomcat StandardWrapperValve.invoke()
    └── FilterChain.doFilter()
        └── ... filters ...
            └── DispatcherServlet.service()    ←── THIS IS THE BOUNDARY
                                                    Tomcat sees it as "just a Servlet"
                                                    Spring sees it as "the Front Controller"
```

**Key insight:** `DispatcherServlet` IS a standard `javax.servlet.http.HttpServlet`. Tomcat treats it like any other servlet — it has no idea Spring MVC exists inside. Spring MVC is **entirely contained within a single servlet**.

### Spring MVC Internal Architecture

```
DispatcherServlet.doDispatch()
    ├── HandlerMapping.getHandler()          → finds @RequestMapping match
    ├── HandlerAdapter.handle()              → invokes controller method
    │   ├── ArgumentResolvers               → @RequestBody, @PathVariable, etc.
    │   ├── Method.invoke()                 → actual controller call
    │   └── ReturnValueHandlers             → @ResponseBody → Jackson → JSON
    └── ViewResolver (for non-REST only)    → renders HTML template
```

### Embedded vs Standalone: What Changes?

| Aspect | Embedded (Spring Boot) | Standalone (WAR in Tomcat) |
|--------|----------------------|---------------------------|
| Who starts Tomcat? | Spring Boot's `main()` | External Tomcat process |
| Classloading | Single classloader | Tomcat parent + webapp child CL |
| Configuration | `application.yml` → programmatic | `server.xml` + `web.xml` |
| Filter registration | `@Bean FilterRegistrationBean` | `web.xml <filter>` entries |
| Valve registration | `TomcatServletWebServerFactory.addContextValves()` | `context.xml` |
| Stack trace | Identical from `Http11Processor` down | Identical |

**Bottom line:** The stack trace looks the same. Embedded just means Spring controls Tomcat's lifecycle.

---

## 15. Architecture Deep Dive: Hibernate & Spring Data JPA

### Where Spring Data Stops and Hibernate Starts

```
SimpleJpaRepository.save(entity)
    └── EntityManager.merge(entity)          ←── THE BOUNDARY
                                                  Spring Data calls JPA API
                                                  Hibernate implements JPA API
        └── SessionImpl.merge()              ←── Hibernate's implementation
```

**Spring Data JPA** = convenience layer that generates `SimpleJpaRepository` implementations from your interfaces. It calls standard JPA methods (`persist`, `merge`, `find`, `createQuery`).

**Hibernate** = the JPA provider that implements those methods. It manages the Session (first-level cache), dirty checking, SQL generation, and the event system.

### Hibernate Internal Architecture

```
┌──────────────────────────────────────────────────────┐
│                   SessionFactory                       │  ← One per app (immutable config)
│  - Entity metamodel, mappings, caches                 │
├──────────────────────────────────────────────────────┤
│                     Session                           │  ← One per transaction (unit of work)
│  - PersistenceContext (1st level cache)              │
│  - ActionQueue (pending SQL statements)              │
│  - Event system (listeners)                          │
├──────────────────────────────────────────────────────┤
│                  Event System                         │
│  merge() → MergeEvent → DefaultMergeEventListener    │
│  persist()→ PersistEvent→ DefaultPersistEventListener │
│  flush() → FlushEvent → DefaultFlushEventListener    │
├──────────────────────────────────────────────────────┤
│              Cascade Engine                           │
│  Walks @OneToMany(cascade=ALL) graph recursively     │
│  Calls session.merge()/persist() on each child       │
├──────────────────────────────────────────────────────┤
│          CollectionType / PersistentBag              │
│  Wraps your List/Set to track dirty state            │
│  replaceElements() during merge                      │
├──────────────────────────────────────────────────────┤
│              JDBC (bottom)                            │
│  PreparedStatement, Connection, actual SQL            │
└──────────────────────────────────────────────────────┘
```

### The merge() Algorithm (what you saw in the stack)

```
1. session.merge(entity)
2. Is entity in PersistenceContext (1st level cache)?
   - YES → return managed copy (already tracked)
   - NO → continue
3. Does entity have an ID?
   - YES → DETACHED: load from DB/cache, copy state onto managed copy
   - NO → TRANSIENT: create new managed copy (will INSERT)
4. For each cascaded association:
   - @OneToMany(cascade=MERGE) → for each child, goto step 1 (recursive)
5. For each collection property during state copy:
   - CollectionType.replaceElements() → copy elements into PersistentBag
     → THIS IS WHERE immutable lists break
```

---

## 16. Embedded vs Standalone Tomcat

| | Embedded | Standalone |
|---|---|---|
| **Boot sequence** | `SpringApplication.run()` → creates `TomcatWebServer` → starts connectors | `catalina.sh start` → deploys WAR → initializes `SpringBootServletInitializer` |
| **Deployment unit** | Fat JAR (includes Tomcat jars) | WAR file dropped into `webapps/` |
| **Tomcat configuration** | Java code via `WebServerFactoryCustomizer` | XML files (`server.xml`, `context.xml`) |
| **Port** | `server.port=8080` in `application.yml` | `<Connector port="8080">` in `server.xml` |
| **Multiple apps** | One app per JVM (typical) | Multiple WARs in one Tomcat instance |
| **Stack trace difference** | None from `Http11Processor` down | None |

---

## 17. Tomcat vs Jetty vs Undertow

All three are servlet containers. Spring Boot supports all three as embedded servers.

| Aspect | Tomcat | Jetty | Undertow |
|---|---|---|---|
| **Architecture** | Valve pipeline | Handler chain | Handler chain |
| **I/O model** | NIO (non-blocking) | NIO | XNIO (non-blocking) |
| **Threading** | Thread-per-request | Thread-per-request (async capable) | Worker thread pool |
| **Customization hooks** | Valves, Listeners, Filters | Handlers, Filters | Handlers, Filters |
| **Stack trace "shape"** | `NioEndpoint` → `Http11Processor` → `CoyoteAdapter` → Valve chain | `ServerConnector` → `HttpChannel` → `Server.handle()` → Handler chain | `Connectors` → `HttpServerExchange` → Handler chain |
| **Difference for your code?** | None — from `FilterChain.doFilter()` onwards, the stack is identical | Same | Same |

**Key takeaway:** Replace Tomcat with Jetty or Undertow and everything from the `ApplicationFilterChain` downward (Spring Security, Spring MVC, your code, Hibernate) stays exactly the same. Only the top "network entry" frames change.

---

## Quick Reference: How to Spot Each Layer in Any Stack Trace

| What you see | What layer you're in |
|---|---|
| `org.apache.tomcat.*`, `org.apache.coyote.*`, `org.apache.catalina.*` | Tomcat |
| `ApplicationFilterChain` | Servlet filter chain (boundary between filters) |
| `org.springframework.security.web.*` | Spring Security filters |
| `DispatcherServlet`, `HandlerAdapter`, `InvocableHandlerMethod` | Spring MVC |
| `$$SpringCGLIB$$`, `CglibAopProxy`, `ReflectiveMethodInvocation` | Spring AOP proxy |
| `TransactionInterceptor`, `TransactionAspectSupport` | @Transactional |
| `AuthorizationManagerBeforeMethodInterceptor` | @PreAuthorize |
| Your package name (`com.gbis.kyc.*`) | YOUR CODE |
| `$Proxy`, `JdkDynamicAopProxy`, `SimpleJpaRepository` | Spring Data JPA |
| `SharedEntityManagerCreator`, `ExtendedEntityManagerCreator` | Spring JPA EntityManager proxy |
| `org.hibernate.internal.SessionImpl` | Hibernate Session |
| `DefaultMergeEventListener`, `Cascade`, `CascadingActions` | Hibernate merge + cascade |
| `CollectionType`, `PersistentBag`, `TypeHelper` | Hibernate collection handling |
| `java.util.ImmutableCollections` | JDK immutable collections |


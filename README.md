# Contribution [#242]: [feat: Add JAX-RS exception mappers for REST API error handling]

**Contribution Number:** [1]  
**Student:** [Brandon Trieu]  
**Issue:** [[GitHub issue link](https://github.com/carlos-emr/carlos/issues/242)]  
**Status:** [Phase III] [In Progress]

---

## Why I Chose This Issue

I chose issue #242 "feat: Add JAX-RS exception mappers for REST API error handling," because I have experience with Java and my goal is to improve my backend development skills. I hope to learn skills such as how to cleanly handle server-side errors. The issue is labeled "good first issue" and has clear guidance, providing the perfect issue to gain valuable experience that meets my desires.

---

## Understanding the Issue

### Problem Description

The issue is that when exceptions are thrown, there are no instructions on how to handle them. This results in a default returning of HTTP 500 with an HTML error page and exposed stack traces regardless of the actual error. The clients receive the wrong HTTP status code and internal details are exposed which creates security risks.

### Expected Behavior

A correct implementation should:
* return correct HTTP status codes
* provide structured JSON error responses
* log full stack traces only server-side for troubleshooting
* never expose internal details (stack traces) to clients

### Current Behavior

Regardless of the error, the server returns:
  * HTTP 500 status code
  * HTML error page with "An error occurred" and a reference ID
  * no structured error code or details

### Affected Components

* AccessDeniedException.java
* applicationContextREST.xml

---

## Reproduction Process

### Environment Setup

* main challenge: setting up dev container
  * the included .devcontainer had step to install Playwright browser which made the installation infinitely stuck for over an hour with no progress or error
  * fix: comment out Playwright RUN blocks in the Dockerfile
  * container built successfully and smoothly
* had some trouble with accessing the application as it directed to http://localhost:8080/oscar, but it was actually http://localhost:8080/carlos

### Steps to Reproduce

1. Install necessary tools (VS Code, Dev Containers extension), clone the fork, and start the dev container (reopen project in container)
2. Log into application at http://localhost:8080/carlos with credentials (username: carlosdoc | password: carlos2026)
3. http://localhost:8080/carlos/ws/rs/rx/drugs/active/1
4. Observed Result: The server returns an HTML error page displaying "An error occurred," an HTTP 500 status code, and a reference ID. The server logs show the full IllegalArgumentException stack trace which confirms that the error stems from an unhandled exception without a JAX-RS mapper implemented.

### Reproduction Evidence

- **Commit showing reproduction:** [[Forked Github repo]](https://github.com/brndntru/carlos/tree/fix-issue-242)
- **Screenshots/logs:** <img width="1883" height="383" alt="image" src="https://github.com/user-attachments/assets/c879ee99-d6c1-4e09-a00d-57d7bee52aa5" /> <img width="920" height="268" alt="image" src="https://github.com/user-attachments/assets/e5a7d259-99b3-4025-88a3-4f5b9224e794" />
tail -100 /usr/local/tomcat/logs/catalina.out | grep -A 20 "AccessDenied\|Exception"
Caused by: java.lang.IllegalArgumentException: No enum constant io.github.carlos_emr.carlos.webserv.rest.to.model.RxStatus.ACTIVE
        at java.base/java.lang.Enum.valueOf(Enum.java:293)
        at io.github.carlos_emr.carlos.webserv.rest.to.model.RxStatus.valueOf(RxStatus.java:31)
        at io.github.carlos_emr.carlos.webserv.rest.RxWebService.drugs(RxWebService.java:185)
        at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
        at java.base/java.lang.reflect.Method.invoke(Method.java:580)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:359)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:158)
        at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:82)
        at io.github.carlos_emr.carlos.webserv.rest.util.WebServiceLoggingAdvice.logAccess(WebServiceLoggingAdvice.java:81)
        at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
        at java.base/java.lang.reflect.Method.invoke(Method.java:580)
        at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:648)
        at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:630)
        at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:70)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:96)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:719)
        at io.github.carlos_emr.carlos.webserv.rest.RxWebService$$SpringCGLIB$$0.drugs(<generated>)

- **My findings:**
    1. Confirmed error as it was triggered but stack trace isn't directly exposed to the client
    2. Wrong HTTP status code, wrong response content type, no structured error info, stack trace on server-side confirms unhandled exception

---

## Solution Approach

### Analysis

The root cause is that the CARLOS REST API doesn't have standardized error handling. There are no JAX-RS instructions for how to handle thrown exceptions, so the server defaults to return an HTML error page with HTTP 500 status code. 

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The problem is that there are unhandled exceptions causing an incorrect return of error information. The CARLOS REST API lacks standardized error handling, so when exceptions are thrown, there are no instructions for how to handle them, causing the server to default to returning an HTML error page with an HTTP 500 status code. This is an issue because the status code is incorrect and the response format is incorrect (HTML instead of JSON). Stack traces are also exposed to clients, creating security vulnerabilities. A fixed implementation should return the correct HTTP status code, provide structured JSON error responses, log full stack traces server-side only, and never expose internal details to clients.

**Match:** The applicationContextREST.xml file has a JAX-RS providers section where the mappers just need to be registered. The Jackson JSOn provider is also already configured. The pattern already exists in JunoEMR, a different project which shares the same OSCAR McMaster heritage as CARLOS. The exception mapper pattern is adapted from JunoEMR's implementation. 

**Plan:** [Step-by-step implementation plan]
1. Find AccessDeniedException
2. Create ErrorResponse.java
3. Create exception mapper classes
4. Register mappers in applicationContextREST.xml
5. Create test files

**Implement:** [[Forked Github repo branch]](https://github.com/brndntru/carlos/tree/fix-issue-242)

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]
* Follow pull request guidelines (target develop, reference related issues, include clear description of what changed and why, add tests for new functionality, keep PRs focused)
* Follow commit message formatting conventions
* Follow mandatory security practices (output encoding, parameterized queries only, authorization checks, file path validation, no PHI in logs)
* Follow code patterns (Strus2 Actions, Spring Integration, package namespace, copyright headers)
* Build passes with make install

**Evaluate:** 
I will test it with the automated testing and also manually test it.

Automated:
* 14 unit tests
* 8 integration tests
* All tests pass with make install --run-tests

Manual:
* Route to http://localhost:8080/carlos/ws/rs/rx/drugs/active/1 in browser
* Check before and after fix:
   * before: HTML response with HTTP 500 status code
   * after: JSON response with HTTP 400 status code
* Check container logs to confirm full stack trace still logged server-side only

---

## Testing Strategy

### Unit Tests
`src/test/java/io/github/carlos_emr/carlos/webserv/rest/exceptionMapping/` — JUnit 5 + AssertJ, 22 tests, all passing.

**ErrorResponseUnitTest**
- [x] `shouldSetTimestampOnConstruction` — `ErrorResponse.of(...)` populates an ISO-8601 UTC timestamp (`yyyy-MM-ddTHH:mm:ssZ`)
- [x] `shouldSerializeToJsonCorrectly` — serialized JSON contains `code`/`message`/`timestamp` in the order pinned by `@JsonPropertyOrder`
- [x] `shouldIncludeDetailsWhenProvided` — `withDetails(...)` populates `details` and it appears in the JSON
- [x] `shouldExcludeDetailsWhenNull` — null `details` is omitted from the serialized JSON

**AccessDeniedExceptionMapperUnitTest**
- [x] `shouldReturn403Status` — maps `AccessDeniedException` to HTTP 403
- [x] `shouldReturnAccessDeniedErrorCode` — body `code` is `ACCESS_DENIED`
- [x] `shouldIncludePermissionInDetails` — `details` carries `permission` + `action` but not the `subject` (demographicNo)
- [x] `shouldLogFullExceptionWithStackTrace` — logs at WARN with the exception attached (stack trace captured server-side)

**SecurityExceptionMapperUnitTest**
- [x] `shouldReturn403Status` — maps `SecurityException` to HTTP 403
- [x] `shouldReturnSecurityErrorCode` — body `code` is `SECURITY_ERROR`

**PatientDirectiveExceptionMapperUnitTest**
- [x] `shouldReturn403Status` — maps `PatientDirectiveException` to HTTP 403
- [x] `shouldReturnPatientDirectiveCode` — body `code` is `PATIENT_DIRECTIVE`
- [x] `shouldLogAtInfoLevel` — logged at INFO (expected policy outcome), not WARN/ERROR

**IllegalArgumentExceptionMapperUnitTest**
- [x] `shouldReturn400Status` — maps `IllegalArgumentException` to HTTP 400
- [x] `shouldReturnValidationErrorCode` — body `code` is `VALIDATION_ERROR`
- [x] `shouldIncludeExceptionMessage` — the exception message is preserved as client validation feedback

**ConversionExceptionMapperUnitTest**
- [x] `shouldReturn400Status` — maps `ConversionException` to HTTP 400
- [x] `shouldReturnConversionErrorCode` — body `code` is `CONVERSION_ERROR`

**GeneralExceptionMapperUnitTest**
- [x] `shouldReturn500Status` — maps any unhandled `Throwable` to HTTP 500
- [x] `shouldReturnInternalErrorCode` — body `code` is `INTERNAL_ERROR`
- [x] `shouldNotExposeExceptionMessage` — body carries a generic message, not the exception text
- [x] `shouldLogFullStackTrace` — logs at ERROR with the full exception attached

### Integration Tests
'ExceptionMapperIntegrationTest` — 8 tests, end-to-end over an in-process CXF JAX-RS server (local transport) with a production-mirroring JSON provider (Jackson+JAXB introspector pair)

- [x] `shouldReturn403ForAccessDeniedException` — stub endpoint throws `AccessDeniedException`; response is HTTP 403 with `code` `ACCESS_DENIED`
- [x] `shouldReturn403ForSecurityException` — throws `SecurityException`; HTTP 403 with `code` `SECURITY_ERROR`
- [x] `shouldReturn403ForPatientDirectiveException` — throws `PatientDirectiveException`; HTTP 403 with `code` `PATIENT_DIRECTIVE`
- [x] `shouldReturn400ForIllegalArgumentException` — throws `IllegalArgumentException`; HTTP 400 with `code` `VALIDATION_ERROR`, and null `details` is omitted from the body
- [x] `shouldReturn400ForConversionException` — throws `ConversionException`; HTTP 400 with `code` `CONVERSION_ERROR`
- [x] `shouldReturn500ForUnhandledException` — throws an unmapped exception; HTTP 500 with `code` `INTERNAL_ERROR`
- [x] `shouldReturnJsonContentType` — error responses are served as `application/json`
- [x] `shouldNotExposeStackTraceInResponse` — the 500 body contains no exception message, class name, or stack-trace markers

### Manual Testing

Verified in the dev container (built + deployed, authenticated session as `carlosdoc`):

- **Bug reproduction & fix** — `GET /ws/rs/rx/drugs/active/1`
  - Before: `HTTP 500`, `text/html` error page (uncaught `IllegalArgumentException` from `RxStatus.valueOf("ACTIVE")`)
  - After: `HTTP 400`, `application/json` → `{"code":"VALIDATION_ERROR","message":"Unknown drug status: active","timestamp":"..."}` — structured JSON, no `details` field, no stack trace
- **Regression check** — `GET /ws/rs/rx/drugs/all/1`: `HTTP 200`, `application/json` with drug data; response structure unchanged vs. before the mapper/introspector changes (confirms the `spring_ws.xml` introspector fix did not alter existing REST output)

---

## Implementation Notes

### Week [1] Progress

- Phase I Complete
 - selected issue #242 - feat: Add JAX-RS exception mappers for REST API error handling
 - created Github repo, contribution README, forked project
 - challenges: not much except choosing an issue that fits my skills, background, and goals

### Week [2] Progress

- Phase II Complete
 - set up local environment with project and dev container
 - created working branch and successfully reproduced issue
 - developed solution plan
 - challenges: issues with container build (kept getting stuck at certain point in installation)

### Week [3] Progress

- Phase III started and in progress
 - reviewed contribution guidelines and continued to analyze and understand issue more thoroughly
 - feat: added ErrorResponse class
 - feat: added JAX-RS exception mappers
 - challenges: more issues with container build so had to rebuild and reconfigure local environment setup, changes to container weren't being seen in local project

### Week [4] Progress

- Phase III in progress
 - fix: registered JAX-RS exception mappers to return JSON erros
 - error codes should now return as the correct code with the correct response page (JSON) instead of HTML
 - challenges: not much, need to create tests to ensure fixes/features are functioning properly

### Week [5] Progress
- Phase III in progress
 - test: added unit tests for JAX-RS
 - fix: registered exception mappers on the /ws/rs JAX-RS server
 - manually tested at localhost directory - properly displays clean structured JSON response with HTTP 400 error code and timestamp 
 - challenges: issues with live verification (building/running tests) against dev container, still displaying error 500 code in HTML (exception mappers not catching fully)

### Week [6] Progress
- Phase III Complete
 - reconfigured unit tests and added 8 integration tests
 - challenges: unit tests were passing but integration tests were failing on server startup

### Code Changes

**Files modified:** [List]
- `webserv/rest/response/ErrorResponse.java` *(new)* — JSON error contract (`code`/`message`/`details`/`timestamp`)
- `webserv/rest/exceptionMapping/` *(new, 6 mappers)* — `AccessDeniedExceptionMapper`, `SecurityExceptionMapper`, `PatientDirectiveExceptionMapper`, `IllegalArgumentExceptionMapper`, `ConversionExceptionMapper`, `GeneralExceptionMapper`
- `webserv/rest/RxWebService.java` — rethrow `RxStatus.valueOf` failure with a clean, FQCN-free validation message
- `resources/applicationContextREST.xml` — register the 6 mappers on the `/services` server
- `resources/spring_ws.xml` — register the 6 mappers on the `/ws/rs/*` server; switch `jacksonObjectMapper` to a Jackson+JAXB introspector pair
- - `ErrorResponseUnitTest`, `AccessDeniedExceptionMapperUnitTest`, `SecurityExceptionMapperUnitTest`, `PatientDirectiveExceptionMapperUnitTest`, `IllegalArgumentExceptionMapperUnitTest`, `ConversionExceptionMapperUnitTest`, `GeneralExceptionMapperUnitTest` (22 unit tests), `ExceptionMapperIntegrationTest` (8 integration tests)

**Key commits:** [Links to important commits]
- `5f4e5be` feat: add ErrorResponse class for structured JSON error responses [Link](https://github.com/carlos-emr/carlos/commit/5f4e5be1afe6fe1cb2ca1c38c9c52ed021a44f0c)
- `5a9740f` feat: add JAX-RS exception mappers for REST API error handling [Link](https://github.com/carlos-emr/carlos/commit/5a9740f9320fce53c2281c4477caed6154b6ffa2)
- `2769920` fix: register JAX-RS exception mappers to return JSON errors [Link](https://github.com/carlos-emr/carlos/commit/2769920644667f7e74b5b0249518037736a81912)
- `b9180b6` test: add unit tests for JAX-RS [Link](https://github.com/carlos-emr/carlos/commit/b9180b6bbd1b964229941639f9b8ea91c847174e)
- `e676198` fix: register exception mappers on the /ws/rs JAX-RS server [Link](https://github.com/carlos-emr/carlos/commit/e67619849c07b38f07f147f83613414cac82903c)
- `cbce03a` fix: aligned REST exception mappers with issue #242 spec and add full test suite [Link](https://github.com/carlos-emr/carlos/commit/cbce03a7df9672950725168fd0b4c2436e598f3a)
- `27172460` fix: honor Jackson annotations on the /ws/rs JSON mapper [Link](https://github.com/carlos-emr/carlos/commit/27172460a2b6ab7308cb886e87f22024c2edb882)

**Approach decisions:** [Why you chose certain approaches]
- **Six mappers matching the issue spec exactly** An initial codebase-derived set (WebApplication/OperationNotSupported mappers) was replaced once the authoritative issue table was confirmed — added `PatientDirective`/`Conversion`, fixed `Security`'s code to `SECURITY_ERROR`
- **Registered on *both* JAX-RS servers** The endpoint that actually 500'd (`/ws/rs/*`) is served by `spring_ws.xml`, not `applicationContextREST.xml`. Registering only in the file the issue mentioned wasn't enough — verified live
- **Clean the message at the source, not in the mapper** `RxStatus.valueOf`'s native message embeds a fully-qualified class name; `IllegalArgumentExceptionMapper` faithfully echoes the message (per spec), and the response stack-trace sanitizer then treats the FQCN as a leak and replaces the body with HTML. Fixing `RxWebService` to throw a clean message keeps the mapper generic and yields a proper JSON 400
- **Fixed the Jackson introspector rather than enabling global `NON_NULL`** Root cause of `"details":null` was a JAXB-only introspector on the `/ws/rs` mapper that ignored `@JsonInclude`. Switching to a Jackson-primary / JAXB-secondary pair honors Jackson annotations while keeping `@Xml*` DTOs working — targeted, and consistent with the sibling server config
- **PHI discipline throughout** Full exceptions/stack traces logged server-side via `LogSafe`; client bodies carry generic messages; PHI-correlating identifiers (`AccessDeniedException` subject / `demographicNo`) never reach the response
- **Every mapper forces `application/json`** so error bodies can't fall through to the XML provider; integration tests run through an in-process CXF server (local transport) with a production-mirroring JSON provider

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]

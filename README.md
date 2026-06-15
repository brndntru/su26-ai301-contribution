# Contribution [#242]: [feat: Add JAX-RS exception mappers for REST API error handling]

**Contribution Number:** [1]  
**Student:** [Brandon Trieu]  
**Issue:** [[GitHub issue link](https://github.com/carlos-emr/carlos/issues/242)]  
**Status:** [Phase I] [Complete]

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

- **Commit showing reproduction:** [[Link to commit in your fork]](https://github.com/brndntru/carlos/tree/fix-issue-242)
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

**Implement:** [[Link to your branch/commits as you work]](https://github.com/brndntru/carlos/tree/fix-issue-242)

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

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

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

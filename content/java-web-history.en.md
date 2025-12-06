---
title: 'History of Java Web Technology: From Java EE to Spring Boot'
date: '2025-12-06T14:24:20+09:00'
math: false
---

> This post explains the [ywcheong/java-web-history](https://github.com/ywcheong/java-web-history) mini-project.


## Understanding Technical Books Published 10 Years Ago


Recently, I’ve been realizing the importance of reading a book more than ever. Of course, arguments like "you should look at the internet rather than books" or "book content becomes outdated quickly" have their merits. However, books provide not just the *how-to* of a technology, but the *context* of its birth. This deep context offers insights that are hard to gain simply by copy-pasting code or following tutorials.


If I were to pick a few books that have been most helpful to me so far, they would be the following. Please note that this is a subjective selection, as I am currently at the stage of starting my career as a backend developer.


- [Good Code, Bad Code](https://product.kyobobook.co.kr/detail/S000202521361)
- [Unit Testing](https://product.kyobobook.co.kr/detail/S000001805070)
- [Toby's Spring 3.1 Vol. 1](https://product.kyobobook.co.kr/detail/S000000935360)
- [Java ORM Standard JPA Programming](https://product.kyobobook.co.kr/detail/S000000935744)


The first two books cover general methodologies applicable regardless of language or tool. On the other hand, the latter two are much more closely aligned with practical work, and I was particularly able to learn the Best Practices of Spring. I felt there was a world of difference between simply learning a language to write "code that works" and writing "code that fits the philosophy of that language and framework." (From this perspective, [Effective Kotlin](https://product.kyobobook.co.kr/detail/S000216681102) is also highly recommended).


Unfortunately, however, both books were published over 10 years ago. *Toby's Spring* came out in 2012, and *Java ORM Standard JPA Programming* in 2015. The Java web development environment at that time was drastically different from today.


I have experience in the full cycle from planning to backend using PHP, and I’ve also built simple Java web applications using JSP and JDBC Connectors. However, if asked whether I had experienced a full-scale "Servlet-based Enterprise Web" environment, the answer was "No."


This lack of experience drastically increased the difficulty of reading these books. While reading *Toby's Spring* Vol. 2, I often hit a roadblock whenever old web concepts like Servlet, JSP, WAR, and WAS poured out. I spent too much energy trying to distinguish whether the technology mentioned in the book was still a valid core concept or a legacy that is now being forgotten.


For example, it was difficult to understand why a hierarchy exists in `ApplicationContext`, or why Application Scope and Singleton Scope beans exist separately. I had to read while searching for answers to questions like 'Why did they divide it like this?', 'Do we still use this in Spring Boot?', and 'If not, why did it become obsolete?' one by one.


One of the advantages of development is that if you're curious about something, you can experiment with it yourself (unlike architecture, where you can't easily build a building just because you're curious about its philosophy). I decided that experiencing the legacy web directly was the fastest way, rather than trying to understand concepts like WAR or WAS solely through text. Therefore, I embarked on a mini-project to implement the same application using technologies from three different eras with the help of an LLM.


### Project Goal


This project aims to implement a web application with a simple Hello endpoint and Task CRUD functionality in the following three ways, following the evolution of Java web technology.


1.  **Java EE (Servlet based)**
2.  **Spring (Legacy)**
3.  **Spring Boot (Modern)**


While the three projects have similarities, they also have many decisive differences. The process of comparing them is quite interesting in itself.


However, I should note that I do not intend to become an expert in legacy web technologies. Since I relied heavily on GPT due to time constraints, there may be differences from the patterns actually used in the field at that time. The purpose of this project lies strictly in **understanding modern Spring more deeply through the past.**


### Want to run the project?


The [ywcheong/java-web-history](https://github.com/ywcheong/java-web-history) repository contains folders for `javaee8`, `spring7`, and `springboot4`. You can reproduce the environment of each era by running the `demo.sh` script included in each folder. The script also includes a function to clean up downloaded files after execution, so please feel free to run it.


{{< callout type="warning" >}}
  The `demo.sh` script was written in an Ubuntu Bash environment. It is highly likely that it will not execute correctly in a Windows environment.
{{< /callout >}}



## Implementation 1: Java EE


[Java EE Demo](https://github.com/ywcheong/java-web-history/tree/main/javaee8)
- **Build Tool:** Maven
- **Server:** Tomcat 9 (Standalone WAS)
- **Packaging:** WAR (`web.xml`)
- **Core:** Java EE Servlet
- **Data:** JPA (`persistence.xml`)


Java EE (Java Platform, Enterprise Edition) is a platform for which Oracle formerly defined the specifications required for enterprise application development, especially web applications. The package name was `javax.*`, but it has now changed to `jakarta.*` as it was transferred to the Eclipse Foundation. Although many features defined here have been obsolete or replaced in modern times, it still contains core technologies that underpin the Java web ecosystem.


The most important of these is the **Servlet** specification. In the modern Spring Framework, the concept of the servlet is abstracted deep inside the framework, so developers rarely encounter it directly. In fact, the concept of a servlet itself is so huge that the process of properly splitting and abstracting it can be seen as the history of Spring. Therefore, examining the prototype of a servlet is a shortcut to understanding the operating principles of Spring.


### What is a Servlet?


Strictly defining a servlet gives us the following:


> **"A small Java component that implements the Java Servlet specification, has its lifecycle managed by a servlet container, operates primarily via multi-threading within a process to handle client requests, and generates responses."**


Let's break this down with an actual code example ([HelloServlet.java](https://github.com/ywcheong/java-web-history/blob/main/javaee8/src/main/java/ywcheong/javaee8/HelloServlet.java)).


1.  **Implementation of Java Servlet Specification**
    To become a servlet, a class must implement the `javax.servlet.Servlet` interface. This interface defines lifecycle methods such as `init()`, `service()`, and `destroy()`.


2.  **Lifecycle Managed by Servlet Container**
    Developers do not create objects using the `new` keyword. The servlet container (e.g., Tomcat) creates the servlet instance and controls the flow of `init() → service() → destroy()`. The container typically creates only one servlet instance and reuses it for efficiency. In other words, the servlet object practically operates like a singleton.


3.  **Operates via Multi-threading**
    Multiple threads access a single servlet instance simultaneously to call `service()`, `doGet()`, `doPost()`, etc. Because of this, if you save state in a servlet's instance variables, serious concurrency issues can occur. Therefore, servlets are designed to be Stateless by principle.


4.  **Request Handling and Response Generation**
    While various protocols are possible according to the spec, in practice, we mostly deal with HTTP-based `HttpServletRequest` and `HttpServletResponse`.


5.  **Component Unit**
    The important point is that a servlet is not a huge application itself, but a small **component** unit that performs a specific function within it.


### WAS, WAR, and the History of Deployment


In the past, the distinction between WAS (Web Application Server) and WS (Web Server) was strict.


*   **WAS (Tomcat, WebLogic, WebSphere, etc.):** A server that accepts requests, allocates threads from a thread pool, executes servlets, and provides Java EE spec features such as transaction and session management.
*   **Web Server (Apache, Nginx):** serves static content or acts as a proxy in front of the WAS.
    *   In modern times, Nginx and others handle infrastructure-level load balancing like a k8s Ingress Controller, and static content is handled by CDNs or S3, so the roles have changed significantly.


At the time, Tomcat was closer to a lightweight WAS focused on servlet container functions rather than a heavy WAS equipped with full enterprise features. Ironically, this lightness became the trigger for Tomcat to establish itself as the standard in the later Spring era.


Meanwhile, servlets were just units of function and were not suitable as units of deployment. This is because dependencies exist between servlets, and multiple servlets gather to form a single use case (e.g., membership registration and withdrawal). Therefore, a deployment unit was needed to bundle related servlets and resources into one, which is the **WAR (Web Application Archive)**.


Modern Spring Boot includes an embedded Tomcat and builds as an Executable JAR, but the old way was as follows:


1.  The developer builds the code to create a **WAR file**.
2.  This file includes a **Deployment Descriptor** called `web.xml`. This XML file is packed with configurations about "which servlet will handle which URL request and which security filters it will pass through." ([web.xml example](https://github.com/ywcheong/java-web-history/blob/main/javaee8/src/main/webapp/WEB-INF/web.xml))
3.  When this WAR file is delivered to the infrastructure/operations team, the operations team deploys it to the WAS (Tomcat, etc.) installed on the server.


You might wonder, "Why didn't they launch a separate server for each WAR?" At that time, hardware resources were precious, and virtualization/isolation technologies like Docker or Kubernetes were not mature. Therefore, the standard operating model was to launch one huge **WAS** (one JVM) on a physical server and deploy multiple **WARs (applications)** on top of it to share resources.


To summarize:


- Unit of Function: **Servlet**
- Unit of Deployment: **WAR** (Contains multiple Servlets)
- Unit of Operation: **WAS** (Contains multiple WARs)


## Implementation 2: Spring


[Spring Demo](https://github.com/ywcheong/java-web-history/tree/main/spring7)
- **Build Tool:** Gradle
- **Server:** Tomcat 11 (Bundled)
- **Packaging:** JAR
- **Core:** Spring Framework
- **Config:** XML (`applicationContext.xml`)
- **Data:** JPA


### Limitations of Java EE


The Java EE era model seemed to work well at first, but limits emerged over time. The most representative problem was that the maintenance cost for functional changes was too high. Looking at the code in the Java EE style ([TaskServlet.java](https://github.com/ywcheong/java-web-history/blob/main/javaee8/src/main/java/ywcheong/javaee8/task/TaskServlet.java)), you can see that the web layer (Servlet), business logic, and data persistence handling are all mixed together.


```java
private void createTask(HttpServletRequest req, EntityManager em) {
    String title = req.getParameter("title");
    Task task = new Task();
    task.setTitle(title);
    em.persist(task);
}
```


You might think this could be solved if the developer just writes code *carefully*, but maintaining clean code without a strict structure is very difficult in practice. Various frameworks appeared to solve this complexity in a formalized way, and among them, the Spring Framework came to lead the market.


### Spring Becomes the Owner, Not a Part


I won't go deeply into Spring's design philosophy, but the point to note here is how Spring integrated with the existing Java EE/WAS environment. Spring did not try to replace the entire WAS from the beginning. Instead, it chose a way to coexist on top of the existing servlet structure. Without changing the entire WAS to Spring's requirements, if you delegate to just one servlet for Spring, it created a structure where the Spring container (`ApplicationContext`) and beans operate within it.


However, as Spring's popularity grew, the situation reversed. If it was an environment using purely Spring, the reason to split servlets into small pieces disappeared. Handling URL routing and logic separation at the Spring MVC controller level was much more flexible and sophisticated. So, Spring set up **DispatcherServlet**, a single servlet just for itself, at the front.


Also, while the existing external WAS method was still mainstream, the practice of deploying multiple WARs to a huge WAS began to crumble little by little. With the improvement of hardware performance and the development of virtualization technology, the method of bundling a lightweight Tomcat inside the application and deploying it began to spread gradually.


{{< callout type="warning" >}}
  A common misconception is that Embedded Tomcat is a feature of the Spring Framework itself, but it is actually a feature provided by Spring Boot. When using the Spring Framework alone without a WAS, you typically add Tomcat directly to the classpath to use it together.
{{< /callout >}}


The Main entry point of a Spring Framework application reflecting all these changes is as follows. You can see the full code in [Main.java](https://github.com/ywcheong/java-web-history/blob/main/spring7/src/main/java/ywcheong/spring7/Main.java). While this demo uses a bundled Tomcat approach—differing from the external WAS method that was mainstream in production at the time—I configured it this way intentionally to demonstrate the transitional appearance.


```java
package ywcheong.spring7;


import org.apache.catalina.Context;
import org.apache.catalina.startup.Tomcat;
import org.springframework.web.context.support.XmlWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;


import java.io.File;


public class Main {
    public static void main(String[] args) throws Exception {
        // 1. Tomcat Configuration - Excluded if external WAS is present
        String port = System.getProperty("server.port", "8080");
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(Integer.parseInt(port));
        tomcat.getConnector();


        // 2. Context Creation (Using temp folder for doc root)
        String docBase = new File(".").getAbsolutePath();
        Context context = tomcat.addContext("", docBase);


        // 3. Load Spring XML Context
        XmlWebApplicationContext springContext = new XmlWebApplicationContext();
        springContext.setConfigLocation("classpath:applicationContext.xml");


        // 4. Register DispatcherServlet
        DispatcherServlet dispatcherServlet = new DispatcherServlet(springContext);
        Tomcat.addServlet(context, "dispatcher", dispatcherServlet);
        context.addServletMappingDecoded("/", "dispatcher");


        // 5. Start Server
        System.out.println("Starting Embedded Tomcat 11 with Spring 7...");
        tomcat.start();
        tomcat.getServer().await();
    }
}
```


To summarize:


- Unit of Function: ~~Servlet~~ **Bean**
- Unit of Deployment: **WAR** (Single `DispatcherServlet`)
- Unit of Operation: **WAS** (Deployed to external WAS or embedded as library)


## Implementation 3: Spring Boot


[Spring Boot Demo](https://github.com/ywcheong/java-web-history/tree/main/springboot4)
- **Build Tool:** Gradle
- **Server:** Tomcat 11 (Embedded)
- **Packaging:** JAR
- **Core:** Spring Boot Framework
- **Config:** Annotation
- **Data:** JPA


### Inconveniences of Spring


The Spring Framework was excellent, but it had the downside that there were many complex configurations developers had to handle personally. Spring Boot was designed to resolve these hassles so that developers could focus only on business logic.


The biggest change is the automation of configuration. In the existing Spring Framework, implementing even simple functions required writing massive XML configurations or Java Config. For example, as seen in [applicationContext.xml](https://github.com/ywcheong/java-web-history/blob/main/spring7/src/main/resources/applicationContext.xml), using JPA required manually registering an `EntityManagerFactory` bean.


```xml
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="packagesToScan" value="ywcheong.spring7.task"/>
    <property name="jpaVendorAdapter">
        <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>
    </property>
    <property name="jpaProperties">
        <props>
            <prop key="hibernate.hbm2ddl.auto">update</prop>
            <prop key="hibernate.dialect">org.hibernate.dialect.H2Dialect</prop>
            <prop key="hibernate.show_sql">true</prop>
        </props>
    </property>
</bean>
```


In fact, in most projects, the JPA implementation and DataSource are often singletons. That is, even though it is an area where auto-configuration is fully possible, developers had to suffer through repetitive configuration tasks. Spring Boot actively utilizes conditional bean registration features like `@ConditionalOnMissingBean` to configure applications with reasonable defaults even if the developer does not configure them separately.


Also, the pain of dependency management has been drastically reduced. In the past, we had to match versions of Hibernate, JUnit, and Logging libraries one by one and resolve conflicts, but now we just need to add a Starter series like `spring-boot-starter-web`. All dependencies required for web development, such as Tomcat, Spring MVC, Jackson, and Logback, are automatically configured with compatible versions.


Finally, Spring Boot officially supports embedded WAS at the framework level. Since `TomcatServletWebServerFactory` automatically creates a Tomcat instance, the boundaries of WAS, WAR, and Servlet have now all been absorbed and hidden under Spring Boot. However, we must remember that at its foundation, the philosophy remains unchanged: launching a WAS and using a single servlet (DispatcherServlet) as an entry point to drag all web requests into the Spring world.


{{< callout type="warning" >}}
  Spring WebFlux is based on Netty, not Tomcat, and basically does not follow the servlet model.
{{< /callout >}}


To summarize:


- Unit of Function: ~~Servlet~~ **Bean**
- Unit of Deployment: **JAR** (Single `DispatcherServlet`)
- Unit of Operation: **Embedded WAS** (Single `TomcatServletWebServerFactory`)


In summary, the history from Java EE to Spring Boot is the history of the structure changing from **WAS : WAR : Servlet = 1 : N : NM** to a **1 : 1 : 1 structure**, and the history of elements like **DispatcherServlet and Tomcat Embedded WAS being hidden under the Spring framework** in the process of simplification.


## So, Why Is This Important?


"So, what?" is actually a valid point. In fact, in a modern Spring Boot environment, there is absolutely no problem developing without knowing the lifecycle of a servlet or how to configure `web.xml`. Indeed, I also learned development by diving in headfirst and writing `@RestController` directly, rather than stepping through textbook concepts like WAS or WAR.


Nevertheless, why did I bother to dig into the codebase of the past?


First, the process itself is enjoyable. Exploring what actually happens behind the abstracted layers is like a fun game. Just as descending into system calls, kernels, and hardware I/O to understand a single `printf` function is interesting, understanding the dynamics of servlets and containers hidden behind the magic-like auto-configuration of Spring Boot was a very fun experience. (I have things to say about Spring's 'magic' too, but I will omit them here).


Second, you realize the value of abstraction. The fact that we can develop without knowing this complex history paradoxically proves how excellent an abstraction Spring provides. Just as you can use `printf` without knowing the CPU's instruction cycle, being able to launch a stable web server without knowing the complexity of servlet containers is a huge benefit provided by the framework. After experiencing the complexity of the past once again, I feel anew how precious the convenience we currently enjoy is.


Finally, the depth of problem-solving changes. When you understand the principles, logical deduction becomes possible rather than vague guessing. If you know the backside of the abstraction, you can more accurately grasp what side effects might occur when settings are changed, or where the root lies when a bug with an unknown cause occurs. Instead of stopping at "it broke when I changed the compiler," you can suspect differences in how the compiler generates bytecode.


I won't have to write `web.xml` in actual work right now, but I (regrettably) prefer the process of exploring *why* something exists, even if it takes a detour, rather than efficient learning where you pick only what is useful. Leaving utility aside, this project was a sufficiently enjoyable process in itself.
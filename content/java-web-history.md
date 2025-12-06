---
title: 'Java 웹 기술의 변화 히스토리: Java EE부터 Spring Boot까지'
date: '2025-12-06T14:24:20+09:00'
math: false
---

> 이 포스트는 [ywcheong/java-web-history](https://github.com/ywcheong/java-web-history) 미니 프로젝트를 설명하는 글입니다.

## 10년 전에 출판된 기술서적 이해하기

최근 그 어느 때보다도 독서의 중요성을 실감하고 있습니다. 물론 "책보다 인터넷을 봐야 한다"거나 "책은 내용이 쉽게 낡는다"라는 주장도 근거가 있습니다. 하지만 책은 단순히 기술을 사용하는 방법뿐만 아니라, 그 기술이 탄생하게 된 맥락을 알려줍니다. 이러한 깊이 있는 맥락은 단순히 코드를 복사해 붙여넣거나 튜토리얼을 따라 하는 것만으로는 얻기 힘든 통찰을 제공합니다.

지금까지 읽었던 책 중에서 저에게 가장 큰 도움이 되었던 책을 몇 권 꼽자면 다음과 같습니다. 저는 현재 백엔드 개발자로 커리어를 시작하는 단계이므로, 주관적인 선정임을 감안해 주시기 바랍니다.

- [내 코드가 그렇게 이상한가요?](https://product.kyobobook.co.kr/detail/S000202521361)
- [단위 테스트](https://product.kyobobook.co.kr/detail/S000001805070)
- [토비의 스프링 3.1 1권](https://product.kyobobook.co.kr/detail/S000000935360)
- [자바 ORM 표준 JPA 프로그래밍](https://product.kyobobook.co.kr/detail/S000000935744)

앞의 두 권은 언어와 도구를 막론하고 적용할 수 있는 범용적인 방법론을 다룹니다. 반면, 뒤의 두 권은 실무와 훨씬 밀접하게 맞닿아 있었고, 특히 Spring의 Best Practice를 배울 수 있었습니다. 언어를 배워서 단순히 "동작하는 코드"를 짜는 것과, "해당 언어와 프레임워크의 철학에 맞는 코드"를 짜는 것은 천지 차이라는 것을 느꼈기 때문입니다. (이 관점에서는 [이펙티브 코틀린](https://product.kyobobook.co.kr/detail/S000216681102)도 추천할 만합니다)

하지만 안타깝게도 두 책 모두 출판된 지 10년이 넘었습니다. 『토비의 스프링』은 2012년, 『자바 ORM 표준 JPA 프로그래밍』은 2015년에 나왔습니다. 당시의 자바 웹 개발 환경은 지금과는 판이했습니다.

저 역시 PHP로 기획부터 백엔드까지 전체 사이클을 경험해 보았고, JSP와 JDBC Connector를 이용한 간단한 Java 웹 애플리케이션도 만들어 본 경험이 있습니다. 하지만 본격적인 '서블릿 기반의 엔터프라이즈 웹' 환경을 경험해 보았냐고 묻는다면, 답은 "아니요"였습니다.

이러한 경험의 부재는 책을 읽는 난이도를 급격히 높였습니다. 현재 『토비의 스프링』 2권을 읽고 있는데, 서블릿, JSP, WAR, WAS 등 오래된 웹 개념이 쏟아져 나올 때마다 턱턱 막히곤 합니다. 책에 나오는 기술이 여전히 유효한 핵심 개념인지, 아니면 이제는 잊혀져 가는 레거시인지를 구분하는 데 너무 많은 에너지가 소모되었습니다.

예를 들어, `ApplicationContext`에 계층 구조(Hierarchy)가 존재한다거나, 애플리케이션 스코프와 싱글톤 스코프의 빈이 왜 별도로 존재하는지 이해하는 것이 어려웠습니다. '왜 굳이 이렇게 나눴지?', '지금의 Spring Boot에서도 이걸 쓰나?', '안 쓴다면 왜 도태되었지?' 같은 의문을 하나하나 검색해 가며 읽어야 했습니다.

개발의 장점 중 하나는 궁금한 점이 있다면 직접 실험해 볼 수 있다는 것입니다(건축 철학이 궁금하다고 건물을 짓기는 어렵겠지요). 책만 보고 WAR나 WAS의 개념을 머리로만 이해하려 하기보다, 직접 레거시 웹을 체험해 보는 것이 가장 빠른 길이라 판단했습니다. 그래서 LLM의 도움을 받아 동일한 애플리케이션을 3가지 다른 시대의 기술로 구현해 보는 미니 프로젝트를 진행하게 되었습니다.

### 프로젝트 목표

이 프로젝트는 간단한 Hello 엔드포인트와 Task CRUD 기능을 가진 웹 애플리케이션을 자바 웹 기술의 발전 흐름에 따라 다음 3가지 방식으로 구현하는 것을 목표로 합니다.

1.  **Java EE (Servlet 기반)**
2.  **Spring (Legacy)**
3.  **Spring Boot (Modern)**

세 프로젝트는 공통점도 있지만, 결정적인 차이점들도 많습니다. 이들을 비교해 보는 과정 자체가 꽤 흥미롭습니다.

단, 한 가지 일러둘 점은 제가 레거시 웹 기술의 전문가가 되고자 하는 것은 아니라는 점입니다. 시간 관계상 GPT의 도움을 많이 받았기 때문에, 실제 당시 현업에서 쓰이던 패턴과는 차이가 있을 수 있습니다. 이 프로젝트의 목적은 어디까지나 **과거를 통해 현대의 Spring을 더 깊이 이해하는 것**에 있습니다.

### 프로젝트를 실행해 보고 싶다면?

[ywcheong/java-web-history](https://github.com/ywcheong/java-web-history) 리포지토리에는 `javaee8`, `spring7`, `springboot4` 폴더가 준비되어 있습니다. 각 폴더에 포함된 `demo.sh` 스크립트를 실행하면 각 시대의 환경을 재현해 볼 수 있습니다. 스크립트에는 실행 종료 후 다운로드된 파일들을 정리하는 기능도 포함되어 있으니 부담 없이 실행해 보시기 바랍니다.

{{< callout type="warning" >}}
  `demo.sh` 스크립트는 Ubuntu Bash 환경에서 작성했습니다. Windows 환경에서는 올바른 실행이 안 될 가능성이 높습니다.
{{< /callout >}}


## 제1구현: Java EE

[Java EE 데모](https://github.com/ywcheong/java-web-history/tree/main/javaee8)
- **Build Tool:** Maven
- **Server:** Tomcat 9 (Standalone WAS)
- **Packaging:** WAR (`web.xml`)
- **Core:** Java EE Servlet
- **Data:** JPA (`persistence.xml`)

Java EE(Java Platform, Enterprise Edition)는 과거 오라클이 엔터프라이즈 애플리케이션, 특히 웹 애플리케이션 개발에 필요한 스펙을 정의하여 제공하던 플랫폼입니다. 패키지명이 `javax.*`였으나, 현재는 이클립스 재단으로 이관되면서 `jakarta.*`로 변경되었습니다. 여기에 정의된 많은 기능이 현대에는 도태되거나 대체되었지만, 여전히 자바 웹 생태계를 지탱하는 핵심 기술들이 포함되어 있습니다.

그중 가장 중요한 것이 바로 **Servlet(서블릿)** 스펙입니다. 현대의 Spring Framework에서는 서블릿의 개념이 프레임워크 내부로 깊숙이 추상화되어 있어, 개발자가 직접 마주할 일은 드뭅니다. 사실 서블릿이라는 개념 자체가 너무 거대해서, 이를 적절히 분할하고 추상화해 온 과정이 곧 Spring의 역사라고도 볼 수 있습니다. 따라서 서블릿의 원형을 살펴보는 것이 Spring의 동작 원리를 이해하는 지름길입니다.

### 서블릿(Servlet)이란 무엇인가?

서블릿을 엄격하게 정의하면 다음과 같습니다.

> **"Java Servlet 스펙을 구현하고, 서블릿 컨테이너에 의해 생명주기가 관리되며, 프로세스 내에서 멀티스레드로 동작하여 클라이언트 요청을 처리하고 응답을 생성하는 작은 자바 컴포넌트"**

이 정의를 실제 코드 예시([HelloServlet.java](https://github.com/ywcheong/java-web-history/blob/main/javaee8/src/main/java/ywcheong/javaee8/HelloServlet.java))와 함께 풀어서 살펴보겠습니다.

1.  **Java Servlet 스펙 구현**
    서블릿이 되려면 `javax.servlet.Servlet` 인터페이스를 구현해야 합니다. 이 인터페이스는 `init()`, `service()`, `destroy()`와 같은 생명주기 메서드를 정의하고 있습니다.

2.  **서블릿 컨테이너에 의한 생명주기 관리**
    개발자가 `new` 키워드로 객체를 생성하지 않습니다. 서블릿 컨테이너(예: Tomcat)가 서블릿 인스턴스를 생성하고, `init() → service() → destroy()`의 흐름을 제어합니다. 컨테이너는 보통 효율을 위해 서블릿 인스턴스를 하나만 생성해 두고 재사용합니다. 즉, 서블릿 객체는 사실상 싱글톤처럼 동작합니다.

3.  **멀티스레드로 동작**
    하나의 서블릿 인스턴스에 대해 여러 스레드가 동시에 접근하여 `service()`, `doGet()`, `doPost()` 등을 호출합니다. 이 때문에 서블릿의 인스턴스 변수에 상태를 저장하면 심각한 동시성 문제가 발생할 수 있습니다. 따라서 서블릿은 기본적으로 무상태(Stateless)로 설계하는 것이 원칙입니다.

4.  **요청 처리와 응답 생성**
    스펙상으로는 다양한 프로토콜이 가능하지만, 실무에서는 거의 HTTP 기반의 `HttpServletRequest`와 `HttpServletResponse`를 다룹니다.

5.  **컴포넌트 단위**
    중요한 점은 서블릿은 거대한 애플리케이션 자체가 아니라, 그 내부에서 특정 기능을 수행하는 작은 **컴포넌트** 단위라는 것입니다.

### WAS와 WAR, 그리고 배포의 역사

과거에는 WAS(Web Application Server)와 WS(Web Server)의 구분이 엄격했습니다.

*   **WAS (Tomcat, WebLogic, WebSphere 등):** 요청을 받아 스레드 풀에서 스레드를 할당하고, 서블릿을 실행하며, 트랜잭션이나 세션 관리 등 Java EE 스펙의 기능을 제공하는 서버입니다.
*   **Web Server (Apache, Nginx):** 정적 콘텐츠를 서빙하거나 WAS 앞단에서 프록시 역할을 수행합니다.
    * 현대에는 Nginx 등이 k8s Ingress Controller처럼 인프라 레벨의 로드밸런싱을 담당하고, 정적 콘텐츠는 CDN이나 S3가 담당하면서 그 역할이 많이 변화했습니다.

당시 Tomcat은 엔터프라이즈급 기능을 모두 갖춘 무거운 WAS라기보다는, 서블릿 컨테이너 기능에 집중한 경량 WAS에 가까웠습니다. 아이러니하게도 이러한 가벼움이 훗날 Spring 시대에 Tomcat이 표준처럼 자리 잡는 계기가 됩니다.

한편, 서블릿은 기능의 단위일 뿐 배포의 단위로는 적절하지 않았습니다. 서블릿끼리 의존성이 존재하고, 여러 서블릿이 모여 하나의 유스케이스(예: 회원가입과 탈퇴)를 구성하기 때문입니다. 따라서 연관된 여러 서블릿과 리소스를 하나로 묶는 배포 단위가 필요했고, 이것이 바로 **WAR (Web Application Archive)** 입니다.

요즘의 Spring Boot는 내장 톰캣을 포함하여 실행 가능한 JAR(Executable JAR)로 빌드되지만, 예전 방식은 다음과 같았습니다.

1.  개발자가 코드를 빌드하여 **WAR 파일**을 생성합니다.
2.  이 파일에는 `web.xml`이라는 **배포 기술자(Deployment Descriptor)** 가 포함됩니다. 여기에는 "어떤 URL 요청이 들어오면 어떤 서블릿이 처리할지, 어떤 보안 필터를 거칠지"에 대한 설정이 XML로 빼곡히 적혀 있습니다. ([web.xml 예시](https://github.com/ywcheong/java-web-history/blob/main/javaee8/src/main/webapp/WEB-INF/web.xml))
3.  생성된 WAR 파일을 인프라/운영팀에 전달하면, 운영팀이 이를 서버에 설치된 WAS(Tomcat 등)에 배포합니다.

"왜 WAR마다 별도의 서버를 띄우지 않았을까?"라는 의문이 들 수 있습니다. 하지만 당시에는 하드웨어 자원이 귀했고, Docker나 Kubernetes 같은 가상화/격리 기술이 성숙하지 않았습니다. 따라서 물리 서버 한 대에 거대한 **WAS 하나**를 띄우고(JVM 하나), 그 위에 여러 개의 **WAR(애플리케이션)** 를 동시에 배포하여 자원을 공유하는 것이 표준적인 운영 모델이었습니다.

정리하자면 다음과 같습니다.

- 기능의 단위: **서블릿**
- 배포의 단위: **WAR** (여러 서블릿 포함)
- 운영의 단위: **WAS** (여러 WAR 포함)

## 제2구현: Spring

[Spring 데모](https://github.com/ywcheong/java-web-history/tree/main/spring7)
- **Build Tool:** Gradle
- **Server:** Tomcat 11 (Bundled)
- **Packaging:** JAR
- **Core:** Spring Framework
- **Config:** XML (`applicationContext.xml`)
- **Data:** JPA

### Java EE의 한계

Java EE 시대의 모델은 처음에는 잘 동작하는 듯했지만, 시간이 지나면서 한계가 드러났습니다. 가장 대표적인 문제는 기능 변경에 따른 유지보수 비용이 너무 높다는 것이었습니다. Java EE 방식의 코드([TaskServlet.java](https://github.com/ywcheong/java-web-history/blob/main/javaee8/src/main/java/ywcheong/javaee8/task/TaskServlet.java))를 보면, 웹 계층(서블릿)과 비즈니스 로직, 그리고 데이터 영속성 처리가 한데 뒤섞여 있는 것을 볼 수 있습니다.

```java
private void createTask(HttpServletRequest req, EntityManager em) {
    String title = req.getParameter("title");
    Task task = new Task();
    task.setTitle(title);
    em.persist(task);
}
```

물론 개발자가 신경 써서 *잘* 짜면 해결될 문제라고 생각할 수도 있지만, 실무에서 엄격한 구조 없이 깔끔한 코드를 유지하는 것은 매우 어려운 일입니다. 이러한 복잡성을 정형화된 방식으로 해결하고자 다양한 프레임워크들이 등장했고, 그중 Spring Framework가 시장을 주도하게 되었습니다.

### Spring은 부품에서 주인이 된다

Spring의 설계 철학을 깊게 다루진 않겠지만, 여기서 주목할 점은 Spring이 기존 Java EE/WAS 환경과 어떻게 통합되었는가입니다. Spring은 처음부터 WAS 전체를 대체하려 하지 않았습니다. 대신, 기존의 서블릿 구조 위에서 공존하는 방식을 택했습니다. WAS 전체를 Spring의 요구사항대로 바꿀 필요 없이, 서블릿 하나만 Spring에게 위임하면 그 안에서 Spring 컨테이너(`ApplicationContext`)와 빈들이 동작하는 구조를 만든 것입니다.

그러나 Spring의 인기가 높아지면서 상황은 역전되었습니다. 순수하게 Spring만 사용하는 환경이라면 굳이 서블릿을 잘게 쪼갤 이유가 사라졌습니다. Spring MVC의 컨트롤러 레벨에서 URL 라우팅과 로직 분리를 처리하는 것이 훨씬 유연하고 세련되었기 때문입니다. 그래서 Spring은 자신만을 위한 단 하나의 서블릿, `DispatcherServlet`을 앞단에 세우게 됩니다.

또한 여전히 기존 외장 WAS 방식이 주류였기는 하나, 거대한 WAS에 여러 WAR를 배포하던 관행도 조금씩 무너지기 시작했습니다. 하드웨어 성능의 향상과 가상화 기술의 발전으로, 가벼운 Tomcat을 애플리케이션 내부에 번들링하여 배포하는 방식이 점차 확산되었습니다.

{{< callout type="warning" >}}
  오해하기 쉬운 부분으로, 내장 Tomcat은 Spring Boot가 제공하는 기능이며 Spring 프레임워크 자체에는 없습니다. Spring 프레임워크만 WAS 없이 사용할 경우 보통 Tomcat을 클래스패스에 직접 추가해 함께 사용하게 됩니다.
{{< /callout >}}

이 모든 변화가 반영된 Spring 프레임워크 애플리케이션의 Main 진입점은 아래와 같습니다. [Main.java](https://github.com/ywcheong/java-web-history/blob/main/spring7/src/main/java/ywcheong/spring7/Main.java)에서 전체 코드를 확인할 수 있습니다. 이 데모에서는 Tomcat을 번들링한 방식을 사용했기에 당시 프로덕션 환경의 주류 방식인 외장WAS와는 차이가 있지만, 과도기적 모습을 보여드리기 위해 의도적으로 이렇게 구성했습니다.

```java
package ywcheong.spring7;

import org.apache.catalina.Context;
import org.apache.catalina.startup.Tomcat;
import org.springframework.web.context.support.XmlWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

import java.io.File;

public class Main {
    public static void main(String[] args) throws Exception {
        // 1. Tomcat 설정 - 외장WAS 있을 경우 제외
        String port = System.getProperty("server.port", "8080");
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(Integer.parseInt(port));
        tomcat.getConnector();

        // 2. Context 생성 (문서 루트는 임시 폴더 사용)
        String docBase = new File(".").getAbsolutePath();
        Context context = tomcat.addContext("", docBase);

        // 3. Spring XML 컨텍스트 로드
        XmlWebApplicationContext springContext = new XmlWebApplicationContext();
        springContext.setConfigLocation("classpath:applicationContext.xml");

        // 4. DispatcherServlet 등록
        DispatcherServlet dispatcherServlet = new DispatcherServlet(springContext);
        Tomcat.addServlet(context, "dispatcher", dispatcherServlet);
        context.addServletMappingDecoded("/", "dispatcher");

        // 5. 서버 시작
        System.out.println("Starting Embedded Tomcat 11 with Spring 7...");
        tomcat.start();
        tomcat.getServer().await();
    }
}
```

정리하자면 다음과 같습니다.

- 기능의 단위: ~~Servlet~~ **Bean**
- 배포의 단위: **WAR** (단일 `DispatcherServlet`)
- 운영의 단위: **WAS** (외장 WAS에 배포 or 라이브러리로 내장)

## 제3구현: Spring Boot

[Spring Boot 데모](https://github.com/ywcheong/java-web-history/tree/main/springboot4)
- **Build Tool:** Gradle
- **Server:** Tomcat 11 (Embedded)
- **Packaging:** JAR
- **Core:** Spring Boot Framework
- **Config:** Annotation
- **Data:** JPA

### Spring의 불편한 점

Spring Framework는 훌륭했지만, 개발자가 직접 처리해야 할 복잡한 설정이 많다는 단점이 있었습니다. Spring Boot는 이러한 번거로움을 해결하고, 개발자가 비즈니스 로직에만 집중할 수 있도록 설계되었습니다.

가장 큰 변화는 설정의 자동화입니다. 기존 Spring Framework에서는 간단한 기능을 구현하려 해도 방대한 XML 설정이나 Java Config를 작성해야 했습니다. 예를 들어 [applicationContext.xml](https://github.com/ywcheong/java-web-history/blob/main/spring7/src/main/resources/applicationContext.xml)에서 보듯, JPA를 사용하려면 `EntityManagerFactory` 빈을 수동으로 등록해야 했습니다.

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

사실 대부분의 프로젝트에서 JPA 구현체와 DataSource는 하나만 사용하는 경우가 많습니다. 즉, 충분히 자동 설정이 가능한 영역임에도 불구하고 개발자는 반복적인 설정 작업에 시달려야 했습니다. Spring Boot는 `@ConditionalOnMissingBean`과 같은 조건부 빈 등록 기능을 적극 활용하여, 개발자가 별도로 설정하지 않아도 합리적인 기본값으로 애플리케이션을 구성해 줍니다.

또한, 의존성 관리의 고통도 획기적으로 줄어들었습니다. 과거에는 Hibernate, JUnit, Logging 라이브러리들의 버전을 일일이 맞추고 충돌을 해결해야 했지만, 이제는 `spring-boot-starter-web` 등 Starter 시리즈 하나만 추가하면 됩니다. Tomcat, Spring MVC, Jackson, Logback 등 웹 개발에 필요한 모든 의존성이 호환되는 버전으로 자동 구성됩니다.

마지막으로, Spring Boot는 내장 WAS를 프레임워크 차원에서 공식 지원합니다. `TomcatServletWebServerFactory`가 자동으로 Tomcat 인스턴스를 생성해 주므로, 이제는 WAS, WAR, 그리고 서블릿의 경계가 Spring Boot 아래로 모두 흡수되어 숨겨졌습니다. 하지만 그 근본에는 WAS를 띄우고, 서블릿 하나(DispatcherServlet)를 진입점으로 삼아 모든 웹 요청을 스프링 세계로 끌고 들어온다는 철학이 변함없이 깔려 있다는 점을 기억해야 합니다.

{{< callout type="warning" >}}
  Spring WebFlux는 Tomcat이 아닌 Netty 기반이며, 기본적으로는 서블릿 모델을 따르지 않습니다.
{{< /callout >}}

정리하자면 다음과 같습니다.

- 기능의 단위: ~~Servlet~~ **Bean**
- 배포의 단위: **JAR** (단일 `DispatcherServlet`)
- 운영의 단위: **내장 WAS** (단일 `TomcatServletWebServerFactory`)

요약하자면, Java EE부터 Spring Boot까지의 역사는 **WAS : WAR : Servlet = 1 : N : NM 구조에서 1 : 1 : 1 구조로 변해간 역사**, 그리고 **단순화되는 과정에서 DispatcherServlet, Tomcat Embedded WAS와 같은 요소가 Spring 프레임워크 아래로 숨겨진 역사**입니다.

## 그래서, 이것이 왜 중요한가?

"그래서, 어쩌라고?"는 사실 올바른 지적입니다. 사실 현대적인 Spring Boot 환경에서는 서블릿의 생명주기나 `web.xml`의 설정법을 몰라도 개발하는 데 전혀 지장이 없습니다. 실제로 저 또한 WAS나 WAR 같은 교과서적인 개념부터 차근차근 밟아온 것이 아니라, 맨땅에 헤딩하듯 `@RestController`를 직접 작성하고 부딪혀보며 개발을 배웠으니까요.

그럼에도 불구하고, 굳이 시간을 들여 과거의 코드베이스를 파헤쳐 본 이유는 무엇일까요?

첫째로, 그 과정 자체가 즐겁습니다. 추상화된 계층 뒤에서 실제로 무슨 일이 일어나는지를 탐구하는 것은 재미있는 놀이와 같습니다. `printf` 함수 하나를 이해하기 위해 시스템 콜과 커널, 하드웨어 I/O까지 내려가 보는 과정이 흥미로운 것처럼, Spring Boot의 마법 같은 자동 설정 뒤에 숨겨진 서블릿과 컨테이너의 역학 관계를 이해하는 것은 매우 재미있는 경험이었습니다. (Spring의 '마법'에 대해서도 할 이야기가 있지만 여기서는 생략하겠습니다)

둘째로, 추상화의 가치를 깨닫게 됩니다. 우리가 이런 복잡한 역사를 몰라도 개발할 수 있다는 사실 자체가 Spring이 얼마나 훌륭한 추상화를 제공하는지를 역설적으로 증명합니다. CPU의 명령어 사이클을 몰라도 `printf`를 쓸 수 있는 것처럼, 서블릿 컨테이너의 복잡성을 몰라도 안정적인 웹 서버를 띄울 수 있다는 것은 프레임워크가 제공하는 거대한 혜택입니다. 과거의 복잡함을 다시 한 번 체험해 보고 나니, 현재 누리고 있는 편리함이 얼마나 소중한지 새삼 느끼게 되었습니다.

마지막으로, 문제 해결의 깊이가 달라집니다. 원리를 이해하면 막연한 추측이 아닌 논리적인 추론이 가능해집니다. 추상화의 뒷면을 알고 있다면, 설정을 변경했을 때 어떤 사이드 이펙트가 발생할지, 혹은 원인을 알 수 없는 버그가 발생했을 때 그 뿌리가 어디에 있는지 더 정확하게 파악할 수 있습니다. 단순히 "컴파일러를 바꿨더니 고장 났다"에서 멈추는 것이 아니라, 컴파일러가 바이트코드를 생성하는 방식의 차이를 의심해 볼 수 있습니다.

당장 실무에서 `web.xml`을 작성할 일은 없겠지만, 저는 쓸모 있는 것만 골라 배우는 효율적인 학습보다는, 조금 돌아가더라도 그것이 왜 존재하는지를 탐구하는 과정을 (아쉽게도) 좋아합니다. 효용성을 떠나, 이 프로젝트는 그 자체로 충분히 즐거운 과정이었습니다.
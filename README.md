Spring MVC 실습
===============

1. Spring MVC 개발 환경 설정
    * Maven Project에서 arhcetype을 webapp으로 생성
        * 생성된 프로젝트의 main폴더 아래에 java폴더와 resources폴더를 생성 일반 폴더로 인식될경우
        * java폴더를 마우스 오른쪽클릭 -> Mark directory as -> Sources Root를 클릭해주시고
        * resources폴더를 마우스 오른쪽클릭 -> Mark directory as -> Resources Root를 클릭해주시면 된다.
    * pom.xml에 관련 의존성 추가
        ~~~
        <spring.version>4.3.5.RELEASE</spring.version>
        ~~~
        ~~~
            <!-- Spring -->
            <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>${spring.version}</version>
            </dependency>
            <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-webmvc</artifactId>
              <version>${spring.version}</version>
            </dependency>
         
            <!-- Servlet JSP JSTL -->
            <dependency>
              <groupId>javax.servlet</groupId>
              <artifactId>javax.servlet-api</artifactId>
              <version>3.1.0</version>
              <scope>provided</scope>
            </dependency>
            <dependency>
              <groupId>javax.servlet.jsp</groupId>
              <artifactId>javax.servlet.jsp-api</artifactId>
              <version>2.3.1</version>
              <scope>provided</scope>
            </dependency>
            <dependency>
              <groupId>jstl</groupId>
              <artifactId>jstl</artifactId>
              <version>1.2</version>
            </dependency>
        ~~~
    * DispatcherServlet을 FrontController로 설정
        * java폴더 아래 설정파일들을 따로 분리하기 위해 config패키지 안에 WebMvcConfigurerAdapter를 상속받은 WebMvcContextConfiguration.class를 생성
            ~~~
            @Configuration
            @EnableWebMvc
            @ComponentScan(basePackages = {"kr.or.connect.mvcexam.controller"})
            public class WebMvcContextConfiguration extends WebMvcConfigurerAdapter {
            }
            ~~~
            * 이 설정 파일은 DispatcherServlet의 기본설정 외에 개발자가 다른 설정을 해주고 싶을때 상속 받아서 자바 클래스 파일을 만들고 web.xml에 init-param으로 설정해서 사용할수 있도록 할 수 있다.
        * WebMvcContextConfiguration.class에 몇가지 메서드를 추가
            ~~~
                @Override
                public void addResourceHandlers(ResourceHandlerRegistry registry) {
                    registry.addResourceHandler("/assets/**").addResourceLocations("classpath:/META-INF/resources/webjars/").setCachePeriod(31556926);
                    registry.addResourceHandler("/css/**").addResourceLocations("/css/").setCachePeriod(31556926);
                    registry.addResourceHandler("/img/**").addResourceLocations("/img/").setCachePeriod(31556926);
                    registry.addResourceHandler("/js/**").addResourceLocations("/js/").setCachePeriod(31556926);
                }
            ~~~
            * addResourceHandler메서드는 요청중에는 컨트롤러의 URL이 매핑되어있는 요청만 들어오는게 아니라 CSS파일, 이미지파일, js파일의 요청을 다 받아버리기 때문에 addResourceHandler의 매개변수 부분인 /css/** 이런 요청이 들어올때 /css경로에서 찾으라는 설정이다.
            ~~~
                // default servlet handler를 사용하게 합니다.
                @Override
                public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
                    configurer.enable();
                }
            ~~~
            * 그 다음 메소드는 configureDefaultServletHandling이란느 것을 오버라이드 하고 있다. 이 메소드는 파라미터로 전달 받은 DefaultServletHandlerConfigurer객체의 enable이라는 메소드를 호출함으로써 DefaultServletHandler를 사용하도록 해준다. 매핑 정보가 없는 URL 요청은 Spring의 DefaultServletHttpRequestHandler가 처리하도록 해준다. DefaultServletHttpRequestHandler는 WAS의 DefaultServlet에게 해당 일을 넘기게 된다. 그러면 WAS는 DefaultServlet이 static 한 자원을 읽어서 보여주게 해준다.
            ~~~
                @Override
                public void addViewControllers(final ViewControllerRegistry registry) {
                    System.out.println("addViewControllers가 호출됩니다. ");
                    registry.addViewController("/").setViewName("main");
                }
            ~~~
            * addViewControllers메소드는 특정 URL에 대한 처리를 컨트롤러 클래스를 작성하지 않고 매핑할 수 있도록 해주는데 요청 자체가 "/" 하고 들어오면 main이라고 하는 이름의 뷰로 보여주도록 하게되는데 이 main은 어디서 찾아내게 되냐면 view name은 ViewResolver라는 객체를 이용해서 찾게되는데 실제 main이라는 이름만 가지고서는 뷰 정보를 찾아낼 수가 없다.
            그래서 뷰 정보는 그 아래 getInternalResourceViewResolver라는 메소드에서 설정된 형태로 뷰를 사용하게 된다. InternalResourceViewResolver를 생성을 하고 있고 이 resolver한테 setPrefix, Suffix를 지정하고 있다. prefix는 앞에 suffix는 뒤에 붙여달라는 뜻이다. 결국 "/" 요청은 WEB-INF/views/main.jsp라는 파일을 찾아주게 된다.
            ~~~
                @Bean
                public InternalResourceViewResolver getInternalResourceViewResolver() {
                    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
                    resolver.setPrefix("/WEB-INF/views/");
                    resolver.setSuffix(".jsp");
                    return resolver;
                }
            ~~~
        * web.xml에 DispatcherServlet 설정
            ~~~
            <!DOCTYPE web-app PUBLIC
             "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
             "http://java.sun.com/dtd/web-app_2_3.dtd" >
            
            <web-app>
              <display-name>Archetype Created Web Application</display-name>
            
              <servlet>
                <servlet-name>mvc</servlet-name>
                <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
                <init-param>
                  <param-name>contextClass</param-name>
                  <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
                </init-param>
                <init-param>
                  <param-name>contextConfigLocation</param-name>
                  <param-value>kr.or.connect.mvcexam.config.WebMvcContextConfiguration</param-value>
                </init-param>
                <load-on-startup>1</load-on-startup>
              </servlet>
              <servlet-mapping>
                <servlet-name>mvc</servlet-name>
                <url-pattern>/</url-pattern>
              </servlet-mapping>
              
            </web-app>
            ~~~
        * main.jsp를 만들어 성공여부 확인
            * main.jsp 작성
            ~~~
            <%@ page contentType="text/html;charset=UTF-8" language="java" %>
            <html>
            <head>
                <title>Title</title>
            </head>
            <body>
            Hello Spring MVC~!!
            </body>
            </html>
            ~~~
            * Tomcat 설정
            * 실행
2. Controller 작성 실습01
    1. 웹 브라우저에서 http://localhost:8080/mvcexam/plusform이라고 요청을 보내면 서버는 웹 브라우저에게 2개의 값을 입력받을 수 있는 입력 창과 버튼이 있는 화면을 출력
        * plusForm.jsp 작성
            ~~~
            <%@ page contentType="text/html;charset=UTF-8" language="java" %>
            <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
                <title>Insert title here</title>
            </head>
            <body>
            <form method="post" action="plus">
                value1 : <input type="text" name="value1"><br>
                value2 : <input type="text" name="value2"><br>
                <input type="submit" value="확인">
            </form>
            </body>
            </html>
            ~~~
        * Controller 작성
            ~~~
            @Controller
            public class PlusController {
                @GetMapping(path = "/plusform")
                public String plusForm(){
                    return "plusForm";
                }
            }
            ~~~
    2. 웹 브라우저에 2개의 값을 입력하고 버튼을 클릭하면 http://localhost:8080/mvcexam/plus URL로 2개의 입력값이 POST방식으로 서버에게 전달한다. 서버는 2개의 값을 더한후
    , 그 결과 값을 JSP에게 request scope으로 전달하여 출력한다.
        * plusResult.jsp 작성
            ~~~
            <%@ page contentType="text/html;charset=UTF-8" language="java" %>
            <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
                <title>Insert title here</title>
            </head>
            <body>
            ${value1} 더하기 ${value2} (은/는) ${result} 입니다.
            </body>
            </html>
            ~~~
        * PlusController에 post요청 처리 추가
            ~~~
            @PostMapping(path = "/plus")
                public String plus(@RequestParam(name = "value1", required = true) int value1,
                                   @RequestParam(name = "value2", required = true) int value2,
                                   ModelMap modelMap){
                    int result = value1 + value2;
                    
                    modelMap.addAttribute("value1", value1);
                    modelMap.addAttribute("value2", value1);
                    modelMap.addAttribute("result", result);
                    
                    return "plusResult";
                }
            ~~~
3. Controller 작성 실습02
4. Controller 작성 실습03
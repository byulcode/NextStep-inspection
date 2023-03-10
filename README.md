#### 1. Tomcat 서버를 시작할 때 웹 애플리케이션이 초기화하는 과정을 설명하라.
* 서블릿 컨테이너는 웹 애플리케이션의 상태를 관리하는 ServletContext를 생성한다.
* ServletContext가 초기화되면 컨텍스트의 초기화 이벤트가 발생한다.
* ServletContextListener의 콜백 메소드, 즉 ContextLoaderListener의 contextInitialized() 메소드가 호출된다.
* jwp.sql 파일에서 SQL문을 실행해 데이터베이스 테이블을 초기화한다.
* 서블릿 컨테이너는 클라이언트의 최초 요청시 DispatcherServlet 인스턴스를 생성한다(생성자 호출). 이 코드에서는 @WebServlet의 loadOnStartup 속성을 이용해 클라이언트의 요청과 관계없이 서블릿 컨테이너(톰캣)가 시작됨과 동시에 서블릿 인스턴스를 생성한다.
* DispatcherServlet의 init() 메소드로 초기화를 진행한다.
* init()메소드 안에서 RequestMapping 객체를 생성한다.
* RequestMapping 인스턴스의 initMapping() 메소드를 호출한다. initMapping() 메소드에서는 요청 URL과 Controller 인스턴스를 맵핑시킨다.

#### 2. Tomcat 서버를 시작한 후 http://localhost:8080으로 접근시 호출 순서 및 흐름을 설명하라.
- http://localhost:8080로 접근하면 서블릿을 호출하기 전 우선 서블릿 필터가 호출된다.(`ResourceFilter` , `CharacterEncodingFilter` 의 doFilter() 실행)
- `ResourceFilter` 의 경우 해당 요청이 정적 자원(css, js, img)이 아니기 때문에 서블릿으로 요청을 위임한다.
- `urlPatterns = "/"` 로 매핑되어 있는 `DispatcherServlet`의 `service()` 메소드가 실행된다.
- service() 메소드는 요청받은 URL에 맞는 Controller 객체를 `RequestMapping`에서 가져온다. 요청 URL은 “/” 이며, 이와 연결된 `HomeController`가 반환된다.
- 요청에 대한 실질적인 작업은 `HomeController`의 `execute()` 메소드가 진행한다. execute() 메소드의 반환값은 `ModelAndView`이다.
- `service()` 메소드는 반환받은 `ModelAndView`의 모델 데이터를 뷰의 `render()` 메소드에 전달한다. 위 요청에서 뷰는 `JspView`다.
- `JspView`는 전달받은 모델 데이터를 “home.jsp”에 전달해 HTML을 생성하고, 응답함으로써 작업을 끝낸다.

#### 3. 질문하기 기능 구현
+ QuestionDao 클래스의 insert() 메소드를 활용하여 해결했다.

#### 4. 로그인한 사용자만 질문 가능하도록 구현
+ UserSessionUtils 클래스를 활용해 구현했다.

#### 5. 답변 목록 동적으로 출력하기
+ show.jsp의 답변 목록 부분을 JSTL로 구현했다.

#### 6. 한글 인코딩 문제 해결
+ CharacterEncodingFilter 상단에  @WebFilter("/*") 설정을 추가해준다.

#### 7. next.web.qna package의 ShowController는 멀티 쓰레드 상황에서 문제가 발생하는 이유에 대해 설명하라.
* Question과 Answer은 클라이언트마다 다른 상태값을 가진다. 그래서 매 요청마다 새로운 인스턴스를 생성해야 한다.
* ShowController는 Question과 List<Answer>을 필드값으로 설정하고 있어 힙 메모리에 저장되어 있는 Question과 List<Answer> 인스턴스를 가리키는 구조로 실행된다.
* 그래서 만약 첫 번째 스레드가 1번 문제를 요청했는데, 두번째 스레드가 execute() 메소드를 요청하면 ShowController가 2번 질문과 답변을 가리키게 된다. 그래서 첫 번째 글에 대한 응답이 1번이 아닌 2번 질문과 답변이 된다.
* Question과 List<Answer>을 execute() 메소드의 로컬변수로 변경해서 문제를 해결했다.
* 멀티스레드가 참조하는 ShowController 인스턴스는 동일하지만, 힙 메모리에 생성되어 있는 Question과 List<Answer>은 서로 다른 인스턴스를 참조하고 있다.

#### 8. 답변 추가시 countOfAnswer의 수를 하나 증가시켜라.
+ QuestionDao에 countOfAnswer의 수를 증가시키는 로직을 추가해서 해결했다.

#### 9. JSON 데이터로 조회할 수 있도록 API 추가
+ /api/qna/list URL로 접근했을 때 질문 목록을 JSON 데이터로 조회할 수 있도록 구현했다.

#### 10. 답변 삭제 기능 구현
+ scripts.js 에 ajax를 추가해서 구현했다.

#### 11. 질문 수정 기능 구현
+ 로그인한 사용자와 질문 작성자가 같을 경우에만 수정이 가능하도록 구현했다.

#### 12. 싱글톤 패턴 적용
+ JdbcTemplate과 DAO들의 생성자를 private으로 막아 싱글톤 패턴을 적용했다.

#### 13. 질문 삭제 기능 구현
+ 질문 삭제가 가능한 경우는 답변이 없는 경우, 질문자와 답변자가 모두 같은 경우다.
+ 서비스 객체를 이용해 컨트롤러의 중복을 제거하는 방식으로 구현했다.


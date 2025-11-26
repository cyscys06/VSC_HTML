타임리프 설명 {
    타임리프: 자바 기반의 템플릿 엔진
    html, xml, css, javascript 지원 가능
    기존엔 WAR 파일을 배포해서 서버를 실행했음
    -> 파일 크기가 크고 변경사항이 있을때마다 파일을 다시 빌드해서 재배포 해야 하는 단점
    -> 이를 jar 파일을 이용해 해결 가능
    jar
    독립적인 자바 어플리케이션(파일이름Application) 파일을 패키징 가능
    -> Application 파일의 main 메서드를 포함하고 있으므로 
    jvm(자바 가상 머신: 자바 파일을 가상에서 실행시킬 수 있음. 우리가 자바 코드 실행하면 나옴)으로 실행 가능
    -> 따라서 별도의 웹 컨테이너 혹은 서버가 필요 없음
    물론 WAR, jar 모두 장단점이 있지만 요즘 추세는 jar 파일 쓰는것

    타임리프는 jar로 실행 가능하고, JSP는 WAR로만 사용 가능해서 요새는 타임리프가 유행

    또한 WAS를 별도로 실행하지 않아도 오프라인에서 수정 가능(편함)
}



타임리프의 여러 가지 표현식 {
    • 간단한 표현:
        ◦ 변수 표현식: ${...}
        ◦ 선택 변수 표현식: *{...}
        ◦ 메시지 표현식: #{...}
        ◦ 링크 URL 표현식: @{...}
        ◦ 조각 표현식: ~{...}

    • 리터럴
        ◦ 텍스트: 'one text', 'Another one!',…
        ◦ 숫자: 0, 34, 3.0, 12.3,…
        ◦ 불린: true, false
        ◦ 널: null
        ◦ 리터럴 토큰: one, sometext, main,…

    • 문자 연산:
        ◦ 문자 합치기: +
        ◦ 리터럴 대체: |The name is ${name}|

    • 산술 연산:
        ◦ Binary operators: +, -, *, /, %
        ◦ Minus sign (unary operator): -

    • 불린 연산:
        ◦ Binary operators: and, or
        ◦ Boolean negation (unary operator): !, not

    • 비교와 동등:
        ◦ 비교: >, <, >=, <= (gt, lt, ge, le)
        ◦ 동등 연산: ==, != (eq, ne)

    • 조건 연산:
        ◦ If-then: (if) ? (then)
        ◦ If-then-else: (if) ? (then) : (else)
        ◦ Default: (value) ?: (defaultvalue)

    • 특별한 토큰:
        ◦ No-Operation: _
}



타임리프 텍스트 출력(th:text 사용) {
    model.addAttribute("data", "Hello Spring!");
    -> addAttribute(name, value): 특정 값을 갖고있는 value 객체를 name이라는 이름으로 생성
    -> View 파트에서 사용할 데이터가 key(name), value(value) 쌍 형태로 전달된다 보면 됨
     
    <span th:text="${data}">
    -> 태그는 상관없으며 span 말고도 다양한 태그 안에서 속성으로써 사용됨
    -> "${data}": data라는 이름으로 저장했던 객체가 갖고있는 값 출력(변수 표현식 사용)

    html 엔티티: html에 있는 예약어들을 의미
    -> 이런 예약어들을 리터럴로 출력하기 위한 방법이 이스케이프 문자(이미아는거)
    model.addAttribute("data", "Hello <b>Spring!</b>");
    -> value 안의 <>기호가 이스케이프로 간주돼서 리터럴로 출력됨
    
    -> 안에서 리터럴 없이 태그로써 사용하려면 th:inline 사용
    h1><span th:inline="none">[[...]] vs [(...)]</span> ([[]]는 text, [()]는 utext)
    -> inline 속성 값이 none이어서 [[]], [()] 해석 못함
    -> 안에 들은 예악어들을 이스케이프가 아니라 예약어의 기능으로써 사용 가능

    text utext 차이 {
        text: html 태그(<>)를 이스케이프 처리해서 출력
        -> 예약어를 사용한다 해도 자동으로 이스케이프 처리 해줘서 그냥 리터럴 문자열로 출력이 가능

        utext: html 태그를 이스케이프 처리하지 않고 출력
        -> <> 입력시 태그로써 사용되는 형태로 출력됨
    }
}



변수 표현식(${}) 사용하는 다양한 방법 {
    예시
    <li>${user.username} = <span th:text="${user.username}"></span></li> 단순히 객체의 프로퍼티 불러오기
    <li>${user['username']} = <span th:text="${user['username']}"></span></li> // 같은방법
    <li>${user.getUsername()} = <span th:text="${user.getUsername()}"></span></li>// 게터로 불러오기
    
    // 리스트일때
    <li>${users[0].username} = <span th:text="${users[0].username}"></span></li> 
    <li>${users[0]['username']} = <span th:text="${users[0]['username']}"></span></li>
    <li>${users[0].getUsername()} = <span th:text="${users[0].getUsername()}"></span></li>
    
    // 맵일때
    <li>${userMap['userA'].username} = <span th:text="${userMap['userA'].username}"></span></li>
    <li>${userMap['userA']['username']} = <span th:text="${userMap['userA']['username']}"></span></li>
    <li>${userMap['userA'].getUsername()} = <span th:text="${userMap['userA'].getUsername()}"></span></li>

    지역변수 선언(th:whth 사용) 
    <div th:with="first=${users[0]}"> // first라는 변수에 users[0]에 있는 객체가 할당됨
    <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p> // first 객체의 username 프로퍼티 값 출력
}



유틸리티 객체(대충 보고 넘어가기) {
    타임리프 유틸리티 객체들**
    `#message` : 메시지, 국제화 처리
    `#uris` : URI 이스케이프 지원
    `#dates` : `java.util.Date` 서식 지원
    `#calendars` : `java.util.Calendar` 서식 지원
    `#temporals` : 자바8 날짜 서식 지원
    `#numbers` : 숫자 서식 지원
    `#strings` : 문자 관련 편의 기능
    `#objects` : 객체 관련 기능 제공
    `#bools` : boolean 관련 기능 제공
    `#arrays` : 배열 관련 기능 제공
    `#lists` , `#sets` , `#maps` : 컬렉션 관련 기능 제공
}



URL 링크 생성(@{} 표현식, th:href 사용) {
    model.addAttribute("param1", "data1");
    model.addAttribute("param2", "data2");
    data1, data2이라는 값을 각각 가지는 객체 param1, param2 생성
    -> param1 = data1; 
    param2 = data2;

    <li><a th:href="@{/hello}">basic url</a></li> 
    // 일반적인 url 생성
    <li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li> 
    // 파라미터 있는 url 생성(변수표현식 ${} 이용, () 부분은 쿼리 파라미터로 간주)
    <li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
    // 경로변수 생성
    <li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>
    // 쿼리 파라미터, 경로변수 둘다 생성

    단순한 URL
    @{/hello} -> /hello

    쿼리 파라미터
    @{/hello(param1=${param1}, param2=${param2})}
    /hello ?param1=data1 & param2=data2
    -> () 에 있는 부분은 쿼리 파라미터로 처리(/ 없음)

    경로 변수
    @{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})} 앞부분은 변수 사용함을 의미, 뒷부분은 변수에 어떤 값이 있는지를 의미
    /hello/data1/data2 -> 변수의 값을 경로로써 사용(/ 있음)
    URL 경로상에 변수가 있으면 () 부분은 경로 변수로 처리된다.

    경로 변수 + 쿼리 파라미터
    @{/hello/{param1}(param1=${param1}, param2=${param2})}
    /hello/data1?param2=data2 
    경로 변수와 쿼리 파라미터를 함께 사용할 수 있다.

    => / 붙은 변수는 경로변수로, / 없는 변수는 쿼리 파라미터 변수로써 사용됨을 알 수 있다
}



리터럴 설명 {
    리터럴: 소스코드에서 고정된 값
    -> "hello"는 문자 리터럴, 10이나 20은 숫자 리터럴

    타임리프에서의 문자 리터럴은 작은 따옴표로 감싸야함
    <li>리터럴 대체 |hello ${data}| = <span th:text="|hello ${data}|"></span></li> 
    -> 리터럴을 대체 가능한 표현식
    <li>"hello world!" = <span th:text="hello world!"></span></li>
    -> th:text는 타임리프 식 
    -> "hello world"는 리터럴 문자열 아니므로 오류
}



각종 연산자 활용(양많아서 조금만) {
    
    <li>${data}?: '데이터가 없습니다.' = <span th:text="${data} ? : '데이터가 없습니다.'"></span></li>
    -> 삼항연산자: ${data}에 value 있으면 아무것도 안함. 없으면 데이터가 없습니다 출력
    
    <li>${nullData}?: _ = <span th:text="${nullData}?: _">데이터가 없습니다.</span></li>
    -> No-operation: 언더바 문자를 활용, 타임리프 식을 실행시키지 않는 기능을 함
}



속성 설정 및 추가 {
    속성 설정 {
        <input type="text" name="mock" th:name="userA" />
        -> th:name으로 기존의 name이라는 속성의 값을 userA로 변경(속성이 없으면 새로 추가함)
    }


    속성 추가 {
        속성 추가 3가지 방식
        `th:attrappend` : 속성 값들의 가장 뒤에 값을 추가한다.
        `th:attrprepend` : 속성 값들의 가장 앞에 값을 추가한다.
        `th:classappend` : class 속성에 자연스럽게 추가한다.

        예시
        <h1>속성 추가</h1>
        th:attrappend = <input type="text" class="text" th:attrappend="class='large'" /><br/> 
        // class라는 이름의 속성이 가진 출력값 바로 뒤에 연달아 값을 붙임(어떤 이름의 속성에 값을 붙일건지 명시할것)
        -> 기존: class = "text", 지금: class = "textlarge"
        th:attrprepend = <input type="text" class="text" th:attrprepend="class='large'" /><br/> 
        // class라는 이름의 속성이 가진 출력값 바로 앞에 값 붙임(어떤 이름의 속성에 값을 붙일건지 명시할것)
        -> 기존: class = "text", 지금: class = "largetext"
        th:classappend = <input type="text" class="text" th:classappend="large" /><br/> 
        // class 속성의 값에만 뒤에 추가(lass 속성의 값에 자동으로 추가됨)
        -> 기존: class = "text", 지금: class = "textlarge"
    }
}



반복(th:each 사용) {
    -> 리스트, 배열, 반복 가능한 객체(Iterable), 열거형 등의 자료구조에서 사용 가능

    반복 상태 유지 기능(자동생성, 생략가능) -> 각 지표의 값을 반복마다 볼 수 있음
    `index` : 0부터 시작하는 값
    `count` : 1부터 시작하는 값
    `size` : 전체 사이즈
    `even` , `odd` : 홀수, 짝수 여부( `boolean` )
    `first` , `last` :처음, 마지막 여부( `boolean` )
    `current` : 현재 객체

    예시
    List<User> list = new ArrayList<>();
    list.add(new User("userA", 10));
    list.add(new User("userB", 20));
    list.add(new User("userC", 30));
    model.addAttribute("users", list);
    // 리스트에 값 추가후 해당 리스트로 users 객체 만듦

    <tr th:each="user : ${users}">
    <td th:text="${user.username}">username</td>
    <td th:text="${user.age}">0</td>
    </tr>
    // 자바 반복문 for (user : users) {}로 보면됨
}



조건식(th:if, th:unless, th:switch, th:case 사용) {
    th:if -> 조건식 true면 이 코드가 있는 태그 실행(false면 태그 랜더링 안함)
    th:unless -> if 반대
    th:switch -> 값을 검사할 변수명 받음
    th:case -> 변수에 할당된 값이 case별 값과 일치할때 각 태그 랜더링
    
    예시
    <span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
    <span th:text="'미성년자'" th:unless="${user.age ge 20}"></span>

    <td th:switch="${user.age}">
        <span th:case="10">10살</span>
        <span th:case="20">20살</span>
        <span th:case="*">기타</span>
        // case = "*": default: 랑 같음
    </td>
}



주석 {
    html 주석 <!-- -->
    타임리프 주석문: <!--/*  */-->
}



블록(th:block 사용) {
    타임리프의 유일한 자체 태그

    예시
    <th:block th:each="user : ${users}"> // 중괄호처럼 사용 가능
        <div>
            사용자 이름1 <span th:text="${user.username}"></span>
            사용자 나이1 <span th:text="${user.age}"></span>
        </div>
        <div>
            요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
        </div>
    </th:block>
}



자바스크립트 인라인(th:inline = "javascript" 사용) {
    model.addAttribute("user", new User("userA", 10));

    <!-- 자바스크립트 인라인 사용 전 -->
    <script>
        var username = [[${user.username}]];
        var age = [[${user.age}]];
        
        //자바스크립트 내추럴 템플릿
        var username2 = /*[[${user.username}]]*/ "test username";
        
        //객체
        var user = [[${user}]];
    </script>
    
    <!-- 자바스크립트 인라인 사용 후 -->
    <script th:inline="javascript">
        var username = [[${user.username}]];
        var age = [[${user.age}]];
        
        //자바스크립트 내추럴 템플릿
        var username2 = /*[[${user.username}]]*/ "test username";
        
        //객체
        var user = [[${user}]];
    </script>

    user.username = "userA"
    user.age = 10 
    이게 맞음

    var username = [[${user.username}]]; 에 대해
    인라인 사용 전 var username = userA;
    인라인 사용 후 var username = "userA";

    인라인을 사용하기 전에는 따옴표 없이 변수에 할당됨 
    -> 오류(문자열 형태 아님) 
    -> [[]]는 text를 의미했으므로 문자열 형태로 받아야하지만(즉 따옴표가 들어가야 하지만) 문자열 형태가 아님

    자바스크립트 인라인 사용 후: [[]]를 인식, 따옴표 포함한 문자열을 제대로 변수에 할당

    자바 내추럴 템플릿 {
        자바 내추럴 템플릿: html 파일을 직접 열어도 동작하는 기능
        -> 서버 없이 html 파일만 랜더링해도 동작하고 html의 태그도 그대로 보여줌
        -> 서버 없이 자바스크립트 코드 실행해도 오류 발생 안함
        
        예시
        let user = /*[[${name}]]*/ "Guest";
        서버 없이 돌리면 "Guest"로, 서버랑 같이 돌리면 서버로부터 전달받은 값으로 출력

        var username2 = /*[[${user.username}]]*/ "test username";
        인라인 사용 전 var username2 = /*userA*/ "test username"; // [[]] 인식 못함
        인라인 사용 후 var username2 = "userA"; // [[]] 인식 함

        var user = [[${user}]];
        인라인 사용 전 var user = BasicController.User(username=userA, age=10);
        인라인 사용 후 var user = {"username":"userA","age":10}; // 객체의 정보를 담은 json파일도 자동으로 만들어줌
    }
    

    each 지원 {
        <script th:inline="javascript">
            [# th:each="user, stat : ${users}"]
            var user[[${stat.count}]] = [[${user}]];
            [/]
            // user[[${stat.count}]]: users 리스트 안에 들어있는 user 객체들의 인덱스 번호
            // [[${user}]]: user 객체의 정보(json 형태)
        </script>

        실행결과
        <script>
            var user1 = {"username":"userA","age":10};
            var user2 = {"username":"userB","age":20};
            var user3 = {"username":"userC","age":30};
            // json 형태로 알아서 출력해주는 모습
        </script>
    }
}



템플릿 조각(th:fragment 사용) {
    예시부터 보기(pdf 41페이지부터 보기)

    정적 부분의 템플릿 조각 가져오기 {
        template/fragment/footer :: copy : template/fragment/footer.html 템플릿에 있는
        th:fragment = "copy" 라는 코드를 가진 태그(즉, copy라는 값을 가진 th:fragment속성을 가지고 있는 태그) 
        부분을 템플릿 조각으로써 가져와서 사용한다(즉, 해당 태그의 innerHTML을 불러온다)는 의미이다.
    }
    

    동적 부분(파라미터)의 템플릿 조각 가져오기 {
        <div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터2')}"></div>
        
        <h1>파라미터 사용</h1>
        <footer>
            <p>파라미터 자리 입니다.</p>
            <p>데이터1</p> // param1
            <p>데이터2</p> // param2
        </footer>
        // param1 = '데이터1', param2 = '데이터2'

        // footer.html 의 copyParam 부분
        <footer th:fragment="copyParam (param1, param2)"> // 데이터1, 데이터2 라는 값 각각 가져옴
            <p>파라미터 자리 입니다.</p>
            <p th:text="${param1}"></p> // 데이터1
            <p th:text="${param2}"></p> // 데이터2
        </footer>
    }
}



템플릿 레이아웃 {
    템플릿 조각에서 개념 확장
    -> 코드 조각을 레이아웃에 넘겨서 사용

    base.html
    <head th:fragment="common_header(title,links)">
    // title, links 2개의 파라미터 템플릿 조각 받음
        <title th:replace="${title}">레이아웃 타이틀</title>
        // title 파라미터로 받은 코드로 대체(replace)
        
        <!-- 공통 -->
        <link rel="stylesheet" type="text/css" media="all" th:href="@{/css/awesomeapp.css}">
        <link rel="shortcut icon" th:href="@{/images/favicon.ico}">
        <script type="text/javascript" th:src="@{/sh/scripts/codebase.js}"></script>
        
        <!-- 추가 -->
        <th:block th:replace="${links}" />
        // 실행결과에 새로 추가될 부분: links 파라미터로 받은 코드로 대체(replace)
    </head>

    layoutMain.html
    <head th:replace="template/layout/base :: common_header(~{::title},~{::link})"> 
    // 현재 페이지에 있는 title, link 태그들을 전부 base.html에 파라미터로 전달
        <title>메인 타이틀</title>
        
        <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
        <link rel="stylesheet" th:href="@{/themes/smoothness/jquery-ui.css}">
        </head>
        <body>
        메인 컨텐츠
    </body>
    

    실행결과
    <!DOCTYPE html>
    <html>
        <head>
            <title>메인 타이틀</title>
            
            <!-- 공통 --> // html의 주석이므로 안사라짐
            <link rel="stylesheet" type="text/css" media="all" href="/css/awesomeapp.css">
            <link rel="shortcut icon" href="/images/favicon.ico">
            <script type="text/javascript" src="/sh/scripts/codebase.js"></script>
            
            <!-- 추가 -->
            <link rel="stylesheet" href="/css/bootstrap.min.css">
            <link rel="stylesheet" href="/themes/smoothness/jquery-ui.css">
        </head>
        <body>
            메인 컨텐츠
        </body>
    </html>


    html 파일을 통째로 파라미터로 전달 {
        layoutFile.html
        <html th:fragment="layout (title, content)" xmlns:th="http://www.thymeleaf.org">
        // 레이아웃 형태로 title, content 파라미터 받음
        // -> layoutFile.html의 html 태그 안에 있는 모든 title을 layoutExtendMain.html의 html 태그의 layout 안 파라미터 값으로 대체

        <div th:replace="${content}">
            <p>레이아웃 컨텐츠</p>
        </div>
        // -> replace의 변수인 content에 할당된 값을 layoutExtendMain.html로부터 전달받은 section태그 내용으로 대체
        
        layoutExtendMain.html
        <html th:replace = "~{template/layoutExtend/layoutFile :: layout(~{::title}, ~{::section})}" xmlns:th = "http://www.thymeleaf.org">
        // 현재 페이지(layoutExtendMain.html)에 있는 모든 title, section 태그 파라미터들을 레이아웃 형태로 layoutFile.html에 전달

        실행결과
        <html>
            <head>
                <title>메인 페이지 타이틀</title>
            </head>
            <body>
                <h1>레이아웃 H1</h1>
                <section>
                <p>메인 페이지 컨텐츠</p>
                <div>메인 페이지 포함 내용</div>
                </section>
                <footer>
                레이아웃 푸터
                </footer>
            </body>
        </html>
    }
}
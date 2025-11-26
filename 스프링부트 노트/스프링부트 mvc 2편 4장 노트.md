상품 저장 과정 {
    정상 과정
    0. 사용자가 입력을 하고싶다는 요청을 컨트롤러에 전송
    -> 컨트롤러는 입력 폼에 해당되는 뷰를 사용자에게 전송
    1. 사용자가 입력 폼에 정보를 입력 후 컨트롤러에 전송
    2. 컨트롤러 내부에서 입력값 검증 후 통과 시 상품을 저장하고 
    상품 상세 화면(redirect/items/{id: 방금 저장한 상품에 해당하는 id를 의미})에 해당하는 url을 redirect 해줌
    3. 사용자는 GET /items/{id}로 상품 상세 화면으로 이동

    오류 발생
    1번 과정에서 검증 통과 못하면
    -> 고객에게 다시 상품 입력 폼을 보여주고 어떤 값이 오류인지를 alert함
}



검증 처리 과정 실습 {
    //검증 오류 결과를 보관
    Map<String, String> errors = new HashMap<>();
    첫번째 String: 에러가 발생한 필드값 이름 저장
    -> 어떤 부분에서 오류가 발생한것인지 개발자가 알기 위해 필요
    두번째 String: 에러 상세 내용 저장
    -> 고객에게 알려줄 에러 내용

    if (!errors.isEmpty()) { // 에러 메세지 하나라도 있으면
        model.addAttribute("errors", errors); // 뷰에 보낼 객체 생성 후
        return "validation/v1/addForm"; // 입력 폼 다시 보내기
    }

    addForm.html 에러 메세지 출력 부분
    <div th:if="${errors?.containsKey('globalError')}" // errors 맵에 globalError라는 키 있으면
        <p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p> // globalError 키가 갖고있는 value 출력
    </div>

    errors에 아무것도 안들어있는 경우(즉 errors가 null인 경우) {
        그런 경우 errors.containsKey()를 호출하면 NullPointerException 이 발생할것
        하지만 errors?. 문법에서 errors가 null이어도 예외 발생 안하고 그냥 null만 반환함
        -> 그래서 errors에 아무것도 없어도 오류 안생김
    }

    
    문제점
    뷰 템플릿에 중복처리 많음
    타입 오류(잘못된 타입 들어가는거) 처리 안됨
    고객이 입력한 문자를 화면에 남겨야 함
    -> 고객이 어떤 값을 입력해서 오류가 발생한건지 알아야 다음부턴 그 값을 입력 안함
    -> 근데 오류가 발생하는 순간 고객이 입력했던 값은 사라져서 고객이 입력한 오류값을 뷰로 출력 못해줌
    -> 고객이 입력한 오류값을 임시로라도 저장할 공간이 필요

    해결
    
    필드값에서 오류 발생한 경우
    bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
    -> FieldError 생성자로 에러 생성한 후 bindingResult에 저장하기
    public FieldError(String objectName, String field, String defaultMessage) {}
    첫번째 매개변수: @ModelAttribute 객체 이름
    두번째 매개변수: 오류가 발생한 필드 이름
    세번째 매개변수: 디폴트 메세지

    필드를 넘어선 다른 부분에서 오류 발생한 경우
    bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야합니다. 현재 값 = " + resultPrice));
    -> FieldError 대신 ObjectError 생성자로 에러 생성한 후 bindingResult에 저장하기
    public ObjectError(String objectName, String defaultMessage) {}
    첫번째 매개변수: @ModelAttribute 객체 이름
    두번째 매개변수: 디폴트 메세지

    @ModelAttribute: 사용자가 http 요청시, 그 요청에 전달할 값을 오브젝트 형태로 매핑해주는 어노테이션

    타임리프 스프링 검증 오류 통합 기능 {
        #fields: #fields 로 BindingResult 가 제공하는 검증 오류에 접근 가능
        th:errors: 해당 필드에 오류가 있는 경우에만 태그를 출력. 없으면 태그 출력 안함 -> th:if 의 편의 버전
        th:errorclass: th:field 에서 지정한 필드에 오류가 있으면 class 정보를 추가

    }

    
    BindingResult 장점 {
        BindingResult가 없으면: 400 오류 뜨면서 컨트롤러 호출 안되고 바로 오류 페이지로 이동
        BindingResult가 있으면: 오류 정보(FieldError)를 BindingResult에 담아서 컨트롤러 정상 호출
    }
    
    FieldError 클래스 {
        FieldError는 오류 발생시 사용자가 입력한 오류값을 저장하는 기능 제공
        -> FieldError의 또다른 생성자 이용

        public FieldError(String objectName, String field, @Nullable Object
        rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
        Object[] arguments, @Nullable String defaultMessage)

        파라미터 목록
        objectName: 오류가 발생한 객체 이름
        field: 오류 필드
        rejectedValue: 사용자가 입력한 값(거절된 값) -> 고객이 입력한 오류값이 직접적으로 저장되는 필드
        bindingFailure: 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분하는 값
        codes: 메시지 코드
        arguments: 메시지에서 사용하는 인자
        defaultMessage: 기본 오류 메시지

        th:field="*{price}" 
        -> 정상 상황일때는 Model 객체 값 사용
        -> 오류 상황일때는 FieldError 객체를 통해 객체에 보관된 오류값을 사용해서 메세지 출력함

        오류 발생시 일어나는 일 정리하면
        -> FieldError 를 생성하면서 사용자가 입력한 값을 저장
        -> 해당 오류를 BindingResult 에 담아서 컨트롤러를 호출

        code, argument의 오류 코드를 사용해서 메세지 찾기 {
            new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1000, 1000000})
            codes: required.item.itemName를 사용해서 메시지 코드를 지정
            -> 메시지 코드는 하나가 아니라 배열로 여러 값을 전달 가능
            -> 순서대로 매칭해서 처음 매칭되는 메시지가 사용됨
            arguments: Object[]{1000, 1000000}를 사용해서 코드의 {0}, {1}로 치환할 값을 전달
        }
    }
    

    문제점
    FieldError, ObjectError 는 다루기 너무 번거로움
    오류 코드 자동화 필요

    -> BindingResult` 가 제공하는 rejectValue(), reject() 사용
    -> FieldError, ObjectError를 직접 생성하지 않고, 깔끔하게 검증 오류 처리 가능

    void rejectValue(@Nullable String field, String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
    field: 오류 필드명
    errorCode: 오류 코드(이 오류 코드는 메시지에 등록된 코드가 아니다. 뒤에서 설명할 messageResolver를 위한 오류 코드이다.)
    errorArgs: 오류 메시지에서 {0}을 치환하기 위한 값
    defaultMessage: 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지
}



오류 메세지 범위 {
    오류 메세지는 세밀하게 작성하거나 단순하게 작성할 수 있음
    -> 두 범위 안에서 적절하게 작성하는것이 중요

    모든 오류에 대해 개발자가 오류 메세지를 전부 작성할 수 없으므로
    -> 중요도가 낮은 메세지는 단순하게 작성한 메세지를 출력하게 하기
    -> 메세지를 구체적인 정도에 따라 여러 레벨로 나누고, 구체적인 것부터 시작해서 점점 덜 구체적인 메세지로 가기
    -> 중요도가 높아서 별도의 에러 메세지를 출력해줘야 하는 경우라면 낮은 레벨(1, 2) 선에서 구체적인 메세지가 출력
    -> 별로 중요하지 않은 에러는 덜 구체적인 메세지를 찾아서 출력하게 할 수 있음


    오류 코드 종류 {
        1. 개발자가 직접 작성한 오류 코드(rejectValue()로 직접 호출)
        2. 스프링이 직접 검증 오류에 추가한 코드
        3. 
    }
}
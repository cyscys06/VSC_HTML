입력폼 처리 {
    th:object : 커맨드 객체를 지정한다.
    *{...} : 선택 변수 식
    -> th:object 에서 선택한 객체에 접근
    th:field: HTML 태그의 id, name, value 속성을 자동으로 만들어줌

    th:object="${item}" : <form> 에서 사용할 객체(item) 지정(선택 변수 식( *{...} )을 적용 가능)
    
    선택변수 식 사용 {
        th:field="*{itemName}"
        *{itemName} = ${item.itemName}임(위에서 th:object 로 item객체를 선택했으므로 선택변수 식을 적용 가능함)
        
        th:field 는 id, name, value 속성을 모두 자동으로 만들어준다.
        id : th:field 에서 지정한 변수 이름과 같다. id="itemName"
        name : th:field 에서 지정한 변수 이름과 같다. name="itemName"
        value : th:field 에서 지정한 변수의 값을 사용한다. value=""
    }


    


    <input type="checkbox" th:field="${item.regions}" th:value="${region.key}" class="form-check-input" disabled>
    -> th:object 사용 안했으면 th:field 부분에 ${item.regions}처럼 
    어떤 객체의 프로퍼티를 사용하는 건지 명시적을로 적어줘야함
}



입력폼 처리 - 체크박스 {
    체크박스 사용할때
    체크박스에 체크를 하면 open이라는 속성 값이 on으로 바뀌고, 
    스프링은 이 문자를 true로 변환해줌
    (단 속성값이 on이 아니면 false가 아니라 null이 됨. open 속성 자체가 서버로 전송되지 않음)
    
    히든 필드 추가 {
        true일때만 전달되는 문제 해결: 히든 필드 추가
        체크박스 이름(open) 앞에 언더바 붙이기(_open)
        -> 스프링에서 체크를 해제했다고 판단(즉 false로 판단. No-operation 연산자 덕분)

        open=on&_open=on
        -> 이상태에서는 스프링이 open이 on 상태인거 확인하고 사용. _open은 신경 안씀
        -> 체크 안된경우 open = null이고, _open만 on(true)인상태(이때 false로 간주함)

        사용자가 일일이 히든 필드를 추가하기 번거로움
        -> <input type="hidden" name="_open" value="on"/> 사용
        -> 타임리프에서 제공하는 히든 필드. 체크박스의 히든 필드(이름:_open)를 자동으로 생성해줌
    }   
    
    <div>
        <div>등록 지역</div>
        <div th:each="region : ${regions}" class="form-check form-check-inline">
        // for (region : regions)
            <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
            <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label">서울</label>
        </div>
    </div>
    -> 같은 이름의 여러개의 체크박스 생성 가능
    -> 이때 name은 같아도 되나 id는 같으면 안됨
    -> 타임리프가 생성한 체크박스마다 id뒤에 1 2 3 숫자 붙여줌

    id 값 사용
    -> ids.prev(), ids.next() 을 제공해서 동적으로 생성되는 id값 사용
}



입력폼 처리 - 라디오버튼 {
    체크박스에서는 체크 안한경우 open 속성 자체가 서버에 전송 안됐었음
    하지만 라디오버튼은 이미 하나가 골라진 채로 시작할 경우 체크 사항을 수정해도 항상 최소 한개는 선택함
    -> 별도의 히든 필드 필요 없음
}



열거형 {
    모델 객체에 열거형 담아서 전달하는 대신 타임리프에서 자바 객체에 직접 접근 가능함 

    예시
    @ModelAttribute("itemTypes")
    public ItemType[] itemTypes() {
    return ItemType.values();
    }
    // 열거형 값들을 저장하는 ItemType 객체의 메서드 itemTypes()
    -> 열거형의 모든 정보가 배열 형태로 반환됨 

    <div th:each="type : ${T(hello.itemservice.domain.item.ItemType).values()}">
}
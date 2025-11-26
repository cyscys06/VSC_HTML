메세지 기능(메세지 표현식 #{} 사용) {
    메세지 기능: 다양한 메세지들을 한 곳에서 관리하는 기능

    -> <label for="itemName" th:text="#{item.itemName}"></label>
    -> 해당되는 데이터를 key를 통해 불러오는 형태로 가져오기

    국제화: 하나의 서비스에 대해 여러 나라의 언어로 동시에 서비스하는 것

    예시

    messages_en.properties // 영어 사용자
    item=Item
    item.id=Item ID
    item.itemName=Item Name
    item.price=price
    item.quantity=quantity
    
    messages_ko.properties // 한국어 사용자
    item=상품
    item.id=상품 ID
    item.itemName=상품명
    item.price=가격
    item.quantity=수량

    어디 나라에서 접근한건지 아는법
    -> HTTP accept-language 헤더 값 사용
    -> 사용자가 직접 언어 선택
    -> 쿠키 사용

    Locale = 현재 속한 지역에 따른 데이터들의 형식
    -> Locale에 따라 해당 지역의 데이터 형식을 사용하고, 없으면 디폴트 형식 사용

    Locale에 대한 properties 파일 기본 이름은 message.properties
    -> messages_en.properties, messages_ko.properties 처럼 파일명 마지막에 Locale 정보 추가


    getMessage {
       public interface MessageSource 인터페이스의 메서드 getMessage
        String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;
        첫번째 매개변수: message 파일에 있는 변수 이름
        -> 변수 이름을 통해 변수에 할당된 값을 불러오기 가능
        두번째 매개변수: 오브젝트 타입 배열
        -> message 파일에 있는 변수에 값을 할당할 수 있음
        세번째 매개변수: Locale 정보
        -> 이 매개변수를 기반으로 Locale 정보를 정하고 그에 따른 properties 파일을 불러와서 사용
        ms.getMessage("hello", null, null) : locale 정보가 null이므로 messages(디폴트)를 사용
        ms.getMessage("hello", null, Locale.KOREA) : locale 정보가 있지만, message_ko파일이 존재하지 않으므로 messages 를 사용 
    }
    

    #{} 표현식 사용 {
        messages.properties
        label.item=상품
        label.item.id=상품 ID
        label.item.itemName=상품명
        label.item.price=가격
        label.item.quantity=수량
        page.items=상품 목록
        page.item=상품 상세
        page.addItem=상품 등록
        page.updateItem=상품 수정
        button.save=저장
        button.cancel=취소

        상품이라는 값을 조회하고 싶으면
        -> <div th:text="#{label.item}"></h2> 사용
        -> <div>상품</h2>으로 랜더링됨
    }


    hello.name=안녕 {0}
    -> {0}에 해당하는 부분에 매개변수를 통해 값을 받아와서 할당 가능
    String result = ms.getMessage("hello.name", new Object[]{"Spring"}, null); // {0}에 Spring이라는 문자열이 적용됨
    -> hello.name = 안녕 Spring이 됨


    언어 선택 {
        스프링은 언어 선택시 accept-language 헤더의 값 사용
        웹에서 언어 변경 할수도 있고, 스프링 자체에서 바꿀수도 있음
        -> 스프링에서 바꾸려면 LocaleResolver 인터페이스 사용해야함'
        -> LocaleResolver: Locale 선택 방식을 변경할 수 있게 해주는 인터페이스
    }
}
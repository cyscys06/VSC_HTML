Bean Validation {
    검증 애노테이션 및 여러 인터페이스의 모음
    -> 검증 로직을 일일히 만들어줘야 하는 번거로움을 해결 가능

    implementation 'org.springframework.boot:spring-boot-starter-validation
    -> 의존성 추가


    FieldError 처리 {

        상품의 정보를 저장하는 Data 클래스
        @Data
        public class Item {
            private Long id;
            
            @NotBlank
            private String itemName;
            
            @NotNull
            @Range(min = 1000, max = 1000000)
            private Integer price;
            
            @NotNull
            @Max(9999)
            private Integer quantity;
            
            public Item() {    

            }

            public Item(String itemName, Integer price, Integer quantity) {
                this.itemName = itemName;
                this.price = price;
                this.quantity = quantity;
            }
        }
                @NotBlank : 빈 문자열·공백 문자열 금지
        @NotNull : null 금지
        @Range : 지정 범위 내의 숫자만 허용
        @Max : 최댓값 제한
        
        다음 등의 어노테이션을 통해 기준을 정하고, 
        @ModelAttribute에서 필드별 타입 변환을 시도함

        실패 시: 오류 발생, 해당 오류는 FieldError로 등록됨
        성공 시: 정상 등록

        예시
        itemName(String 타입) = "A" -> 문자열이므로 타입 변환 성공 -> Bean Validation의 검증 로직 수행
        price(Integer 타입) = "A" -> 숫자 변환 실패 → FieldError 추가, Bean Validation의 검증 로직 미적용
        
        Bean Validation 수행 후
        오류를 FieldError 또는 ObjectError 형태로 BindingResult에 전달
    }
   
    LocalValidatorFactoryBean를 글로벌 Validator로 등록
    -> 글로벌 Validator가 적용되어 있으므로 검증을 적용할 곳에 @Valid ,@Validated 만 적용하면 됨

    
    ObjectError 처리 {
        @ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")
        -> @ScriptAssert 사용하는 방법 있으나 제약 많고 실무에서 사용 어려워서 비추

        권장 방식: ObjectError 객체를 추가해주는 코드 직접 작성

        if (item.getPrice() != null && item.getQuantity() != null) {
            int total = item.getPrice() * item.getQuantity();
            
            if (total < 10000) {
                bindingResult.reject("totalPriceMin",
                new Object[]{10000, total}, null);
            }
        }

        -> 특정 필드에 한해서 처리하는 것 X, 객체 전체의 에러 처리 가능함
        -> 오류 코드는 messageSource를 통해 메세지로 변환
    }

    
    데이터를 수정할 때의 Bean Validation의 한계(group 이용) {
        데이터를 등록할때: 데이터 타입 검증, 필드 검증, 특정 필드의 타입의 최대값을 초과하는 경우 등을 에러로 잡아야 함
        데이터를 수정할때: 데이터를 수정할때는 quantity값을 무한대로 변경할 수 있음. 
        또한 데이터를 등록할때는 id가 필요 없지만, 데이터를 수정할때는 등록할때 생성된 id값이 필수로 들어감

        -> 동일한 모델 객체를 등록할때, 수정할때 각각 다른 방법으로 검증하는 방법 사용(BeanValidation groups)
        -> 등록시에 검증할 기능과 수정시에 검증할 기능을 각각 그룹으로 나누어서 적용 가능

        @Data
        public class Item {
            @NotNull(groups = UpdateCheck.class) // 수정시에만 적용
            private Long id;
            
            @NotBlank(groups = {SaveCheck.class, UpdateCheck.class}) // 등록, 수정 둘 다 쓰는 필드
            private String itemName;
            
            @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
            @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class}) // 등록, 수정 둘 다 쓰는 필드
            private Integer price;
            
            @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
            @Max(value = 9999, groups = SaveCheck.class) // 등록시에만 적용
            private Integer quantity; 
        }

        
    }


    group 한계 및 form 전송 객체 {
        group 한계: 데이터를 등록할때 사용자가 입력하는 값이 Item 클래스(즉 입력값을 저장할 클래스)객체와 딱 들어맞지 않음(약관같은거 추가로 받을수 있어서)
        -> 입력폼의 데이터를 따로 받아 컨트롤러로 전달하는 전용 객체를 만듦 -> 이 객체를 @ModelAttribute로 사용
        -> 이후 데이터를 받은 컨트롤러는 Item 클래스에 들어갈 데이터만 따로 선별해서 Item 객체에 담음
    }
}

@ModelAttribute vs @RequestBody {
    HttpMessageConverter: 사용자가 입력한 데이터가 포함된 http 요청을 받아서 DTO(위에서 말한 입력 폼 같은거)에 전달해주는 역할 한다고 보면됨
    
    공통점: 둘다 사용자로부터 온 http 요청을 받아서 자바 객체로 바꿔주는 역할 함
    
    차이점:
    @ModelAttribute: 필드 단위로 바인딩이 적용됨
    -> 특정 필드가 바인딩 되지 않아도(에러 나도) 나머지 필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있다.
    @RequestBody: HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행되지 않고 예외가 발생한다. 
    -> 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.
}
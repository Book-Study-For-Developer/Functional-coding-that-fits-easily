# 10. 일급 함수 1

- [요약](#요약)
- [코드의 냄새: 함수 이름에 있는 암묵적 인자](#코드의-냄새-함수-이름에-있는-암묵적-인자)
- [리팩터링: 암묵적 인자를 드러내기](#리팩터링-암묵적-인자를-드러내기)
  - [일급인 것과 일급이 아닌 것을 구별하기](#일급인-것과-일급이-아닌-것을-구별하기)
  - [데이터 지향](#데이터-지향)
- [리팩터링: 함수 본문을 콜백으로 바꾸기](#리팩터링-함수-본문을-콜백으로-바꾸기)
  - [함수를 정의하는 방법](#함수를-정의하는-방법)
 

# 요약

- 코드의 냄새: 함수 이름에 있는 암묵적 인자
    - 특징
        1. 거의 똑같이 구현된 함수가 있음
        2. 함수 이름이 구현에 있는 다른 부분을 가리킴
- 리팩터링: 암묵적 인자를 드러내기 - 암묵적 인자가 일급 값이 되도록 함수에 인자 추가
    - 단계
        1. 함수 이름에 있는 암묵적 인자 확인
        2. 명시적 인자 추가
        3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꿈
        4. 함수를 호출하는 곳을 고침
- 리팩터링: 함수 본문을 콜백으로 바꾸기 - 비슷한 함수에 있는 서로 다른 부분을 콜백으로 바꿔 함수에 동작 전달
    - 단계
        1. 함수 본문에서 바꿀 부분의 앞부분과 뒷부분 확인
        2. 리팩터링할 코드를 함수로 빼냄
        3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 빼냄

---

# 코드의 냄새: 함수 이름에 있는 암묵적 인자

- 함수 이름에 있는 암묵적 인자 특징
    1. 함수 구현이 거의 동일
    2. 함수 이름이 구현의 차이 만듦 - 함수 이름에서 서로 다른 부분이 암묵적 인자.

    <br />
    
    <details>
    <summary>코드 냄새 예시</summary>
      
          function setPriceByName(cart, name, price) {
            var item = cart[name];
            // 모두 함수 이름과 이 문자열만 다름. 문자열이 함수 이름에 들어있음.
            var newItem = objectSet(item, "price", price);
            var newCart = objectSet(cart, name, newItem);
            return newCart;
          }
          
          function setShippingByName(cart, name, ship) {
            var item = cart[name];
            var newItem = objectSet(item, "shipping", ship);
            var newCart = objectSet(cart, name, newItem);
            return newCart;
          }
          
          function setQuantityByName(cart, name, quant) {
            var item = cart[name];
            var newItem = objectSet(item, "quantity", quant);
            var newCart = objectSet(cart, name, newItem);
            return newCart;
          }
          
          function setTaxByName(cart, name, tax) {
            var item = cart[name];
            var newItem = objectSet(item, "tax", tax);
            var newCart = objectSet(cart, name, newItem);
            return newCart;
          }
          
          function objectSet(object, key, value) {
            var copy = Object.assign({}, object);
            copy[key] = value;
            return copy;
          }
          
          cart = setPriceByName(cart, "shoe", 13);
          cart = setQuantityByName(cart, "shoe", 3);
          cart = setShippingByName(cart, "shoe", 0);
          cart = setTaxByName(cart, "shoe", 2.34);
        
    </details>
    
    
    
        

# 리팩터링: 암묵적 인자를 드러내기

- 단계
    1. 함수 이름에 있는 암묵적 인자 확인
    2. 명시적 인자 추가
    3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꿈
    4. 함수를 호출하는 곳을 고침
- 리팩터링 - 어떤 필드값이든 설정할 수 있는 set`Field`ByName로 리팩터링
    
    ```tsx
    var validItemFields = ['price', 'quantity', 'shipping', 'tax', 'number'];
    var translations = { 'quantity': 'number' };
    
    function setFieldByName(cart, name, field, value) {
      // 필드명이 문자열이라 발생할 버그 방지 - 런타임에 검사
      // 동적 타입 언어인 JS 대신 정적 타입 시스템 언어인 TS 사용하면 컴파일 타임에 검사 가능
      if (!validItemFields.includes(field))
        throw "Not a valid item field: '" + field + "'.";
      // 만약 필드명 바꿔야 하는 상황 - 내부에서 바꾸면 됨
      // 원래 필드명을 새로운 필드명으로 바꿔줌 (quantity를 number로)
      if (translations.hasOwnProperty(field)) {
        field = translations[field];
      }
      
      var item = cart[name];
      var newItem = objectSet(item, field, value);
      var newCart = objectSet(cart, name, newItem);
      return newCart;
    }
    
    // ? 기존 - 함수 이름에 암묵적 인자 존재
    // cart = setPriceByName(cart, "shoe", 13);
    // cart = setQuantityByName(cart, "shoe", 3);
    // cart = setShippingByName(cart, "shoe", 0);
    // cart = setTaxByName(cart, "shoe", 2.34);
    
    // ? 변경 - 필드명을 일급 값으로 변경. (field라는 인자로 받음)
    cart = setFieldByName(cart, "shoe", 'price', 13);
    cart = setFieldByName(cart, "shoe", 'quantity', 3);
    cart = setFieldByName(cart, "shoe", 'shipping', 0);
    cart = setFieldByName(cart, "shoe", 'tax', 2.34);
    ```
    

## 일급인 것과 일급이 아닌 것을 구별하기

- 일급이 아닌 것
    1. 수식 연산자
    2. 반복문
    3. 조건문
    4. try/catch 블록
- 일급으로 할 수 있는 것
    1. 변수에 할당
    2. 함수의 인자로 넘기기
    3. 함수의 리턴값으로 받기
    4. 배열이나 객체에 담기

## 데이터 지향

- 이벤트와 엔티티에 대한 사실을 표현하기 위해 일반 데이터 구조를 사용하는 프로그래밍 형식
- 데이터를 데이터 그대로 사용 → 여러 방법으로 해석 가능

# 리팩터링: 함수 본문을 콜백으로 바꾸기

- 단계
    1. 함수 본문에서 바꿀 부분의 앞부분과 뒷부분 확인
    2. 리팩터링할 코드를 함수로 빼냄
    3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 빼냄
- `고차 함수`(Higher-order function) - 함수를 인자로 받거나 함수를 리턴할 수 있는 함수
    - forEach()는 함수를 인자로 받는 고차함수. for문 대신 쓰면 됨.
- 단계별로 리팩터링하기
    1. 함수 본문에서 바꿀 부분의 앞부분과 뒷부분 확인
        
        ```tsx
        try {
          saveUserData(user);
        } catch (error) {
          logToSnapErrors(error);
        }
        
        try {
          fetchProduct(productId);
        } catch (error) {
          logToSnapErrors(error);
        }
        ```
        
    2. 리팩터링할 코드를 함수로 빼냄
        
        ```tsx
        function withLogging() {
          try {
            saveUserData(user);
          } catch (error) {
            logToSnapErrors(error);
          }
        }
        
        withLogging();
        ```
        
    3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 빼냄
        
        ```tsx
        function withLogging(f) {
          try {
            f(); // 인자로 받은 함수 호출
          } catch (error) {
            logToSnapErrors(error);
          }
        }
        
        // 인라인 정의 - 익명 함수 사용
        withLogging(function () {
          saveUserData(user); 
        });
        ```
        

## 함수를 정의하는 방법

1. 전역으로 정의하기
2. 지역적으로 정의하기
3. 인라인으로 정의하기

# 08. 계층형 설계 1

- [계층형 설계](#계층형-설계)
  - [패턴 1: 직접 구현](#패턴-1-직접-구현)
    - [3단계 줌 레벨로 살펴보기](#3단계-줌-레벨로-살펴보기)

# 계층형 설계

- 계층형 설계: 소프트웨어를 계층으로 구성하는 기술
  - 각 계층에 있는 함수는 바로 아래 계층에 있는 함수 사용해 정의
  - 계층 구성
    1. 비즈니스 규칙
    2. 기능 위한 동작들
    3. 카피-온-라이트
    4. 언어에서 지원하는 배열 관련 기능
- 계층형 설계 패턴
  1. 직접 구현 - 적절한 정도의 구체화
  2. 추상화 벽 - 세부 구현 감추고 인터페이스 제공
  3. 작은 인터페이스 - 비즈니스 개념 나타내는 중요한 인터페이스는 작고 강력한 동작으로 구성해야
  4. 편리한 계층 - 작업할 때 편리하기 위해 사용해야. 개발자 요구 만족시키며 비즈니스 문제 잘 풀 수 있는 계층으로 구성

## 패턴 1: 직접 구현

- 추가 명세 구현 코드
  - 제대로 설계하지 않고 그냥 기능을 추가한 것 - 유지보수하기 어려움
  - 패턴 1 직접 구현을 따르지 않은 코드. freeTieClip()이 알 필요가 없는 구체적 내용 담고 있음.
    - cart가 배열이라는 정보
    - 배열 돌다가 off-by-one 에러(배열 처리할 때 의도치 않게 마지막 항목 처리하지 못하거나 처리하는 오류) 생기면 실패하는지?
  ```tsx
  function freeTieClip(cart) {
    var hasTie = false;
    var hasTieClip = false;
    for (var i = 0; i < cart.length; i++) {
      var item = cart[i];
      // 장바구니에 넥타이 있는지 확인
      if (item.name === "tie") hasTie = true;
      // 장바구니에 넥타이 클립 있는지 확인
      if (item.name === "tie clip") hasTieClip = true;
    }
    if (hasTie && !hasTieClip) {
      var tieClip = make_item("tie clip", 0);
      // 넥타이 클립 추가
      return add_item(cart, tieClip);
    }
    return cart;
  }
  ```
- 개선 코드
  - 저수준의 코드 추출함 - freeTieClip() 함수에 직접 구현 패턴 적용
  ```tsx
  function freeTieClip(cart) {
    // 추출한 함수 사용
    var hasTie = isInCart(cart, "tie");
    var hasTieClip = isInCart(cart, "tie clip");
    if (hasTie && !hasTieClip) {
      var tieClip = make_item("tie clip", 0);
      return add_item(cart, tieClip);
    }
    return cart;
  }

  // 반복문 추출해 새로 함수 만듦
  function isInCart(cart, name) {
    for (var i = 0; i < cart.length; i++) {
      if (cart[i].name === name) return true;
    }
    return false;
  }
  ```

### 3단계 줌 레벨로 살펴보기

1. 전역 줌 레벨 - 기본 줌 레벨. `계층 사이 상호 관계` 포함한 모든 문제 영역 살펴볼 수 있음.
2. 계층 줌 레벨 - 한 계층과 연결된 바로 아래 계층 볼 수 있는 줌 레벨. `특정 계층의 구현`이 어떻게 돼있는지 알 수 있음.
3. 함수 줌 레벨 - 함수 하나와 바로 아래 연결된 함수들을 볼 수 있음. `특정 함수의 구현` 문제를 찾을 수 있음.

- 예시로 살펴보기
  1. 계층별로 분류한 다이어그램 - 전역 줌 레벨

      ![image](https://github.com/user-attachments/assets/55ffdb4f-404f-4f58-a515-00cf3af72025)

  2. 계층 확대 → 장바구니 기본 동작(basic cart operations)과 그 아래 계층
  3. 함수 확대 → 함수 하나에 집중해보기
     1. remove_item_by_name()에 집중해보기

        - 서로 다른 두 계층에 있는 동작 사용하고 있음 → 중간에 함수 하나 둬서 같은 계층 함수 사용하도록

        ```tsx
        // 기존
        function remove_item_by_name(cart, name) {
          var idx = null;
          for (var i = 0; i < cart.length; i++) {
            if (cart[i].name === name) idx = i;
          }
          if (idx !== null) return removeItems(cart, idx, 1);
          return cart;
        }

        // 반복문 추출
        function remove_item_by_name(cart, name) {
          var idx = indexOfItem(cart, name);
          if (idx !== null) return removeItems(cart, idx, 1);
          return cart;
        }

        function indexOfItem(cart, name) {
          for (var i = 0; i < cart.length; i++) {
            if (cart[i].name === name) return i;
          }
          return null;
        }
        ```

        <img width="360px" src="https://github.com/user-attachments/assets/be044de9-ad2e-483b-be4c-5617d383dd74" /> <img width="360px" src="https://github.com/user-attachments/assets/37f45130-d78e-4260-992f-94b25e287e09" />



     3. setPriceByName()에 집중해보기
        1. 낮은 단계에 있는 다른 함수(indexOfItem()) 사용하기 → 화살표 길이 줄임
        2. 더 낮은 계층에 있는 다른 한쪽 함수(arraySet()) 사용하기 → 화살표 길이 줄임
        3. 새 함수 만들어 배열 인덱스를 직접 참조하는 부분을 없앰 → 분리된 계층 만듦

           ```tsx
           function setPriceByName(cart, name, price) {
             var i = indexOfItem(cart, name);
             if (i !== null) {
               // 배열 인덱스를 직접 참조하지 않도록 변경
               // var item = cart[i];
               var item = arrayGet(cart, i);
               return arraySet(cart, i, setPrice(item, price));
             }
             return cart;
           }

           function indexOfItem(cart, name) {
             for (var i = 0; i < cart.length; i++) {
               // 배열 인덱스를 직접 참조하지 않도록 변경
               // if (cart[i].name === name) return i;
               if (arrayGet(cart, i).name === name) return i;
             }
             return null;
           }

           function arraySet(array, idx, value) {
             var copy = array.slice();
             copy[idx] = value;
             return copy;
           }

           function arrayGet(array, idx) {
             return array[idx];
           }
           ```

           <img width="360px" src="https://github.com/user-attachments/assets/86bf3dc4-521e-49d9-9e0a-0dea57eff179" /> <img width="360px" src="https://github.com/user-attachments/assets/526bc903-b876-47df-92c4-d6f218868e82" />
           
           <img width="360px" src="https://github.com/user-attachments/assets/7514137c-106c-4912-9ae5-49c7954463aa" /> <img width="360px" src="https://github.com/user-attachments/assets/efdba57e-8fe9-4b65-b0c4-0cba52800fa9" /> 





- 직접 구현 패턴 리뷰
  - 직접 구현한 코드는 한 단계의 구체화 수준에 관한 문제만 해결함
    - 같은 구체화 단계에 있도록 구현 노력
  - 계층형 설계는 특정 구체화 단계에 집중할 수 있게 도와줌
  - 호출 그래프는 구체화 단계에 대한 풍부한 단서를 보여줌
  - 함수를 추출하면 더 일반적인 함수로 만들 수 있음
  - 일반적인 함수가 많을 수록 재사용하기 좋음
  - 복잡성을 감추지 않음
    - 계층형 설계에서 모든 계층은 바로 아래 계층에 의존해야 함. → 복잡한 코드를 같은 계층으로 옮기면 안 됨. 더 낮은 수준을 가진 일반적 함수 만들어 직접 구현 패턴 적용해야.

> ‘일반적인 함수’가 `indexOfItem`정도의 세세한 수준까지 ‘일반적이다’라고 정의해야 할지? 함수 하나 당 하나의 역할만 하도록 쪼개는 것으로 충분하지 않을지?
> 
> 3단계 줌 레벨이 설명을 위해 필요하다고 해도 이 개념을 아는 것에 대한 실효성이 있을지

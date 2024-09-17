# 계층형 설계 패턴

1. 직접 구현 - 적절한 정도의 구체화
2. 추상화 벽 - 세부 구현 감추고 인터페이스 제공
3. 작은 인터페이스 - 비즈니스 개념 나타내는 중요한 인터페이스는 작고 강력한 동작으로 구성해야
4. 편리한 계층 - 작업할 때 편리하기 위해 사용해야. 개발자 요구 만족시키며 비즈니스 문제 잘 풀 수 있는 계층으로 구성

## 패턴 2: 추상화 벽

- 세부 구현을 감춘 함수로 이루어진 계층. 구현을 전혀 몰라도 함수를 쓸 수 있음. 필요하지 않은 것은 무시할 수 있도록 간접적인 단계를 만듦
- 해결하는 문제: 팀 간 책임을 명확하게 나눔
- 배열인 장바구니를 객체로 다시 만들기
    - 객체로 만들면
        - 더 효율적
        - 직접 구현 패턴에 더 가까움
        - 특정 위치에 추가하거나 삭제하기가 배열보다 좋음
    - 사용하는 데이터 구조를 완전히 바꿨지만 함수 사용처에서는 변경점 없음 - 추상화 벽의 효과
    
    ```tsx
    function add_item(cart, item) {
      // ? 배열에서 객체로 변경
      // return add_element_last(cart, item);
      return objectSet(cart, item.name, item);
    }
    
    function calc_total(cart) {
      var total = 0;
      var names = Object.keys(cart); // 추가
      // for (var i = 0; i < cart.length; i++) {
      //   var item = cart[i];
      //   total += item.price;
      // }
      for (var i = 0; i < names.length; i++) {
        var item = cart[names[i]];
        total += item.price;
      }
      return total;
    }
    
    // function setPriceByName(cart, name, price) {
    //   var cartCopy = cart.slice();
    //   for (var i = 0; i < cartCopy.length; i++) {
    //     if (cartCopy[i].name === name) cartCopy[i] = setPrice(cartCopy[i], price);
    //   }
    //   return cartCopy;
    // }
    
    function setPriceByName(cart, name, price) {
      if (isInCart(cart, name)) {
        var item = cart[name];
        var copy = setPrice(item, price);
        return objectSet(cart, name, copy);
      } else {
        var item = make_item(name, price);
        return objectSet(cart, name, item);
      }
    }
    
    function remove_item_by_name(cart, name) {
      // var idx = indexOfItem(cart, name);
      // if (idx !== null) return splice(cart, idx, 1);
      // return cart;
      return objectDelete(cart, name);
    }
    
    // 필요 없어져서 지움
    // function indexOfItem(cart, name) {
    //   for (var i = 0; i < cart.length; i++) {
    //     if (cart[i].name === name) return i;
    //   }
    //   return null;
    // }
    
    function isInCart(cart, name) {
      // return indexOfItem(cart, name) !== null;
      // ? 항목이 있는지 확인하는 JS 객체 메서드
      return cart.hasOwnProperty(name);
    }
    
    ```
    
- 사용할 만한 상황
    1. 쉽게 구현을 바꾸기 위해
        - 서버 개발되기 전에 더미 데이터로 처리하는 때처럼 뭔가 바뀔 걸 알고 있지만 아직 준비되지 않은 경우 등
        - 만약을 대비해 불필요한 코드를 작성하는 일이 될 수도 있으니 주의
    2. 코드를 읽고 쓰기 쉽게 만들기 위해
    3. 팀 간에 조율해야 할 것을 줄이기 위해
    4. 주어진 문제에 집중하기 위해
- 패턴 2 추상화 벽 리뷰
    - 추상화 단계의 아래와 위에 있는 코드의 의존성을 없앨 수 있는 강력한 패턴
    - 서로 신경 쓰지 않아도 되는 구체적인 벽을 기준으로 나눠 서로 의존하지 않도록 함
    - 바뀌지 않을지도 모르는 코드를 언젠가 쉽게 바꿀 수 있게 만들려는 함정에 빠지지 말아야 - 쉽게 코드를 고치려고 추상화 벽을 사용하는 건 아님. 팀 간 커뮤니케이션 비용을 줄이고 복잡한 코드를 명확하게 하기 위해 전략적으로 사용해야
    - 신경쓰지 않아도 되는 것을 다루는 게 핵심임
        - 어느 부분을 신경쓰지 않도록 만들면 좋을지
        - 사람들이 몰라도 되는 건 무엇일지
        - 어떤 함수들이 비슷한 세부 사항을 신경 쓰지 않아도 되는 함수들일지

---

## 패턴 3: 작은 인터페이스

- 인터페이스 최소화하면 하위 계층에 불필요한 기능이 쓸데없이 커지는 것을 막을 수 있음
- 시계 할인 마케팅 추가됨: 장바구니 총합이 100달러 넘으면서 장바구니에 시계 있으면 시계를 10% 할인해줌
    
    
    <img width="360px" src="https://github.com/user-attachments/assets/04cf673f-0053-4a7b-9f39-04102b4f06e3" />
    
    > 제발 기한을 맘대로 던지지 말아주세요
    > 
- 구현 방법
    
    <img width="600px" src="https://github.com/user-attachments/assets/b16e0b0f-ffb5-42d4-a95f-19f85d23a3ad" />


    
    1. 추상화 벽에 만들기
        - 해시 맵 데이터 구조로 돼있는 장바구니에 접근 가능
        - 같은 계층에 있는 함수 사용 불가
        - 😢시스템 하위 계층 코드가 늘어남
        
        ```tsx
        function getsWatchDiscount(cart) {
          var total = 0;
          var names = Object.keys(cart);
          for (var i = 0; i < names.length; i++) {
            var item = cart[names[i]];
            total += item.price;
          }
          return total > 100 && cart.hasOwnProperty("watch");
        }
        ```
        
    2. 추상화 벽 위에 만들기
        - 해시 데이터 구조 직접 접근 불가
        - 추상화 벽에 있는 함수 사용해 장바구니에 접근해야
        - 😀직접 구현에 가까움
            - 새 기능을 만들 때 상위 계층에 만드는 게 작은 인터페이스 패턴. 하위 계층 고치지 않고 상위 계층에서 문제 해결 가능.
        
        ```tsx
        function getsWatchDiscount(cart) {
          var total = calcTotal(cart);
          var hasWatch = isInCart("watch");
          return total > 100 && hasWatch;
        }
        ```
        
- 패턴 3 작은 인터페이스 리뷰
    - 상위 계층에 함수 만들 때 가능한 현재 계층에 있는 함수로 구현
    - 작은 인터페이스가 전체 계층에 사용되는 이상적인 모습은 계층이 가진 함수는 완전하고, 적고, 시간이 지나도 바뀌지 않아야 함.
    - 추상화 벽 작게 만들어야 하는 이유
        1. 추상화 벽에 코드가 많을 수록 구현이 변경됐을 때 고쳐야 할 것이 많음
        2. 추상화 벽에 있는 코드는 낮은 수준의 코드임 → 더 많은 버그가 있을 수 있음
        3. 낮은 수준의 코드는 이해하기 더 어려움
        4. 추상화 벽에 코드가 많을수록 팀 간 조율해야 할 것도 많아짐
        5. 추상화 벽에 인터페이스가 많으면 알아야 할 것이 많아 사용하기 어려움

---

## 패턴 4: 편리한 계층

- 구체적인 걸 너무 많이 알아야 하거나 코드가 지저분하다고 느껴지면 패턴 적용, 사용하기 편하다고 생각하면 중지

---

# 그래프로 코드 알아보기

- 요구사항
    - 기능적 요구사항: 소프트웨어가 정확히 해야 하는 일
    - 비기능적 요구사항
        - 유지보수성: 유지보수하기 어렵지 않은지. 요구사항이 바뀌었을 때 가장 쉽게 고칠 수 있는 코드는 어떤 코드인지.
        - 테스트성: 테스트를 어떻게 할 것인지. 어떤 것을 테스트하는 것이 가장 중요한지.
        - 재사용성: 재사용을 잘할 수 있는지. 어떤 함수가 재사용하기 좋은지.

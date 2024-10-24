# 18. 반응형 아키텍처와 어니언 아키텍처

- [독립적인 반응형과 어니언 아키텍처 패턴](#독립적인-반응형과-어니언-아키텍처-패턴)
- [반응형 아키텍처란](#반응형-아키텍처란)
- [어니언 아키텍처란](#어니언-아키텍처란)

## 독립적인 반응형과 어니언 아키텍처 패턴

- 반응형 아키텍처
    - 순차적 액션 단계에 사용
    - 코드에 나타난 순차적 액션의 순서를 뒤집음
    - 효과와 그 효과에 대한 원인을 분리해서 코드에 복잡하게 꼬인 부분을 풀 수 있음
- 어니언 아키텍처
    - 서비스의 모든 단계에 사용
    - 현실 세계와 상호작용하기 위한 서비스 구조를 만듦
    - 웹 서비스, 온도 조절 장치 등

---

## 반응형 아키텍처란

- 애플리케이션을 구조화하는 방법
- 핵심 원칙: 이벤트에 대한 반응으로 일어날 일을 지정하는 것
    - 웹 서비스 - 웹 요청(GET api 등)에 대한 응답에 일어날 일 지정
    - UI - 버튼 클릭과 같은 이벤트에 대한 응답에 일어날 일 지정
    - ⇒ 원래 순차적으로 실행됐던 걸 여러 개의 핸들러에서 실행되도록 나눔
        - 일반적 아키텍처
            
            ```tsx
            var shopping_cart = {};
            
            // ! 핸들러에 모든 액션 시퀀스가 있음 -> 순차적 발생
            
            function add_item_to_cart(name, price) {
              var item = make_cart_item(name, price);
            	// 1. 전역 장바구니에 제품 추가
              shopping_cart = add_item(shopping_cart, item);
            	// 2. 합계 계산
              var total = calc_total(shopping_cart);
            	// 3. 합계 DOM 업데이트
              set_cart_total_dom(total);
            	// 4. 배송 아이콘 업데이트
              update_shipping_icons(shopping_cart);
            	// 5. 세금 DOM 업데이트
              update_tax_dom(total);
            }
            ```
            
        - 반응형 아키텍처:
            
            ```tsx
            var shopping_cart = ValueCell({});
            // 2-b. 합계 계산
            // ? shopping_cart가 바뀔 때 cart_total도 바뀜
            var cart_total = FormulaCell(shopping_cart, calc_total);
            
            function add_item_to_cart(name, price) {
              var item = make_cart_item(name, price);
            	// 1. 전역 장바구니에 제품 추가
            	// 값을 변경하기 위해 값을 직접 사용하지 않고 메서드 호출함
              shopping_cart.update(function (cart) {
                return add_item(cart, item);
              });
            }
            
            // ! 하위 액션은 핸들러 바깥에 있음
            
            // 2-a. 배송 아이콘 업데이트 (장바구니 바뀔 때마다 실행됨)
            shopping_cart.addWatcher(update_shipping_icons);
            // ? cart_total이 바뀌면 DOM이 업데이트됨
            // 2-b-i. 합계 DOM 업데이트
            cart_total.addWatcher(set_cart_total_dom);
            // 2-b-ii. 세금 DOM 업데이트
            cart_total.addWatcher(update_tax_dom);
            ```
            
            ```tsx
            // ? 변경 가능한 값을 일급 함수로 만들기
            function ValueCell(initialValue) {
            	// 변경 불가능한 값을 하나 담아둠
              var currentValue = initialValue;
              var watchers = [];
              return {
            		// 현재 값을 가져옴
                val: function () {
                  return currentValue;
                },
            		// 현재 값에 함수를 적용해 값을 바꿈 (교체 패턴)
                update: function (f) {
                  var oldValue = currentValue;
                  var newValue = f(oldValue);
                  if (oldValue !== newValue) {
                    currentValue = newValue;
            				// 값이 바뀔 때 모든 감시자(watcher)를 실행
                    forEach(watchers, function (watcher) {
                      watcher(newValue);
                    });
                  }
                },
            		// 새 감시자를 추가
                addWatcher: function (f) {
                  watchers.push(f);
                },
              };
            }
            ```
            
            ```tsx
            function FormulaCell(upstreamCell, f) {
              var myCell = ValueCell(f(upstreamCell.val()));
            	// 셀 값을 다시 계산하기 위해 감시자를 추가
              upstreamCell.addWatcher(function (newUpstreamValue) {
                myCell.update(function (currentValue) {
                  return f(newUpstreamValue);
                });
              });
              return {
            		// val(), addWatcher()를 myCell에 위임
                val: myCell.val,
                addWatcher: myCell.addWatcher,
              };
            }
            ```
            
            - 감시자: watcher, callback, listener, observer, event handler
- 코드에 주는 영향
    1. 원인과 효과가 결합한 것을 분리
        - ❗ 문제가 없는데 이렇게 무조건 분리하는 게 마냥 좋진 않음. 액션을 순서대로 표현하는 게 더 명확할 수도 있기 때문. 장바구니 전역 변수처럼 원인과 효과의 중심이 있는 게 아니라면 분리하지 말기
    2. 여러 단계를 파이프라인으로 처리
        - ❗ 여러 단계가 있지만 데이터를 전달하지 않는다면 이 패턴을 사용하지 않는 게 좋음. 데이터를 전달하면 파이프라인이라고 볼 수 없고, 올바른 반응형 아키텍처가 될 수 없음.
    3. 타임라인이 유연해짐
        - 타임라인이 작은 부분으로 분리됨.
        - 공유하는 자원이 없으면 타임라인이 많아저도 문제 없음
            - ❗ 공유하는 자원이 있으면 타임라인이 많아지는 게 좋진 않음

---

## 어니언 아키텍처란

- 현실 세계와 상호작용하기 위한 서비스 구조를 만드는 방법
- 둥글게 겹겹이 쌓인 양파 모양 (1번이 제일 바깥에서 안을 감싸고 있음)
    1. 인터렉션 계층
        - 바깥 세상에 영향을 주거나 받는 액션
    2. 도메인 계층
        - 비즈니스 규칙을 정의하는 계산
            - 도메인 규칙이 액션이 되어야 할지 (도메인 계층은 대부분 계산)
                1. 도인 규칙은 도메인 용어를 사용
                2. 가독성과 어울리는지 따져 봐야
    3. 언어 계층
        - 언어 유틸리티와 라이브러리
- 함수형 시스템이 잘 동작할 수 있는 규칙
    1. 현실 세계와 상호작용은 인터렉션 계층에서 해야
        - 변경과 재사용이 쉬움: 인터렉션 계층이 제일 위에 있어서.
    2. 계층에서 호출하는 방향은 중심 방향
    3. 계층은 외부에 어떤 계층이 있는지 모름

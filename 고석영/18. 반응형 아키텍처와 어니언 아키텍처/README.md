## 반응형 아키텍처

### 반응형 아키텍처란?

> 반응형 아키텍처는 애플리케이션을 구조화하는 방법입니다.

- 이벤트에 대한 반응으로 일어날 일을 지정하는 것이 반응형 아키텍처의 핵심 원칙
- 웹 서비스와 UI에 적용하면 좋음 → 이벤트 핸들러
  - 웹 서비스 → 웹 요청 응답에 일어날 일을 지정
  - UI → 버튼 클릭과 같은 이벤트 응답에 일어날 일을 지정

### 반응형 아키텍처의 절충점

- 원인의 효과와 결합한 것을 분리 → 하려고 하는 것을 정확하게 표현 가능
- 여러 단계의 파이프라인으로 처리 → 액션과 계산을 조합 가능
- 유연한 타임라인 → 순서를 표현하는 방법을 뒤집어 타임라인을 더 짧게 만들기 가능

### 일급 상태 모델: `ValueCell`

> 상태를 일급 함수로 만들어 보자.

#### 값 하나와 두 개의 동작을 가지는 `ValueCell`

```tsx
function ValueCell(initialValue) {
  let currentValue = initialValue // 변경 불가능한 값

  return {
    val: () => currentValue, // 현재 값 가져오는 동작 (get)
    update: function (f) {
      // 교체 패턴으로 현재 값 바꾸는 동작 (set)
      const oldValue = currentValue
      const newValue = f(oldValue)
      currentValue = newValue
    },
  }
}
```

- 값 하나와 두 개의 동작
  - `currentValue`: 변경 불가능한 값
  - `val()`: 값을 읽는 동작
  - `update()`: 현재 값을 바꾸는 동작

#### `add_item_to_cart`에 적용하기

- 값을 변경하기 위해 값을 직접 사용하지 않고 메서드로 호출

```tsx
const shopping_cart = ValueCell({})

function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price)
  shopping_cart.update(function (cart) {
    // 메서드 호출
    return add_item(cart, item)
  })
  const total = calc_total(shopping_cart.val())
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart.val())
  update_tax_dom(total)
}
```

### `ValueCell`을 반응형으로 만들기

#### 감시자(watcher) 개념 추가하기

```tsx
function ValueCell(initialValue) {
  let currentValue = initialValue
  let watchers = [] // 감시자 목록

  return {
    val: () => currentValue,
    update: function (f) {
      const oldValue = currentValue
      const newValue = f(oldValue)
      if (oldValue !== newValue) {
        // 값 변경 여부 확인
        currentValue = newValue
        forEach(watchers, function (watcher) {
          // 모든 감시자 실행
          watcher(newValue)
        })
      }
    },
    addWatcher: function (f) {
      // 새로운 감시자 추가
      watchers.push(f)
    },
  }
}
```

- 감시자: 상태가 바뀔 때마다 실행되는 핸들러 함수
- 감시자로 장바구니가 바뀔 때 할 일을 지정 가능 → 여기서는 배송 아이콘 갱신

> [!TIP]
>
> #### 감시자의 동의어
>
> - 감시자(watcher)
> - 옵저버(observer)
> - 리스너(listener)
> - 이벤트 핸들러(event handler)
> - 콜백(callback)

#### `add_item_to_cart`에 적용하기

```tsx
const shopping_cart = ValueCell({})

function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price)
  shopping_cart.update(function (cart) {
    return add_item(cart, item)
  })
  const total = calc_total(shopping_cart.val())
  set_cart_total_dom(total)
  update_tax_dom(total)
}

shopping_cart.addWatcher(update_shipping_icons)
```

- 감시자가 가져다준 개선점
  - 핸들러 함수가 작아짐 → 책임을 감시자로 넘김
  - `update_shipping_icons` 호출을 줄임 → 하나의 코드로 항상 최신의 장바구니 상태를 반영

### `FormulaCell`으로 파생된 값 계산하기

#### 다른 셀의 변화가 감지되면 값을 다시 계산하는 `FormulaCell`

```tsx
function FormulaCell(upstreamCell, f) {
  const myCell = ValueCell(f(upstreamCell.val())) // ValueCell 재사용
  upstreamCell.addWatcher(function (newUpstreamValue) {
    // 셀 값을 다시 계산하기 위한 감시자 추가
    myCell.update(function (currentValue) {
      return f(newUpstreamValue)
    })
  })

  return {
    val: myCell.val, // myCell에 val() 위임
    addWatcher: myCell.addWatcher, // myCell에 addWatcher() 위임
    // update(set)을 안 넘김
  }
}
```

- 감시하던 상위(upstream) 셀이 바뀌면 상위 값을 가지고 셀 값을 다시 계산
- `update()` 동작을 반환하지 않아 값을 바꾸는 기능은 없지만 감시는 가능
- 감시가 가능하기 때문에 `addWatcher()`로 값 변경 시 실행할 액션 추가 가능

#### `add_item_to_cart`에 적용하기

```tsx
const shopping_cart = ValueCell({})
const cart_total = FomulaCell(shopping_cart, calc_total)

function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price)
  shopping_cart.update(function (cart) {
    return add_item(cart, item)
  })
}

shopping_cart.addWatcher(update_shipping_icons)
cart_total.addWatcher(set_cart_total_dom)
cart_total.addWatcher(update_tax_dom)
```

### 함수형 프로그래밍과 변경 가능한 상태

- 함수형 프로그래밍을 비롯해 모든 소프트웨어는 변경 가능한 상태를 잘 관리해야 함
- 상태를 가능한 한 안전하게 사용하는 것이 중요!

> 셀은 변경할 수 있지만 변경 불가능한 변수에 값을 담아두기 때문에 전역변수보다 더 안전합니다.

- 올바른 값을 유지하는 이유
  - `update()`를 사용할 때 계산을 넘기기 때문
  - 현재 값이 올바른 값이고 계산이 항상 올바른 값을 리턴한다면 → 항상 올바른 값 유지

> `ValueCell`은 타임라인에서 읽거나 쓰는 순서를 보장하지는 않지만, 어떤 값이 저장되어도 그 값이 항상 올바른 값이라는 것을 보장합니다.

> [!NOTE]
>
> #### 함수형 언어/프레임워크에서 `ValueCell` 역할을 하는 것들
>
> - 클로저(Clojure): **Atom**
> - 리액트(React): **Redux store**와 **Recoil atom**
> - 엘릭서(Elixir): **Agent**
> - 하스켈(Haskell): **TVar**

### 반응형 아키텍처가 바꾼 시스템의 결과

#### 결합의 분리는 원인과 효과의 중심을 관리

- 버튼 클릭이라는 원인과 그로 인해 발생하는 배송 아이콘 갱신이라는 효과가 결합
- 해결하기 위해선? 전역 장바구니가 원인과 결과의 중심이므로, 여기에서 원인과 효과를 분리
  > 하나가 추가된다고 하더라도 효과를 추가해도 원인을 고치치 않아도 되고, 원인을 추가해도 효과를 고치지 않아도 되기 때문에 관리해야 할 것이 하나만 늘어납니다.
- 각각 신경써야 하는 것에만 신경쓰기
  - 이벤트 핸들러 → 장바구니를 바꾸는 일에만
  - DOM을 업데이트 하는 곳 → DOM을 갱신하는 것만

#### 여러 단계를 파이프라인으로 처리

- 반응형 아키텍처도 간단한 액션과 계산을 조합해 복잡한 동작을 만들 수 있음 → 파이프라인
- 파이프라인은 작은 액션과 계산을 조합한 하나의 액션
- 각 단계에서 생성된 데이터는 다음 단계의 입력값으로 사용

#### 유연한 타임라인

- 반응형 아키텍처은 순서를 정의하는 방법을 뒤집기 떄문에 자연스럽게 타임라인이 작은 부분으로 분리됨
- 반드시 타임라인을 짧게 만들어야 하는 것은 아님 → 공유하는 자원이 없으면 타임라인이 많아져도 문제가 없음

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0523-02.jpg)

- 장바구니 `ValueCell`
  - 감시자 호출 시 현재 값을 넘겨주기 때문에
  - 감시자 함수가 직접 장바구니 값을 읽지 않아도 됨 → 장바구니 값을 전연변수로 사용하지 않아도 됨
- 합계 `FormulaCell`
  - 감시자를 호출 시 현재 합계 값을 넘겨주기 때문에
  - DOM 업데이트에서 FormulaCell을 직접 읽지 않아도 됨 → DOM을 업데이트하는 모든 곳에서 자신의 DOM만 갱신하면 됨
- 서로 다른 자원을 사용하기 떄문에 안전한 타임라인

## 어니언 아키텍처

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0527-01.jpg)

- 현실 세계와 상호작용하기 위한 서비스 구조를 만드는 방법
- 양파처럼 둥글게 겹겹이 쌓인 모양이라서 어니언 아키텍처
- 어니언 아키텍처는 액션과 계산의 분리, 계층형 설계 방식과 잘 어울림
  - 액션에서 계산을 빼내면 의도하지 않아도 어니언 아키텍처 구조가 됨
  - 계층형 설계로 액션 전파를 확인하고 액션에서 계산을 빼내 하위 액션+계산 조합으로 변경

### 어니언 아키텍처 계층

- 인터랙션 계층: 바깥세상에 영향을 주거나 받는 액션
- 도메인 계층: 비즈니스 규칙을 정의하는 계산
- 언어 계층: 언어 유틸리티와 라이브러리

### 어니언 아키텍처 규칙

- 현실 세계와 상호작용은 인터랙션 계층에서
- 계층에서 호출하는 방향은 중심 방향
- 계층은 외부에 어떤 계층이 있는지 모름

### 전통적인 계층형 아키텍처 vs 함수형 아키텍처

- 데이터베이스 계층과 도메인 계층의 관계
- 함수형 아키텍처는 도메인 계층이 데이터 베이스 계층에 의존하지 않음
- 데이터베이스 동작은 값을 바꾸거나 데이터베이스에 접근하기 때문에 액션
- 데이터베이스는 변경 가능하고 접근하는 모든 것을 액션으로 만든다는 것이 핵심!
- 따라서 데이터베이스를 도메인과 분리하는 것이 중요
- 가장 위에 있는 액션에서 도메인 규칙과 데이터베이스 조합

## 소프트웨어 아키텍처 선택 시 고려해야 할 사항

### 변경과 재사용

> 변경과 재사용이 쉬워야 합니다.

#### 변경과 재사용 관점에서 어니언 아키텍처

> 어니언 아키텍처는 좋은 인프라보다 좋은 도메인을 강조합니다.

- 데이터베이스나 API 호출과 같은 외부 서비스(인터랙션 계층)를 바꾸기 쉬움
  - 인터랙션 계층이 가장 높은 계층에서 사용하기 떄문
- 도메인 계층을 테스트하기 쉬움
  - 도메인 계층이 외부 서비스에 의존하지 않기 때문

#### 🤔 도메인 규칙에서 데이터베이스를 사용해야 하는데요?

- 전형적인 아키텍처에서 도메인 규칙은 데이터베이스를 호출 → 어니언 아키텍처는 다른 방식으로 처리해야 함!

| 전통적인 아키텍처                                                               | 어니언 아키텍처                                                                 |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| ![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0533-02.jpg) | ![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0533-03.jpg) |
| 도메인 규칙이 데이터 베이스에 직접 접근                                         | 핸들러가 데이터베이스에서 데이터를 가져와 도메인에 전달하는 역할                |
| → 도메인이 계산이 될 수 없고 액션이 됨                                          | → 인터랙션 계층에서 값을 가져오고 도메인 계층에서 합산하는 계산이 됨            |

#### 🤔 도메인 규칙이 액션이 되어야 하는 경우는 정말 없나요?

- 액션에서 계산을 빼내는 것이 좋음
- 액션은 작은 로직을 갖는 하위 단계 액션
- 하위 단계 액션이 되는 이유 → 빼낸 도메인 계산과 액션을 상위 단계의 액션으로 조합할 수 있기 때문

### 도메인 규칙

> 도메인 규칙은 도메인 용어를 사용합니다.

#### 도메인 규칙이 아닌 경우

- 데이터베이스를 마이그레이션하는 경우 → 데이터베이스라는 바깥세상과 상호작용하는 인터랙션 계층
- 불안정한 네크워크 문제를 해결하는 경우 → 웹이라는 바깥세상과 상호작용하는 인터랙션 계층

### 가독성

> 문맥에 따라 계산보다 액션이 읽기 좋은 경우가 있습니다.

#### 가독성을 결정하는 요소

- 사용하는 언어
- 사용하는 라이브러리
- 레거시 코드와 코드 스타일
- 개발자들의 습관

#### 현실 세계의 문제와의 균형 유지를 위한 고려 사항

- 코드의 가독성
- 개발 속도
- 시스템 성능

### 도메인 계층에서 선택적인 값 가져오기

> 항상 도메인 계층과 인터랙션 계층을 분리해 도메인 규칙을 계산으로 유지하자.

#### 데이터베이스에서 제품을 가져와서 보고서를 생성하는 `generateReport()`

```tsx
function generateReport(products) {
  return reduce(products, '', function (report, product) {
    return report + product.name + ' ' + product.price + '\n'
  })
}

const productsLastYear = db.fetchProducts('last year')
const reportLastYear = generateReport(productsLastYear)
```

#### 새로운 요구사항: 보고서에 할인 정보 표시하기

```tsx
function generateReport(products) {
	return reduce(products, "", function(report, product) {
		return report + product.name + " " + product.price + "\n"
	})
}

const productsLastYear = db.fetchProducts('last year')
// 제품을 데이터에이스에서 가져오는 가장 상위 계층인 인터랙션 계층에서 진행
const productsWithDiscounts = map(productsLastYear, function(product) {
	if(!product.discountID) product
	return objectSet(product, 'discount', db.fetchProducts(product.discountID)
})
const reportLastYear = generateReport(productsWithDiscounts)
```

## 두 아키택처 패턴은 독립적이다.

반응형 아키텍처: 순차적 액션 단계에 사용

어니언 아키택처: 서비스의 모든 단계에 사용

## 반응형 아키텍처는 무엇인가요?

반응형 아키텍처는 웹 서비스와 UI에 잘 어울린다. 웹 요청 응답에 일어날 일을 지정, UI는 버튼 클릭과 같은 이벤트 응답에 일어날 일을 지정

반응형 아키텍처는 순차적 액션의 순서를 뒤집는다. X를 하고 Y하는 대신 X가 일어나면 언제나 Y를 한다.

- 원인과 효과가 결합한 것을 분리한다.
- 여러 단계를 파이프라인으로 처리한다.
- 타임라인이 유연해진다.

### 셀은 일급 상태이다.

장바구니를 예로 들면 장바구니가 변경될 때 Y를 하도록 하는 것인데, 장바구니가 언제 바뀔지 모른다.
(여기서 셀이라고 정한 이유는 스프레드시트에서 영향을 받아 셀이 바뀌면 함수가 다시 계산하기 때문이다.)

```tsx
function ValueCell(initialValue) {
  const currentValue = initialValue;

  return {
    val: function () {
      return currentValue;
    },
    update: function (f) {
      const oldValue = currentValue;
      const newValue = f(oldValue);
      currentValue = newValue;
    },
  };
}

// 사용 예시
const shipping_cart = ValueCell({});

function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price);
  shoping_cart.update(function (cart) {
    return add_item(cart, item);
  });

  const total = calc_total(shopping_cart.val());
  set_cart_total_dom(total);
  update_shopping_icons(shopping_cart.val());
  update_tax_dom(total);
}
```

앞에서 말했듯이 상태가 바뀌면 알아서 Y를 하도록 만들어야 한다.

그러기 위해서는 감시자(watcher) 개념을 추가해야 한다.

```tsx
function ValueCell(initialValue) {
  const currentValue = initialValue;
  const watchers = [];

  return {
    val: function () {
      return currentValue;
    },
    update: function (f) {
      const oldValue = currentValue;
      const newValue = f(oldValue);
      if (oldValue !== newValue) {
        currentValue = newValue;
        forEach(watchers, function (watcher) {
          watchers(newValue);
        });
      }
    },
    addWatchers: function (f) {
      watchers.push(f);
    },
  };
}
```

react-query도 동일한 패턴을 사용하고 있다. (내가 썼던 블로그 글 ⇒ [https://velog.io/@haryan248/React-query를-뜯어보며-Observer-패턴-알아보기](https://velog.io/@haryan248/React-query%EB%A5%BC-%EB%9C%AF%EC%96%B4%EB%B3%B4%EB%A9%B0-Observer-%ED%8C%A8%ED%84%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0))

watcher를 적용하기

```tsx
const shipping_cart = ValueCell({});

function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price);
  shoping_cart.update(function (cart) {
    return add_item(cart, item);
  });

  const total = calc_total(shopping_cart.val());
  set_cart_total_dom(total);
  // 	update_shopping_icons(shopping_cart.val());
  update_tax_dom(total);
}

shopping_cart.addWatchers(update_shipping_icons);
```

- 핸들러의 함수가 작아짐 ⇒ 원래 하던 것보다 적은 일을 한다.
- 장바구니를 바꾸는 핸들러에서 `update_shipping_icons` 를 부르지 않아도 항상 실행되기 때문에 최신 장바구니를 반영하게 된다.

### FormulaCell은 파생된 값을 계산한다.

어떤 셀은 다른 셀의 값을 최신으로 반영하기 위해 파생될 수 있다. 다른 셀의 변화가 감지되면 값을 다시 계산한다.

```tsx
function FormulaCell(upstreamCell, f) {
	const myCell = ValueCell(f(updstreamCell.val()));

	upstreamCell.addWatcher(function(newUpstreamValue) {
		myCell.update(function(currentValue) {
			return f(newUpstreamValue);
		}
	});

	return {
		val: myCell.val,
		addWatcher: myCell.addWatcher
	}
}
```

감시하던 상위 셀 값이 바뀌면 FormulaCell값이 바뀌게 되고 상위 값을 가지고 다시 계산한다.

```tsx
const shipping_cart = ValueCell({});
const cart_total = FormulaCell(shopping_cart, calc_total);

function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price);
  shoping_cart.update(function (cart) {
    return add_item(cart, item);
  });
}

shopping_cart.addWatchers(update_shipping_icons);
calc_total.addWatcher(set_cart_total_dom);
cart_total.addWatcher(update_tax_dom);
```

상위 셀(shopping_cart)에서 total값을 바꿀 때 DOM을 갱신시켜준다.

반응형 아키텍처로 바꾸게 되면서 3가지가 중요한 영향을 끼친다.

- 원인과 효과가 결합된 것을 분리한다.
- 여러 단계를 파이프라인으로 처리한다.
- 타임라인이 유연해진다.

### 원인과 효과가 결합한 것을 분리한다.

장바구니가 바뀌는 경우는 많이 있다. 이에 따라 코드에서는 항상 배송 아이콘을 갱신해야 한다.
즉, **버튼 클릭이라는 원인**과 그로 인해 **발생하는 배송 아이콘 갱신**이라는 효과가 결합되어 있다.

어떤 이유로든 장바구니가 바뀔 때 배송 아이콘을 갱신하면 된다.

| **장바구니를 바꾸는 방법**                                                                           | **장바구니가 바뀔 때 해야 할 액션**                                                               |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 1. 제품 추가 <br/>2. 제품 삭제 <br/>3. 장바구니 비우기<br/> 4. 수량 변경<br/> 5. 할인 코드 적용<br/> | 1. 배송 아이콘 업데이트 <br/>2. 세금 표시 <br/>3. 합계 표시 <br/>4. 장바구니에 제품 개수 업데이트 |

액션은 언제든 새롭게 추가되거나 제거될 수 있다. 그러면 변경사항이 발생할 때마다 코드를 모두 고쳐야 한다.
원인과 결과가 모두 연결되어 있기 때문에 20개를 건드려야 한다.

**전역 장바구니는 원인과 결과의 중심이다.**

### 여러 단계를 파이프라인으로 처리한다.

조합된 액션은 파이프라인과 같다. 파이프라인은 작은 액션과 계산을 조합한 하나의 액션이라고 볼 수 있다.
자바스크립트를 사용한다면 **Promise로 액션과 계산을 조합해 파이프라인을 구현**할 수 있다.

### 타임라인이 유연해진다.

순서를 정의하는 방법을 뒤집기 때문에 자연스럽게 타임라인이 작은 부분으로 분리된다.

타임라인이 많아지는 것은 좋지 않은데 문제가 없는 케이스가 있다 ⇒ 공유하는 자원이 없으면 많아져도 문제가 없다.

DOM을 갱신하는 모든 곳에서 자신의 DOM만 갱신하면 되기 때문에 타임라인이 유연해진다.

## 어니언 아키텍처는 무엇인가요?

어니언 아키텍처는 현실 세계와 상호작용하기 위한 서비스 구조를 만드는 방법이다.

- 현실 세계와 상호작용은 인터렉션 계층에서 해야 한다.
- 계층에서 호출하는 방향은 중심 방향이다.
- 계층은 외부에 어떤 계층이 있는지 모른다.

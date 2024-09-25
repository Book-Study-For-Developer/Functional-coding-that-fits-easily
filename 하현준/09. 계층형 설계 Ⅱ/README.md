## 패턴 2: 추상화 벽

추상화 벽을 사용하면 세부 구현을 감추고 인터페이스를 제공하여 두뇌 용량의 한계를 극복(🤣)할 수 있다.

추상화 벽에 있는 함수를 사용할 때는 **구현을 전혀 몰라도 함수를 사용**할 수 있게 해준다.

### 추상화 벽의 한계를 시험해보기: 장바구니 데이터 구조 변경

장바구니를 자바스크립트 객체로 다시 만들어 효율적으로 추가하거나 삭제하도록 바꿔보자

(아래 코드들은 책의 코드와 모두 다릅니다.)

`add_item` 객체로 대응하는 코드로 변경하기

```tsx
// 기존 코드
function add_elemnt_last<T>(array: Array, item: T) {
	const new_array = [...array, item];
	return new_array;
}
function add_item(cart: Cart, item: Item) {
	return add_elemnt_last<Item>(cart, item);
}

// 변경된 코드
function objectSet<T, U>(object: T, key: string, value: U) {
	const new_object = { ...object, key: value } ;
	return new_object;
}
function add_item(cart: Cart, item: Item) {
	return objectSet<Cart, Item>(cart, item.name, item);
}
```

`calc_total` 객체로 대응하는 코드로 변경하기

```tsx

// 기존 코드
function calc_total(cart: Cart) {
	const total = cart.reduce((accumulator, currentValue) => {
		return accumulator + currentValue.price;
	}, 0);
	return total;
}

// 변경된 코드
function calc_total(cart: Cart) {
	const total = Object.keys(cart).reduce((accumulator, currentValue) => {
		return accumulator + currentValue.price;
	}, 0);
	return total;
}
```

`isInCart` 객체로 대응하는 코드로 변경하기

```tsx
// 기존 코드
function isInCart(cart: Cart, name: string) {
  return cart.some((cartItem) => {
	  return cartItem.name === name;
  }).length !== 0; 
}

// 변경된 코드
function isInCart(cart: Cart, name: string) {
  return name in cart;
}
```

`setPriceByName` 객체로 대응하는 코드로 변경하기

```tsx
// 기존 코드
// 배열 인덱스를 참조하지 않고 함수를 통해 가져오기
function setPriceByName(cart: Cart, name: string, price: number) {
  const i = indexOfItem(cart, name);
  if(i !== null) {
    const item = arrayGet(cart, i);
    return arraySet(cart, i, setPrice(item, price));
  }
  return cart;
}

function indexOfItem(cart: Cart, name: string) {
  for(var i = 0; i < cart.length; i++) {
   // 책의 코드..
    if(arrayGet(cart, i).name === name) {
      return i;
    }
  }
  return null;
}

function arrayGet(array: Array, idx: number) {
  return array[idx];
}

// 변경된 코드
// 궁금한 점: 책의 코드는 꽤 복잡하게 되어 있는데 이유가 무엇인지..?
function setPriceByName(cart: Cart, name: string, price: number) {  
  if (isInCart(cart, name)) {    
    return cart;
  } 
  
  return objectSet(cart, name, { name, price });
}
```

`remove_item_by_name` 객체로 대응하는 코드로 변경하기

```tsx
// 기존 코드
function remove_item_by_name(cart: Cart, name: string) {
	const idx = indexOfItem(cart, name);
	if (idx !== null) {
		cart.splice(idx, 1);
	}
}

function indexOfItem(cart: Cart, name: string) {
	const index = cart.findIndex((cartItem) => {
		return cartItem.name === name;
	});
	return index === -1 ? null : index;
}

// 변경된 코드
function remove_item_by_name(cart: Cart, name: string) {
	return objectDelete<Cart>(cart, name);
}

function objectDelete<T>(object: T, key: string) {
  const new_object = { ...object };
 
  delete new_object.key;
  return new_object;
}
```

여기서 중요한 점은 추상화 벽 위에 있는 함수가 데이터 구조를 몰라도 된다는 것을 말한다.
즉, **추상화 벽에 있는 함수만 사용하면 되고 장바구니 구현에 대해서는 신경쓰지 않아도 되는 것**이다.

### 추상화 벽을 사용하면 좋을 때

- 쉽게 구현을 바꾸기 위해 ⭐️⭐️
    - 만약을 대비하기 위한 코드는 구현을 하지 않는게 좋다.
- 코드를 읽고 쓰기 쉽게 만들기 위해 ⭐️
- 팀 간에 조율해야 할 것을 줄이기 위해
- 주어진 문제에 집중하기 위해 ⭐️⭐️⭐️

**🧶 리뷰: 추상화 벽으로 추상화 벽 아래에 있는 코드와 위에 있는 코드의 의존성을 없앨 수 있다.**

## 패턴3: 작은 인터페이스

**👊 새로운 요구사항**

- 장바구니에 제품을 100% 초과하고 시계를 구입하면 10% 할인

방법1. 추상화 벽에 만들기

```tsx
function getsWatchDiscount(cart: Cart) {
  const total = Object.keys(cart).reduce((accumulator, currentValue) => {
		return accumulator + currentValue.price;
	}, 0);
  
  return total > 100 && 'watch' in cart;
}
```

방법2. 추상화 벽 위에 만들기

```tsx
function getsWatchDiscount(cart: Cart) {
  const total = calcTotal(cart);
  const hasWatch = isInCart('watch');
  
  return total > 100 && hasWatch;
}
```

**방법1 과 방법 2중 어떤 것이 좋은가?**

방법1은 추상화 벽을 깨지는 않지만 구체적인 구현(총합 계산)을 신경쓰어야 한다. 구체적인 구현을 관리해야하는 포인트가 늘어나게 되어 추상화 벽의 장점을 약화시킨다.

여기서 **상위 계층에 만드는 것을 작은 인터페이스 패턴**이라고 부른다.

> ✨ 여기서 상위 계층에 만들었다는 것은 `getsWatchDiscount` 함수 자체에 구현이 빠지고 아래 계층의 함수들(`calcTotal`, `isInCart`)를 가져와 사용했기 때문에 상위 계층에 만드는 것이며 이를 작은 인터페이스로 볼 수 있는 것이다.

여기서 내가 느낀 점은 중복된 코드를 줄이는 것에 초점을 맞춘다기 보단 추상화 계층을 생각해서 나눠야한다는 마인드로 나눈게 핵심이라고 느꼈다.
> 

**👊 새로운 요구사항**

- 장바구니에 제품을 담을 때 로그를 남긴다.

어디에 로그를 남기는 함수를 넣는 것이 좋은 것인가?

```tsx
function add_item(cart: Cart, item: Item) {
  logAddToCart(globar_user_id, item);
	return objectSet<Cart, Item>(cart, item.name, item);
}
```

logAddToCart 함수는 액션이기 때문에 add_item함수가 액션으로 번지게 된다. 
**계산이었던 함수가 액션으로 바뀌게 되어 테스트에 제한**이 생기게 된다.

그러면 어디에 넣는 것이 좋은가?

**결국 사용자가 장바구니에 넣을때를 체크하고 싶은 로그이므로 이미 액션인 함수인 add_item_to_cart에 넣는 것이 좋은 설계방식이다. ⇒ 🩶**

(5장에서 개선한 코드)

```tsx
// 액션
function add_item_to_cart(cart: Cart, name: string, price: number) {
  shopping_cart = add_item(shopping_cart, name, price);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
  
  // 로그 기록하는 함수가 들어가야 할 위치
  logAddToCart();
}
```

**🧶 리뷰: 추상화 벽을 작게 만들어야 하고, 추상화 벽에 코드를 작게 만들어야 고쳐야할 것 들이 줄어든다.**

## 패턴4: 편리한 계층

이성적인 계층을 만드는 방법에 대한 패턴이다. 즉, 앞서 만들어둔 계층이 편리하다고 느끼는지 또는 별로 크게 도움이 되지 않는다고 느끼는지를 판단해 패턴을 다시 적용해보는 것이다.

## 그래프로 알 수 있는 코드에 대한 정보들

호출 그래프로 알 수 있는 비기능적 요구사항들

- **유지보수성:  요구사항이 바뀌었을때 가장 쉽게 고칠 수 있는 코드란?**
    - 위로 연결된 것이 적은 함수가 바꾸기 쉽다.
    - **자주 바뀌는 코드는 가능한 위쪽**에 있어야 한다. (아래쪽에 있는 코드를 바꾸면 영향도가 크다.)

- **테스트성: 어떤 것을 테스트하는 것이 가장 중요한 것인가?**
    - 위쪽으로 많이 연결된 함수를 테스트하는 것이 더 가치가 있다.
    - **아래쪽에 있는 함수를 테스트**하는 것이 **위쪽에 있는 함수를 테스트하는 것보다 가치**가 있다. (아래쪽에 있는 함수를 테스트하면 자연스럽게 위쪽에 있는 코드도 같이 검증이 된다.)

- **재사용성: 어떤 함수가 재사용하기 좋은가?**
    - 아래쪽에 함수가 적을수록 더 재사용하기 좋다.
    - **낮은 수준의 단계로 함수를 빼내면 재사용성이 더 높아**진다. (여러곳에 얽혀있는 코드는 재사용성이 떨어진다.)
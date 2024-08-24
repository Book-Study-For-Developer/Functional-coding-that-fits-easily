## 동작을 읽기, 쓰기 또는 둘 다로 분류하기

**읽기**

- 데이터에서 정보를 가져온다.
- 데이터를 바꾸지 않는다.

**쓰기**

- 데이터를 바꾼다.

## 카피-온-라이트 원칙의 세 단계

1. 복사본 만들기
2. 복사본 변경하기
3. 복사본 리턴하기

```tsx
function add_element_last<T>(array: Array, item: T) {
	const new_array = [...array, item];
	return new_array;
}
```

`add_element_last` 함수는 데이터를 바꾸지 않고 리턴했기 때문에 **읽기**이다.

### 카피-온-라이트 사용해서 쓰기를 읽기로 바꾸기

장바구니에서 제품의 이름이 같다면 제거하는 함수이다.

```tsx
function remove_item_by_name(cart: Cart, name: string) {
	let idx = null;
	cart.forEach((cart_item, i) => {
		if (cart_item.name === name) {
			idx = i
		}
	})
	if (idx !== null) {
		cart.splice(idx, 1);
	}
}
```

(filter로 하면.. 쉬을 것 같은데,, 일단 패스)

여기서 중요한 점은 **splice를 사용하면 cart가 변경**이 된다. 그렇게에 카피-온-라이트를 적용해보자.

```tsx
function remove_item_by_name(cart: Cart, name: string) {
	const new_cart = cart.slice();
	let idx = null;
	cart.forEach((cart_item, i) => {
		if (cart_item.name === name) {
			idx = i
		}
	})
	if (idx !== null) {
		new_cart.splice(idx, 1);
	}
	return new_cart;
}

// 나였다면..?
function remove_item_by_name(cart: Cart, name: string) {
	// name 과 같지 않는 것 빼고 리턴
	const new_cart = cart.filter((cart_item) => {
		return cart_item.name !== name;
	});
	return new_cart;
}

function delete_handler(name: string) {
	shopping_cart = remove_item_by_name(shopping_cart, name);
	const total = calc_total(shopping_cart);
	set_cart_total_dom(total);
	update_shipping_icons(shopping_cart);
	update_tax_dom(total);
}
```

### 연습해보기

```tsx
let mailing_list = [];

function add_concat(email: string) {
	mailing_list.push(email);
}
function submit_form_handler(event: Event) {
	const form = event.target;
	const email = form.elements["email"].value;
	add_concat(email);
}

// 카피-온-라이트 사용해서 리팩토링하기
// 책의 코드와는 다릅니다.
function add_concat(mailing_list: Array<string>, email: string) {
	const new_mailling_list = [...mailing_list, email];
	return new_mailing_list;
}

function submit_form_handler(event: Event) {
	const form = event.target;
	const email = form.elements["email"].value;
	mailing_list = add_concat(email);
}
```

## 쓰기 + 읽기도 하는 동작을 불변성 지키는 방법은?

1. 쓰기에서 읽기를 분리
2. 쓰기에 카피-온-라이트를 적용

Array.shift() 를 불변성을 유지하도록 바꿔보자.

```tsx
// shift 함수

// 카피-온-라이트 적용하기
function drop_first(array: Array) {
	const array_copy = array.slice();
	array_copy.shift();
	return array_copy;
}
function frist_element(array: Array) {
	return array[0];
}

function shift(array: Array) {
	return {
		first: frist_element(array),
		array: drop_first(array)
	};
}
```

### pop 변경해보기

```tsx

function drop_last(array: Array) {
	const array_copy = array.slice();
	array_copy.pop();
	return array_copy;
}
function last_element(array: Array) {
	return array[array.length - 1];
}
function pop(array: Array) {
	return {
		last: last_element(array),
		array: drop_last(array)
	};
}
```

## 읽기와 쓰기 ↔ 액션, 계산 데이터의 상관 관계

1. 변경 가능한 데이터를 읽는 것은 액션이다. ⭐️
2. 쓰기는 데이터를 변경 가능한 구조를 만든다. ⭐️
3. 어떤 데이터에 쓰기가 없다면 데이터는 변경 불가능한 데이터이다. ⭐️
4. 불변 데이터 구조를 읽는 것은 계산이다. ⭐️⭐️
5. 쓰기를 읽기로 바꾸면 코드에 계산이 많아진다. ⭐️⭐️⭐️

### 불변 데이터 구조를 사용하는 것이 맞을

1. 언제든 최적화 할 수 있다.
2. GC는 매우 빠르다
3. 생각보다 많이 복사하지 않는다.
4. 함수형 프로그래밍 언어에는 빠른 구현체가 있다.

## 객체에 대한 카피-온-라이트

배열과 동일하게 작업하면 되고, `slice`대신 `Object.assign`을 사용하면 된다.

## 생각보다 많이 복사하지 않는다고요?

아래의 코드를 보고 복사본이 몇개 생기는지 확인해보자.

```tsx
function setPriceByName(cart: Cart, name: string, price: number) {
	const cartCopy = cart.slice(); // 복사!
	
	// 책의 코드와 다릅니다.
  cartCopy.forEach((cartItem) => { 
		if (cartItem.name === name) {
			cartItem = setPrice(cartItem, price);
		}
	})
	
	return cartCopy;
}

function setPrice(item: Item; new_price: number) {
	const item_copy = Object.assign({}, item); // 복사!
	item_copy.price = new_price;
	return item_copy;
}

// 나였다면..?
// 그냥 복사도 안하고 바꿀 듯..?
function setPriceByName(cart: Cart, name: string, price: number) {
	const cartCopy = cart.map((cartItem) => { 
		if (cartItem.name === name) {
			return setPrice(cartItem, price);
		}
		return cartItem;
	})
	
	return cartCopy;
}

function setPrice(item: Item; new_price: number) {
	return { name: item.name, price: new_price };
}
```

### 얕은 복사를 그림으로 살펴보기

![image](https://github.com/user-attachments/assets/753914be-319d-47d9-aece-b25e4e1f7faa)

---

![image](https://github.com/user-attachments/assets/7a0792eb-d9de-4ba2-a6cd-6ff7bee0adca)

---

![image](https://github.com/user-attachments/assets/c4c605cd-42c3-4d4a-831c-70608b0130ee)

---

![image](https://github.com/user-attachments/assets/e239c02e-7ac5-45b5-8beb-48cb2a7a47a8)

---

복사본이 변경되지 않는 한 **구조적 공유**를 통해 바뀌지 않는 객체를 가르킨다.

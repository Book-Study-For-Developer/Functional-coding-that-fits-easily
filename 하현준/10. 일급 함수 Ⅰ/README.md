## 코드 냄새: 함수 이름에 있는 암묵적 인자

```tsx
function setPriceByName(cart, name, price) {
  var item = cart[name];
  var newItem = objectSet(item, 'price', price);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

function setShippingByName(cart, name, ship) {
  var item = cart[name];
  var newItem = objectSet(item, 'shipping', ship);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

function setQuantityByName(cart, name, quant) {
  var item = cart[name];
  var newItem = objectSet(item, 'quantity', quant);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

function setTaxByName(cart, name, tax) {
  var item = cart[name];
  var newItem = objectSet(item, 'tax', tax);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

function objectSet(object, key, value) {
  var copy = Object.assign({}, object);
  copy[key] = value;
  return copy;
}
```

위 코드는 냄새로 가득한 코드이다.

- 함수가 거의 비슷하게 생김
- 필드를 결정하는 문자열이 함수 이름에 붙어있음

즉, **함수 이름에 있는 암묵적 인자**때문에 냄새가 나는 것이다.

> 냄새를 맡는 법
> 
> 1. 함수 구현이 거의 똑같다.
> 2. 함수 이름이 구현의 차이를 만든다.
> 

## 리팩토링: 암묵적 인자를 드러내기

암묵적 인자를 드러내기 리팩토링 단계

1. 함수 이름에 있는 암묵적 인자를 확인한다.
2. 명시적인 인자를 추가한다.
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꾼다.
4. 함수를 부르는 곳을 고친다.

리팩터링을 진행해보자

```tsx
function setFieldByName(cart: Cart, name: string, field: string, value: string) {
	const item = cart[name];
	cosnt newItem = objectSet(item, filed, value);
	const newCart = objectSet(cart, name, newItem);
	
	return newCart;
}

// 리팩토링 후 사용 방법
cart = setFieldByName(cart, 'shoe', 'price', 13);
cart = setFieldByName(cart, 'shoe', 'quantity', 3);
cart = setFieldByName(cart, 'shoe', 'shipping', 0);
cart = setFieldByName(cart, 'shoe', 'tax', 2.34);

```

리팩토링을 통해 일반적인 함수 하나로 바꾸었다.

암묵적인 이름을 이제는 인자로 넘길수 있는 값(명시적인 값)로 되었고, 이를 일급이라고 부른다.

(그치만 여기서 문자열로 넘기는 건 위험할 수 있다.) ⇒ 타입스크립트를 사용하면 개선이 가능하긴 하다. 그래도 코드로 방어하는게 더 안전(타입스크립트는 런타임 에러를 막을 수 없기에,,)

## 일급인 것과 일급이 아닌 것을 구별하기

자바스크립트에서 일급이 아닌 것

- 수식 연산자
- 반복문
- 조건문
- try/catch 블록

일급으로 할 수 있는 것

1. 변수에 할당
2. 함수의 인자로 넘기기
3. 함수의 리턴값으로 받기
4. 배열이나 객체에 담기

### 필드명을 문자열로 사용하면 버그가 생기지 않을까?

정적 타입 시스템을 사용하면 컴파일 타임에 검사할 수 있다.

런타임을 통해 검사하는 방식으로 구현해보자

```tsx
const validItemFields = ['price', 'quantity', 'shipping', 'tax'];

function setFieldByName(cart: Cart, name: string, field: string, value: string) {
	if (!validItemFields.includes(field)) {
		throw 'Not a valid item field: ' + "'" + field + "'.";
	}
	const item = cart[name];
	cosnt newItem = objectSet(item, filed, value);
	const newCart = objectSet(cart, name, newItem);
	
	return newCart;
}

// 나였다면 타입스크립트로도 막았을 듯!
function setFieldByName(cart: Cart, name: string, field: keyof Cart, value: string) {
	if (!validItemFields.includes(field)) {
		throw 'Not a valid item field: ' + "'" + field + "'.";
	}
	const item = cart[name];
	cosnt newItem = objectSet(item, filed, value);
	const newCart = objectSet(cart, name, newItem);
	
	return newCart;
}

```

필드명을 외부에서 바뀌었다면 내부에서는 단순히 바꿔주기만 하면 된다.

```tsx
const validItemFields = ['price', 'quantity', 'shipping', 'tax'];
const translations = { 'quantity': 'number' };

function setFieldByName(cart: Cart, name: string, field: string, value: string) {
	if (!validItemFields.includes(field)) {
		throw 'Not a valid item field: ' + "'" + field + "'.";
	}
	
	if (translations.hasOwnProperty(field) {
		field = translations[field];
	}
	
	const item = cart[name];
	cosnt newItem = objectSet(item, filed, value);
	const newCart = objectSet(cart, name, newItem);
	
	return newCart;
}
```

필드명이 일급이기 때문에 가능한 방법이다. 

## 객체와 배열을 너무 많이 쓰는거 아냐?

자바스크립트에서는 객체가 해시 맵과 같은 기능을 해준다. 이때 중요한 것은 데이터를 사용할 때 임의의 인터페이스로 감싸지 않고 그대로 사용하고 있다는 점이다.

데이터를 데이터 그대로 사용하는 것의 중요한 장점은 여러가지 방법으로 해석할 수 있다는 점이다.(**데이터 지향**이라고 하는 중요한 원칙)

### 정적 타입 vs 동적 타입

정답은 없다. 둘 다 장단점이 있어 두 시스템을 충분히 이해하고 어느 것이 좋은지 가릴 수 없음을 이해하는 것이 중요하다.(🤣🤣🤣🤣🤣)

서로 좋고 나쁨을 가지고 있는 타입시스템이므로 어느것이 좋다고 말할 수는 없다. 
(그런거 고민할 시간에 내일을 위해 잠이나 자라 💤)

## 반복문 예제: 먹고 치우기

배열을 순회하는 일반적인 반복문인데, 비슷한 부분을 찾아 하나로 만들어보자

```tsx
// 준비하고 먹기
for(let i = 0; i < foods.length; i++) {
  cosnt food = foods[i];
  cook(food);
  eat(food);
}

// 설거자하기
for(let i = 0; i < dishes.length; i++) {
  const dish = dishes[i];
  wash(dish);
  dry(dish);
  putAway(dish);
}
```

여기서 냄새를 찾아보자,

1. 거의 똑같이 구현된 함수가 있다.
    1. 둘이 거의 동일한 방법으로 반복문 내부가 구현되어있다.
2. 함수 이름이 구현에 있는 다른 부분을 가르킨다.
    1. foods, dish가 서로 각각 다른 부분을 가르킴

```tsx
// 1. 동일한 부분을 items로 변경
function cookAndEatFoods() {
	for(let i = 0; i < foods.length; i++) {
	  cosnt item = foods[i];
	  cook(item);
	  eat(item);
	}
}

function cleanDishes() {
	for(let i = 0; i < dishes.length; i++) {
	  const item = dishes[i];
	  wash(item);
	  dry(item);
	  putAway(item);
	}
}

// 2. 외부에서 받아오도록 수정
function cookAndEatFoods<T>(array: T) {
	for(let i = 0; i < array.length; i++) {
	  cosnt item = array[i];
	  cook(item);
	  eat(item);
	}
}

function cleanDishes<T>(array: T) {
	for(let i = 0; i < array.length; i++) {
	  const item = array[i];
	  wash(item);
	  dry(item);
	  putAway(item);
	}
}

// 3. 함수 내부에 하는 일을 밖으로 뽑아내기
function cookAndEatFoods<T>(array: T) {
	for(let i = 0; i < array.length; i++) {
	  cosnt item = array[i];
	  cookAndEat(item);
	}
}

function cleanDishes<T>(array: T) {
	for(let i = 0; i < array.length; i++) {
	  const item = array[i];
	  clean(item);
	}
}

// 4. 내부에 쓰이는 구조가 매우 비슷하다.
function operateOnArray(array: T, f: U) {
	for(let i = 0; i < array.length; i++) {
		const item = array[i];
		f(item);
	}
}

operateOnArray<Array<Food>, (item: Food) => void>(foods, cookAndEat);
operateOnArray<Array<Dish>, (item: Dish) => void>(dishes, clean);

// 5. 함수의 이름을 일반적인 고차함수로 변경하기
function forEach(array: T, f: U) {
	for(let i = 0; i < array.length; i++) {
		const item = array[i];
		f(item);
	}
}
```

최종적으로 만들어진 함수 `forEach` 는 결국 고차함수로 변경되었다.

리팩터링을 진행했던 단계를 정리해보자

1. 코드를 함수로 감싸기
2. 더 일반적인 이름으로 바꾸기
3. 암묵적 인자를 드러내기
4. 함수 추출하기
5. 암묵적 인자를 드러내기

## 함수 본문을 콜백으로 바꾸기

```tsx
try {
  saveUserData(user);
} catch(error: unknown) {
	logToSanpErrors(error);
}

try {
  fetchProduct(productId);
} catch(error: unknown) {
	logToSanpErrors(error);
}

// saveUserData, fetchProduct 부분을 제외하고는 모두 동일하다.
```

두 코드의 앞부분과 뒷부분은 바뀌지 않는다. 즉 앞뒤를 구분하면 재사용하여 바꿀 수 있다.

1. 본문과 본문의 앞, 뒤를 구분한다.
2. 전체를 함수로 뺀다.
3. 본문 부분을 빼낸 함수의 인자로 전달한 함수로 바꾼다.

앞 뒤 사이에 있는 본문을 인자로 빼내 분리해보자.

```tsx
function withLogging<T>(f: T) {
	try {
		f();
	} catch (error) {
		logToSanpErrors(error);
	}
}

withLogging<VoidFunction>(function() { 
	saveUserData(user);
});
```

위의 문법은 인라인으로 정의한 방법이다. 함수를 전달하는 방법은 세 가지가 있다.

1. 전역으로 정의하기
    1. 외부에 함수를 선언해서 넘기는 방법
2. 지역적으로 정의하기
    1. 함수 내부에 함수를 선언해서 넘기는 방법
3. 인라인으로 정의하기
    1. 함수를 직접 넘기는 방법

> 함수에 데이터값으로 전달하지 않고 함수를 전달하는 이유?
> 
> 함수에 일반적인 데이터를 넘기게 되면 try/catch가 아무 소용이 없기 때문이다.
> 
> 
> ```tsx
> function withLogging(data) {
> 	try {
> 		data;
> 	} catch (error) {
> 		logToSanpErrors(error);
> 	}
> }
> 
> withLogging<VoidFunction>(saveUserData(user));
> ```
> 
> try/catch 이외에도 특정 문맥 안에서 실행되야 하기 위해서는 데이터가 아닌 함수를 넘겨야한다.
>
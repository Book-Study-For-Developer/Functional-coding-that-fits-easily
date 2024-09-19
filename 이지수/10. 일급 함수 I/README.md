코드의 냄새: 함수 이름에 있는 암묵적 인자

- 거의 똑같이 구현된 함수가 있다
- 함수 이름이 구현에 있는 다른 부분을 가리킨다

리팩토링: 암묵적 인자를 드러내기

1. 함수 이름에 잇는 암묵적 인자를 확인한다
2. 명시적인 인자를 추가한다
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꾼다
4. 함수를 호출하는 곳을 고친다

리팩토링: 함수 본문을 콜백으로 바꾸기

1. 함수 본문에서 바꿀 부분의 앞부분과 뒷부분을 확인한다
2. 리팩토링 할 코드를 함수로 빼낸다
3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 빼낸다

## 코드의 냄새: 함수 이름에 있는 암묵적 인자

```tsx
function setPriceByName(cart, name, price) {
  var item = cart[name];
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
```

위의 함수들은 거의 똑같이 생겼고, 필드를 결정하는 문자열이 함수 이름에 있다

→ **함수 이름에 있는 암묵적 인자**

냄새 맡는 법

1. 함수 구현이 거의 똑같다
2. 함수 이름이 구현의 차이를 만든다

함수 이름에서 서로 다른 부분이 암묵적 인자이다

## 리팩터링: 암묵적 인자를 드러내기

1. 함수 이름에 있는 암묵적 인자 확인하기
2. 명시적인 인자를 추가하기
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꾸기
4. 함수를 부르는 곳 고치기

```tsx
// 리팩터링 전
function setPriceByName(cart, name, price) {
  var item = cart[name];
  var newItem = objectSet(item, "price", price);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

cart = setPriceByName(cart, "shoe", 13);
cart = setQuantityByName(cart, "shoe", 3);
cart = setShippingByName(cart, "shoe", 0);
cart = setTaxByName(cart, "shoe", 2.34);

// 리팩터링 후
function setFieldByName(cart, name, field, value) {
  var item = cart[name];
  var newItem = objectSet(item, field, value);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

cart = setFieldByName(cart, "shoe", "price", 13);
cart = setFieldByName(cart, "shoe", "quantity", 3);
cart = setFieldByName(cart, "shoe", "shipping", 0);
cart = setFieldByName(cart, "shoe", "tax", 2.34);
```

리팩터링 전에는 필드명이 함수 이름에 암묵적으로 있었는데,

리팩터링 후 암묵적인 이름은 인자로 넘길 수 있는 값이 되었고, 이를 일급이라고 부른다

⚠️ 필드명을 문자열로 넘기는 것은 안전하지 않을 수 있다.

## 일급인 것과 일급이 아닌 것을 구별하기

자바스크립트에서 일급이 아닌 것

- 수식 연산자
- 반복문
- 조건문
- try/catch 블록

일급으로 할 수 있는 것

- 변수에 할당
- 함수의 인자로 넘기기
- 함수의 리턴값으로 받기
- 배열이나 객체에 담기

### 필드명을 문자열로 사용하면 버그가 생기지 않을까요?

방법1: 컴파일 타임에 검사

- 타입스크립트와 같은 정적 타입 시스템 언어 사용

방법2: 런타임에 검사

```tsx
var validItemFields = ["price", "quantity", "shipping", "tax"];

function setFieldByName(cart, name, field, value) {
  if (!validItemFields.includes(field))
    throw "Not a valid item field: " + "'" + field + "'.";
  var item = cart[name];
  var newItem = objectSet(item, field, value);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

function objectSet(object, key, value) {
  var copy = Object.assign({}, object);
  copy[key] = value;
  return copy;
}
```

만약 내부에서 정의한 필드명이 바뀐다고 해도 사용하는 사람들은 원래의 필드명을 그대로 사용하고 내부에서 바꿔주면 된다

```tsx
var validItemFields = ["price", "quantity", "shipping", "tax", "number"];
var translations = { quantity: "number" };

function setFieldByName(cart, name, field, value) {
  if (!validItemFields.includes(field))
    throw "Not a valid item field: '" + field + "'.";
  if (translations.hasOwnProperty(field)) field = translations[field];
  var item = cart[name];
  var newItem = objectSet(item, field, value);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}
```

필드명이 객체나 배열에 담을 수 있는 일급이기 때문에 가능한 방법이다

## 객체와 배열을 너무 많이 쓰게 됩니다

자바스크립트에서는 객체가 해시 맵과 같은 기능을 해준다

데이터를 사용할 때 임의의 인터페이스로 감싸지 않고 그대로 사용한다는 점이 중요하다. 이것의 장점은 여러가지 방법으로 해석할 수 있다는 점이다.
미래에 어떤 방법으로 해석될지 미리 알수 없기 때문에 필요에 따라 알맞은 방법으로 해석할 수 있어야 함

**데이터 지향**

이벤트와 엔티티에 대한 사실을 표현하기 위해 일반 데이터 구조를 사용하는 프로그래밍 형식

## 어떤 문법이든 일급 함수로 바꿀 수 있습니다

일급이 아닌 + 연산자를 함수로 만들어서 일급 값으로 만들수 있다

```tsx
function plus(a, b) {
  return a + b;
}
```

## 반복문 예제: 먹고 치우기

배열을 순회하는 일반적인 반복문에서 문법적으로 비슷한 부분을 찾아 하나로 만들기

```tsx
// 준비하고 먹기
for (var i = 0; i < foods.length; i++) {
  var food = foods[i];
  cook(food);
  eat(food);
}

// 설거지하기
for (var i = 0; i < dishes.length; i++) {
  var dish = dishes[i];
  wash(dish);
  dry(dish);
  putAway(dish);
}
```

함수 이름에 있는 암묵적 인자 냄새의 특징

1. 거의 똑같이 구현된 함수가 있다
2. 함수 이름이 구현에 있는 다른 부분을 가리킨다

리팩터링 단계

1. 함수 이름에 있는 암묵적 인자를 확인
2. 명시적인 인자를 추가
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꿈
4. 함수를 호출하는 곳을 고침

```tsx
// 원래 코드
for (var i = 0; i< foods.length? i++) {
	var food = foods[i];
	cook(food)
	eat(food);
}

for(var i = 0l i< dishes.length; i++){
	var dish = dishes;
	wash(dish);
	dry(dish);
	putaway(dish);
}

// forEach사용하기
forEach(foods, function(food){
	cook(food);
	eat(food);
})

forEach(dishes, function(dish) {
	wash(dish);
	dry(dish);
	putAway(dish);
})
```

forEach()함수는 배열 전체를 순회할 수 있는 완전한 반복문이고 고차함수임

반복문을 작성할 때 사용했던 패턴을 그대로 담고 있어 반복문 대신 forEach()를 사용하면 된다

리팩터링 단계

1. 코드를 함수로 감싸기
2. 더 일반적인 이름으로 바꾸기
3. 암묵적 인자를 드러내기
4. 함수 추출하기
5. 암묵적 인자를 드러내기

## 리팩터링: 함수 본문을 콜백으로 바꾸기

```tsx
try {
  saveUserData(user);
} catch (error) {
  logToSnapErrors(error);
}

// 함수로 빼낸 코드
function withLogging() {
  try {
    saveUserData(user);
  } catch (error) {
    logToSnapErrors(erorr);
  }
}
withLogging();

// 콜백으로 빼낸 코드 - 인라인으로 함수 정의
function withLogging() {
  try {
    f();
  } catch (error) {
    logToSnapErrors(error);
  }
}

withLogging(function () {
  saveUserData(user);
});
```

함수 본문을 콜백으로 바꾸기 단계

1. 본문과 본문의 앞부분과 뒷부분을 구분하기
2. 전체를 함수로 빼내기
3. 본문 부분을 빼낸 함수의 인자로 전달한 함수로 바꾸기

함수를 정의하고 전달하는 세가지 방법

1. 전역으로 정의하기

   ```tsx
   function saveCurrentUserData() {
     saveUserData(user);
   }

   withLogging(saveCurrentUserData);
   ```

2. 지역으로 정의하기

   ```tsx
   function someFunction() {
     let saveCurrentUserData = function () {
       saveUserData(user);
     };
     withLogging(saveCurrentUserData);
   }
   ```

3. 인라인으로 정의하기

   ```tsx
   withLogging(function () {
     saveUserData(user);
   });
   ```

## 왜 본문을 함수로 감싸서 넘기나요?

코드가 바로 실행되면 안되기 떄문

실행을 미루고 withLogging()함수에서 만든 try/catch 문맥 안에서 실행하도록 코드를 감싼 것

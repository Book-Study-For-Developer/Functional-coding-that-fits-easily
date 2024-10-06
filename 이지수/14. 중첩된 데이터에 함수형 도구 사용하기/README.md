## 객체를 다루기 위한 고차함수

중첩된 장바구니 제품 객체에 값을 바꾸기

제품 크기나 수량을 늘리거나 줄이는 동작에 생긴 많은 중복

## 필드명을 명시적으로 만들기

```tsx
function incrementQuantity(item) {
  var quantity = item["quantity"];
  var newQuantity = quantity + 1;
  var newItem = objectSet(item, "quantity", newQuantity);
  return newItem;
}

function incrementSize(item) {
  var size = item["size"];
  var newSize = size + 1;
  var newItem = objectSet(item, "size", newSize);
  return newItem;
}

/// After expressing argument

function incrementField(item, field) {
  var value = item[field];
  var newValue = value + 1;
  var newItem = objectSet(item, field, newValue);
  return newItem;
}

function decrementField(item, field) {
  var value = item[field];
  var newValue = value - 1;
  var newItem = objectSet(item, field, newValue);
  return newItem;
}

function doubleField(item, field) {
  var value = item[field];
  var newValue = value * 2;
  var newItem = objectSet(item, field, newValue);
  return newItem;
}

function halveField(item, field) {
  var value = item[field];
  var newValue = value / 2;
  var newItem = objectSet(item, field, newValue);
  return newItem;
}
```

## update()도출하기

함수 이름에 있는 암묵적인 인자는 암묵적 인자를 드러내기 리팩터링으로 동작 이름을 명시적인 인자로 변경

함수 본문을 콜백으로 바꾸기 리팩터링으로 동작을 함수 인자로 받도록 수정

```tsx
function incrementField(item, field) {
  var value = item[field];
  var newValue = value + 1;
  var newItem = objectSet(item, field, newValue);
  return newItem;
}

/// Extracted

function incrementField(item, field) {
  return updateField(item, field, function (value) {
    return value + 1;
  });
}

function updateField(item, field, modify) {
  var value = item[field];
  var newValue = modify(value);
  var newItem = objectSet(item, field, newValue);
  return newItem;
}

/// Called update()

function update(object, key, modify) {
  var value = object[key];
  var newValue = modify(value);
  var newObject = objectSet(object, key, newValue);
  return newObject;
}
```

## 값을 바꾸기 위해 update()사용하기

요구사항: 직원 데이터가 있고 직원의 월급을 10퍼 올려주기

```tsx
const employee = {
  name: "Kim",
  salary: 120000,
};

function raise10Percent(salary) {
  return salary * 1.1;
}

update(employee, "salary", raise10Percent);
```

중첩된 문맥 안에 있는 값에 함수를 적용

## 리팩터링: 조회하고 변경하고 설정하는 것을 update()로 교체하기

```tsx
// 리팩터링 전
function incrementField(item, field) {
  var value = item[field]; // 조회
  var newValue = value + 1; // 바꾸기
  var newItem = objectSet(item, field, newValue); // 설정
  return newItem;
}

// 리팩터링 후
function incrementField(item, field) {
  return update(item, field, function (value) {
    return value + 1;
  });
}

// 단게 1: 조회하고 바꾸고 설정하는 것 찾기
function halveField(item, field) {
  var value = item[field];
  var newValue = value / 2;
  var newItem = objectSet(item, field, newValue);
  return newItem;
}

// 단계 2: update()로 교체
function halveField(item, field) {
  return update(item, field, function (value) {
    return value / 2;
  });
}
```

조회하고 변경하고 설정하는 것을 update()로 교체하기는 중첩된 객체에 적용하기 좋다

## 함수형 도구: update()

```tsx
function update(object, key, modify) {
  var value = object[key];
  var newValue = modify(value);
  var newObject = objectSet(object, key, newValue);
  return newObject;
}

function incrementField(item, field) {
  return update(item, field, function (value) {
    return value + 1;
  });
}
```

## 중첩된 데이터에 update() 사용하기

조회하고 변경하고 설정하는 것을 update로 교체하기

```tsx
function incrementSize(item) {
  var options = item.options;
  var size = options.size;
  var newSize = size + 1;
  var newOptions = objectSet(options, "size", newSize);
  var newItem = objectSet(item, "options", newOptions);
  return newItem;
}

// update로 바꿈
function incrementSize(item) {
  var options = item.options;
  var newOptions = update(options, "size", increment);
  var newItem = objectSet(item, "options", newOptions);
  return newItem;
}

// 두번 리팩토링한 코드
function incrementSize(item) {
  return update(item, "options", function (options) {
    return update(options, "size", increment);
  });
}
```

중첩된 객체에 중첩된 update를 사용할 수 있으며 update()를 중첩해 부르면서 더 깊은 단계로 중첩된 객체에도 사용할 수 있다

## updateOption()도출하기

```tsx
function incrementSize(item) {
  return update(item, "options", function (options) {
    return update(options, "size", increment);
  });
}

// 함수 이름에 암묵적 인자를 사용함(increment, size)
function incrementOption(item, option) {
  return update(item, "options", function (options) {
    return update(options, option, increment);
  });
}

// 명시적으로 변경
function updateOption(item, option, modify) {
  return update(item, "options", function (options) {
    return update(options, option, modify);
  });
}
```

아직 ‘options’라는 암묵적 인자가 있다

## update2()도출하기

```tsx
function update2(object, key1, key2, modify) {
  return update(object, key1, function (value1) {
    return update(value1, key2, modify);
  });
}
```

update2사용하기

```tsx
function incrementSize(item) {
  return size + 1;
}
```

## incrementSizeByName()을 만드는 네가지 방법

옵션1: update()와 incrementSize()로 만들기

```tsx
function incrementSizeByName(cart, name) {
  return update(cart, name, incrementSize);
}
```

옵션2: update()와 update2()로 만들기

```tsx
function incrementSizeByName(cart, name) {
  return update(cart, name, function (item) {
    return update2(item, "options", "size", function (size) {
      return size + 1;
    });
  });
}
```

옵션3: update()로 만들기

```tsx
function incrementSizeByName(cart, name) {
  return update(cart, name, function (item) {
    return update(item, "options", function (options) {
      return update(options, "size", function (size) {
        return size + 1;
      });
    });
  });
}
```

옵션4: 조회하고 바꾸고 설정하는 것을 직접 만들기

```tsx
function incrementSizeByName(cart, name) {
  var item = cart[name];
  var options = item.options;
  var size = options.size;
  var newSize = size + 1;
  var newOptions = objectSet(options, "size", newSize);
  var newItem = objectSet(item, "options", newOptions);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}
```

## update3() 도출하기

```tsx
function update3(object, key1, key2, key3, modify) {
  return update(object, key1, function (object2) {
    return update2(object2, key2, key3, modify);
  });
}
```

## nestedUpdate() 도출하기

updateX()를 만들려고 한다면 update()안에 updateX-1()을 불러주면 됨

```tsx
function updateX(object, depth, key1, key2, key3, modify) {
  return update(object, key1, function (value1) {
    return updateX(value1, depth - 1, key2, key3, modify);
  });
}

// 키의 개수와 순서를 지켜서 사용해야함
function updateX(object, keys, modify) {
  var key1 = keys[0];
  var restOfKeys = drop_first(keys);
  return update(object, key1, function (value1) {
    return updateX(value1, restOfKeys, modify);
  });
}

// 중첩된 객체 키를 모두 돌았으면 멈춰야 함
function updateX(object, keys, modify) {
  if (keys.length === 0) return modify(object);
  var key1 = keys[0];
  var restOfKeys = drop_first(keys);
  return update(object, key1, function (value1) {
    return updateX(value1, restOfKeys, modify);
  });
}
```

바꿀 값이 있는 키들만 알면 된다

```tsx
function nestedUpdate(object, keys, modify) {
  if (keys.length === 0) return modify(object);
  var key1 = keys[0];
  var restOfKeys = drop_first(keys);
  return update(object, key1, function (value1) {
    return nestedUpdate(value1, restOfKeys, modify);
  });
}
```

## 안전한 재귀 사용법

1. 종료 조건

   재귀가 멈춰야 하는 곳에 종료 조건이 필요

2. 재귀 호출

   최소 하나이 재귀 호출이 있어야 함

3. 종료 조건에 다가가기

   최소 하나 이상의 인자가 점점 줄어들어 종료 조건에 가까워져야 함

## 재귀 함수가 적합한 이유

원래 배열을 반복할 때

: 배열의 처음부터 끝까지 순서대로 처리하고 결과 배열에 처리한 항목을 추가

중첩된 데이터를 다룰 때

: 점점 아래 단계로 내려가면서 최종값에 도착하면 값을 변경하고, 나오면서 새로운 값을 설정

## 깊이 중첩된 구조를 설계할 때 생각할 점

깊이 중첩된 데이터에 nestedDate()를 쓰려면 긴 경로가 필요하다

but,

키 경로가 길면 중간 객체가 어떤 키를 가졌는지 기억하기 어려움

직접 만든 데이터가 아니라 API를 사용할 경우 더 어려움

기억해야할 것이 너무 많을 때 **추상화 벽**을 사용하면 구체적인 것을 몰라도 됨

## 깊이 중첩된 데이터에 추상화 벽 사용하기

추상화벽을 사용하여 같은 작업을 하면서 알아야할 데이터를 줄일 수 있다

추상화 벽에 함수를 만들고, 의미 있는 이름을 붙이기

```tsx
// 주어진 ID로 블로그 변경하는 함수
function updatePostById(category, id, modifyPost) {
  return nestedUpdate(category, ["posts", id], modifyPost);
}

// 글쓴이를 수정하는 함수
function updateAuthor(post, modifyUser) {
  return update(post, "author", modifyUser);
}

// 사용자 이름을 대문자로 바꾸는 함수
function capitalizeName(user) {
  return update(user, "name", capitalize);
}

// 모두 합치면
updatePostById(blogCategory, "12", function (post) {
  return updateAuthor(post, capitalizeUserName);
});
```

좋아진 점

1. 기억해야할 것 갯수 줄어들었음 4 → 3
2. 동작의 이름이 있으므로 각각의 동작을 기억하기 쉬움

## 앞에서 배운 고차 함수들

- 배열을 반복할 때 for 반복문 대신 사용하기
- 중첩된 데이터를 효율적으로 다루기
- 카피-온-라이트 원칙 적용하기
- try/catch 로깅 규칙을 코드화

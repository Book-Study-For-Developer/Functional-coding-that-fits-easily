# 14. 중첩된 데이터에 함수형 도구 사용하기

- [조회하고 변경하고 설정하는 것을 update()로 교체하기](#조회하고-변경하고-설정하는-것을-update로-교체하기)
- [중첩된 데이터에 update() 사용하기](#중첩된-데이터에-update-사용하기)
- [nestedUpdate() 도출하기](#nestedupdate-도출하기)
- [앞에서 배운 고차 함수들](#앞에서-배운-고차-함수들)

## 조회하고 변경하고 설정하는 것을 update()로 교체하기

1. 값을 조회하고, 바꾸고, 설정하는 것을 찾음
2. 바꾸는 동작을 콜백으로 전달해 update()로 교체

```tsx
// 객체, 바꿀 값의 위치(키), 바꾸는 동작 함수
function update(object, key, modify) {
  var value = object[key]; // 조회
  var newValue = modify(value); // 바꾸기
  var newObject = objectSet(object, key, newValue); // 설정 (복사본 생성)
  return newObject; // 바꾼 객체 리턴
}
```

---

## 중첩된 데이터에 update() 사용하기

```tsx
var shirt = {
  name: "shirt",
  price: 13,
  options: {
    color: "blue",
    size: 3,
  },
};
```

```tsx
function incrementSize(item) {
  var options = item.options; // 조회
  var size = options.size; // 조회
  var newSize = size + 1; // 변경
  var newOptions = objectSet(options, "size", newSize); // 설정
  var newItem = objectSet(item, "options", newOptions); // 설정
  return newItem;
}
```

```tsx
function incrementSize(item) {
  var options = item.options; // 조회
  // 조회, 변경, 설정을 update()로 바꿈
  var newOptions = update(options, "size", increment); // 변경
  var newItem = objectSet(item, "options", newOptions); // 설정
  return newItem;
}
```

```tsx
function incrementSize(item) {
  // 중첩 update 사용 - 중첩된 단계만큼 사용
  return update(item, "options", function (options) {
    return update(options, "size", increment);
  });
}
```

```tsx
// 암묵적 인자 'increment', 'size'를 명시적으로 빼냄
function updateOption(item, option, modify) {
  return update(item, "options", function (options) {
    return update(options, option, modify);
  });
}

// 암묵적 인자 'option' 빼냄. update 2번 중첩
function update2(object, key1, key2, modify) {
  return update(object, key1, function (value1) {
    return update(value1, key2, modify);
  });
}
```

```tsx
function incrementSize(item) {
  return update2(item, "options", "size", function (size) {
    return size + 1;
  });
}
```

---

## nestedUpdate() 도출하기

```tsx
function update3(object, key1, key2, key3, modify) {
  return update(object, key1, function (value1) {
    return update2(value1, key2, key3, modify);
  });
}

function update4(object, key1, key2, key3, key4, modify) {
  return update(object, key1, function (value1) {
    return update3(value1, key2, key3, key4, modify);
  });
}
```

```tsx
function nestedUpdate(object, keys, modify) {
  // 종료 조건 (경로의 길이가 0일 때)
  if (keys.length === 0) return modify(object);
  // 첫 번째 키로 update()를 호출
  var key1 = keys[0];
  // 나머지 키로 재귀 함수 호출
  // 항목을 하나씩 없애며 종료 조건에 가까워짐
  var restOfKeys = drop_first(keys);
  return update(object, key1, function (value1) {
    return nestedUpdate(value1, restOfKeys, modify);
  });
}
```

- 안전한 재귀 사용법
  1. 종료 조건 필요
  2. 재귀 호출하기
  3. 종료 조건에 다가가기

---

## 앞에서 배운 고차 함수들

- 배열을 반복할 때 for 반복문 대신 사용하기 - `forEach()`, `map()`, `filter()`, `reduce()`
- 중첩된 데이터를 효율적으로 다루기 - `update()`, `nestedUpdate()`
- 카피-온-라이트 원칙 사용하기 - `withArrayCopy()`, `withObjectCopy()` 사용하면 카피-온-라이트 맥락 안에서 어떤 동작(콜백)이든 수행 가능
- try/catch 로깅 규칙을 코드화 - `wrapLogging()`은 어떤 함수에 다른 행동이 추가된 함수로 바꿔주는 예제

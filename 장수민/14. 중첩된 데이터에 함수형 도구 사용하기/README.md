## 중첩된 데이터에 함수형 도구 사용하기

중첩된 구조는 복합적인 데이터 구조를 표현하기 위해 자주 사용하지만, 중첩된 데이터를 불변 데이터로 다루는 것은 어렵다. 이번 장에서 배우도록 하자!

### 필드명을 명시적으로 만들기

비슷한 함수 두 개를 보면,

```ts
function incrementQuantity(item) {
  const quantity = item["quantity"];
  const newQuantity = quantity + 1;
  const newItem = objectSet(item, "quantity", newQuantity);
  return newItem;
}

function incrementSize(item) {
  const size = item["size"];
  const newSize = size + 1;
  const newItem = objectSet(item, "size", newSize);
  return newItem;
}
```

함수 이름에 있는 필드명을 통해, **함수 이름에 있는 암묵적 인자**냄새가 있다는 사실을 알 수 있다. **암묵적 인자 드러내기** 리팩터링을 수행하면 다음과 같은 하나의 함수를 얻을 수 있다.

```ts
function incrementField(item, field) {
  const value = item[field];
  const newValue = value + 1;
  const newItem = objectSet(item, field, newValue);
  return newItem;
}
```

그런데, 또 increment처럼 동작을 나타내는 이름이 함수에 들어가있으면서, 생김새도 비슷한 함수들(decrement, double, halve)이 많다. 중복을 없애보자!

### update() 도출하기

**암묵적 인자 드러내기** 리팩터링을 진행하여 동작 이름을 명시적인 인자로 바꾸고 싶지만, 동작은 일반값이 아니라 동작이므로 **함수 본문을 콜백으로 바꾸기** 리팩터링을 진행해야 한다.

```ts
function incrementField(item, field) {
  return update(item, field, function (value) {
    return value + 1;
  });
}

// 처음에는 updateField라는 함수명을 가지고 있었지만 특정 필드를 바꾸는 함수가 아니라서 field는 제외
function update(item, field, modify) {
  const value = item[field];
  const newValue = modify(value);
  const newItem = objectSet(item, field, newValue); // 카피-온-라이트 원칙을 따름
  return newItem;
}

function objectSet(object, key, value) {
  const copy = Object.assign({}, object);
  copy[key] = value;
  return copy;
}
```

### 리팩터링: 조회하고 변경하고 설정하는 것을 update()로 교체하기

```ts
// 리팩터링 전
function incrementField(item, field) {
  const value = item[field]; // 조회
  const newValue = modify(value); // 변경
  const newItem = objectSet(item, field, newValue); // 설정
  return newItem;
}

// 리팩터링 후
function incrementField(item, field) {
  return update(item, field, function (value) {
    return value + 1;
  });
}
```

halve라는 동작에 대해 동일한 리팩터링을 수행하면,

```ts
// 리팩터링 전
function halveField(item, field) {
  const value = item[field]; // 조회
  const newValue = value / 2; // 변경
  const newItem = objectSet(item, field, newValue); // 설정
  return newItem;
}

// 리팩터링 후
function halveField(item, field) {
  return update(item, field, function (value) {
    return value / 2; // 바꾸는 동작을 콜백으로 전달함.
  });
}
```

### 연습 문제 - update()를 사용해 user 레코드에 있는 사용자 이메일 주소에 lowercase() 적용하기

```ts
const user = {
  firstName: "Joe",
  lastName: "Nash",
  email: "JOE@EXAMPLE.COM",
};

function update(object, key, modify) {
  const value = object[key];
  const newValue = modify(value);
  const newObject = objectSet(object, key, newValue);
  return newObject);
}

update(user, 'email', lowercase);
```

---

### 중첩된 update 시각화하기

다음과 같이 중첩된 객체 구조는 충분히 존재할 수 있다.

```ts
const shirt = {
  name: "shirt",
  price: 13,
  options: {
    color: "blue",
    size: 3,
  },
};
```

이 때, 중첩된 options 객체를 다루는 함수를 한 줄씩 살펴보자면,

```ts
function incrementSize(item) {
  const options = item.options; // 조회
  const size = item.size; // 조회
  const newSize = size + 1; // 변경
  const newOptions = objectSet(options, "size", newSize); // 설정
  const newItem = objectSet(item, "options", newOptions); // 설정
  return newItem;
}
```

### 중첩된 데이터에 update() 사용하기

가운데 있는 조회, 변경, 설정에 리팩터링을 적용해보면

```ts
function incrementSize(item) {
  const options = item.options; // 조회
  const newOptions = update(options, "size", increment); // 변경
  const newItem = objectSet(item, "options", newOptions); // 설정
  return newItem;
}

// 위 함수에서 다시 또 조회, 변경, 설정의 한 묶음이 생겼으므로 리팩터링을 한 번 더 진행하면
function incrementSize(item) {
  return update(item, "options", function (options) {
    return update(options, "size", increment);
  });
}
```

### updateOption() 도출하기

update() 안에서 update() 를 호출하는 코드를 일반화해서 `updateOption()`을 만들 수 있다.

그 전에, 코드 안에 냄새(_size라고 하는 구체적인 필드명과 increment라는 구체적인 동작명을 인자로 넘김_)를 제거해야 한다.

```ts
// 명시적인 option 인자와 modify 인자를 넘기면,
function updateOption(item, option, modify) {
  return update(item, "options", function (options) {
    return update(options, option, modify);
  });
}
```

하지만, 여전히 options라고 하는 구체적인 필드명을 사용하고 있으므로 **암묵적 인자 드러내기** 리팩터링을 진행하자.

### update2() 도출하기

두 번 중첩되었다는 사실을 일반적인 형태로 나타내기 위해서 update2라는 이름으로 함수를 작성하고, 리팩터링을 진행하면 다음과 같아진다.

```ts
function update2(object, key1, key2, modify) {
  return update(object, key1, function (value) {
    return update(value1, key2, modify);
  });
}

// update2() 함수를 적용하면,
function incrementSize(item) {
  return update2(item, "options", "size", function (size) {
    return size + 1;
  });
}
```

### 3번 중첩된 형태일 때 update3() 도출하기

```ts
// 리팩터링 전
function incrementSizeByName(cart, name) {
  return update(cart, name, function (item) {
    return update2(item, "options", "size", function (size) {
      return size + 1;
    });
  });
}

// '암묵적 인자 드러내기' 리팩터링 후
function incrementSizeByName(cart, name) {
  return update3(cart, name, "options", "size", function (size) {
    return size + 1;
  });
}

function update3(object, key1, key2, key3, modify) {
  return update(object, key1, function (object2) {
    return update2(object2, key2, key3, modify);
  });
}
```

### nestedUpdate() 도출하기

update2, update3, ... 처럼 계속 만들다보면 패턴이 보인다.

> [!NOTE] 패턴
> `updateX()`를 만들려고 한다면 update() 안에 updateX-1()을 불러주면 된다.
> 단, update1()을 만들려고 하면 update() 안에 update0()를 부르게 되는데 update0는 modify()를 그냥 호출하는 함수가 된다!

```ts
function updateX(object, depth, key1, key2, key3, modify) {
  return update(object, key1, function (value1) {
    return updateX(value1, depth - 1, key2, key3, modify); // 재귀 호출
  });
}
```

하지만, 이렇게 만들면 깊이를 나타내는 depth와 실제 키 개수가 달라지는 문제가 생겨서 배열 자료 구조가 필요하다.

````ts
function updateX(object, keys, modify) {
  if (keys.length === 0) return modify(object); // 재귀의 탈출 조건이자 update0 함수를 대체

  const key1 = keys[0];
  const restOfKeys = drop_first(keys); // 하나씩 놔두고 계속 재귀를 돌리는 형식
  return update(object, key1, function (value1) {
    return updateX(value1, restOfKeys, modify);
  });
}

이제 updateX 함수 이름을 nestedUpdate로 바꾸면 훨씬 더 일반적이게 된다! 😀

### 깊이 중첩된 구조를 설계할 때 생각할 점

깊이 중첩된 데이터에 nestedUpdate()를 쓰려면 긴 키 경로가 필요하다. 그런데, 키 경로가 길면 중간 객체가 어떤 키를 가졌는지 기억하기 어렵다.

```ts
// blog 분류에 있는 12번째 글을 가져와 글쓴이 이름을 대문자로 바꾸는 일을 하는 API
httpGet("http://my-blog.com/api/category/blog", function (blogCategory) {
  renderCategory(nestedUpdate(blogCategory, ['posts', '12', 'author', 'name'], capitalize)); // 중간에 키 배열이 키 경로를 의미한다.
});
````

이를 해결하기 위해서 깊이 중첩된 데이터에 **추상화 벽** 사용하는 것이 좋아보인다.

### 깊이 중첩된 데이터에 추상화 벽 사용하기

중첩된 데이터의 구조를 3주가 지나도 기억할 만한 사람은 거의 없을 정도로 어렵다... 그래서! 문제를 해결하는 열쇠로써 추상화 벽을 이용할 수 있다. **🐋 추상화 벽에 함수를 만들고 의미 있는 이름을 붙여주는 것이다. 🐋**

```ts
// 분류에 있는 블로그 글이 어떤 구조인지 몰라도 함수를 쓸 수 있다.
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

updatePostById(blogCategory, "12", function (post) {
  return updateAuthor(post, captializeName);
});
```

🧨 블로그 카테고리라는 분류의 12라는 아이디를 가진 글에 사용자 이름을 대문자로 만들겠다는 의미가 이전보다 확실히 전달된다고 생각한다! 🧨

책에서는,

1. 기억해야 할 것이 네가지에서 세가지로 줄고, (❓ 인자가 세 개로 줄었다는 걸 의미하는 지 모르겠다... ❓)
2. 동작의 이름이 있으므로 각각의 동작을 기억하기 쉽다는 점을 얘기한다.

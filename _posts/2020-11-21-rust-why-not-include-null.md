---
layout: post
title:  "Rust가 Null을 도입하지 않은 이유"
date:   2020-11-21 20:38:27 +0900
categories: rust
tags: rust c++ reference null
comments: true  
---
`Rust`에는 다른 Programming 언어들이 가지는 `Null` 이라는 기능이 없다.
`Null`은 아무런 값도 가지지 않음을 의미하며, `Null`을 사용하는 언어들은 변수가 `Null`이냐 `Not-Null`이냐로 나뉜다.

2009년 Null의 창시자인 `Tony Hoare`는 "Null References: The Billion Dollar Mistake," 발표에서 아래와 같이 말했다.
> 나는 그것을 십억 달러 짜리 실수라고 부른다. 그 당시에, 나는 객치제향언어에서 
> 참조자를 위한 첫 Comprehensive type system을 만들고 있었다.
> 컴파일러에 의한 자동 검사로, 모든 참조자 사용을 안전하게 만드는 것이 내 목표였다.
> 그러나 나는 Null-reference를 넣는 것을 참을 수가 없었는데, 단순히 그게 구현하기가 쉽기 때문이었다.
> 이로 인해 지난 40년 동안 십억 달러치 손해를 입혔을지도 모르는 셀 수 없이 많은 에러, 취약점, 시스템 크래쉬를 만들어냈다.

`Null`의 문제는 `Null` value를 `Not-Null` value 처럼 사용하려고 할 때 발생한다.
`Null`과 `Non-Null`은 많이 사용되기 때문에, 에러를 만들어내기가 굉장히 쉽다.
그러나 `Null`이 표현하고자 하는 개념 자체는 유용한데, `Null`은 현재 사용 불가능하거나 존재하지 않음을 나타낸다.
`Null`의 문제는 `Null`의 개념 보다는 구현에 존재한다. 그래서 `Rust`는 `Null`을 사용하지 않고, 존재하거나 존재하지 않음을 나타내는 개념을 나타내는 `Option<T>`을 이용한다.

```[rust]
enum Option<T> {
    Some(T),
    None,
}

fn add_one(value: Option<i32>) -> Option<i32> {
    match value {
        None => None,
        Some(i) => Some(i + 1)
    }
}
```
`Option`이 `enum`이기 때문에, `pattern matching`을 위한 `match` syntax와 자주 사용되는데,
값이 존재한다면, `value`는 Some pattern으로 분기할 것이고, 값이 존재하지 않는다면 `value`는 `None`으로 분기한다.
이렇게 보면 `Null`이던 `Option`의 `None`이던 무슨 차이인지 잘 이해가 안갈 것이다.
핵심은 **Option<T>와 T는 결국엔 다른 타입**임에 있다. 컴파일러가 `Option<T>` 값을 마치 확실히 값이 존재하는 것처럼 사용하려고 하는 것을 막는다.
아래의 예제 코드의 에러 메시지는 이를 잘 나타내준다.
```[rust]
let x: i8 = 5;
let y: Option<i8> = Some(5);

// let sum = x + y; error[E0277]: cannot add `std::option::Option<i8>` to `i8`
```

### Reference
1. [Rust Tutorial Book](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html)
2. [rust-lang docs](https://doc.rust-lang.org/std/option/enum.Option.html)
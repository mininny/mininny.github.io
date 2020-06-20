---
title:  "(한글) How \"defer\" operator in Swift actually works"
category: translation
tag: [ios, knowledge_base]
date: 2020-06-20
---

> `defer`연산자가 작동하는 방법
>
> 이 글은 [Sergey Smagleev가 Medium에 올린 글을](https://medium.com/@sergeysmagleev/how-defer-operator-in-swift-actually-works-30dbacb3477b) 번역한 것이다. 

`defer`는 Swift 2에서 추가가 되었지만, 별로 쓰이지도 않고, 많이 알려지지도 않은 연산자다. `defer`은 보통 클로저 안에서 해당 스코프가 끝나기 전에 코드가 실행되게 해준다. 만약에 코드에 반환문이 많거나 똑같은 코드를 복붙하고싶지 않다면 defer가 매우 유용하게 쓰일수 있다. 그래서 보통 defer안에 있는 코드는 현재 작업을 청소하주는 cleanup코드인 경우가 대반사이다. 예를 들면, `nslock`을 사용해서 `thread-safe`한 로직을 만들고 있다면 `defer { lock.unlock() }`으로 절대로 병목현상이나 메모리 릭이 나지 않도록 할수 있다. 

근데, 이 코드가 스코프가 끝나기 전에 불린다고 하지만, 이게 정확히 어디일까? 한번 알아보도록 하자.

## 1번 예 

이 코드를 한번 보자.

```swift
var a = "Hello"

func b() -> String {
    defer { a.append(" world") }
    return a
}
```

매우 간단한 코드이고 defer가 별로 쓸모 없어 보인다. 그냥 `a`를 리턴하는 코드니까, 이런식으로 더 간단하게 만들수 있다:

```swift
func d() -> String {
    a.append(" world")
    return a
}
```

그러면 지금 이 `b()`와 `d()`의 동작이 똑같을까? 똑같은 입력값을 넣었을때 어떤 결과가 나오는지 보도록 하자:

```swift
a = "Hello"
print(b())
a = "Hello"
print(d())
```

여기 출력된 값이 있다:

```
Hello
Hello world
```

아마 이렇게 될줄 알고 있었을 거라고 믿는다. defer문안에 코드를 넣으면 리턴값이 이미 반환된 후에 실행 될것처럼 보일수 있다. 언뜻 출력값을 보면 그런것 같지만, 잘 보면 좀 이상한 점이 있다. 

자, `defer`의 정의를 한번 보자:
> A defer statement is used for executing code **just before** transferring program control outside of the scope that the defer statement appears in.
>
> defer문은 해당 코드의 스코프를 **빠져나가기 바로전에** 코드를 실행시키기 위해서 사용된다. 

하지만 생각해보면, 함수가 리턴을 한후에 어떤 작업을 하는건 사실상 불가능하다. 거의 모든 프로그래밍 언어의 함수들은 `return`문으로 끝나기 때문이다. `return`이 실행되면, 해당 스코프를 빠져나가고, 모든 로컬 데이터들을 다 청소하고, 해당 함수를 스택에서 제거하고, call hierarchy의 상위 함수로 다시 이동하기 때문이다. 

혹시라도 Objective-C의 `autorelease`를 이용해서 함수가 실행을 끝낸 뒤에 리소스들을 정리할때 뭐를 할수 있다고 생각하면 그건 오해다. `autorelease`는 해당 `Run Loop`이 돌때마다 `autorelease pool`이 `release`메시지를 보내는 방식으로 작동을 하지만, defer는 관련된 로직이 전혀 포함되어 있지 않다. 

그래서 실제로 이게 어떻게 작동하는지 알아보기 위해서 [Hopper Disassembler](https://www.hopperapp.com/) 라는것을 사용해보겠다. 작성한 코드를 먼저 컴파일을 해보자: 
```bash
xcrun swiftc your_source_code.swift -o output_file
```

그 다음, Hopper를 설치하고, `output_file`을 프로그램에 넣어보자. Hopper가 컴파일된 파일을 분석하면서 실제로 실행될 executable processor intruction들을 보여줄거다. 그리고, 읽기 편하게 만들기 위해, C언어랑 비슷한 수도코드(pseudocode)도 같이 만든다. 바로 우리가 필요한것이다. 

해당파일이 disassemble된 `b()`함수의 pseudocode이다:
```c
int _$S05test_A01bSSyF() {
    swift_beginAccess(_$S05test_A01aSSvp, &var_18, 0x20, 0x0);
    rcx = *_$S05test_A01aSSvp;
    swift_bridgeObjectRetain(rcx);
    swift_endAccess(&var_18);
    $defer #1 ();
    rax = rcx;
    return rax;
}

int _$S05test_A01bSSyF6$deferL_yyF() {
    var_40 = Swift.String_builtinStringLiteralutf8CodeUnitCountisASCII.init(" world", 0x6, 0x1);
    swift_beginAccess(_$S05test_A01aSSvp, &var_20, 0x21, 0x0, &var_20, 0x21);
    $SSS6appendyySSF(var_40, 0x1);
    swift_endAccess(&var_20);
    rax = swift_bridgeObjectRelease(var_40);
    return rax;
}
```

이해하는데 시간이 좀 걸리니까 천천히 한번 읽어보자. 자, 먼저, 처음 함수가 `b()`를 나타낸다, 그리고 두번째 함수가 `defer`에서 사용될 클로저문을 나타낸다. 그리고 당연하지만, `return`문들이 두 함수의 마지막에 위치해있다. `swift_beginAccess`와 `swift_endAccess`는 전역변수 a를 가져올때 사용이 된다. 그리고 나머지 이상한것들은 지금은 중요하지 않으니 신경쓰지 않아도 된다. 

중요한 코드는 3, 6, 7 번째 줄에 있다. 3번째 줄에서 `a`의 값이 `rcx` 레지스터에 저장이 된다. 그리고, defer에 해당하는 코드는 6번째 줄에서 호출이 된다. defer함수 안에서는, `defer`가 `" world"`라는 새로운 스트링 값을 만들고, `a`에 concatenation으로 추가를 한다. defer에서 `rax`에 어떤값을 지정하고 리턴하는것은 그냥 레지스터에 등록된 값을 리턴할때 사용되는 컨벤션이니 무시해도 좋다. 자, 이제 첫번째 함수로 돌아와서, `defer`가 끝나면, 전에 있던 값이 7번째 줄에서 `rcx`에서 `rax`로 할당이 되고, 그 다음줄에서 리턴이 된다. 

간단히 말하면, `defer`가 불리기 전에, 기존의 값을 저장해두었다가, 그 값을 나중에 다시 불러와서 사용하는 것이다. 그래서 `b()`함수에서 defer를 사용하지 않고 똑같이 만든다면, 그냥 임시적인 변수에 `a`의 값을 저장했다가 바로 리턴하는 꼴이 된다:
```swift
func c() -> String {
    let d = a
    a.append(" world")
    return d
}
```

별로 다를것이 없다. 근데, 왜 굳이 새로운 변수를 추가해서 왜 프로그램을 더 복잡하게 만드는것이고, 혹시 메모리 소모를 더 많이 하는것은 아닐까? 이 질문을 답하기 위해서는 변수들이 값을 지정받을때 어떻게 동작하는지 알아야 한다. 

스위프트에는 두 종류의 데이터 타입이 있다: 값 타입(struct 와 primitive값들) 과 레퍼런스 타입(classes). 만약의 위에 있는 연산을 레퍼런스 타입의 변수로 시도를 했다면 우리가 생각한대로 가장 마지막의 추가를 한 값을 포함해서 리턴이 되었을것이다.

하지만 스위프트의 `String`은 값 타입이다. 해당 값이 복사가 될때마다 새로운 객체가 생성되는 것이다. 사실 이것보다 더 복잡하긴 하다. String은 텍스트의 길이에 관계없이 언제나 16바이트의 wrapper이다. 하지만, 내부에서 텍스트를 실제로 가지고 있는 `character buffer`는 레퍼런스 타읍으로, copy-on-write을 사용해서 변수가 사용될때 메모리사용을 최소화 시킨다. 그렇기 때문에, `a`를 몇번을 복사를 해도 `a.append(" world")`가 실행되기 전까지는 실제로 복사가 한번도 되지 않는다. 

그래서, copy-on-write에게 고맙게도 새로운 변수가 추가된다고 해서 별로 문제될것은 없는것이다. 그래서, 아까 만들었던 `b()`의 disassembled 코드에서 defer만 지우면 `b()`와 `c()`가 비슷해지는거다: 

```c
int _$S05test_A01bSSyF() {
    swift_beginAccess(_$S05test_A01aSSvp, &var_18, 0x20, 0x0);
    rcx = *_$S05test_A01aSSvp;
    swift_bridgeObjectRetain(rcx);
    swift_endAccess(&var_18);
    var_40 = Swift.String_builtinStringLiteralutf8CodeUnitCountisASCII.init(" world", 0x6, 0x1);
    swift_beginAccess(_$S05test_A01aSSvp, &var_20, 0x21, 0x0, &var_20, 0x21);
    $SSS6appendyySSF(var_40, 0x1);
    swift_endAccess(&var_20);
    swift_bridgeObjectRelease(var_40);
    rax = rcx;
    return rax;
}
```
> b()의 disassembled pseudocode

```c
int _$S05test_A01cSSyF() {
    swift_beginAccess(_$S05test_A01aSSvp, &var_20, 0x20, 0x0, &var_20);
    rax = *_$S05test_A01aSSvp;
    swift_bridgeObjectRetain(rax);
    swift_endAccess(&var_20);
    var_80 = Swift.String_builtinStringLiteralutf8CodeUnitCountisASCII.init(" world", 0x6, 0x1);
    swift_beginAccess(_$S05test_A01aSSvp, &var_38, 0x21, 0x0);
    $SSS6appendyySSF(var_80, 0x1);
    swift_endAccess(&var_38);
    swift_bridgeObjectRelease(var_80);
    rax = rax;
    return rax;
}
```
> c()의 disassembled pseudocode


## 2번 예

이제는 이 코드를 한번 보자:
```swift
var a: String? = nil

func b() -> String {
    a = "Hello world"
    defer { a = nil }
    return a!
}

print(b())
```

이 코드는 조금 더 흥미롭다. 어떻게 보면, defer를 사용해서 현재 스코프를 벗어나기 전에 할당된 값들을 정리하는것처럼 보인다. 이 예제에서는 전역 변수 String?이 함수내에서 값을 지정받고, 함수가 끝날때 `nil`로 다시 변환된다. 

이 코드를 실행하면 아무런 문제 없이 `"Hello World"`를 출력한다. 말은 되는것 같지만 다시 한번 보자. 방금 전만해도 defer내부에 있는 `a = nil`이 함수가 리턴하기 전에 실행된다고 하지 않았었나? 그러면 어떻게 강제 언래핑을 했을때 에러가 나지 않았던 걸까?

아까처럼 다시 한번어 Hopper를 사용해서 코드를 분석해보자: 
```c
int _$S10test_force1bSSyF() {
    var_40 = Swift.String_builtinStringLiteralutf8CodeUnitCountisASCII.init("Hello world", 0xb, 0x1);
    swift_beginAccess(_$S10test_force1aSSSgvp, &var_18, 0x21, 0x0, 0x21);
    rdi = *_$S10test_force1aSSSgvp;
    rsi = *qword_100001078;
    *_$S10test_force1aSSSgvp = var_40;
    *qword_100001078 = 0x1;
    _$SSSSgWOe(rdi, rsi);
    swift_endAccess(&var_18);
    swift_beginAccess(_$S10test_force1aSSSgvp, &var_30, 0x20, 0x0);
    rax = *_$S10test_force1aSSSgvp;
    rcx = *qword_100001078;
    var_48 = rax;
    var_50 = rcx;
    _$SSSSgWOy(rax, rcx);
    swift_endAccess(&var_30);
    if (var_48 != 0x0) {
            var_58 = var_48;
            var_60 = var_50;
            $defer #1 ();
            rax = var_58;
    }
    else {
            stack[-168] = "test_force.swift";
            *(int32_t *)(&stack[-168] + 0x20) = 0x1;
            *(&stack[-168] + 0x18) = 0x6;
            *(int32_t *)(&stack[-168] + 0x10) = 0x2;
            *(&stack[-168] + 0x8) = 0x10;
            Swift_fatalErrorMessage first-element-marker  first-element-marker fileline.flags("Fatal error", 0xb, 0x2, "Unexpectedly found nil while unwrapping an Optional value", 0x39, 0x2);
            asm { ud2 };
            rax = loc_100000d90();
    }
    return rax;
}
```

와... 3줄짜리 코드가 갑자기 34줄짜리 코드가 되어버렸다. 이 코드를 한줄 한줄씩 이해하려고 하지말고, 코드를 의미있는 그룹으로 뭉쳐서 하나씩 분석을 해보자. 먼저, 2~9번째 줄은 `b()`함수 내에서 변수 `a`에 `"Hello world"`를 할당해주는 코드이다. defer을 실행하는 코드는 20번째 줄에 있고, 나머지 줄은 모두 `return a!`에 해당하는 코드들이다. 

사실 자세히 보면 `return a!`가 절대로 간단하지 않다. 실제로는 3가지 다른 일이 일어나고 있다:
1. a의 값을 읽고 로컬 스코프에 저장을 한다. 해당 작업은 10~16번째 줄에 있는 두번째 swift_beginAccess/swift_endAccess에서 일어난다. 
2. 1번에서 저장을 한 값을 unwrapping한다. 이 작업은 17~32번째 줄에 있는 if 문에 해당한다. 18~21줄은 해당값이 존재할때 실행되고, 24~31번째 줄은 값이 nil일때 실행되는 크래쉬 코드이다. 
3. 리턴! 33번째 줄에서 드디어 리턴을 한다.

좀 어렵다고 느껴지면, 여기에 색깔별로 정리를 한번 해봤다. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200620/defer-pseudocode.png" alt="">

그리고 이게 원래 b() 함수이다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200620/defer-actual-code.png" alt="">

이 disassembled code에서 defer가 어떻게 사용되는지 분석해보면 매우 많은것을 알수 있다. 예를 들면, `a = nil`가 함수가 리턴되기 전에 불려도 a를 리턴할때는 강제 언래핑을 해도 에러가 나지 않는다. 더 신기한거는 defer가 force unwrapping하기 이전이나 이후가 아닌 force unwrapping중간에 나타난다는 것이다. 정리하면, unwrapping할때 에러가 나는것을 방지하기 위하여, defer를 실행하기 전에 임시 변수에 값을 저장했다가, defer를 실행하고, 그 뒤에 저장해둔 임시 변수를 리턴하는것이다. 이 변수는 변경 되지 않았기 때문에 CoW에 의거하여 새로운 메모리를 할당하지 않는다. 

잘 보면 defer된 코드가 다른 함수 내부까지 전달딘다는게 매우 신기하다. 함수가 끝나기 전에 defer을 실행하는줄 알았지만, 알고보니 함수의 내부까지 깊게 파고 들어가는걸 보니 신기하다. 

## 결론!

`defer`연산자는 사용하는것에 비해 내부 구현이 훨씬더 복잡하고 세밀하게 되어있다. 이 연산자의 진정한 좋은점은 스위프트의 `return`문을 개별의 low level 실행문으로 쪼개서 마지막 명령어(ret / return) 바로 전에 defer문을 실행시킨다는 점이다. 이렇게 함으로써, 추가로 필요없는 변수를 만들 필요 없이 더 예쁘고 자연스로운 코드를 짤수 있게 된것이다. 
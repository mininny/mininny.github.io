---
title:  "(한글) Nearly All Binary Searches and Mergesorts are Broken"
category: translation
date: 2020-06-12
---

> 거의 모든 이진 탐색과 합병정렬(mergesort) 알고리즘들은 틀려있다. 
>
> 이 글은 Joshua Bloch가 Google AI Blog에 올린 [글을](https://ai.googleblog.com/2006/06/extra-extra-read-all-about-it-nearly.html) 번역한 것이다. 

나는 예전에 카네기 멜론 대학에서 Jon Bentley 교수님의 첫 알고리즘 강의에서 교수님이 모든 신입생에게 이진 탐색을 구현해보라고 했던 것이 기억난다. 그리고 우리중의 한명의 구현부를 가지고 매우 세세히 살펴 봤는데, 다른 학생들이랑 비슷하게도 엉망진창이었다. 그 교수님의 다른 강의들과 매한가지로 이 강의는 나에게 매우 큰 인상을 남겼었다. 교수님은 프로그램에 있는 불변값(invariants)들을 언제나 세세히 알고 있으라는 것이었다. 

최근에, 나는 Bentley교수님이 맞다고 했던 그 이진 탐색이 틀렸다는걸 알게됬다. 그리고 교수님이 쓰신 Programming Pearls책에서 수도 없이 사용되고 테스트되어있던 그 코드가 알고보니 버그로 이루어져 있던것이다. 내가 지금 말을 해주면 당신들도 왜 이 버그가 20년동안이나 알려지지 않았던건지 알수 있을거다. 혹시라도 내가 Bentley 교수님의 명성에 누를 끼치기 위해서 이런 글을 쓰는거라고 생각하지 않았으면 한다. 심지어 내가 JDK에 직접 작성했던 이진 탐색도 똑같은 버그가 있고 9년동안 방치되었기 때문에 말이다. 

그래서 버그가 뭔가? 

여기 평범한 이진 탐색 알고리즘이 있다. 내가 실제로 java.util.Arrays에 작성했던 코드다:
```java
public static int binarySearch(int[] a, int key) {
    int low = 0;
    int high = a.length - 1;

    while (low <= high) {
        int mid = (low + high) / 2;
        int midVal = a[mid];

        if (midVal < key)
            low = mid + 1
        else if (midVal > key)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found.
}
```

버그는 바로 이줄에 있다: 
```java
6: int mid = (low + high) / 2;
```

Programming Pearls 책에서 Bentley교수는 이 줄이 `mid`를 `low`와 `high`의 평균값으로 만들고, 가장 근접한 integer로 변환한다고 써놓았다. 언뜻 보면 맞은것처럼 보이는데, 이 경우는 `low`와 `high`의 값이 매우 높은 integer값일때 문제가 발생한다. 더 자세히 말하면, `low`와 `high`를 더한 값이 int의 최대값 (2^31-1) 보다 크면 발생한다. 이 값을 넘어버리면 두 값을 더한값은 오버플로우가 되어서 음수가 되어버리고, `mid`값도 음수가 되어버리는 것이다. C언어에서는 `Array index out of bounds`라는 에러를 발생시킬테고, Java에서는 `ArrayIndexOutOfBoundsException`이 일어날수 있다. 

이 버그는 배열의 길이가 2^30 (약 10억)이 넘어가면 생길수 있다. 물론 80년도때는 상상도 하지 못할 배열의 크기지만, 요즘에는 구글을 비롯해서 많은 곳에서 생길수 있는 데이터다. 그렇다 보니, 알고 보면, 프로그래밍의 세계에서 맞는 버전의 이진 탐색 알고리즘이 나온경우가 몇번 없었다.

그래서 이걸 고칠수 있는 가장 좋은 방법이 뭘까? 여기 한 예시가 있다: 
```java
int mid = low + ((high - low) / 2);
```

조금더 빠르고 명확할수 있는 다른 방식은: 
```java
int mid = (low + high) >>> 1;
```

C언어와 C++에서는 `>>>` 연산자를 못쓰기 때문에 이렇게 할수 있다:
```c++
mid = ((unsigned int)low + (unsigned int)high)) >> 1;
```

자 이제 이진 탐색 알고리즘이 bug-free인걸 알수 있을까? 그렇다고 생각하고 있긴 하지만, 사실은 잘 모른다. 프로그램이 올바르다고 증명하는건 충분하지 않다; 테스트도 해봐야하는것이다. 그리고, 프로그램이 정확하게 동작하는것을 확실하게 알수 있으려면, 모든 입력값에 대해 테스트를 일일이 해봐야하겠지만, 이건 거의 불가능하다. 만약에 여러작업이 동시에 진행되는 복잡한 프로그램이라면, 테스트를 치밀하게 하기에는 더욱더 불가능할 뿐이다. 

이 이진탐색의 버그는 mergesort을 비롯해 다른 divide-and-conquer종류의 모든 알고리즘에 해당한다. 만약에 해당하는 알고리즘에서 위에서 봤던 코드가 발견되면, 코드가 정말 오작동하기 전에 얼른 고쳐라. 

이 버그에서 내가 배운것은 `겸손`이다. 이렇게 짧은 코드를 올바르게 짜기도 힘든데, 요즘 세상은 훨씬더 크고 복잡한 코드에 의존해서 돌아가고 있으니 말이다. 

프로그래머들은 언제나 다른 사람과 다른 생각의 도움이 필요하고, 절대로 그렇지 않다고 생각하면 안된다. 

세밀한 디자인은 좋다. 테스팅도 좋다. 잘 짜진 함수들도 좋다. 코드리뷰도 좋다. 정적 프로그램 분석도 좋다. 하지만 이 모든것들도 버그를 찾기에 완벽한 도구가 아니다: 버그는 언제나 우리와 함께한다. 우리가 아무리 없애려 해도 수십년간 남아있는게 버그다. 그렇기 때문에 우리는 언제나 더더욱 조심히, 방어적으로, 그리고 경계스럽게 프로그래밍을 해야한다. 

---

Update 17 Feb 2008: Thanks to Antoine Trux, Principal Member of Engineering Staff at Nokia Research Center Finland for pointing out that the original proposed fix for C and C++ (Line 6), was not guaranteed to work by the relevant C99 standard (INTERNATIONAL STANDARD - ISO/IEC - 9899 - Second edition - 1999-12-01, 3.4.3.3), which says that if you add two signed quantities and get an overflow, the result is undefined. The older C Standard, C89/90, and the C++ Standard are both identical to C99 in this respect. Now that we've made this change, we know that the program is correct;)
Resources

Programming Pearls - Highly recommended. Get a copy today!
The Sun bug report describing this bug in the JDK
A 2003 paper by Salvatore Ruggieri discussing a related problem - The problem is a bit more general but perhaps less interesting: the average of two numbers of arbitrary sign. The paper does not discuss performance, and its solution is not fast enough for use in the inner loop of a mergesort.
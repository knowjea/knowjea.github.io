---
layout: post
title: "9. Always override hashCode when you override equals (equals를 재정의할 때는 반드시 hashCode도 재정의하라)"
author: "Gyeongjae Gwon"
categories: 개발
tags: [java, effective java]
comments: true
---


자바에서 최상위 객체인 Object는 하위 객체들이 오버라이딩하여 사용하도록 설계된 메서드들이 있다. (equals, hashCode, toString, clone, finalize)
그리고 이 메서드들은 일반 규약이 존재하는데 이를 따르지 않으면 자바에서 제공하는 클래스와 함께 사용할 때 제대로 동작하지 않는다.

이번 장에서는 hashCode 메서드의 일반 규약을 작성한다.

## hashCode 메서드

hashCode 메서드는 오브젝트의 해시 코드 값을 반환한다. 이 메서드는 java.util.HashMap과 같은 해시 테이블을 위해 자바에서 지원하고 있다.
따라서 hashCode 메서드를 규약에 맞게 구현하지 않으면 해시 기반의 컬레션은 오동작하게 된다.

디폴트로 제공하는 Object의 hashCode 메서드는 아래와 같다.
사실 Object의 hashCode는 native함수로 C언어로 작성되어 있다. 좀 더 자세히 알고 싶다면 다음을 참고 [link to hashCode](https://srvaroa.github.io/jvm/java/openjdk/biased-locking/2017/01/30/hashCode.html).

{% highlight java %}
public class Object {
	// ...
	
	public native int hashCode();
}
{% endhighlight %}

 
## hashCode의 일반 규약 (Object 클래스 명세)

* Whenever it is invoked on the same object more than once during an execution of a Java application, the hashCode method must consistently return the same integer, provided no information used in equals comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.

equals에서 객체의 동일성을 판단하기 위한 필드들의 값이 변하지 않는 이상 hashCode의 반환 값은 항상 동일해야 한다. 다만, 프로그램이 재실행 되어도 같을 필요는 없다. 그렇다면 equals안에 검사 필드들의 값이 변할 경우, hashCode 값은 변할 수 있다라는 의미이다.

* If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result.

두번째 규약이 이 포스팅 주제의 핵심이다. 해석하면 ** equals에서 두 객체가 동일하다고 하면, hashCode 값은 반드시 동일해야 한다. **

* It is not required that if two objects are unequal according to the java.lang.Object.equals(java.lang.Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables. 

두번째 규약에서 두 객체가 같다고 판정되면 hashCode 값은 무조건 같아야 한다. 하지만 역으로 두 객체가 다르다고 판정된다고 hashCode 값이 무조건 달라야 할 필요는 없다. 하지만 다르면 해시 테이블의 성능이 향상될 수 있다.
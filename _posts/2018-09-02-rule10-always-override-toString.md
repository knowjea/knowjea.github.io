---
layout: post
title: "10. Always override toString (toSring은 항상 재정의하라)"
author: "Gyeongjae Gwon"
categories: 개발
tags: [java, effective java]
comments: true
---


자바에서 최상위 객체인 Object는 하위 객체들이 오버라이딩하여 사용하도록 설계된 메서드들이 있다. (equals, hashCode, toString, clone, finalize)
그리고 이 메서드들은 일반 규약이 존재하는데 이를 따르지 않으면 자바에서 제공하는 클래스와 함께 사용할 때 제대로 동작하지 않는다.

이번 장에서는 toString 메서드의 일반 규약을 작성한다.

## toString 메서드

toString 메서드는 오브젝트를 스트링으로  이 메서드는 java.util.HashMap과 같은 해시 테이블을 위해 자바에서 지원하고 있다.
따라서 hashCode 메서드를 규약에 맞게 구현하지 않으면 해시 기반의 컬레션은 오동작하게 된다.

디폴트로 제공하는 Object의 hashCode 메서드는 아래와 같다.
사실 Object의 hashCode는 native함수로 C언어로 작성되어 있다. 좀 더 자세히 알고 싶다면 다음을 참고 [link to hashCode](https://srvaroa.github.io/jvm/java/openjdk/biased-locking/2017/01/30/hashCode.html).

{% highlight java %}
public class Object {
	// ...
	
	public native int hashCode();
}
{% endhighlight %}

 
[link to Rule11](https://knowjea.github.io/%EA%B0%9C%EB%B0%9C/2018/08/28/rule9-always-override-hashcode-overriding-equals.html).
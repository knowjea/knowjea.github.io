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

## toString 메서드, 일반 규약

toString 메서드는 오브젝트를 사람이 이해하기 쉬운 문자열로 표현한다. 명세서에는 다음과 같이 규약이 표현되어있다.

**toString의 결과는 간결하지만 유익한 표현이어야하며 사람이 쉽게 읽을 수 있어야합니다.**

하지만 디폴트로 제공하는 Object의 toString은 위 규약을 만족하진 않는다. Object의 toString 메서드는 아래와 같다.

{% highlight java %}
public class Object {
	// ...
	
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
}
{% endhighlight %}

클래스 이름 다음에 @ 기호와 16진수로 표현된 해시 코드가 붙은 문자열을 반환한다. 간결해 보일지는 몰라도 사람이 쉽게 읽고 이해하기는 힘들 것 같다.
따라서 명세서에는 다음과 같은 글이 이어져있다.

**모든 서브 클래스가 이 메소드를 오버라이드 하는 것을 추천합니다.**



## toString 메서드 구현 방법

toString 메서드는 문자열을 print하는 메서드에 오브젝트를 인자로 넣으면 자동으로 오브젝트의 toString 메서드가 호출된다.
따라서 assert, debugger 또는 에러메시지를 전달할 때 한 눈에 객체에 모든 정보를 볼 수 있어야 한다.

**가능하다면 toString 메서드는 객체 내의 중요 정보를 전부 담아 반환해야 한다.**

객체가 만약 아주 크기 때문에 전부 담기 힘들다면, 최대한 요약 표현한다.

{% highlight java %}
public class PhoneNumberWithHashCode {
	private final int areaCode;
	private final int prefix;
	private final int lineNumber;

	public PhoneNumberWithHashCode(int areaCode, int prefix, int lineNumber) {
		this.areaCode = areaCode;
		this.prefix = prefix;
		this.lineNumber = lineNumber;
	}

	@Override
	public String toString() {
		return String.format("(%03d) %03d-%04d", areaCode, prefix, lineNumber);
	}

	public static void main(String[] args) {
		PhoneNumberWithHashCode p1 = new PhoneNumberWithHashCode(111, 654, 7009);

		System.out.println(p1); // (111) 654-7009
	}
}
{% endhighlight %}

가끔씩 API를 사용하는 개발자가 toString에 출력되는 문자열을 가지고 파싱하여 특정 데이터를 생성하는 경우가 있다.
하지만 toString 메서드가 출력하는 문자열은 언제든 변경될 수 가 있는데 이런 문제를 방지하기 위해 미리 주석으로 변경되는 데이터를 명시해야 한다.

 

[link to Rule11](https://knowjea.github.io/%EA%B0%9C%EB%B0%9C/2018/08/28/rule9-always-override-hashcode-overriding-equals.html).
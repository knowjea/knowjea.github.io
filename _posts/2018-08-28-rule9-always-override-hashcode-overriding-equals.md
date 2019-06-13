---
layout: post
title: "9. Always override hashCode when you override equals (equals를 재정의할 때는 반드시 hashCode도 재정의하라)"
author: "Know jea"
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

두번째 규약에서 두 객체가 같다고 판정되면 hashCode 값은 무조건 같아야 한다. 하지만 역으로 두 객체가 다르다고 판정된다고 hashCode 값이 무조건 달라야 할 필요는 없다. 하지만 값이 다르다면 해시 테이블의 성능이 향상될 수 있다는 것은 알고  있어야한다.

만약 해쉬코드 값이 같게 되면 해쉬테이블에서 동일한 버킷에 오브젝트가 해쉬되고 버킷안에 존재하는 링크드리스트 안에서 해당 오브젝트를 찾아서 반환한다. 그러므로 링크드리스트에 수가 증가할 수록 검색하는 속도가 느려지므로 해시 값이 다를수록 좋다는 의미이다.


## 두번째 규약을 지키지 않았을 경우

두번째 규약을 지키지 않으면 어떤 경우에 오동작하는지 예제는 아래와 같다.
 
{% highlight java %}
public class PhoneNumber {
	private final int areaCode;
	private final int prefix;
	private final int lineNumber;

	public PhoneNumber(int areaCode, int prefix, int lineNumber) {
		this.areaCode = areaCode;
		this.prefix = prefix;
		this.lineNumber = lineNumber;
	}

	@Override
	public boolean equals(Object obj) {
		if (obj == this) {
			return true;
		}
		if (!(obj instanceof PhoneNumber)) {
			return false;
		}

		PhoneNumber phoneNumber = (PhoneNumber) obj;

		// Since lineNumber may be the most different, check first.
		return phoneNumber.lineNumber == lineNumber && phoneNumber.prefix == prefix
		        && phoneNumber.areaCode == phoneNumber.areaCode;
	}

	public static void main(String[] args) {
		Map<PhoneNumber, String> map = new HashMap<PhoneNumber, String>();

		PhoneNumber p1 = new PhoneNumber(1, 2, 3);
		PhoneNumber p2 = new PhoneNumber(1, 2, 3);

		System.out.println(p1.equals(p2)); // true

		map.put(p1, "Phone");

		System.out.println(map.get(p1)); // Phone
		System.out.println(map.get(p2)); // null

		System.out.println(p1.hashCode()); // 366712642
		System.out.println(p2.hashCode()); // 1829164700
	}
}
{% endhighlight %}

p1과 p2는 논리적으로 동일하다. 즉, 새롭게 정의한 equals 메서드에서 두 객체는 동일하다고 판단한다.
그 다음 HashMap에 p1을 키로하여 데이터를 삽입하였다. 그 후, HashMap에서 데이터를 꺼낼 때 p1과 p2를 실험해본 결과 p1은 정상적으로 출력되지만 p2는 null을 반환한다.

Map은 동일한 키를 사용한다면 동일한 값을 반환해야 한다. 따라서 우리는 map.get(p2)도 Phone이 출력되어야 한다고 생각한다. 
왜냐하면 일반적으로 사람은 논리적으로 같으면 동일하다고 판단하기 때문이다. 하지만 HashMap은 동일하다는 기준을 hashCode 값을 사용하여 판단한다.

따라서 HashMap에 동일성의 기준과 사람의 동일성의 기준을 같게 하기 위해서 equals 메서드를 재정의하였으면 hashCode 메서드도 재정의해야 한다는 것이다.


## hashCode 메서드 구현 순서

세번째 규약에서 동일하지 않는 객체들끼리는 hashCode가 꼭 다를 필요는 없지만 다르면 성능적으로 좋다고 하였다. 서로 다른 객체들을 모든 가능한 해시 값에 균등하게 배분해야 하는데
수학자들이 그러한 이상적인 hashCode 메서드를 만드는 방법을 정의하였다.

1. Create a int result and assign a non-zero value.

2. For every field f tested in the equals() method, calculate a hash code c by:
	* If the field f is a boolean: calculate (f ? 0 : 1);
	* If the field f is a byte, char, short or int: calculate (int)f;
	* If the field f is a long: calculate (int)(f ^ (f >>> 32));
	* If the field f is a float: calculate Float.floatToIntBits(f);
	* If the field f is a double: calculate Double.doubleToLongBits(f) and handle the return value like every long value;
	* If the field f is an object: Use the result of the hashCode() method or 0 if f == null;
	* If the field f is an array: see every field as separate element and calculate the hash value in a recursive fashion and combine the values as described next.

3. Combine the hash value c with result:
	* result = 37 * result + c

4. Return result

위 PhoneNumber에 구현 예제는 다음과 같다.

{% highlight java %}
public class PhoneNumber {
	// ...
	
	@Override
	public int hashCode() {
		int result = 17;

		result = 31 * result + areaCode;
		result = 31 * result + prefix;
		result = 31 * result + lineNumber;

		return result;
	}

	public static void main(String[] args) {
		Map<PhoneNumberWithHashCode, String> map = new HashMap<PhoneNumberWithHashCode, String>();

		PhoneNumberWithHashCode p1 = new PhoneNumberWithHashCode(1, 2, 3);
		PhoneNumberWithHashCode p2 = new PhoneNumberWithHashCode(1, 2, 3);

		System.out.println(p1.equals(p2)); // true

		map.put(p1, "Phone");

		System.out.println(map.get(p1)); // Phone
		System.out.println(map.get(p2)); // Phone

		System.out.println(p1.hashCode()); // 507473
		System.out.println(p2.hashCode()); // 507473
	}
}
{% endhighlight %}

자주 사용하는 String 클래스의 hashCode는 아래와 같다.
문자 하나하나에 31을 곱하여 처리한 것을 볼 수 있다.

{% highlight java %}
public final class String{

	// ...
    /** Cache the hash code for the string */
    private int hash; // Default to 0
    
    	
	public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
}

{% endhighlight %}

그런데 위 String의 hashCode 메서드를 보면 'hash'라는 변수를 사용하여 0이 아닐 경우에는 'hash' 값을 가진 변수를 리턴한다.
이는 해시코드를 재계산하는 대신 미리 캐시해 두어서 한번만 계산하도록 한 것이다. 다만, 이렇게 캐시를 사용할 경우에는 변경 불가능 클래스여야 한다.
왜냐하면 중요 필드가 변경 될 경우, 해시값도 달라져야 하는데 캐시를 해두고 위 로직처럼 한다면 동일한 해시값을 계속 반환하기 때문이다.

[link to Rule10](https://knowjea.github.io/%EA%B0%9C%EB%B0%9C/2018/09/02/rule10-always-override-toString.html).
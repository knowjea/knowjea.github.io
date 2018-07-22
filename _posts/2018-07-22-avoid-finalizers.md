---
layout: post
title: "Avoid finalize (종료자 사용을 피하라)"
author: "Gyeongjae Gwon"
categories: facts
tags: [sample]
image: garbage-collection.jpg
---

## finalize 메소드

특정 객체에 대한 참조가 더 이상 없다고 판단할 때 가비지 컬렉션이 객체의 finalize를 호출한다.
finalize 메소드는 자바의 최상위 클래스인 Object 클래스에 포함되어 있다. 
가끔씩 개발자는 특정 객체가 소멸될 시점에 어떠한 자원 정리를 위해 해당 메소드를 오버라이딩하여 자신만의 코드를 작성한다.


{% highlight java %}
public class Rule7 {

	@Override
	protected void finalize() throws Throwable {
		super.finalize();
	}
}
{% endhighlight %}

<div class="post_caption">"하지만 finalize는 예측 불가능하며, 대체로 위험하고, 일반적으로 불필요하다."</div>


## 실행을 보장하지 않음

finalize 메소드는 호출되더라도 즉시 실행되라는 보장이 없으며, 반드시 실행된다는 보장도 없다.
따라서 finalize 메소드 안에서 유한한 자원에 메모리 해제, 중요한 상태정보 갱신을 하면 안된다.

{% highlight java %}
public class FileReadUtil {
	private static FileInputStream input;

	static {
		try {
			input = new FileInputStream("c:/out.txt");
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
	}

	public static int readByte() throws IOException {
		return input.read();
	}

	@Override
	protected void finalize() throws Throwable {
		try {
			// Do not do this.
			if (input != null) {
				input.close();
			}
		} finally {
			super.finalize();
		}
	}

}
{% endhighlight %}
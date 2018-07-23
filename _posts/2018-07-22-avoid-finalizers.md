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

그렇다면 명시적으로 반환해야 하는 자원을 삭제하는 방법은 무엇일까..?
파일입출력을 해봤다면 익숙한 구문인 try-catch-finally를 사용하면 된다.

{% highlight java %}
public class TryCatchFinally {
	public static void main(String[] args) throws IOException {
		FileInputStream fileInputStream = new FileInputStream("c:/out.txt");
		try {
			// do somgting
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			fileInputStream.close();
		}
	}
}
{% endhighlight %}


## 예외를 무시

finalize 메소드 안에서 예외가 발생한다고 하더라도, 해당 예외는 무시되며 스택 트레이스도 표시되지 않는다.
또한 확인은 하지 못했지만, 해당 finalize 메소드도 중단된다.
(예제에서 gc가 동작하여 finalize가 호출되도록 하였다. 하지만 system.gc는 finalize 호출을 보장하지는 않지만, 일반적으로는 잘 동작한다.)

{% highlight java %}
public class ExceptionInFinalizeTest {

	public static void main(String[] args) throws Throwable {
		ExceptionInFinalizeTest exceptionInFinalizeTest = new ExceptionInFinalizeTest();
		exceptionInFinalizeTest = null;

		// System.gc does not guarantee finalize, but generally works fine.
		System.gc();
	}

	@Override
	protected void finalize() throws Throwable {
		System.out.println("The finalize method start");

		// Exceptions are ignored.
		System.out.println(2 / 0);

		super.finalize();

		System.out.println("The finalize method end");
	}
}

{% endhighlight %}


## 성능 저하

객체에 finalize 메소드를 작성하면, 프로그램 성능이 심각하게 저하된다고 한다. (약 430배)

그러나, 실제로 확인 할 수 는 없었다. 잘 이해되지 않는 부분이다.
 
Object 클래스에 존재하므로 어떠한 클래스든 finalize 메소드를 가지고 있는 것이 아닌가?

특정한 객체에 finalize 메소드를 오버라이딩 했다고 해서 성능이 왜 저하되는지 이해가 되지 않는다. 

(도와주세요!)



## finalize 그러면 어디에 사용할까?

위에서 finalize 메소드 안에서 유한한 자원에 메모리 해제를 하면 안된다고 하였다. 즉, 다른 방법 (try-catch-finally)으로 메모리 해제를 해야한다.
하지만 API 개발자는 항상 클라이언트가 API를 올바르게 사용하지 않을수도 있다는 것을 고려해야한다.

예를 들어, FileOutputSteam 클래스를 이용하여 파일(자원)을 사용하였다면 반드시 close로 메모리를 해제해야한다.
클라이언트가 close 메소드 사용을 잊을 것을 대비해 언제 호출될지는 모르지만 finalize에 close 메소드를 명시적으로 호출한다.

** 동일한 패턴 클래스들 - FileInputStream, FileOutputSteam, Timer, Connection **

명시적 종료이므로 클라이언트를 위해 로그를 남기는것이 좋으나 위 API개발자는 아쉽게도 하지 않았다.

{% highlight java %}
public class FileInputStream extends InputStream{
	// ...
	
	 /**
     * Ensures that the <code>close</code> method of this file input stream is
     * called when there are no more references to it.
     *
     * @exception  IOException  if an I/O error occurs.
     * @see        java.io.FileInputStream#close()
     */
    protected void finalize() throws IOException {
        if ((fd != null) &&  (fd != FileDescriptor.in)) {
            /* if fd is shared, the references in FileDescriptor
             * will ensure that finalizer is only called when
             * safe to do so. All references using the fd have
             * become unreachable. We can call close()
             */
            close();
        }
    }
}
	
{% endhighlight %}
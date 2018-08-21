---
layout: post
title: "JAVA CountDownLatch (다른 쓰레드를 기다리는 방법)"
author: "Gyeongjae Gwon"
categories: 개발
tags: [java]
image: wating.jpg
comments: true
---


멀티쓰레드 프로그래밍에서 쓰레드들이 모든 작업을 마친 후에 특정한 작업을 해야하는 경우가 있다.
이를 위해 다른 쓰레드들에서 일련의 작업이 완료 될 때까지 대기하도록 Sync를 맞춰주는 기능을 자바는 java.util.concurrent.CountDownLatch(int count)을 제공한다.

대기하는 방식은 명세서에 나와 있다.

CountDownLatch를 인스턴스화할 때 주어진 카운트로 초기화된다. await 메서드를 실행하면 해당 쓰레드는 다른 쓰레드에서 countDown 메서드 호출로 인해 현재 카운트가 0이 될 때까지 대기한다. 그 후에는 대기중인 스레드가 해제된다.


## 예제

아래 예제는 Main 쓰레드는 **latch.await()** 메소드에 의해 대기상태로 진입한다. 이 후 Worker 쓰레드들이 자신의 작업을 마친 후, **latch.countDown()** 메소드를 통해 카운트를 줄여간다. 카운트가 0이 되면 Main 쓰레드를 대기상태에서 해제된다.

{% highlight java %}
public class CountDownLatchExample {
	static final int max = 3;

	public static void testCountDownLatch() throws Exception {
		final CountDownLatch latch = new CountDownLatch(max);

		for (long i = 1; i <= max; i++) {
			new Thread(new Worker(latch, i * 100)).start();
		}

		latch.await();

		System.out.println("########### CountDownLatch End ###########");
	}

	static class Worker implements Runnable {
		private CountDownLatch latch;
		private long n;

		public Worker(CountDownLatch latch, long n) {
			this.latch = latch;
			this.n = n;
		}

		@Override
		public void run() {
			try {
				int cnt = 0;
				for (int i = 0; i < n; i++) {
					cnt++;
				}

				System.out.println(cnt);
			} catch (Exception ex) {
				ex.printStackTrace();
			} finally {
				if (this.latch == null) {
					return;
				}

				latch.countDown();
			}
		}
	}

	public static void main(String[] args) throws Exception {
		System.out.println("start");
		testCountDownLatch();
		System.out.println("end");
	}

	/**
	 * start
	 * 200
	 * 100
	 * 300
	 * ########### CountDownLatch End ###########
	 * end
	 */
}
{% endhighlight %}

<<<<<<< HEAD
=======
아래처럼 예외가 발생해도 "End"는 출력되지 않지만 "Hook Run"이 출력된다.
{% highlight java %}
	// ...
	
	public static void main(String[] args) {
		Runtime.getRuntime().addShutdownHook(new HookThread());
		
		int errorNum = 1 / 0;
		System.out.println("End");
	}
}
{% endhighlight %}


콘솔에서 sleep 동안 인터럽트(Ctrl+C)를 줄 경우에도 "End"는 출력되지 않지만 "hook Run"이 출력된다.
{% highlight java %}
	// ...
	
	public static void main(String[] args) {
		Runtime.getRuntime().addShutdownHook(new HookThread());

		try {
			System.out.println("sleep 3s");
			Thread.sleep(3000);
		} catch (Exception e) {
			e.printStackTrace();
		}

		System.out.println("End");
	}
}
{% endhighlight %}

## 순서를 지정할 수 없음

다중 스레드 실행처럼 Shutdown Hook이 실행되는 순서를 결정할 수 없다.
즉 addShutdownHook() 메소드끼리의 호출 순서에 상관없이 동시에 실행된다.

아래처럼 HookThread를 먼저 등록하고, HookThread2를 등록하였지만
실행할 때마다 출력되는 순서는 랜덤하다.

{% highlight java %}
public class ShutdownHookTest {

	static class HookThread extends Thread {
		@Override
		public void run() {
			System.out.println("Hook Run1");
		}
	}

	static class HookThread2 extends Thread {
		@Override
		public void run() {
			System.out.println("Hook Run2");
		}
	}

	public static void main(String[] args) {
		Runtime.getRuntime().addShutdownHook(new HookThread());
		Runtime.getRuntime().addShutdownHook(new HookThread2());

		System.out.println("End");
	}
}
{% endhighlight %}


## 등록된 Shutdown Hook의 실행을 막는 종료 방법

사용할 일이 별로 없을 것 같지만, Runtime.halt(int)를 사용하면 등록된 Shutdown Hook을 실행하지 않고 종료된다.

만약 종료 중에 Halt가 호출될 경우 (종료 중이라면 등록된 Shutdown Hook 쓰레드들이 동시에 실행되는 상태) 실행 중인 Hook이 마칠 떄까지
대기하지 않고, 종료한다.

아래 예제를 보면 종료되기 전에 halt() 메소드를 호출하였으므로 등록된 Hook이 실행되지 않는다.
{% highlight java %}
	// ...
	
	public static void main(String[] args) {
		Runtime.getRuntime().addShutdownHook(new HookThread());
		Runtime.getRuntime().addShutdownHook(new HookThread2());

		System.out.println("End");
		Runtime.getRuntime().halt(0);
	}
}
{% endhighlight %}

## Shutdown Hook 실행되면, Hook을 실행, 삭제할 수 없음

Shutdown Hook이 시작되면 Hook을 추가할 수도 제거할 수 없다. 시도할 경우 IllegalStateException 예외가 발생한다.

(삭제 메소드는 Runtime.removeShutdownHook() )

아래 예제는 예외가 발생하는 경우이다. Shutdown Hook에 추가한 Hook Thread에 새로운 Hook을 추가하였다.
따라서 종료되면서 IllegalStateException 예외가 발생한다.

{% highlight java %}
public class ShutdownHookTest {

	static class HookThread extends Thread {
		@Override
		public void run() {
			Runtime.getRuntime().addShutdownHook(new HookThread());
			System.out.println("Hook Run1");
		}
	}
	
	// ...

}

{% endhighlight %}


## 이외 유의사항

쓰레드는 언제나 어렵다. 복잡하고 디버깅하기도 힘들다. 
Shutdown Hook도 마찬가지로 쓰레드를 사용하므로 제대로 구현하지 않는다면 원하는 데로 작동하지 않을 수 있다.

프로그램이 종료될 때 모든 쓰레드 Hook이 동시에 실행되며 이외에 쓰레드들도 실행 중일 것이다. 따라서 **thread-safe**하게 작성하여 데드락을 피해야 한다.

뿐만 아니라, 무조건 실행된다는 보장이 없으므로 Shutdown Hook 메소드를 100% 신뢰하지는 않는 것이 좋다.
정말 중요한 작업이라면 명시적인 종료자 메소드를 구현하여 사용하는 게 안전할 것이다.

마지막으로 Hook 쓰레드에서 많은 시간을 소모하면 안된다. 만약 시스템을 종료하는 상황이라면
시스템이 종료되면 JVM이 종료되면서 Hook이 실행되겠지만 시간이 오래 걸릴 경우, 시스템은 무시하고 강제로 종료돼버린다.



이상 addShutdownHook() 메소드에 대한 설명이었다.

회사 제품 소스를 보면서 너무 모르는 자바 클래스와 함수들이 많다. 이렇게 하나씩 공부하다가는 끝이 없겠지만, 얼마나 재밌는가?


>>>>>>> branch 'master' of https://github.com/knowjea/knowjea.github.io.git

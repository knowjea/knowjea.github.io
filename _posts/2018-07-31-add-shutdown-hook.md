---
layout: post
title: "JAVA Runtime.addShutdownHook() (종료될 때 특정 작업 수행)"
author: "Gyeongjae Gwon"
categories: facts
tags: [java]
image: captain-hook.jpg
---


프로그램이 정상 또는 비정상 종료하기 전에 특정 작업을 수행할 수 있도록 자바는 java.lang.Runtime.addShutdownHook(Thread t)을 제공한다.

여기서 정상 또는 비정상 종료의 정의는 명세서에 나와 있다.

{% highlight %}
자바의 가상 머신은 두 가지 종류의 이벤트를 받아 종료한다.
* 프로그램이 정상적으로 종료되거나 System.exit()
* 사용자 인터럽트(Ctrl+C)나 사용자 로그오프 또는 시스템 종료와 같은 시스템 전체 이벤트에 대한 응답 이벤트
{% endhighlight}


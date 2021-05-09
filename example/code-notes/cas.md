---
title: Принцип CAS в Java
tags:
- java
created: 2021-05-08
---

Прежде чем мы углубимся в стратегию *CAS* (Compare And Swap) и как она используется атомарными конструкциями, такими как [AtomicInteger](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html), сначала рассмотрим этот код:

```java
public class MyApp
{
    private volatile int count = 0;
    public void updateVisitors()
    {
       ++count; //increment the visitors count
    }
}
```

Этот пример кода отслеживает количество посетителей приложения. Что-то не так с этим кодом? Что произойдет, если несколько потоков попытаются обновить счетчик? На самом деле проблема в том, что volatile не гарантирует атомарности, а инкремент не является атомарной операцией.
Можем ли мы решить эту проблему, если отметим сам метод как синхронизированный, как показано ниже:

```java
public class MyApp
{
    private int count = 0;
    public synchronized void updateVisitors()
    {
       ++count; //increment the visitors count
    }
}
```

Гарантирует ли этот код атомарность? Да.

Так в чем же проблема?

Он использует блокировку, что приводит к большим задержкам и накладным расходам. Длч преодоления этой проблемы был введен пакет  `java.util.concurrent.atomic`.

```java
public class MyApp
{
    private AtomicInteger count = new AtomicInteger(0);
    public void updateVisitors()
    {
       count.incrementAndGet(); //increment the visitors count
    }
}
```

Каждый атомарный класс включает метод compareAndSet, представляющий механизм оптимистичной блокировки и позволяющий изменить значение value только в том случае, если оно равно ожидаемому значению (т.е. current).
Если значение value было изменено в другом потоке, то оно не будет равно ожидаемому значению.
Следовательно, метод compareAndSet (CAS) вернет значение false, что приведет к новой итерации цикла while в методе getAndAdd.
Таким образом, в очередном цикле в переменную current будет считано обновленное значение value, после чего будет выполнено сложение и новая попытка записи получившегося значения (т.е. next).
Переменные current и next - локальные, и, следовательно, у каждого потока свои экземпляры этих переменных.

```java
public final long incrementAndGet() {
    for (;;) {
        long current = get();
        long next = current + 1;
        if (compareAndSet(current, next))
          return next;
    }
}
```

> Метод compareAndSet реализует механизм оптимистической блокировки. Знакомые с набором команд процессоров специалисты знают, что ряд архитектур имеют инструкцию Compare-And-Swap (CAS), которая является реализацией этой самой операции. Таким образом, на уровне инструкций процессора имеется поддержка необходимой атомарной операции. На архитектурах, где инструкция не поддерживается, операции реализованы иными низкоуровневыми средствами.
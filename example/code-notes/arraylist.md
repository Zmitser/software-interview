---
title: ArrayList
tags:
- java
created: 2021-05-09
---

`ArrayList` — реализует интерфейс `List`.
Как известно, в Java массивы имеют фиксированную длину, и после того как массив создан, он не может расти или уменьшаться.
`ArrayList` может менять свой размер во время исполнения программы, при этом необязательно указывать размерность при создании объекта.
Элементы `ArrayList` могут быть абсолютно любых типов, в том числе и *null*.


Некоторые особенности `ArrayList`:

1. Поддерживает порядок вставки
2. Может содержать повторяющиеся элементы
3. Позволяет произвольный доступ к своим элементам, поскольку он работает на основе индекса
4. Не синхронизирован
5. Может содержать любое количество null элементов.

Алгоритмическая сложность операций получения (`get()`), вставки (`add()`) и удаления(`delete()`)

* `get()`: константное время
* `add()`: Элемент может быть добавлен в начало, середину и конец списка. Если использовать `add(element)`, то добавление произойдет в конец списка.
При условии, что массив не заполнен, сложность будет *O(1)*, в противном случае будет создан новый массив, размер которого в полтора раза больше предыдущего, и все элементы массива скопируются в этот новый массив.
В этом случае алгоритмическая сложность будет O(n). Если элемент добавлен в середину списка или по какому-либо определенному индексу, скажем, по индексу 2, тогда необходимо сдвинуть все элементы на одну позицию вправо. В этом случае алгоритмическая сложность будет O(n)
* `remove ()`: это то же самое, что и `add ()`, если вы хотите удалить элемент по определенному индексу, тогда все элементы справа должны быть сдвинуты на одну позицию влево, что займет O (n), но если элемент удаляется в конце списка, тогда потребуется O(1).  



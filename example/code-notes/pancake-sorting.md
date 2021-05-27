---
title: Блинная сортировка
tags:
    - algorithms
created: 2021-05-27
---

> **Блинная сортировка** (*pancake sort*) – алгоритм сортировки массива, в котором сортировка осуществляется переворотом части массива.

Идея алгоритма заключается в том, чтобы за каждый проход, переместить максимальный элемент в конец массива. Для этого необходимо выполнить следующие шаги:

1. В начале выбрать подмассив равный по длине текущему массиву;
2. Найти в нем позицию максимального элемента;
3. Если максимальный элемент расположен не в конце подмассива, то:
   1. Переворачиваем часть массива от начала до максимального значения;
   2. Переворачиваем весь выбранный подмассив;
4. Уменьшаем рабочую область массива и переходим ко второму шагу.

### Реализация блинной сортировки: 

```kotlin
fun flip(arr: Array<Int>, k: Int) {
    var j = k
    for (i in 0 until j){
        val temp = arr[i]
        arr[i] = arr[j]
        arr[j] = temp
        j--
    }
}

fun maxElementIndex(arr: Array<Int>, k: Int): Int {
    var maxIndex = 0
    for (i in 0 until k) {
        if (arr[maxIndex] < arr[i]) {
            maxIndex = i
        }
    }
    return maxIndex
}

fun pancakeSort(arr: Array<Int>): Array<Int> {
    for (i in arr.size downTo 1) {
        val maxElementIndex = maxElementIndex(arr, i)
        if (maxElementIndex != i - 1) {
            flip(arr, maxElementIndex)
            flip(arr, i - 1)
        }
    }
    return arr
}

```



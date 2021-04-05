---
title: Dependency injection. Основные понятие и пример реализации на Kotlin
tags:
  - tutorials
  - spring
created: 2021-04-04
---

Как известно, даже относительно небольшое Enterprise-приложение может состоять из множества объектов, которые активно взаимодействуют друг с другом и находятся в определенных отношениях зависимости.
В целях наиболее эффективного управления зависимостями между объектами в основу Spring Framework положен паттерн "Внедрение зависимостей" (Dependency Injection).


> Dependency Injection (внедрение зависимости) - это набор паттернов и принципов разработки программного обеспечения, которые позволяют писать слабосвязный код. В полном соответствии с принципом единой обязанности объект отдаёт заботу о построении требуемых ему зависимостей внешнему, специально предназначенному для этого общему механизму (фреймворку). 

Благодаря данному механизму код приложения становится слабосвязанным, что в будущем позволяет повторное использование кода, облегчает тестирование и замену реализации конкретных классов.

Напишем свою собственную реализацию Dependency Injection на языке программирования Kotlin.

Предположим, что у нас есть задача написать класс, который позволит онлайн-магазину оповещать пользователей о новых скидах на товары посредством отправки сообщений по электронной почте и sms.

Класс для отправки электронных писем реализуем следующим образом:

```kotlin
package by.zmitser.dependency.injection.example

interface EmailService {
    fun sendEmail(message: String, receiver: String)
}

class SimpleEmailService() : EmailService {
    override fun sendEmail(message: String, receiver: String) {
        println("Email sent to $receiver with message = '$message'")
    }
}
```

Нечто похожее мы также создадим для отправки SMS-сообщений: 

```kotlin
package by.zmitser.dependency.injection.example

interface SmsService {
    fun sendSms(message: String, phone: String)
}

class SimpleSmsService : SmsService {
    override fun sendSms(message: String, phone: String) {
        println("Sms sent to the phone number $phone with message = '$message'")
    }
}
```

Также в нашей системе будет существовать класс, который будет инкапсулировать в себе отправку сообщений в виде SMS и Email:

```kotlin
package by.zmitser.dependency.injection.example

class UserNotificationService(
    private val emailService: EmailService = SimpleEmailService(),
    private val smsService: SmsService = SimpleSmsService()
) {


    fun notify(user: User, message: String) {
        emailService.sendEmail(message, user.email)
        smsService.sendSms(message, user.phone)
    }
}
```

Ну, и конечно же, куда без метода main:

```kotlin
package by.zmitser.dependency.injection.example

fun main() {
    val  notificationService = UserNotificationService()
    notificationService.notify(User("1@2.by", "+375299169985"), "Скидка на баклажаны")
}
```

Данный код вполне будет работоспособным, но в нем явно прослеживается сильная связанность между компонентами. 
Более того, в классе UserNotificationService нарушается "Принцип единой ответственности" (Single Responsibility Principle), ибо он отвечает не только за инкапсуляцию вызовов методов SmsService и EmailService, но и за создание экземпляров классов.
Попробуем решить проблему нарушения принципа делегированием создания объектов некоторой фабрике (Object Factory):


```kotlin
package by.zmitser.dependency.injection.framework

class ObjectFactory private constructor(private val config: Config = KotlinConfig()) {

    fun <T> createObject(type: Class<T>): T {
        return config.getImplClass(type).getDeclaredConstructor().newInstance()
    }

    companion object {
        val instance = ObjectFactory()
    }
}
```

Создадим также класс, в котором будут определены некоторые конфигурационные настройки 

```kotlin
package by.zmitser.dependency.injection.framework

interface Config {

    fun <T> getImplClass(ifc: Class<T>): Class<T>
}

class KotlinConfig : Config {

    override fun <T> getImplClass(ifc: Class<T>): Class<T> {
        return this.javaClass.classLoader.loadClass(ifc.name) as Class<T>
    }
}
```

Теперь класс UserNotificationService можно переписать следующим образом:

```kotlin
package by.zmitser.dependency.injection.example

import by.zmitser.dependency.injection.framework.ObjectFactory

class UserNotificationService(
    private val emailService: EmailService = ObjectFactory.instance.createObject(SimpleEmailService::class.java),
    private val smsService: SmsService = ObjectFactory.instance.createObject(SimpleSmsService::class.java)
) {


    fun notify(user: User, message: String) {
        emailService.sendEmail(message, user.email)
        smsService.sendSms(message, user.phone)
    }
}
```
Теперь мы видим, что создание объектов было делегировано фабрике. Но UserNotificationService явно определяет, какая реализация ему нужна.
Используем библиотеку СlassGraph и перепишем код:

```kotlin
import io.github.classgraph.ClassGraph

interface Config {

    fun <T> getImplClass(ifc: Class<T>): Class<T>
}

class KotlinConfig(private val packageToScan: String, private val classGraph: ClassGraph = ClassGraph()) : Config {

    override fun <T> getImplClass(ifc: Class<T>): Class<T> {
        val result = classGraph.acceptPackages(packageToScan)
            .scan()
            .allClasses.filter {
                it.implementsInterface(ifc.name)
            }
        if (result.size != 1) {
            throw RuntimeException("$ifc has 0 or more than one impl")
        }
        return result.loadClasses(ifc).first()
    }
}
```

```kotlin
package by.zmitser.dependency.injection.framework

import Config
import KotlinConfig

class ObjectFactory private constructor(private val config: Config = KotlinConfig("by.zmitser.dependency.injection.example")) {

    fun <T> createObject(type: Class<T>): T {
        var implClass: Class<out T> = type
        if (type.isInterface) {
            implClass = config.getImplClass(type)
        }
        return implClass.getDeclaredConstructor().newInstance()
    }

    companion object {
        val instance = ObjectFactory()
    }
}
```

```kotlin
package by.zmitser.dependency.injection.example

import by.zmitser.dependency.injection.framework.ObjectFactory

class UserNotificationService(
    private val emailService: EmailService = ObjectFactory.instance.createObject(EmailService::class.java),
    private val smsService: SmsService = ObjectFactory.instance.createObject(SmsService::class.java)
) {


    fun notify(user: User, message: String) {
        emailService.sendEmail(message, user.email)
        smsService.sendSms(message, user.phone)
    }
}
```

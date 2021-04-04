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







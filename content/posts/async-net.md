+++ 
draft = true
date = 2021-03-18T09:30:06+06:00
title = "Асинхронное программирование в .NET"
description = "Конспекты по асинхронному программированию в .NET"
tags = [".net", "асинхронное программирование", "TAP", "async/await", "Task"]
categories = ["асинхронное программирование"]
+++

## Что такое асинхронный код и для чего он нужен
Асинхронность -  это выполнение программного кода, когда потоки выполнения во время ожидания ответа от блокирующих операций не блокируются, а продолжают свою работу. 

Блокирующие операциии - это как правило операции с различным переферийным оборудованием и программными сервисами, когда после запроса к ним, ответ приходит с некоторой ощутимой задержкой. 

![Разница между синхронным и асинхронным выполнением](/images/async-net/sync-async.png)  

Асинхронность и многопоточность - это не одно и тоже. 

Асинхронность - это логический способ избегать блокировок

Многопоточность - логически/физический способ паралелльного выполнения потоков

Асинхронный код для клиент-серверных дает повышение производительности, для десктопных приложений - повышение отзывчивости. 

## Эволюция асинхронных паттернов в .NET

### Asynchronous Programming Model (APM)
Класс должен организовать выполнение асинхронной операции (асинхронный паттерн). У класса есть метод Begin* и End* методы.  
![Пример 1. Asynchronous Programming Model (APM)](/images/async-net/apm.png)
![Пример 2. Asynchronous Programming Model (APM)](/images/async-net/apm2.png)

Проблемы подхода:
- Неудобство использования, много вспомогательного кода
- Коллбеки, callback hell
- В классах, реализующих такой подход, - коллбеки идут в пул потоков. 

### Task-based Asyncronous Pattern (TAP)

## Базовые кейсы по работе с TAP

## Ключевые моменты при работе с TAP

## Разбор нетипичных проблем

## Детали реализации TAP

## Источники
- [«Асинхронность в .NET — от простого к сложному». Владислав Фурдак, DataArt.](https://www.youtube.com/watch?v=OoSFGENdNPo)
- [Дмитрий Иванов — Async programming in .NET: Best practices](https://www.youtube.com/watch?v=wM-h6P1BJRk)
- [Иван Дашкевич — Yield и async-await: как оно все устроено внутри и как этим воспользоваться](https://www.youtube.com/watch?v=TFsT8bgs024)
- [Awaitable/awaiter pattern and logical micro-threading in C#](https://nikiforovall.github.io/csharp/compcsi/dotnet/2020/10/20/awaitable-pattern.html)
- [Dissecting the async methods in C#](https://devblogs.microsoft.com/premier-developer/dissecting-the-async-methods-in-c/)
- [Евгений Пешков — Многопоточность в .NET: когда производительности не хватает](https://www.youtube.com/watch?v=-tNeYjRNJtY)
- [Async/Await - Best Practices in Asynchronous Programming](https://docs.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
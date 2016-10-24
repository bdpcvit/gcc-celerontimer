========================
"CeleronTimer" C library
========================

Библиотека "Программных Таймеров" - реализация синхронных и асинхронных задержек в прошивке микроконтроллера.

Дискретность отсчёта = 1мс. Достаточна для реализации пользовательского интерфейса в устройствах на микроконтроллере.



Обоснование
-----------

Редкая прошивка обходится без таймеров. "Программные таймеры" - это мощный и универсальный инструмент. Могут быть использованы в любой архитектуре программной прошивки. Но обычно применяются в архитектурах начального уровня сложности: "Суперцикл" и "Конечный/флаговый автомат" - это большинство разрабатываемых прошивок для микроконтроллеров.

* Вводную теорию "что такое программные таймеры? зачем нужны? и как их применять?" можно почитать в Учебном курсе ["AVR. Учебный Курс. Архитектура Программ" от DI HALT.] (<http://easyelectronics.ru/avr-uchebnyj-kurs-arxitektura-programm.html>)
* Ещё одна [реализация программных таймеров] (<https://habrahabr.ru/post/273077/>) замечена на Хабре.

Но замечу: в библиотеке "CeleronTimer" совсем другая реализация (удобнее и функциональнее API, нетребовательная к ресурсам МК), чем представленные выше!


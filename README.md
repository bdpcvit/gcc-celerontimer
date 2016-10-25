========================
"CeleronTimer" C library
========================

Библиотека "Программных Таймеров" - реализация синхронных и асинхронных задержек в прошивке микроконтроллера.

Дискретность отсчёта = 1мс. Достаточна для реализации пользовательского интерфейса (и прикладной логики) в устройствах на микроконтроллере.



Документация
------------

API библиотеки описано в "заголовочном файле" в комментариях к макросам и методам (оно простое).
А чтобы знать с чего начинать, и чтобы дать представление о концепции библиотеки - смотри пример использования ниже:


**\<module1.c\>**

::

    #include "celerontimer.h"


    // Создать глобальный таймер (выделить память), который будет доступен из всех модулей программы
    DELAY_DeclareGlobalTimer(globaltimer1);

    // Создать таймер модульного уровня (выделить память), который будет доступен только для функций текущего модуля программы
    DELAY_DeclareTimer(moduletimer1);


    // Функция "конечного автомата", периодически вызываемая из Суперцикла
    foo()
    {
        // Создать локальный таймер (выделить статическую память), который будет доступен только из кода текущей функции
        DELAY_DeclareTimer(localtimer1);


        if(произошло_событие_1)
            // Запустить таймер на таймаут в 1сек
            DELAY_SetTimer(localtimer1, 1000);


        // Пока длится таймаут - виляем хвостиком (мигаем LEDиком).
        // (Светодиод на порте LED_ON: 128мс горит и 128мс выключен, 50% заполнение)
        LED_ON = DELAY_IsTimerOn(localtimer1) && 
                 (DELAY_SysTick & 0b10000000);


        // За 100мс до окончания таймаута - пикнем бузером.
        // (Бузер с генератором на порте BUZZER_ON)
        BUZZER_ON = DELAY_IsTimerOn(localtimer1) && 
                    DELAY_RemainingTime(localtimer1) < 100;


        // Если таймаут истёк?
        if(DELAY_CheckTimer(localtimer1))
        {
            // Важно: в обработчике события необходимо явно, либо выключить сработавший "одиночный таймер", либо перезапустить "периодический таймер" на следующий интервал отсчёта!
            
            // Выключить "одиночный таймер" (событие обработано)
            DELAY_DisableTimer(localtimer1);
            
            // Перезапустить "периодический таймер" на следующий интервал в 1сек (в данном случае это не нужно - код закомментирован)
            //DELAY_SetTimer(localtimer1, 1000);
        }
        
    }
    
    
    // Функции модуля легко шарят "модульные" и "глобальные" таймеры
    bar()
    {
        DELAY_SetTimer(moduletimer1, 1000);

        DELAY_SetTimer(globaltimer1, 1000);

    }



**\<module1.h\>**

::

    // Декларировать глобальный таймер - расшарить имя для других модулей программы
    DELAY_DeclareExternalTimer(globaltimer1)



**\<module2.c\>**

::

    #include "celerontimer.h"
    #include "module1.h"


    // Функциям из другого модуля доступны глобальные таймеры (если подключены соответствующие external-декларации)
    baz()
    {
        if(DELAY_IsTimerOn(globaltimer1))
            return;
            
        // Следующий код выполняется только когда "globaltimer1" неактивен...
    }




    // И хотя для "глобальных таймеров" доступны любые действия из любого модуля программы, но рекомендую, чисто концептуально, располагать код так, 
    // чтобы Таймер декларировался рядом с кодом который его "обслуживает", т.е. включает и выключает по событиям (методы: DELAY_SetTimer, DELAY_CheckTimer, DELAY_DisableTimer).
    // А в других "внешних модулях" использовать только наблюдательные действия, не изменяющие состояния таймера (методы: DELAY_IsTimerOn, DELAY_RemainingTime).
    
    qux()
    {
        if(DELAY_CheckTimer(globaltimer1))
        {
            DELAY_DisableTimer(globaltimer1);
        }
    }

    // TODO: рефакторинг, перенести функцию qux() в модуль <module1.c>




Требования
----------

Компилятор Си, совместимый с GCC.

Поддерживаются микроконтроллеры любой разрядности: 8-битные, 32-битные и другие.

*Единственное замечание для микроконтроллеров рязрядности ниже 32-бит (в которых реализована фрагментированная арифметика над 32-битными целыми): 
Обработчик прерывания "Аппаратного Таймера", отсчитывающий 1мс интервалы, и в который вы помещаете вызов DELAY_IncSysTick() - этот обработчик должен иметь "наивысший приоритет" (не перебиваться другими прерываниями) среди других обработчиков, которые также используют обращение к "Программным Таймерам".*



  > ENGLISH:
  
  GCC compatible C compiler.
  
  8-bit and 32-bit microcontrollers are supported!
  
  *Just one requirement for 8-bit microcontrollers: 1ms "Hard Timer" interrupt handler, where you place the DELAY_IncSysTick() call, must have highest (or the same) priority among other handlers, which use "Soft Timers"! To avoid fragmentation of DELAY_SysTick variable incrementation process. That is all!*




Обоснование
-----------

Редкая прошивка обходится без таймеров. "Программные таймеры" - это мощный и универсальный инструмент. Могут быть использованы в любой архитектуре программной прошивки. Но обычно применяются в архитектурах начального уровня сложности: "Суперцикл" и "Конечный/флаговый автомат" - это большинство разрабатываемых прошивок для микроконтроллеров.

* Вводную теорию "что такое программные таймеры? зачем нужны? и как их применять?" можно почитать в Учебном курсе ["AVR. Учебный Курс. Архитектура Программ" от DI HALT.] (<http://easyelectronics.ru/avr-uchebnyj-kurs-arxitektura-programm.html>)
* Ещё одна [реализация программных таймеров] (<https://habrahabr.ru/post/273077/>) замечена на Хабре.

Но замечу: в библиотеке "CeleronTimer" совсем другая реализация (удобнее и функциональнее API, нетребовательная к ресурсам МК), чем представленные выше!


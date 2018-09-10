<div class='article-menu'>
  <ul>
    <li>
      <a href="#architecture">Архитектура MVC</a> <ul>
        <li>
          <a href="#models">Модели</a>
        </li>
        <li>
          <a href="#views">Представления</a>
        </li>
        <li>
          <a href="#controllers">Контроллеры</a>
        </li>
      </ul>
    </li>
  </ul>
</div>

<a name='architecture'></a>

# Архитектура MVC

Phalcon поддерживает использование парадигмы объектно-ориентированного программирования и поддержку классов, необходимых для разделения на Модель, Представление, Контроллер (Model, View, Controller, кратко [MVC](https://en.wikipedia.org/wiki/Model–view–controller)). Этот шаблон проектирования активно используется в других веб-фреймворках и обычных приложениях.

Преимущества MVC:

- Отделение бизнес-логики от пользовательского интерфейса и работы с базой данных
- Позволяет располагать разные части в разных местах, что благоприятно сказывается на поддержке и обслуживании

Если вы выберете MVC, все запросы будут выполнятся согласно MVC архитектуре. Phalcon поддерживает MVC своими классами, написанными на C, что позволяет добиться высокой производительности PHP приложения.

<a name='models'></a>

## Модели

Модель представляет собой информацию (данные) приложения и правила работы с этими данными. В основном модель используется для задания правил взаимодействия с соответствующей таблицей в базе данных. В большинстве случаев каждая таблица в базе данных соответствует одной модели в приложении. Основная часть бизнес-логики вашего приложения будет сосредоточена в моделях. [Подробнее про модели](/[[language]]/[[version]]/models).

<a name='views'></a>

## Представления

Представление отвечает за пользовательский интерфейс вашего приложения. Чаще всего это HTML файлы с вставками PHP кода исключительно для вывода данных. Этот слой отвечает за вывод данных в веб-браузер или другой инструмент, который обращается к вашему приложению. [Подробнее про представления](/[[language]]/[[version]]/views).

<a name='controllers'></a>

## Контроллеры

Контроллеры предоставляют "клей" между моделью и представлением. Контроллеры ответственны за обработку входящих запросов от веб-браузера, запрашивание данных у модели и передачу этих данных в представление для вывода. [Подробнее про контроллеры](/[[language]]/[[version]]/controllers).
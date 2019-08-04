archive:
  ref: dispatch-events

---

# Колбэки и события на компонентах

Компоненты, хоть и каждый сам по себе, обычно как-то общаются с остальной частью страницы

Есть несколько способов, при помощи которых компоненты сообщают друг другу о важных событиях, которые в них произошли.

## Колбэки

Колбэк (от англ. callback) -- это функция, которую мы передаём куда-либо и ожидаем, что она будет вызвана при наступлении события.

Например, мы можем добавить в `options` для `Menu` новый параметр -- функцию `onselect`, которая будет вызываться при выборе пункта меню:

```js no-beautify
var menu = new Menu({
  title: "Сладости",
  template: _.template(document.getElementById('menu-template').innerHTML),
  listTemplate: _.template(document.getElementById('menu-list-template').innerHTML,
  items: {
    "donut": "Пончик",
    "cake": "Пирожное",
    "chocolate": "Шоколадка"
  },
*!*
  onselect: showSelected
*/!*
});

*!*
function showSelected(href) {
  alert(href);
}
*/!*
```

В коде меню нужно будет вызывать её, например так:

```js no-beautify
...
  function select(link) {
    options.onselect(link.getAttribute('href').slice(1));
    ...
  }
...
```

Полный пример:

[codetabs src="menu-callback" height="180"]

## Свои события

Как мы уже знаем, в современных браузерах DOM-элементы могут [генерировать произвольные события](/dispatch-events) при помощи встроенных методов, а в IE8- это возможно с использованием фреймворка, к примеру, jQuery.

Воспользуемся ими, чтобы корневой элемент меню генерировал событие, которое мы назовём `select`, при выборе элемента, и передавал в объект события выбранное значение.

Для этого модифицируем функцию `select`:

```js no-beautify
function Menu(options) {
  ...

  function select(link) {
*!*
    var widgetEvent = new CustomEvent("select", {
      bubbles: true,
      // detail - стандартное свойство CustomEvent для произвольных данных
      detail: link.getAttribute('href').slice(1)
    });
    elem.dispatchEvent(widgetEvent);
*/!*
  }

  ...
}
```

Код, который заинтересован в том, чтобы узнавать, что выбрано в меню, подписывается на событие `select` его корневого элемента:

```js
var menu = new Menu(...);

var elem = menu.getElem();

elem.addEventListener('select', function(event) {
  alert( event.detail );
});
```

Вместо `detail` можно было бы выбрать и другое название свойства, но тогда нужно позаботиться о том, чтобы оно не конфликтовало со стандартными. Кроме того, в конструкторе `CustomEvent` разрешено только `detail`, другое свойство понадобилось бы присваивать в отдельной строке.

Полный пример:

[codetabs src="menu-event"]

```warn header="Внимание, инкапсуляция!"
Очень важно, что внешний код ставит обработчик на корневой элемент, но не на внутренние элементы меню.

Строго говоря, он вообще не знает про то, как устроено меню, есть ли там ссылки и какие, или там вообще всё реализовано через кнопки.

Меню для него -- "чёрный ящик". Корневой элемент -- точка доступа к его функционалу. Событие -- не то, которое произошло на ссылке, а "переработанный вариант", интерпретация действия со стороны меню.

Такое правило позволяет нам не опасаться проблем при оптимизации, расширении и даже полной переделке DOM-структуры меню. Коль скоро события и методы сохраняются, внешний код будет работать как прежде.

Ещё раз -- внешний код не имеет права залезать внутрь DOM-структуры меню, ставить там обработчики и так далее.
```

## Итого

Для того, чтобы внешний код мог узнавать о важных событиях, произошедших внутри компонента, используются:

- Колбэки -- функции, которые передаются "снаружи" при создании компонента, и которые он обязуется вызвать при наступлении событий.
- События -- компонент генерирует их на корневом элементе при помощи `dispatchEvent`, а внешний код ставит обработчики при помощи `addEventListener`. Такие события всплывают, если указан флаг `bubbles`, поэтому с ними можно использовать делегирование.

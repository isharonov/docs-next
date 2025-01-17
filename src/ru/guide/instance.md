# Экземпляры приложения & компонента

## Создание экземпляра приложения

Создание любого приложения Vue начинается с создания нового **экземпляра приложения** с помощью функции `createApp`:

```js
const app = Vue.createApp({
  /* опции */
})
```

Экземпляр приложения используется для регистрации «глобальных» вещей, которые будут затем использоваться компонентами внутри этого приложения. Подробнее познакомимся дальше в руководстве, но в качестве небольшого примера:

```js
const app = Vue.createApp({})

app.component('SearchInput', SearchInputComponent)
app.directive('focus', FocusDirective)
app.use(LocalePlugin)
```

Большинство из методов экземпляра приложения возвращают этот же экземпляр, что позволяет составлять вызовы в цепочку:

```js
Vue.createApp({})
  .component('SearchInput', SearchInputComponent)
  .directive('focus', FocusDirective)
  .use(LocalePlugin)
```

Полный список API приложения можно посмотреть в разделе [Справочник API](../api/application-api.md).

## Корневой компонент

Опции, передаваемые в `createApp`, используются для настройки **корневого компонента**. При **монтировании** приложения он используется как стартовая точка для отрисовки.

Приложению требуется примонтироваться в DOM-элемент. Например, если потребуется примонтировать приложение Vue в `<div id="app"></div>`, то необходимо передать `#app`:

```js
const RootComponent = {
  /* опции */
}
const app = Vue.createApp(RootComponent)
const vm = app.mount('#app')
```

В отличие от большинства других методов приложения, `mount` не возвращает экземпляр приложения — вместо этого он возвращает экземпляр корневого компонента.

Vue хоть и не реализует в полной мере [паттерн MVVM](https://ru.wikipedia.org/wiki/Model-View-ViewModel), но архитектура фреймворка во многом вдохновлена им. Поэтому, как правило, переменную с экземпляром приложения называют `vm` (сокращение от ViewModel).

Пусть все примеры на этой странице и нуждаются лишь в одном компоненте, большинство реальных приложений организованы в дерево вложенных, многократно переиспользуемых компонентов. Например, дерево компонентов приложения todo-списка может быть таким:

```
Корневой компонент
└─ TodoList
   ├─ TodoItem
   │  ├─ DeleteTodoButton
   │  └─ EditTodoButton
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```

Каждый компонент будет иметь свой собственный экземпляр компонента `vm`. У некоторых компонентов, таких как `TodoItem`, вероятно, будет несколько экземпляров, отображаемых в один момент времени. Все экземпляры компонентов в этом приложении будут иметь один и тот же экземпляр приложения.

Подробнее на [системе компонентов](component-basics.md) остановимся позже. Сейчас достаточно запомнить, что корневой компонент на самом деле ничем не отличается от любого другого компонента. Опции конфигурации такие же, как и поведение соответствующего экземпляра компонента.

## Свойства экземпляра компонента

Раньше в руководстве встречались свойства `data`. Все свойства, объявленные в `data`, доступны через экземпляр компонента:

```js
const app = Vue.createApp({
  data() {
    return { count: 4 }
  }
})

const vm = app.mount('#app')

console.log(vm.count) // => 4
```

Есть и другие опции компонента, добавляющие в экземпляр компонента пользовательские свойства, например `methods`, `props`, `computed`, `inject` и `setup`. Подробнее о каждой из них поговорим далее в руководстве. Все свойства экземпляра компонента, независимо от того, как они определены, будут доступны в шаблоне компонента.

Через экземпляр компонента Vue также предоставляет доступ к некоторым встроенным свойствам, например `$attrs` и `$emit`. Такие свойства всегда именуются с префиксом `$`, чтобы избежать конфликта имён с пользовательскими свойствами.

## Хуки жизненного цикла

При создании каждый экземпляр проходит через серию шагов инициализации — например, устанавливает наблюдение за данными, компилирует шаблон, монтирует экземпляр в DOM, обновляет DOM при изменении данных. Между шагами вызываются функции, называемые **хуками жизненного цикла**, предоставляющие возможность выполнять код на определённых этапах.

Например, хук [`created`](../api/options-lifecycle-hooks.md#created) можно использовать для запуска кода после создания экземпляра:

```js
Vue.createApp({
  data() {
    return { count: 1 }
  },
  created() {
    // `this` указывает на экземпляр vm
    console.log('счётчик: ' + this.count) // => "счётчик: 1"
  }
})
```

Также есть и другие хуки, которые будут вызываться на различных этапах жизненного цикла экземпляра, например [`mounted`](../api/options-lifecycle-hooks.md#mounted), [`updated`](../api/options-lifecycle-hooks.md#updated) и [`unmounted`](../api/options-lifecycle-hooks.md#unmounted). Все хуки вызываются с контекстом `this`, указывающим на текущий активный экземпляр, который их вызвал.

:::tip Совет
Не используйте [стрелочные функции](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions/Arrow_functions) в свойствах экземпляра и в коллбэках, например `created: () => console.log(this.a)` или `vm.$watch('a', newVal => this.myMethod())`. Так как стрелочные функции не имеют собственного `this`, то `this` в коде будет обрабатываться как любая другая переменная и её поиск будет производиться в области видимости выше, до тех пор пока не будет найдена, часто приводя к ошибкам, таким как `Uncaught TypeError: Cannot read property of undefined` или `Uncaught TypeError: this.myMethod is not a function`.
:::

## Диаграмма жизненного цикла

Ниже представлена диаграмма жизненного цикла экземпляра. Необязательно запоминать всё полностью прямо сейчас, но, по мере изучения и практики разработки, будет полезно к ней обращаться.

<img src="/images/lifecycle.svg" width="840" height="auto" style="margin: 0px auto; display: block; max-width: 100%;" loading="lazy" alt="Хуки жизненного цикла экземпляра">

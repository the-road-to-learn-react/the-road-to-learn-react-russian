# Основы React

В этой главе вы познакомитесь с основами React. Она охватывает состояние и взаимодействие в компонентах, потому что статические компоненты немного скучны, не так ли? Кроме того, вы узнаете о различных способах объявления компонента и о том, как сохранять компоненты компонуемыми и повторно используемыми.

## Внутреннее состояние компонента

Внутреннее состояние компонента, также известное как локальное состояние, позволяет сохранять, изменять и удалять свойства, хранящиеся в вашем компоненте. ES6-класс компонента может использовать конструктор для инициализации внутреннего состояния компонента позже. Конструктор вызывается только один раз, когда компонент инициализируется.

Давайте покажем конструктор класса.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

# leanpub-start-insert
  constructor(props) {
    super(props);
  }
# leanpub-end-insert

  ...

}
~~~~~~~~

Компонент App наследуется от класса `Component`: об этом говорит ключевое слово `extends Component` в объявлении компонента App.

Обязательно вызвать `super(props);`: он устанавливает `this.props` в конструкторе на случай, если вы хотите получить доступ к ним оттуда. В противном случае при доступе к свойствам компонента через `this.props` из конструктора они будут иметь значение `undefined`. В дальнейшем вы узнаете больше о свойствах React.

Теперь в вашем случае начальным состоянием компонента будет список элементов с демо-данными.

{title="src/App.js",lang=javascript}
~~~~~~~~
const list = [
  {
    title: 'React',
    url: 'https://reactjs.org/',
    author: 'Jordan Walke',
    num_comments: 3,
    points: 4,
    objectID: 0,
  },
  ...
];

class App extends Component {

  constructor(props) {
    super(props);

# leanpub-start-insert
    this.state = {
      list: list,
    };
# leanpub-end-insert
  }

  ...

}
~~~~~~~~

Состояние связано с классом с помощью объекта `this`. Таким образом, вы можете получить доступ к локальному состоянию во всём компоненте. Например, его можно использовать в методе `render()`. Ранее мы использовали функцию `map` в методе `render()` со статическом списком, который был определён вне вашего компонента. Теперь мы собираемся использовать этот список из локального состояния компонента.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
# leanpub-start-insert
        {this.state.list.map(item =>
# leanpub-end-insert
          <div key={item.objectID}>
            <span>
              <a href={item.url}>{item.title}</a>
            </span>
            <span>{item.author}</span>
            <span>{item.num_comments}</span>
            <span>{item.points}</span>
          </div>
        )}
      </div>
    );
  }
}
~~~~~~~~

Список теперь является частью компонента. Он находится во внутреннем состоянии компонента. Вы можете добавлять, изменять или удалять элементы в вашем списке. Каждый раз, когда вы будете изменять состояние компонента, будет вызываться метод `render()`. Вот как вы просто можете изменить состояние и убедиться, что компонент повторно отрисовывается и отображает корректные данные из локального состояния.

Но будьте осторожны. Не изменяйте состояние напрямую. Вы должны использовать метод `setState()` для изменения состояния. Вы узнаете об этом в следующей главе.

### Упражнения:

* поэкспериментируйте с локальным состоянием
  * определите больше данных в начальном состоянии в вашем конструкторе
  * используйте состояние в методе `render()`
* узнайте подробнее [конструктор класса в ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Classes#Constructor)

## Инициализация объектов ES6

В JavaScript ES6 вы можете использовать сокращённый синтаксис для более короткой инициализации свойств объектов. Представьте себе следующую инициализацию объекта:

{title="Код",lang="javascript"}
~~~~~~~~
const name = 'Robin';

const user = {
  name: name,
};
~~~~~~~~

Когда имя свойства в объекте совпадает с именем переменной, вы можете использовать следующее:

{title="Код",lang="javascript"}
~~~~~~~~
const name = 'Robin';

const user = {
  name,
};
~~~~~~~~

В вашем приложении вы можете сделать то же самое. Имя переменной списка и имя свойства состояния используют одинаковое имя.

{title="Код",lang="javascript"}
~~~~~~~~
// ES5
this.state = {
  list: list,
};

// ES6
this.state = {
  list,
};
~~~~~~~~

Другим классным помощником являются сокращённые имена методов. В JavaScript ES6 вы можете определять методы в объекте в более лаконичной форме.

{title="Код",lang="javascript"}
~~~~~~~~
// ES5
var userService = {
  getUserName: function (user) {
    return user.firstname + ' ' + user.lastname;
  },
};

// ES6
const userService = {
  getUserName(user) {
    return user.firstname + ' ' + user.lastname;
  },
};
~~~~~~~~

И последнее, но не менее важное: вы можете использовать вычисляемые имена свойств в JavaScript ES6.

{title="Код",lang="javascript"}
~~~~~~~~
// ES5
var user = {
  name: 'Robin',
};

// ES6
const key = 'name';
const user = {
  [key]: 'Robin',
};
~~~~~~~~

Возможно, вычисляемые имена свойств пока не имеют для вас никакого смысла. Зачем они нужны? В следующей главе вы перейдёте к этапу, где вы можете использовать их для перераспределения значений по ключу динамическим способом в объекте. Это здорово для генерации таблиц поиска в JavaScript.

### Упражнения:

* поэкспериментируйте с инициализацией объекта в ES6
* узнайте подробнее, как происходит [инициализация объектов в ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Object_initializer)

## Однонаправленный поток данных

Теперь у нас есть внутреннее состояние в компоненте App. Однако вы ещё не манипулировали локальным состоянием. Состояние является статическим, а следовательно, и сам компонент. Хороший способ познакомиться с манипуляциями с состоянием — реализовать некоторое взаимодействие с компонентом.

Давайте добавим кнопку для каждого элемента в отображаемом списке. Назовём кнопку «Отбросить», она будет удалять элемент из списка. Это может быть полезно в конце концов, когда у вас есть список непрочитанных элементов и вы хотите отклонить те элементы, которые вас не интересуют.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
        {this.state.list.map(item =>
          <div key={item.objectID}>
            <span>
              <a href={item.url}>{item.title}</a>
            </span>
            <span>{item.author}</span>
            <span>{item.num_comments}</span>
            <span>{item.points}</span>
# leanpub-start-insert
            <span>
              <button
                onClick={() => this.onDismiss(item.objectID)}
                type="button"
              >
                Отбросить
              </button>
            </span>
# leanpub-end-insert
          </div>
        )}
      </div>
    );
  }
}
~~~~~~~~

Метод класса `onDismiss()` ещё не определён. Мы сделаем это чуть позже, а пока сфокусируемся на обработчике `onClick` элемента кнопки. Как вы можете видеть, метод `onDismiss()` в обработчике `onClick` заключён в другую функцию. Это стрелочная функция. Таким образом, вам доступно свойство `objectID` объекта `item` для идентификации элемента, который будет отброшен. Альтернативным способом было бы определить функцию вне обработчика `onClick` и передавать только определённую функцию обработчику. В более поздней главе будет дано подробное объяснение темы обработчиков элементов.

Вы заметили, что я использовал несколько строк для элемента кнопки? Обратите внимание: элементы с несколькими атрибутами в один момент становятся сложными для чтения, если записывать их в одну строку. Вот почему элемент кнопки записан в нескольких строк с добавлением отступов для сохранения читаемости кода. Но это не обязательно. Это только моя настоятельная рекомендация по стилю кода.

Теперь нам нужно реализовать функциональность метода `onDismiss()`. Он принимает идентификатор для отклонения элемента. Функция привязывается к классу и, таким образом, становится методом класса. Вот почему вы получаете доступ к ней с помощью `this.onDismiss()`, а не `onDismiss()`. `this` — ваш экземпляр класса. Чтобы определить метод класса `onDismiss()`, вы должны связать его в конструкторе. Привязки (bindings) будут объяснены в другой главе позже.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      list,
    };

# leanpub-start-insert
    this.onDismiss = this.onDismiss.bind(this);
# leanpub-end-insert
  }

  render() {
    ...
  }
}
~~~~~~~~

Следующим шагом нам нужно определить функциональность, бизнес-логику, в вашем классе. Методы класса определяются следующим образом.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      list,
    };

    this.onDismiss = this.onDismiss.bind(this);
  }

# leanpub-start-insert
  onDismiss(id) {
    ...
  }
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

Теперь вы определите, что происходит внутри метода класса. В основном вы хотите удалить элемент с определённым идентификатором из списка и сохранить обновлённый список в вашем локальном состоянии. После этого обновлённый список запустит повторный вызов метода `render()` для его отображения. Удалённый элемент больше не должен появляться.

Вы можете удалить элемент из списка с помощью встроенной JavaScript-функции `filter` в JavaScript. Эта функция принимает функцию в качестве входного параметра. Эта функция имеет доступ к каждому значению в списке, потому что выполняет итерацию по списку. Таким образом, вы можете проверить каждый элемент списка, основываясь на условии фильтрации. Если проверка вычисляется как true, то элемент остаётся в списке. В противном случае он будет исключён из результата. Кроме того, хорошо знать, что эта функция возвращает новый список и не изменяет старый. Она поддерживает соглашение React о наличии неизменяемых структур данных.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
# leanpub-start-insert
  const updatedList = this.state.list.filter(function isNotId(item) {
    return item.objectID !== id;
  });
# leanpub-end-insert
}
~~~~~~~~

На следующем шаге вы можете извлечь функцию и передать её функции `filter`.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
# leanpub-start-insert
  function isNotId(item) {
    return item.objectID !== id;
  }

  const updatedList = this.state.list.filter(isNotId);
# leanpub-end-insert
}
~~~~~~~~

Кроме того, вы можете сделать это более кратко, снова используя стрелочные функции из JavaScript ES6.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
# leanpub-start-insert
  const isNotId = item => item.objectID !== id;
  const updatedList = this.state.list.filter(isNotId);
# leanpub-end-insert
}
~~~~~~~~

Вы могли бы даже снова встроить выражение, по аналогии с обработчиком `onClick` кнопки, но это может стать менее читабельно.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
# leanpub-start-insert
  const updatedList = this.state.list.filter(item => item.objectID !== id);
# leanpub-end-insert
}
~~~~~~~~

Сейчас нажатие на кнопку удаляет содержащий её элемент. Однако состояние ещё не обновлено. Поэтому вы можете, наконец, использовать метод класс `setState()` для обновления списка во внутреннем состоянии компонента.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
  const isNotId = item => item.objectID !== id;
  const updatedList = this.state.list.filter(isNotId);
# leanpub-start-insert
  this.setState({ list: updatedList });
# leanpub-end-insert
}
~~~~~~~~

Теперь снова запустите своё приложение и попробуйте нажать на кнопку «Отбросить». Это должно работать. Теперь вы получили представление, что такое *однонаправленный поток данных* в React. Вы активируете действие в своём представлении с помощью `onClick()`, функция или метод класса изменяет внутреннее состояние компонента и метод `render()` компонента снова вызывается для обновления представления.

### Упражнения:

* узнайте больше про [состояние и жизненный цикл в React](https://ru.react.js.org/docs/state-and-lifecycle.html)

## Привязки

При использовании компонентов класса React ES6 важно изучить привязки в JavaScript-классах. В предыдущей главе вы связали метод класса `onDismiss()` в конструкторе.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      list,
    };

    this.onDismiss = this.onDismiss.bind(this);
  }

  ...
}
~~~~~~~~

Зачем вам это делать в первую очередь? Этап привязки необходим, потому что методы класса автоматически не привязывают `this` к экземпляру класса. Давайте продемонстрируем это с помощью следующего ES6-класса компонента.

{title="Код",lang=javascript}
~~~~~~~~
class ExplainBindingsComponent extends Component {
  onClickMe() {
    console.log(this);
  }

  render() {
    return (
      <button
        onClick={this.onClickMe}
        type="button"
      >
        Нажми на меня
      </button>
    );
  }
}
~~~~~~~~

Компонент отрисовывается просто отлично, но при нажатии на кнопку, вы получите `undefined` в консоли разработчика. Это основной источник багов при использовании React. Если вы хотите получить доступ к `this.state` в своём методе класса, его нельзя получить, поскольку `this` не определён. Поэтому для того, чтобы сделать `this` доступным в методах класса, вам нужно привязать их к `this`.

В следующем классе компонента метод класса правильно привязан в конструкторе класса.

{title="Код",lang=javascript}
~~~~~~~~
class ExplainBindingsComponent extends Component {
# leanpub-start-insert
  constructor() {
    super();

    this.onClickMe = this.onClickMe.bind(this);
  }
# leanpub-end-insert

  onClickMe() {
    console.log(this);
  }

  render() {
    return (
      <button
        onClick={this.onClickMe}
        type="button"
      >
        Нажми на меня
      </button>
    );
  }
}
~~~~~~~~

Если попробовать нажать на кнопку ещё один раз, объект `this` должен быть определён конкретным экземпляром класса, и теперь вы сможете получить доступ к `this.state` или `this.props`, о котором вы узнаете позже.

Привязка метода класса может также произойти в другом месте. Например, это может быть в методе `render()`.

{title="Код",lang=javascript}
~~~~~~~~
class ExplainBindingsComponent extends Component {
  onClickMe() {
    console.log(this);
  }

  render() {
    return (
      <button
# leanpub-start-insert
        onClick={this.onClickMe.bind(this)}
# leanpub-end-insert
        type="button"
      >
        Нажми на меня
      </button>
    );
  }
}
~~~~~~~~

Но вам не следует так делать, потому что при таком подходе метод будет привязываться каждый раз при вызове `render()`. По сути, он выполняется каждый раз при обновлении компонентов, что влияет на производительность. При привязке метода класса в конструкторе, вы привязываете метод только один раз в самом начале создания компонента. Это лучший подход для привязки.

И ещё кое-что, что люди иногда придумывают — определение бизнес-логики методов класса в конструкторе.

{title="Код",lang=javascript}
~~~~~~~~
class ExplainBindingsComponent extends Component {
  constructor() {
    super();

# leanpub-start-insert
    this.onClickMe = () => {
      console.log(this);
    }
# leanpub-end-insert
  }

  render() {
    return (
      <button
        onClick={this.onClickMe}
        type="button"
      >
        Нажми на меня
      </button>
    );
  }
}
~~~~~~~~

Вы также не должны делать этого, потому что со временем эти методы будут загромождать конструктор. Конструктор должен служить для создания экземпляра класса со всеми его свойствами. Вот именно по этой причине методы следует определять вне конструктора.

{title="Код",lang=javascript}
~~~~~~~~
class ExplainBindingsComponent extends Component {
  constructor() {
    super();

    this.doSomething = this.doSomething.bind(this);
    this.doSomethingElse = this.doSomethingElse.bind(this);
  }

  doSomething() {
    // сделать что-то
  }

  doSomethingElse() {
    // сделать что-то ещё
  }

  ...
}
~~~~~~~~

И последнее, но немаловажное, о чём стоит упомянуть — методы класса могут автоматически привязываться, без явной привязки с использованием стрелочных функций в JavaScript ES6.

{title="Код",lang=javascript}
~~~~~~~~
class ExplainBindingsComponent extends Component {
  onClickMe = () => {
    console.log(this);
  }

  render() {
    return (
      <button
        onClick={this.onClickMe}
        type="button"
      >
        Нажми на меня
      </button>
    );
  }
}
~~~~~~~~

Если повторное связывание в конструкторе вас раздражает, вы можете использовать этот способ. Поскольку официальная документация React придерживается использования привязок метода в конструкторе, мы также последуем этому в книге.

### Упражнения:

* попробуйте различные подходы к привязкам и выведите на консоль объект `this`, используя `console.log`.

## Обработчик событий

Данная глава должна дать вам более глубокое понимание обработчиков событий в элементах. В вашем приложении вы используете следующий элемент кнопки для удаления элемента из списка.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

<button
  onClick={() => this.onDismiss(item.objectID)}
  type="button"
>
  Отбросить
</button>

...
~~~~~~~~

Это уже сложный вариант использования, потому что вам нужно передать значение методу класса, и, таким образом, вам нужно обернуть его в другую (стрелочную) функцию. Поэтому в основном это должна быть функция, которая передаётся обработчику события. Следующий код не будет работать, поскольку метод выполняется сразу же при открытии приложения в браузере.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

<button
  onClick={this.onDismiss(item.objectID)}
  type="button"
>
  Отбросить
</button>

...
~~~~~~~~

При использовании `onClick={doSomething()}` функция `doSomething()` будет выполняться немедленно при открытии приложения в вашем браузере. Выполнится выражение в обработчике и поскольку возвращаемое значение больше не будет функцией, ничего не произойдёт при повторном нажатии кнопки в дальнейшем. Но при использовании `onClick={doSomething}` поскольку `doSomething` — функция, она будет выполняться только при нажатии на кнопку. Те же правила применяются к используемому в вашем приложении методу `onDismiss()`.

Однако, просто использовать  `onClick={this.onDismiss}` недостаточно, потому что каким-то образом  нужно передать методу класса свойство `item.objectID` для идентификации элемента, который будет отброшен. Вот почему он может быть завернут в другую функцию, чтобы проникнуть в свойство. Данная концепция называется функциями высшего порядка в JavaScript и будет объяснена вкратце позже.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

<button
  onClick={() => this.onDismiss(item.objectID)}
  type="button"
>
  Отбросить
</button>

...
~~~~~~~~

Обходный путь — определить функцию-обёртку где-то снаружи и передать только определённую функцию обработчику. Поскольку для этого требуется доступ к отдельному элементу, она должна находиться внутри блока функции map.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
        {this.state.list.map(item => {
# leanpub-start-insert
          const onHandleDismiss = () =>
            this.onDismiss(item.objectID);
# leanpub-end-insert

          return (
            <div key={item.objectID}>
              <span>
                <a href={item.url}>{item.title}</a>
              </span>
              <span>{item.author}</span>
              <span>{item.num_comments}</span>
              <span>{item.points}</span>
              <span>
                <button
# leanpub-start-insert
                  onClick={onHandleDismiss}
# leanpub-end-insert
                  type="button"
                >
                  Отбросить
                </button>
              </span>
            </div>
          );
        }
        )}
      </div>
    );
  }
}
~~~~~~~~

В конце концов, это должна быть функция, которая передаётся обработчику элемента. В качестве примера попробуйте использовать следующий код:

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
        {this.state.list.map(item =>
            ...
            <span>
              <button
# leanpub-start-insert
                onClick={console.log(item.objectID)}
# leanpub-end-insert
                type="button"
              >
                Отбросить
              </button>
            </span>
          </div>
        )}
      </div>
    );
  }
}
~~~~~~~~

Она будет выполнена при открытии приложения в браузере, но не при нажатии кнопки. В то время как следующий код запускается только при нажатии кнопки. Эта функция выполняется при запуске обработчика.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

<button
# leanpub-start-insert
  onClick={function () {
    console.log(item.objectID)
  }}
# leanpub-end-insert
  type="button"
>
  Отбросить
</button>

...
~~~~~~~~

Чтобы сделать этот обработчик короче, вы можете снова преобразовать его в стрелочную функцию из JavaScript ES6. Это то, что мы также сделали с методом `onDismiss()`.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

<button
# leanpub-start-insert
  onClick={() => console.log(item.objectID)}
# leanpub-end-insert
  type="button"
>
  Отбросить
</button>

...
~~~~~~~~

Часто новички в React испытывают трудности с темой использования функций в обработчиках событий. Вот почему я попытался объяснить это более подробно. В итоге вы получите следующий код для вашей кнопки, который состоит из краткой однострочной стрелочной функции JavaScript ES6 и имеет доступ к свойству `objectID` объекта `item`.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
  ...

  render() {
    return (
      <div className="App">
        {this.state.list.map(item =>
          <div key={item.objectID}>
            ...
            <span>
# leanpub-start-insert
              <button
                onClick={() => this.onDismiss(item.objectID)}
                type="button"
              >
                Отбросить
              </button>
# leanpub-end-insert
            </span>
          </div>
        )}
      </div>
    );
  }
}
~~~~~~~~

Другая довольно актуальная тема, связанная с производительностью — это последствия использования стрелочных функций в обработчиках событий. Например, обработчик `onClick` для метода `onDismiss()` оборачивает метод в другую стрелочную функции для передачи идентификатора элемента. Поэтому каждый раз при выполнении метода `render()`, обработчик создаёт экземпляр стрелочной функции высшего порядка и запускает его. Это *может* влиять на производительность приложения, но в большинстве случаев вы не заметите разницы. Представьте, что у вас есть огромная таблица с 1000 элементами, и у каждой строки или столбца есть такая стрелочная функция как обработчик события. Здесь стоит думать о влиянии на производительность, и поэтому вы можете реализовать отдельный компонент Button для привязки метода в конструкторе. Но прежде чем это произойдёт, это будет преждевременная оптимизация. Целесообразнее сосредоточиться на изучении React.

### Упражнения:

* попробуйте различные подходы к использованию функций в обработчике `onClick` вашей кнопки

## Взаимодействия с формами и событиями

Давайте добавим ещё одно взаимодействие в приложение для получения опыта работы с формами и событиями в React. Следующее взаимодействие — это функциональность поиска. Поле для поиска будет использоваться для временной фильтрации списка на основе свойства title в элементе.

На первом этапе нужно определить форму с полем для ввода в JSX.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
# leanpub-start-insert
        <form>
          <input type="text" />
        </form>
# leanpub-end-insert
        {this.state.list.map(item =>
          ...
        )}
      </div>
    );
  }
}
~~~~~~~~

В следующем сценарии вы вводите текст в поле ввода и список временно фильтруется по введённой строке. Для возможности фильтровать список на основе значения из поля ввода, нужно сохранить значение в локальном состоянии. Но как получить доступ к значению? Вы можете использовать *синтетические события (synthetic events)* в React для доступа к данным (payload) события.

Давайте определим обработчик `onChange` для поля ввода.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
        <form>
# leanpub-start-insert
          <input
            type="text"
            onChange={this.onSearchChange}
          />
# leanpub-end-insert
        </form>
        ...
      </div>
    );
  }
}
~~~~~~~~

Функция снова привязана к компоненту и, следовательно, к методу класса. Теперь вам нужно определить и привязать этот метод.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      list,
    };

# leanpub-start-insert
    this.onSearchChange = this.onSearchChange.bind(this);
# leanpub-end-insert
    this.onDismiss = this.onDismiss.bind(this);
  }

# leanpub-start-insert
  onSearchChange() {
    ...
  }
# leanpub-end-insert

  ...
}
~~~~~~~~

При использовании обработчика в элементе вы получаете доступ к синтетическому событию React в объявлении колбэка.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

# leanpub-start-insert
  onSearchChange(event) {
# leanpub-end-insert
    ...
  }

  ...
}
~~~~~~~~

У события есть значение поля ввода в его объекте `target`. Следовательно, вы можете обновить локальное состояние с помощью поисковой строки, снова используя `this.setState()`.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  onSearchChange(event) {
# leanpub-start-insert
    this.setState({ searchTerm: event.target.value });
# leanpub-end-insert
  }

  ...
}
~~~~~~~~

Кроме того, важно не забыть определить начальное состояние для свойства `searchTerm` в конструкторе. Поле ввода должно быть пустым в начале, и поэтому значение будет пустой строкой.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      list,
# leanpub-start-insert
      searchTerm: '',
# leanpub-end-insert
    };

    this.onSearchChange = this.onSearchChange.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
  }

  ...
}
~~~~~~~~

Теперь вы сохраняете входное значение во внутреннем состоянии компонента каждый раз при изменении поля ввода.

Краткая заметка об обновлении локального состояния в компоненте React. Было бы справедливо предположить, что при обновлении `searchTerm` с использованием `this.setState()` необходимо также передать список для сохранения его. Но это не совсем так. `this.setState()` в React выполняет поверхностное (неглубокое) слияние объектов (shallow merge). Он сохраняет свойства на одном уровне в объекте состояния при обновлении одного-единственного свойства в нём. Таким образом, состояние списка, даже если вы отбросили элемент из него, останется таким же при обновлении свойства `searchTerm`.

Давайте вернёмся к нашему приложению. Список ещё не фильтрируется на основе значения из поля ввода, хранящегося в локальном состоянии. По сути вам нужно временно фильтровать список, основываясь на значении `searchTerm`. У вас есть всё необходимое для этого. Итак, а как временно фильтровать список? В методе `render()` перед использованием функции `map` на списке можно применить к нему фильтр. Фильтрация будет проходить только в случае, если `searchTerm` соответствует свойству title элемента. Ранее вы использовали встроенную в JavaScript функцию `filter`, поэтому давайте применим её ещё раз. Можно использовать функцию фильтрации перед функцией `map`, потому что она возвращает новый массив, и тем самым функция map может использоваться таким удобным способом.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
        <form>
          <input
            type="text"
            onChange={this.onSearchChange}
          />
        </form>
# leanpub-start-insert
        {this.state.list.filter(...).map(item =>
# leanpub-end-insert
          ...
        )}
      </div>
    );
  }
}
~~~~~~~~

Давайте на этот раз применим функцию фильтрации по-другому. Мы хотим определить вне класса компонента аргумент фильтра и функцию, которая передаётся функции `filter`. Вне компонента у нас нет доступа к его состоянию и поэтому у нас нет доступа к свойству `searchTerm` для выполнения условия фильтра. Мы должны передать `searchTerm` в функцию фильтра и должны вернуть новую функцию для проверки условия. Это называется функцией высшего порядка (higher-order function).

Обычно я бы не упомянул функции высшего порядка, но в книге про React в этом определённо есть смысл, потому что React связан с концепцией, называемой компонентами высшего порядка. Вы познакомитесь с этой концепцией позже в этой книге. А пока давайте сосредоточимся на функциональности фильтра.

Во-первых, определим функцию высшего порядка вне компонента App.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
function isSearched(searchTerm) {
  return function(item) {
    // условие, возвращающее true или false
  }
}
# leanpub-end-insert

class App extends Component {

  ...

}
~~~~~~~~

Функция принимает `searchTerm` и возвращает другую функцию, потому что в конце концов функция `filter` принимает функцию в качестве входного параметра. Возвращаемая функция имеет доступ к объекту элемента, так как она была передана функции `filter`. Кроме того, возвращаемая функция будет использоваться для фильтрация списка на основе условия, определённого в функции. Давайте теперь определим это условие.

{title="src/App.js",lang=javascript}
~~~~~~~~
function isSearched(searchTerm) {
  return function(item) {
# leanpub-start-insert
    return item.title.toLowerCase().includes(searchTerm.toLowerCase());
# leanpub-end-insert
  }
}

class App extends Component {

  ...

}
~~~~~~~~

В условии утверждается, что сопоставляется входящий шаблон (строка) `searchTerm` со свойством title элемента списка. Вы можете сделать это с помощью встроенной функции JavaScript `includes`. Только когда шаблон совпадает, возвращается true и элемент остаётся в списке, иначе элемент удаляется из списка. Но будьте осторожны с сопоставлением по шаблону: нужно помнить про нижний регистр обеих строк. В противном случае между строкой поиска 'redux' и элементом с заголовком 'Redux' не будет соответствия. Поскольку мы работаем над неизменяемым списком и возвращаем новый список с помощью фильтрации, исходный список в локальном состоянии не изменяется вовсе.

Осталось только упомянуть: мы немного сжульничали, использовав встроенную функцию JavaScript `includes`. Это функция из ES6. Как она будет выглядеть в JavaScript ES5? Вы можете использовать функцию `indexOf()` для получения индекса элемента в списке. Если элемент находится в списке, `indexOf()` вернёт его индекс в массиве.

{title="Код",lang="javascript"}
~~~~~~~~
// ES5
string.indexOf(pattern) !== -1

// ES6
string.includes(pattern)
~~~~~~~~

Ещё один элегантный рефакторинг можно сделать с помощью стрелочных функций ES6. Это сделает функцию более лаконичной:

{title="Код",lang="javascript"}
~~~~~~~~
// ES5
function isSearched(searchTerm) {
  return function(item) {
    return item.title.toLowerCase().indexOf(searchTerm.toLowerCase()) !== -1;
  }
}

// ES6
const isSearched = searchTerm => item =>
  item.title.toLowerCase().includes(searchTerm.toLowerCase());
~~~~~~~~

Можно поспорить, какая из функций более удобочитаемая. Лично я предпочитаю вторую. В экосистеме React используются множество концепций функционального программирования. Вы часто будете использовать функцию, возвращающую другую функцию (функции высшего порядка). В JavaScript ES6 вы можете написать это более кратко с помощью стрелочных функций.

И последнее, но не менее важное: вам нужно использовать определённую функцию `isSearched()` для фильтрации списка. Вы передаёте ей свойство `searchTerm` из локального состояния, она возвращает функцию фильтрации входных данных и фильтрует список на основе условия фильтра. После этого функция `map`, применённая на отфильтрованном списке, отображает элемент для каждого элемента списка.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    return (
      <div className="App">
        <form>
          <input
            type="text"
            onChange={this.onSearchChange}
          />
        </form>
# leanpub-start-insert
        {this.state.list.filter(isSearched(this.state.searchTerm)).map(item =>
# leanpub-end-insert
          ...
        )}
      </div>
    );
  }
}
~~~~~~~~

Теперь функциональность поиска должна работать. Попробуйте сами в браузере.

### Упражнения:

* узнайте больше про [события React](https://ru.react.js.org/docs/handling-events.html)
* узнайте подробнее [функции высшего порядка](https://ru.wikipedia.org/wiki/%D0%A4%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F_%D0%B2%D1%8B%D1%81%D1%88%D0%B5%D0%B3%D0%BE_%D0%BF%D0%BE%D1%80%D1%8F%D0%B4%D0%BA%D0%B0)

## Деструктуризация ES6

В JavaScript ES6 есть способ упросить доступ к свойствам объектов и массивов. Он называется деструктуризацией (destructuring). Сравните следующий фрагмент код в JavaScript ES5 и ES6.

{title="Код",lang="javascript"}
~~~~~~~~
const user = {
  firstname: 'Robin',
  lastname: 'Wieruch',
};

// ES5
var firstname = user.firstname;
var lastname = user.lastname;

console.log(firstname + ' ' + lastname);
// выведет: Robin Wieruch

// ES6
const { firstname, lastname } = user;

console.log(firstname + ' ' + lastname);
// выведет: Robin Wieruch
~~~~~~~~

Тогда как в JavaScript ES5 каждый раз требуется новая строка для получения значения свойства объекта, в JavaScript ES6 это можно сделать в одну строку. Передовая практика для читабельности — использование несколько строк при деструктуризации нескольких свойств объекта.

{title="Код",lang="javascript"}
~~~~~~~~
const {
  firstname,
  lastname
} = user;
~~~~~~~~

То же самое относится и к массивами: их то же можно деструктуризировать. Опять же, благодаря использованию нескольких строк код останется читабельным и лёгким для понимания.

{title="Код",lang="javascript"}
~~~~~~~~
const users = ['Robin', 'Andrew', 'Dan'];
const [
  userOne,
  userTwo,
  userThree
] = users;

console.log(userOne, userTwo, userThree);
// выведет: Robin Andrew Dan
~~~~~~~~

Возможно, вы заметили, что таким же образом можно применить деструктуризацию к объекту локального состояния компонента App. Это сократит длину строки кода в функциях `filter` и `map`.

{title="src/App.js",lang=javascript}
~~~~~~~~
  render() {
# leanpub-start-insert
    const { searchTerm, list } = this.state;
# leanpub-end-insert
    return (
      <div className="App">
        ...
# leanpub-start-insert
        {list.filter(isSearched(searchTerm)).map(item =>
# leanpub-end-insert
          ...
        )}
      </div>
    );
~~~~~~~~

Далее показаны различия использования ES5 и ES6 на одном и том же примере:

{title="Код",lang="javascript"}
~~~~~~~~
// ES5
var searchTerm = this.state.searchTerm;
var list = this.state.list;

// ES6
const { searchTerm, list } = this.state;
~~~~~~~~

Данная книга будет использовать в основном JavaScript ES6, поэтому вам стоит придерживаться этого выбора.

### Упражнения:

* изучите подробнее [деструктуризацию в ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

## Контролируемые компоненты

Вы уже узнали об однонаправленном потоке данных в React. Этот же принцип применяется к полю ввода, которое обновляет локальное состояние в зависимости от значения `searchTerm` для фильтрации списка. При изменении состояния, метод `render()` выполняется снова и используется последнее значение `searchTerm` из локального состояния для проверки условия фильтра.

Но разве мы не забыли что-то в элементе поля ввода? У тега HTML input есть атрибут `value`. Значение атрибута, как правило, эквивалентно значению, которое отображается в самом поле ввода. В нашем случае этим значением будет `searchTerm`. Однако, похоже, нам не требуется подобное в React.

Это неправильно. Элементы формы, такие как `<input>`, `<textarea>` и `<select>` хранят своё состояние в обычном HTML. Они изменяют значение внутри, когда кто-то изменяет его извне. В React это называется **неконтролируемым компонентом (uncontrolled component)**, потому что он (сам) обрабатывает своё собственное состояние. В React вы должны убедиться, что эти элементы  **контролируются компонентами**.

Как мы можем этого добиться? Нужно только установить атрибут со значением поля ввода. Значение уже сохранено в свойстве локального состояния `searchTerm`. Так почему бы нам не использовать его?

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, list } = this.state;
    return (
      <div className="App">
        <form>
          <input
            type="text"
# leanpub-start-insert
            value={searchTerm}
# leanpub-end-insert
            onChange={this.onSearchChange}
          />
        </form>
        ...
      </div>
    );
  }
}
~~~~~~~~

Вот и всё. Теперь цикл однонаправленного потока данных для поля ввода — замкнутый. Внутреннее состояние компонента — единственный источник информации для поля ввода.

Всё это управление внутренним состоянием и однонаправленным источником данных, возможно, в новинку для вас. Но как только вы привыкнете к этому всему, это станет естественным потоком данных при реализации приложений на React. В целом, React с однонаправленным потоком данных принёс новый шаблон в мир одностраничных приложений. В настоящее время эта концепция принята несколькими фреймворками и библиотеками.

### Упражнения:

* узнайте больше о [формах в React](https://ru.react.js.org/docs/forms.html)
* изучите подробнее [различные контролируемые компоненты](https://github.com/the-road-to-learn-react/react-controlled-components-examples)

## Разделение компонентов

Сейчас у вас один огромный компонент App, который продолжает расти и в один момент в нём можно будет запутаться. Давайте начнём разделять его на небольшие компоненты.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, list } = this.state;
    return (
      <div className="App">
# leanpub-start-insert
        <Search />
        <Table />
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Вы можете передавать только те свойства компонентов, которые они сами используют. В случае компонента App, ему необходимо передать свойства, управляемые локальным состоянием и его методами класса.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, list } = this.state;
    return (
      <div className="App">
# leanpub-start-insert
        <Search
          value={searchTerm}
          onChange={this.onSearchChange}
        />
        <Table
          list={list}
          pattern={searchTerm}
          onDismiss={this.onDismiss}
        />
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Теперь вы можете определить компоненты рядом с вашим компонентом App. Эти компоненты также будут классовыми компонентами. Они отрисовывают те же самые элементы, как и раньше.

Первый из них — компонент Search.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
  ...
}

# leanpub-start-insert
class Search extends Component {
  render() {
    const { value, onChange } = this.props;
    return (
      <form>
        <input
          type="text"
          value={value}
          onChange={onChange}
        />
      </form>
    );
  }
}
# leanpub-end-insert
~~~~~~~~

Второй — компонент Table.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

# leanpub-start-insert
class Table extends Component {
  render() {
    const { list, pattern, onDismiss } = this.props;
    return (
      <div>
        {list.filter(isSearched(pattern)).map(item =>
          <div key={item.objectID}>
            <span>
              <a href={item.url}>{item.title}</a>
            </span>
            <span>{item.author}</span>
            <span>{item.num_comments}</span>
            <span>{item.points}</span>
            <span>
              <button
                onClick={() => onDismiss(item.objectID)}
                type="button"
              >
                Отбросить
              </button>
            </span>
          </div>
        )}
      </div>
    );
  }
}
# leanpub-end-insert
~~~~~~~~

Итак, теперь у вас есть три класса компонентов. Возможно, вы обратили внимание на объект `props`, доступный через экземпляр класса, используя `this`. `props` — это сокращённая форма для свойств (properties), они хранят все значения, которые вы передали компонентам, когда использовали их в компоненте App. Таким образом, компоненты могут передавать свойства через дерево компонентов.

Путём выделения этих компонентов из компонента App, они стали повторно используемыми где-нибудь ещё. Поскольку эти компоненты получают свои значения с помощью объекта `props`, вы можете передавать различные свойства компонентам каждый раз, когда используете их где-то ещё.

### Упражнения:

* подумайте, какие компоненты вы ещё могли бы разделить, как это было сделано с компонентами Search и Table
  * но не делайте это сейчас, иначе вы столкнётесь с конфликтами в следующих главах

## Компонуемые компоненты

Существует ещё одно небольшое свойство, доступное в объекте `props` — `children`. Вы можете использовать его для передачи элементов вашим компонентам сверху, которые неизвестны самому компоненту, что позволяет образовывать компоненты друг из друга. Давайте посмотрим, как это выглядит при передаче строки текста в качестве дочернего элемента в компонент Search.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, list } = this.state;
    return (
      <div className="App">
# leanpub-start-insert
        <Search
          value={searchTerm}
          onChange={this.onSearchChange}
        >
          Поиск
        </Search>
# leanpub-end-insert
        <Table
          list={list}
          pattern={searchTerm}
          onDismiss={this.onDismiss}
        />
      </div>
    );
  }
}
~~~~~~~~

Теперь компонент Search может деструктурировать свойство `children` из объекта `props`, а затем указать, где он должен отображаться.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Search extends Component {
  render() {
# leanpub-start-insert
    const { value, onChange, children } = this.props;
# leanpub-end-insert
    return (
      <form>
# leanpub-start-insert
        {children} <input
# leanpub-end-insert
          type="text"
          value={value}
          onChange={onChange}
        />
      </form>
    );
  }
}
~~~~~~~~

Теперь текст "Поиск" будет показываться рядом с полем ввода. И в случае, если вы используете компонент Search где-нибудь ещё, вы можете задать другой текст, если хотите. В конце концов, необязательно в качестве children должен быть простой текст. Вы можете передать элемент или дерево элементов (которые снова могут быть инкапсулированы компонентами) в свойство children. Свойство children также позволяет вкладывать (weave) компоненты друг в друга.

### Упражнения:

* прочитайте подробнее про [модель композиции React](https://ru.react.js.org/docs/composition-vs-inheritance.html)

## Повторно используемые компоненты

Повторно используемые и составные компоненты предоставляют возможность создавать иерархии компонентов. Они — основа слоя представления в React. В последних главах говорилось о повторном использовании. Теперь вы можете повторно использовать компоненты Table и Search. Даже компонент App можно повторно применять, создав экземпяляр его где-нибудь в другом месте.

Давайте определим ещё один повторно используемый компонент — Button, который, в конечном итоге, будет повторно использоваться почаще.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Button extends Component {
  render() {
    const {
      onClick,
      className,
      children,
    } = this.props;

    return (
      <button
        onClick={onClick}
        className={className}
        type="button"
      >
        {children}
      </button>
    );
  }
}
~~~~~~~~

Может показаться лишним объявлять подобный компонент. Вы будете использовать компонент `Button` вместо элемента `button`. Он сохранит только `type="button"`. За исключением типа атрибута, вам потребуется определить всё остальное при использовании компонента Button. Но вы должны подумать здесь о долгосрочных инвестициях. Представьте, что у вас есть несколько кнопок в вашем приложении, но вам нужно изменить атрибут, стиль или поведение для кнопки. Без компонента вам придётся рефакторить каждую кнопку. Вместо этого компонент Button гарантирует наличие только одного источника истины. Один компонент Button для рефакторинга всех кнопок одновременно. Один компонент Button — это как одна кнопка для управлениям всем необходимым ей.

Поскольку у вас уже есть элемент кнопки, вы можете использовать компонент Button вместо него. При использовании этого компонента нам не нужно задавать тип атрибута, поскольку в компоненте уже он указан.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Table extends Component {
  render() {
    const { list, pattern, onDismiss } = this.props;
    return (
      <div>
        {list.filter(isSearched(pattern)).map(item =>
          <div key={item.objectID}>
            <span>
              <a href={item.url}>{item.title}</a>
            </span>
            <span>{item.author}</span>
            <span>{item.num_comments}</span>
            <span>{item.points}</span>
            <span>
# leanpub-start-insert
              <Button onClick={() => onDismiss(item.objectID)}>
                Отбросить
              </Button>
# leanpub-end-insert
            </span>
          </div>
        )}
      </div>
    );
  }
}
~~~~~~~~

Компонент Button ожидает свойство `className` в `props`. Атрибут `className` представляет соответствующий HTML атрибут для класса. Но мы не передали имя класса в `className` при использовании компонента Button. В коде компонента Button должно быть явно показано, что `className` необязательный, поэтому при деструктуризации указывается пустая строка в качестве значения по умолчанию.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Button extends Component {
  render() {
    const {
      onClick,
# leanpub-start-insert
      className = '',
# leanpub-end-insert
      children,
    } = this.props;

    ...
  }
}
~~~~~~~~

Теперь каждый раз, когда нет свойства `className` при использовании компонента Button, значение будет пустой строкой вместо `undefined`.

## Объявления компонентов

На текущий момент у вас есть четыре класса компонентов. Но их можно объявить по-другому. Позвольте мне представить функциональные компоненты без состояния в качестве альтернативы классовым компонентам. Прежде чем заняться рефакторингом ваших компонентов, давайте рассмотрим различные типы компонентов в React.

* **Функциональные компоненты без состояния (Functional Stateless Components):** Это компоненты, представленные в виде функций, которые получают входные данные и возвращают вывод. Входные данные — это `props`, а вывод — экземпляр компонента, то есть простой JSX. До сих пор он очень похож на классовый компонент ES6. Однако, функциональные компоненты без состояния — это функции (функциональный), у которых нет локального состояния (поэтому и в названии — без состояния). У вас нет доступа к состоянию и вы не можете обновить состояние, используя `this.state` или `this.setState()`, соответственно, поскольку нет объекта `this`. Кроме того, у функциональных компонентов нет методов жизненного цикла. Мы ещё не рассматривали методы жизненного цикла, но вы уже использовали два из них: `constructor()` и `render()`. Тогда как конструктор выполняется только один раз в течение жизни компонента, метод класса `render()` запускается один раз в начале и каждый раз при обновлениях компонента. Имейте в виду, что у функциональных компонентов без состояния нет методов жизненного цикла, о которых будет рассказано в последующих главах.

* **Классовые компоненты или классы компонентов (ES6 Class Components):** Вы уже использовали этот тип объявления в ваших компонентах. В определении класса они наследуются (расширяются, extend) от компонента React. Ключевое слово `extend` привязывает компоненту все методы жизненного цикла, доступные API компонента React. Именно поэтому вы можете использовать метод `render()`. Кроме того, вы можете хранить и управлять состоянием в классах компонентов через использование `this.state` и `this.setState()`.

* **React.createClass:** Подобное объявление компонента использовалось в старых версиях React и всё ещё используется в React-приложениях на JavaScript ES5. Но [Facebook объявил их устаревшими](https://reactjs.org/blog/2015/03/10/react-v0.13.html) в пользу JavaScript ES6. Они даже добавили [соответствующее предупреждение в версии 15.5](https://reactjs.org/blog/2017/04/07/react-v15.5.0.html). Вы не будете использовать их в этой книге.

Таким образом, в основном осталось только два объявления компонента. Но когда следует предпочесть функциональные компоненты без состояния классовым компонентам? Эмпирическое правило — использовать функциональные компоненты без состояния, если вам не нужны локальное состояние или методы жизненного цикла. Как правило, вы начинаете реализовать свои компоненты именно как функциональные компоненты без состояния. Как только вам нужен доступ к состоянию или методам жизненного цикла — преобразуйте компонент в классовый компонент. В нашем приложении мы поступили как раз наоборот, но всё ради изучения React.

Вернёмся к нашему приложению. Компонент App использует внутреннее состояние. Именно поэтому он должен остаться нетронутым, то есть классом компонента. Но три других классовых компонента не используют состояние, то есть им не нужен доступ к `this.state` или `this.setState()`. Более того, у них нет методов жизненного цикла. Поэтому давайте преобразуем компонент Search в функциональный. Преобразование компонентов Table и Button вы сделаете самостоятельно, в качестве упражнения.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
function Search(props) {
  const { value, onChange, children } = props;
  return (
    <form>
      {children} <input
        type="text"
        value={value}
        onChange={onChange}
      />
    </form>
  );
}
# leanpub-end-insert
~~~~~~~~

Вот в принципе и всё. `props` доступны в объявлении функции, функция возвращает JSX. Однако кое-что ещё можно улучшить. Так как вы уже знаете деструктуризацию, и поэтому довольно эффективным методом является её использование в объявлении функции.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
function Search({ value, onChange, children }) {
# leanpub-end-insert
  return (
    <form>
      {children} <input
        type="text"
        value={value}
        onChange={onChange}
      />
    </form>
  );
}
~~~~~~~~

Стало лучше, но нет предела совершенству. Вы уже знаете, что стрелочные функции из ES6 позволяют писать функции короче. Вы можете удалить тело блока функции. В стрелочных функциях подразумевается неявный возврат, поэтому вы можете удалить выражение `return`. Поскольку ваш функциональный компонент без состояния — это функция, вы можете сделать текущий код функции более кратким.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const Search = ({ value, onChange, children }) =>
  <form>
    {children} <input
      type="text"
      value={value}
      onChange={onChange}
    />
  </form>
# leanpub-end-insert
~~~~~~~~

Последний шаг был особенно полезен для обеспечения того, что только свойства передаются как входные данные, а JSX возвращается в качестве вывода. Между ними нет ничего. Тем не менее, вы можете *что-то сделать* между ними, используя тело блока в вашей стрелочной функции.

{title="Код",lang=javascript}
~~~~~~~~
const Search = ({ value, onChange, children }) => {

  // что-нибудь делать

  return (
    <form>
      {children} <input
        type="text"
        value={value}
        onChange={onChange}
      />
    </form>
  );
}
~~~~~~~~

Но пока нам это не нужно. Поэтому можно сохранить предыдущую версию без тела блока. При использовании блоков с телом кода, люди часто склонны делать много лишнего в функции. Удалив блок с кодом, вы можете сосредоточиться на входных данных и выводе вашей функции.

В этому времени у вас есть один лёгкий функциональный компонент без состояния. Как только вам понадобится внутреннее состояние или методы жизненного цикла, вы всегда можете обратиться к классовым компонентам. Кроме того, вы видели, как JavaScript ES6 может использоваться в компонентах React для того, чтобы сделать их краткими и элегантными.

### Упражнения:

* преобразуйте компоненты Table и Button в функциональные компоненты без состояния
* ознакомьтесь подробнее с [классовыми компонентами и функциональными компонентами без состояния](https://ru.react.js.org/docs/components-and-props.html)

## Стилизация компонентов

Давайте добавим базовые стили к вашему приложению и компонентам. Вы можете повторно использовать файлы *src/App.css* и *src/index.css*. Эти файлы уже должны быть в вашем проекте, потому что вы начали проект с помощью *create-react-app*. Они также должны быть импортированы в файлах *src/App.js* и *src/index.js*. Я подготовил CSS-код, который вы можете просто скопировать и вставить в эти файлы, но не стесняйтесь использовать собственные стили.

Для начала добавим стили для всего приложения.

{title="src/index.css",lang="css"}
~~~~~~~~
body {
  color: #222;
  background: #f4f4f4;
  font: 400 14px CoreSans, Arial,sans-serif;
}

a {
  color: #222;
}

a:hover {
  text-decoration: underline;
}

ul, li {
  list-style: none;
  padding: 0;
  margin: 0;
}

input {
  padding: 10px;
  border-radius: 5px;
  outline: none;
  margin-right: 10px;
  border: 1px solid #dddddd;
}

button {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #dddddd;
  background: transparent;
  color: #808080;
  cursor: pointer;
}

button:hover {
  color: #222;
}

*:focus {
  outline: none;
}
~~~~~~~~

Теперь перейдём к стилизации в файле App.

{title="src/App.css",lang="css"}
~~~~~~~~
.page {
  margin: 20px;
}

.interactions {
  text-align: center;
}

.table {
  margin: 20px 0;
}

.table-header {
  display: flex;
  line-height: 24px;
  font-size: 16px;
  padding: 0 10px;
  justify-content: space-between;
}

.table-empty {
  margin: 200px;
  text-align: center;
  font-size: 16px;
}

.table-row {
  display: flex;
  line-height: 24px;
  white-space: nowrap;
  margin: 10px 0;
  padding: 10px;
  background: #ffffff;
  border: 1px solid #e3e3e3;
}

.table-header > span {
  overflow: hidden;
  text-overflow: ellipsis;
  padding: 0 5px;
}

.table-row > span {
  overflow: hidden;
  text-overflow: ellipsis;
  padding: 0 5px;
}

.button-inline {
  border-width: 0;
  background: transparent;
  color: inherit;
  text-align: inherit;
  -webkit-font-smoothing: inherit;
  padding: 0;
  font-size: inherit;
  cursor: pointer;
}

.button-active {
  border-radius: 0;
  border-bottom: 1px solid #38BB6C;
}
~~~~~~~~

Теперь у нас всё готово для стилизации компонентов. Не забудьте использовать `className` вместо `class` в качестве HTML-атрибута.

Во-первых, добавим классы в классе-компоненте App.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, list } = this.state;
    return (
# leanpub-start-insert
      <div className="page">
        <div className="interactions">
# leanpub-end-insert
          <Search
            value={searchTerm}
            onChange={this.onSearchChange}
          >
            Поиск
          </Search>
# leanpub-start-insert
        </div>
# leanpub-end-insert
        <Table
          list={list}
          pattern={searchTerm}
          onDismiss={this.onDismiss}
        />
# leanpub-start-insert
      </div>
# leanpub-end-insert
    );
  }
}
~~~~~~~~

Во-вторых, сделаем аналогичное для функционального компонента без состояния Table.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Table = ({ list, pattern, onDismiss }) =>
# leanpub-start-insert
  <div className="table">
# leanpub-end-insert
    {list.filter(isSearched(pattern)).map(item =>
# leanpub-start-insert
      <div key={item.objectID} className="table-row">
# leanpub-end-insert
        <span>
          <a href={item.url}>{item.title}</a>
        </span>
        <span>{item.author}</span>
        <span>{item.num_comments}</span>
        <span>{item.points}</span>
        <span>
          <Button
            onClick={() => onDismiss(item.objectID)}
# leanpub-start-insert
            className="button-inline"
# leanpub-end-insert
          >
            Отбросить
          </Button>
        </span>
# leanpub-start-insert
      </div>
# leanpub-end-insert
    )}
# leanpub-start-insert
  </div>
# leanpub-end-insert
~~~~~~~~

На данный момент вы стилизовали приложение и компоненты с помощью простого CSS. Всё это вместе должно выглядеть довольно прилично. Как вы знаете, JSX смешивает HTML и JavaScript. Конечно, можно предположить добавление CSS к этому сочетанию. Это называется встроенными стилями. Вы можете определить JavaScript-объекты и передать их в атрибуте `style` элемента.

Давайте сделаем фиксированную ширину столбцов в компоненте Table, использовав встроенные стили.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Table = ({ list, pattern, onDismiss }) =>
  <div className="table">
    {list.filter(isSearched(pattern)).map(item =>
      <div key={item.objectID} className="table-row">
# leanpub-start-insert
        <span style={{ width: '40%' }}>
          <a href={item.url}>{item.title}</a>
        </span>
        <span style={{ width: '30%' }}>
          {item.author}
        </span>
        <span style={{ width: '10%' }}>
          {item.num_comments}
        </span>
        <span style={{ width: '10%' }}>
          {item.points}
        </span>
        <span style={{ width: '10%' }}>
          <Button
            onClick={() => onDismiss(item.objectID)}
            className="button-inline"
          >
            Отбросить
          </Button>
        </span>
# leanpub-end-insert
      </div>
    )}
  </div>
~~~~~~~~

Так, стиль добавлен. Вы можете определить объекты со стилем вне элементов для большей чистоты кода.

{title="Код",lang="javascript"}
~~~~~~~~
const largeColumn = {
  width: '40%',
};

const midColumn = {
  width: '30%',
};

const smallColumn = {
  width: '10%',
};
~~~~~~~~

Поступив подобным образом, вы можете использовать их в столбцах так: `<span style={smallColumn}>`.

В общем, вы увидите различные мнения и решения для стилизации в React. Сейчас вы использовали чистый CSS и встроенные стили. Это достаточно удобно для старта.

Я не хочу быть слишком самоуверенным, но хочу оставить вам ещё несколько альтернативных вариантов. Вы можете прочитать о них и применить самостоятельно. Но если вы новичок в React, я бы порекомендовал придерживаться чистого CSS и встроенных стилей.

* [styled-components](https://github.com/styled-components/styled-components)
* [CSS Modules](https://github.com/css-modules/css-modules)

{pagebreak}

Вы изучили основы написания собственного React-приложения! Давайте повторим последние темы:

* React
  * использование `this.state` и `setState()` для управления внутренним состоянием компонента
  * передача функций или методов класса элементу обработчика
  * использование форм и событий в React для добавления взаимодействий
  * однонаправленный поток данных — важная концепция в React
  * постигли контролируемые компоненты
  * составлять компоненты с дочерними компонентами и повторно используемыми компонентами
  * использование и реализация классовых компонентов и функциональных компонентов без состояния
  * подходы к стилизации компонентов
* ES6
  * функции, связанные с классом — это методы класса
  * деструктуризация объектов и массивов
  * параметры по умолчанию
* Общее
  * функции высшего порядка

Опять же имеет смысл сделать перерыв. Усвоить полученные значения и применить их самостоятельно на практике. Вы можете поэкспериментировать с исходным кодом, написанным в течение этой главы, его можно в [официальном репозитории](https://github.com/the-road-to-learn-react/hackernews-client/tree/5.2).

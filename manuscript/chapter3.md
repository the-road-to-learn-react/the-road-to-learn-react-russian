# Получение реальных данных с API

Теперь пришло время получать реальные данные через API, поскольку может быть скучно работать с демонстрационными данными.

Если вы не знакомы с API, я рекомендую вам прочитать [моё путешествие по знакомству с API](https://www.robinwieruch.de/what-is-an-api-javascript/).

Вы знаете платформу [Hacker News](https://news.ycombinator.com/)? Это отличный новостной агрегатор на технические темы. В этой книге мы будем использовать API Hacker News для получения популярных (трендовых) историй. Для того, чтобы это сделать есть [базовый](https://github.com/HackerNews/API) и [поисковый](https://hn.algolia.com/api) API для получения данных с этой платформы. Последний предназначен для нашего разрабатываемого приложения для поиска историй на Hacker News. Вы можете открыть спецификацию API для понимания структуры данных.

## Методы жизненного цикла

Вам нужно будет узнать о методах жизненного цикла React до того, как вы начнёте получать данные в ваших компонентах с использованием API. Эти методы — хуки в жизненном цикле React-компонента. Их можно использовать в ES6-классе компонента, но не в функциональных компонентах без состояния.

Помните, в предыдущей главе вы изучали классы в JavaScript ES6 и их использование в React? Помимо метода `render()` существует несколько методов, которые переопределяются в классовых компонентах React. Всё это методы жизненного цикла, давайте погрузимся в них.

Вам уже знакомы два метода жизненного цикла, которые можно использовать в классе компонента: `constructor()` и `render()`.

Конструктор вызывается только тогда, когда экземпляр компонента создан и добавлен в DOM, т.е. компонент проинициализирован. Этот процесс называется монтированием (установкой) компонента.

Метод `render()` также вызывается во время процесса монтирования, но ещё и при обновлении компонента. Каждый раз при изменении состояния или свойств компонента, вызывается метод `render()`.

Теперь вы узнали больше об этих двух методах жизненного цикла и о том, в каких случаях они вызываются. Вы их уже использовали, но есть и другие методы, которые мы сейчас рассмотрим.

У монтирования компонента есть два метода жизненного цикла: `getDerivedStateFromProps()` и `componentDidMount()`. Сначала вызывается конструктор, метод `getDerivedStateFromProps()` вызывается до метода `render()` и `componentDidMount()` вызывается после метода `render()`.

В целом процесс монтирования имеет 4 метода жизненного цикла. Они вызываются в следующем порядке:

* constructor()
* getDerivedStateFromProps()
* render()
* componentDidMount()

Но как насчёт обновления жизненного цикла компонента, которое происходит при изменении состояния или свойств?

* getDerivedStateFromProps()
* shouldComponentUpdate()
* componentWillUpdate()
* render()
* getSnapshotBeforeUpdate()
* componentDidUpdate()

И последнее, но не менее важное — жизненный цикл размонтирования. Для этого процесса доступен только один метод — `componentWillUnmount()`.

В конце концов, вам не нужно знать все эти методы жизненного цикла с самого начала. Это может быть пугающим, тем более не все вы будете использовать. Даже в крупном React-приложении вы будете использовать только некоторые из них, помимо методов `constructor()` и `render()`. Тем не менее, хорошо знать, какой метод жизненного цикла в каких случаях может использоваться:

* **constructor(props)** — вызывается, когда компонент инициализируется. Вы можете установить начальное состояние компонента и связать методы класса в этом методе жизненного цикла.

* **static getDerivedStateFromProps(props, state)** — вызывается перед методом жизненного цикла `render()` как при начальном монтировании, так и при последующих обновлениях. Этот метод должен возвратить объект для обновления состояния или `null`, если ничего не нужно обновлять. Он существует для **редких** случаев использования, когда состояние зависит от изменений в свойствах со временем. Важно знать, что это статический метод, и поэтому у него нет доступа к экземпляру компонента.

* **render()** — требуется обязательно и возвращает элементы в качестве вывода компонента. Метод должен быть чистым, т.е. не изменять состояние компонента. Он получает входные данные в виде свойств и состояния и возвращает элемент.

* **componentDidMount()** — вызывается только один раз при монтировании компонента. Это идеальный момент для выполнения асинхронных запросов на получение данных из API. Полученные данные будут сохранены во внутреннем состоянии компонента для его отображения в методе жизненного цикла `render()`.

* **shouldComponentUpdate(nextProps, nextState)** — всегда вызывается, когда компонент изменяется во время изменения состояния или свойств. Вы будете использовать его в зрелых React-приложениях для оптимизации производительности. В зависимости от возвращаемого из этого метода логического значения компонент и все его дочерние элементы либо будут перерисовываться, либо нет. Этот метод может предотвратить отрисовку компонента.

* **getSnapshotBeforeUpdate(prevProps, prevState)** — вызывается непосредственно перед тем, как последний отрисованный вывод будет зафиксирован в DOM. В редких вариантах использования компонент должен захватить информацию из DOM прежде чем он в принципе будет изменён. Этот метод жизненного цикла позволяет компоненту это сделать. Другой метод (`componentDidUpdate()`) получит любое значение, возвращаемое `getSnapshotBeforeUpdate()` в качестве параметра.

* **componentDidUpdate(prevProps, prevState)** — вызывается сразу после обновления, но не при первоначальной отрисовке метода `render()`. Вы можете использовать его как возможность выполнять операции с DOM или дальнейшие асинхронные запросы. Если ваш компонент реализует метод `getSnapshotBeforeUpdate()`, возвращаемое им значение будет приниматься в качестве параметра `snapshot`.

* **componentWillUnmount()** — вызывается до того, как компонент будет уничтожен (удалён из DOM). Вы можете использовать этот метод для выполнения любых задач очистки.

Вы уже использовали методы `constructor()` и `render()`. Это часто используемые методы жизненного цикла в классовых компонентах. На самом деле только метод `render()` является обязательным, в противном случае вы не возвратите экземпляр компонента.

Существует ещё один метод жизненного цикла — `componentDidCatch(error, info)`. Он представлен в [React 16](https://www.robinwieruch.de/what-is-new-in-react-16/) и используется для обработки ошибок в компонентах. К примеру, список хорошо отображается в приложении. Но что, если список в локальном состоянии случайно установлен в `null` (например, при получении данных из внешнего API запрос завершился неудачно, поэтому вы установили локальное состояние для списка в значение `null`). Впоследствии было невозможно фильтровать и отображать список, поскольку вместо элементов списка — значение `null`. Компонент будет сломан, и приложение полностью потерпит неудачу. Теперь, используя `componentDidCatch()`, вы можете перехватить ошибку, сохранить её в локальном состоянии и необязательно показать сообщение в приложении для уведомления пользователя об ошибке.

### Упражнения:

* узнайте подробнее [методы жизненного цикла в React](https://ru.react.js.org/docs/react-component.html)
* узнайте подробнее о [состоянии, связанном с методами жизненного цикла в React](https://ru.react.js.org/docs/state-and-lifecycle.html)
* узнайте подробнее про [обработку ошибок в компонентах](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

## Получение данных

Теперь вы готовы извлечь данные из API Hacker News. Для этого можно воспользоваться ранее упомянутым методом жизненного цикла — `componentDidMount()`. Для выполнения запроса мы будем использовать нативный в JavaScript fetch.

Прежде чем мы будем его использовать, давайте настроим константы для URL-адреса и параметры по умолчанию, чтобы разбить запрос к API на куски.

{title="src/App.js",lang=javascript}
~~~~~~~~
import React, { Component } from 'react';
import './App.css';

# leanpub-start-insert
const DEFAULT_QUERY = 'redux';

const PATH_BASE = 'https://hn.algolia.com/api/v1';
const PATH_SEARCH = '/search';
const PARAM_SEARCH = 'query=';
# leanpub-end-insert

// ...
~~~~~~~~

В JavaScript ES6 вы можете использовать [шаблонные строки](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Template_literals) для конкатенации строк. В вашем случае вы будете их использовать для конкатенации  URL-адреса к конечной точке (endpoint) API.

{title="Код",lang="javascript"}
~~~~~~~~
// ES6
const url = `${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${DEFAULT_QUERY}`;

// ES5
var url = PATH_BASE + PATH_SEARCH + '?' + PARAM_SEARCH + DEFAULT_QUERY;

console.log(url);
// вывод: https://hn.algolia.com/api/v1/search?query=redux
~~~~~~~~

Это позволит сохранить гибкость структуры URL-адреса в будущем.

Но давайте перейдём к API-запросу, в котором будет использоваться URL-адрес. Весь процесс получения данных приводится сразу, но каждый шаг будет объяснён позже.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
# leanpub-start-insert
      result: null,
      searchTerm: DEFAULT_QUERY,
# leanpub-end-insert
    };

# leanpub-start-insert
    this.setSearchTopStories = this.setSearchTopStories.bind(this);
# leanpub-end-insert
    this.onSearchChange = this.onSearchChange.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
  }

# leanpub-start-insert
  setSearchTopStories(result) {
    this.setState({ result });
  }

  componentDidMount() {
    const { searchTerm } = this.state;

    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}`)
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
      .catch(error => error);
  }
# leanpub-end-insert

  ...
}
~~~~~~~~

В коде много чего происходит. Я думал о том, чтобы разбить его на небольшие части. Тогда снова было бы трудно понять связь каждой части друг с другом. Позвольте мне подробно объяснить каждый шаг.

Во-первых, вы можете удалить демонстрационный список элементов, потому что вы возвращаете реальный список из API Hacker News. Демонстрационные данные больше не используются. Исходное состояние вашего компонента теперь имеет пустой результат и поисковый запрос по умолчанию. Тот же самый поисковый запрос по умолчанию используется в поле ввода компонента Search и в вашем первом запросе.

Во-вторых, вы используете метод жизненного цикла `componentDidMount()` для получения данных после монтирования компонента. При самом первом получении используется поисковый запрос по умолчанию из локального состояния. Он будет получать истории, связанные с "redux", поскольку это параметр по умолчанию.

В-третьих, используется нативный API. Шаблонные строки в JavaScript ES6 позволяют ему составлять URL-адрес с `searchTerm`. URL-адрес — аргумент для нативной функции API fetch. Ответ требуется преобразовать в структуру данных JSON, что является обязательным шагом в нативной функции fetch при работе с JSON-данными, и, наконец, они могут быть сохранены в качестве результата во внутреннем состоянии компонента. Кроме того, блок `catch` используете в случае ошибки. Если во время выполнения запроса произошла ошибка, выполнение функции перейдёт из блока `then` в блок `catch`. В следующей главе книги будет включена обработка ошибок.

И последнее, но не менее важно: не забудьте связать новый метод компонента в конструкторе.

Теперь вы можете использовать полученные данные вместо демонстрационного списка элементов. Однако нужно быть осторожным. Результат содержит не только список данных. [Это сложный объект с метаинформацией и свойством hits, который содержит нужные нам истории](https://hn.algolia.com/api). Вы можете вывести внутреннее состояние через `console.log(this.state);` в методе для `render()` для наглядного отображения.

Следующим шагом будет использование результата для его отрисовки. Мы предотвратим отрисовку, вернув значение `null`, если данных с API нет в первый раз. После того, как запрос к API выполнен успешно, результат будет сохранён в состоянии и компонент App повторно отрисуется с обновлённым состоянием.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
# leanpub-start-insert
    const { searchTerm, result } = this.state;

    if (!result) { return null; }

# leanpub-end-insert
    return (
      <div className="page">
        ...
        <Table
# leanpub-start-insert
          list={result.hits}
# leanpub-end-insert
          pattern={searchTerm}
          onDismiss={this.onDismiss}
        />
      </div>
    );
  }
}
~~~~~~~~

Давайте вспомним, что происходит во время жизненного цикла компонента. Компонент инициализируется через конструктор. После этого он отрисовывается в первый раз. Но вы препятствуете отрисовке чего-либо, поскольку результат в локальном состоянии равен `null`. Разрешено возвращать `null` из компонента, если нет данных для отображения. Затем выполняется метод жизненного цикла `componentDidMount()`. В этом методе вы асинхронно получаете данные из API Hacker News. Как только данные придут, изменится внутреннее состояние компонента в `setSearchTopStories()`. Поскольку локальное состояние обновлено, вступает в действие обновление жизненного цикла. Компонент снова выполняет метод `render()`, но на этот раз с заполненными данными результата во внутреннем состоянии компонента. Компонент, и таким образом, компонент Table с его содержимым будут отрисовываться.

Вы использовали нативный API fetch, поддерживаемый большинством браузеров для выполнения асинхронных запросов. Конфигурация *create-react-app*  гарантирует его поддержку в каждом браузере. Существуют также сторонние пакеты, которыми можно заменить нативный API fetch: [superagent](https://github.com/visionmedia/superagent) и [axios](https://github.com/mzabriskie/axios).

Имейте в виду, что в книге используется сокращённая нотация JavaScript для проверки истинности. В предыдущем примере `if (!result)` вместо `if (result === null)`. То же самое относится и к другим частям примеров на протяжении всей книги. Например, используете `if (!list.length)` вместо `if (list.length === 0)` или `if (someString)` вместо `if (someString !== '')`. Почитайте на эту тему, если вы не слишком знакомы с этим.

Вернёмся к нашему приложению: показывается список историй. Однако сейчас в приложении есть два регресионных бага. Во-первых, кнопка "Отбросить" не работает. Она не знает о сложном объекте результата и по-прежнему работает на обычном списке из локального состояния при отклонении элемента. Во-вторых, при попытке поиска в отображённом списке, он фильтруется на стороне клиента, даже если изначальное получение данных было выполнено посредством поиска историй на стороне сервера. Идеально было бы при использовании компонента Search получать другой объект результата из API. Обе ошибки регрессии будут исправлены в следующих главах.

### Упражнения:

* узнайте подробнее [строковые шаблоны ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Template_literals)
* узнайте подробнее [нативный API fetch](https://developer.mozilla.org/ru/docs/Web/API/Fetch_API)
* узнайте подробнее [получение данных в React](https://www.robinwieruch.de/react-fetching-data/)

## Оператор расширения ES6

Кнопка "Отбросить" не работает, потому что метод `onDismiss()` не знает о сложном результирующем объекте. Она знает только про простой список в локальном состоянии. Но это уже не простой список. Давайте изменим его для работы с результирующим объектом вместо самого списка.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
  const isNotId = item => item.objectID !== id;
# leanpub-start-insert
  const updatedHits = this.state.result.hits.filter(isNotId);
  this.setState({
    ...
  });
# leanpub-end-insert
}
~~~~~~~~

Но что теперь происходит в `setState()`? К сожалению, результат — сложный объект. Список хитов — это только одно из нескольких свойств объекта. Тем не менее, только список обновляется, когда элемент удаляется в результирующем объекте, тогда как остальные свойства остаются такими же.

Один из подходов может заключаться в том, чтобы изменять свойство hits в результирующем объекте. Я продемонстрирую это, но мы не будем так делать.

{title="Код",lang="javascript"}
~~~~~~~~
// не делайте этого
this.state.result.hits = updatedHits;
~~~~~~~~

React охватывает неизменяемые структуры данных. Таким образом, вы не должны изменять объект (или напрямую изменить состояние). Лучший подход — создать новый объект на основе имеющейся информации. Таким образом, ни один из объектов не будет изменён. Вы сохраните неизменяемые структуры данных. Вы всегда будете возвращать новый объект, а не изменять объект.

Поэтому вы можете использовать `Object.assign()` в JavaScript ES6. Он принимает в качестве первого аргумента целевой объект. Все следующие аргументы — исходные объекты. Эти объекты объединяются в целевой объект. Целевой объект может быть пустым. Метод `Object.assign()` удовлетворяет принципу неизменяемости, так что ни один из исходных объектов не изменяется. В итоге код будет выглядеть следующим образом:

{title="Код",lang="javascript"}
~~~~~~~~
const updatedHits = { hits: updatedHits };
const updatedResult = Object.assign({}, this.state.result, updatedHits);
~~~~~~~~

Последующие объекты будут переопределять прежние объединённые объекты, если у них одинаковые имена свойств. Теперь давайте применим его в методе `onDismiss()`:

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
  const isNotId = item => item.objectID !== id;
  const updatedHits = this.state.result.hits.filter(isNotId);
  this.setState({
# leanpub-start-insert
    result: Object.assign({}, this.state.result, { hits: updatedHits })
# leanpub-end-insert
  });
}
~~~~~~~~

Мы эту функцию уже использовали. Но в JavaScript ES6 и будущих релизах JavaScript есть более простой способ. Могу ли я представить вам оператор расширения? Он состоит из трёх точек: `...`. При его использовании каждое значение из массива или объекта копируется в другой массив или объект.

Давайте изучим оператор расширения **массива** ES6, даже если он вам ещё не нужен.

{title="Код",lang="javascript"}
~~~~~~~~
const userList = ['Robin', 'Andrew', 'Dan'];
const additionalUser = 'Jordan';
const allUsers = [ ...userList, additionalUser ];

console.log(allUsers);
// выведет ['Robin', 'Andrew', 'Dan', 'Jordan']
~~~~~~~~

Переменная `allUsers` — это полностью новый массив. Остальные переменные `userList` и `additionalUser` остаются такими же. Вы можете объединить два массива таким образом в новый массив.

{title="Код",lang="javascript"}
~~~~~~~~
const oldUsers = ['Robin', 'Andrew'];
const newUsers = ['Dan', 'Jordan'];
const allUsers = [ ...oldUsers, ...newUsers ];

console.log(allUsers);
// выведет: ['Robin', 'Andrew', 'Dan', 'Jordan']
~~~~~~~~

Теперь давайте посмотрим на оператор расширения объекта. Это не JavaScript ES6. Это [предложение для следующей версии JavaScript](https://github.com/sebmarkbage/ecmascript-rest-spread), которое уже используется сообществом React. Именно поэтому *create-react-app* включило эту возможность в свою конфигурацию.

В основном это то же самое, что и оператор расширения массива в JavaScript ES6, но с объектами. Он копирует каждую пару ключ-значение в новый объект.

{title="Код",lang="javascript"}
~~~~~~~~
const userNames = { firstname: 'Robin', lastname: 'Wieruch' };
const age = 28;
const user = { ...userNames, age };

console.log(user);
// выведет: { firstname: 'Robin', lastname: 'Wieruch', age: 28 }
~~~~~~~~

Может использоваться несколько объектов, как в аналогичном примере с массивом.

{title="Код",lang="javascript"}
~~~~~~~~
const userNames = { firstname: 'Robin', lastname: 'Wieruch' };
const userAge = { age: 28 };
const user = { ...userNames, ...userAge };

console.log(user);
// выведет: { firstname: 'Robin', lastname: 'Wieruch', age: 28 }
~~~~~~~~

В конце концов его можно использовать для замены `Object.assign()`.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
  const isNotId = item => item.objectID !== id;
  const updatedHits = this.state.result.hits.filter(isNotId);
  this.setState({
# leanpub-start-insert
    result: { ...this.state.result, hits: updatedHits }
# leanpub-end-insert
  });
}
~~~~~~~~

Теперь кнопка "Отбросить" должна снова работать, потому что метод `onDismiss()` знает о сложном результирующем объекте и о том, как его обновлять после отклонения элемента из списка.

### Упражнения:

* узнайте получше [Object.assign() из ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
* узнайте подробнее [оператор расширения массива ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Spread_operator)
  * оператор расширения объектов кратко упоминался уже

## Отрисовка по условию

Условная отрисовка применяется довольно рано в React-приложении. Но не в этой книге, потому что не было такого случая использования. Условная отрисовка происходит, когда вы хотите решить — отрисовать тот или иной элемент, или нет. Иногда это означает отрисовать либо элемент, либо вообще ничего. В конце концов, упрощённое использование условной отрисовки может быть сформулировано выражением if-else в JSX.

Объект `result` во внутреннем состоянии объекта равен значению `null` в самом начале. До сих пор компонент App не возвращал элементов, когда `result` не был заполнен с API. Это уже условная отрисовка, потому что из метода жизненного цикла `render()` заранее возвращается что-то в зависимости от определённого условия. Компонент App либо отрисовывает свои элементы, либо ничего не отрисовывает.

Но давайте пойдём дальше. Имеет смысл обернуть компонент Table, единственный компонент, который зависит от `result`, в независимую условную отрисовку. Всё остальное должно отображаться, даже если пока `result` ещё нет. Вы можете просто использовать тернарный оператор в JSX.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
# leanpub-start-insert
    const { searchTerm, result } = this.state;
# leanpub-end-insert
    return (
      <div className="page">
        <div className="interactions">
          <Search
            value={searchTerm}
            onChange={this.onSearchChange}
          >
            Поиск
          </Search>
        </div>
# leanpub-start-insert
        { result
          ? <Table
            list={result.hits}
            pattern={searchTerm}
            onDismiss={this.onDismiss}
          />
          : null
        }
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Это второй вариант выражения условной отрисовки. Третий вариант заключается в логическом операторе `&&`. В JavaScript выражение `true && 'Привет, мир'` всегда выполняется как 'Привет, мир'. А выражение `false && 'Привет, мир'` всегда будет `false`.

{title="Код",lang="javascript"}
~~~~~~~~
const result = true && 'Привет, мир';
console.log(result);
// выведет: Привет, мир

const result = false && 'Привет, мир';
console.log(result);
// выведет: false
~~~~~~~~

В React вы можете использовать подобное поведение. Если условие равняется true, выражение после оператора `&&` будет выведено. Если условие равняется `false`, React проигнорирует и пропустит выражение. Это подходит в случае условной отрисовки компонента Table, поскольку он должен возвращать Table, а может его не вернуть.

{title="src/App.js",lang=javascript}
~~~~~~~~
{ result &&
  <Table
    list={result.hits}
    pattern={searchTerm}
    onDismiss={this.onDismiss}
  />
}
~~~~~~~~

Сейчас были показаны несколько подходов использования условной отрисовки в React. Вы можете узнать [больше о других альтернативах в исчерпывающем списке примеров условной отрисовки](https://www.robinwieruch.de/conditional-rendering-react/). Кроме того, вы узнаете об их различных вариантах использования и применения их.

В конце концов, вы должны иметь возможность видеть полученные данные в приложении. Когда данные находятся только в процессе получения, отображается всё, кроме таблицы. После того как результат будет получен из запроса и сохранён в локальном состоянии, будет отображена таблица, потому что метод `render()` запускается снова, и условие разрешается в пользу отображения компонента таблицы.

### Упражнения:

* ознакомьтесь подробнее с [различными способами условной отрисовки](https://www.robinwieruch.de/conditional-rendering-react/)
* узнайте больше про [отрисовку по условию в React](https://ru.react.js.org/docs/conditional-rendering.html)

## Поиск на стороне клиента и на стороне сервера

Теперь, когда вы используете компонент поиска со своим полем ввода, пришло время фильтровать список. Сейчас это происходит на стороне клиента, а теперь вы будете использовать фильтрацию на стороне сервера. В противном случае вы будете иметь дело только с первым запросом, который вы сделали в методе `componentDidMount()` с параметром поиска по умолчанию.

Вы можете определить метод `onSearchSubmit()` в компонента App, который получает результаты из Hacker News API при выполнении поиска в компоненте Search.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      result: null,
      searchTerm: DEFAULT_QUERY,
    };

    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.onSearchChange = this.onSearchChange.bind(this);
# leanpub-start-insert
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
# leanpub-end-insert
    this.onDismiss = this.onDismiss.bind(this);
  }

  ...

# leanpub-start-insert
  onSearchSubmit() {
    const { searchTerm } = this.state;
  }
# leanpub-end-insert

  ...
}
~~~~~~~~

Метод `onSearchSubmit()` должен использовать ту же функциональность, что и метод жизненного цикла `componentDidMount()`, но на этот раз с изменённым поисковым запросом из локального состояния, а не с начальным запросом поиска по умолчанию. Таким образом, вы можете извлечь функциональность как метод класса для повторного использования.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      result: null,
      searchTerm: DEFAULT_QUERY,
    };

    this.setSearchTopStories = this.setSearchTopStories.bind(this);
# leanpub-start-insert
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
# leanpub-end-insert
    this.onSearchChange = this.onSearchChange.bind(this);
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
  }

  ...

# leanpub-start-insert
  fetchSearchTopStories(searchTerm) {
    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}`)
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
      .catch(error => error);
  }
# leanpub-end-insert

  componentDidMount() {
    const { searchTerm } = this.state;
# leanpub-start-insert
    this.fetchSearchTopStories(searchTerm);
# leanpub-end-insert
  }

  ...

  onSearchSubmit() {
    const { searchTerm } = this.state;
# leanpub-start-insert
    this.fetchSearchTopStories(searchTerm);
# leanpub-end-insert
  }

  ...
}
~~~~~~~~

Теперь в компоненте Search нужно добавить дополнительную кнопку. Кнопка должна явно вызывать запрос на поиск. В противном случае вы будете получать данные с API Hacker News каждый раз, когда изменяется поле ввода. Скорее всего вы захотите это сделать в обработчике `onClick()`.

В качестве альтернативы вы можете отложить (задержать выполнение) функцию `onChange()` и обойтись без этой кнопки. Но это добавит больше сложности и, возможно, не будет иметь желаемого эффекта. Давайте оставим простой вариант без задержки.

Сначала передадим метод `onSearchSubmit()` в компонент Search.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, result } = this.state;
    return (
      <div className="page">
        <div className="interactions">
          <Search
            value={searchTerm}
            onChange={this.onSearchChange}
# leanpub-start-insert
            onSubmit={this.onSearchSubmit}
# leanpub-end-insert
          >
            Поиск
          </Search>
        </div>
        { result &&
          <Table
            list={result.hits}
            pattern={searchTerm}
            onDismiss={this.onDismiss}
          />
        }
      </div>
    );
  }
}
~~~~~~~~

Во-вторых, создадим кнопку в компоненте Search. У кнопки будет атрибут `type="submit"`, а у формы будет свой атрибут `onSubmit` для передачи метода `onSubmit()`. Вы можете повторно использовать свойство `children`, но на этот раз в нём будет содержаться текст кнопки.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const Search = ({
  value,
  onChange,
  onSubmit,
  children
}) =>
  <form onSubmit={onSubmit}>
    <input
      type="text"
      value={value}
      onChange={onChange}
    />
    <button type="submit">
      {children}
    </button>
  </form>
# leanpub-end-insert
~~~~~~~~

В компоненте Table можно удалить функциональность фильтра, потому что больше не будет фильтра (поиска) на стороне клиента. Не забудьте также удалить функцию `isSearched()`: она больше не будет использоваться. Результат приходит непосредственно из API Hacker News после того, как была нажата кнопка "Поиск".

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, result } = this.state;
    return (
      <div className="page">
        ...
        { result &&
          <Table
# leanpub-start-insert
            list={result.hits}
            onDismiss={this.onDismiss}
# leanpub-end-insert
          />
        }
      </div>
    );
  }
}

...

# leanpub-start-insert
const Table = ({ list, onDismiss }) =>
# leanpub-end-insert
  <div className="table">
# leanpub-start-insert
    {list.map(item =>
# leanpub-end-insert
      ...
    )}
  </div>
~~~~~~~~

Если вы попытаетесь выполнить данный код сейчас, вы заметите, что браузер перезагружается. Это стандартное поведение браузера для колбэка при отправки формы HTML. В React вы часто столкнётесь с методом события `preventDefault()` для предотвращения нативного поведения браузера.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
onSearchSubmit(event) {
# leanpub-end-insert
  const { searchTerm } = this.state;
  this.fetchSearchTopStories(searchTerm);
# leanpub-start-insert
  event.preventDefault();
# leanpub-end-insert
}
~~~~~~~~

Теперь вы сможете искать различные истории по Hacker News. Прекрасно, сейчас вы взаимодействуете с реальным API. Больше не должно быть поиска на стороне клиента.

### Упражнения:

* узнайте больше [синтетические события в React](https://ru.react.js.org/docs/events.html)
* поэкспериментируйте с [API Hacker News](https://hn.algolia.com/api)

## Получение данных с разбивкой на страницы

Вы внимательно изучили структуру возвращаемых данных до сих пор? [API Hacker News](https://hn.algolia.com/api) возвращает больше данных, чем просто список историй (hits). Как раз этот API возвращает список историй с разбивкой на страницы. Свойство `page`, равное `0` в первом ответе, может быть использовано для получения большего количества с делением на страницы в качестве результата. Вам нужно передать следующую страницу с тем же самым поисковым запросом к API.

Давайте расширим вспомогательные константы для API, чтобы мы могли работать с данными, разбитыми на страницы.

{title="src/App.js",lang=javascript}
~~~~~~~~
const DEFAULT_QUERY = 'redux';

const PATH_BASE = 'https://hn.algolia.com/api/v1';
const PATH_SEARCH = '/search';
const PARAM_SEARCH = 'query=';
# leanpub-start-insert
const PARAM_PAGE = 'page=';
# leanpub-end-insert
~~~~~~~~

Теперь можно использовать новую константу для добавления параметра страницы к запросу API.

{title="Код",lang="javascript"}
~~~~~~~~
const url = `${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${DEFAULT_QUERY}&${PARAM_PAGE}`;

console.log(url);
// выведет: https://hn.algolia.com/api/v1/search?query=redux&page=
~~~~~~~~

Метод `fetchSearchTopStories()` принимает страницу в качестве второго аргумента. Если второй аргумент не предоставлен, будет использоваться страница `0` для первоначального запроса. Таким образом, методы `componentDidMount()` и `onSearchSubmit()` извлекают первую страницу по первому запросу. Каждая дополнительная выборка будет отображать следующую страницу, передавая её вторым аргументом.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

# leanpub-start-insert
  fetchSearchTopStories(searchTerm, page = 0) {
    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}`)
# leanpub-end-insert
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
      .catch(error => error);
  }

  ...

}
~~~~~~~~

Аргумент страницы использует значения параметров по умолчанию из JavaScript ES6, чтобы вернуться к странице `0` в случае, если функции не предоставлен аргумент страницы.

Теперь мы знаем, как получить текущую страницу из ответа API в `fetchSearchTopStories()`. Вы можете использовать этот метод при нажатии на кнопку, чтобы получить ещё больше историй в обработчике кнопки `onClick`. Давайте воспользуемся компонентом Button для выборки большего количества данных с разбивкой на страницы из API Hacker News. Вам нужно только определить обработчик `onClick()`, который принимает текущий поисковый запрос пользователя и следующую страницу (текущая страница + 1).

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, result } = this.state;
# leanpub-start-insert
    const page = (result && result.page) || 0;
# leanpub-end-insert
    return (
      <div className="page">
        <div className="interactions">
        ...
        { result &&
          <Table
            list={result.hits}
            onDismiss={this.onDismiss}
          />
        }
# leanpub-start-insert
        <div className="interactions">
          <Button onClick={() => this.fetchSearchTopStories(searchTerm, page + 1)}>
            Больше историй
          </Button>
        </div>
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Кроме того, в методе `render()` вам надо убедиться, что по умолчанию страница 0, когда результата пока нет. Помните, что метод `render()` вызывается до того, как данные будут извлечены асинхронно в методе жизненного цикла `componentDidMount()`.

Но мы пропустили ещё один шаг. Вы получаете следующую страницу данных, но она не переопределяет предыдущую страницу данных. Было бы идеально объединять старый и новый список историй из локального состояния и нового объекта результата. Давайте скорректируем функциональность добавления новых данных, вместо того чтобы переопределять их.

{title="src/App.js",lang=javascript}
~~~~~~~~
setSearchTopStories(result) {
# leanpub-start-insert
  const { hits, page } = result;

  const oldHits = page !== 0
    ? this.state.result.hits
    : [];

  const updatedHits = [
    ...oldHits,
    ...hits
  ];

  this.setState({
    result: { hits: updatedHits, page }
  });
# leanpub-end-insert
}
~~~~~~~~

Несколько моментов в методе `setSearchTopStories()` нужно отметить. Во-первых, вы получаете истории и текущую страницу из результата.

Во-вторых, вам нужно проверить, есть ли уже старые истории. Если страница равна 0, значит это новый поисковый запрос от `componentDidMount()` или `onSearchSubmit()`. Истории пустые. Но когда вы нажимаете на кнопку «Больше историй», чтобы получить больше данных с разбивкой по страницам, текущая страница больше не будет равняться 0. Это следующая страница. Старые истории уже хранятся в состоянии компонента и поэтому могут быть использованы.

В-третьих, вы не хотите переопределять старые истории. Вы можете объединить старые и новые истории из последнего запроса API. Объединение обоих списков может быть выполнено с помощью оператора расширения массива из JavaScript ES6.

В-четвёртых, вы сохранили объединённые истории и страницу в состоянии локального компонента.

Мы можем сделать последнюю правку. Когда вы пытаетесь нажать на кнопку «Больше историй», будут получены только несколько элементов списка. URL-адрес API может быть расширен для получения большего количества элементов списка с каждым запросом. Опять же, вы можете добавить дополнительные пути для запроса в виде констант.

{title="src/App.js",lang=javascript}
~~~~~~~~
const DEFAULT_QUERY = 'redux';
# leanpub-start-insert
const DEFAULT_HPP = '100';
# leanpub-end-insert

const PATH_BASE = 'https://hn.algolia.com/api/v1';
const PATH_SEARCH = '/search';
const PARAM_SEARCH = 'query=';
const PARAM_PAGE = 'page=';
# leanpub-start-insert
const PARAM_HPP = 'hitsPerPage=';
# leanpub-end-insert
~~~~~~~~

Теперь вы можете использовать константы для расширения URL-адреса к API-запросу.

{title="src/App.js",lang=javascript}
~~~~~~~~
fetchSearchTopStories(searchTerm, page = 0) {
# leanpub-start-insert
  fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
# leanpub-end-insert
    .then(response => response.json())
    .then(result => this.setSearchTopStories(result))
    .catch(error => error);
}
~~~~~~~~

После этого запрос к API Hacker News получает больше элементов списка за один запрос, чем раньше. Как вы можете видеть, мощный API, такой как API Hacker News, даёт вам множество способов экспериментировать с данными из реального мира. Вы должны это использовать, чтобы приложить усилия, когда узнаете что-нибудь новое, более захватывающее. Вот [как я узнал о расширении возможностей, предоставляемых API](https://www.robinwieruch.de/what-is-an-api-javascript/) при изучении нового языка программирования или библиотеки.

### Упражнения:

* узнайте больше [параметры по умолчанию из ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions/Default_parameters)
* поэкспериментируйте с [параметрами API Hacker News](https://hn.algolia.com/api)

## Кеш клиента

При каждой отправке формы поиска отправляется запрос к API Hacker News. Вы можете поискать «redux», а затем «react», и в конце концов снова «redux». Таким образом, получаются 3 запроса. Но вы дважды искали «redux», и оба раза для получения данных потребовалось полное асинхронное получение данных в обе стороны. В кеше на стороне клиента вы сохраните каждый результат. Когда выполняется запрос к API, проверяется, есть ли уже результат. Если он есть, используется кеш. В противном случае для получения данных выполняется запрос к API.

Для обеспечения существования клиентского кеша для каждого результата, вам нужно сохранить несколько результатов (`results`), а не один результат (`result`) во внутреннем состоянии компонента. Объектом результатов будет объект с поисковой строкой в качестве ключа и результатом запроса в качестве значения. Каждый результат, полученный от API, будет сохранён с соответствующей поисковой строкой в качестве ключа.

На данный момент ваш результат в локальном состоянии выглядит примерно так:

{title="Код",lang="javascript"}
~~~~~~~~
result: {
  hits: [ ... ],
  page: 2,
}
~~~~~~~~

Представьте, что вы сделали два запроса API. Один для поисковой строки «redux» и ещё один для «react». Объект результатов будет выглядеть следующим образом:

{title="Код",lang="javascript"}
~~~~~~~~
results: {
  redux: {
    hits: [ ... ],
    page: 2,
  },
  react: {
    hits: [ ... ],
    page: 1,
  },
  ...
}
~~~~~~~~

Давайте реализуем кеширование на стороне клиента с помощью `setState()`. Во-первых, переименуйте объект `result` в `results` в начальном состоянии компонента. Во-вторых, определите временный `searchKey`, который используется для хранения каждого результата (`result`).

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
# leanpub-start-insert
      results: null,
      searchKey: '',
# leanpub-end-insert
      searchTerm: DEFAULT_QUERY,
    };

    ...

  }

  ...

}
~~~~~~~~

Ключ `searchKey` должен быть установлен перед каждым запросом. Он отражает `searchTerm`. Вы могли бы задаться вопросом: почему мы не используем `searchTerm` в первую очередь? Это важно понять, прежде чем продолжать реализацию. `searchTerm` — это неустойчивая (временная) переменная, поскольку она изменяется каждый раз при вводе в поле поиска. Однако в конце вам понадобится постоянная переменная. Она определяет недавно отправленный поисковый запрос к API и может использоваться для получения правильного результата из объекта с результатами. Это указатель на ваш текущий результат в кеше и, следовательно, её можно использовать для отображения текущего результата в методе `render()`.

{title="src/App.js",lang=javascript}
~~~~~~~~
componentDidMount() {
  const { searchTerm } = this.state;
# leanpub-start-insert
  this.setState({ searchKey: searchTerm });
# leanpub-end-insert
  this.fetchSearchTopStories(searchTerm);
}

onSearchSubmit(event) {
  const { searchTerm } = this.state;
# leanpub-start-insert
  this.setState({ searchKey: searchTerm });
# leanpub-end-insert
  this.fetchSearchTopStories(searchTerm);
  event.preventDefault();
}
~~~~~~~~

Теперь вам нужно подправить код для сохранения результата во внутреннее состояние компонента. Он должен хранить каждый результат с использованием ключа `searchKey`.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  setSearchTopStories(result) {
    const { hits, page } = result;
# leanpub-start-insert
    const { searchKey, results } = this.state;

    const oldHits = results && results[searchKey]
      ? results[searchKey].hits
      : [];
# leanpub-end-insert

    const updatedHits = [
      ...oldHits,
      ...hits
    ];

    this.setState({
# leanpub-start-insert
      results: {
        ...results,
        [searchKey]: { hits: updatedHits, page }
      }
# leanpub-end-insert
    });
  }

  ...

}
~~~~~~~~

`searchKey` будет использоваться в качестве ключа для сохранения обновлённых историй и страницы в объекте `results`.

Во-первых, вы должны получить `searchKey` из состояния компонента. Помните, что `searchKey` устанавливается в `componentDidMount()` и `onSearchSubmit()`.

Во-вторых, старые истории должны быть объединены с новыми историями, как и раньше. Но на этот раз старые истории получаются из объекта `results` с помощью `searchKey` в качестве ключа.

В-третьих, новый результат может быть сохранён в объекте `results` состояния. Рассмотрим объект `results` в `setState()`.

{title="src/App.js",lang=javascript}
~~~~~~~~
results: {
  ...results,
  [searchKey]: { hits: updatedHits, page }
}
~~~~~~~~

Нижняя часть гарантирует сохранение обновлённого результата с ключом `searchKey` в объекте результатов. Значение — это объект со свойствами hits и page. `searchKey` — это поисковая строка. Вы уже изучили синтаксис `[searchKey]: ...`. Это вычисляемое имя свойства в ES6. Оно помогает динамически распределять значения в объекте.

Верхняя часть должна добавить все остальные результаты по `searchKey` в состоянии с помощью оператора расширения объектов. В противном случае вы потеряете все результаты, которые вы сохранили ранее.

Теперь вы сохраняете все результаты по поисковой строке. Это первый шаг для реализации кеша. На следующем шаге вы можете получить результат в зависимости от не временного значения в `searchKey` в объекте результатов. Вот почему вам сначала нужно было ввести `searchKey` как постоянную переменную. В противном случае процесс поиска будет нарушен, если вы будете использовать временный `searchTerm` для получения текущего результата, потому что это значение может измениться, если вы будете использовать компонент Search.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
# leanpub-start-insert
    const {
      searchTerm,
      results,
      searchKey
    } = this.state;

    const page = (
      results &&
      results[searchKey] &&
      results[searchKey].page
    ) || 0;

    const list = (
      results &&
      results[searchKey] &&
      results[searchKey].hits
    ) || [];

# leanpub-end-insert
    return (
      <div className="page">
        <div className="interactions">
          ...
        </div>
# leanpub-start-insert
        <Table
          list={list}
          onDismiss={this.onDismiss}
        />
# leanpub-end-insert
        <div className="interactions">
# leanpub-start-insert
          <Button onClick={() => this.fetchSearchTopStories(searchKey, page + 1)}>
# leanpub-end-insert
            Больше историй
          </Button>
        </div>
      </div>
    );
  }
}
~~~~~~~~

Поскольку по умолчанию у нас пустой список, потому что нет результата по `searchKey`, мы можем сейчас сэкономить условную отрисовку для компонента Table. Кроме того, вам нужно будет передать `searchKey`, а не` searchTerm` в кнопку «Больше историй». В противном случае постраничное получение данных зависит от значения `searchTerm`, которое является изменчивым. Кроме того, не забудьте сохранить свойство `searchTerm` для поля ввода в компоненте Search.

Функциональность поиска должна снова заработать. Она сохраняет все результаты от API Hacker News.

Кроме того, метод `onDismiss()` должен быть улучшен. Он по-прежнему работает с объектом `result`. Теперь он должен иметь дело с несколькими результатами (`results`).

{title="src/App.js",lang=javascript}
~~~~~~~~
  onDismiss(id) {
# leanpub-start-insert
    const { searchKey, results } = this.state;
    const { hits, page } = results[searchKey];
# leanpub-end-insert

    const isNotId = item => item.objectID !== id;
# leanpub-start-insert
    const updatedHits = hits.filter(isNotId);

    this.setState({
      results: {
        ...results,
        [searchKey]: { hits: updatedHits, page }
      }
    });
# leanpub-end-insert
  }
~~~~~~~~

Кнопка «Отбросить» должна снова работать.

Тем не менее, ничто не мешает приложению отсылать запрос к API при каждом запросе на поиск. Несмотря на то, что результат уже может быть, нет проверки, которая предотвращает запрос. Таким образом, функциональность кеша ещё не завершена. Она кеширует результаты, но не использует их. Последним шагом было бы предотвратить запрос к API, когда результат имеется в кеше.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {

    ...

# leanpub-start-insert
    this.needsToSearchTopStories = this.needsToSearchTopStories.bind(this);
# leanpub-end-insert
    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
    this.onSearchChange = this.onSearchChange.bind(this);
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
  }

# leanpub-start-insert
  needsToSearchTopStories(searchTerm) {
    return !this.state.results[searchTerm];
  }
# leanpub-end-insert

  ...

  onSearchSubmit(event) {
    const { searchTerm } = this.state;
    this.setState({ searchKey: searchTerm });
# leanpub-start-insert

    if (this.needsToSearchTopStories(searchTerm)) {
      this.fetchSearchTopStories(searchTerm);
    }

# leanpub-end-insert
    event.preventDefault();
  }

  ...

}
~~~~~~~~

Теперь на клиенте делается запрос к API только один раз, хотя вы дважды ищете поисковую строку. Даже данные с разбивкой на страницы с несколькими страницами кешируются таким образом, потому что вы всегда сохраняете последнюю страницу для каждого результата в объекте `results`. Разве это не мощный подход для внедрения кеширования в ваше приложение? API Hacker News предоставляет всё необходимое для того, чтобы эффективно кешировать данные с разбивкой на страницы.

## Обработка ошибок

Всё готово для работы с API Hacker News. Мы даже попробовали реализовать элегантный способ кеширования результатов из API и использовать возможность API для постраничного получения бесконечного списка историй. Но есть недостающая часть. К сожалению, сегодня она часто пропускается при разработке приложений — обработка ошибок. Слишком просто реализовать работающий сценарий функционирования приложения, не задумываясь о тех ошибках, которые могут произойти во время его работы.

В этом разделе главы вы познакомитесь с эффективным решением для добавления обработки ошибок для вашего приложения в случае ошибочного запроса API. Вы уже узнали о необходимых строительных блоках в React для реализации обработки ошибок: локальное состояние и отрисовка по условию. В основном, ошибка — это всего лишь другое состояние в React. При возникновении ошибки вы сохраните её в локальном состоянии и отобразите её, используя условную отрисовку в компоненте. Вот и всё. Давайте реализуем обработку ошибок в компоненте App, потому что прежде всего это компонент, который используется для получения данных из API Hacker News. Во-первых, нам нужно свойство для ошибки в локальном состоянии. Оно инициализируется значением `null`, но ему будет присвоен объект ошибки в случае неудачного запроса.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      results: null,
      searchKey: '',
      searchTerm: DEFAULT_QUERY,
# leanpub-start-insert
      error: null,
# leanpub-end-insert
    };

    ...
  }

...

}
~~~~~~~~

Во-вторых, вы можете использовать блок `catch` в нативном fetch для сохранения объекта ошибки в локальном состоянии с помощью `setState()`. Каждый раз, когда запрос к API потерпит неудачу, будет выполнен код в блоке catch.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  fetchSearchTopStories(searchTerm, page = 0) {
    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
# leanpub-start-insert
      .catch(error => this.setState({ error }));
# leanpub-end-insert
  }

  ...

}
~~~~~~~~

В-третьих, вы можете получить объект ошибки из своего локального состояния в методе `render()` и отобразить сообщение об ошибке с помощью условной отрисовки в React.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const {
      searchTerm,
      results,
      searchKey,
# leanpub-start-insert
      error
# leanpub-end-insert
    } = this.state;

    ...

# leanpub-start-insert
    if (error) {
      return <p>Что-то произошло не так.</p>;
    }
# leanpub-end-insert

    return (
      <div className="page">
        ...
      </div>
    );
  }
}
~~~~~~~~

Вот так. Если вы хотите проверить, работает ли обработка ошибок, вы можете изменить URL-адрес API на любой несуществующий.

{title="src/App.js",lang=javascript}
~~~~~~~~
const PATH_BASE = 'https://hn.foo.bar.com/api/v1';
~~~~~~~~

При использовании такого значения для константы пути к API вы должны получить сообщение об ошибке вместо отображения приложения. Это зависит от вас, где вы хотите поместить условную отрисовку для сообщения об ошибке. В данном случае всё приложение не будет отображаться. Это не самый лучший опыт для пользовательского взаимодействия. А что насчёт отображения компонента Table или сообщения об ошибке? Остальная часть приложения даже в случае ошибки всё равно будет отображаться.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const {
      searchTerm,
      results,
      searchKey,
      error
    } = this.state;

    const page = (
      results &&
      results[searchKey] &&
      results[searchKey].page
    ) || 0;

    const list = (
      results &&
      results[searchKey] &&
      results[searchKey].hits
    ) || [];

    return (
      <div className="page">
        <div className="interactions">
          ...
        </div>
# leanpub-start-insert
        { error
          ? <div className="interactions">
            <p>Something went wrong.</p>
          </div>
          : <Table
            list={list}
            onDismiss={this.onDismiss}
          />
        }
# leanpub-end-insert
        ...
      </div>
    );
  }
}
~~~~~~~~

Напоследок не забудьте вернуть прежний URL-адрес API.

{title="src/App.js",lang=javascript}
~~~~~~~~
const PATH_BASE = 'https://hn.algolia.com/api/v1';
~~~~~~~~

Ваше приложение должно по-прежнему работать, но на этот раз с обработкой ошибок в случае неудачного запроса к API.

### Упражнения:

* узнайте подробнее про [обработку ошибок в компонентах React](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

## Использование Axios вместо Fetch

В одном из предыдущих разделов вы внедрили нативный API fetch для выполнения запроса на платформу Hacker News. Браузер позволяет использовать этот нативный API fetch. Однако не все браузеры, особенно старые, поддерживают его. Кроме того, как только вы начнёте тестировать своё приложение в окружении браузера без пользовательского интерфейса (без браузера, а это только имитация), могут возникнуть проблемы с API fetch. Такое окружение браузера из консоли может быть при написании и выполнении тестов для вашего приложения, которые не запускаются в реальном браузере. Есть несколько способов сделать работу fetch в старых браузерах (через полифилы) и в тестах ([isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)), но в этой книге мы не будем заниматься этим.

Альтернативным способом решения этой проблемы было бы заменить нативный API fetch стабильной библиотекой, такой как [axios](https://github.com/axios/axios). Axios — это библиотека, которая решает только одну проблему, но делает это качественно: выполнение асинхронных запросов к удалённым API. Вот почему вы будете использовать его в этой книге. Конкретно этот раздел главы продемонстрирует вам, как вы можете заменить библиотеку (которая является нативным API браузера в этом случае) другой библиотекой. На абстрактном уровне он должен показать вам, как вы всегда можете найти решение для причуд (например, старых браузеров, тестов браузеров без пользовательского интерфейса) в веб-разработке. Поэтому никогда не переставайте искать решения, если что-то мешает вам.

Давайте посмотрим, как нативный API fetch можно заменить на axios. На самом деле всё сказанное ранее звучит сложнее, чем есть на самом деле. Во-первых, вам нужно установить axios в командной строке:

{title="Командная строка",lang="text"}
~~~~~~~~
npm install axios
~~~~~~~~

Во-вторых, импортировать axios в файл компонента App:

{title="src/App.js",lang=javascript}
~~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import axios from 'axios';
# leanpub-end-insert
import './App.css';

...
~~~~~~~~

И последнее, но не менее важное: вы можете использовать его вместо `fetch()`. Его использование почти идентично нативному API fetch. Он принимает URL в виде аргумента и возвращает промис. Вам больше не нужно преобразовывать возвращённый ответ JSON. Axios делает это за вас и обёртывает результат в объект `data` в JavaScript. Таким образом, не забудьте адаптировать свой код к возвращаемой структуре данных.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  fetchSearchTopStories(searchTerm, page = 0) {
# leanpub-start-insert
    axios(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
      .then(result => this.setSearchTopStories(result.data))
# leanpub-end-insert
      .catch(error => this.setState({ error }));
  }

  ...

}
~~~~~~~~

Это всё, что нужно для замены fetch на axios в этой главе. В вашем коде вы вызываете `axios()`, который по умолчанию использует HTTP GET-запрос. Вы можете сделать запрос GET явным образом, вызвав `axios.get()`. Также вы можете использовать другой HTTP-метод, такой как HTTP POST с помощью `axios.post()`. Сейчас вы уже видите, что axios — это мощная библиотека для выполнения запросов к удалённым API. Я часто рекомендую использовать его вместо нативного API fetch, когда ваши запросы API усложняются или вам приходится сталкиваться со странностями веб-разработки, связанными с промисами. Кроме того, в следующей главе вы познакомитесь с тестированием своего приложения. Тогда вам больше не нужно будет беспокоиться о браузере или окружении консольного браузера.

Я хочу представить ещё одно улучшение запроса к Hacker News в компоненте App. Представьте, что ваш компонент монтируется, когда веб-страница отображается в браузере в первый раз. В `componentDidMount()` компонент делает запрос, но затем, поскольку ваше приложение произвело некую навигацию, вы переходите от этой страницы к другой странице. Ваш компонент App демонтируется, но все ещё остаётся ожидающий запрос из вашего жизненного цикла `componentDidMount()`. Он попытается использовать `this.setState()` в конечном итоге в блоке промиса `then()` или `catch()`. Возможно, в первый раз вы увидите следующее предупреждение в командной строке или в выводе консоли разработчика браузера:

{title="Командная строка",lang="text"}
~~~~~~~~
Warning: Can only update a mounted or mounting component. This usually means you called setState, replaceState, or forceUpdate on an unmounted component. This is a no-op.
~~~~~~~~

Вы можете решить эту проблему, прервав запрос, когда ваш компонент размонтируется или предотвратить вызов `this.setState()` в размонтированном компоненте. Это передовая практика в React, хотя ей следуют далеко не все разработчики, чтобы оставить чистое приложение без каких-либо раздражающих предупреждений. Однако в текущем API промисов не реализует прерывание запроса. Таким образом, вам нужно самостоятельно справиться с этой проблемой. Возможно также, это та причина, по которой не многие разработчики следуют этой лучшей практике. Следующая реализация выглядит скорее как обходное решение, чем поддерживаемая в дальнейшем реализация. Исходя из этого вы можете самостоятельно решить, хотите ли вы это реализовывать, чтобы обойти предупреждение из-за размонтированного компонента. Тем не менее, помните об этом предупреждении, если оно появится в следующей главе этой книги или в вашем собственном приложении. Теперь в подобных случаях вы знаете как справиться с этим.

Давайте перейдём к решению проблемы. Вы можете добавить поле класса, которое содержит состояние жизненного цикла вашего компонента. Он может быть инициализирован как `false`, когда компонент инициализируется, и изменяется на `true`, когда компонент установлен, а затем снова устанавливается на `false` при удалении компонента. Таким образом, вы можете отслеживать состояние жизненного цикла вашего компонента. Это поле не имеет ничего общего с локальным состоянием, которое хранится и модифицируется с помощью `this.state` и `this.setState()`, поскольку у вас должен быть к нему доступ непосредственно в экземпляре компонента, не полагаясь на управление локальным состоянием React. Более того, это не приводит к повторной отрисовке компонента, когда поле класса изменяется таким образом.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
# leanpub-start-insert
  _isMounted = false;
# leanpub-end-insert

  constructor(props) {
    ...
  }

  ...

  componentDidMount() {
# leanpub-start-insert
    this._isMounted = true;
# leanpub-end-insert

    const { searchTerm } = this.state;
    this.setState({ searchKey: searchTerm });
    this.fetchSearchTopStories(searchTerm);
  }

# leanpub-start-insert
  componentWillUnmount() {
    this._isMounted = false;
  }
# leanpub-end-insert

  ...

}
~~~~~~~~

Наконец, вы можете использовать эти знания, чтобы не прервать сам запрос, а избежать вызов `this.setState()` в вашем экземпляре компонента, даже если компонент уже удалён. Это предотвратит упомянутое предупреждение.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  fetchSearchTopStories(searchTerm, page = 0) {
    axios(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
# leanpub-start-insert
      .then(result => this._isMounted && this.setSearchTopStories(result.data))
      .catch(error => this._isMounted && this.setState({ error }));
# leanpub-end-insert
  }

  ...

}
~~~~~~~~

В целом в главе показано, как вы можете заменить одну библиотеку другой библиотекой в React. Если у вас возникнут какие-либо проблемы, вы можете использовать обширную экосистему JavaScript-библиотек, чтобы помочь самому себе. Кроме того, вы видели способ, как можно избежать вызова `this.setState()` в React на размонтированном компоненте. Если вы получше изучите библиотеку axios, вы также найдёте способ отменить запрос. Вам предстоит больше узнать об этой теме.

### Упражнения:

* прочитайте про то, [почему выбор фреймворка имеет значение](https://www.robinwieruch.de/why-frameworks-matter/)
* узнайте больше про [альтернативный синтаксис для определения компонента](https://github.com/the-road-to-learn-react/react-alternative-class-component-syntax)

{pagebreak}

Вы научились взаимодействовать с API в React! Давайте повторим последние темы:

* React
  * Методы жизненного цикла ES6-класса компонента для разных случаев использования
  * `componentDidMount()` для взаимодействий с API
  * отрисовка по условию
  * синтетические события на формах
  * обработка ошибок
  * отмена удалённого API-запроса
* ES6 и за его пределами
  * шаблонные строки для объединения строк
  * оператор расширения для неизменяемых структур данных
  * вычисляемые имена свойств
  * поля класса
* Общее
  * Работа с API Hacker News
  * API нативного fetch в браузере
  * Поиск на стороне клиента и сервера
  * Разбивка на страницы данных
  * Кеширование на стороне клиента
  * Использование axios в качестве альтернативы для нативного API fetch

Повторю снова, имеет смысл сделать перерыв. Усвоить полученные знания и применить их на практике самостоятельно. Вы можете поэкспериментировать с исходным кодом, который вы написали к настоящему времени. Исходный код можно найти в [официальном репозитории](https://github.com/the-road-to-learn-react/hackernews-client/tree/5.3.1).

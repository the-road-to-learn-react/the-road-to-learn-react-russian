# Получение реальных данных с API

Теперь пришло время получать реальные данные через API, поскольку может быть скучно работать с демонстрационными данными.

Если вы не знакомы с API, я рекомендую вам [мое путешествие по моему знакомоству с API](https://www.robinwieruch.de/what-is-an-api-javascript/).

Вы знаете платформу [Hacker News](https://news.ycombinator.com/)? Это отличный новостной агрегатор на технические темы. В этой книге мы будем использовать API Hacker News для получения трендовых историй. Для того, чтобы это сделать есть [базовый](https://github.com/HackerNews/API) и [поисковый](https://hn.algolia.com/api) API для получения данных с этой платформы. Последний предназначен для нашего разрабатываемого приложения для поиска историй на Hacker News. Вы можете открыть спецификацию API для понимания структуры данных.

## Методы жизненного цикла

Вам нужно будет узнать о методах жизненного цикла React, до того, как вы начнёте получение данных в ваших компонентах с использованием API. Эти методы — хуки в жизненном цикле React-компоненте. Их можно использовать в классе компонента из ES6, но не в функциональных компонентах без состояния.

Помните, в предыдущей главе вы изучали классы в JavaScript ES6 и как их использовать в React? Помимо метода `render()` существуют несколько методов, которые переопределяются в классовых компонентах React. Всё это методы жизненного цикла, давайте погрузимся в них:

Вам уже знакомы два методы жизненного цикла, которые можно использовать к классе компонента: `constructor()` и `render()`.

Конструктор вызывается только тогда, когда экземпляр компонента создан и добавлен в DOM, т.е. компонент проинициализирован. Этот процесс называется монтированием (установкой) компонента.

Метод `render()` также вызывается во время процесса монтирования, но ещё и при обновлении компонента. Каждый раз при изменении состояния или свойств компонента, вызывается метод `render()`.

Теперь вы узнали больше об этих двух методах жизненного цикла и в каких случаях они вызываются. Также, вы уже использовали их, но есть и другие, которые вы сейчас рассмотрим.

У монтирования компонента есть два метода жизненного цикла: `getDerivedStateFromProps()` и `componentDidMount()`. Сначала вызывается конструктор, метод `getDerivedStateFromProps()` вызывается до метода `render()` и `componentDidMount()` вызывается после метода `render()`.

В целом процесс монтирования имеет 4 метода жизненного цикла. Они вызываются в следующем поряке:

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

И последнее, но не менее важное — демонтирование жизненного цикла. Для этого процесса доступен только один метод — `componentWillUnmount()`.

В конце концов, вам не нужно знать все эти методы жизненного цикла с самого начала. Это может быть пугающим, но вы не будете все их использовать. Даже в крупном React-приложении вы будете использовать только некоторые из них помимо методов `constructor()` и `render()`. Тем не менее, хорошо знать, какой метод жизненного цикла в каких случаях может использоваться:

* **constructor(props)** - вызывается, когда компонент инициализируется. Вы можете установить начальное состояние компонента и связать методы класса во время этого метода жизненного цикла.

* **static getDerivedStateFromProps(props, state)** — вызывается перед методом жизненного цикла `render()` как при начальном монтировании, так и при последующих обновлениях. Этот метод должен возвратит объект для обновления состояния или `null`, если ничего не нужно обновлять. Он существует для **редких** случаев использования, когда состояние зависит от изменений в свойствах со временем. Важно знать, что это статический метод, и поэтому у него нет доступа к экземпляру компонента.

* **render()** - этот метод жизненного цикла является обязательным и возвращает элементы в качестве вывода компонента. Метод должен быть чистым, т.е. не изменить состояние компонента. Он получает входные данные в виде свойств и состояния и возвращает элемент.

* **componentDidMount()** - вызывается только один раз при монтировании компонента. Это идеальное время для выполнения асинхронных запросов на получение данных из API. Полученные данные будут сохранены во внутреннем состоянии компонента для его отображения в методе жизненного цикла `render()`.

* **shouldComponentUpdate(nextProps, nextState)** - всегда вызывается, когда компонент изменяется во время изменения состояние или свойств. Вы будете использовать его в зрелых React-приложений для оптимизации производительности. В зависимости от возвращаемого логического значения из метода жизненного цикла, компонент и все его дочение элементы будут отрисовываться или нет в зависимости от обновления жизненного цикла. Этот метод может предотвратить отрисовку компонента.

* **getSnapshotBeforeUpdate(prevProps, prevState)** — вызывается непосредственно перед тем, как последний отрисованный вывод будет зафиксирован в DOM. В редких вариантах использования компонент должен захватить информацию из DOM прежде чем он в принципе будет изменён. Этот метод жизненного цикла позволяет компоненту это сделать. Другой метод (`componentDidUpdate()`) получит любое значение, возвращаемое `getSnapshotBeforeUpdate()` в качестве параметра.

* **componentDidUpdate(prevProps, prevState, snapshot)** - The lifecycle method is immediately invoked after updating occurs, but not for the initial render. You can use it as opportunity to perform DOM operations or to perform further asynchronous requests. If your component implements the `getSnapshotBeforeUpdate()` method, the value it returns will be received as the `snapshot` parameter.

* **componentDidUpdate(prevProps, prevState)** - вызывается сразу после обновления, но не при первоначальной отрисовке. метода `render()`. Вы можете использовать его в виде возможности выполнять операции с DOM или выполнять дальнейшие асинхронные запросы. Если ваш компонент реализует метод `getSnapshotBeforeUpdate()`, возвращаемое им значение будет приниматься в качестве параметра `snapshot`.

* **componentWillUnmount()** - вызывается до того, как компонент будет уничтожен. Вы можете использовать этот мето для выполнения любых задач очистки.

Вы уже использовали методы `constructor()` и `render()`. Это часто используемые методы жизненного цикла в классовых компонентах. На самом обязательный только метол `render()`, в противном случае вы не возвратите экземпляр компонента.

Существует ещё один метод жизненного цикла — `componentDidCatch(error, info)`. Он представлен в [React 16](https://www.robinwieruch.de/what-is-new-in-react-16/) и используется для обработки ошибок в компонентах. К примеру, список хорошо отображается в приложении. Но, что если список в локальном состоянии случайно установлен в `null` (например, при получении данных из внешнего API, но запрос не завершился неудачно, поэтому вы установили локальное состояние для списка в значение `null`). Впоследствии было невозможно фильтровать и отображать список, поскольку вместо элементов списка — значение `null`. Компонент будет сломан, и приложение полностью потерпит неудачу. Теперь, используя `componentDidCatch()`, вы можете перехватить ошибку, сохранить её в локальном состоянии и необязательно показать сообщение в приложении для уведомления пользователя об ошибке.

### Упражнения:

* узнать подробнее про [методы жизненного цикла в React](https://reactjs.org/docs/react-component.html)
* узнать подробнее о [состоянии, связанном с методами жизненного цикла в React](https://reactjs.org/docs/state-and-lifecycle.html)
* узнать подробнее про [обработку ошибок в компонентах](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

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

В JavaScript ES6 вы можете использовать [шаблонные строки](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals) для конкатенации строк. В вашем случае вы будете его использовать для конкатенации  URL-адреса к конечной точки (endpoint) API.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// ES6
const url = `${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${DEFAULT_QUERY}`;

// ES5
var url = PATH_BASE + PATH_SEARCH + '?' + PARAM_SEARCH + DEFAULT_QUERY;

console.log(url);
// вывод: https://hn.algolia.com/api/v1/search?query=redux
~~~~~~~~

Это позволит сохранить гибкость структуры URL-адреса в будущем.

Но давайте перейдем к API-запросу, в котором будет использоваться URL-адрес. Весь процесс получения данных приводится сразу, но каждый шаг будет объяснен позже.

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

В коде много чего происходит. Я думал о том, чтобы разбить его на небольшие части. Тогда снова было бы трудно понять связи каждой части друг к другу. Позвольте мне подробно объяснить каждый шаг..

Во-первых, вы можете удалить демонстрационный список элементов, потому что вы возвращаете реальный список из API Hacker News. Демонстрационные данные больше не используются. Исходное состояние вашего компонента теперь имеет пустой результат и поисковый запрос по умолчанию. Тот же самый поисковый запрос по умолчанию используется в поле ввода компонента Search и в вашем первом запросе.

Во-вторых, вы используете метод жизненного цикла `componentDidMount()` для получения данных после монтирования компонента. При самом первом получении используется поисковый запрос по умолчанию из локального состояния. Он будет получить истории, связанные с "redux", поскольку это параметр по умолчанию.

В-третьих, используется нативный API. Шаблонные строки в JavaScript ES6 позволяют ему составлять URL-адрес с `searchTerm`. URL-адрес — аргумент для нативной функции API fetch. Ответ требуется преобразовать в структуру данных JSON, что является обязательным шагом в нативной функции fetch при работе с JSON-данными, и, наконец, они могут сохранены в качестве результата во внутреннем состоянии компонента. Кроме того, блок `catch` используете в случае ошибки. Если во время произошла ошибка, выполнение функции перейдет из блока `then` в блок `catch`. В следующей главе книги будет включена обработка ошибок.

И последнее, но не менее важно: не забудьте связать новый метод компонента в конструкторе.

Теперь вы можете использовать полученные данные вместо демонстрационного списка элементов. Однако нужно быть осторожным. Результат содержит не только список данных. [Это сложный объект с метаинформацией и свойством hits, который содержит нужные нам истории](https://hn.algolia.com/api). Вы можете вывести внутреннее состояние через `console.log(this.state);` в методе для `render()` для наглядного отображения.

Следующим шагом будет использование результата для его отрисовки. Мы предотвратим отрисовку, вернув значение `null`, если данных с API нет в первый раз. После того, как запрос к API выполнен успешно, результат будет сохранен в состоянии и компонент App повторно отрисуется с обновлённым состоянием.

In the next step, you will use the result to render it. But we will prevent it from rendering anything, so we will return null, when there is no result in the first place. Once the request to the API succeeded, the result is saved to the state and the App component will re-render with the updated state.

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

Давайте вспомним, что происходит во время жизненного цикла компонента. Компонент инициализируется через конструктор. После этого он отрисовывается в первый раз. Но вы препятствуете отрисовки чего-либо, поскольку результат в локальном состоянии равен `null`. Разрешено возвращать `null` из компонента, если нечего нет данных для отображения. Затем выполняется метод жизненного цикла `componentDidMount()`. В этом методе вы асинхронно получаете данные из API Hacker News. Как только данные придут, изменится внутреннее состояние компонента в `setSearchTopStories()`. Поэтому вступает в действие обновление жизненного цикла, потому что локальное состояние обновлено. Компонент снова выполняет метод `render()`, но на этот раз с заполненными данными результата во внутреннем состоянии компонента. Компонент, и таким образом, компонент Table с его содержимым будут отрисовываться.

Вы использовали нативный API fetch, поддерживаемый большинством браузеров для выполнения асинхронных запросов. Конфигурация *create-react-app*  гарантирует его поддержку в каждом браузере. Существуют также сторонние пакеты, которыми можно заменить нативный API fetch: [superagent](https://github.com/visionmedia/superagent) and [axios](https://github.com/mzabriskie/axios).

Имейте в виду, что в книге используется сокращённая нотация JavaScript для проверки истинности. В предыдущем примере `if (!result)` вместо `if (result === null)`. То же самое относится и к другим частям примеров на протяжении всей книги. Например, используете `if (!list.length)` вместо `if (list.length === 0)` или `if (someString)` вместо `if (someString !== '')`. Почитайте тему, если вы не слишком знакомы с этим.

Вернёмся к нашему приложению: показывается список историй. Однако сейчас в приложении есть два регресионных бага. Во-первых, кнопка "Отбросить" не работает. Она не знает об сложным объекте результата и по-прежнему работает на обычным списке из локального состояния при отклонении элемента. Во-вторых, при попытке поиска в отображенном списке, он фильтруется на стороне клиента, даже если изначальное получение данных было выполнено посредством поиска историй на стороне сервера. Идеально было бы в таком случае было бы получение другого объекта результата из API при использовании компонента Search. Обе ошибки регрессии будет исправлены в следующих главах.

### Упражнения:

* узнать подробнее про [строковые шаблоны ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals)
* узнать подробнее про [нативный API fetch](https://developer.mozilla.org/en/docs/Web/API/Fetch_API)
* узнать подробнее про [получение данных в React](https://www.robinwieruch.de/react-fetching-data/)

## ES6 Spread Operators

Кнопка "Отбросить" не работает, потому что метод `onDismiss()` не знает о сложным результирующем объекте. Он знает только про простой список в локальном состоянии. Но это уже не простой список. Давайте изменим его для работы с результирующим объектом вместо с самим списком.

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

{title="Code Playground",lang="javascript"}
~~~~~~~~
// не делайте этого
this.state.result.hits = updatedHits;
~~~~~~~~

React охватывает неизменяемые структуры данных. Таким образом, вы не должны изменять объект (или напрямую изменить состояние). Лучший подход — создать новый объект на основе имеющийся информации. Таким образом, ни один из объектов не будет изменён. Вы сохраните неизменяемые структуры данных. Вы всегда будет возвращать новый объект и никогда не изменять объект.

Поэтому вы можете использовать `Object.assign()` в JavaScript ES6. Он принимает в качестве первого аргумента целевой объект. Все следующие аргументы — исходные объекты. Эти объекты объединяются в целевой объект. Целевой объект может быть пустым. Он охватывает неизменяемость, потому что ни один из исходных объектов не изменяется. В итоге код будет выглядеть следующим образом:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const updatedHits = { hits: updatedHits };
const updatedResult = Object.assign({}, this.state.result, updatedHits);
~~~~~~~~

Последующие объекты будут переопределять прежние объединённые объекты, когда у них одинаковые имена свойств. Теперь давайте применим это в методе `onDismiss()`:

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

Мы это уже использовали. Но в JavaScript ES6 и будущих релизах JavaScript есть более простой способ. Могу ли представить вам spread operator? Он состоит из трёх точек: `...` При его использовании каждое значение из массива или объекта копируется в другой массив или объект.

Давайте изучим оператор **array** ES6, даже если он вам ещё не нужен.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const userList = ['Robin', 'Andrew', 'Dan'];
const additionalUser = 'Jordan';
const allUsers = [ ...userList, additionalUser ];

console.log(allUsers);
// выведет ['Robin', 'Andrew', 'Dan', 'Jordan']
~~~~~~~~

Переменная `allUsers` — это полностью новый объект. Остальные переменные `userList` и `additionalUser` остаются такими же. Вы можете объединить два массива таким образом в новый массив.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const oldUsers = ['Robin', 'Andrew'];
const newUsers = ['Dan', 'Jordan'];
const allUsers = [ ...oldUsers, ...newUsers ];

console.log(allUsers);
// выведет ['Robin', 'Andrew', 'Dan', 'Jordan']
~~~~~~~~

Теперь давайте посмотрим на оператора object spread operator. Это не JavaScript ES6. Это [предложение для следующей версии JavaScript](https://github.com/sebmarkbage/ecmascript-rest-spread), которое уже используется сообществом React. Именно поэтому *create-react-app* включило эту возможно в свою  конфигурацию.

В основном это то же самое, что и оператор JavaScript ES6 array spread, но с объектами. Он копирует каждую пару ключ-значение в новый объект.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const userNames = { firstname: 'Robin', lastname: 'Wieruch' };
const age = 28;
const user = { ...userNames, age };

console.log(user);
// выведет { firstname: 'Robin', lastname: 'Wieruch', age: 28 }
~~~~~~~~

Несколько объект может использоваться, как в аналогичном примере с массивом.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const userNames = { firstname: 'Robin', lastname: 'Wieruch' };
const userAge = { age: 28 };
const user = { ...userNames, ...userAge };

console.log(user);
// выведет { firstname: 'Robin', lastname: 'Wieruch', age: 28 }
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

Теперь кнопка "Отбросить" должна снова работать, потому что метод `onDismiss()` знает о сложном результирующем объекте и как его обновлять после отклонения элемента из списка.

### Упражнения:

* read more about the [ES6 Object.assign()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
* read more about the [ES6 array spread operator](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator)
  * the object spread operator is briefly mentioned

## Условная отрисовка

Условная отрисовка вводится довольно рано в React-приложения. Но не в этой книге, потому что не было такого случая использования. Условная отрисовка происходит, когда вы хотите принять решение об отрисовке того или иного элемента. Иногда это означает отрисовать элемент, либо вообще ничего. В конце концов, упрощённое использование условной отрисовки может быть сформулировано выражением if-else в JSX.

Объект `result` во внутреннем состоянии объекта равен значению `null` в самом начале. До сих пор компонент App не возвращал элементов, когда `result` не был заполнен с API. Это уже условная отрисовка, потому что возвращается заранее что-то из метода жизненного цикла `render()` в зависимости от определённого условия. Компонент App либо отрисовывает свои элементы, либо ничего не отрисовывает.

Но давайте пойдём дальше. Имеет смысл обернуть компонент Table, который является единственным компонентом, который зависит от `result`, в независимой условной отрисовки. Всё остальное должно отображаться, даже если пока `result` ещё нет. Вы можете просто использовать тернарный оператор в JSX.

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

Это второй вариант выражения условной отрисовки. Третий вариант заключается в логическом операторе `&&`. В JavaScript выражение `true && 'Hello World'` всегда выполняется как 'Hello World'. А выражение `false && 'Hello World'` всегда будет false.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const result = true && 'Hello World';
console.log(result);
// выведет Hello World

const result = false && 'Hello World';
console.log(result);
// выведет false
~~~~~~~~

В React вы можете использовать подобное поведение. Если условие равняется true, выражение после оператора `&&` будет выведено. Если условие равняется false, React проигнорирует и пропустит выражение. Это подходит в случае условной отрисовки компонента Table, поскольку он должен возвращать Table, а может его не вернуть.

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

Сейчас были показаны несколько подходов использования условной отрисовки в React. Вы можете узнать [больше о друших альтернатив в исчерпывающем списке примеров условной отрисовки для подходов условной отрисовки](https://www.robinwieruch.de/conditional-rendering-react/). Кроме того, вы узнаете об их различных вариантах использования и применения ихyou will get to know their different use cases and when to apply them.

В конце концов,вы должны иметь возможность видеть полученные данные в приложении. Всё, кроме таблицы, отображается, когда получение данные находятся только в процессе получения. После того как результат будет получен из запроса и сохранен в локальном состоянии, будет отображена таблица, потому что метод `render()` запускается снова, и условие разрешается в пользу отобрражения компонента таблицы.

### Exercises:

* read more about [different ways for conditional renderings](https://www.robinwieruch.de/conditional-rendering-react/)
* read more about [React conditional rendering](https://reactjs.org/docs/conditional-rendering.html)

## Поиск на стороне клиента и на стороне сервера

Теперь когда вы используете компонент поиска со своим полем ввода, вы будете фильтровать список. Это происходит на стороне клиента. Теперь вы будете использовать для поиска на стороне сервера. В противном случаевы будете иметь дело только с первым запроосо, который вы получили на `componentDidMount()` с параметром поиска по умолчанию.

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

Метод `onSearchSubmit()` должен использовать ту же функциональность, что и метод жизненного цикла `componentDidMount()`, но на этот раз с измененнем поисковым запросом из локального состояния, а не с начальным запросом поиска по умолчанию. Таким образом, вы можете извлечь функциональность как метод класса для повторного использования.

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

Теперь компонент Search должен добавить дополнительную кнопку. Кнопка должна явно вызывать запроса на поиск. В противном случае вы будете получать данные с Hacker News API каждый раз, когда изменяется поле ввода. Вы вы хотите сделать это явно в обработчике  `onClick()` .

В качестве альтернативы вы можете отбросить (отложить, задержать) функцию `onChange()` и сохрнаить эту кнопку, но в этом время она добавит больше слложности и, возможно, не будет желаемого эффекта . Давайте его оставим его простым без задержки.

Сначала передаём метод `onSearchSubmit()` в компонент Search.

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
            Search
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

Во-вторых, введём компонент Search. У кнопки будет `type="submit"`, и форма использует свой атрибут `onSubmit()` для передачи метода `onSubmit()`. Вы можете повторно использовать свойство children, но на этот раз оно будет использовать как содержимое кнопки.

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

В компоненте Table можно удалить функциональность фильтра, потому что больше не будет фильтра (поиска) на стороне клиента. Не забудьте также удалить функцию `isSearched()`. Он больше не будет использоваться. Результат приходит непосредственно из Hacker News API после того, как была нажата кнопка  "Поиск".

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

Когда вы пытаетесь выполнить сейчас, вы заметите, что браузер перезагружается. Это стандартное поведение браузера для колбэка при отправки формы HTML. В React вы часто столкнётесь с методом события `preventDefault()` для подавления поведения нативного браузера.

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

Теперь вы сможете искать различные истории по Hacker News. Идеально, вы взаимодействуете с реальным API. Больше не должно быть поиска на стороне клиента.

### Exercises:

* read more about [synthetic events in React](https://reactjs.org/docs/events.html)
* experiment with the [Hacker News API](https://hn.algolia.com/api)

## Получение данные с разбивкой на страницы

Вы внимательно рассмотрели структуры возвращённых данных ещё? [Hacker News API](https://hn.algolia.com/api) больше данных, чем просто список историй hits. Именно он возвращает список с разбивкой страниц. Свойство страницы, равное `0` в первом ответе, может быть использовано для получения большего количество подсписков в качестве результата. Вам нужно передать следующую страницу с тем же самим поисковым запросам к API.

Давайте расширим вспомогательные константы для API, чтобы он мог работать с разбивкой на страницы данных.

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
{title="Code Playground",lang="javascript"}
~~~~~~~~
const url = `${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}`;

console.log(url);
// выведет: https://hn.algolia.com/api/v1/search?query=redux&page=
~~~~~~~~

Метод `fetchSearchTopStories()` будет принимать страницу в качестве второго аргумента. Если второй аргумент не предоставлен, будет использоваться страница `0` для первоначального запроса. Таким образом, методы `componentDidMount()` и `onSearchSubmit()` извлекают первую страницу по первому запросу. Каждая дополнительная выборка будет отображать следующую страницу, передавая её вторым аргументом.

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

Аргумент страницы использует значения параметров по умолчанию из JavaScript ES6, чтобы вернуться к странице `0` в случае, если для функции не предоставлен аргумент страницы.

Теперь мы знаем, как получить текущую страницу из ответа API в `fetchSearchTopStories()`. Вы можете использовать этот метод при нажатии на кнопку для получения ещё больше историй в обработчике кнопки `onClick`. Давайте используем компонент Button для выборки большего количества данных с разбивкой на страницы из API Hacker News. Вам нужно только определить обработчик `onClick()`, который принимает текущий поисковый запрос пользователя и следующую страницу (текущая страница + 1).

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

Во-вторых, вам нужно проверить, есть ли уже старые истории. Когда страница равна 0, значит это новый поисковый запрос от `componentDidMount()` или `onSearchSubmit()`. Истории пустые. Но когда вы нажимаете на кнопку «Больше историй», чтобы получить больше данных с разбивкой по страницам, текущая страница больше не будет равняться 0. Это следующая страница. Старые истории уже хранятся в состоянии компонента и поэтому могут быть использованы.

В-третьих, вы не хотите переопределять старые истории. Вы можете объединить старые и новые истории из последнего запроса API. Объединение обоих списков может быть выполнено с помощью оператора расширения массива из JavaScript ES6.

В-четвертых, вы установили объединенные истории и страницу в состоянии локального компонента.

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

### Exercises:

* read more about [ES6 default parameters](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters)
* experiment with the [Hacker News API parameters](https://hn.algolia.com/api)

## Кеш клиента

При каждой отправке формы поиска отправляется запрос к API Hacker News. Вы можете поискать «redux», а «затем react», и в конце концов снова «redux». Таким образом, получаются 3 запроса. Но вы дважды искали «redux», и оба раза для получения данных потребовалось полное асинхронное получение данных в обе стороны. В кеше на стороне клиента вы сохраните каждый результат. Когда выполняется запрос к API, проверяется, есть ли уже результат. Если он есть, используется кеш. В противном случае для получения данных выполняется запрос к API.

Для обеспечения существования клиентского кеша для каждого результата, вам нужно сохранить несколько результатов, а не один результат во внутреннем состоянии компонента. Объектом результатов будет объект с поисковой строкой в качестве ключа, а результат в виде значения. Каждый результат API будет сохранен по поисковой строку (ключю).

На данный момент ваш результат в локальном состоянии выглядит примерно так:

{title="Code Playground",lang="javascript"}
~~~~~~~~
result: {
  hits: [ ... ],
  page: 2,
}
~~~~~~~~

Представьте, что вы сделали два запроса API. Один для поисковой строки «redux» и ещё один для «react». Объект результатов должен выглядеть следующим образом:

{title="Code Playground",lang="javascript"}
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

Давайте реализуем кеширование на стороне клиента с помощью `setState()`. Во-первых, переименуйте объект `result` в `results` в первоначальном состоянии компонента. Во-вторых, определите временный `searchKey`, который используется для хранения каждого результата (`result`).

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

Ключ `searchKey` должен быть установлен перед каждым запросом. Он отражает `searchTerm`. Вы могли бы задаться вопросом: почему мы не используем `searchTerm` в первую очередь? Это важная часть для понимания, прежде чем продолжить реализацию. `searchTerm` — это неустойчивая (временная) переменная, поскольку она изменяется каждый раз при вводе в поле поиска. Однако в конце вам понадобится постоянная переменная. Она определяет недавно отправленный поисковый запрос к API и может использоваться для получения правильного результата из объекта с результатами. Это указатель на ваш текущий результат в кеше и, следовательно, её можно использовать для отображения текущего результата в методе `render()`.

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

В-третьих, новый результат может быть задан в объекте `searchKey` состояния. Рассмотрим объект `searchKey` в `setState()`.

{title="src/App.js",lang=javascript}
~~~~~~~~
results: {
  ...results,
  [searchKey]: { hits: updatedHits, page }
}
~~~~~~~~

Нижняя часть гарантирует сохранение обновленного результата с помощью `searchKey` в объекте результатов. Значение — это объект со свойствами hits и page. `searchKey` — это поисковая строка. Вы уже изучили синтаксис `[searchKey]: ...`. Это имя вычисляемого значения ES6. Оно помогает динамически распределять значения в объекте.

Верхняя часть должна добавить все остальные результаты по `searchKey` в состоянии с помощью оператора расширения объектов. В противном случае вы потеряете все результаты, которые вы сохранили ранее.

Теперь вы сохраняете все результаты по поисковой строке. Это первый шаг для включения кеша. На следующем шаге вы можете получить результат в зависимости от не временного значения в `searchKey` в объекте результатов. Вот почему вам нужно было ввести `searchKey` в первую очередь как не временную переменную. В противном случае процесс поиска будет нарушен, если вы будете использовать временный `searchTerm` для получения текущего результата, потому что это значение может измениться, если вы будете использовать компонент Search.

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

Поскольку по умолчанию у нас пустой список, потому что нет результата по `searchKey`, мы можем сейчас сэкономить условную отрисовку для компонента Table. Кроме того, вам нужно будет передать `searchKey`, а не` searchTerm` в кнопку «Больше историй». В противном случае получение данных по развивке по страницах зависит от значения `searchTerm`, которое является изменчивым. Кроме того, не забудьте сохранить свойство `searchTerm` для поля ввода в компоненте Search.

Функциональность поиска должна снова заработать. Она сохраняет все результаты от API Hacker News.

Кроме того, метод `onDismiss()` должен быть улучшен. Он по-прежнему относится к объекту `result`. Теперь он должен иметь дело с несколькими результатами (`results`).

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

Тем не менее, ничто не мешает приложению отсылать запрос к API при каждой отправке на поиск. Несмотря на то, что результат уже может быть, нет проверки, которая предотвращает запрос. Таким образом, функциональность кеша ещё не завершена. Она кеширует результаты, но не использует их. Последним шагом было бы предотвратить запрос к API, когда результат имеется в кеше.

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

Теперь на клиенте делается запрос к API только один раз, хотя вы дважды ищете поисковую строку. Даже данные с разбивкой на страницы с несколькими страницами кешируются таким образом, потому что вы всегда сохраняете последнюю страницу для каждого результата в объекте `results`. Разве это не мощный подход для внедрения кеширования в ваше приложение? API Hacker News предоставляет  всё необходимое для того, чтобы эффективно кешировать данные с пагинацией.

## Обработка ошибок

Всё готово для работы с API Hacker News. Мы даже попробовали сделать элегантный способ кешировать результаты из API и использовать его возможность с пагинацией списка для получения бесконечного списка историй из API. Но есть недостающая часть. К сожалению, сегодня она часто пропускается при разработке приложений — обработка ошибок. Слишком просто реализовать работающий сценарий функционирования приложения, не задумываясь о тех ошибках, которые могут произойти во время его работы.

В этом разделе главы вы познакомитесь с эффективным решением для добавления обработки ошибок для вашего приложения в случае ошибочного запроса API. Вы уже узнали о необходимых строительных блоках в React для реализации обработки ошибок: локальное состояние и условный рендеринг. В основном, ошибка — это всего лишь другое состояние в React. При возникновении ошибки вы сохраните её в локальном состоянии и отобразите её, используя условную отрисовку в компоненте. Вот и всё. Давайте реализуем обработку ошибок в компоненте App, потому что прежде всего это компонент, который используется для получения данных из API Hacker News. Во-первых, нам нужно свойство для ошибки в локальном состоянии. Он инициализируется значением null, но ему будет присвоен объект ошибки в случае неудачного запроса.

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

Во-вторых, вы можете использовать блок catch в нативном fetch для хранения объекта ошибки в локальном состоянии с помощью `setState()`. Каждый раз, когда запрос к API терпит неудачу, будет выполнен код в блоке catch.

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
      return <p>Something went wrong.</p>;
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

Afterward, you should get the error message instead of your application. It is up to you where you want to place the conditional rendering for the error message. In this case, the whole app isn't displayed anymore. That wouldn't be the best user experience. So what about displaying either the Table component or the error message? The remaining application would still be visible in case of an error.


При использовании такого значения для константы пути к API вы должны получить сообщение об ошибке вместо отображения приложения. Это зависит от вас, где вы хотите поместить условную отрисовку для сообщения об ошибке. В данном случае всё приложение не будет отображаться. Это не самый лучший опыт пользовательского взаимодействия. Так что насчёт отображения компонента Table или сообщения об ошибке? Остальная часть приложения даже в случае ошибки всё равно будет отображаться.

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

### Exercises:

* read more about [React's Error Handling for Components](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

## Axios instead of Fetch

В одном из предыдущих разделов вы внедрили нативный API fetch для выполнения запроса на платформу Hacker News. Браузер позволяет использовать этот нативный API fetch. Однако не все браузеры, особенно старые браузеры, поддерживают его. Кроме того, как только вы начнёте тестировать свое приложение в окружении браузера без пользовательского интерфейса (без браузера, а это только имитация), могут возникнуть проблемы с API fetch. Такое окружение браузера из консоли может произойти при написании и выполнении тестов для вашего приложения, которые не запускаются в реальном браузере. Есть несколько способов сделать работу fetch в старых браузерах (через полифилы) и в тестах ([isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)), но в этой книге мы не будем заниматься этим.

Альтернативным способом решения этой проблемы было бы заменить нативный API fetch стабильной библиотекой, такой как [axios](https://github.com/axios/axios). Axios — это библиотека, которая решает только одну проблему, но делает это качественно: выполнение асинхронных запросов к удаленным API. Вот почему вы будете использовать его в этой книге. Конкретно глава должна показать вам, как вы можете заменить библиотеку (которая является нативным API браузера в этом случае) с другой библиотекой. На абстрактном уровне он должен показать вам, как вы всегда можете найти решение для причуд (например, старых браузеров, тестов браузеро без пользовательского интерфейса) в веб-разработке. Поэтому никогда не переставайте искать решения, если что-то мешает вам.

Давайте посмотрим, как нативный API fetch можно заменить на axios. На самом деле всё сказанное раньше звучит сложнее, чем есть. Во-первых, вы должны установить axios в командной строке:

{title="Command Line",lang="text"}
~~~~~~~~
npm install axios
~~~~~~~~

Во-вторых, вы можете импортировать axios в файл компонента App:

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

Это всё, что нужно для замены fetch на axios в этой главе. В вашем коде вы вызываете `axios()`, который по умолчанию использует HTTP GET-запрос. Вы можете сделать запрос GET явным образом, вызвав `axios.get()`. Также вы можете использовать другой HTTP-метод, такой как HTTP POST с помощью `axios.post()`. Сейчас вы уже видите, что axios — это мощная библиотека для выполнения запросов к удалённым API. Я часто рекомендую использовать его вместо нативного API fetch, когда ваши запросы API усложняются или вам приходится сталкиваться со странностями веб-разработки, связанными с промисами. Кроме того, в следующей главе вы познакомитесь с тестированием своего приложения. Тогда вам больше не нужно беспокоиться о браузере или окружении консольного браузера.

Я хочу представить еще одно улучшение запроса к Hacker News в компоненте App. Представьте, что ваш компонент монтируется, когда веб-страница отображается в браузере в первый раз. В `componentDidMount()` компонент делает запрос, но затем, поскольку ваше приложение ввело какую-то навигацию, вы переходите от этой страницы к другой странице. Ваш компонент App демонтируется, но все ещё остается ожидающий запрос из вашего жизненного цикла `componentDidMount()`. Он попытается использовать `this.setState()` в конечном итоге в блоке промиса `then()` или `catch()`. Возможно, в первый раз вы увидите следующее предупреждение в командной строке или в выводе консоли разработчика браузера:

{title="Command Line",lang="text"}
~~~~~~~~
Warning: Can only update a mounted or mounting component. This usually means you called setState, replaceState, or forceUpdate on an unmounted component. This is a no-op.
~~~~~~~~

Вы можете решить эту проблему, прервав запрос, когда ваш компонент демонтируется или предотвращает вызов `this.setState()` в демонтированном компоненте. Это передовая практика в React, хотя ей следуют далеко не все разработчики, чтобы оставить чистое приложение без каких-либо раздражающих предупреждений. Однако в текущем API промисов не реализует прерывание запроса. Таким образом, вам нужно самому справиться с этой проблемой. Возможно также, это та причина, по которой не многие разработчики следуют этой переводой практике. Следующая реализация выглядит скорее как обходное решение, чем поддерживаемая реализация. Исходя из этого вы можете самостоятельно решить, хотите ли вы его реализовать, чтобы обойти предупреждение из-за размонтированного компонента. Тем не менее, помните об этом предупреждении, если оно появится в следующей главе этой книги или в вашем собственном приложении. Тогда в подобных случаях вы теперь знаете как справиться с этим.

Давайте начнём обойти это. Вы можете ввести поле класса, которое содержит состояние жизненного цикла вашего компонента. Он может быть инициализирован как `false`, когда компонент инициализируется, изменен на `true`, когда компонент установлен, а затем снова устанавливается на `false`, когда компонент размонтирован. Таким образом, вы можете отслеживать состояние жизненного цикла вашего компонента. Он не имеет ничего общего с локальным состоянием, которое хранится и модифицируется с помощью `this.state` и `this.setState()`, поскольку у вас должен быть  к нему доступ непосредственно на экземпляре компонента, не полагаясь на управлене локальным состоянием React. Более того, это не приводит к повторной отрисовки компонента, когда поле класса изменяется таким образом.

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

Наконец, вы можете использовать эти знания, чтобы не прервать сам запрос, а избежать вызов `this.setState()` на вашем экземпляре компонента, даже если компонент уже размонтирован. Это предотвратит упомянутое предупреждение.

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

В целом в главе показано, как вы можете заменить одну библиотеку другой библиотекой в React. Если у вас возникнут какие-либо проблемы, вы можете использовать обширную экосистему библиотеку в JavaScript, чтобы помочь самому себе. Кроме того, вы видели способ, как можно избежать вызова `this.setState()` в React на размонтированном компоненте. Если вы получше изучите библиотеку axios, вы также найдёте способ отменить запрос в первую очередь тоже. Вам предстоит больше узнать об этой теме.

### Exercises:

* read more about [why frameworks matter](https://www.robinwieruch.de/why-frameworks-matter/)
* learn more about [an alternative React component syntax](https://github.com/the-road-to-learn-react/react-alternative-class-component-syntax)

{pagebreak}

You have learned to interact with an API in React! Let's recap the last chapters:

* React
  * ES6 class component lifecycle methods for different use cases
  * componentDidMount() for API interactions
  * conditional renderings
  * synthetic events on forms
  * error handling
  * aborting a remote API request
* ES6 and beyond
  * template strings to compose strings
  * spread operator for immutable data structures
  * computed property names
  * class fields
* General
  * Hacker News API interaction
  * native fetch browser API
  * client- and server-side search
  * pagination of data
  * client-side caching
  * axios as an alternative for the native fetch API

Again it makes sense to take a break. Internalize the learnings and apply them on your own. You can experiment with the source code you have written so far. You can find the source code in the [official repository](https://github.com/the-road-to-learn-react/hackernews-client/tree/5.3.1).

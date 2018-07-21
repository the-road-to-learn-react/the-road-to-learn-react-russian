# Управление состоянием в React и за его пределами

Вы уже изучили основы управления состоянием в React в предыдущих главах. В данной главе мы немного углубимся в эту тему. Вы узнаете о передовых практиках, как их применять, а также как использовать стороннюю библиотеку для управления состоянием.

## Подъём состояния

В вашем приложении только компонент App — классовый компонент с состоянием. Он обрабатывает все состояния в приложении и содержит много логики в своих методах. Возможно, вы заметили, что передаёте много свойств компоненту Table. Большинство из них используются только в компоненте Table. В заключение можно утверждать, что нет никакого смысла в том, что компонент App знает обо всём этом.

Вся функциональность сортировки используется только в компоненте Table. Вы можете перенести её в компонент Table, потому что компонент App вообще не должен знать об этом. Процесс рефакторинга подсостояния от одного компонента к другому известен как *подъём состояния*. В вашем случае вам нужно переместить состояние, неиспользуемое в компоненте App, в компонент Table. Состояние перемещается от родительского к дочернему компоненту.

Для работы с состоянием и методами класса в компоненте Table, его нужно преобразовать в компонент ES6-класса. Рефакторинг из функционального компонента без состояния в компонент класса ES6 — это просто.

Компонент Table как функциональный компонент без состояния выглядит примерно так:

{title="src/App.js",lang=javascript}
~~~~~~~~
const Table = ({
  list,
  sortKey,
  isSortReverse,
  onSort,
  onDismiss
}) => {
  const sortedList = SORTS[sortKey](list);
  const reverseSortedList = isSortReverse
    ? sortedList.reverse()
    : sortedList;

  return(
    ...
  );
}
~~~~~~~~

Компонент Table в виде класса ES6 компонента будет таким:

{title="src/Table.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
class Table extends Component {
  render() {
    const {
      list,
      sortKey,
      isSortReverse,
      onSort,
      onDismiss
    } = this.props;

    const sortedList = SORTS[sortKey](list);
    const reverseSortedList = isSortReverse
      ? sortedList.reverse()
      : sortedList;

    return (
      ...
    );
  }
}
# leanpub-end-insert
~~~~~~~~

Поскольку вам нужно работать с состоянием и методами в своём компоненте, вам нужно добавить конструктор и начальное состояние.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Table extends Component {
# leanpub-start-insert
  constructor(props) {
    super(props);

    this.state = {};
  }
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

Теперь вы можете переместить состояния и методы класса, относящиеся к функциональности сортировки из вашего компонента App в компонент Table.

{title="src/Table.js",lang=javascript}
~~~~~~~~
class Table extends Component {
  constructor(props) {
    super(props);

# leanpub-start-insert
    this.state = {
      sortKey: 'NONE',
      isSortReverse: false,
    };

    this.onSort = this.onSort.bind(this);
# leanpub-end-insert
  }

# leanpub-start-insert
  onSort(sortKey) {
    const isSortReverse = this.state.sortKey === sortKey && !this.state.isSortReverse;
    this.setState({ sortKey, isSortReverse });
  }
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

Не забудьте удалить перемещённое состояние и метод `onSort()` из компонента App.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
  _isMounted = false;

  constructor(props) {
    super(props);

    this.state = {
      results: null,
      searchKey: '',
      searchTerm: DEFAULT_QUERY,
      error: null,
      isLoading: false,
    };

    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
    this.onSearchChange = this.onSearchChange.bind(this);
    this.needsToSearchTopStories = this.needsToSearchTopStories.bind(this);
  }

  ...

}
~~~~~~~~

Кроме того, вы можете сделать API компонента Table более лёгким. Удалите свойства, которые передаются ему из компонента App, потому что теперь они обрабатываются внутри компонента Table.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
# leanpub-start-insert
    const {
      searchTerm,
      results,
      searchKey,
      error,
      isLoading
    } = this.state;
# leanpub-end-insert

    ...

    return (
      <div className="page">
        ...
        { error
          ? <div className="interactions">
            <p>Что-то пошло не так.</p>
          </div>
          : <Table
# leanpub-start-insert
            list={list}
            onDismiss={this.onDismiss}
# leanpub-end-insert
          />
        }
        ...
      </div>
    );
  }
}
~~~~~~~~

Теперь в компоненте Table вы можете использовать внутренний метод `onSort()` и внутреннее состояние Table.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Table extends Component {

  // ...

  render() {
# leanpub-start-insert
    const {
      list,
      onDismiss
    } = this.props;

    const {
      sortKey,
      isSortReverse,
    } = this.state;
# leanpub-end-insert

    const sortedList = SORTS[sortKey](list);
    const reverseSortedList = isSortReverse
      ? sortedList.reverse()
      : sortedList;

    return(
      <div className="table">
        <div className="table-header">
          <span style={{ width: '40%' }}>
            <Sort
              sortKey={'TITLE'}
# leanpub-start-insert
              onSort={this.onSort}
# leanpub-end-insert
              activeSortKey={sortKey}
            >
              Заголовок
            </Sort>
          </span>
          <span style={{ width: '30%' }}>
            <Sort
              sortKey={'AUTHOR'}
# leanpub-start-insert
              onSort={this.onSort}
# leanpub-end-insert
              activeSortKey={sortKey}
            >
              Автор
            </Sort>
          </span>
          <span style={{ width: '10%' }}>
            <Sort
              sortKey={'COMMENTS'}
# leanpub-start-insert
              onSort={this.onSort}
# leanpub-end-insert
              activeSortKey={sortKey}
            >
              Комментарии
            </Sort>
          </span>
          <span style={{ width: '10%' }}>
            <Sort
              sortKey={'POINTS'}
# leanpub-start-insert
              onSort={this.onSort}
# leanpub-end-insert
              activeSortKey={sortKey}
            >
              Очки
            </Sort>
          </span>
          <span style={{ width: '10%' }}>
            Архив
          </span>
        </div>
        { reverseSortedList.map((item) =>
          ...
        )}
      </div>
    );
  }
}
~~~~~~~~

Ваше приложение по-прежнему должно работать, хотя вы сделали большой рефакторинг. Вы переместили функциональность и состояние ближе к другому компоненту. Другие компоненты снова стали более лёгкими. Кроме того, API компонента стал более лёгким, поскольку он внутренне выполняет функции сортировки.

Процесс подъёма состояния может идти по обратному пути: от дочернего к родительскому компоненту. Это называется _подъём состояния вверх_. Представьте, что вы имеете дело с внутренним состоянием в дочернем компоненте. Теперь вы хотите удовлетворить требование по отображению состояния в своём родительском компоненте. Вам нужно будет поднять состояние к вашему родительскому компоненту. Но это идёт ещё дальше. Представьте, что вы хотите показать состояние в одноуровневом компоненте вашего дочернего компонента. Снова вам нужно будет поднять состояние до вашего родительского компонента. Родительский компонент имеет дело с внутренним состоянием, но предоставляет его для обоих дочерних компонентов.

### Упражнения:

* прочитайте подробнее про [поднятие состояния в React](https://ru.react.js.org/docs/lifting-state-up.html)
* прочитайте больше про поднятие состояния в [статье про изучение React перед использованием Redux](https://www.robinwieruch.de/learn-react-before-using-redux/)

## Пересмотр: setState()

До сих пор вы использовали React `setState()` для управления состоянием вашего внутреннего компонента. Вы можете передать объект функции, в которой вы можете частично обновить внутреннее состояние.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState({ foo: bar });
~~~~~~~~

Но `setState()` принимает не только объект. Второй вариант использования этого метода включает передачу функции для обновления состояния.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState((prevState, props) => {
  ...
});
~~~~~~~~

Когда это может пригодиться? Существует один важный случай использования, когда имеет смысл передать функцию вместо объекта. Это когда вы обновляете состояние в зависимости от предыдущего состояния или свойства. Если вы не используете функцию, управление внутренним состоянием может вызвать баги (ошибки).

Но почему это вызывает баги при использовании объекта вместо функции, когда обновление зависит от предыдущего состояния или свойства? Метод React `setState()` — асинхронный. React группирует вызовы `setState()` и выполняет их рано или поздно. Может случиться так, что предыдущее состояние или свойство изменились между тем, когда вы будете использовать его в вызове `setState()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const { fooCount } = this.state;
const { barCount } = this.props;
this.setState({ count: fooCount + barCount });
~~~~~~~~

Представьте, что `fooCount` и `barCount`, таким образом, состояние или свойство, изменяются где-то ещё асинхронно, когда вы вызываете `setState()`. В растущем приложении у вас будет более одного вызова `setState()` в приложении. Поскольку `setState()` выполняется асинхронно, вы можете использовать его в примере на устаревших значениях.

С помощью функционального подхода функция в `setState()` — колбэк, который работает с состоянием и свойствами во время выполнения колбэк-функции. Даже при том, что `setState()` является асинхронным, с функцией он принимает состояние и свойства в момент его выполнения.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState((prevState, props) => {
  const { fooCount } = prevState;
  const { barCount } = props;
  return { count: fooCount + barCount };
});
~~~~~~~~

Теперь вернёмся к вашему коду, чтобы исправить это поведение. Вместе мы исправим его для одного места, где используется `setState()` и полагается на состояние или свойства. Впоследствии вы сможете исправить это и в других местах.

Метод `setSearchTopStories()` опирается на предыдущее состояние и, следовательно, является прекрасным примером использования функции вместо объекта в `setState()`. Сейчас так выглядит следующий фрагмент кода.

{title="src/App.js",lang=javascript}
~~~~~~~~
setSearchTopStories(result) {
  const { hits, page } = result;
  const { searchKey, results } = this.state;

  const oldHits = results && results[searchKey]
    ? results[searchKey].hits
    : [];

  const updatedHits = [
    ...oldHits,
    ...hits
  ];

  this.setState({
    results: {
      ...results,
      [searchKey]: { hits: updatedHits, page }
    },
    isLoading: false
  });
}
~~~~~~~~

Вы извлекаете значения из состояния, но обновляете состояние в зависимости от предыдущего состояния асинхронно. Теперь вы можете использовать функциональный подход для предотвращения ошибок из-за устаревшего состояния.

{title="src/App.js",lang=javascript}
~~~~~~~~
setSearchTopStories(result) {
  const { hits, page } = result;

# leanpub-start-insert
  this.setState(prevState => {
    ...
  });
# leanpub-end-insert
}
~~~~~~~~

Вы можете переместить весь блок, который вы уже внедрили в эту функцию. Вам нужно только изменить, что вы работаете уже с `prevState`, а не с `this.state`.

{title="src/App.js",lang=javascript}
~~~~~~~~
setSearchTopStories(result) {
  const { hits, page } = result;

  this.setState(prevState => {
# leanpub-start-insert
    const { searchKey, results } = prevState;

    const oldHits = results && results[searchKey]
      ? results[searchKey].hits
      : [];

    const updatedHits = [
      ...oldHits,
      ...hits
    ];

    return {
      results: {
        ...results,
        [searchKey]: { hits: updatedHits, page }
      },
      isLoading: false
    };
# leanpub-end-insert
  });
}
~~~~~~~~

Это исправит проблему с устаревшим состоянием. Однако есть ещё одно улучшение. Поскольку это функция, её можно извлечь для улучшения читабельности. Это ещё одно преимущество использования функции перед объектом — функция может работать вне компонента. Но вам нужно использовать функцию высшего порядка, чтобы передать результат. В конце концов, вы хотите обновить состояние на основе полученного результата из API.

{title="src/App.js",lang=javascript}
~~~~~~~~
setSearchTopStories(result) {
  const { hits, page } = result;
  this.setState(updateSearchTopStoriesState(hits, page));
}
~~~~~~~~

Функция `updateSearchTopStoriesState()` должна возвращать функцию. Это функция высшего порядка. Вы можете определить эту функцию высшего порядка вне вашего компонента App. Обратите внимание на то, как теперь меняется объявление функции.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const updateSearchTopStoriesState = (hits, page) => (prevState) => {
  const { searchKey, results } = prevState;

  const oldHits = results && results[searchKey]
    ? results[searchKey].hits
    : [];

  const updatedHits = [
    ...oldHits,
    ...hits
  ];

  return {
    results: {
      ...results,
      [searchKey]: { hits: updatedHits, page }
    },
    isLoading: false
  };
};
# leanpub-end-insert

class App extends Component {
  ...
}
~~~~~~~~

Вот и всё. Использование функции вместо объекта в `setState()` исправляет потенциальные ошибки, но повышает читаемость и поддержку вашего кода. Кроме того, код становится тестируемым вне компонента приложения. Вы можете экспортировать его и написать тест в виде упражнения.

### Упражнение:

* прочитать больше о том, как [в React правильно использовать состояние](https://reactjs.org/docs/state-and-lifecycle.html#using-state-correctly)
* экспортировать updateSearchTopStoriesState из файла
  * напишите тест для него, который передаёт данные (истории, страницы) и составленное предыдущее состояние и, наконец, ожидает новое состояние
* улучшите методы `setState()` для использования функции
  * но только тогда, когда это имеет смысл, т.е. тогад, когда есть зависимость от свойства или состояния
* снова запустите свои тесты и убедитесь, что всё в порядке

## Укрощение состояния

Предыдущие главы показали, что управление состоянием может быть важной темой в более крупных приложениях. В целом, не только React, но и много фреймворков SPA противостоят этому. За последние годы приложения стали более сложными. Одна из серьёзных проблем в веб-приложениях в настоящее время — укрощение и контроль состояния.

По сравнению с другими решениями, React уже сделал большой шаг вперёд. Однонаправленный поток данных и простой API для управления состоянием в компоненте являются незаменимыми. Эти концепции упрощают управление состоянием и изменениями в вашем состоянии на уровне компонентов и в определенной степени на уровне приложения.

В условиях роста приложения становится всё труднее размышлять об изменениях состояния. Вы можете допустить баги, работая с устаревшим состоянием при использовании объекта вместо функции в `setState()`. Вы должны поднять состояние, чтобы поделиться необходимым или скрыть ненужное состояние между компонентами. Может случиться так, что компонент должен поднять состояние, потому что его родственный (одноуровневый) компонент зависит от него. Возможно, компонент находится далеко в дереве компонентов и, следовательно, вам нужно разделить это состояние по всему дереву компонентов. И наконец, компоненты в большей степени задействованы в управлении состоянием. Но ведь основная ответственность компонентов должна представлять пользовательский интерфейс, не так ли?

Из-за всех этих причин существуют независимые решения по управлению состоянием и его сохранению. Эти решения используются не только в React. Однако это то, что делает экосистему React таким мощным местом. Вы можете использовать различные решения проблем, связанных с состоянием. Чтобы решить проблему масштабирования управления состоянием, вы, возможно, слышали о библиотеках [Redux](http://redux.js.org/docs/introduction/) или [MobX](https://mobx.js.org/ ). Вы можете использовать любое из этих решений в приложении React. Они включают расширения, такие как [react-redux](https://github.com/reactjs/react-redux) и [mobx-react](https://github.com/mobxjs/mobx-react), чтобы интегрировать их в слой представления React.

Redux и MobX выходят за рамки данной книги. Когда вы закончите книгу, вы получите руководство о том, как вы можете продолжать изучать React и его экосистему. Одним из путей обучения может стать изучение Redux. Прежде чем вы погрузитесь в тему внешнего управления состоянием, я могу порекомендовать прочитать [эту статью][https://www.robinwieruch.de/redux-mobx-confusion/). Она призвана дать вам лучшее понимание того, как изучать внешнее управление состоянием.

### Упражнения:

* узнайте больше о [внешнем управлении состоянием и о том, как его изучать](https://www.robinwieruch.de/redux-mobx-confusion/)
* ознакомьтесь с моей второй книгой про [управление состоянием в React](https://roadtoreact.com/)

{pagebreak}

Вы изучили некоторые продвинутые техники управления состоянием в React! Давайте повторим последние темы:

* React
  * поднятие состояние вверх и вниз до нужных компонентов
  * `setState()` может использовать функцию в качестве аргумента для предотвращения ошибок, связанных с устаревшим состоянием
  * существующие сторонние решения, которые помогут вам приручить состояние

Исходный код можно найти в [официальном репозитории](https://github.com/the-road-to-learn-react/hackernews-client/tree/5.6).

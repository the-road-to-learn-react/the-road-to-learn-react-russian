# Продвинутые React-компоненты

В этой главе основное внимание будет уделено реализации продвинутых компонентов React. Вы узнаете о компонентах высшего порядка (higher-order component) и о том, как их реализовать. Кроме того, вы погрузитесь в более сложные темы в React и реализуете сложные взаимодействия с ним.

## Ссылка на DOM-элемент

Иногда вам нужно взаимодействовать с вашими DOM-узлами в React. Атрибут `ref` дает вам доступ к узлу в ваших элементах. Обычно это антишаблон в React, потому что вам следует использовать декларативный способ работы и однонаправленный поток данных. Вы узнали об этом, когда добавили своё первое поле ввода для поиска. Но есть определённые случаи, когда вам нужен доступ к узлу DOM. В официальной документации упоминаются три варианта использования:

* для использования API DOM (фокус, воспроизведение мультимедиа и т.д.)
* для вызова императивных анимаций DOM-узлов
* для интеграции со сторонней библиотекой, которой нужен DOM-узел (например, [D3.js](https://d3js.org/))

Давайте сделаем это на примере с помощью компонента Search. Когда приложение отрисовывается в первый раз, поле ввода должно быть сфокусировано. Это один из вариантов, когда вам нужен доступ к API DOM. В этой главе вы узнаете, как это работает, но поскольку это не очень полезно для самого приложения, мы опустим изменения после главы. Однако вы можете сохранить его для своего приложения.

В общем, вы можете использовать атрибут `ref` как в функциональных компонентах без состояния, так и в ES6-классах компонентов. В варианте использования примера с фокусом вам понадобится метод жизненного цикла. Вот почему подход сначала демонстрируется с использованием атрибута `ref` с классовым компонентом.

Первоначальный шаг — миграция с функционального компонента без состояния на компонент класса ES6.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
class Search extends Component {
  render() {
    const {
      value,
      onChange,
      onSubmit,
      children
    } = this.props;

    return (
# leanpub-end-insert
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
# leanpub-start-insert
    );
  }
}
# leanpub-end-insert
~~~~~~~~

Объект `this` ES6-класса компонента помогает нам ссылаться на DOM-узел с атрибутом `ref`.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Search extends Component {
  render() {
    const {
      value,
      onChange,
      onSubmit,
      children
    } = this.props;

    return (
      <form onSubmit={onSubmit}>
        <input
          type="text"
          value={value}
          onChange={onChange}
# leanpub-start-insert
          ref={(node) => { this.input = node; }}
# leanpub-end-insert
        />
        <button type="submit">
          {children}
        </button>
      </form>
    );
  }
}
~~~~~~~~

Теперь вы можете сфокусировать поле ввода, когда компонент монтируется с использованием объекта `this`, соответствующего метода жизненного цикла и API DOM.

{title="src/App.js",lang=javascript}
~~~~~~~~
class Search extends Component {
# leanpub-start-insert
  componentDidMount() {
    if (this.input) {
      this.input.focus();
    }
  }
# leanpub-end-insert

  render() {
    const {
      value,
      onChange,
      onSubmit,
      children
    } = this.props;

    return (
      <form onSubmit={onSubmit}>
        <input
          type="text"
          value={value}
          onChange={onChange}
          ref={(node) => { this.input = node; }}
        />
        <button type="submit">
          {children}
        </button>
      </form>
    );
  }
}
~~~~~~~~

Поле ввода должно быть сфокусировано при отрисовки приложения. Это в основном это для использования атрибута `ref`.

Но как бы вы получили доступ к `ref` в функциональном компоненте, не имеющем состояния, без использования объекта `this`? Следующий пример демонстрирует такой случай.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Search = ({
  value,
  onChange,
  onSubmit,
  children
}) => {
# leanpub-start-insert
  let input;
# leanpub-end-insert
  return (
    <form onSubmit={onSubmit}>
      <input
        type="text"
        value={value}
        onChange={onChange}
# leanpub-start-insert
        ref={(node) => input = node}
# leanpub-end-insert
      />
      <button type="submit">
        {children}
      </button>
    </form>
  );
}
~~~~~~~~

Теперь вы сможете получить доступ к элементу ввода DOM. В примере использования c фокусировкой поле ввода это не поможет вам, потому что у вас нет метода жизненного цикла в функциональном компоненте без состояния, чтобы получить фокус на элементе. Но в будущем вы можете столкнуться с другими вариантами использования, когда имеет смысл использовать функциональный компонент без состояния с атрибутом `ref`.

### Упражнения

* узнать подробнее про [использование атрибута `ref` в React](https://www.robinwieruch.de/react-ref-attribute-dom-node/)
* узнать подробнее в целом про [атрибут `ref` в React](https://reactjs.org/docs/refs-and-the-dom.html)

## Загрузка ...

Теперь вернемся к приложению. Возможно, вам понадобиться показать индикатор загрузки при отправке поискового запроса к API Hacker News. Поскольку запрос является асинхронным, вы, возможно, захотите дать своему пользователю обратную связь, что сейчас коё-что произойдет. Давайте определим повторно используемый компонент Loading в файле *src/App.js*.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Loading = () =>
  <div>Загрузка ...</div>
~~~~~~~~

Теперь вам нужно свойство для хранения состояния загрузки. В зависимости от значения состояния загрузки вы можете позднее отобразить компонент Loading.

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
# leanpub-start-insert
      isLoading: false,
# leanpub-end-insert
    };

    ...
  }

  ...

}
~~~~~~~~

Начальное значение такого свойства `isLoading` устанавливается в значение `false`. Вы не загружаете ещё что-либо перед тем, как компонент App смонтирован.

Когда вы делаете запрос, то устанавливаете состояние загрузки в значение `true`. В конце концов, когда запрос будет успешным, вы можете установить состояние загрузки обратно на значение `false`.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  setSearchTopStories(result) {
    ...

    this.setState({
      results: {
        ...results,
        [searchKey]: { hits: updatedHits, page }
      },
# leanpub-start-insert
      isLoading: false
# leanpub-end-insert
    });
  }

  fetchSearchTopStories(searchTerm, page = 0) {
# leanpub-start-insert
    this.setState({ isLoading: true });
# leanpub-end-insert

    axios(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
      .then(result => this._isMounted && this.setSearchTopStories(result.data))
      .catch(error => this._isMounted && this.setState({ error }));
  }

  ...

}
~~~~~~~~

На последнем шаге вы будете использовать компонент Loading в App. Условная отрисовка на основании значения состоянии загрузки будет определять, отображать ли компонент Loading или компонент Button. Последний компонент — это кнопка для получения дополнительных (больше) данных.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const {
      searchTerm,
      results,
      searchKey,
      error,
# leanpub-start-insert
      isLoading
# leanpub-end-insert
    } = this.state;

    ...

    return (
      <div className="page">
        ...
        <div className="interactions">
# leanpub-start-insert
          { isLoading
            ? <Loading />
            : <Button
              onClick={() => this.fetchSearchTopStories(searchKey, page + 1)}>
              More
            </Button>
          }
# leanpub-end-insert
        </div>
      </div>
    );
  }
}
~~~~~~~~

Первоначально компонент Loading будет отображаться при запуске приложения, потому что вы совершаете запрос в `componentDidMount()`. Компонент Table отсутствует, потому что список пуст. Когда ответ приходит от API Hacker News, показывается результат, состояние загрузки устанавливается на значение `false`, а компонент Loading исчезает. Вместо этого отображается кнопка «Больше историй» для получения ещё одной порции данных. После получения дополнительных данных кнопка получения снова исчезнет и вместо неё отобразиться компонент Loading.

### Упражнения:

* использовать библиотеку, такую как [Font Awesome](https://fontawesome.io/), чтобы показать иконку загрузки вместо надписи "Загрузка ..."

## Компоненты высшего порядка

Компоненты высшего порядка (Higher-order components, HOC) — продвинутая концепция React. Компоненты высшего порядка эквивалентны функциям высшего порядка. Они принимают любые входные данные — чаще всего компонент, но ещё необязательные аргументы — и возвращают компонент в качестве возвращаемого значения. Возвращаемый компонент является расширенной версией переданного в HOC компонента и может использоваться в JSX.

Компоненты высшего порядка используются в разных случаях. Они могут подготавливать свойства, управлять состоянием или изменять представление компонента. Одним из вариантов использования может быть использование HOC в качестве помощника для условной отрисовки. Представьте, что у вас есть компонент List, отображающий список элементов или вообще ничего, потому что список пуст или равняется `null`. С использованием HOC можно ничего не отрисовывать, если нет списка. С другой стороны, компоненту простого List больше не нужно беспокоиться о несуществующем списке. Его задача только отрисовать список.

Давайте создадим простой HOC, который принимает компонент и возвращает компонент. Вы можете поместить его в файл *src/App.js*.

{title="src/App.js",lang=javascript}
~~~~~~~~
function withFoo(Component) {
  return function(props) {
    return <Component { ...props } />;
  }
}
~~~~~~~~

Есть одно чистое соглашение — префикс имени HOC начинается с `with`. Поскольку вы используете JavaScript ES6, вы можете более кратко сделать HOC с помощью стрелочных функций из ES6.

{title="src/App.js",lang=javascript}
~~~~~~~~
const withFoo = (Component) => (props) =>
  <Component { ...props } />
~~~~~~~~

В этом примере входной компонент будет оставаться таким же, как и выходной компонент. Ничего интересного не происходит. Он отрисовывает один и тот же экземпляр компонента и передаёт все свойства возвращаемому компоненту. Но это бесполезно. Давайте улучшим получаемый из HOC компонент. Компонент на выходе должен показывать компонент Loading, когда значение состояния загрузки равняется `true`, в противном случае он должен отображать переданный компонент. Отрисовка по условию — отличный вариант использования HOC.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const withLoading = (Component) => (props) =>
  props.isLoading
    ? <Loading />
    : <Component { ...props } />
# leanpub-end-insert
~~~~~~~~

Судя по значению свойства загрузки вы можете применить условную отрисовку. Функция вернет компонент Loading или входной компонент.

В общем случае может быть очень эффективным расширение объекта, например, объектом `props` из предыдущего примера, на входном компоненте. Смотрите разницу в следующем фрагменте кода.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// деструктуризация свойств перед их передачей
const { foo, bar } = props;
<SomeComponent foo={foo} bar={bar} />

// но можно использовать оператор расширения для передачи всех свойств объекта
<SomeComponent { ...props } />
~~~~~~~~

Есть одна маленькая деталь, которую следует избегать: сейчас вы передаёте все свойства, включая `isLoading`, путём расширения объекта во входной компонент. Однако входной компонент может не поддерживать свойство `isLoading`, тогда вы можете использовать деструктуризацию оставшихся свойств из ES6 для решения этой проблемы.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const withLoading = (Component) => ({ isLoading, ...rest }) =>
  isLoading
    ? <Loading />
    : <Component { ...rest } />
# leanpub-end-insert
~~~~~~~~

Он выбирает одно свойство из объекта, но сохраняет оставшиеся свойства в объекте. Он также поддерживает работу с несколькими свойствами. Возможно, вы уже прочитали об этом в [деструктурирующем присваивании](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).

Теперь вы можете использовать HOC в своём JSX. В приложении может показываться либо кнопка «Больше историй», либо компонент Loading. Компонент Loading уже инкапсулирован в HOC, но отсутствует передаваемый компонент. В случае показа компонента Button или компонента Loading, компонент Button будет входным компонентом для HOC. Улучшенный выходной компонент — компонент ButtonWithLoading.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Button = ({
  onClick,
  className = '',
  children,
}) =>
  <button
    onClick={onClick}
    className={className}
    type="button"
  >
    {children}
  </button>

const Loading = () =>
  <div>Загрузка ...</div>

const withLoading = (Component) => ({ isLoading, ...rest }) =>
  isLoading
    ? <Loading />
    : <Component { ...rest } />

# leanpub-start-insert
const ButtonWithLoading = withLoading(Button);
# leanpub-end-insert
~~~~~~~~

Теперь всё установлено. В качестве последнего шага вам необходимо использовать компонент ButtonWithLoading, который получает состояние загрузки в виде дополнительного свойства. В то время как HOC использует свойство загрузки, все другие свойства передаются компоненту Button.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    ...

    return (
      <div className="page">
        ...
        <div className="interactions">
# leanpub-start-insert
          <ButtonWithLoading
            isLoading={isLoading}
            onClick={() => this.fetchSearchTopStories(searchKey, page + 1)}
          >
            Больше историй
          </ButtonWithLoading>
# leanpub-end-insert
        </div>
      </div>
    );
  }
}
~~~~~~~~

Когда вы снова запустите свои тесты, вы заметите, что ваш тест снимком компонента App завершился неудачей. В командной строке различия могут выглядеть с помощью унифицированного формата diff следующим образом:

{title="Command Line",lang="text"}
~~~~~~~~
-    <button
-      className=""
-      onClick={[Function]}
-      type="button"
-    >
-      Больше историй
-    </button>
+    <div>
+      Загрузка ...
+    </div>
~~~~~~~~

Вы можете либо исправить компонент сейчас, когда вам кажется, что в нём что-то не так, либо можете принять новый снимок. Поскольку вы реализовали компонент Loading в этой главе, вы можете принять изменённый снимок в командной строке в интерактивном тесте.

Компоненты высшего порядка — передовая методика в React. У неё есть несколько целей, таких как улучшенная возможность повторного использования компонентов, более сильная абстракция (???), композиция (компоновка) компонентов и управлениями свойствами, состоянием и представлением. Не переживайте, если не сразу понимаете всё это. Потребуется время, чтобы привыкнуть к этому всему.

Я призываю вас прочитать [простое введение в компоненты высшего порядка](https://www.robinwieruch.de/gentle-introduction-higher-order-components/). Это даст вам ещё один подход для их изучения, показывает элегантный способ использования их в функциональном программировании и, в частности, решает проблему условной отрисовки с компонентами высшего порядка.

### Упражнения:

* прочитайте [лёгкое введение в компоненты высшего порядка](https://www.robinwieruch.de/gentle-introduction-higher-order-components/)
* поэкспериментируйте с HOC, который вы создали
* подумайте о случае использования, где другой HOC будет иметь смысл
  * реализуйте такой HOC, если есть в этом необходимость

## Продвинутая сортировка

Вы уже реализовали поиск на стороне клиента и сервера. Поскольку у вас есть компонент Table, имеет смысл улучшить таблицу с расширенными возможностями. Как насчет внедрения функциональности сортировки по каждому столбцу при нажатии на заголовки столбцов компонента Table?

Можно было бы написать свою собственную функцию сортировки, но лично я предпочитаю использовать вспомогательную библиотеку для как раз таких случаев. [Lodash](https://lodash.com/) — одна из этих утилитарных библиотек, но вы можете использовать любую другую библиотеку, которая вам подходит. Давайте установим Lodash и воспользуемся её функциональностью сортировки.

{title="Command Line",lang="text"}
~~~~~~~~
npm install lodash
~~~~~~~~

Теперь вы можете импортировать Lodash в файле *src/App.js* .

{title="src/App.js",lang=javascript}
~~~~~~~~
import React, { Component } from 'react';
import axios from 'axios';
# leanpub-start-insert
import { sortBy } from 'lodash';
# leanpub-end-insert
import './App.css';
~~~~~~~~

У вас есть несколько столбцов в компоненте Table. Есть столбцы для заголовков, авторов, комментариев и очков. Вы можете определить функции сортировки, где каждая функция принимает список и возвращает список элементов, отсортированных по определённому свойству. Кроме того, вам понадобится одна функция сортировки по умолчанию, которая не сортирует, а возвращает обычный (неотсортированный) список. Это будет вашим первоначальным состоянием.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

# leanpub-start-insert
const SORTS = {
  NONE: list => list,
  TITLE: list => sortBy(list, 'title'),
  AUTHOR: list => sortBy(list, 'author'),
  COMMENTS: list => sortBy(list, 'num_comments').reverse(),
  POINTS: list => sortBy(list, 'points').reverse(),
};
# leanpub-end-insert

class App extends Component {
  ...
}
...
~~~~~~~~

Как вы можете увидеть сами, две функции сортировки возвращают список в обратном порядке. Это из-за того, что нужно отображение элементов с наиболее большим количеством комментариев и очками, вместо того элементов с наименьшим количеством данных при сортировке списка в первый раз.

Объект `SORTS` позволяет вам ссылаться на любую функцию сортировки.

Снова ваш компонент App отвечает за сохранение состояния сортировки. Первоначальным состоянием будет функция сортировки по умолчанию, которая не сортирует элементы вообще, а возвращает входной список в качестве вывода.

{title="src/App.js",lang=javascript}
~~~~~~~~
this.state = {
  results: null,
  searchKey: '',
  searchTerm: DEFAULT_QUERY,
  error: null,
  isLoading: false,
# leanpub-start-insert
  sortKey: 'NONE',
# leanpub-end-insert
};
~~~~~~~~

После выбора другого ключа сортировка `sortKey`, скажем, `AUTHOR`, список будет отсортирован соответствующей функцией сортировки из объекта `SORTS`.

Теперь вы можете определить новый метод класса в компоненте App, который просто устанавливает `sortKey` в локальном состоянии компонента. После этого `sortKey` можно использовать для получения функции сортировки для её применения в вашем списке.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
  _isMounted = false;

  constructor(props) {

    ...

    this.needsToSearchTopStories = this.needsToSearchTopStories.bind(this);
    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
    this.onSearchChange = this.onSearchChange.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
# leanpub-start-insert
    this.onSort = this.onSort.bind(this);
# leanpub-end-insert
  }

  ...

# leanpub-start-insert
  onSort(sortKey) {
    this.setState({ sortKey });
  }
# leanpub-end-insert

  ...

}
~~~~~~~~

Следующий шаг — передать метод и `sortKey` в ваш компонент Table.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const {
      searchTerm,
      results,
      searchKey,
      error,
      isLoading,
# leanpub-start-insert
      sortKey
# leanpub-end-insert
    } = this.state;

    ...

    return (
      <div className="page">
        ...
        <Table
          list={list}
# leanpub-start-insert
          sortKey={sortKey}
          onSort={this.onSort}
# leanpub-end-insert
          onDismiss={this.onDismiss}
        />
       ...
      </div>
    );
  }
}
~~~~~~~~

Компонент Table отвечает за сортировку списка. Он принимает одну из функций из объекта `SORT` по ключу `sortKey` и передаёт список в качестве входных данных. После этого он выводит отсортированной список.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const Table = ({
  list,
  sortKey,
  onSort,
  onDismiss
}) =>
# leanpub-end-insert
  <div className="table">
# leanpub-start-insert
    {SORTS[sortKey](list).map(item =>
# leanpub-end-insert
      <div key={item.objectID} className="table-row">
        ...
      </div>
    )}
  </div>
~~~~~~~~

Теоретически список будет отсортирован одной из функций. Но сортировка по умолчанию установлена на `NONE`, поэтому элементы не сортируются. До сих пор никто не выполнял метод `onSort()` для изменения `sortKey`. Расширим компонент Table, добавив заголовки столбцов, использующие компоненты Sort в столбцах для сортировки по каждому столбцу.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Table = ({
  list,
  sortKey,
  onSort,
  onDismiss
}) =>
  <div className="table">
# leanpub-start-insert
    <div className="table-header">
      <span style={{ width: '40%' }}>
        <Sort
          sortKey={'TITLE'}
          onSort={onSort}
        >
          Заголовок
        </Sort>
      </span>
      <span style={{ width: '30%' }}>
        <Sort
          sortKey={'AUTHOR'}
          onSort={onSort}
        >
          Автор
        </Sort>
      </span>
      <span style={{ width: '10%' }}>
        <Sort
          sortKey={'COMMENTS'}
          onSort={onSort}
        >
          Комментарии
        </Sort>
      </span>
      <span style={{ width: '10%' }}>
        <Sort
          sortKey={'POINTS'}
          onSort={onSort}
        >
          Очки
        </Sort>
      </span>
      <span style={{ width: '10%' }}>
        Архив
      </span>
    </div>
# leanpub-end-insert
    {SORTS[sortKey](list).map(item =>
      ...
    )}
  </div>
~~~~~~~~

Каждый компонент Sort получает определённую функцию `sortKey` и общую функцию `onSort()`, передавая ей `sortKey` для установки конкретного ключа.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Sort = ({ sortKey, onSort, children }) =>
  <Button onClick={() => onSort(sortKey)}>
    {children}
  </Button>
~~~~~~~~

Как вы можете видеть, компонент Sort переиспользует общий компонент Button. При нажатии на кнопку каждому из компонентов Button передаётся `sortKey`, который будет установлен методом `onSort()`. Теперь есть возможность сортировать список при нажатии на заголовки столбцов.

Можно сделать ещё одно небольшое улучшение внешнего вида. Пока кнопка в заголовке столбца выглядит немного нелепо. Давайте дадим кнопке в компоненте Sort собственный CSS-класс, используя `className`.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Sort = ({ sortKey, onSort, children }) =>
# leanpub-start-insert
  <Button
    onClick={() => onSort(sortKey)}
    className="button-inline"
  >
# leanpub-end-insert
    {children}
  </Button>
~~~~~~~~

Теперь кнопка выглядит красиво. Следующая цель заключалась бы в реализации обратной сортировки. Список должен быть отсортирован в обратном порядке по двойному нажатию на компонент Sort. Во-первых, нужно определить новое свойство для обратной сортировки с булевым значением. Сортировка может быть либо прямой, либо обратной.

{title="src/App.js",lang=javascript}
~~~~~~~~
this.state = {
  results: null,
  searchKey: '',
  searchTerm: DEFAULT_QUERY,
  error: null,
  isLoading: false,
  sortKey: 'NONE',
# leanpub-start-insert
  isSortReverse: false,
# leanpub-end-insert
};
~~~~~~~~

Теперь в вашем методе сортировки вы можете вычислить, отсортирован ли список в обратном порядке. В обратном порядке, если значение состояния `sortKey` совпадает с переданным `sortKey`, а значение состояния обратного сортировки уже не равняется `true`.

{title="src/App.js",lang=javascript}
~~~~~~~~
onSort(sortKey) {
# leanpub-start-insert
  const isSortReverse = this.state.sortKey === sortKey && !this.state.isSortReverse;
  this.setState({ sortKey, isSortReverse });
# leanpub-end-insert
}
~~~~~~~~

Снова вы можете передать свойство, указывающее на обратную сортировку в компонент Table.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const {
      searchTerm,
      results,
      searchKey,
      error,
      isLoading,
      sortKey,
# leanpub-start-insert
      isSortReverse
# leanpub-end-insert
    } = this.state;

    ...

    return (
      <div className="page">
        ...
        <Table
          list={list}
          sortKey={sortKey}
# leanpub-start-insert
          isSortReverse={isSortReverse}
# leanpub-end-insert
          onSort={this.onSort}
          onDismiss={this.onDismiss}
        />
        ...
      </div>
    );
  }
}
~~~~~~~~

Теперь для вычисления данных в компоненте Table необходима стрелочная функция.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
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
# leanpub-end-insert
    <div className="table">
      <div className="table-header">
        ...
      </div>
# leanpub-start-insert
      {reverseSortedList.map(item =>
# leanpub-end-insert
        ...
      )}
    </div>
# leanpub-start-insert
  );
}
# leanpub-end-insert
~~~~~~~~

Сортировка в обратном порядке должна сейчас работать.

И последнее, но не менее важное: вам нужно разобраться с одним открытым вопросом улучшения пользовательского интерфейса. Может ли пользователь определить, по какому столбцу сейчас происходит сортировка?Пока это невозможно. Давайте предоставим пользователю такую визуальную возможность.

Каждый компонент Sort уже получает свой конкретный `sortKey`. Его можно использовать для определения активной в данный момент сортировки. Вы можете передать `sortKey` из внутреннего состояния компонента в качестве активного ключа сортировки в свой компонент Sort.

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
    <div className="table">
      <div className="table-header">
        <span style={{ width: '40%' }}>
          <Sort
            sortKey={'TITLE'}
            onSort={onSort}
# leanpub-start-insert
            activeSortKey={sortKey}
# leanpub-end-insert
          >
            Заголовок
          </Sort>
        </span>
        <span style={{ width: '30%' }}>
          <Sort
            sortKey={'AUTHOR'}
            onSort={onSort}
# leanpub-start-insert
            activeSortKey={sortKey}
# leanpub-end-insert
          >
            Автор
          </Sort>
        </span>
        <span style={{ width: '10%' }}>
          <Sort
            sortKey={'COMMENTS'}
            onSort={onSort}
# leanpub-start-insert
            activeSortKey={sortKey}
# leanpub-end-insert
          >
            Комментарии
          </Sort>
        </span>
        <span style={{ width: '10%' }}>
          <Sort
            sortKey={'POINTS'}
            onSort={onSort}
# leanpub-start-insert
            activeSortKey={sortKey}
# leanpub-end-insert
          >
            Очки
          </Sort>
        </span>
        <span style={{ width: '10%' }}>
          Архив
        </span>
      </div>
      {reverseSortedList.map(item =>
          ...
      )}
    </div>
  );
}
~~~~~~~~

Теперь в компоненте Sort можно узнать, основываясь на `sortKey` и` activeSortKey`, активен ли данный столбец сортировки или нет. Добавьте компоненту Sort ещё дополнительный атрибут `className`, применяемый при сортировке для визуального различия столбцов.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const Sort = ({
  sortKey,
  activeSortKey,
  onSort,
  children
}) => {
  const sortClass = ['button-inline'];

  if (sortKey === activeSortKey) {
    sortClass.push('button-active');
  }

  return (
    <Button
      onClick={() => onSort(sortKey)}
      className={sortClass.join(' ')}
    >
      {children}
    </Button>
  );
}
# leanpub-end-insert
~~~~~~~~

Подобный способ определения CSS-классов, используя константу `sortClass` немного неуклюжий, не считаете так? Существует замечательная небольшая библиотека для элегантной установки CSS-классов. Сначала нужно её установить.

{title="Command Line",lang="text"}
~~~~~~~~
npm install classnames
~~~~~~~~

И теперь нужно импортировать её в верху файла *src/App.js*.

{title="src/App.js",lang=javascript}
~~~~~~~~
import React, { Component } from 'react';
import axios from 'axios';
import { sortBy } from 'lodash';
# leanpub-start-insert
import classNames from 'classnames';
# leanpub-end-insert
import './App.css';
~~~~~~~~

Теперь вы можете воспользоваться ею для определения CSS-классов в зависимости от условий через всё тот же атрибут `className` компонента.

{title="src/App.js",lang=javascript}
~~~~~~~~
const Sort = ({
  sortKey,
  activeSortKey,
  onSort,
  children
}) => {
# leanpub-start-insert
  const sortClass = classNames(
    'button-inline',
    { 'button-active': sortKey === activeSortKey }
  );
# leanpub-end-insert

  return (
# leanpub-start-insert
    <Button
      onClick={() => onSort(sortKey)}
      className={sortClass}
    >
# leanpub-end-insert
      {children}
    </Button>
  );
}
~~~~~~~~

Теперь же, когда вы выполните тесты, то увидите неудачные снимки тестов, а также не прошедшие модульные тесты компонента Table. Поскольку вы изменили внешний вид компонентов, вы можете принять эти снимки. Но вам нужно исправить модульный тест. В вашем файле *src/App.test.js* вам необходимо указать ключи `sortKey` и `isSortReverse` с логическим значением для компонента Table.

{title="src/App.test.js",lang=javascript}
~~~~~~~~
...

describe('Table', () => {

  const props = {
    list: [
      { title: '1', author: '1', num_comments: 1, points: 2, objectID: 'y' },
      { title: '2', author: '2', num_comments: 1, points: 2, objectID: 'z' },
    ],
# leanpub-start-insert
    sortKey: 'TITLE',
    isSortReverse: false,
# leanpub-end-insert
  };

  ...

});
~~~~~~~~

Ещё раз вам может потребоваться принять неудачные снимки компонента Table, потому что были расширены его свойства.

Наконец, работа с расширенной сортировкой завершена.

### Упражнения:

* используйте библиотеку, такую как [Font Awesome](https://fontawesome.io/), чтобы указать на применяемую (обратную) сортировку
  * это может быть иконка стрелки вверх или стрелки вниз рядом с каждым заголовком в компоненте Sort
* узнайте больше про [библиотеку classnames](https://github.com/JedWatson/classnames)

{pagebreak}

Вы изучили продвинутые технологии компонентов React! Давайте повторим последние пройденные темы:

* React
  * атрибут `ref` для ссылок на DOM-узлы
  * Компоненты высшего порядка — это распространённый способ создания продвинутых компонентов
  * внедрение расширенных взаимодействий в React
  * назначение CSS-классов по условию с помощью небольшой вспомогательной библиотеки classNames
* ES6
  * деструктуризация осташихся свойств для разделения объектов и массивов

Исходный код можно найти в [официальном репозитории](https://github.com/the-road-to-learn-react/hackernews-client/tree/5.5).
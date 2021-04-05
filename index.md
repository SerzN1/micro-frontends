Методы, стратегии и рецепты для создания __современного веб-приложения несколькими командами__, которые могут __работать независимо__.

## Что такое Micro Frontends?

Термин __Micro Frontends__ впервые появился в [ThoughtWorks Technology Radar](https://www.thoughtworks.com/radar/techniques/micro-frontends) в конце 2016 года. Эта техника распространяет концепции микросервисов на мир фронтерд разработки. Нынешняя тенденция заключается в создании многофункционального и мощного браузерного приложения, известного как одностраничное приложение (SPA), которое находится на вершине архитектуры микрослужб. Со временем фронтенд слой, часто разрабатываемый отдельной командой, разрастается и его становится все труднее поддерживать. Это то, что мы называем [фронтенд-монолитом](https://www.youtube.com/watch?v=pU1gXA0rfwc).

Идея микро-интерфейсов состоит в том, чтобы думать о веб-сайте или веб-приложении как о композиции функций, принадлежащих независимым командам. Каждая команда отвечает за __отдельную область бизнеса__ или __миссию__, о которой она заботится и на которой специализируется. Команда является кросс-функциональной и развивает свои функции от начала до конца, от базы данных до пользовательского интерфейса.

Однако эта идея не нова. Он имеет много общего с [автономными системами](http://scs-architecture.org/) концепция. В прошлом подобные подходы назывались [Frontend Integration for Verticalized Systems](https://dev.otto.de/2014/07/29/scaling-with-microservices-and-vertical-decomposition), но микро-фронтенды - это явно более дружелюбный и менее громоздкий термин.

__Monolithic Frontends__

<img alt="Monolithic Frontends" src="./ressources/diagrams/organisational/monolith-frontback-microservices.png" loading="lazy" />

__Organisation in Verticals__

<img alt="End-To-End Teams with Micro Frontends" src="./ressources/diagrams/organisational/verticals-headline.png" loading="lazy" />

## Что такое современное веб-приложение?

Во введении я использовал аббревиатуру SPA. Давайте определимся с предположениями, связанными с этим термином.

Чтобы представить это в более широкой перспективе, [Aral Balkan](https://ar.al/) написал сообщение в блоге о том, что он называет континуумом [Documents‐to‐Applications Continuum](https://ar.al/notes/the-documents-to-applications-continuum) он придумывает концепцию скользящей шкалы, где сайт, построенный из статических документов, соединенных ссылками, находится на левом конце шкалы, а чистое поведение управляемое, бесконтентное приложение, подобное онлайн-редактору фотографий, находится на правом краю шкалы.

Если вы хотите разместить свой проект на __левой стороне этого спектра__, то __интеграция на уровне веб-сервера__ хорошый выбор. С помощью этой модели сервер собирает и __объединяет HTML-строки__ из всех компонентов, составляющих запрашиваемую пользователем страницу. Обновления производятся путем перезагрузки страницы с сервера или замены ее частей с помощью ajax. [Gustaf Nilsson Kotte](https://twitter.com/gustaf_nk/) написал [всеобъемлющую статью](https://gustafnk.github.io/microservice-websites/) на эту тему.

Когда ваш пользовательский интерфейс должен обеспечивать __мгновенную обратную связь__, даже при ненадежных соединениях, чистый серверный сайт больше не является достаточным. Для реализации таких методов, как [Optimistic UI](https://www.smashingmagazine.com/2016/11/true-lies-of-optimistic-user-interfaces/) или [Skeleton Screens](http://www.lukew.com/ff/entry.asp?1797) вы также должны иметь возможность __обновить__ свой пользовательский интерфейс __на самом устройстве__. Термин Google [Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/) метко описывает __балансирующий акт__ того, как быть хорошим представителем интернета(progressive enhancement) и одновременно с этим обеспечивать производительность на уровне нативного приложения. Этот вид приложения расположен где-то __примерно в середине сайта-приложения__ на нашей шкале. Здесь уже недостаточно исключительно серверного решения. Мы должны переместить интеграцию __в браузер__, и именно этому посвящена эта статья.

## Основные идеи, лежащие в основе Micro Frontends

* __Будьте технологически независимы__ <br>каждая команда должна иметь возможность выбирать и обновлять свой стек без необходимости координировать свои действия с другими командами. [Custom Elements] (#the-dom-is-the-api) - отличный способ скрыть детали реализации, предоставляя нейтральный интерфейс другим пользователям.
* __Изолируйте командный код__ <br>не используйте общую среду выполнения, даже если все команды используют один и тот же фреймворк. Создавайте независимые приложения, которые являются автономными. Не полагайтесь на общие состояния или глобальные переменные.
* __Установите командные префиксы__ <br>создайте соглашения об именовании там, где изоляция еще невозможна. Пространство имен CSS, события, Local Storage и cookie позволяют избежать коллизий и уточнить права собственности.
* __Отдавайте предпочтение собственным функциям браузера по сравнению с пользовательскими API__ <br>используйте [события браузера для связи] (#parent-child-communication--DOM-modification) вместо создания глобальной системы PubSub. Если вам действительно нужно создать кросс-командный API, постарайтесь сделать его как можно более простым.
* __Создайте отказоустойчивый сайт__ <br>ваша фича должна быть полезна, даже если возникла ошибка JavaScript или библиотека еще не была загружена. Используйте [Universal Rendering] (#serverside-rendering--universal-rendering) и прогрессивное улучшение для улучшения воспринимаемой производительности.

---

## DOM - это API

[Custom Elements](https://developers.google.com/web/fundamentals/getting-started/primers/customelements), аспект интероперабельности из спецификации веб-компонентов, являются хорошим примитивом для интеграции в браузер. Каждая команда создает свой компонент __с использованием своей веб-технологии выбора__ и __оборачивает его внутри пользовательского элемента__ (например, `<order-minicart></order-minicart>`). Спецификация DOM этого конкретного элемента (tag-name, attributes & events) действует как контракт или публичный API для других команд. Преимущество заключается в том, что они могут использовать компонент и его функциональность без необходимости знать реализацию. Они просто должны уметь взаимодействовать с домом.

Но пользовательские элементы сами по себе не являются решением всех наших потребностей. Для решения проблемы прогрессивного улучшения, универсального рендеринга или маршрутизации нам нужны дополнительные части программного обеспечения.

Эта страница разделена на две основные области. Сначала мы обсудим [Композицию-Страницы] (#page-composition) - как собрать страницу из компонентов, принадлежащих разным командам. После этого мы покажем примеры реализации [Постраничного перехода](#page-transition) на клиенте.

## Композиция Страницы

Рядом с __клиент__ и __серверными__ интеграции кода, написанного в __различных структур__ сама по себе, есть много побочных тем, которые должны быть рассмотрены: механизмы __изолировать от JS__, __в CSS избежать конфликтов__, __загрузке ресурсов__ по мере необходимости __общие ресурсы__ между командами, ручка __данные выборки__ и думать о хорошем __загрузке государств__ для пользователей. Мы будем углубляться в эти темы шаг за шагом.

### Базовый Прототип

Страница товара этой модели тракторного магазина послужит основой для следующих примеров.

Он оснащен __элементом выбора__ для переключения между тремя различными моделями тракторов. При изменении имиджа продукта обновляются название, цена и рекомендации. Существует также кнопка __buy__, которая добавляет выбранный вариант в корзину, и __мини-корзина__ вверху, которая соответственно обновляется.

[<img alt="Пример 0 - страница продукта-обычный JS" src="./ressources/video/model-store-0.gif" loading="lazy" />](./0-модель-магазин/)

[попробуйте в браузере](./0-model-store/) & [посмотреть код](https://github.com/serzn1/micro-frontends/tree/master/0-model-store)

Весь HTML генерируется на стороне клиента с использованием __чистого JavaScript__ и строк шаблона ES6 без __дополнительных зависимостей__. Код использует простое разделение состояния / разметки и повторно визуализирует всю клиентскую часть HTML при каждом изменении - никакого причудливого DOM диффинга и __никакого универсального рендеринга__ на данный момент. Также __нет разделения команд__ - [код](https://github.com/serzn1/micro-frontends/tree/master/0-model-store) записывается в один файл js / css.

### Интеграция На Стороне Клиента

В этом примере страница разделена на отдельные компоненты/фрагменты, принадлежащие трем командам. __Команда Checkout__ (blue) теперь отвечает за все, что касается процесса покупки, а именно за кнопку __buy__ и мини - корзину__. __Команда Inspire__ (green) управляет рекомендациями __продукта__ на этой странице. Сама страница является собственностью __команды__ (red).

[![Пример 1 - Страница Продукта-Композиция](./ressources/screen/three-teams.png)](./1-композиция-только для клиента/)

[попробуйте в браузере](./1-composition-client-only/) & [посмотреть код](https://github.com/serzn1/micro-frontends/tree/master/1-composition-client-only)

__Команда Product__ решает, какая функциональность включена и где она расположена в макете. Страница содержит информацию, которая может быть предоставлена самим продуктом Team, например название продукта, изображение и доступные варианты. Но он также включает в себя фрагменты (пользовательские элементы) из других команд.

### Как создать пользовательский элемент?

Возьмем в качестве примера кнопку __buy__. Группа продукции включает в себя кнопку, просто добавив `<blue-buy sku="t_porsche"></blue-buy>` в нужное место в разметке. Для работы этой команды кассу должен зарегистрировать элемент `blue-buy` на странице.

```javascript
class BlueBuy extends HTMLElement {
    connectedCallback() {
        this.innerHTML = `<button type="button">buy for 66,00 €</button>`;
    }

    disconnectedCallback() { ... }
}
window.customElements.define('blue-buy', BlueBuy);
```

Теперь каждое время когда браузер находит тег `blue-buy`, вызывается `connectedCallback`. `this` является ссылкой на корневой DOM элемент этого пользовательского элемента. Для работы с этими элементами могут быть использованы все свойства и методы как для работы со стандартным DOM элементом, например `innerHTML` или `getAttribute()`.

<img alt="Пользовательские элементы в действии" src="./ressources/video/custom-element.gif" loading="lazy" />

Есть единственное требование присвоении имени вашему элементу, которое определяет спецификация, заключается в том, что имя должно __включать тире (-)__ для поддержания совместимости с предстоящими новыми HTML-тегами. В следующих примерах используется соглашение об именовании `[team_color]-[feature]`. Пространство имен `team` защищает от коллизий, и таким образом право собственности на элемиент становится очевидным, просто взглянув на DOM.

### Связь Родитель-Наследник (Parent-Child) / Модификация DOM

Когда пользователь выбирает другой трактор в __селекторе вариантов__, кнопка __buy должна быть обновлена__ соответствующим образом. Для достижения этой цели командный продукт может просто __удалить__ существующий элемент из DOM __и вставить__ новый.

```javascript
container.innerHTML;
// => <blue-buy sku="t_porsche">...</blue-buy>
container.innerHTML = '<blue-buy sku="t_fendt"></blue-buy>';
```

Метод `disconnectedCallback` старого элемента будет вызван синхронно, чтобы дать элементу шанс сделать все необходимые операции, например, удалить обработчики событий. После этого будет вызван `connectedCallback` только что созданного элемента `t_fendt`.

Другой более быстродейственный вариант - обновить аттрибут `sku` на уже существующем элементе.

    document.querySelector('blue-buy').setAttribute('sku', 't_fendt');

Если __команда Product__ использует шаблонизатор, который сравниваете дом, как React, это будет сделано автоматически алгоритмом.

<img alt="Изменение аттрибута пользовательского элемента" src="./ressources/video/custom-element-attribute.gif" loading="lazy" />

Чтобы поддержать это, вы можете реализовать `attributeChangedCallback` и указать список `observedAttributes`, для которых этот метод должен быть вызван.

```javascript
const prices = {
    t_porsche: '66,00 €',
    t_fendt: '54,00 €',
    t_eicher: '58,00 €',
};

class BlueBuy extends HTMLElement {
    static get observedAttributes() {
        return ['sku'];
    }
    connectedCallback() {
        this.render();
    }
    render() {
        const sku = this.getAttribute('sku');
        const price = prices[sku];
        this.innerHTML = `<button type="button">buy for ${price}</button>`;
    }
    attributeChangedCallback(attr, oldValue, newValue) {
        this.render();
    }
    disconnectedCallback() {...}
}
window.customElements.define('blue-buy', BlueBuy);
```

Чтобы избежать повторения кода, используется метод `render()`, который вызывается из `connectedCallback` и `attributeChangedCallback`. Этот метод объединяет необходимые данные в новую разметку innerHTML. Если вы решите использовать более сложный механизм шаблонов или фреймворк внутри пользовательского элемента, то именно здесь необходимо сделать его инициализацию.

### Поддержка браузерами

The above example uses the Custom Element V1 Spec which is currently [supported in Chrome, Safari and Opera](http://caniuse.com/#feat=custom-elementsv1). But with [document-register-element](https://github.com/WebReflection/document-register-element) a lightweight and battle-tested polyfill is available to make this work in all browsers. Under the hood, it uses the [widely supported](http://caniuse.com/#feat=mutationobserver) Mutation Observer API, so there is no hacky DOM tree watching going on in the background.

### Совместимость с фреймворками

Поскольку пользовательские элементы являются веб-стандартом, все основные JavaScript-фреймворки, такие как Angular, React, Preact, Vue или Hyperapp, поддерживают их. Но когда вы вдаетесь в детали, в некоторых фреймворках все еще есть несколько проблем с реализацией. На сайте [Custom Elements Everywhere](https://custom-elements-everywhere.com/) [Rob Dodson](https://twitter.com/rob_dodson) собрал набор тестов совместимости, которые показывают все еще нерешенные проблемы совместимости.

### Наследник-Родитель (Child-Parent) и коммуникация менду компонентами 1го уровня / DOM события

Передача атрибутов не является достаточной для всех типов взаимодействий. В нашем примере __мини-корзина должна обновляться__, когда пользователь выполняет __клик на кнопке Купить__.

Оба фрагмента принадлежат Team Checkout (blue), поэтому они могли бы создать какой-то внутренний JavaScript API, который позволяет мини-корзине знать, когда была нажата кнопка. Но это потребовало бы, чтобы экземпляры компонентов знали друг о друге, что также было бы нарушением изоляции.

Более чистый способ-использовать механизм PubSub, где компонент может публиковать сообщение, а другие компоненты могут подписываться на определенные темы. К счастью, браузеры имеют эту встроенную функцию. Именно так работают события браузера, такие как `click`, `select` или `mouseover`. В дополнение к собственным событиям существует также возможность создавать события более высокого уровня с помощью `new CustomEvent(...)`. События всегда привязаны к узлу DOM, на котором они были созданы/отправлены. Большинство местных событий также используют всплытие событий (bubbling). Это позволяет прослушивать все события в определенном поддереве DOM. Если вы хотите прослушать все события на странице, прикрепите прослушиватель событий к элементу window. Вот как выглядит создание события `blue:basket:changed` на примере:

```javascript
class BlueBuy extends HTMLElement {
    [...]
    connectedCallback() {
        [...]
        this.render();
        this.firstChild.addEventListener('click', this.addToCart);
    }
    addToCart() {
        // maybe talk to an api
        this.dispatchEvent(new CustomEvent('blue:basket:changed', {
            bubbles: true,
        }));
    }
    render() {
        this.innerHTML = `<button type="button">buy</button>`;
    }
    disconnectedCallback() {
        this.firstChild.removeEventListener('click', this.addToCart);
    }
}
```

Теперь мини-корзина может подписаться на это событие в `window` и получать уведомления, когда она должна обновить свои данные.

```javascript
class BlueBasket extends HTMLElement {
    connectedCallback() {
        [...]
        window.addEventListener('blue:basket:changed', this.refresh);
    }
    refresh() {
        // fetch new data and render it
    }
    disconnectedCallback() {
        window.removeEventListener('blue:basket:changed', this.refresh);
    }
}
```

При таком подходе фрагмент мини-корзины добавляет прослушиватель к элементу DOM, который находится вне его области видимости (`window`). Это должно быть нормально для многих приложений, но если вам это не нравится, вы также можете реализовать подход, при котором сама страница (команда Product) прослушивает событие и уведомляет мини-корзину, вызывая `refresh()` на элементе DOM.

```javascript
// page.js
const $ = document.getElementsByTagName;

$('blue-buy')[0].addEventListener('blue:basket:changed', function() {
    $('blue-basket')[0].refresh();
});
```

Явный вызыв метода DOM довольно редко используется, но их можно найти, например, в [API Video элемента](https://developer.mozilla.org/de/docs/Web/HTML/Using_HTML5_audio_and_video#Controlling_media_playback). По возможности следует предпочесть использование декларативного подхода (изменение атрибутов).

## Отрисовка на сервере (SSR) / Universal Rendering

Пользовательские элементы отлично подходят для интеграции компонентов внутри браузера. Но при создании сайта, доступного в интернете, велика вероятность того, что первоначальная производительность будет иметь сущуственное значение, и пользователи будут видеть белый экран до тех пор, пока все фреймворки js не будут загружены и выполнены. Кроме того, полезно подумать о том, что произойдет с сайтом, если JavaScript выйдет из строя или будет заблокирован. [Jeremy Keith](https://adactio.com/) объясняет важность этого в своей электронной книге / подкасте [Resilient Web Design](https://resilientwebdesign.com). Поэтому возможность рендеринга основного контента на сервере является ключевой. К сожалению, спецификация веб-компонента вообще не говорит о рендеринге сервера. Нет JavaScript, нет пользовательских элементов :(

### Пользовательские элементы + Server Side Includes (SSI) = ❤️

Чтобы сделать серверную отрисовку рабочей, давайте отрефакторим предыдущий пример. Каждая команда имеет свой собственный `express` сервер, и метод `render()` пользовательского элемента также доступен по url-адресу.

```html
$ curl http://127.0.0.1:3000/blue-buy?sku=t_porsche
<button type="button">buy for 66,00 €</button>
```

Имя тега пользовательского элемента используется в качестве URL, а атрибуты становятся параметрами запроса. Теперь есть способ серверного рендеринга содержимого каждого компонента. В сочетании с пользовательским элементом `<blue-buy>`, достигается что-то совсем близкое к __универсальному веб-компоненту__:

```html
<blue-buy sku="t_porsche">
    <!--#include virtual="/blue-buy?sku=t_porsche" -->
</blue-buy>
```

Комментарий `#include` является частью [SSI](https://en.wikipedia.org/wiki/Server_Side_Includes), которая является функцией, доступной на большинстве веб-серверов. Да, это тот же самый метод, который использовался в те далекие дни, чтобы встроить текущую дату на наши статичные веб-сайты. Есть также несколько альтернативных методов, таких как [ESI](https://en.wikipedia.org/wiki/Edge_Side_Includes), [nodesi](https://github.com/Schibsted-Tech-Polska/nodesi), [compoxure](https://github.com/tes/compoxure) и [tailor](https://github.com/zalando/tailor), но для наших проектов SSI зарекомендовала себя как простое и невероятно стабильное решение.

Комментарий `#include` заменяется ответом от `/blue-buy?sku=t_porsche` перед тем, как веб-сервер отправит полную страницу в браузер. Конфигурация в nginx выглядит следующим образом:

```javascript
upstream team_blue {
    server team_blue:3001;
}
upstream team_green {
    server team_green:3002;
}
upstream team_red {
    server team_red:3003;
}

server {
    listen 3000;
    ssi on;

    location /blue {
        proxy_pass  http://team_blue;
    }
    location /green {
        proxy_pass  http://team_green;
    }
    location /red {
        proxy_pass  http://team_red;
    }
    location / {
        proxy_pass  http://team_red;
    }
}
```

The directive `ssi: on;` enables the SSI feature and an `upstream` and `location` block is added for every team to ensure that all urls which start with `/blue` will be routed to the correct application (`team_blue:3001`). In addition the `/` route is mapped to team red, which is controlling the homepage / product page.

Директива `ssi: on;` включает функцию SSI, и для каждой команды добавляется блок `upstream` и `location`, чтобы гарантировать, что все URL-адреса, начинающиеся с `/blue`, будут перенаправлены в правильное приложение (`team_blue:3001`). Помимо этого корневой адрес `/` принадлежит команде `red`, которая контролирует главную страницу / страницу товара.

Эта анимация показывает магазин тракторов в браузере, с __отключенным JavaScript__.

[<img alt="Serverside Rendering - Disabled JavaScript" src="./ressources/video/server-render.gif" loading="lazy" />](./ressources/video/server-render.mp4)

[inspect the code](https://github.com/serzn1/micro-frontends/tree/master/2-composition-universal)

Кнопки выбора варианта теперь являются реальными ссылками, и каждый щелчок приводит к перезагрузке страницы. Терминал справа иллюстрирует процесс направления запроса на страницу в `red` команде, которая управляет страницей продукта, а затем разметка дополняется фрагментами из команд `blue` и `green`.

При повторном включении JavaScript будут видны только сообщения журнала сервера для первого запроса. Все последующие изменения трактора обрабатываются на стороне клиента, как и в первом примере. В более позднем примере данные продукта будут извлечены из JavaScript и загружены через REST api по мере необходимости.

Вы можете обкатать этот код в вашем локальном окружении. Необходимо только установить [Docker Compose](https://docs.docker.com/compose/install/).

```javascript
git clone https://github.com/serzn1/micro-frontends.git
cd micro-frontends/2-composition-universal
docker-compose up --build
```

Затем Docker запескает nginx на 3000 порту и собирает ораз nodejs приложениядля каждой команды. Когда вы открываете [http://127.0.0.1:3000/](http://127.0.0.1:3000/) в браузере, вы должны увидеть красный трактор. Комбинированные логи `docker-compose` позволяет легко видеть, что происходит в сети. К сожалению, нет никакого способа контролировать цвета в логах, поэтому вам придется смириться с тем, что команда `blue` может быть выделена зеленым цветом :)

Файлы `src` сопоставляются в отдельные контейнеры, и nodejs приложение перезапустится, когда вы внесете изменение кода. Изменение `nginx.conf` требует перезапуска `docker-compose`, чтобы применить изменения. Так что не стесняйтесь поиграться с этим и дать обратную связь.

### Закрузка данных & Состояния загрузки

Недостатком подхода SSI/ESI является то, что __самый медленный фрагмент определяет время отклика__ всей страницы.
Поэтому хорошо, когда ответ фрагмента может быть кэширован.
Для фрагментов, которые дорого производить и трудно кэшировать, часто рекомендуется исключить их из первоначального рендеринга.
Они могут быть загружены асинхронно в браузере.
В нашем примере фрагмент `green-recos`, который показывает персонализированные рекомендации, является отличным примером для этого.

Одним из возможных решений было бы то, что команда `red` просто пропускает SSI Include инструкцию.

**До**

```html
<green-recos sku="t_porsche">
    <!--#include virtual="/green-recos?sku=t_porsche" -->
</green-recos>
```

**После**

```html
<green-recos sku="t_porsche"></green-recos>
```

*Важно знать: Пользовательские элементы [не могут быть самозакрывающимися тегами](https://developers.google.com/web/fundamentals/architecture/building-components/customelements#jsapi), поэтому версия `<green-recos sku="t_porsche" />` не будет работать корректно.*

<img alt="Reflow" src="./ressources/video/data-fetching-reflow.gif" style="width: 500px" loading="lazy" />

Отрисовка происходит только в браузере.
Но, как видно из анимации, это изменение теперь привело к __существенному reflow__ страницы.
Область рекомендаций изначально пуста.
Код команды `green` загружается и выполняется.
Делается вызов API для получения персонализированной рекомендации.
Выводится разметка рекомендаций и запрашиваются соответствующие изображения.
Фрагмент теперь нуждается в большем пространстве и меняет высоту страницы.

Существуют различные варианты, чтобы избежать раздражающего скачка страницы, подобного этому.
Команда `red`, которая управляет страницей, может __зафиксировать высоту рекомендаций__.
На адаптивном сайте часто сложно определить высоту, потому что она может отличаться для разных размеров экрана.
Но более важный вопрос заключается в том, что такого рода межкомандное соглашение создает тесную связь между командами `red` и `green`.
Если команда `green` хочет ввести дополнительный подзаголовок в элементе `reco`, она должна будет согласовать новую высоту блока с командой `red`.
Обе команды должны были бы развернуть свои изменения одновременно, чтобы избежать сломанной компоновки.

Лучший способ - использовать технику, называемую [Sceleton Screens/Экран плейсхолдер](https://blog.prototypr.io/luke-wroblewski-introduced-skeleton-screens-in-2013-through-his-work-on-the-polar-app-later-fd1d32a6a8e7).
Команда `red` оставляет SSI `green-recos` включенным в разметку.
Кроме того, команда `green` изменяет __метод рендеринга на стороне сервера__ своего фрагмента таким образом, чтобы он создавал __схематическую версию содержимого__.
Разметка __плейсхолдера__ может повторно использовать части стилей макета реального контента.
Таким образом, плейсхолдер __резервирует необходимое пространство__ на сайте и заполнение фактического содержимого не приводит к скачку.

<img alt="Skeleton Screen" src="./ressources/video/data-fetching-skeleton.gif" style="width: 500px" loading="lazy" />

Техника экрана-плейсхолдера (Skeleton Screen) также __очень полезна на клиенте__.
В момент появления пользовательских элементов в DOMе они могут быть __показаны мгновенно__, задолго до момента загрузки реальных данных с сервера.

Даже при __изменении атрибута__, например для _элемента выбора_, вы можете переключиться на экран плейсхолдер до тех пор, пока не поступят новые данные.
Таким образом, пользователь осведомлен о том, что что-то происходит во фрагменте страницы.
Но когда ваш API реагирует достаточно быстро, короткое мерцание плейсхолдера между старым и новым состоянием может раздражать пользователей.
Кеширование старых данных или использование тайм-аутов может помочь избавить пользователей от раздражения.
Поэтому используйте эту технику с умом и постарайтесь получить обратную связь от пользователей.

## Навигация между страницами

__скоро...__

Подрисывайтесь на [Github Repo](https://github.com/serzn1/micro-frontends) и будьте в курсе событий

## Дополнительные ресурсы

* [Книга: Micro Frontends in Action](https://www.manning.com/books/micro-frontends-in-action?a_aid=mfia&a_bid=5f09fdeb) Написано мной. В настоящий момент в статусе Mannings Early Access Programm (MEAP)
* [Доклад: Micro Frontends - MicroCPH, Copenhagen 2019](https://www.youtube.com/watch?v=wCHYILvM7kU) ([Слайды](https://noti.st/naltatis/zQb2m5/micro-frontends-the-nitty-gritty-details-or-frontend-backend-happyend)) The Nitty Gritty Details or Frontend, Backend, 🌈 Happyend
* [Доклад: Micro Frontends - Web Rebels, Oslo 2018](https://www.youtube.com/watch?v=dTW7eJsIHDg) ([Слайды](https://noti.st/naltatis/HxcUfZ/micro-frontends-think-smaller-avoid-the-monolith-love-the-backend)) Think Smaller, Избегайте монолита, ❤️the Backend
* [Слайды: Micro Frontends - JSUnconf.eu 2017](https://speakerdeck.com/naltatis/micro-frontends-building-a-modern-webapp-with-multiple-teams)
* [Доклад: Break Up With Your Frontend Monolith - JS Kongress 2017](https://www.youtube.com/watch?v=W3_8sxUurzA) Elisabeth Engel рассказывает о реализации Micro Frontends в gutefrage.net
* [Статья: Micro Frontends](https://martinfowler.com/articles/micro-frontends.html) Article by Cam Jackson on Martin Fowlers Blog
* [Пост: Micro frontends - a microservice approach to front-end web development](https://medium.com/@tomsoderlund/micro-frontends-a-microservice-approach-to-front-end-web-development-f325ebdadc16) Tom Söderlund объясняет ключевые понятия и делится ссылками на тему
* [Пост: Microservices to Micro-Frontends](http://www.agilechamps.com/microservices-to-micro-frontends/) Sandeep Jain подитожит ключевые принципы лежащие в основе микросервисов и микро фронтендов
* [Коллекция ссылок: Micro Frontends by Elisabeth Engel](https://micro-frontends.zeef.com/elisabeth.engel?ref=elisabeth.engel&share=ee53d51a914b4951ae5c94ece97642fc) обширный список постов, докладов, инструментов и других ресурсов по теме
* [Awesome Micro Frontends](https://github.com/ChristianUlbrich/awesome-microfrontends) список актуальных ссылок от Christian Ulbrich 🕶
* [Custom Elements Everywhere](https://custom-elements-everywhere.com/) Making sure frameworks and custom elements can be BFFs
* Заказать трактор вы можете на сайте [manufactum.com](https://www.manufactum.com/) :)<br>_Этот магазин разраьботан двумя командами с использованием изложенной выше техники микрифронтендов._

## Сопутствующие техники

- [Посты: Cookie Cutter Scaling](https://paulhammant.com/categories.html#Cookie_Cutter_Scaling) David Hammet написал серию статей по теме.
- [Вики: Java Portlet Specification](https://en.wikipedia.org/wiki/Java_Portlet_Specification) Спецификация, в которой рассматриваются аналогичные темы для построения корпоративных порталов.

---

## Скоро...

* Примеры использования
  * Навигация между страницами
    * soft vs. hard навигация
    * universal router
  * ...
* Интересные темы
  * Изолирование CSS / Понятный UI / Стилизация & Библиотеки шаблонов
  * Производительность при первой загрузке
  * Производительность во время работы
  * Подгрузка CSS
  * Подгрузка JS
  * Интеграционное тестирование
  * ...

## Крнтрибуторы

* [Koike Takayuki](https://github.com/koiketakayuki) перевел сайт на [Японский](https://micro-frontends-japanese.org/).
* [Jorge Beltrán](https://github.com/scipion) перевел сайт на [Испанский](https://micro-frontends-es.org).
* [Sergei Babin](https://github.com/serzn1) перевел сайт на [Русский](https://serzn1.github.io/micro-frontends/).
- [Bruno Carneiro](https://github.com/Tautorn) перевел сайт на [Португальский](https://tautorn.github.io/micro-frontends/).
- [Soobin Bak](https://github.com/soobing) перевел сайт на [Корейский](https://soobing.github.io/micro-frontends/).

Этот сайт сгенерирован при помощи Github Pages. Вы можете найти исходники на [serzn1/micro-frontends](https://github.com/serzn1/micro-frontends/).

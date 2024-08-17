# Computer Science

## СОДЕРЖАНИЕ
- Immer (#immer)

___________________________________________________________________________________________________
Shallow copy - поверхностное копирование, т.е. пересоздание с нуля только первого уровня объекта (вложенные объекты буду унаследованы по ссылке)

Structural sharing -
- возвращает независимую копию данных
- если в части исходной структуры данные НЕ поменялись, то их же и возвращаем (копируем по ссылке)
- если в части исходной структуры данные поменялись, то эти части пересоздаем с нуля с новыми значениями (пересоздаем через shallow copy)
- если в части исходной структуры данные поменялись ГЛУБОКО, то весь путь от этого глубокого изменения до корня на каждом уровне должен быть пересоздан через shallow copy (.../Object.assign)

В примере ниже мы хотим поменять m на 321. Это изменение вызовет shallow copy объекта под ключом k. Тогда для объекта y свойство k поменялось, а значит весь объект под ключом y тоже надо прогнать через shallow copy. Тоже самое для объекта x с измененным ключом y. И таким формируется путь изменений. y2 затронут при этом не будет, скопируется по ссылке

const x = {
    y: {
        k: {
            m: 123
        }
    },
    y2: ...
}

Structural cloning -  синоним deep copy. С недавних пор официальное api браузера structuredClone (клонирует многие вещи, но не все, не клонирует методы). Не путать с Structural sharing

Reflect 
- Определение встроенный объект js, содержащий набор методов-обёрток вокруг внутренних методов (стандартного ECMASCRIPT) объекта по спецификации, т.е. ([[Get]], [[Set]], [[Delete]], [[Construct]], ...)
- Стандартный объект ECMASCRIPT - объект, поведение которого жёстко определено стандартными алгоритмами спецификации
- [[Get]], [[Set]], [[Delete]], [[Construct]] - не просто слова в квадратных скобках, это название стандартных алгоритмов спецификации, которые выполняют определенную последовательность шагов при взаимодействии с объектом (например, удаление, получение свойств и т.д.)
- Объекты, у которых поведение не совпадает с этими стандартными алгоритмамиЮ называются экзотическими. Например, Array, потому что его поведение установки нового свойства затрагивает перерасчет специфического только ему свойства length

Proxy
- встроенный в js декоратор над объектом
- в параметрах принимает исходный объект и объект с настройками декоратора (ловушки/trap)
- Настройки декоратора также перехыватывают алгоритмы (стандартного ECMASCRIPT) поведения объекта. Мы, программисты, благодаря этому перехвату можем изменить поведение target (исходного объекта)
- из-за того что proxy действует не только в мире js, но и за его пределами (обращаясь к внутренним свойствами вроде [[Get]] и т.д.), то на чужой территории програмимсту надо вести себя в соответствии с чужими законами, в данном случае согласно спецификации ECMASCRIPT (например, перенаправление корректного this => proxy намертво связан с исходным объектом и это может приводить к проблемам при наследовании в прототипной цепи; возвращение булевого флага при set; правильной обработки ошибок; ...)
- Программист самостоятельно обрабатывает все вышеуказзаные случаи, либо полагается на Reflect, который берёт обработку на себя. Важно помнить о Reflect, который поможет программисту. Даже Reflect не может закрыть все corner cases, как например, внутренние слоты (например [[MapData]] это аналог внтуренних свойств объекта вроде [[Get]], ...) Map, Set, приватные поля классов и тогда программисту все же приходится самостоятельно обрабатывать эти случаи
- Revoke Proxy возможность разорвать связь между исходным объектом и proxy

Immer - библиотека, которая позволяет писать код в mutable стиле. Наружу отдаем immutable копию данных, используя под капотом structural sharing

Термины библиотек и Immer:
- Produce внешнее api, которое принимает 2 параметра: base и recipe, возвращает новую immutable копию данных
- Recipe колбек который имеет параметр draft и меняет его в мутирующем стиле
- Base исходные пользовательские данные (не обязательно объект, м.б. и примитивы, maps, ...)
- Copy => shallow copy base
- State объект, содержащий base + copy + метаинформация
- Draft это прокси над state

История развития immer
- Рассмотрим react. Так react вызывает перерендер через сопоставление ссылок, а не вложенной не структуры, то важно на каждое изменение возвращать новую копию данных даже если оно произошло очень глубоко (принцип иммутабельности). Immer идеально ложится на данную философию
- шаг 1 решения проблемы. Вместо работы с исходными данными ВСЕГДА создаём копию. DeepClone копия затратно по ресурсам, поэтому используем shallow copy
- шаг 2 решения проблемы. Shallow copy имеет недостатки в виде НЕ обработки вложенных структуру данных и возврата каждый раз новой копии, даже если данные не поменялись
- шаг 3 решения проблемы. Содание обёртку, именуемую state. Исходные данные - base, shallowCopy от base - copy. Обёртка также содержит любую доп. информацию для коректной работы state в виде meta. Недостаток третьего шага - пользователь вместо изначальной data работает со сложным api в виде state, лишний головняк
- шаг 4 решения проблемы. Прячем state за proxy. Снаружи человек обращается будто бы к data, а внутри всё реализуется через proxy, используя structural sharing
- шаг 5 решения проблемы. Любые вложенные объекты обрабатываются рекурсивно создавая для КАЖДОГО из них свой proxy над своим state. Чтобы вернуть пользователю не систему state, а нормальный имутабельный объект, необходимо пробежаться по всем этим state и собрать новый имутабельный объект
- в процессе работы может образоваться микс base, copy, state. Чтобы избежать ситуации, когда снаружи у нас обычный base, а внутри изменен глубоко вложенный state, нам необходимо всю цепочку объектов, включая внешний base пересоздать (это философия structural sharing). Для этого флаг загрязнение глубоковложенного state распространяется на весь путь до верхнего base
- ![immer_history](/img/immer.png)

- produce внутри себя выполняет следующие действия:
  - scope = getCurrentScope() // singleton на весь вызов одного produce
  - draft = createProxy(base)
  - result = recipe(draft)
  - processResult(result, scope) // нормализация финального объекта для пользователя
- scope содержит:
  - drafts_  (собрание всех draft данного produce)
  - immer_   (экземпляр immer от имени которого вызван produce, часто используется вместо this во внутренней логике)
  - parent_  (родительский scope, produce можно вызывать внутри produce)
  - canAutoFreeze (все возвращаемые копии объекта можно замораживать)
  - unFinalizedDraft (количество незавершенных drafts. Завершенный draft - м.б. это draft который в функции processResult был обработан)
- state содержит:
  - base
  - copy
  - proxy
  - revoke
  - parent (родительский proxy, не путать с родителььским scope)
  - scope
  - isModified
  - isFinalized
- latest служебная функция, которая возвращет либо copy, либо base из state
- peek(obj, prop) выдает актуальное значение по переданному ключу: 
  - если obj === draft, то latest(draft)[prop]
  - если не draft, то obj[prop]
- get ловушка:
  - принимает 2 основных параметра: state и property (значение которого хотим получить)
  - вызываем latest(state)[prop] и получаем value
  - анализируем value на примитив. Если да, то return
  - если value не примитив и не proxy if(value === peek(state.base_, prop)), то создаем proxy и записываем его в state.copy[prop]
  - если value уже proxy, то вернуть его и ничего не делать
- set 
  - принимается 3 основных параметра: state, property, value
  - если пришедшее value равно уже имеющемуся, то запись не производить
  - если пришедшее value НЕ равно уже имеющемуся и флаг isModified => true, то производим запись в copy
  - если пришедшее value НЕ равно уже имеющемуся и флаг isModified => false, то производим запись в copy и загрязняем весь путь до корня
  - никаких proxy не создается, они создаются в get
  - если в set мы передаем obj в качестве value И под таким property уже лежит proxy над данным obj, то immer выкидывает proxy и устанавливает obj (corner case)


RTK (Redux tool kit)
- deriving - выведение данных на основе других данных. Не надо заводить отдельный state для deriving data
- cache/memoization - cache (получение данных не из источника, например, copy); memoization (частный случай cache для ускорения получения результата, для предотвращения повторного запуска функции)
- selector - функция, которая дерайвит данные 
- selector лучше иметь ближе к источнику данных (source of truth), иначе во всех местах, где этот селектор используется, его придется исправлять
- selectors можно располгать там, где есть source of truth (middleware, sage, reduce, component, ...)

Философия RTK, HOS (selectors высшего порядка)
- redux в своей логике обновления state использует structural sharing (НЕ deepClone)
- high-order functions пришло из ункционального декларативног опрограммирования. Там функции представляют собой не процесс вычисления, сопоставление входын хи выходных данных (как на бумаге). Такогих понятий, как замыкание, создание вложенных функций там нет. Поэтому единственным 2 способами использовать функцию в функции являются либо передача в параметрах, либо возвращение в качестве результата. Однако философия императивного js иная, с этой точки зрения можно взять функцию из замыкания, что также будет high-order function
-  Бывают selectors первого порядка, работают с источником данных. Selectors высшего порядка, которые работают с результатами selectors низшего порядка. Можно ли назвать selectors высшего порядка high-order selectors, такого определения нет нигде, но это удобно???
- createSelector принимает параметры: последнйи является output selector, все до него input selectors.Selectors низшего порядка являются input для High-order-selectors/HOS (selectors высшего порядка).  output callback являющийся частью HOS (selectors высшего порядка) рассчитывает результаты на основе результатов input selectors
- input selector не должен всегда возвращать новый результат (в случае redux весь state), иначе теряется смысл оптимизириующей логики HOS (selectors высшего порядка), в частном случае мемоизации
- output selector не имеет смысла, если он не производит трансформирующую логику. Легче сразу забрать данные от низшего selector без создания HOS (selectors высшего порядка), т.е. без использования re-select
- философски redux and СУБД схожи в сути нормализации данных, селектировании, ..., но сильно отличаются технически

- HOS (selectors высшего порядка) м.б. input selector для другого selector
- HOS (selectors высшего порядка) кеширует данные для одного потребителя. Если один HOS (selectors высшего порядка) используется 2 и более потребителями, то они могут перезаписывать cache и смысли мемоизации текущего HOS (selectors высшего порядка) теряется
- Вариант решения проблемы:
  - фабрика selectors
  - вручную создавать HOS (selectors высшего порядка) для каждого потребителя
  - использовать re-reselect (кеширование по дополнительному и редко меняющемуся атрибуту. Среди параметров скидываемых для HOS и этот параметр или их комбинация будет ключом cache)
- memoization спасает при тяжелых вычислениях
- HOS спасает при ссылочных типах данных (в случае возврата каждый раз новой ссылки селектором нижнего порядка, calculation над этими ссылками м. возвращать постоянное значени, что предотвратит react от перерендера)
- если логика программиста НЕ подразумевает calculation, а только селектрование, то смысла в HOS нет
- не нужно создавать селектора первого уронвя на каждое свойство state, это ухудшает читаеомсть/поддерживаемость кода

- каскадная мемоизация (cascade memoization) - мемоизация вложенных уровней
- локальные и глобальные селекторы - локальные селекторы получают кусок store, глобальные - весь store
- В официальной доке reselect, redux и наших определениях есть расхождения. Синхронизация терминов:
  - HOS / Memoized selector / Output Selector (исп. на сайте reselect)
  - Selector низшего порядка / Dependencies / Input Selector 
  - Result Func / Output Selector (исп. на сайте redux)
  
________________________________________________________________________________________________________


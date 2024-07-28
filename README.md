# Computer Science

## СОДЕРЖАНИЕ
- Immer (#immer)

___________________________________________________________________________________________________
Immer - библиотека, которая позволяет писать код в mutable стиле. Наружу отдаем immutable копию данных, используя под капотом structural sharing

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

Термины библиотек и Immer:
- Produce внешнее api, которое принимает 2 параметра: base и recipe, возвращает новую immutable копию данных
- Recipe колбек который имеет параметр draft и меняет его в мутирующем стиле
- Base исходные пользовательские данные (не обязательно объект, м.б. и примитивы, maps, ...)
- Copy shallow copy base
- State объект, содержащий base + copy + метаинформация
- Draft это прокси над state

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
- Ппограммист самостоятельно обрабатывает все вышеуказзаные случаи, либо полагается на Reflect, который берёт обработку на себя. Важно помнить о Reflect, который поможет программисту. Даже Reflect не может закрыть все corner cases, как например, внутренние слоты (например [[MapData]] это аналог внтуренних свойств объекта вроде [[Get]], ...) Map, Set, приватные поля классов и тогда программисту все же приходится самостоятельно обрабатывать эти случаи
- Revoke Proxy возможность разорвать связь между исходным объектом и proxy

________________________________________________________________________________________________________


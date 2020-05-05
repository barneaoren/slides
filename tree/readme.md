# Tree - единый AST чтобы править всеми

Здравствуйте, меня зовут Дмитрий Карловский и я.. профессиональный велосипедостроитель с десятилетним стажем. И сегодня я научу вас делать не просто велосипеды, а болиды формулы один, на примере разработки идеального формата данных.

Я уже [рассказывал о нём](https://habr.com/ru/post/248147/) 5 лет назад, что привело к жарким дебатам, повлёшим за собой небольшие изменения синтаксиса. Поэтому позвольте рассказать с чистого листа что он представляет из себя на текущий момент.

```
Спикер \Дмитрий Карловский
Место \Undefined Meetup #1
Время 2019-09-17
```

Это - текстовая версия одноимённого выступления на [Undefined Meetup #1](https://events.epam.com/events/undefined-meetup-1). Вы можете [читать её как статью](https://github.com/nin-jin/slides/blob/master/tree/readme.md), либо [открыть в интерфейсе проведения презентаций](https://nin-jin.github.io/slides/tree/), либо [посмотреть видео](https://youtu.be/vBqJWQzPB3Y?t=5652).

# План

- Проанализировать популярные текстовые форматы данных 💩
- С нуля разработать новый формат без недостатков 👽
- Показать примеры применения нового формата 👾

# Форматы

Сравнивать мы будем 5 форматов.

| **Формат**
|-----------
| XML
| JSON
| YAML
| TOML
| Tree

Про первые три не слышал только глухой. А вот два последних для многих - тёмные лошадки. Ну ничего, сегодня я пролью на них свет.

## Пример XML

XML - некогда самый популярный формат, можно сказать "технологический стандарт". Но не смотря на всю свою мощь, сейчас он изживает своё, так как является слишком сложным для современного веб-разработчика.

```xml
<!DOCTYPE svg
	PUBLIC "-//W3C//DTD SVG 1.1//EN"
	"http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd"
>
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">
	<circle r="30" cx="50" cy="50" fill="orange" />
</svg>
```

## Пример JSON

На смену XML приходит более простой и дерзкий формат данных - JSON.

```json
{
	"name": "example",
	"version": "1.0.0",
	"description": "example package",
	"main": "index.js",
	"repository": "https://example.org",
	"author": "aninymous",
	"license": "MIT"
}
```

Если вы считаете, что вот он, идеал, то прошу меня заранее извинить, так как далее я буду вас расстраивать.

## Пример YAML

Кто-то уже пророчит YAML на смену JSON.

```yaml
Date: 2001-11-23 15:03:17 -5
User: ed
Fatal:
  Unknown variable "bar"
Where:
  file: TopClass.py
  line: 23
  code: |
    x = MoreObject("345\n")
```

Благодаря лучшей человекочитаемости он уже обрёл популярность в сфере ручного написания конфигурационных файлов.

## Пример TOML

Про TOML же мало кто слышал. Однако, взгляните на пример и вам станет ясно, почему я о нём вообще упоминаю.

```toml
[servers]

[servers.alpha]
ip = "10.0.0.1"
dc = "eqdc10"

[servers.beta]
ip = "10.0.0.2"
dc = "eqdc10"
```

Да, это фактически, стандартизированный INI-конфиг, которого покусал JSON. В результате чего он вобрал в себя худшее из обоих миров.

## Пример Tree

Наконец, в качестве спойлера, позвольте показать вам минимальный не пустой файл в формате Tree, который мы и будем разрабатывать далее.

```tree
spoiler
```

# Модели данных

Разные форматы основаны на разных моделях данных. Выбранная модель отвечает на следующие два вопроса.

- Какие данные мы можем записать и прочитать без бубна? 🥁
- Как записывать данные не вписывающиеся в модель? 👠

Ни один формат не способен поддержать всё многообразие типов предметных областей, поэтому неизбежно возникает необходимость упаковки данных в некоторый формат и обратной распаковки.

## Модель XML

XML основан на модели типизированных элементов в которых находится один словарь атрибутов и один список вложенных типизированных узлов.

- NodeList
- Element Node (`<br/>`)
- Attribute Node (`tabindex="1"`)
- Text Node (`Hello, World!`)
- CDATA Node (`<![CDATA[ ... ]]>`)
- Processing Instruction Node (`<? ... ?>`)
- Comment Node (`<!-- ... -->`)
- Document Node
- Document Type Node (`<!DOCTYPE html>`)

### Расширяемость модели XML

...

### Недостатки модели XML

Это довольно гибкая модель, однако она имеет ряд ограничений: значениями атрибутов могут быть только строки, а вложенный список узлов может быть только один. Не смотря на то, что формат XML и так не самый простой, банальный словарь с поддеревьями в качестве значений требует дополнительных соглашений. Например, что некоторые элементы используются для описания ключей в родительском элементе и такие элементы в родительском должны быть лишь в одном экземпляре.

```html
<panel>
	<head>Вы уверены?</head>
	<body>
		<button>Да</button>
		<button>Нет</button>
	</body>
</panel>
```

## Модель JSON

Модель JSON основана на том, что всё дерево состоит из нетипизированных списков и словарей. Плюс ограниченный набор примитивов в качестве листьев дерева.

- Null
- Boolean
- Number
- String
- Array
- Dictionary

### Недостатки модели JSON

Наивно было бы полагать, что двух типов структурных узлов хватит для всего. Например, представления банального синтаксического дерева сложно назвать удобоваримыми, как бы мы его ни упаковали в JSON.

```json
{
	"type" : "returning",
	"expression" : {
		"type" : "multiplication",
		"left" : {
			"type" : "number",
			"value" : 2
		},
		"right" : {
			"type" : "number",
			"value" : 2
		}
	}
}
```

```json
[ "returning",
	[ "expression" ,
		[ "multiplication",
			[ "number" , 2 ]
			[ "number" , 2 ]
		]
	]
]
```

## Модель YAML

Модель YAML во многом аналогична модели JSON. Разве что тут есть поддержка времени и внутренних ссылок.

- Null
- Boolean
- Number
- String
- DateTime
- Array
- Dictionary
- Alias
- Reference
- Document
- **TypeTags**

### Расширение пмодели YAML

Главное преимущество YAML в аннотациях типа, позволяющих объяснять процессору какой алгоритм использовать для распаковки данных.

```
--- !!omap
- Mark McGwire: 65
- Sammy Sosa: 63
- Ken Griffy: 58
```

В данном примере мы говорим парсеру "возьми этот список пар ключ-значение" и преобразуй его в объект OrderedMap (упорядоченный словарь).

### Недостатки модели YAML

...

## Модель TOML

Модель TOML более приземлённая. Например, тут различаются целые и вещественные числа, что важно для компилируемых языков.

- Boolean
- Integer
- Float
- String
- DateTime
- Array
- Dictionary

С расширяемостью же тут всё так же плохо, как и в JSON.

## Модель Tree

Какой бы набор базовых типов мы ни выбрали, его не хватит для всего. А значит неизбежно потребуется некотороый код упаковки и распаковки. А работать с таким кодом проще всего, когда число разных типов узлов минимально, так как для каждого типа требуется писать отдельную ветку логики. В то же время гибкость требуется максимальная. Поэтому нам хватит всего двух типов узлов.

- Struct Node
- Name Node

Именованные узлы служат для описания иерархии, а узлы данных хранят некоторые бинарные данные. Любой узел может хранить в себе список любых других узлов, чем достигается недостижимая в иных форматах гибкость.

## Расширяемость модели

Итого по расширяемости всё очень плохо. Популярные форматы либо расширяемые, но неимоверно переусложнённые, либо простые, но совершенно не расширяемые.

|                 | XML  | JSON | YAML | TOML | Tree
|-----------------|------|------|------|------|-----
| Расширяемость   | ✅  | ❌   | ✅  | ❌   | ✅
| Число паттернов | 90   | 30   | 210  | 90   | **10**

Обратите внимание на YAML. Его грамматика насчитывает две сотни паттернов. Он настолько сложен, что вы скорее всего не найдёте ни одной полной и правильной реализации парсера. Да что уж там, даже два одинаково работающих парсера JSON [нужно ещё поискать](https://habr.com/ru/company/mailru/blog/314014/), а ведь там всего 30 паттернов.

Нашей целью же будет создание предельно простого, не допускающего разночтений, но в то же время максимально расширяемого формата.

# Удобочитаемость

Наглядность синтаксиса важна при самых различных сценариях работы с форматом.

- При написании ✍🏽
- При ревью кода 📖
- При разрешении конфликтов 🤝🏽
- При отладке 🔧
- При изучении 🔬

Скорость вашей работы и предсказуемость её результатов напрямую зависит от способа сериализации формата.

## Удобочитаемость XML

XML построен вокруг текста, внутрь которого вкрапляются теги с дополнительной информацией. Пока этой информации не много, всё хорошо, но чем её больше, тем сложнее воспринимать текст, что нивелирует полезность этой фичи.

```xml
Привет, Алиса!
Как дела?
Не могла бы ты принести мне сейчас кофе?

<message>
	<greeting>
		Привет, <a href="http://example.org/user/alice">Алиса</a>!
	</greeting>
	<body>
		<s>Как дела?</s><br/>
		Не могла бы ты принести мне
		<time datetime="1979-10-14T12:00:00.001-04:00">сейчас</time>
		кофе?
	</body>
</message>
```

## Удобочитаемость JSON

XML хотя бы поддерживает многострочный текст, а вот JSON, например, этим похвастаться уже не может. Форматы этого типа идут от информаионной структуры, в которую уже вкрапляются текстовые и не только текстовые значения. 

```
{ "greetings": "Привет, Алиса!\nКак дела?\nНе могла бы ты принести мне кофе?\n" }
```

## Удобочитаемость YAML

В YAML большой выбор способов представления одного и того же.

- 5 типов строк 😣
- 4 модификатора обработки пробелов 😥

Главное - их не перепутать, иначе вы рискуете неправильно интерпретировать написанное. Но зато читаемо, да.

# Экранирование

- Нужно отличать конструкции формата от собственно данных 😵
- Желательно не терять в наглядности данных 🤓
- Желательно не переусложнять редктирование 🤬

## Экранирование в XML

```xml
foo > 0 & foo < 10


<code>foo &gt; 0 &amp; foo &lt; 10</code>
```

## Экранирование в JSON

```
/"[\s\S]*"/
```

```json
"\"[\\s\\S]*\""
```

## Экранирование в YAML

- 5 типов строк 😣
- 4 модификатора обработки пробелов 😥

## Экранирование в Tree

	Нет 😲

# Минификация

- Читаемое форматирование много весит 🐘
- Компактное форматирование плохо читается 💀

## Минификация XML

```xml
<users>
	<user>
		<name>Alice</name>
		<age>20</age>
	</user>
</users>


<!-- 13% less -->
<users><user><name>Alice</name><age>20</age></user></users>
```

## Минификация JSON

```json
{
	"users": [
		{
			"name": "Alice",
			"age": 20
		}
	]
}


// 30% less
{"users":[{"name":"Alice","age":20}]}
```

## Статистика по минификации

|                  | XML  | JSON     | YAML | TOML | Tree     
|------------------|------|----------|------|------|----------
| Читаемый         | 195% | 140%     | 125% | 110% | **100%** 
| Минифицированный | 170% | **100%** | -    | -    | -        

Скачать [файлы](https://github.com/nin-jin/tree.d/tree/master/formats).

# Холивары

- Табы или пробелы? 🤼‍♂️
- 2 или 4 символа? 🤼‍♀️
- возврат каретки? ⚡
- выравнивание? 🤺
- правила линтера/форматтера? 🔥
- когда запускать линтер/форматтер? 🚧

# Скорость обработки

```
serialization:    foo\bar    =>  "foo\\bar"

parsing:         "foo\\bar"  =>   foo\bar
```

# Координаты ошибки

|         | XML | JSON | YAML | TOML | Tree
|---------|-----|------|------|------|-----
| Адрес   | ✅  | ❌  | ❌   | ❌  | ✅
| Позиция | ❌  | ❌  | ❌   | ❌  | ✅

# Поточная обработка

|                      | XML | JSON | YAML | TOML | Tree
|----------------------|-----|------|------|------|-----
| Поточная обработка   | ❌  | ❌  | ✅   | ✅  | ✅

# Формат Tree

- Простота синтаксиса ✌
- Никакого экранирования 🤘
- Бинарная безопасность 👌
- Никаких вольностей 🤙
- Никакой минификации 👍
- Минимальный размер 👐
- Гарантированная читаемость 🖖
- Поточная обработка 💪
- Точные координаты ☝

## Просто tree-узел 

```tree
house
```

## Список tree-узлов

```tree
house
	roof
	wall
	door
	window
	floor
```

## Глубокая tree-иерархия

```tree
house
	roof
	wall
		door
		window
			glass
	floor
```

## Когда ребёнок остаётся один в дереве

```tree
street
	house
		wall
			door
			window
```

```tree
street house wall
	window
	door
```

## Сырые данные

```tree
\Любые данные \(^_^)/
```

## Многострочные данные

```tree
\
	\Тут 🐱‍💻
	\   очень 🐱‍👓
	\        много 🐱‍👤
	\             котиков 🐱‍🏍
```

## Разные типы узлов

```tree
user
	name \Jin
	age \35
	hobby
		\kendo 🐱‍👤
		\dance 🕺🏽
		\role play 🎭
```

# Языки основанные на форматах

| Формат         | **Языки**
|----------------|-------------------------
| XML            | XHTML, SVG, XSLT, ...  
| JSON           | JSON Schema, json:api, ...
| YAML           | yaml.org/type 
| TOML           | -
| Tree           | grammar.tree, xml.tree, json.tree, view.tree, ...

## Язык grammar.tree

```tree
tree .is .optional .list_of line

line .is .sequence
	.optional indent
	.optional nodes
	new_line

nodes .is .sequence
	.optional .list_of struct
	.optional data
	.with_delimiter space

struct .is .list_of .byte
	.except special
```

```tree
data .is .sequence
	data_prefix
	.optional .list_of .byte
		.except new_line

special .is .any_of
	new_line
	data_prefix
	indent
	space

new_line .is .byte \0A
indent .is .list_of .byte \09
data_prefix .is .byte \5C
space .is .list_of .byte \20
```

## Язык grammar.tree vs EBNF

```tree
tree .is .optional .list_of line

line .is .sequence
	.optional indent
	.optional nodes
	new_line

nodes .is .sequence
	.optional .list_of struct
	.optional data
	.with_delimiter space
```

```abnf
tree = { line };

line = [ indent ],
	[ nodes ],
	new_line;

nodes = data |
	struct,
	{ space , struct },
	[ space , data ];
```

## Язык xml.tree vs XML

```tree
! doctype html
html
	meta @ charset \utf-8
	link
		@ href \web.css
		@ rel \stylesheet
	script @ src \web.js
	body
		div @ mol_view_root \$my_app
```

```xml
<!doctype html>
<html>

	<meta charset="utf-8" />
	<link href="web.css" rel="stylesheet" />
	<script src="web.js"></script>

	<body>
		<div mol_view_root="$my_app"></div>
	</body>

</html>
```

## Язык json.tree vs JSON

```tree
* user *
	name \Jin
	age 35
	hobby /
		\kendo 🐱‍👤
		\dance 🕺🏽
		\role play 🎭
```

```json
{
	"user": {
		"name": "Jin",
		"age": 35,
		"hobby" : [
			"kendo 🐱‍👤",
			"dance 🕺🏽",
			"role play 🎭"
		]
	}
}
```

## Язык view.tree vs TypeScript

```tree
$my_details $mol_view
	sub /
		<= Pager $mol_paginator
			value?val <=> page?val 0
```

```js
class $my_details extends $mol_view {

	sub() { return [ this.Pager() ] }

	@ $mol_mem Pager() {
		const Pager = new $mol_paginator
		Pager.value = val => this.page( val )
		return Pager
	}

	@ $mol_mem page( val = 0 ) {
		return val
	}

}
```

# API

| Формат         | Языки                      | **API**
|----------------|----------------------------|------------
| XML            | XHTML, SVG, XSLT, ...      | DOM, SAX
| JSON           | JSON Schema, json:api, ... | -
| YAML           | yaml.org/type              | -
| TOML           | -                          | -
| Tree           | xml.tree, json.tree, ...   | AST

## JSON AST

```
{
  "user": {}
}
```

```
{
  "type" : "Object",
  "children" : [
    {
      "type" : "Property",
      "key" : {
        "type": "Identifier",
        "value": "user",
        "raw": "\"user\""
      }
      "value": {
        "type": "Object",
	"children": []
      }
    }
  ]
}
```

## AST Tree

```tree
user
	name \Jin
	age 35
	hobby
		\kendo 🐱‍👤
		\dance 🕺🏽
		\role play 🎭
```

```tree
user
	name \Jin
	age 35
	hobby
		\kendo 🐱‍👤
		\dance 🕺🏽
		\role play 🎭
```

## Свойства узла Tree

```typescript
interface $mol_tree {
	
	type : string
	data : string
	sub : $mol_tree[]

	baseUri : string   // https:/\/example.org
	row : number       // 30
	col : number       // 5

	value : string
	uri : string       // https://example.org#30:5 
	
}
```

# String <=> Tree <=> JSON

```
interface $mol_tree {

	static fromString( str : string , baseUri? : string ) : $mol_tree
	static fromJSON( str : string , baseUri? : string ) : $mol_tree
	
	constructor( fields : Partial< $mol_tree > )

	toString() : string
	toJSON() : string

}
```

# Поддержка редакторами

- [VSCode](https://github.com/nin-jin/vscode-language-tree)
- [Atom](https://github.com/nin-jin/atom-language-tree)
- [Sublime](https://github.com/yurybikuzin/Smol-sublime)
- [SynWrite](http://www.uvviewsoft.com/synwrite/)

# Поддержка языками

- [TypeScript](https://github.com/eigenmethod/mol/tree/master/tree)
- [D](https://github.com/nin-jin/tree.d)

# Итоги

|                           | XML  | JSON     | YAML | TOML | Tree     
|---------------------------|------|----------|------|------|----------
| Размер                    | 195% | 140%     | 125% | 110% | **100%** 
| Число паттернов           | 90   | 30       | 210  | 90   | **10**
| Удобочитаемость           | ❌  | ❌       | ✅  | ✅   | ✅
| Не нужно экранирование    | ❌  | ❌       | ❌  | ❌   | ✅
| Точные координаты ошибки  | ❌  | ❌       | ❌  | ❌   | ✅
| Поточная обработка        | ❌  | ❌       | ✅  | ✅   | ✅
| Расширяемая модель данных | ✅  | ❌       | ✅  | ❌   | ✅
| Широкая распросранённость | ✅  | ✅       | ❌  | ❌   | ❌

# Идеи

- Tree представление иных форматов при редактировании
- Tree как формат логирования
- Tree как формат общения консольных утилит
- Tree представление LISP-подобного языка
- Tree как универсальный AST

## access.log.tree - логи доступа

```
193.34.12.132 - - [2011-10-20T12:46:08+04:00] "GET /index.html HTTP/1.1" 200 4435

```

```tree
access
	ip \193.34.12.132
	time \2011-10-20T12:46:08+04:00
	request \GET /index.html HTTP/1.1
	response \200
	size \4435
```

## tree-tools - убийца PowerShell

```
> cat access.log.tree | table ip time request

193.34.12.132	2011-10-20T12:46:08+04:00	GET /index.html HTTP/1.1
193.34.12.132	2011-10-20T12:46:10+04:00	GET /index.css HTTP/1.1
193.34.12.132	2011-10-20T12:46:20+04:00	GET /index.js HTTP/1.1


> cat access.log.tree | filter time >= 2019-09 | table ip request

193.34.12.132	GET /index.html HTTP/1.1
193.34.12.132	GET /index.css HTTP/1.1
193.34.12.132	GET /index.js HTTP/1.1
```

[hyoo-ru/mill](https://github.com/hyoo-ru/mill/)

## jack.tree - LISP без скобочек

```lisp
(cdr (butlast '(1 2 3 4)))
```

```tree
cut-head
	cut-tail
		tree
			1
			2
			3
			4
```

[$mol_jack](https://github.com/eigenmethod/mol/tree/master/jack)

## Единый AST чтобы правиль всеми

![Стандарты](https://habrastorage.org/webt/9i/nr/cn/9inrcnhwl6ijiyfj88wqb1bvh1q.png)

[$mol_tree](https://github.com/eigenmethod/mol/tree/master/jack)

# Куда пойти, куда податься

- Эти слайды: [nin-jin/slides/tree](https://github.com/nin-jin/slides/tree/master/tree)
- Всё о Tree: [nin-jin/tree.d](https://github.com/nin-jin/tree.d)
- Чат о языках: [lang_idioms](https://teleg.run/lang_idioms)

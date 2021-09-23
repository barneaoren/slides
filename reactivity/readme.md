# Разбираемся в сортах реактивности

Сравним различные подходы к реактивному программированию. Вытащим на поверхность их подводные камни. И разберём как реактивность решает или наоборот усугубляет проблемы.

Это - текстовая расшифровка выступления на [SECON.Weekend Frontend](https://secon.ru/projects/seconweekend). Вы можете [прочитать как статью](https://github.com/nin-jin/slides/tree/master/reactivity), либо [открыть в интерфейсе проведения презентаций](https://nin-jin.github.io/slides/reactivity/).

# Виды активностей

- Интерактивность
- Реактивность

## Интерактивность

- Система выполнила только то, что просили.
- И ждёт дальнейших команд.

## Реактивность

- Система выполнила то, что просили.
- Плюс сама обновила всё приложение.

# Что нужно для реактивности?

- Состояния
- Акции
- Реакции
- Инварианты
- Каскад
- Рантайм

## Состояния

![](reactivity-states.svg)

## Акции

![](reactivity-actions.svg)

## Реакции

![](reactivity-reactions.svg)

## Инварианты

![](reactivity-invariants.svg)

## Каскад

![](reactivity-cascade.svg)

## Рантайм

![](reactivity-runtime.svg)

# Paradigm: Парадигма

- 🧐 PP: Процедурная
- 🤓 OOP: Объектная
- 🤯 FP: Функциональная

## 🧐 PP: Процедурная парадигма

Эпизодически запускается процедура обновления, которая что-то читает, что-то пишет. Простейшая рализация..

```javascript
let Name = 'Jin'
let Count
let Short

setInterval( ()=> { Count = Name.length } )
setInterval( ()=> { Short = Count < 5 } )
```

Примерно так с рядом оптимизаций работают Angular и Svelte. 

## 🤓 OOP: Объектная парадигма
## 🤯 FP: Функциональная парадигма

# Origin: Кто инициатор обновления?

- 📮 Push: Зависимость проталкивает
- 🚂 Pull: Зависимый затягивает

## 📮 Push: Зависимость проталкивает

```javascript
class State {
	
	@mem
	get Name() { return 'Jin' }
	set Name( Name ) { this.Count = Name.length }
	
	@mem
	set Count( Count ) { this.Short = Count < 5 }
	
	@mem
	set Short( Short ) {}
	
}
```

## 🚂 Pull: Зависимый затягивает

```javascript
class State {
	
	@mem
	get Name() { return 'Jin' }
	
	@mem
	get Count() { return this.Name.length }
	
	@mem
	get Short() { return this.Count < 5 }
	
}
```

# Observing: Наблюдение

- 🔗 Observers: Список подписчиков
- 🎆 Events: Возникновение события
- 🔭 Polling: Периодическая сверка

## 🔗 Observers: Список подписчиков
## 🎆 Events: Возникновение события
## 🔭 Polling: Периодическая сверка

Состяния хранят лишь значения и всё. Рантайм периодически сверяет текущее значение с предыдущим. И если они отличаются - пушит в зависимые состояния новые значения. Так работает Angular.

# Energetic: Энергичность реакций

- ❌ Instant: Мгновенные
- ⭕ Defer: Отложенные
- ✅ Lazy: Ленивые

# Order: Порядок реакций

- ⭕ Subscribe: По времени подписки
- ❌ Event: По времени возникновения события
- ✅ Code: По положению в программе

# Consistency: Согласованность состояния

- ✅ Strong: Гарантированнная
- ⭕ Eventual: В конечном счёте
- ❌ Relaxed: Не гарантированна

# Error: Поведение в исключительных ситуациях

- ❌ Unstable: Нестабильная работа
- ❌ Stop: Прекращение работы
- ✅ Store: Запоминание ошибки и ожидание восстановления
- ✅ Rollback: Откат к стабильному состоянию

# Recursion: Циклические зависимости

- 🔁 Allow: Допустимы
- ⛔ Fail: Приводят к ошибке
- 🚫 Impossible: Невозможны

# DataFlow: Конфигурация потоков данных

- 💪 Manual: Ручная
- 🚕 Auto: Автоматическа

# Реактивные библиотеки

| Lib        | Paradigm | Origin    | Observing       | Energetic    | Order         | Consistency | Error        | DataFlow
|------------|----------|-----------|-----------------|--------------|---------------|-------------|--------------|----------
| RxJS       | 🤯 FP   | 📮 Push   | 🔗 Observers ❓ | ❌ Instant   | ⭕ Subscribe | ⭕ Eventual | ❌ Stop     | 💪 Manual
| MobX       | 🤓 OOP  | 🚂 Pull    | 🎆 Events      | ✅ Lazy      | ✅ Code      | ✅ Strong   | ✅ Store    | 🚕 Auto
| $mol_atom2 | 🤓 OOP  | 🚂 Pull    | 🔗 Observers   | ✅ Lazy      | ✅ Code      | ✅ Strong   | ✅ Store    | 🚕 Auto
| CellX      | 🤓 OOP  | 🚂 Pull    | 🎆 Events      |               |              |             |              | 🚕 Auto
| Reatom     |          |           |                 | ✅ Lazy      |              | ✅ Strong    | ✅ Rollback  | 💪 Manual
| Effector   |          | 📮 Push   |                 | ❌ Instant   |              |              |               | 💪 Manual

# Реактивные фреймворки

| Lib        | Paradigm | Origin    | Observing       | Energetic    | Order         | Consistency | Error        | DataFlow
|------------|----------|-----------|-----------------|--------------|---------------|-------------|--------------|----------
| React      | 🧐 PP   | 📮 Push   | 🔭 Polling      | ⭕ Defer    | ✅ Code       |             |              | 💪 Manual
| Angular    | 🧐 PP   | 📮 Push   | 🔭 Polling      | ⭕ Defer    | ✅ Code ❓    | ❌ Relaxed  | ❌ Unstable | 🚕 Auto
| Vue        | 🤓 OOP  | 🚂 Pull   | 🔗 Observers ❓ | ✅ Lazy     |               |             |              | 🚕 Auto
| Svelte     | 🧐 PP   | 📮 Push   | 🔭 Polling      | ⭕ Defer    |               |             |              | 🚕 Auto

# Конфигурации зависимостей

## Последовательность

## Много зависимостей

## Много зависимых

## Переключение зависимостей

## Переключение зависимых

# Паразитные вычисления

## Нерелевантные вычисления

## Серия обновлений одного состояния

## Одновременное обновление разных состояний

## Обновление на эквивалентные данные

## Отсутствие изменений

# Нелинейная деградация

# Паразитные побочные эффекты

## Временный откат

## Множественная реакция на одно действие

## Непредсказуемый порядок реакций

## Ошибки доступа

# Эпичная битва

*большая табличка*

## Награждение победителя

## Что ещё глянуть

https://github.com/artalar/state-management-specification

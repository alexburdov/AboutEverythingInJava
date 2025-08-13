### Жизненный цикл контекста Spring-а

Жизненный цикл контекста состоит из этапов:
- Этап обновления (refresh) - автоматический
- Этап запуска (start) - вызывается методом ApplicationContext#start
- Этап остановки (stop) - вызывается методом ApplicationContext#stop
- Этап закрытия (close) - автоматический

#### Этап обновления контекста

- BeanFactory создает BeanFactoryPostProcessor-ы используя конструктор без аргументов
- ApplicationContext вызывает метод BeanFactoryPostProcessor#postProcessBeanFactory
- BeanFactory creates BeanPostProcessor-ы
- ApplicationContext регистрирует BeanPostProcessor-ы
- Инициализация singleton bean-компонентов. Подробности в Жизненный цикл bean-компонента.
- ApplicationContext проверяет флаг SmartLifecycle#isRunning и вызывает метод SmartLifecycle#start, если флаг имеет значение false
- ApplicationContext публикует ContextRefreshedEvent
- Методы обратного вызова, помеченные аннотацией @EventListener с типом параметра метода ContextRefreshedEvent, обрабатывают это событие. Также здесь может быть ApplicationListener

#### Этап запуска контекста

- ApplicationContext проверяет флаг Lifecycle#isRunning и вызывает метод Lifecycle#start, если флаг имеет значение false
- ApplicationContext проверяет флаг SmartLifecycle#isRunning и вызывает метод SmartLifecycle#start, если флаг имеет значение false. Да-да, контекст второй раз проходиться по объектам реализующие интерфейс SmartLifecycle
- ApplicationContext публикует ContextStartedEvent
- Методы обратного вызова, помеченные аннотацией @EventListener с типом параметра метода ContextStartedEvent, обрабатывают это событие. Также здесь может быть ApplicationListener

#### Этап остановки контекста

- ApplicationContext проверяет флаг SmartLifecycle#isRunning и вызывает метод SmartLifecycle#stop, если флаг имеет значение true
- ApplicationContext проверяет флаг Lifecycle#isRunning и вызывает метод Lifecycle#stop, если флаг имеет значение true
- ApplicationContext публикует ContextStoppedEvent
- Методы обратного вызова, помеченные аннотацией @EventListener с типом параметра метода ContextStoppedEvent, обрабатывают это событие. Также здесь может быть ApplicationListener

#### Этап закрытия контекста

- ApplicationContext публикует ContextClosedEvent
- Методы обратного вызова, помеченные аннотацией @EventListener с типом параметра метода ContextClosedEvent, обрабатывают это событие. Также здесь может быть ApplicationListener
- ApplicationContext проверяет флаг SmartLifecycle#isRunning и вызывает метод SmartLifecycle#stop, если флаг имеет значение true
- ApplicationContext проверяет флаг Lifecycle#isRunning и вызывает метод Lifecycle#stop, если флаг имеет значение true
- Уничтожение bean-компонентов. Подробности в Жизненный цикл bean-компонента

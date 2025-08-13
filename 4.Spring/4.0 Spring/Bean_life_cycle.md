### Жизненный цикл bean-компонента

Жизненный цикл bean-компонента состоит из этапов:
- Этап инициализации
- Этап уничтожения

#### Этап инициализации bean-компонента
- BeanFactory создает bean-компонент
- Срабатывает статический блок инициализации
- Срабатывает не статический блок инициализации
- Внедрение зависимостей на основе конструктора
- Внедрение зависимостей на основе setter-ов
- Отрабатывают методы стандартного набора *Aware интерфейсов
- BeanPostProcessor#postProcessBeforeInitialization обрабатывает bean-компонент
- InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization вызывает методы обратного вызова, помеченные аннотацией @PostConstruct
- BeanFactory вызывает метод InitializingBean#afterPropertiesSet
- BeanFactory вызывает метод обратного вызова, зарегистрированный как initMethod
- BeanPostProcessor#postProcessAfterInitialization обрабатывает bean-компонент

#### Этап уничтожения bean-компонента
- InitDestroyAnnotationBeanPostProcessor.postProcessBeforeDestruction вызывает методы обратного вызова, отмеченные как@PreDestroy
- BeanFactory вызывает метод InitializingBean#destroy
- BeanFactory вызывает метод обратного вызова, зарегистрированный как destroyMethod





---
title: В чём преимущество Dependency Injection (DI) & Inversion of Control (IoC) в Spring
date: 2024-01-14 19:15:00 +0300
categories: [Познавательное]
tags: [заметки, junior]
---

Одной из ключевых особенностей, которую предложил в своё время Spring Framework была концепция инверсии контроля
(_Inversion of Control - IoC_) и внедрение зависимостей (_Dependency Injection - DI_). Эти механизмы позволяют более
гибко управлять созданием объектов и их жизненным циклом. Благодаря чему уменьшается связность между компонентами и
улучшается их тестируемость.

Давайте рассмотрим на примерах эти концепции и сравним их с вариантом, без применения IoC и DI.

## Снижение связности между компонентами (DI & IoC)

Рассмотрим небольшой пример с сервисом заказов (OrderService), которому для выполнения логики по обработке заказа
требуется вызвать другой сервис - инвентаризационный сервис (InventoryService)

**Без использования Spring, DI и IoC:**

```java
public class OrderService {
  private InventoryService inventoryService = new InventoryServiceImpl();

  public void processOrder(Order order) {
    if (inventoryService.isAvailable(order.getItem())) {
      // process the order
    }
  }
}
```

В примере выше `OrderService` сильно связан с конкретной реализацией `InventoryService`. Если мы захотим
изменить `InventoryServiceImpl` на другую реализацию (например `MyNewInventoryService`), это потребует изменения кода в
классе `OrderService`, что
нарушает [принцип открытости/закрытости](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B8%D0%BD%D1%86%D0%B8%D0%BF_%D0%BE%D1%82%D0%BA%D1%80%D1%8B%D1%82%D0%BE%D1%81%D1%82%D0%B8/%D0%B7%D0%B0%D0%BA%D1%80%D1%8B%D1%82%D0%BE%D1%81%D1%82%D0%B8).

**С использованием Spring, DI и IoC:**

```java
public class OrderService {
  private InventoryService inventoryService;

  @Autowired
  public OrderService(InventoryService inventoryService) {
    this.inventoryService = inventoryService;
  }

  public void processOrder(Order order) {
    if (inventoryService.isAvailable(order.getItem())) {
      // process the order
    }
  }
}
```

В данном варианте `OrderService` опирается на интерфейс `InventoryService`, а конкретная реализация будет инъецирована
посредством Spring-контейнера (используя IoC & DI). Такой вариант позволяет очень просто подменить
реализацию `InventoryServiceImpl` на любую другую без изменений кода внутри `OrderService`, т.е. Spring сам найдёт в
своём контексте новую реализацию `InventoryService` и инъецирует её в `OrderService`.

> Небольшое уточнение, в данном случае подразумевается замена `InventorySerivceImpl` на другую реализацию, т.е. в один
> момент времени существует всего одна реализация `InventoryService`.
> {: .prompt-info }

## Улучшение тестирования

**Без Spring, DI и IoC:**

Если мы захотим протестировать `OrderService`, то мы столкнёмся с трудностями, т.к. в тестируемом
классе `OrderServiceTest` нам придётся напрямую инициировать `InventoryServiceImpl`, что в свою очередь усложняет
изоляцию тестируемого сервиса от внутренних компонентов, т.к. инициация `InventoryServiceImpl` заставит инициировать
также и его зависимости (если они есть).

```java
public class OrderServiceTest {

  @Test
  public void testProcessOrder() {
    /*
     * В тесте инициируется реальный класс,
     * следовательно, тест может упасть из-за проблем внутри InventoryServiceImpl,
     * что, в свою очередь, усложняет процесс тестирования
     */
    InventoryService inventoryService = new InventoryServiceImpl();
    OrderService orderService = new OrderService(inventoryService);

    orderService.processOrder(new Order(...));

    // Asserts and verifications
    verify(mockInventoryService).isAvailable(any(Item.class));
  }
}
```

**С использованием Spring, DI и IoC:**

Используя DI можно "замокировать" зависимость на `InventoryService`, таким образом при написании теста мы можем
фокусироваться только на поведении `OrderService`, не задумываясь о реализации `InventoryService`.
Такой вариант улучшает дальнейшую поддерживаемость теста и уменьшает его сложность.

```java
public class OrderServiceTest {

  @Test
  public void testProcessOrder() {
    InventoryService mockInventoryService = mock(InventoryService.class);
    when(mockInventoryService.isAvailable(any(Item.class))).thenReturn(true);

    OrderService orderService = new OrderService(mockInventoryService);
    orderService.processOrder(new Order(...));

    // Asserts and verifications
    verify(mockInventoryService).isAvailable(any(Item.class));
  }
}
```

Таким образом, DI и IoC важны, поскольку они способствуют созданию программного обеспечения, которое проще поддерживать,
масштабировать и адаптировать под изменения бизнес-логики, что является критически важными качествами для разработки
приложений корпоративного уровня.

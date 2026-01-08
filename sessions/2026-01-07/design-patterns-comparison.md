# 设计模式对比详解

## 1. 适配器模式 vs 装饰器模式

### 核心区别

| 维度 | 适配器模式 | 装饰器模式 |
|------|-----------|-----------|
| **目的** | 让不兼容的接口可以一起工作 | 动态地给对象添加新功能 |
| **关注点** | 接口转换 | 功能扩展 |
| **关系** | 适配器与被适配对象是"不同"的 | 装饰器与被装饰对象是"相同"的 |
| **使用时机** | 已有类接口不兼容时 | 需要动态添加功能时 |

### 代码对比

#### 适配器模式：接口转换Markdown Preview Enhanced 

```java
// 目标接口
public interface Target {
    void request();
}

// 被适配的类（接口不兼容）
public class Adaptee {
    public void specificRequest() {
        System.out.println("特殊请求");
    }
}

// 适配器：让 Adaptee 适配 Target 接口
public class Adapter implements Target {
    private Adaptee adaptee;
    
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
    
    @Override
    public void request() {
        // 转换接口：调用被适配对象的方法
        adaptee.specificRequest();
    }
}

// 使用：让不兼容的接口可以工作
Target target = new Adapter(new Adaptee());
target.request();  // 现在可以用了
```

**关键点**：适配器改变了接口，让原本不兼容的类可以一起工作。

#### 装饰器模式：功能扩展

```java
// 组件接口
public interface Component {
    void operation();
}

// 具体组件
public class ConcreteComponent implements Component {
    public void operation() {
        System.out.println("基础操作");
    }
}

// 装饰器：保持相同接口，但添加新功能
public class Decorator implements Component {
    protected Component component;
    
    public Decorator(Component component) {
        this.component = component;
    }
    
    @Override
    public void operation() {
        component.operation();  // 先执行原有功能
        // 然后添加新功能
        System.out.println("装饰器添加的功能");
    }
}

// 使用：在原有功能基础上添加新功能
Component component = new Decorator(new ConcreteComponent());
component.operation();
// 输出：
// 基础操作
// 装饰器添加的功能
```

**关键点**：装饰器保持相同接口，在原有功能基础上添加新功能。

### 实际应用对比

**适配器模式**：
- 整合第三方库：让第三方库适配现有接口
- 系统集成：新旧系统接口不兼容
- Java 示例：`Arrays.asList()` 将数组适配为 List

**装饰器模式**：
- Java I/O：`BufferedReader` 装饰 `FileReader`，添加缓冲功能
- Spring AOP：`TransactionInterceptor` 装饰方法，添加事务功能
- Servlet：`HttpServletRequestWrapper` 装饰请求对象

---

## 2. 观察者模式 vs 发布-订阅模式

### 核心区别

| 维度 | 观察者模式 | 发布-订阅模式 |
|------|-----------|-------------|
| **耦合度** | 观察者直接订阅主题，耦合度高 | 通过中间媒介，耦合度低 |
| **通信方式** | 主题直接通知观察者 | 通过消息代理传递消息 |
| **关系** | 主题知道所有观察者 | 发布者和订阅者互不知晓 |
| **灵活性** | 相对固定 | 更灵活，支持动态添加 |

### 代码对比

#### 观察者模式：直接通知

```java
// 观察者接口
public interface Observer {
    void update(String message);
}

// 主题（被观察者）
public class Subject {
    private List<Observer> observers = new ArrayList<>();
    
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    public void notifyObservers(String message) {
        // 直接通知所有观察者
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
}

// 观察者
public class ConcreteObserver implements Observer {
    private String name;
    
    public ConcreteObserver(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String message) {
        System.out.println(name + " 收到：" + message);
    }
}

// 使用
Subject subject = new Subject();
Observer observer1 = new ConcreteObserver("观察者1");
Observer observer2 = new ConcreteObserver("观察者2");

subject.attach(observer1);  // 直接注册到主题
subject.attach(observer2);

subject.notifyObservers("消息");
// 主题直接调用观察者的 update 方法
```

**关键点**：主题直接维护观察者列表，直接调用观察者方法。

#### 发布-订阅模式：通过中间媒介

```java
// 事件总线（中间媒介）
public class EventBus {
    private Map<String, List<Subscriber>> subscribers = new HashMap<>();
    
    // 订阅
    public void subscribe(String topic, Subscriber subscriber) {
        subscribers.computeIfAbsent(topic, k -> new ArrayList<>())
                   .add(subscriber);
    }
    
    // 发布
    public void publish(String topic, String message) {
        List<Subscriber> topicSubscribers = subscribers.get(topic);
        if (topicSubscribers != null) {
            for (Subscriber subscriber : topicSubscribers) {
                subscriber.onMessage(message);
            }
        }
    }
}

// 订阅者接口
public interface Subscriber {
    void onMessage(String message);
}

// 发布者
public class Publisher {
    private EventBus eventBus;
    
    public Publisher(EventBus eventBus) {
        this.eventBus = eventBus;
    }
    
    public void publish(String topic, String message) {
        // 发布到事件总线，不知道谁会收到
        eventBus.publish(topic, message);
    }
}

// 订阅者
public class ConcreteSubscriber implements Subscriber {
    private String name;
    
    public ConcreteSubscriber(String name) {
        this.name = name;
    }
    
    @Override
    public void onMessage(String message) {
        System.out.println(name + " 收到：" + message);
    }
}

// 使用
EventBus eventBus = new EventBus();

// 发布者和订阅者都只和事件总线交互，互不知晓
Publisher publisher = new Publisher(eventBus);
Subscriber subscriber1 = new ConcreteSubscriber("订阅者1");
Subscriber subscriber2 = new ConcreteSubscriber("订阅者2");

// 订阅者订阅主题
eventBus.subscribe("news", subscriber1);
eventBus.subscribe("news", subscriber2);

// 发布者发布消息
publisher.publish("news", "新闻消息");
// 通过事件总线传递，发布者和订阅者解耦
```

**关键点**：发布者和订阅者通过中间媒介（事件总线）通信，彼此解耦。

### 实际应用对比

**观察者模式**：
- Java 的 `Observer` 和 `Observable`（已废弃）
- GUI 事件处理：按钮点击事件
- Model-View 架构：模型变化通知视图

**发布-订阅模式**：
- 消息队列：RabbitMQ、Kafka
- Spring 事件机制：`ApplicationEventPublisher`
- 事件驱动架构：微服务间通信

---

## 3. 策略模式 vs 工厂模式

### 核心区别

| 维度 | 策略模式 | 工厂模式 |
|------|---------|---------|
| **目的** | 封装算法，让算法可以互换 | 封装对象创建过程 |
| **关注点** | 如何使用对象（行为） | 如何创建对象（创建） |
| **时机** | 运行时选择算法 | 创建时选择对象类型 |
| **关系** | 策略对象被使用 | 工厂对象创建产品 |

### 代码对比

#### 策略模式：选择算法

```java
// 策略接口（算法）
public interface PaymentStrategy {
    void pay(double amount);
}

// 具体策略
public class AlipayStrategy implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("支付宝支付：" + amount);
    }
}

public class WechatPayStrategy implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("微信支付：" + amount);
    }
}

// 上下文：使用策略
public class PaymentContext {
    private PaymentStrategy strategy;  // 持有策略对象
    
    public PaymentContext(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;  // 可以动态切换策略
    }
    
    public void executePayment(double amount) {
        strategy.pay(amount);  // 使用策略
    }
}

// 使用：关注的是如何使用策略
PaymentContext context = new PaymentContext(new AlipayStrategy());
context.executePayment(100.0);  // 使用支付宝策略

context.setStrategy(new WechatPayStrategy());
context.executePayment(100.0);  // 切换到微信策略
```

**关键点**：策略模式关注的是**如何使用**算法，可以在运行时切换不同的算法。

#### 工厂模式：创建对象

```java
// 产品接口
public interface PaymentProcessor {
    void process(double amount);
}

// 具体产品
public class AlipayProcessor implements PaymentProcessor {
    public void process(double amount) {
        System.out.println("支付宝处理：" + amount);
    }
}

public class WechatPayProcessor implements PaymentProcessor {
    public void process(double amount) {
        System.out.println("微信处理：" + amount);
    }
}

// 工厂：创建产品
public class PaymentProcessorFactory {
    public static PaymentProcessor createProcessor(String type) {
        if ("ALIPAY".equals(type)) {
            return new AlipayProcessor();  // 创建对象
        } else if ("WECHAT".equals(type)) {
            return new WechatPayProcessor();  // 创建对象
        }
        throw new IllegalArgumentException("不支持的类型");
    }
}

// 使用：关注的是如何创建对象
PaymentProcessor processor = PaymentProcessorFactory.createProcessor("ALIPAY");
processor.process(100.0);  // 使用创建的对象
```

**关键点**：工厂模式关注的是**如何创建**对象，封装了对象创建的逻辑。

### 组合使用示例

```java
// 工厂创建策略对象
public class PaymentStrategyFactory {
    public static PaymentStrategy createStrategy(String type) {
        if ("ALIPAY".equals(type)) {
            return new AlipayStrategy();
        } else if ("WECHAT".equals(type)) {
            return new WechatPayStrategy();
        }
        throw new IllegalArgumentException("不支持的类型");
    }
}

// 使用：工厂创建策略，上下文使用策略
PaymentStrategy strategy = PaymentStrategyFactory.createStrategy("ALIPAY");
PaymentContext context = new PaymentContext(strategy);
context.executePayment(100.0);
```

**关键点**：工厂模式负责**创建**策略对象，策略模式负责**使用**策略对象。

### 实际应用对比

**策略模式**：
- Java 的 `Comparator`：不同的排序策略
- Spring 的 `ResourceLoader`：不同的资源加载策略
- 算法选择：快速排序、归并排序等

**工厂模式**：
- Spring 的 `BeanFactory`：创建 Bean 对象
- JDBC 的 `DriverManager`：创建数据库连接
- 日志框架：创建不同的日志记录器

---

## 总结对比表

| 模式对比 | 适配器 | 装饰器 | 观察者 | 发布-订阅 | 策略 | 工厂 |
|---------|--------|--------|--------|----------|------|------|
| **主要目的** | 接口转换 | 功能扩展 | 直接通知 | 解耦通信 | 算法封装 | 对象创建 |
| **关注点** | 兼容性 | 扩展性 | 通知机制 | 消息传递 | 行为选择 | 创建过程 |
| **耦合度** | 中等 | 低 | 高 | 低 | 低 | 低 |
| **使用时机** | 接口不兼容 | 动态添加功能 | 状态变化通知 | 解耦通信 | 算法切换 | 对象创建 |

---

## 记忆技巧

1. **适配器 vs 装饰器**：
   - 适配器：让**不同**的东西可以一起工作（接口转换）
   - 装饰器：给**相同**的东西添加功能（功能扩展）

2. **观察者 vs 发布-订阅**：
   - 观察者：直接打电话通知（直接耦合）
   - 发布-订阅：通过中介传递消息（解耦）

3. **策略 vs 工厂**：
   - 策略：关注**如何使用**（行为）
   - 工厂：关注**如何创建**（创建）


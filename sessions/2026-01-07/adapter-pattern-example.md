# 适配器模式示例代码

## 场景：整合第三方支付接口

### 问题：接口不兼容

```java
// 现有系统使用的支付接口
public interface PaymentProcessor {
    void pay(double amount);
    void refund(String orderId);
}

// 现有实现
public class AlipayProcessor implements PaymentProcessor {
    public void pay(double amount) {
        System.out.println("支付宝支付：" + amount);
    }
    public void refund(String orderId) {
        System.out.println("支付宝退款：" + orderId);
    }
}

// 新的第三方支付 SDK（接口不兼容）
public class WechatPaySDK {
    // 方法名不同，参数类型不同
    public void makePayment(int money) {  // 注意：参数是 int，不是 double
        System.out.println("微信支付：" + money);
    }
    public void processRefund(String transactionId) {  // 参数名不同
        System.out.println("微信退款：" + transactionId);
    }
}
```

### 解决方案：适配器模式

```java
// 适配器：让 WechatPaySDK 适配 PaymentProcessor 接口
public class WechatPayAdapter implements PaymentProcessor {
    private WechatPaySDK wechatPaySDK;
    
    public WechatPayAdapter(WechatPaySDK wechatPaySDK) {
        this.wechatPaySDK = wechatPaySDK;
    }
    
    // 适配 pay 方法
    @Override
    public void pay(double amount) {
        // 转换参数类型：double -> int
        int money = (int) amount;
        // 调用第三方 SDK 的方法
        wechatPaySDK.makePayment(money);
    }
    
    // 适配 refund 方法
    @Override
    public void refund(String orderId) {
        // 参数名不同，但可以直接传递
        wechatPaySDK.processRefund(orderId);
    }
}

// 使用方式
public class PaymentService {
    public void processPayment() {
        // 使用适配器，让第三方 SDK 适配现有接口
        PaymentProcessor processor = new WechatPayAdapter(new WechatPaySDK());
        
        // 现在可以像使用其他支付处理器一样使用
        processor.pay(100.50);
        processor.refund("ORDER123");
        
        // 系统代码不需要修改，只需要添加适配器
    }
}
```

## 适配器模式的两种实现方式

### 1. 对象适配器（推荐）- 使用组合

```java
public class WechatPayAdapter implements PaymentProcessor {
    private WechatPaySDK adaptee;  // 被适配的对象
    
    public WechatPayAdapter(WechatPaySDK adaptee) {
        this.adaptee = adaptee;
    }
    
    @Override
    public void pay(double amount) {
        adaptee.makePayment((int) amount);
    }
}
```

### 2. 类适配器 - 使用继承（Java 不支持多继承，不常用）

```java
// 如果 WechatPaySDK 是类，可以继承它
// 但 Java 不支持多继承，所以这种方式不常用
```

## 实际应用场景

1. **整合第三方库**：让第三方库适配现有接口
2. **系统重构**：新旧系统接口不兼容时
3. **版本兼容**：新版本 API 适配旧版本接口
4. **数据格式转换**：XML 适配器、JSON 适配器等


# 建造者模式示例代码

## 问题场景：构建复杂的用户对象

### 传统方式的问题（使用构造函数）

```java
// 问题：参数太多，容易出错，可读性差
public class User {
    private String name;
    private int age;
    private String email;
    private String phone;
    private String address;
    private boolean isVip;
    private String company;
    
    // 构造函数参数太多，难以维护
    public User(String name, int age, String email, String phone, 
                String address, boolean isVip, String company) {
        this.name = name;
        this.age = age;
        this.email = email;
        this.phone = phone;
        this.address = address;
        this.isVip = isVip;
        this.company = company;
    }
    
    // 使用起来很困难
    User user = new User("张三", 25, "zhangsan@example.com", "13800138000",
                        "北京市", true, "ABC公司");
    // 参数顺序容易搞错，可读性差
}
```

### 建造者模式解决方案

```java
// 1. 产品类
public class User {
    private final String name;        // 必填
    private final int age;            // 必填
    private final String email;       // 可选
    private final String phone;        // 可选
    private final String address;     // 可选
    private final boolean isVip;      // 可选
    private final String company;     // 可选
    
    // 私有构造函数，只能通过 Builder 创建
    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
        this.phone = builder.phone;
        this.address = builder.address;
        this.isVip = builder.isVip;
        this.company = builder.company;
    }
    
    // Getter 方法
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getEmail() { return email; }
    // ... 其他 getter
    
    // 2. 建造者类（内部静态类）
    public static class Builder {
        // 必填参数
        private final String name;
        private final int age;
        
        // 可选参数（有默认值）
        private String email = "";
        private String phone = "";
        private String address = "";
        private boolean isVip = false;
        private String company = "";
        
        // 必填参数通过构造函数传入
        public Builder(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        // 可选参数通过链式调用设置
        public Builder email(String email) {
            this.email = email;
            return this;  // 返回自身，支持链式调用
        }
        
        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public Builder address(String address) {
            this.address = address;
            return this;
        }
        
        public Builder isVip(boolean isVip) {
            this.isVip = isVip;
            return this;
        }
        
        public Builder company(String company) {
            this.company = company;
            return this;
        }
        
        // 构建方法
        public User build() {
            // 可以在这里进行参数验证
            if (name == null || name.isEmpty()) {
                throw new IllegalArgumentException("姓名不能为空");
            }
            if (age < 0 || age > 150) {
                throw new IllegalArgumentException("年龄无效");
            }
            return new User(this);
        }
    }
}

// 3. 使用方式
public class UserService {
    public void createUser() {
        // 链式调用，可读性强，参数清晰
        User user = new User.Builder("张三", 25)
            .email("zhangsan@example.com")
            .phone("13800138000")
            .address("北京市")
            .isVip(true)
            .company("ABC公司")
            .build();
        
        // 只设置必填参数
        User simpleUser = new User.Builder("李四", 30)
            .build();
        
        // 灵活组合
        User vipUser = new User.Builder("王五", 28)
            .email("wangwu@example.com")
            .isVip(true)
            .build();
    }
}
```

## 建造者模式的优势

1. **可读性强**：链式调用，参数含义清晰
2. **灵活性高**：可选参数可以任意组合
3. **参数验证**：在 build() 方法中统一验证
4. **不可变对象**：可以创建 final 字段的对象
5. **易于扩展**：新增参数只需在 Builder 中添加方法


## 设计模式 —— 构建者

而Builder模式则是兼具了构造函数的线程安全性和JavaBeans可读性优点。其主要原理是在类的内部构造一个内部类——Builder类，Builder类通过类似set的方法对参数进行初始化，最后调用build()方法创建其所属类的新对象并将其自身返回给这个新对象，一次性完成构造工作。

**代码如下：**

User.java:

```
public class User {

    private String id;             // id(必填)
    private String name;         // 用户名(必填)
    private String email;         // 邮箱(可选)
    private int age;             // 年龄(可选)
    private String phoneNumber; // 电话(可选)
    private String address;     // 地址(可选)

    /****
     * 构建器
     */
    public static class  Builder {

        private String id;            // id(必填)
        private String name;          // 用户名(必填)
        private String email;        // 邮箱(可选)
        private int age;            // 年龄(可选)
        private String phoneNumber;// 电话(可选)
        private String address;    // 地址(可选)

        public Builder(String id,String name) {
            super();
            this.id = id;
            this.name = name;
        }

        public  Builder age(int age) {
            this.age = age;
            return this;
        }
        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder phoneNumber(String phoneNumber) {
            this.phoneNumber = phoneNumber;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }
        public User builer() {
            return new User(this);
        }
    }

    private User(Builder builder){
        this.id = builder.id;
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.phoneNumber = builder.phoneNumber;
        this.address = builder.address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                ", phoneNumber='" + phoneNumber + '\'' +
                ", address='" + address + '\'' +
                '}';
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

**测试代码如下：**

```
    public static void main(String[] args) {
        User user = new User.Builder(UUID.randomUUID().toString(),"")
                .address("aaa")
                .age(20)
                .email("sss@ewe")
                .builer();
        System.out.println(user.toString());
    }
    
结果：

User{id='c3427d2a-5e0e-4546-af0e-d715c1392423', name='', email='sss@ewe', age=20, phoneNumber='null', address='aaa'}

```

### 总结

构建对象时，如果碰到类有很多参数——其中很多参数类型相同而且很多参数可以为空时，我更喜欢Builder模式来完成。当参数数量不多、类型不同而且都是必须出现时，通过增加代码实现Builder往往无法体现它的优势。在这种情况下，理想的方法是调用传统的构造函数。再者，如果不需要保持不变，那么就使用无参构造函数调用相应的set方法吧。


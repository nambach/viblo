Lập trình khai báo (declarative programming) là một kĩ thuật mang lại rất nhiều lợi ích: code ngắn gọn hơn, dễ thay đổi, dễ bảo trì và mở rộng.

Java là một ngôn ngữ thuần túy OOP theo hướng lập trình mệnh lệnh (imperative programming), "trường phái" ngược lại của declarative. Nhưng kể từ Java 8 với lambda expression, Java đã có thể tham gia vào cuộc chơi này.

Trong bài viết này, mình sẽ giới thiệu một số kĩ thuật để code Java theo phong cách declarative.

> Một "chân lý" quan trọng chính là cố gắng tận dụng `List` và `Map` nhiều nhất có thể.

# 1. Sử dụng `List` thay cho `if else`

Trong trường hợp cần so sánh nhiều điều kiện, mà các điều kiện ấy hoặc cùng AND `&&` hoặc cùng OR `||`, thì chúng ta có thể suy nghĩ việc sử dụng `List` để gom các phép so sánh lại.

Xét ví dụ sau đây, kiểm tra một đối tượng xem nó có phải số nguyên hay không.
```java
public boolean isInteger(Object o) {
    if (o instanceof Byte || o instanceof Short ||
        o instanceof Integer || o instanceof Long) {
        return true;
    }
    return false;
}
```

Chúng ta có thể thay thế bằng `List` như sau:
```java
static final List<Class<?>> INT = Arrays.asList(
    Byte.class, Short.class, Integer.class, Long.class);

public boolean isInteger(Object obj) {
    return INT.stream().anyMatch(aClass -> aClass.isInstance(obj));
}
```

Code lúc này sẽ dễ mở rộng hơn. Giả sử muốn thêm/bớt 1 điều kiện thì chỉ cần sửa `List INT`, thay vì phải "đụng chạm" đến code thực thi. Hơn nữa, chỉ với một chút refactor, chúng ta có thể khiến đoạn code trên tái sử dụng với các loại data khác.

```java
static List<Class<?>> INT = Arrays.asList(Byte.class, Short.class, Integer.class, Long.class);
static List<Class<?>> DECIMAL = Arrays.asList(Float.class, Double.class);
static List<Class<?>> DATE = Arrays.asList(Date.class, LocalDate.class, Calendar.class);

public static <T> boolean isInstanceOf(Collection<Class<?>> classes, T obj) {
    return classes.stream().anyMatch(aClass -> aClass.isInstance(obj));
}
...

boolean isInteger = isInstanceOf(INT, value);
boolean isDecimal = isInstanceOf(DECIMAL, value);
[...]
```

**Bonus:**

> Nếu là phép `||` thì ta dùng `.anyMatch()`, nếu là phép `&&` thì ta dùng `.allMatch()`.

# 2. Sử dụng `Predicate`, `Function`, `Consumer` thay cho method

Lợi ích lớn nhất khi dùng lambda expression, chính là việc có thể gán method vào 1 đối tượng và lưu vào `List` hoặc `Map`. (Đây là đặc tính [function as first-class citizen](https://en.wikipedia.org/wiki/First-class_function) trong lập trình hàm)


Xét ví dụ sau, đối tượng `Account` là một tài khoản trong ngân hàng.

```java
class Account {
    String owner;
    int balance;
}

static void addMoney(Account acc, int amount) {
    acc.balance += amount;
}

static void subtractMoney(Account acc, int amount) {
    if (acc.balance < amount) {
        System.out.println("Balance not enough.");
        return;
    }
    acc.balance -= amount;
}

static void sendOwnerNotification(Account acc) {
    System.out.println("Your current balance is " + acc.balance);
}
```

Lúc này khi thực hiện thao tác cộng/trừ số dư, hoặc thông báo số dư hiện tại, ta có thể viết như sau:

```java
public static void doTransaction1(Account account) {
    addMoney(account, 100);
    addMoney(account, 500);
    subtractMoney(account, 200);
    sendOwnerNotification(account);
}

public static void doTransaction2(Account account) {
    addMoney(account, 200);
    sendOwnerNotification(account);
    subtractMoney(account, 100);
    addMoney(account, 800);
    addMoney(account, 2000);
    sendOwnerNotification(account);
}

...
```

Đây là cách làm truyền thống, thuần túy mệnh lệnh (imperative). Và dễ nhận thấy, các transaction đang rơi vào tình trạng **hardcode**, thay đổi 1 transaction bắt buộc phải build-deploy lại app .

Với sự ra đời của lambda expression, chúng ta có thể cải tiến bằng cách dùng `Consumer<T>`.

```java
List<Consumer<Account>> transaction1 = Arrays.asList(
    (account) -> addMoney(account, 100),
    (account) -> addMoney(account, 500),
    (account) -> subtractMoney(account, 200),
    (account) -> sendOwnerNotification(account)
);

List<Consumer<Account>> transaction2 = ...

static void doTransaction(Account account, List<Consumer<Account>> transaction) {
    for (Consumer<Account> step : transaction) {
        step.accept(account);
    }
}
```

Nhờ lambda expression mà chúng ta có thể "đóng gói" những step của 1 transaction thành list và truyền vào ở dạng param. Code trở nên ngắn gọn và có thể tái sử dụng. Thậm chí có thể cho user thực hiện một transaction do chính họ quy định bằng cách gọi API (điều mà hardcode không làm được).

# 3. Sử dụng `Map` thay cho `switch case`

Trong một số trường hợp, nếu các logic bên trong `switch case` có sự tương đồng và lặp lại, chúng ta có thể cân nhắc sử dụng `Map`.

Xét ví dụ sau, có 3 lựa chọn thanh toán khi đăng kí membership ở 1 website. 

```java
enum Policy {
    MONTHLY,
    YEARLY,
    LIFE_TIME
}
```

Mỗi lựa chọn có 1 mức giá khác nhau. Code xử lý thanh toán có thể viết đại loại như sau.

```java
switch (policy) {
    case MONTHLY:
        doPayment(account, 17.5);
        break;
    case YEARLY:
        doPayment(account, 180);
        break;
    case LIFE_TIME:
        doPayment(account, 1000);
        break;
    default:
        // do something
}
```

Chúng ta refactor bằng cách sử dụng `Map`.
```java
static Map<Policy, Consumer<Account>> options = new HashMap<>();
static {
    options.put(MONTHLY, acc -> doPayment(acc, 17.5));
    options.put(YEARLY, acc -> doPayment(acc, 180));
    options.put(LIFE_TIME, acc -> doPayment(acc, 100));
}
...

Consumer<Account> option = options.get(policy);
if (option != null) {
    option.accept(account);
} else {
    // default do something
}
```

Dễ dàng nhận thấy, cũng như ví dụ 1 và 2, chúng ta cố gắng tách logic nghiệp vụ từ dạng **code** sang dạng **constant** và khiến phần code xử lý abstract nhất có thể.

Bạn có thể nói rằng số lượng line of code không có nhiều sự khác biệt. Nhưng một trong những ưu điểm của việc sử dụng `Map`, là ta có thể **kiểm tra đối tượng dạng object** như `Class<?>`, trong khi `switch` bị giới hạn ở bốn kiểu data là số nguyên, kí tự, String và enum. (Hiện tại Java 17 đã hỗ trợ điều này với tính năng pattern matching dành cho `switch`)

Xét ví dụ sau, khi ghi dữ liệu ra file Excel với thư viện Apache POI. Thông thường, chúng ta sẽ phải dùng *if else* bởi *switch* không hỗ trợ `Class<T>`.

```java
Class<?> type = cellValue.getClass();

if (type.equals(long.class) || type.equals(Long.class)) {
    cell.setCellValue((long) cellValue);

} else if (type.equals(double.class) || type.equals(Double.class)) {
    cell.setCellValue((double) cellValue);

} else if (type.equals(Date.class)) {
    cell.setCellValue((Date) cellValue);
    
} else if ...
```

Nỗi sợ hãi khi maintain tỉ lệ thuận với số lượng if-else có trong code. Hãy cùng refactor lại bằng `Map`.

```java
static Map<Class<?>, BiConsumer<Cell, Object>> handlers = new HashMap<>();
static {
    handlers.put(Long.class, (cell, val) -> cell.setCellValue((long) val));
    handlers.put(long.class, (cell, val) -> cell.setCellValue((long) val));
    handlers.put(Date.class, (cell, val) -> cell.setCellValue((Date) val));
    handlers.put(Double.class, (cell, val) -> cell.setCellValue((double) val));
    [...]
}
...

BiConsumer<Cell, Object> handler = handlers.get(cellValue.getClass());
if (handler != null) {
    handler.accept(cell, cellValue);
}
```

Mỗi lần muốn thêm một loại dữ liệu mới, chỉ cần sửa phần constant `handlers`, chứ không cần "đụng chạm" vào logic xử lý, giúp cho phần code này luôn sạch sẽ gọn gàng.

# Nhận xét

3 kĩ thuật được nêu trên đều có một đặc điểm chung, đó là chúng ta luôn cố gắng **tách dữ liệu nghiệp vụ** từ dạng *code* ở phần body sang dạng *constant* (input param). Lợi ích lớn nhất của điều này là *ngăn ngừa hardcode* nghiệp vụ. Chúng ta có thể tách và lưu nghiệp vụ xuống DB, mỗi lần thay đổi chỉ cần cập nhật DB mà không cần compile/build lại app.

Ngoài ra, code xử lý cũng ngắn gọn và sạch sẽ hơn.

# Kết

Trên đây là một vài kinh nghiệm đúc kết của mình. Các kĩ thuật này hoàn toàn có thể áp dụng với những ngôn ngữ khác, chứ không duy nhất cho Java. Declarative programming là một chủ đề thú vị và vẫn còn nhiều thứ để nghiên cứu. 

> Mình sẽ không gọi là Functional Programming (FP), bởi Java vốn dĩ rất nặng về OOP và vẫn còn ở rất xa với tiêu chuẩn của FP. FP có nhiều pattern cao cấp (curry, recursion) mà đôi khi áp dụng không cẩn thận có thể khiến code trở nên khó maintain hơn thay vì OOP. Chỉ dừng ở [cấp độ Declarative](https://en.wikipedia.org/wiki/Programming_paradigm) là đủ rồi.

Hẹn gặp lại các bạn trong những bài viết tiếp theo.

© 2021 Nam Bach.  All rights reserved.
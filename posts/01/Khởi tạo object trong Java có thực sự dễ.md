# Giới thiệu

Khởi tạo object trong Java, một vấn đề cơ bản nhưng có khá nhiều khía cạnh để phân tích. Hãy cùng nhau điểm qua một vài phương pháp: sử dụng constructor/static method, pattern builder, annotation builder. Và cuối cùng, bài viết sẽ giới thiệu một hướng tiếp cận mới bằng cách sử dụng chaining method + lambda expression.

# Sử dụng Constructor

Cách đơn giản nhất: tạo constructor có số lượng argument tương ứng. Khi muốn thay đổi argument, chúng ta có thể thay đổi constructor hiện tại (cách này không an toàn lắm), hoặc tạo thêm constructor mới. Khi đó class của chúng ta sẽ xuất hiện khá nhiều constructor mà không rõ ngữ cảnh sử dụng.

```java
public Book(String isbn, String title) {
    this.isbn = isbn;
    this.title = title;
}

public Book(String isbn, String title, String author) {
    this.isbn = isbn;
    this.title = title;
    this.author = author;
}

public Book(String isbn, String title, String author, String subCategory, Category category) {
    this.isbn = isbn;
    this.title = title;
    this.author = author;
    this.subCategory = subCategory;
    this.category = category;
}
```

Để giải quyết việc này, chúng ta có thể chuyển sang static method.

```java
public static Book quickInit(String isbn, String title) {
    Book book = new Book();
    book.isbn = isbn;
    book.title = title;
    return book;
}

public static Book fullInit(String isbn, String title, String author, String subCategory, Category category) {
    Book book = new Book();
    book.isbn = isbn;
    book.title = title;
    book.author = author;
    book.subCategory = subCategory;
    book.category = category;
    return book;
}
```

# Sử dụng pattern/annotation Builder

Tuy nhiên, việc dùng constructor vẫn kém linh hoạt trong trường hợp chúng ta muốn tùy biến số lượng argument. Để giải quyết vấn đề đó, chúng ta có thể sử dụng [pattern Builder](https://refactoring.guru/design-patterns/builder) (chỉ phù hợp khi business logic của bạn thật sự phức tạp), hoặc đơn giản hơn là dùng [annotation @Builder](https://projectlombok.org/features/Builder) của Lombok. Cách này cho phép ta hoàn toàn tùy biến số lượng argument muốn khởi tạo.

```java
Book book = Book.builder()
    .title("Sapiens: A Brief History of Humankind")
    .author("Yuval Noah Harari")
    .subCategory("History")
    .build();
```

Bạn có thể thắc mắc, tại sao phải cầu kì như vậy, có thể dùng constructor rỗng và setter là xong mà? Hãy thử nghĩ đến trường hợp bạn chỉ muốn khai báo trong một dòng duy nhất như dưới đây:

```java
List<Book> nonFictions = Arrays.asList(
    Book.builder().title("Sapiens: A Brief History of Humankind").author("Yuval Noah Harari").build(),
    Book.builder().title("The Defining Decade").author("Meg Jay").build(),
    Book.builder().title("The State of Affairs").author("Esther Perel").build()
);
```

## Đặt vấn đề

Vẫn còn một vài bất tiện khi sử dụng annotation Builder. Đó là khi class của chúng ta có sử dụng generic type parameter.

Xem xét ví dụ sau, chúng ta muốn xuất dữ liệu của một danh sách các object bất kì dưới dạng bảng.

![Xuất data dưới dạng bảng](https://images.viblo.asia/3d4a9fa3-ded0-4557-9a64-58dfef9a2916.png)

Ta sẽ định nghĩa các `Column` của bảng tương ứng với mỗi field bên trong object.

```java
public class Column<T> {
    String title;
    String fieldName;
    Function<T, ?> customExtractor;
}
```

Khi đó một Table sẽ là list các `Column` của đối tượng mà ta muốn xuất dữ liệu.

```java
Table<Book> = [
  Column("Book ID", "isbn"),
  Column("Name", "title"),
  Column("Category", book -> book.category.name)
]
```

Đoạn mã giả này mô tả một table của class `Book`, với cột đầu là ID của cuốn sách lấy từ field `isbn`, cột hai là tên cuốn sách lấy từ field `title`, và cột cuối là thể loại sách lấy từ `name` của `category` của cuốn sách, chúng ta sử dụng một function để chỉ định cách lấy giá trị cột này.

Dưới đây là code tương ứng trong Java nếu áp dụng [annotation @Builder](https://projectlombok.org/features/Builder).

```java
List<Column<Book>> bookTable = Arrays.asList(
    Column.<Book>builder().title("Book ID").fieldName("isbn").build(),
    Column.<Book>builder().title("Name").fieldName("title").build(),
    Column.<Book>builder().title("Category").customExtractor(book -> book.getCategory().getName()).build()
);
```

Để ý thấy, chúng ta luôn phải thêm type `<Book>` mỗi lần gọi `.builder()`, nếu không compiler sẽ không nhận diện được `T` trong `Function<T, ?> customExtractor` chính xác là class nào.

# Giải pháp: Function as a builder
Chúng ta sẽ cần sửa lại class `Column` một chút để *bắt chước* tính năng của [annotation @Builder](https://projectlombok.org/features/Builder), bằng cách **khai báo chaining method ngay trong class gốc**.

(Chaining method: một hàm trả về object đã gọi nó - `return this`)

```java
public class Column<T> {
    String title;
    String fieldName;
    Function<T, ?> customExtractor;

    public Column<T> title(String title) {
        this.title = title;
        return this;
    }

    public Column<T> fieldName(String fieldName) {
        this.fieldName = fieldName;
        return this;
    }

    public Column<T> customValue(Function<T, ?> customExtractor) {
        this.customExtractor = customExtractor;
        return this;
    }
}
```

Để compiler có khả năng resolve class `T` tại thời điểm runtime, chúng ta sẽ tạo 1 static method nhận vào một  `Function<Column<T>, Column<T>>` - đây chính là **yếu tố mấu chốt** của bài toán khởi tạo object. 

```java
public class Column<T> {
    ...

    public static <T> Column<T> add(Function<Column<T>, Column<T>> builder) {
        return builder.apply(new Column<>());
    }
}
```

Function `builder` sẽ đảm nhiệm việc chuyển đổi object rỗng `new Column<>()` thành object có đầy đủ các tham số khởi tạo. Code của chúng ta sẽ có kết quả như sau:

```java
List<Column<Book>> bookTables = Arrays.asList(
    add(c -> c.title("Book ID").fieldName("isbn")),
    add(c -> c.title("Name").fieldName("title")),
    add(c -> c.title("Category").customValue(book -> book.getCategory().getName()))
);
```

Code đã sạch đẹp, dễ đọc, gần như không có yếu tố dư thừa trùng lặp, gần giống với đoạn mã giả ban đầu. Thật thú vị, đúng không?

## Đánh giá

**1.** Việc áp dụng kĩ thuật **chaining method** (bắt chước từ [annotation @Builder](https://projectlombok.org/features/Builder)) cho phép chúng ta khởi tạo object với số lượng argument tùy ý mà không cần khai báo nhiều constructor khác nhau.

**2.** Sử dụng **`Function<Entity, Entity> builder`** giúp chúng ta loại bỏ từ khóa `new`
- Không còn phải lặp lại những dòng `new` khó chịu như `new Class()`, `new Class<Type>()`, `new Class<>()` hay `Class.<Type>init()`. Bản thân function đã tự động resolve generic type tại thời điểm runtime.
- Khi dùng static method để khởi tạo object, chúng ta luôn phải tốn một khoản không gian kí tự cho tên class và tên static method khởi tạo: `LongClassName.init()...`
- Trong khi đó, nhờ vào *lambda expression*, chúng ta chỉ phải tốn nhiều nhất 6 kí tự cho việc gọi hàm khởi tạo: **`o -> o...`** Bạn hoàn toàn có thể tự thay đổi tên param input của function để nó gợi nhớ hơn, hoặc đơn giản, cứ để ngắn gọn như vậy, bởi tên của method invoke đã quyết định ngữ cảnh rồi (như ví dụ `Column.add()` ở trên)

# Kết
Sau khi tham khảo tư liệu để hỗ trợ cho bài viết này, mình thấy 1 trang có đề cập các [design pattern hiện đại dành cho Java 8](https://jaxenter.com/patterns-java-8-lambdas-127635.html) bằng cách áp dụng lambda expression, tuy nhiên những kĩ thuật ở trang này cũng không thật sự giống với cách mình đang sử dụng.

Mình tình cờ tìm ra pattern này trong lúc code library hỗ trợ đọc/ghi Excel theo phong cách khai báo - mình là một fan cuồng  declarative programming. Nếu bạn có hứng thú, ghé qua [Github](https://github.com/nambach) của mình cũng như [dự án Excel](https://github.com/nambach/ExcelUtil) mà mình đang làm nhé.

Tất nhiên, đây không phải là một phát hiện mới mẻ gì - bạn có thể tham khảo dự án [SpringFu, JaFu](https://github.com/spring-projects-experimental/spring-fu/tree/main/jafu) của đội ngũ Spring, để xem cách họ apply pattern này. Mình nghĩ đây là một pattern khá tiện lợi mà bạn có thể áp dụng cho những dự án trong tương lai.

## Liên kết ngoài
1.  Pattern Builder: https://refactoring.guru/design-patterns/builder
2.  Project Lombok - annotation Builder: https://projectlombok.org/features/Builder
3.  Creational patterns with Java 8: https://jaxenter.com/patterns-java-8-lambdas-127635.html
4.  Project SpringFu: https://github.com/spring-projects-experimental/spring-fu/tree/main/jafu
5.  Dự án của mình: https://github.com/nambach/ExcelUtil
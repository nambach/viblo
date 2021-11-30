Khởi tạo object trong Java, một vấn đề cơ bản nhưng có khá nhiều khía cạnh để phân tích.

Hãy cùng nhau điểm qua một vài phương pháp: sử dụng constructor/static method, pattern builder, annotation builder. Và cuối cùng, bài viết sẽ giới thiệu một hướng tiếp cận mới bằng cách sử dụng *chaining method + lambda expression*.

# Phương pháp truyền thống
## 1. Sử dụng Constructor

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

Để giải quyết vấn đề ngữ cảnh, chúng ta có thể chuyển sang static method.

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

## 2. Sử dụng annotation Builder

Tuy nhiên, việc dùng constructor vẫn kém linh hoạt trong trường hợp chúng ta muốn tùy biến số lượng argument. Để giải quyết vấn đề đó, chúng ta có thể sử dụng [annotation @Builder](https://projectlombok.org/features/Builder) của Lombok. Cách này cho phép ta hoàn toàn tùy biến số lượng argument muốn khởi tạo.

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

## 3. Vấn đề với generic type

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

Đoạn mã giả (pseudo-code) này mô tả một table của class `Book`, với cột đầu là ID của cuốn sách lấy từ field `isbn`, cột hai là tên cuốn sách lấy từ field `title`, và cột cuối là thể loại sách lấy từ `name` của `category` của cuốn sách, chúng ta sử dụng một function để chỉ định cách lấy giá trị cột này.

Dưới đây là code tương ứng trong Java nếu áp dụng [annotation @Builder](https://projectlombok.org/features/Builder).

```java
List<Column<Book>> bookTable = Arrays.asList(
    Column.<Book>builder().title("Book ID").fieldName("isbn").build(),
    Column.<Book>builder().title("Name").fieldName("title").build(),
    Column.<Book>builder().title("Category").customExtractor(book -> book.getCategory().getName()).build()
);
```

Để ý thấy, chúng ta luôn phải thêm type `<Book>` mỗi lần gọi `.builder()` (đây gọi là chỉ định type tường minh - explicit type), nếu không compiler sẽ không nhận diện được `T` trong `Column<T>` chính xác là class nào. 

Hơn nữa, dòng nào cũng phải gọi ra 2 lệnh `.builder()` và `.build()`.

# Giải pháp: Function as a builder

Đây là một giải pháp độc đáo nhưng lại rất đơn giản.

Chúng ta sẽ không sử dụng Lombok Builder. Thay vào đó, chúng ta sửa lại class `Column` để nó ***bắt chước hành vi chaining method*** của annotation Builder.

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

Để compiler có khả năng resolve class `T` tại thời điểm runtime, chúng ta sẽ tạo 1 static method nhận vào một  `Function<Column<T>, Column<T>>`. Đây là **yếu tố then chốt** của bài toán khởi tạo object. 

```java
public class Column<T> {
    ...

    public static <T> Column<T> create(Function<Column<T>, Column<T>> builder) {
        return builder.apply(new Column<>());
    }
}
```
_(Có thể thay thế `Function<Column<T>, Column<T>>` bằng `UnaryOperator<Column<T>>` cho ngắn gọn)_

Function `builder` sẽ đảm nhiệm việc chuyển đổi object rỗng `new Column<>()` thành object có đầy đủ các tham số khởi tạo.

Code của chúng ta sẽ có kết quả như sau:

```java
List<Column<Book>> bookTables = Arrays.asList(
    create(c -> c.title("Book ID").fieldName("isbn")),
    create(c -> c.title("Name").fieldName("title")),
    create(c -> c.title("Category").customValue(book -> book.getCategory().getName()))
);
```

Code đã gọn gàng sạch sẽ, không dư thừa trùng lặp, gần giống với đoạn pseudo-code ban đầu.

# Kết luận

**1.** Việc áp dụng kĩ thuật **chaining method** (bắt chước từ [annotation @Builder](https://projectlombok.org/features/Builder)) cho phép chúng ta khởi tạo object với số lượng argument tùy ý mà không cần khai báo nhiều constructor khác nhau.

**2.** Sử dụng **`Function<Entity, Entity> builder`** giúp chúng ta loại bỏ từ khóa `new`
- Không còn phải lặp lại những dòng `new` khó chịu như `new Class()`, `new Class<Type>()`, `new Class<>()` hay `Class.<Book>init()`.
- `Function` giúp tự động resolve generic type tại thời điểm runtime, không cần phải chỉ định explicit type.
- Nhờ *lambda expression*, chúng ta chỉ tốn nhiều nhất 6 kí tự cho việc gọi hàm khởi tạo: **`o -> o...`** Bạn hoàn toàn có thể tự thay đổi tên param input của function để nó gợi nhớ hơn, hoặc đơn giản, cứ để ngắn gọn như vậy, bởi tên của method invoke đã quyết định ngữ cảnh rồi (như ví dụ `Column.create()` ở trên)

# Liên kết ngoài
Nếu các bạn cảm thấy pattern này thú vị, thì có thể tham khảo các project bên dưới để xem nó mang lại sự tiện lợi như thế nào.

- **ExcelUtil** library: github.com/nambach/ExcelUtil
- **SpringFu** (from Spring team): github.com/spring-projects-experimental/spring-fu/tree/main/jafu

Hẹn gặp lại các bạn trong những bài viết sau.

© 2021 Nam Bach.  All rights reserved.
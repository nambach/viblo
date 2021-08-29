# Giới thiệu

Trong [bài viết trước](https://viblo.asia/p/co-mot-noi-so-mang-ten-timezone-Qbq5Q6MXKD8), mình đã giới thiệu các bạn những khái niệm cơ bản liên quan đến ngày giờ & múi giờ. Bài viết này mình sẽ phân tích một số lỗi thường gặp trong thực tế.

# Nhắc lại các khái niệm

Trước tiên chúng ta sẽ ôn lại các khái niệm cơ bản
- **moment**: thời gian tuyệt đối
- **rtime** (relative/represent time): thời gian tương đối, hoặc cũng có thể gọi là thời gian chỉ để hiển thị
- **offset**: độ lệch UTC
- **zone**: múi giờ

## **moment** - thời gian tuyệt đối

Là một khoảng khắc cụ thể (moment) trong dòng thời gian.

> Ví dụ: Khoảng khắc Việt Nam vô địch AFF Cup vào lúc 19:30 ngày 15/12/2018

Khi biểu diễn, cần có đủ hai thành phần: **ngày giờ** + **ngữ cảnh nơi chốn**. Ngữ cảnh ở đây chính là múi giờ (zone). Các múi giờ được đặc trưng bởi một độ lệch thời gian (offset) so với giờ phối hợp quốc tế UTC. Độ lệch được biểu diễn dưới dạng `±hh:mm`.

> Việt Nam thuộc múi giờ Đông Dương (Indochina Time - ICT) có độ lệch `UTC+07:00`

Trong máy tính, moment được biểu diễn dưới dạng [Epoch Time](https://vi.wikipedia.org/wiki/Th%E1%BB%9Di_gian_Unix) - số giây trôi qua kể từ 00:00:00 ngày 1 tháng 1 năm 1970 theo giờ UTC.

## **rtime** - thời gian hiển thị

Là thời gian không kèm múi giờ hay độ lệch UTC, chỉ mang tính tương đối, không dùng để đối chiếu so sánh.

> - Tiết học bắt đầu lúc 12h45
> - Ca làm việc bắt đầu lúc 15h
> - Các ngày lễ, ngày kỉ niệm hằng năm

Khi thêm zone hoặc offset vào rtime, chúng ta sẽ có một moment.
```
moment = rtime + (zone or offset)
```

# Một số vấn đề thường gặp

## 1. Bạn có đang chọn *đúng class* để xử lý?

Việc chọn nhầm kiểu dữ liệu có thể bắt nguồn từ việc nhầm lẫn giữa rtime và moment - bạn muốn xử lý thời gian dạng rtime nhưng lại chọn kiểu moment và ngược lại.

Chúng ta cùng xem lại danh sách các class ngày giờ trong Java tương ứng với 2 loại thời gian.

| Loại thời gian | Java Class |
| ----------- | ----------- |
| *rtime* | (Java 1.8) `LocalDate`, `LocalTime`, `LocalDateTime` |
| *moment* | `java.util.Date`, `Calendar`<br>(sql) `java.sql.Date`, `java.sql.Time`, `java.sql.Timestamp`<br> (Java 1.8) `Instant`, `OffsetDateTime`, `ZonedDateTime` |

Chỉ từ phiên bản 1.8 mới hỗ trợ lưu thời gian dạng rtime. Bạn có thể thắc mắc, vậy Java 1.7 trở về trước thì xử lý kiểu gì? Nhiều người sẽ nhận ra, trước đây họ không có ý niệm về *rtime* hay *moment* - họ luôn dùng `Date` cho mọi trường hợp, nếu có dính đến timezone thì dùng `Calendar`, nếu có thời gian biểu lặp đi lặp lại thì dùng `String` và `SimpleDateFormat` - họ làm như vậy mà vẫn cứ đúng đấy thôi.

Tất nhiên đó là trước khi Java 1.8 ra mắt. Kể từ phiên bản 1.8, với thiết kế của bộ API DateTime mới, chúng ta sẽ cần thay đổi một chút cách suy nghĩ về ngày giờ - bằng việc phân loại *rtime/moment*.

1. Mọi thao tác so sánh đối chiếu thời gian, nên quy tất cả về *moment*, cụ thể ở đây là `Instant`.
2. Muốn tạo một mốc thời gian cụ thể tại một múi giờ cụ thể (moment), chúng ta bắt đầu từ việc khởi tạo *rtime* với `LocalDateTime` (parse từ chuỗi string hoặc từ những số day, month, year, hour:min:sec), sau đó gắn vào đó một zone hay một offset.
(Xem lại ở [bài viết trước](https://viblo.asia/p/co-mot-noi-so-mang-ten-timezone-Qbq5Q6MXKD8#_2-ap-dung-vao-code-9))

## 2. Bạn có đang chọn đúng class để *lưu trữ*?

Cho dù đã phân biệt được 2 loại thời gian và chọn đúng class để thao tác, chúng ta vẫn có khả năng *chọn nhầm kiểu lưu trữ* bên dưới database.

Trong Java, chúng ta sử dụng JDBC - [Java Database Connectivity](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/) - một API cho phép hệ thống tương tác với database. Dưới đây là bảng mapping kiểu dữ liệu tổng quát của JDBC *(Nguồn [Oracle](https://docs.oracle.com/cd/E19830-01/819-4721/beajw/index.html))*

| Java Type | JDBC Type |
| ----------- | ----------- |
| `java.util.Date` | `DATE` (Oracle only) <br> `TIMESTAMP` (all other databases) |
| `java.sql.Date` | `DATE` |
| `java.sql.Time` | `TIME` |
| `java.sql.Timestamp` | `TIMESTAMP` |


> Có thể bạn sẽ bắt đầu bối rối, "DATE, TIME, TIMESTAMP - ủa cái này quen, hồi đó mình có gặp rồi nè, nhưng mà sao có cảm giác nó không ăn nhập với 2 khái niệm rtime/moment nhỉ?"

Well, bình tĩnh nhé. Thực ra nó... còn phức tạp hơn như vậy. Ứng với mỗi Database Provider (Postgres, MySQL...), chúng ta phải tuân theo một kiểu mapping khác nhau.

Mặc dù vậy, việc đưa về hệ quy chiếu *rtime/moment* cũng không quá khó khăn. Mình sẽ làm thử với Postgres và MySQL.

### Postgres: TIMESTAMP hay TIMESTAMPZ ?

Postgres cung cấp 2 kiểu timestamp là có timezone (TIMESTAMPZ) và không timezone (TIMESTAMP). Thoạt nhìn chúng ta sẽ cho rằng timestamp luôn là moment. Tuy nhiên, [Postgres Tutorial](https://www.postgresqltutorial.com/postgresql-timestamp/) giải thích:

> - TIMESTAMP: lưu ngày và giờ, nhưng không chứa thông tin múi giờ. Khi thay đổi múi giờ của database server, giá trị của timestamp giữ nguyên không đổi.
> - TIMESTAMPZ: là một **zone-aware** timestamp - khi một moment được lưu xuống, Postgres sẽ convert về UTC và lưu theo kiểu TIMESTAMP ở trên. Mỗi khi đọc lên, Postgres sẽ lấy TIMESTAMP đó convert về múi giờ của server và trả về cho backend. Nói một cách ngắn gọn, khi lưu xuống ngày giờ ở timezone nào thì đọc lên vẫn là ngày giờ đó, ở timezone đó.

Như vậy đã rõ ràng, TIMESTAMP chính là *rtime*, còn TIMESTAMPZ chính là *moment*. Tài liệu của [JDBC Driver for Postgres](https://jdbc.postgresql.org/documentation/head/java8-date-time.html) đưa ra bảng mapping như sau:

| PostgreSQL™ | Java SE 8 |
| ----- | -----|
| DATE | LocalDate |
| TIME | LocalTime |
| TIMESTAMP | LocalDateTime |
| TIMESTAMPZ | OffsetDateTime |

Có thể thấy, cách mapping kiểu dữ liệu của Postgres hoàn toàn khớp với hệ quy chiếu *rtime/moment*.

*(Chúng ta không dùng kiểu `ZonedDateTime` để mapping TIMESTAMPZ, bởi `ZonedDateTime` có chứa logic xử lý Daylight Saving Time - trong khi bản chất TIMESTAMPZ không chứa thông tin như vậy)*

> "Ê nhưng mà theo ông nói, `Date` là moment, nếu TIMESTAMP là rtime, vậy tui map `Date` với TIMESTAMP thì nó lưu xuống DB kiểu gì?"

Để trả lời câu hỏi này, chúng ta phải hiểu bản chất của `Date` - bản chất nó là moment (hiển nhiên, vì nó chứa Epoch Time), nhưng cũng có thể xem nó là *rtime* - Java tự động convert `Date` về timezone của hệ thống.

```java
    System.out.println(new Date());             // Sun Aug 29 11:05:37 ICT 2021
    System.out.println(ZonedDateTime.now());    //2021-08-29T11:05:37.119+07:00[Asia/Bangkok]
```

Do vậy, khi dùng `Date` để mapping với TIMESTAMP, ở thời điểm lưu xuống DB, `Date` được đưa về timezone của backend, phần rtime hiển thị sẽ được lưu vào TIMESTAMP. Ở thời điểm đọc lên, giá trị TIMESTAMP đó sẽ được gắn vào timezone của backend.

> "Ủa, vậy sao trước giờ tui sử dụng `Date` hoặc `java.sql.Timestamp` để mapping TIMESTAMP mà nó vẫn đúng, không bị lỗi?"

Đó là vì timezone của backend và database server ở các môi trường LOCAL, DEV và PROD luôn nhất quán với nhau. Vấn đề sẽ phát sinh nếu vô tình thay đổi timezone và phá vỡ sự nhất quán của hệ thống hiện tại.

Bạn có thể tự kiểm chứng bằng cách lưu `Date` xuống DB dạng TIMESTAMP, sau đó đổi múi giờ của backend rồi đọc lại giá trị TIMESTAMP đó ở bằng class `Date`, chắc chắn sẽ có sự sai lệch.

### MySQL: DateTime hay Timestamp ?

[Tài liệu của MySQL](https://dev.mysql.com/doc/refman/8.0/en/datetime.html) mô tả:

> Khi lưu TIMESTAMP, giá trị sẽ được convert từ timezone hiện tại về UTC. Khi đọc lên sẽ đi qua bước ngược lại. Quy trình này không áp dụng đối với kiểu DATETIME.

Như vậy dễ dàng kết luận, DATETIME chính là *rtime*, còn TIMESTAMP là *moment*. Tuy nhiên, JDBC Driver của MySQL là Connector/J có sự khác biệt so với Postgres. Để hiểu chính xác cách mapping dữ liệu, bạn có thể tham khảo [tài liệu của Connector/J](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-time-instants.html). Thêm nữa, giới hạn lưu trữ của TIMESTAMP trong MySQL là `2038-01-19 03:14:07.999999`, vốn được biết đến với tên gọi **Sự cố năm 2038** - [Year 2038 Problem](https://en.wikipedia.org/wiki/Year_2038_problem), là vấn đề xảy ra khi chúng ta lưu moment dạng Epoch Time bằng một số nguyên 32-bit (vấn đề này không gặp ở Postgres).

### Best Practices

Đối với *moment*, chúng ta sẽ theo hướng dẫn của JDBC driver (như đã phân tích ở trên) mà chọn kiểu dữ liệu phù hợp (ở trên backend cũng như ở dưới database).

Đối với *rtime* (thời gian có tính lặp lại, ví dụ: thời khóa biểu, lịch làm việc, các ngày lễ kỉ niệm...), **tốt nhất nên lưu dạng string** vì tính dễ đọc, dễ debug và không gây hiểu nhầm. Chúng ta sẽ cần thêm vài bước trung gian để parse và gắn zone/offset vào, tuy nhiên chuyện đó không tốn nhiều công sức. Nếu không dùng string, thì hãy đảm bảo tuân theo hướng dẫn từ nhà phát triển JDBC driver.


## 3. Bạn có đang ***thiết kế*** giao diện đúng cách?

Vâng, bạn không nghe nhầm đâu. Có phải bạn đã từng thấy một control chọn ngày như thế này đúng không?


Và chắc hẳn, bạn cũng từng thấy một control chọn cả ngày lẫn giờ như thế này rồi chứ?


Tất nhiên không có vấn đề gì với 2 chiếc datetimepicker này. Nhưng mình có một tình huống. Giả sử chúng ta đang thiết kế một ứng dụng meeting online. Ở bước chọn thời gian bắt đầu cuộc họp, ta sử dụng chiếc datetimepicker số 2. Theo bạn, ngày giờ mà user chọn có chính xác hay không?

Câu trả lời: ***hên xui***.

Phần lớn các thư viện datetimepicker sẽ trả về thời gian dựa theo múi giờ đang cài đặt trên máy client. Hãy thử đặt trường hợp user của bạn sử dụng một chiếc máy tính họ mượn của ai đó, với một múi giờ khác. Lúc này bạn sẽ không bao giờ biết liệu thời gian mà user chọn và gửi về từ frontend có đáng tin cậy hay không:

1. Khả năng thứ nhất: user không nhận ra sự khác biệt về thời gian. Họ chỉ quan tâm cái app họ đang xài mà thôi. Chắc chắn giá trị ngày giờ mà họ chọn sẽ sai lệch.

2. Khả năng thứ hai: user nhận ra sự khác biệt múi giờ ở máy tính họ đang xài. Họ có thể sửa lại cho đúng với múi giờ của họ, và sửa lại thêm một lần nữa khi trả lại chiếc máy tính đang mượn. Trường hợp không thể sửa múi giờ, họ sẽ cố gắng quy đổi về đúng giá trị họ mong muốn. Tuy nhiên, sau tất cả, họ vẫn bối rối và tự hỏi, *cái thời gian mà mình đang chọn rốt cuộc là thời gian ở múi giờ nào nhỉ?*

### Giải pháp

Hướng giải quyết của chúng ta rất đơn giản - hãy tham khảo giải pháp của những "ông lớn".


Chúng ta thấy, hệ thống của Outlook phục vụ cho user của toàn cầu, do đó việc thêm vào lựa chọn timezone cũng là chuyện dễ hiểu. Còn đối với Facebook, họ vẫn dùng loại datetimepicker truyền thống, nhưng kèm theo đó hiển thị rõ ràng giá trị thời gian mà user chọn sẽ thuộc múi giờ nào (múi giờ của máy tính họ đang xài). Điểm chung của 2 hướng tiếp cận này chính là ***tính rõ ràng*** - user sẽ chắc chắn ngày giờ họ chọn là *"như ý họ"* (có đúng múi giờ không?).

Đối với những ứng dụng chỉ dùng trong một địa phương/quốc gia, tốt nhất vẫn nên theo hướng tiếp cận như Facebook (hiển thị rõ múi giờ cho user), để tránh những hiểu nhầm như tình huống đặt ra.
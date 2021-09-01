*(Bài viết này nằm trong series ["Có một nỗi sợ mang tên DateTime"](https://viblo.asia/s/co-mot-noi-so-mang-ten-datetime-z45bx8DqZxY))*

Trong [bài viết trước](https://viblo.asia/p/co-mot-noi-so-mang-ten-timezone-Qbq5Q6MXKD8), mình đã giới thiệu các bạn những khái niệm cơ bản liên quan đến ngày giờ & múi giờ. Bài viết này mình sẽ phân tích một số lỗi thường gặp trong thực tế.

# Nhắc lại các khái niệm

Trước tiên chúng ta sẽ ôn lại các khái niệm cơ bản
- **moment**: thời gian tuyệt đối
- **rtime** (relative/represent time): thời gian tương đối, hoặc cũng có thể gọi là thời gian chỉ để hiển thị
- **offset**: độ lệch UTC
- **zone/timezone**: múi giờ

## **moment**

Là một khoảng khắc cụ thể trong dòng thời gian lịch sử.

> Ví dụ: Khoảng khắc Việt Nam vô địch AFF Cup vào lúc 19:30 ngày 15/12/2018

Khi biểu diễn, cần có đủ hai thành phần: **ngày giờ** + **ngữ cảnh nơi chốn**. Ngữ cảnh ở đây chính là múi giờ (zone). Các múi giờ được đặc trưng bởi một độ lệch thời gian (offset) so với giờ phối hợp quốc tế UTC. Độ lệch được biểu diễn dưới dạng `±hh:mm`.

> Việt Nam thuộc múi giờ Đông Dương (Indochina Time - ICT) có độ lệch `UTC+07:00`

Trong máy tính, moment được biểu diễn dưới dạng [Epoch Time](https://vi.wikipedia.org/wiki/Th%E1%BB%9Di_gian_Unix) - số giây trôi qua kể từ 00:00:00 ngày 1 tháng 1 năm 1970 theo giờ UTC.

## **rtime**

Là thời gian chỉ mang tính hiển thị (represent time, gọi tắt là rtime), không kèm múi giờ hay độ lệch, không dùng để đối chiếu so sánh.

> - Tiết học bắt đầu lúc 12h45
> - Ca làm việc bắt đầu lúc 15h
> - Các ngày lễ, ngày kỉ niệm hằng năm

Khi thêm zone hoặc offset vào rtime, chúng ta sẽ có một moment.
```
moment = rtime + (zone or offset)
```

Nếu chưa phân biệt được *rtime/moment*, bạn có thể [xem lại bài viết trước](https://viblo.asia/p/co-mot-noi-so-mang-ten-timezone-Qbq5Q6MXKD8).

# 3 sai lầm phổ biến

## 1. Bạn có đang chọn *đúng class* để xử lý?

Việc chọn nhầm kiểu dữ liệu có thể bắt nguồn từ việc nhầm lẫn giữa rtime và moment - bạn muốn xử lý thời gian dạng rtime nhưng lại chọn kiểu moment và ngược lại.

Chúng ta cùng xem lại danh sách các class ngày giờ trong Java tương ứng với 2 loại thời gian.

| Loại thời gian | Java Class |
| ----------- | ----------- |
| *rtime* | (Java 1.8) `LocalDate`, `LocalTime`, `LocalDateTime` |
| *moment* | `java.util.Date`, `Calendar`<br><br>(sql) `java.sql.Date`, `java.sql.Time`, `java.sql.Timestamp`<br><br> (Java 1.8) `Instant`, `OffsetDateTime`, `ZonedDateTime` |

Loại thời gian rtime chỉ mới xuất hiện ở Java 1.8 - vậy trước đó xử lý kiểu gì?

> Nhiều người sẽ nhận ra, trước đây họ không có ý niệm về *rtime* hay *moment* - họ luôn dùng `Date` cho mọi trường hợp, nếu có dính đến timezone thì dùng `Calendar`, nếu có thời gian biểu lặp đi lặp lại thì dùng `String` và `SimpleDateFormat` - họ làm như vậy mà vẫn cứ đúng đấy thôi.

Để lý giải chuyện này, chúng ta phải hiểu bản chất của `Date` - bản chất nó là moment (hiển nhiên rồi, vì nó chứa Epoch Time), nhưng cũng có thể xem nó là *rtime* - Java tự động convert về timezone của hệ thống.

```java
System.out.println(ZonedDateTime.now());    // 2021-08-29T11:05:37.119+07:00[Asia/Bangkok]
System.out.println(new Date());             // Sun Aug 29 11:05:37 ICT 2021
// Thời gian hiển thị của Date trùng khớp với ZonedDateTime
```

Nhiều người hay nhầm lẫn và không nắm được bản chất của `Date`, dẫn đến việc vô tình tạo ra bug khi timezone hệ thống bị thay đổi. 

Với thiết kế của bộ DateTime mới, việc xử lý thời gian sẽ trở nên *chặt chẽ hơn* bằng cách đưa về hệ quy chiếu *rtime/moment*.

1. Mọi thao tác so sánh đối chiếu thời gian, nên quy tất cả về *moment*, cụ thể là `Instant`.
2. Muốn tạo một mốc thời gian cụ thể tại một múi giờ cụ thể (moment), chúng ta bắt đầu từ việc khởi tạo *rtime* với `LocalDateTime` (parse từ chuỗi string hoặc từ những số day, month, year, hour:min:sec), sau đó gắn vào đó một timezone hay một offset. (Xem chi tiết ở [bài viết trước](https://viblo.asia/p/co-mot-noi-so-mang-ten-timezone-Qbq5Q6MXKD8#_2-ap-dung-vao-code-9))

## 2. Bạn có đang chọn đúng class để *lưu trữ*?

Cho dù đã phân biệt được 2 loại thời gian và chọn đúng class để thao tác, chúng ta vẫn có khả năng *chọn nhầm kiểu lưu trữ* bên dưới database.

Trong Java, chúng ta sử dụng JDBC - [Java Database Connectivity](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/) - một API cho phép hệ thống tương tác với database. Dưới đây là bảng mapping kiểu dữ liệu tổng quát của JDBC *(Nguồn [Oracle](https://docs.oracle.com/cd/E19830-01/819-4721/beajw/index.html))*

| Java Type | JDBC Type |
| ----------- | ----------- |
| `java.util.Date` | `DATE` (Oracle only) <br> `TIMESTAMP` (all other databases) |
| `java.sql.Date` | `DATE` |
| `java.sql.Time` | `TIME` |
| `java.sql.Timestamp` | `TIMESTAMP` |


> "DATE, TIME, TIMESTAMP - ủa cái này quen, hồi đó mình có gặp rồi nè, nhưng mà sao có cảm giác nó không ăn nhập với 2 khái niệm rtime/moment nhỉ?"

Well, bình tĩnh nhé. Thực ra nó... còn phức tạp hơn như vậy. Ứng với mỗi Database Provider (Postgres, MySQL...), chúng ta phải tuân theo một kiểu mapping khác nhau.

Mặc dù vậy, việc đưa về hệ quy chiếu *rtime/moment* lại tương đối dễ. Mình sẽ làm mẫu cho các bạn.

### Postgres: TIMESTAMP hay TIMESTAMPZ ?

Postgres cung cấp 2 kiểu timestamp là có timezone (TIMESTAMPZ) và không timezone (TIMESTAMP). Thoạt nhìn chúng ta sẽ cho rằng timestamp luôn là moment. Tuy nhiên, [Postgres Tutorial](https://www.postgresqltutorial.com/postgresql-timestamp/) giải thích:

> - TIMESTAMP: lưu ngày và giờ, nhưng không chứa thông tin timezone. Khi thay đổi timezone của database-server, giá trị của timestamp ***giữ nguyên*** không đổi.
> - TIMESTAMP**Z**: là một **zone-aware** timestamp - khi một moment được lưu xuống, Postgres sẽ convert về UTC và lưu kiểu TIMESTAMP (như trên). Khi đọc lên sẽ làm thao tác ngược lại: giá trị timestamp được convert về timezone của database-server và trả về cho backend. Nói một cách ngắn gọn, khi lưu xuống ngày giờ ở timezone nào thì đọc lên vẫn là ngày giờ đó, ở timezone đó.

Như vậy đã rõ ràng, TIMESTAMP chính là *rtime*, còn TIMESTAMP**Z** chính là *moment*. Tài liệu của [JDBC Driver for Postgres](https://jdbc.postgresql.org/documentation/head/java8-date-time.html) đưa ra bảng mapping như sau:

| PostgreSQL™ | Java SE 8 |
| ----- | -----|
| DATE | LocalDate |
| TIME | LocalTime |
| TIMESTAMP | LocalDateTime |
| TIMESTAMPZ | OffsetDateTime |

Có thể thấy, cách mapping kiểu dữ liệu của Postgres hoàn toàn khớp với hệ quy chiếu *rtime/moment*.

*(Chúng ta không dùng kiểu `ZonedDateTime` để mapping TIMESTAMPZ, bởi `ZonedDateTime` có chứa logic xử lý Daylight Saving Time - trong khi bản chất TIMESTAMPZ không chứa thông tin như vậy)*

> "Nếu `Date` là moment, TIMESTAMP là rtime, vậy tui map `Date` với TIMESTAMP thì nó lưu xuống DB kiểu gì? (Trước giờ tui vẫn hay làm như vậy á)"

Như đã giải thích trước đó, bạn phải hiểu được bản chất class `Date`. Khi dùng `Date` để mapping với TIMESTAMP, ở thời điểm lưu xuống DB, `Date` được đưa về timezone của backend, phần rtime sẽ được lưu vào Postgres. Ở thời điểm đọc lên, timestamp sẽ được gắn thêm timezone của backend trước khi gán ngược cho `Date`.

<div align="center">
    <img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/03/img/DateToTimestamp.png" />
</div>

> "Ủa, vậy sao trước giờ tui sử dụng `Date` hoặc `java.sql.Timestamp` để mapping TIMESTAMP mà nó vẫn đúng, không bị lỗi?"

Đó là vì timezone của backend và database server ở các môi trường LOCAL, DEV và PROD luôn nhất quán với nhau. Vấn đề sẽ phát sinh nếu thay đổi timezone và phá vỡ sự nhất quán của hệ thống hiện tại.

Bạn có thể tự kiểm chứng bằng cách lưu `Date` xuống Postgres dạng TIMESTAMP, sau đó đổi timezone của backend rồi đọc lại giá trị đó, chắc chắn sẽ có sự sai lệch.

### MySQL: DateTime hay Timestamp ?

*(Lưu ý là Timestamp của MySQL và Postgres không giống nhau)*

[Tài liệu của MySQL](https://dev.mysql.com/doc/refman/8.0/en/datetime.html) mô tả:

> Khi lưu Timestamp, giá trị sẽ được convert từ timezone hiện tại về UTC. Khi đọc lên sẽ đi qua bước ngược lại. Quy trình này không áp dụng đối với kiểu DateTime.

Như vậy dễ dàng kết luận, DateTime chính là *rtime*, còn Timestamp là *moment*. Tuy nhiên, JDBC Driver của MySQL là Connector/J có sự khác biệt so với Postgres. Để hiểu chính xác cách mapping dữ liệu, bạn có thể tham khảo [tài liệu của Connector/J](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-time-instants.html).

Thêm nữa, giới hạn lưu trữ của Timestamp trong MySQL là `2038-01-19 03:14:07.999999`, vốn được biết đến với tên gọi **Sự cố năm 2038** - [Year 2038 Problem](https://en.wikipedia.org/wiki/Year_2038_problem), là vấn đề xảy ra khi chúng ta lưu moment dạng Epoch Time bằng một số nguyên 32-bit (vấn đề này không gặp ở Postgres).

### Best Practices

Đối với *moment*, chúng ta sẽ theo hướng dẫn của JDBC driver mà chọn kiểu dữ liệu phù hợp (ở backend cũng như ở database).

Đối với *rtime* (thời gian có tính lặp lại, ví dụ: thời khóa biểu, lịch làm việc, các ngày lễ kỉ niệm...), nếu bạn không rành về bộ DateTime của Java 1.8 thì **tốt nhất nên lưu dạng string** vì tính dễ đọc, dễ debug và không gây hiểu nhầm. Chúng ta sẽ cần thêm vài bước trung gian để parse và gắn zone/offset vào, tuy nhiên chuyện đó không tốn nhiều công sức. Nếu không dùng string, thì hãy đảm bảo tuân theo hướng dẫn từ nhà phát triển JDBC driver.


## 3. Bạn có đang ***thiết kế*** giao diện đúng cách?

Vâng, bạn không nghe nhầm đâu. Có phải bạn đã từng thấy một control chọn ngày giờ như thế này đúng không?

<div align="center">
<img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/03/img/DateTimePicker.png" />
<br />
<span>

*[frontbackend.com](https://frontbackend.com/a-jquery-plugin-for-date-and-time-picker)*

</span>
</div>

Tất nhiên rồi, đây là một thiết kế phổ biến, không có vấn đề gì với chiếc datetimepicker này cả.

Nhưng mình có một tình huống. Giả sử chúng ta đang thiết kế một ứng dụng meeting online. Để chọn thời gian bắt đầu cuộc họp, chúng ta sử dụng datetimepicker như hình trên. Theo bạn, ngày giờ mà user chọn có *đáng tin cậy* hay không?

Câu trả lời: ***hên xui***.

Phần lớn các thư viện datetimepicker sẽ trả về thời gian dựa theo múi giờ đang cài đặt trên máy client. Thử đặt trường hợp user đang sử dụng máy tính mượn của ai đó, với một múi giờ khác. Lúc này bạn sẽ không bao giờ biết liệu thời gian mà user chọn và gửi về từ frontend có đáng tin cậy hay không:

1. Khả năng thứ nhất: user không nhận ra sự khác biệt về thời gian. Họ chỉ tin vào những thứ họ nhìn thấy trong app. Chắc chắn giá trị ngày giờ sẽ sai lệch.

2. Khả năng thứ hai: user nhận ra sự khác biệt múi giờ ở máy tính họ đang xài. Họ có thể sửa lại múi giờ cho đúng. Nếu không thể sửa (do không có quyền chẳng hạn), họ sẽ cố gắng quy đổi về đúng giá trị mong muốn. Tuy nhiên, sau tất cả, họ vẫn bối rối và tự hỏi, *"cái thời gian mà mình đang chọn rốt cuộc là giờ ở đâu nhỉ?"*

### Giải pháp

Hướng giải quyết của chúng ta rất đơn giản - hãy tham khảo giải pháp của những "ông lớn".

<div align="center">

<img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/03/img/DateTimePicker-outlook.png" />
<br />

*Màn hình tạo Meeting của Outlook*

</div>

<div align="center">

<img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/03/img/DateTimePicker-fb.png" />
<br />

*Màn hình tạo Event của Facebook*

</div>

Chúng ta thấy, hệ thống của Outlook phục vụ cho user của toàn cầu, do đó việc thêm vào lựa chọn timezone cũng là chuyện dễ hiểu. Còn đối với Facebook, họ chỉ dùng loại datetimepicker truyền thống, nhưng kèm theo đó ***hiển thị rõ ràng*** giá trị thời gian mà user chọn sẽ ***thuộc múi giờ nào*** (múi giờ của máy tính họ đang xài - như ví dụ trong hình GMT +7)

Điểm chung của 2 hướng tiếp cận này chính là ***tính rõ ràng*** của thời gian - user sẽ biết cách để chọn đúng giá trị mà họ muốn.

Đối với những hệ thống chỉ dùng trong một địa phương hoặc một quốc gia, tốt nhất nên theo hướng tiếp cận như Facebook (hiển thị rõ múi giờ cho user), để tránh những hiểu nhầm không mong muốn.

# Kết

Hy vọng bài viết này giúp các bạn giảm thiểu sai sót khi làm việc với ngày giờ và múi giờ.

Hẹn gặp lại các bạn ở những bài viết tiếp theo.

<br>

© 2021 Nam Bach.  All rights reserved.
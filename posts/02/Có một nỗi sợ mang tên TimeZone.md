Từ lâu việc xử lý thời gian đã là một chủ đề mang lại nhiều cơn đau đầu cho các developers, đặc biệt nếu phải xử lý thời gian theo nhiều [múi giờ](https://vi.wikipedia.org/wiki/M%C3%BAi_gi%E1%BB%9D) khác nhau. Bài viết này sẽ giúp bạn "đả thông kinh mạch" để thoát khỏi nỗi sợ này.

Một số thuật ngữ mình sẽ sử dụng trong bài viết:
- **moment**: thời gian tuyệt đối
- **rtime** (relative/represent time): thời gian tương đối, hoặc cũng có thể gọi là thời gian chỉ để hiển thị
- **offset**: độ lệch
- **zone**: múi giờ

# Có 2 loại thời gian

Để hiểu được bản chất vấn đề, chúng ta cần nắm rõ 2 khái niệm sau:

## 1. Thời gian tuyệt đối

Là một **khoảng khắc cụ thể (moment)** trong dòng chảy lịch sử.

> - Khoảng khắc Bác Hồ đọc tuyên ngôn độc lập năm 1945
> - Khoảng khắc Việt Nam lần đầu tiên kết nối internet vào ngày 19/11/1997
> - Khoảng khắc Việt Nam vô địch AFF Cup vào lúc 19:30 ngày 15/12/2018

Khi nói về thời gian tuyệt đối, cần có đủ hai thành phần: **ngày giờ** + **ngữ cảnh nơi chốn**.

> Đêm giao thừa năm 2021 ở Việt Nam sẽ khác với đêm giao thừa 2021 ở Florida (Mỹ). Trong lúc bạn đang đón giao thừa tại Việt Nam, thì khách hàng của bạn ở Florida vẫn đang ăn trưa - tại Florida đang là 13:00 ngày 31/12.

Ngữ cảnh ở đây chính là **múi giờ (zone)**. Các múi giờ được đặc trưng bởi một **độ lệch thời gian (offset)** so với [giờ phối hợp quốc tế UTC](https://vi.wikipedia.org/wiki/Gi%E1%BB%9D_Ph%E1%BB%91i_h%E1%BB%A3p_Qu%E1%BB%91c_t%E1%BA%BF). Độ lệch được biểu diễn dưới dạng `±hh:mm`.
> Việt Nam thuộc múi giờ Đông Dương (Indochina Time - ICT) có độ lệch `UTC+07:00` , nghĩa là đồng hồ ở Việt Nam chạy nhanh hơn 7 tiếng so với đồng hồ của UTC.

Trong máy tính, moment được biểu diễn dưới dạng [Epoch Seconds](https://vi.wikipedia.org/wiki/Th%E1%BB%9Di_gian_Unix) - số giây trôi qua kể từ 00:00:00 ngày 1 tháng 1 năm 1970 theo giờ UTC.
## 2. Thời gian tương đối

 Là thời gian **chỉ dùng để hiển thị** (*relative/represent time* - gọi ngắn gọn là *rtime*), không bao gồm ngữ cảnh múi giờ. 
 
 > - Người dân các nước đón giao thừa vào lúc 00:00 ngày 1/1 hàng năm.
 > - Ngày Quốc tế Phụ nữ là ngày 8/3
 > - Giờ đi làm bắt đầu lúc 8:00 sáng và kết thúc lúc 17:00 chiều
 
 Trong sinh hoạt thường ngày, khi muốn đối chiếu thời gian, chúng ta không thể dùng rtime, mà phải thêm vào một múi giờ hoặc một độ lệch để rtime trở nên tuyệt đối (moment) rồi mới đem đi so sánh.
 
 ` moment = rtime + (zone or offset)`
 
 > Sếp (onsite ở Nhật): - Anh em hôm nay họp lúc 4h chiều nhé. <br>
 > Bạn: - 4h chiều bên anh **JST - Japan Standard Time (UTC+9:00)** hay là bên tụi em **ICT (UTC+7:00)** ? <br>
 > Sếp: - À quên, 4h chiều bên anh, tức là 2h chiều bên tụi em đó.

# Và một vài quy ước

Sau khi đã phân biệt được rtime và moment, chúng ta sẽ tìm hiểu cách để biểu diễn chúng.

## 1. Tiêu chuẩn ISO-8601

Được công bố vào năm 1988, ISO-8601 là một tiêu chuẩn quốc tế mô tả một quy tắc chung để viết ngày giờ, tiện cho việc liên lạc & trao đổi thông tin liên quan đến thời gian.

Dưới đây là một moment được viết theo tiêu chuẩn ISO-8601, bao gồm ngày, giờ và offset

<div align="center">
    <img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/02/img/Picture2.png" />
</div>

Như vậy, để biểu diễn rtime, chúng ta chỉ cần bỏ đi phần offset. Và ngược lại, khi gắn offset vào rtime (ngày giờ), chúng ta có moment.

` moment = rtime + offset`

Để ý chúng ta thấy, moment trong tiêu chuẩn ISO-8601 chỉ sử dụng offset mà không đề cập đến tên của múi giờ.

## 2. IANA Time Zone Database
Hay còn gọi là **tz database**, là một bộ database tổng hợp thông tin của toàn bộ múi giờ trên thế giới, được quản lý bởi [tổ chức ICANN](https://en.wikipedia.org/wiki/ICANN).

Trong **tz database**, một múi giờ sẽ có tên gọi dựa trên vị trí địa lý của nó, theo dạng `Area/Location`, trong đó `area` là tên của lục địa hoặc đại dương, `location` là tên của thành phố hoặc hòn đảo.
> Múi giờ ở thành phố Hồ Chí Minh có tên là `Asia/Ho_Chi_Minh`<br>
> Múi giờ ở Auckland (New Zealand) có tên là `Pacific/Auckland`<br>
> Tuy nhiên, không phải thành phố nào cũng có múi giờ riêng. Tham khảo danh sách đầy đủ [ở đây](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

Lúc này có thể bạn sẽ hỏi
> "Ủa rồi đẻ ra thêm cái này làm gì? Sao không dùng mấy cái chữ viết tắt múi giờ (ICT) hay là offset (+7:00) gì đó á? Với hồi nãy ông mới bảo tôi, biểu diễn moment chỉ cần thêm offset vào rtime là đủ - ông lừa tôi à?"

Đúng là để biểu diễn thời gian tuyệt đối, chỉ cần thời gian tương đối (ngày giờ) và offset là đủ.

Cuộc sống sẽ cứ êm đềm như vậy, nếu không xuất hiện khái niệm **Daylight Saving Time**.

## 3. Daylight Saving Time (DST) là cái gì?

Nếu bạn chưa biết, một thành phố có thể sử dụng *2 múi giờ luân phiên* trong năm.

> Ở Victoria (Úc), vào lúc 2:00 sáng ngày chủ nhật đầu tiên của tháng 10 (bắt đầu mùa hè), toàn bộ đồng hồ địa phương sẽ được vặn để chạy nhanh thêm 1 giờ. Đến chủ nhật đầu tiên tháng 4 năm sau, đồng hồ sẽ vặn để trả lại về thời gian cũ. <br><br> Do đó, múi giờ của Victoria sử dụng có tên là [AET (Australian Eastern Time)](https://www.timeanddate.com/time/zones/aet) sẽ bao gồm cả 2 múi giờ, là giờ chuẩn **AEST (Standard) +10:00** và giờ mùa hè **AEDT  (Daylight) +11:00** <br><br> (Nguồn: [australia.com](https://www.australia.com/en/facts-and-planning/useful-tips/time-zones.html))

Tại sao lại có hiện tượng lạ này?

>[Daylight Saving Time](https://en.wikipedia.org/wiki/Daylight_saving_time) - hay còn gọi là [Quy ước giờ mùa hè / Giờ tiết kiệm ánh sáng ban ngày](https://vi.wikipedia.org/wiki/Quy_%C6%B0%E1%BB%9Bc_gi%E1%BB%9D_m%C3%B9a_h%C3%A8)  được đề xuất lần đầu năm 1784 bởi Benjamin Franklin. Ông nhận thấy vào mùa hè, trời sẽ mau sáng (*"Đêm tháng 5 chưa nằm đã sáng"*) 😎. Nếu mọi người thức dậy sớm vào mùa hè, họ có thể tận dụng nguồn ánh sáng mặt trời để làm việc vào ban ngày, và đi ngủ sớm vào ban đêm để tiết kiệm nhiên liệu đốt. Tuy nhiên, mãi đến thế kỉ 20, các quốc gia mới bắt đầu nghiêm túc quan tâm đến phương pháp này, nhằm đối phó với cuộc khủng hoảng nhiên liệu xảy ra sau 2 đợt chiến tranh thế giới. Hiện nay đã có khoảng [70 quốc gia](http://www.webexhibits.org/daylightsaving/g.html#:~:text=Today%2C%20approximately%2070%20countries%20utilize,some%20form%20of%20daylight%20saving.) áp dụng Daylight Saving vào các khu vực lãnh thổ riêng biệt.

Điều này dẫn chúng ta đến 1 sự thật "kinh hoàng": tồn tại những múi giờ mang *2 giá trị offset luân phiên nhau* trong năm 😱😱😱

Đây chính là lý do bộ dữ liệu **IANA tz database** ra đời:
- Nhằm *chuẩn hóa* lại tên gọi của các múi giờ (các múi giờ luôn viết tắt, điều này có thể gây hiểu nhầm - AST là Arabia Standard Time, Arabia Summer Time hay Atlantic Standard Time?)
- Mỗi múi giờ sẽ chứa 2 offset, một dành cho độ lệch chuẩn thông thường, một dành cho thời điểm Daylight Saving. Nếu một múi giờ không sử dụng Daylight Saving, 2 offset này có giá trị bằng nhau.

Ok, lý thuyết như vậy đủ rồi. Bây giờ chúng ta sẽ tìm hiểu code thực tế như thế nào.
# Java Date Time API

Khi xử lý thời gian, trước đây  chúng ta thường dùng `Date`, `SimpleDateFormat` và `Calendar`. Kể từ phiên bản 1.8 trở đi, Java cung cấp những API mạnh mẽ để xử lý ngày - giờ.

| **Offset/Zone** | **Thời gian tương đối (rtime)** | **Thời gian tuyệt đối (moment)** |
| -------- | -------- |-------- |
 | **`ZoneOffset`**: chứa thông tin độ lệch theo format `±hh:mm` <br><br> **`ZoneId`**: chứa tên của múi giờ theo tiêu chuẩn **tz database** | **`LocalDate`**: lưu thông tin một ngày (ngày, tháng, năm) <br><br>  **`LocalTime`**: lưu thông tin một giờ (giờ, phút, giây) với độ chính xác nanoseconds  <br><br> **`LocalDateTime`**: lưu cả ngày lẫn giờ| **`Instant`**:  biển diễn một *epoch time* với độ chính xác nanosecond. <br><br> **`OffsetDateTime`**: lưu ngày giờ và một offset (không kèm múi giờ) <br><br> **`ZonedDateTime`**: lưu ngày giờ kèm theo múi giờ (zone ID) |

<br>

Để ý ta thấy, đội ngũ Java sử dụng chữ "Local" để ám chỉ giờ địa phương. Do đó các class bắt đầu bằng chữ `Local` sẽ biểu diễn *rtime* - thời gian hiển thị của địa phương. Bản thân các class `Local` không chứa thông tin về múi giờ hoặc độ lệch.

 Ngoài ra, (có thể bạn đã biết), **`Date`** là một *moment*, thể hiện ngày giờ với TimeZone mặc định của server. Tuy nhiên do [thiếu sót trong thiết kế ban đầu](https://stackoverflow.com/questions/1969442/whats-wrong-with-java-date-time-api)  nên phần lớn method đã bị deprecated và thay thế bởi `Instant`
 
 ## 1. Diagram
 
 Dưới đây là diagram tóm tắt cách chuyển đổi qua lại giữa các class

<div align="center">
    <img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/02/img/DateTimeAPI.png">
</div>

## 2. Áp dụng vào code

Nguyên tắc chung:
- Khi cần tạo ra một thời gian moment, chúng ta sẽ tạo ra `LocalDateTime` (rtime), xác định `ZoneId` (zone + offset), và kết hợp chúng lại thành `ZonedDateTime` (moment).
- Toàn bộ thao tác so sánh - đối chiếu thời gian, ***nên*** quy đổi về `Instant` (nếu bạn là người mới dùng API này và xài chưa quen)
- Một lưu ý quan trọng: ngoại trừ class  `Date` và `Calendar`, các API phiên bản 1.8 cung cấp đều là *immutable object* (nghĩa là bạn không thể modify object hiện tại, và toàn bộ các hàm set ngày giờ đều tạo ra object mới).

Hãy cùng xem qua một vài thao tác xử lý, giả sử chúng ta muốn xuất báo cáo được tạo trong năm 2020.

```java
class Report {
    Date createdAt;
}

[...]

public void processReportIn2020() {
    ZoneId zoneId = ZoneId.of("Asia/Ho_Chi_Minh");
    // Để lấy ra moment đầu năm 2020 tại HCM, chúng ta sẽ tạo một rtime
    // tại thời điểm 0 giờ ngày 1/1, và ghép múi giờ vào
    Instant beginOfYear = LocalDateTime.parse("2020-01-01T00:00:00").atZone(zoneId).toInstant();
    // Tương tự với cuối năm
    Instant endOfYear = LocalDateTime.parse("2020-12-31T23:59:59").atZone(zoneId).toInstant();

    // Lọc ra báo cáo trong năm 2020
    List<Report> thisYearReports = allReports
            .stream()
            .filter(r -> isBetween(r.createdAt, beginOfYear, endOfYear))
            .collect(Collectors.toList());
            
    // xử lý báo cáo
    [...]
}

// Kiểm tra xem thời gian của báo cáo có nằm giữa 2 khoảng instant hay không
boolean isBetween(Date d, Instant start, Instant end) {
    Instant timePoint = d.toInstant();
    return (start.equals(timePoint) || start.isBefore(timePoint)) &&
           (timePoint.equals(end) || timePoint.isBefore(end));
}
```

Như bạn thấy, nếu đã phân biệt được rõ ràng đâu là *moment*, đâu là *rtime*, việc xử lý thời gian trở nên vô cùng đơn giản.

## 3. Một vài thao tác dịch chuyển múi giờ

Với Date Time API 1.8, việc chuyển đổi qua lại giữa các múi giờ cũng linh hoạt dễ dàng. Hãy xem xét ví dụ sau:
```java
// Để biết được múi giờ ở Florida có tên trong tz database là gì,
// chúng ta tra cứu trên trang web https://time.is
ZoneId florida = ZoneId.of("America/New_York");

// Thời gian hiện tại ở HCM (+07:00)
ZonedDateTime now = ZonedDateTime.now();
now.toLocalDateTime();  // 2021-08-22, 20:24

// Cũng là hiện tại nhưng ở Florida (-04:00)
ZonedDateTime nowAtFlorida = now.withZoneSameInstant(florida);
nowAtFlorida.toLocalDateTime(); // 2021-08-22, 09:24
```

Nhìn vào tên method `.withZoneSameInstant(ZoneId)` có thể đoán được ngay chức năng của nó là dịch chuyển từ timezone này sang timezone khác mà vẫn giữ nguyên giá trị moment ("sameInstant").

```java
String newYear = "2021-01-01T00:00:00";
ZoneId hcm = ZoneId.of("Asia/Ho_Chi_Minh");

// Giao thừa 2021 tại HCM
ZonedDateTime newYearEveHCM = LocalDateTime.parse(newYear).atZone(hcm);

// Giao thừa 2021 tại Florida
ZonedDateTime newYearEveFlorida = newYearEveHCM.withZoneSameLocal(florida);
```
Trái ngược với ví dụ trước, ở đây `.withZoneSameLocal(ZoneId)` sẽ giữa nguyên rtime ("sameLocal") và gắn vào đó một timezone khác, do đó giá trị tuyệt đối (moment) của `newYearEveHCM` và `newYearEveFlorida` hoàn toàn khác nhau.


## 4. Vậy còn Daylight Saving Time (DST)?

Thật may mắn là *chúng ta **không cần** làm gì hết!*

 JRE và JDK sẽ handle toàn bộ việc thay đổi liên quan đến DST nếu chúng ta sử dụng `ZonedDateTime` và cung cấp `ZoneId` theo tiêu chuẩn của **tz database**.

Chúng ta sẽ tái hiện lại thời điểm bắt đầu DST tại Victoria ở ví dụ đã nói ở trên, lúc 2h sáng chủ nhật đầu tiên của tháng 10 (là ngày 04/10/2020)
```java
    ZoneId zId = ZoneId.of("Australia/Melbourne"); // https://time.is/Victoria
   
   // Vì 2h sẽ xảy ra DST, chúng ta sẽ lấy thời gian sớm hơn 5 phút
    ZonedDateTime before = LocalDateTime.parse("2020-10-04T01:55:00")
                                        .atZone(zId);
```
> 01:55 &nbsp;&nbsp;&nbsp;  // before.toLocalTime(); <br>
> +10:00 &nbsp; // before.getOffset();

Sau đó cộng thêm 10 phút để qua thời điểm DST. 
```java
    ZonedDateTime after = before.plusMinutes(10);
```
> 03:05 &nbsp;&nbsp;&nbsp;  // after.toLocalTime(); <br>
> +11:00 &nbsp; // after.getOffset();

Chúng ta thấy thời gian đã tự động cộng thêm 1 tiếng, và độ dời offset cũng đã tự thay đổi từ +10:00 thành +11:00

Trong tương lai, nếu có bất kì thay đổi nào liên quan đến DST tại một địa phương, dữ liệu sẽ được cập nhật tại [website của **tz database**](https://www.iana.org/time-zones) và được chỉnh sửa trong [lần phát hành tiếp theo của JRE](https://www.oracle.com/java/technologies/tzdata-versions.html).

# Tóm lại

Chúng ta đã hiểu được khái niệm *moment* (tgian tuyệt đối) và *rtime* (tgian tương đối).
- Trong máy tính, một moment được biểu diễn bằng con số epoch_seconds - số giây trôi qua kể từ 00:00:00 ngày 1/1/1970 theo giờ UTC.
- Trong sinh hoạt thường ngày, một moment được biểu diễn bằng cách ghép *zone* (múi giờ) hoặc *offset* (độ lệch UTC) vào rtime bất kì. Ta có công thức: `moment = rtime + (zone or offset)`
- Một zone tương ứng với 1 offset, tuy nhiên cũng có trường hợp một zone gắn liền với 2 offset trong năm (Daylight Saving Time). Mặc dù vậy, tại một thời điểm bất kì trong năm, zone vẫn chỉ mang 1 giá trị offset duy nhất.
- Như vậy, có thể viết lại công thức một cách tổng quát thành: `moment = rtime + offset`. ISO-8601 là tiêu chuẩn quốc tế biểu diễn moment theo công thức này.
- UTC có offset bằng 0. Từ công thức tổng quát chúng ta có hệ quả:
```
moment = rtime(city_A) + offset(city_A) 
       = rtime(city_B) + offset(city_B) 
       = ...
       = rtime(UTC) + offset(UTC)
       = rtime(UTC)
```

<br>
Ứng dụng vào trong code:

- Java 1.8 cung cấp những Date Time API mạnh mẽ dưới dạng *immutable object*. Các API có thể phân loại thành 2 nhóm moment và rtime. Phân biệt được chúng giúp ta dễ dàng quyết định khi nào thì sử dụng loại nào.
- Từ hệ quả đã suy ra, bằng cách giữ nguyên giá trị moment, khi thay đổi zone/offset, chúng ta sẽ lấy được giá trị rtime tương ứng tại địa phương. Method `ZonedDateTime.withZoneSameInstant()` giúp ta dễ dàng làm việc này.
- Để xử lý Daylight Saving, chúng ta dùng định danh múi giờ theo tiêu chuẩn **IANA - tz database** (tra cứu tên múi giờ tại https://time.is). Phần còn lại sẽ được `ZonedDateTime` lo liệu.

# Kết

Vậy là mình đã giới thiệu xong những khái niệm quan trọng. Một khi đã nắm rõ chúng, mình tin rằng việc xử lý ngày giờ không còn là nỗi sợ hãi quá lớn của các developers Java 😊

Tuy nhiên, **vẫn sẽ có những sai lầm** tạo ra bug nếu chúng ta không chú ý, hẹn gặp lại các bạn ở phần tiếp theo.

<div align="center">
  <img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/02/img/9gag.jpg" />
  <br />
  <span>

  *Thậm chí Marvel đã quên mất sự chênh lệch múi giờ giữa Wakanda và New York - [9gag](https://9gag.com/gag/aWx7GOq)*

  </span>
</div>



## Liên kết ngoài
- ISO 8601: https://en.wikipedia.org/wiki/ISO_8601
- Tz database (IANA time zone database): https://en.wikipedia.org/wiki/Tz_database
- Handle Daylight Saving in Java: https://www.baeldung.com/java-daylight-savings

<br>

© 2021 Nam Bach.  All rights reserved.
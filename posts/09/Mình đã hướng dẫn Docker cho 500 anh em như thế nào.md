Năm 2020 mình có upload lên Youtube một [video hướng dẫn tự học Docker](https://www.youtube.com/watch?v=yWCse8S2qsM&ab_channel=nbstudio). Ở thời điểm hiện tại, video đã được 20,000 lượt view, hơn 700 lượt like cùng nhiều phản hồi tích cực. Quan trọng hơn, chiếc video này là một điểm nhấn trong CV gây ấn tượng mạnh với các nhà tuyển dụng, góp phần không nhỏ để mình có được công việc hiện tại. Bài này để chia sẻ những điều mình đã học hỏi được trong quá trình thực hiện video đó.

# 1. Nguyên nhân & động lực

## Từ một đứa ất ơ, mình tự học Docker như thế nào?

Cuối năm 2019, team mình nhận thông báo về một dự án migration, yêu cầu migrate hệ thống microservice từ Cloud Foundry sang nền tảng của AWS. Nhiệm vụ của mình lúc đó là tìm hiểu về Docker & Kubernetes, sử dụng môi trường AWS của công ty để thử nghiệm việc deployment và báo cáo lại tính khả thi, cũng như ước lượng thời gian triển khai cho khách hàng.

Tại thời điểm đó, hầu như các dự án trong đơn vị đều là maintain hệ thống cũ, nên chẳng ai có cơ hội để tiếp xúc với các công nghệ hiện đại đang phổ biến như thế này. Toàn bộ hiểu biết của mình về cloud cũng như Docker, là một con số 0 tròn trĩnh.

Mình xin sếp khoảng 2 tuần để tự học trước về Docker & Kubernetes, mò một vài service cơ bản của AWS (ban đầu là xin 1 tháng cơ vì mình cảm thấy hơi áp lực, sếp kêu lâu quá, thế là phải giảm xuống nửa tháng và cuống cuồng tìm resource để tự học 😅). 1 tuần sau đó mình đã bắt đầu viết những file deployment cơ bản để thử nghiệm triển khai các app từ Cloud Foundry lên EKS của AWS. Toàn bộ quá trình pilot thử nghiệm chỉ kéo dài hơn 1 tháng, sau đó team mình đã nhận dự án và triển khai thành công tốt đẹp.

## Tự học như thế nào cho hiệu quả?

Thông thường khi tìm hiểu một công nghệ mới, mình thường bỏ tiền mua những khóa học trên Udemy hoặc Pluralsight. Theo quan điểm cá nhân, đó là cách vừa nhanh vừa hiệu quả nhất. Làm tutorial sẽ nhanh có kết quả, nhưng đôi khi bạn sẽ không có cơ hội hiểu sâu sắc về những thứ mình đang làm. Đọc document thì sẽ hiểu sâu sắc, nhưng bù lại tốn quá nhiều thời gian vì mình đọc khá chậm, nghe nhìn thì sẽ nhanh hơn.

> Hồi đấy mình học Docker miễn phí tại [learndocker.online](https://learndocker.online/courses).

## Chia sẻ là cách học hiệu quả nhất

Dự án migration được khách hàng đánh giá cao, sếp mới nhờ mình mở lớp training Docker & K8s cho các anh em trong đơn vị. Mình đồng ý, vì thực ra dạy lại cho người khác là một cách học cực kì hiệu quả (có tên gọi là [kỹ thuật Feynman](https://www.facebook.com/watch/?v=1781213878564212)). Đại ý của kỹ thuật này là, khi nghiên cứu một vấn đề, hãy thử truyền đạt lại cho một đứa trẻ (hoặc một người không có chuyên môn) bằng ngôn ngữ đơn giản. Khi đó chúng ta sẽ phát hiện nhiều thiếu sót trong hiểu biết của mình, và tiếp tục củng cố kiến thức cho sâu sắc và có hệ thống.

Sau 2 buổi training, mình quyết định quay lại thành video, một phần là để tập luyện cho việc truyền tải được trôi chảy, rõ ràng rành mạch hơn, một phần là để các anh em trong đơn vị có tư liệu về xem lại.

# Thực hiện

## 1. Review kiến thức & phác dàn ý

Để phác được dàn ý buổi chia sẻ, việc quan trọng nhất là review lại toàn bộ những thứ chúng ta đã học. Tùy vào lượng kiến thức truyền đạt nhiều đến đâu, chúng ta cần suy nghĩ ***một trình tự chia sẻ hợp lý***, móc nối các khái niệm sao cho có logic.

> Ví dụ với video Docker của mình, mình dự tính sẽ chia sẻ **đầy đủ những khái niệm cơ bản**, từ container, image registry cho đến volume, port mapping, Dockerfile. Mình sẽ bỏ qua những phần nâng cao (người xem sẽ tự nghiên cứu thêm). Tuy nhiên ở phần Dockerfile, đây là phần kiến thức có tính ứng dụng quan trọng, mình sẽ đi sâu vào phần này với các khái niệm như **build context**, **cached layers**.
>
> Sau khi xác định được tất cả những khái niệm cần chia sẻ, mình sắp xếp chúng theo trình tự từ đơn giản đến phức tạp. Cái xuất hiện trước sẽ là tiền đề giới thiệu cái đến sau. Từ đó có được outline sơ bộ.
>
> ![Outline](https://raw.githubusercontent.com/nambach/viblo/master/posts/09/01-Outline.png)


## 2. Làm slide & triển khai lời thoại (script)

Từ outline ban đầu, chúng ta tiến hành làm slide PowerPoint.

Đây là công đoạn tốn thời gian nhiều nhất. Chúng ta sẽ cần tưởng tượng trong đầu, cách chúng ta nói về vấn đề đó, cách mà slide trình chiếu sẽ hiển thị ra sao, để giúp người xem hình dung một cách trực quan từng câu chữ mà họ nghe.

Do vậy, khi làm slide, mình sẽ note luôn lời thoại cần nói vào mục "Notes" bên dưới slide. Phần lời thoại sẽ được review nhiều lần cho rõ ràng, ngắn gọn, trách lủng củng. Lưu ý chúng ta chỉ note đại ý nhằm không bị "khớp" trong lúc thuyết trình, chứ không phải học vẹt từng câu từng chữ một.

![Slides](https://raw.githubusercontent.com/nambach/viblo/master/posts/09/02-slides.png)

## 3. Tập luyện

Phần này bao gồm 2 việc: 

- **Tập luyện thuyết trình**: cho dù là đứng trong phòng họp hay nói chuyện qua màn hình video, chúng ta đều phải nắm rõ những gì sẽ truyền đạt. Không thể vừa nói vừa nhìn chăm chăm vào slide hoặc vào phần nhắc lời thoại, người xem - cho dù trực tiếp hay gián tiếp qua video - sẽ cảm thấy những thứ họ nghe thật không đáng tin cậy. Ngoài ra, ngữ điệu (intonation) cũng là một yếu tố khiến phần thuyết trình của bạn bớt tẻ nhạt "buồn ngủ", trở nên sống động và cuốn hút hơn.

- **Tập luyện các thao tác thực hành (demo)**: phần này khá quan trọng nhưng thường hay bị xem nhẹ. Các thao tác demo, nếu không tập dợt trước sẽ dễ bị khựng và mắc lỗi, hoặc gặp trục trặc ngoài ý muốn. Điều này sẽ gây cảm giác khó chịu, khiến người xem bực mình, mất cảm tình với người chia sẻ.

## 4. Ghi hình

Sau khi đã chuẩn bị thật chỉnh chu cho video của bạn, chúng ta bắt đầu ghi hình.

Phần này khá đơn giản. Một tip nhỏ là nếu phần thuyết trình quá dài, chúng ta nên chia nhỏ ra ghi nhiều phần riêng lẻ, sau đó ghép lại. Như vậy sẽ đỡ mệt hơn, cũng như nếu có lỗi xảy ra thì cũng không cần ghi hình lại từ đầu.

> Mình sử dụng phần mềm [ActivePresenter](https://atomisystems.com/download) để ghi hình trên máy tính.
>
> Việc quay video đương nhiên sẽ đỡ áp lực hơn là thuyết trình có người nghe trực tiếp. Đây là một cách tập luyện khá hay giúp chúng ta tự tin hơn cho những đợt thuyết trình quan trọng.

## 5. Hậu kì

Đôi khi một số video cần thêm các hiệu ứng animation cho đẹp mắt. Mình thì không rành lắm nên chỉ dùng PowerPoint đơn thuần thôi.

# Vài điều sau cùng

## Đón nhận phê bình

Khi đã công khai trên mạng xã hội thì có rất nhiều người vào nhận xét về sản phẩm của bạn. Những góp ý đúng và xây dựng thì chúng ta tiếp thu. Những lời cảm ơn và khen ngợi thì chúng ta biết ơn. Còn lại, thi thoảng bạn sẽ gặp những người, họ chẳng quan tâm những điều tốt bạn làm, chỉ cần có lỗi là họ ào vào soi mói, phê phán. Những người này thì... thôi kệ vậy, có thể dùng tính năng phê duyệt bình luận của Youtube tạm cho nó nhẹ đầu 😅.

## Gửi đến "tôi" trong tương lai

Đôi khi mình nghĩ rằng, viết blog hay làm video tutorial, thì người hưởng lợi nhiều nhất chính là mình trong tương lai. CNTT vốn dĩ thay đổi liên tục. Hôm nay bạn làm Java, nhưng ngày mai bạn làm C#, ngày kia làm Golang... Việc chuyển đổi công nghệ nhiều cũng dễ khiến chúng ta quên mất những gì "tinh túy" ngày xưa đã từng lĩnh hội. Viết blog hay làm video, là một cách *sao lưu* hiệu quả, để sau này khi tình cờ quay lại công nghệ cũ, chúng ta vẫn có nguồn tài liệu để tham khảo và học hỏi, từ chính bản thân chúng ta của ngày xưa.

Chúc các bạn sẽ có thể bắt đầu những dự án của bản thân ngay hôm nay. Cheers! 🍻
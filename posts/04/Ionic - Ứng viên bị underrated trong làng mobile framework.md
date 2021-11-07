Mỗi khi nhắc đến lập trình mobile đa nền tảng (iOS & Android), hầu hết mọi người sẽ nghĩ đến React Native hoặc Flutter. Có một lựa chọn khác đang bị underrated nhưng thực chất vô cùng tiềm năng, đó chính là Ionic Framework.

# Disclaimer

Tất nhiên không thể tự nhiên mà đem Ionic đi so sánh với React Native hay Flutter được, vì chúng thuộc 2 loại framework khác nhau: một bên là Hybrid sử dụng WebView để load và chạy HTML, một bên là Cross-platform build toàn bộ app thành các native control tương ứng với platform - chúng khác nhau về bản chất, quy trình phát triển và hiệu năng sử dụng.

Tuy nhiên, nếu xét về mặt nghiệp vụ, nếu ứng dụng mobile không đặt nặng về việc tương tác phần cứng thiết bị (các tính năng native như GPS, Bluetooth, NFC...) hoặc không yêu cầu quá cao về mặt hiệu suất, thì Ionic có thể là lựa chọn rất đáng xem xét.

# Ionic là gì?

Ionic là một framework cho phép bạn xây dựng ứng dụng mobile bằng phương pháp Hybrid, nghĩa là toàn bộ ứng dụng sẽ được xây dựng trên nền tảng web thông thường (HTML, CSS, JS) nhưng với giao diện của di động. Sau đó sử dụng một component WebView để nhúng và chạy nội dung HTML đó. Do vậy, có thể xem ứng dụng Hybrid tương tự như việc chúng ta chạy website có responsive bằng trình duyệt của di động.

> Nghe có vẻ lừa đảo nhỉ? Chẳng khác nào việc sửa lại responsive của website hiện tại, nhét vào một cái browser, tải lên AppStore và bảo với khách hàng, đây là app di động mà các bạn yêu cầu ("Có khác quái gì website hiện tại đâu!?")

Tuy nhiên, sự khác biệt ở chỗ, Hybrid App vẫn cho phép chúng ta sử dụng hầu hết các *tính năng phần cứng của thiết bị* (điều mà trình duyệt mobile không làm được). Và bởi vì chạy trên WebView nên các component của Ionic đều được tối ưu về mặt hiệu suất, trong khi vẫn đảm bảo UI/UX mượt mà phù hợp với từng loại platform, một sự tiện lợi về mặt thời gian và công sức dành cho các developer trước giờ vẫn luôn theo lối desktop-driven.

Về bản chất, Ionic bao gồm các web component được build sẵn với UI/UX thích ứng linh động cho iOS và Android. Chúng được sử dụng để build codebase HTML. Để build ra file chạy trên iOS và Android, chúng ta cần sử dụng 1 thư viện generate ra thư mục project của iOS và Android tương ứng. Có 2 lựa chọn, **Apache Cordova** và **CapacitorJS**. CapacitorJS ra đời sau này, được phát triển bởi chính đội ngũ Ionic dựa trên người tiền nhiệm Cordova của Apache.

# So sánh
## 1. Learning Curve

Ionic được xem là loại framework "mì ăn liền" dành cho dân front-end để viết ứng dụng mobile. Bởi cơ chế build Hybrid, bản chất ứng dụng mobile lúc này không khác gì một ứng dụng web thông thường.

- Đầu tiên cần tìm hiểu bộ CLI và các component có sẵn. Bước này tương đối đơn giản, bản chất chúng chỉ là các web component, Ionic hỗ trợ ở cả 3 dạng framwork là Angular, React, Vue, và cả vanilla Javascript.
- Tiếp theo, cần tìm hiểu về thư viện CapacitorJS để tạo ra thư mục project iOS hoặc Android tương ứng. Khi đã có thư mục project iOS và Android, quá trình clean-compile-build ra file `.ipa` và `.apk` là như nhau cho dù chúng ta sử dụng framework nào đi chăng nữa.

Như vậy, xét về khả năng lĩnh hội, Ionic **tốn ít công sức nhất** để tìm hiểu và biết cách sử dụng, nếu đem so sánh với React Native hoặc Flutter

> Để code React Native, bạn phải biết ReactJS, tuy nhiên những hiểu biết về ReactJS không phải lúc nào cũng đúng 100% với React Native - có một số vấn đề về performance ở React Native chắc chắn sẽ rất khó xơi. Còn với Flutter, bạn phải học Dart, một ngôn ngữ hoàn toàn mới, và hướng tiếp cận, cách tối ưu và các best practices cũng rất khác thông thường.

## 2. UI/UX Customizable

Nhờ bản chất của Hybrid là HTML, Ionic cho phép khả năng customize các component ở mức linh hoạt tối đa, như cách chúng ta style và animate HTML thông thường.

Trong khi đó, bản chất của React Native và Flutter là các native control, việc styling cho các control này sẽ tương đối khó nhằn và có nhiều giới hạn. Do vậy, hầu hết các trường hợp, việc google một thư viện có sẵn thay vì đi code lại từ đầu sẽ hiệu quả hơn rất nhiều, cả về mặt thời gian công sức lẫn độ tối ưu của code.

Thêm vào đó, mỗi framework sẽ có một hệ thống animation riêng, sẽ phải tốn một khoảng thời gian và công sức để học và thành thạo chúng.

## Performance

Đây thường là yếu tố có tính quyết định chủ chốt. Phần lớn mọi người sẽ loại bỏ Ionic ra khỏi các lựa chọn khi xây dựng mobile app, bởi cơ chế hoạt động của Hybrid app được cho là *"chậm hơn rất nhiều so với React Native & Flutter"*.
> "Sử dụng WebView để chạy HTML CSS thì làm sao nhanh bằng việc build ra native app như React Native hay Flutter được?"

Tuy nhiên, Hybrid app có thật sự "chuối" như giang hồ vẫn hay đồn đại? 

Có một sự thật là bản thân mình trước đây luôn dè bỉu dòng framework này, thời mà Phonegap vẫn còn nổi tiếng và Ionic vẫn chưa được nhiều người biết đến. Khi dự án ở công ty yêu cầu làm app mobile, mình chọn ngay React Native mà không cần suy nghĩ. Đến app mobile thứ hai, mình thử chuyển sang Ionic và code trên Angular, một phần để người khác có thể bảo trì sau này khi mình rời công ty (team mình có 3 front-end Angular nhưng không ai biết React Native, các team khác cũng vậy, khó kiếm được người để maintain code React Native), một phần là vì sau khi học thử [khóa giới thiệu về Ionic 2 trên egghead](https://egghead.io/courses/building-apps-with-ionic-2), mình bị ấn tượng bởi giao diện của các component Ionic nhìn rất giống các native control trên iOS và Android, mặc dù chúng chỉ là HTML và CSS. Và bất ngờ nhất là khi build, publish và chạy trên thiết bị vật lí, ***hầu như không thể nhận ra sự khác biệt của app React Native và app Ionic***, bởi độ mượt khi scroll, navigate, các thao tác pan, zoom, long press... đều nhạy và mượt như nhau. Nếu không tin, bạn có thể thử.

> "The performance of the Ionic application is not as good as compared to native mobile applications. **However, the performance gap is not noticeable** for most of the average users." - [javatpoint.com](https://www.javatpoint.com/ionic-vs-phonegap)

Như vậy, xét về performance, Ionic *có chậm hơn* so với native app, nhưng nếu được code cẩn thận, biết tận dụng kĩ thuật performance optimization trong front-end, như là việc sử dụng immutable object khi quản lý state,  `PureComponent` trong React hay là `ChangeDetectionStrategy.OnPush` trong Angular, thì hiệu suất của các app Ionic cũng nhanh không thua kém gì native app.

Nếu bạn có ý định tìm hiểu Ionic, hoặc đã làm việc với Ionic nhưng gặp các vấn đề về hiệu suất, thì đây là một bài viết đáng đọc của tác giả Josh Morony: [Ionic Framework Is Fast (But Your Code Might Not Be)](https://eliteionic.com/tutorials/ionic-framework-is-fast-but-your-code-might-not-be/).

> (Có thể bạn đã biết) Tối ưu performance cho React Native là cả **một nghệ thuật** mà không phải ngày một ngày hai là có thể lĩnh hội được. Tổ chức Callstack có xuất bản [một ebook dày 123 trang](https://callstack.com/data/The_Ultimate_Guide_to_React_Native_Optimization_Ebook-Callstack_FINAL.pdf) nói về những quy tắc để tối ưu performance của React Native, được nhiều người trong giới đánh giá cao. Bởi thế, nhiều trường hợp app mobile được code bằng React Native, nhưng performance vẫn cực kì tệ hại là chuyện bình thường.


## Native features

Bên cạnh performance thì đây là yếu tố thứ hai khiến nhiều người loại bỏ Ionic ra khỏi cuộc chơi ngay từ đầu.

Thực ra, Ionic chỉ bao gồm các UI components, khả năng tương tác với phần cứng được quyết định bởi Cordova hoặc CapacitorJS. Chúng cho phép sử dụng các tính năng phần cứng thông qua các plugin được đóng gói dưới dạng Javascript API. Số lượng các plugin của Cordova nhiều và ổn định hơn CapacitorJS, nhưng CapacitorJS hiện đang được cộng đồng hỗ trợ mạnh mẽ, và cũng cho phép import sử dụng toàn bộ plugin từ Cordova.

Do vậy, mặc dù Cordova & CapacitoJS không có khả năng tương tác với 100% tính năng vật lí của phần cứng như native apps, nhưng số lượng tính năng đã được hỗ trợ (tính đến thời điểm hiện tại của bài viết) cũng nhiều đủ để xây dựng gần như bất kì ứng dụng mobile nào: **Push Notification, GPS, Barcode Scan, Bluetooth, Map...**

## Development

Ionic không hỗ trợ hot-reload trên virtual device hay physical device. Do đó, nếu ứng dụng đặt nặng vấn đề tương tác phần cứng (GPS, bluetooth...) thì việc phát triển sẽ tương đối tốn thời gian. Còn nếu không, Ionic sẽ là lựa chọn hoàn hảo, thậm chí ưu thế hơn, nhờ việc **live-reload ngay trên browser** mà không cần mở virtual/physical device để hiển thị. Đây là một trong những lý do mình khá thích ở Ionic, nhờ nó mà tốc độ làm ứng dụng mobile nhanh hơn gấp nhiều lần so với khi code bằng React Native (xin lỗi các fan React Native nhé).

# Kết luận

Không có framework nào là "fit-for-all", là tốt nhất cả. Tùy vào các trường hợp nghiệp vụ khác nhau mà mỗi framework sẽ phát huy thế mạnh riêng của mình.

Chỉ là cảm nhận của cá nhân mình, có vẻ Ionic đang bị đánh giá quá thấp so với tiềm năng và lợi ích mà nó mang lại. Hy vọng sau bài viết này, các bạn sẽ có cái nhìn thiện cảm và hấp dẫn hơn với Ionic, để nó có một vị trí xứng đáng trong danh sách các lựa chọn framework mobile tiếp theo của bạn.
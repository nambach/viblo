_(Xem bài viết trước: ["Làm backend với Java Spring, tôi nên biết những gì?"](https://viblo.asia/p/lam-backend-voi-java-spring-toi-nen-biet-nhung-gi-LzD5d9eoKjY))_

Trong bài viết này mình xin tiếp tục tổng hợp một số chủ đề nâng cao trong lập trình backend Java.

# 4. Concurrency & Real-time computing

**1. Multithreading**

Lập trình song song (hay còn gọi là lập trình đa luồng)  là một chủ đề nâng cao tương đối khó (nếu không muốn gọi là rất khó).

Chủ đề này trải dài từ các vấn đề cơ bản như việc triển khai đa luồng bằng các API có sẵn của Java, đến việc hiểu biết sâu sắc về cơ chế hoạt động bên dưới của CPU, để tối ưu việc thiết kế và triển khai thread cho ứng dụng. Một số keyword có thể kể đến như **context switching**, **false sharing**, **happens-before relationship**... 

>> Khả năng hiểu biết sâu sắc và tường tận cách hoạt động của một công cụ để có thể sử dụng nó một cách tốt nhất và tối ưu nhất được gọi là **[mechanical-sympathy](https://wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/wat.concept.mechanical-sympathy.en.html#:~:text=Mechanical%20sympathy%20is%20when%20you,have%20to%20have%20Mechanical%20Sympathy.)**.

Một số nguồn tham khảo về multithreading và concurrency trong Java:
- [Series Concurrency của Jenkov](http://tutorials.jenkov.com/java-concurrency/index.html): đây là một series đầy đủ và chi tiết về toàn bộ các khía cạnh trong lập trình đa luồng Java.
- Sách ["Java Concurrency in practice"](https://www.amazon.com/dp/0321349601): mặc dù xuất bản từ 2006 nhưng [cuốn sách này vẫn rất đáng đọc](https://dzone.com/articles/does-java-concurrency-in-practice-still-valid-toda) vì nó xoay quanh những nguyên lý nền tảng. Các API có thể thay đổi, nhưng nền tảng của đa luồng thì sẽ mãi bất biến với kiến trúc máy tính hiện tại.
- [Series multithreading bằng tiếng Việt](https://gpcoder.com/category/java-core/multi-thread) của anh Phan Thanh Giang.
- [Bài phân tích nâng cao về multithreading](https://demtv.hashnode.dev/mechanical-sympathy) của anh Dem Trần.

**2. Websocket with STOMP**

Xử lý real-time với websocket không còn là chủ đề xa lạ với các developers. Giải pháp thông dụng nhất đối với Spring là [sử dụng STOMP (Stream Text-Oriented Messaging Protocol)](https://www.baeldung.com/websockets-api-java-spring-client) được cung cấp trong dependency `spring-messaging`. Tuy nhiên việc tổ chức kiến trúc ở các hệ thống lớn sẽ phức tạp hơn, đòi hỏi có sự hỗ trợ của các concurrency framework.

**3. Concurrency Framework**

Lập trình đa luồng khó, nhưng nếu có sự trợ giúp của các concurrency framework, thì sẽ dễ dàng hơn nhiều, miễn là tuân theo các pattern mà framework quy định.

2 keywords quan trọng nhất liên quan đến concurrency framework chính là **reactive programming** và **non-blocking IO**. [Riêng từ khóa "reactive programming" thôi là đã bao hàm rất nhiều chủ đề thú vị](https://www.baeldung.com/java-reactive-systems): functional programming, reactive streams, message driven, back-pressure, non-blocking communication... Nói cho đơn giản, thì reactive-programming sẽ giúp server tăng khả năng chịu tải và chịu lỗi (resilient), phản hồi nhanh hơn (responsive), dễ giao tiếp với nhau, dễ nâng cấp và mở rộng (highly scalable).

Một số framework có thể kể đến như:
- [Akka](https://akka.io): một trong những concurrency framework nổi tiếng nhất, hoạt động dựa trên [mô hình Actor](https://kipalog.com/posts/Elixir-Erlang--Actor-model-va-Concurrency) (Actor-model) lấy cảm hứng từ ngôn ngữ Erlang. Akka hỗ trợ bằng Java lẫn Scala.
- [Play Framework](https://www.playframework.com): được phát triển trên nền tảng của Akka, Play chú trọng vào sự đơn giản và dễ dùng. Nếu các bạn muốn tập trung vào web MVC như HTML UI, REST endpoints, API gateway... thì Play là 1 lựa chọn phù hợp.
- [Vert.x](https://vertx.io): framework này cung cấp các tính năng tương tự như Akka, nhưng hoạt động dựa trên mô hình event-driven và hỗ trợ đa ngôn ngữ. Bởi sự gọn gàng linh hoạt, Vert.x thường được chọn để triển khai microservice.
- [Spring Webflux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html): từng được biết đến với tên gọi là [Project Reactor](https://projectreactor.io), sau này khi được Pivotal tài trợ, nó trở thành một nhánh trong hệ sinh thái Spring. Nếu bạn đã từng làm việc với các thư viện ReactiveX như RxJS, thì đây chính là framework dựa trên nền tảng đó. Spring Webflux là một mảnh ghép hoàn chỉnh cho các dự án Spring MVC cần giải pháp đa luồng và non-blocking IO.

# 5. Microservice (MSA)
Microservice không còn là trend mà đã dần trở thành một kĩ thuật tất yếu đối với các lập trình viên hiện đại.

**1. Containerize with Docker**

Công nghệ container chắc hẳn đã không còn xa lạ với mọi người. Mỗi khi nhắc đến microservice không thể không kể đến container, bởi những tính chất độc đáo của nó. 
> - Isolated: mỗi container có behavior giống hệt như một máy ảo, do đó các module chạy trên các container khác nhau sẽ không rơi vào tình trạng xung đột tài nguyên hệ thống như xung đột file system hoặc network.
> - Lightweight: container rất nhẹ, không nặng như máy ảo, bởi bản chất nó là các tiến trình (process) chạy trên OS. Start hoặc shutdown một container là cực kì nhanh, khiến cho việc scale up/scale down hệ thống microservice cũng dễ dàng hơn nhiều.
> - Immutable: khi containerize một ứng dụng, toàn bộ dependency và code sẽ được đóng gói kín thành 1 image. Điều này giúp đảm bảo sự toàn vẹn (consistency) - hệ thống trên production sẽ luôn ổn định như cách nó chạy trong máy local của bạn, không bao giờ có lỗi xung đột version.
> - Stateless: container không bao giờ tạo ra thay đổi về mặt dữ liệu, thay vào đó chúng ta sẽ sử dụng volume như một external storage. Điều này giúp các service không bị ràng buộc dữ liệu lẫn nhau, shutdown/restart cực kì thoải mái, dễ dàng scale hệ thống.

Hiện nay Docker là công nghệ phổ biến nhất cung cấp giải pháp container. Có một số alternative như Podman, Rancher... nhưng Docker vẫn chiếm vị trí hàng đầu. Hiện tại đang có rất nhiều nguồn học Docker miễn phí, nhưng bởi quá nhiều nên đôi khi mãi vẫn không học xong đến nơi đến chốn 😅.

Nếu các bạn là newbie, mình cực kì recommend trang [**learndocker.online**](https://learndocker.online). Hoàn toàn miễn phí và thực sự chất lượng. Anh instructor có tâm đến nỗi học xong 2/4 module là donate liền cho ảnh 😄. Chỉ trong vòng 1 tháng, từ một đứa không biết gì về container & Docker, mình đã có thể tự handle toàn bộ các task deployment trong dự án, training lại cho các anh em trong team, thậm chí tự làm [video hướng dẫn Docker](https://youtu.be/yWCse8S2qsM) cho 500 anh em trên youtube và có các phản hồi rất tích cực 😁.

**2. Container ochestration with Kubernetes**

Đối với các hệ thống lớn thì số lượng container có thể lên đến hàng chục hàng trăm. Lúc này cần có một giải pháp để quản lý và điều phối các container sao cho chúng hoạt động trơn tru cùng nhau.

Các giải pháp có thể kể đến: phổ biến nhất với Kubernetes (K8s), Docker Swarm, Redhat Openshift, Nomad... Nếu có thêm kinh nghiệm với K8s, lập trình viên sẽ được đánh giá cao và có nhiều lợi thế khi tuyển dụng.

**3. Spring Boot Actuator**

[Spring Boot Actuator](https://www.baeldung.com/spring-boot-actuators) là một dependency không thể thiếu khi triển khai production. Các chức năng mà nó cung cấp có thể kể đến như:
- **Health-check**: việc kiểm tra trạng thái đang hoạt động của ứng dụng backend trong microservice rất quan trọng. Các công cụ như K8s sẽ dựa vào health status của container để có thể tự restart lại container trong trường hợp ứng dụng bị treo do lỗi runtime.
- **Monitoring**: khi triển khai microservice, bởi có quá nhiều container chạy trên cùng một machine nên cần thiết phải giám sát chặt chẽ các tài nguyên như RAM, CPU, network bandwidth... để khi có sự cố, hệ thống sẽ tự gửi mail thông báo cho developers và có những cách xử lý tạm thời cho phù hợp.
- **Programmatically manipulation**: sẽ có những trường hợp chúng ta muốn [restart app Spring Boot bằng code thay vì phải gõ terminal](https://www.baeldung.com/java-restart-spring-boot-app), nhờ đó mà việc start/stop container linh hoạt hơn, thì Actuator chính là giải pháp.

**4. API communication**

Việc giao tiếp giữa các microservice là một khía cạnh quan trọng. 

Nếu triển khai theo mô hình Spring Cloud (toàn bộ microservice đều được viết bằng Java Spring) thì chúng ta có thư viện FeignClient hỗ trợ việc gọi API và Hystrix với vai trò *circuit breaker*.

> [Circuit breaker](https://www.baeldung.com/spring-cloud-circuit-breaker), đúng với nghĩa tiếng Việt của nó - "cầu chì" - là [một pattern nhằm tăng khả năng chịu lỗi (fault-tolerance)](https://medium.com/nerd-for-tech/design-patterns-for-microservices-circuit-breaker-pattern-ba402a45aac2). Trong hệ thống, khi các microservice phụ thuộc chồng chéo lên nhau, nếu một request giữa 2 service bị fail, sẽ có khả năng xảy ra 1 chuỗi thất bại dây chuyền (cascading failures) lên toàn bộ hệ thống, khiến user không thể sử dụng app. Lúc này circuit breaker đóng vai trò là điểm bảo vệ để ngăn chặn phản ứng dây chuyền đó và gửi thông báo cho developer để xử lý kịp thời.

Trong mô hình microservice hiện đại, không phải container nào cũng viết bằng Java. Do đó việc giao tiếp lúc này không khác gì gọi API thông thường. Một số thư viện hỗ trợ có thể kể đến như RestTemplate (có sẵn trong SpringMVC), [OkHttp](https://square.github.io/okhttp), [Retrofit](https://square.github.io/retrofit).

> Hiện tại [gRPC](https://www.baeldung.com/grpc-introduction) là một xu thế đang được dùng để thay cho REST bởi hiệu năng vượt trội của nó - payload được gửi đi ở dạng binary thay vì text json, và sử dụng giao thức HTTP/2 thay vì 1 như của REST.

**5. Message queue communication**

Việc giao tiếp thông qua [message queue](https://toidicodedao.com/2019/10/08/message-queue-la-gi-ung-dung-microservice) thay vì gọi request trực tiếp đến nhau là một pattern để giảm sự phụ thuộc (loose-coupled) và tăng khả năng mở rộng (scalability) của hệ thống.

> Thử tưởng tượng chúng ta có 5 service trong hệ thống. Khi toàn bộ service gọi trực tiếp lẫn nhau, chỉ cần loại bỏ một service sẽ gây ra sự sụp đổ trong hệ thống, bởi 4 service kia đều bị phụ thuộc vào service này.
><div align="center">
>    <img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/07/message-queue-1.png" />
></div>
> Trong khi đó, nếu dùng message queue, các service sẽ chẳng biết đến nhau mà chỉ biết một queue duy nhất. Queue lúc này đóng vai trò như trạm bưu điện trung gian. Loại bỏ hay thêm vào một service sẽ không gây sụp đổ phụ thuộc.
><div align="center">
>    <img src="https://raw.githubusercontent.com/nambach/viblo/master/posts/07/message-queue-2.png" />
></div>

Một số các message queue đang được sử dụng phổ biến hiện nay là Apache Kafka, RabitMQ, AmazonSQS...

**6. Spring Native & GraalVM**

Một trong những khuyết điểm của Spring Boot chính là kích cỡ file jar khá lớn, thời gian start app lâu, tiêu tốn RAM tương đối nhiều - cái giá phải trả cho tốc độ xử lý tính toán cực nhanh của nó.

Đội ngũ Spring và GraalVM đã cho ra đời [dự án Spring Native](https://atekco.io/20211108-toi-uu-spring-container-voi-spring-native) để xử lý các khuyết điểm này. Bằng cách tối ưu quy trình build app thành các *native image*, thời gian start app cũng như kích thước RAM tiêu thụ giảm đi đáng kể. Tuy nhiên đổi lại, thời gian build ở máy local sẽ lâu hơn và khối lượng RAM tiêu thụ lúc build cũng "kinh khủng" không kém. (trade-offs 😅)

Hiện tại Spring Native đang ở giai đoạn thử nghiệm (experimental).

# 6. Better source code with Kotlin
Java là một ngôn ngữ tương đối thuần khiết và dễ học. Khuyết điểm lớn nhất của nó là sự rườm rà (verbose) - nghĩa là chúng ta phải tốn quá nhiều code cho một tính năng nếu so sánh với một số ngôn ngữ khác.

Thử so sánh 2 đoạn code giữa Java và Ruby.

```java
class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

```ruby
print "Hello World!"
```

> Mỗi ngôn ngữ lập trình có một mức độ verbosity nhất định. Có thể bạn sẽ cho rằng cứ chọn cái nào viết ngắn gọn nhất là ổn. Tuy nhiên, bản chất của sự ngắn gọn mà các ngôn ngữ có được là do nó đang che đi những đoạn code phức tạp bên dưới. Càng ngắn gọn nghĩa là tính abstract càng cao, bạn không có quyền can thiệp sâu hơn vào code thực thi, dẫn đến việc performance không được tối ưu 100% bởi những sự lặp lại dư thừa vô hình. Điều này lý giải tại sao ngôn ngữ "rườm rà" như C, C++ lại có hiệu năng cao.<br><br>
> Tuy nhiên, quá verbose dài dòng sẽ rất khó maintain. Trong phần mềm, khả năng bảo trì là yếu tố sống còn của dự án. Do vậy, developer cần biết cách cân bằng giữa 2 cán cân verbosity-performance để lựa chọn ngôn ngữ cho phù hợp.

Qua mỗi lần release của Java, chúng ta thấy được nỗ lực của Oracle trong việc tinh giản Java để nó trở nên "hấp dẫn", "sexy" hơn trong mắt lập trình viên.

Tuy vậy, vẫn còn một lựa chọn không kém phần hay ho, chính là Kotlin. Được giới thiệu lần đầu tiên vào năm 2011 bởi Jetbrains, syntax của Kotlin đột phá đến nỗi Google quyết định chọn Kotlin làm ngôn ngữ chính thức cho mã nguồn Android. Những năm gần đây, Kotlin luôn được bình chọn là một trong những ngôn ngữ yêu thích nhất trong các khảo sát của Stackoverflow.

Migrate source code từ Java sang Kotlin là một quyết định đáng để suy nghĩ, bởi Kotlin giúp viết code ngắn gọn đơn giản hơn, nhưng vẫn giữ được hiệu năng cao mà Java mang lại.

# Kết

Như vậy mình đã tổng hợp xong một số chủ đề nâng cao trong Java. Ở level này có thể các bạn đã là middle hoặc senior trong dự án (hardcore lắm 🥴). Bản thân mình thì vẫn chưa đến trình độ biết hết mọi thứ trong bài viết này, nhưng ít nhất nó vẫn có thể dùng để tham khảo như một roadmap hoặc để ôn tập hệ thống lại để đi phỏng vấn chẳng hạn 😁.

Hẹn gặp các bạn trong những chủ đề tiếp theo.
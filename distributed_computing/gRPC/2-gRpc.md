## gRPC

- Trong phần đầu chúng ta đã làm quen với khái niệm request-response ròi cũng như đề cập tới RPC, vậy còn gRPC thì sao, nó có liên quan gì tới RPC không ? Câu trả lời là có. Nó là kết quả một quá trình hợp tác giữa `Google` và `Square` để  thay thế  có một công nghệ liên quan tới RPC riêng trước đó mà họ nghiên cứu với tên gọi là `Stubby`. Về  cơ bản bạn có thể  hiểu đơn giản đó là gRPC là một công nghệ mã nguồn được công khai từ bản mã `Stubby` trước đó và được bảo trợ dưới `Google`, và chữ `g` ở đây là chỉ để  ám chỉ `Google`. tuy vậy đừng vội nghĩ rằng nó là như vậy, vì sau bản cập nhật chính thức khi nhắc tới `gRPC` sẽ được ngầm hiểu đó là công nghệ `gRPC Remote Procedure Calls` và chữ `g` như một cách để ám chỉ `gRPC`. Ngoài ra với các bản cập nhật về  sau chữ `g` như một cách để  chỉ mã hiệu của các phiên bản vd như bản gRPC 1.1 nó có nghĩa là `good`, 1.2 nó có nghĩa `green`, etc ( để  biết chi tiết bạn có thể  xem tại [đây](https://github.com/grpc/grpc/blob/master/doc/g_stands_for.md)).

- Trước khi đi vào tìm hiểu chi gRPC trước tiên chúng ta cần phải hiểu rõ cơ bản một số  khái niệm về  HTTP & REST, HTTP/2, Message, Stream, etc.

### HTTP & REST

- Đầu tiên chúng ta phải nhắc tới khái niệm về  HTTP & REST. Như bạn biết trong phần đầu của chương chúng ta đã đề  cập tới giao thức Request-Response và 
thực hiện chúng thông qua `TCP`. Tuy nhiên nếu như trước đó thông qua một số  dòng lệnh, hay một số  ứng dụng desktop thì ngày nay với sự xuất hiện của World Wide Web (www) cùng với đó là rất nhiều trang web, HTTP đã được tạo lên và được tích hợp vào trong các trình duyệt web. HTTP là viết tắt của cụm từ ( Giao thức truyền tải siêu văn bản ) `Hyper-Text Transfer Protocol`, nó được xậy dựng ở tầng Application của mô hình `TCP/IP` và được sử  dụng như một cách để  giao tiếp giữa các client với các các server trong mô hình client/server trong giao thức Request-Response. Bạn có thể  hình dung thông qua sơ đồ  sau: 

```
Application Layer -- e.g. HTTP
----
Transport Layer -- e.g. TCP
----
Internet Layer -- e.g. IP
----
Link Layer -- e.g. IEEE 802.2
```

- Vậy còn Rest là gì ? REST là viết tắt của cụm từ `REpresentational State Transfer`, đây là một công nghệ, kiến trúc được xây dựng dựa trên giao thức HTTP. Trong công nghệ này sẽ đưa ra một số  các ràng buộc, quy ước dựa trên giao thức HTTP nhằm cung cấp một chuẩn tương tác cũng như khả năng mở rộng đối với một hệ thống. Nói một cách đơn giản với giao thức HTTP nếu như chúng ta tuân theo một bộ quy tắc các ràng buộc và quy ước này vào hệ thống chúng ta sẽ xây dựng được một hệ thống REST. Vậy chính xác nhưng quy ước đó là gì ? Trước khi đi vào tìm hiểu tôi sẽ giới thiệu một vài thành phần trong kiến trúc REST trước đó đã:

- Resources ( tài nguyên ): Chính xác thì tài nguyên là gì ? Cơ bản tài nguyên ở đây bất cứ thứ gì có thể  truyền tải được, ví dụ như một đoạn văn bản, danh sách người dùng, danh sách video, các bản nhạc, hình ảnh, etc. Chúng là bất cứ thứ gì mà chúng ta có thể  khai thác được trên mạng. Mỗi tài nguyên này sẽ ứng một đoạn URI được định nghĩa trong các service của hệ thống. vd khi ta nói 
tới `users/quan` thì nó mang hàm ý là sẽ lấy thông tin, có tên là `quan` và `users` ở đây mang ý nghĩa để biểu lộ danh sách người dùng.

- Representations: Ở phần trên khi gọi tới `users/quan`, chúng ta sẽ nhận được thông tin kết quả về  một người có tên là `quan` và hiển thị tạm như sau:

```json
{
   'name': 'quan',
   'age' : 26
}
```

- Vậy đoạn thông tin trên là một resources của user đúng ko ? Oh không hẳn vậy, nó là resource nhưng là một Representation của resource. Ở đây ta sẽ có định nghĩa một cách khái quát như sau : Representation là một cách để  biểu diễn các thực thể  ( `entities` ) của resource dưới nhiều dịnh dạng, vd như đoạn text đó có thể  là html, json, text, xml hay như một hình ảnh có thể  là png, jpg, etc. Trong đó khi client gửi một request tới server thông qua việc xác định tài nguyên cụ thể qua URI, nó sẽ gửi kèm một số  thông tin ( thường sẽ chứa trong `header` hay có thể  chức trong chính URI ), bằng cách này chúng ta có thể  xác định được định dạng, vd:

```
GET /users/quan HTTP/1.1
Host: localhost
Authorization: user:pass
Content-Type: application/json

{
   'name': 'quan',
   'age' : 26
}
```

- Ở đây chúng ta đã đính kèm một thông tin về  kiêu định dạng `json` mà server cần trả về .

- Tiếp theo chúng ta sẽ đi vào tìm hiểu các quy ước ràng buộc trong kiến trúc REST:

<br>
<p align="center">
  <img src="rest-architecture.png"/>
</p>
<br>

- Quy ước đầu tiên trong REST đó là Client/Server: Hệ thống chuẩn REST là một hệ thống hoạt động theo mô hình Client/Server. Trong Client có thể  là người sử  dụng thông qua một giao diện web hay một chương trình nào đó thực hiện kết nối tới Server, Còn Server cố thể  là một hay nhiều và chúng bao gồm các service có công việc là lắng nghe các request từ phía client và xử  lí ( lưu trữ dư liệu, lấy thông tin). Các request không nhất thiết phải xử  lí trên một server hay một service mà có thể phù thuộc vào tình huóng, yêu cầu mà xử  lí ở service khác nhau thông qua một cân bằng tải ( load balancer ). Bằng cách này giúp cho hệ thống có thể  dễ  dàng mở rộng hơn. 

- Statelessness ( phi trạng thái hay trạng thái không lưu trữ ): Một ứng dụng hay nói cách khác là theo mô hình client/server thì cả client lẫn server sẽ không lưu bất cứ trạng thái nào của nhau cả. Bất cứ khi nào client tạo ra một request tới server thì thông tin của client sẽ được đóng gói lại một cách đầy đủ và gửi kèm tới server. Server sẽ không cần phải lưu trữ ở một nơi nào đó mà chỉ cần trích xuất thông tin xử lí và trả lại cho client.

- Cacheable ( khả năng cache ) : Trong stateleness có một vấn đề  đó là mỗi khi client gửi một request nó yêu cầu kèm theo một gói tin mang thông tin đầy đủ về  client đó, cách làm này có lợi cho server nhưng bù lại khi chuyền tải trên mạng ( network ) độ trẽ có thể  sẽ cao khi phâỉ truyền tải gói tin qua lại giữa client-server. Do đó bằng phương pháp cache, server có thể  dựa trên đầu vào request của client , tiến hành xử  lí và cache dữ liệu lại, sau đó ở các lần request sau nếu nhưng không có gì thay đổi về  yêu cầu, server sẽ chỉ việc lấy từ cache ra và trả vê response. Bằng cách này client có thể  nhận được thông tin một cách nhanh chóng, giảm tải việc xử  lí các yêu cầu lặp lại trên server.

- Uniform interface ( Chuẩn hóa giao tiếp ): Đây là một trong những quy ước có thể  nó là quan trọng tạo lên sự khác biệt giữa kiến trúc REST và các kiến trúc khác như SOAP, HTTP Thông thường.
Bằng cách định nghĩa, đưa ra các quy ước thống nhất để  giao tiếp với các thành phẩn trong hệ thống ( bao gồm phương thức, định dạng giao tiếp, HATEOAS, etc ) REST sẽ giúp bạn đơn giản hóa khi hơn trong việc giao tiếp trao đổi giữa client và server. Như bạn biết trong HTTP 1.1 quy định khoảng 9 phương thức: 
  + GET: lấy thông tin của một hay nhiều tài nguyên.
  + POST: tạo mới một tài nguyên. 
  + PUT: tạo mới hay cật nhật lại thông tin đối với một tài nguyên.
  + DELETE: xóa một tài nguyên.
  + HEAD: lấy thông tin về  thành phần header 
  + OPTIONS: miêu tả các chức năng giao tiếp cho nguồn mục tiêu.
  + CONNECT: Thiết lập một tunnel tới Server được xác định bởi URI đã cung cấp.
  + PATCH: Cập nhật một thành phần, thuộc tính của tài nguyên
  + TRACE: Trình bày một vòng lặp kiểm tra thông báo song song với path tới nguồn mục tiêu.

- Trong 9 phương thức này thì POST và GET được sử  dụng trong hầu hết các trường hợp , thậm chí chúng còn được thay thế  cho PUT và DELETE. Điều này đôi khi khiến cho người đọc có thể  rối, khó hiểu do đó REST khuyến nghị việc sử  dụng các phương thức này đúng với ý nghĩa mà được tạo ra trong HTTP methods.  

- Layered system ( phân lớp hệ thông ): Để  giảm bớt sự phức tạp cũng như tăng khả năng mở rộng kiến trúc REST cho phép bạn tách biệt các chức năng thành các lớp khác nhau ví dụ như bạn có thể  đặt chức năng xử  lí auth ở một service gọi là A, sau đó đạt chức năng lưu trữ thông tin client ở service gọi là B, chức nằng Upload ảnh ở service gọi là C, và giao tiếp chúng với nhau qua HTTP. 


- Vậy rốt cục thì REST và HTTP có liên quan gì tới RPC mà chúng ta đang nói tới. Oh có nếu như bạn tìm hiểu về  công nghệ cũng như các hệ thống RPC, bạn sẽ thấy có rất nhiều các công nghệ khác nhau dựa trên RPC và dựa trên kiến trúc HTTP, REST, vd như Swagger hay XML-RPC, JSON-RPC. Và để  hiểu rõ cũng như có sự so sánh chúng ta sẽ đi sau vào một số  công nghệ có liên quan tới RPC:


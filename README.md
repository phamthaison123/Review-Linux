# Dữ liệu ngoài chuỗi

Mẫu này minh họa cách bạn có thể sử dụng [Dịch vụ sự kiện dựa trên kênh ngang hàng](https://hyperledger-fabric.readthedocs.io/en/latest/peer_event_services.html)
để sao chép dữ liệu trên mạng chuỗi khối của bạn sang cơ sở dữ liệu ngoài chuỗi.
Sử dụng cơ sở dữ liệu ngoài chuỗi cho phép bạn phân tích dữ liệu từ mạng của mình hoặc
xây dựng bảng điều khiển mà không làm giảm hiệu suất của ứng dụng của bạn.

Mẫu này sử dụng [Fabric network event listener](https://hyperledger.github.io/fabric-sdk-node/release-1.4/tutorial-channel-events.html) từ Node.JS Fabric SDK để ghi dữ liệu vào trường hợp địa phương của
Đi văngDB.

## Bắt đầu

Mẫu này sử dụng mã ứng dụng Node Fabric SDK để kết nối với một phiên bản đang chạy
của mạng thử nghiệm Fabric. Hãy chắc chắn rằng bạn đang chạy như sau
các lệnh từ thư mục `off_chain_data/legacy-javascript`.

### Khởi động mạng

Sử dụng lệnh sau để bắt đầu mạng mẫu:

```
./startFabric.sh
```

Lệnh này sẽ triển khai một phiên bản của mạng thử nghiệm Fabric. Mạng lưới
bao gồm một dịch vụ đặt hàng, hai tổ chức ngang hàng với một tổ chức ngang hàng, và
một CA cho mỗi tổ chức. Lệnh này cũng tạo một kênh có tên `mychannel`. Các
Chuỗi mã `asset-transfer-basic` sẽ được cài đặt trên cả hai thiết bị ngang hàng và được triển khai cho
kênh.

### Cấu hình

Cấu hình cho trình nghe được lưu trữ trong tệp `config.json`:

```
{
     "peer_name": "peer0.org1.example.com",
     "channelid": "kênh của tôi",
     "use_couchdb": đúng,
     "create_history_log":đúng,
     "couchdb_address": "http://admin:password@localhost:5990"
}
```

`peer_name:` là đồng đẳng đích của người nghe.
`channelid:` là tên kênh cho các sự kiện khối.
`use_couchdb:` Nếu được đặt thành true, các sự kiện sẽ được lưu trữ trong phiên bản cục bộ của
Đi văngDB. Nếu được đặt thành false, thì chỉ nhật ký sự kiện cục bộ sẽ được lưu trữ.
`create_history_log:` Nếu đúng, một tệp nhật ký cục bộ sẽ được tạo với tất cả
khối thay đổi.
`couchdb_address:` là địa chỉ cục bộ cho cơ sở dữ liệu CouchDB ngoài chuỗi có tên người dùng và mật khẩu.

### Tạo một phiên bản của CouchDB

Nếu bạn đặt tùy chọn "use_couchdb" thành true trong `config.json`, thì bạn có thể chạy
lệnh sau bắt đầu một phiên bản cục bộ của CouchDB bằng docker:

```
docker run -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password --publish 5990:5984 --detach --name offchaindb couchdb:3.2.2
khởi động docker offchaindb
```


### Cài đặt phụ thuộc

Bạn cần cài đặt Node.js phiên bản 8.9.x để sử dụng mã ứng dụng mẫu.
Thực hiện các lệnh sau để cài đặt các phụ thuộc cần thiết:

```
cài đặt npm
```

### Bắt đầu Trình xử lý Sự kiện Kênh

Sau khi chúng tôi đã cài đặt các phụ thuộc ứng dụng, chúng tôi có thể sử dụng SDK Node.js
để tạo danh tính mà ứng dụng nghe của chúng tôi sẽ sử dụng để tương tác với
mạng. Chạy lệnh sau để đăng ký người dùng quản trị:
```
nút đăng kýAdmin.js
```

Sau đó, bạn có thể chạy lệnh sau để đăng ký và đăng ký một ứng dụng
người dùng:

```
nút đăng kýUser.js
```

Sau đó, chúng tôi có thể sử dụng người dùng ứng dụng của mình để bắt đầu trình xử lý sự kiện khối:

```
nút blockEventListener.js
```

Nếu lệnh thành công, bạn sẽ thấy đầu ra của người nghe đang đọc
các khối cấu hình của `mychannel` ngoài các khối đã ghi
sự chấp thuận và cam kết của định nghĩa mã chuỗi nội dung.

`blockEventListener.js` tạo một trình nghe có tên "offchain-listener" trên
kênh `mychannel`. Người nghe ghi từng khối được thêm vào kênh vào một
bản đồ xử lý được gọi là BlockMap cho mục đích lưu trữ và đặt hàng tạm thời.
`blockEventListener.js` sử dụng `nextblock.txt` để theo dõi khối mới nhất
đã được người nghe truy xuất. Số khối trong `nextblock.txt` có thể là
đặt thành số khối trước đó để phát lại các khối trước đó. Tập tin
cũng có thể bị xóa và tất cả các khối sẽ được phát lại khi trình nghe khối
đã bắt đầu.

`BlockProcessing.js` chạy dưới dạng trình nền và kéo từng khối theo thứ tự từ
Bản đồ khối. Sau đó, nó sử dụng bộ đọc-ghi của khối đó để trích xuất thông tin mới nhất
dữ liệu giá trị khóa và lưu trữ nó trong cơ sở dữ liệu. Các khối cấu hình của
mychannel không có bất kỳ dữ liệu nào vào cơ sở dữ liệu vì các khối không chứa
bộ đọc-ghi.

Trình lắng nghe sự kiện kênh cũng ghi siêu dữ liệu từ mỗi khối vào tệp nhật ký
được định nghĩa là channelid_chaincodeid.log. Trong ví dụ này, các sự kiện sẽ được ghi vào
một tệp có tên `mychannel_basic.log`. Điều này cho phép bạn ghi lại lịch sử của
những thay đổi được thực hiện bởi mỗi khối cho mỗi khóa ngoài việc lưu trữ giá trị mới nhất
của nhà nước thế giới.

**Lưu ý:** Để blockEventListener.js chạy trong cửa sổ đầu cuối. Mở một
cửa sổ mới để thực hiện các phần tiếp theo của bản demo.

### Tạo dữ liệu trên blockchain

Bây giờ, trình nghe của chúng tôi đã được thiết lập, chúng tôi có thể tạo dữ liệu bằng mã chuỗi nội dung
và sử dụng ứng dụng của chúng tôi để sao chép dữ liệu vào cơ sở dữ liệu của chúng tôi. Mở một cái mới
terminal và điều hướng đến thư mục `fabric-samples/off_chain_data/legacy-javascript`.

Bạn có thể sử dụng tệp `addAssets.js` để thêm dữ liệu mẫu ngẫu nhiên vào blockc

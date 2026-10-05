Đánh giá Spotify Web API theo 9 tiêu chí
01. Tài nguyên là danh từ (Resource Naming)
  - Đánh giá: Đạt (Rất tốt)  
  - Chi tiết:  
    URL tập trung vào danh từ chỉ thực thể âm nhạc và người dùng:    
    /v1/tracks/{id}    
    /v1/albums/{id}    
    /v1/artists/{id}/top-tracks    
    /v1/playlists/{playlist_id}/tracks    
    /v1/me (Tài nguyên đại diện cho Current User profile)  
  - Method thể hiện action rõ ràng:  
    GET /v1/me/playlists: Lấy danh sách playlist của người dùng    
    POST /v1/users/{user_id}/playlists: Tạo playlist mới    
    DELETE /v1/playlists/{playlist_id}/tracks: Xóa bài hát khỏi playlist  
   - Ngoại lệ: Nhóm API Player điều khiển playback có một số endpoint mang tính hành động như /v1/me/player/pause, /v1/me/player/next do tính chất RPC/remote control của trình phát nhạc

02. Naming nhất quán (Consistent Naming)
  - Đánh giá: Đạt (Khá)  
  - Chi tiết:  
    Path: Chữ thường, các từ ghép dùng kebab-case hoặc danh từ số nhiều rõ ràng: /top-tracks, /audio-features, /currently-playing    
    Query parameter: Dùng snake_case nhất quán: ?limit=20, ?offset=0, ?market=VN, ?include_groups=album,single    
    Collection: Đồng nhất sử dụng danh từ số nhiều: /albums, /artists, /playlists, /tracks, /categories

03. Status code đúng nghĩa (Proper HTTP Status Codes)
  - Đánh giá: Đạt (Rất tốt)
  - Chi tiết:  
    Spotify tuân thủ rất tốt quy ước HTTP status code, không trả về 200 OK giả khi có lỗi:    
    200 OK: Trả dữ liệu thành công    
    201 Created: Tạo playlist hoặc thêm resource thành công    
    204 No Content: Các lệnh điều khiển player thành công (play, pause, next) hoặc khi follow/unfollow một artist    
    304 Not Modified: Hỗ trợ caching qua ETag (nếu nội dung không đổi)    
    401 Unauthorized: Token hết hạn (Access token expired) hoặc không hợp lệ    
    403 Forbidden: Người dùng không có quyền (ví dụ cố gắng chỉnh sửa playlist riêng tư của người khác)    
    429 Too Many Requests: Vượt ngưỡng Rate Limit

04. Idempotency rõ ràng
  - Đánh giá: Đạt một phần (Trung bình)  
  - Chi tiết:
    Các phương thức PUT (ví dụ PUT /v1/me/following để follow artist) và DELETE đều là idempotent    
    Chưa có Idempotency-Key cho POST: Các thao tác như POST /v1/playlists/{id}/tracks (thêm bài hát vào playlist) nếu gửi lại 2 lần do mạng lag sẽ dẫn đến việc bài hát đó bị nhân đôi (duplicate) trong playlist
    
05. Error response có cấu trúc (Structured Error Response)
  - Đánh giá: Đạt (Tốt)  
  - Chi tiết:  
    Spotify bọc lỗi trong object error với cấu trúc rất ngắn gọn, rõ ràng:
    {
      "error": {
        "status": 401,
        "message": "The access token expired"
      }
    }
    Một số endpoint xác thực OAuth trả thêm mã lỗi chi tiết error_description
    Điểm trừ: Spotify chưa áp dụng chuẩn RFC 7807 (type, title, detail, instance)
    
06. Pagination rõ ràng
  - Đánh giá: Đạt (Xuất sắc)  
  - Chi tiết:  
    Spotify hỗ trợ cả Offset-based và Cursor-based Pagination (đối với danh sách follow)    
    Mô hình Paging Object rất chuẩn và tiện lợi, trả về đầy đủ metadata:
    {
      "items": [...],
      "href": "https://api.spotify.com/v1/me/tracks?offset=0&limit=20",
      "limit": 20,
      "offset": 0,
      "total": 134,
      "next": "https://api.spotify.com/v1/me/tracks?offset=20&limit=20",
      "previous": null
    }
    Client chỉ cần gọi trực tiếp URL trong trường next để tải trang tiếp theo
    Giới hạn (Limit): Mặc định là 20, chặn trần tối đa thường là limit=50
    
07. Filter/Sort đa dạng
  - Đánh giá: Đạt (Tốt)  
  - Chi tiết:  
    Search & Filter: Endpoint GET /v1/search cực kỳ mạnh mẽ, hỗ trợ cú pháp lọc trường chuyên sâu qua query:    
      q=track:Stay artist:Justin Bieber year:2020-2022 genre:pop    
      Hỗ trợ lọc theo thị trường địa lý qua tham số market=VN  
  - Điểm trừ: Không hỗ trợ sparse fieldsets (không chọn được trả về mỗi id, name, duration_ms mà buộc phải nhận trọn vẹn Track Object tương đối nặng).

08. Authentication & Security
  - Đánh giá: Đạt (Xuất sắc)  
  - Chi tiết:  
    Authentication: Tuân theo chuẩn OAuth 2.0 (Authorization Code Flow, Client Credentials Flow)    
    Token bắt buộc truyền qua Header: Authorization: Bearer <access_token>. Tuyệt đối không cho phép truyền token trên URL  
    Rate Limit: Áp dụng rolling window. Khi bị throttle, trả về mã 429 Too Many Requests kèm header Retry-After: <số giây> chỉ định rõ client phải đợi bao lâu trước khi thử lại

09. Versioning + Deprecation
  - Đánh giá: Đạt một phần (Trung bình - Khá)  
  - Chi tiết:  
    Versioning: Sử dụng URL prefix ngay từ đầu ([https://api.spotify.com/v1/](https://api.spotify.com/v1/)...)  
    Deprecation Policy: Spotify có trang tin tức/changelog nhà phát triển và gửi email thông báo deprecation  
    Điểm trừ: Spotify rất ít khi nâng lên /v2/ mà thường xóa bỏ trực tiếp các trường hoặc endpoint cũ trong chính /v1/ (ví dụ: từng đóng một loạt Audio Analysis/Recommendations endpoint cho free tier mà không đổi version prefix), đôi khi gây breaking changes cho các thư viện bên thứ ba.

# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** 2L **Thành viên:** Nguyễn Thành Luân - 2A202602769, Nguyễn Đức Long - 2A202602917

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | strongsort | 0.3 | 0.5 | Theo số liệu có nhãn: bỏ sót là lỗi chính (FN 14633/18581 người), hộp giả ít (FP 236), đổi ID ít (IDSW 29). | conf 0.15 (HOTA 29.3, FP 1416, IDSW 118); conf 0.5 (HOTA 26.6, FN 15514); iou 0.7 (HOTA 29.3, IDSW 58); bytetrack (HOTA 26.9), botsort (HOTA 23.1) |
| video_2 (phố đêm, tĩnh, rất đông) | strongsort | 0.3 | 0.5 | strongsort phát hiện nhiều người hơn bytetrack (bytetrack bỏ qua), nhưng các hộp thêm đó không ổn định, hay mất. Cả hai đều có trường hợp một người bị mất hộp khi người khác đi sát, rồi hiện lại với ID mới. | bytetrack, botsort (bỏ sót nhiều hơn; botsort giống bytetrack ở cảnh này) |
| video_3 (camera di động, ảnh nhỏ) | strongsort | 0.3 | 0.5 | Một người bị mất hộp rồi hiện lại: bytetrack gán ID mới, strongsort nối lại đúng ID cũ (ID3). | bytetrack (đổi ID sau khi mất) |
| video_4 (trong nhà, camera di chuyển) | strongsort | 0.3 | 0.5 | strongsort nhận ra người sớm hơn bytetrack. Không có hộp giả trên kính phản chiếu, mọi hộp đều là người. | bytetrack (nhận người muộn hơn) |
| video_5 (trên xe bus, giao lộ đông) | strongsort | 0.3 | 0.5 | Một người bị biển báo che: bytetrack đổi ID, strongsort giữ đúng ID. | bytetrack (đổi ID khi bị che) |

Ghi chú: `iou` và `conf` chỉ được quét đầy đủ trên `video_1` (có nhãn). Bốn video còn lại dùng `conf=0.3`, `iou=0.5` mặc định, chưa quét.

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

Cấu hình nộp: `strongsort`, `conf=0.3`, `iou=0.5`, chạy đủ 600 frame.

```
HOTA: nop_bai-pedestrian           HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            30.387    17.797    52.05     18.197    80.812    54.818    85.543    84.286    30.75     36.536    80.22     29.309

CLEAR: nop_bai-pedestrian          MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag
video_1                            19.821    82.198    19.977    21.248    94.359    11.29     20.968    67.742    16.039    3948      14633     236       29        7         13        42        139

Identity: nop_bai-pedestrian       IDF1      IDR       IDP       IDTP      IDFN      IDFP
video_1                            31.97     19.585    86.974    3639      14942     545

Count: nop_bai-pedestrian          Dets      GT_Dets   IDs       GT_IDs
video_1                            4184      18581     67        62
```

Tóm tắt: HOTA 30.4, MOTA 19.8, IDF1 32.0. Recall thấp (21%), độ chính xác cao (94%): lỗi chính là bỏ sót người, không phải hộp giả hay đổi ID.

So sánh các tracker (conf 0.3, iou 0.5, video_1):

| Tracker | HOTA | MOTA | IDF1 | IDSW |
|---|---|---|---|---|
| strongsort | 30.4 | 19.8 | 32.0 | 29 |
| ocsort | 28.8 | 19.9 | 30.2 | 36 |
| deepocsort | 28.7 | 19.9 | 29.8 | 32 |
| bytetrack | 26.9 | 17.3 | 25.7 | 12 |
| botsort | 23.1 | 17.0 | 20.7 | 34 |

Quét `conf` (strongsort, iou 0.5): 0.15 → HOTA 29.3, FP 1416, IDSW 118; 0.3 → HOTA 30.4; 0.5 → HOTA 26.6, FN 15514. Quét `iou` (conf 0.3): 0.4 → HOTA 30.37; 0.5 → 30.39; 0.7 → 29.3, IDSW 58.

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

**video_3 (camera di động, ảnh nhỏ).** Có một người bị mất hộp rồi xuất hiện lại. bytetrack gán ID mới cho người đó, còn strongsort nối lại đúng ID cũ. bytetrack chỉ dựa vào chuyển động, mà camera di chuyển làm vị trí dự đoán lệch so với thực tế, nên khi người hiện lại ở chỗ khác nó không ghép được. strongsort có thêm so sánh ngoại hình (Re-ID), nên nhận ra người đó dù vị trí đã đổi. Vì vậy camera di động làm tracker có Re-ID hợp hơn. Đây là quan sát trên một người cụ thể, chưa đo trên toàn video.

**video_5 (xe bus, giao lộ đông).** Một người bị biển báo che khuất. bytetrack đổi ID khi người đó hiện lại, strongsort giữ đúng ID. Cơ chế giống video_3: camera rung và di chuyển làm dự đoán chuyển động kém tin cậy, trong khi ngoại hình vẫn dùng được để nối lại track sau khi bị che.

**video_2 (phố đêm, rất đông).** Ở đây hai tracker giống nhau hơn. Khi một người đi sát vào người khác, hộp của người bị che biến mất rồi hiện lại với ID mới, ở cả bytetrack lẫn botsort. Hai tracker cùng mất hộp một lúc, nên nhiều khả năng nguyên nhân nằm ở detector (người bị che, thiếu sáng, ảnh nhỏ) hơn là ở thuật toán ghép; chúng tôi chưa vẽ detection thô để xác nhận. strongsort bắt được nhiều người hơn bytetrack nhưng các hộp thêm này nhấp nháy. Ban đêm ngoại hình khó phân biệt nên lợi thế của Re-ID ít rõ hơn ở video_3 và video_5.

**video_1 (có nhãn).** strongsort có HOTA và IDF1 cao nhất trong năm tracker (30.4 và 32.0) và ít đổi ID (29). Lỗi lớn nhất của mọi tracker là bỏ sót người (FN khoảng 14600/18581): hạ `conf` xuống 0.15 bắt thêm khoảng 1350 người nhưng hộp giả tăng gấp 6 lần và đổi ID tăng gấp 4 lần nên HOTA giảm. Ngưỡng detector không bù được giới hạn của detector cố định ở ảnh 640 px.

## 4. Nếu có thêm thời gian

Chúng tôi sẽ vẽ thêm detection thô (hộp xám) cạnh track để tách lỗi bỏ sót của detector khỏi lỗi mất track của tracker, nhất là ở video_2. Chúng tôi cũng sẽ quét `conf` mịn hơn (0.2 đến 0.4) và quét `conf`/`iou` cho cả bốn video không có nhãn thay vì dùng giá trị mặc định.

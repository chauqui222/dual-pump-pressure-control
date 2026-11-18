# Dual Pump Pressure Control

**Dự án cá nhân mô phỏng điều khiển áp suất trạm bơm hai bơm bằng Siemens PLC, HMI và SCADA.**

Hệ thống vận hành theo nguyên tắc **Duty/Standby**: một bơm chạy để duy trì áp suất, bơm còn lại dự phòng. Dự án bao gồm phân tích yêu cầu, thiết kế hệ thống, lập trình PLC, xây dựng giao diện và chuẩn bị kiểm thử nghiệm thu mô phỏng.

**Video demo:** https://www.youtube.com/watch?v=ZnKtfwD-krY

## Chức năng chính

| Chức năng | Mô tả |
|---|---|
| Manual | Chọn P1/P2, START/STOP và đặt tốc độ trực tiếp trên HMI. |
| Auto và PID | Chọn Duty và điều chỉnh tốc độ để áp suất PV bám setpoint SP. |
| Duty alternation | Luân phiên bơm ưu tiên ở mỗi Auto START hợp lệ; có thể chọn bơm còn lại nếu bơm ưu tiên không sẵn sàng. |
| Failover | Trong Auto, lỗi bơm đang chọn kích hoạt chuyển sang Standby sau điều kiện xác nhận dừng; giới hạn một lần chuyển tự động mỗi chu trình. |
| Interlock | Chọn một bơm nhận RUN, yêu cầu phản hồi dừng của bơm còn lại. |
| Trip và Reset | Thiếu nước hoặc áp suất cao gây Trip khi đang vận hành; Reset yêu cầu điều kiện hồi phục và không tự tạo lệnh RUN. |
| HMI | Vận hành, trạng thái bơm/trạm, áp suất, Alarm/ACK và Trend. |
| SCADA | Hiển thị Trend lịch sử SP/PV và ghi dữ liệu mỗi giây. |

STOP hoặc chuyển Manual/Auto hủy chu trình hiện tại và yêu cầu START mới. Lỗi bơm trong Manual làm dừng chu trình, không tự chuyển bơm.

## Công nghệ sử dụng

- **Engineering:** Siemens TIA Portal V20.
- **PLC:** S7-1500 CPU 1511-1 PN, mô phỏng bằng S7-PLCSIM V20.
- **Ngôn ngữ:** LAD cho logic điều khiển; SCL cho mô hình áp suất.
- **Điều khiển:** state machine và PID_Compact.
- **Giao diện:** Siemens WinCC HMI Runtime và WinCC Unified PC Runtime V20.
- **Dữ liệu:** Pressure_Log, chu kỳ ghi 1 giây;

## Cấu trúc chương trình

| Khối | Vai trò |
|---|---|
| `Main [OB1]` | Gọi mô phỏng VFD, điều khiển trạm bơm và ánh xạ alarm. |
| `FB_PumpStation` | Manual/Auto, Duty selection, Failover, Trip/Reset và lệnh bơm. |
| `FB_VFD_Sim` × 2 | Tạo phản hồi Ready, Running, Stopped và Fault cho hai bơm. |
| `FB_Process_Sim` | Mô hình áp suất bậc nhất, dải 0–10 bar. |
| `OB_Process [OB30]` | Gọi mô hình áp suất, tính điều kiện cho phép PID và gọi PID_Compact. |
| `DB_IO`, `DB_HMI`, `DB_Sim`, `DB_Alarms` | Dữ liệu thiết bị, lệnh vận hành, tín hiệu giả lập và alarm. |

State của trạm: **0 IDLE → 10 STARTING → 20 RUNNING**; **30 FAILOVER** khi chuyển bơm và **40 TRIPPED** khi chốt Trip.

## Chạy mô phỏng

1. Chuẩn bị TIA Portal V20, S7-PLCSIM V20 và các thành phần WinCC phù hợp với thiết bị HMI/SCADA trong project.
2. Mở file Archive `.zap20` trong TIA Portal và khôi phục vào thư mục làm việc.
3. Compile PLC/HMI/SCADA, tải chương trình vào PLCSIM và đặt CPU ở RUN.
4. Khởi chạy HMI Runtime và WinCC Unified PC Runtime; kiểm tra kết nối đọc SP/PV. PLC được cấu hình IP `192.168.0.1`.
5. Đặt `DB_Sim.Water_Level_OK = TRUE`, xóa hai `InjectFault`; bảo đảm bơm dừng và không còn Trip trước khi START.
6. Thử Manual, Auto/PID, luân phiên và tạo lỗi Duty trong Auto. Sử dụng FAT Report để đánh dấu kết quả từng test.

SCADA chỉ đọc SP/PV và lưu lịch sử; các lệnh vận hành thực hiện trên HMI.

## Tài liệu và kiểm thử

| Tài liệu | Nội dung |
|---|---|
| `FDS_Dual_Pump_Pressure_Control.docx` | Yêu cầu chức năng và phạm vi mô phỏng. |
| `System_Architecture_Diagram.docx` | Kiến trúc mô phỏng và thiết bị dự kiến. |
| `IO_List.xlsx` | Danh sách tín hiệu và ánh xạ dữ liệu. |
| `BOM.xlsx` | Danh sách thiết bị dự kiến. |
| `FAT_Simulation_Report_Dual_Pump.docx` | 15 test case với tiêu chí nghiệm thu và cột PASS/FAIL. |

**Trạng thái:** đã lập trình và chạy mô phỏng các chức năng chính.

## Giới hạn

- Đây là **dự án mô phỏng**; chưa thực hiện đấu dây, tích hợp VFD thật hoặc commissioning hiện trường. G120 và động cơ 7.5 kW là thiết bị dự kiến.
- Phản hồi VFD được giả lập tức thời. Áp suất được tạo trực tiếp bằng bar; chưa có scaling 4–20 mA, Sensor Fault, chẩn đoán truyền thông hay telegram PROFINET với G120.

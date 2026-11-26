# JJClab_HW13
參考筆記: https://github.com/wsunccake/sle15_notes/blob/master/content/ch4.md

# 練習 0. 實驗拓樸與通用操作模式
## 0-1. 角色與互通

| 小題  | 任務                                                                 |回答|
| ----- | -------------------------------------------------------------------- |---|
| 0-1-1 | 指定哪台是 Server、哪台是 Client，寫下 hostname 與 IP                |Server: sle15-b, 192.168.146.100; Client: sle15-a, 192.168.146.129|
| 0-1-2 | Server `ping` Client、Client `ping` Server，記錄是否通、RTT          |有，如附圖|
| 0-1-3 | 說明後續為什麼幾乎每個服務都要「本機 active」加上「Client 測一次」   |Server 的服務狀態顯示 active，只能證明服務程序正常執行，無法保證 Client 可以連線。|

<img width="837" height="205" alt="image" src="https://github.com/user-attachments/assets/1668dd0e-92f5-4532-b086-f924a19116f3" />
<img width="843" height="198" alt="image" src="https://github.com/user-attachments/assets/b1b3844b-6565-4768-b116-056882e582af" />

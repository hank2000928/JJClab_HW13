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

## 0-2. 通用六步  
| 步驟 | 通用作法                         | 你會用的代表指令 |
| ---- | -------------------------------- | ---------------- |
| 1    | 安裝套件                         |  zypper install openssh |
| 2    | 編輯設定檔（必要時先備份）      |  vi /etc/ssh/sshd_config  |
| 3    | 檢查語法（若工具支援）          |  sshd -t  |
| 4    | 開機啟用並立即啟動              |  systemctl enable --now sshd  |
| 5    | 防火牆永久放行後 reload          |  firewall -cmd --permanent --add-sercive=ssh ; firewall-cmd --reload|
| 6    | 從 Client 驗證功能               | ssh 使用者名稱@server-IP|

# 練習 1. systemd：服務與開機管理
## 1-1. 演進與 runlevel／target
### 1-1-1  以表格對照 **SysVinit**、**Upstart**、**systemd** 至少三項差異 
|比較項目	|SysVinit|	Upstart|	systemd|
|---|---|---|---|
|啟動方式	|依腳本順序執行	|事件驅動	|相依關係與平行啟動|
|服務管理	|service、/etc/init.d/	|initctl|	systemctl|
|啟動管理	|Runlevel	|事件與相容 Runlevel|	Target|
|服務設定	|Shell 腳本	|Job 設定檔	|Unit 檔案|
|適用情境	|傳統 Linux	|過渡時期的部分發行版|	現代主流 Linux|


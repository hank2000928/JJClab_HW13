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
### 1-1-1 以表格對照 **SysVinit**、**Upstart**、**systemd** 至少三項差異 
|比較項目	|SysVinit|	Upstart|	systemd|
|---|---|---|---|
|啟動方式	|依腳本順序執行	|事件驅動	|相依關係與平行啟動|
|服務管理	|service、/etc/init.d/	|initctl|	systemctl|
|啟動管理	|Runlevel	|事件與相容 Runlevel|	Target|
|服務設定	|Shell 腳本	|Job 設定檔	|Unit 檔案|
|適用情境	|傳統 Linux	|過渡時期的部分發行版|	現代主流 Linux|

### 1-1-2 SysV **runlevel 3**／**5** 分別對應哪個 systemd **target**？
### 1-1-3 runlevel 0／1／6 常見對應哪些 target？        
|SysV Runlevel|	systemd Target|	用途|
|---|---|---|
|0|	poweroff.target	|關機|
|1|	rescue.target	|救援／單使用者模式|
|3|	multi-user.target	|多使用者文字模式|
|5|	graphical.target	|圖形介面模式|
|6|	reboot.target	|重新開機|
### 1-1-4 查目前預設 target、改為 `multi-user.target` 的指令各是什麼？  
<img width="841" height="164" alt="image" src="https://github.com/user-attachments/assets/7bdc90ef-8ccf-4863-966d-5618daf3437f" />
系統知道指令的位置，但一般使用者無法直接透過目前的 PATH 找到它，改用"sudo /usr/sbin/runlevel"

### 1-1-5 想在命令列切到文字多使用者模式，應使用哪道指令？`isolate` 有何風險？ 
|指令	|功能	|是否立即切換|
|---|---|---|
|systemctl get-default	|查詢預設 Target	|否|
|systemctl set-default multi-user.target	|修改下次開機的預設模式	|否|
|systemctl isolate multi-user.target	|立即切換到指定 Target|	是|

isolate 可能停止目前 Target 不需要的服務。如果透過 SSH 操作，可能影響連線。

### 1-1-6 舊文件寫 `service foo start` 或 `/etc/init.d/foo`，在 SLES 15 多半對應什麼？
|舊版|SLE15|
|---|---|
|service sshd start|sudo systemctl start sshd|
|service sshd stop|sudo systemctl stop sshd|
|/etc/init.d/sshd restart|sudo systemctl restart sshd|

## 1-2. 生命週期指令
|指令	|功能	|改變開機自啟？|
|---|---|---|
|start	|啟動服務	|否|
|stop	|停止服務|	否|
|restart	|停止後重新啟動|	否|
|reload	|要求服務重新讀取設定（須支援）|	否|
|enable	|設定開機自動啟動|	是|
|disable|	取消開機自動啟動|	是|
|enable --now|	設定開機自啟並立即啟動|	是|
|is-active	|查詢服務是否執行	|否|
|is-enabled|	查詢是否設定開機自啟|	否|

| 小題  | 問題                                                                 | 作答 |
| ----- | -------------------------------------------------------------------- | ---- |
| 1-2-1 | `list-units` 與 `list-unit-files` 差別？                             | list-units 主要查看 systemd 已載入的 Unit 及目前運作狀態；list-unit-files 主要查看 Unit 檔案與是否已設定開機自動啟動。     |
| 1-2-2 | 改 unit 檔後為什麼要 `daemon-reload`？忘記做會看到什麼現象？         | 修改 Unit 檔案後，需要執行 daemon-reload，讓 systemd 重新載入設定。若忘記執行，systemd 可能繼續使用舊的 Unit 定義，甚至提示設定檔已在磁碟上變更。     |
| 1-2-3 | 寫出「立即啟動 + 開機啟用 + 看日誌」的指令序列                       |systemctl enable --now sshd; systemctl status sshd; journalctl -u sshd -n 30 --no-pager|
| 1-2-4 | 日誌為何優先 `journalctl -u <svc>`，再查服務自己的 log 檔？          |journalctl 能協助判斷服務是否成功啟動，以及 systemd 是否有相關錯誤。若需要進一步分析服務內部行為，再查看該程式自己的日誌。|

<img width="839" height="278" alt="image" src="https://github.com/user-attachments/assets/a75c8718-82d7-4e30-9995-c791ab98f16a" />

## 1-3 自訂 unit（單機「服務端」部署）
### 1-3-1 寫出完整 `myapp.service`（含 `After`、`ExecStart`、`WantedBy`）
<img width="837" height="185" alt="image" src="https://github.com/user-attachments/assets/17f06384-b24e-4ad4-99ea-bf122923fba3" />
程式概念: 建立無限迴圈。只要程式沒有被停止，就會持續執行。輸出目前日期、時間，以及 myapp running。等待五秒，再回到迴圈開頭。
開始賦予執行權限等後續操作:  

<img width="838" height="707" alt="image" src="https://github.com/user-attachments/assets/daf06bfa-cf38-46a5-bc32-0519d4835d00" />
|區段／參數|	功能|
|---|---|
|[Unit]	|定義服務基本資訊及相依關係|
|Description=	|服務的文字說明|
|After=network.target	|啟動順序安排在 network.target 之後|
|[Service]	|定義程序如何執行|
|Type=simple|	systemd 直接管理前景執行的主程序|
|User=JJCadmin	|使用一般帳號執行程式|
|ExecStart=	|指定要執行的程式|
|Restart=on-failure	|程序異常失敗時自動重新啟動|
|[Install]	|定義 Unit 的啟用方式|
|WantedBy=multi-user.target|	設定 enable 時與 multi-user.target 建立啟用關係|
### 1-3-2 執行 `daemon-reload` → `enable --now` → `status` → `journalctl -u myapp`

<img width="837" height="684" alt="image" src="https://github.com/user-attachments/assets/b3cb3a98-9f4b-4833-bdd3-613f57a50cea" />
### 1-3-3 說明為何修改套件原廠 unit 時優先用 drop-in，而不是直接改 `/usr/lib/...` 
/usr/lib/systemd/system/ 主要存放套件提供的原廠 Unit，直接修改可能在套件更新時被覆蓋。使用 drop-in 可以將自訂設定保存在 /etc/systemd/system/，避免直接修改原廠檔案，也方便後續維護與追蹤設定變更。
### 1-3-4 | 實驗結束後 `disable --now myapp`（可選刪檔）
<img width="838" height="393" alt="image" src="https://github.com/user-attachments/assets/10d929a2-8e0b-48ea-a65d-ef3bf6352f4b" />

# 練習 2. SSH：Client／Server 架設
## 2-1. 協定演進
### 2-1-1 比較 `rsh`／`rlogin`、`Telnet`、`SSH` 在「傳輸是否加密」上的差異
| 協定 | 傳輸加密 | 用途 | 安全性 |
|---|---|---|---|
| rlogin | 無 | 遠端登入 | 不安全 |
| rsh | 無 | 遠端執行指令 | 不安全 |
| Telnet | 無 | 遠端互動式終端機 | 不安全 |
| SSH | 有 | 安全遠端登入、執行指令及傳檔 | 較安全 |
### 2-1-2 | SSH 的加密、多種驗證、完整性保護、擴充（SFTP／埠轉發）各代表什麼？ 
1. 加密（Encryption）: 加密保護 Client 與 Server 傳輸的資料，降低被旁觀者直接讀取的風險。
2. 多種驗證（Authentication）: SSH 支援多種登入驗證方式，例如密碼與公鑰驗證。
3. 完整性保護（Integrity）: SSH 透過加密協定中的完整性驗證機制，讓通訊雙方能偵測傳輸資料是否遭到未授權修改。
4. 擴充功能（Extensions）: SSH 不只能遠端登入，還支援 SFTP、SCP 與 Port Forwarding（埠轉發）等功能。
### 2-1-3 為什麼明文協定不該再用於管理面？  
因為 rlogin、rsh 與 Telnet 缺乏傳輸加密，攻擊者若能截取網路封包，可能讀取登入憑證及操作內容。SSH 提供加密、使用者與伺服器身分驗證，以及資料完整性保護，因此現代 Linux 遠端管理應優先使用 SSH。
## 2-2. Server 架設
### 2-2-1 安裝並使 `sshd` 開機啟用且立即 `active` 
<img width="839" height="608" alt="image" src="https://github.com/user-attachments/assets/4d069ae3-83ad-4be7-b803-1dade8f62d4a" />
### 2-2-2 防火牆永久放行 `ssh` 並 reload，記錄 `list-services` 
<img width="842" height="289" alt="image" src="https://github.com/user-attachments/assets/693e2bb7-d73e-4e2a-bb62-21c0ab2cb47b" />
### 2-2-3 | 說明為何改 `sshd_config` 前要備份，且改完要 `sshd -t` 再 `reload`
必須先備份，因為一旦錯誤修改 SSH 的登入方式、網路位址或連接埠，可能導致無法再從遠端登入。保留原始設定可以協助復原。
修改完成後，應先執行："sudo /usr/sbin/sshd -t"，其中-t 表示測試 SSH Server 設定檔的基本有效性，包括設定語法與部分必要條件。
確認檢查通過後，再執行 systemctl reload sshd，讓 SSH Server 重新讀取設定。相較於 restart，reload 通常可以降低對既有連線的影響。
## 2-3. Client 連線驗證
### 2-3-1 | 從 Client 以密碼（或既有金鑰）登入 Server，記錄指令與成功證明 

<img width="831" height="267" alt="image" src="https://github.com/user-attachments/assets/6f245e0a-bb0f-4889-855a-aceb43d77419" />
### 2-3-2 | （建議）Client 產生金鑰並把公鑰放到 Server 的 `authorized_keys`
<img width="840" height="611" alt="image" src="https://github.com/user-attachments/assets/4554bf43-fc0e-409d-88bc-a049a5b5b7b3" />
| 參數 | 意義 |
|---|---|
| -t rsa | 使用 RSA 金鑰 |
| -b 4096 | 金鑰長度為 4096 bits |
| -f | 指定儲存檔名 |
<img width="831" height="125" alt="image" src="https://github.com/user-attachments/assets/b711e1b2-6ab7-44b2-867b-bc4983535517" />
## 2-4. `sshd_config` 強化（實驗機；先確認金鑰可登入）
| 參數 | 用途 | 設定示例 |
|---|---|---|
| Port | SSH 監聽的 TCP 埠 | 22 |
| ListenAddress | 指定 SSH 監聽的本機 IP | 192.168.146.100 |
| PermitRootLogin | 控制 root 是否允許透過 SSH 登入 | no |
| PasswordAuthentication | 允許或禁止密碼驗證 | no |
| PubkeyAuthentication | 允許或禁止公鑰驗證 | yes |
| PermitEmptyPasswords | 是否允許空密碼帳號登入 | no |
| X11Forwarding | 是否允許轉送遠端圖形程式 | no |
| MaxAuthTries | 單次 SSH 連線允許的驗證嘗試次數 | 3 |
| AllowUsers / AllowGroups | 限制允許登入的使用者或群組 | JJCadmin |
| LoginGraceTime | 登入驗證完成前允許的時間 | 60s |
### 2-4-1 寫出禁止 root 密碼直登、改用金鑰的建議參數組合
|參數|設定|
|---|---|
|Port |22|
|PermitRootLogin| no|
|PubkeyAuthentication| yes|
|PasswordAuthentication| no|
|KbdInteractiveAuthentication |no|
|PermitEmptyPasswords |no|
|X11Forwarding |no|
|MaxAuthTries| 3|
|LoginGraceTime |60|
|AllowUsers |JJCadmin|
### 2-4-2 關閉密碼前必須先確認什麼？為什麼要留備援 session？
假設把 Server 設定成：PasswordAuthentication no
但 Client 的公鑰沒有正確設定，重新連線時就可能出現：Permission denied (publickey).
此時如果已經關閉原本的 SSH 視窗，就可能失去遠端登入能力。
因此正確的操作方式是：
1. 確認 sle15-a 使用公鑰可以登入 sle15-b
2. 保留一個已登入 sle15-b 的 SSH Session，並確保 VMware 本機主控台可使用
3. 備份原本的 sshd_config
4. 修改設定，執行 sshd -t 與有效設定檢查
5. 執行 systemctl reload sshd，從另一個新視窗再次測試登入

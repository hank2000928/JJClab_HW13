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
### 2-4-3 若改 `Port 2222`，列出必須同步修改的項目（Client 連線方式、防火牆等

| 項目 | 需要修改或確認的內容 |
|---|---|
| Server 設定 | sshd_config 設為 Port 2222 |
| 防火牆 | 允許 2222/tcp |
| SSH Client | 連線時增加 -p 2222 |
| SCP / SFTP | 使用對應的新埠參數 |
| 驗證 | 確認 2222 正在監聽且 Client 可登入 |
### 2-4-4 | 改非標準埠為什麼只是「模糊安全」，不能取代金鑰與權限控管？
SSH 的預設 Port 是 TCP 22。將它改成 2222，可能減少一些只針對 Port 22 的自動化登入嘗試，但不代表 SSH 就變得隱蔽，攻擊者仍可透過連接埠掃描找出提供 SSH 的新 Port。
故修改 SSH Port 主要能減少部分針對預設埠的自動化探測與攻擊流量，並不能真正隱藏 SSH 服務，也無法取代加密驗證、金鑰管理、使用者權限限制及防火牆管制。

<img width="849" height="396" alt="image" src="https://github.com/user-attachments/assets/e63d03ec-4879-47ed-99fd-050a9141ec60" />
<img width="839" height="752" alt="image" src="https://github.com/user-attachments/assets/ab6e4526-3b4d-4b71-ab5a-3f8d9e1c04bf" />
<img width="843" height="275" alt="image" src="https://github.com/user-attachments/assets/0ffa1be6-5418-4616-9523-8d2a47592c3d" />

# 練習 3. Chrony：內網時間 Server／Client 架設
## 3-1. 概念
### 3-1-1 比較 NTP、SNTP、Chrony 的適用情境（含 VM／不穩網路）

| 項目 | 本質 | 適用情境 |
|---|---|---|
| NTP | 網路時間同步協定 | 伺服器、工作站、叢集 |
| SNTP | 簡化的 NTP 同步機制 | 較簡單的裝置或時間同步用途 |
| Chrony | NTP 的軟體實作 | Linux、VM、網路不穩定的環境 |

###  3-1-2 Stratum 分層與「多伺服器冗餘」在防什麼？ 
Stratum 表示 NTP 時間來源的層級；多伺服器冗餘可以降低單一來源故障的影響，並協助辨識異常時間來源。Stratum 表示同步層級，從最精準的0號開始，但不代表 Stratum 數字越小，實際時間誤差就一定越低。
### 3-1-3 `iburst` 與 `makestep 1.0 3` 各自解決什麼問題？

| 數值 | 意義 |
|---|---|
| 1.0 | 時間誤差超過 1 秒時，允許直接調整系統時間 |
| 3 | 只在前 3 次時鐘更新中允許這種直接調整 |
### 3-1-4 `driftfile`、`rtcsync`、`allow`、`local stratum 10` 各做什麼？

| 參數 | 用途 |
|---|---|
| driftfile | 保存系統時鐘的頻率偏差估計，供下次啟動參考 |
| rtcsync | 允許透過核心機制定期將系統時間同步至硬體時鐘（RTC） |
| allow | 指定哪些 Client 網段可以向本機取得 NTP 時間 |
| local stratum 10 | 無法連接外部時間來源時，允許以本機時鐘作為備用參考 |

### 3-1-5 | 用 `chronyc tracking` 如何判讀是否已同步？
<img width="834" height="271" alt="image" src="https://github.com/user-attachments/assets/ea89cfbc-6258-46ab-8b61-934cae31a0a7" />
| 欄位 | 意義 |
|---|---|
| Reference ID | 目前參考的時間來源 |
| Stratum | 本機同步層級 |
| System time | 目前系統時間相對於 Chrony 估計 NTP 時間的偏差 |
| Last offset | 最近一次估計的時間偏差 |
| RMS offset | 一段時間內偏差的均方根估計 |
| Frequency | 系統時鐘的頻率校正量，單位 ppm |
| Leap status | 同步及閏秒相關狀態；Normal 通常表示正常同步狀態 |

## 3-2. Server 架設（對內網提供 NTP）
<img width="839" height="488" alt="image" src="https://github.com/user-attachments/assets/a2bb840c-fb01-4d7f-990a-5c16528f9be8" />
<img width="841" height="591" alt="image" src="https://github.com/user-attachments/assets/0e14f8fa-bfdd-4dfc-8572-fef2fee33774" />
<img width="847" height="537" alt="image" src="https://github.com/user-attachments/assets/f4bc3517-1fad-4490-a8c4-67a2f50cd5d4" />
<img width="839" height="539" alt="image" src="https://github.com/user-attachments/assets/433009ff-b19b-4dd2-9302-9511b4f8c311" />

## 3-3. Client 架設與驗證
### 3-3-1 Client 的時間源是否顯示為 Server IP？附 `chronyc sources` 
<img width="822" height="415" alt="image" src="https://github.com/user-attachments/assets/44a49841-f1a6-4bd4-bbdc-02f0bf605527" />
| 設定 | 意義 |
|---|---|
| server | 指定單一 NTP 時間伺服器 |
| 192.168.146.100 | sle15-b 的 IP |
| iburst | 初次建立同步時快速取得多次時間測量值 |

### 3-3-2 從 `tracking` 判斷是否同步（stratum、offset 概念）
<img width="825" height="506" alt="image" src="https://github.com/user-attachments/assets/75248027-b2a6-4c35-8e3b-db9e72a18980" />

| 檢查項目 | 實際結果 | 判讀 |
|---|---|---|
| 時間來源 | 192.168.146.100 | 確實指向 sle15-b |
| 來源狀態 | ^* | 已選定 sle15-b 作為同步來源 |
| 來源 Stratum | 3 | sle15-b 的同步層級 |
| Client Stratum | 4 | sle15-a 比 Server 低一個同步層級 |
| Reach | 17（八進位） | 最近四次輪詢成功，尚未累積到完整八次 |
| System time | 0.000000088 秒 | 約快 0.088 微秒 |
| Leap status | Normal | 目前時間同步狀態正常 |

### 3-3-3 說明 VM 環境為何更應使用 Chrony，並確認可達上游或內網 NTP
虛擬機的系統時間可能受到宿主機負載、CPU 排程、暫停及恢復等因素影響，產生時間偏移。Chrony 能持續估計系統時鐘的偏移與頻率誤差，並在網路不穩定或 VM 恢復運作後重新建立時間同步。
在本次實驗中，sle15-b 作為內網 NTP Server，先與外部 NTP 同步，再提供時間給 sle15-a。這能讓兩台 VM 使用一致的時間來源，有助於日誌分析、身分驗證及後續叢集服務。

# 練習 4. NIS：集中帳號 Client／Server 架設
計畫在 sle15-b 建立一個測試帳號，再讓 sle15-a 查詢它。
## 4-1. 概念

| 小題  | 問題                                                                 | 作答 |
| ----- | -------------------------------------------------------------------- | ---- |
| 4-1-1 | 說明 `ypinit -m` 與後續 `ypmake`／`make` 的時機與差異|ypinit -m 用於首次建立 NIS Master Server 的資料庫；ypmake 用於帳號、群組等來源資料變更後重新建置 NIS Map，以更新 Client 能查詢到的內容。|
| 4-1-3 | `/var/yp/securenets` 用途是什麼？|檔案應在第一次啟動 NIS Server 前完成設定。 如果不存在或沒有適當限制，NIS 可能接受更廣泛來源的請求。|
| 4-1-4 | 為什麼帳號一改，Server 必須重建 map，Client 才看得到？|NIS Server 以 Map 資料庫提供集中帳號資訊。當來源帳號或群組資訊變更時，需要重新建置 Map，讓 Client 查詢到更新後的資料。|

### 4-1-2 設計一組只允許 `192.168.1.0/24` 的 `/etc/ypserv.conf` 規則（含最後 deny）
示例規則如下：

127.0.0.1:*:*:none  
192.168.146.0/255.255.255.0:*:*:none  
*:*:*:deny  

規則的欄位依序是："host:domain:map:security"  
其中 none 代表允許，deny 代表拒絕；規則會依順序比對。  
第一行允許本機，第二行允許 VMware NAT 網段，第三行拒絕其他 IPv4 來源。  
不過，ypserv.conf 的這種規則主要針對 IPv4，還需要搭配 /var/yp/securenets 與防火牆，不能單靠它保護 NIS。  
## 4-2. Server 架設
### 4-2-1 設定 NIS domain（寫入 `defaultdomain`），初始化 master 資料庫 
<img width="857" height="132" alt="image" src="https://github.com/user-attachments/assets/bd819998-3c91-4020-b930-b03fbd3a1640" />

### 4-2-2 寫出 `ypserv.conf` 允許本機 + `192.168.1.0/24`、其餘 deny   

127.0.0.0/255.0.0.0:*:*:none: 允許本機 IPv4 Loopback 網段存取。  
192.168.146.0/255.255.255.0:*:*:none: 允許你的 VMware NAT 內網存取。  
*:*:*:deny: 拒絕其他未符合前面規則的 IPv4 來源。  
<img width="859" height="758" alt="image" src="https://github.com/user-attachments/assets/a624bb97-858c-45bb-ad9d-fde6ea4f9cfc" />
<img width="863" height="633" alt="image" src="https://github.com/user-attachments/assets/e1a27a85-5458-4f7e-9467-438d49793359" />

<img width="860" height="504" alt="image" src="https://github.com/user-attachments/assets/66843592-38e3-4ba4-a240-7763e577d06f" />

### 4-2-3 在 Server 建一個測試帳號後執行 `ypmake`，說明為何要重建  

ypmake／make 的用途：當來源帳號資料變更時，重新產生供 NIS Client 查詢的資料庫。
<img width="863" height="343" alt="image" src="https://github.com/user-attachments/assets/aae86a3d-c5ef-4b0c-b665-64785aba5ff2" />
<img width="860" height="782" alt="image" src="https://github.com/user-attachments/assets/b8f3bf21-7078-4d82-9943-b429f71c0e1b" />
<img width="863" height="631" alt="image" src="https://github.com/user-attachments/assets/abd8f5f9-d86d-4a56-9cce-81597985e3bc" />

### 4-2-4 `ypserv` 與 `rpcbind` 皆為 `active`   

<img width="862" height="611" alt="image" src="https://github.com/user-attachments/assets/44063e76-7ad9-4621-b76f-ca9713f73554" />
<img width="841" height="185" alt="image" src="https://github.com/user-attachments/assets/7725310c-5bf8-428e-8fc6-61dd73a6da8b" />
<img width="868" height="671" alt="image" src="https://github.com/user-attachments/assets/2c95b970-eae8-462a-9581-e677a8b9acb0" />
<img width="866" height="698" alt="image" src="https://github.com/user-attachments/assets/da98fb3b-88bc-4cd8-8491-75dab8e01e8f" />


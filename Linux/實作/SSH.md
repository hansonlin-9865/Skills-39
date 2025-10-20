# SSH

### 1. 安裝 SSH 套件

```bash
apt install ssh -y
```

### 2. 設定 SSH

進入 `sshd_config` 設定 SSH 內容

```bash
vi /etc/ssh/sshd_config
```

並將 Root 登入開啟以便管理<font color=red>（如題目規定不使用 Root 則不開啟）</font></br>
這裡也可以設定其他 SSH 的內容，像是加密連線之類的，此篇先以沒有任何要求狀態下設定。

輸入 `32` 按 Enter 因該會在 `PermitRootLonig` 選項前面，按 Delete 解掉註解，並將後面的 `prohibit-password` 刪除改成 `yes` 儲存退出

![image](Files/SSH/1.png)

### 3. 重啟 SSH Services

```bash
systemctl reload sshd
```

### 4. 連線測試

開啟其他主機並確任是同一個網域，開啟 Terminal 使用 SSH 連線到 root ，這裡以 Windows Server 連線 SFTP 做示範

開啟 CMD，並且輸入以下內容：（localhost 表示主機 IP，請輸入對應 IP 位置）

```bash
sftp root@localhost
```

![image](Files/SSH/1.png)

圖片中有詢問是否信任此主機憑證，打 Yes 就好。

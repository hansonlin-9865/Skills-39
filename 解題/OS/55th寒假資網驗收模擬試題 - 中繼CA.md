# 55th寒假資網驗收模擬試題 - 中繼CA

###### 題目作者：李祈緯

## 題目

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/1.png)

## 題目解析

需要在一部主機上創建一個 Root CA，創建完成後在 windc 上創建一個 Sub CA 的申請檔，在使用剛創建完的 Root CA 進行簽發及發佈，再將簽發後的憑證檔匯入 windc 做後需其他服務的憑證簽發。

## 解題

### 1. 設定 CRL 及 AIA 位置

題目中另一台 Server 為 lnxsrv (Debian)，所以我們使用 Easy-RSA 來做 Root CA 的創建工作。

先安裝完 Easy-RSA，然後不要急著初始化，先進入以下文件進行修改：

```bash
cd /usr/share/easy-rsa/
vi x509-types/COMMON
```

打開後裡面有 CRL 及 AIA 的設定，<font color=red>這裡必須設定，否則後續服務是無法啟動的！！</font>

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/2.png)

先將 crlDistributionPoints 和 authorityInfoAccess 取消註解，並將後方網址改成需要的網址及 `$.crl` 和 `$.crt` 檔案名稱，後面會架設站台做發佈註銷服務器，<font color=red>請一定要確認網址，不然全部要重頭開始！！</font>

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/3.png)

完成後 `:wq` 儲存並退出

### 2. 建立 Root CA

先初始化環境

```bash
./easyrsa init-pki
```

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/4.png)

創建 Root CA

```bash
./easyrsa build-ca
```

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/5.png)

系統會要求你輸入 CA key 密碼及 PEM 密碼，輸入後會詢問 Root CA 要叫什麼名字

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/6.png)

這裡依題目要求設定為 "Skills Root CA"

### 3. 取得 Sub CA 請求檔 (`.req`)

題目要求 Sub CA 要在 windc 上並且作為後續憑證簽發端，所以我們現在要先創建一個憑證申請檔，讓 Root CA 簽發並生成憑證鏈

先安裝 AD CS 服務，並進入到初始化介面

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/7.png)

Next -> 將第一個項目勾選 -> Next -> Next

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/8.png)

到了 CA Type 這裡選擇 "Subordinate CA" -> Next

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/9.png)

這裡加密方式選擇 "Create a new private key" -> Next -> Next

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/10.png)

這裡將 "Common name for this CA" 改成需要的 Sub CA 名稱，這裡依題目設定為 "SC2025 Sub CA" -> Next 到底

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/11.png)

按 Configure

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/12.png)

結果顯示驚嘆號是正常的，因為還沒導入憑證鏈，按 close 關閉

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/13.png)

### 4. 簽發 Sub-CA 並創建憑證鏈

設定完 Sub-CA 後 `C:\` 底下會有一個 `.req` 檔案，將他匯入到 lnxsrv 的 `/usr/share/easy-rsa/pki/reqs` 底下（建議可以先更改名稱不然預設名稱很長）

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/14.png)

然後在 lnxsrv 上面執行

```bash
./easyrsa sign-req ca $$$
```

- ca -> 簽發憑證
- $$$ -> req 檔名稱（不用加檔案格式）

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/15.png)

然後導出憑證鏈

```bash
./easyrsa export-p7 sub-CA
```

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/16.png)

### 5. 將憑證鏈匯入到 AD CS 中

將剛剛導出的 `.p7b` 檔匯入到 windc 中

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/17.png)

- lcd -> 本地端指定路徑
- ./desktop -> 預設目錄為 `C:\Users\Administrator`，指定底下的 desktop 資料夾（方便存取）

然後開啟 Certification Authority，將主機 右鍵 -> All Tasks -> Install CA Certificate...

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/18.png)

找到剛剛匯入的 `.p7b` 檔後按 Open

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/19.png)

這裡系統顯示這憑證鏈有些問題，不管他按 OK 繼續

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/20.png)

### 6. 設定 AD CS 的 CRL 及 AIA 位置

再來要設定 AD CS 的 CRL 和 AIA，你說剛剛不是新增了嗎？

不不不，剛剛新增的是 lnxsrv 上的 Easy-RSA 的發佈及註銷位置，但因為後需憑證都會由 AD CS 簽發，所以你還是要在設定一次，兩邊是分開的。

如果沒有設定呢？那就會發生簽出來的憑證和 Sub and Root CA 的 CRL 和 AIA 不一樣，對於一些服務來說是有影響的。

將主機 右鍵 -> Properties

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/21.png)

這裡會跳出提示，告訴你沒有啟動此服務，按 OK 繼續

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/22.png)

點擊上方的 Extensions，將 CDP 及 AIA 裡預設的項目都勾選掉，並加入需要的 CRL 及 AIA 網址，將有的項目都勾選上

<video src="Files/55th寒假資網驗收模擬試題%20-%20中繼CA/更改AD%20CS%20CRL及AIA資訊.mp4" controls muted=true width=100%></video>

### 7. 導出 `.crl` 及架設 CRL 站台

在 Easy-RSA 上執行 `gen-crl` 生成 `.pem` 檔案

```bash
./easyrsa gen-crl
```

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/23.png)

然後將 .pem 轉換成 `.crl` 檔

```bash
openssl crl -inform PEM -in crl.pem -outform DER -out $.crl
```

- $.crl -> 自己取名稱，建議先更改成跟剛剛 CRL 設定一樣的檔名

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/24.png)

#### CRL 站台 - Nginx

將 .crl 導入到指定資料夾，使用 Nginx 並將網頁目錄指定到此資料夾，並設定對應的網址及 DNS。

<video src="Files/55th寒假資網驗收模擬試題%20-%20中繼CA/CRL%20-%20Nginx.mp4" controls muted=true width=100%></video>

#### CRL 站台 - IIS

將 .crl 檔案匯入到指定資料夾底下，並開起 IIS 將網頁目錄指定到此資料夾，並設定對應的網址及 DNS。

<video src="Files/55th寒假資網驗收模擬試題%20-%20中繼CA/CRL%20-%20IIS.mp4" controls muted=true width=100%></video>

### 8. 查看結果

回到 AD CS，並且啟動服務器

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/25.png)

`Win + R` 執行 `certlm.msc`

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/26.png)

展開 Personal，對 Certificates 右鍵 -> All Tasks -> Request New Certificate...

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/27.png)

Next -> Next -> 選取需要的憑證 -> Enroll

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/28.png)

Succeeded 代表成功！

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/29.png)

點擊 View Certificate 查看憑證資訊是否正確

![image](Files/55th寒假資網驗收模擬試題%20-%20中繼CA/30.png)

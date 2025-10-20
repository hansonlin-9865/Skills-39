# Markdown小本本

此專門記錄我在選寫 Markdown 時常用的指令，僅供參考..

1. 文檔摺疊

    ```html
    <details>
    <summary>[$標題名稱]</summary>
    [$內容]
    </details>
    ```

    > ex.
    >
    > ```html
    > <details>
    > <summary>test</summary>
    > 你好
    > </details>
    > ```
    >
    > <details>
    > <summary>test</summary>
    > 你好
    > </details>

2. 文字顏色

    ```html
    <font color="[$顏色]">[$內容]</font>
    ```

    > ex.
    >
    > ```html
    > <font color="red">Test Text.</font>
    > ```
    >
    > <font color="red">Test Text.</font>

3. 引用區塊

    ```markdown
    > [$內容]
    ```

    > ex.
    > 這裡已經範例了...不用我寫了吧

4. 插入影片

    ```html
    <video src="[＄影片位置]" controls muted=true width=100%></video>
    ```

    > ex.
    >
    >```html
    ><video src="/Linux/實作/Files/Networking/網卡-選取複製貼上.mov" controls muted=true width=100%></video>
    >```
    >
    ><video src="/linux/實作/Files/Networking/網卡-選取複製貼上.mov" controls muted=true width=100%></video>

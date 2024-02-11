*2023.03.02*

[[technical]]

介紹AWS lambda與使用方法
## AWS Lambda
![[Pasted image 20230608160941.png|400]]
[Documentation](https://aws.amazon.com/lambda/?nc1=h_ls)
AWS Lambda is a serverless, event-driven compute service that lets you run code for virtually any type of application or backend service without provisioning or managing servers.

AWS Lambda是一個透過事件觸發的程式碼執行服務，以我們的使用場景可能的案例如下
1. 使用者點擊連結要取得S3上的資料，可以透過API Gateway觸發lambda到S3取得資料後回傳給使用者
2. 使用者上傳檔案到S3後觸發lambda執行某段code (處理檔案or寫資料進DB)
![[Pasted image 20230608162136.png]]
詳細說明可以看官方文件

## 架設 AWS Lambda

### Create Function
到 AWS lambda 點擊 Create function 後可以選擇從０開始或使用AWS提供的樣板，其中的語言支援 .NET, Node.js, Python, Java, Go, Ruby等，除了名稱,語言,架構,權限(此lambda可以使用的服務),以外其他設定都可以之後再改。
![[Pasted image 20230608162420.png|500]]

### Add Layers (option)
一般要架設在lambda的功能會在本機開發+測試後才會放上去，但因為很常會用到第三方的套件，這一步就是利用Docker先建好這個lambda需要的環境後打包成zip，讓這個lambda function可以使用第三方的套件。


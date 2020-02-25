# AWS VPC
### VPC（Virtual Private Cloud，虛擬私有雲端）
VPC 是 Amazon EC2 的 Networking layer。可以說是 AWS 服務的基礎架構，透過 VPC 規劃出使用者所需要的網路架構，並在上面建置使用者所需的服務，如後端及資料庫就可放在 Private Subnet 之中，只有在同一個 VPC 或 Subnet 的裝置能夠存取，而前端或是提供服務的 API 則可放在 Public Subnet 之下，可供其他外部人員存取。
本節目標示建立一個混合公開子網路與私有子網路結構的 VPC，並介紹組成 VPC 的要素：
* 建立 VPC
* 建立 Subnet
* 建立 Internet Gateway
* 建立 Route Table
* 建立 NAT Gateway


![](https://i.imgur.com/zt0v1F9.png)

### 建立 VPC
可將 VPC 視為 AWS 上的一組網路架構，在同一帳戶下可有複數個 VPC，而 VPC 之間也相互獨立，VPC 之間的溝通可透過 Internet Gateway（如同一般的網路路由方式）或是透過 Peering Connections 功能達成。
進入 [AWS Console](https://console.aws.amazon.com/console/home) 並來到 VPC Dashboard 畫面
![](https://i.imgur.com/LNLRdec.png)
點選 **Your VPCs->Create VPC**
![](https://i.imgur.com/cn5BYrl.png)
命名 VPC 並指定一組 IPv4 CIDR (Classless Inter-Domain Routing)
![](https://i.imgur.com/NPsrdw4.png)

### 建立 Subnet
點選 **Subnets->Create Subnet**
![](https://i.imgur.com/jZ7J7sc.png)
分別建立三組子網路：
* **Public Subnet (10.10.20.0/24)**
* **Private Subnet1 (10.10.10.0/24)**
* **Private Subnet2 (10.10.9.0/24)**
並將 **Public Subnet** 自動分配 Public IPv4 設定啟用，啟用後該 Subnet 下的設備將會自動分配一組可用的 IPv4 位址。
![](https://i.imgur.com/lCnXwz4.png)
![](https://i.imgur.com/4vRLEhN.png)
![](https://i.imgur.com/qkmDzYI.png)
### 建立 Internet Gateway
Internet Gateway 所扮演的角色是負責 VPC 與外部網路間的連通，提供 VPC 中各個執行個體連接網際網路時的目標路由表，以及提供 VPC 內執行個體 NAT 的服務。（[AWS Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)）
可以做個小測試發現，在現階段如果在 Public Subnet 建立一個 EC2 並分配 Public IPv4 位址，該 EC2 執行個體仍無法由外部連線至其中，原因是因為該 VPC 中並沒有與網際網路端的閘道以及相對應的路由表。要具備網際網路的連線功能需建立起相對應的 Internet Gateway 以及路由表。
建立一個 Internet Gateway 並命名為 IGW-VPC-tutorial 並將其 Attach 至剛才建立的 VPC 上。
![](https://i.imgur.com/3aCeVYd.png)
![](https://i.imgur.com/iGqJG0P.png)
建立後並測試 EC2 的連線能力，仍發現無法由外部網路連線至其中，原因為少了相對應的路由表，因此下一步為建立路由表。
### 建立 Route Table
在建立好 VPC 的同時，AWS會自動建立一個預設的 Route Table，內容為該 VPC 所分配的 CIDR（本範例中為 10.10.0.0/16），在本階段的目標是完善架構圖中由綠色標示的連線能力及路由表。

點選 **Route Tables** 並於 **Route** 子分頁編輯該路由表，加入預設路由（0.0.0.0/0）指定為 Internet Gateway
可將該路由表命名為 Route-public 以便辨識。
![](https://i.imgur.com/6YnE93y.png)
![](https://i.imgur.com/PRGp58q.png)
近一步在 **Subnet Associations->Edit subnet associations** 設定該路由表只生效於 Public Subnet 之中。 ![](https://i.imgur.com/W4IN2Pe.png)

### 建立 NAT Gateway
要賦予 Private Subnet 能夠與網際網路連線而又不分配 Public IPv4 的方法就是接下來要介紹的 NAT Gateway，NAT Gateway 是讓 Private Subnet 透過路由將流量傳到放置於 Public Subnet 的 NAT Gateway 並透過此方式由 NAT Gateway 轉介至 Internet Gateway。透過此方式能夠讓 Private Subnet 擁有間接連線至網際網路的能力。
**注意：NAT Gateway屬於 VPC 中的收費服務（無關免費階段帳號, see [VPC pricing](https://aws.amazon.com/tw/vpc/pricing/)）。**
點選 **NAT Gateway->Create NAT Gateway**
選擇將 NAT Gateway 配置在 Public Subnet 之中，並分配一組 Elastic IP 給 NAT Gateway。
**註: Elastic IP 概念像是申請一組固定 IP 給特定的服務使用**
![](https://i.imgur.com/jaCCS6c.png)
接下來加入一組新的路由表給 Private Subnet 使用。
**Route tables->Create route table**
命名為 Route-private 並加入到 VPC-tutorial 之下
![](https://i.imgur.com/CPTfDZE.png)
編輯路由表，將預設路由設為 NAT Gateway
![](https://i.imgur.com/KzbhqeG.png)
![](https://i.imgur.com/zzbcAJc.png)
完成路由表編輯後，將此路由規則透過 **Subnet Assiciations** 套用在 Private Subnet
![](https://i.imgur.com/3gYPwku.png)
### 小結
上述步驟讓我們完成了如下圖的網路架構：
![](https://i.imgur.com/zt0v1F9.png)
Public Subnet 與 Internet Gateway 是負責該 VPC 與網際網路連通的介面，可將前端服務或是可讓外部人員存取的服務放在 Public Subnet，該子網域的會分配 Public IPv4 位址且預設路由為 Internet Gateway。
Private Subnet 無法直接被外部網路存取，只能透過 Public Subnet 存取到該子網路，可將後端服務或是資料庫等放置在 Private Subnet，而 Private Subnet 擁有間接存取網際網路的能力，流量路由從 NAT Gateway 送到網際網路端。

## Reference
[AWS Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
[AWS Documentation](https://docs.aws.amazon.com/)
## License
MIT license


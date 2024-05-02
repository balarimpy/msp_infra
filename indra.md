%%{init: {'theme':'base', 'themeVariables': { 'primaryColor': '#61676C', 'edgeLabelBackground':'#e8e8e8', 'tertiaryColor': '#61676C' }}}%%
graph TD
    subgraph "Cloud Environment Code For Testing bm code"
        apiGateway["API Gateway<br>Manages API requests<br>Provides security and routing"]
        loadBalancer["Load Balancer<br>Distributes traffic<br>Provides high availability"]
        subgraph "Cloud Infrastructure"
            merchantWebInstance1["Merchant Web Instance 1<br>JavaScript, Angular<br>Runs the merchant web application"]
            merchantWebInstance2["Merchant Web Instance 2<br>JavaScript, Angular<br>Runs the merchant web application"]
            merchantDbPrimary["Primary Merchant DB<br>MongoDB<br>Primary database for merchant data"]
            merchantDbSecondary["Secondary Merchant DB<br>MongoDB<br>Secondary database for merchant data"]
            onRampingWebInstance1["On-Ramping Web Instance 1<br>JavaScript, Angular<br>Runs the on-ramping web application"]
            onRampingWebInstance2["On-Ramping Web Instance 2<br>JavaScript, Angular<br>Runs the on-ramping web application"]
            onRampingDbPrimary["Primary On-Ramping DB<br>MongoDB<br>Primary database for on-ramping data"]
            onRampingDbSecondary["Secondary On-Ramping DB<br>MongoDB<br>Secondary database for on-ramping data"]
            virtualWalletCluster["Virtual Wallet Service<br>Java, Spring<br>Manages virtual wallet operations"]
            virtualWalletDb["Virtual Wallet DB<br>Graph DB<br>Stores virtual wallet data"]
            transactionCluster["Transaction Service<br>Java, Spring<br>Handles transaction processing"]
            transactionDb["Transaction DB<br>PostgreSQL<br>Stores transaction data"]
        end
    end
    subgraph "On-Premises Environment"
        firewall["Firewall<br>Secures on-premises network<br>Allows controlled access"]
    end
    merchantPortal["Merchant Portal<br>External web portal for merchants"]
    onRampingPortal["On-Ramping Portal<br>External web portal for on-ramping"]
    api["API<br>External API for integration"]
    sca["SCA<br>External Strong Customer Authentication service"]

    merchantPortal --> firewall
    onRampingPortal --> firewall
    api --> firewall
    sca --> firewall
    firewall --> apiGateway
    apiGateway --> loadBalancer
    loadBalancer --> merchantWebInstance1
    loadBalancer --> merchantWebInstance2
    merchantWebInstance1 --> merchantDbPrimary
    merchantWebInstance2 --> merchantDbSecondary
    merchantDbPrimary -.-> merchantDbSecondary
    loadBalancer --> onRampingWebInstance1
    loadBalancer --> onRampingWebInstance2
    onRampingWebInstance1 --> onRampingDbPrimary
    onRampingWebInstance2 --> onRampingDbSecondary
    onRampingDbPrimary -.-> onRampingDbSecondary
    merchantWebInstance1 --> virtualWalletCluster
    merchantWebInstance2 --> virtualWalletCluster
    onRampingWebInstance1 --> virtualWalletCluster
    onRampingWebInstance2 --> virtualWalletCluster
    virtualWalletCluster --> virtualWalletDb
    merchantWebInstance1 --> transactionCluster
    merchantWebInstance2 --> transactionCluster
    onRampingWebInstance1 --> transactionCluster
    onRampingWebInstance2 --> transactionCluster
    transactionCluster --> transactionDb

    note right of "Cloud Environment"
        **Cloud Environment**
        - API Gateway manages API requests and provides security and routing
        - Load Balancer distributes traffic across multiple web instances for high availability
        - Merchant Web Application instances with primary and secondary databases
        - On-Ramping Web Application instances with primary and secondary databases
        - Virtual Wallet Service and Graph DB for managing virtual wallet operations
        - Transaction Service and PostgreSQL DB for handling transaction processing
    end note

    note right of "On-Premises Environment"
        **On-Premises Environment**
        - Firewall secures the on-premises network and allows controlled access to cloud services
    end note

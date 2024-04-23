flowchart TD

subgraph "External Connectivity"
    MSP["Merchant Portal"] --> Firewall
    ORP["On Ramping Portal"] --> Firewall
    API["API"] --> Firewall
    SCA["SCA"] --> Firewall
end

subgraph "Firewall Configuration"
    Firewall --> APIGateway["API Gateway"]
end

subgraph "API Gateway"
    APIGateway --> LB["Load Balancer"]
end

subgraph "Load Balancers"
    LB --> MerchantWeb["Merchant Service Web"]
    LB --> OnrampingWeb["On-ramping Service Web"]
end

subgraph "Service Clustering"
    MerchantWeb --> MerchantDB["Merchant DB (mongoDB)"]
    MerchantCluster["Merchant Service"] --> MerchantDB
    OnrampingWeb --> OnrampingDB["Onramping DB (mongoDB)"]
    OnrampingCluster["On-ramping Service"] --> OnrampingDB
    VirtualWalletCluster["Virtual Wallet Service"] --> VirtualWalletDB["Virtual Wallet DB (graphDB)"]
    TransactionCluster["Transaction Service"] --> TransactionDB["Transaction DB (Postgresql)"]
    MerchantCluster --> VirtualWalletCluster
end


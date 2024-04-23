@startuml

skinparam componentStyle uml2

package "External Connectivity" {
    [Merchant Portal] as MSP
    [On Ramping Portal] as ORP
    [API] as API
    [SCA] as SCA
}

package "Firewall Configuration" {
    [Firewall] as Firewall
}

package "API Gateway" {
    [API Gateway] as APIGateway
}

package "Load Balancers" {
    [Load Balancer] as LB 
}



package "Service Clustering" {
    [Merchant Service Web] as MerchantWeb
    [Merchant Service ] as MerchantCluster
    [Merchant DB (mongoDB)] as MerchantDB
    [On-ramping Service Web] as OnrampingWeb
    [On-ramping Service ] as OnrampingCluster
    [Onramping DB (mongoDB)] as OnrampingDB
    [Virtual Wallet Service ] as VirtualWalletCluster
    [Virtual Wallet DB (graphDB)] as VirtualWalletDB 
    [Transaction Service ] as TransactionCluster
    [Transaction DB (Postgresql)] as TransactionDB
}

MSP -down-> Firewall
SCA -down-> Firewall
ORP -down-> Firewall
API -down-> Firewall

Firewall -down-> APIGateway
APIGateway -down-> LB 

LB -down-> MerchantWeb
LB -down-> OnrampingWeb

MerchantWeb -down-> MerchantDB
MerchantCluster -down-> MerchantDB
OnrampingWeb -down-> OnrampingDB
OnrampingCluster -down-> OnrampingDB
VirtualWalletCluster -down-> VirtualWalletDB
TransactionCluster -down-> TransactionDB
MerchantCluster -down-> VirtualWalletCluster

@enduml

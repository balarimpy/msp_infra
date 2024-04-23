@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_TOP_DOWN()

System_Ext(merchantPortal, "Merchant Portal", "External web portal for merchants")
System_Ext(onRampingPortal, "On-Ramping Portal", "External web portal for on-ramping")
System_Ext(api, "API", "External API for integration")
System_Ext(sca, "SCA", "External Strong Customer Authentication service")

System_Boundary(c1, "Cloud Environment") {
    Container(apiGateway, "API Gateway", "Manages API requests", "Provides security and routing")
    
    Container(loadBalancer, "Load Balancer", "Distributes traffic", "Provides high availability")
    
    Container(cloudInfrastructure, "Cloud Infrastructure", "Hosts application components") {
        Container(merchantWebInstance1, "Merchant Web Instance 1", "JavaScript, Angular", "Runs the merchant web application")
        Container(merchantWebInstance2, "Merchant Web Instance 2", "JavaScript, Angular", "Runs the merchant web application")
        ContainerDb(merchantDbPrimary, "Primary Merchant DB", "MongoDB", "Primary database for merchant data")
        ContainerDb(merchantDbSecondary, "Secondary Merchant DB", "MongoDB", "Secondary database for merchant data")
        
        Container(onRampingWebInstance1, "On-Ramping Web Instance 1", "JavaScript, Angular", "Runs the on-ramping web application")
        Container(onRampingWebInstance2, "On-Ramping Web Instance 2", "JavaScript, Angular", "Runs the on-ramping web application")
        ContainerDb(onRampingDbPrimary, "Primary On-Ramping DB", "MongoDB", "Primary database for on-ramping data")
        ContainerDb(onRampingDbSecondary, "Secondary On-Ramping DB", "MongoDB", "Secondary database for on-ramping data")
        
        Container(virtualWalletCluster, "Virtual Wallet Service", "Java, Spring", "Manages virtual wallet operations")
        ContainerDb(virtualWalletDb, "Virtual Wallet DB", "Graph DB", "Stores virtual wallet data")
        
        Container(transactionCluster, "Transaction Service", "Java, Spring", "Handles transaction processing")
        ContainerDb(transactionDb, "Transaction DB", "PostgreSQL", "Stores transaction data")
    }
}

System_Boundary(c2, "On-Premises Environment") {
    Container(firewall, "Firewall", "Secures on-premises network", "Allows controlled access")
}

Rel(merchantPortal, firewall, "Accesses", "HTTPS")
Rel(onRampingPortal, firewall, "Accesses", "HTTPS")
Rel(api, firewall, "Calls", "HTTPS")
Rel(sca, firewall, "Integrates with", "HTTPS")

Rel(firewall, apiGateway, "Forwards requests", "Secured Connection")
Rel(apiGateway, loadBalancer, "Routes requests", "HTTP")

Rel(loadBalancer, merchantWebInstance1, "Forwards requests", "HTTP")
Rel(loadBalancer, merchantWebInstance2, "Forwards requests", "HTTP")
Rel(merchantWebInstance1, merchantDbPrimary, "Reads/Writes to", "MongoDB")
Rel(merchantWebInstance2, merchantDbSecondary, "Reads/Writes to", "MongoDB")
Rel(merchantDbPrimary, merchantDbSecondary, "Replicates data", "MongoDB Replication")

Rel(loadBalancer, onRampingWebInstance1, "Forwards requests", "HTTP")
Rel(loadBalancer, onRampingWebInstance2, "Forwards requests", "HTTP")
Rel(onRampingWebInstance1, onRampingDbPrimary, "Reads/Writes to", "MongoDB")
Rel(onRampingWebInstance2, onRampingDbSecondary, "Reads/Writes to", "MongoDB")
Rel(onRampingDbPrimary, onRampingDbSecondary, "Replicates data", "MongoDB Replication")

Rel(merchantWebInstance1, virtualWalletCluster, "Calls", "JSON/HTTPS")
Rel(merchantWebInstance2, virtualWalletCluster, "Calls", "JSON/HTTPS")
Rel(onRampingWebInstance1, virtualWalletCluster, "Calls", "JSON/HTTPS")
Rel(onRampingWebInstance2, virtualWalletCluster, "Calls", "JSON/HTTPS")
Rel(virtualWalletCluster, virtualWalletDb, "Reads/Writes to", "Graph DB")

Rel(merchantWebInstance1, transactionCluster, "Calls", "JSON/HTTPS")
Rel(merchantWebInstance2, transactionCluster, "Calls", "JSON/HTTPS")
Rel(onRampingWebInstance1, transactionCluster, "Calls", "JSON/HTTPS")
Rel(onRampingWebInstance2, transactionCluster, "Calls", "JSON/HTTPS")
Rel(transactionCluster, transactionDb, "Reads/Writes to", "PostgreSQL")

SHOW_LEGEND()

note right of c1
    **Cloud Environment**
    - API Gateway manages API requests and provides security and routing
    - Load Balancer distributes traffic across multiple web instances for high availability
    - Merchant Web Application instances with primary and secondary databases
    - On-Ramping Web Application instances with primary and secondary databases
    - Virtual Wallet Service and Graph DB for managing virtual wallet operations
    - Transaction Service and PostgreSQL DB for handling transaction processing
end note

note right of c2
    **On-Premises Environment**
    - Firewall secures the on-premises network and allows controlled access to cloud services
end note

@enduml

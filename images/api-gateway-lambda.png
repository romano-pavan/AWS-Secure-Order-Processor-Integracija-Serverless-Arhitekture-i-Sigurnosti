# AWS-Secure-Order-Processor-Integracija-Serverless-Arhitekture-i-Sigurnosti
Ovaj projekt demonstrira implementaciju sigurne, visoko dostupne i skalabilne serverless arhitekture na AWS-u. Sustav simulira backend proces checkouta u web shopu, odvajajući javni API sloj od privatnog sloja za obradu podataka i pohranu.

```mermaid
graph TD
Korisnik((Korisnik / Frontend)) -->|1. HTTP POST| API[API Gateway]

subgraph VPC [AWS VPC - 10.0.0.0/16]
    API -.->|2. Proxy Integracija| Lambda
    
    subgraph PublicSubnet [Public Subnet - 10.0.1.0/24]
        NAT[NAT Gateway]
    end
    
    subgraph PrivateSubnet [Private Subnet - 10.0.2.0/24]
        Lambda[Lambda Funkcija]
    end
    
    Lambda -->|3. Izlaz na Internet| NAT
    Lambda -->|5. Interni AWS Promet| VPCE[VPC Gateway Endpoint]
    Lambda -.->|7. Dohvaćanje ključa| SM[AWS Secrets Manager]
end

NAT -->|4. API Poziv s tokenom| Stripe[Vanjski API npr. Stripe]
VPCE -->|6. Siguran upis| DB[(DynamoDB)]

classDef aws fill:#FF9900,stroke:#232F3E,stroke-width:2px,color:black;
class API,Lambda,NAT,VPCE,DB,SM aws;
```


Proces Implementacije (Dokumentiran Snimkama Zaslona)
Ovaj repozitorij služi kao dokaz praktičnog razumijevanja AWS infrastrukture i sigurnosnih principa po principu najmanje privilegije (Least Privilege).

1. Konfiguracija VPC-a i Mrežna Izolacija
Započelo se s kreiranjem prilagođenog VPC-a (10.0.0.0/16) s dva subneta:

Public Subnet (10.0.1.0/24): Sadrži NAT Gateway i ima rutu prema Internet Gatewayu (IGW).

Private Subnet (10.0.2.0/24): Ovdje je smještena Lambda funkcija. Izlazni promet usmjeren je prema NAT Gatewayu.




 # AWS-Sustav-Za-Sigurnu-Obradu-Narudzbi-Integracija-Serverless-Arhitekture-i-Sigurnosti
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


Proces Implementacije (dokumentiran snimkama zaslona)
Ovaj repozitorij služi kao dokaz praktičnog razumijevanja AWS infrastrukture i sigurnosnih principa po principu najmanje privilegije (Least Privilege).

1. Konfiguracija VPC-a i Mrežna Izolacija

   
Započelo se s kreiranjem prilagođenog VPC-a (10.0.0.0/16) s dva subneta:

Public Subnet (10.0.1.0/24): Sadrži NAT Gateway i ima rutu prema Internet Gatewayu (IGW).

Private Subnet (10.0.2.0/24): Ovdje je smještena Lambda funkcija. Izlazni promet usmjeren je prema NAT Gatewayu.

![vpc](images/vpc-kreranje.jpg)


![vpc](images/privat_and_public_subnet.jpg)


![vpc](images/public-subnet.png)


![vpc](images/private-subnet.png)


2. Sigurna pohrana podataka i optimizacija troškova
Za pohranu narudžbi korištena je DynamoDB tablica. Kako bi se osiguralo da promet ne izlazi na javni internet i kako bi se smanjili troškovi prijenosa podataka kroz NAT Gateway, implementiran je VPC Gateway Endpoint za DynamoDB.


![VPC Endpoint](images/endpoint-vpc.png)


![Ažurirana Private Route Tablica](images/dynmodb-route-private-rt.png)


3. Poslovna Logika u Privatnom okruženju (Lambda)
Lambda funkcija (Python) prima zahtjeve od API Gatewaya.

Dodijeljena joj je IAM uloga s AWSLambdaVPCAccessExecutionRole dozvolom za kreiranje ENI-ja u privatnom subnetu, te dozvolom za pisanje u bazu.

Smještena je isključivo u privatni subnet.

![vpc](images/lambda-role.png)

![vpc](images/lambda-private-subnet.png)


4. Javno Sučelje (API Gateway)
Kreiran je REST API (/narudzba POST metoda) koji služi kao ulazna točka za klijente.

Korištena je Lambda Proxy Integration za prosljeđivanje kompletnog HTTP zahtjeva privatnoj Lambda funkciji.

![vpc](images/api-gateway-lambda.png)

Rezultati Testiranja


Sustav je uspješno testiran slanjem POST zahtjeva na produkcijski endpoint API Gatewaya.

API Gateway uspješno prosljeđuje zahtjev u privatnu mrežu.

Lambda funkcija  vraća status 200 OK.

Zapis je uspješno kreiran u DynamoDB tablici.

![vpc](images/test-narudzbe.png)


![vpc](images/dynamodb-new-entry.png)












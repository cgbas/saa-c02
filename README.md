# Notas de estudo para SAA-C02

Repositório com minhas notas de estudo para a certificação de Solutions Architect - Associate, da Amazon.

## Índice
- [Notas de estudo para SAA-C02](#notas-de-estudo-para-saa-c02)
  - [Índice](#índice)
  - [Armazenamento e Migração de Dados](#armazenamento-e-migração-de-dados)
    - [S3](#s3)
    - [Snowball](#snowball)
    - [StorageGateway](#storagegateway)
    - [S3 Transfer Acceleration](#s3-transfer-acceleration)
    - [RRS](#rrs)
    - [Hospedando um website com S3](#hospedando-um-website-com-s3)
    - [Cloudfront](#cloudfront)
  - [EC2](#ec2)
    - [EBS](#ebs)
      - [Criando um volume](#criando-um-volume)
    - [Snapshots](#snapshots)
    - [Data Lifecycle Manager](#data-lifecycle-manager)
    - [User Data](#user-data)
    - [Instance Hibernation](#instance-hibernation)
    - [Security Groups](#security-groups)
    - [EFS](#efs)
  - [AWS Marketplace](#aws-marketplace)
  - [WAF](#waf)
  - [Cloudformation](#cloudformation)

---
## Armazenamento e Migração de Dados
### S3

Ou Simple storage Service, serviço de armazenamento com alta disponibilidade e durabilidade. Possui diversas classes de armazenamento. Possuem classes de armazenamento que no geral se baseiam na frequência de acesso:

* Frequente:
  * Standard
    * Mais comum, para objetos com acesso frequente
    * 99.999999999% (nove dígitos após a vírgula) de durabilidade
    * 99.999999999% de disponibilidade
* Infrequente:
  * Standard-IA
  * Glacier Deep Storage
    * Restore leva algumas horas
* Frequência Desconhecida ou que pode alterar
  * S3 Intelligent-Tiering
    * Movimenta automaticamente entre classes de acesso baseado na frequência que um arquivo é solicitado

**Importante**:
* Bucket names são globais e, portanto, únicos
* Em questões que mencionam apenas o S3, sem informar a classe, assumir que a questão fala da classe Standard.
* Tamanho mínimo de arquivo: 1 byte
* Tamanho máximo de arquivo: 5TB

### Snowball

Serviço: Migration & Tranfer (_Não_ é um serviço de storage)
Tranferência para applicance e então para o S3 sem utilizar internet

**Utilização**:
* Create Job
* Plan Jon (import into S3, export from S3, local compute & storage only)
* Configurações do Job (bucket, etc)
* Shipping
* Configurações de segurança
* Confirm Order


**Opções**: 
* Snowball Edge (100TB, 256b encryption)
* Snowmobile (100PT, contêiner de 15m)	

### StorageGateway
* FileGateway
* VolumeGateway
* TapeGateway - VTL
	
### S3 Transfer Acceleration

Exemplo: temos um site hospedado no Brasil e, para acelerar o acesso de um usuário no Japão podemos resolver isso com CDN, que irá realizar a cópia para um Edge location próximo do usuário no primeiro request e manter o cache por um tempo.

Com o transfer acceleration, temos a mesma analogia porém com o S3, pois se temos dados em um bucket em sa-east-1 e queremos fazer um upload de uma localização próxima de eu-west-2. O que acontece na verdade é que o upload acontece para um edge location em EU e através do s3 transfer acceleration os dados são enviados para o bucket em sa-east-1 através da rede interna da amazon.

**Como habilitar?**
* Dentro de advance settings no bucket, é possível habilitar/desabilitar o serviço, portanto é algo individual por bucket. Ao habilitar, a URL da bucket é alterada.

### RRS

Reduced Replication Storage, 0,01% de perda em comparação ao S3 standard, por ter menos replicação, torna-se um serviço mais barato.

Garante 99,99% em durabilidade e disponibilidade. **Não** é recomendado para dados críticos e não substituíveis. Útil por exemplo para thumbnails.

### Hospedando um website com S3

**Em properties**:
* Habilitar Static Website Hosting
* "use this bucket to host a static website"
* Apontar o index document e página de erro
* Ao habilitar, s3 fornece uma url

**IMPORTANTE**: Em um caso de uso onde ao hospedar um site não é do interesse que as imagens sejam acessadas diretamente, para que seu site seja acessado para isso, é possível remover o acesso público às imagens, servindo-as apenas a URLs assinadas e com data de expiração (signed URLs)

### Cloudfront

O [Cloudfront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) é um serviço para acelerar a distribuição de conteúdo estático (.html, .css, etc.) e dinâmico. O conteúdo é entregue por uma rede global de datacenters, os Edge Locations.

Para entregar com a melhor performance possível, os requests são roteados para o Edge Location com a menor latência. Se o Edge Location já possuir o conteúdo, ele é servido imediatamente, do contrário o mesmo é copiado da origem definida e fica em cache no destino, para servir imediatamente novos requests.

**Possíveis origens**:
* HTTP Servers (ex: web server)
  * Pode ser um EC2 ou um servidor próprio, conhecido por *custom origin*
* S3
* Canal MediaPackage

Arquivos servidos pelo Cloudfront são chamados *objetos*. Para utilizar o Cloudfront criamos uma *distribution* que irá receber um nome de domínio após sua criação ou pode ter um domínio customizado também. Nessa distribuição é definida a origem (*origin servers*) e se os requests irão gerar logs. A configuração dessa distribuição também é enviada para todos os Edge Locations.

Por padrão o cache em uma Edge Location tem um prazo de 24h para expiração através de cabeçalhos nos arquivos, podendo ser alterado para um mínimo de 0 segundos e não possui um valor máximo.

Caso você use um S3 como *origin server* pode optar por deixar os objetos públicos no bucket ou então mantê-los privados e somente [URLs assinadas](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html) podem acessá-los.

---
## EC2

O Elastic Compute Cloud, ou [EC2](https://aws.amazon.com/ec2/?nc1=h_ls), é um serviço que provê *instâncias* computacionais com vários [tipos](https://aws.amazon.com/ec2/instance-types/) de otimização:
* compute (c e hpc);
  * Alto poder de CPU
* memory (r, x, High Memory e z1d (alta frequência));
  * então alto poder de memória
* storage (i e d);
  * Alto IO
* accelerated computing (p, g, f1, vt1, trn (training) e inf (inference))
  * GPUs
* general purpouse (mac, t, m e A1)
  
[Valores](https://aws.amazon.com/ec2/pricing/) das instâncias. Vale ressaltar que além dos tipos de otimização, as instâncias podem ser classificadas em:
* Spot
  * Até 90% de economia sobre instâncias on-demand
  * Útil para aplicações sem tanta exigência por ínicio/término exato do processamento
  * **NÃO** recomendada para aplicações que não podem ser interrompidas
  * Pode viabilizar aplicações que precisem de custo muito baixo de operacionalização
  * Útil para momentos pontuais que necessitem de capacidade extra de processamento
* On-Demand
  * Cobradas por hora e até por segundo
    * Primeiros 60s são cobrados cheios, subsequentes são segundo a segundo.
  * Recomendadas para cargas de trabalho não preditíveis, porém que não podem ser interrompidas
* Dedicated Hosts
  * Recomendada para utilização com licenças de software Server-bound, como por exemplo SQL Server, Windows Server, etc.
  * Pode ser utilizada para cumprir requisitos de compliance
  * Contratadas on-Demand (por hora) ou então com um *reservation*, possibilitando até 70% de economia
  
E também temos a opção de ativar um **Saving Plan**, onde um desconto é fornecido mediante uma reserva adiantada da instância por 1 ou 3 anos. O Saving Plan pode ser ativado em instâncias EC2 e Fargate.

Para estimar os custos na AWS, podemos utilizar o [Pricing Calculator](https://calculator.aws/#/). Para visualizar todas suas instâncias EC2, utilize o [Global View](https://us-east-1.console.aws.amazon.com/ec2globalview/home?region=us-east-1#)

### EBS

O [Elastic Block Store](https://aws.amazon.com/pt/ebs/) provê discos HDD e SSD. São projetados para fornecer 99.999% de disponibilidar (5 noves). 

* Volumes SSD são mais recomendados para bancos de dados e volumes de inicialização, workloads transacionais. Dependência em IOPs.

* Volumes HDD são recomendados para workloads com tranferência intensa, como MapReduce ou então processamento de logs. Dependência em MB/s. Ainda em HDD há a opção Cold para dados que uma vez processados são pouco acessados e têm poucas verificações diárias.

#### Criando um volume

**EC2** => Elastic Block Storage => Volumes => Create Volume (...) 

**Informar**: 
+ Type
+ Size
+ AZ
+ Encryption (opcional)
+ Snapshot ID (opcional, apenas se for utilizar um como base))
+ Tags

**=> Create Volume**

### Snapshots

Serve como sistema de backup tanto para volumes como para instâncias

**EC2** => Elastic Block Storage => Snapshots

Resource types:
+ Instance
+ Volume

Para restaurar um snapshot, temos duas opções:
+ Criar um novo volume e utilizar o snapshot como base
+ Criar uma AMI e utilizar o snapshot como base

A diferença aqui é que no caso do EBS Volume, não necessariamente a máquina ira subir com as mesmas configurações, o volume pode ser simplesmente um disco a mais, enquanto no segundo caso a instância irá subir utilizando aquele snapshot como imagem base.


### Data Lifecycle Manager

O [Data Lifecycle Manager (DLM)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/snapshot-lifecycle.html) suporta:

+ Snapshots de Volumes EBS
  + Snapshots sucessivos são incrementais, portando ao remover um, somente os dados únicos daquele snapshots são apagados
+ AMIs baseadas em EBS

O DLM inclui algumas tags automaticamente, para diferenciar snapshots e AMIs criadas de alguma outra maneira, são elas:

* aws:dlm:lifecycle-policy-id
* aws:dlm:lifecycle-schedule-name
* aws:dlm:expirationTime
  * somente para *schedule* com uma  política de retenção baseada em duração
* dlm:managed

**Cotas**

|Descrição |	Cota|
| --- | --- |
|Políticas de Lifecycle por Região|100
|Tags por recurso|45

### User Data

Como [user-data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) é possível fornecer shell-scripts ou então diretivas de cloud-init que serão executadas durante o *launch* da instância. Durante a criação, em advanced details é possível fornecer esse user-data como texto, arquivo ou já em base64. Para alterar o que você configurou em user-data em uma instância já existente, é necessário que a mesma esteja em estado *stopped* e então em *Actions, Instance settings, Edit user data*.

### Instance Hibernation

Instâncias podem ser [hibernadas](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enabling-hibernation.html) desde que isso seja configurado durante a criação. Existem também alguns [pré-requisitos](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hibernating-prerequisites.html), como por exemplo ter o volume root com encriptação habilitada, não possuir mais de 150gb de RAM e somente suportar instâncias on-demand.

### Security Groups

[Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) são equivalentes a firewalls virtuais de uma instância, permitindo que criemos regras de entrada (inbound) e saída (outbound). SGs são stateful, portanto mesmo que você não tenha uma regra de entrada mas o request foi originado da instância, a mesma irá receber a resposta com sucesso. Caso você não forneça um SG durante a criação da instância, a mesma será adicionada ao SG *default*. É possível associar mais de um SG para uma instância e todos serão avaliados para decidir se algum tipo de tráfego deve ser permitido. Vale ressaltar que SGs chegam até a camada 4, portanto não há proteção de ataque na camada de aplicação (camada 7), para isso usamos o WAF.

### EFS

O [Elastic File System](https://aws.amazon.com/efs/?nc1=h_ls) permite que você tenha mais de uma instância anexada a um mesmo volume, algo que não é possível com o EBS. Ele é provisionado na ideia set-and-forget, sem tamanho mínimo e com pagamento apenas para o armazenamento utilizado. Recomendado para armazenamento com acesso infrequente, se adaptando ao *throughput* dinamicamente. Dentro de uma instância amazon linux, basta instalar o pacote `amazon-efs-utils` e realizar um mount (por IP ou DNS) após isso. É importante que o SG do EFS tenha uma regra para permitir acesso (inbound) via o SG das instâncias que irão anexá-lo.

---
## AWS Marketplace

O [AWS Marketplace](https://aws.amazon.com/mp/marketplace-service/overview/) é um catálogo digital que permite a contratação de soluções de terceiros em um modelo de utilização pay-per-use. São soluções curadas que no geral permitem implementação de diversos níveis de controle e governança. Há também cenários onde a solução de marketplace consiste em um ambiente pré-configurado e o usuário contratante deve fornecer sua própria licença (ex: Splunk Enterprise).

## WAF

O [WAF](https://aws.amazon.com/waf/?nc1=h_ls) pode ser utilizado em um Cloudfront, ALB (EC2), APIGateway ou APPSync (GraphQL APIS). Trata-se de um firewall de Camada 7 (aplicação), para proteção contra DDoS e conteúdo web malicioso.

## Cloudformation

O [Cloudformation](https://aws.amazon.com/cloudformation/?nc1=h_ls) é um serviço de IAC. Permite a criação de infraestrutura através de um template code em formato YAML ou JSON.



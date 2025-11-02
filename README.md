# 🔐 Guia Completo de Monitoramento, Segurança e Gerenciamento na AWS

> Documentação detalhada sobre CloudWatch, CloudTrail, CloudFormation, IAM, Policies e Roles

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazon-aws)](https://aws.amazon.com/)
[![Security](https://img.shields.io/badge/Security-First-red?style=for-the-badge&logo=security)](https://aws.amazon.com/security/)

## 📑 Índice

- [AWS CloudWatch](#-aws-cloudwatch)
- [AWS CloudTrail](#-aws-cloudtrail)
- [AWS CloudFormation](#-aws-cloudformation)
- [AWS IAM (Identity and Access Management)](#-aws-iam)
- [Policies e Roles](#-policies-e-roles)

---

## 📊 AWS CloudWatch

### O que é CloudWatch?

Amazon CloudWatch é um serviço de **monitoramento e observabilidade** que fornece dados e insights acionáveis para monitorar aplicações, responder a mudanças de performance e otimizar a utilização de recursos.

### Componentes Principais

#### 1. CloudWatch Metrics

**Métricas** são dados sobre a performance de seus recursos AWS.

**Métricas Padrão (Built-in):**
```
✅ EC2: CPUUtilization, NetworkIn/Out, DiskReadOps
✅ RDS: DatabaseConnections, ReadLatency, WriteLatency
✅ Lambda: Invocations, Duration, Errors, Throttles
✅ ELB: RequestCount, TargetResponseTime, HTTPCode_Target_5XX
✅ S3: BucketSizeBytes, NumberOfObjects
```

**Métricas Customizadas:**
- Você pode enviar suas próprias métricas
- Útil para métricas de aplicação (vendas, usuários ativos, etc)
- Usa a API PutMetricData

**Conceitos Importantes:**
- **Namespace**: Container lógico para métricas (ex: AWS/EC2)
- **Dimension**: Par nome/valor que identifica uma métrica (ex: InstanceId=i-123)
- **Timestamp**: Data/hora da métrica
- **Unit**: Unidade de medida (Bytes, Seconds, Percent, Count)
- **Period**: Intervalo de tempo da agregação (1, 5, 10, 30 segundos ou múltiplos de 60)

#### 2. CloudWatch Logs

Sistema de **gerenciamento e análise de logs** centralizado.

**Hierarquia:**
```
Log Group (ex: /aws/lambda/my-function)
  └── Log Stream (ex: 2024/11/01/[$LATEST]abc123)
      └── Log Events (linhas de log com timestamp)
```

**Recursos:**
- 📥 **Ingestão**: Coleta logs de EC2, Lambda, ECS, CloudTrail, etc.
- 🔍 **Insights**: Query logs usando linguagem SQL-like
- 📊 **Métricas de Filtro**: Cria métricas a partir de padrões em logs
- 💾 **Retenção**: Configurável de 1 dia a 10 anos
- 📤 **Exportação**: Para S3, Elasticsearch, Kinesis

**Exemplo de Log Insights Query:**
```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)
| sort @timestamp desc
| limit 20
```

#### 3. CloudWatch Alarms

**Alarmes** monitoram métricas e executam ações quando limites são atingidos.

**Estados do Alarme:**
- 🟢 **OK**: Métrica dentro do limite
- 🔴 **ALARM**: Métrica violou o limite
- 🟡 **INSUFFICIENT_DATA**: Dados insuficientes para avaliar

**Ações Possíveis:**
```
✅ Enviar notificação SNS
✅ Executar Auto Scaling action
✅ Executar EC2 action (stop, terminate, reboot, recover)
✅ Criar Systems Manager incident
```

**Exemplo: Alarme de CPU Alta**
```json
{
  "AlarmName": "HighCPUUtilization",
  "MetricName": "CPUUtilization",
  "Namespace": "AWS/EC2",
  "Statistic": "Average",
  "Period": 300,
  "EvaluationPeriods": 2,
  "Threshold": 80,
  "ComparisonOperator": "GreaterThanThreshold",
  "AlarmActions": ["arn:aws:sns:us-east-1:123456789:alerts"]
}
```

#### 4. CloudWatch Events / EventBridge

**EventBridge** (evolução do CloudWatch Events) é um barramento de eventos serverless.

**Tipos de Eventos:**
- 🔄 **Scheduled**: Cron ou rate expressions
- 🎯 **Event Pattern**: Reage a mudanças em serviços AWS
- 🔗 **Custom**: Eventos de suas aplicações

**Exemplo: Executar Lambda toda noite às 2h**
```json
{
  "schedule": "cron(0 2 * * ? *)",
  "target": {
    "arn": "arn:aws:lambda:us-east-1:123456789:function:BackupDB"
  }
}
```

#### 5. CloudWatch Dashboards

Interface visual para **monitoramento em tempo real**.

**Widgets Disponíveis:**
- 📈 Line graphs (gráficos de linha)
- 📊 Stacked area charts
- 🔢 Number displays
- 📉 Bar charts
- 🔤 Text widgets (markdown)

### CloudWatch Agent

**Agent** instalado em EC2/on-premises para coletar métricas e logs do sistema.

**Métricas Adicionais:**
```
✅ Memória RAM (não disponível nativamente)
✅ Disk swap utilization
✅ Disk space utilization
✅ Page file utilization
✅ Processos específicos
```

### Pricing Model

**Métricas:**
- Primeiras 10 métricas customizadas: grátis
- Após isso: $0.30 por métrica/mês

**Logs:**
- $0.50 por GB de ingestão
- $0.03 por GB de armazenamento/mês

**Alarmes:**
- Standard: $0.10 por alarme/mês
- High resolution: $0.30 por alarme/mês

### Casos de Uso

```
✅ Monitoramento de performance de aplicações
✅ Troubleshooting de problemas em produção
✅ Detecção proativa de anomalias
✅ Otimização de custos (identificar recursos ociosos)
✅ Conformidade e auditoria (logs centralizados)
✅ Automação com base em eventos
```

### Boas Práticas

- 📊 Crie dashboards específicos por equipe/serviço
- 🔔 Configure alarmes para métricas críticas
- 📈 Use métricas customizadas para KPIs de negócio
- 💾 Defina políticas de retenção adequadas para logs
- 🔍 Use Log Insights para análise avançada
- 🎯 Configure filtros de métricas para alertas específicos
- 📤 Exporte logs antigos para S3 (mais barato)

---

## 🔍 AWS CloudTrail

### O que é CloudTrail?

AWS CloudTrail é um serviço de **auditoria e governança** que registra todas as ações (API calls) realizadas na sua conta AWS, fornecendo histórico completo para segurança, conformidade e troubleshooting.

### Como Funciona?

```
┌─────────────┐
│   User/App  │
│  faz ação   │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│   API Call      │
│  (Ex: RunInstance)│
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│  CloudTrail     │
│  registra log   │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│  S3 Bucket      │
│  armazena logs  │
└─────────────────┘
```

### Tipos de Eventos

#### 1. Management Events (Control Plane)

Operações de **gerenciamento** de recursos AWS.

**Exemplos:**
```
✅ Criar/deletar recursos (CreateBucket, TerminateInstances)
✅ Configurar segurança (AttachUserPolicy, PutBucketPolicy)
✅ Configurar regras de roteamento
✅ Configurar logging
```

**Read vs Write:**
- **Read**: Eventos que leem informações (DescribeInstances, ListBuckets)
- **Write**: Eventos que modificam recursos (CreateUser, DeleteBucket)

#### 2. Data Events (Data Plane)

Operações de **acesso a dados** dentro de recursos.

**Exemplos:**
```
✅ S3 object-level operations (GetObject, PutObject, DeleteObject)
✅ Lambda function invocations
✅ DynamoDB operations (PutItem, DeleteItem, UpdateItem)
```

⚠️ **Importante**: Data events são de alto volume e têm custo adicional.

#### 3. Insights Events

CloudTrail **detecta automaticamente** atividades incomuns.

**Detecta:**
- 🚨 Picos de provisionamento de recursos
- 🚨 Excesso de IAM actions
- 🚨 Gaps em manutenção periódica
- 🚨 Padrões anormais de uso

### Trail Configuration

**Tipos de Trail:**

**Single Region Trail**
```
- Registra eventos de uma região específica
- Mais econômico
- Útil para recursos específicos de região
```

**Multi-Region Trail**
```
- Registra eventos de todas as regiões
- Inclui regiões globais (IAM, CloudFront, etc)
- Recomendado para auditoria completa
```

**Organization Trail**
```
- Aplica-se a todas as contas de uma AWS Organization
- Centraliza logs em uma conta principal
- Simplifica governança multi-conta
```

### Formato do Log Event

```json
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAI234567890EXAMPLE",
    "arn": "arn:aws:iam::123456789012:user/Alice",
    "accountId": "123456789012",
    "userName": "Alice"
  },
  "eventTime": "2024-11-01T12:34:56Z",
  "eventSource": "ec2.amazonaws.com",
  "eventName": "RunInstances",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "203.0.113.11",
  "userAgent": "aws-cli/2.0.0",
  "requestParameters": {
    "instanceType": "t3.micro",
    "imageId": "ami-0abcdef1234567890"
  },
  "responseElements": {
    "instancesSet": {
      "items": [{
        "instanceId": "i-0123456789abcdef0"
      }]
    }
  }
}
```

### Integrações

**CloudTrail integra-se com:**

```
✅ S3: Armazenamento de longo prazo
✅ CloudWatch Logs: Análise e alertas em tempo real
✅ EventBridge: Automação baseada em eventos
✅ Athena: Queries SQL em logs
✅ SNS: Notificações de entrega de logs
✅ KMS: Criptografia de logs
```

### Validação de Integridade

CloudTrail pode criar **digest files** para validar que logs não foram alterados.

**Recursos:**
- 🔐 Assinatura digital de cada log
- ✅ Validação com AWS CLI
- 📋 Conformidade com auditoria

```bash
# Validar integridade dos logs
aws cloudtrail validate-logs \
  --trail-arn arn:aws:cloudtrail:us-east-1:123456789:trail/MyTrail \
  --start-time 2024-11-01T00:00:00Z \
  --end-time 2024-11-01T23:59:59Z
```

### Casos de Uso

```
✅ Auditoria de segurança e conformidade
✅ Investigação de incidentes de segurança
✅ Rastreamento de mudanças em recursos
✅ Troubleshooting de problemas operacionais
✅ Análise forense após breach
✅ Conformidade regulatória (SOC, PCI, HIPAA)
✅ Monitoramento de atividades privilegiadas
```

### Exemplo de Cenário Real

**Investigação: "Quem deletou meu bucket S3?"**

1. Ir ao CloudTrail Console
2. Event history → Filter by Event name: `DeleteBucket`
3. Encontrar o evento
4. Ver detalhes:
   - Quem: User/Role que executou
   - Quando: Timestamp exato
   - De onde: IP de origem
   - O quê: Nome do bucket deletado

### Pricing

**Management Events:**
- Primeiro trail: GRÁTIS
- Trails adicionais: $2.00 por 100,000 eventos

**Data Events:**
- S3: $0.10 por 100,000 eventos
- Lambda: $0.20 por 100,000 eventos

**Insights Events:**
- $0.35 por 100,000 write events analisados

### Boas Práticas

- ✅ Habilite CloudTrail em todas as regiões
- 🔐 Use KMS para criptografar logs
- 📍 Centralize logs em uma conta de auditoria dedicada
- ⏰ Configure retenção adequada no S3
- 🔔 Crie alarmes CloudWatch para eventos críticos
- 🛡️ Habilite validação de integridade
- 🔒 Proteja o bucket S3 com MFA Delete
- 👁️ Habilite Insights para detecção de anomalias

---

## 🏗️ AWS CloudFormation

### O que é CloudFormation?

AWS CloudFormation é um serviço de **Infrastructure as Code (IaC)** que permite modelar, provisionar e gerenciar recursos AWS usando templates declarativos (JSON ou YAML).

### Conceitos Fundamentais

#### Template

Arquivo de texto (JSON/YAML) que define os recursos AWS a serem criados.

**Estrutura Básica:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Minha primeira stack'

Parameters:
  # Inputs do usuário

Mappings:
  # Valores estáticos (lookup tables)

Conditions:
  # Lógica condicional

Resources:
  # Recursos AWS (OBRIGATÓRIO)

Outputs:
  # Valores retornados
```

#### Stack

**Stack** é uma coleção de recursos AWS gerenciados como uma única unidade.

**Características:**
- 📦 Criação, atualização e deleção em conjunto
- 🔄 Rollback automático em caso de erro
- 🏷️ Versionamento e histórico de mudanças
- 🔗 Pode referenciar outras stacks

#### Change Set

**Preview** das mudanças antes de aplicá-las na stack.

**Benefícios:**
- 👀 Ver o que será modificado, deletado ou criado
- ⚠️ Identificar mudanças que causam replacement
- ✅ Aprovar mudanças antes de executar

### Seções do Template

#### 1. Parameters

Inputs para customização do template.

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
    Description: Tipo da instância EC2
  
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Nome do key pair para SSH
```

#### 2. Mappings

Tabelas de lookup para valores estáticos.

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-2:
      AMI: ami-0d1cd67c26f5fca19
    eu-west-1:
      AMI: ami-0bdb1d6c15a40392c
```

**Uso:** `!FindInMap [RegionMap, !Ref 'AWS::Region', AMI]`

#### 3. Conditions

Lógica condicional para criação de recursos.

```yaml
Conditions:
  CreateProdResources: !Equals [!Ref Environment, prod]
  IsUsEast1: !Equals [!Ref 'AWS::Region', us-east-1]

Resources:
  ProdDatabase:
    Type: AWS::RDS::DBInstance
    Condition: CreateProdResources
    Properties:
      # ...
```

#### 4. Resources (OBRIGATÓRIO)

Define os recursos AWS a serem criados.

```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SecurityGroupIds:
        - !Ref MySecurityGroup
      Tags:
        - Key: Name
          Value: MyWebServer
  
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
```

#### 5. Outputs

Valores exportados da stack.

```yaml
Outputs:
  InstanceId:
    Description: ID da instância EC2
    Value: !Ref MyEC2Instance
    Export:
      Name: !Sub ${AWS::StackName}-InstanceId
  
  WebsiteURL:
    Description: URL do site
    Value: !Sub 'http://${MyEC2Instance.PublicDnsName}'
```

### Intrinsic Functions

Funções built-in do CloudFormation.

| Função | Descrição | Exemplo |
|--------|-----------|---------|
| `!Ref` | Referencia parâmetro ou recurso | `!Ref MyInstance` |
| `!GetAtt` | Obtém atributo de recurso | `!GetAtt MyInstance.PublicIp` |
| `!FindInMap` | Lookup em Mappings | `!FindInMap [Map, Key, Value]` |
| `!Join` | Concatena strings | `!Join ['-', [web, server]]` |
| `!Sub` | Substituição de variáveis | `!Sub '${Env}-server'` |
| `!Select` | Seleciona item de lista | `!Select [0, !GetAZs '']` |
| `!Split` | Divide string | `!Split [',', 'a,b,c']` |
| `!If` | Condicional | `!If [Condition, True, False]` |
| `!Equals` | Comparação | `!Equals [!Ref Env, prod]` |
| `!ImportValue` | Importa output de outra stack | `!ImportValue NetworkStackVPC` |

### Stack Updates

**Comportamentos de Update:**

**Update with No Interruption**
- ✅ Recurso atualizado sem downtime
- Exemplo: Mudar tags de EC2

**Update with Some Interruption**
- ⚠️ Recurso sofre breve interrupção
- Exemplo: Resize de instância EC2

**Replacement**
- 🔴 Recurso é deletado e recriado
- Exemplo: Mudar tipo de instância RDS
- ⚠️ Pode causar perda de dados!

### Nested Stacks

**Stacks dentro de stacks** para reutilização e organização.

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/bucket/network.yaml
      Parameters:
        VpcCIDR: 10.0.0.0/16
```

**Benefícios:**
- 📦 Modularização
- ♻️ Reutilização de templates
- 🎯 Separação de responsabilidades

### Stack Policies

Protege recursos específicos contra updates não intencionais.

```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:Delete",
      "Resource": "LogicalResourceId/ProductionDatabase"
    },
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    }
  ]
}
```

### Helper Scripts

Scripts Python que facilitam configuração de instâncias EC2.

**cfn-init**: Busca e interpreta metadata do template  
**cfn-signal**: Sinaliza sucesso/falha da criação  
**cfn-get-metadata**: Retorna metadata de um recurso  
**cfn-hup**: Detecta mudanças em metadata e executa ações

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []
          files:
            /var/www/html/index.html:
              content: !Sub |
                <h1>Hello from ${AWS::StackName}</h1>
          services:
            sysvinit:
              httpd:
                enabled: true
                ensureRunning: true
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t3.micro
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource MyInstance --region ${AWS::Region}
          /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource MyInstance --region ${AWS::Region}
```

### Drift Detection

Detecta mudanças manuais feitas fora do CloudFormation.

**Como funciona:**
1. CloudFormation compara estado real vs esperado
2. Identifica recursos modificados
3. Mostra diferenças específicas

```bash
# Detectar drift
aws cloudformation detect-stack-drift --stack-name MyStack

# Ver resultados
aws cloudformation describe-stack-resource-drifts --stack-name MyStack
```

### Casos de Uso

```
✅ Provisionamento consistente de ambientes
✅ Disaster recovery (recriar infraestrutura rapidamente)
✅ Ambientes dev/staging/prod idênticos
✅ Versionamento de infraestrutura
✅ Auditoria e compliance
✅ Automação de CI/CD
✅ Documentação viva da arquitetura
```

### Boas Práticas

- 📦 Use nested stacks para modularização
- 🏷️ Padronize naming conventions
- 🔐 Use Secrets Manager/Parameter Store para dados sensíveis
- ✅ Valide templates antes de deploy
- 📝 Documente parâmetros e recursos
- 🔄 Use Change Sets antes de updates críticos
- 🎯 Defina DeletionPolicy para recursos importantes
- 🛡️ Implemente Stack Policies para proteção
- 👀 Monitore drift regularmente

---

## 🔐 AWS IAM (Identity and Access Management)

### O que é IAM?

AWS IAM é o serviço de **gerenciamento de identidades e acessos** que controla quem pode acessar seus recursos AWS e o que podem fazer com eles.

### Princípios Fundamentais

#### 1. Identity-based vs Resource-based Policies

**Identity-based**: Anexadas a usuários, grupos ou roles
```
"Quem pode fazer o quê?"
```

**Resource-based**: Anexadas a recursos (S3, SNS, SQS, Lambda)
```
"Quem pode acessar este recurso?"
```

#### 2. Princípio do Menor Privilégio

**Sempre conceda apenas as permissões mínimas necessárias.**

❌ Ruim: `AdministratorAccess` para tudo  
✅ Bom: Permissões específicas por função

### Componentes Principais

#### 1. Users (Usuários)

**Identidades permanentes** que representam pessoas ou aplicações.

**Características:**
- 👤 Um por pessoa/aplicação
- 🔑 Credenciais de longo prazo
- 🏷️ Pode pertencer a múltiplos grupos
- 🚫 Máximo 5,000 usuários por conta

**Tipos de Credenciais:**
- **Console password**: Para acesso web
- **Access Keys**: Para CLI/API/SDK
- **SSH keys**: Para CodeCommit
- **MFA device**: Autenticação multifator

```bash
# Criar usuário via CLI
aws iam create-user --user-name alice

# Criar access key
aws iam create-access-key --user-name alice
```

#### 2. Groups (Grupos)

**Coleção de usuários** para gerenciamento simplificado.

**Características:**
- 👥 Agrupa usuários com permissões similares
- 📋 Policies anexadas ao grupo aplicam-se a todos membros
- 🚫 Grupos NÃO podem conter outros grupos
- 🎯 Um usuário pode estar em até 10 grupos

**Exemplo de Estrutura:**
```
Developers Group
  ├── Alice
  ├── Bob
  └── Charlie

Admins Group
  ├── Dave
  └── Eve
```

#### 3. Roles (Funções)

**Identidades assumíveis** temporariamente por entidades confiáveis.

**Casos de Uso:**
```
✅ EC2 acessar S3
✅ Lambda acessar DynamoDB
✅ Cross-account access
✅ Federação (SSO, SAML)
✅ Serviços AWS assumindo roles
```

**Componentes de uma Role:**
- **Trust Policy**: Quem pode assumir a role
- **Permission Policy**: O que a role pode fazer

**Exemplo de Trust Policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "lambda.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

### Policies (Políticas)

#### Estrutura de uma Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ListBucket",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": "arn:aws:s3:::my-bucket",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Elementos:**
- **Version**: Versão da linguagem de policy (sempre `2012-10-17`)
- **Statement**: Array de permissões
  - **Sid**: Identificador do statement (opcional)
  - **Effect**: `Allow` ou `Deny`
  - **Action**: Ações AWS permitidas/negadas
  - **Resource**: Recursos AWS afetados
  - **Condition**: Condições opcionais

#### Tipos de Policies

**1. AWS Managed Policies**
```
✅ Criadas e mantidas pela AWS
✅ Não podem ser modificadas
✅ Atualizadas automaticamente
Exemplo: AdministratorAccess, ReadOnlyAccess
```

**2. Customer Managed Policies**
```
✅ Criadas por você
✅ Reutilizáveis
✅ Versionadas
✅ Até 6,144 caracteres
```

**3. Inline Policies**
```
✅ Embutidas diretamente em user/group/role
✅ Relacionamento 1:1
✅ Deletadas junto com a identidade
✅ Use apenas para exceções específicas
```

### Policy Evaluation Logic

**Ordem de avaliação:**

```
1. Explicit DENY → Nega imediatamente
2. Explicit ALLOW → Permite (se não houver deny)
3. Implicit DENY → Nega por padrão (se nada foi permitido)
```

**Regra de Ouro:** `Deny` sempre vence!

```
┌─────────────┐
│ Explicit    │
│   DENY?     │──Yes──▶ DENY
└──────┬──────┘
       │No
       ▼
┌─────────────┐
│ Explicit    │
│  ALLOW?     │──Yes──▶ ALLOW
└──────┬──────┘
       │No
       ▼
     DENY
(Implicit Deny)
```

### Permission Boundaries

**Limite máximo de permissões** que uma entidade pode ter.

```
Effective Permissions = Identity-based Policy ∩ Permission Boundary
```

**Exemplo:**
```
Identity Policy: S3*, EC2*, DynamoDB*
Permission Boundary: S3*, EC2*
─────────────────────────────────────
Effective: S3*, EC2* (DynamoDB bloqueado)
```

### IAM Credentials Report

Relatório de **auditoria de credenciais** de todos os usuários.

**Informações incluídas:**
- 🔑 Status de access keys
- 🔐 Última rotação de senha
- ✅ MFA habilitado?
- 📅 Última utilização de credenciais
- 🎯 Usuários inativos

```bash
# Gerar relatório
aws iam generate-credential-report

# Baixar relatório
aws iam get-credential-report --output text --query Content | base64 -d > report.csv
```

### Access Analyzer

**Analisa policies** para identificar recursos compartilhados externamente.

**Benefícios:**
- 🔍 Identifica acessos não intencionais
- 🚨 Alerta sobre recursos públicos
- ✅ Valida policies antes de deploy
- 📊 Findings organizados por severidade

### IAM Best Practices

#### Segurança

- 🔐 **Habilite MFA** para usuários privilegiados (especialmente root)
- 🔑 **Rotacione access keys** regularmente (90 dias)
- 👤 **Um usuário por pessoa** (nunca compartilhe credenciais)
- 🚫 **Não use root account** para operações diárias
- 🔒 **Use roles** em vez de embed credentials em código
- 📱 **Habilite MFA para deleção** de recursos críticos

#### Gerenciamento

- 🎯 **Princípio do menor privilégio** sempre
- 👥 **Use grupos** para gerenciar permissões
- 📋 **Customer managed policies** para políticas reutilizáveis
- 🏷️ **Tag resources** para organização
- 📊 **Audite regularmente** com Access Analyzer
- 🗑️ **Remova credenciais não utilizadas**

#### Monitoramento

- 📈 **Monitore CloudTrail** para atividades IAM
- 🔔 **Alarmes CloudWatch** para ações sensíveis
- 📑 **Revise credential reports** mensalmente
- 🎯 **Use IAM Access Advisor** para identificar permissões não usadas

---

## 📜 Policies e Roles

### Deep Dive em IAM Policies

#### Policy Variables

Variáveis dinâmicas em policies para personalização.

**Variáveis Disponíveis:**
```
${aws:username}        - Nome do usuário IAM
${aws:userid}          - ID único do usuário
${aws:PrincipalTag/key} - Tag do principal
${aws:SourceIp}        - IP de origem
${aws:CurrentTime}     - Data/hora atual
${aws:SecureTransport} - Se usa HTTPS
${aws:RequestedRegion} - Região solicitada
```

**Exemplo: Acesso por usuário a própria pasta S3**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:ListBucket"
    ],
    "Resource": "arn:aws:s3:::company-bucket",
    "Condition": {
      "StringLike": {
        "s3:prefix": ["home/${aws:username}/*"]
      }
    }
  },
  {
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::company-bucket/home/${aws:username}/*"
  }]
}
```

#### Condition Keys

**Operadores de Condição:**

| Operador | Descrição | Exemplo |
|----------|-----------|---------|
| `StringEquals` | String exata | `"StringEquals": {"aws:username": "alice"}` |
| `StringLike` | String com wildcards | `"StringLike": {"s3:prefix": ["*.jpg"]}` |
| `NumericGreaterThan` | Número maior que | `"NumericGreaterThan": {"s3:max-keys": "10"}` |
| `DateGreaterThan` | Data posterior | `"DateGreaterThan": {"aws:CurrentTime": "2024-01-01T00:00:00Z"}` |
| `IpAddress` | Range de IP | `"IpAddress": {"aws:SourceIp": "203.0.113.0/24"}` |
| `Bool` | Booleano | `"Bool": {"aws:SecureTransport": "true"}` |
| `ArnLike` | ARN com wildcards | `"ArnLike": {"aws:SourceArn": "arn:aws:s3:::bucket/*"}` |

**Exemplo: Permitir apenas acesso HTTPS**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "s3:*",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "Bool": {
        "aws:SecureTransport": "false"
      }
    }
  }]
}
```

**Exemplo: Restrição por horário comercial**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "ec2:*",
    "Resource": "*",
    "Condition": {
      "DateGreaterThan": {"aws:CurrentTime": "2024-01-01T09:00:00Z"},
      "DateLessThan": {"aws:CurrentTime": "2024-12-31T18:00:00Z"}
    }
  }]
}
```

### Service Control Policies (SCPs)

**SCPs** são policies aplicadas no nível de **AWS Organizations**.

#### Características

- 🏢 Aplicam-se a **contas inteiras** ou OUs
- 🛡️ Agem como **guardrails** (limites máximos)
- 🚫 Não concedem permissões, apenas restringem
- 👑 Não afetam root user da conta
- 📊 Hierárquicas (herdam da OU pai)

#### Estratégias de SCP

**1. Allow List (Whitelist)**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:*",
      "s3:*",
      "rds:*"
    ],
    "Resource": "*"
  }]
}
```

**2. Deny List (Blacklist)**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": [
      "ec2:TerminateInstances",
      "rds:DeleteDBInstance"
    ],
    "Resource": "*"
  }]
}
```

#### Casos de Uso para SCPs

```
✅ Prevenir desativação de CloudTrail
✅ Restringir regiões permitidas
✅ Bloquear serviços específicos
✅ Enforçar criptografia
✅ Prevenir deleção de recursos críticos
✅ Controlar custos limitando tipos de instância
```

**Exemplo: Restringir Regiões**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "StringNotEquals": {
        "aws:RequestedRegion": [
          "us-east-1",
          "us-west-2"
        ]
      }
    }
  }]
}
```

### IAM Roles - Casos Avançados

#### 1. Cross-Account Access

**Cenário:** Conta A quer acessar recursos da Conta B

**Conta B (Recursos):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::ACCOUNT-A-ID:root"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

**Conta A (Assume role):**
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT-B-ID:role/CrossAccountRole \
  --role-session-name MySession
```

#### 2. Service-Linked Roles

**Roles predefinidas** criadas por serviços AWS.

**Características:**
- 🔗 Vinculadas a serviços específicos
- 🚫 Não podem ser modificadas
- ✅ Criadas automaticamente quando necessário
- 🎯 Permissões mínimas para o serviço funcionar

**Exemplo:** `AWSServiceRoleForAutoScaling`

#### 3. Instance Profile

**Container** para IAM role usada por EC2.

```bash
# Criar role
aws iam create-role --role-name EC2-S3-Access --assume-role-policy-document file://trust-policy.json

# Anexar policy
aws iam attach-role-policy --role-name EC2-S3-Access --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Criar instance profile
aws iam create-instance-profile --instance-profile-name EC2-S3-Profile

# Adicionar role ao profile
aws iam add-role-to-instance-profile --instance-profile-name EC2-S3-Profile --role-name EC2-S3-Access

# Associar a instância
aws ec2 associate-iam-instance-profile --instance-id i-1234567890abcdef0 --iam-instance-profile Name=EC2-S3-Profile
```

### Resource-Based Policies

Algumas **resources** suportam policies anexadas diretamente.

#### S3 Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123456789012:user/alice"
    },
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

#### SNS Topic Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "lambda.amazonaws.com"
    },
    "Action": "SNS:Publish",
    "Resource": "arn:aws:sns:us-east-1:123456789012:my-topic"
  }]
}
```

#### Lambda Function Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "s3.amazonaws.com"
    },
    "Action": "lambda:InvokeFunction",
    "Resource": "arn:aws:lambda:us-east-1:123456789012:function:MyFunction",
    "Condition": {
      "StringEquals": {
        "AWS:SourceAccount": "123456789012"
      },
      "ArnLike": {
        "AWS:SourceArn": "arn:aws:s3:::my-bucket"
      }
    }
  }]
}
```

### Exemplos Práticos de Policies

#### 1. Developer Policy (Read-Write sem Delete)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances",
        "s3:List*",
        "s3:Get*",
        "s3:PutObject",
        "dynamodb:*",
        "lambda:*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "ec2:TerminateInstances",
        "s3:DeleteObject",
        "s3:DeleteBucket",
        "dynamodb:DeleteTable",
        "lambda:DeleteFunction"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 2. Read-Only Analyst Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "cloudwatch:Describe*",
      "cloudwatch:Get*",
      "cloudwatch:List*",
      "logs:Describe*",
      "logs:Get*",
      "logs:FilterLogEvents",
      "s3:GetObject",
      "s3:ListBucket",
      "athena:*"
    ],
    "Resource": "*"
  }]
}
```

#### 3. Emergency Admin Access (Time-Limited)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "DateGreaterThan": {"aws:CurrentTime": "2024-11-01T00:00:00Z"},
      "DateLessThan": {"aws:CurrentTime": "2024-11-01T23:59:59Z"}
    }
  }]
}
```

#### 4. Enforce MFA for Sensitive Operations

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "s3:List*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:TerminateInstances",
        "rds:DeleteDBInstance",
        "s3:DeleteBucket"
      ],
      "Resource": "*",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

### Policy Simulator

Ferramenta para **testar policies** sem risco.

**Acesso:** https://policysim.aws.amazon.com

**Funcionalidades:**
- ✅ Testar policies antes de aplicar
- 🔍 Debugar problemas de permissão
- 📊 Ver resultado de múltiplas policies combinadas
- 🎯 Simular diferentes contextos (IP, região, etc)

```bash
# CLI
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names s3:GetObject s3:PutObject \
  --resource-arns arn:aws:s3:::my-bucket/*
```

### Troubleshooting IAM Issues

#### Problema: "Access Denied"

**Checklist de Debug:**

1. ✅ **Identity-based policy** permite a ação?
2. ✅ **Resource-based policy** permite (se aplicável)?
3. ✅ **Permission boundary** não bloqueia?
4. ✅ **SCP** não bloqueia (se Organizations)?
5. ✅ **Session policy** não restringe (se assume role)?
6. ✅ Há **explicit deny** em alguma policy?
7. ✅ **Condition keys** são satisfeitas?

#### Ferramenta: IAM Access Advisor

Mostra **quando serviços foram acessados** pela última vez.

**Benefício:** Identificar permissões não utilizadas

```bash
aws iam generate-service-last-accessed-details \
  --arn arn:aws:iam::123456789012:user/alice
```

#### CloudTrail para Auditoria

**Query para encontrar negações:**

```sql
SELECT
  eventTime,
  userIdentity.userName,
  eventName,
  errorCode,
  errorMessage
FROM cloudtrail_logs
WHERE errorCode = 'AccessDenied'
ORDER BY eventTime DESC
LIMIT 100
```

---

## 🎓 Cenários Práticos e Casos de Uso

### Cenário 1: Startup Crescendo

**Desafio:** Organizar permissões para equipe crescente

**Solução:**
```
1. Criar grupos por função:
   - Developers
   - DevOps
   - Analysts
   - Admins

2. Anexar managed policies apropriadas
3. Criar custom policies para casos específicos
4. Implementar MFA para admins
5. Rotação de keys a cada 90 dias
```

### Cenário 2: Multi-Account Organization

**Desafio:** Gerenciar 50+ contas AWS

**Solução:**
```
1. Usar AWS Organizations
2. Aplicar SCPs para guardrails
3. Centralizar CloudTrail logs
4. Cross-account roles para auditoria
5. Control Tower para governance
```

### Cenário 3: Conformidade Regulatória

**Desafio:** Atender SOC2, PCI-DSS, HIPAA

**Solução:**
```
1. CloudTrail habilitado em todas contas
2. CloudWatch Logs para análise
3. IAM Access Analyzer
4. Config Rules para compliance
5. GuardDuty para threat detection
6. Security Hub para visão unificada
```

### Cenário 4: Incident Response

**Desafio:** Instância comprometida

**Procedimento:**
```
1. Isolar instância (Security Group isolado)
2. Snapshot EBS para forense
3. CloudTrail: buscar ações suspeitas
4. Revogar access keys comprometidas
5. Rotacionar credenciais expostas
6. Implementar GuardDuty findings
```

---

## 📚 Recursos Adicionais

### Documentação Oficial

- [CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)
- [CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)

### Ferramentas Úteis

- **AWS CLI**: Interface de linha de comando
- **CloudFormation Designer**: Editor visual de templates
- **IAM Policy Simulator**: Teste policies sem risco
- **AWS Config**: Avaliação contínua de conformidade
- **AWS Organizations**: Gerenciamento multi-conta
- **AWS Control Tower**: Governança automatizada

### Certificações Relevantes

- ☁️ **AWS Certified Solutions Architect** - Associate/Professional
- 🔧 **AWS Certified Developer** - Associate
- 🔐 **AWS Certified Security** - Specialty
- 🏗️ **AWS Certified SysOps Administrator**

### Workshops e Hands-On

- [AWS Well-Architected Labs](https://wellarchitectedlabs.com/)
- [AWS Workshops](https://workshops.aws/)
- [AWS Security Workshops](https://security.awsworkshops.io/)

### Blogs e Comunidade

- [AWS Security Blog](https://aws.amazon.com/blogs/security/)
- [AWS DevOps Blog](https://aws.amazon.com/blogs/devops/)
- [AWS re:Post](https://repost.aws/)

---

## ⚡ Comandos AWS CLI Essenciais

### CloudWatch

```bash
# Listar métricas
aws cloudwatch list-metrics --namespace AWS/EC2

# Obter estatísticas de métrica
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-11-01T00:00:00Z \
  --end-time 2024-11-01T23:59:59Z \
  --period 3600 \
  --statistics Average

# Criar alarme
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "CPU above 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold

# Buscar logs
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --start-time 1698796800000 \
  --filter-pattern "ERROR"
```

### CloudTrail

```bash
# Listar trails
aws cloudtrail list-trails

# Obter status de trail
aws cloudtrail get-trail-status --name my-trail

# Buscar eventos
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances \
  --max-results 10
```

### CloudFormation

```bash
# Criar stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t3.micro

# Atualizar stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template-v2.yaml

# Deletar stack
aws cloudformation delete-stack --stack-name my-stack

# Descrever stack
aws cloudformation describe-stacks --stack-name my-stack

# Listar recursos
aws cloudformation list-stack-resources --stack-name my-stack

# Validar template
aws cloudformation validate-template --template-body file://template.yaml

# Criar change set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template-v2.yaml
```

### IAM

```bash
# Listar usuários
aws iam list-users

# Criar usuário
aws iam create-user --user-name alice

# Anexar policy a usuário
aws iam attach-user-policy \
  --user-name alice \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Criar access key
aws iam create-access-key --user-name alice

# Listar roles
aws iam list-roles

# Criar role
aws iam create-role \
  --role-name MyRole \
  --assume-role-policy-document file://trust-policy.json

# Obter policy
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Gerar credential report
aws iam generate-credential-report
aws iam get-credential-report --output text --query Content | base64 -d
```

---

## 🔥 Dicas de Otimização e Economia

### CloudWatch

💰 **Economize com:**
- Reduzir retenção de logs antigos
- Usar métricas de resolução padrão (1 min) em vez de alta resolução
- Exportar logs antigos para S3
- Deletar métricas customizadas não utilizadas

### CloudTrail

💰 **Economize com:**
- Usar um trail multi-região em vez de múltiplos trails
- Filtrar data events para apenas recursos críticos
- Lifecycle policy no S3 para arquivar logs antigos para Glacier

### CloudFormation

⚡ **Otimize com:**
- Usar nested stacks para evitar limites de template
- DependsOn apenas quando necessário (automaticamente resolvido)
- DeletionPolicy: Retain para recursos críticos
- UpdateReplacePolicy para comportamento em updates

---

## 🎯 Resumo de Boas Práticas

### Monitoramento (CloudWatch)

✅ Configure alarmes para métricas críticas  
✅ Use dashboards para visibilidade  
✅ Implemente logging estruturado  
✅ Configure retenção apropriada  
✅ Use Log Insights para análise  

### Auditoria (CloudTrail)

✅ Habilite em todas as regiões  
✅ Ative validação de integridade  
✅ Centralize logs em conta dedicada  
✅ Integre com CloudWatch para alertas  
✅ Revise regularmente eventos críticos  

### Infraestrutura (CloudFormation)

✅ Versionamento de templates  
✅ Use change sets para updates  
✅ Implemente stack policies  
✅ Modularize com nested stacks  
✅ Valide templates antes de deploy  

### Segurança (IAM)

✅ Menor privilégio sempre  
✅ MFA para contas privilegiadas  
✅ Rotação regular de credenciais  
✅ Use roles em vez de access keys  
✅ Audite permissões regularmente  


---

<div align="center">

⭐ Se este guia foi útil, considere dar uma estrela!

**Mantenha-se seguro e monitore tudo! 🔐📊**

[⬆ Voltar ao topo](#-guia-completo-de-monitoramento-segurança-e-gerenciamento-na-aws)

</div>

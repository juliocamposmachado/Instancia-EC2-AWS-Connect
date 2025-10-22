
# 🚀 Guia de Criação e Configuração de Instâncias EC2 na AWS via CLI

Este guia fornece um passo a passo completo para **criação, conexão e configuração** de instâncias EC2, utilizando tanto o **Console AWS** quanto o **AWS CLI**.  
O objetivo é consolidar conhecimentos práticos sobre administração de servidores em nuvem.

---

## 🎯 Objetivos

1. Criar uma instância **Bastion Host** pelo Console AWS  
2. Conectar-se ao Bastion Host via **EC2 Instance Connect**  
3. Criar uma instância **Web Server** utilizando **AWS CLI**

---

## 💻 Passo a Passo Detalhado

### 🧩 1. Criar a Instância Bastion Host (via Console AWS)

1. Acesse o **Console AWS** e procure por **EC2**  
2. Clique em **Launch instance**  
3. Configure os parâmetros conforme abaixo:

| Parâmetro | Valor |
|------------|--------|
| Nome | Bastion Host |
| AMI | Amazon Linux 2 |
| Tipo de instância | t3.micro |
| Key pair | *Proceed without key pair* |
| VPC/Subnet | Lab VPC / Public Subnet |
| Security Group | Bastion security group |
| Descrição do SG | Permitir conexões SSH |
| IAM Role | Bastion-Role |

4. Clique em **Launch instance**

---

### 🔐 2. Conectar ao Bastion Host

1. No Console AWS, selecione a instância **Bastion Host**  
2. Clique em **Connect** → **EC2 Instance Connect**  
3. Em seguida, clique em **Connect** para abrir o terminal da instância

---

### 🌐 3. Criar Web Server via AWS CLI

Conectado ao **Bastion Host**, execute os comandos abaixo em sequência.

#### 3.1 Definir Região e Recuperar AMI

```bash
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
export AWS_DEFAULT_REGION=${AZ::-1}

AMI=$(aws ssm get-parameters \
  --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 \
  --query 'Parameters[0].[Value]' \
  --output text)

echo "✅ AMI usada: $AMI"
````

---

#### 3.2 Recuperar Subnet

```bash
SUBNET=$(aws ec2 describe-subnets \
  --filters 'Name=tag:Name,Values=Public Subnet' \
  --query 'Subnets[].SubnetId' \
  --output text)

echo "✅ Subnet usada: $SUBNET"
```

---

#### 3.3 Recuperar Security Group

```bash
SG=$(aws ec2 describe-security-groups \
  --filters Name=group-name,Values=WebSecurityGroup \
  --query 'SecurityGroups[].GroupId' \
  --output text)

echo "✅ Security Group usado: $SG"
```

---

#### 3.4 Baixar Script de Inicialização

```bash
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-1-23732/171-lab-JAWS-create-ec2/s3/UserData.txt
cat UserData.txt
```

---

#### 3.5 Criar Instância Web Server

```bash
INSTANCE=$(
aws ec2 run-instances \
  --image-id $AMI \
  --subnet-id $SUBNET \
  --security-group-ids $SG \
  --user-data file:///home/ec2-user/UserData.txt \
  --instance-type t3.micro \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Web Server}]' \
  --query 'Instances[*].InstanceId' \
  --output text
)

echo "🚀 Instância criada: $INSTANCE"
```

---

#### 3.6 Verificar Status da Instância

```bash
aws ec2 describe-instances --instance-ids $INSTANCE \
  --query 'Reservations[].Instances[].State.Name' \
  --output text
```

---

#### 3.7 Obter Endereço Público do Web Server

```bash
aws ec2 describe-instances --instance-ids $INSTANCE \
  --query 'Reservations[].Instances[].PublicDnsName' \
  --output text
```

> 📌 Copie o endereço retornado e cole no navegador para acessar o servidor web.

---

## 🧠 Observações Importantes

* Certifique-se de que a **VPC**, **Subnet** e **Security Groups** já estejam criados previamente.
* Para ambientes de produção, utilize **pares de chaves (Key Pairs)** e políticas de segurança restritivas.
* O script `UserData.txt` realiza a instalação automática dos pacotes necessários no Web Server.

---

## 🏁 Resultado Esperado

Ao final deste procedimento, você terá:

* Um **Bastion Host** funcional para acesso seguro via SSH
* Um **Web Server** ativo, acessível por DNS público
* Domínio prático sobre operações EC2 utilizando **AWS CLI**

---

📘 **Autor:** Julio Campos Machado
🧩 **Empresas:** Like Look Solutions | Rádio Tatuapé FM
📧 **Contato:** [juliocamposmachado@gmail.com](mailto:juliocamposmachado@gmail.com)

```

---

Deseja que eu adicione também uma seção de **pré-requisitos técnicos** (como: “ter AWS CLI configurado”, “possuir IAM Role com permissões adequadas”, etc.) e uma **versão em inglês** para uso profissional no GitHub internacional? Isso deixaria o documento mais completo e atrativo.
```

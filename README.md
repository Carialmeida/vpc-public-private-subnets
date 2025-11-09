# ☁️ Configurando uma VPC na AWS

## 📘 Visão Geral

A **Amazon Virtual Private Cloud (Amazon VPC)** permite criar uma seção logicamente isolada na Nuvem AWS, onde é possível executar recursos de forma totalmente controlada.  
Neste laboratório, foi criada uma **VPC completa com sub-redes públicas e privadas**, um **gateway de internet**, um **gateway NAT**, e **instâncias EC2** com diferentes níveis de acesso.

O objetivo é compreender como funciona a arquitetura de rede dentro da AWS e como os recursos se comunicam entre si de forma segura e segmentada.

---

## 🏗️ Arquitetura Final

A arquitetura implementada contém:

- 🟢 **VPC** com bloco CIDR `10.0.0.0/16`
- 🌐 **Sub-rede pública** (`10.0.0.0/24`)
- 🔒 **Sub-rede privada** (`10.0.2.0/23`)
- 🚪 **Internet Gateway (IGW)** para tráfego público
- 🔁 **NAT Gateway** para acesso controlado à internet pela sub-rede privada
- 💻 **Instância EC2 Bastion Server** (na sub-rede pública)
- 🧱 **Instância EC2 privada** (acessada via Bastion)
- 🗺️ **Tabelas de rotas** separadas para tráfego público e privado

---

## 🎯 Objetivos

Ao final deste laboratório, foi possível:

- Criar uma **VPC com sub-rede pública e privada**
- Configurar **tabelas de roteamento** para tráfego interno e externo
- Conectar a VPC à internet através de um **Internet Gateway**
- Criar e associar um **NAT Gateway** à sub-rede pública
- Iniciar e configurar um **servidor bastion**
- Conectar-se à instância privada via bastion

---

## ⏱️ Duração

Tempo médio de conclusão: **~45 minutos**

---

## 🧭 Passo a Passo

### 🧩 1. Criar uma VPC
1. Acesse o console da **VPC**
2. Selecione **“Criar VPC”**
3. Configure:
   - **Nome:** `Lab VPC`
   - **Bloco CIDR IPv4:** `10.0.0.0/16`
   - **Bloco CIDR IPv6:** Nenhum
4. Ative:
   - ✅ **Atribuição de nomes DNS**
   - ✅ **Hostnames DNS**

---

### 🌍 2. Criar Sub-redes

#### Sub-rede pública
- **Nome:** `Public Subnet`
- **Bloco CIDR:** `10.0.0.0/24`
- **Atribuir IP público automaticamente:** Ativado

#### Sub-rede privada
- **Nome:** `Private Subnet`
- **Bloco CIDR:** `10.0.2.0/23`

---

### 🌐 3. Criar o Internet Gateway

1. No menu **Gateways da Internet**, selecione **Criar gateway de internet**
   - Nome: `Lab IGW`
2. Selecione o IGW criado → **Ações → Anexar à VPC**
   - Escolha a `Lab VPC`

---

### 🚦 4. Configurar Tabelas de Rotas

#### Tabela de rota privada
- Renomeie para `Private Route Table`
- Rotas:
  - `10.0.0.0/16` → local

#### Criar tabela de rota pública
1. Clique em **Criar tabela de rotas**
   - Nome: `Public Route Table`
   - VPC: `Lab VPC`
2. Adicione rota:
   - Destino: `0.0.0.0/0`
   - Target: **Internet Gateway (Lab IGW)**
3. Associe a sub-rede pública a esta tabela

---

### 🔐 5. Iniciar o Servidor Bastion

1. Acesse o console **EC2 → Iniciar instância**
2. Configurações:
   - Nome: `Bastion Server`
   - AMI: `Amazon Linux 2023`
   - Tipo: `t3.micro`
   - Sub-rede: `Public Subnet`
   - IP público: **Ativado**
3. Crie um grupo de segurança:
   - Nome: `Bastion Security Group`
   - Regras de entrada:
     - Tipo: `SSH`
     - Origem: `0.0.0.0/0`

---

### 🔁 6. Criar o NAT Gateway

1. No console **NAT Gateways → Criar NAT Gateway**
   - Nome: `Lab NAT Gateway`
   - Sub-rede: `Public Subnet`
   - Alocar um IP elástico
2. Atualize a **Private Route Table**:
   - Adicione rota:
     - Destino: `0.0.0.0/0`
     - Target: **NAT Gateway**

---

## 🧠 Resultado Final

A rede foi configurada com sucesso!  
Agora:
- A **sub-rede pública** tem acesso direto à internet via **IGW**
- A **sub-rede privada** acessa a internet indiretamente via **NAT Gateway**
- O **bastion host** serve como ponto de acesso seguro para a instância privada

---

## 🖼️ Diagrama da Arquitetura

```plaintext
                   +----------------------------+
                   |        Internet            |
                   +-------------+--------------+
                                 |
                             (IGW - Lab IGW)
                                 |
                         +-------+--------+
                         |   Public Subnet  |
                         |  Bastion Server  |
                         |  NAT Gateway     |
                         +-------+----------+
                                 |
                        (Private Route Table)
                                 |
                         +-------+--------+
                         |  Private Subnet |
                         |  App Instance   |
                         +-----------------+

# PT/BR

# Projeto de Integração Financeira Multi-Empresa com n8n e Omie

Este projeto contém um conjunto de workflows do n8n desenvolvidos para automatizar e sincronizar operações financeiras (Contas a Pagar, Contas a Receber e Plano de Contas) entre múltiplas instâncias do ERP Omie. A solução utiliza um banco de dados **Supabase** como intermediário para mapear IDs e centralizar informações, garantindo a consistência dos dados entre a matriz e as filiais.

##  Tecnologias Utilizadas

* **n8n:** Plataforma de automação de workflows, orquestrando todo o processo.
* **Omie ERP:** Sistema de gestão de origem e destino dos dados financeiros.
* **Supabase (PostgreSQL):** Banco de dados utilizado como "De-Para" para mapear e armazenar a relação entre os registros das diferentes instâncias do Omie.
* **Webhooks:** Tecnologia usada para iniciar os workflows em tempo real a partir de eventos gerados no Omie.

##  Visão Geral dos Workflows

O projeto é composto por três workflows principais, cada um com uma responsabilidade específica no ecossistema de integração.

### 1. `Contas a Pagar (CaP)`

Este workflow automatiza a replicação de contas a pagar entre a instância de origem e as instâncias de destino do Omie.

* **Gatilho:** É acionado via **Webhook** quando uma conta a pagar é criada ou baixada na origem.
* **Funcionalidade Principal:**
    1.  **Inclusão:** Ao receber um novo título, o workflow consulta o Supabase para "traduzir" o código do fornecedor para o ID correspondente na filial de destino, formata as datas e cria o título na instância correta.
    2.  **Baixa (Exclusão):** Ao receber um evento de baixa, ele consulta o Supabase para encontrar os lançamentos correspondentes e realiza a baixa (lançando um pagamento com desconto) nas filiais onde o título foi replicado.

### 2. `Contas a Receber (CaR)`

De forma similar ao CaP, este workflow gerencia a sincronização de contas a receber, garantindo que os recebíveis lançados em uma filial sejam espelhados na matriz (ou vice-versa).

* **Gatilho:** Acionado por **Webhook** a partir de eventos de inclusão ou baixa de contas a receber.
* **Funcionalidade Principal:**
    1.  **Inclusão:** Além de traduzir o ID do cliente, este fluxo também realiza o mapeamento de **vendedores** e **contas correntes** através de tabelas de "De-Para" no Supabase antes de criar o título na instância de destino.
    2.  **Baixa (Exclusão):** Segue a mesma lógica do CaP, utilizando o Supabase para identificar os títulos vinculados e realizar a baixa correspondente.

### 3. `Sincronização de Contas Correntes (CCO)`

Este é um workflow administrativo de execução **manual**, projetado para garantir a padronização do plano de contas (contas correntes) em todo o ecossistema.

* **Gatilho:** **Manual**, deve ser executado a partir da interface do n8n.
* **Funcionalidade Principal:**
    1.  Busca a lista completa de contas correntes da instância "Matriz".
    2.  Itera sobre cada uma das filiais pré-configuradas.
    3.  Compara a lista da Matriz com a lista da filial atual.
    4.  Se uma conta existir na Matriz mas não na filial, o workflow a cria automaticamente, garantindo que o plano de contas permaneça consistente e padronizado em todas as instâncias.

##  Arquitetura e Lógica Central

A lógica fundamental do projeto baseia-se no uso do **Supabase como uma camada de mapeamento**. Como cada instância do Omie possui IDs únicos para seus próprios registros (clientes, fornecedores, contas, etc.), o Supabase armazena a relação entre eles.

* **Exemplo:** O cliente "Empresa X" pode ter o `ID 123` na filial SP e o `ID 456` na matriz PR. O Supabase armazena uma linha que diz `id_sp: 123` está associado a `id_pr: 456`.

Isso permite que os workflows de n8n atuem como um tradutor inteligente, garantindo a integridade referencial dos dados ao movê-los entre os sistemas.

##  Como Utilizar

1.  **Configuração:** Certifique-se de que as credenciais do Supabase e de todas as instâncias do Omie estão corretamente configuradas nas variáveis de ambiente ou nos nós correspondentes do n8n.
2.  **Workflows de CaP e CaR:** Ative os workflows no n8n. Configure os webhooks dentro do Omie para apontar para as URLs de produção fornecidas pelos nós "Webhook" de cada fluxo. A partir daí, a operação é automática.
3.  **Workflow de CCO:** Execute este workflow manualmente através do botão "Execute Workflow" na interface do n8n sempre que houver atualizações no plano de contas da matriz que precisem ser replicadas para as filiais.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# EN/US

# Multi-Company Financial Integration Project with n8n and Omie

This project contains a set of n8n workflows developed to automate and synchronize financial operations (Accounts Payable, Accounts Receivable and Chart of Accounts) between multiple instances of ERP Omie. The solution uses a **Supabase** database as an intermediary to map IDs and centralize information, ensuring data consistency between the headquarters and branches.

## Technologies Used

* **n8n:** Workflow automation platform, orchestrating the entire process.
* **Omie ERP:** Source and destination management system for financial data.
* **Supabase (PostgreSQL):** Database used as "From-To" to map and store the relationship between records from different Omie instances.
* **Webhooks:** Technology used to start workflows in real time based on events generated in Omie.

## Workflows Overview

The project is made up of three main workflows, each with a specific responsibility in the integration ecosystem.

### 1. `Accounts Payable (CaP)`

This workflow automates the replication of accounts payable between the source instance and the target Omie instances.

* **Trigger:** It is activated via **Webhook** when a payable account is created or downloaded at origin.
* **Main Functionality:**
    1. **Inclusion:** Upon receiving a new title, the workflow queries Supabase to "translate" the supplier code to the corresponding ID in the target branch, formats the dates, and creates the title in the correct instance.
    2. **Write-off (Deletion):** When receiving a write-off event, it queries Supabase to find the corresponding entries and carries out the write-off (posting a discounted payment) in the branches where the security was replicated.

### 2. `Accounts Receivable (CaR)`

Similar to CaP, this workflow manages the synchronization of accounts receivable, ensuring that receivables posted at a branch are mirrored at headquarters (or vice versa).

* **Trigger:** Triggered by **Webhook** from accounts receivable inclusion or write-off events.
* **Main Functionality:**
    1. **Inclusion:** In addition to translating the customer ID, this flow also performs the mapping of **sellers** and **current accounts** through "From-To" tables in Supabase before creating the title in the target instance.
    2. **Write-off (Exclusion):** Follows the same logic as CaP, using Supabase to identify linked securities and carry out the corresponding write-off.

### 3. `Current Account Synchronization (CCO)`

This is a **manual** administrative workflow designed to ensure standardization of the chart of accounts (current accounts) across the entire ecosystem.

* **Trigger:** **Manual**, must be executed from the n8n interface.
* **Main Functionality:**
    1. Search for the complete list of current accounts in the "Matrix" instance.
    2. Iterates over each of the pre-configured branches.
    3. Compare the Headquarters list with the current branch list.
    4. If an account exists in the Head Office but not in the branch, the workflow automatically creates it, ensuring that the chart of accounts remains consistent and standardized across all instances.

## Architecture and Core Logic

The fundamental logic of the project is based on the use of **Supabase as a mapping layer**. Since each Omie instance has unique IDs for its own records (customers, suppliers, accounts, etc.), Supabase stores the relationship between them.

* **Example:** The customer "Company X" may have the `ID 123` in the SP branch and the `ID 456` in the PR matrix. Supabase stores a line that says `id_sp: 123` is associated with `id_pr: 456`.

This allows n8n workflows to act as an intelligent translator, ensuring the referential integrity of data when moving it between systems.

## How to Use

1. **Configuration:** Ensure that the credentials for Supabase and all Omie instances are correctly configured in the environment variables or corresponding n8n nodes.
2. **CaP and CaR workflows:** Activate workflows in n8n. Configure the webhooks within Omie to point to the production URLs provided by each stream's "Webhook" nodes. From then on, the operation is automatic.
3. **CCO Workflow:** Execute this workflow manually using the "Execute Workflow" button in the n8n interface whenever there are updates to the head office's chart of accounts that need to be replicated to the branches.

# 🚚 Logistics & Freight Automation Workflows (n8n)

Sistema integrado de automação e auditoria logística desenvolvido em **n8n**. O projeto automatiza a extração de documentos fiscais (NFS-e/PDF), consulta e recuperação de XMLs de transporte via API SOAP, conciliação de ocorrências de frete (B2C x Lupeon) e auditoria de pagamentos com marcação dinâmica em planilhas do Google Sheets.

## 📌 Sumário

- [Visão Geral](#-visão-geral)
- [Modules & Workflows](#-modules--workflows)
  - [1. AUTO CSV (Geração de CSV para Lupeon)](#1-auto-csv-geração-de-csv-para-lupeon)
  - [2. ANÁLISE LUPEON (Conciliação B2C x Lupeon)](#2-análise-lupeon-conciliação-b2c-x-lupeon)
  - [3. PAGOS (Auditoria de Pagamentos MA2 x PepsiCo)](#3-pagos-auditoria-de-pagamentos-ma2-x-pepsico)
- [🛠️ Tecnologias Utilizadas](#️-tecnologias-utilizadas)
- [⚙️ Arquitetura do Processo](#️-arquitetura-do-processo)
- [🚀 Como Importar e Usar](#-como-importar-e-usar)
- [📄 Licença](#-licença)

## 🔍 Visão Geral

No setor de logística e gestão de fretes, a validação manual de Notas Fiscais de Serviço (NFS-e), a verificação de Ocorrências (OCs) nos sistemas TMS e o cruzamento de relatórios de pagamento são processos suscetíveis a falhas e alto consumo de tempo.

Esta suíte de workflows no **n8n** resolve esses gargalos:

1. **Reduz o tempo manual** de digitação e geração de arquivos `.csv` para importação de fretes.
2. **Identifica divergências e pendências** de lançamento de ocorrências de frete extra entre o B2C e o sistema Lupeon.
3. **Automatiza a auditoria financeira** cruzando relatórios de pagamento da PepsiCo com planilhas de controle no Google Sheets em tempo real.

## 📦 Modules & Workflows

### 1. AUTO CSV (Geração de CSV para Lupeon)

- **Função:** Leitura automática de PDFs de NFS-e de pastas locais, extração de dados via Expressões Regulares (Regex), consulta automática no Webmail Zimbra via SOAP para download do XML da NFe de origem e formatação do arquivo CSV exigido pelo TMS Lupeon.
- **Destaques Técnicos:**
  - Validação de layout de NFS-e (Prefeitura de São Paulo / Eventos de Deslocamento Vazio).
  - Autenticação e requisições SOAP para busca e download de anexos XML no Zimbra.
  - Movimentação automática dos PDFs processados via scripts **PowerShell**.

### 2. ANÁLISE LUPEON (Conciliação B2C x Lupeon)

- **Função:** Leitura dos relatórios do Monitoramento B2C e do cadastro do Lupeon para verificar se todas as OCs (Ocorrências) válidas foram corretamente lançadas no sistema.
- **Destaques Técnicos:**
  - Filtragem inteligente por regra de negócio (separa OCs elegíveis para cobrança como reentrega, retorno parcial, etc., de OCs sem custo/descartadas).
  - Cruzamento de dados de relatórios Excel pesados usando PowerShell COM Objects e Base64.
  - Geração automática do relatório consolidado de divergências (`Relatorio_Pendencias_DT.xlsx`).

### 3. PAGOS (Auditoria de Pagamentos MA2 x PepsiCo)

- **Função:** Comparação automática do relatório de fretes pagos pela PepsiCo (`.xls`) contra a planilha de controle no Google Sheets.
- **Destaques Técnicos:**
  - Consulta multi-abas em lote (`batchGetByDataFilter`) via **Google Sheets API v4**.
  - Destaque visual automático (formatação condicional via API pintando as células encontradas em cor Ciano).
  - Exportação automática de relatório em CSV contendo apenas as notas faltantes/não localizadas (`Notas_Faltantes_MA2.csv`).

## 🛠️ Tecnologias Utilizadas

- **Orquestração de Processos:** [n8n](https://n8n.io/)
- **Linguagem Principal de Scripting:** JavaScript (Node.js ES6+)
- **Automação de SO / Arquivos:** PowerShell (Windows Management)
- **APIs e Protocolos:**
  - Google Sheets API v4 (REST & Service Accounts)
  - Zimbra Webmail API (SOAP / XML over HTTPS)
- **Processamento de Dados:** Regular Expressions (Regex), Excel Extraction (XLSX/XLS), JSON Mapping.

## ⚙️ Arquitetura do Processo

```
[ NFS-e / PDFs em Disco (Z:) ] ──► [ AUTO CSV ] ──► [ Zimbra SOAP API ] ──► [ CSV Lupeon ]
                                                                                │
[ Relatórios B2C & COLETAS ] ────► [ ANÁLISE LUPEON ] ◄─────────────────────────┘
                                           │
                                           ▼
                            [ Relatório de Pendências .xlsx ]

[ Relatório Pagos PepsiCo ] ──► [ PAGOS ] ──► [ Google Sheets API v4 ] ──► [ Destaque Ciano + CSV Faltantes ]
```

## 🚀 Como Importar e Usar

### Pré-requisitos

- Instância do **n8n** rodando localmente ou em servidor (Docker / Node.js).
- Acesso de leitura/escrita aos diretórios locais mapeados (caso utilize executores PowerShell no Windows).
- Credenciais de Conta de Serviço (*Service Account*) no **Google Cloud Console** com permissão na planilha do Google Sheets.
- Credenciais de acesso ao servidor Zimbra (E-mail e Senha de integração).

### Passos para Importação

1. **Faça o clone ou download deste repositório:**
   ```bash
   git clone https://github.com/sousa2008/n8n-logistics-automation
   ```

2. **Importar os Workflows no n8n:**
   - Acesse seu painel do **n8n**.
   - Vá em **Workflows** -> **Import from File**.
   - Selecione cada um dos arquivos `.json` presentes no repositório (`AUTO CSV.json`, `ANALISE LUPEON.json`, `PAGOS.json`).

3. **Configuração de Variáveis e Credenciais:**
   - Substitua os marcadores `EMAIL-AQUI`, `SENHA-AQUI`, `ID-PLANILHA-AQUI` e `ENDERECO-AQUI` dentro dos nós de cada workflow pelas suas respectivas configurações e caminhos do seu ambiente.
   - Configure a credencial do **Google API Service Account** nos nós de integração do Google Sheets no workflow `PAGOS`.

## 📄 Licença

Este projeto está sob a licença [MIT](LICENSE).

---
*Desenvolvido para automação, auditoria e otimização de processos logísticos.*
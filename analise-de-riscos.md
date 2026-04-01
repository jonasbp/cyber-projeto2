# Análise de Riscos Cibernéticos — ABC PLACE
**Roteiro 2 &mdash; Cybersegurança**
Jonas Bonfá Pelegrina
---

## 1. Descrição das Ameaças e Vulnerabilidades

### 1.1 Vulnerabilidades Identificadas


#### VUL-01 — Ausência de Boas Práticas de Segurança (Hardening)
Dispositivos de rede (switches, roteadores, firewalls) e servidores mantêm as **configurações padrão de fábrica**. Isso inclui senhas padrão (ex: `admin/admin`, `root/root`), serviços e portas desnecessários ativos, e protocolos inseguros habilitados, cada dispositivo representa uma porta de entrada para atacantes.

*Como pode ser explorado:* Um atacante que realiza uma varredura de rede com ferramentas como Nmap pode identificar serviços expostos com configurações default e obter acesso usando credenciais padrão conhecidas publicamente.

#### VUL-02 — Uso de Contas Privilegiadas Compartilhadas
A equipe de TI utiliza os usuários `administrator` (Windows) e `root` (Linux) para acessar servidores. Não há contas individuais, não há MFA (Multi-Factor Authentication) e não existe o princípio do menor privilégio. Todos os técnicos têm acesso irrestrito a todos os sistemas.

*Como pode ser explorado:* Um técnico mal-intencionado ou uma conta comprometida tem acesso total a toda a infraestrutura. Além disso, sem auditoria individual, é impossível rastrear quem executou qual ação, dificultando a resposta a incidentes.

#### VUL-03 — Ausência de Gestão de Vulnerabilidades
Não existe política de atualização de sistemas, nem análise periódica do estado da rede. Sistemas operacionais, aplicações (WordPress/WooCommerce) e plugins podem estar com versões desatualizadas e vulnerabilidades conhecidas publicamente catalogadas no CVE.

*Como pode ser explorado:* O WooCommerce e seus plugins são alvos frequentes. Vulnerabilidades como a CVE-2023-28121 (WooCommerce Payments) permitiram acesso de nível administrador sem autenticação. Sem gestão de patches, o sistema pode estar vulnerável por meses após a divulgação pública de uma falha crítica.

#### VUL-04 — Acesso Remoto Inseguro (Sem VPN)
O acesso administrativo aos servidores é realizado diretamente pela internet, sem o uso de VPN. Credenciais de acesso (SSH, RDP, painéis web) trafegam expostas ou com proteção insuficiente.

*Como pode ser explorado:* Um atacante posicionado entre o técnico e o servidor (ataque Man-in-the-Middle) pode capturar credenciais de acesso. Além disso, serviços como RDP expostos diretamente na internet são alvos constantes de ataques automatizados.

#### VUL-05 — Ausência de Firewall e WAF
Não há política de firewall configurada e não existe Web Application Firewall (WAF) protegendo a plataforma de e-commerce. Qualquer origem pode se comunicar com qualquer serviço na infraestrutura.

*Como pode ser explorado:* Sem firewall, ataques de varredura, brute force e exploração de serviços ficam sem nenhum bloqueio. Sem WAF, requisições maliciosas contendo payloads de SQLi, XSS ou outras injeções chegam diretamente à aplicação sem filtragem.

#### VUL-06 — Aplicação e SGBD no Mesmo Servidor
A plataforma WooCommerce e seu banco de dados MySQL/MariaDB estão hospedados no mesmo servidor físico/virtual. Não há segmentação de rede entre a camada de apresentação (web) e a camada de dados.

*Como pode ser explorado:* Se um atacante compromete a aplicação web via SQLinjection, ele obtém acesso direto ao banco de dados. A inexistência de uma rede separada elimina qualquer camada adicional de proteção, facilitando o dump completo de todos os dados de clientes.

#### VUL-07 — Ausência de Monitoramento de Segurança
Não há ferramentas para monitorar logs, detectar comportamento anômalo ou alertar sobre atividade suspeita.

*Como pode ser explorado:* Atacantes realizam movimentos lentos e silenciosos para não serem detectados. Sem monitoramento, um atacante pode permanecer dentro da rede por semanas ou meses, exfiltrando dados gradualmente antes de uma ação mais destrutiva.

---

### 1.2 Ameaças Mapeadas

#### AME-01 — Ameaça Interna (Insider Threat / Hacking Interno)
**Descrição:** Funcionários ou ex-funcionários com conhecimento interno da infraestrutura podem abusar de acessos privilegiados para roubar dados, sabotar sistemas ou instalar backdoors. O fato de um ex-funcionário ter configurado toda a infraestrutura e potencialmente ainda conhecer as credenciais agrava significativamente este risco.

**Vetor de exploração:** Uso de credenciais root/administrator compartilhadas para acessar, copiar ou destruir dados sem deixar rastros auditáveis.

---

#### AME-02 — Malware e Ransomware
**Descrição:** Softwares maliciosos que podem comprometer, criptografar ou destruir os dados e sistemas da empresa.

**Vetor de exploração:** Phishing direcionado, exploração de vulnerabilidades não corrigidas nos servidores ou download acidental de software malicioso. Sem antivírus e sem monitoramento, o malware pode se propagar pela rede inteira antes de ser detectado.

---

#### AME-03 — Negação de Serviço (DoS / DDoS)
**Descrição:** Ataques que visam tornar a plataforma de vendas indisponível, seja por inundação de requisições (volumétrico) ou por exploração de vulnerabilidades de protocolo.

**Vetor de exploração:** Sem WAF, sem firewall de perímetro e sem serviços de mitigação de DDoS, a plataforma é diretamente acessível e vulnerável. E-commerces são alvos frequentes durante datas comemorativas de alta receita.

---

#### AME-04 — Injeção de Código (SQLi, XSS, RCE)
**Descrição:** Ataques que exploram falhas na validação de entrada de dados para executar comandos maliciosos no servidor ou banco de dados.

**Vetor de exploração:** Formulários de busca, login, checkout e campos de pedido da plataforma WooCommerce podem conter payloads SQLi. Sem WAF e com o banco de dados no mesmo servidor, um SQLinjection bem-sucedido resulta em controle total sobre todos os dados dos clientes.


---

#### AME-05 — Comprometimento de Informações Sensíveis
**Descrição:** Tentativa ou sucesso na destruição, corrupção ou divulgação não autorizada de dados corporativos ou de clientes, incluindo informações de propriedade intelectual.

**Vetor de exploração:** Combinação de múltiplas vulnerabilidades — acesso sem VPN + contas root compartilhadas + sem criptografia em repouso — cria múltiplos caminhos para exfiltração de dados. Em um e-commerce, os dados mais valiosos são: CPF, endereços, dados de cartões de crédito e histórico de compras.

---

#### AME-06 — Interceptação de Comunicações
**Descrição:** Captura de dados em trânsito por atacantes posicionados na rota de comunicação entre o técnico de TI e os servidores, ou entre clientes e a plataforma.

**Vetor de exploração:** Acesso administrativo sem VPN expõe credenciais em canais potencialmente não criptografados. Ferramentas como Wireshark ou Ettercap permitem capturar credenciais de login, tokens de sessão, CPFs e dados de cartão em tempo real.

---

## 2. Análise de Riscos e Impacto na Cadeia de Valor

### 2.1 Metodologia de Avaliação

Utilizamos a metodologia de **Probabilidade × Impacto**, com escala de 1 a 4 para cada dimensão:

| Classificação | Score | Significado |
|---|---|---|
| Extremo | > 9 | Risco inaceitável — ação imediata obrigatória |
| Alto | 7–9 | Geralmente inaceitável — ação urgente |
| Moderado | 3–6 | Possivelmente aceitável — monitorar e mitigar |
| Baixo | 1–2 | Risco aceitável com controles básicos |

### 2.2 Tabela de Riscos

| # | Ameaça | Prob. | Impacto | Score | Classificação |
|---|---|:---:|:---:|:---:|---|
| 1 | Ransomware / Malware | 4 | 4 | **16** | Extremo |
| 2 | Injeção de Código (SQLi/XSS) | 4 | 4 | **16** | Extremo |
| 3 | Credenciais Padrão / Brute Force | 4 | 4 | **16** | Extremo |
| 4 | DoS / DDoS | 4 | 4 | **16** | Extremo |
| 5 | Ameaça Interna (Insider) | 3 | 4 | **12** | Extremo |
| 6 | Comprometimento de Informações | 3 | 4 | **12** | Extremo |
| 7 | Eavesdropping / MITM | 3 | 4 | **12** | Extremo |
| 8 | Persistência Não Detectada | 4 | 3 | **12** | Extremo |

**Resultado: 100% das ameaças foram classificadas como risco Extremo.**

### 2.3 Impacto na Cadeia de Valor

A cadeia de valor da ABC PLACE como e-commerce se estrutura em:

```
[Fornecedores] → [Estoque/CD] → [Plataforma Online] → [Logística] → [Clientes]
```

Os impactos mapeados por elo da cadeia:

**Plataforma Online (impacto direto):**
- DDoS ou Ransomware → plataforma indisponível → R$ 0 de receita durante o ataque
- SQLi → base de clientes comprometida → LGPD: multa de até 2% do faturamento (limitada a R$ 50M)
- Comprometimento de dados de cartão → chargeback em massa, bloqueio de gateway de pagamento

**Operações e Estoque:**
- Ransomware nos servidores → sistema de gestão de estoque inoperante → CD paralisado
- Insider → manipulação de pedidos, desaparecimento de estoque registrado digitalmente

**Reputação e Mercado:**
- Vazamento de dados → notícias negativas → queda de 20–40% em conversão estimada pós-incidente
- Perda de certificações (PCI-DSS para pagamentos) → impossibilidade de processar cartões

**Estratégia de Migração Cloud:**
- Incidente grave durante a migração → interrupção indefinida do projeto estratégico
- Reputação com investidores e parceiros comprometida

---

## 3. Planos de Ação para Mitigação dos Riscos

### 3.1 Fase 1 — Ações Emergenciais (0–30 dias)

| Prioridade | Ação | Vulnerabilidade Mitigada | Responsável | Prazo |
|:---:|---|---|---|---|
| 1 | Alterar TODAS as credenciais padrão de dispositivos e servidores | VUL-01, VUL-02 | TI | 3 dias |
| 2 | Implantar VPN para acesso administrativo remoto | VUL-04 | TI | 5 dias |
| 3 | Ativar WAF e configurar regras de firewall de perímetro | VUL-05 | TI | 7 dias |
| 4 | Criar contas individuais e habilitar MFA para a equipe de TI | VUL-02 | TI / RH | 7 dias |
| 5 | Realizar backup completo, offsite e testar restauração | VUL-03, VUL-07 | TI | 5 dias |

**Justificativa:** Estas ações eliminam os vetores mais imediatos de comprometimento e garantem a capacidade de recuperação caso um incidente ocorra antes das demais fases.

### 3.2 Fase 2 — Estruturação (30–90 dias)

| Ação | Detalhes | Responsável | Prazo |
|---|---|---|---|
| Separar Aplicação e Banco de Dados | Migrar SGBD para servidor dedicado em rede privada | TI / Dev | 30 dias |
| Hardening dos Servidores | Aplicar CIS Benchmark; desabilitar serviços desnecessários; instalar EDR/antivírus | TI | 45 dias |
| Implementar Monitoramento (SIEM/IDS) | Ferramentas como Wazuh (open-source) ou Splunk; centralizar logs | TI / SOC | 60 dias |
| Gestão de Vulnerabilidades | Processo de patch management; scanner de vulnerabilidades (OpenVAS/Nessus); primeiro pentest | TI | 60 dias |
| Política de Segurança da Informação | Elaborar PSI; classificação de dados; procedimentos de incidente; alinhamento LGPD | TI / Jurídico | 60 dias |
| Treinamento e Conscientização | Programa para todos os colaboradores: phishing, senhas, notificação de incidentes | RH / TI | 90 dias |

### 3.3 Fase 3 — Maturidade (90+ dias)

| Ação | Detalhes | Responsável |
|---|---|---|
| Framework de Segurança | Adotar NIST CSF ou iniciar jornada ISO 27001 | CISO / Diretoria |
| Pentest Anual | Contratar empresa especializada para testes externos e internos | TI / Terceiros |
| PAM — Gestão de Acessos Privilegiados | Implantar CyberArk, BeyondTrust ou similar | TI |
| Conformidade LGPD | Nomear DPO; mapear dados pessoais; processo de notificação à ANPD | Jurídico / DPO |
| Segurança Cloud-Native | Zero Trust, IAM robusto, CSPM no ambiente cloud migrado | TI / Cloud |
| Cultura de Segurança | Programa contínuo de treinamentos e simulações de phishing | RH / TI |

---

## 4. Recomendações Finais

### Priorização por Impacto Imediato

As 3 ações com maior relação custo/benefício de implementação imediata são:

1. **Alterar credenciais padrão** — custo zero, elimina o vetor de ataque mais explorado
2. **Implantar VPN** — baixo custo, elimina exposição administrativa à internet
3. **Ativar WAF** — custo moderado, protege diretamente a plataforma de vendas e os dados de clientes

### Pré-requisito para a Migração Cloud

A migração cloud **não deve avançar** sem a conclusão da Fase 1. Migrar uma infraestrutura insegura para a cloud não resolve os problemas de segurança — apenas os amplifica, pois aumenta a superfície de ataque e adiciona novas camadas de configuração que precisam ser protegidas.

### Consideração sobre LGPD

A ABC PLACE processa dados pessoais e financeiros de milhares de clientes. Em caso de vazamento de dados, a LGPD (Lei 13.709/2018) prevê:
- Multa de até 2% do faturamento do último exercício
- Limite de R$ 50 milhões por infração
- Obrigação de notificação à ANPD e aos titulares afetados
- Danos reputacionais de longo prazo

A adequação à LGPD deve ser tratada como prioridade jurídica e estratégica, não apenas técnica.

---

Utilizado ferramentas de IA para este trabalho
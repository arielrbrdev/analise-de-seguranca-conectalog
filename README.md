# 🔍 Projeto Final: Resposta a Incidentes e Análise Forense em Dispositivo IoT

## 🛡️ Destaques

* [cite_start]**Foco:** Blue Team (Resposta a Incidentes, Threat Hunting)[cite: 60, 100].
* [cite_start]**Ameaça Simulada:** Compromisso de um dispositivo IoT (`SmartBox CL-2000`) para exfiltração de dados via DNS Tunneling[cite: 69, 92].
* [cite_start]**Habilidades Demonstradas:** Análise de pacotes (Wireshark), Inteligência de Ameaças (OSINT), Análise Forense de Firmware (Linux embarcado), Mapeamento MITRE ATT&CK, Elaboração de Plano de Correção[cite: 64, 92, 101, 138, 286].

---

## 🚀 O Desafio (Cenário do Incidente)

[cite_start]A equipe de segurança da empresa ConectaLog detectou um **volume anormal de tráfego de rede** e temia um vazamento de dados sigilosos (exfiltração)[cite: 67, 69].

[cite_start]A missão da consultoria era investigar o dispositivo SmartBox CL-2000, suspeito de ser o ponto de origem, e responder a quatro perguntas críticas [cite: 70-79]:

1.  O dispositivo é seguro?
2.  Dados estão sendo vazados?
3.  Existem evidências de comprometimento?
4.  Quais são as vulnerabilidades e o plano de correção?

---

## 🛠️ Metodologia e Descobertas

O processo investigativo foi dividido em três fases principais:

### 1. Análise de Rede e DNS Tunneling (Fase 1)

* [cite_start]**Anomalia Identificada:** O host suspeito (192.168.1.50) foi responsável por mais de 95% das consultas DNS, todas direcionadas a um único domínio: `update.dyn-DNS-free.com`[cite: 91].
* [cite_start]**Vetor de Exfiltração:** A análise forense no Wireshark confirmou **DNS Tunneling** (encapsulamento de dados em longas *strings* de subdomínios e *beaconing* em intervalos regulares)[cite: 92, 94].
* [cite_start]**Mapeamento ATT&CK:** Esta técnica foi mapeada para **T1071.004** (Protocolo Application Layer: DNS)[cite: 96].

### 2. Caça à Ameaça (Threat Hunting / OSINT) (Fase 2)

* [cite_start]**IoCs Coletados:** O domínio de Comando e Controle (C2) malicioso **`army-lk.org`** e o IP **`156.234.249.236`** foram extraídos do tráfego de rede[cite: 134, 135].
* [cite_start]**Reputação Maliciosa:** A investigação via VirusTotal e WHOIS confirmou que o domínio é um *lookalike* de um domínio oficial [cite: 115] [cite_start]e foi registrado recentemente (típico de infraestrutura *ad-hoc*)[cite: 105].
* [cite_start]**Associação:** O IP C2 foi associado ao malware **Cobalt Strike** [cite: 129][cite_start], confirmando que se trata de uma ameaça persistente avançada (APT)[cite: 130].

### 3. Análise de Causa Raiz (Forense de Firmware) (Fase 3)

A análise do *firmware* do dispositivo revelou o vetor de entrada e o método de escalada:

| Vetor | Descoberta | Citação |
| :--- | :--- | :--- |
| **Acesso Inicial** | [cite_start]**Credenciais Padrão/Fracas:** Conta `maint` com senha trivial (`maint`) exposta no arquivo `/etc/shadow`[cite: 169, 174]. [cite_start]| [cite: 177] |
| **Serviço Vulnerável** | [cite_start]**Dropbear SSHd v2017.75** (desatualizado e sem *hardening*)[cite: 181, 183]. [cite_start]| [cite: 192] |
| **Escala de Privilégios** | [cite_start]**Vulnerabilidade de Permissão:** O script `check_updates.sh` [cite: 196] é **executado como root**, mas possui **permissão de escrita para o usuário `maint`**[cite: 246]. [cite_start]| [cite: 256] |
| **Encadeamento Final** | [cite_start]O atacante usou a senha fraca (`maint/maint`) para SSH, editou o script para inserir código malicioso, e obteve controle total (root) na próxima execução do script[cite: 261]. [cite_start]| [cite: 259-261] |

---

## ✅ Conclusão e Plano de Correção

[cite_start]A investigação confirmou que o dispositivo **não é seguro** e que dados estão sendo vazados[cite: 266]. [cite_start]A causa raiz é a combinação de **credenciais fracas** e uma **configuração insegura** de scripts privilegiados[cite: 249].

As principais recomendações incluem:

* [cite_start]**Mitigação Imediata:** Desabilitar ou modificar todas as credenciais padrão[cite: 287].
* **Correção de Vulnerabilidades:** Atualizar o Dropbear SSH [cite: 288] e, crucialmente, **revisar as permissões** do `check_updates.sh` para que apenas `root` possa modificá-lo[cite: 289].
* [cite_start]**Monitoramento:** Inserir os IoCs (`army-lk.org` e `156.234.249.236`) nas listas de bloqueio do firewall e SIEM[cite: 293].

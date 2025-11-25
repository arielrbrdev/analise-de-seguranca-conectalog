# 🔍 Projeto Final: Resposta a Incidentes e Análise Forense em Dispositivo IoT

## 🛡️ Destaques

* **Foco:** Blue Team (Resposta a Incidentes, Threat Hunting).
* **Ameaça Simulada:** Compromisso de um dispositivo IoT (`SmartBox CL-2000`) para exfiltração de dados via DNS Tunneling.
* **Habilidades Demonstradas:** Análise de pacotes (Wireshark), Inteligência de Ameaças (OSINT), Análise Forense de Firmware (Linux embarcado), Mapeamento MITRE ATT&CK, Elaboração de Plano de Correção.

---

## 🚀 O Desafio (Cenário do Incidente)

A equipe de segurança da empresa ConectaLog detectou um **volume anormal de tráfego de rede** e temia um vazamento de dados sigilosos (exfiltração).

A missão da consultoria era investigar o dispositivo SmartBox CL-2000, suspeito de ser o ponto de origem, e responder a quatro perguntas críticas:

1.  O dispositivo é seguro?
2.  Dados estão sendo vazados?
3.  Existem evidências de comprometimento?
4.  Quais são as vulnerabilidades e o plano de correção?

---

## 🛠️ Metodologia e Descobertas

O processo investigativo foi dividido em três fases principais:

### 1. Análise de Rede e DNS Tunneling (Fase 1)

* **Anomalia Identificada:** O host suspeito (192.168.1.50) foi responsável por mais de 95% das consultas DNS, todas direcionadas a um único domínio: `update.dyn-DNS-free.com`.
* **Vetor de Exfiltração:** A análise forense no Wireshark confirmou **DNS Tunneling** (encapsulamento de dados em longas *strings* de subdomínios e *beaconing* em intervalos regulares).
* **Mapeamento ATT&CK:** Esta técnica foi mapeada para **T1071.004** (Protocolo Application Layer: DNS).

### 2. Caça à Ameaça (Threat Hunting / OSINT) (Fase 2)

* **IoCs Coletados:** O domínio de Comando e Controle (C2) malicioso **`army-lk.org`** e o IP **`156.234.249.236`** foram extraídos do tráfego de rede.
* **Reputação Maliciosa:** A investigação via VirusTotal e WHOIS confirmou que o domínio é um *lookalike* de um domínio oficial e foi registrado recentemente (típico de infraestrutura *ad-hoc*).
* **Associação:** O IP C2 foi associado ao malware **Cobalt Strike**, confirmando que se trata de uma ameaça persistente avançada (APT).

### 3. Análise de Causa Raiz (Forense de Firmware) (Fase 3)

A análise do *firmware* do dispositivo revelou o vetor de entrada e o método de escalada:

| Vetor | Descoberta |
| :--- | :--- |
| **Acesso Inicial** | **Credenciais Padrão/Fracas:** Conta `maint` com senha trivial (`maint`) exposta no arquivo `/etc/shadow`. |
| **Serviço Vulnerável** | **Dropbear SSHd v2017.75** (desatualizado e sem *hardening*). |
| **Escala de Privilégios** | **Vulnerabilidade de Permissão:** O script `check_updates.sh` é **executado como root**, mas possui **permissão de escrita para o usuário `maint`**. |
| **Encadeamento Final** | O atacante usou a senha fraca (`maint/maint`) para SSH, editou o script para inserir código malicioso, e obteve controle total (root) na próxima execução do script. |

---

## ✅ Conclusão e Plano de Correção

A investigação confirmou que o dispositivo **não é seguro** e que dados estão sendo vazados. A causa raiz é a combinação de **credenciais fracas** e uma **configuração insegura** de scripts privilegiados.

As principais recomendações incluem:

* **Mitigação Imediata:** Desabilitar ou modificar todas as credenciais padrão.
* **Correção de Vulnerabilidades:** Atualizar o Dropbear SSH e, crucialmente, **revisar as permissões** do `check_updates.sh` para que apenas `root` possa modificá-lo.
* **Monitoramento:** Inserir os IoCs (`army-lk.org` e `156.234.249.236`) nas listas de bloqueio do firewall e SIEM.

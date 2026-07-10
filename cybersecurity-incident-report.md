# 🛡️ Cybersecurity Incident Report
**Analista:** Davi Santos  
**Data:** 2024  
**Classificação:** 🔴 Crítico  
**Tipo de Incidente:** Ataque de Negação de Serviço — SYN Flood (DoS)

---

## 📋 Seção 1 — Identificação do Tipo de Ataque

### 🔍 Hipótese do Incidente

A mensagem de timeout de conexão apresentada aos visitantes do site indica que o servidor web (`192.0.2.1`) ficou incapaz de processar novas conexões legítimas. Com base na análise dos logs de tráfego TCP/HTTP capturados pelo Wireshark, a causa mais provável é um **ataque de SYN Flood**, uma variante de ataque de **Negação de Serviço (DoS)**.

### 📊 O que os logs revelam

| Observação | Detalhe |
|---|---|
| **IP atacante** | `203.0.113.0` |
| **IP do servidor** | `192.0.2.1` |
| **Porta alvo** | `443` (HTTPS) |
| **Porta de origem do atacante** | `54770` |
| **Primeiro pacote suspeito** | Pacote #52 — timestamp 3.390692s |
| **Padrão identificado** | Centenas de pacotes `[SYN]` consecutivos sem conclusão do handshake |
| **Duração observada** | De ~3.39s até ~51.82s (quase 50 segundos de ataque contínuo) |
| **Total de SYNs maliciosos** | 100+ pacotes apenas do IP `203.0.113.0` |

### 🚨 Classificação do Evento

> **SYN Flood — Ataque de Negação de Serviço (DoS)**  
> O IP `203.0.113.0` enviou um volume massivo de pacotes `[SYN]` ao servidor, esgotando sua tabela de conexões semi-abertas (backlog) e impedindo que usuários legítimos estabelecessem conexão.

---

## 📋 Seção 2 — Como o Ataque Causou o Mau Funcionamento

### 🤝 O Three-Way Handshake TCP (funcionamento normal)

Para que qualquer visitante acesse o site, o protocolo TCP exige uma sequência de 3 etapas:

```
Cliente                          Servidor
   |                                 |
   |──── [SYN] ─────────────────────>|  1️⃣ Cliente solicita conexão
   |                                 |
   |<─── [SYN, ACK] ────────────────|  2️⃣ Servidor confirma e reserva recursos
   |                                 |
   |──── [ACK] ─────────────────────>|  3️⃣ Cliente confirma — conexão estabelecida!
   |                                 |
```

**Etapa 1 — SYN (Synchronize):**  
O cliente envia um pacote `SYN` ao servidor sinalizando que deseja iniciar uma conexão. O servidor recebe e reserva recursos em sua memória (entrada na tabela de conexões semi-abertas).

**Etapa 2 — SYN-ACK (Synchronize-Acknowledge):**  
O servidor responde com um pacote `SYN-ACK`, confirmando que recebeu a solicitação e está aguardando a confirmação final do cliente. Os recursos permanecem reservados.

**Etapa 3 — ACK (Acknowledge):**  
O cliente responde com um `ACK` final, completando o handshake. A conexão é estabelecida e o servidor libera os recursos reservados para uso ativo.

---

### 💥 O que acontece durante um ataque SYN Flood

Quando um agente malicioso envia um grande volume de pacotes `SYN` simultaneamente:

```
Atacante (203.0.113.0)           Servidor (192.0.2.1)
   |                                 |
   |──── [SYN] ─────────────────────>|  ✅ Servidor reserva recursos
   |──── [SYN] ─────────────────────>|  ✅ Servidor reserva recursos
   |──── [SYN] ─────────────────────>|  ✅ Servidor reserva recursos
   |──── [SYN] ─────────────────────>|  ⚠️  Backlog enchendo...
   |──── [SYN] ─────────────────────>|  ⚠️  Backlog enchendo...
   |──── [SYN] ─────────────────────>|  🔴 Backlog CHEIO
   |                                 |
   ❌ ACK nunca é enviado            ❌ Recursos nunca são liberados
```

O servidor envia `SYN-ACK` para cada pacote recebido e aguarda o `ACK` final — que **nunca chega**. Isso faz com que cada entrada na tabela de conexões semi-abertas permaneça ocupada até expirar pelo timeout, esgotando progressivamente a capacidade do servidor.

---

### 📜 O que os logs indicam e como isso afetou o servidor

**Fase 1 — Tráfego normal (pacotes 47–54):**  
Usuários legítimos (`198.51.100.23`, `198.51.100.14`) completam o handshake normalmente e acessam `/sales.html` com resposta `HTTP 200 OK`.

**Fase 2 — Início do ataque (pacote 52):**  
O IP `203.0.113.0` começa a enviar pacotes `[SYN]` repetidos na porta `54770→443`, sem nunca completar o handshake. O servidor responde com `[SYN, ACK]` em cada um, reservando recursos.

**Fase 3 — Degradação (pacotes 63–77):**  
Novos usuários legítimos (`198.51.100.5`, `198.51.100.16`) tentam conectar, mas o servidor já começa a rejeitar com `[RST, ACK]`. O usuário `198.51.100.5` recebe `HTTP 504 Gateway Time-out` no pacote #77.

**Fase 4 — Colapso total (pacotes 78+):**  
O servidor passa a responder apenas com `[RST, ACK]` para todas as novas conexões. O IP atacante continua enviando `[SYN]` ininterruptamente até o final do log (pacote 214+), mantendo o servidor sobrecarregado.

---

## 🛡️ Seção 3 — Recomendações de Mitigação

| Medida | Descrição |
|---|---|
| **SYN Cookies** | Habilitar SYN Cookies no servidor para não reservar recursos antes do ACK final |
| **Rate Limiting** | Limitar o número de conexões SYN por IP por segundo no firewall |
| **Bloqueio de IP** | Bloquear imediatamente `203.0.113.0` via firewall/ACL |
| **Firewall WAF** | Implementar Web Application Firewall com proteção contra DoS |
| **SIEM Alert** | Criar regra no SIEM para alertar quando um único IP enviar >50 SYN/s |
| **Anycast / CDN** | Distribuir tráfego via CDN para absorver volumetria de ataques futuros |

---

## 📎 Evidências

| Arquivo | Descrição |
|---|---|
| [Wireshark_TCP_HTTP_log_-_TCP_log.pdf](./Wireshark_TCP_HTTP_log_-_TCP_log.pdf) | Log completo da captura de tráfego TCP/HTTP |

---

## ✅ Conclusão

A análise do tráfego capturado confirma a ocorrência de um **ataque SYN Flood** originado do IP `203.0.113.0`, direcionado ao servidor `192.0.2.1` na porta `443`. O ataque esgotou a tabela de conexões semi-abertas do servidor, tornando-o incapaz de processar requisições legítimas e resultando em erros `504 Gateway Timeout` para os usuários. A implementação imediata de **SYN Cookies** e **rate limiting** são as ações prioritárias de resposta.

---

*Relatório produzido como parte do portfólio de Cibersegurança — Davi Santos*  
*Ferramenta utilizada: Wireshark | Protocolo analisado: TCP/HTTP*

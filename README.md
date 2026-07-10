# 📡 Wireshark Traffic Analysis — SYN Flood Detection

> Análise real de tráfego TCP/HTTP com identificação de ataque de Negação de Serviço (DoS) via SYN Flood.

---

## 🎯 Objetivo

Analisar logs de captura de tráfego de rede para identificar comportamentos anômalos, classificar o tipo de ataque e propor medidas de mitigação — simulando o trabalho real de um analista de segurança.

---

## 🚨 Resultado da Análise

| Campo | Detalhe |
|---|---|
| **Tipo de Ataque** | SYN Flood — Negação de Serviço (DoS) |
| **IP Atacante** | `203.0.113.0` |
| **Alvo** | `192.0.2.1` porta `443` (HTTPS) |
| **Impacto** | Servidor indisponível — erro `504 Gateway Timeout` |
| **Duração** | ~50 segundos de flood contínuo observado |
| **Pacotes maliciosos** | 100+ SYNs sem conclusão de handshake |

---

## 🧰 Ferramentas Utilizadas

![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![TCP/IP](https://img.shields.io/badge/TCP%2FIP-Protocol-informational?style=for-the-badge)

---

## 📂 Estrutura do Repositório

```
wireshark-analysis/
├── cybersecurity-incident-report.md   ← Relatório completo do incidente
├── logs/
│   └── TCP_HTTP_log.pdf               ← Log original da captura Wireshark
└── README.md
```

---

## 📖 Resumo Técnico

### Three-Way Handshake (normal)
```
Cliente  ──[SYN]──────────────────► Servidor
Cliente  ◄─[SYN, ACK]────────────── Servidor
Cliente  ──[ACK]──────────────────► Servidor  ✅ Conexão estabelecida
```

### SYN Flood (ataque)
```
Atacante ──[SYN]──────────────────► Servidor  ⚠️  Recursos reservados
Atacante ──[SYN]──────────────────► Servidor  ⚠️  Recursos reservados
Atacante ──[SYN]──────────────────► Servidor  🔴 Backlog esgotado
         ✖ ACK nunca enviado                  ❌ Servidor sobrecarregado
```

### Linha do tempo do incidente
```
t=3.39s   → Início do SYN Flood (pacote #52)
t=7.33s   → Primeiro 504 Gateway Timeout (pacote #77)
t=7.38s   → Servidor passa a rejeitar conexões com RST, ACK
t=51.82s  → Ataque ainda em curso no fim do log
```

---

## 🛡️ Recomendações

- ✅ Habilitar **SYN Cookies** no servidor
- ✅ Implementar **rate limiting** por IP no firewall
- ✅ Bloquear IP `203.0.113.0` via ACL
- ✅ Configurar alerta no **SIEM** para >50 SYN/s por IP
- ✅ Adotar **CDN/Anycast** para distribuir volumetria

---

## 📄 Relatório Completo

👉 [cybersecurity-incident-report.md](./cybersecurity-incident-report.md)

---

*Portfólio de Cibersegurança — Davi Santos*  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/davi-santos-8b9569286)

# 🏢 NUVEXA CORPORATE NETWORK

Projeto avançado de infraestrutura de rede corporativa de alta disponibilidade, implementado no **Cisco Packet Tracer**.

Empresa fictícia **Nuvexa Tecnologia** — Matriz em São Paulo (6 andares, ~150 usuários) e Filial em Guarulhos (3 andares, ~60 usuários), interligadas por WAN, com acesso à Internet, Data Center, DMZ, Wi-Fi, VoIP, redundância, segurança e monitoramento.

---

## 📂 Estrutura da entrega

```
NUVEXA-CORPORATE-NETWORK/
│
├── Nuvexa-Corporate-Network.pkt   (arquivo do Packet Tracer)
├── README.md                      (este arquivo)
│
├── DIAGRAMS/
│   ├── physical-topology.png
│   └── logical-topology.png
│
├── DOCUMENTATION/
│   ├── ip-addressing.md            (plano de endereçamento IP)
│   ├── vlan-plan.md                (plano de VLANs)
│   ├── equipamentos.md             (tabela de equipamentos + IP de gerência)
│   ├── security.md                 (ACLs, Port Security, SSH, hardening)
│   └── troubleshooting.md          (10 testes obrigatórios + 10 falhas)
│
└── EVIDENCE/
    ├── ospf.png              hsrp.png           stp.png
    ├── etherchannel.png      acl.png            dhcp.png
    ├── nat.png               connectivity-tests.png
```

---

## 🏗️ Arquitetura hierárquica

A rede segue o modelo de 3 camadas (Core / Distribution / Access):

```
                         🌍 INTERNET (200.200.200.0/30)
                                │
                          [ RTR-BORDA ]  ── DMZ (172.16.1.0/29): SRV-WEB-DMZ
                                │  (NAT/PAT)
              ┌─────────────────┴──────────────────┐
        [ CORE-01 ] ══ EtherChannel (Po1/LACP) ══ [ CORE-02 ]   ← CORE + Distribution (L3, HSRP)
              │                                      │
     ┌────────┼──────────┬──────────┬──────────┬─────┼─────┐
  SW-AND1  SW-AND2   SW-AND3    SW-AND4   SW-AND5  SW-AND6           ← ACCESS (por andar)
   (Adm)   (Fin/RH)  (Dev/QA)   (TI)      (Dir)   (Data Center)
     │
  PCs / Telefones IP / Access Points / Impressoras

        MATRIZ (SP)  ──── RTR-WAN-SP ══ WAN (10.99.0.0/30) ══ RTR-WAN-GRU ──── FILIAL (GRU)
                                                                     │
                                                              [ SW-FIL-L3 ]
                                                              │     │     │
                                                          SW-FIL-A1 A2   A3
```

- **Core (CORE-01 / CORE-02):** switches multilayer 3560, fazem roteamento InterVLAN, HSRP (gateways virtuais) e são interligados por EtherChannel LACP. CORE-01 é o Root Bridge do STP.
- **Access (SW-AND1..6):** switches 2960, um por andar, com portas de acesso, voice VLAN e trunks redundantes para os dois Core.
- **Filial:** SW-FIL-L3 (3560) faz o roteamento local; SW-FIL-A1..3 (2960) são o acesso por andar.
- **Borda/WAN:** RTR-BORDA faz NAT/PAT e DMZ; RTR-WAN-SP e RTR-WAN-GRU formam a WAN entre as cidades.

---

## 🧰 Tecnologias implementadas

| Área | Implementação |
|------|---------------|
| Switching / VLAN | 14 VLANs, trunks 802.1Q, VLAN nativa 999, voice VLAN |
| Redundância L2 | Rapid PVST+, Root Bridge (CORE-01), Secondary (CORE-02), PortFast, BPDU Guard |
| Agregação | EtherChannel LACP (Po1) entre os Core |
| Redundância L3 | HSRP v2 (gateway virtual em cada VLAN) |
| Roteamento | InterVLAN (SVI) + OSPF Multi-Area (Area 0, 10, 20) |
| WAN | Enlace SP ↔ GRU roteado por OSPF |
| Endereçamento | IPv4 com VLSM (/24, /27, /28, /30) |
| Serviços | DHCP centralizado + DHCP Relay, DNS interno, NTP, Syslog |
| Borda | NAT/PAT (overload), DMZ |
| Segurança | ACLs (5 regras), Port Security (sticky), SSH v2, banner, senhas |
| Wireless | Wi-Fi corporativo (VLAN 100) e visitantes (VLAN 110) |
| Voz | VoIP com Data VLAN + Voice VLAN (VLAN 90) |

---

## 🔑 Credenciais de acesso (equipamentos de rede)

| Tipo | Valor |
|------|-------|
| Usuário (SSH/login local) | `admin` / `Nuvexa@2026` |
| Enable secret | `Nuvexa@Enable` |
| Console | `Nuvexa@Con` |

> Acesso remoto **somente por SSH v2** (Telnet desabilitado). Banner de acesso restrito configurado em todos os equipamentos.

---

## ✅ Checklist do desafio

- [x] Topologia hierárquica (Core / Distribution / Access)
- [x] 6 andares Matriz + 3 andares Filial
- [x] 14 VLANs (plano completo)
- [x] Endereçamento IP com VLSM e justificativa
- [x] Trunks 802.1Q (VLAN nativa 999, allowed vlan, voice vlan)
- [x] InterVLAN routing
- [x] HSRP (gateway virtual + teste de falha)
- [x] STP Rapid PVST+ (Root Bridge, PortFast, BPDU Guard)
- [x] EtherChannel LACP
- [x] OSPF Multi-Area (0, 10, 20)
- [x] WAN Matriz ↔ Filial
- [x] DHCP centralizado + DHCP Relay (ip helper-address)
- [x] DNS interno com registros
- [x] NTP
- [x] Syslog
- [x] NAT/PAT (overload)
- [x] DMZ
- [x] ACLs (5 regras)
- [x] Port Security (sticky, violation shutdown)
- [x] SSH + banner + senhas (sem Telnet)
- [x] Wi-Fi corporativo (VLAN 100) e visitantes (VLAN 110)
- [x] VoIP (Data VLAN + Voice VLAN)
- [x] 10 testes obrigatórios
- [x] Documentação e evidências

---

## 📖 Onde ler cada coisa

- Plano de IP → `DOCUMENTATION/ip-addressing.md`
- Plano de VLANs → `DOCUMENTATION/vlan-plan.md`
- Equipamentos e gerência → `DOCUMENTATION/equipamentos.md`
- Segurança (ACLs, Port Security, SSH) → `DOCUMENTATION/security.md`
- Testes e troubleshooting → `DOCUMENTATION/troubleshooting.md`
- Evidências (prints) → pasta `EVIDENCE/`

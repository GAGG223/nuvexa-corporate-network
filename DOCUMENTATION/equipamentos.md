# 🖥️ TABELA DE EQUIPAMENTOS — Nuvexa

## Matriz (São Paulo)

| Equipamento | Modelo | Função | IP de gerência / principal |
|-------------|--------|--------|----------------------------|
| CORE-01 | 3560-24PS | Core L3, roteamento InterVLAN, HSRP Active, Root Bridge STP | 10.10.70.2 (mgmt) |
| CORE-02 | 3560-24PS | Core L3, HSRP Standby, Secondary Root | 10.10.70.3 (mgmt) |
| SW-AND1 | 2960 | Access 1º andar (Administrativo) + APs + telefones | 10.10.70.11 |
| SW-AND2 | 2960 | Access 2º andar (Financeiro / RH) | 10.10.70.12 |
| SW-AND3 | 2960 | Access 3º andar (Dev / QA) | 10.10.70.13 |
| SW-AND4 | 2960 | Access 4º andar (TI / NOC) | 10.10.70.14 |
| SW-AND5 | 2960 | Access 5º andar (Diretoria) | 10.10.70.15 |
| SW-AND6 | 2960 | Access 6º andar (Data Center) | 10.10.70.16 |
| RTR-BORDA | 2911 | Roteador de borda, NAT/PAT, DMZ | 10.99.0.17 |
| RTR-WAN-SP | 2911 | Roteador WAN lado Matriz | 10.99.0.1 / 10.99.0.13 |

## Filial (Guarulhos)

| Equipamento | Modelo | Função | IP de gerência / principal |
|-------------|--------|--------|----------------------------|
| RTR-WAN-GRU | 2911 | Roteador WAN lado Filial | 10.99.0.2 / 10.99.1.1 |
| SW-FIL-L3 | 3560-24PS | Core/roteamento da Filial | 10.99.1.2 |
| SW-FIL-A1 | 2960 | Access 1º andar Filial | — |
| SW-FIL-A2 | 2960 | Access 2º andar Filial | — |
| SW-FIL-A3 | 2960 | Access 3º andar Filial | — |

## Servidores

| Servidor | IP | Serviços |
|----------|-----|----------|
| SRV-DHCP | 10.10.80.4 | DHCP centralizado + NTP |
| SRV-DNS | 10.10.80.5 | DNS interno + Syslog |
| SRV-WEB-DMZ | 172.16.1.2 | Web (HTTP) na DMZ |
| INTERNET | 200.200.200.2 | Simulação de host externo |

## Dispositivos finais (exemplos)

| Dispositivo | VLAN | Endereço |
|-------------|------|----------|
| PC-ADM-1 / PC-ADM-2 | 10 | DHCP (10.10.10.100+) |
| PC-FIN-1 / PC-FIN-2 | 20 | DHCP (10.10.20.100+) |
| PC-DEV-1 / PC-DEV-2 | 40 | DHCP (10.10.40.100+) |
| PC-TI-1 / PC-TI-2 | 60 | DHCP (10.10.60.10+) |
| PC-DIR-1 / PC-DIR-2 | 10 | DHCP |
| LAP-CORP | 100 | DHCP (Wi-Fi NUVEXA-CORP) |
| LAP-GUEST | 110 | DHCP (Wi-Fi NUVEXA-GUEST) |
| FONE-ADM-1 / FONE-ADM-2 | 90 | DHCP (voz) |
| PC-FIL-A1/A2/A3-* | 210/220/230 | DHCP / estático |

---

## Roteamento OSPF — Router IDs

| Equipamento | Router ID | Áreas |
|-------------|-----------|-------|
| CORE-01 | 1.1.1.1 | Area 10 (VLANs Matriz) + Area 0 (backbone) |
| RTR-WAN-SP | 2.2.2.2 | Area 0 |
| RTR-WAN-GRU | 3.3.3.3 | Area 0 + Area 20 |
| SW-FIL-L3 | 4.4.4.4 | Area 20 (Filial) |
| RTR-BORDA | 5.5.5.5 | Area 0 (+ default-information originate) |

---

## Wi-Fi (SSIDs)

| SSID | VLAN | Acesso |
|------|------|--------|
| NUVEXA-CORP | 100 | Funcionários — acesso completo |
| NUVEXA-GUEST | 110 | Visitantes — somente Internet |

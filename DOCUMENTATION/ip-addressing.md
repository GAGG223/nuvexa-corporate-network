# 🌐 PLANO DE ENDEREÇAMENTO IP — Nuvexa

## Estratégia geral

- **Matriz:** bloco `10.10.0.0/16` — cada VLAN recebe uma sub-rede própria.
- **Filial:** bloco `10.20.0.0/16` — redes separadas da Matriz.
- **Enlaces roteador-a-roteador:** `10.99.0.0/x` com máscara /30 (só 2 hosts).
- **DMZ:** `172.16.1.0/29`.
- **Internet simulada:** `200.200.200.0/30`.
- **Convenção:** o 3º octeto acompanha o número da VLAN (VLAN 10 → 10.10.10.0). O gateway de cada VLAN é o `.1` (IP virtual do HSRP); CORE-01 usa `.2` e CORE-02 usa `.3`.

---

## Tabela de endereçamento — MATRIZ (10.10.0.0/16)

| VLAN | Nome | Rede | Máscara | CIDR | Gateway (HSRP) | Faixa de hosts | Broadcast | Hosts |
|------|------|------|---------|------|----------------|----------------|-----------|-------|
| 10 | ADMIN | 10.10.10.0 | 255.255.255.0 | /24 | 10.10.10.1 | .2–.254 | .255 | 254 |
| 20 | FINANCE | 10.10.20.0 | 255.255.255.0 | /24 | 10.10.20.1 | .2–.254 | .255 | 254 |
| 30 | HR | 10.10.30.0 | 255.255.255.224 | /27 | 10.10.30.1 | .2–.30 | .31 | 30 |
| 40 | DEV | 10.10.40.0 | 255.255.255.0 | /24 | 10.10.40.1 | .2–.254 | .255 | 254 |
| 50 | QA | 10.10.50.0 | 255.255.255.224 | /27 | 10.10.50.1 | .2–.30 | .31 | 30 |
| 60 | IT | 10.10.60.0 | 255.255.255.224 | /27 | 10.10.60.1 | .2–.30 | .31 | 30 |
| 70 | MANAGEMENT | 10.10.70.0 | 255.255.255.224 | /27 | 10.10.70.1 | .2–.30 | .31 | 30 |
| 80 | SERVERS | 10.10.80.0 | 255.255.255.240 | /28 | 10.10.80.1 | .2–.14 | .15 | 14 |
| 90 | VOICE | 10.10.90.0 | 255.255.255.0 | /24 | 10.10.90.1 | .2–.254 | .255 | 254 |
| 100 | WIFI-CORP | 10.10.100.0 | 255.255.255.0 | /24 | 10.10.100.1 | .2–.254 | .255 | 254 |
| 110 | WIFI-GUEST | 10.10.110.0 | 255.255.255.0 | /24 | 10.10.110.1 | .2–.254 | .255 | 254 |
| 120 | CCTV | 10.10.120.0 | 255.255.255.224 | /27 | 10.10.120.1 | .2–.30 | .31 | 30 |
| 130 | IOT | 10.10.130.0 | 255.255.255.224 | /27 | 10.10.130.1 | .2–.30 | .31 | 30 |

### IPs reais dos Core (para HSRP)

| VLAN | IP Virtual (gateway) | CORE-01 | CORE-02 |
|------|----------------------|---------|---------|
| 10 | 10.10.10.1 | 10.10.10.2 | 10.10.10.3 |
| 20 | 10.10.20.1 | 10.10.20.2 | 10.10.20.3 |
| 30 | 10.10.30.1 | 10.10.30.2 | 10.10.30.3 |
| 40 | 10.10.40.1 | 10.10.40.2 | 10.10.40.3 |
| 50 | 10.10.50.1 | 10.10.50.2 | 10.10.50.3 |
| 60 | 10.10.60.1 | 10.10.60.2 | 10.10.60.3 |
| 80 | 10.10.80.1 | 10.10.80.2 | 10.10.80.3 |
| 90 | 10.10.90.1 | 10.10.90.2 | 10.10.90.3 |
| 100 | 10.10.100.1 | 10.10.100.2 | 10.10.100.3 |
| 110 | 10.10.110.1 | 10.10.110.2 | 10.10.110.3 |

> CORE-01 = HSRP priority 110 (Active) · CORE-02 = priority 100 (Standby) · preempt habilitado.

---

## Tabela de endereçamento — FILIAL (10.20.0.0/16)

| VLAN | Nome | Rede | Máscara | CIDR | Gateway | Faixa de hosts | Broadcast | Hosts |
|------|------|------|---------|------|---------|----------------|-----------|-------|
| 210 | FIL-ADMIN | 10.20.10.0 | 255.255.255.224 | /27 | 10.20.10.1 | .2–.30 | .31 | 30 |
| 220 | FIL-FINANCE | 10.20.20.0 | 255.255.255.224 | /27 | 10.20.20.1 | .2–.30 | .31 | 30 |
| 230 | FIL-OPS | 10.20.30.0 | 255.255.255.224 | /27 | 10.20.30.1 | .2–.30 | .31 | 30 |

> Gateway da filial é o IP `.1` real no SW-FIL-L3 (sem HSRP — filial não redundante).

---

## Enlaces ponto-a-ponto (/30)

| Enlace | Rede | Lado A | Lado B |
|--------|------|--------|--------|
| WAN: SP ↔ GRU | 10.99.0.0/30 | RTR-WAN-SP = 10.99.0.1 | RTR-WAN-GRU = 10.99.0.2 |
| CORE-01 ↔ RTR-WAN-SP | 10.99.0.12/30 | RTR-WAN-SP = 10.99.0.13 | CORE-01 = 10.99.0.14 |
| RTR-WAN-GRU ↔ SW-FIL-L3 | 10.99.1.0/30 | RTR-WAN-GRU = 10.99.1.1 | SW-FIL-L3 = 10.99.1.2 |
| CORE-01 ↔ RTR-BORDA | 10.99.0.16/30 | RTR-BORDA = 10.99.0.17 | CORE-01 = 10.99.0.18 |

---

## DMZ e Internet

| Rede | Uso | Endereços |
|------|-----|-----------|
| 172.16.1.0/29 | DMZ (serviços públicos) | Gateway 172.16.1.1 (RTR-BORDA) · SRV-WEB-DMZ 172.16.1.2 |
| 200.200.200.0/30 | Internet simulada | RTR-BORDA 200.200.200.1 · INTERNET 200.200.200.2 |

---

## Equipamentos com IP estático

| Equipamento | VLAN/Rede | IP |
|-------------|-----------|-----|
| SRV-DHCP (também NTP) | 80 | 10.10.80.4 |
| SRV-DNS (também Syslog) | 80 | 10.10.80.5 |
| SRV-WEB-DMZ | DMZ | 172.16.1.2 |
| INTERNET (simulado) | pública | 200.200.200.2 |
| Core / Switches (gerência) | 70 | 10.10.70.x |

> PCs, laptops e telefones IP recebem endereço via **DHCP**. Servidores, roteadores e switches usam IP fixo.

---

## Faixas DHCP configuradas (SRV-DHCP 10.10.80.4)

| Pool | Rede | Gateway | DNS | Início |
|------|------|---------|-----|--------|
| VLAN10-ADMIN | 10.10.10.0/24 | 10.10.10.1 | 10.10.80.5 | 10.10.10.100 |
| VLAN20-FINANCE | 10.10.20.0/24 | 10.10.20.1 | 10.10.80.5 | 10.10.20.100 |
| VLAN30-HR | 10.10.30.0/27 | 10.10.30.1 | 10.10.80.5 | 10.10.30.10 |
| VLAN40-DEV | 10.10.40.0/24 | 10.10.40.1 | 10.10.80.5 | 10.10.40.100 |
| VLAN50-QA | 10.10.50.0/27 | 10.10.50.1 | 10.10.80.5 | 10.10.50.10 |
| VLAN60-IT | 10.10.60.0/27 | 10.10.60.1 | 10.10.80.5 | 10.10.60.10 |
| VLAN90-VOICE | 10.10.90.0/24 | 10.10.90.1 | 10.10.80.5 | 10.10.90.100 |
| VLAN100-WIFI | 10.10.100.0/24 | 10.10.100.1 | 10.10.80.5 | 10.10.100.100 |
| VLAN110-GUEST | 10.10.110.0/24 | 10.10.110.1 | 10.10.80.5 | 10.10.110.100 |

> As VLANs recebem o pedido DHCP via **DHCP Relay** (`ip helper-address 10.10.80.4`) configurado nas SVIs dos Core.

---

## Justificativa do subnetting (VLSM)

O projeto usa máscaras de tamanhos diferentes para dimensionar cada rede conforme a necessidade, evitando desperdício e melhorando o controle de segurança:

- **/24 (254 hosts):** VLANs grandes ou de alto crescimento — ADMIN, FINANCE, DEV, VOICE, WIFI-CORP, WIFI-GUEST. Wi-Fi e voz têm muitos dispositivos e demanda variável, então recebem folga ampla.
- **/27 (30 hosts):** departamentos médios — HR, QA, IT, MANAGEMENT, CCTV, IOT e as VLANs da filial. Cobrem ~15–30 dispositivos com folga saudável.
- **/28 (14 hosts):** SERVERS. Poucos servidores; sub-rede pequena facilita o controle por ACL e a segurança do Data Center.
- **/30 (2 hosts):** todos os enlaces roteador-a-roteador (WAN e ponto-a-ponto). Como só há 2 pontas, /30 é ideal e não desperdiça endereços.

Como as bases são /16, sobra amplo espaço para expansão futura (basta alocar novas sub-redes nos blocos ainda não usados, ex.: 10.10.140.0, 10.10.150.0...).

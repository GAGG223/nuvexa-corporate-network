# 🔐 SEGURANÇA — Nuvexa

Documentação das políticas de segurança: ACLs, Port Security e hardening dos equipamentos.

---

## 1. ACLs (Access Control Lists)

As ACLs são estendidas e nomeadas, aplicadas nas SVIs dos **dois Core** (por causa do HSRP, o tráfego pode passar por qualquer um). Ordem de leitura: de cima para baixo, para na primeira que combina.

### Regra 1 — Wi-Fi Visitantes (VLAN 110) não acessa redes internas

Visitante navega só na Internet; bloqueado de Matriz, Filial e DMZ. Libera antes o gateway, DHCP e DNS para o serviço básico funcionar.

```cisco
ip access-list extended GUEST-RESTRITO
 permit ip 10.10.110.0 0.0.0.255 host 10.10.110.1
 permit udp 10.10.110.0 0.0.0.255 host 10.10.80.4 eq 67
 permit udp 10.10.110.0 0.0.0.255 host 10.10.80.5 eq 53
 deny   ip 10.10.110.0 0.0.0.255 10.10.0.0 0.0.255.255
 deny   ip 10.10.110.0 0.0.0.255 10.20.0.0 0.0.255.255
 deny   ip 10.10.110.0 0.0.0.255 172.16.1.0 0.0.0.7
 permit ip 10.10.110.0 0.0.0.255 any
!
interface vlan 110
 ip access-group GUEST-RESTRITO in
```
**Resultado testado:** Guest → Internet = OK · Guest → servidor interno (10.10.80.5) = bloqueado (Destination host unreachable). Corresponde ao **Teste 4**.

### Regra 2 — Gerenciamento (VLAN 70) só pela equipe de TI (VLAN 60)

```cisco
ip access-list extended PROTEGE-MGMT
 permit ip 10.10.60.0 0.0.0.31 10.10.70.0 0.0.0.31
 deny   ip any 10.10.70.0 0.0.0.31
 permit ip any any
!
interface vlan 70
 ip access-group PROTEGE-MGMT out
```
**Efeito:** somente a rede de TI (10.10.60.0/27) alcança a rede de gerência (10.10.70.0/27); qualquer outra origem é bloqueada.

### Regra 3 — Financeiro (VLAN 20) não acessa Desenvolvimento (VLAN 40)

```cisco
ip access-list extended FIN-BLOQUEIA-DEV
 deny   ip 10.10.20.0 0.0.0.255 10.10.40.0 0.0.0.255
 permit ip any any
!
interface vlan 20
 ip access-group FIN-BLOQUEIA-DEV in
```
**Resultado testado:** Financeiro → Dev (10.10.40.x) = bloqueado · Financeiro → outras redes = OK. Corresponde ao **Teste 3**.

### Regras 4 e 5 — Acesso controlado aos Servidores (VLAN 80)

Usuários corporativos acessam os serviços autorizados (Web, DNS, DHCP, ICMP); Guest é bloqueado explicitamente; demais tráfego controlado.

```cisco
ip access-list extended CONTROLA-SERVERS
 permit tcp 10.10.0.0 0.0.255.255 10.10.80.0 0.0.0.15 eq 80
 permit tcp 10.10.0.0 0.0.255.255 10.10.80.0 0.0.0.15 eq 53
 permit udp 10.10.0.0 0.0.255.255 10.10.80.0 0.0.0.15 eq 53
 permit udp 10.10.0.0 0.0.255.255 10.10.80.0 0.0.0.15 eq 67
 permit icmp 10.10.0.0 0.0.255.255 10.10.80.0 0.0.0.15
 deny   ip 10.10.110.0 0.0.0.255 10.10.80.0 0.0.0.15
 permit ip any any
!
interface vlan 80
 ip access-group CONTROLA-SERVERS out
```
**Efeito:** usuários corporativos usam os serviços autorizados; a rede de servidores tem acesso controlado; visitantes não alcançam o Data Center.

### Resumo das políticas

| Regra | Origem | Destino | Ação |
|-------|--------|---------|------|
| 1 | WIFI-GUEST (110) | Redes internas / DMZ | Negar (só Internet) |
| 2 | Qualquer ≠ TI | MANAGEMENT (70) | Negar |
| 3 | FINANCE (20) | DEV (40) | Negar |
| 4 | Corporativo | SERVERS (80) — serviços autorizados | Permitir |
| 5 | Guest / não autorizado | SERVERS (80) | Negar |

---

## 2. Port Security

Aplicado nas portas de acesso (portas de PC/servidor). Aprende o MAC automaticamente (sticky) e desliga a porta em caso de violação.

```cisco
interface range fastEthernet0/1 - 2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2          ! 1 nos switches de servidores
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

- **maximum 2:** permite PC + telefone IP na mesma porta (Data + Voice). Nos switches de servidores usa-se `maximum 1`.
- **sticky:** o MAC do primeiro dispositivo é aprendido e gravado.
- **violation shutdown:** se um dispositivo não autorizado conectar, a porta entra em err-disabled.

**Verificação:**
```cisco
show port-security
show port-security interface fastEthernet0/1
```
**Recuperar porta após violação:**
```cisco
interface fastEthernet0/1
 shutdown
 no shutdown
```

---

## 3. Hardening dos equipamentos (SSH, senhas, banner)

Aplicado em todos os switches e roteadores.

```cisco
hostname <NOME>
ip domain-name nuvexa.local
username admin privilege 15 secret Nuvexa@2026
enable secret Nuvexa@Enable
crypto key generate rsa       ! módulo 1024 bits
ip ssh version 2
!
line vty 0 4
 transport input ssh          ! somente SSH (Telnet desabilitado)
 login local
line console 0
 password Nuvexa@Con
 login
!
service password-encryption
banner motd #
****************************************************
  ACESSO RESTRITO - NUVEXA TECNOLOGIA
  Somente pessoal autorizado.
  Atividades sao monitoradas e registradas.
****************************************************
#
```

### Medidas de segurança adicionais

| Medida | Implementação |
|--------|---------------|
| Acesso remoto seguro | SSH v2 (Telnet desabilitado nas VTY) |
| Autenticação | Usuário local `admin` + enable secret |
| Senhas cifradas | `service password-encryption` |
| Banner legal | MOTD de acesso restrito |
| VLAN nativa segura | Native VLAN 999 (blackhole) nos trunks |
| Portas não usadas | Jogadas na VLAN 999 e desligadas (shutdown) |
| Proteção STP | PortFast + BPDU Guard nas portas de acesso |
| Segmentação | 14 VLANs + ACLs entre segmentos |
| DMZ | Serviços públicos isolados da rede interna |

### Credenciais

| Tipo | Valor |
|------|-------|
| Usuário (SSH) | `admin` / `Nuvexa@2026` |
| Enable | `Nuvexa@Enable` |
| Console | `Nuvexa@Con` |

**Teste do SSH (de um PC):**
```
ssh -l admin 10.10.10.2
```
Solicita a senha, exibe o banner e autentica — confirmando acesso remoto seguro.

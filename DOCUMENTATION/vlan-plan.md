# 🧩 PLANO DE VLANs — Nuvexa

## Tabela de VLANs

| VLAN | Nome | Departamento / Função | Localização |
|------|------|-----------------------|-------------|
| 10 | ADMIN | Administrativo | Matriz 1º / 5º andar |
| 20 | FINANCE | Financeiro | Matriz 2º andar |
| 30 | HR | Recursos Humanos | Matriz 2º andar |
| 40 | DEV | Desenvolvimento | Matriz 3º andar |
| 50 | QA | Qualidade | Matriz 3º andar |
| 60 | IT | Tecnologia da Informação | Matriz 4º andar |
| 70 | MANAGEMENT | Gerenciamento dos equipamentos | Todos |
| 80 | SERVERS | Servidores | Data Center (6º andar) |
| 90 | VOICE | Telefonia IP (VoIP) | Todos os andares |
| 100 | WIFI-CORP | Wi-Fi corporativo | Todos os andares |
| 110 | WIFI-GUEST | Wi-Fi visitantes | Todos os andares |
| 120 | CCTV | Câmeras de segurança | Todos os andares |
| 130 | IOT | Dispositivos IoT | Todos os andares |
| 999 | BLACKHOLE | Portas não utilizadas | Todos os switches |

## VLANs da Filial (redes separadas)

| VLAN | Nome | Função |
|------|------|--------|
| 210 | FIL-ADMIN | Atendimento / Administrativo (1º andar) |
| 220 | FIL-FINANCE | Financeiro / Operações (2º andar) |
| 230 | FIL-OPS | TI / Servidores (3º andar) |

---

## Configuração das VLANs (aplicada nos switches)

```cisco
vlan 10
 name ADMIN
vlan 20
 name FINANCE
vlan 30
 name HR
vlan 40
 name DEV
vlan 50
 name QA
vlan 60
 name IT
vlan 70
 name MANAGEMENT
vlan 80
 name SERVERS
vlan 90
 name VOICE
vlan 100
 name WIFI-CORP
vlan 110
 name WIFI-GUEST
vlan 120
 name CCTV
vlan 130
 name IOT
vlan 999
 name BLACKHOLE
```

> Cada switch recebe apenas as VLANs necessárias ao seu andar + as comuns (voz, Wi-Fi, blackhole).

---

## Trunks 802.1Q

Todos os enlaces entre switches usam trunk 802.1Q com:
- **VLAN nativa:** 999 (BLACKHOLE) — proteção contra VLAN hopping
- **Allowed VLANs:** apenas as VLANs necessárias em cada enlace
- **Encapsulation:** dot1q (nos 3560)

Exemplo (trunk de um switch de acesso para o Core):
```cisco
interface gigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,90,100,110,999
```

---

## Portas de acesso (Data VLAN + Voice VLAN)

Portas de usuário carregam a VLAN de dados e a VLAN de voz simultaneamente:
```cisco
interface fastEthernet0/1
 switchport mode access
 switchport access vlan 10       ! dados (PC)
 switchport voice vlan 90        ! voz (telefone IP)
 spanning-tree portfast
 spanning-tree bpduguard enable
```

---

## Portas não utilizadas (segurança)

Todas as portas livres são jogadas na VLAN 999 e desligadas:
```cisco
interface range fastEthernet0/3 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
```

---

## Verificação

```cisco
show vlan brief
show interfaces trunk
```

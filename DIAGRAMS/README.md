# 🗺️ DIAGRAMAS — Nuvexa

Coloque nesta pasta os dois diagramas exigidos:

- `physical-topology.png` — diagrama físico
- `logical-topology.png` — diagrama lógico

---

## Como exportar o diagrama físico do Packet Tracer

1. Abra o arquivo `Nuvexa-Corporate-Network.pkt`.
2. Organize os equipamentos de forma clara (Matriz à esquerda, WAN no meio, Filial à direita).
3. Menu **File → Print** (ou tecle a captura de tela) e salve/exporte a imagem da topologia.
4. Salve como `physical-topology.png` nesta pasta.

O diagrama físico deve representar:
- Andares (SW-AND1 a 6 / SW-FIL-A1 a 3)
- Switches, roteadores e servidores
- Links (cabos) e dispositivos finais (PCs, telefones, APs, impressoras)

---

## Diagrama lógico (o que representar)

Crie um diagrama (pode ser no próprio PT, no draw.io, ou similar) mostrando a camada lógica:

- **VLANs e sub-redes** (ver `../DOCUMENTATION/ip-addressing.md`)
- **Gateways HSRP** (IP virtual .1 de cada VLAN)
- **Áreas OSPF:** Area 0 (backbone/WAN), Area 10 (Matriz), Area 20 (Filial)
- **WAN** entre RTR-WAN-SP (10.99.0.1) e RTR-WAN-GRU (10.99.0.2)
- **DMZ** (172.16.1.0/29) e saída para Internet (NAT/PAT)

Salve como `logical-topology.png` nesta pasta.

---

## Referência rápida da topologia lógica

```
                     INTERNET 200.200.200.0/30
                            │  NAT/PAT
                       [ RTR-BORDA ] ── DMZ 172.16.1.0/29
                            │  Area 0
        Area 10        [ CORE-01 ]═Po1═[ CORE-02 ]        Area 10
      (VLANs Matriz)        │  HSRP (gateway .1)  │
                    ┌───────┴────────┐            │
                 SW-AND1..6 (acesso por andar)    │
                            │
                     [ RTR-WAN-SP ] 10.99.0.1
                            ║ Area 0 (WAN 10.99.0.0/30)
                     [ RTR-WAN-GRU ] 10.99.0.2
                            │ Area 20
                     [ SW-FIL-L3 ] ── SW-FIL-A1..3
                       (VLANs 210/220/230)
```

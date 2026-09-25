# 🧪 TESTES E TROUBLESHOOTING — Nuvexa

## PARTE 1 — Os 10 testes obrigatórios

| # | Teste | Comando / Ação | Resultado esperado | Status |
|---|-------|----------------|--------------------|--------|
| 1 | PC Administrativo → Gateway | `ping 10.10.10.1` do PC-ADM-1 | Sucesso (Reply) | ✅ |
| 2 | PC Financeiro → Servidor autorizado | `ping 10.10.80.5` do PC-FIN-1 | Sucesso | ✅ |
| 3 | PC Financeiro → Desenvolvimento | `ping 10.10.40.100` do PC-FIN-1 | Bloqueado (ACL) | ✅ |
| 4 | Guest → Servidor interno | `ping 10.10.80.5` do LAP-GUEST | Bloqueado (ACL) | ✅ |
| 5 | PC Matriz → PC Filial | `ping 10.20.10.10` do PC-ADM-1 | Sucesso (WAN/OSPF) | ✅ |
| 6 | PC → Internet | `ping 200.200.200.2` do PC-ADM-1 | Sucesso (NAT/PAT) | ✅ |
| 7 | Desligar CORE principal | shutdown na Vlan do CORE-01 | HSRP mantém gateway (CORE-02 assume) | ✅ |
| 8 | Desligar link do EtherChannel | shutdown Fa0/11 (uma perna do Po1) | Comunicação segue pelo link restante | ✅ |
| 9 | Desligar link redundante L2 | shutdown de um trunk switch↔Core | STP reconverge sem loop | ✅ |
| 10 | Verificar vizinhança OSPF | `show ip ospf neighbor` | Vizinhos em FULL | ✅ |

### Detalhamento dos principais testes

**Teste 7 — Redundância HSRP**
1. No PC-ADM-1: `ping -t 10.10.10.1` (contínuo).
2. No CORE-01: `interface vlan 10` → `shutdown`.
3. Observado: 1–3 timeouts e os Reply voltam (CORE-02 assume como Active).
4. `show standby brief` no CORE-02 mostra State = Active.
5. Religar CORE-01 (`no shutdown`) → volta a Active por preempt.
- Evidência: `EVIDENCE/hsrp.png`

**Teste 8 — EtherChannel**
1. `show etherchannel summary` → Po1 (SU) com Fa0/11(P) e Fa0/12(P), protocolo LACP.
2. `interface fastEthernet0/11` → `shutdown` (derruba uma perna).
3. Comunicação entre os Core continua pela Fa0/12.
- Evidência: `EVIDENCE/etherchannel.png`

**Teste 10 — OSPF**
- `show ip ospf neighbor` nos roteadores/Core mostra os vizinhos em FULL (1.1.1.1, 2.2.2.2, 3.3.3.3, 4.4.4.4, 5.5.5.5).
- `show ip route ospf` mostra as redes da Filial como `O IA` e a rota default como `O*E2`.
- Evidência: `EVIDENCE/ospf.png`

### Comandos de verificação úteis

```cisco
show ip interface brief
show vlan brief
show interfaces trunk
show standby brief
show spanning-tree vlan 10
show etherchannel summary
show ip ospf neighbor
show ip route
show ip nat translations
show port-security
```

---

## PARTE 2 — Desafio de Troubleshooting (10 falhas)

Modelo para documentar cada falha proposital introduzida na rede. Preencha durante a demonstração.

### Falha 1 — VLAN removida do trunk
- **Problema:** VLAN 10 removida do `allowed vlan` no trunk SW-AND1↔CORE-01.
- **Sintoma:** PCs da VLAN 10 perdem o gateway (não pingam 10.10.10.1).
- **Comandos:** `show interfaces trunk`, `show vlan brief`.
- **Diagnóstico:** VLAN 10 ausente na lista de VLANs permitidas do trunk.
- **Causa:** filtro de VLAN incorreto no trunk.
- **Correção:** `switchport trunk allowed vlan add 10`.
- **Teste:** `ping 10.10.10.1` volta a responder.
- **Resultado:** Resolvido.

### Falha 2 — Gateway incorreto
- **Problema:** PC configurado com gateway errado (ex.: 10.10.10.254).
- **Sintoma:** acessa a rede local, mas não sai para outras redes/Internet.
- **Comandos:** `ipconfig` no PC, `ping` no gateway.
- **Diagnóstico:** gateway não corresponde ao IP virtual HSRP (.1).
- **Causa:** erro de configuração no host.
- **Correção:** ajustar gateway para 10.10.10.1 (ou usar DHCP).
- **Teste:** ping para outra rede funciona.
- **Resultado:** Resolvido.

### Falha 3 — ACL incorreta
- **Problema:** ACL sem o `permit` do gateway/DNS antes dos `deny`.
- **Sintoma:** VLAN inteira perde acesso, inclusive ao próprio gateway.
- **Comandos:** `show access-lists`, `show ip interface vlan X`.
- **Diagnóstico:** ordem das entradas bloqueando tráfego essencial.
- **Causa:** ACL muito restritiva / ordem incorreta.
- **Correção:** reescrever a ACL com permits específicos antes dos deny.
- **Teste:** serviços essenciais voltam; bloqueio-alvo mantido.
- **Resultado:** Resolvido.

### Falha 4 — OSPF configurado incorretamente
- **Problema:** rede anunciada na área errada (ex.: Filial em area 10).
- **Sintoma:** vizinhança não forma ou rotas não aparecem.
- **Comandos:** `show ip ospf neighbor`, `show ip route ospf`.
- **Diagnóstico:** área inconsistente entre vizinhos.
- **Causa:** número de área errado no `network`.
- **Correção:** corrigir a área no processo OSPF.
- **Teste:** vizinhos em FULL e rotas aprendidas.
- **Resultado:** Resolvido.

### Falha 5 — Interface desligada
- **Problema:** interface (uplink/SVI) em `shutdown`.
- **Sintoma:** segmento inteiro sem comunicação.
- **Comandos:** `show ip interface brief` (administratively down).
- **Diagnóstico:** interface administrativamente desligada.
- **Causa:** shutdown indevido.
- **Correção:** `no shutdown`.
- **Teste:** interface up/up, tráfego restabelecido.
- **Resultado:** Resolvido.

### Falha 6 — DHCP Relay incorreto
- **Problema:** `ip helper-address` ausente ou apontando para IP errado.
- **Sintoma:** PCs recebem APIPA (169.254.x.x), DHCP failed.
- **Comandos:** `show run interface vlan X` (ou observar o host).
- **Diagnóstico:** pedido DHCP não chega ao servidor.
- **Causa:** helper-address ausente/errado na SVI.
- **Correção:** `ip helper-address 10.10.80.4` na SVI.
- **Teste:** PC recebe IP correto via DHCP.
- **Resultado:** Resolvido.

### Falha 7 — EtherChannel inconsistente
- **Problema:** portas do canal com modos/VLANs diferentes entre os lados.
- **Sintoma:** canal não sobe (portas suspensas), link instável.
- **Comandos:** `show etherchannel summary`.
- **Diagnóstico:** parâmetros inconsistentes entre as pontas.
- **Causa:** configuração divergente das portas-membro.
- **Correção:** igualar modo (LACP active) e config de trunk nos dois lados.
- **Teste:** Po1 em (SU) com portas (P).
- **Resultado:** Resolvido.

### Falha 8 — VLAN errada em uma porta de acesso
- **Problema:** porta do PC na VLAN errada (ex.: 999 em vez de 10).
- **Sintoma:** PC não pega DHCP / não alcança o gateway certo.
- **Comandos:** `show vlan brief`.
- **Diagnóstico:** porta associada à VLAN incorreta.
- **Causa:** access vlan errada.
- **Correção:** `switchport access vlan 10`.
- **Teste:** PC volta a operar na VLAN correta.
- **Resultado:** Resolvido.

### Falha 9 — Rota ausente
- **Problema:** rede não aprendida/anunciada no OSPF.
- **Sintoma:** destino inalcançável entre segmentos.
- **Comandos:** `show ip route`, `show ip ospf neighbor`.
- **Diagnóstico:** rota faltando na tabela.
- **Causa:** `network` não configurado para a sub-rede.
- **Correção:** adicionar o `network` correto na área certa.
- **Teste:** rota aparece e ping funciona.
- **Resultado:** Resolvido.

### Falha 10 — NAT configurado incorretamente
- **Problema:** interface interna/externa marcada errado, ou ACL do NAT sem a rede.
- **Sintoma:** hosts internos não acessam a Internet.
- **Comandos:** `show ip nat translations`, `show run | include nat`.
- **Diagnóstico:** ausência de traduções ao gerar tráfego.
- **Causa:** `ip nat inside/outside` incorreto ou ACL do NAT incompleta.
- **Correção:** marcar inside/outside corretos e incluir a rede na ACL do NAT.
- **Teste:** `show ip nat translations` mostra a tradução; ping à Internet OK.
- **Resultado:** Resolvido.

---

## Observações reais do projeto (lições aprendidas)

Durante a construção, os problemas mais relevantes enfrentados e resolvidos foram:

1. **HSRP travado (Speak/Listen):** causado por `mac-address` fixo herdado nas SVIs do 3560. Solução: `no mac-address` na interface Vlan.
2. **Loop de camada 2:** dois enlaces entre equipamentos sem agregação corretos causavam instabilidade no HSRP. Solução: definir Root Bridge (Rapid PVST+) e usar EtherChannel LACP entre os Core.
3. **EtherChannel em portas erradas:** agrupar portas que iam para switches diferentes derrubou os trunks. Solução: confirmar que as portas-membro ligam o MESMO par de equipamentos antes de criar o canal.
4. **Config aplicada no equipamento errado:** SVIs de Core criadas por engano num switch de acesso geraram IP duplicado. Solução: remover as interfaces indevidas.
5. **DHCP em VLAN sem gateway:** SVI/helper-address ausente fazia o host cair em APIPA. Solução: criar a SVI com HSRP + `ip helper-address`.

> Dica de diagnóstico validada no projeto: **teste sempre a partir de um PC/host** (que tem IP completo), não a partir de switches de acesso L2, que dão falso negativo no ping originado localmente.

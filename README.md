# Inter-AS MPLS L3VPN — Option B

## 1. Objetivo

Laboratório de extensão de serviços L3VPN entre dois Sistemas Autónomos (AS) distintos, utilizando a **Option B** (RFC 4364) para interconexão de VPNv4 entre ASBRs via MP-eBGP.

## 2. Topologia

- 2x AS distintos (AS 100 e AS 200)
- 2x PE por AS, com VRFs de cliente configuradas
- 2x ASBR (um por AS), interligados via eBGP direto — sem VRF na interconexão
- MPLS LDP habilitado no core de cada AS
- CE simulando cliente final em cada extremidade

## 3. Option A vs Option B

| | Option A | Option B |
|---|---|---|
| Interconexão ASBR-ASBR | VRF + subinterface por VPN | Sessão MP-eBGP única (VPNv4) |
| Escalabilidade | Baixa — 1 VRF por cliente na borda | Alta — troca de rotas via BGP |
| Rótulos MPLS | Trocados apenas localmente por VRF | Label VPN preservado end-to-end |
| Complexidade operacional | Simples, mas não escala | Maior no setup, escalável no crescimento |

## 4. Configuração — pontos-chave

**ASBR — interconexão entre AS:**
```
router bgp 100
 neighbor <ip-asbr-remoto> remote-as 200
 address-family vpnv4
  neighbor <ip-asbr-remoto> activate
  neighbor <ip-asbr-remoto> next-hop-self
```

**PE — VRF do cliente:**
```
vrf definition CLIENTE_A
 rd 100:1
 route-target export 100:1
 route-target import 100:1
!
address-family ipv4 vrf CLIENTE_A
 redistribute connected
```

> Configurações completas de cada dispositivo estão nos ficheiros individuais desta pasta.

## 5. Verificação

**Tabela BGP VPNv4 no ASBR** — confirma rotas recebidas de ambos os AS:
```
show bgp vpnv4 unicast all summary
show bgp vpnv4 unicast rd 100:1
```

**Conectividade end-to-end entre clientes em AS distintos:**
```
traceroute vrf CLIENTE_A <ip-destino>
```
Confirma encaminhamento via backbone MPLS com troca de rótulos correta entre domínios.

## 6. Principais aprendizagens

- Diferença prática entre redistribuição de rótulos por VRF (Option A) e propagação direta de rotas VPNv4 rotuladas via BGP (Option B)
- Impacto do `next-hop-self` na sessão ASBR-ASBR na preservação do label binding
- Desafios de escalabilidade ao estender serviços L3VPN entre domínios administrativos diferentes — relevante para ambientes multi-AS de Service Provider

---

**Stack:** IOS/IOS-XE · MPLS · MP-BGP · VRF · Inter-AS VPN

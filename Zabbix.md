# 📚 Manual Técnico Operacional: Dashboard de Backbone (Zabbix)

**Analista Responsável:** Fernando Jacobsen Rodrigues / Analista de Sistemas
**Data da Elaboração:** 18/11/2025
**Objetivo:** Fornecer diretrizes operacionais para a interpretação, diagnóstico e escalonamento dos alertas críticos da rede Core, OLTs e Borda (BGP/NE40).

---

## 1. Visão Geral do Painel

<div style="text-align: center;">
    <img src="/img/Zabbix/Painel.png" alt="Tela de Provisionamento" width="100%">

</div>
<div style="text-align: center;">
    <img src="/img/Zabbix/Painel_2.png" alt="Tela de Provisionamento" width="100%">

</div>

**O dashboard "Backbone" é o principal painel de _status_ da infraestrutura, agregando dados de falhas por criticidade (Lista de Problemas) e status visual (Mapa e Honeycombs).**

| Área                                | Widgets Envolvidos                               | Função Primária                                                    |
| :---------------------------------- | :----------------------------------------------- | :----------------------------------------------------------------- |
| **I. Monitoramento de Incidentes**  | OLT, Clientes e Backbone                         | Identificar o equipamento e o serviço afetado (Texto).             |
| **II. Topologia e Causa Raiz**      | Mapa de Backbone                                 | Localizar geograficamente e logicamente o ponto de falha (Visual). |
| **III. Estado Crítico de Serviços** | Comunicação Adyl, Monitoramento de Luz, PEER BGP | Visualização binária (UP/DOWN) de serviços essenciais.             |
| **IV. Performance e Capacidade**    | Gráficos NE40 BGP                                | Avaliar volumetria de tráfego e saturação.                         |

---

## 2. Seção I: Monitoramento de Incidentes (Lista de Problemas)

### 2.1. OLT (Optical Line Terminal)

<div style="text-align: center;">
    <img src="/img/Zabbix/OltPanel.png" alt="Tela de Provisionamento" width="100%">

</div>

Este widget lista problemas exclusivos da infraestrutura GPON/OLT.

| Severidade             | Alerta Típico                                   | Ação Primária                                                                                                                         |
| :--------------------- | :---------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------ |
| **Crítica (Vermelho)** | Status da VLAN alterado / Placa PON Inoperante. | **Prioridade:** Validar configuração da VLAN e o estado do hardware (placa/SFP) da OLT. Investigar alertas > 24h (falha persistente). |

### 2.2. Clientes e Backbone

<div style="text-align: center;">
    <img src="/img/Zabbix/DedBack.png" alt="Tela de Provisionamento" width="100%">

</div>
Agrega falhas de transporte (Trunk, EoIP) e degradação de serviços de clientes dedicados.

| Status                | Definição                                                  | Ação Primária                                                                                                                                      |
| :-------------------- | :--------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Warning (Amarelo)** | Alto uso de banda (> 90%), _High ICMP ping response time_. | **Performance:** Abrir chamado N2 para ajuste de QoS, _load balancing_ ou investigação de interferência/saturação de enlace.                       |
| **High (Laranja)**    | _Link Down_ (em _Trunk_ ou túneis EoIP/VLANs).             | **Falha:** Tratar como rompimento ou falha de protocolo. Verificar imediatamente as interfaces físicas no host afetado (**Ex: Trunk BR Digital**). |

---

## 3. Seção II: Topologia e Causa Raiz (Mapa de Backbone)

<div style="text-align: center;">
    <img src="/img/Zabbix/Map.png" alt="Tela de Provisionamento" width="100%">

</div>
O mapa fornece a visão lógica da rede. É a principal ferramenta para _Root Cause Analysis_ (RCA).

- **Navegabilidade (Drill-down):** Clique em qualquer nó (ícone azul, ex: "Matriz") com status de incidente (halo vermelho) para acessar o **Submapa** detalhado ou a lista de problemas do equipamento.
- **Status do Nó:** O nó **"Matriz"** na imagem, com 1 Incidente (Vermelho), deve ser o ponto inicial de qualquer investigação que envolva BGP ou links de longa distância.
- **Status de Links (Linhas):**
  - **Verde:** Link UP, saúde normal.
  - **Vermelho (Link Triggers):** Confirmação de que o _Link Down_ foi detectado via triggers e está sendo visualmente alertado no mapa.

| Evento             | Ação no Mapa                                                                                          |
| :----------------- | :---------------------------------------------------------------------------------------------------- |
| **Nó Vermelho**    | Clicar no nó. Correlacionar com a seção 2.2 para saber qual link caiu.                                |
| **Linha Vermelha** | Indicação de que o circuito entre as duas localidades está rompido ou com falha crítica de protocolo. |

---

## 4. Seção III: Estado Crítico de Serviços (Honeycombs)

Visualização de status binário (ativo/inativo).

### 4.1. Monitoramento de Luz

<div style="text-align: center;">
    <img src="/img/Zabbix/MonitorLuz.png" alt="Tela de Provisionamento" width="100%">

</div>
Monitora a presença de energia elétrica da concessionária nos PoPs. Ao passar o maouse em cima o combo expande para melhor vizualização.

- **Status:** **Inativo (0) / Vermelho.**
  - **Significado:** Falha na energia elétrica (AC). O PoP está operando com banco de baterias ou gerador.
  - **Prioridade:** Alta. Enviar alerta para o time de Infra e verificar a autonomia do banco de baterias.

### 4.2. PEER BGP

<div style="text-align: center;">
    <img src="/img/Zabbix/Bgp.png" alt="Tela de Provisionamento" width="100%">

</div>
Monitora a sessão de roteamento BGP.

- **Status:** **Vermelho.**
  - **Significado:** Sessão BGP **DOWN** com o _peer_ (ex: 177.5... ou 172.2...). O tráfego não está sendo roteado por este caminho.
  - **Prioridade:** Crítica. Verificar reachability e configuração do roteador de borda (NE40).

### 4.3. Comunicação Adyl

<div style="text-align: center;">
    <img src="/img/Zabbix/Uplink.png" alt="Tela de Provisionamento" width="100%">

</div>
Monitora o status das interfaces físicas do _Uplink_ primário.

- **Status:** **UP (1) / Verde.**
  - **Significado:** Interfaces físicas (ex: 100GE0/3/0) estão ativas e negociadas.

---

## 5. Seção IV: Performance e Capacidade (Gráficos NE40 BGP)

<div style="text-align: center;">
    <img src="/img/Zabbix/UpGraf.png" alt="Tela de Provisionamento" width="100%">

</div>

Monitoramento em tempo real do tráfego nas interfaces de Core (roteadores NE40). São duas interfaces que ligam a empresa com a internet.

| Padrão                           | Cenário                                                       | Ação Primária                                                                                    |
| :------------------------------- | :------------------------------------------------------------ | :----------------------------------------------------------------------------------------------- |
| **Pico (20 Gbps)**               | Padrão diurno normal (horário nobre).                         | Nenhuma. Utilizar para planejamento de capacidade (_capacity planning_).                         |
| **Platô (Flatline)**             | Tráfego travado no limite máximo da interface (ex: 100 Gbps). | **Gargalo:** Acionar NOC para ativar novos links ou desviar tráfego (desbalanceamento).          |
| **Queda Abrupta (Drop to Zero)** | Gráfico cai instantaneamente.                                 | **Interrupção:** Correlacionar com o _Link Down_ no Mapa e acionar o fornecedor/equipe de fibra. |

---

## 6. Fluxo de Análise e Escalonamento (SOP)

Ao receber um alerta no dashboard, siga a ordem: **MAPA $\rightarrow$ PROBLEMAS $\rightarrow$ GRÁFICOS.**

1.  **Localização (Mapa):** Identifique o nó principal afetado (ex: "Matriz" vermelho).
2.  **Diagnóstico (Problemas):** Clique no nó afetado para ver a lista de problemas e identificar a causa raiz (ex: BGP Down causado por falta de LUZ, ou Link EoIP Down).
3.  **Impacto (Gráficos):** Verifique o gráfico do NE40 para confirmar se a falha está impactando a vazão de tráfego.

| Severidade   | Escopo                                       | Tempo de Resposta (SLA) | Escalonamento                                |
| :----------- | :------------------------------------------- | :---------------------- | :------------------------------------------- |
| **Desastre** | Falha de LUZ ou Múltiplos Peers BGP Caídos.  | Imediato (< 15 min)     | N2 $\rightarrow$ N3/NOC/Engenharia           |
| **Alta**     | Link Down (Trunk / EoIP) ou VLAN Inoperante. | < 1 hora                | N1 $\rightarrow$ N2/Infraestrutura           |
| **Média**    | Latência Alta / Alto Uso de Banda.           | < 4 horas               | N1 $\rightarrow$ N2 (Análise de Performance) |

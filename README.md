# Prática de Simulação de Ambiente Hierárquico — Configurações de VLANs Tipo Trunk (Atividade 07)

**Instituição:** Instituto Federal de Educação, Ciência e Tecnologia de Rondônia (IFRO) — *Campus* Porto Velho Zona Norte  
**Curso:** Superior de Tecnologia em Redes de Computadores  
**Disciplina:** Comutação de Redes Locais (TCN.0639)  
**Docente:** Prof. Jhordano Malacarne Bravim  
**Autor(a):** Mariane Gonçalves Dantas 
**Ferramenta:** Cisco Packet Tracer

## 1. Objetivo da Atividade
Configurar o entroncamento (`switchport mode trunk`) nas portas que agrupam tráfego e interligam os switches nas camadas de **Acesso**, **Distribuição** e **Núcleo**, garantindo a passagem das **VLANs 10 (`Alunos`)** e **20 (`Visitantes`)** por toda a topologia.

## 2. Diagrama da Topologia

<img width="847" height="626" alt="Print Packet Tracer" src="https://github.com/user-attachments/assets/d16ba96f-f1da-441f-9160-d8afffe07db2" />

## 3. Fundamentação sobre Enlaces Trunk (IEEE 802.1Q)
* **Encapsulamento IEEE 802.1Q:** Insere uma etiqueta (*tag*) de **4 bytes** no quadro Ethernet antes de enviá-lo pelo cabo *trunk*, contendo o campo `TPID (0x8100)` e o **VID (*VLAN Identifier*) de 12 bits** (identificando VLANs de `1` a `4094`). O switch receptor lê o VID, identifica a VLAN de destino e remove a etiqueta antes de encaminhar o quadro para uma porta de acesso.
* **Porta Trunk e Multiplexação:** Permite que quadros de múltiplas VLANs (10 e 20) trafeguem pelo mesmo cabo físico sem perderem seu isolamento lógico.
* **VLAN Nativa:** É a VLAN cujos quadros atravessam o enlace *trunk* sem etiqueta (*untagged*). Por padrão nos switches Cisco, é a **VLAN 1**, devendo ser mantida idêntica nos dois lados do enlace.
* **Cadastro das VLANs na Distribuição e no Núcleo:** As VLANs 10 e 20 foram criadas também nos switches `DIST01`, `DIST02`, `NUCLEO01` e `NUCLEO02`, pois um switch só encaminha pelo *trunk* as VLANs que existem ativas em seu banco de dados local.

## 4. Documentação Técnica e Mapeamento dos Enlaces Trunk

| Switch Local | Interface Local | Switch Vizinho | Interface Vizinha | Modo | VLANs Permitidas | VLAN Nativa |
| :--- | :---: | :--- | :---: | :---: | :---: | :---: |
| `Switch4` (`ACESSO1`) | `Gig0/1` (ou `Fa0/24`) | `Switch2` (`DIST01`) | `Fa0/1` | `trunk (802.1Q)` | `10,20` | `1` |
| `Switch5` (`ACESSO2`) | `Gig0/1` (ou `Fa0/24`) | `Switch2` (`DIST01`) | `Fa1/1` | `trunk (802.1Q)` | `10,20` | `1` |
| `Switch6` (`ACESSO3`) | `Gig0/1` (ou `Fa0/24`) | `Switch3` (`DIST02`) | `Fa0/1` | `trunk (802.1Q)` | `10,20` | `1` |
| `Switch7` (`ACESSO4`) | `Gig0/1` (ou `Fa0/24`) | `Switch3` (`DIST02`) | `Fa1/1` | `trunk (802.1Q)` | `10,20` | `1` |
| `Switch2` (`DIST01`) | `Fa2/1, Fa3/1` | `Switch0` (`NUCLEO01`) | `Fa8/1, Fa9/1` | `trunk (802.1Q)` | `10,20` | `1` |
| `Switch3` (`DIST02`) | `Fa4/1, Fa5/1` | `Switch1` (`NUCLEO02`) | `Fa8/1, Fa9/1` | `trunk (802.1Q)` | `10,20` | `1` |
| `Switch0` (`NUCLEO01`) | `Fa2/1 a Fa5/1` | `Switch1` (`NUCLEO02`) | `Fa2/1 a Fa5/1` | `trunk (802.1Q)` | `10,20` | `1` |

## 5. Relação Sequencial de Comandos Cisco IOS

### 5.1 Uplinks dos Switches da Camada de Acesso (`ACESSO1` a `ACESSO4`)
```text
enable
configure terminal
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit
end
copy running-config startup-config
```

### 5.2 Switches da Camada de Distribuição (`DIST01` e `DIST02`)
```text
enable
configure terminal
hostname DIST01

vlan 10
 name Alunos
exit
vlan 20
 name Visitantes
exit

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit
interface FastEthernet1/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit
interface FastEthernet2/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit
interface FastEthernet3/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit

end
copy running-config startup-config
```

### 5.3 Switches da Camada de Núcleo (`NUCLEO01` e `NUCLEO02`)
```text
enable
configure terminal
hostname NUCLEO01

vlan 10
 name Alunos
exit
vlan 20
 name Visitantes
exit

interface range FastEthernet2/1 - 5/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit
interface FastEthernet8/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit
interface FastEthernet9/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit

end
copy running-config startup-config
```

## 6. Validação dos Enlaces Trunk (`DIST01` / `Switch2`)
A saída do comando `show interfaces trunk` confirma que os enlaces de interligação estão ativos e permitindo o tráfego das VLANs 10 e 20 no domínio de gerenciamento e no estado de encaminhamento (*forwarding*) do Spanning Tree:

<img width="700" height="708" alt="CLI switch" src="https://github.com/user-attachments/assets/24718cc0-aff7-493f-82bf-1bbded6b0247" />


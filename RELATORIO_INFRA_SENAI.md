# RELATÓRIO TÉCNICO DE INFRAESTRUTURA E SEGURANÇA - PROJETO SENAI

**Data/Hora da Auditoria:** 20/11/2026 00:36:35  
**Topologia:** Multi-Site (Matriz + Filial + Nuvem Privada) com Anel VPN WireGuard + OSPF  

---

## 1. Inventário de Máquinas Virtuais (VirtualBox)

```text
"Firewall-Matriz" {1f56b4e3-f449-4ecc-8310-bb26baced705}
"Srv-App-Matriz" {e7ce624a-83c5-497a-8820-17880a0ae4f3}
"PC-Colaborador" {fa160dbf-0369-4623-a7e5-771e97b14169}
"PC-TI" {d43aed9a-9a80-4e04-aff6-e014a029888f}
"IoT-Matriz" {404655b4-9866-49d8-aec2-a1060e7a31fa}
"Firewall-Filial" {3cfe732f-27b0-4e65-8e92-fedb148ecbbc}
"PC-Vendas-Filial" {d5feb58f-7f4b-43de-aaab-407ae6aed149}
"IoT-Filial" {dc5d766c-d646-4598-a6bf-877870384749}
"Nuvem-DB" {4801f0c0-dc3d-4b59-b45e-1afaec9d7274}
```

## 2. Endereçamento IP e Saúde dos Recursos

| Ambiente | Hostname | RAM Livre | Disco | Interfaces e Sub-redes |
| :--- | :--- | :--- | :--- | :--- |

## 3. Conectividade Criptografada (Anel VPN WireGuard & OSPF)

## 4. Políticas de Firewall e Segmentação (iptables)

## 5. Validação de Serviços, IoT, Banco Cloud e Logs

```text

=== SERVICOS MATRIZ ===
active
active
=== LOGS AUDITORIA APP (ULTIMOS 5) ===
Nov 20 03:08:04 srv-app sudo:  vagrant : TTY=pts/0 ; PWD=/home/vagrant ; USER=root ; COMMAND=/usr/bin/grep App-Clientes-VLAN3 /var/log/syslog
Nov 20 03:08:57 srv-app sudo:  vagrant : TTY=pts/0 ; PWD=/home/vagrant ; USER=root ; COMMAND=/usr/bin/grep App-Clientes-VLAN3 /var/log/auth.log
Nov 20 03:34:10 srv-app App-Clientes-VLAN3: LOGIN SUCESSO: usuario=vendas perfil=VENDAS_FILIAL ip=10.1.20.10
Nov 20 03:34:10 srv-app App-Clientes-VLAN3: CLIENTE CADASTRADO NUVEM: cpf_cnpj=98.765.432/0001-10 por=vendas ip=10.1.20.10
Nov 20 03:36:41 srv-app sudo:  vagrant : TTY=pts/0 ; PWD=/home/vagrant ; USER=root ; COMMAND=/usr/bin/grep App-Clientes-VLAN3 /var/log/auth.log
=== LOGS ALARMES IoT (ULTIMOS 5) ===
Nov 20 02:42:17 iot-matriz IoT-Matriz-Sensores: ALARME ACIONADO! PIR=False, LDR=651lx, Dist=135cm
bash: -c: line 1: syntax error near unexpected token `('
bash: -c: line 1: `echo ""; echo "=== LEITURA SENSORES IoT FILIAL (VLAN 10) ==="; curl -s [http://10.1.10.10:8080](http://10.1.10.10:8080)'

=== POSTGRESQL NUVEM (10.2.0.10) ===
active
could not change directory to "/home/vagrant": Permission denied
 id |          nome           |      cpf_cnpj      | origem_cadastro | criado_por 
----+-------------------------+--------------------+-----------------+------------
  1 | Cliente Filial Cloud SA | 98.765.432/0001-10 | 10.1.20.10      | vendas
(1 row)

```

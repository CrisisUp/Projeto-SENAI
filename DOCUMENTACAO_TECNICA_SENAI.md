# DOCUMENTAÇÃO TÉCNICA E INVENTÁRIO DE REDE - PROJETO SENAI

**Gerado em:** 20/11/2026 00:43  
**Arquitetura:** Multi-Site (Matriz, Filial e Nuvem Privada) — Topologia em Anel VPN + OSPF

---

## 1. Plano de Endereçamento IP e Sub-redes (VLANs)

| Localidade | Segmento / VLAN | Sub-rede (CIDR) | Gateway | Finalidade |
| :--- | :--- | :--- | :--- | :--- |
| **Matriz** | VLAN 3 (Servidores) | `10.0.3.0/24` | `10.0.3.254` | Aplicação Web Flask (RBAC) e Servidor Central Syslog |
| **Matriz** | VLAN 5 (Colaboradores) | `10.0.5.0/24` | `10.0.5.254` | Estações de trabalho (acesso somente leitura na aplicação) |
| **Matriz** | VLAN 8 (TI / Admin) | `10.0.8.0/24` | `10.0.8.254` | Administração total da aplicação, firewalls e SSH na Nuvem |
| **Matriz** | VLAN 10 (IoT Matriz) | `10.0.10.0/24` | `10.0.10.254` | Sensores físicos (PIR, LDR, Ultrassônico) isolados |
| **Filial** | VLAN 20 (Vendas) | `10.1.20.0/24` | `10.1.20.254` | Estações de vendas (NAC por MAC Address + saída via Matriz) |
| **Filial** | VLAN 10 (IoT Filial) | `10.1.10.0/24` | `10.1.10.254` | Sensores físicos locais com envio remoto de alarmes |
| **Nuvem** | VPC Privada (Cloud DB) | `10.2.0.0/24` | `10.2.0.10` | Banco de dados PostgreSQL 14 (SSL/TLS) isolado |
| **Anel VPN** | Túnel Matriz ↔ Nuvem | `172.16.1.0/30` | Punto-a-Ponto | Interface `wg-nuvem` / `wg-matriz` (UDP 51820 - OSPF Cost 10) |
| **Anel VPN** | Túnel Filial ↔ Nuvem | `172.16.2.0/30` | Punto-a-Ponto | Interface `wg-nuvem` / `wg-filial` (UDP 51821 - OSPF Cost 20) |
| **Anel VPN** | Túnel Matriz ↔ Filial | `172.16.3.0/30` | Punto-a-Ponto | Interface `wg-anel` (UDP 51822 - OSPF Cost 10) |

## 2. Inventário das Máquinas Virtuais (Coleta em Tempo Real)

| Ambiente | VM (Vagrant) | Hostname | Porta SSH Host | MAC / Observação | Interfaces e IPs Ativos |
| :--- | :--- | :--- | :--- | :--- | :--- |

## 3. Credenciais, Perfis de Acesso (RBAC) e Serviços

| Sistema / Serviço | Endpoint / Host | Usuário | Senha | Perfil / Permissões |
| :--- | :--- | :--- | :--- | :--- |
| **App Clientes (Web)** | `http://10.0.3.10:8080` | `ti` | `ti123` | `TI_ADMIN` — Criar, Listar e Excluir clientes |
| **App Clientes (Web)** | `http://10.0.3.10:8080` | `vendas` | `vendas123` | `VENDAS_FILIAL` — Criar e Listar clientes |
| **App Clientes (Web)** | `http://10.0.3.10:8080` | `colab` | `colab123` | `COLABORADOR` — Somente Leitura (bloqueado para escrita) |
| **PostgreSQL (Cloud)** | `10.2.0.10:5432` (DB: `senai_clientes`) | `app_user` | `SenhaForteNuvem2026!` | `SELECT, INSERT, UPDATE, DELETE` (somente via SSL de `10.0.3.10`) |
| **VMs Linux (Vagrant)** | Todas as 9 instâncias | `vagrant` | `vagrant` (ou chave SSH) | Acesso administrativo via `vagrant ssh <nome-vm>` |

## 4. Chaves Públicas da Infraestrutura VPN (WireGuard)

```text
Matriz (firewall-matriz) :   nU5UL6TxAhP0crThqsAJJM+qpCC/U5GbDOkM6uEcxzk=
Filial (firewall-filial) :   pBf6Sy+Am5jCpSt5XG/hyeZ/NSO5mOyHONw+IL1ow2k=
Nuvem  (nuvem-db)        :   Ylv8A02MrpSjw39TSBDBn3gox18RCGBCbyhUMjkXCEo=
```

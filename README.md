# Infraestrutura Corporativa Multi-Site Segura (Matriz, Filial e Nuvem) — Projeto SENAI

Arquitetura de rede corporativa distribuída provisionada como **Infraestrutura como Código (IaC)** com **Vagrant + VirtualBox**, implementando **Defesa em Profundidade**, microssegmentação por VLANs, controle de acesso na camada de enlace (**NAC**), malha **VPN WireGuard em Anel com Failover Dinâmico OSPF (FRRouting)**, aplicação web **HTTPS/TLS** com **RBAC**, persistência em **PostgreSQL SSL na Nuvem** e telemetria **IoT** centralizada.

---

## 1. Visão Geral da Arquitetura

A infraestrutura simula uma empresa com três localidades interconectadas sob o princípio de **Menor Privilégio** e **Bloqueio por Padrão (DROP)**:

* **Matriz (10.0.x.0/24):** Sede corporativa que concentra a saída única de Internet (NAT/Firewall), o servidor de aplicação web (HTTPS), o servidor central de logs (
syslog), a administração de TI e sensores IoT locais.
* **Filial (10.1.x.0/24):** Escritório remoto sem acesso direto à Internet local — todo tráfego externo e interno atravessa obrigatoriamente os túneis VPN criptografados e passa pelo controle de acesso baseado em **MAC Address (NAC)**.
* **Nuvem Privada (10.2.0.0/24):** Ambiente isolado (VPC) hospedando o banco de dados relacional **PostgreSQL 14**, acessível exclusivamente via túnel VPN criptografado e autenticação **SSL/TLS (hostssl)**.

```mermaid
flowchart TB
    INTERNET((Internet / WAN<br/>Saída Centralizada via Matriz))

    subgraph NUVEM["NUVEM PRIVADA (VPC Cloud - 10.2.0.0/24)"]
        DB[("nuvem-db<br/>IP: 10.2.0.10 | WAN: 192.168.1.16<br/>PostgreSQL 14 SSL :5432<br/>OSPF Router-ID: 3.3.3.3")]
    end

    subgraph MATRIZ["MATRIZ CORPORATIVA (10.0.x.0/24)"]
        FWM["firewall-matriz<br/>WAN: 192.168.1.13 | NAT: enp0s3<br/>iptables + WireGuard + FRR (OSPF)<br/>OSPF Router-ID: 1.1.1.1"]
        APP["VLAN 3: srv-app (10.0.3.10)<br/>Nginx HTTPS :443 + Flask RBAC<br/>Central Rsyslog :514"]
        COLAB["VLAN 5: pc-colab (10.0.5.10)<br/>Perfil: Somente Leitura"]
        TI["VLAN 8: pc-ti (10.0.8.10)<br/>Perfil: Admin Total + SSH Nuvem"]
        IOTM["VLAN 10: iot-matriz (10.0.10.10)<br/>Sensores PIR + LDR + Ultrassônico"]
        FWM --- APP
        FWM --- COLAB
        FWM --- TI
        FWM --- IOTM
    end

    subgraph FILIAL["FILIAL ESCRITÓRIO (10.1.x.0/24)"]
        FWF["firewall-filial<br/>WAN: 192.168.1.15 (Sem NAT local)<br/>NAC por MAC + WireGuard + FRR<br/>OSPF Router-ID: 2.2.2.2"]
        VENDAS["VLAN 20: pc-vendas-filial (10.1.20.10)<br/>MAC: 08:00:27:AA:BB:10<br/>Perfil: Cadastro + Leitura"]
        IOTF["VLAN 10: iot-filial (10.1.10.10)<br/>Sensores PIR + LDR + Ultrassônico"]
        FWF --- VENDAS
        FWF --- IOTF
    end

    INTERNET <==>|"Tráfego Web Matriz + Filial"| FWM
    FWM <==>|"wg-nuvem (172.16.1.0/30) | OSPF Custo 10"| DB
    FWM <==>|"wg-anel (172.16.3.0/30) | OSPF Custo 10"| FWF
    FWF <-.->|"wg-nuvem (172.16.2.0/30) | OSPF Custo 20 (Failover)"| DB

2. Inventário da Infraestrutura e Plano de Endereçamento

O ambiente é composto por 9 máquinas virtuais Ubuntu 22.04 LTS (ubuntu/jammy64) distribuídas em três diretórios (matriz/, filial/, nuvem/):AmbienteVM (Vagrant)HostnameSub-rede / VLANIP InternoPorta SSH HostFunção PrincipalMatrizfirewall-matrizfw-matrizCore / WAN192.168.1.132222Roteamento inter-VLAN, Firewall iptables, NAT Internet e VPNMatrizservidor-appsrv-appVLAN 3 (10.0.3.0/24)10.0.3.102201Proxy Reverso Nginx HTTPS (:443), App Flask (127.0.0.1:8080) e RsyslogMatrizpc-colaboradorpc-colabVLAN 5 (10.0.5.0/24)10.0.5.102202Estação de trabalho corporativa (acesso somente leitura)Matrizpc-tipc-tiVLAN 8 (10.0.8.0/24)10.0.8.102203Estação administrativa de TI (gestão total + SSH na Nuvem)Matriziot-matriziot-matrizVLAN 10 (10.0.10.0/24)10.0.10.102204Simulador de sensores físicos (PIR, LDR e distância)Filialfirewall-filialfw-filialCore Filial192.168.1.152322Filtro NAC por MAC Address, VPN WireGuard e roteamento OSPFFilialpc-vendas-filialpc-vendas-filialVLAN 20 (10.1.20.0/24)10.1.20.102301Estação homologada (MAC 08:00:27:AA:BB:10) para vendasFilialiot-filialiot-filialVLAN 10 (10.1.10.0/24)10.1.10.102302Sensores físicos da Filial com envio remoto de alarmesNuvemnuvem-dbnuvem-dbVPC (10.2.0.0/24)10.2.0.102422Banco PostgreSQL 14 (senai_clientes) com SSL obrigatório e Security Group3. Controles de Segurança ImplementadosPerímetro e Segmentação (iptables Stateful):Política padrão DROP em INPUT e FORWARD nos firewalls e na Nuvem.Isolamento estrito entre VLANs: estações comuns (VLAN 5) e dispositivos IoT (VLAN 10) não possuem acesso administrativo aos servidores nem ao banco de dados.Controle de Acesso à Rede (NAC) e Tráfego Centralizado na Filial:O firewall-filial valida a origem por IP + Endereço MAC (08:00:27:AA:BB:10). Dispositivos não autorizados são bloqueados na entrada.O NAT local da Filial foi removido: toda navegação de Internet da Filial atravessa a VPN e passa pela inspeção do firewall-matriz.Anel VPN Criptografado + Alta Disponibilidade (WireGuard + OSPF):Três túneis ponto-a-ponto (172.16.1.0/30, 172.16.2.0/30, 172.16.3.0/30) formando um triângulo fechado.Roteamento dinâmico via FRRouting (ospfd) com timers rápidos (hello 2s / dead 6s): em caso de queda do link direto Matriz ↔ Nuvem, o tráfego converge automaticamente via Filial sem perda de serviço.Hardening de Aplicação Web e Banco de Dados:Nginx Proxy Reverso + TLS: Redirecionamento forçado HTTP (:80) para HTTPS (:443), cabeçalhos HSTS, X-Frame-Options e X-Content-Type-Options, com Flask isolado em 127.0.0.1:8080.Autenticação e RBAC: Senhas armazenadas com hash criptográfico com salt (PBKDF2-SHA256), cookies HttpOnly + Secure e controle granular por perfil (TI_ADMIN, VENDAS_FILIAL, COLABORADOR).PostgreSQL SSL (hostssl): Aceita conexões SQL (5432/TCP) exclusivamente do IP 10.0.3.10/32 usando scram-sha-256 e criptografia TLS obrigatória.Auditoria Centralizada e Monitoramento IoT:Todos os eventos de login, falhas de autenticação, tentativas de escalação de privilégio e alarmes dos sensores IoT (presença, luminosidade e distância) são enviados via syslog para o repositório central no srv-app.4. Credenciais de Demonstração (RBAC)PerfilUsuárioSenhaPermissões na Aplicação (https://10.0.3.10)Administrador TI (VLAN 8)titi123Acesso total: listar, cadastrar e excluir clientes + SSH na NuvemEquipe Vendas (Filial)vendasvendas123Listar e cadastrar clientes (exclusão bloqueada)Colaborador (VLAN 5)colabcolab123Somente leitura (tentativas de cadastro/exclusão geram alerta de auditoria)5. Como Subir e Validar o AmbientePré-requisitosOracle VirtualBox 7.x+HashiCorp Vagrant 2.4+Git e PowerShell (Windows)Subir toda a infraestruturaPowerShell# 1. Subir Matriz (5 VMs)
Set-Location .\matriz; vagrant up

# 2. Subir Filial (3 VMs)
Set-Location ..\filial; vagrant up

# 3. Subir Nuvem (1 VM)
Set-Location ..\nuvem; vagrant up
Comandos Rápidos de Auditoria e TestesPowerShell# Verificar saúde das 9 instâncias no VirtualBox
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list runningvms

# Testar acesso HTTPS + RBAC a partir da estação de TI (Matriz)
Set-Location .\matriz
vagrant ssh pc-ti -c 'curl -k -s -c /tmp/c.txt -X POST -d "usuario=ti&senha=ti123" https://10.0.3.10/login > /dev/null && curl -k -s -b /tmp/c.txt https://10.0.3.10/api/clientes'

# Auditar logs de segurança e alarmes IoT centralizados no srv-app
vagrant ssh servidor-app -c 'sudo grep "App-Clientes-VLAN3" /var/log/auth.log | tail -n 10'
6. Documentação Complementar no RepositórioDOCUMENTACAO_TECNICA_SENAI.md: Plano de endereçamento detalhado, portas SSH, credenciais e chaves públicas VPN.RELATORIO_INFRA_SENAI.md: Relatório de auditoria técnica com saída em tempo real dos túneis WireGuard, tabelas OSPF, regras de firewall iptables e diagrama de topologia.

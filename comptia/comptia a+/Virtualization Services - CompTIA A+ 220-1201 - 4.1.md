## Virtualization

Run many different operating systems on the same physical device.

![[assets/Virtualization Services - CompTIA A+ 220-1201 - 4.1-1.png|700]] 

- Each application instance has its **own OS**
- Adds overhead and complexity
- Virtualization is relatively expensive (CPU, RAM e disco por VM)

## The hypervisor

**Virtual Machine Manager (VMM)** — manages the virtual platform and the guest operating systems.

- May require a CPU that supports virtualization
    - Can improve performance significantly
- Handles hardware management for the guests:
    - CPU
    - Networking
    - Security

## Hypervisor types

![[assets/Virtualization Services - CompTIA A+ 220-1201 - 4.1-2.png|700]]

|            | Type 1 — Bare Metal                         | Type 2 — Hosted                                          |
| ---------- | ------------------------------------------- | -------------------------------------------------------- |
| Onde roda  | Direto no hardware, **é o próprio OS**      | Roda **dentro** de um OS existente                       |
| VMs        | Rodam sobre o hypervisor                    | Rodam sobre o OS do host                                 |
| Exemplos   | VMware ESXi, Microsoft Hyper-V, Xen Project | VMware Workstation, Oracle VirtualBox, Parallels Desktop |
| Uso típico | Datacenter, servidor                        | Desktop, lab, estudo                                     |

## Resource requirements

- **CPU** — processor support for virtualization
    - Intel: **VT** (Virtualization Technology)
    - AMD: **AMD-V**
- **Memory** — above and beyond the host OS requirements
- **Disk space**
    - Each guest OS has its own image
    - Precisa de espaço para o OS **e** os dados de cada guest
- **Network**
    - Configurable per guest OS
    - Virtual switch

## Network requirements

Most client-side VMMs ship with their own virtual (internal) networks.

| Modo                    | Como funciona                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **Shared (NAT)**        | A VM compartilha o IP do host físico. Usa IP privado internamente e NAT para converter no IP do host |
| **Bridged**             | A VM é um dispositivo na rede física, com IP próprio da rede                                         |
| **Private / Host-only** | A VM só fala com a rede virtual, sem comunicação externa                                             |

## Hypervisor security

- O hypervisor é um alvo cobiçado pelos atacantes
    - Ainda sem vulnerabilidades significativas conhecidas
- **VM escaping**
    1. O malware percebe que está numa VM
    2. Compromete o hypervisor
    3. Pula de um guest OS para outro
- Muitos serviços hospedados são ambientes virtuais
    - Malware no servidor de um cliente pode coletar informação de outro

## Guest OS security

- Every guest is self-contained — trate como um computador real
- Use traditional security controls:
    - Host-based firewall
    - Anti-virus / anti-spyware
- Cuidado com **rogue VMs** — atacantes instalando o próprio sistema no ambiente
- VMs self-contained fornecidas por terceiros são arriscadas: você não sabe o que roda ali dentro

## Virtual Desktop Infrastructure (VDI)

- As aplicações rodam de fato num servidor remoto
    - O dispositivo local vira só teclado, mouse e tela
    - Também chamado **Desktop as a Service (DaaS)**
- OS mínimo no cliente — sem grande necessidade de memória ou CPU
- Requisito pesado de **rede**: tudo acontece pelo fio

## Application containerization

- **Container**
    - Contém tudo que a aplicação precisa: código + dependências
    - Uma unidade padronizada de software
- Um processo isolado num sandbox
    - Self-contained
    - As apps não interagem entre si
- **Container image**
    - Padrão de portabilidade
    - Leve — usa o **kernel do host**
    - Separação segura entre aplicações

## Virtualized × Containerized

![[assets/Virtualization Services - CompTIA A+ 220-1201 - 4.1-3.png|700]]

|            | Virtualização                        | Containers                        |
| ---------- | ------------------------------------ | --------------------------------- |
| OS         | Um OS completo por VM                | Um único OS, kernel compartilhado |
| Peso       | Pesado (GBs, boot lento)             | Leve (MBs, sobe em segundos)      |
| Isolamento | Isolamento total de hardware virtual | Isolamento de processo no sandbox |
| Gerência   | Hypervisor                           | Container engine (**Docker**)     |

> Só um OS = mais simples, mais leve, mais rápido.
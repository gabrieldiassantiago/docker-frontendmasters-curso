# Containers - Fundamentos Técnicos

## 🧱 O que é um Container (definição real)

Um container é basicamente a combinação de três mecanismos do Linux:

```
Container = chroot + namespaces + cgroups
```

- **chroot**: isolamento de filesystem
- **namespaces**: isolamento de recursos do sistema
- **cgroups**: controle/limitação de recursos

## 1️⃣ chroot - Isolamento de Sistema de Arquivos

### O que faz
- Muda o root filesystem de um processo
- O processo vê apenas um diretório como se fosse o sistema inteiro
- Cria um "jail" (prisão) para arquivos

### Como funciona
```bash
# Criar novo root
mkdir /my-new-root

# Copiar binários necessários
mkdir /my-new-root/bin
cp /bin/bash /bin/ls /my-new-root/bin/

# Copiar bibliotecas (verificar dependências com ldd)
ldd /bin/bash
mkdir /my-new-root/lib
cp <bibliotecas> /my-new-root/lib/

# Executar chroot
chroot /my-new-root bash
```

### Limitações importantes
⚠️ **chroot NÃO isola processos**

Um processo dentro do chroot ainda pode:
- Ver processos do host
- Enviar sinais (kill)
- Interferir em outros processos

**Resultado**: isolamento parcial apenas do filesystem

## 2️⃣ Namespaces - Isolamento de Recursos do Kernel

### Problema resolvido
Mesmo em chroot, processos ainda enxergam o sistema global.

### O que são
Namespaces criam visões isoladas do sistema para cada processo.

### Tipos de Namespaces
| Namespace | Isola |
|-----------|-------|
| PID | processos |
| NET | rede |
| MOUNT | sistemas de arquivos |
| UTS | hostname |
| IPC | comunicação entre processos |
| USER | usuários/permissões |

### Comando principal
```bash
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /better-root bash

# Montar filesystems necessários
mount -t proc none /proc    # process namespace
mount -t sysfs none /sys    # filesystem
mount -t tmpfs none /tmp    # filesystem
```

### Resultado
✔️ Não vê processos externos  
✔️ Não interfere no host  
✔️ Possui stack de rede própria

## 3️⃣ cgroups - Controle de Recursos

### Problema resolvido
Mesmo isolado, um processo ainda pode:
- Consumir toda CPU
- Consumir toda RAM  
- Criar milhares de processos (fork bomb)

### Como funciona
Interface baseada em pseudo-filesystem: `/sys/fs/cgroup/`

```bash
# Criar cgroup
mkdir /sys/fs/cgroup/sandbox

# Mover processo para cgroup
echo <PID> > /sys/fs/cgroup/sandbox/cgroup.procs

# Habilitar controllers
echo "+cpuset +cpu +io +memory +hugetlb +pids +rdma" > /sys/fs/cgroup/cgroup.subtree_control
```

### Limites comuns

#### 🧠 Memória
```bash
echo 83886080 > /sys/fs/cgroup/sandbox/memory.max  # 80MB
```

#### ⚡ CPU  
```bash
echo "5000 100000" > /sys/fs/cgroup/sandbox/cpu.max  # 5% de um core
```

#### 🔢 Processos (anti fork bomb)
```bash
echo 3 > /sys/fs/cgroup/sandbox/pids.max
```

### Fork Bomb Protection
```bash
# Fork bomb (NÃO execute sem proteção!)
:(){ :|:& };:

# Com cgroup limitado a 3 PIDs, o fork bomb é contido
```

## 📊 Resumo Técnico

| Tecnologia | Resolve |
|------------|---------|
| chroot | isolamento de arquivos |
| namespaces | isolamento do sistema |
| cgroups | controle de recursos |

## 🕰️ História dos Containers

### Containers existiam antes do Docker

- **1979**: chroot (Unix v7)
- **2000**: FreeBSD Jails
- **2004**: Solaris Zones  
- **2008**: LXC (Linux Containers)
- **2013**: Docker (facilita uso de containers)

### O que o Docker fez
Docker **NÃO inventou** containers. Ele resolveu problemas de usabilidade:

**Antes do Docker:**
- Configurar chroot manualmente
- Lidar com namespaces
- Configurar cgroups
- Montar networking
- Copiar libs
- Criar root filesystem

**Docker trouxe:**
✔️ Imagens versionadas (Dockerfile)  
✔️ Cache de layers  
✔️ Distribuição fácil (Docker Hub)  
✔️ CLI simples  
✔️ Padrão para empacotar apps

## 🧠 Coisas Importantes

### Container vs VM
```
VM: hardware virtual → kernel → sistema → app
Container: kernel já existe → app inicia
```

**Resultado:**
- Container: milissegundos
- VM: segundos/minutos

### Stack real do Docker
```
Docker CLIso muda completamente como você usa containers.
   ↓
Docker Engine  
   ↓
containerd
   ↓
runc
   ↓
Linux kernel (namespaces + cgroups)
```

### Conceitos-chave
- Container = **processo Linux isolado**, não VM
- Usa o **mesmo kernel** do host
- **Imagem** = template imutável
- **Container** = instância rodando da imagem

---

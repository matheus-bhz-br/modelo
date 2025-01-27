# Documentação: Configuração de Volume Replicado GlusterFS

## Índice
1. Introdução
2. Arquitetura
3. Pré-requisitos
4. Processo de Instalação
5. Problemas Encontrados e Soluções
6. Melhores Práticas
7. Verificação e Testes

## 1. Introdução

Este documento detalha o processo de configuração de um cluster GlusterFS com três nós em configuração replicada. O ambiente foi configurado em janeiro de 2025 usando GlusterFS 10.3 em sistemas Debian.

### Configuração Final
- 3 nós em replicação total
- Volume montado via FUSE
- Replicação síncrona de dados
- Self-heal ativo

## 2. Arquitetura

### Nós do Cluster
- Nó 1 (rede): 172.29.224.106
- Nó 2 (hailo): 172.29.221.194
- Nó 3 (dados): 172.29.218.187

### Estrutura de Diretórios
- Ponto de montagem: `/mnt/glusterfs`
- Diretório dos bricks: `/media/matheus/rede/data/gluster/gluster_volume`

## 3. Pré-requisitos

### Em Todos os Nós
```bash
# Instalação do GlusterFS
sudo apt install glusterfs-server

# Configuração do Firewall
sudo ufw allow 24007,24008/tcp
sudo ufw allow 49152:49251/tcp
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw reload
```

### Portas Necessárias
- 24007: Gerenciamento do GlusterFS
- 24008: Comunicação entre nós
- 49152:49251: Portas dinâmicas para bricks
- Portas específicas dos bricks (identificadas durante a configuração)

## 4. Processo de Instalação

### 4.1 Preparação dos Nós
```bash
# Em todos os nós
sudo systemctl start glusterd
sudo systemctl enable glusterd
```

### 4.2 Configuração do Cluster
```bash
# No primeiro nó (rede)
sudo gluster peer probe 172.29.221.194
sudo gluster peer probe 172.29.218.187

# Verificar status
sudo gluster peer status
```

### 4.3 Criação do Volume
```bash
# Criar diretórios em todos os nós
sudo mkdir -p /media/matheus/rede/data/gluster/gluster_volume
sudo chown -R nobody:nogroup /media/matheus/rede/data/gluster/gluster_volume

# Criar volume replicado
sudo gluster volume create volume1 replica 3 \
    172.29.224.106:/media/matheus/rede/data/gluster/gluster_volume \
    172.29.221.194:/media/matheus/rede/data/gluster/gluster_volume \
    172.29.218.187:/media/matheus/rede/data/gluster/gluster_volume

# Iniciar o volume
sudo gluster volume start volume1
```

### 4.4 Montagem do Volume
```bash
# No nó onde deseja montar
sudo mkdir -p /mnt/glusterfs
sudo mount.glusterfs 172.29.224.106:volume1 /mnt/glusterfs
```

## 5. Problemas Encontrados e Soluções

### 5.1 Problema: Diretório já parte de um volume
**Erro:** "/media/matheus/rede/data/gluster/replica is already part of a volume"

**Solução:**
1. Renomear/mover diretório existente
2. Criar novo diretório com nome diferente
3. Limpar metadados do GlusterFS se necessário:
```bash
sudo rm -rf /var/lib/glusterd/*
sudo systemctl restart glusterd
```

### 5.2 Problema: Falha na Montagem
**Erro:** "Mount failed. Check the log file for more details."

**Solução:**
1. Verificar conectividade das portas
2. Liberar portas específicas dos bricks
3. Reinstalar cliente GlusterFS
4. Verificar logs detalhados

### 5.3 Problema: Conectividade
**Solução:**
```bash
# Liberar portas específicas dos bricks
sudo ufw allow 57066/tcp
sudo ufw allow 59763/tcp
sudo ufw allow 56225/tcp
sudo ufw reload
```

## 6. Melhores Práticas

### 6.1 Preparação
- Planejar estrutura de diretórios antecipadamente
- Documentar IPs e hostnames
- Verificar requisitos de rede antes da instalação
- Configurar NTP para sincronização de tempo

### 6.2 Configuração
- Usar nomes descritivos para volumes
- Implementar monitoramento desde o início
- Configurar backup dos metadados do GlusterFS
- Documentar todas as alterações

### 6.3 Manutenção
- Monitorar regularmente logs e status
- Manter backup dos arquivos de configuração
- Testar recuperação de falhas periodicamente
- Manter documentação atualizada

## 7. Verificação e Testes

### 7.1 Verificação do Cluster
```bash
# Status dos peers
sudo gluster peer status

# Informações do volume
sudo gluster volume info
sudo gluster volume status
```

### 7.2 Teste de Replicação
```bash
# Criar arquivo de teste
sudo touch /mnt/glusterfs/test.txt

# Verificar em todos os nós
ls -la /media/matheus/rede/data/gluster/gluster_volume/
```

### 7.3 Verificação de Montagem
```bash
# Verificar ponto de montagem
df -h | grep glusterfs
mount | grep glusterfs
```

## Conclusão

Esta configuração estabelece um cluster GlusterFS de três nós com replicação total, proporcionando alta disponibilidade e redundância de dados. As principais dificuldades encontradas foram relacionadas à conectividade de rede e configuração de portas, que foram resolvidas com a correta configuração do firewall e liberação das portas necessárias.

Para futuras implementações, recomenda-se:
1. Planejamento detalhado da estrutura de rede
2. Documentação prévia de todos os requisitos
3. Testes de conectividade antes da instalação
4. Implementação de monitoramento desde o início
5. Planejamento de backup e recuperação

Esta documentação deve ser mantida atualizada conforme alterações são realizadas no ambiente.
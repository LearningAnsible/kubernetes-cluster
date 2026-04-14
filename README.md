# Kubernetes Cluster - Ansible Role

[![License: CC BY-NC-SA](https://img.shields.io/badge/License-CC_BY--NC--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode)

Rol de Ansible para instalar y configurar un cluster de Kubernetes en Red Hat Enterprise Linux 10 usando **kubeadm**.

## Descripción

Este rol automatiza la instalación y configuración de un cluster de Kubernetes en sistemas RHEL 10. Incluye la instalación de componentes esenciales como:

- **containerd** - runtime de contenedores
- **Kubernetes** - orquestación de contenedores (kubectl, kubeadm, kubelet)
- **Flannel** - plugin de red (CNI)

El rol soporta tanto instalaciones de cluster multi-nodo como clusters de nodo único (control-plane que actúa como worker).

## Requisitos

- Red Hat Enterprise Linux 10
- Ansible 2.2 o superior
- Acceso root o permisos sudo en los nodos destino
- Conectividad de red entre los nodos

## Características

✅ Instalación automatizada de Kubernetes  
✅ Configuración de containerd como runtime  
✅ Instalación y configuración de Flannel (CNI)  
✅ Soporte para clusters de nodo único  
✅ Configuración automática de hosts y puertos  
✅ Deshabilitación de swap  

## Variables Principales

Las principales variables se encuentran en `vars/main.yml`:

```yaml
# CIDR de la red de pods (debe coincidir con la configuración de Flannel)
pod_network_cidr: "10.244.0.0/16"

# Dirección IP del nodo master
master_ip: "192.168.1.94"

# Nombre del nodo master
master_hostname: "kube-master"

# Instalar como cluster de nodo único
install_single_node_cluster: true
```

## Estructura del Rol

```
├── defaults/           # Variables por defecto
├── files/              # Archivos estáticos
├── handlers/           # Handlers de Ansible
├── meta/               # Información del rol
├── tasks/              # Tareas principales
│   ├── main.yml                    # Orquestador de tareas
│   ├── disable_swap.yml            # Deshabilitar swap
│   ├── config_files.yml            # Configurar archivos
│   ├── config_hosts.yml            # Configurar /etc/hosts
│   ├── config_ports.yml            # Configurar puertos
│   ├── install_containerd.yml      # Instalar runtime
│   ├── install_flannel.yml         # Instalar CNI
│   ├── install_kubernetes.yml      # Instalar K8s
│   ├── start_cluster.yml           # Iniciar cluster
│   └── single_node.yml             # Config de nodo único
├── templates/          # Plantillas Jinja2
└── vars/               # Variables del rol
```

## Uso

### 1. Configurar el intento

Editar `vars/main.yml` con los parámetros de tu cluster:

```yaml
pod_network_cidr: "10.244.0.0/16"
master_ip: "YOUR_IP"
master_hostname: "YOUR_HOSNAME"
install_single_node_cluster: true
```

### 2. Crear un playbook

```yaml
---
- name: Instalar cluster Kubernetes
  hosts: k8s_nodes
  become: yes
  roles:
    - kubernetes_cluster
```

### 3. Ejecutar el playbook

```bash
ansible-playbook -i inventory playbook.yml
```

## Tareas Disponibles

El rol incluye las siguientes tareas (puede habilitarlas/deshabilitarlas en `tasks/main.yml`):

| Tarea | Descripción |
|-------|-------------|
| `disable_swap.yml` | Deshabilita swap en los nodos |
| `config_files.yml` | Crea archivos de configuración necesarios |
| `install_containerd.yml` | Instala y configura containerd |
| `install_kubernetes.yml` | Instala kubectl, kubeadm y kubelet |
| `config_ports.yml` | Configura puertos del firewall |
| `config_hosts.yml` | Configura resolución de nombres |
| `start_cluster.yml` | Inicia el cluster de Kubernetes |
| `install_flannel.yml` | Instala plugin de red Flannel |
| `single_node.yml` | Remueve taints para nodos únicos |

## Configuración de Nodo Único

Para instalar un cluster de nodo único (desarrollo/pruebas):

```yaml
install_single_node_cluster: true
```

Esto configurará el nodo control-plane como worker, permitiendo que ejecute cargas de trabajo.

## Componentes Instalados

- **Kubernetes v1.35** - Desde repositorio oficial de Kubernetes
- **containerd** - Runtime de contenedores OCI
- **Flannel** - Plugin de red (CNI)
- **kubeadm** - Herramienta de bootstrapping
- **kubectl** - Cliente de línea de comandos
- **kubelet** - Agente del nodo

## Inventario de ejemplo

```yaml
kube:
    hosts:
        localhost:
            ansible_user: ansible
            ansible_password: ansible
            ansible_host_key_checking: false
```

## Configuración de Red

El rol utiliza **Flannel** como plugin de red (CNI). La configuración por defecto:

- **CIDR de pods**: `10.244.0.0/16`
- **Backend**: VXLAN (puerto 8472/UDP)

Estas configuraciones pueden ajustarse modificando las variables.

## Añadir un worker al cluster

Para añadir un worker al cluster, hemos de ejecutar el siguiente comando en el nodo master:

```bash
kubeadm token create --print-join-command
```

Esto nos mostrará un comando como el siguiente:

```bash
kubeadm join 192.168.1.94:6443 --token abc123.xyzabc123 --discovery-token-ca-cert-hash sha256:abc123...
```

### Pasos para añadir el worker:

1. **En el nodo master**: Ejecutar el comando anterior para obtener el token de unión
2. **En el worker**: Ejecutar el comando con sudo:
   ```bash
   sudo kubeadm join 192.168.1.94:6443 --token abc123.xyzabc123 --discovery-token-ca-cert-hash sha256:abc123...
   ```
3. **Verificar conexión**: En el master, esperar unos segundos y ejecutar:
   ```bash
   kubectl get nodes
   ```

El worker debería aparecer en la lista con estado `Ready` en pocos minutos.

## Troubleshooting

### Verificar estado del cluster

```bash
kubectl get nodes
kubectl get pods -A
kubectl get svc -A
```

### Ver logs del kubelet

```bash
systemctl status kubelet
journalctl -u kubelet -n 50
```

### Información del nodo

```bash
kubectl describe node <node-name>
```

## Autor

- **Alejandro López**

## Licencia

Este proyecto está bajo la licencia Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0).

No se permite el uso comercial sin permiso del autor.

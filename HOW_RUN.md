# Guía de Ejecución

## 📋 Prerrequisitos

Antes de ejecutar el módulo, asegúrate de tener:

- **Ansible 2.2+** instalado en tu máquina de control
- **Red Hat Enterprise Linux 10** en los nodos destino
- **Conectividad SSH** a los nodos (sin requerir contraseña o con credenciales configuradas)
- **Acceso root o permisos sudo** en los nodos
- **Red estable** entre los nodos

## 🔧 Paso 1: Configurar las Variables

Edita el archivo `vars/main.yml` con tus parámetros:

```bash
nano vars/main.yml
```

Define las variables según tu entorno:

```yaml
pod_network_cidr: "10.244.0.0/16"      # CIDR de la red de pods (Flannel)
master_ip: "192.168.1.94"               # IP del nodo master
master_hostname: "kube-master"          # Hostname del nodo master
worker_ip: "192.168.1.95"               # IP del nodo worker (si aplica)
worker_hostname: "kube-worker"          # Hostname del nodo worker (si aplica)
install_single_node_cluster: false      # true si deseas un cluster de nodo único
```

## 📝 Paso 2: Configurar el Inventario

Edita el archivo `inventory` con los datos de conexión SSH:

```bash
nano inventory
```

Asegúrate de que los hosts y credenciales sean correctos:

```yaml
kube:
    hosts:
        kube-master:
            ansible_user: ansible           # Usuario SSH
            ansible_password: ansible      # Contraseña SSH
            ansible_host_key_checking: false
        kube-worker:
            ansible_user: ansible
            ansible_password: ansible
            ansible_host_key_checking: false
```

**Alternativa más segura:** Usa claves SSH en lugar de contraseñas:
```yaml
kube:
    hosts:
        kube-master:
            ansible_user: ansible
            ansible_host: 192.168.1.94
            ansible_private_key_file: ~/.ssh/id_rsa
            ansible_host_key_checking: false
```

## ✅ Paso 3: Verificar Conectividad

Verifica que Ansible puede conectar con los nodos:

```bash
ansible all -i inventory -m ping
```

Salida esperada:
```
kube-master | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## 🚀 Paso 4: Ejecutar el Playbook Principal

Ejecuta el playbook para instalar y configurar el cluster Kubernetes:

```bash
ansible-playbook -i inventory main.yml
```

**Opciones útiles:**

```bash
# Con modo verbose (más detalles de ejecución)
ansible-playbook -i inventory main.yml -v

# Modo extra verbose (mucha más información)
ansible-playbook -i inventory main.yml -vv

# Simular sin hacer cambios (dry-run)
ansible-playbook -i inventory main.yml --check
```

## 📊 Paso 5: Monitorear la Ejecución

Durante la ejecución, el playbook realizará:

1. ✅ Inclusión de variables
2. ✅ Deshabilitación de SWAP (comentado por defecto)
3. ✅ Instalación de containerd (runtime de contenedores)
4. ✅ Instalación de Kubernetes (kubeadm, kubectl, kubelet)
5. ✅ Configuración de puertos y firewall
6. ✅ Inicialización del cluster (en nodo master)
7. ✅ Instalación de Flannel (CNI plugin)
8. ✅ Configuración de nodo único (si aplica)

## ⚙️ Paso 6: Conficurar cliente kubectl

Una vez completado el job de ansible ejectamos lo siguente para tener la conexion de kubectl con el cluster:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## ✔️ Paso 7: Verificar la Instalación

Una vez completado, verifica que el cluster esté funcionando:

### En el nodo master:

```bash
# Verificar nodos
kubectl get nodes

# Verificar pods del sistema
kubectl get pods -n kube-system

# Verificar estado del cluster
kubectl cluster-info
```

### Estado esperado:

```
NAME          STATUS   ROLES           AGE     VERSION
kube-master   Ready    control-plane   5m      v1.xx.x
kube-worker   Ready    <none>          3m      v1.xx.x
```

## 🔄 Paso 8: Agregar Workers (Opcional)

2. **Obtén el token de unión en el master:**
   ```bash
   sudo kubeadm token create --print-join-command
   ```

3. **Ejecuta el comando de unión en el worker (como root):**
   ```bash
   sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

## ⚠️ Solución de Problemas

### Error: "Host key verification failed"

**Solución:** Desactiva la verificación de claves en SSH:
```bash
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook -i inventory main.yml
```

### Error: "Connection refused" o "No route to host"

**Solución:** Verifica:
- Conectividad de red entre nodos
- Firewall habilitado/deshabilitado
- IPs correctas en inventory y vars/main.yml

### Verificar estado de la instalación

```bash
# Ver logs de kubelet
sudo journalctl -u kubelet -n 50

# Ver logs de Kubernetes
sudo kubectl describe node <nombre-nodo>
```

## 🎯 Casos de Uso Comunes

### Instalar un cluster de nodo único

Edita `vars/main.yml`:
```yaml
install_single_node_cluster: true
```

El nodo master actuará como worker, permitiendo ejecutar pods en él.

### Reinstalar desde cero

```bash
# Limpiar el cluster anterior (en master)
sudo kubeadm reset --force

# Ejecutar el playbook nuevamente
ansible-playbook -i inventory main.yml
```

## 📚 Referencia Rápida

| Comando | Descripción |
|---------|-------------|
| `ansible-playbook -i inventory main.yml` | Ejecutar instalación completa |
| `ansible all -i inventory -m ping` | Verificar conectividad |
| `kubectl get nodes` | Ver estado de nodos |
| `kubectl get pods -A` | Ver todos los pods |
| `sudo kubeadm token create --print-join-command` | Token para agregar workers |
| `kubectl apply -f <archivo.yaml>` | Desplegar aplicación |

---

**Última actualización:** Abril 2026  
**Versión:** 1.0

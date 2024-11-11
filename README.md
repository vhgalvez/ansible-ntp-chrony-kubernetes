# Ansible NTP FreeIPA Kubernetes

Este repositorio proporciona una configuración automatizada para sincronizar el tiempo (NTP) dentro de un entorno de clúster Kubernetes utilizando Ansible. FreeIPA actúa como el servidor NTP central, mientras que los clientes usan `chrony` y `systemd-timesyncd` para garantizar una sincronización horaria precisa.

## Descripción del Proyecto

El objetivo de este proyecto es asegurar la sincronización horaria en un clúster Kubernetes. FreeIPA sirve como servidor NTP y cada nodo en el clúster sincroniza su tiempo a través de `chrony` (en el servidor) o `systemd-timesyncd` (en los clientes).

## Características

- **Configuración Automatizada del Servidor NTP**: Configura `chrony` en FreeIPA para los servicios de protocolo de tiempo de red (NTP).
- **Sincronización de Clientes**: Configura `systemd-timesyncd` en los nodos clientes para un alineamiento horario consistente.
- **Configuración del Firewall**: Abre los puertos necesarios para NTP en el servidor FreeIPA.
- **Fuentes NTP Personalizables**: Especifica fácilmente servidores NTP preferidos en la configuración.

## Requisitos Previos

- **Ansible**: Asegúrate de tener Ansible instalado en el nodo de control.
- **Acceso a los Nodos de FreeIPA y Kubernetes**: El archivo de inventario debe configurarse con acceso SSH.
- **Rocky Linux**: Ambiente recomendado para el nodo FreeIPA, aunque debería ser compatible con otros sistemas.

## Configuración del Inventario

Crea un archivo de inventario (`inventory.ini`) para especificar tus nodos. A continuación, un ejemplo de formato:

```ini
[freeipa_servers]
10.17.3.11 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22

[clients]
10.17.4.21 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.4.22 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.4.23 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.4.24 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.4.25 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.4.26 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.4.27 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.3.12 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.3.13 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
10.17.3.14 ansible_user=<nombre_usuario> ansible_ssh_private_key_file=<ruta_clave_ssh>
```


### Resumen de Recursos para Máquinas Virtuales

| Nombre de VM      | CPU | Memoria (MB) | IP           | Nombre de Dominio                 | Tamaño de Disco (GB) | Hostname     |
|------------------|-----|--------------|--------------|-----------------------------------|----------------------|--------------|
| master1          | 2   | 4096         | 10.17.4.21   | master1.cefaslocalserver.com      | 50                   | master1      |
| master2          | 2   | 4096         | 10.17.4.22   | master2.cefaslocalserver.com      | 50                   | master2      |
| master3          | 2   | 4096         | 10.17.4.23   | master3.cefaslocalserver.com      | 50                   | master3      |
| worker1          | 2   | 4096         | 10.17.4.24   | worker1.cefaslocalserver.com      | 50                   | worker1      |
| worker2          | 2   | 4096         | 10.17.4.25   | worker2.cefaslocalserver.com      | 50                   | worker2      |
| worker3          | 2   | 4096         | 10.17.4.26   | worker3.cefaslocalserver.com      | 50                   | worker3      |
| bootstrap        | 2   | 4096         | 10.17.4.27   | bootstrap.cefaslocalserver.com    | 50                   | bootstrap    |
| freeipa1         | 2   | 2048         | 10.17.3.11   | freeipa1.cefaslocalserver.com     | 32                   | freeipa1     |
| loadbalancer1    | 2   | 2048         | 10.17.3.12   | loadbalancer1.cefaslocalserver.com | 32                  | loadbalancer1|
| postgresql1      | 2   | 2048         | 10.17.3.13   | postgresql1.cefaslocalserver.com  | 32                   | postgresql1  |
| helper           | 2   | 2048         | 10.17.3.14   | helper.cefaslocalserver.com       | 32                   | helper_node  |

### Resumen de Recursos para Nodos de Kubernetes

| Nodo               | Sistema Operativo       | Función                                    | Cantidad |
| ------------------ | ----------------------- | ------------------------------------------ | -------- |
| Load Balancer Node | Rocky Linux             | Balanceo de tráfico con Traefik            | 1        |
| FreeIPA Node       | Rocky Linux             | DNS y autenticación                        | 1        |
| PostgreSQL Node    | Rocky Linux             | Base de datos central para microservicios  | 1        |
| Master Node        | Flatcar Container Linux | Administración de API de Kubernetes        | 3        |
| Worker Nodes       | Flatcar Container Linux | Ejecución de microservicios y aplicaciones | 3        |
| Bootstrap Node     | Flatcar Container Linux | Nodo inicial para configurar el clúster    | 1        |


## Uso

1. **Clonar el Repositorio**

   ```bash
   git clone https://github.com/yourusername/ansible-ntp-freeipa-kubernetes.git
   cd ansible-ntp-freeipa-kubernetes
   ```

2. **Ejecutar el Playbook de Ansible**

   Ejecuta el playbook principal para configurar NTP en el servidor FreeIPA y los nodos clientes.

   ```bash
   ansible-playbook -i inventory.ini ntp_setup.yml
   ```

3. **Verificar Sincronización**

   En cada nodo cliente, puedes verificar el estado de sincronización de la hora usando:

   ```bash
   timedatectl status
   ```

## Estructura del Proyecto

- **ntp_setup.yml**: Playbook principal para configurar NTP en el servidor FreeIPA y los clientes.
- **templates/chrony.conf.j2**: Plantilla para configurar chrony en el servidor FreeIPA.
- **inventory.ini**: Archivo de inventario de ejemplo (modifícalo según sea necesario para tu entorno).

## Detalles de Configuración

### FreeIPA (Servidor NTP)

- Instala y configura chrony para usar las fuentes NTP especificadas.
- Abre el puerto 123 en el firewall para la comunicación NTP.

### Nodos de Kubernetes (Clientes)

- Configura systemd-timesyncd para sincronizarse con el servidor FreeIPA.
- Habilita la sincronización horaria en el arranque del sistema.

## Solución de Problemas

Si surgen problemas de sincronización de tiempo, verifica lo siguiente:

- **Configuración del Firewall**: Asegúrate de que el puerto 123 esté abierto en el servidor FreeIPA.
- **Estado del Servicio**: Verifica que chronyd en el servidor FreeIPA y systemd-timesyncd en los clientes estén en ejecución.
- **Conectividad de Red**: Confirma que todos los nodos puedan comunicarse con el servidor FreeIPA.

Usa los siguientes comandos para depurar:

```bash
# En el servidor FreeIPA
sudo chronyc sources -v

# En los clientes
timedatectl status
```




## Licencia

Este proyecto está licenciado bajo la Licencia MIT.

## Autor

Victor Galvez [https://github.com/vhgalvez](https://github.com/vhgalvez)

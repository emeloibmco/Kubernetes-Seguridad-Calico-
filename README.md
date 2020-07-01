# Políticas de Calico sobre un clúster IKS 😺☁️ 
En esta guía encontrará la descripción detallada de cómo utilizar las políticas de red de Calico para bloquear del tráfico en un clúster de Kubernetes en IBM Cloud.
## Descripción general 📌
Para el desarrollo de esta guía se desplegó una aplicación en un nodo del clúster "Cluster IKS-Demo" en IBM cloud y una base de datos MongoDB en un segundo nodo de este clúster. El objetivo es bloquear el tráfico de entrada a la base de datos de nuestra aplicación utilizando las políticas de red de Calico, ya que de forma predeterminada, los servicios Kubernetes NodePort, LoadBalancer e Ingress hacen que la app esté disponible en todas las interfaces de red de clúster públicas y privadas y, por motivos de seguridad, es conveniente no permitir el tráfico a nuestra base de datos donde pueden encontrarse datos delicados.

La siguiente imagen ilustra la arquitectura trabajada a lo largo de la guía.

<img src="https://github.com/JulianaLeonGonzalez/Politicas_Calico/blob/master/Arquitectura.jpg" />

## Prerrequisitos 📌
* [Instalación y configuración de la CLI de Calico ](https://cloud.ibm.com/docs/containers?topic=containers-network_policies#cli_install)
* [Creación de un clúster IKS](https://cloud.ibm.com/docs/containers?topic=containers-clusters&locale=es#clusters_ui
)

## Visualización de políticas de red 📌
**Paso 1:** Inicie sesión en IBM Cloud. Si tiene un ID federado, utilice ibmcloud login --sso para iniciar la sesión. Especifique su nombre de usuario y utilice el URL proporcionado en la salida de la CLI para recuperar el código de acceso de un solo uso. Adicionalmente apunte al grupo de recursos donde se encuentra su clúster.
```sh
ibmcloud login
ibmcloud target -g <resource_group_name>
```
**Paso 2:** Obtenga una lista de todos los clústeres de la cuenta para obtener el nombre del clúster y su correspondiente ID.
```sh
ibmcloud ks cluster ls
```
**Paso 3:** Descargue la configuración de Kubernetes con los párametros de Calico.
```sh
ibmcloud ks cluster config --cluster <cluster_name_or_ID> --admin --network
```
**Paso 4:** El comando anterior le arrojará una ruta, copiela y úsela en el siguiente comando para guardar nuestro archivo de configuración en una variable que facilite los pasos siguientes.
```sh
$config = <ruta/calicoctl.cfg>
```
**Paso 5:** Consulte todas las políticas de red de Calico y de Kubernetes que se han creado para el clúster.
```sh
calicoctl get GlobalNetworkPolicy –o wide –config=$config
```
## Adición de políticas de red 📌

Calico le permite crear una gran variedad de políticas mediante diferentes métodos, por ejemplo, podrá bloquear todo tipo de tráfico a sus nodos, o solo bloquear el tráfico proveniente de algunas IPs especificadas en lo que calico llama "listas blancas", todo dependerá de los requerimientos de seguridad de su aplicación. Como se explicó anteriormente, para esta aplicación se necesita denegar todo el tráfico entrante a la base de datos, esto se hará denegando todo el tráfico cuyo puerto de destino corresponda al puerto en el que esta expuesta la base de datos.

Para esto, defina su política de red de Calico mediante la creación de un script de configuración (.yaml).

**Paso 1:** Creamos el archivo YAML con nuestra política personalizada en la raíz de nuestra aplicación. El archivo que contiene la política aplicada en esta guía la encontrará [aquí.](https://github.com/emeloibmco/Kubernetes-Mood-Marbles-Backend/blob/master/calicoPolicies/deny-nodeports.yaml)

**Paso 2:** Aplique las políticas al clúster.
```sh
 calicoctl apply -f <file_name>.yaml --config=$config
```
Finalmente podrá comprobar el funcionamiento de las políticas aplicadas accediendo desde su navegador de preferencia a la IP:PORT donde se encuentra desplegada su base de datos, verá un mensaje de error ya que no se permitirá la entrada de su petición.

## Referencias 🔗
Encuentre más información sobre las políticas de cálico en: [Control del tráfico con políticas de red](https://cloud.ibm.com/docs/containers?topic=containers-network_policies)

Encuentre más información sobre el archivo .yaml en: [Política de red global](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy)
## Autores ✒️
Equipo IBM Cloud Tech sales Colombia.

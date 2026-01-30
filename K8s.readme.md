# Helm commands

* Crear configuración `helm create <nombre>`
* Aplicar configuración inicial: `helm install <nombre> .`
* Aplicar actualizaciones: `helm upgrade <nombre> .`

# K8s commands

* Obtener pods, deployments y services: `kubectl get <pods | deployments | services>`
* Revisar todos pods: `kubectl describe pods`
* Revisar un pod: `kubectl describe pod <nombre>`
* Eliminar pod: `kubectl delete pod <nombre>`
* Revisar logs: `kubectl logs <nombre>`



# Crear deployment:
```
kubectl create deployment <nombre> --image=<registro/url/imagen> --dry-run=client -o yaml > deployment.yml
```

# Crear service
```
kubectl create service clusterip <nombre> --tcp=<8888> --dry-run=client -o yaml > service.yml 
**kubectl create service nodeport <nombre> --tcp=<3000> --dry-run=client -o yaml > service.yml**
```
* **clusterip**: solo se puede acceder desde dentro del cluster
* **nodeport**: se puede acceder desde fuera del cluster


# Secrets

* Crear secretos, varios a la vez, o uno por uno.
```
kubectl create secret generic <nombre> --from-literal=key=value

kubectl create secret generic secret1 --from-literal=key1=value1 --from-literal=key2=value2
```
* Obtener los secretos `kubectl get secrets`
* Ver el contenido de un secreto `kubectl get secrets <nombre> -o yaml`

## Editar un secret
La forma más fácil es borrarlo y volverlo a crear pero si es más de un secret, no vamos a querer perder los demás.
Recordar que los secrets están en `base64`, por lo que si queremos editar un secret, debemos hacerlo en `base64`.

1. Editar el secret con `kubectl edit secret <nombre>` esto invocará el editor
2. Cambiar el valor (se puede usar un editor en [línea para convertir a base64](https://www.rapidtables.com/web/tools/base64-decode.html))
3. Tocar **i** para insertar líneas y editar el archivo
4. Poner el valor a decodificar en una nueva línea
5. Presionar **esc** y luego `:. ! base64 -D` para decodificar el valor
6. Presionar **i** para insertar o editar el valor
7. Presionar **esc** y luego `:. ! base64` para codificar el valor
8. Editar nuevamente el archivo **i** y dejar la línea en su posición
9. Presionar **esc** y luego **:wq** para guardar y salir



## Configurar secretos de Google Cloud para obtener las imágenes

1. Crear secreto:
```
kubectl create secret docker-registry gcr-json-key --docker-server=SERVIDOR-DE-GOOGLE-docker.pkg.dev --docker-username=_json_key --docker-password="$(cat 'PATH/DE/Tienda Microservices IAM.json')" --docker-email=TU_CORREO@gmail.com
```

2. Path del secreto para que use la llave:
```
kubectl patch serviceaccounts default -p '{ "imagePullSecrets": [{ "name":"gcr-json-key" }] }'
```


## Exportar y aplicar configuraciones con archivos (secrets en este caso)
* Para exportar los archivos de configuración

```
kubectl get secret <nombre> -o yaml > <nombre>.yml
```

* Aplicar la configuración basado en el archivo
```
kubectl create -f <nombre>.yml
```


## IMPORTANTE, si usas minicube a la hora de probar endpoints se hace el siguiente comando y ese es en lugar de localhost
```
minikube service client-gateway-service --url
```






## NO ALCANCE A LOS CREDITOS PARA PREPARAR EL KUBERNETES ENGINE ASI QUE AQUI LOS PASOS PARA DESPUES
1. entrar a los clusters de servicio creado y dar clic en connect y copiar el comando para conectar con gcloud
2. si da error de CRITICAL: auth-plugin te lleva a un enlace para instalar el plugin de auth, pero en caso omiso solo muestra que se creo
3. al ejecutar el comando de 
```
kubectl get confing get-contexts
```
muestra que esta seleccionando, pero si quieres regresar a estar de manera local se usa lo siguiente
```
kubectl config use-context <nombrecontexto>
```
## configurar GKE
1. revisar estar en la carpeta de tienda
2. hacer el siguiente comando
```
helm create tienda-gke
```
### nota: si hubo un error de levantamiento de configuracion hacer uso de describe o de logs para ver los mensajes de error que se muestran, esto puede pasar porque no se tomaron los valores de los secretos

3. exportar la config del anterior espacio de trabajo del desktop
```
kubectl config get-context //para ver el espacio de trabajo
kubectl confing use-context <nombre>
kubectl get secret auth-secrets -o yaml > auth-secret.yml //hacer esto para todas las config realizadas
``` 
# OJO, no hacer commit porque se genera el yaml con los secrets y puede exponerlos

4. regresar al contexto
```
kubectl config get-context //para ver el espacio de trabajo
kubectl confing use-context <nombre> //para regresar al contexto de google
```
5. crear los secrets en el entorno
```
kubectl create -f <nombre>.yml
```
6. borrar los archivos yaml, tener en cuenta que si se crea 2 carpetas hacer eliminacion de archivos innecesarios

## Borrar archivos innecesarios
1. abrir de donde se creo esa carpeta con las nuevas config y copiar el chart y ponerle el mismo nombre al de tienda
2. borrar esa carpeta y regresar a la consola y ver los pods 
3. acutalizar con helm
```
helm upgrade tienda-gke .
```
4. y revisar para confirmar que se haya desinstalado para ver que este bien

## BALANCEADORES DE CARGA - Ingress
1. En la parte de kubernetes engine en la seccion de Gateways, services & Ingress
2. configurar en la parte de yml en la carpeta de ingress creada en templates
3. usar el comando helm para actualizar la tienda
4. se deben de ver ahora en el ingress de googlecloud **se tarda un rato en reflejarse
5. al abrir la direccion ip debe mostrar el health-check creados //no corre con SSL
6. comprobar que funcione la ip dada para que muestre los mensajes creados
7. para activar el ssl usar la siguiente documentacion:

https://docs.cloud.google.com/kubernetes-engine/docs/how-to/managed-certs?hl=es-419
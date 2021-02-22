# wordpress-minikube
* References:https://armand.gr/blog/posts/kubernetes-helm-wordpress/
Tech used:

    1. minikube
    2. helm
    3. minio
    4. velera

Steps:

    1. With minikube already in place, deploy prometheus-stack helm chart
	
	helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --create-namespace -n monitoring

    2. Deploy minio server which we will use an alternate to AWS S3. Provide the ucket name and mino specific credentials in minio.yaml file. 

       helm install minio -f minio.yaml minio/minio --create-namespace -n velero
       
    3. Access minio service using below URL:
	minikube service --url minio -n velero
    4. Deploy velero for backups: 
        1. Valero needs credentials to access minio. We will save the credentials in a file and pass it.
        2. We will install velero while making it think that we are using AWS S3 even though we use Minio.
	velero install \
    		--provider aws \
    		--use-restic \
    		--plugins velero/velero-plugin-for-aws:v1.0.0 \
    		--bucket velero \
    		--secret-file ./minio.credentials \
    		--backup-location-config region=minio,s3ForcePathStyle=true,s3Url=$(minikube service --url minio -n velero) \
    		--snapshot-location-config region="default"
	
    5. Deploy Wordpress instances:
        1. We will deploy bitnami wordpress helm chart with 2 instances.
        2. Prepare wordpress.yaml file with custom values.
	helm install wordpress -f wordpress.yaml bitnami/wordpress --create-namespace -n wordpress
    6. Now get the URL for wordpress service by running below command:
	minikube service --url wordpress -n wordpress

    7. Monitoring using grafana dashboard: We can access the grafana on localhost on port 7000 by port forwarding. Login using the credentials provided during helm chart install.
	kubectl port-forward service/kube-prometheus-stack-grafana -n monitoring 7000:80  
    8. Backups: As already mentioned, we will use velero for backup of the services. 
	velero backup create wordpress --include-namespaces wordpress  

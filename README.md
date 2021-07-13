# wordpress

No diretório wordpress contem a versão do wordpress 4.8 / Mysql 5.6 testados no K8S.

No diretório wordpresslatest contem a versão do wordpress latest / Mysql 5.7 testados no K8S.

Para criar essa stack aplica o comando, seguindo os passos abaixo:

```
kubectl create -f arquivo.yml
```

## Para instalar do wordpress no K8S pode seguir os sequintes passos.

#### Criar um namaspace separado.

>Por que, separar?
>
>Questão de organização, segurança dos apps que irão rodar em seu cluster


#### Importando o arquivo de secrets.

>O arquivo de secrets guarda as informações de usuário, senha, host e banco de dados.
>Essas informações são configuradas na imagem docker quando foi feito seu build.
>DockerBuild [referências](https://docs.docker.com/engine/reference/builder/)

```
ex: MYSQL_ROOT_PASSWORD: dGV4dG8K
```
>Alguns containers tem suas variáveis de ambiente específicas, nesse caso adicionei as que irei precisar para o mysql e o wordpress.
>
>Usei uma conversão simples de base 64.
>
>Gerado da seguinte forma:

```
#echo “texto” | base64
#dGV4dG8K
```

#### Criando o Persitente Volume para o Banco de Dados.
>
> No K8S o volume persistente é formado por StoreageClass, PersistenteVolume, PersitenteVolumeClain, onde:
> A ordem de criação é importante para a correta utilização do recurso, vou passar uma por uma para didaticamente entender como funciona:
> link para [documentação](https://kubernetes.io/pt-br/docs/concepts/storage/persistent-volumes/).
> 
1. se cria o StorageClass:

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: mysql-wordpress
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer  # coloquei esse parâmetro aguardar o primeiro deploy do pod ser criado.
```
2. se cria o PersistenteVolume:

>Onde tem que se atentar ao tamanho, path e tipo e modo de acesso do volume.
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-wp-pv-volume
  labels:
    type: local
  namespace: wordpress  # Conferir o namaspace
spec:
  storageClassName: mysql-wordpress
  capacity:
    storage: 8Gi      # se atentar ao tamanho
  accessModes:
    - ReadWriteOnce     # modo de acesso apenas ao POD
  hostPath:
    path: "/mnt/data/wp4801"    # ponto de montagem
```

3. se cria o PersitenteVolumeClain:


```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-wp-pv-claim 
  namespace: wordpress  # Conferir o namaspace
spec:
  storageClassName: mysql-wordpress # Conferir StorageClass
  accessModes:
    - ReadWriteOnce  # modo de acesso apenas ao POD
  resources:
    requests:
      storage: 8Gi  # se atentar ao tamanho

```

#### Criando o deploy para o banco de dados.
>
>Para um Pod ser executado no kubernetes é preciso fazer um deploy (colocar em funcionamento um container, com um codigo desenvolvido e injetado dentro do container) é preciso criar um serviço onde vai expor as portas para acesso, interno ou externo do cluster. 
>No caso do banco de dados ele só vai ser visível internamente pelo Pod de aplicação do wordpress.
>

```
clusterIP: None
```
>
> As secrets para acesso ao mysql
> 
```
      env:
        - name: MYSQL_USER # Conferir nome da secret
          valueFrom:
            secretKeyRef:
              name: wordpress-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD # Conferir nome da secret
          valueFrom:
            secretKeyRef:
              name: wordpress-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: wordpress-secret
              key: MYSQL_DATABASE
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: wordpress-secret
              key: MYSQL_ROOT_PASSWORD

```
>
> O Volume que foi criado anteriomente.
> 
```
    volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-wp-pv-claim # Conferir nome do Volume.
```
#### Checando os logs do banco de dados.

>
> Com o comando abaixo vejo os logs do Pod onde o mysql está instalado.
```
kubectl logs -n wordpress mysql-69df96bbf7-p65bg
```
>
> Linhas do Log indicando que o Pod está online
> 
```
2021-07-08 10:51:50 1 [Note] Event Scheduler: Loaded 0 events
2021-07-08 10:51:50 1 [Note] mysqld: ready for connections.
Version: '5.6.51'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```
#### Acessando o banco de dados e verificação.

>
> Com o comando criamos um Pod cliente do mysql e acessamos o banco.
>
```
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -n wordpresslatest -- mysql -hmysql -umeuusuario -pminhasenha
```


#### Criando o Persistente Volume para o WordPress.

>
> Esse processo segue o mesmo modo do que já foi falado no topico anterior "Criando o Persitente Volume para o Banco de Dados".
>

#### Criando o deploy para o Wordpress.

>
> Esse processo segue o mesmo modo do que já foi falado no topico anterior "criando o deploy para o banco de dados".
>

#### Acessando e concluindo a instalação do Wordpress.
>
>Para acessar a aplicação temos que pegar a porta que gerada para o Pod
>
```
#kubectl get service -n wordpress

NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
mysql       ClusterIP   None             <none>        3306/TCP       5h1m
wordpress   NodePort    10.103.247.154   <none>        80:32560/TCP   4h57m

```
>
> O endereço CLUSTER-IP é o endereço interno do K8S, para acessar e terminar configuração pegasse o endereço da placa de rede do cluster com a porta.
> 
```
curl  http://192.168.2.1:32560

HTTP/1.1 200 OK
Date: Thu, 08 Jul 2021 15:57:55 GMT
Server: Apache/2.4.38 (Debian)
X-Powered-By: PHP/7.4.21
Link: <http://172.19.6.1:31888/wp-json/>; rel="https://api.w.org/"
Vary: Accept-Encoding
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8

```

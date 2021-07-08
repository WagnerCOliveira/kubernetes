# wordpress

No diretório wordpress contem a versão do wordpress 4.8 / Mysql 5.6 testados no K8S.


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
> A ordem de criação é importante para didaticamente entender como funciona:
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

#### criando o deploy para o banco de dados.

#### checando os logs do banco de dados.

#### acessando o banco de dados e verificação.

#### criando o Persistente Volume para o WordPress.

#### criando o deploy para o Wordpress.

#### acessando e concluindo a instalação do Wordpress.

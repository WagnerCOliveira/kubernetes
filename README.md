# kubernetes
Estudos e Deploy em kubernetes

## Como implantar wordpress em um cluster kubernetes k8s.

# Motivação:

	1. Aprender o funcionamento do engine do K8S.
	2. Montar os arquivos "yaml" necessários apartir de imagens prontas.
	3. Monitorar o kubernetes.
	4. CI/CD


# wordpress 4.8 / Mysql 5.6

  Logo dentro do diretório wordpress na branch master,
	
  A instalação do wordpress no K8S da sequinte forma:

	1. Criando um namaspace separado.
	2. importando o arquivo de secrets.
	3. Criando o Persitente Volume para o Banco de Dados.
	4. Criando o deploy para o banco de dados.
	5. Checando os logs do banco de dados.
	6. Acessando o banco de dados e verificação.
	7. Criando o Persistente Volume para o WordPress.
	8. Criando o deploy para o Wordpress.
	9. Acessando e concluindo a instalação do Wordpress.

# atividade_aws_linux
Atividade do estágio Compass.UOL
## Requisitos da atividade: 
## 1 ) Instância AWS:
   - Chave pública para acesso ao ambiente
   - Amazon Linux 2
   - t3.pequeno
   - SSD de 16GB
   - 1 Elastic IP associado à instância
   - Portas de comunicação liberadas
     - 22/TCP (SSH)
     - 111/TCP e UDP (RPC)
     - 2049/TCP/UDP (NFS)
     - 80/TCP (HTTP)
     - 43/TCP (HTTP)
## 2) Configurações Linux:
- Configurar o NFS entregue;
- Crie um diretório dentro do sistema de arquivos do NFS com seu nome;
- Subir um apache no servidor - o apache deve estar online e rodando;
- Crie um script que valide se o serviço está online e envie o resultado da validação para o seu diretório no nfs;
- O script deve conter - Dados HORA + nome do serviço + Status + mensagem personalizada de ONLINE ou offline;
- O script deve gerar 2 arquivos de saida: 1 para o serviço online e 1 para o serviço OFFLINE;
- Execução automatizada do script a cada 5 minutos.
  # 1 ) Instância AWS
  ### Criando pares de chaves
 - Na console aws perquise por `ec2`.
 - No menu erquerdo em "Rede e Segurança" selecione `Pares de chaves` > Criar Par de chave.
 - Crie um nome e selecione o fomato `.pem`.
 - Clique em "Criar Par de Chaves".
  ### Criando Grupo de Segurança 
  - No menu esquerdo em "rede e segurança" selecione `Security Groups` > Criar Grupo de segurança.
  - Dê um nome, uma descrição e selecione a VPC criada anteriormente.
  - Clique em "Adicionar Regras de Entrada".
  - Adicione as regras para liberar as portas de comunicação para acesso público: (22/TCP, 111/TCP e
UDP, 2049/TCP/UDP, 80/TCP, 443/TCP).

| Tipo              | Protocolo | Intervalo de portas | Origem     | Descrição |
|-------------------|-----------|----------------------|------------|-----------|
| SSH               | TCP       | 22                   | 0.0.0.0/0  | SSH       |
| TCP personalizado | TCP       | 80                   | 0.0.0.0/0  | HTTP      |
| TCP personalizado | TCP       | 443                  | 0.0.0.0/0  | HTTPS     |
| TCP personalizado | TCP       | 111                  | 0.0.0.0/0  | RPC       |
| UDP personalizado | UDP       | 111                  | 0.0.0.0/0  | RPC       |
| TCP personalizado | TCP       | 2049                 | 0.0.0.0/0  | NFS       |
| UDP personalizado | UDP       | 2049                 | 0.0.0.0/0  | NFS       |
  - Clique em "Criar Grupo de Segurança".
### Criando instância ec2
- No menu esquerdo clique em `Instâncias` > Executar Instâncias.
- Configure as tags para instâncias e volumes.
- Selecione a imagem "Amazon Linux 2 AMI(HVM), SSD Volume Type".
- Tipo de instancia "t3.small".
- (Será usada a VPC Padrão).
- 16 GB de armazenamento gp2(SSD).
- Selecione o par de chaves criado anteriormente.
- Em "Redes de Segurança" selecione a VPC criada e o Grupo de Gegunrança criado e configurado anteriormente.
- Clique em "Executar Instância".
### Criando e alocando ip elástico 
- No menu esquerdo em "rede e segurança" selecione `IPs Elásticos`> Alocar Indereço IP.
- Selecione o grupo de borda de rede "us-east-1".
- Clique em "Alocar" > Ações > associar ip elastico.
- Tipo de recurso "instância"
- Selecione a instância criada anteriomente
- Clique em "Associar"
  ### Criando sistema de arquivos EFS
  - Na console aws pesquise por `EFS`
  - No menu esquerto clique em `Sistema de arquivos` > Criar sistema de arquivos
  - Selecione a VPC Padrão
  - clique em "Criar "
   # 2) Configurações Linux:
  ### NFS:
  - Faça login remotamente no terminal via SSH
  - Crie um diretório para a montagem do NFS
  - instale os utilitarios EFS e NFS
    ```
    sudo yum install -y amazon-efs-utils
    sudo yum install nfs-utils
    sudo systemctl start nfs
    ```
  - No diretório de montagem monte o NFS com os comando :
   
    ```
    mount -t nfs <DNS_EFS> :/<diretórios-criado>
    ```
     - Para ter acesso ao DNS vá em EFS > Sistema de arquivo > Visualizar detalhes
   - Abra o arquivo `/etc/fstab` e ensira:
   ```
    <DNS>:/ /nfs nfs defaul 0 0
     
  ```
   - Crie um diretório dentro do NFS com seu nome.
    ### Apache :
   - Instale o Apache com:
     ```
     sudo yum attpd
     sudo systemctl start httpd
     sudo systemctl enable httpd
     ```
    ### Criando os Scripts
  - Digite o comando abaixo para criar um arquivo
    ```
     sudo nano nome-arquivo.sh
    
    ```
    
  - Insira os comandos :
 

   ```
    #! /bin/bash

    #dados por hora
    DADOS= $( data + ' %d/%m/%Y como %T ' )
 
    #status do serviço Apache
    STATUS= $( status systemctl httpd )

    #Diretório onde os arquivos de log serão armazenados
    SEU_DIRETORIO= " caminho para o diretório "

    #Condicional para verificar se o Apache está ativo ou não e imprimir o resultado nos arquivos
    #Se a saída da variável for igual a "active (running)", escreva que o Apache está online
    #Se não, escreva que o Apache está offline
    if [[ " ${STATUS} "  ==  * " ativo (em execução) " * ]] ;  então
       echo  " O Apache está online - ${DATA} "  >>  " ${SEU_DIRETORIO} /servico_online.txt "
   outro
       echo  " O Apache está offline - ${DATA} "  >>  " ${SEU_DIRETORIO} /servico_offline.txt "
   fi

  ```


  - Execute o comdo abaixo:

```

chmod +x nome-do-arquivo.sh

```


 - Na pasta com seu nome na NFS, escreva o comando:


```
chmod 775 /diretorio-criado/seu nome/*

```


- Agora para o script realizar a verificação e ser executado a cada 5 minutos
  - No diretório /etc abra o arquivo crontab e adicione:


   ```
   */5 * * * * bash /nome-do-script.sh
   ```



 Executando tudo, o script irá gerar uma verificação de status do Apache a cada 5 minutos, criará uma mensagem personalizada de Online e Offline, também irá informa a data , hora e o nome do serviço e tudo dentro de um arquivo.txt gerado no diretorio com seu nome

    

      

    

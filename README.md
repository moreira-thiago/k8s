# linuxtips-ansible
![ansible_k8s](https://miro.medium.com/max/570/0*3vW4ow1OBpXwn_KV.jpg)

Projeto para automação do provisionamento e deploy de um cluster Kubernetes com Helm na AWS utilizando Ansible e AWX.


## Configurando o ambiente antes de rodar os playbooks
1. Acesse sua conta AWS e crie um key_pair: AWS console > Services: EC2 > Network & Security: Key Pairs
2. Salve a chave recém criada na pasta `~/.ssh`
3. Crie um usuário com a policy AdministratorAccess, ou crie uma customizada e associe a este usuário: AWS Console > Services: IAM > Users: Add User
4. Baixe o arquivo .csv gerado no final
6. Instale o ansible e o pip:
    ``` shell
    # Debian family
    sudo apt install -y ansible python-pip
    ## Ubuntu 20.04 <---- Por algum motivo, sabe-se lá qual, o pacote python-pip não está disponível no repositório padrão
    curl https://bootstrap.pypa.io/get-pip.py --output get-pip.py && python2 /tmp/get-pip.py

    # Redhat family
    sudo yum install -y ansible python-pip

    # Archlinux family
    sudo pacman -S --noconfirm ansible python-pip
    ```
7. Instale o boto3 e awscli: 
    ``` shell
    sudo pip install boto3
    sudo pip install awscli
    ```
8. Configure suas credenciais da AWS no awscli:
    ``` shell
    # 1. Insira o valor de aws_access_key_id do arquivo .csv baixado durante a criação do seu usuário no IAM
    # 2. Insira o valor de aws_secret_access_key do arquivo .csv baixado durante a criação do seu usuário no IAM
    # 3. Informe a região da AWS em que você irá criar os recursos. Região utilizada neste projeto: us-east-1
    aws configure --profile default

    # Verifique se deu tudo certo: Irá retornar um JSON com os dados de acesso do teu usuário. Pressione CTRL+c para sair do comando.
    aws iam list-access-keys
    ```

## Executando os playbooks
1. Acesse a pasta *provisioning*, e crie as instâncias EC2: ```ansible-playbook -i hosts main.yml```
2. Acesse a pasta *install_k8s*, e instale o docker e k8s: ```ansible-playbook -i hosts main.yml```
3. Acesse a pasta *deploy_app_v1*, e instale o docker e k8s: ```ansible-playbook -i hosts main.yml```
4. Pegue o IP Público de uma das 3 instâncias EC2 criadas e acesse os endereços: IP_DO_SERVER:32222, IP_DO_SERVER:32111 e IP_DO_SERVER:32111/metrics
5. Acesse a pasta *deploy_app_v2*, e instale o docker e k8s: ```ansible-playbook -i hosts main.yml```
6. Pegue o IP Público de uma das 3 instâncias EC2 criadas e acesse os endereços: IP_DO_SERVER:32222, IP_DO_SERVER:32111 e IP_DO_SERVER:32111/metrics


## Fases do projeto
- [x] provisioning => Criação das instâncias para o cluster
    ```
    - Criação do Security Group
    - Criação de 3 instâncias EC2 do tipo t2.medium, com base na AMI ami-0dba2cb6798deb6d8(Ubuntu 20.04)
    - Arquivo hosts será populado com os IPs públicos e privados das instâncias EC2 criadas
    ```

- [x] install_k8s => Criação do cluster
    ```
    - Instalação do docker, kubeadm, kubelet e kubeadm nas 3 instâncias EC2 recém criadas
    - Criação do cluster
    - Associação dos workers ao nó master
    - Instalação do Helm
    - Deploy do Prometheus Operator
    ```


## Dicas
- Desabilitar mensagens de warnings e necessidade de confirmação para adicionar novos servers ao seu `~/.ssh/known_hosts`
    ``` conf
    # Altere os parâmetros do arquivo /etc/ansible/ansible.cfg conforme abaixo
    host_key_checking = False
    deprecation_warnings = False
    ```
- Acessando instâncias EC2 via SSH
    ``` shell
    ssh-agent zsh
    ssh-add ~/.ssh/ansible_k8s.pem
    ssh ubuntu@IP_DO_SERVER
    ```

- As instâncias EC2 do tipo t3.medium não fazem parte do "plano gratuito de testes"(free tier) da AWS, portanto não se esqueça de dar um `terminate` nas instâncias e remover o Security Group após terminar os testes neste projeto, caso contrário será cobrado pelo tempo utilizado, cerca de 0.0464 por hora na região da Virgínia.

- Queira ou não, você será cobrado pelo tempo utilizado das instâncias, mas é um valor ínfimo. Durante a implementação deste projeto, pelo meu dashboard de billing, foram computadas 14.307 horas, o que me custo $0.66, pelo menos até hoje: 19/10/2020 às 13:12

# Documentação Passo a Passo Da Atividade Prática

## Configuração de Ambiente AWS e Linux

Esta documentação descreve o processo de configuração de um ambiente AWS e a instalação e configuração do NFS, Apache, e um script de validação em uma instância Amazon Linux 2.

### **Passo 1: Gerar uma Chave Pública para Acesso ao Ambiente AWS**

Para acessar a instância EC2, você precisará de uma chave SSH. Siga estas etapas:

1. Acesse o [Console de Gerenciamento da AWS](https://aws.amazon.com/).

2. No painel de serviços, clique em "EC2" para acessar o serviço EC2.

3. Clique em "Pares de chaves" para criar uma nova Chave SSH.

4. Selecione "Criar par de chaves".

5. Escolha o nome e clique em "Criar par de chaves".

### **Passo 2: Criar uma Instância EC2**

Agora, vamos criar uma instância EC2 com o sistema operacional Amazon Linux 2.

1. Acesse o [Console de Gerenciamento da AWS](https://aws.amazon.com/).

2. No painel de serviços, clique em "EC2" para acessar o serviço EC2.

3. Clique em "Launch Instances" para criar uma nova instância.

4. Escolha a imagem "Amazon Linux 2 AMI" e clique em "Select".

5. Escolha o tipo de instância "t3.small" e clique em "Next: Configure Instance Details".

6. Deixe as configurações padrão e clique em "Next: Add Storage".

7. Configure o armazenamento conforme necessário e clique em "Next: Add Tags".

8. Adicione tags (opcional) e clique em "Next: Configure Security Group".

9. Configure as regras de segurança para permitir tráfego nas portas desejadas (por exemplo, 22/TCP, 111/TCP e UDP, 2049/TCP/UDP, 80/TCP, 443/TCP).

10. Revise as configurações e clique em "Launch".

11. Escolha uma chave SSH existente ou selecione a chave que você gerou no Passo 1 e clique em "Launch Instances".

12. Sua instância EC2 está em execução. Anote o "IPv4 Public IP" para uso posterior.

### **Passo 3: Gerar um Elastic IP e Anexar à Instância EC2**

Para garantir que seu endereço IP público seja persistente, você pode criar um Elastic IP e associá-lo à sua instância.

1. No painel EC2, navegue até "Elastic IPs".

2. Clique em "Allocate new address" e depois em "Allocate".

3. Selecione o Elastic IP criado e clique em "Actions" > "Associate IP Address".

4. Escolha a instância EC2 que você criou e clique em "Associate".

### **Passo 4: Liberar as Portas de Comunicação para Acesso Público**

As regras de segurança do grupo associado à sua instância determinam quais portas estão abertas.

1. No painel EC2, navegue até "Security Groups" e selecione o grupo associado à sua instância.

2. Clique na guia "Inbound Rules" e adicione regras para permitir o tráfego nas portas necessárias, por exemplo:
   - SSH (22/TCP)
   - RPC (111/TCP/UDP)
   - NFS (2049/TCP/UDP)
   - HTTP (80/TCP)
   - HTTPS (443/TCP)

### **Passo 5: Configurar o NFS**

Agora, vamos configurar o servidor NFS na instância EC2.

1. Acesse sua instância EC2 via SSH usando a chave privada que você gerou no Passo 1:
   ```bash
   ssh -i /mnt/Chave_SSH.pem ec2-user@3.218.11.33
   ```

2. Instale o servidor NFS:
   ```bash
   sudo yum install nfs-utils
   ```

3. Abra o arquivo de configuração das exportações NFS:
   ```bash
   sudo nano /etc/exports
   ```

4. Adicione uma linha para exportar um diretório. Por exemplo, para exportar `/var/www/html`:
   ```
   /var/www/html *(rw,sync,no_root_squash)
   ```

   - `/var/www/html` é o diretório que você deseja compartilhar.
   - `*` permite que qualquer cliente acesse.
   - `rw` permite leitura e escrita.
   - `sync` sincroniza as operações de escrita.
   - `no_root_squash` permite que o root acesse.

5. Salve o arquivo e saia.

6. Inicie o serviço NFS:
   ```bash
   sudo systemctl start nfs-server
   ```

7. Certifique-se de que o serviço NFS seja iniciado automaticamente na inicialização:
   ```bash
   sudo systemctl enable nfs-server
   ```

### **Passo 6: Criar um Diretório no Filesystem do NFS**

Agora, vamos criar um diretório dentro do sistema de arquivos NFS.

1. Na sua instância EC2, execute o seguinte comando para criar um diretório com seu nome:
   ```bash
   sudo mkdir /var/www/html/Carlos
   ```
### **Passo 7: Subir o Apache**

Agora, vamos instalar e configurar o servidor Apache.

1. Instale o servidor Apache:
   ```bash
   sudo yum install httpd
   ```

2. Inicie o serviço do Apache:
   ```bash
   sudo systemctl start httpd
   ```

3. Certifique-se de que o Apache seja iniciado automaticamente na inicialização:
   ```bash
   sudo systemctl enable httpd
   ```

### **Passo 8: Criar um Script de Validação**

Agora, vamos criar um script que validará o status do serviço Apache e registrará as informações.

1. Crie um arquivo chamado `clock_check.sh` no diretório inicial do usuário:
   ```bash
   touch ~/clock_check.sh
   ```

2. Abra o script com um editor de texto, por exemplo, o Nano:
   ```bash
   nano ~/clock_check.sh
   ```

3. Adicione o seguinte código ao script:
   ```bash
   #!/bin/bash

   data=$(date +"%Y-%d-%m %T")
   servico="Apache"
   status=$(systemctl is-active httpd)
   menssagem=""

   if [ "$status" == "active" ]; then
     menssagem="Online"
   else
     menssagem="Offline"
   fi

   echo "$data - $servico - Status: $status - $menssagem" >> /var/www/html/Carlos/servico.txt
   ```

4. Salve o arquivo e saia.

5. Dê permissões de execução ao script:
   ```bash
   chmod +x ~/clock_check.sh
   ```

### **Passo 9: Execução Automatizada do Script a Cada 5 Minutos**

Agora, vamos agendar a execução automática do script usando a tabela cron.

1. Abra a tabela cron para edição:
   ```bash
   crontab -e
   ```

2. Adicione a seguinte linha para executar o script a cada 5 minutos:
   ```bash
   */5 * * * * ~/clock_check.sh
   ```

3. Salve o arquivo e saia.

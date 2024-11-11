# Guia Completo para Configuração de Buckets S3 e Deploy de API em EC2 na AWS - Marcos Martins

## Índice

1. [Links](#links)
2. [Aos Instrutores](#aos-instrutores)
3. [Passo a Passo para Configurar o Token de Autenticação Globalmente no Postman](#passo-a-passo-para-configurar-o-token-de-autenticação-globalmente-no-postman)
4. [Como Subir um Arquivo para o Amazon S3](#como-subir-um-arquivo-para-o-amazon-s3)
   - [1. Criar um Bucket](#1-criar-um-bucket)
   - [2. Carregar Arquivos para o Bucket](#2-carregar-arquivos-para-o-bucket)
   - [3. Obter e Testar a URL Pública](#3-obter-e-testar-a-url-pública)
5. [Criando uma Instância EC2](#criando-uma-instância-ec2)
   - [1. Criar uma VPC](#1-criar-uma-vpc)
   - [2. Criar o Security Group](#2-criar-o-security-group)
   - [3. Gerar uma Chave SSH](#3-gerar-uma-chave-ssh)
   - [4. Importar a Chave SSH na AWS](#4-importar-a-chave-ssh-na-aws)
   - [5. Configurar a Instância EC2](#5-configurar-a-instância-ec2)
6. [Deploy da API](#deploy-da-api)
   - [1. Conectar-se à Instância EC2](#1-conectar-se-à-instância-ec2)
   - [2. Deploy](#2-deploy)
7. [Referências](#referências)

---

## Links

- documentação Swagger no S3:
  https://amzn-s3-swagger-compass-car-docs.s3.us-east-2.amazonaws.com/swagger-API-documentation.html

- Link para acessar este documento no README para melhor visualização:
  https://github.com/Marcos-m97/Guia-para-Configura-o-de-Buckets-S3-e-Deploy-de-API-em-EC2-na-AWS/blob/main/README.md


## Aos Instrutores

- **Credenciais da Seed (usuário admin)** para inserir na rota de login e testar a API:

  ```json
  {
    "email": "compasspb@compass.com",
    "password": "ABC123456"
  }
  ```

- A rota de login é publica [Post]: http://ec2-3-147-63-136.us-east-2.compute.amazonaws.com:8080/api/v1/login

  As demais rotas são privadas (protegidas pelo middlwere JWT): para utilizar estas rotas é necessario preencher no campo **headers** do postman e inserir `key: **Authorization**  value: **Bearer <token>**`
  onde <token> deve ser substituido pelo token gerado ao enviar uma requisição com "email" e "password" validos na rota de login.

- Rotas privadas:
  (Cada endpoint listado abaixo oferece suporte aos métodos HTTP de CRUD (GET, POST, PUT, DELETE), conforme detalhado na documentação da API)

- Usuarios: http://ec2-3-147-63-136.us-east-2.compute.amazonaws.com:8080/api/v1/users
- Clientes: http://ec2-3-147-63-136.us-east-2.compute.amazonaws.com:8080/api/v1/clientes
- Carros: http://ec2-3-147-63-136.us-east-2.compute.amazonaws.com:8080/api/v1/carros
- Pedidos: http://ec2-3-147-63-136.us-east-2.compute.amazonaws.com:8080/api/v1/pedidos

- O passo anterior para acessar as rotas privadas deve ser feito em cada endpoint da API, como alternativa, segue um passo a passo de como otimizar este processo para facilitar o teste em todos os endpoints:

### Passo a Passo para Configurar o Token de Autenticação Globalmente no Postman

- Criar ou Editar um Ambiente no Postman:
- No Postman, clique em **Environments** no canto superior direito.
- Selecione um ambiente existente ou crie um novo clicando em **+ Add**.
- Adicione uma nova variável chamada `authToken`:
  - **Nome:** `authToken`
  - **Valor Inicial:** insira o token JWT (ex.: `eyJhbGciOiJIUzI1NiIsInR5cCI...`)
- Clique em **Save** para salvar o ambiente.

- Configurar a Autenticação na Coleção:
- Na barra lateral esquerda, localize a coleção onde estão as rotas da sua API.
- Clique nos três pontos ao lado do nome da coleção e selecione **Edit**.
- Vá até a aba **Authorization**.
  - No campo **Type**, selecione **Bearer Token**.
  - No campo **Token**, insira `{{authToken}}` para usar o valor da variável definida no ambiente.
- Clique em **Save** para salvar essa configuração para toda a coleção.

- Desativar a Autenticação para a Rota de Login:
- Ainda na barra lateral esquerda, selecione a requisição de **Login** dentro da coleção.
- Vá até a aba **Authorization** dessa requisição específica.
  - No campo **Type**, selecione **No Auth**.
  - Isso irá sobrescrever a configuração da coleção para esta requisição, garantindo que a rota de login não envie o token.
- Salve a requisição.

- Selecionar o Ambiente e Testar:
- No canto superior direito do Postman, selecione o ambiente onde você definiu a variável `authToken`.
- Agora, todas as requisições da coleção usarão automaticamente o token salvo no ambiente, exceto a de login.

- Atualizar o Token Quando Expirar:
- Quando o token expirar, basta ir novamente em **Environments**.
- Localize o ambiente e atualize o valor da variável `authToken` com o novo token JWT.
- Clique em **Save**. Todas as requisições que utilizam `{{authToken}}` irão automaticamente usar o novo valor.

Com esses passos, o token é aplicado globalmente a todas as rotas, exceto na rota de login.

# Como subir um arquivo Para o Amazon S3:

## 1 Criar um bucket

### Selecionar a Região

Selecione a região de Ohio - `us-east-2`.

### Acesse o Console do S3:

Clique em **Create Bucket**.

### Configure as Opções do Bucket:

#### Campo 1 - Configuração Geral

- **Nome do Bucket**: Escolha um nome único e descritivo (exemplo: `compass-car-swagger-docs`).
  > O nome deve ser exclusivo no namespace global e seguir as regras de nomenclatura do S3.
  > Aqui é possível escolher um bucket criado anteriormente para copiar as configurações (caso tenha um).

#### Campo 2 - Propriedades do Objeto

> A **Propriedade de objetos do Amazon S3** permite definir quem possui os objetos enviados para o bucket e controlar o uso de ACLs (listas de controle de acesso). Quando ACLs estão desativadas, o proprietário do bucket possui todos os objetos, e o acesso é gerenciado apenas com políticas de bucket.

1. **Permitir Acesso Público**

   - Confirme que deseja **permitir o acesso público**, pois o bucket será usado para hospedar um arquivo público.

2. **Escolher ACLs: Manter Desabilitadas ou Habilitar ACLs**

   ### Opção 1: Manter ACLs **Desabilitadas**

   - Apenas políticas de bucket (em JSON) poderão ser usadas para tornar os arquivos públicos.

   - **Instruções para configurar a política de bucket**:

     - Após finalizar a criação do bucket, acesse a aba **Permissões** do bucket.
     - Em **Política de bucket**, insira a política JSON abaixo:

       ```json
       {
         "Version": "2012-10-17",
         "Statement": [
           {
             "Effect": "Allow",
             "Principal": "*",
             "Action": "s3:GetObject",
             "Resource": "arn:aws:s3:::nome-do-seu-bucket/*"
           }
         ]
       }
       ```

       > **Explicação das chaves JSON**:
       >
       > - `"Effect": "Allow"` — Permite o acesso ao recurso especificado.
       > - `"Principal": "*"` — Permite acesso para todos (acesso público).
       > - `"Action": "s3:GetObject"` — Permite o acesso de leitura aos objetos do bucket.
       > - `"Resource": "arn:aws:s3:::nome-do-seu-bucket/*"` — Define o bucket e todos os objetos dentro dele como recursos alvo.

   ### Opção 2: **Habilitar ACLs**

   - Ao habilitar ACLs, você poderá configurar o acesso público diretamente para cada arquivo.

   - **Instruções para configurar ACLs no bucket**:
   - No bucket, acesse a aba **Permissões**.
   - Clique em **Lista de controle de acesso (ACL)** e edite as permissões, concedendo **Leitura** para todas as opções, mantendo as permissões completas para o proprietário do bucket.

    **Definir Permissões de Acesso Público ao Fazer Upload**:
   - Na aba **Permissões** do upload, selecione a opção **Conceder acesso público de leitura**.
   - Um aviso será exibido com a mensagem:
   - "Qualquer pessoa no mundo poderá acessar os objetos especificados…"
   - Marque o checkbox:
   - "Compreendo o risco de conceder acesso público de leitura aos objetos especificados".

   - **Alternativa para tornar o arquivo público após o upload**:
     - Acesse o bucket no S3.
     - Selecione o arquivo desejado, clique em **Ações** e escolha **Tornar público usando ACLs**.

3. Escolha uma das opções de propriedade dos objetos do bucket

> **Nota**: Para o caso do desafio, a opção recomendada é **Proprietário do Bucket Preferido**.

- **Proprietário do Bucket Preferido**

  - Se novos objetos enviados para este bucket especificarem a ACL pré-configurada `"bucket-owner-full-control"`, eles pertencerão ao proprietário do bucket.
  - Caso contrário, os objetos pertencerão ao usuário que os enviou.

- **Autor do Objeto**
  - O autor do objeto permanece como o proprietário, o que significa que cada usuário que faz upload de um arquivo mantém o controle individual sobre ele.

> Recomendações:
> Marque **Proprietário do Bucket Preferido** para simplificar o gerenciamento e garantir que o proprietário do bucket mantenha controle total sobre os objetos.
> Marque **Autor do Objeto** apenas se for necessário que cada usuário mantenha controle individual sobre seus próprios uploads — isso geralmente não é necessário para buckets públicos ou de uso geral.

#### Campo 3 - Configurações de Bloqueio do Acesso Público

- Desmarque todas as opções para tornar o bucket público.

#### Campo 4 - Versionamento de Bucket

- Ative o **versionamento** para manter múltiplas versões de um objeto no mesmo bucket.

#### Campo 5 - Criptografia

- Selecione **SSE-S3** para criptografia no lado do servidor com chaves gerenciadas pelo Amazon S3.

> O S3 gerencia as chaves de criptografia automaticamente É uma opção básica e econômica, aplicada a todos os objetos no bucket sem configuração adicional de chaves. Ideal se você quer criptografia sem necessidade de gerenciar chaves.

- Chave do bucket Selecione **Desativar**

> O uso de uma chave de bucket do S3 para SSE-KMS reduz os custos de criptografia ao diminuir as chamadas para o AWS KMS. As chaves de bucket do S3 não são compatíveis com o DSSE-KMS.

### Finalize a Criação do Bucket

- Revise as configurações e clique em **Create bucket**.

## 3. Carregar Arquivos para o Bucket

### Acesse o Bucket Criado

- Clique no nome do bucket para acessar suas configurações.

### Iniciar o Upload do Arquivo

- Clique em **Upload** e depois em **Add files**.
- Selecione, em seu computador, o arquivo que sera carrregado.
- Se optou por **habilitar ACLs**, escolha **Conceder acesso público de leitura** na aba **Permissões**.
- Clique em **Upload** para iniciar o envio do arquivo ao bucket.

## 4. Obter e Testar a URL Pública

### Copiar a URL do Objeto

- Clique no arquivo carregado e copie a **URL**.
- Cole a URL em um navegador para verificar se o HTML está acessível publicamente.

## Processo finalizado:

### Resumo:

- Crie bucket com acesso público.
- Desmarcar o bloqueio de acesso público.
- Fazer upload do arquivo HTML e tornar o arquivo público (usando ACLs).
- Testar a URL para garantir que o arquivo está acessível.

# Criando uma Instância EC2

## 1. Criar uma VPC (Virtual Private Cloud)

> Uma VPC define o ambiente de rede onde a instância opera, fornecendo isolamento e controle sobre a comunicação com outros recursos.
> Na criação da VPC, define-se:
> Sub-redes (públicas ou privadas) para organizar e isolar os recursos.
> Segurança usando grupos de segurança e listas de controle de acesso (NACLs).
> Rotas e Gateways para controlar o tráfego de entrada e saída

### instruções:

- No console da AWS, acesse **VPC**.

- Clique em **Criar VPC** e selecione **VPC e muito mais**.

  > Esta opção permite editar os recursos da VPC

- Marcar o campo de **Geração automática da etiqueta de nome** e definir um nome para as etiquetas da VPC

  > O nome será usado para gerar automaticamente etiquetas de Nome para todos os recursos na VPC.

- Configure os campos: Bloco CIDR IPv4, Número de zonas de disponibilidade, Número de sub-redes públicas, Número de sub-redes privadas:

- **Bloco CIDR IPv4**

  - Deixe a opção padrão: `10.0.0.0/16`.

- **Bloco CIDR IPv6**

  - Deixe a opção: **Nenhum bloco CIDR IPv6**.

- **Locação**

  - Mantenha a opção **Padrão**.

- **Número de zonas de disponibilidade (AZs)**

  - Mantenha a opção **2**.
    > Escolha o número de AZs onde as sub-redes serão provisionadas.

- **Número de sub-redes públicas**

  - Mantenha a opção **2**.
    > O número de sub-redes públicas que serão adicionadas à VPC. Use sub-redes públicas para aplicações web que precisam estar acessíveis publicamente.

- **Número de sub-redes privadas**

  - Mantenha a opção **2**.
    > O número de sub-redes privadas na VPC. Use sub-redes privadas para proteger recursos que não precisam de acesso público.

- **Gateways NAT (USD)**

  - Mantenha a opção **Nenhuma**.
    > Escolha o número de zonas de disponibilidade (AZs) onde criar gateways NAT. Há cobrança para cada gateway NAT.

- **Endpoints da VPC**
  - Mantenha a opção **Gateway do S3**.
    > Endpoints podem reduzir cobranças do gateway NAT e melhorar a segurança ao acessar o S3 diretamente da VPC.
- **Opções de DNS**
  - Marque as duas caixas:
    - **Habilitar nomes de host DNS**
    - **Habilitar resolução de DNS**

Clique em **Criar VPC** para finalizar as configurações.

## 2. Criar o Security Group

> Um **Security Group** age como um firewall virtual para suas instâncias EC2, controlando o tráfego de entrada e saída.  
> Regras de entrada controlam o tráfego que entra na instância, e regras de saída controlam o tráfego que sai dela.  
> Quando uma instância é lançada, você pode especificar um ou mais Security Groups. Se nenhum for especificado, o EC2 usa o Security Group padrão da VPC.

#### instruções:

- Acesse a aba **Instâncias**.

  - No painel lateral, encontre a seção **Rede e Segurança** e clique em **Security Groups**.

- Clique em **Criar Grupo de Segurança**.

- Configure os **Detalhes Básicos**:

  - **Nome do Grupo de Segurança**: Defina um nome (não pode ser alterado após a criação).
  - **Descrição**: Insira uma breve descrição, como `[Permitir acesso SSH aos devs]`.

- Selecione a VPC criada anteriormente.

- Defina as **Regras de Entrada e Saída**:

  > **Nota**: Grupos de segurança permitem toda a saída de dados por padrão, mas não permitem a entrada. Por isso, é importante definir as regras de entrada.

  - **Regras de Entrada**:

    - **Conexão SSH**: Permite que um desenvolvedor se conecte e manipule a API.

      - Clique em **Adicionar Regra** e defina:
        - **Tipo**: `SSH`
        - **Porta**: `22` (preenchido automaticamente)
        - **Origem**: `Qualquer IPV4 (0.0.0.0/0)`

    - **Conexão para a API (TCP personalizado)**:
      - Clique em **Adicionar Regra** e defina:
        - **Tipo**: `TCP personalizado`
        - **Porta**: Defina a porta usada pela API (exemplo: `8080`).
          > Certifique-se de que corresponde à porta configurada no servidor (como no Express).
        - **Origem**: `Qualquer IPV4 (0.0.0.0/0)`
          > Isso permite que todos os endereços IP acessem a API publicamente.

  - **Regras de Saída**:
    - **Tipo**: Todo o tráfego
    - **Origem**: Qualquer IPV4

- Clique em **Criar Grupo de Segurança** para finalizar.

## 3. Gerar uma Chave SSH (Windows 10)

> O Windows 10 já vem com o cliente SSH instalado, o que permite gerar uma chave SSH para autenticação e conexão segura entre seu computador e uma instância EC2 na AWS.

Uma chave SSH é composta por um par de chaves:

- **Chave Privada**: Fica armazenada localmente e nunca deve ser compartilhada.
- **Chave Pública**: É enviada para a instância EC2 e permite que o servidor reconheça quem possui a chave privada correspondente.

#### Passos para Gerar a Chave SSH no PowerShell

- Inicie o **PowerShell** como administrador.

- Execute os seguintes comandos:

  - **Acessar o SSH**:
    ssh

  - **Gerar a Chave SSH**:
    ssh-keygen

    > Durante o processo, você será solicitado a definir uma senha ou deixar o campo em branco para prosseguir sem senha.

  - **Definir Pasta de Armazenamento**:

    - Escolha uma pasta para armazenar as chaves. O sistema sugerirá uma pasta padrão (geralmente em `C:\Users\User\.ssh`).

  - **Listar as Chaves**:
    dir ~/.ssh

  - **Copiar a Chave Pública**:
    - Copie a chave pública para configurar na instância EC2:
      Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
      > Substitua `~/.ssh/id_ed25519` pelo caminho da chave gerada pelo comando `ssh-keygen`.

#### Exemplo de Caminho para Salvar as Chaves

Se utilizar o caminho sugerido, as chaves serão armazenadas, por exemplo em: C:\Users\User.ssh

## 4. Importar a Chave SSH na AWS

- No console da AWS, acesse o painel lateral e vá para **Rede e Segurança** > **Pares de Chaves**.

- Clique em **Ações** e selecione **Importar par de chaves**.

- Preencha o campo **Configurações de importação**:

  - **Nome**: Defina um nome para a chave (por exemplo: `minha-ssh`).
  - Escolha uma das opções para importar a chave:

    - **Carregar o arquivo** onde a chave pública foi salva.
    - Ou **colar o conteúdo copiado** do comando abaixo:

      Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard

- **Exemplo de Chave Pública**:
  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIKw2M7ApKjAoYh3PQeXYQbust4BmjyT4O5OLbDbpU92 user@DESKTOP-R09II9I

## 5. Configurar a Instância EC2

> Uma instância do Amazon EC2 é um servidor virtual na nuvem que você pode configurar para executar aplicações, hospedar websites ou gerenciar dados de forma flexível e escalável.  
> Ela permite escolher o sistema operacional, CPU, memória, armazenamento e configurações de rede, além de ser acessível remotamente via SSH.

#### instruções

- Acesse a página do **EC2** no console da AWS e clique em **Executar Instância**.

- Configure os **Nomes e Tags**:

  - Defina tags de acordo com os atributos da política (ABAC) atribuída às contas do **Scholarship Program**.
    - `Chave`: **Name** | `Valor`: **teste** | `Tipo de Recurso`: **Instâncias e Volumes**
    - `Chave`: **CostCenter** | `Valor`: **teste** | `Tipo de Recurso`: **Instâncias e Volumes**
    - `Chave`: **Project** | `Valor`: **teste** | `Tipo de Recurso`: **Instâncias e Volumes**

- Selecione a **Imagem de Aplicação e de Sistema Operacional (Imagem de Máquina da Amazon - AMI)**:

  - Utilize a **Amazon Linux 2023** (qualificada para o nível gratuito).
    > A AMI é um modelo com a configuração de software (sistema operacional, servidor de aplicações e aplicações) necessário para executar a instância.

- No campo **Tipo de Instância**:

  - Selecione **t2.micro**.
    > `t2.micro` é projetado para workloads de baixa a moderada intensidade e oferece 1 vCPU e 1 GB de memória.

- Na seção **Par de Chaves**:

  - Selecione o par de chaves criado anteriormente (por exemplo, **minha-ssh**).

- Na seção **Configurações de Rede**:

  - Clique em **Editar** e configure:
    - **VPC**: Selecione a VPC criada anteriormente.
    - **Sub-rede**: Selecione a sub-rede pública 1 (exemplo: `Projeto-subnet-public1-us-east2A`).
    - **Atribuir IP Público**: Habilite esta opção.
    - **Firewall (grupos de segurança)**: Selecione **Grupos de segurança existentes** e escolha o Security Group criado anteriormente.

- Clique em **Executar Instância** para finalizar a configuração.

Após a criação, será gerado o ID de uma instância EC2.  
**Exemplo de ID**: `i-0ea4c40d1ba324149`

# Deploy da API

## 1. Conectar-se à Instância EC2

### instruções:

- Acesse a página inicial do **EC2**.

- No campo **Recursos**, clique em **Instâncias em execução**.

- Selecione a instância criada anteriormente e clique em **Conectar**.

- Na janela de conexão:

  - Selecione a aba **Cliente SSH**.
  - Copie o valor da **DNS pública** da instância.
    > **Exemplo de DNS Pública**: `ec2-3-147-63-136.us-east-2.compute.amazonaws.com`

- Copie o comando SSH fornecido no final da janela, pois será usado para conectar-se via **PowerShell**.

  - **Exemplo de Comando**:
    `ssh -i "C:\Users\User\.ssh\id_ed25519" ec2-user@ec2-3-147-63-136.us-east-2.compute.amazonaws.com`
    > O comando inclui o caminho e o nome da chave privada e a DNS pública da instância.

- No **PowerShell**, cole e execute o comando copiado para iniciar a conexão com a instância EC2:

  - Ao executar o comando pela primeira vez, o terminal solicitará a confirmação da conexão. Digite **Yes** para aceitar.
  - Caso tenha definido uma senha ao gerar a chave SSH, insira-a quando solicitado.

-   Se a conexão for fechada na primeira tentativa, insira o comando novamente para garantir uma conexão bem-sucedida.

- Verifique se o sistema operacional da instância precisa de atualizações com o comando:
  `sudo yum update -y`

## 2. Deploy

#### instruções:

- Instalar o Git
  `sudo yum install -y git`

- Instalar o Node.js
  `sudo yum install -y nodejs`

> Verificar a instalação do Node.js e npm:
> `node -v` Mostra a versão do Node.
> `npm -v` Mostra a versão do npm

- Checar se a porta configurada na API corresponde à porta configurada na instância EC2 (8080)

- Clonar o repositório da API do GitHub
  `git clone <URL-do-repositorio>`

> Observação: Para clonar o repositório, pode ser necessário gerar um token no GitHub e copiar o código do token no comando:
> Exemplo: git clone https://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@github.com/Marcos-m97/NODE-AWS-PB-compass-desafio-3.git

- Verificar se atualizaçoes foram feitas no codigo da API
  `git pull origin main`

- Acessar o diretório do repositório clonado
  `cd <minha-api>`

  > ATENÇÃO: Quando a conexão SSH for encerrada, será necessário acessar o repositório novamente
  > Exemplo: `cd NODE-AWS-PB-compass-desafio-3`

- Instalar as dependências do projeto
  `npm install`

- Transpilar o código TypeScript para JavaScript
  `npx tsc`

- Iniciar a aplicação
  `npm start` (script configurado no package.json para iniciar a aplicação)

- Instalar o PM2 para manter a API funcionando mesmo após encerrar a conexão SSH:

  > Utilizar "sudo" como prefixo devido a permissões de admin no Linux
  > `sudo npm install -g pm2`

- (Opcional) Configurar o PM2 para reiniciar a API automaticamente caso a instância EC2 seja reiniciada
  `pm2 startup`

  > O comando `pm2 startup` gera um comando que deve ser copiado e colado no terminal, exemplo:
  > Exemplo: `sudo env PATH=$PATH:/usr/bin /usr/local/lib/node_modules/pm2/bin/pm2 startup systemd -u ec2-user --hp /home/ec2-user`

- Rodar a aplicação com o PM2
  `pm2 start dist/index.js --name "minha-api"`

- Concluir o startup com o comando save
  `pm2 save`

> Outra opção é apenas o comando direto, mas caso a instância seja interrompida ou reiniciada, será necessário reiniciar o PM2 manualmente: `pm2 start dist/index.js --name "minha-api"`

> Caso tenha problema ao reiniciar o servidor após encerrar a conexão SSH ("Error: listen EADDRINUSE: address already in use :::8080"),
> buscar o processo que ainda está rodando: `sudo lsof -i :8080`
> encerrar o processo: `sudo kill -9 39158 (39158 é o PID do processo de exemplo)`

# Referencias:

https://israelbarberino-dev.notion.site/VPC-Virtual-Private-Cloud-12ea01dcbda18000bd5aee45e22568ad
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html?icmpid=docs_ec2_console
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-launch-tutorials.html
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/tutorial-launch-my-first-ec2-instance.html
https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

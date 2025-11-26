# ☁️ Configurando e Autenticando o AWS ECR

Usar um `Container Registry` na nuvem, como o `AWS ECR`, exige um passo de autenticação prévia, pois ele é um serviço privado. Diferente do `Docker Hub` (que você geralmente autentica uma vez com seu usuário Docker), o `ECR` usa as credenciais da `AWS` para gerenciar o acesso, garantindo a segurança empresarial.

Para interagir com o `ECR`, precisaremos da `**AWS CLI (Command Line Interface)**`.

## 1. ⚙️ Instalação e Configuração da AWS CLI

A `AWS CLI` é a ferramenta de linha de comando oficial da `Amazon Web Services`. Ela permite que você gerencie e interaja com quase todos os serviços da `AWS`, incluindo o `ECR`, diretamente do seu terminal.

### 1.1. Instalação

Você deve primeiro instalar a AWS CLI na sua máquina. O processo varia dependendo do seu sistema operacional (Linux, macOS ou Windows).

> Para instruções de instalação detalhadas e oficiais, consulte a documentação da AWS, buscando por "[Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)".

Para verificar o sucesso da instalação use o comando `aws --version`

### 1.2. Configuração Inicial

Após a instalação, você precisa fornecer à `CLI` suas credenciais de acesso da `AWS`. Isso é feito usando o comando `aws configure`

```bash
aws configure
```

O terminal solicitará as seguintes informações:

* `AWS Access Key ID`: A chave pública de acesso da sua conta/usuário IAM.

* `AWS Secret Access Key`: A chave secreta privada associada à sua Access Key ID.

  * ⚠️ MUITO IMPORTANTE: Trate esta chave como uma senha. Nunca a exponha ou a coloque em códigos abertos (como seu GitHub).

* `Default region name`: A região da AWS que você usará como padrão (ex: `us-east-1` ou `sa-east-1` para São Paulo).

* `Default output format`: O formato de saída dos comandos (geralmente json).

Você consegue obter estas informações ao criar uma Chaves de acesso no seu console AWS.

Atualmente o caminho para isto é: Clique no seu nome de usuário > Credenciais de segurança > Chaves de acesso > Criar chave de acesso

> Para NERDs 🤓: Ao concluir, suas credenciais serão salvas localmente (geralmente em um arquivo `~/.aws/credentials`).

## 2. 🔑 Autenticando o Docker com o ECR

O Docker não sabe automaticamente como lidar com a segurança e as credenciais do ECR. O processo é de duas etapas:

1. Usamos a `AWS CLI` para pedir um token de autenticação temporário ao `ECR`.

2. Passamos esse token diretamente para o `Docker CLI` para que ele possa se logar no Registro.

O comando que faz todo esse trabalho de forma eficiente é o `get-login-password`:

> 💡 Dica: https://docs.aws.amazon.com/cli/latest/

### O Comando de Autenticação

Você deve executar o comando abaixo, substituindo as tags adequadamente. O comando vai cumprir dos dois passos mencionados anteriormente automaticamente,

```bash
# 1. A AWS CLI busca um token de autenticação do ECR na região especificada
# 2. O token é enviado via pipeline (|) para o comando docker login

aws ecr get-login-password --region <sua-regiao> | docker login --username AWS --password-stdin <ID_da_Sua_Conta>.dkr.ecr.<sua-regiao>.amazonaws.com
```

#### Detalhes do Comando:

* `aws ecr get-login-password`: Este comando gera um token de autenticação que é válido por um período limitado (cerca de 12 horas).

* `| docker login`: O token é passado via pipeline (|) para o comando docker login.

* `--username AWS`: O nome de usuário para o ECR é sempre AWS.

* `--password-stdin`: Indica ao Docker para ler a senha da entrada padrão (onde o token da AWS está sendo enviado).

* `<ID_da_Sua_Conta>.dkr.ecr.<sua-regiao>.amazonaws.com`: Este é o endereço completo do Registro ECR. (O repositório que você já deve ter criado de antemão para subir a sua imagem)

#### 💡 Quando Autenticar?

Como o token tem validade limitada (atualmente de 12h), você precisará rodar este comando novamente sempre que o token expirar e você tentar executar um `docker push` ou `docker pull` no ECR. É uma medida de segurança importante para Registros privados.

### 3. 🖼️ Fluxo Completo (ECR)

Com a autenticação feita, o fluxo para publicar sua imagem `widget-server:v1` no `ECR` segue exatamente o mesmo padrão de `tag` e `push` que você já mapeou, mas agora com credenciais válidas:

1. **Criação do Repositório ECR**: 

💡Lembre-se da nota: no ECR, o repositório deve ser criado antes do push, via AWS Console ou AWS CLI. Isso pode ser feito tanto pelo console da AWS (via tela) ou por meio do comando abaixo:

```bash
# Exemplo de criação de repositório via CLI
aws ecr create-repository --repository-name widget-server --region <sua-regiao>
```

2. **Autenticação**:

O passo que acabamos de aprender.

```bash
aws ecr get-login-password --region <sua-regiao> | docker login --username AWS --password-stdin <ID_da_Sua_Conta>.dkr.ecr.<sua-regiao>.amazonaws.com
```

3. **Tagging**:

Marcando a imagem com o endereço completo do ECR.
Processo semelhante a *"criar um commit local"*

```bash
docker tag widget-server:v1 <ID_da_Sua_Conta>.dkr.ecr.<sua-regiao>.amazonaws.com/widget-server:v1
```

4. **Push**: 

Enviando a imagem para o repositório ECR.

```bash
docker push <ID_da_Sua_Conta>.dkr.ecr.<sua-regiao>.amazonaws.com/widget-server:v1
```

Este ciclo de autenticação garante que apenas usuários autorizados pela sua conta AWS consigam interagir com suas imagens privadas.

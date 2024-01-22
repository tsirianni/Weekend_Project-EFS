# Antes de criar o stack

Antes de iniciar a prática com a criação do stack via CloudFormation, tenha certeza de que:

- [ ] A AWS CLI v2 está configurada para realizar os comandos pelo terminal
- [ ] As credenciais estão propriamente configuradas para utilização da AWS CLI, seja via access key, SSO ou outra forma de autenticação de sua preferência
- [ ] **NÃO** está utilizando o usuário root, seguindo assim as boas práticas recomendadas pela AWS.
- [ ] Na seção `Parameters` no arquivo **infra.yml** você substituiu os valores default das propriedades `SubnetIdOne` e `SubnetIdTwo` pelas IDs as subnets da sua conta.

## Criando o Stack

Todos os recursos contigos no arquivo `infra.yml` serão automaticamente criados pelo serviço CloudFormation. Para tanto, utilize o seguinte comando na cli para criar o stack:

`aws cloudformation create-stack --stack-name EFS-STACK --template-body file://infra.yml`

**Observação**: Se estiver utilizando um perfil em específico para a autenticação, lembre-se de incluir no comando `--profile` seguido do nome do perfil.

Agora, ao visitar a página do serviço CloudFormation, deve encontrar um stack chamado `EFS-STACK` com status **CREATE_IN_PROGRESS**. Aguarde até que o status seja atualizado para **CREATE_COMPLETE**.

# Acessando as máquinas virtuais via SSH

## Obtendo a chave de acesso

Para acessar as máquinas virtuais, precisaremos da chave .pem criada. Quando uma key-pair é criada via CloudFormation, ela fica armazenada no serviço **AWS Systems Manager Parameter Store** sob a seguinte rota `/ec2/keypair/{key_pair_id}`.

> [!NOTE]
> Para maiores detalhes sobre a criação de key-pairs com cloudFormation, [clique aqui](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-keypair.html#aws-resource-ec2-keypair-syntax).

Logo, primeiro executaremos um comando para obter a ID da key-pair atrelada às máquinas virtuais e depois executaremos outro comando para realizar o download do arquivo .pem desta chave.

**Para obter a ID da chave**
`aws ec2 describe-key-pairs --filters Name=tag:Weekend_Project,Values=EFS --query KeyPairs[*].KeyPairId --output text`.

Utilizamos como filtro a tag atrelada á chave, garantindo assim que mesmo que possua outras chaves na sua conta apenas a chave criada nesta prática será retornada. O resultado deve ser parecido com o seguinte:

`key-XXXXXXXXXXXXXXXXX`

**Para realizar o donwload do arquivo .pem**
`aws ssm get-parameter --name /ec2/keypair/key-XXXXXXXXXXXXXXXXX --with-decryption --query Parameter.Value --output text > myKeyPair.pem`.

Agora, em seu diretório deve haver um arquivo chamado **myKeyPair.pem**. Utilizaremos este arquivo para acessar às máquinas virtuais.

Antes de acessar às máquinas, alteraremos as permissões deste arquivo para que sejam mais restritivas, evitando assim erros na hora de executar o comando de acesso via SSH. Execute `chmod 600 ./myKeyPair.pem`.

## Acessando às máquinas

Na página de detalhes do Stack (dentro do serviço CloudFormation), há uma seção chamada `Outputs`, onde nela possuímos alguns valores referentes à rota pública de acesso às máquinas virtuais. Iniciaremos conectando no primeiro webserver, substitua no comando abaixo o campo indicado pelo valor obtido em **WebServerOnePublicDNSName**:

`ssh -i /myKeyPair.pem ec2-user@endereço-dns-público-aqui`

Responda `yes` (após realizar o step mencionado abaixo) quando o terminal lhe perguntar se deseja continuar com a conexão.

> [!CAUTION]
> Verifique a fingerprint antes de continuar com a conexão para evitar ataques to tipo _man-in-the-middle_. Maiores detalhes de como fazer essa verificação podem ser encontrados [aqui](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html).

Agora deve estar conectado à máquina virtual.

# Instalando a lib de configuração

Para configurar a máquina virtual para utilizar o EFS, utilizaremos a lib `amazon-efs-utils`.

> [!TIP]
> Se não quiser utilizar `sudo` em todos os comandos, pode aumentar suas permissões executando `sudo su` e executar os próximos comandos como root.

instale-a através do seguinte comando:

`sudo yum install -y amazon-efs-utils`

Após a instalação, crie um diretório (que será o compartilhado entre as máquinas) com:

`sudo mkdir shared_data`

Por fim, obtenha a ID do _file system_ através da seção Outputs e realize a conexão com o EFS através do seguinte comando (subtituindo a ID onde indicado):

`sudo mount -t efs {id-do-file-system-aqui} shared_data/`

Por fim, acesse o diretório com **cd** e crie um arquivo chamado **test.txt** com `sudo touch test.txt` e escreva no arquivo que a primeira máquina já está configurada com `sudo echo "Mounted on first EC2" >> test.txt`.

Meus parabéns, acaba de configurar a primeira máquina virtual para utilizar o sistema de arquivos.

# Configurando a segunda máquina

Repita o processo de adquirir o endereço público, conectar-se à máquina virtual, instalar a lib da AWS, criar o repositório (com o mesmo nome - shared_data) e executar o comando de mount com a id do file system. Agora, ao acessar o diretório com **cd** e listar os arquivos, já deve observar que possui acesso ao arquivo **test.txt**.

Meus parabéns, ambas as máquinas virtuais possuem acesso à pasta shared_data/ e podem criar/alterar arquivos armazenados no sistema de arquivos. Isso conclui a prática!

# Clean up

Para deletar todos os recursos criados e evitar cobranças indesejadas em sua conta basta deletar o stack, o cloudFormation irá deletar todos os recursos criados por ele.

> [!CAUTION]
> Não delete nenhum recurso criado pelo cloudFormation manualmente, pois isso pode causar problemas quando solicitar a exclusão automática dos recursos. Sempre gerencie os recursos criados pelo cloudFormation através dele, seja para criar, atualizar ou deletar recursos.

Para deletar os recursos, execute: `aws cloudFormation delete-stack --stack-name EFS-STACK`

# Weekend Project | EFS

Este é um pequeno projeto onde busco aplicar meus conhecimentos e praticar a utlização de serviços da AWS (Amazon Web Services). Neste projeto, crio um NFS (Network File System) através do serviço EFS e duas instâncias de EC2 (Elastic Compute Cloud) e as configuro para que possam criar/acessar/modificar arquivos compartilhados via EFS.

Este projeto deve ser de fácil reprodução, para tanto eu utilizo o serviço CloudFormation para criar a infraestrutura como código (em yml), permitindo assim que outras pessoas possam facilmente reproduzir esta prática.

# Sobre os Recursos Criados

Além do EFS e das máquinas virtuais, são criados também Security Groups, Mount Targets, Key-Pair entre outros recursos. Abaixo, o detalhamento da necessidade de cada recurso.

## EC2's Security Group | Key Pair

Tanto os security groups quanto a chave (key pair) são necessários para que possamos acessar as máquinas virtuais via SSH e configurá-las para utilizar o NFS.

O security group permite acesso às máquinas via TCP na porta 22 e a chave é criada em formato .pem (tipo rsa).

## Mount Target's Security Group

Cada subnet que pretendemos utilizar precisa de um mount target com um security group que permita acesso ao mesmo. Contudo, indicamos como a origem do acesso o security group das máquinas virtuais.

## EC2 Instances

Criamos duas máquinas virtuais para poder demonstrar que ambas possuem acesso aos arquivos compartilhados. Ao configurar a primeira criamos um arquivo teste, e ao configurar a segunda observaremos que teremos acesso ao arquivo teste criado pela primeira máquina, concluindo assim a prática.

## Outputs

São destacados nesta seção algumas informações necessárias para concluir a prática, para a execução de alguns comandos precisaremos da ID do File System, e do endereço público das máquinas virtuais.

# Step by Step (Linux)

Todos os detalhes e comandos necessários para a conclusão da prática (não incluem a configuração da AWS CLI) estão no arquivo `step-by-step.md`, contendo também explicações sobre a necessidade dos comandos (quando aplicável).

Lembrando que os comandos e procedimentos utilizados nesta prática são focados para usuários Linux, não suportando portando usuários Windows.

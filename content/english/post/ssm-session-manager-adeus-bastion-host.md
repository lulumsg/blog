---
title: "Adeus bastion host: acessando EC2 privada com SSM Session Manager"
date: 2026-09-02T10:00:00-03:00
description: "Como substituir o bastion host por SSM Session Manager para acessar instâncias EC2 privadas com mais segurança e sem porta 22 aberta."
draft: false
tags: ["AWS", "SSM", "Session Manager", "EC2", "seguranca", "CloudOps"]
categories: ["CloudOps"]
---

Fala, pessoal!

Se você ainda mantém um bastion ou jump server para acessar instâncias em subnet privada, este post é para você. Vou mostrar como o Session Manager, parte do AWS Systems Manager, resolve esse acesso de forma mais segura, e por que isso elimina vários problemas de uma vez só.

O bastion tradicional tem um conjunto de dores bem conhecido:

- É uma instância a mais rodando 24/7, com IP público, custo de hora e de IPv4
- Tem a porta 22 aberta para a internet (ou para uma lista de IPs que ninguém mais lembra por que está lá)
- Depende de distribuir e rotacionar arquivos `.pem` entre as pessoas do time
- Quando alguém sai da empresa, você precisa lembrar de remover a chave pública de todo servidor
- Não tem auditoria: você sabe que houve um login, mas não sabe o que a pessoa digitou

Isso não quer dizer que o bastion esteja sempre errado. Dependendo da necessidade da empresa, de uma exigência de compliance ou de uma arquitetura de rede já desenhada em torno dele, manter o bastion pode continuar sendo a escolha certa. O que eu quero mostrar aqui é a alternativa para quem tem liberdade de escolher e quer resolver essas dores.

Com o Session Manager, nada disso existe. O acesso é controlado por IAM, não há porta aberta, e cada sessão pode ser gravada no S3 ou no CloudWatch Logs.

## Como funciona por baixo

Isso vale a pena entender, porque é o que explica a maior parte dos problemas de configuração.

O agente do SSM roda dentro da instância e **abre uma conexão de saída** para o serviço do Systems Manager. É sempre a instância que inicia a conversa: a AWS nunca bate na porta dela. Quando você roda `aws ssm start-session`, sua chamada vai para a API do Systems Manager, que usa aquele canal já estabelecido para entregar sua sessão.

Consequência direta: seu Security Group **não precisa de nenhuma regra de entrada**. Zero. Pode deixar o inbound vazio.

## Os três pré-requisitos

### 1. O agente instalado e rodando

O SSM Agent já vem pré-instalado no Amazon Linux 2 e 2023, nas AMIs oficiais de Ubuntu, e nas AMIs de Windows Server. Se você usa uma AMI própria ou uma distro que não traz, precisa instalar.

Para verificar se está rodando:

```bash
sudo systemctl status amazon-ssm-agent
```

### 2. A role de IAM na instância

Esse é o motivo número um de "instância não aparece na lista". A instância precisa de um instance profile com a policy gerenciada `AmazonSSMManagedInstanceCore`.

```bash
# Cria a role
aws iam create-role \
  --role-name EC2-SSM-Role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Anexa a policy gerenciada
aws iam attach-role-policy \
  --role-name EC2-SSM-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

# Cria o instance profile e associa
aws iam create-instance-profile --instance-profile-name EC2-SSM-Profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-SSM-Profile \
  --role-name EC2-SSM-Role

# Anexa a uma instância já existente
aws ec2 associate-iam-instance-profile \
  --instance-id i-0123456789abcdef0 \
  --iam-instance-profile Name=EC2-SSM-Profile
```

Se a instância já estava rodando, reinicie o agente para ele pegar as novas credenciais.

### 3. Caminho de saída até o serviço

A instância precisa alcançar três endpoints na porta 443:

- `ssm.<região>.amazonaws.com`
- `ssmmessages.<região>.amazonaws.com`
- `ec2messages.<região>.amazonaws.com`

Se a subnet privada já tem NAT Gateway, funciona sem mais nada. Mas o cenário mais interessante é o **sem NAT nenhum**, quando você cria VPC Endpoints de interface e economiza o custo do NAT no processo:

```bash
for service in ssm ssmmessages ec2messages; do
  aws ec2 create-vpc-endpoint \
    --vpc-id vpc-0123456789abcdef0 \
    --service-name com.amazonaws.us-east-1.$service \
    --vpc-endpoint-type Interface \
    --subnet-ids subnet-0123456789abcdef0 \
    --security-group-ids sg-0123456789abcdef0 \
    --private-dns-enabled
done
```

Dois detalhes que já me custaram tempo:

- O **Security Group dos endpoints** precisa liberar 443 de entrada vindo das instâncias. Se você esquecer, o sintoma é exatamente igual ao de role faltando: a instância simplesmente não aparece.
- Se for só Session Manager, `ssm` e `ssmmessages` bastam. A partir da versão 3.3.40.0, o agente prefere o `ssmmessages` sempre que ele está disponível. Mas se você também usa Run Command, Patch Manager e afins, mantenha os três. E adicione um Gateway Endpoint de S3 (que é gratuito) para o agente conseguir se atualizar sozinho.

## Verificando

```bash
aws ssm describe-instance-information \
  --query 'InstanceInformationList[].{ID:InstanceId,Nome:ComputerName,Status:PingStatus,Agente:AgentVersion}' \
  --output table
```

Se a instância aparece com `Online`, está tudo certo.

## Conectando

No console, é o botão **Connect → Session Manager** na tela da instância. Mas o fluxo bom mesmo é pelo terminal. Primeiro instale o plugin (no macOS, com Homebrew):

```bash
brew install --cask session-manager-plugin
```

E então:

```bash
aws ssm start-session --target i-0123456789abcdef0
```

Você cai num shell como o usuário `ssm-user`, que tem sudo sem senha por padrão. Sem chave, sem porta 22, sem bastion.

## O recurso que convence todo mundo: port forwarding

Aqui é onde o Session Manager deixa de ser "substituto do SSH" e vira algo melhor. Você consegue tunelar uma porta local até um serviço na VPC, inclusive um RDS que não está na instância:

```bash
aws ssm start-session \
  --target i-0123456789abcdef0 \
  --document-name AWS-StartPortForwardingSessionToRemoteHost \
  --parameters '{"host":["meu-banco.abc123.us-east-1.rds.amazonaws.com"],"portNumber":["5432"],"localPortNumber":["5432"]}'
```

Com esse comando rodando, seu DBeaver ou psql aponta para `localhost:5432` e conversa com o RDS privado. Acabou a necessidade de bastion para acesso a banco.

Para uma porta na própria instância (um serviço rodando em 8080, por exemplo), use o documento `AWS-StartPortForwardingSession`.

## E se eu realmente precisar de SSH?

Dá para tunelar SSH por cima do Session Manager, útil quando você precisa de `scp`, `rsync` ou encaminhamento de agente. Adicione ao seu `~/.ssh/config`:

```
Host i-* mi-*
  ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```

E use normalmente:

```bash
ssh ec2-user@i-0123456789abcdef0
```

Nesse modo você ainda precisa da chave SSH e do sshd rodando na instância, mas a porta 22 continua fechada para o mundo, porque o tráfego chega pelo túnel do SSM.

## Auditoria: gravando as sessões

Essa é a parte que faz o pessoal de segurança e de compliance gostar de você. Dá para registrar tudo que foi digitado em cada sessão.

No console: **Systems Manager → Session Manager → Preferences → Edit**. Marque o destino (S3 e/ou CloudWatch Logs), e ative a criptografia com KMS.

Um cuidado importante: se você ativar "Enforce KMS encryption", a role da instância **e** a identidade de quem abre a sessão precisam de permissão de `kms:GenerateDataKey` naquela chave. Sem isso, as sessões passam a falhar, e o erro não é dos mais claros.

## Controle de acesso por tag

Como tudo passa por IAM, você consegue coisas que com chave SSH seriam trabalhosas. Por exemplo, dar ao time de desenvolvimento acesso apenas às instâncias de homologação:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ssm:StartSession",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ssm:resourceTag/Ambiente": "homologacao"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": "ssm:StartSession",
      "Resource": "arn:aws:ssm:*:*:document/AWS-StartPortForwardingSession"
    },
    {
      "Effect": "Allow",
      "Action": ["ssm:TerminateSession", "ssm:ResumeSession"],
      "Resource": "arn:aws:ssm:*:*:session/${aws:username}-*"
    }
  ]
}
```

Repare no último bloco: ele garante que cada pessoa só encerra a própria sessão, não a dos colegas.

Outra configuração que vale ativar é o **Run As**, nas preferências do Session Manager. Em vez de todo mundo virar `ssm-user`, cada pessoa entra com um usuário do sistema definido por tag na identidade IAM, e assim o histórico do shell e os arquivos criados ficam atribuídos a quem realmente fez.

## Quando não funciona: checklist rápido

Se a instância não aparece em `describe-instance-information`, é quase sempre um destes:

1. **Falta a role** ou a policy `AmazonSSMManagedInstanceCore` não está anexada
2. **Falta caminho de saída**: sem NAT e sem VPC Endpoints, ou o SG do endpoint sem 443 liberado
3. **Private DNS desabilitado** no endpoint de interface, então o nome não resolve para o IP privado
4. **Agente parado ou desatualizado**
5. **Endpoints duplicados** com private DNS ativo para o mesmo serviço, causando conflito de resolução

Rodando dentro da instância, esse comando faz o diagnóstico e aponta o que falhou:

```bash
sudo /usr/local/amazon-ssm-agent/ssm-cli get-diagnostics
```

## Vale a pena?

Na minha experiência, sim, e por uma margem grande. Você elimina uma instância, um IP público, um conjunto de chaves e uma porta aberta, e ainda ganha auditoria e controle de acesso granular de brinde. O único custo real é o dos VPC Endpoints, se você optar por eles, e mesmo assim costuma sair mais barato que manter o bastion de pé.

O caminho de migração que eu recomendo: configure o Session Manager em paralelo, deixe o time usar por duas ou três semanas com o bastion ainda no ar, e só depois derrube. Assim ninguém fica sem acesso no meio de um incidente.

Alguma dúvida ou algum cenário que eu não cobri? Me chama no [LinkedIn](https://www.linkedin.com/in/luciana-silva-151504122).

Até a próxima!

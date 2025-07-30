# Infraestrutura Web na AWS com Monitoramento Automatizado
Este projeto tem como objetivo criar um ambiente de servidor web na AWS com monitoramento automático, utilizando uma instância EC2 com Ubuntu, um servidor Nginx e um script que envia alertas para o Discord caso o site fique fora do ar.

## Etapas do Projeto

### 1. Criação da Instância EC2 na AWS
### VPC
A VPC funciona como uma rede privada dentro da infraestrutura da AWS. Nela, foi definido um bloco CIDR de `10.0.0.0/16`, o que oferece um amplo espaço de endereçamento IP, permitindo a criação de múltiplas sub-redes com diferentes finalidades.

### Sub-redes públicas e privadas
Dentro dessa VPC, criei duas sub-redes públicas e duas privadas. As públicas são usadas para hospedar recursos com acesso à internet, como a instância EC2 que serve o site. As privadas ficam reservadas para simulações de ambientes internos mais seguros.

Para criar uma sub-rede basta associar ela a uma VPC, escolher uma zona de disponibilidade (us-east-1a por exemplo), e configurar um bloco CIDR IPv4 para a sub-rede (10.0.1.0/24 por exemplo).
  
### Criação da instância EC2
Com a VPC criada, criei uma instância EC2 usando a imagem Ubuntu Server 24.04, dentro de uma das sub-redes públicas. Escolhi o tipo t2.micro. No processo de criação, criei um novo par de chaves SSH e baixei o arquivo .pem, que é essencial para conectar à instância com segurança via terminal.

### Internet Gateway e Tabela de Rotas
Depois criei um Internet Gateway e associei ele à VPC. Isso é necessário para que as sub-redes públicas possam acessar a internet. Em seguida, editei a tabela de rotas padrão da VPC para direcionar o tráfego da sub-rede pública para o Internet Gateway recém-criado.

### Elastic IP
Por padrão, o IP público de uma instância EC2 muda a cada reinicialização. Isso pode prejudicar o funcionamento do script de monitoramento, pois o IP utilizado no script de monitoramento pode mudar toda vez que a instância for reiniciada, causando falhas no envio de notificações.

Para garantir que o IP público da instância EC2 seja fixo e não mude após reinicializações, criei e associei um Elastic IP à instância EC2. O Elastic IP é um IP público estático, fornecido pela AWS, que pode ser associado a qualquer instância EC2 e que permanecerá o mesmo, independentemente de reinicializações da instância.

Para alocar um Elastic IP, vá até "Endereços IP Elásticos" no console EC2, clique em "Alocar endereço IP Elástico" e confirme. Depois, clique em "Ações" > "Associar endereço IP Elástico", selecione a instância EC2 desejada e confirme a associação.

Depois que todas as configurações haviam sido finalizadas foi possível acessar a EC2 na WSL por meio do comando.
```bash
ssh -i chaveseguranca.pem ubuntu@elasticip
```

### 2. Configuração do Nginx
Após conectar na instância via SSH, o próximo passo foi a instalação do servidor Nginx com os seguintes comandos:
```bash
sudo apt update
sudo apt install nginx -y
```

Para testar se estava funcionando, substituí o arquivo index.html padrão por uma página simples com a linha:

```bash
echo "<h1>Servidor funcionando!</h1>" | sudo tee /var/www/html/index.html
```

Depois disso, acessei o Elastic IP pelo navegador e confirmei que a página aparecia normalmente.

![print](imagens/imagem-site.png)

### 3. Webhook do Discord
Para que o script consiga enviar alertas, é necessário criar um webhook do Discord, que serve como ponto de comunicação entre o servidor e o canal de notificação. Para isso, criei um servidor no Discord e após isso cliquei na opção de Editar canal. Em seguida, acessei a aba Integrações e cliquei em Webhooks. Selecionei Novo Webhook, escolhi um nome para ele ("monitoramento"), e copiei a URL do webhook gerada. 

### 4. Script de Monitoramento
Na etapa seguinte, criei um script chamado `monitor.sh` dentro do diretório /home/ubuntu. Esse script faz uma verificação a cada minuto no site. Se ele estiver fora do ar, o script envia um alerta para o canal do Discord utilizando o webhook, e também salva logs no arquivo /var/log/meuScript.log. O conteúdo do script ficou assim:

```bash
#!/bin/bash

WEBHOOK="https://discordapp.com/api/webhooks/UrlDaWebhook"
ARQUIVO_LOG="/var/log/meuScript.log"
URL="http://ElasticIP/"

DATA=$(date '+%d/%m/%y - %H:%M')

# Variável recebe apenas o código do status da requisição
STATUS=$(curl -s -o /dev/null -w "%{http_code}" $URL)

# Verifica se o status é diferente de 200, se sim quer dizer que o site está fora do ar
if [ "$STATUS" != "200" ]
then
        MENSAGEM="$DATA - Site fora do ar"
        echo "$MENSAGEM" >> "$ARQUIVO_LOG"

        # Envia uma mensagem para o discord via webhook informando que site está fora do ar
        curl -H "Content-Type: application/json" \
                -X POST \
                -d "{\"content\": \"$MENSAGEM\"}" \
                "$WEBHOOK"

else
        MENSAGEM="$DATA - Site funcionando"
        echo "$MENSAGEM" >> "$ARQUIVO_LOG"
fi
```

### 5. Agendamento 
Depois de tornar o script executável com o comando `chmod +x monitor.sh`, adicionei a execução automática usando o comando:
```bash
crontab -e
```
E adicionado a linha de comando:
```bash
* * * * * /home/ubuntu/monitor.sh
```

Isso garante que o script rode a cada minuto.

### 6. Conclusão
Após todos os processos realizados o script de monitoramento foi testado e apresentou o comportamento esperado:

Para testar, parei o Nginx usando `sudo systemctl stop nginx` e aguardei. Logo recebi a mensagem de alerta no Discord informando que o site estava fora do ar.

![print](imagens/alertas-script.png)

Ao reiniciar o Nginx com `sudo systemctl start nginx`, percebe-se que o arquivo .log também está em seu funcionamento correto e registrando as mensagens.

![print](imagens/mensagens-log.png)


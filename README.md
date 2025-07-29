# Infraestrutura Web na AWS com Monitoramento Automatizado
Este projeto consiste na criação de uma infraestrutura básica na AWS utilizando uma instância EC2 com Ubuntu, configuração de servidor web Nginx, e um script de monitoramento que envia alertas via Discord sempre que o site estiver fora do ar.

## Etapas do Projeto

### 1. Criação da Instância EC2 na AWS
### VPC
A VPC funciona como uma rede privada dentro da infraestrutura da AWS. Nela, foi definido um bloco CIDR de 10.0.0.0/16, o que oferece um amplo espaço de endereçamento IP, permitindo a criação de múltiplas sub-redes com diferentes finalidades.

### Sub-redes públicas e privadas
Dentro da VPC, foram criadas quatro sub-redes:
- Duas Sub-redes públicas: utilizadas para hospedar recursos que precisam estar acessíveis pela internet, como a instância EC2 com o servidor web.
- Duas Sub-Redes privadas: criadas para simular um ambiente mais seguro e realista, normalmente utilizadas para recursos internos, como bancos de dados ou serviços que não devem ser expostos diretamente à internet.
  
### Criação da instância EC2
Dentro de uma das sub-redes públicas, foi criada uma instância do tipo t2.micro utilizando a imagem do Ubuntu Server 24.04. Essa instância serviu como nosso servidor web.

### Autenticação com par de chaves
Para garantir um acesso seguro à instância, foi gerado um par de chaves SSH no momento da criação da EC2. A chave privada (.pem) foi baixada e armazenada localmente. Ela é essencial para autenticar o acesso via terminal, funcionando como uma identidade digital.

### Internet Gateway e Tabela de Rotas
Para permitir que a sub-rede pública tivesse acesso à internet, foi criado e associado um Internet Gateway à VPC. Também foi configurada uma tabela de rotas direcionando o tráfego da sub-rede pública para o gateway, possibilitando a comunicação externa da instância EC2.

### Elastic IP
Por padrão, o IP público de uma instância EC2 muda a cada reinicialização. Isso pode prejudicar o monitoramento do site e o acesso externo. Por esse motivo, foi associado um Elastic IP à instância, garantindo que o endereço IP permaneça fixo independentemente de reinicializações, mantendo o funcionamento consistente do sistema de monitoramento.


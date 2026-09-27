# Tech Challenge — Fase 4

## 1. Infraestrutura Gerenciada e Portabilidade

O projeto **Tech Challenge — Fase 4** mantém a estrutura de infraestrutura implementada na fase anterior, utilizando serviços gerenciados do **Microsoft Azure** para executar os containers, armazenar as imagens e disponibilizar a aplicação para acesso externo.

### Provedor de nuvem

A infraestrutura está provisionada no **Microsoft Azure**, integrando os serviços de Kubernetes gerenciado, registro privado de imagens e exposição da aplicação.

### Orquestração dos containers

A aplicação está implantada em um cluster **Azure Kubernetes Service (AKS)**, responsável pela orquestração dos containers e pela manutenção das cargas de trabalho.

A implantação utiliza configurações Kubernetes para definir os componentes da aplicação e sua comunicação, sem execução manual de containers em máquinas virtuais isoladas.

### Registro privado de imagens

As imagens dos containers estão armazenadas no **Azure Container Registry (ACR)**. O AKS possui acesso autorizado ao ACR para obter as imagens utilizadas na implantação, mantendo o armazenamento centralizado e o acesso restrito aos componentes autorizados.

### Exposição da aplicação

A aplicação está acessível externamente por meio de **Load Balancer ou Ingress Controller** configurado no cluster AKS. Esse mecanismo recebe as requisições externas e as encaminha aos serviços da aplicação executados no Kubernetes.

### Portabilidade

A utilização de containers e configurações Kubernetes permite reaproveitar a estrutura de implantação em outros ambientes compatíveis. Uma eventual migração exige ajustes nos recursos específicos do Azure, como registro de imagens, permissões, rede e armazenamento.

### Implantação da aplicação

Para colocar a aplicação em execução, devem ser aplicados os arquivos **`.yml` localizados na pasta `k8s`** do projeto. Esses arquivos contêm as configurações dos recursos Kubernetes necessários à implantação no AKS.

Com o `kubectl` instalado e configurado para acessar o cluster AKS, execute o seguinte comando na raiz do projeto:

```bash
kubectl apply -f k8s/
```

O comando aplica os manifestos da pasta, criando ou atualizando os recursos definidos. O cluster obtém as imagens disponíveis no ACR e inicia os componentes da aplicação.

Após a aplicação dos manifestos, verifique o estado dos componentes:

```bash
kubectl get pods
kubectl get services
kubectl get ingress
```

Caso os recursos utilizem um namespace específico, acrescente `-n <namespace>` aos comandos de consulta.

Quando os componentes estiverem prontos e a exposição externa estiver disponível, a aplicação poderá ser acessada pelo endereço configurado no Load Balancer ou Ingress.

## 2. Persistência Poliglota e Alta Performance

O projeto utiliza persistência poliglota, combinando tecnologias de armazenamento adequadas aos diferentes tipos de dados e cargas de trabalho da aplicação.

Essa abordagem permite tratar dados flexíveis e de maior volumetria em uma solução NoSQL, buscando baixa latência nas operações de leitura e escrita.

### Banco de dados NoSQL

A solução utiliza **MongoDB**, um banco de dados orientado a documentos que permite armazenar informações com estruturas flexíveis.

Sua adoção possibilita evoluir os dados armazenados conforme as necessidades da aplicação, acomodando atributos e estruturas que podem variar entre os documentos.

### Cenário de uso do MongoDB

O MongoDB atende à persistência de dados que exigem flexibilidade de estrutura ou tratamento de volumes crescentes de informações. Essa responsabilidade complementa os demais mecanismos de persistência do projeto, permitindo utilizar cada tecnologia de acordo com as características dos dados e das operações.

### Integração do MongoDB com .NET

A integração entre a aplicação .NET e o MongoDB é realizada por meio do driver oficial **MongoDB.Driver**.

O driver permite estabelecer a conexão com o banco e executar operações de inclusão, consulta, atualização e exclusão de documentos.

### Cache distribuído

O projeto utiliza **Redis** como camada de cache distribuído para armazenar temporariamente dados e reduzir a quantidade de acessos ao banco de dados principal.

O cache é compartilhado entre as instâncias da aplicação executadas no AKS, permitindo que diferentes réplicas acessem os mesmos dados em cache.

### Cenário de uso do Redis

O Redis armazena temporariamente resultados de consultas onerosas, permitindo reutilizar informações sem repetir a consulta ao banco principal a cada requisição.

Quando o dado está disponível no cache, a aplicação retorna o resultado armazenado. Caso contrário, consulta o banco principal e armazena o resultado no Redis para atender às próximas requisições.

### Integração do Redis com .NET

A integração utiliza os recursos de cache distribuído do ecossistema .NET, por meio da biblioteca **StackExchange.Redis** ou da abstração **IDistributedCache** do ASP.NET Core.

Essa integração permite gravar, recuperar e remover dados do cache, além de configurar o prazo de expiração das informações.

### Expiração e atualização dos dados

Os dados são armazenados com um tempo de expiração (**TTL**), limitando sua permanência no cache. Quando uma informação é alterada no banco principal, o conteúdo correspondente no cache deve ser atualizado ou invalidado para evitar o retorno de dados desatualizados.

### Desempenho e baixa latência

A separação das responsabilidades de persistência permite direcionar as cargas de trabalho ao armazenamento mais adequado. A reutilização dos dados no Redis reduz as consultas repetidas e as viagens de ida e volta ao banco principal (*round-trips*), contribuindo para diminuir a latência das respostas e a carga sobre a persistência.

O atendimento à baixa latência depende da modelagem dos documentos, dos índices utilizados nas consultas e dos recursos disponíveis no ambiente. A comprovação do desempenho deve ser realizada por meio de medições e testes de carga.

## 3. Busca Avançada com Elasticsearch

O projeto utiliza um serviço gerenciado de **Elasticsearch** para realizar buscas no catálogo de jogos, independentemente das consultas tradicionais ao banco de dados relacional.

### Sincronização do catálogo

A lógica de indexação está implementada na **CatalogAPI**. Sempre que um jogo é inserido ou editado no banco principal, o documento correspondente no Elasticsearch é criado ou atualizado.

Essa sincronização permite que as buscas reflitam as informações do catálogo, mantendo o banco principal como fonte dos dados e o Elasticsearch como mecanismo especializado de pesquisa.

### Endpoint de busca

O endpoint **`/search`** disponibiliza consultas ao catálogo com suporte a:

- **Fuzzy Search:** tolerância a erros de digitação, permitindo encontrar jogos mesmo quando o termo pesquisado apresenta pequenas diferenças em relação ao conteúdo indexado.
- **Ordenação por relevância:** apresentação dos resultados conforme sua correspondência com os termos informados pelo usuário.

## 4. Automação de CI/CD com Azure DevOps

O projeto utiliza o **Azure DevOps**, por meio do **Azure Pipelines**, para automatizar a integração e a entrega contínuas dos serviços **UsersAPI** e **CatalogAPI**.

O fluxo integra as alterações da branch **`main`** ao ambiente de produção no AKS, permitindo rastrear o código, as imagens e as implantações de cada versão.

### Compilação e testes

A pipeline realiza a compilação dos serviços e a execução dos testes unitários automatizados. A continuidade da entrega depende do sucesso dessas etapas, evitando a publicação de versões com falhas de compilação ou nos testes.

### Containerização e publicação

Após a validação, a pipeline gera as imagens Docker dos serviços com **tags de versionamento**, permitindo identificar cada versão publicada.

As imagens são enviadas ao ACR, registro privado utilizado pelo AKS para obter os containers da aplicação.

### Implantação no Kubernetes

A pipeline aplica os arquivos **`.yml` da pasta `k8s`**, atualizando os serviços com as versões das imagens publicadas no ACR.

A implantação utiliza a estratégia **Rolling Update**, substituindo gradualmente os containers para manter a aplicação disponível durante a atualização. A continuidade do atendimento depende da disponibilidade de réplicas e da configuração adequada das verificações de prontidão.

### Rastreabilidade com Azure DevOps

O Azure DevOps centraliza a execução da pipeline e mantém o histórico das compilações, dos testes e das implantações.

Essa integração permite relacionar cada entrega ao código que a originou e acompanhar o resultado das etapas, mantendo a versão em produção alinhada à última execução bem-sucedida da pipeline da branch **`main`**.

## 5. Segurança e Gestão de Segredos

O projeto utiliza a injeção de segredos em tempo de execução para disponibilizar credenciais aos serviços, mantendo informações sensíveis fora do código-fonte e dos manifestos versionados.

### Ausência de credenciais no código

Strings de conexão, chaves de API e senhas não são armazenadas diretamente no código-fonte nem nos arquivos **`.yml` da pasta `k8s`**.

Os manifestos contêm apenas referências aos segredos necessários à execução dos serviços. Valores codificados em Base64 também são tratados como informações sensíveis e não devem ser incluídos no repositório.

### Injeção de segredos

As credenciais são fornecidas aos containers por meio de **Secrets do Kubernetes** ou da integração do cluster com o **Azure Key Vault**, conforme a configuração do ambiente.

Durante a inicialização dos serviços, os segredos são disponibilizados por variáveis de ambiente ou arquivos montados nos containers, permitindo o acesso aos bancos de dados e aos demais serviços utilizados pela aplicação.

### Controle de acesso

O acesso aos segredos é restrito aos componentes autorizados. Essa separação permite gerenciar as credenciais independentemente das imagens Docker e evita a necessidade de incorporar informações sensíveis durante a compilação ou a publicação dos serviços.

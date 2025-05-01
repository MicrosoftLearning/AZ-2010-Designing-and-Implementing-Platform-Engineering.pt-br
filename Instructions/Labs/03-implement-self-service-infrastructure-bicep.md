---
lab:
  title: Implementar infraestrutura de autoatendimento com o Bicep
  module: Strategic Platform Road Mapping
---

# Laboratório 03: implementar infraestrutura de autoatendimento com o Bicep

## Tempo estimado: 20 minutos

## Pré-requisitos

- Assinatura do Azure - Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/).
- Conhecimento básico dos serviços do Azure e da CLI do Azure.
- Visual Studio Code instalado com a extensão Bicep.
- CLI do Azure instalada e configurada no seu computador local.
- Familiaridade com conceitos de Infraestrutura como Código (IaC).

## Objetivos

- Configure o Bicep no seu ambiente.
- Crie um modelo Bicep para definir recursos do Azure.
- Implante um Serviço de Aplicativo do Azure com banco de dados SQL no back-end.
- Imponha governança usando rótulos e políticas.
- Implemente dimensionamento automatizado com o Bicep.

## Exercício 1: criar um modelo Bicep para implantar recursos do Azure

Em um ambiente de engenharia de plataforma, os desenvolvedores precisam de uma maneira de provisionar a infraestrutura de forma consistente e eficiente. Este laboratório orientará você sobre o uso do Bicep, uma ferramenta de IaC (infraestrutura como código), para implantar e gerenciar você mesmo recursos do Azure. Você criará um modelo Bicep para provisionar um grupo de recursos, um Serviço de Aplicativo do Azure, um Banco de Dados SQL do Azure e uma Conta de Armazenamento enquanto impõe governança com rótulos e políticas.

### Tarefa 1: instalar a CLI do Bicep

1. Abra o seu terminal local.
1. Para verificar se o Bicep está instalado, execute:

   ```bash
   az bicep version
   ```

   Se o Bicep não estiver instalado, instale-o com:

   ```bash
   az bicep install
   ```

1. Confirme a instalação executando:

   ```bash
   az bicep version
   ```

   Você deve ver uma saída como CLI do Bicep versão XYZ.

   > **OBSERVAÇÃO:** É preciso ter a [CLI do Azure](https://learn.microsoft.com/cli/azure/install-azure-cli) instalada.

### Tarefa 2: criar um modelo do Bicep

1. No seu terminal local, crie um novo arquivo:

   ```bash
   code main.bicep
   ```

1. Copie e cole o seguinte código Bicep no arquivo:

   ```bicep
   param location string = 'eastus2'
   param appServicePlanName string = 'bicepAppPlan'
   param webAppName string = 'bicep-webapp'
   param storageAccountName string = 'biceplabstorage'
   param sqlServerName string = 'bicep-sqlserver'
   param sqlDatabaseName string = 'bicepdb'
   param sqlAdminUser string
   @secure()
   param sqlAdminPassword string

   targetScope = 'resourceGroup'

   resource storage 'Microsoft.Storage/storageAccounts@2021-09-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'StorageV2'
   }

   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }

   resource webApp 'Microsoft.Web/sites@2021-02-01' = {
     name: webAppName
     location: location
     properties: {
       serverFarmId: appPlan.id
     }
   }

   resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
     name: sqlServerName
     location: location
     properties: {
       administratorLogin: sqlAdminUser
       administratorLoginPassword: sqlAdminPassword
       version: '12.0'
     }
   }

   resource sqlDb 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
     name: sqlDatabaseName
     location: location
     parent: sqlServer
     properties: {
       collation: 'SQL_Latin1_General_CP1_CI_AS'
       maxSizeBytes: 2147483648
     }
   }
   ```

   > **OBSERVAÇÃO:** você pode instalar a extensão Bicep no Visual Studio Code para obter realce de sintaxe e IntelliSense para arquivos Bicep.

   > **IMPORTANTE:** verifique se storageAccountName, webAppName e sqlServerName são globalmente únicos. Se a implantação falhar devido a conflitos de nome, modifique os nomes conforme for necessário.

   > **OBSERVAÇÃO:** Se você encontrar um erro relacionado à região, tente implantar em uma região diferente alterando o valor do `location`parâmetro.

1. Salve e feche o arquivo.

### Tarefa 3: implantar o modelo com a CLI do Azure

1. Execute o seguinte comando para criar o grupo de recursos:

   ```bash
   az group create --name BicepLab-RG --location centralus

   ```

   > **OBSERVAÇÃO :** talvez seja necessário fazer logon na sua conta do Azure se ainda não estiver autenticado. Para obter instruções detalhadas, consulte [Autenticar no Azure usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

2. Implante o modelo no grupo de recursos:

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

   > **OBSERVAÇÃO :** Substitua a SuaSenhaSegura123! com uma senha forte e azureuser com um nome de usuário válido.

3. Aguarde até que a implantação seja concluída. Você verá uma mensagem de confirmação.
4. No portal do Azure, navegue até Grupos de Recursos.
5. Localize BicepLab-RG e clique nele.
6. Verifique se os recursos foram criados com êxito.

Você implantou com êxito um Serviço de Aplicativo do Azure com um back-end de Banco de Dados SQL usando um modelo Bicep. Agora, você pode gerenciar sua infraestrutura como código e implantar recursos de forma consistente e eficiente. A próxima etapa seria integrar esse modelo ao pipeline de CI/CD, para fazer implantações automatizadas.

## Exercício 2: impor governança com rótulos e políticas

Em um ambiente de infraestrutura de autoatendimento, é essencial impor governança para se garantir conformidade e controle de custos. Rótulos e políticas são dois mecanismos principais para se conseguir isso.

### Tarefa 1: adicionar rótulos aos recursos

1. Execute o seguinte comando para adicionar um rótulo ao grupo de recursos:

   ```bash
   az group update --name BicepLab-RG --set tags.Owner='PlatformEngineering'
   ```

1. Verifique se o rótulo foi adicionado, executando:

   ```bash
   az group show --name BicepLab-RG --query tags
   ```

### Tarefa 2: criar uma política para impor a rotulagem

1. Crie um arquivo de política JSON:

   ```bash
   code tagging-policy.json
   ```

1. Copie e cole a seguinte definição de política no final do arquivo:

   ```json
   {
     "if": {
       "not": {
         "field": "tags[Owner]",
         "exists": "true"
       }
     },
     "then": {
       "effect": "deny"
     }
   }
   ```

1. Crie a definição de política usando o arquivo JSON:

   ```bash
    az policy definition create --name 'tagging-policy' --display-name 'Enforce tagging' --rules @tagging-policy.json --mode All
   ```

1. Atribua a política ao grupo de recursos:

   ```bash
   az policy assignment create --name 'tagging-policy-assignment' --display-name 'Enforce tagging' --policy "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policyDefinitions/tagging-policy" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG"

   ```

   > **IMPORTANTE:** Substitua os dois campos<subscription-id> pela sua ID de assinatura do Azure.

1. Verifique se a política foi atribuída, executando:

   ```bash
   az policy assignment list --output table
   ```

### Tarefa 3: testar a imposição de políticas

1. Para remover o rótulo do grupo de recursos, execute o seguinte comando:

   ```bash
   az tag update --resource-id /subscriptions/<subscription-id>/resourceGroups/BicepLab-RG --operation delete --tags Owner
   ```

   > **IMPORTANTE:** Substitua o campo <subscription-id> pela sua ID de assinatura do Azure.

1. Você deve ver uma mensagem de erro indicando que a operação foi negada devido à imposição da política. O rótulo não deve ser removido. Isso confirma que a política está funcionando conforme o esperado. Para resolver o erro, remova temporariamente a atribuição de política e execute novamente o comando.

   ```bash
   az policy exemption create --name 'tagging-policy-exemption' --policy-assignment "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG/providers/Microsoft.Authorization/policyAssignments/tagging-policy-assignment" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG" --display-name "Temporary exemption to remove tag" --exemption-category Waiver
   ```

   > **IMPORTANTE:** Substitua os dois campos<subscription-id> pela sua ID de assinatura do Azure.

Você impôs com êxito a governança usando rótulos e políticas. Os desenvolvedores agora serão obrigados a marcar recursos com o rótulo Proprietário, garantindo a devida propriedade e responsabilidade. Você pode criar políticas adicionais para impor outros requisitos de governança, como convenções de nomenclatura de recursos, tipos de recursos e locais de recursos.

## Exercício 3: implementar dimensionamento automatizado com o Bicep

Em um ambiente de engenharia de plataforma, garantir que os aplicativos possam ser dimensionados com eficiência é um requisito fundamental. Dimensionamento automatizado melhora o desempenho, otimiza a utilização de recursos e minimiza os custos. Neste exercício, você implementará o dimensionamento automático para um Serviço de Aplicativo do Azure usando o Bicep.

### Tarefa 1: definir uma política de dimensionamento automático no Bicep

1. Abra o arquivo main.bicep e localize a seção existente do Plano do Serviço de Aplicativo:

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }
   ```

1. Modifique o recurso appPlan para usar o SKU S1 (que dá suporte a dimensionamento automático):

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'S1'
       tier: 'Standard'
     }
     properties: {
       perSiteScaling: false
       maximumElasticWorkerCount: 10
     }
   }
   ```

1. Imediatamente após esse recurso, adicione a configuração autoscaleSetting:

   ```bicep
   resource autoscaleSetting 'Microsoft.Insights/autoscaleSettings@2024-01-01-preview' = {
   name: 'autoscale-rule'
   location: location
   properties: {
    profiles: [
      {
        name: 'defaultProfile'
        capacity: {
          minimum: '1'
          maximum: '5'
          default: '1'
        }
        rules: [
          {
            metricTrigger: {
              metricName: 'CpuPercentage'
              metricResourceUri: resourceId('Microsoft.Web/serverfarms', appServicePlanName)
              operator: 'GreaterThan'
              threshold: 75
              timeAggregation: 'Average'
              statistic: 'Average'
              timeWindow: 'PT5M'
              timeGrain: 'PT1M'
            }
            scaleAction: {
              direction: 'Increase'
              type: 'ChangeCount'
              value: '1'
              cooldown: 'PT1M'
            }
          }
        ]
      }
    ]
   }
   }
   ```

1. Salve o arquivo.
1. Valide o arquivo Bicep atualizado:

   ```bash
   az bicep build --file main.bicep
   ```

1. Implante o modelo atualizado:

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

1. Verifique se a política de dimensionamento automático foi aplicada ao Plano do Serviço de Aplicativo. Acesse o portal do Azure, navegue até o Plano do Serviço de Aplicativo, selecione a folha Escalar horizontalmente (Plano do Serviço de Aplicativo) e verifique se a regra de dimensionamento automático está configurada.

   > **OBSERVAÇÃO:** se você quiser testar o comportamento do dimensionamento automático, pode simular um uso intenso da CPU no Serviço de Aplicativo executando um teste de carga ou gerando tráfego. O Serviço de Aplicativo deve se dimensionar automaticamente com base na regra definida.

Você implementou com êxito o dimensionamento automatizado para um Serviço de Aplicativo do Azure usando o Bicep. Isso garante que seus aplicativos possam lidar com aumento do tráfego e da demanda com eficiência, melhorando o desempenho e reduzindo custos.

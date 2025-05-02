---
lab:
  title: Implementar o Computador de Desenvolvimento da Microsoft
  module: Implement Developer Self-Service
---

# Laboratório 01: implementar o Computador de Desenvolvimento da Microsoft

## Tempo estimado: 60 minutos

## Pré-requisitos

- Um locatário do Microsoft Entra com três contas de usuário pré-criadas (e, opcionalmente, três grupos do Microsoft Entra pré-criados) representando três funções diferentes envolvidas em implantações do Computador de Desenvolvimento da Microsoft. Para fins de clareza, os nomes de usuário e grupo nas instruções do laboratório devem correspondendo às informações na tabela a seguir:

| Usuário              | Agrupar                        | Função                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | Engenheiro de plataforma     |
| devlead01         | DevCenter_Dev_Leads          | Líder da equipe de desenvolvimento |
| devuser01         | DevCenter_Dev_Users          | Desenvolvedor             |

- Uma assinatura do Azure para hospedar recursos do Computador de Desenvolvimento da Microsoft associados ao locatário do Microsoft Entra que hospeda as contas de usuário e grupo
- Uma assinatura do Microsoft Intune associada ao mesmo locatário do Microsoft Entra que a assinatura do Azure
- Uma licença do Microsoft Intune atribuída às três contas de usuário pré-criadas do Microsoft Entra
- Você vai precisar de uma conta e repositório do GitHub.
  - Para criar uma conta no GitHub, siga as instruções no artigo [Como criar uma conta no GitHub](https://docs.github.com/get-started/quickstart/creating-an-account-on-github).
  - Para criar um repositório GitHub, siga as instruções no artigo [Criar um novo repositório](https://docs.github.com/repositories/creating-and-managing-repositories/creating-a-new-repository).
- Um repositório GitHub criado como uma bifurcação de https://github.com/microsoft/devcenter-catalog
- Um repositório GitHub criado como uma bifurcação de https://github.com/MicrosoftLearning/contoso-co-eShop

## Objetivos

Depois de realizar este workshop, você será capaz de:

- Implementar um ambiente do Computador de Desenvolvimento da Microsoft
- Personalizar um ambiente do Computador de Desenvolvimento da Microsoft

## Exercício 1: implementar um ambiente do Computador de Desenvolvimento da Microsoft

Neste exercício, você usará um conjunto de recursos fornecidos pela Microsoft para implementar um ambiente de Computador de Desenvolvimento da Microsoft. Essa abordagem se concentra em minimizar o esforço envolvido na criação de uma solução funcional de autoatendimento para desenvolvedores.

O exercício é composto pelas seguintes tarefas:

- Tarefa 1: criar um centro de desenvolvimento
- Tarefa 2: examinar as configurações do centro de desenvolvimento
- Tarefa 3: criar uma definição de computador de desenvolvimento
- Tarefa 4: criar um projeto
- Tarefa 5: criar um pool de computadores de desenvolvimento
- Tarefa 6: configurar permissões
- Tarefa 7: avaliar um computador de desenvolvimento

### Tarefa 1: criar um centro de desenvolvimento

Nesta tarefa, você cria um centro de desenvolvimento do Azure que será usado durante o primeiro exercício deste laboratório. Um centro de desenvolvimento é um serviço de engenharia de plataforma que centraliza a criação e o gerenciamento de ambientes de desenvolvimento e implantação escaláveis e pré-configurados, otimizando a colaboração e a utilização de recursos para equipes de desenvolvimento de software.

1. Inicie um navegador da Web e navegue até o portal do Azure em `https://portal.azure.com`.
1. Quando precisar se autenticar, conecte-se usando a sua conta de usuário do Microsoft Entra.
1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **`Dev centers`**.
1. Na página de **Centros de desenvolvimento**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um centro de desenvolvimento**, especifique as seguintes configurações e selecione **Avançar: Configurações**:

   | Configuração                                                                 | Valor                                                        |
   | ----------------------------------------------------------------------- | ------------------------------------------------------------ |
   | Assinatura                                                            | O nome da assinatura do Azure que você está usando neste laboratório |
   | Grupo de recursos                                                          | O nome de um **novo** grupo de recursos** rg-devcenter-01**     |
   | Nome                                                                    | **devcenter-01**                                             |
   | Location                                                                | **(EUA) Leste dos EUA**                                             |
   | Anexar um catálogo de início rápido – definições de ambiente de implantação do Azure | Habilitado                                                      |
   | Anexar um catálogo de início rápido – tarefas de personalização do computador de desenvolvimento              | Desabilitado                                                     |

   > **Observação:** você vai configurar um catálogo com tarefas de personalização habilitadas no segundo exercício deste laboratório.

1. Na guia **Configurações** da página **Criar um centro de desenvolvimento**, especifique as seguintes configurações e selecione **Revisar + Criar**:

   | Configuração                                    | Valor   |
   | ------------------------------------------ | ------- |
   | Habilitar catálogos por projeto                | Habilitado |
   | Permitir rede hospedada pela Microsoft nos projetos | Habilitado |
   | Habilitar instalação do Agente do Azure Monitor    | Habilitado |

   > **Observação:** por design, os recursos de catálogos anexados ao centro de desenvolvimento estão disponíveis para todos os projetos dentro dele. A configuração **Habilitar catálogos por projeto** também possibilita anexar catálogos adicionais a projetos selecionados arbitrariamente.

   > **Observação:** os computadores de desenvolvimento podem ser conectados a uma rede virtual em sua própria assinatura do Azure ou a uma hospedada pela Microsoft, dependendo se eles precisam se comunicar com recursos no seu ambiente. Se não houver essa necessidade, ao habilitar a configuração **Permitir rede hospedada pela Microsoft em projetos**, você introduz a opção de conectar computadores de desenvolvimento a uma rede hospedada pela Microsoft, minimizando a sobrecarga de gerenciamento e configuração.

   > **Observação:** a configuração **Habilitar instalação do agente do Azure Monitor** dispara automaticamente a instalação do agente do Azure Monitor em todos os computadores de desenvolvimento no centro de desenvolvimento.

1. Na guia **Examinar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   > **Observação:** aguarde até que o projeto seja provisionado. O criação do projeto pode levar cerca de um minuto.

1. Na página de **Implantação concluída**, selecione **Ir para o recurso**.

### Tarefa 2: examinar as configurações do centro de desenvolvimento

Nesta tarefa, você examina a definição de configuração básica do centro de desenvolvimento criado na tarefa anterior.

1. No navegador da Web que exibe o portal do Azure, na página **devcenter-01**, no menu de navegação vertical no lado esquerdo, expanda a seção **Configuração do ambiente** e selecione **Catálogos**.
1. Na página **devcenter-01 \| Catálogos**, observe que o centro de desenvolvimento está configurado com o catálogo **quickstart-environment-definitions**, que aponta para o repositório `https://github.com/microsoft/devcenter-catalog.git`GitHub .
1. Verifique se a coluna **Status** contém o item **Sincronização bem-sucedida**. Em caso contrário, use a seguinte sequência de etapas para recriar o catálogo:

   1. Marque a caixa de seleção ao lado do item de catálogo gerada automaticamente **quickstart-environment-definitions** e, na barra de ferramentas, selecione **Excluir**.
   1. Na página **Catálogos do devcenter-01\|**, selecione **+ Adicionar**.
   1. No painel **Adicionar catálogo**, na caixa de texto **Nome**, insira**`quickstart-environment-definitions-fix`**, na seção **Local do catálogo**, selecione **GitHub**, no **Tipo de autenticação**, selecione **Aplicativo GitHub**, deixe a caixa de seleção **Sincronizar automaticamente este catálogo** habilitada e selecione **Entrar com o GitHub**.
   1. Na janela **Entrar com o GitHub**, insira as credenciais do GitHub e selecione **Entrar**.

      > **Observação:** essas credenciais do GitHub fornecem acesso a um repositório do GitHub criado como um fork de <https://github.com/microsoft/devcenter-catalog>

   1. Quando solicitado, na janela **Autorizar Microsoft DevCenter**, selecione **Autorizar Microsoft DevCenter**.
   1. De volta ao painel **Adicionar catálogo**, na lista suspensa **Repositório**, selecione **devcenter-catalog**, na lista suspensa **Ramificação**, aceite o item **ramificação padrão**, no **Caminho da pasta**, insira **`Environment-Definitions`** e selecione **Adicionar**.
   1. De volta à página **Catálogos do devcenter-01\|**, verifique se a sincronização foi concluída com êxito monitorando o item na coluna **Status**.

1. Na página **Catálogos do devcenter-01\|**, selecione o item **quickstart-environment-definitions-fix**.
1. Na página **quickstart-environment-definitions-fix**, examine a lista de definições de ambiente predefinidas.

   > **Observação:** cada item representa uma definição de um ambiente de implantação do Azure definido em uma respectiva subpasta da pasta **Environment-Definitions** do repositório `https://github.com/microsoft/devcenter-catalog.git`GitHub.

   > **Observação:** um ambiente de implantação é uma coleção de recursos do Azure definidos em um modelo chamado de definição de ambiente. Os desenvolvedores podem usar essas definições para implantar a infraestrutura que servirá para hospedar suas soluções. Para obter mais informações sobre os ambientes de implantação do Azure, consulte o artigo do Microsoft Learn [O que são Ambientes de Implantação do Azure?](https://learn.microsoft.com/azure/deployment-environments/overview-what-is-azure-deployment-environments)

### Tarefa 3: criar uma definição de computador de desenvolvimento

Nesta tarefa, você cria uma definição de computador de desenvolvimento. Sua finalidade é definir o sistema operacional, as ferramentas, as configurações e os recursos que servem de modelo para a criação de ambientes de desenvolvimento consistentes e personalizados (chamados de computadores de desenvolvimento).

1. No navegador da Web que exibe o portal do Azure, na página **devcenter-01**, no menu de navegação vertical no lado esquerdo, expanda a seção **Configuração do Computador de Desenvolvimento** e selecione **Definições do Computador de Desenvolvimento**.
1. Na página **Definições do computador de desenvolvimento devcenter-01\|**, selecione **+ Criar**.
1. Na janela **Criar um novo dispositivo**, especifique as seguintes configurações e, em seguida, selecione **Criar**:

   | Configuração            | Valor                                                                                |
   | ------------------ | ------------------------------------------------------------------------------------ | ----------------------- |
   | Nome               | **devbox-definition-01**                                                             |
   | Imagem              | \*\*Visual Studio 2022 Enterprise no Windows 11 Enterprise + Microsoft 365 Apps 24H2 | Hibernação suportada\*\* |
   | Versão da imagem      | **Mais recente**                                                                           |
   | Computação            | **8 vCPU, 32 GB de RAM**                                                                |
   | Armazenamento            | **SSD de 256GB**                                                                       |
   | Habilitar hibernação | Habilitado                                                                              |

   > **Observação:** aguarde até que a definição do computador de desenvolvimento seja criada. Isso deverá levar menos de 1 minuto.

### Tarefa 4: criar um projeto de centro de desenvolvimento

Nessa tarefa, você cria um projeto de centro de desenvolvimento. Um projeto de centro de desenvolvimento normalmente corresponde a um projeto de desenvolvimento dentro da sua organização. Por exemplo, você pode criar um projeto para o desenvolvimento de um aplicativo de linha de negócios e outro projeto para o desenvolvimento do site da empresa. Todos os projetos em um centro de desenvolvimento compartilham as mesmas definições de caixa de desenvolvimento, conexão de rede, catálogos e galerias de computação. Você pode considerar a criação de vários projetos de centro de desenvolvimento se tiver vários projetos de desenvolvimento que tenham administradores de projeto e requisitos de permissões de acesso separados.

1. No navegador da Web que exibe o portal do Azure, na página **devcenter-01**, no menu de navegação vertical no lado esquerdo, expanda a seção **Gerenciar** e selecione **Projetos**.
1. Na página de **Projetos devcenter-01\|**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um projeto**, especifique as seguintes configurações e selecione **Avançar: gerenciamento de computador de desenvolvimento**:

   | Configuração        | Valor                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Assinatura   | O nome da assinatura do Azure que você está usando neste laboratório |
   | Grupo de recursos | **rg-devcenter-01**                                          |
   | Centro de desenvolvimento     | **devcenter-01**                                             |
   | Nome           | **devcenter-project-01**                                     |
   | Descrição    | **devcenter-project-01**                                     |

1. Na guia **Gerenciamento de computador de desenvolvimento** da página **Criar um projeto**, especifique as seguintes configurações e selecione **Avançar: Catálogos**:

   | Configuração                 | Valor |
   | ----------------------- | ----- |
   | Habilitar limites de computador de desenvolvimento   | Sim   |
   | Computadores de desenvolvimento por desenvolvedor | **2** |

1. Na guia **Catálogo** da página **Criar projeto**, especifique as seguintes configurações e selecione **Revisar + Criar**:

   | Configuração                            | Valor   |
   | ---------------------------------- | ------- |
   | Definições de ambiente de implantação | Habilitado |
   | Definições da imagem                  | Habilitado |

   > **Observação:** como você habilitou catálogos no nível do projeto ao criar o centro de desenvolvimento, para catálogos anexados a um projeto de desenvolvimento, essas configurações determinam quais itens de catálogo são importados para o projeto com a sincronização.

1. Na guia **Revisar + criar** da página **Criar um centro de desenvolvimento**, selecione **Criar**:

   > **Observação:** aguarde a criação do projeto. Isso deverá levar menos de 1 minuto.

1. Na página de **Implantação concluída**, selecione **Ir para o recurso**.

### Tarefa 5: criar um pool de computadores de desenvolvimento

Nessa tarefa, você cria um pool de computadores de desenvolvimento no projeto do centro de desenvolvimento criado na tarefa anterior. Os pools de computadores de desenvolvimento são usados por usuários de computadores de desenvolvimento para criar computadores de desenvolvimento. Um pool de computador de desenvolvimento vincula uma definição de computador de desenvolvimento a uma conexão de rede. Em geral, pode-se escolher entre conexões hospedadas pela Microsoft ou suas próprias conexões de rede do Azure. A conexão de rede determina o local de um computador de desenvolvimento hospedado e seu acesso a outros recursos locais e de nuvem. Considere criar um pool de computador de desenvolvimento com uma conexão de rede mais próxima dos usuários da computador de desenvolvimento. Além disso, para reduzir o custo de execução dos computadores de desenvolvimento, você pode configurar um pool de computadores de desenvolvimento para desligá-los diariamente em um horário predefinido.

1. No navegador da Web que exibe o portal do Azure, na página **devcenter-project-01** , no menu de navegação vertical no lado esquerdo, expanda a seção **Gerenciar** e selecione **Pools de computadores de desenvolvimento**.
1. Na página de pools de computadores de desenvolvimento **devcenter-project-01\|**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um computador de desenvolvimento**, especifique as seguintes configurações e selecione **Criar**:

   | Configuração                                                                                                           | Valor                                    |
   | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
   | Nome                                                                                                              | **devcenter-project-01-devbox-pool-01**  |
   | Definição                                                                                                        | **devbox-definition-01**                 |
   | Conexão de rede                                                                                                | **Implantar em uma rede hospedada da Microsoft** |
   | Região                                                                                                            | **(EUA) Leste dos EUA**                         |
   | Habilitar logon único                                                                                             | Habilitado                                  |
   | Privilégios de Criador do Computador de Desenvolvimento                                                                                        | **Administrador Local**                  |
   | Ativar parada automática dentro do cronograma                                                                                      | Habilitado                                  |
   | Hora de término                                                                                                         | **19:00**                             |
   | Fuso horário                                                                                                         | Seu fuso horário atual                   |
   | Ativar hibernação em caso de desconexão                                                                                    | Habilitado                                  |
   | Período de cortesia em minutos                                                                                           | **60**                                   |
   | Confirmo que a minha organização tem licenças de Benefícios Híbridos do Azure, que se aplicam a todos os computadores de desenvolvimento neste pool | Habilitado                                  |

   > **Observação:** aguarde até que o pool de computadores de desenvolvimento seja criado. Isso pode levar cerca de dois minutos.

### Tarefa 6: configurar permissões

Nessa tarefa, você atribui permissões adequadas relacionadas ao computador de desenvolvimento da Microsoft às três entidades de segurança do Microsoft Entra que foram provisionadas em seu ambiente de laboratório. Essas entidades de segurança correspondem a funções típicas em cenários de engenharia de plataforma:

| Usuário              | Agrupar                        | Função                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | Engenheiro de plataforma     |
| devlead01         | DevCenter_Dev_Leads          | Líder da equipe de desenvolvimento |
| devuser01         | DevCenter_Dev_Users          | Desenvolvedor             |

O computador de desenvolvimento da Microsoft utiliza o controle de acesso baseado em função do Azure (Azure RBAC) para controlar acesso a funcionalidade ao nível do projeto. Os engenheiros de plataforma devem ter controle total para criar e gerenciar centros de desenvolvimento, seus catálogos e projetos. Isso requer efetivamente a função de proprietário ou colaborador, dependendo se eles também precisam da capacidade de delegar permissões a outras pessoas. Líderes da equipe de desenvolvimento devem receber a função de Administrador de Projeto do centro de desenvolvimento, que concede a capacidade de executar tarefas administrativas em projetos do Computador de Desenvolvimento da Microsoft. Os usuários do computador de desenvolvimento precisam criar e gerenciar seus próprios computadores de desenvolvimento, que estão associados à função Usuário do Computador de Desenvolvimento.

> **Observação:** você começa atribuindo permissões ao grupo do Microsoft Entra destinado a conter contas de usuário de engenheiro de plataforma.

1. No navegador da Web que exibe o portal do Azure, navegue até a página **devcenter-01** e, no menu de navegação vertical no lado esquerdo, selecione **Controle de acesso (IAM)**.
1. Na página **IAM (controle de acesso) devcenter-01\|**, selecione **+ Adicionar** e, na lista suspensa, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, selecione **Função de administrador com privilégios**, na lista de funções, selecione **Proprietário** e então selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, verifique se a opção **Usuário, grupo ou serviço principal** está selecionada e clique em **+ Selecionar membros**.
1. No painel **Selecionar membros**, pesquise e selecione **`DevCenter_Platform_Engineers`** e clique em **Selecionar**.
1. De volta à guia **Membros** da página **Adicionar atribuição de função**, selecione **Avançar**.
1. Na guia **Condições** da página **Adicionar atribuição de função**, na seção **O que o usuário pode fazer**, selecione a opção **Permitir que o usuário atribua todas as funções (altamente privilegiadas)** e selecione **Avançar**.
1. Na guia **Revisar + atribuir** da página **Adicionar atribuição de função**, selecione **Revisar + atribuir**.

   > **Observação:** em seguida, você atribui permissões ao grupo do Microsoft Entra destinado a conter contas de usuário líder da equipe de desenvolvimento.

1. De volta à página **IAM (controle de acesso) do devcenter-01\|**, no menu de navegação vertical no lado esquerdo, expanda a seção **Gerenciar**, selecione **Projetos** e, na lista de projetos, selecione **devcenter-project-01**.
1. Na página **devcenter-project-01**, no menu de navegação vertical no lado esquerdo, selecione **Controle de Acesso (IAM)**.
1. Na página **IAM (controle de acesso)\| devcenter-project-01**, selecione **+ Adicionar** e, na lista suspensa, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, verifique se a guia **Funções de trabalho** está selecionada, na lista de funções, selecione **Administrador de Projeto do DevCenter** e selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, verifique se a opção **Usuário, grupo ou serviço principal** está selecionada e clique em **+ Selecionar membros**.
1. No painel **Selecionar membros**, pesquise e selecione **`DevCenter_Dev_Leads`** e clique em **Selecionar**.
1. De volta à guia **Membros** da página **Adicionar atribuição de função**, selecione **Avançar**.
1. Na guia **Revisar + atribuir** da página **Adicionar atribuição de função**, selecione **Revisar + atribuir**.

   > **Observação:** por último, atribua permissões ao grupo do Microsoft Entra destinado a conter contas de usuário de desenvolvedor.

1. De volta à página **Controle de acesso (IAM) devcenter-project-01\|**, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, verifique se a guia **Funções de trabalho** está selecionada, na lista de funções, selecione **Usuários do Computador de Desenvolvimento do Centro de Desenvolvimento** e selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, verifique se a opção **Usuário, grupo ou serviço principal** está selecionada e clique em **+ Selecionar membros**.
1. No painel **Selecionar membros**, pesquise e selecione **`DevCenter_Dev_Users`** e clique em **Selecionar**.
1. De volta à guia **Membros** da página **Adicionar atribuição de função**, selecione **Avançar**.
1. Na guia **Revisar + atribuir** da página **Adicionar atribuição de função**, selecione **Revisar + atribuir**.

### Tarefa 7: avaliar um computador de desenvolvimento

Nessa tarefa, você avalia uma funcionalidade de computador de desenvolvimento usando uma conta de usuário de desenvolvedor do Microsoft Entra.

1. Inicie uma aba anônima/privada no navegador da Web e navegue até o portal do desenvolvedor do Computador de Desenvolvimento da Microsoft em `https://aka.ms/devbox-portal`.
1. Quando solicitado a entrar, forneça as credenciais da conta de usuário **devuser01**.
1. Na página **Bem-vindo, devuser01** do portal do desenvolvedor do Computador de Desenvolvimento da Microsoft, selecione **+ Novo Computador de Desenvolvimento**.
1. No painel **Adicionar computador de desenvolvimento**, na caixa de texto **Nome**, insira**`devuser01box01`**
1. Examine outras informações apresentadas no painel **Adicionar um computador de desenvolvimento**, inclusive nome do projeto, especificações do pool do computador de desenvolvimento, status de suporte a hibernação e tempo de desligamento agendado. Além disso, observe a opção de aplicar personalizações e a notificação de que a criação do computador de desenvolvimento pode levar até 65 minutos.

   > **Observação:** os nomes de computadores de desenvolvimento devem ser únicos em um projeto.

1. No painel **Adicionar computador de desenvolvimento**, selecione **Criar**.

   > **Observação:** não espere que o computador de desenvolvimento seja criado. Em vez disso, prossiga diretamente para o próximo exercício deste laboratório. No entanto, considere retornar a este exercício e percorrer o restante desta tarefa depois de concluir o próximo exercício.

1. Depois que o computador de desenvolvimento estiver totalmente provisionado e em execução, conecte-se a ele selecionando a opção **Conectar via aplicativo**.

   > **Observação:** a conectividade com um computador de desenvolvimento pode ser estabelecida usando-se um aplicativo Windows de Área de Trabalho Remota, um cliente de Área de Trabalho Remota (mstsc.exe) ou diretamente em uma janela de navegador da Web.

1. Se você vir o prompt com o título **Este site está tentando abrir a Central de Conexão Remota da Microsoft**, selecione **Abrir**. Isso iniciará automaticamente uma sessão de Área de Trabalho Remota para o computador de desenvolvimento.
1. Quando as credenciais forem solicitadas, autentique-se fornecendo o nome de usuário e a senha da conta **devuser01**.
1. Na sessão da Área de Trabalho Remota para o computador de desenvolvimento, verifique se sua configuração inclui uma instalação de aplicativos do Visual Studio 2022 e do Microsoft 365 Apps.

   > **Observação:** você pode desligar o computador de desenvolvimento diretamente do portal do desenvolvedor do Computador de Desenvolvimento da Microsoft como um usuário de desenvolvimento, selecionando primeiro o símbolo de reticências na interface do **computador de desenvolvimento** e, em seguida, selecionando **Desligar** no menu em cascata. Se preferir, como engenheiro de plataforma ou líder de equipe de desenvolvimento, você pode controlar o ciclo de vida do computador de desenvolvimento na seção **Pools de computadores de desenvolvimento** do projeto do centro de desenvolvimento correspondente.

## Exercício 2: Personalizar um ambiente do Computador de Desenvolvimento da Microsoft

Neste exercício, você personaliza a funcionalidade do ambiente do Computador de Desenvolvimento da Microsoft. Essa abordagem se concentra na extensão das alterações que você pode aplicar ao implementar uma solução de autoatendimento personalizada para desenvolvedores.

O exercício é composto pelas seguintes tarefas:

- Tarefa 1: criar uma galeria de computação do Azure e anexá-la ao centro de desenvolvimento
- Tarefa 2: configurar autenticação e autorização para o Construtor de Imagens do Azure
- Tarefa 3: criar uma image personalizada usando o Construtor de Imagens do Azure
- Tarefa 4: criar uma conexão de rede do centro de desenvolvimento do Azure
- Tarefa 5: adicionar definições de imagem a um projeto do centro de desenvolvimento do Azure
- Tarefa 6: criar um pool de computadores de desenvolvimento personalizados
- Tarefa 7: avaliar um computador de desenvolvimento personalizado

### Tarefa 1: criar uma galeria de computação do Azure e anexá-la ao centro de desenvolvimento

Nesta tarefa, você cria uma galeria de computação do Azure e a anexa ao centro de desenvolvimento criado no exercício anterior deste laboratório. Uma galeria é um repositório armazenado em sua assinatura do Azure que ajuda a criar estrutura e organização em torno de imagens personalizadas. Depois de anexar uma galeria de computação a um centro de desenvolvimento com imagens, você pode criar definições de computador de desenvolvimento com base em imagens armazenadas na galeria de computação.

1. Inicie um navegador da Web e navegue até o portal do Azure em `https://portal.azure.com`.
1. Quando precisar se autenticar, entre usando a sua conta Microsoft.
1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **`Azure compute galleries`**.
1. Nas **galerias de computação do Azure**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar galeria de computação do Azure**, especifique as seguintes configurações e selecione **Avançar: Método de compartilhamento**:

   | Configuração        | Valor                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Assinatura   | O nome da assinatura do Azure que você está usando neste laboratório |
   | Grupo de recursos | **rg-devcenter-01**                                          |
   | Nome           | **compute_gallery_01**                                       |
   | Região         | **(EUA) Leste dos EUA**                                             |

1. Na guia **Método de compartilhamento** da página **Criar galeria de computação do Azure**, deixe a opção padrão **Controle de acesso baseado em função (RBAC)** selecionada e, em seguida, selecione **Revisar + Criar**.
1. Na guia **Examinar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   > **Observação:** aguarde até que o projeto seja provisionado. A criação da galeria de computação do Azure deve levar menos de 1 minuto.

1. No portal do Azure, pesquise e selecione **`Dev centers`** e, na página **Centros de desenvolvimento**, selecione **devcenter-01**.
1. Na página **devcenter-01**, no menu de navegação vertical no lado esquerdo, expanda a seção **Configuração do computador de desenvolvimento** e selecione **Galerias de computação do Azure**.
1. Na página **galerias de computação do Azure devcenter-01 \|**, selecione **+ Adicionar**.
1. No painel **Adicionar galeria de computação do Azure**, na lista suspensa **Galeria**, selecione **compute_gallery_01** e, em seguida, selecione **Adicionar**.

   > **Observação:** se você receber uma mensagem de erro: "_Este centro de desenvolvimento não tem uma identidade atribuída pelo sistema ou pelo usuário. Galerias não podem ser adicionadas até que uma identidade tenha sido atribuída._" é preciso atribuir uma identidade atribuída pelo sistema ao centro de desenvolvimento.
   > Para fazer isso, no portal do Azure, na página **devcenter-01**, no menu de navegação vertical no lado esquerdo, selecione **Identidade** em Configurações, na guia **Atribuído pelo sistema**, defina a opção **Status** como **Ativado** e selecione **Salvar**.

### Tarefa 2: configurar autenticação e autorização

Nessa tarefa, você cria uma identidade gerenciada atribuída pelo usuário que será usada pelo Construtor de Imagens do Azure para adicionar imagens à galeria de computação do Azure criada na tarefa anterior. Você também configura as permissões necessárias criando uma função RBAC (controle de acesso baseado em função) personalizada e atribuindo-a à identidade gerenciada. Isso permite que você use o Construtor de Imagens do Azure na próxima tarefa para criar uma imagem personalizada.

1. No portal do Azure, selecione o ícone da barra de ferramentas do **Cloud Shell** para abrir o painel do Cloud Shell e, se necessário, selecione **Alternar para o PowerShell** para iniciar uma sessão do PowerShell e, na caixa de diálogo **Alternar para o PowerShell no Cloud Shell**, selecione **Confirmar**.

   > **Observação:** se esta for a primeira vez que você está abrindo o Cloud Shell, na caixa de diálogo **Bem-vindo ao Azure Cloud Shell**, selecione **PowerShell**, no painel **Introdução**, selecione a opção **Nenhuma Conta de Armazenamento necessária** e, na lista suspensa **Assinatura**, selecione o nome da assinatura do Azure que você está usando neste laboratório.

1. Na sessão do PowerShell do painel do Cloud Shell, execute os seguintes comandos para garantir que todos os provedores de recursos necessários estejam registrados:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
   Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
   Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
   Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
   Register-AzResourceProvider -ProviderNamespace Microsoft.Network
   ```

1. Execute o seguinte comando para instalar os módulos do PowerShell necessários (quando solicitado, digite **A** e pressione a tecla **Enter**):

   ```powershell
   'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
   ```

1. Execute os seguintes comandos para configurar variáveis que serão referenciadas em todo o processo de criação da imagem:

   ```powershell
   $currentAzContext = Get-AzContext
   # the target Azure subscription ID
   $subscriptionID=$currentAzContext.Subscription.Id
   # the target Azure resource group name
   $imageResourceGroup='rg-devcenter-01'
   # the target Azure region
   $location='eastus'
   # the reference name assigned to the image created by using the Azure Image Builder service
   $runOutputName="aibWinImg01"
   # image template name
   $imageTemplateName="templateWinVSCode01"
   # the Azure compute gallery name
   $computeGallery = 'compute_gallery_01'
   ```

1. Execute os seguintes comandos para criar uma identidade gerenciada atribuída pelo usuário (o Construtor de Imagens de VM usa a identidade de usuário fornecida para armazenar imagens na Galeria de Computação do Azure de destino):

   ```powershell
   # Install the Azure PowerShell module to support AzUserAssignedIdentity
   Install-Module -Name Az.ManagedServiceIdentity
   # Generate a pseudo-random integer to be used for resource names
   $timeInt=$(get-date -UFormat "%s")

   # Create an identity
   $identityName='identityAIB' + $timeInt
   New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName -Location $location
   $identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
   $identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
   ```

1. Execute os seguintes comandos para conceder à identidade gerenciada atribuída pelo usuário recém-criada as permissões necessárias para armazenar imagens no grupo de recursos **rg-devcenter-01**:

   ```powershell
   # Set variables
   $imageRoleDefName = 'Custom Azure Image Builder Image Def' + $timeInt
   $aibRoleImageCreationUrl = 'https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json'
   $aibRoleImageCreationPath = 'aibRoleImageCreation.json'

   # Customize the role definition file
   Invoke-WebRequest -Uri $aibRoleImageCreationUrl -OutFile $aibRoleImageCreationPath -UseBasicParsing
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<subscriptionID>', $subscriptionID) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<rgName>', $imageResourceGroup) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $aibRoleImageCreationPath

   # Create a role definition
   New-AzRoleDefinition -InputFile  ./aibRoleImageCreation.json

   # Assign the role to the VM Image Builder user-assigned managed identity within the scope of the **rg-devcenter-01** resource group
   New-AzRoleAssignment -ObjectId $identityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
   ```

### Tarefa 3: criar uma image personalizada usando o Construtor de Imagens do Azure

Nessa tarefa, você usa o Construtor de Imagens do Azure para criar uma imagem personalizada com base em um modelo existente do ARM (Azure Resource Manager) que define uma imagem do Windows 11 Enterprise com o Choco e o Visual Studio Code instalados automaticamente. O Construtor de Imagens de VM do Azure simplifica consideravelmente o processo de definição e provisionamento de imagens de VM. Ele depende de uma configuração de imagem que você especifica para configurar um pipeline de imagem automatizado. Posteriormente, os desenvolvedores poderão usar essas imagens para provisionar seus computadores de desenvolvimento.

1. Na sessão do PowerShell do painel do Cloud Shell, execute os seguintes comandos para criar uma definição de imagem a ser adicionada à galeria de computação do Azure que você criou na primeira tarefa deste exercício:

   ```powershell
   # ensure that the image definition security type property is set to 'TrustedLaunch'
   $securityType = @{Name='SecurityType';Value='TrustedLaunch'}
   $features = @($securityType)
   # Image definition name
   $imageDefName = 'imageDefDevBoxVSCode01'

   # Create the image definition
   New-AzGalleryImageDefinition -GalleryName $computeGallery -ResourceGroupName $imageResourceGroup -Location $location -Name $imageDefName -OsState generalized -OsType Windows -Publisher 'Contoso' -Offer 'vscodedevbox' -Sku '1-0-0' -Feature $features -HyperVGeneration 'V2'
   ```

   > **Observação:** uma imagem de computador de desenvolvimento deve atender a vários requisitos, inclusive uso de Geração 2, Hyper-V v2 e Windows 10 ou 11 Enterprise versão 20H2 ou posterior. Para obter a lista completa, consulte o artigo do Microsoft Learn [Configurar a Galeria de Computação do Azure para o Computador de Desenvolvimento da Microsoft](https://learn.microsoft.com/en-us/azure/dev-box/how-to-configure-azure-compute-gallery).

1. Execute os seguintes comandos para criar um arquivo vazio chamado template.json, que conterá um modelo do ARM que define uma imagem do Windows 11 Enterprise com Choco e Visual Studio Code instalados automaticamente:

   ```powershell
   Set-Location -Path ~
   $templateFile = 'template.json'
   Set-Content -Path $templateFile -Value ''
   ```

1. Na sessão do PowerShell do Cloud Shell, use o editor de nano texto para adicionar o seguinte conteúdo ao arquivo recém-criado:

   > **Observação:** para abrir o editor de texto nano, execute o comando `nano ./template.json`. Para salvar as alterações e sair do editor de texto nano, pressione **Ctrl+X**, depois **Y** e, finalmente, **Enter**.

   ```json
   {
     "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "imageTemplateName": {
         "type": "string"
       },
       "api-version": {
         "type": "string"
       },
       "svclocation": {
         "type": "string"
       }
     },
     "variables": {},
     "resources": [
       {
         "name": "[parameters('imageTemplateName')]",
         "type": "Microsoft.VirtualMachineImages/imageTemplates",
         "apiVersion": "[parameters('api-version')]",
         "location": "[parameters('svclocation')]",
         "dependsOn": [],
         "tags": {
           "imagebuilderTemplate": "win11multi",
           "userIdentity": "enabled"
         },
         "identity": {
           "type": "UserAssigned",
           "userAssignedIdentities": {
             "<imgBuilderId>": {}
           }
         },
         "properties": {
           "buildTimeoutInMinutes": 100,
           "vmProfile": {
             "vmSize": "Standard_DS2_v2",
             "osDiskSizeGB": 127
           },
           "source": {
             "type": "PlatformImage",
             "publisher": "MicrosoftWindowsDesktop",
             "offer": "Windows-11",
             "sku": "win11-21h2-ent",
             "version": "latest"
           },
           "customize": [
             {
               "type": "PowerShell",
               "name": "Install Choco and Vscode",
               "inline": [
                 "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))",
                 "choco install -y vscode"
               ]
             }
           ],
           "distribute": [
             {
               "type": "SharedImage",
               "galleryImageId": "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>",
               "runOutputName": "<runOutputName>",
               "artifactTags": {
                 "source": "azureVmImageBuilder",
                 "baseosimg": "win11multi"
               },
               "replicationRegions": ["<region1>", "<region2>"]
             }
           ]
         }
       }
     ]
   }
   ```

1. Execute os seguintes comandos para substituir espaços reservados no template.json pelos valores específicos do seu ambiente do Azure:

   ```powershell
   $replRegion2 = 'eastus2'
   $templateFilePath = '.\template.json'
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<subscriptionID>', $subscriptionID | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<rgName>', $imageResourceGroup | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<runOutputName>', $runOutputName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<imageDefName>', $imageDefName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<sharedImageGalName>', $computeGallery | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region1>', $location | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region2>', $replRegion2 | Set-Content -Path $templateFilePath
   ((Get-Content -Path $templateFilePath -Raw) -Replace '<imgBuilderId>', $identityNameResourceId) | Set-Content -Path $templateFilePath
   ```

1. Execute o seguinte comando para enviar o modelo ao serviço Construtor de Imagens do Azure (o serviço processa o modelo enviado baixando todos os artefatos dependentes, como scripts, e armazenando-os em um grupo de recursos de preparo, cujo nome inclui o prefixo **IT\_**, para criar uma imagem de máquina virtual personalizada:

   ```powershell
   New-AzResourceGroupDeployment -ResourceGroupName $imageResourceGroup -TemplateFile $templateFilePath -Api-Version "2020-02-14" -imageTemplateName $imageTemplateName -svclocation $location
   ```

1. Execute o seguinte comando para invocar o processo de criação da imagem:

   ```powershell
   Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Run -Force
   ```

1. Execute o seguinte comando para determinar o estado de provisionamento da imagem:

   ```powershell
   Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup | Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState
   ```

   > **Observação:** a seguinte saída indica que o processo de compilação foi concluído com êxito:

   ```powershell
   Name                LastRunStatusRunState LastRunStatusMessage ProvisioningState
   ----                --------------------- -------------------- -----------------
   templateWinVSCode01 Succeeded                                  Succeeded
   ```

1. Se preferir, para monitorar o progresso da compilação, use o seguinte procedimento:

   1. No portal do Azure, pesquise e selecione **`Image templates`**.
   1. Na página **Modelos de imagem**, selecione **templateWinVSCode01**.
   1. Na página **templateWinVSCode01**, na seção **Essentials**, observe o valor do item **Estado de execução da compilação**.

   > **Observação:** o processo de compilação pode levar cerca de 30 minutos. Não espere a conclusão, prossiga para a próxima tarefa deste exercício.

1. Depois que a compilação for concluída, no portal do Azure, pesquise e selecione **`Azure compute galleries`**.
1. Na página **Galerias de computação do Azure**, selecione **compute_gallery_01**.
1. Na página **compute_gallery_01**, verifique se a guia **Definições** está selecionada e, na lista de definições, selecione **imageDefDevBoxVSCode**.
1. Na página **imageDefDevBoxVSCode**, selecione a guia **Versões** e verifique se o item **1.0.0 (versão mais recente)** aparece na lista com o **Estado de Provisionamento** definido como **Bem-sucedido**.
1. Selecione o item **1.0.0 (versão mais recente)**.
1. Na página **1.0.0 (compute_gallery_01/imageDefDevBoxVSCode/1.0.0)**, examine as configurações de versão da imagem da VM.

### Tarefa 4: criar uma conexão de rede do centro de desenvolvimento do Azure

Nesta tarefa, você configura a rede do centro de desenvolvimento do Azure para ser usada em um cenário que requer conectividade privada com recursos hospedados em uma rede virtual do Azure. Ao contrário da rede hospedada da Microsoft que você usou no primeiro exercício deste laboratório, as conexões de rede virtual também dão suporte a cenários híbridos (fornecendo conectividade a recursos locais) e ao ingresso híbrido de computadores de desenvolvimento do Azure no Microsoft Entra (além do suporte para ingresso no Microsoft Entra).

1. Na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **`Virtual networks`**.
1. Na página **Redes virtuais**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar rede virtual**, especifique as seguintes configurações e selecione **Avançar**:

   | Configuração        | Valor                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Assinatura   | O nome da assinatura do Azure que você está usando neste laboratório |
   | Grupo de recursos | **rg-devcenter-01**                                          |
   | Nome           | **vnet-01**                                                  |
   | Location       | **(EUA) Leste dos EUA**                                             |

1. Na guia **Segurança** da página **Criar rede virtual**, examine as configurações existentes sem alterar seus valores padrão e selecione **Avançar**.
1. Na guia **Endereços IP** da página **Criar rede virtual**, examine as configurações existentes sem alterar seus valores padrão e selecione **Revisar + Criar**.
1. Na guia **Revisar + Criar** da página **Criar rede virtual**, selecione **Criar**.

   > **Observação:** aguarde até que a rede virtual seja criada. Isso deverá levar menos de 1 minuto.

1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **`Network connections`**.
1. Na página **Conexões de rede**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar conexão de rede**, especifique as seguintes configurações e selecione **Revisar + Criar**:

   | Configuração         | Valor                                                        |
   | --------------- | ------------------------------------------------------------ |
   | Assinatura    | O nome da assinatura do Azure que você está usando neste laboratório |
   | Grupo de recursos  | **rg-devcenter-01**                                          |
   | Nome            | **network-connection-vnet-01**                               |
   | Rede virtual | **vnet-01**                                                  |
   | Sub-rede          | **default**                                                  |

1. Na guia **Revisar + Criar** da página **Criar rede virtual**, selecione **Criar**.

   > **Observação:** aguarde até a conexão de rede ser criada. Isso pode levar cerca de um minuto.

1. No portal do Azure, pesquise e selecione **`Dev centers`** e, na página **Centros de desenvolvimento**, selecione **devcenter-01**.
1. Na página **devcenter-01**, no menu de navegação vertical no lado esquerdo, expanda a seção **Configuração do computador de desenvolvimento** e selecione **Rede**.
1. Na página **Rede do devcenter-01\|**, selecione **+ Adicionar**.
1. No painel **Adicionar conexão de rede**, na lista suspensa **Conexão de rede**, selecione **network-connection-vnet-01** e, em seguida, selecione **Adicionar**.

   > **Observação:** não espere que a conexão de rede seja criada, prossiga para a próxima tarefa. Adicionar uma conexão de rede pode levar cerca de 1 minuto,

### Tarefa 5: adicionar definições de imagem a um projeto do centro de desenvolvimento do Azure

Nesta tarefa, você adiciona definições de imagem a um projeto do centro de desenvolvimento do Azure criado no primeiro exercício deste laboratório. As definições de imagem combinam um Azure Marketplace ou uma imagem personalizada com tarefas configuráveis que definem modificações adicionais a serem aplicadas à imagem subjacente. Uma definição de imagem pode ser usada para criar uma nova imagem (contendo todas as alterações, inclusive aquelas aplicadas por tarefas) ou para criar pools de computadores de desenvolvimento diretamente. A criação de uma imagem reutilizável minimiza o tempo necessário para provisionamento do computador de desenvolvimento.

Para configurar a geração de imagens para personalizações de equipe do Computador de Desenvolvimento da Microsoft, os catálogos no nível do projeto devem ser habilitados (o que você já concluiu no primeiro exercício deste laboratório). Nesta tarefa, você define as configurações de sincronização de catálogo para o projeto. Isso envolve anexar um catálogo que contenha arquivos de definição de imagem.

1. Na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **`Dev centers`**.
1. Na página **Centros de desenvolvimento**, selecione **devcenter-01**.
1. Na página **devcenter-01**, no menu de navegação vertical no lado esquerdo, na seção **Gerenciar**, selecione **Projetos**.
1. Na página **Projetos devcenter-01\|**, na lista de projetos, selecione **devcenter-project-01**.
1. Na página **devcenter-project-01**, no menu de navegação vertical no lado esquerdo, expanda a seção **Configurações** e selecione **Catálogos**.
1. Na página **Catálogos devcenter-project-01\|**, selecione **+ Adicionar**.
1. No painel **Adicionar catálogo**, na caixa de texto **Nome**, insira **`image-definitions-01`**, na seção **Origem do catálogo**, selecione **GitHub**, no **Tipo de autenticação**, selecione **Aplicativo GitHub**, deixe a caixa de seleção **Sincronizar automaticamente este catálogo** habilitada e selecione **Entrar com o GitHub**.
1. Se for solicitado, na janela **Entrar com o GitHub**, insira suas credenciais do GitHub e selecione **Entrar**.

   > **Observação:** você precisa fazer um fork dohttps://github.com/MicrosoftLearning/contoso-co-eShop repositório para sua conta do GitHub antes de concluir esta etapa.

1. Se for solicitado, na janela **Autorizar Microsoft DevCenter**, selecione **Autorizar Microsoft DevCenter**.
1. De volta ao painel **Adicionar catálogo**, na lista suspensa **Repositório**, selecione **contoso-co-eShop**, na lista suspensa **Ramificação**, aceite o item **Ramificação padrão**, no **Caminho da pasta**, insira **`.devcenter/catalog/image-definitions`** e selecione **Adicionar**.
1. De volta à página **Catálogos devcenter-project-01\|**, verifique se a sincronização foi concluída com êxito monitorando o item na coluna **Status**.
1. Selecione o link **Sincronização bem-sucedida** na coluna **Status**, examine o painel de notificação resultante, verifique se 3 itens foram adicionados ao catálogo e feche o painel selecionando o símbolo **x** no canto superior direito.
1. De volta à página **Catálogos devcenter-project-01\|**, selecione **image-definitions-01** e verifique se ela contém três itens chamados **ContosoBaseImageDefinition**, **backend-eng** e **frontend-eng**.

   > **Observação:** para os fins deste laboratório, teste a funcionalidade da definição de imagem **frontend-eng**, que tem o seguinte conteúdo:

   ```yaml
   $schema: "1.0"
   name: "frontend-eng"
   # Using the "Windows 11 Enterprise 24H2" image as the base
   image: microsoftwindowsdesktop_windows-ent-cpc_win11-24H2-ent-cpc
   description: "This definition is for the eShop frontend engineering environment"

   tasks:
     - name: ~/winget
       description: Install Visual Studio Code
       parameters:
         package: Microsoft.VisualStudioCode
         runAsUser: true
   ```

1. No portal do Azure, navegue de volta para a página **devcenter-project-01**, no menu de navegação vertical no lado esquerdo, expanda a seção **Gerenciar**, selecione **Definições de imagem** e verifique se a página exibe as mesmas 3 definições de imagem identificadas anteriormente nesta tarefa.

### Tarefa 6: criar um pool de computadores de desenvolvimento personalizados

Nesta tarefa, você usa as definições de imagem recém-provisionadas para criar um pool de computadores de desenvolvimento. O pool também utiliza a conexão de rede que você configurou anteriormente neste exercício.

1. No portal do Azure que exibe a página de **definições de imagem devcenter-project-01\|**, no menu de navegação vertical no lado esquerdo, na seção **Gerenciar**, selecione **pools de computadores de desenvolvimento**.
1. Na página de pools de computadores de desenvolvimento **devcenter-project-01\|**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um computador de desenvolvimento**, especifique as seguintes configurações e selecione **Criar**:

   | Configuração                                                                                                           | Valor                                                 |
   | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
   | Nome                                                                                                              | **devcenter-project-01-devbox-pool-02**               |
   | Definição                                                                                                        | **frontend-eng**                                      |
   | Computação                                                                                                           | **8 vCPU, 32 GB de RAM**                                 |
   | Armazenamento                                                                                                           | **SSD de 256GB**                                        |
   | Conexão de rede                                                                                                | **Implantar em uma conexão de rede na minha organização** |
   | Nome da conexão de rede                                                                                           | **network-connection-vnet-01**                        |
   | Habilitar logon único                                                                                             | Habilitado                                               |
   | Privilégios de Criador do Computador de Desenvolvimento                                                                                        | **Administrador Local**                               |
   | Ativar parada automática dentro do cronograma                                                                                      | Habilitado                                               |
   | Hora de término                                                                                                         | **19:00**                                          |
   | Fuso horário                                                                                                         | Seu fuso horário atual                                |
   | Ativar hibernação em caso de desconexão                                                                                    | Habilitado                                               |
   | Período de cortesia em minutos                                                                                           | **60**                                                |
   | Confirmo que a minha organização tem licenças de Benefícios Híbridos do Azure, que se aplicam a todos os computadores de desenvolvimento neste pool | Habilitado                                               |

   > **Observação:** aguarde até que o pool de computadores de desenvolvimento seja criado. Isso pode levar cerca de dois minutos.

### Tarefa 7: avaliar um computador de desenvolvimento personalizado

Nesta tarefa, você avalia a funcionalidade de um computador de desenvolvimento personalizado usando uma conta de usuário desenvolvedor do Microsoft Entra.

> **Observação:** considerando-se o tempo extra necessário para se concluir esta tarefa, sua conclusão é opcional.

1. Inicie uma aba anônima/privada no navegador da Web e navegue até o portal do desenvolvedor do Computador de Desenvolvimento da Microsoft em `https://aka.ms/devbox-portal`.
1. Quando solicitado a entrar, forneça as credenciais da conta de usuário **devuser01**.
1. Na página **Bem-vindo, devuser01** do portal do desenvolvedor do Computador de Desenvolvimento da Microsoft, selecione **+ Novo Computador de Desenvolvimento**.
1. No painel **Adicionar um computador de desenvolvimento**, na caixa de texto **Nome**, insira **`devuser01box02`**.
1. Na lista suspensa **Pool de Computadores de Desenvolvimento**, selecione **devcenter-project-01-devbox-pool-02**.
1. Examine outras informações apresentadas no painel **Adicionar um computador de desenvolvimento**, inclusive nome do projeto, especificações do pool do computador de desenvolvimento, status de suporte a hibernação e tempo de desligamento agendado. Além disso, observe a opção de aplicar personalizações e a notificação de que a criação do computador de desenvolvimento pode levar até 65 minutos.
1. No painel **Adicionar computador de desenvolvimento**, selecione **Criar**.
1. Depois que o computador de desenvolvimento estiver totalmente provisionado e em execução, conecte-se a ele selecionando a opção **Conectar via aplicativo**.
1. Se você vir o prompt com o título **Este site está tentando abrir a Central de Conexão Remota da Microsoft**, selecione **Abrir**. Isso iniciará automaticamente uma sessão de Área de Trabalho Remota para o computador de desenvolvimento.
1. Quando as credenciais forem solicitadas, autentique-se fornecendo o nome de usuário e a senha da conta **devuser01**.
1. Na sessão da Área de Trabalho Remota para o computador de desenvolvimento, verifique se sua configuração inclui uma instalação do Visual Studio Code.

   > **Observação:** você também pode validar o resultado da personalização selecionando o símbolo de reticências na interface do **seu computador de desenvolvimento** e, em seguida, selecionando **Personalizações** no menu em cascata.

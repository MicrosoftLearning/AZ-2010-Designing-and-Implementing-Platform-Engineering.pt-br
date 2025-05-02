---
lab:
  title: Implementar monitoramento em tempo real com o Azure Monitor
  module: Observability and Continuous Improvement
---

# Laboratório 02: implementar monitoramento em tempo real com o Azure Monitor

## Tempo estimado: 20 minutos

## Pré-requisitos

- Assinatura do Azure - Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/).
- Conhecimento básico dos conceitos de monitoramento e observabilidade.
- Compreensão básica de Serviços do Azure.

## Objetivos

- Criar um aplicativo Web de exemplo.
- Habilite e configure o Azure Monitor e o Application Insights.
- Crie painéis personalizados para visualizar as métricas do aplicativo.
- Configure alertas para notificar as partes interessadas sobre anomalias de desempenho.
- Analise os dados de desempenho para possíveis melhorias.

## Exercício 1: criar um aplicativo Web de exemplo

Como engenheiro de plataforma, você precisa garantir que os aplicativos em execução na sua plataforma sejam observáveis e monitorados continuamente. Neste laboratório, você vai criar um aplicativo Web de exemplo para o Azure, configurar o Azure Monitor e o Application Insights, configurar painéis personalizados e criar alertas para acompanhar o desempenho do aplicativo.

### Tarefa 1: criar um aplicativo Web do Azure

1. Abra o Portal do Azure.
1. Na barra de pesquisa, digite e selecione Serviços de Aplicativos.
1. Clique em + Criar e selecione Aplicativo Web.
1. Na guia Básico:
   - Assinatura: Selecione sua assinatura do Azure.
   - Grupo de recursos: clique em Criar novo, insira **`monitoringlab-rg`** e clique em OK.
   - Nome: digite um nome único, como **`monitoringlab-webapp`**.
   - Publicar: selecione Código.
   - Pilha de runtime: escolha .NET 8 (LTS).
   - Região: selecione uma região próxima de você.
1. Clique em Revisar + Criar e em Criar.
1. Aguarde a conclusão da implantação e selecione "Ir para o recurso".
1. Na guia Visão geral, clique na URL para verificar se o aplicativo Web está em execução.

### Tarefa 2: verificar o Application Insights

1. No recurso Aplicativo Web, role até a seção Monitoramento.
1. Clique em Application Insights.
1. O Application Insights já está habilitado para este aplicativo Web. Clique no link para abrir o recurso Application Insights.
1. No recurso Application Insights, clique no Painel do Aplicativo para exibir os dados de desempenho fornecidos pelo painel padrão.

## Exercício 2: configurar o Azure Monitor e os painéis

### Tarefa 1: acessar o Azure Monitor

1. No Portal do Azure, procure pela palavra Monitor e clique nela.
1. Clique em Métricas no painel esquerdo.
1. Na seção Escopo, selecione o aplicativo Web na assinatura e no grupo de recursos em que você implantou o aplicativo Web.
1. Clique em Aplicar e observe as métricas disponíveis para o aplicativo Web.

### Tarefa 2: adicionar métricas-chave ao painel

1. Na seção Escopo, selecione o seu aplicativo Web (monitoringlab-webapp).
1. Em Métrica, escolha Tempo de resposta.
1. Defina a Agregação como Média e clique em + Adicionar métrica.
1. Repita o processo com outras métricas.
   - Tempo de CPU (contagem)
   - Solicitações (média)
1. Clique no painel e fixe no painel.
1. Selecione o tipo Compartilhado e selecione a assinatura e o painel monitoringlab-webapp.
1. Clique em Fixar no painel.
1. Clique no ícone do painel no painel esquerdo para exibir o painel.
1. Verifique se as métricas são exibidas no painel e estão sendo atualizadas em tempo real.

## Exercício 3: criar alertas

### Tarefa 1: definir condições e ações de alerta

1. No Azure Monitor, clique em Alertas.
1. Clique em Criar + e selecione Regra de alerta.
1. Em Escopo, selecione seu aplicativo Web (monitoringlab-webapp) e clique em Aplicar.
1. Em Condição, clique no campo Nome do sinal e selecione Tempo de resposta.
1. Configure a regra de alerta:
   - Tipo de limite: dinâmico
   - Tipo de agregação: Média
   - O valor é: maior ou menor que
   - Sensibilidade do limite: alta
   - Quando avaliar: verifique a cada 1 minuto e olhe para trás em 5 minutos.
1. Em Ações, selecione Usar ações rápidas.
1. Digite:
   - Nome do grupo de ações: WebAppMonitoringAlerts
   - Nome de exibição: WebAlert
   - Email: insira o seu endereço de email.
1. Clique em Save (Salvar).
1. Clique em Avançar: Detalhes.
1. Insira um Nome `WebAppResponseTimeAlert` e selecione Nível de gravidade Detalhado.
1. Clique em Examinar + criar e depois Criar.

   > **Observação:** sua regra de alerta agora está criada e acionará uma notificação por email quando o tempo de resposta exceder o limite. Você pode forçar o disparo do alerta enviando um grande número de solicitações para o aplicativo Web. Por exemplo, você pode usar o teste de carga do Azure ou uma ferramenta como o Apache JMeter.

1. Volte para Azure Monitor > Alertas.
1. Clique em Regras de alerta e veja a regra de alerta que você criou.

## Exercício 4: analisar dados de desempenho

### Tarefa 1: revisar métricas coletadas

1. No portal do Azure, acesse Application Insights.
1. Clique no Painel do Aplicativo.
1. Clique no bloco Desempenho para analisar os tempos de resposta e a carga do servidor. Você também pode exibir o número de solicitações e solicitações com falha.
1. Clique em Analisar com pastas de trabalho e selecione Análise do contador de desempenho.
1. Clique em Pastas de trabalho.
1. Selecione a pasta de trabalho Análise de Desempenho em Desempenho.
1. Você pode ver os dados de desempenho do aplicativo Web.

> **Observação:** você pode personalizar a pasta de trabalho para incluir métricas e filtros adicionais.

<h1 align="center"> Plano de Testes: Sistema Zero-E-Mission </h1>

## 1. Identificação do Projeto

<div align="center"> 

  | Item                     | Informação                    |
  | ------------------------ | ----------------------------- |
  | **Nome do Projeto**      | Zero-E-Mission                |
  | **Versão**               | 1.0                           |
  | **Equipe Responsável**   | Maycon Siqueira / Pedro Moura |
  | **Data do Planejamento** | 08/08/2026                    |
  
</div>

---

## 2. Objetivos dos Testes

### O que a equipe pretende verificar?

A equipe tem como objetivo principal validar a estabilidade, a precisão e a responsividade da arquitetura MVVM construída para o sistema Zero-E-Mission. Especificamente, os testes focarão em verificar:

- **Precisão da Qualidade do Combustível:** Avaliar se o componente *Analyser* e o *ClassificationService* estão processando corretamente as matrizes de dados e classificando a qualidade do combustível dentro das faixas de tolerância exatas estipuladas pelas regras de negócio.
- **Sincronia do Módulo VISCOM:** Verificar se a captura e a conversão de frames em tempo real (utilizando a classe *VisionService* em conjunto com o *MatHelpers*) ocorrem sem sobrecarregar a thread principal da interface (UI), garantindo que o vídeo seja processado de forma fluida e sem "engasgos".
- **Comunicação do Módulo Tradicional:** Testar a resiliência do sistema ao alternar entre o recebimento de dados reais de sensores físicos e a injeção de dados simulados (Modo Simulação), validando se as lógicas de negócio respondem de forma idêntica a ambos os estímulos.
- **Persistência e Integridade de Dados:** Assegurar que cada análise, perfil de calibração (*CalibrationProfile*) e amostra de referência (*CalibrationSample*) sejam corretamente gravados e recuperados através do *AppDbContext* e *ResultadosDbContext* (banco de dados SQLite), sem perda de informações ou corrupção de tabelas.
- **Resiliência de Background:** Certificar-se de que o *WindowsService* consegue manter as regras de monitoramento operando sem interrupções, mesmo que a interface gráfica (WPF) não esteja em exibição

### Quais riscos pretende reduzir?

Sendo um projeto cujo sucesso está atrelado à otimização operacional e ao impacto ambiental positivo, os testes visam mitigar riscos críticos que poderiam comprometer a credibilidade do sistema Zero-E-Mission:

- **Risco de Falsos Positivos/Negativos:** O maior risco do projeto é o sistema aprovar um combustível fora dos padrões ou reprovar uma operação normal. Os testes rigorosos de Valor Limite no motor de classificação buscam anular esse risco.
- **Congelamento da Interface (Deadlocks):** Como a aplicação lida com fluxos de vídeo no Módulo VISCOM e atualizações constantes na *JanelaGaugeViewModel*, existe o risco da interface congelar (Crash/Freeze). Os testes reduzirão esse risco garantindo o uso correto de programação assíncrona.
- **Perda de Calibração durante a Execução:** Um risco grave é o sistema "esquecer" os parâmetros de calibração (*CalibrationService*) ao alternar entre as abas Tradicional e VISCOM, o que invalidaria as métricas coletadas na sequência.
- **Falhas Silenciosas:** Reduzir o risco de erros de hardware (como a desconexão da câmera ou do sensor) ocorrerem sem que o usuário seja alertado, garantindo que o sistema sempre informe o status real da conexão.

### O que será considerado como evidência de qualidade?

Para que o sistema seja considerado apto para operação contínua e demonstre excelência técnica perante a banca avaliadora, os seguintes critérios serão aceitos como evidência de qualidade:

- **Responsividade Visual Imediata:** A atualização visual dinâmica através do *SeverityToColorConverter*, garantindo que a Janela Gauge mude de cor instantaneamente e sem atrasos assim que um limite de emissão ou falha de qualidade for detectado pelo sistema.
- **Auditoria 100% Rastreável:** A presença de logs detalhados, íntegros e sem falhas de gravação, gerados ativamente pelo *Logger* e gerenciados pelo *ILogService* e *LogEntry*, comprovando que todos os eventos do sistema (erros, calibrações e mudanças de status) estão documentados para suporte.
- **Troca de Contexto Estável:** A alternância entre o Módulo Tradicional e o Módulo VISCOM ocorrendo de forma limpa na *MainWindow*, mantendo o estado global coerente através do *MainViewModel* e do *BaseViewModel*, sem duplicação de processos na memória ou instâncias "fantasmas" de câmeras abertas.
- **Tratamento de Exceções Elegante:** A ausência absoluta de fechamentos abruptos (crashes). Qualquer erro (como banco de dados inacessível ou falha na leitura matricial) deve resultar em uma notificação amigável na tela do usuário, com o registro silencioso do erro via serviço de logs (*LogsViewModel*).

---

## 3. Escopo

O escopo define exatamente as fronteiras da atuação da equipe de testes, estabelecendo com clareza quais componentes da arquitetura MVVM construída em C# e WPF passarão por escrutínio e quais elementos externos estão fora da responsabilidade de validação deste ciclo.

### O que será testado

A fase de testes cobrirá exaustivamente as camadas lógicas, visuais e de persistência do sistema Zero-E-Mission, com foco nos seguintes componentes e fluxos operacionais:

- **Módulo Tradicional e Integração de Sensores:** Será testada toda a cadeia de recepção e tratamento de dados físicos, garantindo o funcionamento integrado do *SensorInterface* e a injeção de dados sintéticos através do *SensorSimulator*. O objetivo é validar como a aplicação reage a leituras dinâmicas de qualidade do combustível e se a interface reflete essas métricas em tempo real.
- **Módulo VISCOM (Visão Computacional):** Avaliação profunda do motor de processamento de imagens, testando a estabilidade da classe *VisionService* durante a captura contínua de frames. Serão validados os cálculos matriciais e o tratamento de imagens executados pelo *MatHelpers*, assegurando que a conversão visual não estoure o consumo de memória (*Memory Leaks*).
- **Camada de Persistência e Banco de Dados (SQLite):** Validação integral da gravação, leitura e atualização de dados utilizando o **Entity Framework**. Os testes abrangerão o contexto de aplicação (*AppDbContext*) e o contexto específico de resultados (*ResultadosDbContext*), garantindo que as classes *AnalysisRecord* e *AnalysisResult* sejam persistidas corretamente no SQLite pela classe *ResultadosService*, sem corrupção de dados sob alto volume de registros.
- **Serviços de Classificação e Lógica de Negócio:** Testes das regras matemáticas e estatísticas embutidas no *Analyser* e no *ClassificationService*. Serão aplicadas baterias de testes com diferentes parâmetros para atestar a exatidão no diagnóstico das emissões.
- **Integração Visual e Comandos (Arquitetura MVVM):** Serão testados todos os vínculos (*Bindings*) entre as *Views* (arquivos .xaml) e as *ViewModels*. Isso inclui validar se os cliques na interface disparam corretamente o *RelayCommand*, verificar o gerenciamento global no *MainViewModel* e no *ZeroEMissionViewModel*, e atestar o comportamento visual dinâmico gerado pelo *SeverityToColorConverter* e pela *JanelaGaugeViewModel*.
- **Auditoria, Logs e Background:** Validação do rastreamento de operações através do *Logger* e da interface *ILogService*, assegurando que cada *LogEntry* seja guardado. Também será testada a resiliência do *WindowsService* para manter a plataforma rodando de forma invisível no sistema operacional.
- **Motor de Calibração:** Teste rigoroso do ciclo de vida das calibrações de base do sistema, assegurando que os módulos *CalibrationService*, *CalibrationProfile* e *CalibrationSample* consigam gerar e manter parâmetros de referência exatos.

### O que não será testado

Para garantir o foco da equipe e o cumprimento dos prazos de entrega, os seguintes aspectos não farão parte desta rodada de testes sistêmicos:

- **Infraestrutura Externa e Redes Corporativas:** O sistema é projetado para operar com autonomia e persistência local (SQLite). Portanto, não serão realizados testes de latência de rede, estresse de servidores externos institucionais (como servidores do Iveco Group ou redes do SENAI-MG) ou simulações de ataques de negação de serviço (DDoS).
- **Desgaste e Aferição Física de Hardware:** Embora o sistema se comunique com o módulo de sensores e câmeras, a equipe não testará a vida útil física, o desgaste mecânico ou o aquecimento térmico das câmeras conectadas, focando estritamente na camada de recepção de software.
- **Sistemas Operacionais não Homologados:** Os testes ficarão restritos ao ambiente Windows 10 ou superior, necessário para a execução do .NET Desktop Runtime e do serviço de background, não abrangendo tentativas de execução em Linux ou macOS.
- **Performance de Câmeras em Altíssima Resolução não Suportada:** Os testes de visão computacional serão delimitados às resoluções e taxas de quadros (FPS) documentadas nos requisitos, não cobrindo o comportamento do *VisionService* perante a injeção de hardwares não padronizados (como câmeras 8K de cinema).

### Funcionalidades prioritárias

Alinhado ao rigor exigido no Desafio para a Descarbonização, as funcionalidades que receberão a maior carga horária de testes (com maior volume de cenários de Particionamento de Equivalência) são:

- **Algoritmo de Detecção de Anomalias:** A precisão do *ClassificationService* e do *Analyser* ao interpretar os dados do combustível, pois uma falha aqui compromete o objetivo central do sistema.
- **Transição Estável entre Módulos:** A estabilidade do *MainViewModel* e da *MainWindow* ao realizar a troca entre a recepção de dados sintéticos via *SensorSimulator* e o feed de vídeo em tempo real do Módulo VISCOM (*ViscomViewModel*). O sistema não pode travar a thread visual (UI Thread) durante essa transição.
- **Persistência Crítica de Dados Operacionais:** Garantir que o banco SQLite (*ResultadosDbContext*) sempre registre o evento final de cada análise para fins de comprovação e auditoria, mesmo que o sistema seja encerrado abruptamente.

---

## 4. Base de Teste

A Base de Teste compreende toda a estrutura documental, arquitetural e de regras de negócio que servirá como fonte para a modelagem dos Casos de Teste. Para garantir que a plataforma Zero-E-Mission cumpra seu papel crítico no monitoramento e na descarbonização operacional, os testes serão fundamentados nos seguintes tópicos:

### Regras de Negócio e Domínio

A principal base para os testes de cálculo e diagnóstico será a documentação das regras de negócio atreladas à qualidade do combustível e aos limites de emissões aceitáveis.

- **Critérios de Aceitação de Emissões:** Tabelas contendo os valores exatos e as tolerâncias que definem o que é uma operação "Normal", "Em Alerta" ou "Crítica".
- **Lógica de Severidade:** As especificações de como o *ClassificationService* e o *Analyser* devem cruzar as métricas brutas com os perfis estabelecidos para gerar um diagnóstico preciso.
- **Perfis de Calibração:** As especificações operacionais que ditam como um *CalibrationProfile* é criado e como uma *CalibrationSample* deve ser validada antes de ser considerada uma referência confiável pelo *CalibrationService*.

### Especificações dos Sensores (Módulo Tradicional)

Para testar a injeção e o processamento de dados físicos, a equipe utilizará a documentação de integração de sensores.

- **Contratos de Interface:** As definições de arquitetura da *SensorInterface*, determinando exatamente quais tipos de dados (ex: temperatura, densidade, opacidade) o sistema está preparado para receber.
- **Comportamento de Simulação:** As regras de geração de dados sintéticos pelo *SensorSimulator*, definindo as faixas de valores pré-definidos que devem ser injetados no sistema durante o Modo Simulação para testar a responsividade do *ZeroEMissionViewModel* e da *JanelaGaugeViewModel*.

### Especificações de Visão Computacional (Módulo VISCOM)

Os testes do módulo de câmera serão baseados nos limites de processamento de imagem definidos para o projeto.

- **Parâmetros de Captura:** Requisitos de resolução mínima, taxa de quadros (FPS) esperada e formato de cor exigidos pelo *VisionService* para que a análise seja válida.
- **Cálculo Matricial:** O comportamento esperado dos algoritmos de conversão e análise de pixels executados na classe *MatHelpers*.

### Modelos de Arquitetura (MVVM em C# / WPF)

O design pattern escolhido para o sistema ditará como os testes de integração e de interface (UI) serão desenhados.

- **Separação de Preocupações:** O diagrama de arquitetura MVVM, que define que as regras de interface não podem vazar para a lógica de negócio. Os testes validarão o trânsito de dados entre o *BaseViewModel*, o *MainViewModel* e as *Views* em XAML (*MainWindow.xaml*, *JanelaGaugeViewModel.xaml*).
- **Mapeamento de Comandos:** A especificação de botões e ações do usuário, mapeados estritamente pelo *RelayCommand*, garantindo que eventos na tela disparem exclusivamente as funções pretendidas.

### Especificações de Persistência e Banco de Dados (SQLite)

A base para validar se o histórico de análises está seguro será o modelo de dados (Schema) construído via Entity Framework Core.

- **Modelagem de Entidades:** O dicionário de dados contendo a estrutura exata exigida pelas classes *AnalysisRecord*, *AnalysisResult* e *ResultadosModels*.
- **Restrições e Relacionamentos:** As regras de chaves primárias/estrangeiras e obrigatoriedade de campos definidas no *AppDbContext* e no *ResultadosDbContext*.
- **Regras de Auditoria:** As regras de histórico do sistema (*ILogService* e *LogEntry*), que exigem que qualquer erro, falha técnica ou mudança na calibração seja obrigatoriamente salva e registrada de forma permanente pela ferramenta principal de gravação (*Logger*)

---

## 5. Abordagem de testes

Para garantir que o sistema Zero-E-Mission atenda aos rigorosos padrões exigidos para o monitoramento de emissões, a equipe adotará uma estratégia de testes em múltiplas camadas. A abordagem combinará validações técnicas de código com simulações de uso no mundo real, garantindo a estabilidade tanto do Módulo Tradicional (sensores) quanto do Módulo VISCOM (câmeras).

### Testes funcionais

O foco aqui é garantir que o sistema faz exatamente o que deveria fazer segundo as regras de negócio.

- **Validação de Cálculos:** Verificação isolada das classes *Analyser* e *ClassificationService* para atestar que os algoritmos matemáticos geram o diagnóstico correto (ex: se o nível de emissão x e a densidade y resultam no alerta correto).
- **Ações de Interface:** Testar se os cliques nos botões da interface disparam as funções corretas através do *RelayCommand* (ex: clicar em "Salvar Perfil" deve efetivamente criar um *CalibrationProfile* novo).

### Testes não funcionais

O foco será avaliar o comportamento do sistema sob estresse, sua performance e resiliência, fatores críticos para um software que roda continuamente.

- **Performance e Memória:** Monitoramento intensivo do consumo de memória RAM e CPU durante o uso do Módulo VISCOM. Como o *VisionService* e o *MatHelpers* processam imagens em tempo real, os testes garantirão que o sistema descarte corretamente os frames antigos (evitando Memory Leaks).
- **Estabilidade em Segundo Plano:** Avaliação do *WindowsService* rodando ininterruptamente por longos períodos (ex: 24 horas), garantindo que a aplicação não trave e continue registrando logs de forma autônoma.

### Testes de sistema

Validação da aplicação como um todo, rodando no ambiente idêntico ao de produção.

- **Integração Total:** Testar o ecossistema Zero-E-Mission montado (UI + Banco de Dados + Serviço Windows). O objetivo é garantir que todas as ferramentas, desde o *MainViewModel* até o sistema de logs autônomo, funcionem de maneira orquestrada sem conflitos de concorrência.

### Testes de integração

Aqui, a equipe avaliará se as diferentes "peças" do software conversam bem entre si.

- **Fluxo de Dados:** Testar a jornada completa da informação. Por exemplo, testar se o dado gerado pelo Modo Simulação é recebido pelo *ZeroEMissionViewModel*, refletido visualmente pela *JanelaGaugeViewModel* e, por fim, gravado com sucesso no *SQLite* através do *ResultadosService* e *ResultadosDbContext*.

### Testes de aceitação

Esta é a fase final de validação, focada em quem vai usar o sistema no dia a dia ou avaliar seus resultados operacionais.

- **Validação de Stakeholders:** Demonstrações e testes práticos para confirmar se a solução atende aos objetivos de redução de impacto ambiental e monitoramento traçados para o Desafio para a Descarbonização. O critério de sucesso é o cliente (ou a banca avaliadora) confirmar que as métricas apresentadas na tela são úteis, legíveis e confiáveis.

### Testes exploratórios

Uma abordagem menos roteirizada e mais criativa, focada em tentar "quebrar" o sistema com comportamentos inesperados do usuário.

- **Estresse de Interface:** Navegar rapidamente entre o *ViscomViewModel* e o Módulo Tradicional, desconectar e conectar a câmera abruptamente durante uma análise, e redimensionar a *MainWindow* repetidas vezes para checar se os elementos visuais distorcem ou se a thread principal (UI Thread) congela.

### Testes baseados em cenários

Execução de roteiros que imitam situações reais do dia a dia da operação industrial.

- **Jornada do Operador:** Criar um cenário completo, como: "Um operador inicia o turno, cria um novo perfil de calibração utilizando a classe *CalibrationSample*, inicia a gravação de dados, presencia um pico de emissão (simulado pelo sistema), verifica se a Janela Gauge ficou vermelha através do *SeverityToColorConverter* e encerra extraindo um relatório das anomalias".

### Reteste e Regressão

A disciplina de qualidade que será aplicada sempre que a equipe de desenvolvimento fizer uma alteração no código ou corrigir um bug.

- **Reteste:** Focado exclusivamente no problema apontado. Se um teste anterior reprovou porque o *AppDbContext* não estava salvando a data corretamente, o reteste vai verificar unicamente se a data agora está sendo salva.
- **Regressão:** Focado no impacto colateral. Após consertar o salvamento da data no banco, a equipe rodará a regressão nas telas de consulta e histórico para garantir que a alteração não quebrou algo que antes funcionava perfeitamente.

---

## 6. Técnicas de projeto de testes

Para otimizar o tempo da equipe e garantir a máxima cobertura técnica, o projeto Zero-E-Mission empregará um conjunto específico de técnicas de modelagem de testes. Estas técnicas foram selecionadas por sua eficácia em validar tanto sistemas orientados a dados (como o processamento estatístico de emissões) quanto sistemas reativos de interface (WPF).

### Particionamento de Equivalência

Esta técnica será utilizada para reduzir o número infinito de entradas possíveis de sensores e câmeras a grupos (partições) que o sistema deve tratar da mesma maneira. Em vez de testar todos os números possíveis de níveis de emissão, a equipe testará valores representativos de cada classe.

- **Aplicação no Módulo Tradicional:** Se a regra de negócio determina que as métricas de qualidade do combustível processadas pelo *Analyser* são categorizadas em três níveis, criaremos três partições válidas e uma inválida.
   - ***Partição 1 (Aceitável):*** Leituras de emissão entre 0 e 50 (testaremos o valor 25).
   - ***Partição 2 (Alerta):*** Leituras entre 51 e 80 (testaremos o valor 65).
   - ***Partição 3 (Crítico):*** Leituras acima de 81 (testaremos o valor 90).
   - ***Partição Inválida:*** Valores negativos injetados pelo *SensorSimulator* (testaremos o valor -10, esperando que o sistema rejeite a entrada com uma exceção mapeada pelo *Logger*).
- **Justificativa:** Reduz o tempo de execução dos testes garantindo que, se o sistema processar o valor 65 corretamente, ele também processará o valor 66 da mesma forma no *ClassificationService*.

### Análise de valor limite

Considerada a técnica mais crítica para este projeto, ela foca exatamente nas fronteiras das partições de equivalência, pois é onde a maioria dos defeitos lógicos (erros de > (maior que) versus ≥ (maior ou igual a)) ocorre no código.

- **Aplicação nos Alertas Visuais:** Para garantir que a *JanelaGaugeViewModel* e o *SeverityToColorConverter* atualizem a interface gráfica no momento exato em que o limite ambiental é ultrapassado, testaremos rigorosamente as bordas das regras de negócio.
   - Se o limite crítico de acionamento do alerta vermelho é 81, a equipe testará exatamente os valores 80 (deve permanecer amarelo/alerta), 81 (deve mudar instantaneamente para vermelho/crítico) e 82 (deve manter vermelho).
- **Justificativa:** Assegura precisão milimétrica nas lógicas de tolerância de emissões, garantindo que o painel de monitoramento nunca exiba um Falso Positivo ou um Falso Negativo ao operador.

### Tabela de decisão

Sistemas de monitoramento lidam com múltiplas condições ocorrendo ao mesmo tempo. A Tabela de Decisão será usada para mapear como as lógicas de backend (*MainViewModel*) reagem à combinação de diferentes estados.

- **Aplicação no Acionamento do Sistema:** Mapearemos o comportamento esperado do software com base em três variáveis booleanas (Verdadeiro/Falso): Modo Simulação Ativo, Acionamento Motor Ativo e Detecção de Falha Crítica.
   - ***Cenário A: Modo Simulação = ON | Detecção Crítica = ON*** <br> **Resultado esperado:** A Janela Gauge alerta o usuário visualmente, mas o comando real de desligamento do motor físico não é enviado (pois é simulação).
   - ***Cenário B: Modo Simulação = OFF | Detecção Crítica = ON*** <br> **Resultado esperado:** A Janela Gauge alerta o usuário e o sistema dispara o comando de corte/desligamento do motor.
- **Justificativa:** Garante que a integração entre hardware físico, injeção de dados simulados e as respostas do *ZeroEMissionViewModel* estejam cobrindo todas as combinações lógicas possíveis, evitando acionamentos acidentais em ambiente real.

### Testes baseados em cenários

Esta técnica modelará testes de ponta a ponta que imitam um fluxo operacional contínuo no ambiente de produção.

- **Aplicação na Jornada do Operador:** A equipe executará scripts que envolvem múltiplos componentes trabalhando em uníssono. O cenário incluirá:
   1. O operador abre o sistema e muda imediatamente para a tela da câmera (Módulo VISCOM / *ViscomViewModel*).
   2. Ele liga a câmera para iniciar a captura de vídeo pelo sistema (*VisionService*).
   3. Com a imagem capturada, o operador calibra a amostra e o sistema aprova a calibração inicial (*CalibrationProfile* e *CalibrationService*).
   4. O processamento visual detecta automaticamente um problema na qualidade do combustível.
   5. O sistema salva esse alerta de erro direto no banco de dados (*AppDbContext* e *ResultadosService*).
- **Justificativa:** Comprova que as peças isoladas do software funcionam perfeitamente quando integradas na vida real, validando o ecossistema como uma solução viável para descarbonização.

### Testes exploratórios

Uma técnica baseada na intuição e na experiência do testador, executada sem roteiros estritos, com o objetivo de encontrar falhas não mapeadas pela especificação original.

- **Aplicação na Resiliência do Hardware e UI:** O testador tentará "quebrar" a aplicação executando ações abruptas.
   - Redimensionar a tela (*MainWindow*) múltiplas vezes por segundo durante o pico de processamento matricial do *MatHelpers* para validar se o cálculo do redimensionamento do vídeo causa engasgos ou travamento (deadlock).
   - Desconectar o cabo da câmera repentinamente (no caso da câmera sendo por USB) no meio da análise do Módulo VISCOM. O objetivo é confirmar que o programa não vai travar ou fechar sozinho, e sim que as ferramentas de segurança (*Logger* e *ILogService*) vão registrar o erro de forma controlada.
- **Justificativa:** Descobre falhas de usabilidade e vulnerabilidades de integração de hardware que testes estritamente baseados em regras matemáticas normalmente deixam passar.

---

## 7. Casos de Teste

Os testes abaixo foram criados para verificar o comportamento do sistema tanto quando tudo dá certo quanto quando ocorrem erros. Eles garantem que a comunicação entre as telas e a inteligência do programa seja segura e fique sempre registrada.

| ID | Funcionalidade | Cenário | Entrada | Resultado Esperado | Resultado Obtido | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-001** | **Módulo Tradicional (Simulador)** | Validação de leitura de emissão em nível **Normal**. | O *SensorSimulator* injeta o valor 15 (abaixo do limite de alerta). | O sistema processa o dado. A janela atualiza o ponteiro e mantém a cor Verde. Nenhum alerta crítico é disparado. | — | — |
| **CT-002** | **Módulo Tradicional (Alerta)** | Validação de leitura em nível **Crítico**. | O *SensorSimulator* injeta o valor 85 (acima do limite de segurança). | O sistema identifica a falha. A Janela Gauge muda para Vermelho imediatamente e o evento crítico é salvo no banco de dados. | — | — |
| **CT-003** | **VISCOM (Início e Calibração)** | Configuração inicial da análise visual antes de iniciar a medição. | Usuário acessa a aba VISCOM, inicia a câmera e calibra amostra. | O feed de vídeo inicia sem travar a tela. O sistema avalia o ambiente físico, aprova a calibração e inicia a análise. | — | — |
| **CT-004** | **VISCOM (Detecção de Falha)** | Câmera detecta anomalia na qualidade do combustível após calibração. | A câmera capta uma alteração na cor/fluidez do combustível em tempo real. | O processamento visual detecta o problema. O sistema gera um alerta na tela e salva o erro direto no banco de dados automaticamente. | — | — |
| **CT-005** | **VISCOM (Resiliência)** | Desconectar a câmera à força durante o funcionamento. | O cabo USB da câmera é removido fisicamente enquanto o vídeo está sendo analisado. | O sistema não fecha sozinho (sem *crash*). O erro é salvo no histórico e a tela volta para o Módulo Tradicional com um aviso amigável. | — | — |
| **CT-006** | **Persistência de Dados** | Gravação de um histórico de resultados durante a análise em andamento. | Usuário clica no botão "Salvar CSV". | O sistema grava o resultado em um arquivo CSV e no banco SQLite com sucesso. A ação também fica registrada no histórico de logs. | — | — |
| **CT-007** | **Interface / MVVM (Navegação)** | Alternar rapidamente entre as abas Tradicional e VISCOM durante uma análise. | Cliques repetidos alternando entre as duas abas principais do sistema. | A navegação ocorre de forma fluida. O sistema não congela, a câmera não trava e os dados da análise continuam processando nos bastidores. | — | — |
| **CT-008** | **Menu de Desenvolvedor** | Testar bloqueio do acionamento do motor físico no Modo Simulação. | Ativar "Modo Simulação" e disparar um alerta crítico. | A interface mostra o alerta visual (Gauge vermelho), mas o comando elétrico para desligar o motor físico **não** é enviado. | — | — |
---

## 8. Análise de Riscos

A identificação de riscos tem como objetivo prever falhas técnicas antes que elas aconteçam no ambiente de produção. Como o Zero-E-Mission lida com processamento contínuo de imagens, comunicação com sensores e gravação em um banco de dados local, os riscos mapeados focam em gargalos de performance, estabilidade da interface e precisão das medições.

Abaixo estão os principais riscos do projeto e o que faremos para resolvê-los:

| Risco | Impacto | Probabilidade | Ação de Mitigação (Como vamos resolver) |
| :--- | :--- | :--- | :--- |
| **Banco de Dados Bloqueado** <br> O banco de dados só permite uma gravação por vez. Se a interface visual e o serviço em segundo plano tentarem salvar dados exatamente ao mesmo tempo, o sistema pode apresentar erro. | **Alto**<br>(Pode causar perda de histórico) | **Média** | **Ação:** Implementar simulações de múltiplas gravações simultâneas. Garantir que as ferramentas do banco usem rotinas de espera (retry) e salvamento assíncrono para evitar conflitos. |
| **Vazamento de Memória no VISCOM** <br> O processamento de vídeo em tempo real usa muita memória RAM. Se os quadros (frames) analisados não forem descartados corretamente, a memória do computador vai encher até o programa fechar sozinho. | **Alto** <br> (Fecha a aplicação) | **Média** | **Ação:** Realizar testes não funcionais de estresse, deixando o Módulo VISCOM rodando por horas sem interrupções. Monitorar se o sistema está liberando a memória corretamente após cada ciclo de análise. |
| **Diagnóstico Incorreto (Falso Positivo/Negativo)** <br> O sistema acusar combustível adulterado quando ele está normal, ou pior, aprovar um combustível poluente por erro matemático. | **Crítico** <br> (Fere o objetivo do projeto) | **Baixa** | **Ação:** Aplicar rigorosamente a técnica de **Análise de Valor Limite** nos testes. Garantir que as lógicas das bordas (ex: transição exata do valor 80 para 81) sejam precisas. |
| **Travamento da Tela** <br> Como o sistema faz cálculos pesados, existe o risco da tela principal congelar, fazendo com que o ponteiro pare de se mover ou que os botões não respondam ao clique. | **Médio** <br> (Prejudica a usabilidade) | **Alta** | **Ação:** Fazer testes rigorosos na tela do sistema quando ele estiver processando muitos dados. O objetivo é garantir que os cálculos pesados fiquem rodando nos bastidores, sem fazer o visual do programa travar. |
| **Desconexão de Hardware** <br> O cabo da câmera ou do sensor soltar durante a operação, fazendo o sistema parar de receber dados inesperadamente. | **Alto** <br> (Interrompe a análise) | **Alta** | **Ação:** Realizar testes de desconexão forçada. Garantir que, ao invés de exibir um erro fatal, o sistema avise o operador de forma amigável e grave o ocorrido em segurança usando as ferramentas de histórico. |

---

## 9. Critérios de Entrada e Saída

Os critérios de entrada e saída garantem que o esforço de testes seja produtivo. Eles evitam que a equipe perca tempo testando um sistema incompleto (entrada) e impedem que um software com falhas graves seja entregue como finalizado (saída).

### Critérios de Entrada

Antes de executar o primeiro caso de teste, as seguintes condições devem ser obrigatoriamente cumpridas:

- **Ambiente Configurado:** O computador de testes deve estar rodando o Windows (necessário para o serviço de segundo plano) e o banco de dados deve ser criado no momento da primeira análise, indicando que está pronto para receber dados.
- **Equipamentos Conectados:** A câmera (para o módulo VISCOM) e o simulador de sensores (para o módulo tradicional) devem estar conectados e sendo reconhecidos pelo sistema operacional.
- **Código Estabilizado (Build):** A versão do software que será testada deve abrir normalmente, sem apresentar erros de compilação logo na tela inicial.
- **Dados de Referência Preparados:** A equipe já deve ter em mãos os valores matemáticos exatos que definem o que é um combustível "Normal", "Alerta" ou "Crítico", para poder comparar com o que vai aparecer na tela.

### Critérios de Saída

A fase de testes só será considerada concluída e o sistema liberado para entrega quando as seguintes metas forem alcançadas:

- **100% de Execução dos Testes Prioritários:** Todos os cenários focados na visão computacional (VISCOM), nos cálculos de severidade e no salvamento do banco de dados devem ser executados até o fim.
- **Zero Defeitos Críticos ou Altos:** Nenhum erro que impeça o uso do sistema pode estar aberto. Isso significa que o programa não pode fechar sozinho, a tela não pode congelar durante a análise de vídeo, e não pode haver erros na classificação da qualidade do combustível.
- **Consumo de Memória Estável:** Após rodar o módulo de câmera (VISCOM) por um tempo prolongado, o consumo de memória do computador não pode estourar (o que provaria que o sistema não está acumulando "lixo" na memória).
- **Rastreabilidade Comprovada:** Todos os alertas e simulações de erro gerados pela equipe durante os testes devem constar perfeitamente salvos no arquivo de histórico do sistema.
- **Riscos Residuais Aceitos:** Qualquer defeito pequeno ou de baixa prioridade que sobrar (como um texto um pouco desalinhado na tela) deve estar documentado e aceito formalmente pela equipe de desenvolvimento como um risco que não afeta o objetivo principal da descarbonização.

---

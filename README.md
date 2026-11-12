<div align="center">

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1sa3_Sdc9xZIWs27RA6NbmQGB6v6VDQxG?usp=sharing)

</div>

<div align="center">
  <h4>ECM514 – Ciência de Dados</h4>
  <h4>Projeto Desafio do 2º Semestre</h4>
  <h1>Detecção e Diagnóstico de Anomalias em Sinais de Vibração</h1>
</div>

<div align="center">

[![Apresentação do projeto no YouTube](https://img.youtube.com/vi/dMsA8Dg0oVE/maxresdefault.jpg)](https://www.youtube.com/watch?v=dMsA8Dg0oVE)

▶️ [Assistir à apresentação no YouTube](https://www.youtube.com/watch?v=dMsA8Dg0oVE)

</div>

## Integrantes

| Nome | RA |
|---|---|
| André Renato Almeida Abreu | 22.01255-9 |
| Arthur Jin Woo Lee | 22.01519-0 |
| Fernando Minharro Alves Gimenez | 22.00833-0 |
| Leonardo Henrique Dias Moura | 21.01837-5 |
| Leonardo Tagliati da Silva | 22.00170-0 |

---

## Sumário

**Relatório Técnico**
- [1. Descrição do Dataset](#1-descrição-do-dataset)
- [2. Fundamentação](#2-fundamentação)
- [3. Metodologia](#3-metodologia)
- [4. Resultados](#4-resultados)
- [5. Discussão Crítica e Conclusões](#5-discussão-crítica-e-conclusões)
- [6. Referências Bibliográficas](#6-referências-bibliográficas)
- [7. Declaração de Uso de Inteligência Artificial](#7-declaração-de-uso-de-inteligência-artificial)

---

## 1. Descrição do Dataset

### Origem e setup físico do experimento

Os dados utilizados são do dataset público **UOEMD-VAFCVS** (*University of Ottawa Electric Motor Dataset – Vibration and Acoustic Faults under Constant and Variable Speed Conditions*), documentado por Sehri e Dumond (2024). Os dados foram coletados em uma versão modificada da bancada de testes SpectraQuest Machinery Fault & Rotor Dynamics Simulator. Para a aquisição, o sistema foi instrumentado com:

- **Três acelerômetros:** o Acelerômetro 1 (PCB 603C01), fixado por ímã no drive end do motor, e os Acelerômetros 2 e 3 (PCB 623C01), fixados por prisioneiro nos alojamentos dos rolamentos do eixo, à esquerda e à direita.
- **Um microfone (PCB 130F20):** para captação acústica, posicionado a cerca de 2 cm do alojamento do rolamento esquerdo, em suporte isolado.
- **Um sensor de temperatura:** integrado ao Acelerômetro 1, na superfície do motor.
- **Um inversor de frequência (VFD):** para controle de rotação.

Os dados foram registrados via LabVIEW.

### Motor e introdução das falhas

- Motor de indução trifásico Marathon Electric D396, 3,0 HP, 60 Hz, 3600 RPM nominais, com rolamentos do tipo 6205.
- Foram usados **8 motores do mesmo modelo**: 1 saudável e 7 com uma falha cada. As falhas foram introduzidas artificialmente pelo fabricante da bancada (SpectraQuest). O artigo do dataset não informa a severidade de cada falha.
- Como cada classe corresponde a um motor físico diferente, parte do que um classificador aprende pode ser a "identidade" do motor e não apenas a falha. Essa limitação é discutida na [Discussão Crítica](#5-discussão-crítica-e-conclusões).

### Taxa de amostragem e formato dos dados

- **Taxa de amostragem:** 42.000 Hz (amostras por segundo).
- **Duração:** cada arquivo de registro contém exatos 10 segundos de gravação.
- **Amostras por arquivo:** 420.000 amostras por sensor, distribuídas em 5 colunas, nesta ordem: Acelerômetro 1, Microfone, Acelerômetro 2, Acelerômetro 3 e Temperatura.
- A pasta `1_Time` contém apenas o eixo de tempo (0 a 10 s) e não é um sinal; por isso ela é excluída do mapeamento dos arquivos.

### Tipos de falha rotulados

O dataset contempla 8 classes de saúde da máquina:

| Código | Classe |
|---|---|
| H-H | Saudável (*Healthy*) |
| R-U | Desbalanceamento do Rotor (*Rotor Unbalance*) |
| R-M | Desalinhamento do Rotor (*Rotor Misalignment*) |
| S-W | Falha no Enrolamento do Estator (*Stator Winding Fault*) |
| V-U | Desequilíbrio de Tensão e Falta de Fase (*Voltage Unbalance*) |
| B-R | Rotor Empenado (*Bowed Rotor*) |
| K-A | Barras do Rotor Quebradas (*Broken Rotor Bars*) |
| F-B | Rolamentos Defeituosos (*Faulty Bearings*) |

### Condições de operação

Os testes foram realizados sob múltiplas combinações, indicadas pelos números no final do nome dos arquivos (ex: `R_U_1_0`):

- **Primeiro número (velocidade):** 1 = 15 Hz, 2 = 30 Hz, 3 = 45 Hz e 4 = 60 Hz (constantes); 5 = rampa de 15 a 45 Hz, 6 = rampa de 30 a 60 Hz, 7 = rampa de 45 a 15 Hz e 8 = rampa de 60 a 30 Hz.
- **Segundo número (carga):** 0 = sem carga e 1 = com carga.

### Quantidade de amostras por classe

Como há 8 perfis de velocidade (4 constantes + 4 variáveis) e 2 perfis de carga (sem carga + com carga), cada uma das 8 classes possui exatamente **16 arquivos** (128 no total).

- **Amostras por classe:** 16 arquivos × 420.000 = 6.720.000 amostras temporais brutas disponíveis para cada tipo de falha (e para a classe saudável).
- Após o janelamento (janelas de 1 s com 50% de sobreposição), cada arquivo gera 19 janelas, ou seja, **304 janelas por classe**.

| Classe | Arquivos | Janelas de treino (fold 0) | Janelas de teste (fold 0) |
|---|---|---|---|
| B-R | 16 | 228 | 76 |
| F-B | 16 | 228 | 76 |
| H-H | 16 | 228 | 76 |
| K-A | 16 | 228 | 76 |
| R-M | 16 | 228 | 76 |
| R-U | 16 | 228 | 76 |
| S-W | 16 | 228 | 76 |
| V-U | 16 | 228 | 76 |
| **Total** | **128** | **1.824** | **608** |

### Análise exploratória (60 Hz constante, sem carga)

| Classe | RMS (m/s²) | RMS / RMS do H-H | Pico da FFT (Hz) | Amplitude 1× fr (m/s²) | Amplitude ~3× fr (m/s²) | Energia > 2 kHz (%) | RMS > 2 kHz (m/s²) |
|---|---|---|---|---|---|---|---|
| H-H | 5,06 | 1,00 | 119,6 | 0,91 | 1,05 | 10,41 | 1,63 |
| R-U | 18,96 | 3,75 | 177,7 | 4,75 | 15,22 | 0,15 | 0,73 |
| R-M | 36,13 | 7,14 | 177,4 | 4,06 | 31,70 | 0,09 | 1,08 |
| S-W | 18,53 | 3,66 | 177,4 | 2,20 | 21,71 | 0,17 | 0,76 |
| V-U | 51,05 | 10,09 | 177,7 | 1,77 | 58,52 | 0,09 | 1,56 |
| B-R | 22,29 | 4,40 | 177,6 | 4,38 | 18,28 | 0,26 | 1,15 |
| K-A | 61,37 | 12,13 | 177,4 | 1,35 | 64,46 | 0,70 | 5,13 |
| F-B | 38,07 | 7,52 | 177,5 | 2,00 | 43,80 | 1,33 | 4,38 |

- **Motor saudável (H-H):** RMS de 5,1 m/s². O maior pico da FFT está em ~120 Hz (2× fr), seguido de componentes entre 140 e 160 Hz e do 1× fr (0,91 m/s²). O espectrograma mostra linhas horizontais estáveis, como esperado em velocidade constante.
- **Todos os motores com falha** têm vibração global muito maior que a do saudável (RMS de 18,5 m/s² no S-W a 61,4 m/s² no K-A, ou seja, de 3,7 a 12,1 vezes o RMS do H-H) e o maior pico da FFT em ~177,5 Hz (3× fr), componente que no motor saudável é pequena (1,05 m/s²).
- **Desbalanceamento (R-U), desalinhamento (R-M) e rotor empenado (B-R):** amplitude em 1× fr entre 4,1 e 4,8 m/s², cerca de 5 vezes a do motor saudável, coerente com falhas mecânicas ligadas à rotação do eixo.
- **Desequilíbrio de tensão (V-U) e barras quebradas (K-A):** os maiores níveis de vibração (RMS de 51,1 e 61,4 m/s²), concentrados no pico de ~177,5 Hz.
- **Rolamento (F-B):** entre as classes com falha, é a de maior fração de energia acima de 2 kHz (1,33%, contra 0,09% a 0,70% nas demais), conteúdo de alta frequência típico de impactos em rolamentos. Essa fração relativa, porém, deve ser lida com cuidado: no H-H ela é de 10,4%, porque a energia total do motor saudável é baixa, e em valor absoluto o conteúdo acima de 2 kHz do F-B (RMS de 4,4 m/s²) fica próximo ao do K-A (5,1 m/s²). O indício mais específico de defeito de rolamento é o espectro de envelope.

Como cada classe corresponde a um motor físico diferente (SEHRI; DUMOND, 2024), parte dessas diferenças, em especial o nível global de vibração e o pico em 3× fr comum aos 7 motores com falha, pode refletir diferenças entre os motores, e não apenas a falha. Isso torna a separação entre saudável e falha relativamente fácil, inclusive para um limiar simples de RMS, e é uma limitação importante para generalizar os resultados.

### Análise complementar: efeito da velocidade e espectro de envelope

1. **Efeito da velocidade no desbalanceamento:** a força de desbalanceamento é proporcional a $m \cdot e \cdot \omega^2$. A 60 Hz, a amplitude em 1× fr do R-U é 4,75 m/s², contra 0,91 m/s² do H-H. A 15 Hz, os valores caem para 0,153 e 0,072 m/s²: a diferença entre o motor desbalanceado e o saudável passa de 3,8 m/s² para apenas 0,08 m/s². A força de desbalanceamento cresce com ω², então a 15 Hz ela é cerca de 1/16 da força a 60 Hz. Na prática, a amplitude em 1× fr do R-U caiu 31 vezes entre 60 e 15 Hz. Em baixa velocidade, o sinal do motor desbalanceado fica muito próximo do sinal saudável, o que antecipa a dificuldade de detecção do R-U a 15 Hz vista nos resultados.
2. **Espectro de envelope do rolamento:** o sinal é filtrado em 2–10 kHz e demodulado pela Transformada de Hilbert (RANDALL; ANTONI, 2011). As frequências teóricas de defeito nas pistas externa (BPFO) e interna (BPFI) são calculadas para a geometria padrão do rolamento 6205 (9 esferas, diâmetro da esfera de 7,94 mm e diâmetro primitivo de 39,04 mm), já que o artigo do dataset informa apenas o modelo do rolamento. O F-B tem picos de envelope que não aparecem no H-H: o maior pico não síncrono fica em 4,83 a 4,89× fr a 45 e 60 Hz, com e sem carga (a 30 Hz, fica em 1,85 a 1,86× fr). No H-H, os picos de envelope são harmônicos da rotação. Duas conclusões:
   - O F-B tem, de fato, uma modulação não síncrona e repetitiva, típica de defeitos de rolamento (RANDALL; ANTONI, 2011), o que apoia a explicação física para o bom desempenho nessa classe.
   - Esse componente não coincide com as frequências teóricas BPFO (3,585× fr) e BPFI (5,415× fr) calculadas para a geometria padrão do 6205. A geometria real do rolamento, ou a localização do defeito, deve ser diferente da assumida; portanto, as features `envelope_bpfo_rel` e `envelope_bpfi_rel` medem frequências que não são as do defeito.

---

## 2. Fundamentação

### Modelos não supervisionados para detecção de anomalias (*novelty detection*)

Em cenários industriais reais, dados operacionais sadios são abundantes, enquanto ocorrências de falhas específicas são escassas (RANDALL, 2011). Portanto, a detecção de anomalias é frequentemente modelada como um problema de classe única (*one-class*):

- **Isolation Forest (IF):** algoritmo de aprendizagem não supervisionada voltado especificamente para a detecção eficiente de anomalias (*outlier detection*), baseado no princípio de particionamento recursivo (LIU et al., 2008). O método explora a premissa de que anomalias são instâncias minoritárias e geometricamente disjuntas, exigindo menos partições (cortes aleatórios em árvores de decisão) para serem isoladas em comparação aos dados operacionais normais. Na monitoração de condição de motores, o Isolation Forest destaca-se por sua alta escalabilidade e eficiência em matrizes de features de alta dimensionalidade, permitindo a identificação imediata de desvios estruturais sem a necessidade de dados de falhas rotulados na fase de treinamento.
- **One-Class SVM (OC-SVM):** baseado na teoria estatística de aprendizado, o OC-SVM mapeia os vetores de features do estado saudável para um espaço de alta dimensionalidade via funções kernel (como RBF), buscando construir um hiperplano de margem máxima que separa a normalidade da origem (SCHÖLKOPF et al., 2001). É altamente robusto em espaços vetoriais complexos.
- **Local Outlier Factor (LOF):** diferente de abordagens globais, o LOF é um algoritmo baseado na densidade local de vizinhança (BREUNIG et al., 2000). Ele mede o grau em que um ponto é isolado em relação aos seus vizinhos mais próximos, tornando-se altamente eficaz para detectar anomalias sutis em distribuições de dados não uniformes de vibração industrial.

### Abordagens de diagnóstico: ensemble, geometria e reconstrução

Uma vez isolada a anomalia, o diagnóstico visa categorizar a falha sem depender exclusivamente de classificadores supervisionados tradicionais:

- **Random Forest (RF):** método de aprendizagem supervisionada baseado em ensemble de árvores de decisão, construídas de forma independente por meio de amostragem com reposição (*bagging*) e seleção aleatória de subconjuntos de features (BREIMAN, 2001). O modelo combina o veredito de múltiplas árvores para realizar a classificação multiclasse, mitigando o risco de overfitting inerente a árvores isoladas. No diagnóstico de máquinas rotativas, o Random Forest é amplamente utilizado por sua capacidade de mapear fronteiras de decisão não lineares complexas no espaço de features estatísticas e espectrais, além de fornecer métricas de importância de variáveis que auxiliam na validação física e na discussão crítica das falhas.
- **Prototipagem Geométrica (Nearest Centroid):** fundamentada na separabilidade espacial de classes no espaço vetorial de features estatísticas e espectrais, o classificador mede a distância euclidiana entre a nova observação e os centróides médios de cada tipo de falha (TIBSHIRANI et al., 2002).
- **Autoencoders para Reconstrução (Multi-Model Autoencoders):** redes neurais autoassociativas que comprimem o sinal em um espaço latente de menor dimensão e o reconstroem (KRAMER, 1991). Quando múltiplos modelos especialistas são treinados — um para cada condição de falha —, o diagnóstico baseia-se na premissa de que a rede específica daquela falha apresentará o menor Erro Quadrático Médio (MSE) de reconstrução.

### Features de vibração

- **Envelope (Transformada de Hilbert):** RMS, curtose e fator de crista do envelope são essenciais para extrair a modulação provocada por falhas em componentes rotativos (rolamento e barras do rotor) (RANDALL; ANTONI, 2011).
- **Ordens da rotação:** o 1× está ligado a desbalanceamento e rotor empenado, e o 2× a desalinhamento (RANDALL, 2011).
- **Envelope nas frequências de defeito do rolamento** (BPFO e BPFI), relativo à mediana do espectro de envelope (SMITH; RANDALL, 2015).

### Organização dos experimentos

- **Pares detector → classificador:** cada detector é combinado com um classificador para demonstrar um sistema em cascata, no qual o detector filtra as janelas anômalas e o classificador diagnostica o tipo de falha: **Isolation Forest → Random Forest**, **One-Class SVM → Nearest Centroid** e **LOF → Autoencoders**. Os pares são uma escolha de demonstração, e não uma dependência entre os métodos. Por isso, os três classificadores também são avaliados no mesmo conjunto de teste completo, o que permite compará-los de forma justa, e a cascata é avaliada de ponta a ponta (uma falha não detectada conta como erro de diagnóstico).
- **Baseline:** como referência simples, é usado um detector por limiar de RMS, inspirado na norma ISO 20816-1 (ISO, 2016), que avalia a severidade de vibração pelo valor RMS global. A norma usa velocidade de vibração (mm/s, na faixa de 10 a 1000 Hz), enquanto o dataset fornece aceleração (m/s²) e não há zonas definidas para este motor. Por isso, o baseline usa a mesma ideia, um único limiar sobre o RMS, calibrado com os dados saudáveis do treino: o limiar é o percentil 95 do RMS das janelas H-H. Isso equivale a cerca de 5% de falsos positivos no treino, o mesmo critério usado nos detectores de aprendizado de máquina (`contamination = nu = 0.05`).

---

## 3. Metodologia

### Divisão treino/teste, janelamento e treino

- **Divisão por arquivo, antes do janelamento:** cada arquivo (uma combinação de classe, velocidade e carga) é a unidade da divisão. A divisão usa `StratifiedKFold` com 4 folds sobre a tabela de arquivos, estratificada por classe e carga: cada fold de teste tem 4 arquivos de cada classe (2 sem carga e 2 com carga, 25% dos dados), e nenhum arquivo aparece ao mesmo tempo em treino e teste. Como cada arquivo é o seu próprio grupo, uma divisão agrupada (`StratifiedGroupKFold`) não é necessária: dividir a tabela de arquivos já impede o leakage, e a estratificação por carga garante folds equilibrados. O fold 0 é usado nas análises detalhadas (curvas ROC e matrizes de confusão), e os 4 folds são usados na validação cruzada, que reporta média ± desvio das métricas.
- **Janelas de 1 s (42.000 amostras):** na menor rotação (15 Hz), a janela contém 15 rotações completas, e a resolução espectral de 1 Hz é suficiente para separar os harmônicos 1×, 2× e 3× da rotação.
- **Sobreposição de 50%:** gera 19 janelas por arquivo de 10 s. As janelas de um mesmo arquivo são correlacionadas; por isso, além das métricas por janela, também são reportadas métricas agregadas por arquivo.
- **Sensores:** são janelados os 3 acelerômetros e o microfone. O Acelerômetro 1 (drive end do motor) é o sensor dos modelos principais; os demais sensores entram no estudo de fusão de sensores (ablação de features).
- **Padronização:** o `StandardScaler` é ajustado apenas com os dados de treino.
- **Detecção (*novelty detection*):** os detectores são treinados somente com janelas saudáveis (H-H) do treino. Os parâmetros `contamination = 0.05` (Isolation Forest e LOF) e `nu = 0.05` (One-Class SVM) definem o limiar de decisão para que cerca de 5% das janelas saudáveis do treino fiquem acima dele. O `n_neighbors = 20` do LOF segue o valor sugerido por Breunig et al. (2000).
- **Diagnóstico:** os classificadores são treinados com todas as classes do treino e avaliados no mesmo conjunto de teste completo.
- **Autoencoders:** gargalo de 8 neurônios, early stopping por validação e seed fixa. Os 2 arquivos de validação de cada classe são sorteados com seed fixa, um sem carga e um com carga, para que a parada considere as duas cargas e condições de velocidade variadas.
- **Validação anti-leakage:** após a criação das janelas, verifica-se que nenhum arquivo de treino está presente no conjunto de teste.
- **Reprodutibilidade:** as seeds são fixas e as versões das bibliotecas estão impressas no início do notebook (e listadas no `requirements.txt`).

### Extração de features

Para cada janela são extraídas features nos domínios do tempo, da frequência e do envelope. A extração é feita para os 4 sensores (prefixos `acc1_`, `acc2_`, `acc3_` e `mic_`), mas os modelos principais usam apenas o Acelerômetro 1; os demais sensores entram no estudo de ablação.

1. **Domínio do tempo**
   - **RMS:** medida da energia global da vibração ($\text{RMS} = \sqrt{\frac{1}{N}\sum x_i^2}$).
   - **Pico a pico e valor de pico:** indicadores de amplitudes extremas no sinal.
   - **Curtose:** sensível a choques e impactos (crucial para falhas de rolamento).
   - **Assimetria:** avalia a falta de simetria na distribuição do sinal.
   - **Fator de crista, fator de forma e fator de impulso:** razões estruturais que destacam variações abruptas em relação à energia média do sinal.
2. **Domínio da frequência (FFT)**
   - **Frequência de pico:** identifica a frequência dominante na janela.
   - **Média e desvio padrão espectral:** centro de massa e dispersão do espectro, ponderados pela potência ($|X(f)|^2$).
   - **Energia em bandas de frequência:** soma da potência em bandas de 0–100 Hz, 100–500 Hz, 500–2000 Hz, 2000–10000 Hz e 10000–21000 Hz (até a frequência de Nyquist), em valor absoluto e relativo à energia total. Os modelos principais usam as 4 primeiras bandas em valor absoluto; a banda de 10–21 kHz e as energias relativas entram apenas nos conjuntos da ablação.
3. **Envelope do sinal (Transformada de Hilbert)**
   - RMS, curtose e fator de crista do envelope.
4. **Features físicas normalizadas pela rotação**
   - **Rotação estimada:** a rotação nominal da janela vem do perfil de velocidade do arquivo (interpolada nas rampas) e é refinada pelo pico do espectro entre 0,8 e 1,2 vezes esse valor.
   - **Ordens 1×, 2× e 3×:** amplitude do espectro nos harmônicos da rotação, dividida pelo RMS da janela.
   - **Envelope nas frequências de defeito do rolamento:** amplitude do espectro de envelope (banda de 2–10 kHz) em BPFO e BPFI, relativa à mediana do espectro de envelope.

**Conjuntos de features usados nos modelos:**
- **Detecção:** as 20 features originais do Acelerômetro 1 (tempo, frequência com as 4 bandas absolutas e envelope).
- **Diagnóstico:** as 20 features originais + as 5 features físicas (ordens 1×, 2× e 3×, BPFO e BPFI).

### Avaliações complementares

- **Comparação no mesmo conjunto de teste (fold 0):** taxa de detecção por classe de falha, por velocidade e por carga; agregação por arquivo (voto majoritário das 19 janelas), mais próxima de uma decisão real sobre um registro de 10 s; e uma regra de alarme temporal que só dispara após 3 janelas anômalas consecutivas (1,5 s de sinal).
- **Validação cruzada por arquivo (4 folds):** em cada fold, o scaler, os detectores e os classificadores são treinados de novo apenas com o treino do fold.
- **Ablação dos conjuntos de features:** remoção das features redundantes (`time_mean`, `time_std`, `freq_spectral_energy` e energias absolutas das bandas, substituídas pelas relativas), bandas relativas, features normalizadas pela rotação e fusão de sensores.
- **Sensibilidade dos hiperparâmetros:** `contamination`, `nu` e `n_neighbors`.
- **Generalização entre condições de operação:** treino sem carga → teste com carga (e o inverso); velocidade de 15 Hz (código 1) fora do treino; e velocidade de 60 Hz (código 4) fora do treino.

---

## 4. Resultados

### Tabela resumo de resultados (fold 0)

| Dataset | Técnica | Tipo de falha identificada | Taxa de detecção (TPR) | Falso positivo (FPR) | Acurácia por classe (se houver diagnóstico) |
|---|---|---|---|---|---|
| UOEMD-VAFCVS | Baseline RMS (ISO 20816) | Não se aplica (detecção de anomalia) | 68,4% | 6,6% | Não se aplica |
| UOEMD-VAFCVS | Isolation Forest | Não se aplica (detecção de anomalia) | 92,7% | 15,8% | Não se aplica |
| UOEMD-VAFCVS | One-Class SVM | Não se aplica (detecção de anomalia) | 83,1% | 18,4% | Não se aplica |
| UOEMD-VAFCVS | Local Outlier Factor (LOF) | Não se aplica (detecção de anomalia) | 88,5% | 17,1% | Não se aplica |
| UOEMD-VAFCVS | Random Forest | Todas as 8 classes (B-R, F-B, H-H, K-A, R-M, R-U, S-W, V-U) | Não se aplica | Não se aplica | B-R: 89% · F-B: 100% · H-H: 100% · K-A: 58% · R-M: 70% · R-U: 74% · S-W: 24% · V-U: 49% |
| UOEMD-VAFCVS | Nearest Centroid | Todas as 8 classes (B-R, F-B, H-H, K-A, R-M, R-U, S-W, V-U) | Não se aplica | Não se aplica | B-R: 51% · F-B: 12% · H-H: 84% · K-A: 11% · R-M: 0% · R-U: 39% · S-W: 20% · V-U: 7% |
| UOEMD-VAFCVS | Multi-Model Autoencoders | Todas as 8 classes (B-R, F-B, H-H, K-A, R-M, R-U, S-W, V-U) | Não se aplica | Não se aplica | B-R: 88% · F-B: 89% · H-H: 100% · K-A: 43% · R-M: 29% · R-U: 72% · S-W: 20% · V-U: 30% |

**Métricas complementares (fold 0 e validação cruzada de 4 folds, média ± desvio):**

| Técnica | AUC (fold 0) | Acurácia global / cascata (fold 0) | Validação cruzada (4 folds) |
|---|---|---|---|
| Baseline RMS (ISO 20816) | 0,852 | Não se aplica | TPR 81,1% ± 10,4% · FPR 10,9% ± 17,6% · AUC 0,910 ± 0,051 |
| Isolation Forest | 0,971 | Não se aplica | TPR 93,8% ± 4,0% · FPR 19,7% ± 15,7% · AUC 0,970 ± 0,005 |
| One-Class SVM | 0,921 | Não se aplica | TPR 88,9% ± 5,7% · FPR 15,1% ± 8,9% · AUC 0,939 ± 0,039 |
| Local Outlier Factor (LOF) | 0,942 | Não se aplica | TPR 90,6% ± 5,5% · FPR 11,2% ± 5,5% · AUC 0,955 ± 0,023 |
| Random Forest | Não se aplica | 70,4% / 70,4% (cascata com Isolation Forest) | Acurácia 74,7% ± 4,6% · Cascata 74,1% ± 5,1% |
| Nearest Centroid | Não se aplica | 28,0% / 27,3% (cascata com One-Class SVM) | Acurácia 32,4% ± 3,0% · Cascata 33,9% ± 4,9% |
| Multi-Model Autoencoders | Não se aplica | 59,0% / 56,2% (cascata com LOF) | Acurácia 58,3% ± 3,6% · Cascata 55,7% ± 6,1% |

> As matrizes de confusão completas (8 classes) de cada classificador, as curvas ROC e as tabelas detalhadas por fold estão nas saídas do [notebook](Detecção_e_Diagnóstico_de_Anomalias_em_Sinais_de_Vibração_Final_REVISADO_v2.ipynb), nas seções "Diagnóstico da Falha" de cada modelo.

### Resultados por fold (validação cruzada)

| Fold | RMS TPR | RMS FPR | RMS AUC | IF TPR | IF FPR | IF AUC | OC-SVM TPR | OC-SVM FPR | OC-SVM AUC | LOF TPR | LOF FPR | LOF AUC | RF Acc | RF Cascata | NC Acc | NC Cascata | AE Acc | AE Cascata |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 0,684 | 0,066 | 0,852 | 0,927 | 0,158 | 0,971 | 0,831 | 0,184 | 0,921 | 0,885 | 0,171 | 0,942 | 0,704 | 0,704 | 0,280 | 0,273 | 0,590 | 0,562 |
| 1 | 0,814 | 0,000 | 0,964 | 0,908 | 0,000 | 0,974 | 0,876 | 0,132 | 0,908 | 0,878 | 0,039 | 0,944 | 0,725 | 0,702 | 0,336 | 0,334 | 0,539 | 0,479 |
| 2 | 0,808 | 0,000 | 0,884 | 0,919 | 0,263 | 0,963 | 0,882 | 0,250 | 0,931 | 0,874 | 0,105 | 0,944 | 0,811 | 0,811 | 0,336 | 0,367 | 0,576 | 0,561 |
| 3 | 0,938 | 0,368 | 0,941 | 0,996 | 0,368 | 0,969 | 0,968 | 0,039 | 0,995 | 0,989 | 0,132 | 0,989 | 0,747 | 0,747 | 0,347 | 0,383 | 0,627 | 0,627 |

### Detecção de anomalias

**Baseline de RMS:** o limiar de RMS (8,71 m/s²) gera 6,6% de falsos positivos no fold 0 (5 das 76 janelas saudáveis, todas do arquivo H_H_8_1, rampa de 60 a 30 Hz com carga) e detecta apenas 68,4% das janelas com falha (AUC = 0,852). As falhas perdidas são as de vibração global baixa: a 15 Hz (código 1), nenhuma janela com falha ultrapassa o limiar, e a 30 Hz (código 2) só metade delas. Como o RMS cresce com a rotação e com a carga, um limiar único fica alto demais para as falhas em baixa velocidade e baixo demais para o motor saudável em alta rotação com carga.

**Isolation Forest:**
- **Fold 0:** detectou 92,7% das janelas com falha (TPR), com 15,8% de falsos positivos (12 de 76 janelas saudáveis) e AUC de 0,971, a maior AUC entre os detectores neste fold. Os falsos positivos vêm quase todos do arquivo H_H_8_1, que tem 53% das janelas acima do limiar.
- **Estabilidade:** na validação cruzada, o TPR médio é 93,8% ± 4,0%, e o FPR médio é 19,7% ± 15,7% (de 0% a 36,8% conforme o fold). Como o detector é treinado com apenas 12 arquivos saudáveis, o FPR depende de quais condições saudáveis ficaram no treino. A AUC é estável (0,970 ± 0,005): o ranqueamento das janelas é bom, e a variação vem principalmente do limiar definido por `contamination`.
- **Onde falha:** as falhas não detectadas se concentram na velocidade de 15 Hz (66,7% de detecção), no R-U (75,0%) e no K-A (73,7%). Os arquivos R_U_1_0 e K_A_1_1 não têm nenhuma janela detectada. Isso é coerente com a física: a força de desbalanceamento é proporcional a ω², e a 15 Hz o sinal do motor com falha fica muito próximo do saudável.

**One-Class SVM:**
- **Fold 0:** TPR de 83,1%, FPR de 18,4% e AUC de 0,921. É o detector com mais falsos alarmes neste fold: 14 das 76 janelas saudáveis, vindas principalmente do arquivo H_H_1_0 (15 Hz, sem carga, com 42% das janelas). Na validação cruzada, o FPR é 15,1% ± 8,9% e a AUC é 0,939 ± 0,039.
- **Falhas pouco detectadas:** detecta apenas 67,1% das janelas de S-W, 71,1% das de K-A e 50,0% das falhas a 15 Hz. A fronteira RBF ajustada nas janelas saudáveis fica larga demais nas regiões de baixa energia (baixa velocidade), onde falhas e motor saudável se sobrepõem, e ao mesmo tempo marca como anômalo o H_H_1_0, uma condição saudável de baixa energia.

**Local Outlier Factor:**
- **Fold 0:** TPR de 88,5%, FPR de 17,1% e AUC de 0,942. Os falsos positivos vêm dos arquivos H_H_2_1 (42% das janelas) e H_H_8_1 (26%).
- **Estabilidade:** na validação cruzada, o LOF tem o menor FPR médio entre os detectores (11,2% ± 5,5%) e a menor variação de FPR entre os folds. O Isolation Forest tem a maior AUC média (0,970, contra 0,955) e o maior TPR (93,8%, contra 90,6%), mas com FPR médio de 19,7% e muito mais variável. A escolha entre os dois depende do custo de um alarme falso em relação ao de uma falha não detectada.
- **Onde falha:** perde as falhas a 15 Hz (57,0% de detecção), o R-U (75,0%) e o K-A (72,4%), além de parte do S-W (82,9%).

**Comparação dos detectores (fold 0):**
- **Falhas fortes são detectadas por todos os detectores de aprendizado de máquina:** B-R e R-M têm 100% de detecção nos três, e F-B e V-U ficam entre 82% e 100%. O K-A fica entre 71% e 74% apenas porque um dos seus arquivos de teste está a 15 Hz (K_A_1_1).
- **Falhas em baixa velocidade são o principal ponto fraco:** a 15 Hz, o baseline não detecta nenhuma janela com falha, e os detectores ficam entre 50% (OC-SVM) e 67% (Isolation Forest). Nas demais velocidades, o Isolation Forest detecta 100% das falhas (98,9% na rampa de 45 a 15 Hz). O arquivo R_U_1_0 não é detectado por nenhum método, e o K_A_1_1 tem apenas 1 janela detectada, pelo LOF.
- **O efeito da carga depende do detector:** com carga, o TPR do Isolation Forest quase não muda (de 92,9% para 92,5%), mas o do OC-SVM cai de 86,8% para 79,3% e o do LOF de 91,0% para 86,1%. O baseline detecta mais com carga (72,6%, contra 64,3%).
- **Os falsos positivos se concentram em poucos arquivos:** H_H_8_1 gera alarmes falsos nos quatro detectores, H_H_2_1 gera 42% no LOF, e H_H_1_0 gera 42% no OC-SVM. As 20 features de detecção não são normalizadas pela rotação, então os detectores aprendem a "faixa de velocidade" junto com a "saúde" do motor.
- **Agregação por arquivo e alarme temporal:** o tamanho efetivo do teste é de 32 arquivos, e não de 608 janelas. Com voto majoritário por arquivo, o baseline, o OC-SVM e o LOF não geram alarme falso nos 4 arquivos saudáveis do teste; o Isolation Forest erra 1 de 4 (H_H_8_1) e detecta 92,9% dos arquivos com falha (26 de 28). Com a regra de alarme após 3 janelas anômalas consecutivas, o FPR por janela cai para 10,5% no Isolation Forest, 7,9% no OC-SVM e 6,6% no LOF, ao custo de um TPR menor.
- **Validação cruzada:** pela AUC média, a ordem é Isolation Forest (0,970) > LOF (0,955) > OC-SVM (0,939) > baseline de RMS (0,910). Todos os métodos de aprendizado de máquina superam o baseline em AUC e em TPR; em FPR médio, porém, o baseline (10,9%) fica no mesmo nível do LOF.

### Diagnóstico de falhas

**Random Forest** — 70,4% de acurácia no fold 0 e 74,7% ± 4,6% na validação cruzada (o fold 0 é o pior dos 4 folds).
- **H-H:** 76 de 76 janelas corretas. Porém, 36 janelas com falha foram classificadas como H-H (19 de S-W e 17 de R-U), reduzindo a precisão da classe para 0,68 — o erro mais grave, pois equivale a uma falha não detectada. Esses erros se concentram nos arquivos a 15 Hz R_U_1_0 e S_W_1_0.
- **F-B:** 76 de 76 janelas corretas, com 7 falsos positivos (vindos do S-W). As features mais importantes do modelo são as energias de banda e os momentos espectrais (bandas de 2–10 kHz e de 500–2000 Hz, média e desvio espectral), e não as features de envelope.
- **B-R:** 68 de 76 janelas corretas (89,5%); os erros vão principalmente para o R-M (6 janelas).
- **R-M → B-R:** 19 janelas de R-M foram classificadas como B-R, concentradas no arquivo R_M_1_0 (15 Hz). As duas falhas mecânicas geram vibração ligada à rotação do eixo ($1\times f_r$ e $2\times f_r$); sem medição de fase ou de órbita do eixo, as features espectrais globais ficam parecidas.
- **K-A → V-U:** 24 janelas. As duas falhas são elétricas e alteram o campo magnético do motor. Como o motor é alimentado por inversor, não existe uma frequência de rede fixa, e nenhuma feature mede bandas laterais.
- **S-W ↔ V-U:** 18 janelas de S-W preditas como V-U, e 21 de V-U como S-W. O S-W é a classe com pior acurácia (23,7%). Ambas compartilham a mesma raiz eletromagnética: um curto entre espiras no estator altera a impedância de uma fase de forma semelhante a um desequilíbrio nas tensões de entrada.
- **Cascata IF → RF:** a acurácia não muda no fold 0 (70,4%) e cai pouco na validação cruzada (74,1% ± 5,1%): as janelas que o Isolation Forest deixa passar (15 Hz) já eram, em grande parte, classificadas como H-H ou com a classe errada pelo Random Forest.

**Nearest Centroid** — pior desempenho: 28,0% no fold 0 e 32,4% ± 3,0% na validação cruzada (o acaso, com 8 classes, seria 12,5%).
- **H-H:** 64 de 76 janelas corretas (84%), mas precisão de apenas 0,65, porque 35 janelas com falha caem no centróide saudável.
- **R-M:** nenhum acerto (47 janelas para K-A e 19 para B-R). **K-A:** 11% (33 para B-R e 23 para S-W). **F-B:** 12% (58 para K-A), contra 100% no Random Forest.
- **Justificativa:** cada classe reúne 8 perfis de velocidade e 2 cargas; a nuvem de cada classe é multimodal, e um único centróide médio fica em uma região onde há poucas janelas reais. A distância euclidiana também dá o mesmo peso a todas as features.
- **Cascata OC-SVM → NC:** 27,3% no fold 0 e 33,9% ± 4,9% na validação cruzada.

**Multi-Model Autoencoders** — 59,0% no fold 0 e 58,3% ± 3,6% na validação cruzada.
- **H-H:** 76 de 76 (precisão de 0,84). **F-B:** 68 de 76 (89%). **B-R:** 67 de 76 (88%).
- **Confusões:** R-M → B-R (22 janelas; R-M com 29% de acerto), K-A → R-U (29 janelas; K-A com 43%), V-U → S-W (24 janelas; V-U com 30%), S-W com 20% de acerto.
- **Justificativa:** cada autoencoder é treinado apenas com a sua classe, mas nada impede que ele também reconstrua bem janelas de outras classes próximas. O critério de menor MSE não é treinado para separar as classes, as redes são rasas e treinadas com apenas 10 arquivos por classe.
- **Cascata LOF → AE:** 56,2% no fold 0 e 55,7% ± 6,1% na validação cruzada.

**Comparação dos classificadores:** no mesmo conjunto de teste, o Random Forest é claramente o melhor (70,4% por janela), seguido pelos autoencoders (59,0%) e pelo Nearest Centroid (28,0%). Com voto majoritário por arquivo, o Random Forest acerta 78,1% dos arquivos (25 de 32) e os autoencoders 71,9% (23 de 32). Dos 7 arquivos errados pelo Random Forest, 5 estão a 15 Hz (K_A_1_1, R_M_1_0, R_U_1_0, S_W_1_0 e S_W_1_1), e 2 deles (R_U_1_0 e S_W_1_0) são classificados como saudáveis. A acurácia de ponta a ponta da cascata fica próxima da do classificador sozinho, o que confirma que **o gargalo do sistema é o diagnóstico, e não a detecção**.

### Ablação dos conjuntos de features

- **As features físicas (ordens e envelope) ajudam o diagnóstico, mas atrapalham a maior parte da detecção:** acrescentá-las às 20 originais aumenta a acurácia do Random Forest de 73,6% para 74,7% e a do Nearest Centroid de 27,2% para 32,4%. Por outro lado, aumenta o FPR do Isolation Forest (de 19,7% para 22,4%) e do OC-SVM (de 15,1% para 27,0%) e reduz a AUC do LOF (de 0,955 para 0,872), embora o FPR do LOF caia de 11,2% para 8,9%. Por isso, a detecção usa as 20 features originais e o diagnóstico usa as 25.
- **Remover as features redundantes quase não muda o diagnóstico:** o conjunto sem redundantes, com bandas relativas e features físicas, teve 76,0% no Random Forest (contra 74,7%) e elevou a AUC do Isolation Forest para 0,996, mas com FPR de 25,0%. A diferença está dentro do desvio entre os folds, e o conjunto de diagnóstico foi mantido para não escolher o modelo pelos mesmos folds em que ele é avaliado.
- **Fusão de sensores:** com as features sem redundantes, relativas e físicas, a AUC do Isolation Forest fica entre 0,992 e 0,999, mas a fusão aumenta os alarmes falsos (FPR de 37,8% com 3 acelerômetros e 28,9% com 4 sensores). Com as 20 features originais por sensor, os 4 sensores dão o **menor** FPR do Isolation Forest (13,2%) e a maior AUC do OC-SVM (0,950), mas aumentam o FPR do OC-SVM (21,1%) e do LOF (19,4%). No Random Forest, os conjuntos com fusão ficam entre 73,1% e 76,2%.
- **Cuidado com diferenças pequenas:** com 4 folds e desvios de cerca de 5 pontos, diferenças de 1 a 3 pontos no Random Forest não são conclusivas.

### Sensibilidade dos hiperparâmetros dos detectores

- **`contamination` controla o compromisso entre TPR e FPR:** no Isolation Forest, passar de 0,05 para 0,01 reduz o FPR médio de 19,7% para 1,3%, e o TPR cai de 93,8% para 79,3%. No LOF, `contamination = 0.02` dá TPR de 88,8% com FPR de 6,9%.
- **O FPR no teste é maior que o valor de `contamination`:** os arquivos saudáveis do teste estão em condições de operação diferentes das do treino, e o limiar calibrado no treino não se transfere integralmente.
- **O `nu` do OC-SVM quase não muda o FPR (de 14,8% a 21,7%):** o FPR vem principalmente da forma da fronteira RBF.
- **`n_neighbors` do LOF:** 10 vizinhos dão a maior AUC (0,975), mas FPR de 22,7%; 35 vizinhos dão o menor FPR (4,3%), mas TPR de 83,5%. O valor usado, 20, é um compromisso (TPR de 90,6% e FPR de 11,2%).
- **Recomendação:** para manutenção, um limiar mais conservador (`contamination` entre 0,01 e 0,02) reduz bastante os alarmes falsos com perda moderada de detecção. Os resultados principais mantêm 0,05 e 20 vizinhos para seguir o critério definido antes da avaliação.

### Generalização entre condições de operação

- **Carga não vista:** treinando sem carga e testando com carga, a acurácia do Random Forest cai de 74,7% para 37,3%, e o FPR do Isolation Forest e do LOF sobe para 53,3% e 50,7%. No sentido inverso, o Random Forest fica em 43,3%, enquanto os detectores mantêm FPR baixo (0% no Isolation Forest e 11,8% no LOF). O diagnóstico não generaliza para uma carga que não viu no treino — a limitação mais importante do trabalho para uso real.
- **Velocidade não vista:** com 15 Hz fora do treino, os detectores encontram só cerca de metade das falhas (Isolation Forest 51,9% e LOF 51,1%), e o Random Forest acerta 49,7%. Com 60 Hz fora do treino, os detectores separam perfeitamente falha e saudável no ranqueamento (AUC = 1,000), mas o Isolation Forest dá alarme falso em 50% das janelas saudáveis. O Random Forest fica em 68,8%.
- **Conclusão:** as features globais mudam com a velocidade e a carga, e os modelos aprendem a condição de operação junto com a falha.

---

## 5. Discussão Crítica e Conclusões

### O que funcionou

- **Detecção de falhas fortes:** os três detectores, treinados apenas com dados saudáveis, detectam 100% das janelas de B-R e R-M no fold 0 e todas as falhas nas velocidades de 45 e 60 Hz. Na validação cruzada, a AUC média vai de 0,939 (OC-SVM) a 0,970 (Isolation Forest), acima do baseline de RMS (0,910).
- **Diagnóstico com Random Forest:** 74,7% ± 4,6% de acurácia em 8 classes na validação cruzada, com 100% de acerto em H-H e F-B no fold 0, e 74,1% ± 5,1% na cascata de ponta a ponta.
- **Protocolo de avaliação:** divisão por arquivo antes do janelamento, estratificada por classe e carga; scaler ajustado só no treino; seeds fixas; versões das bibliotecas registradas; e validação cruzada de 4 folds. Os números da Tabela Resumo são gerados diretamente pelas células do notebook.

### O que não funcionou, e por quê

- **Falhas em baixa velocidade:** a 15 Hz, o baseline de RMS não detecta nenhuma falha, e o melhor detector (Isolation Forest) detecta 66,7% das janelas. O arquivo R_U_1_0 não é detectado por nenhum método, e o K_A_1_1 tem apenas 1 janela detectada. A força de desbalanceamento é proporcional a ω²: a 15 Hz, ela é cerca de 1/16 da força a 60 Hz, e a amplitude em 1× fr do motor desbalanceado (0,15 m/s²) fica próxima da do motor saudável (0,07 m/s²).
- **Falsos positivos instáveis:** com o limiar padrão, o FPR médio vai de 11,2% (LOF) a 19,7% (Isolation Forest) e varia de 0% a 36,8% entre os folds. Os detectores são treinados com apenas 12 arquivos saudáveis, um por condição de operação, e as features de detecção não são normalizadas pela rotação: condições saudáveis diferentes das do treino parecem anômalas.
- **Confusões no diagnóstico:** entre falhas mecânicas ligadas à rotação (B-R, R-M e R-U) e entre falhas eletromagnéticas (K-A, V-U e S-W). As features são globais; não há análise de fase, de órbita do eixo, de bandas laterais nem de corrente elétrica, que ajudariam a separar essas falhas.
- **Nearest Centroid e autoencoders:** um protótipo por classe não representa classes que mudam com a condição de operação (32,4% de acurácia), e o critério de menor erro de reconstrução não é treinado para separar as classes (58,3%).
- **Generalização entre condições:** o Random Forest cai para 37,3% quando a carga do teste não foi vista no treino, e o FPR dos detectores passa de 50%.
- **Features físicas e fusão de sensores:** as features de ordem e envelope melhoram o Random Forest (de 73,6% para 74,7%) e o Nearest Centroid, mas pioram a maior parte dos detectores; a fusão de sensores aumenta a AUC, e o efeito sobre os alarmes falsos depende do conjunto de features. As frequências BPFO e BPFI calculadas para a geometria padrão do 6205 não coincidem com o pico de envelope observado no F-B (~4,9× fr).

### Limitações dos dados e da avaliação

- **Um motor por classe:** o dataset usa 8 motores físicos diferentes, um por classe (SEHRI; DUMOND, 2024). Na EDA a 60 Hz, todos os motores com falha têm vibração global de 3,7 a 12,1 vezes maior que a do saudável e o mesmo pico dominante em 3× fr, que no motor saudável é pequeno. Parte do que os modelos aprendem pode ser a identidade do motor, e não a falha. Não é possível separar esses efeitos com este dataset, e os resultados provavelmente superestimam o desempenho em um motor novo.
- **Tamanho efetivo da amostra:** as 19 janelas de um arquivo são correlacionadas, e o teste de cada fold tem 32 arquivos (4 saudáveis). Cada janela saudável vale cerca de 1,3 ponto de FPR, e diferenças pequenas entre os métodos não são estatisticamente conclusivas.
- **Seleção de modelo sem validação aninhada:** os conjuntos de features e a discussão dos hiperparâmetros (`contamination`, `nu` e `n_neighbors`) usam os mesmos 4 folds em que os resultados são reportados. Os hiperparâmetros principais foram mantidos nos valores definidos antes da avaliação, mas a escolha dos conjuntos de features olhou esses folds. O viés esperado é pequeno, pois as diferenças entre os conjuntos são menores que o desvio entre os folds, mas uma estimativa sem viés exigiria validação cruzada aninhada.
- **Severidade desconhecida:** o artigo do dataset não informa a severidade das falhas, então não é possível avaliar a detecção em função da severidade.

### O que faríamos com mais tempo ou mais dados

- **Mais motores por classe** (ou a mesma falha introduzida em motores diferentes), para separar a falha da identidade do motor, e mais registros saudáveis em cada condição de operação.
- **Limiar por condição de operação** (velocidade e carga), ou normalização das features pelo comportamento saudável em cada condição.
- **Geometria real do rolamento**, para calcular BPFO, BPFI e BSF corretos, e análise de ordens com rastreamento da rotação (*order tracking*) nas rampas.
- **Fusão de sensores com seleção de features ou regularização**, para aproveitar o ganho de AUC sem aumentar os alarmes falsos.
- **Validação cruzada aninhada** para a escolha dos conjuntos de features e dos hiperparâmetros.
- **Validação deixando uma condição de operação fora do treino** como protocolo principal, simulando a operação em condições não vistas.
- **Melhorias específicas por modelo:** um centróide por classe e por condição de operação, ou a distância de Mahalanobis, no Nearest Centroid; ajustar o limiar do OC-SVM pelo FPR desejado; treinar os autoencoders com um termo que penalize a reconstrução de outras classes, usar um classificador sobre o vetor de erros de reconstrução das 8 redes ou usar as representações latentes como features do Random Forest.

---

## 6. Referências Bibliográficas

BREIMAN, L. Random Forests. *Machine Learning*, v. 45, n. 1, p. 5-32, 2001.

BREUNIG, M. M. et al. LOF: identifying density-based local outliers. *Proceedings of the 2000 ACM SIGMOD International Conference on Management of Data*, p. 93-104, 2000.

ISO 20816-1:2016. *Mechanical vibration: measurement and evaluation of machine vibration*. Disponível em: <https://www.iso.org/standard/63180.html>.

KRAMER, M. A. Nonlinear principal component analysis using autoassociative neural networks. *AIChE Journal*, v. 37, n. 2, p. 233-243, 1991.

LIU, F. T.; TING, K. M.; ZHOU, Z-H. Isolation Forest. *Proceedings of the 2008 Eighth IEEE International Conference on Data Mining (ICDM)*, p. 413-422, 2008.

RANDALL, R. B.; ANTONI, J. Rolling element bearing diagnostics: a tutorial. *Mechanical Systems and Signal Processing*, v. 25, n. 2, p. 485-520, 2011. DOI: [10.1016/j.ymssp.2010.07.017](https://doi.org/10.1016/j.ymssp.2010.07.017).

RANDALL, R. B. *Vibration-based Condition Monitoring: Industrial, Aerospace and Automotive Applications*. Chichester: Wiley, 2011.

SCHÖLKOPF, B. et al. Estimating the support of a high-dimensional distribution. *Neural Computation*, v. 13, n. 7, p. 1443-1472, 2001.

SEHRI, M.; DUMOND, P. University of Ottawa constant and variable speed electric motor vibration and acoustic fault signature dataset. *Data in Brief*, v. 53, 110144, 2024. DOI: [10.1016/j.dib.2024.110144](https://doi.org/10.1016/j.dib.2024.110144).

SMITH, W. A.; RANDALL, R. B. Rolling element bearing diagnostics using the Case Western Reserve University data: a benchmark study. *Mechanical Systems and Signal Processing*, v. 64-65, p. 100-131, 2015. DOI: [10.1016/j.ymssp.2015.03.021](https://doi.org/10.1016/j.ymssp.2015.03.021).

TIBSHIRANI, R. et al. Diagnosis of multiple cancer types by shrunken centroids of gene expression. *Proceedings of the National Academy of Sciences*, v. 99, n. 10, p. 6567-6572, 2002. DOI: [10.1073/pnas.082099299](https://doi.org/10.1073/pnas.082099299).

---

## 7. Declaração de Uso de Inteligência Artificial

Este trabalho utilizou as ferramentas de IA **Claude Code** (Anthropic) e **Gemini** (Google) nas seguintes etapas:

- **Avaliação do notebook:** revisão da metodologia, reprodução dos resultados e identificação de problemas.
- **Implementação das correções:** apoio na reescrita de códigos de análises e documentação geral de cada etapa, além da discussão a partir das saídas do notebook.
- **Revisão final:** conferência dos números citados no texto com as saídas das células, troca da divisão dos folds por `StratifiedKFold` estratificado por classe e carga, registro das versões das bibliotecas e inclusão da tabela quantitativa da EDA.
- **Documentação do repositório:** montagem deste README a partir dos blocos de texto do notebook.

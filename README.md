# PULE DO GATO

Protótipo de análise de futebol desenvolvido coletivamente no **EDScript Hackathon 2026**, na categoria **Dados & Inteligência Artificial**.

## Problema e proposta

Acompanhar uma partida envolve interpretar informações dispersas: placar, estatísticas ao vivo, histórico das equipes e desempenho dos jogadores. O PULE DO GATO propõe reunir esses dados em uma interface que facilite a leitura do jogo por usuários da plataforma Esportes da Sorte.

A solução combina dados esportivos, estatísticas descritivas e explicações geradas por IA. A proposta também contempla a investigação de possíveis surpresas esportivas, conhecidas como “zebras”, mas esse detector não foi concluído no protótipo.

## Principais funcionalidades

- **Partidas ao vivo:** consulta à BetsAPI com equipes, placar e tempo de jogo.
- **Painel de análise:** reúne dados da partida e explicações geradas por IA, com consultas ao backend a cada 15 segundos.
- **Indicadores descritivos:** índice de pressão calculado pela divisão de ataques perigosos pelo minuto da partida e agregações de equipes a partir de CSVs locais.
- **Contexto histórico:** classificação heurística do perfil ofensivo com base nas finalizações de uma partida disponível no StatsBomb Open Data.
- **Perfil de jogador:** estatísticas individuais de uma base local e geração de insights com Gemini.
- **Interface responsiva:** navegação entre partidas, painel de análise e perfil de jogador.

Algumas telas e indicadores utilizam dados demonstrativos ou valores de contingência. O projeto representa a implementação do hackathon e possui fluxos parciais.

## Demonstração

Espaço reservado para capturas reais do sistema, a serem adicionadas futuramente.

| Tela | Conteúdo previsto para a captura |
| --- | --- |
| Listagem de partidas | Equipes, placar e tempo de jogo |
| Painel de análise | Indicadores e explicações da partida |
| Perfil de jogador | Estatísticas individuais e insight gerado por IA |

## Tecnologias

| Camada | Tecnologias |
| --- | --- |
| Frontend | **React 19**, **TypeScript**, Vite, React Router, Tailwind CSS e Lucide React |
| Backend | **Python**, **FastAPI**, Uvicorn, Pydantic e HTTPX |
| Processamento de dados | pandas e arquivos CSV locais |
| Dados esportivos | **BetsAPI** e **StatsBomb Open Data**, via statsbombpy |
| IA generativa | **Google Gemini** e **Groq** |
| Configuração | Variáveis de ambiente e python-dotenv |

## Arquitetura e dados

```text
Frontend React / TypeScript
          |
          | HTTP / JSON
          v
Backend Python / FastAPI
          |-- BetsAPI: partidas e estatísticas ao vivo
          |-- CSVs locais: estatísticas de jogadores e equipes
          |-- StatsBomb: eventos históricos
          |-- Gemini / Groq: explicações e conteúdo do painel
```

O backend organiza as integrações em serviços e utiliza caches em memória para a listagem e as análises generativas. Não há banco de dados ou autenticação implementados.

- **BetsAPI:** fornece partidas e estatísticas mediante token de acesso.
- **StatsBomb Open Data:** fornece eventos de competições e temporadas históricas predefinidas, sem cobertura garantida das partidas atuais.
- **Base local de jogadores:** o serviço denominado FBref lê `backend/data/players_data_light-2025_2026.csv`, com dados de cinco ligas europeias; não consulta o FBref em tempo de execução. A procedência, a data de extração e as condições de redistribuição ainda precisam ser documentadas.

A relação implementada com a Esportes da Sorte consiste em links para o site externo. Não há integração com contas, saldo ou execução de apostas.

## Como a IA é utilizada

O backend reúne estatísticas da BetsAPI, agregações da base local e a classificação histórica do StatsBomb em um prompt. O **Gemini** gera explicações, recomendações textuais e conteúdo estruturado para o painel; o **Groq** atua como alternativa quando disponível. Os insights individuais de jogadores utilizam apenas Gemini.

**Não há modelo preditivo treinado, calibração ou avaliação de acurácia no repositório.** Percentuais e recomendações produzidos pelos modelos de linguagem não são probabilidades estatisticamente validadas e não devem ser interpretados como estimativas confiáveis para apostas. O índice de pressão e a classificação por finalizações são cálculos e regras determinísticas.

## Execução local

### Pré-requisitos

- Python compatível com `backend/requirements.txt`, com suporte a `venv`. O projeto não declara uma versão de Python.
- Node.js compatível com Vite 8 (`^20.19.0` ou `>=22.12.0`) e npm.
- Token da BetsAPI para dados ao vivo e chave de Gemini e/ou Groq para geração do painel.
- Acesso à internet para instalação de dependências e uso das integrações externas.

### Backend

A partir da raiz do repositório, em Linux ou macOS:

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Configure as variáveis no ambiente do processo ou em um arquivo local `backend/.env`, preenchendo apenas as credenciais dos serviços utilizados:

```dotenv
BETS_API_TOKEN=
GEMINI_API_KEY=
GROQ_API_KEY=
```

| Variável | Finalidade |
| --- | --- |
| `BETS_API_TOKEN` | Consultar partidas e estatísticas na BetsAPI |
| `GEMINI_API_KEY` | Gerar análises do painel e insights de jogadores |
| `GROQ_API_KEY` | Disponibilizar uma alternativa de geração para o painel |

Não versione credenciais. Sem chaves de IA, o painel pode exibir conteúdo de contingência, que não corresponde a uma análise gerada por modelo.

Com o ambiente virtual ativo e ainda em `backend/`:

```bash
python -m uvicorn app.main:app --reload --port 8000
```

API: `http://localhost:8000`. Documentação interativa: `http://localhost:8000/docs`.

### Frontend

Em outro terminal, a partir da raiz:

```bash
cd frontend
npm ci
npm run dev
```

Acesse o endereço informado pelo Vite, normalmente `http://localhost:5173`, origem permitida no CORS do backend.

O endereço padrão da API é `http://localhost:8000`. Para configurá-lo explicitamente, defina `VITE_API_URL` no ambiente do Vite ou em um arquivo local `frontend/.env`:

```dotenv
VITE_API_URL=http://localhost:8000
```

Variáveis com prefixo `VITE_` são públicas no frontend e não devem conter segredos. Outra origem para o frontend exige adequação do CORS no backend.

Para gerar o build, execute em `frontend/`:

```bash
npm run build
```

Os comandos refletem a configuração do repositório; a instalação e a inicialização não foram executadas nesta revisão documental.

## Limitações conhecidas

- **Detector de zebras incompleto:** depende de um método inexistente; sua análise contextual ainda não executa inferência e não há busca de notícias implementada.
- **Recursos demonstrativos:** dashboard, seletores e alguns indicadores usam dados locais ou fixos. As escolhas de esporte e liga não filtram a API; a listagem não filtra por índice de pressão. A projeção de dez minutos apenas exibe conteúdo recebido, sem nova simulação ou processamento de uma janela real desse período.
- **Dados e indicadores:** a cobertura histórica é limitada. O campo `xg_total` usa gols sem pênaltis como proxy e não representa expected goals.
- **Fluxos parciais:** há inconsistências no placar do detalhe e na resposta de timeout da listagem; a rota `/perfil` sem identificador permanece em carregamento, e o scouting possui tratamento incompleto de falhas.
- **Dependências e validação:** o funcionamento depende de credenciais, quotas e disponibilidade dos provedores, inclusive dos modelos configurados. Não há testes automatizados ou controle de acesso aos endpoints.

As análises são experimentais e não oferecem garantia de resultados esportivos ou financeiros.

## Desenvolvimento coletivo

O PULE DO GATO é resultado do trabalho coletivo da equipe no EDScript Hackathon 2026. O repositório reúne a interface, as integrações de dados e a aplicação de IA generativa construídas durante o evento.

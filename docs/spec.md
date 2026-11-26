# VagaCerta: especificação do MVP

## Para quem é

Pessoas que procuram o primeiro emprego em TI (estágio e júnior) e perdem horas lendo vagas no LinkedIn e em outros sites, sem saber se o próprio currículo combina com elas.

## Problema

- Ler dezenas de vagas toma tempo.
- É difícil saber se o currículo atende ao que a vaga pede e o que falta.
- Muitas vagas aparecem acima do nível de quem está começando.
- Os sites mostram vagas, mas não explicam por que ela combina com você.

## O que o MVP faz

1. **Conta:** cadastro e login.
2. **Currículo:** o usuário envia o PDF; o app extrai o texto e monta um perfil (nível, tecnologias, experiência) que o usuário confere e corrige.
3. **Preferências:** modalidade (remoto, híbrido, presencial), cidade e, se quiser, faixa salarial.
4. **Descoberta de vagas:** o app busca vagas pela API do Adzuna com as preferências e lista título, local, trecho e link, já com um pré-filtro de senioridade.
5. **Análise:** o usuário cola a descrição completa de uma vaga e o app mostra os requisitos, as evidências do currículo, a cobertura em % e a opinião da IA.
6. **Vagas salvas:** as vagas analisadas ficam guardadas para o usuário comparar.

## O que fica de fora do MVP

- Análise automática da descrição completa (depende de uma fonte que entregue o texto inteiro).
- Raspagem de sites que não permitem.
- Outras áreas além de TI.
- Leitura de PDF escaneado (OCR).
- Pagamento e planos.
- Candidatura automática.
- Visual elaborado.

## Planejado para depois do MVP

- **Aplicativo de celular:** o uso principal tende a ser no celular, então é prioridade alta depois do MVP.
- **Preparação para entrevista** e "o que estudar para a vaga".

## Arquitetura

- **Backend:** API REST em Java com Spring Boot. Toda a regra de negócio fica nele.
- **Front-end:** React, como site responsivo mobile-first, construído **depois** que o back estiver fechado. Ele apenas consome a API.
- Um aplicativo nativo de celular fica para depois do MVP e reaproveita a mesma API.

## Ordem de construção

1. **Back-end, por fatias**, cada uma validada no Postman (coleção salva em `docs/`) e com testes automatizados nas regras de negócio:
    1. Conta e login (JWT)
    2. Currículo (upload do PDF, extração do texto, perfil)
    3. Preferências e descoberta de vagas (Adzuna)
    4. Análise da vaga (requisitos, evidências, cobertura em %, opinião da IA)
    5. Vagas salvas
2. **Back-end pronto** quando o fluxo completo funcionar no Postman, do cadastro até a vaga salva.
3. **Front-end React mobile-first**, tela por tela, sobre a API já pronta.

## Requisitos da API pensando no celular

- Autenticação por token (JWT).
- Listas paginadas e respostas enxutas.
- Limite de tamanho no upload do PDF.
- Erros sempre no mesmo formato JSON.
- CORS liberado só para a origem do front.

## Estrutura do repositório

- Um único repositório para backend e front-end.
- Pastas na raiz: `backend/` (Spring Boot), `frontend/` (React) e `docs/`.

## De onde vêm as vagas

- Teste com a API do Adzuna (Brasil): há volume, mas a descrição vem cortada em cerca de 500 caracteres, às vezes mascarada, e muitas vagas vêm sem o nome da empresa.
- Por isso a API serve para **descobrir** vagas e fazer o pré-filtro; a **análise de requisitos** roda só sobre a descrição completa colada pelo usuário.
- Vaga com texto mascarado, curto demais ou sem requisitos técnicos: o app avisa "não dá para analisar esta vaga" e não calcula percentual.

## Dados guardados

- **Conta:** e-mail e senha (guardada com hash, nunca em texto).
- **Currículo:** o texto extraído e o perfil montado.
- **Preferências** de busca.
- **Vagas salvas** e suas análises.
- **Não guardado:** o PDF original. Dados de contato (telefone, e-mail, endereço) são removidos do texto antes de ir para a IA.
- Apagar a conta apaga todos os dados do usuário.

## Como a IA entra

Princípio: o código decide e valida; a IA interpreta e explica.

**O que o código faz (sem IA)**
- Cadastro, login e armazenamento.
- Filtros objetivos: modalidade, cidade e faixa salarial.
- Remover vagas duplicadas e marcar vagas encerradas.
- Calcular a cobertura em %.
- Validar o formato da resposta da IA e conferir as citações.

**O que a IA faz**
1. Lê o currículo e extrai o perfil estruturado, que o usuário confirma.
2. Lê a vaga e extrai os requisitos (tecnologias, nível, modalidade, anos de experiência), marcando cada um como obrigatório ou desejável.
3. Para cada requisito, indica se há evidência no currículo, com citação literal.
4. Escreve uma opinião curta.

## Currículo em PDF

- O app extrai o texto do PDF.
- PDF sem texto extraível (escaneado ou só imagem) é recusado com mensagem clara.
- Validar tipo e tamanho máximo do arquivo.
- Guardar só o texto extraído.

## Cobertura de requisitos

- O código calcula: requisitos atendidos ÷ requisitos totais (obrigatórios podem ter peso maior).
- A tela mostra o número com a conta ("6 de 9 requisitos") e a lista de cada requisito.
- É "cobertura de requisitos", e não "chance de ser contratado".

## Explicação com evidência

- Para cada requisito, o app monta a frase a partir de dados verificados: o requisito, se foi encontrado e a citação literal do currículo.
- A citação só aparece se o código confirmou que ela existe no texto do currículo.
- A opinião da IA aparece separada, rotulada como "Opinião da IA", e não pode afirmar nada sobre o currículo além dos dados verificados.
- Requisitos implícitos ficam marcados como "provável" e não contam como atendidos.

## Regras de senioridade

- O perfil tem um nível. A IA sugere a partir do currículo e o usuário confirma. A palavra final é sempre da pessoa.
- Nível da vaga em duas camadas: primeiro regras simples no título (estágio, trainee, júnior/jr, pleno, sênior/sr); a IA só entra quando a regra não decide.
- Vagas acima do nível não são escondidas: ficam em seção separada, com aviso. Na dúvida, mostra com aviso.
- O usuário pode corrigir o nível de uma vaga.

## Limites, custo e privacidade com IA

- A chave da API fica só em variável de ambiente, nunca no Git.
- Limite de gasto mensal na conta da API e limite de análises por usuário por dia.
- Mandar à IA só o necessário do currículo.

## Como medimos a qualidade da IA

- A resposta da IA é validada contra um formato fixo; se vier inválida, tenta mais uma vez e depois mostra erro.
- Conjunto de teste: cerca de 30 vagas reais com os requisitos e o nível classificados por mim.
- Medimos requisitos acertados, inventados e esquecidos, a validade das citações e os erros do tipo "escondeu uma vaga que servia".
- Toda mudança de prompt ou de modelo reexecuta esse conjunto.

## Decisões em aberto

- Qual provedor e modelo de IA (custo, privacidade, qualidade), comparando nas 30 vagas.
- Qual biblioteca extrai o texto do PDF.
- Quanto histórico de análises guardar.
- Validar com cerca de 5 pessoas que estão procurando vaga se o problema e o fluxo fazem sentido.
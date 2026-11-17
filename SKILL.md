---
name: supa-rpg-characters
description: >-
  Identifica personagens em uma lista de prompts, roteiro ou cenas de RPG, associa
  às imagens de referência na pasta Personagens, monta um plano tabular para
  aprovação obrigatória e gera as ilustrações consistentes em lote na pasta Output.
  Use sempre que o usuário enviar múltiplos prompts/cenas de RPG, pedir para
  vincular personagens às referências, manter consistência visual de personagens
  entre cenas ou gerar ilustrações em lote na pasta Output.
argument-hint: "[lista de prompts, cenas ou caminho para arquivo de roteiro]"
metadata:
  version: "2.1.0"
  language: "pt-br"
---

# Supa RPG Characters — Geração Consistente de Cenas com Referências

> **Idioma e Fidelidade:** Conduza toda a interação, plano e relatórios em **Português Brasileiro (PT-BR)**. Preserve o texto original dos prompts na geração (traduzindo, expandindo ou alterando o estilo apenas se o usuário solicitar explicitamente).

Transforma uma lista de prompts ou cenas de RPG em ilustrações visualmente consistentes dentro de `Output/`, vinculando cada personagem citado às suas respectivas imagens de referência em `Personagens/` e executando a geração em lote somente após aprovação explícita do plano.

---

## Fluxo Obrigatório

### 1. Resolver pastas e catalogar referências
1. **Localizar diretórios:**
   - Busque `Personagens/` e `Output/` na raiz do workspace ativo.
   - Caso `Personagens/` não exista na raiz do workspace ativo, verifique subpastas imediatas antes de interromper o fluxo e peça o caminho exato ao usuário.
   - Crie `Output/` automaticamente caso ainda não exista. Se `Output/` já contiver arquivos numerados (ex.: `01_...` a `04_...`), identifique o maior prefixo numérico existente (`N`) para oferecer no plano a opção de continuar a numeração a partir de `N+1`, além de avisar caso algum nome planejado vá sobrescrever um arquivo existente.
2. **Catalogar imagens recursivamente:**
   - Liste todas as imagens em `Personagens/` com extensões `.png`, `.jpg`, `.jpeg` e `.webp` (ignorando arquivos ocultos ou temporários).
   - **Agrupamento inteligente:**
     - Se a imagem estiver na raiz de `Personagens/` (ex.: `Morto Vivo.png` ou `Morto Vivo_02.png`), normalize o nome-base removendo sufixos numéricos (`_1`, `_02`, `-alt`) para agrupar múltiplas referências do mesmo personagem.
     - Se a imagem estiver dentro de uma subpasta (ex.: `Personagens/Kael/frente.png`, `Personagens/Kael/armadura.png`), use o nome da subpasta (`Kael`) como nome do personagem e associe todas as imagens daquela pasta a ele.
   - Monte o catálogo interno: `Personagem -> [caminhos absolutos normalizados]`.

**Critério de conclusão:** Todos os arquivos de referência catalogados com caminhos absolutos válidos e agrupados por personagem.

---

### 2. Analisar entrada e identificar personagens nos prompts
1. **Extrair e numerar prompts:**
   - Se o usuário fornecer um caminho de arquivo (`.txt`, `.md`), leia-o primeiro.
   - Extraia cada cena/prompt mantendo a ordem original e numere sequencialmente com dois dígitos (`01`, `02`, `03`... ou três dígitos se houver 100+ cenas).
   - Se cada cena trouxer versão em português e versão em inglês (ex.: `Prompt (Inglês - Recomendado para IA)`), preserve ambas na análise e pergunte/sugira no plano qual versão usar como base (recomendando a versão em inglês quando indicada para IA).
2. **Mapear personagens:**
   - Para cada prompt, identifique todos os personagens mencionados — considerando nome exato, variações de grafia/acentuação, diminutivos, apelidos, títulos ou referências inequívocas a um personagem do catálogo.
   - Associe cada personagem identificado ao(s) caminho(s) de referência correspondente(s).
3. **Classificar pendências explicitamente:**
   - **Sem referência:** Personagem citado no prompt que não possui imagem correspondente em `Personagens/`.
     - Quando o personagem pertencer a um cenário conhecido (ex.: *Tormenta*, *D&D*, campanhas publicadas), ofereça ou execute uma pesquisa web (`search_web`) para coletar traços visuais canônicos que sirvam de guia complementar ao prompt.
   - **Ambíguo:** Termo ou nome que pode corresponder a dois ou mais personagens do catálogo.
   - **Sem personagens:** Cena puramente de cenário, item ou criatura genérica sem personagem nomeado (marcar como `Sem referência de personagem`).

**Critério de conclusão:** Cada prompt numerado tem sua lista de personagens mapeada para caminhos exatos ou marcada com sua pendência específica.

---

### 3. Montar o Plano e aguardar aprovação (Gate obrigatório)
Apresente o plano completo ao usuário e **PARE imediatamente**, aguardando aprovação explícita antes de ler imagens ou invocar qualquer geração.

O plano deve conter:
1. **Resumo de diretórios e parâmetros:**
   - Caminhos absolutos de `Personagens/` e `Output/`.
   - Parâmetros globais assumidos (ex.: `aspect ratio` padrão, estilo visual se especificado no prompt, e alerta caso algum arquivo já exista em `Output/`).
2. **Tabela de associação:**

| # | Arquivo em `Output/` | Personagens identificados | Referências (`Personagens/`) | Resumo do prompt / Notas |
| :--- | :--- | :--- | :--- | :--- |
| `01` | `01_<slug-limpo>.<ext>` | Nome(s) | `[Arquivo.png](file:///...)` | Trecho inicial do prompt ou observação |

   *(Nota: gere `<slug-limpo>` em minúsculas, sem acentos, espaços ou caracteres inválidos no Windows, limitado a ~30 caracteres).*

3. **Pendências e perguntas rápidas (se aplicável):**
   - Liste claramente personagens **Sem referência** ou **Ambíguos**.
   - Pergunte se deseja definir ou alterar proporção (`aspect ratio`, ex.: `16:9`, `1:1`, `3:4`), traço/estilo global ou nomes de saída.
   - Quando aplicável, confirme também: idioma-base (se houver prompts bilíngues), continuação da numeração após arquivos já existentes em `Output/` e uso de pesquisa de lore para personagens sem imagem de referência.
4. **Pedido de confirmação:**
   - Encerre solicitando aprovação direta (ex.: *"Revise o plano acima. Posso iniciar a geração ou deseja ajustar alguma referência, proporção ou nome?"*).

**Ciclo de revisão:** Se o usuário solicitar alterações (trocar referências, ignorar cenas, mudar aspect ratio, etc.), atualize o plano e confirme novamente antes de prosseguir para o Passo 4.

---

### 4. Ancoragem visual e geração em lote em `Output/`
Execute este passo **somente após aprovação explícita** do plano (com todos os ajustes incorporados).

1. **Ancoragem visual (Leitura única por referência):**
   - Use `read_file` para inspecionar cada imagem de referência que será utilizada no plano aprovado (apenas uma vez por arquivo único, mesmo que apareça em várias cenas).
   - Para cada personagem, sintetize uma **âncora visual objetiva e curta** (espécie/idade aparente, traços faciais, cor e corte de cabelo, vestimenta/armadura marcante, paleta de cores e estilo artístico da imagem).
   - **Identidade vs. variação de cena:** Separe explicitamente o que é **identidade imutável** (espécie, tom de pele, rosto, olhos, cabelo/serpentes/crânio, marcas/tatuagens) do que deve ser **adaptado à cena** conforme o roteiro (ex.: traje diferente da referência, máscara removida/afastada para mostrar expressão facial ou mudança simbólica na cor de um item/luz).
   - *Objetivo:* Evitar troca ou mistura de características (*identity bleeding*) quando dois ou mais personagens contracenarem juntos.

2. **Geração via `image-generator` em lotes paralelos:**
   - Execute as cenas aprovadas invocando o subagente `image-generator` (`invoke_subagent` com `TypeName: "image-generator"`).
   - Processe em lotes paralelos controlados de **3 a 4 cenas por vez** para equilibrar velocidade e estabilidade.
   - Estruture o `Prompt` enviado a cada `image-generator` contendo obrigatoriamente:
     1. **Cena original:** O prompt aprovado na íntegra (sem omitir detalhes).
     2. **Parâmetros técnicos:** Proporção (`aspect ratio`) e diretriz de estilo acordados no plano.
     3. **Vincular referências e anti-mistura (quando houver 1+ personagens):**
        - Liste cada personagem com seu caminho absoluto (`Nome` → `caminho/absoluto.png`) e sua **âncora visual curta**.
        - Instrua explicitamente o subagente a preservar fielmente o rosto, cabelo, silhueta, vestimenta e estilo de cada personagem conforme sua respectiva imagem de referência, mantendo-os visualmente distintos na composição sem misturar traços ou roupas entre eles.
     4. **Prefixo sugerido e retorno do caminho:** Informe o prefixo sugerido `<NN>_<slug-limpo>` e peça para retornar o caminho exato salvo por `generate_image` (não exija que o subagente copie ou converta para `.png` em `Output/`, pois o `image-generator` grava `.jpg` no diretório de artefatos e a cópia para `Output/` é feita pelo agente principal).

3. **Normalização de saída e resiliência a falhas:**
   - Verifique o caminho retornado por cada chamada de `image-generator`.
   - Se a imagem tiver sido salva em diretório temporário/artifacts em vez de `Output/`, copie-a imediatamente para `Output/<nome-planejado>.<ext>` preservando a extensão real do arquivo gerado.
   - **Falha pontual:** Se uma cena falhar, não interrompa o restante do lote; conclua as demais cenas e tente 1 regeração da cena que falhou antes de reportar.

**Critério de conclusão:** Todas as cenas aprovadas foram processadas e salvas em `Output/` com os nomes planejados.

---

### 5. Verificação e entrega final
1. **Auditoria de arquivos:**
   - Liste o conteúdo de `Output/` e confirme que todos os arquivos planejados existem e possuem tamanho maior que 0 bytes.
2. **Relatório de entrega:**
   - Apresente uma tabela ou lista final com links clicáveis `file:///...` para todas as imagens geradas em `Output/`.
   - Destaque imediatamente caso alguma cena tenha falhado após a tentativa de recuperação.
   - Informe ao usuário que, caso queira ajustar qualquer imagem específica, basta pedir a regeração pelo número da cena (ex.: *"Refaça a cena 03 deixando a iluminação mais escura"*), reutilizando o catálogo e as âncoras já carregados sem refazer todo o lote.

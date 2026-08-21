# 📖 Documentação Técnica de Mapeamento de BI — `Global_ReportTest`

> **Motor Central de Renderização Dinâmica de Relatórios e Propostas em PDF**  
> **Versão:** 2026.1  
> **Autor:** Nicolas Pereira  
> **Framework:** Adianti Framework + Dompdf + MariaDB / MySQL

---

## 📌 1. Visão Geral da Arquitetura

O `Global_ReportTest` é o componente central responsável por processar e renderizar qualquer layout HTML cadastrado no sistema (`Global_Report`), convertendo tags dinâmicas e laços de repetição em documentos PDF estruturados.

O motor utiliza uma **arquitetura híbrida de dados em 4 camadas**:
1. **Views de BI (`bi_*`) [Prioridade Máxima]:** Fontes de dados tratadas com joins complexos, regras fiscais e formatação de negócio.
2. **Fallback Físico Automático:** Se a view de BI não retornar registros no laço filho, o motor consulta diretamente a tabela física legada.
3. **Enriquecimento Dinâmico (`enrichFallbackData`):** Preenchimento automático em tempo de execução de fotos (produtos/serviços), códigos de barras (CODE128), descrições de CNAE, unidades e processos PCP.
4. **Injeção Global de Identidade Visual (`crm_site_corpo`):** Imagens de capa, logos, banners, cabeçalhos e rodapés institucionais são convertidos para Base64 e disponibilizados como variáveis globais para **todos** os relatórios.

---

## 🗂️ 2. Dicionário de Mapeamentos Estáticos

Módulos com estruturas fixas mapeados no método `getSchemaMapping()`:

| Módulo Lógico | View / Tabela Master | Chave do Loop (`<!--[tag]-->`) | View / Tabela do Laço | Tabela Física (Fallback) | Chave Estrangeira (FK) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CRM Site** | `bi_crm_site` | `corpos` / `crm_site_corpo` | `bi_crm_site_corpo` | `crm_site_corpo` | `crm_site_id` |
| **CRM Site Corpo** | `bi_crm_site_corpo` | *(Nenhum)* | — | — | — |
| **Atendimento Clínico** | `bi_clinic_atendimento` | `servicos` | `bi_clinic_atendimento_servico` | `clinic_atendimento_servico` | `clinic_atendimento_id` |
| **Compras Requisitos** | `bi_compra_requisito` | `produtos` | `bi_compra_requisito_produto` | `compra_requisito_produto` | `compra_requisito_id` |
| **Consórcio** | `bi_consorcio` | *(Nenhum)* | — | — | — |
| **CRM Briefing** | `bi_crm_briefing` | *(Nenhum)* | — | — | — |
| **Currículo** | `bi_curriculo` | `cursos`<br>`experiencias`<br>`idiomas`<br>`profissoes`<br>`vagas_emprego` | `bi_curriculo_curso`<br>`bi_curriculo_experiencia`<br>`bi_curriculo_idioma`<br>`bi_curriculo_profissao`<br>`bi_curriculo_vaga_emprego` | `curriculo_curso`<br>`curriculo_experiencia`<br>`curriculo_idioma`<br>`curriculo_profissao`<br>`curriculo_vaga_emprego` | `curriculo_id` |
| **Balanço de Estoque** | `bi_estoque_balanco` | `produtos` | `bi_estoque_balanco_produto` | `estoque_balanco_produto` | `estoque_balanco_id` |
| **Comissão Faturamento** | `bi_fatur_comis` | `gerados` | `bi_fatur_comis_gerado` | `fatur_comis_gerado` | `fatur_comis_id` |
| **Contratos** | `bi_fatur_contrato` | `animais`<br>`parcelas`<br>`imoveis`<br>`pessoas`<br>`produtos`<br>`servicos`<br>`servicos_plano`<br>`veiculos` | `bi_fatur_contrato_clinic_animal`<br>`bi_fatur_contrato_financ_titulo`<br>`bi_fatur_contrato_imovel`<br>`bi_fatur_contrato_pessoa`<br>`bi_fatur_contrato_produto`<br>`bi_fatur_contrato_servico`<br>`bi_fatur_contrato_servico_plano`<br>`bi_fatur_contrato_veiculo` | `fatur_contrato_clinic_animal`<br>`fatur_contrato_financ_titulo`<br>`fatur_contrato_imovel`<br>`fatur_contrato_pessoa`<br>`fatur_contrato_produto`<br>`fatur_contrato_servico`<br>`fatur_contrato_servico_plano`<br>`fatur_contrato_veiculo` | `fatur_contrato_id` |
| **Desconto Títulos** | `bi_financ_desc_titulo` | `titulos` | `bi_financ_desc_titulo_financ_titulo` | `financ_desc_titulo_financ_titulo` | `financ_desc_titulo_id` |
| **Inadimplência** | `bi_financ_inad` | *(Nenhum)* | — | — | — |
| **Recibos Financeiros** | `bi_financ_recibo` | *(Nenhum)* | — | — | — |
| **PCP 2 OP** | `bi_pcp2_op` | `produtos`<br>`materiais`<br>`processos` | `bi_pcp2_op_produto`<br>`bi_pcp2_op_produto_lista_mat`<br>`bi_pcp2_op_produto_processo` | `pcp2_op_produto`<br>`pcp2_op_produto_lista_mat`<br>`pcp2_op_produto_processo` | `pcp2_op_id` |
| **PCP Apontamentos** | `bi_pcp_apontamento` | *(Nenhum)* | — | — | — |
| **PCP Ficha** | `bi_pcp_ficha` | `equipamentos`<br>`especificacoes`<br>`processos`<br>`produtos`<br>`testes_qualidade` | `bi_pcp_ficha_equipamento`<br>`bi_pcp_ficha_especificacao`<br>`bi_pcp_ficha_processo`<br>`bi_pcp_ficha_produto`<br>`bi_pcp_ficha_teste_qld` | `pcp_ficha_equipamento`<br>`pcp_ficha_especificacao`<br>`pcp_ficha_processo`<br>`pcp_ficha_produto`<br>`pcp_ficha_teste_qld` | `pcp_ficha_id` |
| **Produção 1 (PCP OP)** | `bi_pcp_op` | `equipamentos`<br>`processos`<br>`produtos`<br>`materiais`<br>`testes_qualidade` | `bi_pcp_op_equipamento`<br>`bi_pcp_op_processo`<br>`bi_pcp_op_produto`<br>`bi_pcp_op_produto`<br>`bi_pcp_op_teste_qld` | `pcp_op_equipamento`<br>`pcp_op_processo`<br>`pcp_op_produto`<br>`pcp_op_produto`<br>`pcp_op_teste_qld` | `pcp_op_id` |
| **Pessoas** | `bi_pessoa` | `analises_pf`<br>`enderecos`<br>`atividades` | `bi_pessoa_analise_pf`<br>`bi_pessoa_endereco`<br>`bi_pessoa_tributo_tab_atividade` | `pessoa_analise_pf`<br>`pessoa_endereco`<br>`pessoa_tributo_tab_atividade` | `pessoa_id` |
| **Pessoa Análise PF** | `bi_pessoa_analise_pf` | `emails`<br>`enderecos`<br>`fones`<br>`negativos`<br>`participacoes` | `bi_pessoa_analise_pf_email`<br>`bi_pessoa_analise_pf_endereco`<br>`bi_pessoa_analise_pf_fone`<br>`bi_pessoa_analise_pf_negativo`<br>`bi_pessoa_analise_pf_participacao` | `pessoa_analise_pf_email`<br>`pessoa_analise_pf_endereco`<br>`pessoa_analise_pf_fone`<br>`pessoa_analise_pf_negativo`<br>`pessoa_analise_pf_participacao` | `pessoa_analise_pf_id` |
| **Produtos** | `bi_produto` | `materiais`<br>`tabelas`<br>`unidades` | `bi_produto_lista_material`<br>`bi_produto_tabela`<br>`bi_produto_unidade` | `produto_lista_material`<br>`produto_produto_tabela`<br>`produto_produto_unidade` | `produto_id` |
| **Serviços** | `bi_servico` | `movimentos` | `bi_servico_movimento` | `servico_movimento` | `servico_id` |

---

## 🔀 3. Mapeamentos Polimórficos Dinâmicos

### 3.1. Faturamento e Notas Fiscais (`fatur_nf` / `danfe`)
O motor identifica o campo `tipo` da tabela `fatur_nf` no banco de dados para chavear a consulta:

* **Se `tipo = 'entrada'`**:
  * **Master:** `bi_fatur_nf_entrada`
  * **Parcelas (`<!--[parcelas]-->`):** `bi_pagar` *(fallback: `financ_titulo`)*
* **Se `tipo = 'saida'`**:
  * **Master:** `bi_fatur_nf_saida`
  * **Parcelas (`<!--[parcelas]-->`):** `bi_receber` *(fallback: `financ_titulo`)*
* **Laços comuns (ambos os tipos):**
  * `produtos` → `bi_estoque_movimento` *(fallback: `estoque_movimento`)*
  * `servicos` → `bi_servico_movimento` *(fallback: `servico_movimento`)*

---

### 3.2. Pedidos, Orçamentos, Ordens de Serviço e PDV (`pedido`)
O motor identifica o campo `tipo` da tabela `pedido` (`entrada` ou `saida`) e utiliza **queries SQL dedicadas** para carregar campos calculados em tempo real:

#### Campos Calculados no Registro Master:
* `{$desconto_unificado}`: Soma de `desconto_servico + desconto_produto`.
* `{$desc_perc_unificado}`: Soma de `desconto_produto_perc + desc_perc_servico`.
* `{$total_geral}`: Soma exata de `total_descritivo + total_servico + total_produto`.
* `{$total_descritivo_soma}`: Soma de `total_descritivo + total_descritivo_desconto`.
* `{$total_geral_calculado}`: Soma de `total_descritivo + total_servico`.
* `{$total_geral_bruto_calculado}`: Soma de `subtotal_produto + subtotal_servico`.

#### Laços de Repetição de Pedidos:
1. **Híbridos (BI / Fallback Físico):**
   * `descritivos` → `bi_pedido_[entrada/saida]_descritivo` *(fallback: `pedido_descritivo`)*
   * `parcelas` → `bi_pedido_[entrada/saida]_financ_titulo` *(fallback: `pedido_financ_titulo`)*
   * `produtos` → `bi_pedido_[entrada/saida]_produto` *(fallback: `pedido_produto`)*
   * `produtos_separar` → `bi_pedido_[entrada/saida]_produto_separar` *(fallback: `pedido_produto`)*
   * `servicos` → `bi_pedido_[entrada/saida]_servico` *(fallback: `pedido_servico`)*
   * `equipamentos` → `bi_pedido_equip` *(fallback: `pedido_equip`)*
   * `perifericos` → `bi_pedido_equip_perifer` *(fallback: `pedido_equip_perifer`)*
   * `pedido_image` → `bi_pedido_image` *(fallback: `pedido_image`)*

2. **Laços com Consultas SQL Diretas:**
   * `participantes`: Consulta os envolvidos na tabela `pedido_pessoa` vinculados com `pessoa`.
   * `enderecos`: Executa joins em `pessoa_endereco`, `local_municipio`, `local_uf` e `local_pais` trazendo os endereços completos da pessoa.
   * `produtos_condicao`: Filtra itens com condição `'TROUXE'` ou `'A TRAZER'`.
   * `produtos_condicao_null`: Filtra itens sem condição definida (`IS NULL`).

---

## 🎨 4. Identidade Visual Global (`crm_site_corpo`)

O motor realiza uma consulta global na tabela `crm_site_corpo` para todos os registros com `ativo = 1`, convertendo imagens para Base64 e injetando as variáveis no cabeçalho master de **qualquer layout**:

| Tipo (`tipo`) | Tag Principal (Imagem / Conteúdo) | Tag Descrição / Nome | Tag Conteúdo Texto |
| :--- | :--- | :--- | :--- |
| **Logo** | `{$site_logo}` ou `{$site_logo_img}` | `{$site_logo_descricao}` | — |
| **Home / Capa** | `{$site_home}` ou `{$site_home_img}` | `{$site_home_descricao}` | `{$site_home_conteudo}` |
| **Favicon** | `{$site_favicon}` | `{$site_favicon_descricao}` | — |
| **Cabeçalhos** | `{$site_cabecalho_01}` .. `_03` | `{$site_cabecalho_01_descricao}` | `{$site_cabecalho_01_conteudo}` |
| **Rodapés** | `{$site_rodape_01}` .. `_03` | `{$site_rodape_01_descricao}` | `{$site_rodape_01_conteudo}` |
| **Banners** | `{$site_banner_01}` .. `_06` | `{$site_banner_01_descricao}` | `{$site_banner_01_conteudo}` |
| **Corpos** | `{$site_corpo_01}` .. `_06` | `{$site_corpo_01_descricao}` | `{$site_corpo_01_conteudo}` |
| **Títulos** | `{$site_titulo_01}` .. `_06` | `{$site_titulo_01_descricao}` | `{$site_titulo_01_conteudo}` |
| **Barras** | `{$site_barra_01}` .. `_06` | `{$site_barra_01_descricao}` | `{$site_barra_01_conteudo}` |
| **Links** | `{$site_link_01}` .. `_06` | `{$site_link_01_descricao}` | `{$site_link_01_url}` |

---

## 🛠️ 5. Enriquecimento Automático de Dados (`enrichFallbackData`)

Quando registros são processados nos laços, o motor garante o preenchimento de campos essenciais que possam estar ausentes nas views:

* **Produtos:**
  * Busca `descricao`, `cod_outro`, `cod_barra`, `preco_venda` e `image` diretamente da tabela `produto`.
  * Se a foto principal for nula, busca a primeira foto da galeria em `produto_image`.
  * Busca a sigla da unidade de medida em `produto_unidade`.
  * Busca a descrição da referência em `produto_referencia`.
* **Serviços:**
  * Busca a descrição em `servico`.
  * Busca a imagem do serviço em `servico_image` (ou no campo `image` de `servico`), preenchendo as tags `{$image}` e `{$servico_imagem}`.
* **Códigos de Barra (CODE128):**
  * Gera automaticamente imagens de código de barras em Base64 para os itens do laço (`{$imagem_codigo_barras}`, `{$imagem_cod_barra}`, `{$imagem_cod_outro}`).
* **Formas de Pagamento:** Recupera descrições da tabela `financ_forma_pgto`.
* **CNAE / Tributação:** Recupera código e descrição da atividade econômica em `tributo_tab_atividade`.
* **Processos PCP:** Recupera descrições da tabela `pcp_processo`.

---

## ⚙️ 6. Recursos e Modificadores Especiais

1. **Ordenação Alfabética Dinâmica (`_alfa` / `_asc`):**
   * Adicionar o sufixo no laço HTML faz o motor ordenar os itens alfabeticamente pela descrição.
   * *Exemplo:* `<!--[produtos_alfa]-->` ou `<!--[servicos_asc]-->`.
2. **Totalizadores Automáticos de Laços:**
   * O motor gera somatórios automáticos no registro master para qualquer laço numérico:
     * `{$produtos_qtde_sum}` e `{$produtos_total_sum}`
     * `{$servicos_qtde_sum}` e `{$servicos_total_sum}`
     * `{$qtde_sum}` e `{$total_sum}` (atalhos diretos).
3. **Exposição de Equipamento Único no Cabeçalho:**
   * Se o pedido possuir equipamentos vinculados, os dados do **primeiro equipamento** são copiados diretamente para o cabeçalho master (acessíveis via `{$equip_descricao}`, `{$equip_numero_serie}`, etc., sem precisar abrir laço).
4. **Casas Decimais Parametrizadas:**
   * Lê as configurações de casas decimais cadastradas na tabela `global_parametro` (ID = 1) para formatar preços e quantidades conforme o módulo (Entrada vs. Saída / NF vs. Pedido).
5. **Prevenção de Imagens Quebradas no Dompdf:**
   * Toda imagem local é convertida para `data:image/...;base64,...`.
   * Recomenda-se incluir no CSS do HTML: `img[src=""], img:not([src]) { display: none !important; }`.

---

## 📋 7. Cheat Sheet de Tags Comuns (Guia Rápido)

### Cabeçalho / Master
* **Unidade Emitente:** `{$pessoa_unidade_descricao}`, `{$pessoa_unidade_cpf_cnpj}`, `{$pessoa_unidade_pj_inscricao_estadual}`, `{$p_unidade_fone1}`, `{$p_unidade_email}`, `{$unidade_image}`, `{$pessoa_unidade_logradouro}`, `{$pessoa_unidade_numero}`, `{$pessoa_unidade_local_municipio_descricao}`, `{$pessoa_unidade_local_uf_sigla}`, `{$pessoa_unidade_cep}`.
* **Cliente / Destinatário:** `{$pessoa_id}`, `{$pessoa_descricao}`, `{$pessoa_cpf_cnpj}`, `{$pessoa_pj_inscricao_estadual}`, `{$pessoa_fone1}`, `{$pessoa_email}`, `{$contato}`, `{$logradouro}`, `{$numero}`, `{$bairro}`, `{$local_municipio_descricao}`, `{$local_uf_descricao}`, `{$cep}`.
* **Vendedor / Responsável:** `{$pessoa_responsavel_descricao}`, `{$p_unidade_fone1}`.
* **Valores e Totais:** `{$subtotal_produto}`, `{$subtotal_servico}`, `{$total_pedido}`, `{$total_geral}`, `{$desconto_unificado}`, `{$desc_perc_unificado}`.
* **Prazos e Datas:** `{$cadastro}`, `{$data_hoje}`, `{$previsao}`, `{$entrega}`, `{$financ_prazo_pgto_descricao}`.
* **Observações:** `{$obs1}`, `{$obs2}`.

### Laço de Produtos (`<!--[produtos]-->`)
* `{$produto_id}`, `{$produto_descricao}`, `{$produto_referencia_descricao}`, `{$sigla}`, `{$qtde}`, `{$preco}`, `{$desconto}`, `{$total_bruto}`, `{$total}`, `{$image}`, `{$imagem_codigo_barras}`.

### Laço de Serviços (`<!--[servicos]-->`)
* `{$servico_id}`, `{$servico_descricao}`, `{$complemento}`, `{$qtde}`, `{$preco}`, `{$total_bruto}`, `{$total}`, `{$image}`.

### Laço de Parcelas (`<!--[parcelas]-->`)
* `{$parcela}`, `{$vencimento}`, `{$valor}`, `{$financ_forma_pgto_descricao}`.

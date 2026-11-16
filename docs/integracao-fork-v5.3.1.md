# Integração do fork com v5.3.1 — revisão e ensaio de migração

Base do fork: `94d3bd3df306c4522670da15d3dd999f58621763` (4.55.4).
Upstream master e tag v5.3.1: `6def1e2311a9dd987e7bfae2db185c2ea65897a8`.
Comparação revalidada em 16/11/2026: 44 commits exclusivos do fork, 138 do upstream.
Esta branch preserva os dois históricos por merge e não autoriza publicação.

## Resolução dos conflitos

- 423 conflitos modify/delete: os blobs do fork eram idênticos ao ancestral;
  somente o modo executável havia mudado. Foram aceitas as remoções da v5,
  incluindo views, bibliotecas e assets legados. Os modos dos arquivos restantes
  foram normalizados conforme upstream.
- `.gitignore`: mantidas as exclusões de fotos de perfil e storage da v5;
  o conteúdo gerado do cache HTMLPurifier também é ignorado.
- `general_helper.php`: preservados o filtro da v5, as duas instâncias de
  HTMLPurifier e a customização de cache fora de vendor. A pasta inicial do
  cache agora acompanha o código; precisa ser gravável pelo PHP na instalação.
- `Mapos_model.php`: removidos métodos legados substituídos pela v5, preservando
  `getEstatisticasFinanceiroMes()` do fork para integrações que ainda o chamem.
- Compose: preservados banco externo e rotação de logs do fork; mantidos PHP 8.5
  e bind localhost do upstream. Removido o bypass de requisitos do Composer.
  O teste de exposição de portas considera os serviços efetivamente publicados.
- Nginx: regras v5 também no template usado pelo Compose; o alias redundante
  de assets foi substituído pelo comportamento do upstream.
- Assets próprios já versionados no fork permanecem no histórico e na branch.
  Não foram copiados dados nem credenciais de uma instalação.

## Situação da revisão

O dashboard oferece pagamento e vencimento; a regra temporal do fork foi
portada para os componentes v5 e testada, conforme detalhado abaixo.
Restam a preparação 4.55.5, restauração e ensaio de migração em cópia isolada.
Não há aprovação para deploy.

## Validação da integração inicial (antes desta correção)

Em containers temporários ARM64 com PHP 8.5.10, limites de CPU/memória e sem
montagens de produção; testes sem rede externa e sem banco de produção:

- `composer install` com lock e requisitos de plataforma, sem bypass.
- `composer test`: 1.599 testes, 7.316 assertions, zero falhas e warnings;
  uma depreciação e um skip reportados pela suíte, baseline upstream mantida.
- `composer analyse`: sem erros; PHP-CS-Fixer nos três arquivos PHP alterados.
- `npm run test:js`: 190 testes aprovados.
- `git diff --check upstream/master`: aprovado para as adaptações do fork.
  A comparação com o fork antigo inclui whitespace preexistente no upstream.
- Nginx temporário, sem rede externa/portas publicadas: `.env`, `.git/HEAD`,
  `banco.sql`, `composer.json`, logs e docs retornam 403; favicon retorna 200.

Esses resultados não comprovam migração de uma instalação real, restauração,
login real, OS/vendas reais, integrações API nem fidelidade visual/PDF após
migração. Para isso, seguir `migracao-4x-para-5.md` em cópia restaurada, preparando
primeiro a 4.55.5, storage externo, configuração e backup completo verificável.
Não usar este Compose diretamente sobre containers ou volumes existentes.

## Dashboard financeiro: duas visões (16/11/2026)

A customização v4 em `Mapos_model::getEstatisticasFinanceiroMes()` agrupa
pelo vencimento e não filtra a baixa. A v5 agrupa pelo pagamento e exige
`baixado = 1`. O painel agora oferece um seletor nativo **Visão financeira**,
junto do ano, com submissão GET e sem novo endpoint ou JavaScript:

| Visão | Data e inclusão |
| --- | --- |
| Pagamento (realizado), padrão v5 | Mês do pagamento, somente baixados |
| Vencimento (pagos e pendentes) | Mês do vencimento, inclusive pendentes e vencidos, com baixa zero ou NULL |

O ano é aplicado à data escolhida. Lançamentos sem essa data não entram.
O card **Saldo do mês**, sua tendência e sparkline continuam por pagamento
do ano atual, independentemente do filtro do balanço. Permissões e ocultação
de valores permanecem nos componentes existentes.

Ambas as visões usam o líquido da listagem v5: `valor_desconto` positivo
prevalece; caso contrário, `valor - COALESCE(desconto, 0)`. Isso preserva
o critério temporal e a inclusão das pendências do fork, mas não reproduz
a antiga assimetria: despesas brutas e desconto percentual recalculado apenas
nas receitas. Descontos legados devem ser reconciliados na cópia migrada.

Não existe status financeiro “cancelado” em `lancamentos`. O status
Cancelado de OS/venda não elimina, sozinho, o lançamento existente.
A exclusão de fatura remove o lançamento de ambas as visões conforme o
modelo atual; desfazer baixa mantém o vencimento e retira do realizado.
Não foi adicionada uma nova regra de cancelamento.

Testes comportamentais com SQLite isolado cobrem essas regras, descontos,
virada de mês/ano, ano bissexto, meses/anos vazios, seleção inválida,
estados vazios, permissões e renderização do KPI independente do filtro.
Nenhuma migration, alteração de schema ou configuração é necessária.

Verificação desta correção: PHPUnit **1.606 testes / 7.361 assertions**,
sem falhas (uma depreciação e um skip preexistentes); **190 testes JS**;
`npm ci && npm run build`, PHP-CS-Fixer nos seis arquivos PHP alterados.
Os assets gerados não mudaram: as classes utilizadas já estavam no build.
PHPStan sem erros; verificações de escape e scripts inline incluídas na suíte.
O CI do PR valida novamente o commit final.


## Pré-requisitos da próxima fase (somente documentação)

As evidências da auditoria de 16/11/2026 registraram publicação **4.55.3**
(SHA `fbfa5dc`), diferente da master do fork; PHP CLI/FPM 8.5.10 em ARM64,
banco externo MySQL 9.4 com 28 tabelas e migration `20240503170400`.
Não foram revalidadas nesta missão. A segurança Nginx/Cloudflare foi concluída
em missão separada e não foi repetida.

Antes de qualquer publicação, ainda é necessário:

1. Preparar e validar a passagem para **4.55.5** em cópia isolada, conforme
   [guia upstream](migracao-4x-para-5.md), incluindo customizações locais.
2. Provar restauração independente. A auditoria registrou snapshot remoto,
   inventário e `restic check` aprovados, mas não restauração completa.
   Verificar código/dependências, configuração secreta, dump consistente e
   uploads/anexos com checksums e recuperação funcional.
3. Preparar laboratório ARM64 sem volumes compartilhados com produção:
   cópias restauradas, portas locais/restritas, rede e recursos limitados;
   desativar cron, SMTP, gateways, webhooks e demais efeitos externos.
   Confirmar novamente PHP >= 8.5 e extensões do guia na imagem escolhida.
4. Configurar e testar `STORAGE_PATH` fora da raiz web, permissões e backup
   independente. Na auditoria essa variável não estava configurada.
5. Ensaiar migrations **somente na cópia**, reconciliar saldos nas duas
   visões e testar login/permissões, OS, vendas, anexos e PDF. A API estava
   habilitada: verificar clientes, autenticação e renovação de tokens v5.
6. Preparar releases versionadas e rollback conjunto de código, banco e
   arquivos; medir o ensaio para estimar downtime. Nenhum tempo de parada
   ou rollback funcional está comprovado até a restauração e o ensaio.

A correção do dashboard remove a incompatibilidade temporal conhecida.
O PR permanece em rascunho: não representa aprovação de migração ou deploy.

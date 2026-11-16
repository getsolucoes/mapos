# Integração do fork com v5.3.1 — revisão necessária

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

## Pendência funcional que impede aprovação para deploy

O fork calculava o gráfico mensal por **vencimento**, incluindo lançamentos
pendentes. A v5 usa `Financeiro_model::balancoAnual()` por **pagamento**, somente
baixados, e aplica o cálculo líquido novo. O método antigo foi preservado, mas
o painel v5 não o chama. Portanto, esta integração **não mantém a equivalência
do gráfico mensal**. Antes de aprovar, definir e testar o comportamento desejado
na tela v5. Não basta conservar código sem chamador para considerar a regra portada.

## Validação

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

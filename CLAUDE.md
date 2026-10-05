# App de gastos do Marcelo

App pessoal (PWA) que mostra e organiza os gastos do Cartão XP. Converse em português do Brasil, de forma simples (o usuário não é programador). Este repositório é PÚBLICO: nunca commitar senhas, tokens, a chave secreta do Supabase ou o e-mail do usuário.

## Arquivos privados

Ficam FORA deste repo, em `C:\Users\marce\Documents\gastos-privado\` (têm token e e-mail; nunca trazer para cá): `script-da-planilha.gs` (Apps Script atual) e `supabase-setup.sql`.

## Arquitetura

- **Site** (este repo, GitHub Pages: https://mrcelomn.github.io/gastos/): `index.html` único (CSS e JS inline), `manifest.webmanifest`, ícones `icon-180/192/512.png`. Usa supabase-js v2 via jsdelivr. Cor principal #B5215E, logo de barras em SVG.
- **Supabase** (projeto `nkcypuoosdsdbokokdhh`, região São Paulo). A chave publishable fica no `index.html` (é pública por design; os dados são protegidos por login + RLS).
  - Tabelas: `gastos` (data, motivo, valor, coluna 'Crédito'|'Débito', categoria, estabelecimento, origem 'app'|'sms'|'importado'), `regras` (contem, motivo, categoria, coluna, ordem: menor vale primeiro), `periodos` (nome, inicio, fatura, pensao; o atual é o de início mais recente; fatura/pensao alimentam a aba Fatura do app: PIX = fatura − pensão − débito, pensão padrão 625), `sms_log`, `config` (sem políticas; guarda `email_dono`, `token_sms`, `planilha_url`).
  - RLS: tudo liberado só quando `eh_dono()` (e-mail do JWT = `config.email_dono`).
  - RPCs: `registrar_sms(texto, token)` (chamado pelo atalho do iPhone), `aplicar_regras_pendentes()`, `espelho_dados(token)`, `espelho_informar_planilha(token, url)`, `planilha_url()`.
  - O SQL completo (com token e e-mail) está em `gastos-privado\supabase-setup.sql`.
- **Atalho do iPhone**: automação de Mensagem (qualquer remetente, contém "final 2393" (cartão adicional do Marcelo), executar imediatamente; o banco só lança se o texto tiver "Compra Aprovada de R$", o resto vira "Não reconhecido" no sms_log). Depois do POST, mostra uma notificação com a resposta do servidor. Faz POST para `https://nkcypuoosdsdbokokdhh.supabase.co/rest/v1/rpc/registrar_sms` com cabeçalho `apikey` e corpo JSON `{texto: Entrada do Atalho, token: <token_sms>}`.
- **Espelho na planilha Google**: Apps Script vinculado à planilha "GASTOS - SETEMBRO" (versão atual: `gastos-privado\script-da-planilha.gs`). A cada 10 min (ou menu Gastos > Sincronizar agora, só existe na planilha de setembro) reescreve a aba Diário (tabela do Sheets: Data | Motivo | Crédito | Débito | Categoria) de CADA mês, mantendo a formatação, e escreve fatura/pensão na aba Totais (célula abaixo de FATURA e de PENSÃO). Mapa mês→planilha na propriedade `PLANILHAS`, conferido a cada execução pelo NOME da planilha = nome do período (se não bater, acha a certa pelo nome; por isso não renomear planilhas). Só roda na planilha principal (constante `PLANILHA_PRINCIPAL` = GASTOS - SETEMBRO); as cópias dos meses levam o script, mas nele nada roda. Mês novo no app → copia a planilha mais recente, limpa Diário e FATURA. Sentido único: app → planilha.

## Convenções do usuário

- Motivos: Restaurante (restaurante físico), Ifood (99Food/iFood), Uber (99 Ride, 99*, UberRides), Mercado, Papelaria, Chat GPT, Claude, Amazon, Farmácia, Academia, Lavanderia.
- Categorias: Refeições, Locomoção, Mercado, Assinatura, Acadêmico, Compras, Saúde.
- Coluna Débito = pago pelo Marcelo (não é tipo de cartão).
- Mesmo valor + mesmo lugar no mesmo dia = lança uma vez. Dias diferentes = lança todos.
- Lugar desconhecido entra com categoria "?" e o app pede para classificar (e cria regra).

- Virada de mês: começa no dia seguinte ao fechamento da fatura (ex.: fatura até 29/09 → mês novo em 30/09).
- Na fatura (CSV do XP), só os gastos de portador MARCELO NEVES entram no app; os de RONALDO NEVES ficam de fora.

## Estado em 05/10/2026

Feito: Supabase configurado, GitHub Pages no ar, app no iPhone, planilha ligada, abas antigas apagadas, aba Fatura no app, setembro conferido com a fatura (67 gastos, R$ 2.624,83), GASTOS - OUTUBRO criado (início 30/09).
Planilhas de setembro e outubro conferidas e certas; script com `PLANILHA_PRINCIPAL` colado. Atalho testado manualmente (lançou). Pendente: confirmar na próxima compra real que o SMS dispara sozinho. Se não lançar e não houver linha no sms_log, o pedido foi recusado antes de registrar (token, apikey ou nomes dos campos) ou a automação não disparou.

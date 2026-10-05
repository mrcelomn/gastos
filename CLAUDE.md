# App de gastos do Marcelo

App pessoal (PWA) que mostra e organiza os gastos do Cartão XP. Converse em português do Brasil, de forma simples (o usuário não é programador). Este repositório é PÚBLICO: nunca commitar senhas, tokens, a chave secreta do Supabase ou o e-mail do usuário.

## Arquitetura

- **Site** (este repo, GitHub Pages: https://mrcelomn.github.io/gastos/): `index.html` único (CSS e JS inline), `manifest.webmanifest`, ícones `icon-180/192/512.png`. Usa supabase-js v2 via jsdelivr. Cor principal #B5215E, logo de barras em SVG.
- **Supabase** (projeto `nkcypuoosdsdbokokdhh`, região São Paulo). A chave publishable fica no `index.html` (é pública por design; os dados são protegidos por login + RLS).
  - Tabelas: `gastos` (data, motivo, valor, coluna 'Crédito'|'Débito', categoria, estabelecimento, origem 'app'|'sms'|'importado'), `regras` (contem, motivo, categoria, coluna, ordem: menor vale primeiro), `periodos` (nome, inicio; o atual é o de início mais recente), `sms_log`, `config` (sem políticas; guarda `email_dono`, `token_sms`, `planilha_url`).
  - RLS: tudo liberado só quando `eh_dono()` (e-mail do JWT = `config.email_dono`).
  - RPCs: `registrar_sms(texto, token)` (chamado pelo atalho do iPhone), `aplicar_regras_pendentes()`, `espelho_dados(token)`, `espelho_informar_planilha(token, url)`, `planilha_url()`.
  - O SQL completo (com token e e-mail) está no arquivo `1-supabase-setup.sql` que o usuário baixou. Não está neste repo de propósito.
- **Atalho do iPhone**: automação de Mensagem (remetente "Cartão XP" = 29190, contém "Compra Aprovada", executar imediatamente). Faz POST para `https://nkcypuoosdsdbokokdhh.supabase.co/rest/v1/rpc/registrar_sms` com cabeçalho `apikey` e corpo JSON `{texto: Entrada do Atalho, token: <token_sms>}`.
- **Espelho na planilha Google** "GASTOS - SETEMBRO": Apps Script vinculado (arquivo `3-planilha-Código.gs` que o usuário baixou) que a cada 10 min reescreve a aba Diário (tabela do Sheets: Data | Motivo | Crédito | Débito | Categoria) com os gastos do período atual, mantendo a formatação. Quando surge um período novo no app, copia a planilha, limpa e passa a escrever na cópia.

## Convenções do usuário

- Motivos: Restaurante (restaurante físico), Ifood (99Food/iFood), Uber (99 Ride, 99*, UberRides), Mercado, Papelaria, Chat GPT, Claude, Amazon, Farmácia, Academia, Lavanderia.
- Categorias: Refeições, Locomoção, Mercado, Assinatura, Acadêmico, Compras, Saúde.
- Coluna Débito = pago pelo Marcelo (não é tipo de cartão).
- Mesmo valor + mesmo lugar no mesmo dia = lança uma vez. Dias diferentes = lança todos.
- Lugar desconhecido entra com categoria "?" e o app pede para classificar (e cria regra).

## Estado em 05/10/2026

Feito: banco escrito e testado (Postgres local), app testado com mock, arquivos do site neste repo.
Pendente (do lado do usuário):
1. Supabase: rodar `1-supabase-setup.sql` no SQL Editor; criar o usuário em Authentication > Users (Auto Confirm) e desligar "Allow new users to sign up".
2. GitHub: Settings > Pages > Deploy from a branch > main / (root).
3. iPhone: abrir o site no Safari, entrar, Adicionar à Tela de Início.
4. Atalho: trocar URL, adicionar cabeçalho apikey, trocar token.
5. Apps Script da planilha: apagar Tela.gs e Index.html, colar `3-planilha-Código.gs` no Código.gs, rodar `configurar`, arquivar a implantação antiga do app web.
6. Depois: conferir se o Diário bate com os 78 gastos importados; apagar as abas antigas Regras, Log e App (pedir confirmação).

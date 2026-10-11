# Conciliador de Recebíveis · 361°

Concilia os pedidos da Shopify com as cobranças e o extrato de liquidação do Pagar.me: agenda de recebíveis parcela a parcela, previsto x realizado, MDR cobrado x tabela, estornos e divergências.

- **Site:** GitHub Pages (este repositório).
- **Login e controle de acesso:** Google Apps Script + uma planilha Google "Acessos".
- **Dados das bases:** ficam só no navegador de quem usa. Nada é enviado para o GitHub nem para o Google.

---

## Como funciona o acesso

```
Pessoa pede acesso no site ──► e-mail para o administrador (maxximarj@gmail.com)
                                    │ botão "Analisar solicitação"
                                    ▼
                 admin.html: cria usuário e senha ──► e-mail com o acesso para a pessoa
                                    │
Pessoa entra com usuário e senha ──► o conciliador abre
```

- O conciliador está **criptografado** neste repositório (`app.enc`). Só abre depois do login, quando o Apps Script entrega a chave. Quem baixar o repositório não consegue usar a ferramenta nem ver a tabela de taxas.
- A cada abertura, o sistema confere se o usuário continua ativo. Suspenso = não abre mais.
- Senhas são guardadas com hash e salt, nunca em texto.
- O usuário pode trocar a própria senha no menu com o nome dele, no topo do conciliador.

### Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | Tela de login e de pedido de acesso. Depois do login, abre o conciliador |
| `admin.html` | Gestão de acessos: solicitações, usuários, novo usuário |
| `app.enc` | O conciliador, criptografado |
| `config.js` | Endereço do Apps Script (o único arquivo que você edita) |
| `.nojekyll` | Faz o GitHub Pages publicar os arquivos como estão |

O código do Apps Script (`Acessos.gs`) **não fica no repositório**, porque contém a chave que abre o conciliador.

---

## Migração para o GitHub (passo a passo)

Faça na ordem. Leva uns 20 minutos.

### 1. Planilha e Apps Script (controle de acesso)

> Use a conta Google **maxximarj@gmail.com**: os e-mails saem dela, e é nela que você recebe os pedidos.

1. Crie uma planilha Google chamada **Acessos · Conciliador 361**.
2. Na planilha: **Extensões → Apps Script**.
3. Apague o conteúdo de `Código.gs` e cole **todo** o conteúdo do arquivo `Acessos.gs`. Salve (Ctrl+S).
4. Volte para a planilha e recarregue a página. Aparece o menu **🔐 Acessos**.
5. **🔐 Acessos → Configurar / criar administrador.** Na primeira vez, o Google pede autorização: *Revisar permissões → escolha a conta → Avançado → Acessar (não seguro) → Permitir*. É o seu próprio script pedindo para usar a planilha e enviar e-mail.
6. Informe o seu usuário de administrador (ex.: `tadeu`) e a senha. São criadas as abas **Usuários**, **Solicitações** e **Log**.
7. (Opcional) **🔐 Acessos → Testar envio de e-mail.**

### 2. Publicar o Apps Script como App da Web

1. No Apps Script: **Implantar → Nova implantação**.
2. Engrenagem ⚙ → **App da Web**.
3. **Executar como:** Eu. **Quem pode acessar:** Qualquer pessoa.
4. **Implantar** → copie a **URL do app da Web** (termina em `/exec`).

> "Qualquer pessoa" é necessário para o site conseguir falar com o script. Mesmo assim, ninguém entra sem usuário e senha, e a planilha continua privada.

### 3. Repositório no GitHub

1. Em github.com: **New repository**, nome `conciliador-361`, **Public**, Create.
2. **Add file → Upload files** e arraste: `index.html`, `admin.html`, `app.enc`, `config.js`, `.nojekyll`, `README.md`.
   - O `.nojekyll` é um arquivo oculto. Se o seu computador não mostrar, crie pelo GitHub: **Add file → Create new file**, nome `.nojekyll`, conteúdo vazio, Commit.
3. Abra `config.js` no GitHub, clique no lápis ✏️, troque `COLE_AQUI_A_URL_DO_APPS_SCRIPT` pela URL do passo 2 e faça o **Commit changes**.

### 4. Ligar o GitHub Pages

1. No repositório: **Settings → Pages**.
2. **Source:** Deploy from a branch. **Branch:** `main`, pasta `/ (root)` → **Save**.
3. Em 1 ou 2 minutos aparece o endereço: `https://SEU-USUARIO.github.io/conciliador-361/`.

### 5. Avisar o Apps Script do endereço do site

1. No Apps Script, na linha `SITE_URL:`, coloque o endereço do passo 4 (com `/` no final).
2. Salve e **Implantar → Gerenciar implantações → ✏️ (editar) → Versão: Nova versão → Implantar**.
   Assim a URL `/exec` continua a mesma. Se criar uma *nova implantação*, a URL muda e é preciso atualizar o `config.js`.

### 6. Testar

1. Abra o site e entre com o administrador.
2. Clique no seu nome no topo → **Gerenciar acessos**.
3. Numa janela anônima, peça acesso com outro e-mail e veja o pedido chegar no seu Gmail.

### 7. Desligar a versão antiga

- **Netlify:** Site configuration → Delete this site.
- **A base salva no navegador** (botão Salvar dados) fica guardada por endereço de site. No endereço novo, suba as bases uma vez e clique em **Salvar dados**.

---

## Rotina do administrador

| Tarefa | Onde |
|---|---|
| Aprovar ou negar um pedido | Link do e-mail → `admin.html` → **Aprovar e enviar acesso** ou **Negar** |
| Criar acesso sem pedido | `admin.html` → **Novo usuário** |
| Resetar senha | `admin.html` → **Usuários** → **Nova senha** (vai por e-mail) |
| Bloquear alguém | `admin.html` → **Usuários** → **Suspender** |
| Ver quem entrou e quando | Planilha → abas **Usuários** (último acesso) e **Log** |

- **Limite do Gmail:** cerca de 100 e-mails por dia em conta gratuita. O painel mostra quantos ainda restam.
- **Tempo de sessão:** 12 horas (`SESSAO_HORAS` no Apps Script).
- **Derrubar todas as sessões:** no Apps Script, *Configurações do projeto → Propriedades do script* → apague `SEGREDO` e rode **🔐 Acessos → Configurar** de novo.

---

## Atualizar o conciliador

Quando houver uma versão nova, só o arquivo `app.enc` muda:

1. No repositório: **Add file → Upload files** → arraste o novo `app.enc` → **Commit changes**.
2. Em 1 ou 2 minutos, todo mundo já abre a versão nova.

A chave de criptografia está no `Acessos.gs` (`CHAVE_APP`). Quem gerar um `app.enc` novo precisa usar a mesma chave. Se a chave mudar, troque nos dois lugares ao mesmo tempo.

---

## Regras da conciliação (resumo)

- **Pedido → cobrança:** `Payment References` da Shopify = `Code` do Pagar.me. Vale a cobrança paga; se não houver, a mais recente.
- **Cobrança → extrato:** ID da transação + nº da parcela (Pix "-" = parcela 1).
- **Data prevista:** regra do Pagar.me, de 30 em 30 dias, em dia útil (Pix: mesmo dia).
- **Estorno parcial:** distribuído entre as parcelas futuras, como no Pagar.me ("Estornos a lançar").
- **MDR:** conferido por venda quitada, contra a tabela contratada por bandeira e número de parcelas.
- **Divergência:** lançamentos do extrato sem parcela na agenda, com o motivo e o número do pedido.

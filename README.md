# Landing pública de Gate 1 — G001-04 · GBP Dentistas

**URL pública:** <https://elucidata-ventures.github.io/gbp-dentistas/>

Hospedagem: GitHub Pages (org `elucidata-ventures`, repo público
`gbp-dentistas`, branch `main`, raiz do repo). Custo **R$ 0**.
Este diretório (`ventures/G001-04/landing/`) **é** o repositório: `git push`
daqui publica.

| URL | Arquivo | O que é |
|---|---|---|
| `/gbp-dentistas/` | `index.html` | Landing da oferta (R$ 247/mês, sem fidelidade) |
| `/gbp-dentistas/auditoria/` | `auditoria/index.html` | Explica que auditorias são páginas privadas por clínica + lista os 3 exemplos |
| `/gbp-dentistas/auditoria/exemplo-1.html` (…2, 3) | `auditoria/exemplo-*.html` | Auditorias de demonstração, anonimizadas (“Clínica Exemplo”, vizinhos numerados), `noindex` |
| `/gbp-dentistas/proposta/` | `proposta/index.html` | Página da proposta (cópia de `../assets/proposta.html`) |

Zero dependência externa: HTML + CSS inline, sem CDN, sem fonte remota, sem JS
de terceiros. Abre offline. `lang="pt-BR"`, meta description e Open Graph em
`index.html`.

## 1. Como a captura funciona (RESOLVIDO — não há mais placeholder)

O CTA **não é mais** um `mailto:` com placeholder. Desde 07/08/2026 as três
superfícies (`index.html`, `proposta/index.html` e cada auditoria gerada)
carregam o mesmo **widget de intenção**, cuja fonte canônica única é
`../assets/widget_intencao.html`. Não dependemos de P-011 (Instagram) nem de
P-018 (alias de e-mail): o prospect digita o canal DELE em "onde te respondo".

Cada clique manda o lead para **dois destinos**, de propósito:

| # | Destino | Papel | Retenção |
|---|---|---|---|
| 1 | `https://captura.elucidata.vc/v1/g001-04` (POST urlencoded) | **Fonte de verdade do gate.** É a resposta DELE que libera o "Recebido" na tela. | indefinida |
| 2 | `https://ntfy.sh/elucidata-gbp-dentistas-k7m3q9x2` (POST JSON) | Sinal imediato. Fire-and-forget: falhar aqui é silencioso e não afeta o usuário. | **~12 h** |
| 3 | `https://captura.elucidata.vc/f/g001-04` (link visível + `<noscript>`) | Escape sem JS / com bloqueador. Mesmos campos, mesmo destino. | indefinida |

Ler os leads (rodar de `ventures/G001-04/`):

```bash
python3 ../../ops/captura.py --novos    # só o que chegou desde a última leitura
python3 ../../ops/captura.py --csv      # planilha
python3 ../../ops/captura.py --apagar N # exclusão a pedido do titular (LGPD)
```

Campos do formulário hospedado: `../assets/form_captura.json` (republicar com
`python3 ../../ops/captura.py --publicar assets/form_captura.json`). Os rótulos
de `interesse` no widget são **idênticos** às opções do formulário hospedado,
para os dois caminhos caírem na mesma coluna:

- `pedido_proposta` → "Quero a proposta completa"
- `aceite_preco` → "Aceito os R$ 247/mês e quero começar"

**Regra anti-autoengano (não remover):** `aceite_preco` só é enviado se a função
`precoVisivel()` achar "R$ 247/mês" escrito no DOM da mesma tela. Aceite sem
preço na tela não conta para o gate.

**Nunca** escreva o token do endpoint em HTML, commit ou diário — o POST de lead
é público por natureza e não usa token.

> Nota: `../assets/proposta.html` continua sendo um template antigo com
> `{{CTA_HREF}}`/`EMAIL_CONTATO` e **sem** widget. Ele não é publicado; a página
> viva é `proposta/index.html`. Não use aquele arquivo como fonte.

## 2. Como publicar uma auditoria personalizada nova

O gerador vive em `../assets/gerar_auditoria.py` (só stdlib). Ele escreve
`<slug>.html` + `<slug>__email.txt`; o HTML vai para `auditoria/<slug>.html`,
que já sai com `noindex,nofollow` e **não é linkado em lugar nenhum** — o link
só existe no e-mail do prospect.

Comando de uma linha (a partir de `ventures/G001-04/landing/`), onde
`../data/prospects.csv` pode ser um CSV com **um único prospect**:

```bash
python3 ../assets/gerar_auditoria.py ../data/prospects.csv ../data/concorrentes.csv --out /tmp/aud && python3 -c "import glob,io,os,shutil;[ (lambda s,d: io.open(d,'w',encoding='utf-8').write(io.open(s,encoding='utf-8').read().replace('{{DOMINIO}}','elucidata-ventures.github.io/gbp-dentistas')))(f,'auditoria/'+os.path.basename(f)) for f in glob.glob('/tmp/aud/*.html')]" && git add auditoria && git commit -m "Auditorias novas" && git push
```

O que a linha faz: gera em `/tmp/aud`, copia os HTMLs para `auditoria/`
substituindo o placeholder `{{DOMINIO}}` pela URL real do Pages, commita e
publica. Os `__email.txt` ficam em `/tmp/aud` (não vão para o repo público).

Link para colar no e-mail (`{{LINK_AUDITORIA}}`):
`https://elucidata-ventures.github.io/gbp-dentistas/auditoria/<slug>.html`
— o `<slug>` é `nome-do-consultorio-cidade` normalizado, e o gerador imprime
cada slug na saída.

O Pages leva ~1–3 min para refletir o push. Verificar:
`curl -sI https://elucidata-ventures.github.io/gbp-dentistas/auditoria/<slug>.html`
até dar `HTTP/2 200`.

## 3. Regras que a página respeita (não quebrar)

- **Nada de dado inventado**: sem depoimento, sem logo de cliente, sem
  “mais de X clínicas atendidas”, sem promessa de resultado clínico.
- **Nenhuma lista de prospects publicada.** `auditoria/index.html` não lista
  clínica nenhuma; os exemplos são fictícios e estão marcados como
  demonstração dentro da própria página.
- **Sem e-mail/@ /telefone inventado** enquanto P-011/P-018 estiverem pendentes.
- **Custo R$ 0**: sem domínio próprio, sem conta nova, sem serviço pago.
  Domínio próprio (P-002) é opcional depois — quando existir, é só apontar
  o CNAME; a página não depende dele.

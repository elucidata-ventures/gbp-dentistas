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

## 1. Como ligar o CTA (1 linha por página)

Hoje o CTA está com placeholder porque o fundo ainda não tem e-mail nem
Instagram da família GBP — pendências **P-011** (conta Instagram) e **P-018**
(alias `familia-gbp@…` no Workspace). Nada de e-mail ou @ inventado na página.

Quando a identidade existir, edite **estas linhas** (as ocorrências dentro do
comentário `CTA_CANAL` no topo de cada arquivo são só documentação — pode
deixar como estão):

| Arquivo | Linha | Conteúdo hoje | Trocar por (exemplo com e-mail) |
|---|---|---|---|
| `index.html` | **301** | `<a class="botao" href="{{CTA_HREF}}">Quero a auditoria do meu perfil</a>` | `href="mailto:familia-gbp@elucidata.com.br?subject=Quero%20a%20auditoria%20do%20meu%20perfil"` |
| `index.html` | **302** | `<p class="fallback">{{CTA_FALLBACK}}</p>` | `<p class="fallback">Ou escreva para familia-gbp@elucidata.com.br com o nome e a cidade do consultório.</p>` |
| `proposta/index.html` | **154** | `<a class="botao" href="{{CTA_HREF}}">Aceitar a proposta — R$ 247/mês</a>` | `href="mailto:familia-gbp@elucidata.com.br?subject=Aceito%20a%20proposta%20de%20R%24247%2Fm%C3%AAs"` |
| `proposta/index.html` | **155** | `<p class="alt">{{CTA_FALLBACK}}</p>` | `<p class="alt">Ou responda o e-mail da sua auditoria com “aceito, vamos começar”.</p>` |

Se o canal for Instagram: `href="https://instagram.com/<handle>"` e fallback
“Ou mande uma mensagem no direct do @&lt;handle&gt;.”.

Comando de 1 linha para o caso e-mail (troca as 4 linhas de uma vez):

```bash
python3 - <<'PY'
import io,glob
EMAIL="familia-gbp@elucidata.com.br"   # <<< preencher
for f in ["index.html","proposta/index.html"]:
    t=io.open(f,encoding="utf-8").read()
    t=t.replace('href="{{CTA_HREF}}"',
        'href="mailto:%s?subject=Quero%%20a%%20auditoria%%20do%%20meu%%20perfil"'%EMAIL)
    t=t.replace("{{CTA_FALLBACK}}",
        "Ou escreva para %s com o nome e a cidade do consultório."%EMAIL)
    io.open(f,"w",encoding="utf-8").write(t)
PY
git commit -am "Liga o CTA no canal da familia GBP" && git push
```

Depois disso, repita a mesma edição em `../assets/proposta.html` (fonte da
página de proposta) para as duas cópias não divergirem.

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

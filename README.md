# LP Stella Moraes

Landing page da **Stella Moraes — Psicóloga Clínica (CRP-DF 01/25023)**, Brasília/DF.
Objetivo único da página: **levar o visitante a abrir uma conversa no WhatsApp** para agendar consulta.

| | |
|---|---|
| **Stack** | HTML + CSS + JavaScript puros (sem framework, sem build, sem `npm`) |
| **Servidor** | Nginx (imagem `nginx:alpine`) dentro de um container Docker |
| **Hospedagem** | EasyPanel, que faz o build do `Dockerfile` direto deste repositório |
| **Branch de produção** | `main` |

---

## Sumário

1. [Como publicar uma alteração](#1-como-publicar-uma-alteração)
2. [Rodar localmente](#2-rodar-localmente)
3. [Estrutura do projeto](#3-estrutura-do-projeto)
4. [Mapa da página (onde está cada seção)](#4-mapa-da-página)
5. [Alterações mais comuns — passo a passo](#5-alterações-mais-comuns)
6. [Comportamentos em JavaScript](#6-comportamentos-em-javascript)
7. [Infraestrutura: Docker, Nginx e cache](#7-infraestrutura-docker-nginx-e-cache)
8. [Checklist antes de subir](#8-checklist-antes-de-subir)
9. [Débitos técnicos conhecidos](#9-débitos-técnicos-conhecidos)

---

## 1. Como publicar uma alteração

Não existe etapa de build. **Publicar = fazer push na `main`.**

```bash
git add .
git commit -m "descreva a alteração"
git push origin main
```

O EasyPanel lê este repositório, executa o `Dockerfile` e sobe um novo container.
Se o *auto-deploy* estiver ligado no serviço do EasyPanel, o site atualiza sozinho após o push;
caso contrário, clique em **Deploy** no painel do serviço.

> ⚠️ **Mudou `style.css`, `script.js` ou alguma imagem?** Leia a seção [Cache](#cache--atenção) — sem isso,
> visitantes recorrentes podem continuar vendo a versão antiga por até 30 dias.

---

## 2. Rodar localmente

Escolha **uma** das opções:

**a) VS Code + Live Server (mais simples)**
Abra a pasta no VS Code, clique com o botão direito em `index.html` → *Open with Live Server*.
A porta já está configurada em [.vscode/settings.json](.vscode/settings.json): **http://localhost:5501**.

**b) Python (sem instalar nada)**
```bash
python3 -m http.server 5501
# abrir http://localhost:5501
```

**c) Docker (idêntico à produção — use para validar antes de subir)**
```bash
docker build -t lp-stella .
docker run --rm -p 8080:80 lp-stella
# abrir http://localhost:8080
```

> Não abra o `index.html` com duplo clique (`file://`). Funciona parcialmente, mas não reproduz o comportamento do servidor.

---

## 3. Estrutura do projeto

```
.
├── index.html          # Todo o conteúdo e estrutura da página (única página do site)
├── style.css           # Todo o visual: cores, tipografia, layout, responsivo, animações
├── script.js           # Menu mobile, animações de entrada, header ao rolar, carrossel de avaliações
├── imagens/
│   ├── logo_stella_lp.png     # Logo (header, footer e favicon) — 1920×1080
│   ├── stella_inicio.jpeg     # Foto do topo (círculo) e do card "Profissional" — ~1158×1207
│   └── stella_meio_lp.jpeg    # Foto da seção "Onde e como atendo" — ~1170×1541 (retrato)
├── Dockerfile          # Imagem de produção (nginx:alpine + arquivos do site)
├── nginx.conf          # Configuração do servidor: cache, gzip, headers de segurança
├── .dockerignore       # O que NÃO entra na imagem Docker
└── .vscode/            # Porta do Live Server (só para desenvolvimento)
```

**Regra de ouro:** texto/conteúdo → `index.html` · aparência → `style.css` · comportamento → `script.js`.

---

## 4. Mapa da página

A página é uma única rolagem. Para achar um bloco, busque (`Ctrl/Cmd + F`) pelo **comentário** ou pelo **`id`** indicado.

| # | Seção na tela | Buscar em `index.html` | `id` (âncora) | Bloco em `style.css` |
|---|---|---|---|---|
| — | Cabeçalho + menu | `<!-- Barra superior -->` | `#topo` | `HEADER PREMIUM`, `ESTADO AO ROLAR`, `RESPONSIVO HEADER` |
| 1 | Topo (título + foto) | `<!-- HERO -->` | — | `HERO` |
| 2 | Por que escolher meu atendimento | `<!-- BENEFÍCIOS -->` | `#beneficios` | `GRIDS` (`.feature-card`) |
| 3 | Como funciona (3 passos) | `<!-- COMO FUNCIONA -->` | `#como-funciona` | `GRIDS` (`.step-card`) |
| 4 | Especialidades / transtornos | `<!-- ESPECIALIDADES -->` | `#especialidades` | `GRIDS` (`.specialty-card`) |
| 5 | Onde e como atendo | `<!-- SOBRE A CLÍNICA -->` | `#sobre` | `ABOUT` |
| 6 | Sobre a profissional | `<!-- SOBRE A PROFISSIONAL -->` | — | `ABOUT` (`.professional-*`) |
| 7 | Diferenciais | `<!-- DIFERENCIAIS -->` | — | `GRIDS` (`.premium-card`) |
| 8 | Avaliações do Google (carrossel) | `<!-- PROVAS SOCIAIS` | `#depoimentos` | `GOOGLE REVIEWS` |
| 9 | Perguntas frequentes | `<!-- FAQ -->` | `#faq` | `FAQ` |
| 10 | Chamada final | `<!-- CTA FINAL -->` | — | `CTA FINAL` |
| — | Rodapé | `<!-- Footer -->` | — | `FOOTER` |
| — | Botão flutuante "Agendar" | `<!-- Botão flutuante -->` | — | `FLOATING CTA` |

Os blocos do `style.css` são separados por cabeçalhos assim — basta buscar pelo nome:

```css
/* =========================
   HERO
========================= */
```

---

## 5. Alterações mais comuns

### 5.1 Número ou mensagem do WhatsApp

O link aparece em **7 lugares** (header, menu mobile, topo, "Como funciona", chamada final, rodapé e botão flutuante).
Altere **todos de uma vez** com busca e substituição global no editor (`Ctrl/Cmd + Shift + H` no VS Code):

```
Buscar:      wa.me/5561982552606
Substituir:  wa.me/55DDDNUMERO
```

Formato do link: `https://wa.me/<código país><DDD><número>?text=<mensagem codificada para URL>`.
Para trocar a mensagem pré-preenchida, codifique o texto novo em
[urlencoder.org](https://www.urlencoder.org/) e substitua o trecho após `?text=` nos 7 links.

Confira que nenhum ficou para trás:
```bash
grep -c "wa.me/" index.html   # deve continuar retornando 7
```

### 5.2 Textos

Todo texto visível está em `index.html`, dentro da seção correspondente (ver [mapa](#4-mapa-da-página)).
Não há textos gerados por JavaScript.

### 5.3 Fotos

1. Coloque a nova imagem em `imagens/`.
2. **Recomendado:** use um **nome de arquivo novo** (ex.: `stella_inicio_v2.jpg`) e atualize o `src` no `index.html`.
   Isso evita o problema de [cache](#cache--atenção).
3. Formato: JPG ou WebP, até ~300 KB. Fotos grandes deixam a página lenta no celular.

| Onde aparece | Arquivo atual | Busque no `index.html` | Formato ideal |
|---|---|---|---|
| Círculo no topo da página | `stella_inicio.jpeg` | `class="portrait-photo"` | quadrado, rosto centralizado |
| Card "Profissional" | `stella_inicio.jpeg` | `class="pro-avatar-photo"` | quadrado |
| Seção "Onde e como atendo" | `stella_meio_lp.jpeg` | `class="panel-photo"` | retrato (vertical) |
| Logo (header, rodapé, ícone da aba) | `logo_stella_lp.png` | `logo_stella_lp.png` (5 ocorrências) | PNG com fundo transparente |

Sempre mantenha o atributo `alt` descritivo (acessibilidade e SEO).

### 5.4 Cores e identidade visual

A paleta fica nas variáveis do topo do `style.css`, no bloco `:root` (seção `RESET E BASE`).
A cor da marca é o vinho **`#640d13`**.

```css
:root {
  --primary: #640d13;   /* cor principal da marca */
  --text:    #1f1114;   /* texto padrão */
  --muted:   #6f5b5f;   /* texto secundário */
  --bg:      #ffffff;   /* fundo */
  ...
}
```

> ⚠️ Por histórico, a cor `#640d13` e sua versão `rgba(100, 13, 19, …)` também estão **escritas diretamente**
> em várias regras (header, menu, botões). Para trocar a cor da marca por completo, faça busca e substituição
> global por `#640d13` **e** por `100, 13, 19` em `style.css`.

### 5.5 Fontes

Carregadas do Google Fonts no `<head>` do `index.html` (comentário `<!-- Tipografia -->`):
- **Inter** — texto corrido (`body` em `style.css`)
- **Plus Jakarta Sans** — títulos

Para trocar: altere o `<link>` do Google Fonts e os `font-family` correspondentes no `style.css`.

### 5.6 Avaliações do Google

Na seção `<!-- PROVAS SOCIAIS`. Cada avaliação é um bloco `<article class="review-card">`.
Para adicionar uma, **copie um `<article>` inteiro** e troque o nome e o texto. O carrossel se ajusta sozinho — não precisa mexer no JS.

Atualize também o total exibido: busque por `37 avaliações`.

> As avaliações são **estáticas** (copiadas manualmente do Google). Não há integração automática.

### 5.7 Perguntas frequentes (FAQ)

Na seção `<!-- FAQ -->`. Cada pergunta é um bloco:

```html
<details class="faq-item">
  <summary>Pergunta aqui?</summary>
  <p>Resposta aqui.</p>
</details>
```

O atributo `open` (presente só na primeira) faz a pergunta já aparecer aberta.

### 5.8 Cards (benefícios, especialidades, diferenciais, passos)

Mesma lógica: copie um `<article>` existente da seção e edite. Os grids são responsivos.
Se mudar a **quantidade** de cards, confira o número de colunas no bloco `GRIDS` do `style.css`
(`.benefits-grid` e `.premium-grid` usam 4 colunas; `.specialties-grid` usa 3).

### 5.9 Menu de navegação

Os links do menu existem em **dois lugares**, que devem ser mantidos iguais:
- `<nav class="nav desktop-nav">` — menu do computador
- `<div class="mobile-menu" id="mobileMenu">` — menu do celular

Cada link aponta para o `id` de uma seção (ex.: `href="#faq"` → `<section id="faq">`).
Para criar um link novo, dê um `id` à seção e adicione o `<a>` nos dois menus. Os links do rodapé (`Links rápidos`) são uma terceira lista independente.

### 5.10 SEO (título no Google e compartilhamento)

No `<head>` do `index.html`:

| O quê | Onde |
|---|---|
| Título da aba / do Google | `<title>` |
| Descrição no Google | `<meta name="description">` |
| Prévia ao compartilhar (WhatsApp, Instagram etc.) | `<meta property="og:…">` |

### 5.11 Endereço, Instagram e CRP

Esses dados se repetem pela página. Use busca global no `index.html`:
- Endereço: `MontBlanc` (aparece em várias seções e no rodapé)
- Instagram: `instagram.com/psistellamoraes_`
- CRP: `01/25023` (no `<title>`, nas meta tags, no topo, no card da profissional e no rodapé)

---

## 6. Comportamentos em JavaScript

Tudo em [script.js](script.js), dividido em 4 blocos independentes (cada um com um cabeçalho de comentário):

| Bloco | O que faz | Ajustes possíveis |
|---|---|---|
| `MENU MOBILE` | Abre/fecha o menu ☰ no celular e fecha ao clicar num link | — |
| `REVEAL ON SCROLL` | Elementos com a classe `reveal` aparecem suavemente ao entrar na tela | `threshold` (quanto do elemento precisa estar visível). Atraso escalonado: classes `delay-1` / `delay-2` no HTML |
| `HEADER COM ESTADO AO ROLAR` | Ao rolar, o header vira uma "pílula" flutuante (classe `is-scrolled`) | Ponto de disparo em `getHeaderTriggerPoint()`. Visual no bloco `ESTADO AO ROLAR` do CSS |
| `CARROSSEL DE AVALIAÇÕES` | Passa sozinho, setas, arrastar com mouse/dedo, pausa no hover | `AUTOPLAY_INTERVAL` (padrão `4500` ms) |

**Acessibilidade:** se o usuário tiver "reduzir movimento" ativado no sistema, as animações e o autoplay do carrossel são desligados
(`prefers-reduced-motion` no JS e no bloco `ACESSIBILIDADE` do CSS). Mantenha esse comportamento.

**Para adicionar a animação de entrada a um elemento novo:** basta incluir `class="reveal"` nele.

---

## 7. Infraestrutura: Docker, Nginx e cache

### Dockerfile

Copia **explicitamente** apenas estes itens para o Nginx:

```
index.html · style.css · script.js · imagens/ · nginx.conf
```

> ⚠️ **Criou um arquivo novo na raiz** (ex.: `robots.txt`, `sitemap.xml`, outra página `.html`, uma pasta `fonts/`)?
> Ele **não vai para produção** até você adicionar uma linha `COPY` correspondente no `Dockerfile`.
> Arquivos dentro de `imagens/` já são copiados automaticamente.

O container tem *healthcheck* (requisição a `/` a cada 30 s) — o EasyPanel usa isso para saber se o site está no ar.

### nginx.conf

- **gzip** ligado para HTML/CSS/JS/SVG.
- **Headers de segurança:** `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`; versão do Nginx oculta.
- **Arquivos ocultos** (`.git`, `.env`, …) bloqueados.
- Qualquer rota desconhecida devolve o `index.html`.
- **HTTPS/SSL** não é tratado aqui — é responsabilidade do EasyPanel (proxy na frente do container).

### Cache — atenção

| Arquivo | Cache no navegador |
|---|---|
| `index.html` | **nenhum** — sempre baixa a versão mais nova |
| `.css`, `.js`, imagens, fontes | **30 dias**, marcado como `immutable` |

Consequência prática: se você alterar `style.css` mantendo o mesmo nome, quem já visitou o site
**pode continuar vendo o CSS antigo por até 30 dias**. Para forçar a atualização, mude a referência no `index.html` a cada alteração:

```html
<link rel="stylesheet" href="style.css?v=2" />   <!-- incremente o número -->
<script src="script.js?v=2"></script>
```

Para imagens, use um nome de arquivo novo (ver [5.3](#53-fotos)).

---

## 8. Checklist antes de subir

- [ ] Testei localmente no computador **e** em largura de celular (DevTools → modo responsivo, ~375 px).
- [ ] Todos os botões de WhatsApp abrem a conversa certa (`grep -c "wa.me/" index.html` → 7).
- [ ] Imagens novas têm `alt`, são leves (< ~300 KB) e o caminho em `src` está correto (maiúsculas/minúsculas importam no servidor Linux).
- [ ] Se alterei CSS/JS, incrementei o `?v=` no `index.html`.
- [ ] Se criei arquivo novo fora de `imagens/`, adicionei o `COPY` no `Dockerfile`.
- [ ] `git status` não mostra arquivos apagados por engano — principalmente em `imagens/`.
- [ ] Após o deploy, abri o site em aba anônima para confirmar.

---

## 9. Débitos técnicos conhecidos

Itens que funcionam, mas valem ajuste quando houver tempo:

- **Link do WhatsApp repetido 7×** no HTML. Melhoria: centralizar em um único lugar (ex.: atributo `data-whatsapp` preenchido via JS) para editar em um ponto só.
- **Cor da marca fixa no CSS** em vários pontos, em vez de usar só `var(--primary)` (ver [5.4](#54-cores-e-identidade-visual)).
- **CSS sem uso:** classes `.testimonial-card`, `.testimonials-grid`, `.orb*`, `.particles` sobraram de versões anteriores.
- **SEO:** faltam `og:image`, `<link rel="canonical">`, `robots.txt`, `sitemap.xml` e dados estruturados (`schema.org/Psychologist`).
- **Menu "Sobre"** aponta para `#sobre`, que é a seção *Onde e como atendo*; a seção *Sobre a profissional* não tem `id`.
- **Imagens** em JPEG/PNG grandes (logo em 1920×1080). Converter para WebP e redimensionar melhora o carregamento no celular.
- **Cache sem versionamento automático** de CSS/JS (ver [Cache](#cache--atenção)).

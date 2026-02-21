---
title: "Do Zero ao Deploy: Como publiquei meu site com Hugo e Cloudflare Pages"
date: 2026-02-21T18:00:00-03:00
description: "Um guia prático sobre a criação deste blog utilizando Hugo, VS Code e Cloudflare."
tags: ["hugo", "cloudflare", "tutorial", "git", "webdev"]
categories: ["Tutorial"]
---

Sejam bem-vindos ao meu primeiro post! Hoje vou mostrar os bastidores de como este site nasceu. Escolhi uma stack moderna, rápida e, o melhor de tudo, totalmente gratuita para hospedar.

## O que é o Hugo?

O **Hugo** é um dos geradores de sites estáticos (SSG) mais populares do mundo. Escrito em Go, sua principal característica é a velocidade absurda: ele consegue construir milhares de páginas em poucos segundos. 

Diferente do WordPress, o Hugo não precisa de um banco de dados. Ele transforma arquivos de texto (Markdown) em arquivos HTML prontos para serem servidos, o que torna o site extremamente seguro e leve.

## Escolhendo o Visual: Temas

Uma das partes mais divertidas é escolher o design. O lugar oficial para encontrar temas é o [Hugo Themes](https://themes.gohugo.io/). Lá você encontra centenas de opções para blogs, portfólios e sites institucionais.

Para este projeto, escolhi o **Anatole**, um tema minimalista, com duas colunas e um modo escuro elegante que valoriza o conteúdo.

---

## O Passo a Passo do Projeto

Aqui está o roteiro de como tirei a ideia do papel usando meu MacBook e o **VS Code**.

### 1. Preparação e Git
Tudo começa com o controle de versão. O **Git** é essencial porque ele permite que a Cloudflare "enxergue" as mudanças no código. Após instalar o Hugo, iniciamos o projeto e conectamos o tema como um submódulo para facilitar atualizações futuras:

```
hugo new site blog
cd blog
git init
git submodule add [https://github.com/lxndrblz/anatole.git](https://github.com/lxndrblz/anatole.git) themes/anatole
```
### 2. Personalização no VS Code
Com o projeto aberto no VS Code, ajustamos a estrutura. O segredo do Hugo moderno está na pasta config/_default/. Lá, editamos os arquivos principais:

hugo.toml: Para configurar o idioma (pt-br), o link final do site (baseURL) e o título da aba.

params.toml: Onde a mágica acontece! Mudamos a foto de perfil, o nome do autor e a bio.

menus.toml: Onde organizamos (ou comentamos usando #) os links do topo do site como Home, Blog e Sobre.

### 3. Subindo para o GitHub
Com o site rodando perfeitamente no servidor local (localhost:1313), enviamos o código para o GitHub. Isso cria um backup seguro e permite o deploy automático:
```
git add .
git commit -m "Configuração completa do blog"
git push -u origin main
```
### 4. Deploy na Cloudflare Pages
A Cloudflare Pages é onde eu hospedo o site. Conectamos o repositório do GitHub e configuramos o ambiente de compilação. Um detalhe técnico importante foi definir a variável HUGO_VERSION como 0.145.0 nas configurações da Cloudflare, garantindo compatibilidade com as funções mais recentes do tema.

### Conclusão
Ter um site estático hoje em dia é a melhor escolha para quem busca performance e simplicidade. O fluxo Hugo + GitHub + Cloudflare cria uma esteira de publicação automática: eu escrevo no VS Code, faço um "push" e, em segundos, o site está atualizado para todo o mundo ver!

Espero que esse guia ajude quem também quer criar seu cantinho na web de forma profissional e elegante.

Até o próximo post and keep learning =) 
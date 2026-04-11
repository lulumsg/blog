---
title: "Personalizando seu terminal do Mac"
date: 2026-04-04T16:50:00-03:00
description: "Guia passo a passo para configurar a stack definitiva: iTerm2, Oh My Zsh, Powerlevel10k e Auto-suggestions."
tags: ["tutorial", "iterm2", "zsh", "macos", "produtividade"]
categories: ["Setup"]
---

Se você, assim como eu, estava acostumado(a) com um certo nível de customização do terminal (vim do Ubuntu) e deseja algo melhor do que o Terminal padrão do macOS, aqui está um tutorial para personalizar seu terminal com o **iTerm2**, **Oh My Zsh** e o **Powerlevel10k**.

Meu terminal ficou assim:

![Terminal](/images/terminal.png)

---

### Passo 1: Instalar o iTerm2 usando o Homebrew

O primeiro passo é substituir o terminal nativo pelo **iTerm2**, que oferece recursos como split panes e busca avançada. A forma que escolhi de fazer isso no Mac é via Homebrew.

No seu terminal atual, execute:
```bash
brew install --cask iterm2
```

Eu utilizei o esquema de cores Vesper. Ele traz um visual muito bacana para mim. Para baixar, use o comando abaixo:

```bash
curl -L https://raw.githubusercontent.com/mbadolato/iTerm2-Color-Schemes/master/schemes/Vesper.itermcolors -o Vesper.itermcolors
```

Para definir o tema no seu:

Acesse iTerm → Settings → Profiles → Colors → Color Presets.


Se você quiser mais temas, confira o [iTerm2 Color Schemes](https://iterm2colorschemes.com/).

---

### Passo 2: Instalar o Oh My Zsh

Com o iTerm2 instalado, o próximo passo é instalar o **Oh My Zsh**, um framework para gerenciar sua configuração do Zsh de forma fácil e poderosa.

No terminal, cole e execute:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Passo 3: Instalar as Fontes (Nerd Fonts)

O tema Powerlevel10k exige uma fonte com suporte a ícones (Nerd Fonts). A recomendada é a **Meslo Nerd Font**, mas você pode escolher outra fonte.

1. Baixe as fontes MesloLGS NF recomendadas pelo [repositório oficial clicando aqui](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k).
2. Instale as fontes no seu Mac.
3. No iTerm2, vá em **iTerm2 → Settings → Profiles → Text**.
4. Na seção **Font**, selecione a `MesloLGS NF`.

### Passo 4: Instalar o Powerlevel10k

Agora vamos instalar o **Powerlevel10k**, que é um tema super rápido, flexível e famoso da comunidade.

Clone o repositório do tema com o comando:

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Em seguida, abra o arquivo `~/.zshrc` no seu editor favorito (por exemplo, `nano ~/.zshrc`) e procure pela variável `ZSH_THEME`. Altere para o valor abaixo:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Salve e reinicie o seu terminal (ou execute `source ~/.zshrc`). Ao fazer isso, o assistente de configuração do Powerlevel10k vai abrir automaticamente e você pode seguir respondendo as perguntas interativas para deixar o visual como você preferir! Caso o assistente não abra, ou se quiser refazer a configuração depois, rode:

```bash
p10k configure
```

### Passo 5: Plugins (Auto-suggestions e Syntax Highlighting)

Plugins são essenciais para ganhar produtividade real no terminal. Dois dos mais recomendados são o `zsh-autosuggestions` (sugere comandos com base no histórico que você já digitou) e o `zsh-syntax-highlighting` (colore de verde comandos válidos e de vermelho os comandos que contêm algum erro de digitação).

1. Clone os repositórios oficiais na pasta custom do Oh My Zsh:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

2. Volte a abrir o seu `~/.zshrc`, procure pela linha com um conjunto de plugins, geralmente algo como `plugins=(git)`, e modifique adicionando os novos:

```zsh
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

3. Recarregue novamente com `source ~/.zshrc`.

---

### Dica Bônus: Atalhos de Navegação no iTerm2

Para fechar o meu setup, eu gosto de configurar alguns atalhos de teclado no iTerm2 que vão me poupar muito tempo para pular palavras ou limpar linhas ([baseado no guia do Marius Schulz](https://mariusschulz.com/blog/keyboard-shortcuts-for-jumping-and-deleting-in-iterm2)).

Vá em **iTerm2 → Settings → Profiles → Keys → Key Mappings** (ou abra aba "Keys" global) e adicione as seguintes combinações clicando no `+`:

- **Pular para o início da palavra (`⌥ ←`)**: 
  - Action: Send Escape Sequence
  - Esc+: `b`
- **Pular para o final da palavra (`⌥ →`)**: 
  - Action: Send Escape Sequence
  - Esc+: `f`
- **Pular para o início da linha (`⌘ ←`)**:
  - Action: Send Hex Code
  - Hex Code: `0x01`
- **Pular para o final da linha (`⌘ →`)**:
  - Action: Send Hex Code
  - Hex Code: `0x05`
- **Apagar a palavra inteira para trás (`⌥ ⌫`)**:
  - Action: Send Hex Code
  - Hex Code: `0x17`
- **Apagar a linha inteira (`⌘ ⌫`)**:
  - Action: Send Hex Code
  - Hex Code: `0x15`

Pronto! Com esses passos, o seu novo terminal turbinado com iTerm2, Oh My Zsh e Powerlevel10k vai te ajudar com fluxos de trabalho incrivelmente mais produtivos :)
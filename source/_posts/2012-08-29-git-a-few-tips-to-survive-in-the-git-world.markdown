---
layout: post
title: "Sobrevivendo ao mundo GIT - 1"
date: 2012-08-29 20:10
comments: true
sharing: true
categories: [git, git tips, truques git, trocando mensagem de commits, change commits message]
---
Algumas vezes no processo diário de desenvolvimento e versionamento dos nossos fontes, nos vemos em situações nas quais temos que lidar com o GIT para realizar algumas tarefas não tanto usuais.

Neste post e alguns no futuro, pretendo mostrar como procedo em alguns destes casos:

<h3>Puts... Errei a mensagem de commit! Como faço para trocá-la?</h3>

Tranquilo! 

Se o seu commit foi o último realizado no branch local, simplesmente use:

```
git commit --amend -m "nova mensagem"
```

Se quer trocar a mensagem de outros commits, use:
(assumindo que você possui um editor padrão configurado no ~/.gitconfig)
```
git reabase -i SHA_DO_COMMIT_ANTERIOR_AO_QUE_SE_DESEJA_EDITAR_A_MENSAGEM
```
O "-i" acima representa o modo interativo, onde o GIT permite controle manual sob o processo de rebase (em breve tentarei postar algo mais profundo sobre o rebase).

Bom, após executar o comando, será exibida uma tela como esta, onde será possível, dentre outras coisas, trocar a mensagem do commit desejado:

```
pick f318e91 Implementando filtro anual 
pick 582a73e Implementando componente de filtro de intervalo de datas

# Rebase a9fda57..582a73e onto a9fda57
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message

```
Basta modificar a opção <strong>pick</strong> por <strong>reword</strong> ou <strong>r</strong> para cada commit que se deseja editar a mensagem, salvar o arquivo e fechar o editor.

Feito isso, será aberto um arquivo para cada commit apontado anteriormente com o comando 'reword', exibindo a tela para edição no formato abaixo (supondo que selecionei o comando para 'reword' do primeiro commit acima):
```
Implementando componente de filtro de intervalo de datas

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# Not currently on any branch.
# Changes to be committed:
#   (use "git reset HEAD^1 <file>..." to unstage)
#
```
Aí, basta editar a mensagem, salvar novamente e sair e as alterações serão aplicadas em nível de branch local.
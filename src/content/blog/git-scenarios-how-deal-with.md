---
title: 'Sobrevivendo ao Git no dia a dia: Como não destruir o histórico do seu time (e manter a sanidade)'
description: 'Aprenda a resolver conflitos de merge, branches desatualizadas, hotfixes em produção e commits na branch errada de forma profissional.'
pubDate: 'May 28 2026'
heroImage: '../../assets/git-scenarios-how-deal-with.png'
---

Recentemente, numa mentoria, escutei a seguinte frase: 

**"Quando trabalhamos sob pressão, somos reduzidos a nosso nível de treino".** 

Numa profissão pragmática como a nossa, exige-se prática constante com as ferramentas e linguagens que temos à disposição. Se não enxergarmos esses recursos como uma extensão do nosso próprio corpo, em momentos de alta pressão — como quando surge um bug em produção às 16h de uma sexta-feira — a tendência é entrar em desespero e ficar sem reação.

Por isso, é fundamental estarmos tecnicamente preparados para situações assim. Só então conseguimos manter a calma, tomar decisões com clareza e ter domínio sobre o problema.

Reuni aqui, na minha humilde opinião, as quatro situações mais comuns e perigosas que enfrentamos no dia a dia de desenvolvimento utilizando nosso querido Git.


<br />

#### 1. Conflitos de merge no pull request

##### O cenário

Você terminou sua feature normalmente.

Tudo parecia perfeito.

Mas ao abrir o pull request:

```txt
This branch has conflicts that must be resolved
```

Outro desenvolvedor alterou exatamente o mesmo trecho do arquivo que você modificou.

O git agora não consegue decidir sozinho qual versão deve permanecer.

---

##### Por que isso acontece?

O git consegue unir alterações automaticamente apenas quando elas acontecem em regiões diferentes do código.

Quando duas pessoas modificam:

- o mesmo arquivo
- a mesma linha
- ou blocos muito próximos

o merge automático falha.

---

##### Como resolver corretamente

Primeiro atualize sua branch local com a branch principal do projeto.

<div class="terminal">

```bash
git checkout feature/payment-flow
git fetch origin
git rebase origin/develop
```

</div>

Agora o git mostrará os conflitos.

Você verá algo parecido com isso:

```txt
<<<<<<< HEAD
Mensagem antiga
=======
Mensagem nova
>>>>>>> develop
```

Esses marcadores representam:

| marcador | significado |
|---|---|
| `HEAD` | sua alteração |
| `develop` | alteração da branch principal |

---

##### Resolvendo o conflito

Agora você deve decidir:

- qual versão manter
- se vai unir ambas
- ou se criará uma nova solução

Depois de corrigir o arquivo:

<div class="terminal">

```bash
git add .
git rebase --continue
```

</div>

Se novos conflitos aparecerem, repita o processo.

---

##### Erro comum

Muita gente resolve conflito apagando tudo rapidamente até “o código voltar a funcionar”.

Isso é extremamente perigoso.

Conflito de merge não é problema de sintaxe.

É problema de contexto.

Você precisa entender:

- o que sua feature alterou
- o que entrou recentemente no projeto
- qual regra de negócio deve sobreviver

---

<br />
<br />

#### 2. Branch desatualizada e histórico caótico

##### O cenário

Você passou vários dias trabalhando em uma feature.

Enquanto isso:

- dezenas de commits entraram na `develop`
- outros desenvolvedores alteraram arquivos importantes
- o projeto evoluiu bastante

Agora sua branch está completamente desatualizada.

---

##### O erro mais comum

Muitos desenvolvedores fazem isso:

<div class="terminal">

```bash
git merge develop
```

</div>

O resultado normalmente é:

- histórico poluído
- commits duplicados
- múltiplos merges desnecessários
- dificuldade de rastrear mudanças

---

##### A abordagem mais limpa

Antes do rebase, organize seus commits.

Se sua branch está assim:

```txt
fix
fix 2
tentativa
agora vai
ajuste final
```

transforme isso em algo coerente.

---

##### Usando rebase interativo

<div class="terminal">

```bash
git rebase -i HEAD~5
```

</div>

O git abrirá algo parecido:

```txt
pick a1b2 fix
pick c3d4 fix 2
pick e5f6 tentativa
pick g7h8 agora vai
pick i9j0 ajuste final
```

Troque os próximos commits para `squash`:

```txt
pick a1b2 fix
squash c3d4 fix 2
squash e5f6 tentativa
squash g7h8 agora vai
squash i9j0 ajuste final
```

Agora você terá apenas um commit limpo.

---

##### Atualizando a branch corretamente

Depois disso:

<div class="terminal">

```bash
git fetch origin
git rebase origin/develop
```

</div>

Isso reaplica sua feature sobre a versão mais recente da `develop`.

---

##### Por que isso melhora o histórico?

Porque o histórico fica linear.

Sem múltiplos merges intermediários.

Sem “merge da develop na feature da feature da feature”.

---

##### Quando evitar rebase

Evite rebase em branches compartilhadas por outras pessoas.

Rebase reescreve histórico.

E isso pode causar problemas para o restante do time.

---

<br />
<br />

#### 3. Bug crítico em produção enquanto a develop está instável

##### O cenário

O sistema em produção quebrou.

Você precisa corrigir imediatamente.

Mas existe um problema:

A branch `develop` possui funcionalidades incompletas que ainda não podem ir para produção.

---

##### O erro mais perigoso

Fazer deploy da `develop`.

Isso pode colocar no ar:

- código experimental
- funcionalidades inacabadas
- migrations incompletas
- refactors instáveis

---

##### A solução correta: hotfix

A ideia do hotfix é simples:

Criar uma correção emergencial baseada diretamente na branch de produção.

---

##### Criando a branch de hotfix

<div class="terminal">

```bash
git checkout main
git pull origin main
git checkout -b hotfix/payment-timeout
```

</div>

Agora você possui uma branch isolada apenas para a correção crítica.

---

##### Aplicando a correção

Faça apenas o necessário.

Evite:

- refatorar
- reorganizar código
- alterar arquitetura
- “aproveitar para melhorar outra coisa”

Hotfix existe para resolver urgência.

---

##### Finalizando

<div class="terminal">

```bash
git add .
git commit -m "fix: corrige timeout em produção"
```

</div>

Depois:

<div class="terminal">

```bash
git checkout main
git merge hotfix/payment-timeout
```

</div>

Agora o deploy pode ser feito com segurança.

---

##### O passo que muita gente esquece

A correção também precisa voltar para a `develop`.

Caso contrário:

o bug reaparecerá futuramente.

---

##### Sincronizando a correção

<div class="terminal">

```bash
git checkout develop
git merge hotfix/payment-timeout
```

</div>

Agora ambas as branches possuem a correção.

---

<br />
<br />

#### 4. Commit na branch errada

##### O cenário

Você esqueceu de criar sua branch de feature.

E acabou commitando diretamente na:

- `main`
- `develop`

Isso acontece o tempo inteiro.

Principalmente em dias corridos.

---

##### Se você ainda não fez push

Então o problema é simples de resolver.

Primeiro crie a branch correta:

<div class="terminal">

```bash
git checkout -b feature/user-avatar
```

</div>

Seus commits irão junto automaticamente.

---

##### Agora limpe a branch errada

Volte para a branch original:

<div class="terminal">

```bash
git checkout develop
```

</div>

Depois remova os commits locais:

<div class="terminal">

```bash
git reset --hard HEAD~1
```

</div>

Ou:

<div class="terminal">

```bash
git reset --hard HEAD~2
```

</div>

Dependendo da quantidade de commits.

---

##### Entendendo o HEAD~1

Isso significa:

```txt
voltar um commit antes do atual
```

Visualmente:

```txt
A --- B --- C
```

Depois do reset:

```txt
A --- B
```

---

##### E se o push já tiver sido feito?

Nesse caso evite `reset`.

O mais seguro normalmente é:

<div class="terminal">

```bash
git revert
```

</div>

Porque o revert cria um novo commit desfazendo as alterações sem reescrever o histórico remoto.

---

##### Como evitar isso no futuro

A melhor proteção é configurar regras no repositório:

- bloquear push direto
- exigir pull request
- exigir aprovação
- proteger `main` e `develop`
- exigir pipeline passando

GitHub, GitLab e Bitbucket possuem isso nativamente.

---
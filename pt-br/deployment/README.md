# Colocando em Produção

O Heroku torna fácil a publicação das aplicações. É uma plataforma perfeita para
pequenas e médias aplicações web que se dispoem a sacrificar um pouco de
flexibilidade na sua infraestrutura em troca de um ambiente livre de
complicações no que se refere a publicação e manutenção.

Estou optando por publicar as nossas aplicações web no Heroku ao longo deste
tutorial, por que em minha experiência este tem sido o meio mais rápido de
se ter uma aplicação web de pé e executando em tempo mínimo. Lembre-se de que o
foco deste tutorial está em como construir aplicações web em Go, sem ser pego
pelas diversas distrações de provisionamento, configuração, publicação e
manutenção das máquinas em que o nosso código Go irá executar.

## Configuração

Se você ainda não tem uma conta no Heroku, crie uma em
[id.heroku.com/signup](https://id.heroku.com/signup). É rápido, fácil e grátis.

O gerenciamento e configuração da aplicação é feito através do Cinto de
Utilidades do Heroku (Heroku toolbelt), que é uma ferramenta de linha de
comando livre e mantida pelo Heroku. Nós a usaremos para criar as nossas
aplicações no Heroku. Você pode obtê-la em
[toolbelt.heroku.com](https://toolbelt.heroku.com/).

## Alterando o Código

Para garantir que a nossa aplicação do capítulo anterior executará no Heroku,
nós teremos que realizar algumas modificações. O Heroku nos fornece uma variável
de ambiente chamada `PORT` (porta) e espera que a nossa aplicação web execute
nesta porta. Vamos começar importando o pacote "os", deste modo nós seremos
capazes de realizar a leitura da variável de ambiente `PORT`:

``` go
import (
    "net/http"
    "os"

    "github.com/russross/blackfriday"
)
```

Em seguida, nós teremos de realizar a leitura da variável de ambiente `PORT`,
verificar se ela está definida, e se estiver, nós deveremos colocar a aplicação
para escutar nesta porta, ao invés da porta que foi definida de maneira fixa no
código (8080).  

``` go
port := os.Getenv("PORT")
if port == "" {
  port = "8080"
}
```

Para finalizar, passaremos esta porta na nossa chamada para
`http.ListenAndServe`:

``` go
http.ListenAndServe(":"+port, nil)
```

O código final deverá ficar parecido com isso:

``` go
package main

import (
    "net/http"
    "os"

    "github.com/russross/blackfriday"
)

func main() {
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    http.HandleFunc("/markdown", GenerateMarkdown)
    http.Handle("/", http.FileServer(http.Dir("public")))
    http.ListenAndServe(":"+port, nil)
}

func GenerateMarkdown(rw http.ResponseWriter, r *http.Request) {
    markdown := blackfriday.MarkdownCommon([]byte(r.FormValue("body")))
    rw.Write(markdown)
}
```

## Configuração

Nós precisaremos de alguns pequenos arquivos de configuração para indicar ao
Heroku como executar a nossa aplicação. O primeiro arquivo é o `Procfile`, o
qual nos permite definir quais processos deverão ser executados para a nossa
aplicação. Por padrão, o Go irá nomear o executável utilizando o nome do
diretório onde estiver o código do seu package main. Por exemplo, se a minha
aplicação web estiver em `GOPATH/github.com/codegangsta/bwag/deployment`, meu
`Procfile` deverá ficar assim:

```
web: deployment
```

Especificamente no caso de aplicações Go, nós também teremos que criar um
arquivo `.godir` para indicar ao Heroku qual diretório é de fato o nosso
diretório de pacotes.

```
deployment
```

## Publicação

Uma vez que todas estas coisas estejam em seu devido lugar, o Heroku torna fácil
a sua publicação.

Inicialize o projeto como um repositório do Git
``` bash
git init
git add -A
git commit -m "Initial Commit"
```

Crie a sua aplicação no Heroku (especificando o buildpack do Go)
``` bash
heroku create -b https://github.com/kr/heroku-buildpack-go.git
```

Envie o código para o Heroku e veja a sua aplicação ser publicada!
``` bash
git push heroku master
```

Veja a sua aplicação em seu navegador:
``` bash
heroku open
```

# Colocando em Produção

O Heroku torna fácil a publicação das aplicações. É uma plataforma perfeita para
pequenas e médias aplicações web que se dispoem a sacrificar um pouco de
flexibilidade na sua infraestrutura em troca de um ambiente livre de
complicações no que se refere a publicação e manutenção.

Estou escolhendo publicar as nossas aplicações web no Heroku ao longo deste
tutorial, por que em minha experiência este tem sido o meio mais rápido de
se ter uma aplicação web de pé e executando em tempo mínimo. Lembre-se de que o
foco deste tutorial está em como construir aplicações web em Go, sem ser pego
pelas diversas distrações de provisionamento, configuração, publicação e
manutenção das máquinas em que o nosso código Go irá executar.

## Configuração

Se você ainda não tem uma conta no Heroku, crie uma em
[id.heroku.com/signup](https://id.heroku.com/signup). É rápido, fácil e grátis.

O gerenciamento e configuração da aplicação é feito através do Heroku toolbelt,
que é uma ferramenta de linha de comando livre e mantida pelo Heroku. Nós a
usaremos para criar as nossas aplicações no Heroku. Você pode obtê-la em
[toolbelt.heroku.com](https://toolbelt.heroku.com/).

## Alterando o Código

Para garantir que a nossa aplicação do capítulo anterior vai executar no Heroku,
nós vamos ter que realizar algumas modificações. O Heroku nos fornece uma
variável de ambiente chamada `PORT` (porta) e espera que a nossa aplicação web
execute nesta porta. Vamos começar importando o pacote "os", deste modo nós
seremos capazes de realizar a leitura da variável de ambiente `PORT`:

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

## Configuration

We need a couple small configuration files to tell Heroku how it should run our
application. The first one is the `Procfile`, which allows us to define which
processes should be run for our application. By default, Go will name the
executable after the containing directory of your main package. For instance,
if my web application lived in `GOPATH/github.com/codegangsta/bwag/deployment`, my
`Procfile` will look like this:

```
web: deployment
```

Specifically to run Go applications, we need to also specify a `.godir` file to
tell Heroku which dir is in fact our package directory.

```
deployment
```

## Deployment

Once all these things in place, Heroku makes it easy to deploy.


Initialize the project as a Git repository:
``` bash
git init
git add -A
git commit -m "Initial Commit"
```

Create your Heroku application (specifying the Go buildpack):
``` bash
heroku create -b https://github.com/kr/heroku-buildpack-go.git
```

Push it to Heroku and watch your application be deployed!
``` bash
git push heroku master
```

View your application in your browser:
``` bash
heroku open
```

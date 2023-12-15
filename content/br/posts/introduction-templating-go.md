+++
title = 'Introdução a templating em Go'
date = 2023-12-15
draft = false
tags = ['go','templating','introduction']
+++

Computadores e linguagens de programação surgiram para facilitar as nossas vidas e automatizar as tarefas do cotidiano. No dia a dia de nós, programadores e engenheiros de software, muitas vezes temos que criar diversos arquivos semelhantes, cujo um ou outro campo muda de forma sutil. Um exemplo claro são arquivos de configuração, faturas, XMLs, HTMLs ou qualquer arquivo que nos permita gerar arquivos similares mudando poucos pontos. Existe uma solução simples para esse problema: criar um template e ir alterando só as partes que preciso de forma manual! Contudo, isso não é a melhor forma de resolver o problema, pois ela não é escalável. Além de que podemos utilizar tecnologia para facilitar nossas vidas.

Go fornece o pacote [text/template](https://pkg.go.dev/text/template) que permite a criação de templates e também fornece ferramentas que ajudam a preencher os campos deste template. Um template é nada mais que um arquivo de texto, tendo como covenção usar `.tmpl`, com algumas marcações especificas dentro do texto que dizem o que pode ser feito, entre elas pode-se fazer condicionais, laços e etc. Para uma lista mais completa de quais ações podem ser executadas, recomendo ler a [documentação](https://pkg.go.dev/text/template#hdr-Actions) que detalha todas as possibilidades. Para exemplificar, vamos criar nosso template de uma fatura:

```tmpl
Invoice #{{ .Number }}
---------------------------

Item name - Unitary price - Quantity - Total price
---------------------------
{{- range .Items }}
{{ .Name }} -  ${{ .Price }} - {{ .Quantity }} - ${{ .TotalPrice -}}
{{ end }}

---------------------------
Total Value   {{ .TotalPrice }}
```

Neste template, podemos perceber algumas ações executadas:
- `{{ .Number }}` renderiza o valor para o número da fatura.
- `{{- range .Items }}` itera entre os itens da fatura.
- `{{ .TotalPrice -}}` renderiza o valor para o preço total dos itens e remove espaços ou quebras de linha após o texto.

Com o template criado, vamos salvar o arquivo como `invoice.tmpl` e agora precisamos que o template receba as informações necessárias. A maneira de se fazer isso é definindo uma estrutura de dados com os campos que o template usa. No nosso caso, precisamos de duas estruturas, que são a `Invoice` e a `Item`.

```golang
type Invoice struct {
	Number     string
	Items      []Item
	TotalPrice float64
}

type Item struct {
	Name       string
	Price      float64
	Quantity   int
	TotalPrice float64
}
```

E com as estruturas definidas, lemos o template e renderizamos o resultado com os dados informados:

```golang
func main() {
	invoice := Invoice{Number: "1234", Items: []Item{{Name: "Item 1", Price: 12.34, Quantity: 2, TotalPrice: 24.68}, {Name: "Item 2", Price: 56.78, Quantity: 1, TotalPrice: 56.78}}, TotalPrice: 81.46}
	file := "invoice.tmpl"

  tmpl, err := template.New(file).ParseFiles(file)
	if err != nil {
		panic(err)
	}

	err = tmpl.Execute(os.Stdout, invoice)
	if err != nil {
		panic(err)
	}
}
```

O que importa pra nós nesse código são os comandos:
- `template.New(file).ParseFiles(file)`. Este comando recebe qual o nome do template a ser utilizado e quais arquivos vão ser usados como base. Ele vai inicializar um template de fato.
- `tmpl.Execute(os.Stdout, invoice)`. Este segundo comando recebe dois parâmetros: a saída, no caso Stdout, e quais os dados de entrada, no caso, uma Invoice. Ele renderiza o template com os dados e joga na saída desejada.

Se executarmos nosso código, temos:

```text
Invoice #1234
---------------------------

Item name - Unitary price - Quantity - Total price
---------------------------
Item 1 -  $12.34 - 2 - $24.68
Item 2 -  $56.78 - 1 - $56.78

---------------------------
Total Value   81.46
```

Viu como é simples utilizar templates com Go? Conseguimos resolver o problema de forma simples e escalável. Caso você queira, também pode checar o código fonte neste [repositório](https://github.com/mfbmina/templating-golang). Também é possível utilizar templates HTML, mas vamos deixar isso pra uma próxima postagem.

Caso tenha gostado dessa postagem, siga meu blog para mais contéudos assim! Você também pode me encontrar no **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** ou **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**

Edit 1: Queria agradecer ao meu amigo @Cassio Botaro, pela revisão do texto e por ter dado algumas idéias! Muito obrigado, mano!

+++
title = 'Introduction to templating with Go'
date = 2023-12-15
draft = false
tags = ['go','templating','introduction']
+++

Computer and programming languages were born to make our lives easier,  as they automatize day-to-day tasks. Programmers and software engineers usually have to build files that are almost the same as others, where the only change is one field or another. For example, there are configuration files, invoices, XMLs, HTMLs or any file which we can use to build other files. There is a simple solution for this problem: we can create a template and change the parts that I need manually! That works, but it is not the best way to deal with the problem because it is scalable. We can use technology to help.

Go gives the [text/template](https://pkg.go.dev/text/template) package, which allows developers to create their own templates and also gives the tools to deal with them. A template is just a `.tmpl` file with some special marks inside the text. These marks allow specific actions like conditionals, loops, etc. For a more complete list of actions, you should check the [documentation](https://pkg.go.dev/text/template#hdr-Actions) which has them detailed. As an example, we will create our own invoice template:

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

In this template, we can see some actions being executed:
- `{{ .Number }}` renders the invoice number.
- `{{- range .Items }}` loop through the invoice's items.
- `{{ .TotalPrice -}}` renders the total price of the items and removes the extra spaces or empty lines after the text.

Now that we have the template, let's save it as `invoice.tmpl` and we must give the template all the necessary data. The way of doing that is by defining a struct with all required fields. In our example, we will need two structs: invoice and item.

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

With the structs in hand, let's read the template and render the result with the provided data:

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

What matters in this code are the two functions:
- `template.New(file).ParseFiles(file)`. it receives the template's name and which files will be used as base files. It will initialize a template.
- `tmpl.Execute(os.Stdout, invoice)`: it receives two arguments, where the first is where the response should be rendered, in our case stdout, and the data that will be used. It renders the template with the data and returns it in the desired output.

If we run our code, we will see:
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

See how easy is to use templates with Go? We have solved the issue in a simple and scalable way. You can also check all the source code in this repository. It's also possible to use HTML templates, but we can talk about that in the next post.

If you've liked this post, you can follow my blog for more content like this. You can also find me on **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** or **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**

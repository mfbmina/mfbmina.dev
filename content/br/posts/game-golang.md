+++
title = 'Criando um Sliding Puzzle em Go'
date = 2024-05-13T14:56:44-03:00
draft = false
+++

Escrever um jogo é uma ótima maneira de se começar a programar, principalmente pois diversas pessoas começaram a programar por que queriam criar jogos para computadores ou até mesmo video game. Inclusive, um dos meus primeiros "projetos" foi um joguinho, ainda quando cursava o curso técnico em Informática Industrial, em 2010. 

No curso, utilizávamos Python e implementamos um slide puzzle. Foi bem desafiador na época, pois tive que entender da mecânica do jogo, de como criar uma GUI, mas entreguei o projeto. Quando comecei a trabalhar com Ruby, também fiz uma implementação para comparar com o que já tinha feito em Python. 

![Sliding puzzle](/img/posts/sliding-puzzle.png)

Decidi então escrever um sliding puzzle usando Go. O primeiro passo foi escrever o que chamei de `core`, ou seja, a parte principal do jogo. Para isso, defini uma estrutura chamada `Play` que contém o tabuleiro e as posições `x` e `y` do valor nulo. O tabuleiro pode ser representado com um array, contudo acho mais simples uma representação usando uma matriz quadrada, e pro nosso caso, escolhi uma 3x3. Para representar o valor nulo, o valor escolhido foi o `0`.

```go
var DEFAULT_TABLE = [3][3]int{{1, 2, 3}, {4, 5, 6}, {7, 8, 0}}

type Play struct {
	Table    [3][3]int
	EmptyRow int
	EmptyCol int
}
```

Depois precisamos gerar uma nova partida com um tabuleiro aleatório. Para embalharar, percorremos cada célula do tabuleiro e trocamos por outra de aleátoria. Isso nos garante que vamos ter o mínimo de embaralhamento em nosso tabuleiro.

```go
func NewPlay() *Play {
	t, x, y := generateRandomTable()
	return &Play{
		Table:    t,
		EmptyRow: x,
		EmptyCol: y,
	}
}

func generateRandomTable() ([3][3]int, int, int) {
	t := DEFAULT_TABLE
	s := 3
	xEmpty := 0
	yEmpty := 0

	for i, r := range t {
		for j := range r {
			x := rand.Intn(s)
			y := rand.Intn(s)
			t[i][j], t[x][y] = t[x][y], t[i][j]

			if t[i][j] == 0 {
				xEmpty = i
				yEmpty = j
			}

			if t[x][y] == 0 {
				xEmpty = x
				yEmpty = y
			}
		}
	}

	return t, xEmpty, yEmpty
}
```

O próximo passo foi implementar os movimentos de subida, descida, direita e esquerda. Aqui vamos sempre olhar para o valor nulo e é ele que vamos movimentar. Caso não seja possível andar com o `0`, nós retornamos um erro.

```go
func (p *Play) Up() error {
	if p.EmptyRow == 0 {
		return fmt.Errorf("can't move up")
	}

	p.Table[p.EmptyRow][p.EmptyCol], p.Table[p.EmptyRow-1][p.EmptyCol] = p.Table[p.EmptyRow-1][p.EmptyCol], p.Table[p.EmptyRow][p.EmptyCol]
	p.EmptyRow = p.EmptyRow - 1

	return nil
}

func (p *Play) Down() error {
	if p.EmptyRow == 2 {
		return fmt.Errorf("can't move down")
	}

	p.Table[p.EmptyRow][p.EmptyCol], p.Table[p.EmptyRow+1][p.EmptyCol] = p.Table[p.EmptyRow+1][p.EmptyCol], p.Table[p.EmptyRow][p.EmptyCol]
	p.EmptyRow = p.EmptyRow + 1

	return nil
}

func (p *Play) Left() error {
	if p.EmptyCol == 0 {
		return fmt.Errorf("can't move left")
	}

	p.Table[p.EmptyRow][p.EmptyCol], p.Table[p.EmptyRow][p.EmptyCol-1] = p.Table[p.EmptyRow][p.EmptyCol-1], p.Table[p.EmptyRow][p.EmptyCol]
	p.EmptyCol = p.EmptyCol - 1

	return nil
}

func (p *Play) Right() error {
	if p.EmptyCol == 2 {
		return fmt.Errorf("can't move right")
	}

	p.Table[p.EmptyRow][p.EmptyCol], p.Table[p.EmptyRow][p.EmptyCol+1] = p.Table[p.EmptyRow][p.EmptyCol+1], p.Table[p.EmptyRow][p.EmptyCol]
	p.EmptyCol = p.EmptyCol + 1

	return nil
}
```

Por fim, nos resta só verificar se o tabuleiro está num estado de vitória ou não.

```go
func (p *Play) IsWin() bool {
	return p.Table == DEFAULT_TABLE
}
```

A primeira interface gráfica que implementei foi para o STDOUT do terminal. Para seguir a interface de `View`, precisamos apenas definir uma função de `Render`. Definimos também as teclas que são utilizáveis e ficamos num loop que se quebra em duas situações: 
- ou o usuário apertou a tecla de saída
- ou o usuário venceu a partida

```go
var KEYS = map[string]string{
	"up":    "w",
	"left":  "a",
	"down":  "s",
	"right": "d",
	"quit":  "q",
}

type Stdout struct {
	Play *core.Play
}

func NewStdout() *Stdout {
	return &Stdout{Play: core.NewPlay()}
}

func (s *Stdout) Render() {
	k := ""
	w := false

	for !w && !isQuit(k) {
		s.printTable()

		k = getMove()
		err := s.move(k)
		if err != nil {
			fmt.Println(err)
		}

		w = s.Play.IsWin()
	}

	if w {
		fmt.Println("You win!")
	}
}

func (s *Stdout) move(k string) error {
	switch k {
	case KEYS["up"]:
		return s.Play.Up()
	case KEYS["left"]:
		return s.Play.Left()
	case KEYS["down"]:
		return s.Play.Down()
	case KEYS["right"]:
		return s.Play.Right()
	case KEYS["quit"]:
		return nil
	default:
		return fmt.Errorf("Invalid key. Play again.")
	}
}

func (s *Stdout) printTable() {
	for _, row := range s.Play.Table {
		for _, col := range row {
			fmt.Printf("%d ", col)
		}
		fmt.Printf("\n")
	}
}

func isQuit(k string) bool {
	return KEYS["quit"] == k
}

func getMove() string {
	reader := bufio.NewReader(os.Stdin)
	t, _ := reader.ReadString('\n')
	return strings.TrimSuffix(t, "\n")
}
```

Contudo, ao jogar algumas vezes percebi que algumas vezes o jogo era impossível de ser resolvido. Pesquisando, descobri que o problema vem do meu algoritmo de embaralhamento. Ao trocar uma célula por outra, fazemos algumas trocas que nunca seriam realizadas em um tabuleiro físico. Mas, para nossa sorte, isso é possível de identificar e resolver ao contar o número de inversões necessárias para resolver o jogo. Se o número for par ele é solucionável e se for impar, não.

```go
func solvablePuzzle(t [3][3]int) bool {
	inversions := 0
	for i, r := range t {
		for j, c := range r {
			if c == 0 {
				continue
			}
			for x := i; x < 3; x++ {
				for y := 0; y < 3; y++ {
					if x == i && y <= j {
						continue
					}
					if t[x][y] == 0 {
						continue
					}
					if t[x][y] < c {
						inversions += 1
					}
				}
			}
		}
	}

	if inversions%2 == 0 {
		return true
	}

	return false
}
```

Caso o jogo seja insolucionável, realizamos uma última troca e garantimos que o jogo tenha uma solução.

```go
func generateRandomTable() ([3][3]int, int, int) {
	// ...

	if !solvablePuzzle(t) {
		t[0][0], t[0][1] = t[0][1], t[0][0]
	}

	return t, xEmpty, yEmpty
}
```

Finalmente temos nosso jogo funcional! Podemos agora implementar uma GUI para o nosso sliding puzzle! Escolhi a biblioteca [Ebiten](https://github.com/hajimehoshi/ebiten), uma engine open source que nos permite criar jogos 2D. Ela nos obriga a implementar uma interface que define as funções `Update`, `Draw` e `Layout`.

A função `Layout` é a mais simples delas: define o tamanho da janela. 

```go
func (u *UI) Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int) {
	return 900, 900
}
```

Já a função `Draw` é responsável por desenhar a sua tela e é executada a cada frame. Para representar nosso tabuleiro, decidi utilizar imagens de 300x300, para cada um dos números válidos. Assim, a cada iteração, a função pega o valor do tabuleiro e representa visualmente. Se o jogo estiver num estado de vitória, ele exibe uma imagem de parabéns!

```go
//go:embed assets/*
var assets embed.FS

func (u *UI) Draw(screen *ebiten.Image) {
	for x, row := range u.Play.Table {
		for y, value := range row {
			if value == 0 {
				continue
			}

			img := loadImage(fmt.Sprint(value))
			op := &ebiten.DrawImageOptions{}
			fX := float64(x)
			fY := float64(y)
			op.GeoM.Translate(300*fY, 300*fX)

			screen.DrawImage(img, op)
		}
	}

	if u.Play.IsWin() {
		screen.Clear()
		img := loadImage("win")
		screen.DrawImage(img, nil)
	}
}

func loadImage(name string) *ebiten.Image {
	fName := fmt.Sprintf("assets/%s.png", name)
	f, err := assets.Open(fName)
	if err != nil {
		panic(err)
	}
	defer f.Close()

	img, _, err := image.Decode(f)
	if err != nil {
		panic(err)
	}

	return ebiten.NewImageFromImage(img)
}
```

A função `Update` é responsável por atualizar o estado do jogo, ou seja, de fato realizar alguma ação no tabuleiro. Definimos aqui qual o comportamento esperado ao se apertar alguma tecla. No `core`, nós mapeamos qualquer ação ao valor zero, contudo, ao traduzir isso para uma GUI, faz mais sentido inverter a lógica. O usuário aperta para baixo, pois quer mover o número para baixo e não o zero.

```go
func (u *UI) Update() error {
	if inpututil.IsKeyJustPressed(ebiten.KeyQ) {
		return fmt.Errorf("Quit")
	}
	if u.Play.IsWin() {
		return nil
	}
	if inpututil.IsKeyJustPressed(ebiten.KeyDown) {
		u.Play.Up()
	}
	if inpututil.IsKeyJustPressed(ebiten.KeyUp) {
		u.Play.Down()
	}
	if inpututil.IsKeyJustPressed(ebiten.KeyLeft) {
		u.Play.Right()
	}
	if inpututil.IsKeyJustPressed(ebiten.KeyRight) {
		u.Play.Left()
	}

	return nil
}
```

E por fim, definimos também nossa função `Render`, para seguir nosso contrato de UI.

```go
type UI struct {
	Play *core.Play
}

func NewUI() *UI {
	return &UI{Play: core.NewPlay()}
}

func (u *UI) Render() {
	ebiten.SetWindowSize(900, 900)
	ebiten.SetWindowTitle("Puzzle Game")
  
	if err := ebiten.RunGame(u); err != nil {
		log.Fatal(err)
	}
}
```

Temos nosso jogo pronto! Foi um projeto bem interessante de se criar e me mostrou que implementar joguinhos é sempre uma ótima maneira de se aprender conceitos novos e reforçar conhecimentos em alguma linguagem de programação. Além disso, o Ebitten nos permite criar jogos 2D usando a nossa linguagem favorita e distribuir para diversas plataformas, até mesmo para Web usando WebAssembly ou XBOX. Se quiser executar o código fonte, ele se encontra [aqui](https://github.com/mfbmina/puzzle). Me diga o que você achou dessa postagem nos comentários e fica uma pergunta: qual jogo você quer criar usando Go?

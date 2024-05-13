+++
title = 'Building a Sliding Puzzle with Go'
date = 2024-05-13T14:56:44-03:00
draft = false
tags = ["go", "gamedev"]
+++

Building a game is an excellent way of starting programming. Tons of people have started programming because they wanted to create computer games. Even me, back in 2010, had one of my first projects to write a small game when I was still in my Technician course.

In this course, we used Python to develop a slide puzzle. It was very challenging because I had to learn about the game mechanics and GUI, but I could handle the project. When I started to work with Ruby, I also built one to compare with what I did with Python.

![Sliding puzzle](/img/posts/sliding-puzzle.png)

I decided to build a sliding puzzle with Go. The first step was building the core, the main parts of the game. Therefore, I created a struct called `Play`, which has the table and positions `x` and `y` for the nil value. The table could be represented with an array, but I think the quadratic matrix represents it better. Also, to represent the nil value, I chose the `0`. 

```go
var DEFAULT_TABLE = [3][3]int{{1, 2, 3}, {4, 5, 6}, {7, 8, 0}}

type Play struct {
	Table    [3][3]int
	EmptyRow int
	EmptyCol int
}
```

After that, we will create a new play with a random table. To randomize it, we go through each cell and change it for another one by random. That guarantees us that we will have a minimum randness at our table.

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

Next, we can implement the moves for up, down, left, and right. Here, we will always move the nil value. If we can't move it, we will return an error.

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

At last, we can check if the table is in the win position.

```go
func (p *Play) IsWin() bool {
	return p.Table == DEFAULT_TABLE
}
```

The first GUI that I created was for the terminal's STDOUT. I have defined a View interface, and in order to follow it, we will need the `Render` function. We also define which keys are valid to play, and we keep a loop that only breaks if:
- the user presses the quit key
- the user wins

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

But when I played it sometimes, I noticed that some games were impossible to solve. Researching a bit, I discovered that my randomization was causing the issue because when changing a cell for another one, we can make some moves that will never occur in a real table. But we can identify and fix this issue by counting the number of inversions needed to solve the game. If the number of inversions is even, it is solvable. If it is odd, it is not.

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

If the game is unsolvable, we make a last change that guarantees the game has a solution.

```go
func generateRandomTable() ([3][3]int, int, int) {
	// ...

	if !solvablePuzzle(t) {
		t[0][0], t[0][1] = t[0][1], t[0][0]
	}

	return t, xEmpty, yEmpty
}
```

We finally have a functional game! Now, we can work on a GUI for our sliding puzzle. I've choose [Ebiten](https://github.com/hajimehoshi/ebiten), an open source engine that allows us to build 2D games. It makes us implement an interface with the functions `Update`, `Draw` e `Layout`.

Implementing `Layout` is the simplest one: it defines the windows size. 

```go
func (u *UI) Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int) {
	return 900, 900
}
```

`Draw` is responsible for drawing what is shown on the screen and runs for every frame. To represent our table, I had chosen to use 300x300 images for each one of the valid numbers. So, at each iteration, the functions look at the table and represent it visually. If the game is at the win state, it renders the congratulations image.

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

The `Update` function is responsible for updating the game itself. In other words, it makes something happen by defining the behavior when pressing any key. At the `core`, we mapped any action to the zero value, but when translating to a GUI, it makes more sense to reverse the logic. The user hits down because he wants to move the number below, not the zero.


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

We also define our function `Render`, so the `View` contract is followed.

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

And our game is ready! It was a really cool project to work on. It showed me that building games is always a great way to learn new things, and it is also a way to reinforce knowledge. Also, Ebiten allows us to build 2D games using our favorite language, and we can distribute it to several platforms, including Web with WebAssembly or even XBOX. If you wish to run the code, you can find it [here](https://github.com/mfbmina/puzzle). Please give me some feedback about this blog post, and I have one last question for you: which game do you want to build using Go?

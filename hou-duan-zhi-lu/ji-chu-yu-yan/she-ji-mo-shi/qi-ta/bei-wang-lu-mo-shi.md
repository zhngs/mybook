# 备忘录模式

### 一.go代码

```go
package memento

import "fmt"

type Memento interface{}

type Game struct {
	hp, mp int
}

type gameMemento struct {
	hp, mp int
}

func (g *Game) Play(mpDelta, hpDelta int) {
	g.mp += mpDelta
	g.hp += hpDelta
}

func (g *Game) Save() Memento {
	return &gameMemento{
		hp: g.hp,
		mp: g.mp,
	}
}

func (g *Game) Load(m Memento) {
	gm := m.(*gameMemento)
	g.mp = gm.mp
	g.hp = gm.hp
}

func (g *Game) Status() {
	fmt.Printf("Current HP:%d, MP:%d\n", g.hp, g.mp)
}
```

使用方法如下：

```go
package memento

func ExampleGame() {
	game := &Game{
		hp: 10,
		mp: 10,
	}

	game.Status()
	progress := game.Save()

	game.Play(-2, -3)
	game.Status()

	game.Load(progress)
	game.Status()

	// Output:
	// Current HP:10, MP:10
	// Current HP:7, MP:8
	// Current HP:10, MP:10
}
```

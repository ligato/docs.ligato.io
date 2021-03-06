# Hello World

---

Tutorial code: [Hello World][code-link]

In this tutorial, you will create a Ligato agent that 
contains a plugin called HelloWorld. This plugin prints "Hello World" to the log.

!!! Note
    The `agent` is a Ligato software component providing plugin life-cycle management functions. 

---

Let's start with the plugin. Every plugin must implement the `Plugin` interface
defined in the [cn-infra/infra](https://github.com/ligato/cn-infra/blob/master/infra/infra.go) package:

```go
type Plugin interface {
	// Init is called in the agent`s startup phase.
	Init() error
	// Close is called in the agent`s cleanup phase.
	Close() error
	// String returns unique name of the plugin.
	String() string
}
```

Implement the `Plugin` interface methods for the HelloWorld plugin:

```go
type HelloWorld struct{}

func (p *HelloWorld) String() string {
	return "HelloWorld"
}

func (p *HelloWorld) Init() error {
	log.Println("Hello World!")
	return nil
}

func (p *HelloWorld) Close() error {
	log.Println("Goodbye World!")
	return nil
}
```
Note that the `HelloWorld struct {}` is empty. Your plugin does not 
have any data, so all you need is an empty struct that supports the 
`Plugin` interface.

---

Some plugins require additional initialization after the base system is up. 
If required, you can optionally define the `AfterInit` method for your
plugin. This method will be executed after the `Init` method has been called for
all plugins. 

The `AfterInit()` method originates from the `PostInit` interface
defined in the cn-infra/infra package:

```go
type PostInit interface {
	// AfterInit is called once Init() of all plugins have returned without error.
	AfterInit() error
}
```

---

Next, create an instance of the HelloWorld plugin. Then, create 
a new agent and tell it about the HelloWorld plugin:

```go
func main() {
    	p := new(HelloWorld)    
    	a := agent.NewAgent(agent.Plugins(p))
    	// ...
}
```

You can use `agent` options to add the list of plugins to the agent at the agent's creation time. The code block above demonstrates how the  `agent.Plugins()` option adds an instance of the HelloWorld plugin to the agent.

`agent.AllPlugins` is another option to add the HelloWorld plugin instance to the agent. In addition, the `agent.AllPlugins()` option can add any dependencies that the HelloWorld plugin might have. Since your plugin has no dependencies, the simpler `agent.Plugins()` option will suffice.

---

Finally, start the agent using the agent's `Run()` method. This method will initialize the agent's plugins by calling their `Init()` and `AfterInit()` methods, and then wait for an interrupt from the user.


```go
if err := a.Run(); err != nil {
	log.Fatalln(err)
}
```
When an interrupt, such as `ctrl-c`, arrives from the user, the `Close()` method will be called on the agent's plugins, and the agent will exit.

---

**Run the Hello World tutorial code**

1. Open a terminal session.
<br>
<br>
2. Change to the 01_hello-world folder:
```
cn-infra git:(master) cd examples/tutorials/01_hello-world
```
3. Run code:
```
go run main.go
```
Example output:
```
INFO[0000] Starting agent version: v0.0.0-dev            BuildDate= CommitHash= loc="agent/agent.go(134)" logger=agent
2020/01/17 10:25:02 Hello World!
2020/01/17 10:25:02 All systems go!
INFO[0000] Agent started with 1 plugins (took 0s)        loc="agent/agent.go(179)" logger=agent
```

Example output after interrupt:

```
^CINFO[0030] Signal interrupt received, stopping.          loc="agent/agent.go(196)" logger=agent
INFO[0030] Stopping agent                                loc="agent/agent.go(269)" logger=agent
2020/01/17 10:25:32 Goodbye World!
INFO[0030] Agent stopped                                 loc="agent/agent.go(291)" logger=agent
```

[code-link]: https://github.com/ligato/cn-infra/tree/master/examples/tutorials/01_hello-world
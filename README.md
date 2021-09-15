![](https://avatars.githubusercontent.com/u/82073077?s=400&u=f51fd1a2c01103122f249b4539fafb2495a109b1&v=4)
# jarivs

1. 根据配置 `config{}` 生成对应的 `default.yml` 配置文件。 
2. 读取依次配置文件 `default.yml, config.yml` + `分支配置文件.yml` + `环境变量`
    + 根据 GitlabCI, 分支配置文件 `config.xxxx.yml`
    + 如没有 CI, 读取本地文件: `local.yml`

## requeire

1. config 对象中的结构体中， 使用 `env:""` tag 才能的字段才会被解析到 **default.yml** 中。 也只有这些字段才能通过 **配置文件** 或 **环境变量** 进行初始化赋值。

2. config 中的对象需要有  `SetDefaults()` 和 `Init()` 方法。
    + `SetDefaults` 方法用于结构体设置默认值
    + `Init` 方法用于根据默认值初始化


## example

```go
package main

import (
	"fmt"

	"github.com/gin-gonic/gin"
	"github.com/go-jarvis/jarvis"
)

type Server struct {
	Listen string `env:"addr"`
	Port   int    `env:"port"`

	engine *gin.Engine
}

func (s *Server) SetDefaults() {
	if s.Port == 0 {
		s.Port = 80
	}
}

func (s *Server) Init() {
	s.SetDefaults()

	if s.engine == nil {
		s.engine = gin.Default()
	}
}

func (s Server) Run() error {
	addr := fmt.Sprintf("%s:%d", s.Listen, s.Port)

	return s.engine.Run(addr)
}

func main() {
	server := &Server{}

	app := jarvis.App{
		Name: "Demo",
	}

	config := &struct {
		Server *Server
	}{
		Server: server,
	}
	// app.Save(config)

	app.Conf(config)
	// fmt.Println(config.Server.Port)

	server.Run()

}

```
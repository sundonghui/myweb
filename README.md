#Gem Web Framework
Gem is a web framework written in Golang. It features a martini-like API with much better performance, up to 40 times faster. If you need performance and good productivity, you will love Gem.

## Start using it
Run:

```
go get "github.com/sundonghui/gem"
```
Then import it in your Golang code:

```
import ""github.com/sundonghui/gem""
```


##API Examples

#### Create most basic PING/PONG HTTP endpoint
```go 
import "github.com/sundonghui/gem"

func main() {
    r := gem.Default()
    r.GET("ping", func(c *gem.Context){
        c.String("pong")
    })
    
    // Listen and server on 0.0.0.0:8080
    r.Run(":80")
}
```

#### Using GET, POST, PUT, PATCH and DELETE

```go
func main() {
    // Creates a gem router + logger and recovery (crash-free) middlwares
    r := gem.Default()
    
    r.GET("/someGet", getting)
    r.POST("/somePost", posting)
    r.PUT("/somePut", putting)
    r.DELETE("/someDelete", deleting)
    r.PATCH("/somePATCH", patching)

    // Listen and server on 0.0.0.0:8080
    r.Run(":8080")
}
```

#### Parameters in path

```go
func main() {
    r := gem.Default()
    
    r.GET("/user/:name", func(c *gem.Context) {
        name := c.Params.ByName("name")
        message := "Hello "+name
        c.String(200, message)
    })
}
```


#### Grouping routes
```go
func main() {
    r := gem.Default()
    
    // Simple group: v1
    v1 := r.Group("/v1")
    {
        v1.POST("/login", loginEndpoint)
        v1.POST("/submit", submitEndpoint)
        v1.POST("/read", readEndpoint)
    }
    
    // Simple group: v1
    v2 := r.Group("/v2")
    {
        v2.POST("/login", loginEndpoint)
        v2.POST("/submit"", submitEndpoint)
        v2.POST("/read"", readEndpoint)
    }

    // Listen and server on 0.0.0.0:8080
    r.Run(":8080")
}
```


#### Blank Gem without middlewares by default

Use

```go
r := gem.New()
```
instead of

```go
r := gem.Default()
```


#### Using middlewares
```go
func main() {
    // Creates a router without any middlware by default
    r := gem.New()
    
    // Global middlwares
    r.Use(gem.Logger())
    r.Use(gem.Recovery())
    
    // Per route middlwares, you can add as many as you desire.
    r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

    // Authorization group
    // authorized := r.Group("/", AuthRequired())
    // exactly the same than:
    authorized := r.Group("/")
    // per group middlwares! in this case we use the custom created
    // AuthRequired() middlware just in the "authorized" group.
    authorized.Use(AuthRequired())
    {
        authorized.Use.POST("/login", loginEndpoint)
        authorized.Use.POST("/submit", submitEndpoint)
        authorized.Use.POST("/read", readEndpoint)
        
        // nested group
        testing := authorized.Group("testing")
        testing.GET("/analytics", analyticsEndpoint)
    }
   
    // Listen and server on 0.0.0.0:8080
    r.Run(":8080")
}
```


#### JSON parsing and validation

```go
type LoginJSON struct {
    User     string `json:"user" binding:"required"`
    Password string `json:"password" binding:"required"`
}

func main() {
    r := gem.Default()
    
    r.POST("/login", func(c *gem.Context) {
        var json LoginJSON
        
        // If EnsureBody returns false, it will write automatically the error
        // in the HTTP stream and return a 400 error. If you want custom error 
        // handling you should use: c.ParseBody(interface{}) error
        if c.EnsureBody(&json) {
            if json.User=="manu" && json.Password=="123" {
                c.JSON(200, gem.H{"status": "you are logged in"})
            }else{
                c.JSON(401, gem.H{"status": "unauthorized"})
            }
        }
    })
}
```

#### XML, and JSON rendering

```go
func main() {
    r := gem.Default()
    
    // gem.H is a shortcup for map[string]interface{}
    r.GET("/someJSON", func(c *gem.Context) {
        c.JSON(200, gem.H{"message": "hey", "status": 200})
    })
    
    r.GET("/moreJSON", func(c *gem.Context) {
        // You also can use a struct
        var msg struct {
            Message string
            Status  int
        }
        msg.Message = "hey"
        msg.Status = 200
        c.JSON(200, msg.Status)
    })
    
    r.GET("/someXML", func(c *gem.Context) {
        c.XML(200, gem.H{"message": "hey", "status": 200})
    })
}
```


####HTML rendering

Using LoadHTMLTemplates()

```go
func main() {
    r := gem.Default()
    r.LoadHTMLTemplates("templates/*")
    r.GET("index", func(c *gem.Context) {
        obj := gem.h{"title": "Main website"}
        c.HTML(200, "templates/index.tmpl", obj)
    })
}
```

You can also use your own html template render

```go
import "html/template"
func main() {
    r := gem.Default()
    html := template.ParseFiles("file1", "file2")
    r.HTMLTemplates = html
}
```


#### Custom Middlewares

```go
func Logger() gem.HandlerFunc {
    return func(c *gem.Context) {
        t : time.Now()
        
        // Set example variable
        c.Set("example", "12345")
        
        // before request
        
        c.Next()
        
        // after request
        latency := time.Since(t)
        log.Print(latency)
    }
}

func main() {
    r := gem.New()
    r.Use(Logger())
    
    r.GET("test", func(c *gem.Context){
        example := r.Get("example").(string)
        
        // it would print: "12345"
        log.Println(example)
    })
}
```




#### Custom HTTP configuration

Use `http.ListenAndServe()` directly, like this:

```go
func main() {
    router := gem.Default()
    http.ListenAndServe(":8080", router)
}
```
or

```go
func main() {
    router := gem.Default()

    s := &http.Server{
	    Addr:           ":8080",
	    Handler:        router,
	    ReadTimeout:    10 * time.Second,
	    WriteTimeout:   10 * time.Second,
	    MaxHeaderBytes: 1 << 20,
    }
    s.ListenAndServe()
}
```
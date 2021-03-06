# 垃圾代码书写准则

[![State-of-the-art Shitcode](https://img.shields.io/static/v1?label=State-of-the-art&message=Shitcode&color=7B5804)](https://github.com/trekhleb/state-of-the-art-shitcode)

## 不要处理错误

```go
//这怎么可能出错误所以 不需要判断
    var u models.Userinfo
    con, _ := Upgrade.Upgrade(c.Writer, c.Request, nil)
    _, msg, _ := con.ReadMessage()
    json.Unmarshal(msg, &u)
//浪费时间
    var user models.Userinfo
    con, err := Upgrade.Upgrade(c.Writer, c.Request, nil)
    _, msg, err := con.ReadMessage()
    err = json.Unmarshal(msg, &user)
    if err != nil {
        return
    }
```

## 高深的变量名

```go
//高人的变量名 自己也不知到表示了啥
vcons := &Client{con: con, info: usr}
    err := vcons.Menu()
    if err != nil {
        con.Close()
        return
    }
//一下就被猜出来的
    client := &Client{con: con, info: user}
    err := client.Menu()
    if err != nil {
        return
    }
```

## 魔数

```go
// 谁也不知道1-4 是表示啥
switch choose {
    case byte('1'):
        c.NMatches()
        break
    case byte('2'):
        c.Enter()
        break
    case byte('3'):
        c.Watch()
        break
    case byte('4'):

// 小秘密就被发现了
var (
    ChooseNewMatch   = byte('1')
    ChooseEnterMatch = byte('2')
    ChooseWatchMatch = byte('3')
    ChooseQuit       = byte('4')
)
switch choose {
    case ChooseNewMatch:
        c.NMatches()
        break
    case ChooseEnterMatch:
        c.Enter()
        break
    case ChooseWatchMatch:
        c.Watch()
        break
    case CHooseQuit:
        c.con.Close()
////
```

## 长续航

```go
// 未对接受消息进行封装 以及一些预判断


func (r *Room) Game() {
    //初始化返回棋盘 以及初始化用户侧信息
    var s []rune
    m := ChessTable()
    r.Info = m
    s = r.ReturnDate(m)
    table := NewLogic(m)
    //开始对局
    r.Table = table
    r.Client[0].con.WriteMessage(websocket.TextMessage, []byte(string(s)))
    r.Client[1].con.WriteMessage(websocket.TextMessage, []byte(string(s)))
    //
    //r.Client[0].Fal = false
    r.Client[0].Room = r
    r.Client[1].Room = r
    go r.Broad()
    for {
        r.Client[0].con.WriteMessage(websocket.TextMessage, []byte(string(s)))
        _, msg, _ := r.Client[0].con.ReadMessage()
        for {
            if strings.HasPrefix(string(msg), "msg") {
                chatmsg := strings.TrimPrefix(string(msg), "msg")
                r.Client[1].con.WriteMessage(websocket.TextMessage, []byte(chatmsg))
                _, msg, _ = r.Client[0].con.ReadMessage()
                continue
            }
            //判断用户直到输入正确信息
            ok, err := r.Junge(msg, 0)
            if err == nil {
                break
            }
            if ok {
                r.Client[0].con.WriteMessage(websocket.TextMessage, []byte("你赢了"))
                r.Client[1].con.WriteMessage(websocket.TextMessage, []byte("你输了"))
                return
            }
            r.Client[0].con.WriteMessage(websocket.TextMessage, []byte(err.Error()))
            _, msg, _ = r.Client[0].con.ReadMessage()
        }
        //b, _ := WinOr(0, r)
        s = r.ReturnDate(m)
        //if b {
        //    s = []rune("你输了")
        //}
        fmt.Println(s, string(m[0][0]))
        r.Msg <- string(s)
        r.Client[1].con.WriteMessage(websocket.TextMessage, []byte(string(s)))
        _, msg, _ = r.Client[1].con.ReadMessage()

        for {
            if strings.HasPrefix(string(msg), "msg") {
                chatmsg := strings.TrimPrefix(string(msg), "msg")
                r.Client[0].con.WriteMessage(websocket.TextMessage, []byte(chatmsg))
                _, msg, _ = r.Client[1].con.ReadMessage()
                continue

            }
            ok, err := r.Junge(msg, 1)
            if err == nil {
                break
            }
            if ok {
                r.Client[0].con.WriteMessage(websocket.TextMessage, []byte("你输了"))
                r.Client[1].con.WriteMessage(websocket.TextMessage, []byte("你赢了"))
            }
            r.Client[1].con.WriteMessage(websocket.TextMessage, []byte(err.Error()))
            _, msg, _ = r.Client[1].con.ReadMessage()
        }
        //b, _ = WinOr(1, r)

        s = r.ReturnDate(m)
        r.Msg <- string(s)

    }
}
```

## 分包不规范

```go
文件夹 PATH 列表
卷序列号为 A4BA-F779
│      
├─api // 该api中包含model 层的定义 当时图省事 没有进行迁移
│     // 该文件中同时包括逻辑层 把应该放到service 的文件放在了这里
│      acess.go  
│      center.go
│      match.go
│      menu.go
│      plateinit.go
│      user.go
│      
├─controller
│      router.go
│      
├─dao     //dao 层 将本应属于本层的逻辑 例如注册逻辑 移出去了
│      room.go
│      sqlinit.go
│      
├─hander  //该层多余 没有先进行项目规划 和目录结构分层 
│      readmsg.go
│      sendmsg.go
│      
├─models // models 存在多余文件 
│      chessman.go
│      mmodels.go
│      requst.go
│      umodels.go
│      
├─service     //该层属于空层了
│      user.go
│      
└─utils
        img.png
        token.go


//同时 没有对websockt 进行良好的封装
```

## 不写注释

```go
全篇极少注释 没有对一些混乱的代码进行说明
```

## 代码幽灵

```go
当发现某个函数 或者某个语句块逻辑存在问题时就直接将其注释掉而不是直接删除 
或许觉得后面可能还会用到 但其实并没有用上
```

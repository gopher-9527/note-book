# klog自定义输出到文件

在项目中，有时候我们需要将不同错误类型输出到文件，进行日志分类和处理。klog本身没有提供这样的能力，但是klog提供了自定义 output 的方法，所以我们可以通过自己实现 writer 的方式自定义输出。

## 自定义 writer

我们需要将 info、warn、error、fatal 的日志输出到不同的文件，所以定义了下面的 writer：

```go
type LogWriter struct {
	mu       sync.Mutex
	infoLog  *os.File
	warnLog  *os.File
	errorLog *os.File
	fatalLog *os.File
}
```

然后怎么实现 write 方法呢？我们需要区分输出的日志级别，然后将其输出到对应的文件。先看下 klog 输出日志的方式。 klog 会为日志生成日志级别的前缀：

![img](https://cdn.nlark.com/yuque/0/2024/png/115938/1731642420839-aeca3d34-e22f-46a0-88ca-ce9c736591c1.png)

[代码实现](https://github.com/kubernetes/klog/blob/fa118f5f4770fcbb04cc83941789cef9511dfa72/internal/buffer/buffer.go#L133)， 其中 [Char 的定义](https://github.com/kubernetes/klog/blob/fa118f5f4770fcbb04cc83941789cef9511dfa72/internal/severity/severity.go#L39)如下：

```go
const Char = "IWEF"
```

我们可以通过最后输出的日志的第一个字符确定日志级别，实现的 writer 如下：

```go
func (w *LogWriter) Write(p []byte) (n int, err error) {
	w.mu.Lock()
	defer w.mu.Unlock()
	// Write all logs to respective files
	switch string(p[:1]) {
	case "I": // Info log starts with "I"
		return w.infoLog.Write(p)
	case "W": // Warning log starts with "W"
		return w.warnLog.Write(p)
	case "E": // Error log starts with "E"
		return w.errorLog.Write(p)
	case "F": // Fatal log starts with "F"
		return w.fatalLog.Write(p)
	default: // Default write logs to info
		return w.infoLog.Write(p)
	}
}
```

最后，将实现的 witer 设置为 klog 的输出：

```go
// 设置输出
klog.SetOutput(writer)

// 设置其他参数
klog.InitFlags(nil)
klog.LogToStderr(false)
if err := flag.Set("one_output", "true"); err != nil {
    panic(err)
}

if err := flag.Set("logtostderr", "false"); err != nil {
    panic(err)
}

if err := flag.Set("alsologtostderr", "true"); err != nil {
    panic(err)
}
```

## 完整代码

```go
package logger

import (
    "flag"
    "os"
    "sync"

    "k8s.io/klog/v2"
)

// LogWriter wraps file descriptors for different log levels
type LogWriter struct {
    mu       sync.Mutex
    infoLog  *os.File
    warnLog  *os.File
    errorLog *os.File
    fatalLog *os.File
}

var writer *LogWriter

func (w *LogWriter) Write(p []byte) (n int, err error) {
    w.mu.Lock()
    defer w.mu.Unlock()
    // Write all logs to respective files
    switch string(p[:1]) {
        case "I": // Info log starts with "I"
        return w.infoLog.Write(p)
        case "W": // Warning log starts with "W"
        return w.warnLog.Write(p)
        case "E": // Error log starts with "E"
        return w.errorLog.Write(p)
        case "F": // Fatal log starts with "F"
        return w.fatalLog.Write(p)
        default: // Default write logs to info
        return w.infoLog.Write(p)
    }
}

func Close() {
    _ = writer.infoLog.Close()
    _ = writer.warnLog.Close()
    _ = writer.errorLog.Close()
    _ = writer.fatalLog.Close()
}

func Setup(logdir string) {
    if logdir == "" {
        logdir = "./log"
    }
    err := os.Mkdir(logdir, 0777)
    if err != nil && !os.IsExist(err) {
        panic(err)
    }
    // Open log files
    infoLog, err := os.OpenFile(logdir+"/info.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        panic(err)
    }
    warnLog, err := os.OpenFile(logdir+"/warn.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        panic(err)
    }
    errorLog, err := os.OpenFile(logdir+"/error.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        panic(err)
    }
    fatalLog, err := os.OpenFile(logdir+"/fatal.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        panic(err)
    }

    writer = &LogWriter{
        infoLog:  infoLog,
        warnLog:  warnLog,
        errorLog: errorLog,
        fatalLog: fatalLog,
    }

    // Set klog output to custom writer
    klog.SetOutput(writer)
    klog.InitFlags(nil)
    klog.LogToStderr(false)
    if err := flag.Set("one_output", "true"); err != nil {
        panic(err)
    }

    if err := flag.Set("logtostderr", "false"); err != nil {
        panic(err)
    }

    if err := flag.Set("alsologtostderr", "true"); err != nil {
        panic(err)
    }
}
```
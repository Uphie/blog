---
title: bad file descriptor
author: Uphie
date: 2022-12-22 21:13:00 +0800
categories: [技术]
tags: [go,文件描述符]
math: true
toc: true
---


web 后端程序启动时，日志打印部分出错：
```
zerolog: could not write event: write logs/broker.log: bad file descriptor
```

原因在于 zerologger.Logger 创建时使用的 file 对象，如下：
```go
func GetLogger(logDir, logName string) *zerolog.Logger {
	zerolog.TimeFieldFormat = zerolog.TimeFormatUnixMs
	zerolog.ErrorStackMarshaler = pkgerrors.MarshalStack
	zerolog.TimestampFieldName = "t"
	zerolog.LevelFieldName = "l"

	// 创建目录
	logDir = strings.ReplaceAll(logDir, " ", "")
	logDir = strings.TrimRight(logDir, "/")
	if logDir == "" {
		log.Fatal("log dir is invalid")
	}
	if _, err := os.Stat(logDir); os.IsNotExist(err) {
		// create log dir
		err = os.MkdirAll(logDir, 0o666)
		if err != nil {
			log.Fatal(err)
		}
	}

	logName = strings.ReplaceAll(logName, " ", "")
	if logName == "" {
		logName = "broker.log"
	} else if !strings.HasSuffix(logName, ".log") {
		logName = logName + ".log"
	}

	logPath := fmt.Sprintf("%s/%s", logDir, logName)
	//var err error
	var file *os.File
	if _, err := os.Stat(logPath); os.IsNotExist(err) {
		// create log file
		file, err = os.Create(logPath)
		if err != nil {
			log.Fatal(err)
		}
	} else {
		// open log file
		file, err = os.Open(logPath)
		if err != nil {
			log.Fatal(err)
		}
	}
	logger := zerolog.New(file)
	return &logger
}
```

在日志文件不存在时，file 是由 `os.Open()` 得来，但 `os.Open` 实际上实现如下：
```go
// Open opens the named file for reading. If successful, methods on
// the returned file can be used for reading; the associated file
// descriptor has mode O_RDONLY.
// If there is an error, it will be of type *PathError.
func Open(name string) (*File, error) {
	return OpenFile(name, O_RDONLY, 0)
}
```
可以看到，返回的 file 只读，拿去写就会有问题。

再来说下文件描述符（file descriptor），文件描述符是一个标识计算机操作系统中打开的文件（广义上的文件，下同）的唯一非负整数。它描述了一个数据源，以及数据源怎样被访问。

每个被打开的文件都至少有一个文件描述符，可以同时有多个文件描述符。

当一个进程成功打开一个文件描述符，操作系统内核返回一个文件描述符，该文件描述符指向一个全局文件表的项。文件表向包含被打开文件的 inode、字节偏移量，以及数据流的访问限制（只读、只写，等等）。

[![文件描述符.png](https://www.computerhope.com/jargon/f/file-descriptor.jpg)](https://www.computerhope.com/jargon/f/file-descriptor.jpg)


言归正传，解决方法就是把文件打开方式修改一下：

将 `file, err = os.Open(logPath)` 修改为 `file, err = os.OpenFile(logPath, os.O_RDWR, 0)` 即可。

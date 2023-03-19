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
可以看到，返回的 file 关联的文件描述符是只读模式！不用于写，所以才 `bad file descriptor`。

将 `file, err = os.Open(logPath)` 修改为 `file, err = os.OpenFile(logPath, os.O_RDWR, 0)` 即可。

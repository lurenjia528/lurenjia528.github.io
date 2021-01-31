---
title: cobra与glog冲突
slug: chinese-test
date: 2020-04-07
categories:
- ubuntu
- go
tags:
- go
- cobra
- glog
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
cobra与glog冲突
<!--more-->



# cobra glog

cobra 和glog的flags会冲突

解决：
```go

func main() {
	cmd.Execute()
}

func init(){
	pflag.CommandLine.AddGoFlagSet(flag.CommandLine)
	flag.Set("logtostderr","true")
	flag.Parse()
}
```

``` go
func init() {
	if os.Args[1] == "-h" || os.Args[1] == "--help" {
		_ = rootCmd.Help()
		os.Exit(1)
	}
	//cobra.OnInitialize(initConfig)
	rootCmd.AddCommand(image.BatchCmd)
}
```

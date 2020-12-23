# crd
## 准备工作
- 创建项目crd
- 在项目下创建pkg/apis/foo/v1目录
- 在pkg/apis/foo下创建register.go
- 在pkg/apis/foo/v1下创建doc.go, types.go, register.go
- copy https://github.com/kubernetes/code-generator.git到$GOPATH/src/k8s.io下(我用的是release-1.16分支。如果需要特定的分支则先切换分支再继续)
    1. cd 到cmd/client-gen,生成client-go并copy到【与当前项目平级的位置】
    2. cd 到cmd/deepcopy-gen，生成deepcopy-gen并copy到【与当前项目平级的位置】
    3. cd 到cmd/lister-gen，生成lister-gen并copy到【与当前项目平级的位置】
    4. cd 到cmd/informer-gen，生成informer-gen并copy【到与当前项目平级的位置】
    5. cd 到hack，copy boilerplate.go.txt 到【项目根目录下】

## 生成
- ./deepcopy-gen --go-header-file crd/boilerplate.go.txt --input-dirs crd/pkg/apis/foo/v1 --output-package crd/pkg/client
- ./client-gen --go-header-file crd/boilerplate.go.txt --input-dirs crd/pkg/apis/foo/v1 --output-package crd/pkg/client
- ./lister-gen --go-header-file crd/boilerplate.go.txt --input-dirs crd/pkg/apis/foo/v1 --output-package crd/pkg/client
- ./informer-gen --go-header-file crd/boilerplate.go.txt --input-dirs crd/pkg/apis/foo/v1 --output-package crd/pkg/client

## 成果
````
[root@main crd]# tree
.
├── boilerplate.go.txt
├── go.mod
├── go.sum
├── pkg
│   ├── apis
│   │   └── foo
│   │       ├── register.go
│   │       └── v1
│   │           ├── deepcopy_generated.go
│   │           ├── doc.go
│   │           ├── register.go
│   │           └── types.go
│   └── client
│       ├── externalversions
│       │   ├── factory.go
│       │   ├── foo
│       │   │   ├── interface.go
│       │   │   └── v1
│       │   │       ├── interface.go
│       │   │       └── student.go
│       │   ├── generic.go
│       │   └── internalinterfaces
│       │       └── factory_interfaces.go
│       ├── foo
│       │   └── v1
│       │       ├── expansion_generated.go
│       │       └── student.go
│       └── internalclientset
│           ├── clientset.go
│           ├── doc.go
│           ├── fake
│           │   ├── clientset_generated.go
│           │   ├── doc.go
│           │   └── register.go
│           └── scheme
│               ├── doc.go
│               └── register.go
└── README.md

14 directories, 24 files
````

## FAQ
 1. why need boilerplate.go.txt ?
 code-generator的client-gen, deepcopy-gen, lister-gen, informer-gen在启动的时候默认会读取$Projetc/hack目录下的boilerplate.go.txt文件，因此在上述的准备工作中把code-generator/hack/boilerplate.go.txt文件copy到了当前目录下，并在生成模块，通过--go-header-file指定了boilerplate.go.txt文件位置。
 同样也可以参考https://github.com/kubernetes/sample-controller中的hack/update-codegen.sh脚本里面也有类似的说明。
    ````
    F1223 11:22:28.005533    6641 deepcopy.go:131] Failed loading boilerplate: open k8s.io/code-generator/hack/boilerplate.go.txt: no such file or directory
    goroutine 1 [running]:
    k8s.io/klog/v2.stacks(0xc00000e001, 0xc000218000, 0x99, 0xe9)
        D:/go/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:1026 +0xb9
    k8s.io/klog/v2.(*loggingT).output(0x99c620, 0xc000000003, 0x0, 0x0, 0xc00010f960, 0x9776b5, 0xb, 0x83, 0x0)
        D:/go/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:975 +0x19b
    k8s.io/klog/v2.(*loggingT).printf(0x99c620, 0xc000000003, 0x0, 0x0, 0x0, 0x0, 0x78a150, 0x1e, 0xc000042990, 0x1, ...)
        D:/go/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:750 +0x191
    k8s.io/klog/v2.Fatalf(...)
        D:/go/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:1502
    k8s.io/gengo/examples/deepcopy-gen/generators.Packages(0xc00010f8f0, 0xc00007ef00, 0x76774c, 0x6, 0xc00010f8f0)
        D:/go/pkg/mod/k8s.io/gengo@v0.0.0-20201113003025-83324d819ded/examples/deepcopy-gen/generators/deepcopy.go:131 +0x135
    k8s.io/gengo/args.(*GeneratorArgs).Execute(0xc00007ef00, 0xc00003de38, 0x76774c, 0x6, 0x795040, 0x0, 0x0)
        D:/go/pkg/mod/k8s.io/gengo@v0.0.0-20201113003025-83324d819ded/args/args.go:206 +0x1a9
    main.main()
        D:/go/src/github.com/kubernetes/code-generator/cmd/deepcopy-gen/main.go:77 +0x4ce
    
    goroutine 6 [chan receive]:
    k8s.io/klog/v2.(*loggingT).flushDaemon(0x99c620)
        D:/go/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:1169 +0x8b
    created by k8s.io/klog/v2.init.0
        D:/go/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:417 +0xdf
    ````
 2. why code-generator binary can't be put in crd project ?

    if code-generator binary is in crd project, when you exec the binary like this:
    ````
    [root@main crd]# ./deepcopy-gen --go-header-file ./boilerplate.go.txt --input-dirs pkg/apis/foo/v1 --output-package pkg/client
      F1223 14:29:16.195241    3092 main.go:82] Error: Failed making a parser: unable to add directory "pkg/apis/foo/v1": unable to import "pkg/apis/foo/v1": go/build: importGo pkg/apis/foo/v1: exit status 1
      can't load package: package pkg/apis/foo/v1: malformed module path "pkg/apis/foo": missing dot in first path element
    
    or
    
    [root@main crd]# ./deepcopy-gen --go-header-file crd/boilerplate.go.txt --input-dirs crd/pkg/apis/foo/v1 --output-package crd/pkg/client
    F1223 14:26:26.236143   28472 main.go:82] Error: Failed making a parser: unable to add directory "crd/pkg/apis/foo/v1": unable to import "crd/pkg/apis/foo/v1": go/build: importGo crd/pkg/apis/foo/v1: exit status 1
    can't load package: package crd/pkg/apis/foo/v1: malformed module path "crd/pkg/apis/foo": missing dot in first path element
    ````
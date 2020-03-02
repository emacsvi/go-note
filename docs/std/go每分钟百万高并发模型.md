---
title: "go每分钟百万高并发模型"
slug: "handling-1-million-requests-per-minute-with-golang"
date: "2018-11-23 01:00:05"
categories:
    - go
tags:
    - go
---

# 使用Go语言每分钟处理1百万请求（译）
---

 在[Malwarebytes ][1]我们经历了显著的增长，自从我一年前加入了硅谷的公司，一个主要的职责成了设计架构和开发一些系统来支持一个快速增长的信息安全公司和所有需要的设施来支持一个每天百万用户使用的产品。我在反病毒和反恶意软件行业的不同公司工作了12年，从而我知道由于我们每天处理大量的数据，这些系统是多么复杂。

 有趣的是，在过去的大约9年间，我参与的所有的web后端的开发通常是通过Ruby on Rails技术实现的。不要错怪我。我喜欢Ruby on Rails，并且我相信它是个令人惊讶的环境。但是一段时间后，你会开始以ruby的方式开始思考和设计系统，你会忘记，如果你可以利用多线程、并行、快速执行和小内存开销，软件架构本来应该是多么高效和简单。很多年期间，我是一个`c/c++`、Delphi和c#开发者，我刚开始意识到使用正确的工具可以把复杂的事情变得简单些。

 作为首席架构师，我不会很关心在互联网上的语言和框架战争。我相信效率、生产力。代码可维护性主要依赖于你如何把解决方案设计得很简单。

# 问题

当工作在我们的匿名遥测和分析系统中，我们的目标是可以处理来自于百万级别的终端的大量的POST请求。web处理服务可以接收包含了很多payload的集合的JSON数据，这些数据需要写入Amazon S3中。接下来，map-reduce系统可以操作这些数据。

按照习惯，我们会调研服务层级架构，涉及的软件如下：

- Sidekiq
- Resque
- DelayedJob
- Elasticbeanstalk Worker Tier
- RabbitMQ
- and so on…

搭建了2个不同的集群，一个提供web前端，另外一个提供后端处理，这样我们可以横向扩展后端服务的数量。

但是，从刚开始，在 讨论阶段我们的团队就知道我们应该使用Go，因为我们看到这会潜在性地成为一个非常庞大（ large traffic）的系统。我已经使用了Go语言大约2年时间，我们开发了几个系统，但是很少会达到这样的负载（amount of load）。

我们开始创建一些结构，定义从POST调用得到的web请求负载，还有一个上传到S3 budket的函数。

```go
type PayloadCollection struct {
    WindowsVersion  string    `json:"version"`
    Token           string    `json:"token"`
    Payloads        []Payload `json:"data"`
}

type Payload struct {
    // [redacted]
}

func (p *Payload) UploadToS3() error {
    // the storageFolder method ensures that there are no name collision in
    // case we get same timestamp in the key name
    storage_path := fmt.Sprintf("%v/%v", p.storageFolder, time.Now().UnixNano())

    bucket := S3Bucket

    b := new(bytes.Buffer)
    encodeErr := json.NewEncoder(b).Encode(payload)
    if encodeErr != nil {
        return encodeErr
    }

    // Everything we post to the S3 bucket should be marked 'private'
    var acl = s3.Private
    var contentType = "application/octet-stream"

    return bucket.PutReader(storage_path, b, int64(b.Len()), contentType, acl, s3.Options{})
}
```

# 本地Go routines方法

刚开始，我们采用了一个非常本地化的POST处理实现，仅仅尝试把发到简单go routine的job并行化：

```
func payloadHandler(w http.ResponseWriter, r *http.Request) {

    if r.Method != "POST" {
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }

    // Read the body into a string for json decoding
    var content = &PayloadCollection{}
    err := json.NewDecoder(io.LimitReader(r.Body, MaxLength)).Decode(&content)
    if err != nil {
        w.Header().Set("Content-Type", "application/json; charset=UTF-8")
        w.WriteHeader(http.StatusBadRequest)
        return
    }

    // Go through each payload and queue items individually to be posted to S3
    for _, payload := range content.Payloads {
        go payload.UploadToS3()   // <----- DON'T DO THIS
    }

    w.WriteHeader(http.StatusOK)
}
```

对于中小负载，这会对大多数的人适用，但是大规模下，这个方案会很快被证明不是很好用。我们期望的请求数，不在我们刚开始计划的数量级，当我们把第一个版本部署到生产环境上。我们完全低估了流量。

上面的方案在很多地方很不好。没有办法控制我们产生的go routine的数量。由于我们收到了每分钟1百万的POST请求，这段代码很快就崩溃了。

# 再次尝试

我们需要找一个不同的方式。自开始我们就讨论过，  我们需要保持请求处理程序的生命周期很短，并且进程在后台产生。当然，这是你在Ruby on Rails的世界里必须要做的事情，否则你会阻塞在所有可用的工作 web处理器上，不管你是使用puma、unicore还是passenger（我们不要讨论JRuby这个话题）。然后我们需要利用常用的处理方案来做这些，比如Resque、 Sidekiq、 SQS等。这个列表会继续保留，因为有很多的方案可以实现这些。

所以，第二次迭代，我们创建了一个缓冲channel，我们可以把job排队，然后把它们上传到S3。因为我们可以控制我们队列中的item最大值，我们有大量的内存来排列job，我们认为只要把job在channel里面缓冲就可以了。

```go
var Queue chan Payload

func init() {
    Queue = make(chan Payload, MAX_QUEUE)
}

func payloadHandler(w http.ResponseWriter, r *http.Request) {
    ...
    // Go through each payload and queue items individually to be posted to S3
    for _, payload := range content.Payloads {
        Queue <- payload
    }
    ...
}
```

接下来，我们再从队列中取job，然后处理它们。我们使用类似于下面的代码：

```go
func StartProcessor() {
    for {
        select {
        case job := <-Queue:
            job.payload.UploadToS3()  // <-- STILL NOT GOOD
        }
    }
}
```

说实话，我不知道我们在想什么。这肯定是一个满是Red-Bulls的夜晚。这个方法不会带来什么改善，我们用了一个 有缺陷的缓冲队列并发，仅仅是把问题推迟了。我们的同步处理器同时仅仅会上传一个数据到S3，因为来到的请求远远大于单核处理器上传到S3的能力，我们的带缓冲channel很快达到了它的极限，然后阻塞了请求处理逻辑的queue更多item的能力。

我们仅仅避免了问题，同时开始了我们的系统挂掉的倒计时。当部署了这个有缺陷的版本后，我们的延时保持在每分钟以常量增长。

![此处输入图片的描述][2]

# 最好的解决方案

我们讨论过在使用用Go channel时利用一种常用的模式，来创建一个二级channel系统，一个来queue job，另外一个来控制使用多少个worker来并发操作JobQueue。

想法是，以一个恒定速率并行上传到S3，既不会导致机器崩溃也不好产生S3的连接错误。这样我们选择了创建一个Job/Worker模式。对于那些熟悉Java、C#等语言的开发者，可以把这种模式想象成利用channel以golang的方式来实现了一个worker线程池，作为一种替代。

```go
var (
    MaxWorker = os.Getenv("MAX_WORKERS")
    MaxQueue  = os.Getenv("MAX_QUEUE")
)

// Job represents the job to be run
type Job struct {
    Payload Payload
}

// A buffered channel that we can send work requests on.
var JobQueue chan Job

// Worker represents the worker that executes the job
type Worker struct {
    WorkerPool  chan chan Job
    JobChannel  chan Job
    quit        chan bool
}

func NewWorker(workerPool chan chan Job) Worker {
    return Worker{
        WorkerPool: workerPool,
        JobChannel: make(chan Job),
        quit:       make(chan bool)}
}

// Start method starts the run loop for the worker, listening for a quit channel in
// case we need to stop it
func (w Worker) Start() {
    go func() {
        for {
            // register the current worker into the worker queue.
            w.WorkerPool <- w.JobChannel

            select {
            case job := <-w.JobChannel:
                // we have received a work request.
                if err := job.Payload.UploadToS3(); err != nil {
                    log.Errorf("Error uploading to S3: %s", err.Error())
                }

            case <-w.quit:
                // we have received a signal to stop
                return
            }
        }
    }()
}

// Stop signals the worker to stop listening for work requests.
func (w Worker) Stop() {
    go func() {
        w.quit <- true
    }()
}
```

我们已经修改了我们的web请求handler，用payload创建一个Job实例，然后发到JobQueue channel，以便于worker来获取。

```go
func payloadHandler(w http.ResponseWriter, r *http.Request) {

    if r.Method != "POST" {
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }

    // Read the body into a string for json decoding
    var content = &PayloadCollection{}
    err := json.NewDecoder(io.LimitReader(r.Body, MaxLength)).Decode(&content)
    if err != nil {
        w.Header().Set("Content-Type", "application/json; charset=UTF-8")
        w.WriteHeader(http.StatusBadRequest)
        return
    }

    // Go through each payload and queue items individually to be posted to S3
    for _, payload := range content.Payloads {

        // let's create a job with the payload
        work := Job{Payload: payload}

        // Push the work onto the queue.
        JobQueue <- work
    }

    w.WriteHeader(http.StatusOK)
}
```

在web server初始化时，我们创建一个Dispatcher，然后调用Run()函数创建一个worker池子，然后开始监听JobQueue中的job。

```go
dispatcher := NewDispatcher(MaxWorker)
dispatcher.Run()
```

下面是dispatcher的实现代码：

```go
type Dispatcher struct {
    // A pool of workers channels that are registered with the dispatcher
    WorkerPool chan chan Job
}

func NewDispatcher(maxWorkers int) *Dispatcher {
    pool := make(chan chan Job, maxWorkers)
    return &Dispatcher{WorkerPool: pool}
}

func (d *Dispatcher) Run() {
    // starting n number of workers
    for i := 0; i < d.maxWorkers; i++ {
        worker := NewWorker(d.pool)
        worker.Start()
    }

    go d.dispatch()
}

func (d *Dispatcher) dispatch() {
    for {
        select {
        case job := <-JobQueue:
            // a job request has been received
            go func(job Job) {
                // try to obtain a worker job channel that is available.
                // this will block until a worker is idle
                jobChannel := <-d.WorkerPool

                // dispatch the job to the worker job channel
                jobChannel <- job
            }(job)
        }
    }
}
```

注意到，我们提供了初始化并加入到池子的worker的最大数量。因为这个工程我们利用了Amazon Elasticbeanstalk带有的docker化的Go环境，所以我们常常会遵守[12-factor][3]方法论来配置我们的生成环境中的系统，我们从环境变了读取这些值。这种方式，我们控制worker的数量和JobQueue的大小，所以我们可以很快的改变这些值，而不需要重新部署集群。

```go
var (
    MaxWorker = os.Getenv("MAX_WORKERS")
    MaxQueue  = os.Getenv("MAX_QUEUE")
)
```

# 直接结果

我们部署了之后，立马看到了延时降到微乎其微的数值，并未我们处理请求的能力提升很大。

![此处输入图片的描述][4]

Elastic Load Balancers完全启动后，我们看到ElasticBeanstalk 应用服务于每分钟1百万请求。通常情况下在上午时间有几个小时，流量峰值超过每分钟一百万次。

我们一旦部署了新的代码，服务器的数量从100台大幅 下降到大约20台。

![此处输入图片的描述][5]

我们合理配置了我们的集群和自动均衡配置之后，我们可以把服务器的数量降至4x EC2 c4.Large实例，并且Elastic Auto-Scaling设置为如果CPU达到5分钟的90%利用率，我们就会产生新的实例。

![此处输入图片的描述][6]


# 总结

在我的书中，简单总是获胜。我们可以使用多队列、后台worker、复杂的部署设计一个复杂的系统，但是我们决定利用Elasticbeanstalk 的auto-scaling的能力和Go语言开箱即用的特性简化并发。

我们仅仅用了4台机器，这并不是什么新鲜事了。可能它们还不如我的MacBook能力强大，但是却处理了每分钟1百万的写入到S3的请求。

处理问题有正确的工具。当你的 Ruby on Rails 系统需要更强大的web handler时，可以考虑下ruby生态系统之外的技术，或许可以得到更简单但更强大的替代方案。

[文章原文][7]


[1]: http://www.malwarebytes.org/
[2]: https://raw.githubusercontent.com/itfanr/articles-about-golang/master/2016-10/1-1.png
[3]: http://12factor.net/
[4]: https://raw.githubusercontent.com/itfanr/articles-about-golang/master/2016-10/1-2.png
[5]: https://raw.githubusercontent.com/itfanr/articles-about-golang/master/2016-10/1-3.png
[6]: https://raw.githubusercontent.com/itfanr/articles-about-golang/master/2016-10/1-4.png
[7]: http://marcio.io/2015/07/handling-1-million-requests-per-minute-with-golang/

## 自己设计的源码：
---
主要有三个文件:
- 一个**worker.go**真正并发工作的执行者。
- 一个**dispatch.go**这个是调度worker的。
- 另外一个是初始化和生产任务的文件。

**worker.go**:
```go
package cron
var (
	MaxWorker = 100
	MaxQueue = 500
)

// Job represents the job to be run
type Job interface {
	Do() error
}

// A buffered channel what we can send work requests on.
// var JobQueue chan Job

// Worker represents the worker that executes the job
type Worker struct {
	WorkerPool chan chan Job
	JobChannel chan Job
	quit chan bool
}

func NewWorker(workerPool chan chan Job) Worker {
	return Worker{
		WorkerPool:workerPool,
		JobChannel:make(chan Job),
		quit:make(chan bool) }
}

// Start method starts the run loop for the worker, listening for a quit channel in
// case we need to stop it
func (w Worker) Start() {
	go func() {
		for {
			// register the current worker into the worker queue.
			w.WorkerPool <- w.JobChannel

			select {
			case job := <-w.JobChannel:
				// we have  received a work request.
				if err := job.Do(); err != nil {
					// log.Println("Error do : ", err)
				}
			case <-w.quit:
				// we have received a signal to stop
				return
			}
		}
	}()
}

func (w Worker) Stop() {
	go func() {
		w.quit <- true
	}()
}
```

**dispatch.go**:
```go
package cron

type Dispatcher struct {
	// A pool of workers channels that are registered with the dispatcher
	WorkerPool chan chan Job
	maxWorkers int
}

func NewDispatcher(maxWorkers int) *Dispatcher {
	pool := make(chan chan Job, maxWorkers)
	return &Dispatcher{WorkerPool:pool, maxWorkers:maxWorkers}
}

func (d *Dispatcher) Run() {
	// starting n number of workers
	for i := 0; i < d.maxWorkers; i++ {
		worker := NewWorker(d.WorkerPool)
		worker.Start()
	}

	go d.dispatch()
}

func (d *Dispatcher) dispatch() {
	for {
		select {
		case job := <-JobQueue:
			// a job request has been received
			go func(job Job) {
				// try to obtain a worker job channel that is available.
				// this will block until a worder is idle
				jobChannel := <-d.WorkerPool

				// dispatch the job to the worker job channel
				jobChannel <- job

			}(job)
		}
	}
}
```

调用的地方**main.go**:
```go
// 我当时做原始交易用的，每一个需要处理的数据都是已经签名完成的交易指针
type Payload struct {
	signTx *types.Transaction
}

// consumer实际上就是将交易发送到区块链节点网络上,实现Job的Do接口即可。
func (p *Payload) Do() error {
	return SendTx(p.signTx)
}

// 在main.go之中初始化dispatch信息。它会自动创建worker的。
func StoreInit(maxqueue, maxworker int) {
	MaxQueue = maxqueue
	MaxWorker = maxworker
	JobQueue = make(chan *Payload, MaxQueue)

	d := NewDispatcher(MaxWorker)
	d.Run()
}

// 
func createRawTransaction() {
  // 生产一个signTx, 并且放入队列, Worker们自动会取出来处理。
	JobQueue <- &Payload{signTx}
}
```

## 参考文献

- [1 millon qps](http://marcio.io/2015/07/handling-1-million-requests-per-minute-with-golang/)
- [一百万请求](https://github.com/itfanr/articles-about-golang/blob/master/2016-10/1.handling-1-million-requests-per-minute-with-golang.md)
- [利用interface方式](https://blog.csdn.net/jeanphorn/article/details/79018205)

# 策略模式

### 一.go代码

<pre class="language-go" data-line-numbers><code class="lang-go">package templatemethod

import "fmt"

type Downloader interface {
	Download(uri string)
}

type template struct {
	implement
	uri string
}

type implement interface {
	download()
	save()
}

func newTemplate(impl implement) *template {
	return &#x26;template{
		implement: impl,
	}
}

// 这里是稳定的代码
func (t *template) Download(uri string) {
	t.uri = uri
	fmt.Print("prepare downloading\n")
	t.implement.download()
	t.implement.save()
	fmt.Print("finish downloading\n")
}

// 不要这样写，覆盖父类方法，会违反里氏替换原则
<strong>// func (t *template) save() {
</strong>//	fmt.Print("default save\n")
// }

type HTTPDownloader struct {
	*template
}

func NewHTTPDownloader() Downloader {
	downloader := &#x26;HTTPDownloader{}
	template := newTemplate(downloader)
	downloader.template = template
	return downloader
}

func (d *HTTPDownloader) download() {
	fmt.Printf("download %s via http\n", d.uri)
}

func (*HTTPDownloader) save() {
	fmt.Printf("http save\n")
}

type FTPDownloader struct {
	*template
}

func NewFTPDownloader() Downloader {
	downloader := &#x26;FTPDownloader{}
	template := newTemplate(downloader)
	downloader.template = template
	return downloader
}

func (d *FTPDownloader) download() {
	fmt.Printf("download %s via ftp\n", d.uri)
}

func (*FTPDownloader) save() {
	fmt.Printf("ftp save\n")
}
</code></pre>

使用方式如下：

```go
package templatemethod

func ExampleHTTPDownloader() {
	var downloader Downloader = NewHTTPDownloader()

	downloader.Download("http://example.com/abc.zip")
	// Output:
	// prepare downloading
	// download http://example.com/abc.zip via http
	// http save
	// finish downloading
}

func ExampleFTPDownloader() {
	var downloader Downloader = NewFTPDownloader()

	downloader.Download("ftp://example.com/abc.zip")
	// Output:
	// prepare downloading
	// download ftp://example.com/abc.zip via ftp
	// default save
	// finish downloading
}
```

### 二.特性

策略模式满足开闭原则，每当有新策略，新建一个子类即可。

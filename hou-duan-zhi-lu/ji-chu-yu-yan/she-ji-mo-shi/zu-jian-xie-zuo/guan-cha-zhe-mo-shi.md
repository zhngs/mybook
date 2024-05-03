# 观察者模式

### 一.go代码

```go
package observer

import "fmt"

type Subject struct {
	observers []Observer
	context   string
}

func NewSubject() *Subject {
	return &Subject{
		observers: make([]Observer, 0),
	}
}

func (s *Subject) Attach(o Observer) {
	s.observers = append(s.observers, o)
}

func (s *Subject) notify() {
	for _, o := range s.observers {
		o.Update(s)
	}
}

func (s *Subject) UpdateContext(context string) {
	s.context = context
	s.notify()
}

// 这里接口依赖了一个具体struct，不推荐，抽象应该依赖抽象，而不是具体的细节
type Observer interface {
	Update(*Subject)
}

type Reader struct {
	name string
}

func NewReader(name string) *Reader {
	return &Reader{
		name: name,
	}
}

func (r *Reader) Update(s *Subject) {
	fmt.Printf("%s receive %s\n", r.name, s.context)
}
```

使用方式：

```go
package observer

func ExampleObserver() {
	subject := NewSubject()
	reader1 := NewReader("reader1")
	reader2 := NewReader("reader2")
	reader3 := NewReader("reader3")
	subject.Attach(reader1)
	subject.Attach(reader2)
	subject.Attach(reader3)

	subject.UpdateContext("observer mode")
	// Output:
	// reader1 receive observer mode
	// reader2 receive observer mode
	// reader3 receive observer mode
}
```

### 二.c++代码

#### 1.版本一

```cpp
class FileSplitter {
    ProgressBar* m_progressBar; // 这里依赖具体的对象
public:
    void split() {
        // 分割过程中通知ProgressBar
    }
};

class MainForm : public Form {
    ProgressBar* progressBar;

public:
    void Button1_Click() {	
	FileSplitter splitter(progressBar);
	splitter.split();
    }
};
```

#### 2.版本二

```cpp
class IProgress{
public:
    virtual void DoProgress(float value)=0;
    virtual ~IProgress(){}
};


class FileSplitter {
    List<IProgress*>  m_iprogressList;
	
public:
    void split(){
	// 分割过程中调用onProgress，这里变成了稳定的代码
    }

    void addIProgress(IProgress* iprogress) {
	m_iprogressList.push_back(iprogress);
    }

    void removeIProgress(IProgress* iprogress) {
	m_iprogressList.remove(iprogress);
    }

protected:
    virtual void onProgress(float value) {
	List<IProgress*>::iterator itor=m_iprogressList.begin();
	while (itor != m_iprogressList.end()) {
	    (*itor)->DoProgress(value);
	    itor++;
	}
    }
};

class MainForm : public Form, public IProgress {
    ProgressBar* progressBar;
public:
    void Button1_Click(){
	ConsoleNotifier cn;
	FileSplitter splitter;
	splitter.addIProgress(this);
	splitter.addIProgress(&cn);
	splitter.split();
	splitter.removeIProgress(this);

    virtual void DoProgress(float value){
	progressBar->setValue(value);
    }
};

class ConsoleNotifier : public IProgress {
public:
    virtual void DoProgress(float value) {
	cout << ".";
    }
};
```

### 三.特性

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

解决问题：观察者模式将数据生产者和消费者隔离开，当有多个消费者时，满足开闭原则。

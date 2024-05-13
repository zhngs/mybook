# 模板方法

### 一.c++代码

#### 1.版本一

```cpp
class Application {
  public:
	bool Step2() {
		cout << "myStep2" << endl;
		return true;
	}

	void Step4() {
		cout << "myStep4" << endl;
	}
};

int main() {
	Library lib;
	Application app;

	lib.Step1();

	if (app.Step2()) {
		lib.Step3();
	}

	for (int i = 0; i < 4; i++) {
		app.Step4();
	}

	lib.Step5();
}
```

#### 2.版本二

```cpp
class Library {
  public:
    // 这里是稳定的代码
    void Run() {
        Step1();
        if (Step2()) {
            Step3();
        }
        for (int i = 0; i < 4; i++) {
            Step4();
        }
        Step5();
    }
    virtual ~Library() {}

  protected:
    void Step1() {
        cout << "Step1" << endl;
    }
    void Step3() {
        cout << "Step3" << endl;
    }
    void Step5() {
        cout << "Step5" << endl;
    }

    virtual bool Step2() = 0; // 等待被实现
    virtual void Step4() = 0; // 等待被实现
};

//应用程序开发人员
class Application : public Library {
  protected:
    virtual bool Step2() {
        //... 子类重写实现
    	cout << "override Step2" << endl;
    	return true;
    }

    virtual void Step4() {
	//... 子类重写实现
	cout << "override Step4" << endl;
    }
};

int main() {
    Library *pLib = new Application();
    pLib->Run();
    delete pLib;
}
```

### 二.特性

<figure><img src="../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

解决问题：将某些具体逻辑延迟放在子类中实现。

模板方法是最简单的多态实现，满足开闭原则，满足依赖倒置原则。

go没办法直观地实现模板方法，可以将run和step分成两个接口来实现。

go有几点需要注意，从初始化和方法调用角度来看：

* 匿名字段是struct，且不是指针，可以看作泛化关系，也可以看作组合+泛化关系。
* 匿名字段是struct，且是指针，可以看作聚合+泛化关系。
* 匿名字段是interface，一般接口中的类型是指针，可以看作聚合+实现关系。


# 抽象工厂

### 一.go代码

```go
package abstractfactory

import "fmt"

// OrderMainDAO 为订单主记录
type OrderMainDAO interface {
	SaveOrderMain()
}

// OrderDetailDAO 为订单详情纪录
type OrderDetailDAO interface {
	SaveOrderDetail()
}

// DAOFactory DAO 抽象模式工厂接口
type DAOFactory interface {
	CreateOrderMainDAO() OrderMainDAO
	CreateOrderDetailDAO() OrderDetailDAO
}

// RDBMainDAO 关系型数据库的OrderMainDAO实现
type RDBMainDAO struct{}

// SaveOrderMain ...
func (*RDBMainDAO) SaveOrderMain() {
	fmt.Print("rdb main save\n")
}

// RDBDetailDAO 为关系型数据库的OrderDetailDAO实现
type RDBDetailDAO struct{}

// SaveOrderDetail ...
func (*RDBDetailDAO) SaveOrderDetail() {
	fmt.Print("rdb detail save\n")
}

// RDBDAOFactory 是RDB 抽象工厂实现
type RDBDAOFactory struct{}

func (*RDBDAOFactory) CreateOrderMainDAO() OrderMainDAO {
	return &RDBMainDAO{}
}

func (*RDBDAOFactory) CreateOrderDetailDAO() OrderDetailDAO {
	return &RDBDetailDAO{}
}

// XMLMainDAO XML存储
type XMLMainDAO struct{}

// SaveOrderMain ...
func (*XMLMainDAO) SaveOrderMain() {
	fmt.Print("xml main save\n")
}

// XMLDetailDAO XML存储
type XMLDetailDAO struct{}

// SaveOrderDetail ...
func (*XMLDetailDAO) SaveOrderDetail() {
	fmt.Print("xml detail save")
}

// XMLDAOFactory 是XML 抽象工厂实现
type XMLDAOFactory struct{}

func (*XMLDAOFactory) CreateOrderMainDAO() OrderMainDAO {
	return &XMLMainDAO{}
}

func (*XMLDAOFactory) CreateOrderDetailDAO() OrderDetailDAO {
	return &XMLDetailDAO{}
}
```

使用方式如下：

```go
package abstractfactory

// 这是稳定的代码
func getMainAndDetail(factory DAOFactory) {
	factory.CreateOrderMainDAO().SaveOrderMain()
	factory.CreateOrderDetailDAO().SaveOrderDetail()
}

func ExampleRdbFactory() {
	var factory DAOFactory
	factory = &RDBDAOFactory{}
	getMainAndDetail(factory)
	// Output:
	// rdb main save
	// rdb detail save
}

func ExampleXmlFactory() {
	var factory DAOFactory
	factory = &XMLDAOFactory{}
	getMainAndDetail(factory)
	// Output:
	// xml main save
	// xml detail save
}
```

### 二.c++代码

#### 1.版本一

```cpp
class EmployeeDAO { 
public:
    vector<EmployeeDO> GetEmployees(){
        SqlConnection* connection = new SqlConnection();
        connection->ConnectionString = "...";
        
        SqlCommand* command = new SqlCommand();
        command->CommandText="...";
        command->SetConnection(connection);

        SqlDataReader* reader = command->ExecuteReader();
        while (reader->Read()){
            // ...
        }
    }
};
```

#### 2.版本二

```cpp
//数据库访问有关的基类
class IDBConnection {};
class IDBConnectionFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
};

class IDBCommand {};
class IDBCommandFactory{
public:
    virtual IDBCommand* CreateDBCommand()=0;
};

class IDataReader {};
class IDataReaderFactory{
public:
    virtual IDataReader* CreateDataReader()=0;
};

//支持SQL Server
class SqlConnection: public IDBConnection {};
class SqlConnectionFactory:public IDBConnectionFactory {};

class SqlCommand: public IDBCommand {};
class SqlCommandFactory:public IDBCommandFactory {};

class SqlDataReader: public IDataReader {};
class SqlDataReaderFactory:public IDataReaderFactory {};

//支持Oracle
class OracleConnection: public IDBConnection {};
class OracleCommand: public IDBCommand {};
class OracleDataReader: public IDataReader{};

class EmployeeDAO{
    IDBConnectionFactory* dbConnectionFactory;
    IDBCommandFactory* dbCommandFactory;
    IDataReaderFactory* dataReaderFactory;
    
public:
    // 这里使用了三个factory，虽然这里的代码是稳定的，但是这三个factory实际上是有关联的
    vector<EmployeeDO> GetEmployees() {
        IDBConnection* connection = dbConnectionFactory->CreateDBConnection();
        connection->ConnectionString("...");

        IDBCommand* command = dbCommandFactory->CreateDBCommand();
        command->CommandText("...");
        command->SetConnection(connection); //关联性

        IDBDataReader* reader = command->ExecuteReader(); //关联性
        while (reader->Read()){
            // ...
        }

    }
};
```

#### 3.版本三

```cpp
//数据库访问有关的基类
class IDBConnection {};
class IDBCommand {};
class IDataReader {};
// 这里将三个不同的抽象基类放在同一个factory中，这三个抽象基类必须是相互关联的
class IDBFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
    virtual IDBCommand* CreateDBCommand()=0;
    virtual IDataReader* CreateDataReader()=0;
};

//支持SQL Server
class SqlConnection: public IDBConnection {};
class SqlCommand: public IDBCommand {};
class SqlDataReader: public IDataReader {};
class SqlDBFactory:public IDBFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
    virtual IDBCommand* CreateDBCommand()=0;
    virtual IDataReader* CreateDataReader()=0;
 
};

//支持Oracle
class OracleConnection: public IDBConnection {};
class OracleCommand: public IDBCommand {};
class OracleDataReader: public IDataReader {};

class EmployeeDAO{
    IDBFactory* dbFactory;
public:
    vector<EmployeeDO> GetEmployees(){
        IDBConnection* connection = dbFactory->CreateDBConnection();
        connection->ConnectionString("...");

        IDBCommand* command = dbFactory->CreateDBCommand();
        command->CommandText("...");
        command->SetConnection(connection); //关联性

        IDBDataReader* reader = command->ExecuteReader(); //关联性
        while (reader->Read()){
            // ...
        }

    }
};
```

### 三.特性

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

解决问题：是对工厂方法进行拓展，支持多类产品。

缺点：

* 不支持增加新类型的产品，否则需要更改工厂接口。

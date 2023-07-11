# 工作日志——sgh

## 主要实现

### 支付的两种方式——退钱几乎同理

支付宝支付

银行卡支付

TIPS:我目前的想法是，一个用户都是使用同一个账户，只是通过不同的支付途径来扣取钱支付车票，类似于用银行卡支付用的是学校银行卡的钱，支付宝支付也是直接使用银行卡的钱。

* 实现思路：使用策略模式
* 检验方法：比较丑陋，使用的是日志的方式，记录每一次交易的结果

我把Account类归属在了UserEntity下，也就是每一个使用该系统的用户必须绑定一个自己的account；把策略类放在了OrderEntity下面（但是后面想一想好像没有必要保存策略的类型，我们应该是给两个选项让用户选择用哪个，然后我们直接保存并使用？）

* 注意：account没有持久化，它只是作为一个属性归属在userEntity下，不像其他的属性，会有数据库对其进行存储，当然，如果想的话，可以实现记录一个int型的余额，足够支持项目要求。

Account类具体实现就是

```
public class Account {
    private String accountNumber; // 卡号
    private double balance; // 余额

    public Account(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }
	// 获取卡号
    public String getAccountNumber();
    
    // 相当于对卡进行充值
    public void recharge(double balance)

	// 获取余额
    public double getBalance() 
    
    // 支出
    public void deduct(double amount) throws InsufficientFundsException 

	// 退回
    public void refund(double amount) 
    
    // 写账户日志
    public void writeToFile(String message) 
}
```

具体的方法类就不展示了，比较好理解

* 唯一需要注意的一点就是：考虑到对account的修改要保存下去，所以策略对象使用了account之后，还需要让userEntity对象再setAccount操作完之后的account对象。
* 还有一个InsufficientFundsException类是用于处理卡内余额不支持支付的输出的。

* 额外的注意点（在XXXEntity文件中）

  ```
  @Transient
      // 此处表示不添加到数据库中
      private Account account;
  ```

### 实现了里程积分

实现的内部有些粗糙，需要修改！！！

首先就是在UserEntity中添加这样一个对象，并且设置它是int，所以可以放到数据库中

* 注意：若想要放到数据库中，需要修改一个文件（userServiceimp，在其中的register函数中要同理加上MileagePoints（0），将其初始化为0，否则会一直是null）

然后就是MileagePoints类实现对应的积分的折扣，这边使用表驱动。这个类的主要作用就是返回一个rate。

* 注意：就是这边有点粗糙，我是直接使用的积分在某个区间打某个折扣，但是实际要求似乎不是

测试：还是用的日志的方法测试的。

原价100，积分0，少10%，实际消费90，应该没有问题

```
Recharge $10000.0 to account 10086
Account 10086 has 10000.0
卡号为10086的用户使用支付宝支付：90.0 元
Deducted $90.0 from account 10086
Account 10086 has 9910.0
```

### 最离谱的就是这个日志系统

简言之就是一个.txt文件用于记录用户的账户进行交易的相关信息

* 采用这种方法比较简单，可用于测试，但是实际的使用还需要看

### 存在的问题和接下来需要实现的部分

* 这个.txt记录的方式应该是不完整的
* 是否需要持久化account，还是我们就持久化一个余额，或者说新建一个关于账户的数据库
* 前端代码部分，在支付时应该是点击支付之后给用户选择什么支付方式，用户支付之后，会停在一个界面，此时可支持退款【任重道远】
* 暂时就这些。
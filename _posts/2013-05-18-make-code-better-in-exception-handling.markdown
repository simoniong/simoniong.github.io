---
layout: post
title: "Make code better in Exception handling"
date: 2013-05-18 15:04
comments: true
categories: [ Clean Code ]
---

This is the first post for "make code better" series, which will focus on how to write code in a clean way that every human-programmer can easily understand, maintain and extend.

{% blockquote %}
The purpose of writing better code is to reduce WTFs/min during code review session
{% endblockquote %}
{% img http://www.osnews.com/images/comics/wtfm.jpg %}

If the code being written is really bad and difficult to read, then this funny scenario will always happend during code review session. If there's no code-review session in your company culture, somebody will still put long fingers to authors while handling your legacy messy-and-shitty code.

## Purpose of Exception hanlding
According to wikipedia, error handling is an approach to handled process by saving the current state of execution in a predefined place and switching the execution to a specific subroutine known as an exception handler. 

In genernal, the purpose of Exception handling is to seperate concern on different flow -- execution flow and exception handling flow. 

Unfortunatly, there's a lot of anti-use-pattern cases, which make this two flows mixed-up, and lead to the code being difficult to read.

## Anti-use-pattern of Exception handling
### return error code instead of raise exception
In some early stages programming languages, such as C, there's no exception mechanism, one need to check the return code after calling methods, and determine what need to do in next step. After introduce exception handling, people keep writting code in this ancient way. Following code is a typical anti-use example.

``` ruby atm.rb
class ATM
  def initialize( terminal )
    @terminal = terminal 
  end

  def withdraw( amount )
    error_code = @terminal.withdraw( amount )
    return 0 if error_code == 'ok'

    if error_code == 'E32 code'
      log "E32 Error happend"
    elsif error_code == 'DAB code lost'
      log "DAB code happend"
    elsif error_code == 'network lost'
      log "network lost"
    elsif error_code == 'not enough balance'
      log "not enough blance in main account"
    else
      log "Invalid error happend"
    end
    return 1
  end
end
```

Checking error_code make code structure messy, while people start reading withdraw method, he need to read through every error_code, and figure out WTF is that errors and why withdraw need to handling this kind of error. But the main purpose of this method is to withdraw menoy. Have you lost the focus on this "withdraw" method? And how long does it take you need to read the error_code part, and understand you don't need it in the mean time?

Here's the little refactor after introduce exception handling part.

``` ruby
class ATM
  def initialize( terminal )
    @terminal = terminal 
  end

  def withdraw( amount )
    begin
      @terminal.withdraw( amount )
    rescue E32Error => e
      log "E32 Error happend"
    rescue DABError => e
      log "DAB code happend"
    rescue NetworkLostError => e
      log "network lost"
    rescue NotEnoughBalanceError => e
      log "not enough blance in main account"
    rescue Exception => e
      log "Invalid error happend"
    end
  end

end
```

``` ruby terminal.rb
class Terminal
  def withdraw( withdraw_amount )
    ... 
    try_to_withdraw( withdraw_amount )
    ...
    raise E32Error.new if state == 'E32'
    raise DABError.new if state == 'DAB'
    raise NetworkLostError if state == 'connection lost'
    raise NotEngoughBalanceError if account.balance < withdraw_amount
    ...
  end
end
```

A little better right now, after reading this withdraw method, It's easy to seperate the main logic and error handling part. People even don't need to read exception handling part if he is only focus on how to do the withdraw ( @terminal object's withdraw method ). And the exception handling part can be even better with a little refactor by moving out exception logic into a seperate method.

``` ruby
class ATM
  def initialize( terminal )
    @terminal = terminal 
  end

  def withdraw( amount )
    begin
      @terminal.withdraw( amount )
    rescue Exception => e
      exception_handling_for_withdraw( e )
    end
  end

  protected

  def exception_handling_for_withdraw( exception )
    error_class = exception.class.name
    case error_class
    when 'E32Error'
      log "E32 Error happend"
    when 'DABError'
      log "DAB code happend"
    when 'NetworkLostError'
      log "network lost"
    when 'NotEnoughBalanceError'
      log "not enough blance in main account"
    else
      log "Invalid error happend"
    end
  end

end

```

### raise exception in receiver's perspective

Back to our ATM example, do you have some question mark while reading method "exception_handling_for_withdraw"? Someone may ask: What does E32Error mean? Why ATM need to take care of DABError? WTF is DABError? The mistake here is to raise exceptions in receiver's perspective. But actually, the better way is to raise the exception in the caller's perspective. Here's a refactor version of writing the code in caller ( ATM )'s perspective.

``` ruby atm.rb
class ATM
  def withdraw( amount )
    begin
      @terminal.withdraw( amount )
    rescue Exception => e
      exception_handling_for_withdraw( e )
    end
  end

  protected

  def exception_handling_for_withdraw( exception )
    error_class = exception.class.name
    case error_class 
    when 'NotEnoughBalanceError'
      log "not enough blance in main account"
    when 'TerminalError'
      log "terminal error happend"
    end
  end

end
```

``` ruby terminal.rb
class Terminal
  def withdraw( amount )
    begin
      ...
      try_to_withdraw
      ...
      raise E32Error.new if state == 'E32'
      raise DABError.new if state == 'DAB'
      raise NotEngoughBalanceError if account.balance < amount
      ...
    rescue NotEnoughBalanceError => e
      raise e
    rescue Exception => e
      raise TerminalError.new( 
    end
  end
end
```

After this refactor, exception_handling_for_withdraw only need to concern two kind of errors, the error that current account with not enough balance, and any other errors that may happend in the terminal side, which is no big deal to ATM side.

There's another bonus when we finish this refactor -- the ability to define and seperate the interface of internal system ( ATM ) and external system ( Terminal ). Any Terminal class, which agree to implemented withdraw interface and raise only two kind of Exception ( NotEnoughBalanceError, and TerminalError ), can easiy port to the ATM system. If Terminal part is change, it will not affect ATM system as long as the interface is not change. and the ATM system can easily change it's terminal in the near future, for example, it can use a LandBaseTerminal to deal with Network problem, or sometime later, change to some WifiTerminal without any cost.




If you have any questions, feel free to drop down your comments.

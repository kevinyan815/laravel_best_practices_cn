### 单一责任原则

一个方法应该只有一个职责，一个强大的功能由多个单一职责的方法组合而成，而不是把所有功能都写到一个方法里，如果那样的话方法复用的可能性几乎为零，并且难以维护。

一个类同样应该是针对单一事物的抽象，里边包含的方法和属性应该只与类代表的事物有关。

错误：

```
public function getFullNameAttribute()
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

正确：

```
public function getFullNameAttribute()
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient()
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong()
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort()
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

同时单一责任原则也符合关注点分离原则的设计思路，先将复杂问题做合理的分解,再分别仔细研究问题的不同侧面(关注点),最后综合各方面的结果,合成整体的解决方案。

---
title: "Automapper_flattern"
date: 2022-10-20T16:49:14+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "automapperflattern"
---

# automapper flattern
- 可以對應到Customer 下的Name
- 來源沒有Total的話會用GetTotal()方法的值
```C#
void Main()
{
	var configuration = new MapperConfiguration(cfg =>
	{
		cfg.CreateMap<Order, OrderDto>();
		cfg.CreateMap<Test, Order>();
	});
	var mapper = configuration.CreateMapper();
	
	var customer = new Customer () { Name = "Tom" };
	var temp = new Test {Customer = customer,Total=80M};
	var order = mapper.Map<Order>(temp);
	
	var test=mapper.Map<OrderDto>(order);
	test.Dump();
}
public class Order
{
	public Customer Customer { get; set; }

	public decimal Total{get;set;}
	public decimal GetTotal()
	{
		return 100M;
	}
}
public class Customer
{
	public string Name { get; set; }
}
public class OrderDto
{
	public string CustomerName { get; set; }
	public decimal Total { get; set; }
}
public class Test
{
	public Customer Customer { get; set; }

	public decimal Total{get;set;}
}

```
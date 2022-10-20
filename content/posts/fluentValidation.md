---
title: "FluentValidation作法"
date: 2022-10-20T16:55:10+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "FluentValidation作法"
---

# .Net Core FluentValidation作法

### [文件](https://docs.fluentvalidation.net/en/latest/installation.html)
### 安裝套件
```
Install-Package FluentValidation.AspNetCore
```
### startUp.cs設定
會去偵測組件所在位置所有繼承AbstractValidator的類別
```
 services.AddControllers(options =>
            { 
            })
                .AddJsonOptions(options => options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter()))
                .AddFluentValidation(fv => fv.RegisterValidatorsFromAssemblyContaining<Startup>());
```
參考這篇[文章](https://dotblogs.com.tw/shadowkk/2019/07/23/140127)
後面需加上把model validator的驗證關掉
```
 services.Configure<ApiBehaviorOptions>(options =>
            {
               options.SuppressModelStateInvalidFilter = true;
            });
```
還有另一個解法是用[stackoverflow](https://stackoverflow.com/questions/46374994/correct-way-to-disable-model-validation-in-asp-net-core-2-mvc)的作法，但其實會有問題
踩到的雷是[FromBody]不會有問題，但用[HttppGet]打api還是會驗證
要依照[連結](https://github.com/dotnet/aspnetcore/issues/30938)的做法去調整
```C#
 public class NullObjectModelValidator : IObjectModelValidator
    {
        public void Validate(ActionContext actionContext,
            ValidationStateDictionary validationState, string prefix, object model)
        {
            foreach (var entry in actionContext.ModelState.Values)
            {
                // or ModelValidationState.Valid
                entry.ValidationState = ModelValidationState.Skipped;
            }
        }
    }
```
### 寫Validator
原則上可以參考文件寫法，有很多客製跟條件寫法
```
public class AddMessageParameterValidator : AbstractValidator<AddMessageParameter>
    {
        public AddMessageParameterValidator()
        {
            

            RuleFor(c => c.Phone)
                .NotNull()
                .WithName("手機號碼")
                .WithMessage("請輸入{PropertyName}")
                .NotEmpty()
                .WithMessage("請輸入{PropertyName}")
                .Must(phone => Regex.IsMatch(phone ?? "", @"^[\d]{10}$"))
                .WithMessage("{PropertyName}格式不正確");

            RuleFor(c => c.Name)
                .NotNull()
                .WithName("姓名")
                .WithMessage("請輸入{PropertyName}")
                .NotEmpty()
                .WithMessage("請輸入{PropertyName}")
                .Must(name => Regex.IsMatch(name ?? "", @"^[^\W_]+$")) //排除特殊字元
                .WithMessage("姓名格式不正確");

            RuleFor(c => c.RentId)
                .NotNull()
                .WithName("物件編號")
                .WithMessage("請輸入{PropertyName}")
                .NotEmpty()
                .WithMessage("請輸入{PropertyName}");

            RuleFor(c => c.IPAddress)
                .Must(ip => Regex.IsMatch(ip ?? "", @"\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"))
                .When(c => string.IsNullOrEmpty(c.IPAddress) is false)
                .WithName("IP")
                .WithMessage("{PropertyName}格式有誤");
        }
    }
```

### 驗證的ActionFilter
回傳格式依照專案規則訂定
```
public class CustomValidatorAttribute : ActionFilterAttribute
    {
        private readonly Type _validatorType;

        /// <summary>
        /// Initializes a new instance of the <see cref="CustomValidatorAttribute"/> class.
        /// </summary>
        /// <param name="validatorType">Type of the validator.</param>
        public CustomValidatorAttribute(Type validatorType)
        {
            this._validatorType = validatorType;
        }

        public override async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
            var parameters = context.ActionArguments;
            if (parameters.Count <= 0)
            {
                await base.OnActionExecutionAsync(context, next);
            }

            var parameter = parameters.FirstOrDefault();
            if (parameter.Value == null)
            {
                context.Result = new BadRequestObjectResult("未輸入 Parameter");
            }
             var validator = context.HttpContext.RequestServices.GetService(this._validatorType) as IValidator;
            //var validator = Activator.CreateInstance(this._validatorType) as IValidator;
            var validationContext = new ValidationContext<object>(parameter.Value);
            var validationResult = await validator.ValidateAsync(validationContext);

            if (validationResult.IsValid.Equals(false))
            {
                var error = validationResult.Errors.FirstOrDefault();
                var failResult = new FailResultViewModel
                {
                    Method = $"{context.HttpContext.Request.Path} - {context.HttpContext.Request.Method}",
                    Status = "ValidationError",
                    Error = new FailInformation
                    {
                        ErrorCode = 30001,
                        Message = "輸入資料驗證錯誤",
                        Description = error.ErrorMessage
                    }
                };

                context.Result = new BadRequestObjectResult(failResult);
            }

            await base.OnActionExecutionAsync(context, next);
        }
    }
```
